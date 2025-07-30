# 🔍 Bright Security API Scan GitHub Action

This GitHub Action performs an **incremental API vulnerability scan** using [Bright Security](https://brightsec.com/) against an OpenAPI/Swagger file. It supports **optional Repeater usage** for scanning internal or non-public environments, making it suitable for both external and internal API assessments.

## 📦 What This Workflow Does

- Uploads an OpenAPI/Swagger spec to Bright
- Optionally uses a Repeater (if provided)
- Runs a discovery process to identify entrypoints
- Waits for discovery completion
- Retrieves discovered entrypoints
- Initiates a vulnerability scan using the Bright CLI

## 🧩 Inputs

| Name         | Description                                  | Required | Type   | Default               |
|--------------|----------------------------------------------|----------|--------|------------------------|
| `archive_path` | Path to the OpenAPI/Swagger file (e.g., `./swagger.json`) | ✅ Yes | `string` | N/A |
| `scan_name`  | Optional name for the scan (timestamp appended) | ❌ No  | `string` | `"Incremental API Scan"` |

## 🔐 Secrets

| Name                | Description                                  | Required |
|---------------------|----------------------------------------------|----------|
| `BRIGHT_API_TOKEN`  | Your Bright API token                        | ✅ Yes   |
| `BRIGHT_PROJECT_ID` | The target Bright project ID                 | ✅ Yes   |
| `BRIGHT_REPEATER_ID` | Optional Repeater ID for internal testing   | ❌ No    |

## ⚙️ Usage

This workflow is designed to be **called as a reusable workflow**.

### Example `main.yml` Caller Workflow

```yaml
name: Trigger Bright API Scan

on:
  workflow_dispatch:

jobs:
  scan:
    uses: your-org/your-repo/.github/workflows/bright-api-scan.yml@main
    with:
      archive_path: ./swagger.json
      scan_name: "Nightly API Scan"
    secrets:
      BRIGHT_API_TOKEN: ${{ secrets.BRIGHT_API_TOKEN }}
      BRIGHT_PROJECT_ID: ${{ secrets.BRIGHT_PROJECT_ID }}
      BRIGHT_REPEATER_ID: ${{ secrets.BRIGHT_REPEATER_ID }}
````

## 🔁 Conditional Repeater Support

The workflow automatically checks if a `BRIGHT_REPEATER_ID` was provided:

* If **provided**, discovery and scan commands use the Repeater for internal API access.
* If **not provided**, everything runs without the Repeater.

## 📤 Output

The workflow logs the following information during execution:

* Archive upload status
* Repeater detection and usage
* Discovery results
* Entrypoints used for scanning
* Scan ID upon successful scan creation

## 🐳 Docker Requirement

This action uses the Bright CLI via Docker:

* `docker` must be available on the GitHub runner (which it is by default on `ubuntu-latest` runners)

## ✅ Requirements

* Bright Security account and project setup
* Valid OpenAPI/Swagger definition
* GitHub repository with CI/CD permissions
* GitHub Actions enabled

## 🔒 Security Considerations

* Avoid checking secrets into version control.
* Consider using environment-specific Repeater IDs.
* Validate OpenAPI specs before uploading to Bright.

## 🧠 Additional Resources

* [Bright CLI Documentation](https://docs.brightsec.com/)
* [GitHub Actions Documentation](https://docs.github.com/en/actions)
* [Bright Security Product Overview](https://brightsec.com/)

---

