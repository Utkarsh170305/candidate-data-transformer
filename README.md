# Candidate Data Transformer

## Overview

This is a command-line application that transforms recruiter CSV data and resume text files into consolidated candidate profiles through extraction, normalization, matching, merging, projection, and validation. The tool matches candidate records across multiple input sources, reconciles duplicate or conflicting data based on confidence metrics, and outputs clean candidate JSON profiles.

---

## Requirements

- Python 3.11 or later
- Standard library modules only (no external library dependencies)

---

## Installation

```bash
git clone https://github.com/Lakshya-developer24/Candidate-Data-Transformer.git
cd Candidate-Data-Transformer
```

---

## Project Structure

```text
candidate-data-transformer/
├── candidate_transformer/
├── configs/
├── docs/
├── input/
│   ├── recruiter/
│   └── resumes/
├── output/
└── tests/
```

---

## Input Format

### Recruiter CSV

The recruiter CSV export must contain candidate rows with headers representing candidate properties.

Expected columns:
- `name`: Candidate's full name.
- `email`: Candidate's primary email address (used for matching).
- `phone`: Candidate's contact phone number.
- `company`: Latest employer company name.
- `title`: Latest job title.

Sample CSV content:
```csv
name,email,phone,company,title
Jane Doe,Jane.Doe@Example.com,(415) 555-0199,Acme Corp,Senior Software Engineer
```

### Resume Files

- Resumes must be UTF-8 encoded text files with the extension `.txt`.
- There must be exactly one candidate's resume per file.

Sample resume content (`jane.txt`):
```text
Jane Doe
Senior Backend Engineer

SUMMARY
Backend engineer with 6 years of experience building distributed systems.

SKILLS
Python, AWS, Docker, Kubernetes, JS

EXPERIENCE
Acme Corp — Senior Software Engineer
Jan 2021 - Present

Globex Inc — Software Engineer
Jun 2018 - Dec 2020

EDUCATION
B.S. Computer Science, State University, 2018

CONTACT
jane.doe@example.com
+91 415 555 0199
```

### Configuration File

The configuration file is optional. It controls field filtering, key renaming, confidence metrics inclusion, provenance metadata mapping, and missing-value validation behavior.

Sample config.json:
```json
{
  "fields": [
    {
      "from": "full_name",
      "to": "name"
    },
    {
      "from": "emails[0]",
      "to": "email"
    },
    {
      "from": "skills",
      "to": "skills"
    }
  ],
  "include_confidence": false,
  "include_provenance": false,
  "on_missing": "null"
}
```

---

## Running the Project

### Using the default configuration

If the `--config` argument is omitted, the application uses the built-in default configuration and projects all supported fields.

```bash
python3 -m candidate_transformer.cli \
    --csv input/recruiter/recruiter.csv \
    --resumes input/resumes \
    --out output
```

### Using a custom configuration

To customize field mapping, metadata inclusion, or missing value handling, provide a configuration file.

```bash
python3 -m candidate_transformer.cli \
    --csv input/recruiter/recruiter.csv \
    --resumes input/resumes \
    --config configs/custom_config.json \
    --out output
```

---

## Sample Run

Terminal output:
```text
Processed 2 candidates: 1 succeeded, 1 failed. See output/profiles.json and output/failed_candidates.json.
```

---

## Sample Output (Default Configuration)

Sample `profiles.json` (default configuration):

```json
[
  {
    "candidate_id": "cand_86e0b9e56c17",
    "full_name": {
      "value": "Jane Doe",
      "confidence": 1.0,
      "source": [
        "recruiter_csv",
        "resume_txt"
      ],
      "method": "agreed"
    },
    "emails": [
      {
        "value": "jane.doe@example.com",
        "confidence": 1.0,
        "source": [
          "recruiter_csv",
          "resume_txt"
        ],
        "method": "agreed"
      }
    ],
    "phones": [
      {
        "value": "+14155550199",
        "confidence": 0.95,
        "source": "recruiter_csv",
        "method": "csv_direct"
      }
    ],
    "headline": {
      "value": "Senior Backend Engineer",
      "confidence": 0.6,
      "source": "resume_txt",
      "method": "resume_inferred"
    },
    "skills": [
      {
        "value": "Python",
        "confidence": 0.8,
        "source": "resume_txt",
        "method": "resume_keyword"
      },
      {
        "value": "AWS",
        "confidence": 0.8,
        "source": "resume_txt",
        "method": "resume_keyword"
      }
    ],
    "overall_confidence": 0.83
  }
]
```

Sample `failed_candidates.json`:

```json
[
  {
    "identifier": "bad@example.com",
    "source": "recruiter_csv",
    "stage": "extraction",
    "reason": "CSV row is missing required field 'name'"
  }
]
```

---

## Output Files

- `profiles.json`: Stores the successfully processed, merged, projected, and validated candidate profiles.
- `failed_candidates.json`: Stores candidate-level processing failures containing failure stage and details.

---

## Running the Tests

```bash
python3 -m unittest discover -s tests/unit

python3 -m unittest discover -s tests/integration
```

---

## Notes

- Resume files must be `.txt` UTF-8 formatted text files.
- Candidate IDs are generated automatically using sha256 hashes of identifiers.
- Output files are recreated on every run.
- Empty runtime directories are preserved using `.gitkeep`.
- Configuration controls projected output fields.
