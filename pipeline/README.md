
## Pipeline Scripts

### `Ingestion.py`

`Ingestion.py` handles the raw Adzuna API ingestion step. It loads API credentials from a local credentials file, iterates through configured job categories, and sends paginated requests to the Adzuna search endpoint using a focused keyword query such as `computer science`.

Each API response is parsed from the `results` field. Job records are deduplicated by job ID, tagged with ingestion metadata such as timestamp and category, and written to a local raw output file. The script also logs request URLs and response information so API failures, pagination issues, and rate limiting are easier to troubleshoot.

The output of this step is the raw `jobs_*` dataset. This file represents the direct API response data before scraping, enrichment, cleanup, or database upload.

### `Data_Enrichment Slow Crawler.py`

`Data_Enrichment.py` takes the raw `jobs_*` file and adds additional fields needed by the backend search and ranking flow. For each job record, the script reads the `redirect_url`, sends an HTTP request, resolves the final URL, and attempts to parse the returned HTML with BeautifulSoup.

When the redirect resolves to an Adzuna job details page, the page usually contains a usable `section.adp-body` element. The script extracts this section, stores the raw description HTML, and creates a cleaned plain-text job description. This provides a much richer description than the original API response.

The script also adds metadata and derived fields, including `state` from `location.area`, resolved URL, status code, content type, and flags for missing or unusable descriptions.

External employer redirects are less reliable. When the redirect leaves Adzuna and lands on an employer site, the request is more likely to be blocked, rate limited, or return HTML that does not match the expected structure. These cases are tracked as failed, blocked, or missing-description records instead of being treated as successful enrichments.

The enrichment process also uses a checkpointing system so long runs do not have to restart from the beginning if the script stops, gets blocked, or fails partway through. As each job is processed, progress is saved locally, and enriched records are written incrementally instead of only being written at the end. This makes the crawler more reliable because the script can resume from the last saved position and avoid reprocessing records that were already completed.

The enriched records are written to a separate `enriched_*` output file so the raw ingestion file remains unchanged and enrichment can be rerun safely.






Below is a sample response from the Adzuna API, and then after is the api response schema. The top section contains metadata about the request, including the total number of records available. The mean field represents the average salary across all returned records.
Individual job records are located under the "results" field.
If salary_is_predicted = 1, the salary value is estimated by Adzuna rather than provided directly.
Adzuna returns up to 50 results per page. Broad searches across the three relevant categories can return 500,000+ results, so it’s important to narrow queries using the what parameter.

Example using the what parameter:
https://api.adzuna.com/v1/api/jobs/us/search/1?app_id=8e88dd1a&app_key=501524232a49925c03eec7a276d7fb5f&results_per_page=50&category=scientific-qa-jobs&what=computer+science

{
  "__CLASS__": "Adzuna::API::Response::JobSearchResults",
  "count": 223050,
  "mean": 154364.27,
  "results": [
    {
      "__CLASS__": "Adzuna::API::Response::Job",
      "id": "5701856729",
      "title": "IAM Engineer III",
      "description": "About the Job: We are seeking an IAM Engineer to join our Identity Governance & Administration (IGA) team to design, build, and operate identity governance capabilities, while developing and supporting Java-based applications and services that enable IAM at scale. The ideal candidate has hands-on experience with SailPoint (IdentityIQ and/or ISC) and strong Java development skills to build and maintain custom software solutions. What's the Job: Implement and maintain IGA capabilities including l…",
      "created": "2026-04-16T16:47:43Z",
      "adref": "eyJhbGciOiJIUzI1NiJ9.eyJpIjoiNTcwMTg1NjcyOSIsInMiOiIxSklJSGQwNThSR2M5XzFOc1hKeWRRIn0.wKPKoiwr6yLkJNvCIklM7e7IT-zraVWKtBLoT57hNyU",
      "redirect_url": "https://www.adzuna.com/land/ad/5701856729?se=1JIIHd058RGc9_1NsXJydQ&utm_medium=api&utm_source=8e88dd1a&v=3C8A519DC98ED7FD16382ACDC55305FCF6D0FACB",
      "salary_min": 164398.53,
      "salary_max": 164398.53,
      "salary_is_predicted": "1",
      "latitude": 43.053096,
      "longitude": -87.932603,
      "company": {
        "__CLASS__": "Adzuna::API::Response::Company",
        "display_name": "Northwestern Mutual"
      },
      "category": {
        "__CLASS__": "Adzuna::API::Response::Category",
        "label": "IT Jobs",
        "tag": "it-jobs"
      },
      "location": {
        "__CLASS__": "Adzuna::API::Response::Location",
        "display_name": "Milwaukee, Milwaukee County",
        "area": [
          "US",
          "Wisconsin",
          "Milwaukee County",
          "Milwaukee"
        ]
      }
    }
  ]
}




SCHEMA: 
{
  "type": "object",
  "properties": {
    "location": {
      "type": "object",
      "properties": {
        "area": {
          "type": "array",
          "items": {
            "type": "string"
          }
        },
        "display_name": {
          "type": "string"
        },
        "__CLASS__": {
          "type": "string"
        }
      },
      "required": [
        "area",
        "display_name",
        "__CLASS__"
      ]
    },
    "company": {
      "type": "object",
      "properties": {
        "__CLASS__": {
          "type": "string"
        },
        "display_name": {
          "type": "string"
        }
      },
      "required": [
        "__CLASS__"
      ]
    },
    "created": {
      "type": "string",
      "format": "date-time"
    },
    "adref": {
      "type": "string"
    },
    "title": {
      "type": "string"
    },
    "redirect_url": {
      "type": "string",
      "format": "uri"
    },
    "salary_max": {
      "type": "number"
    },
    "salary_min": {
      "type": "number"
    },
    "id": {
      "type": "string"
    },
    "salary_is_predicted": {
      "type": "string"
    },
    "description": {
      "type": "string"
    },
    "__CLASS__": {
      "type": "string"
    },
    "category": {
      "type": "object",
      "properties": {
        "label": {
          "type": "string"
        },
        "tag": {
          "type": "string"
        },
        "__CLASS__": {
          "type": "string"
        }
      },
      "required": [
        "label",
        "tag",
        "__CLASS__"
      ]
    },
    "TIMESTAMP": {
      "type": "string",
      "format": "date-time"
    },
    "Category": {
      "type": "string"
    },
    "contract_time": {
      "type": "string"
    },
    "contract_type": {
      "type": "string"
    },
    "latitude": {
      "type": "number"
    },
    "longitude": {
      "type": "number"
    }
  },
  "required": [
    "location",
    "company",
    "created",
    "adref",
    "title",
    "redirect_url",
    "salary_max",
    "salary_min",
    "id",
    "salary_is_predicted",
    "description",
    "__CLASS__",
    "category",
    "TIMESTAMP",
    "Category"
  ]
}
