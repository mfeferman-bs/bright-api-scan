This GitHub Action workflow, titled **"Bright full API scan using swagger"**, automates a **complete API vulnerability scan** using the [Bright Security CLI](https://docs.brightsec.com/docs/cli-overview). It performs this scan based on a provided Swagger (OpenAPI) file and optionally uses a Repeater for internal environments.

Here’s a detailed breakdown of what it does:

---

## 🔁 Trigger: `workflow_call`

```yaml
on:
  workflow_call:
```

This makes the workflow **reusable** from other workflows using `uses:`. It accepts the following inputs and secrets:

### 📥 Inputs

* `archive_path` (required): Path to the Swagger/OpenAPI file
* `scan_name` (optional): Custom scan name

### 🔐 Secrets

* `BRIGHT_API_TOKEN`: API access token for Bright
* `BRIGHT_PROJECT_ID`: ID of the Bright project
* `BRIGHT_PROJECT_AUTH_ID`: Authentication profile to use for the scan
* `BRIGHT_REPEATER_ID`: (optional) Repeater ID for internal application access

---

## 🛠️ Job: `bright_api_scan`

### 1. **Checkout Repository**

```yaml
- name: Checkout code
```

Pulls down your source repo so the `archive_path` file can be accessed.

---

### 2. **Upload the OpenAPI File to Bright**

```bash
brightsec/cli archive:upload ...
```

Uploads the Swagger file to Bright and saves the resulting `archive_id` for use in discovery.

If this fails, the job exits with an error.

---

### 3. **Check for Repeater**

```bash
if [ -n "${{ secrets.BRIGHT_REPEATER_ID }}" ]; then ...
```

Sets a flag (`has_repeater=true|false`) that will determine whether to include the Repeater in subsequent Bright CLI calls.

---

### 4. **Run Discovery**

```bash
brightsec/cli discovery:run ...
```

Starts a **Discovery Scan** using the uploaded OpenAPI archive. This tells Bright to explore the API and generate a dynamic list of reachable endpoints.

Uses `--auth` to apply authentication (usually a bearer token, cookie, or login logic).

If `BRIGHT_REPEATER_ID` is set, it includes the Repeater for scanning internal APIs.

---

### 5. **Wait for Discovery to Complete**

```yaml
uses: NeuraLegion/wait-for-discovery@release
```

Waits (up to 10 minutes) for the discovery scan to finish.

---

### 6. **Get Discovered Entry Points**

```bash
brightsec/cli entrypoints:list ...
```

Fetches the list of discovered entrypoints (i.e., dynamic endpoints Bright found) and builds them into a string of `-e <id>` flags.

If no entrypoints are found, the workflow errors out.

---

### 7. **Run the Full Vulnerability Scan**

```bash
brightsec/cli scan:run ...
```

Performs a **full vulnerability scan** against the entrypoints, using multiple test buckets:

* `api`: API-specific tests (auth, rate-limiting, etc.)
* `client_side`: Client-side attacks like DOM XSS
* `server_side`: Server-based issues (injections, etc.)
* `legacy`: Older or edge-case vulnerabilities
* `business_logic`: Logical abuse (e.g., BOLA)
* `cve`: Known CVE checks

The scan is timestamped and optionally includes the `--repeater` flag.

---

## ✅ Summary

This reusable GitHub Action:

1. Accepts a Swagger file and project configuration
2. Uploads the Swagger to Bright
3. Performs a dynamic **Discovery** of all reachable endpoints
4. Waits for discovery completion
5. Collects discovered entrypoints
6. Runs a **full DAST scan** using Bright’s CLI with all vulnerability buckets

> 🔒 Optional: If `BRIGHT_REPEATER_ID` is provided, it enables scanning **internal environments** that are not exposed publicly.

This is ideal for **automating API scanning in CI/CD pipelines** with high coverage and minimal manual work.
