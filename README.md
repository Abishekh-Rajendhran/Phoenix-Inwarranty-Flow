# Phoenix Inwarranty Flow – API Test Automation

This project runs automated API tests for the **Phoenix Inwarranty Flow** system using Postman and Newman. Tests run automatically on every code push, on a daily schedule, and can also be triggered manually. After each run, an HTML report is published and an email notification is sent.

---

## What This Project Does

The Inwarranty Flow is a repair job management system that handles the entire lifecycle of a warranty repair — from a customer walking in, to the job being assigned, repaired, quality-checked, and finally delivered back to the customer.

This project tests all the APIs involved in that flow across five different user roles:

| Role | What They Do |
|---|---|
| **Front Desk** | Logs in, creates jobs, searches jobs, handles delivery |
| **Supervisor** | Assigns jobs to engineers |
| **Engineer** | Updates repair status |
| **QC (Quality Control)** | Inspects completed repairs |
| **Front Desk (Delivery)** | Closes the job and delivers the device to the customer |

---

## Files in This Project

| File | What It Is |
|---|---|
| `Inwarranty-flow Collection.postman_collection.json` | All the API test cases (150+ requests) |
| `QA.postman_environment.json` | Environment config — base URL and variables for the QA server |
| `testData.csv` | Test data (customer names, phone numbers, emails) used by the collection |
| `Collection_Workflow.yml` | GitHub Actions workflow that runs the tests automatically |

---

## How the Tests Are Organized

Tests follow the full repair job journey in order:

1. **Login** – Each user role logs in and gets an auth token
2. **User Details** – Verify the logged-in user's profile
3. **Count** – Check dashboard counts
4. **Create Job** – Front desk creates a new repair job (uses CSV test data)
5. **Search Job** – Look up a job by job number
6. **Job Details** – Get full details of a job
7. **Assign Job** – Supervisor assigns the job to an engineer
8. **Repair** – Engineer marks the repair as complete
9. **QC** – Quality control team inspects and approves
10. **Delivery** – Front desk delivers the device back to the customer

Each step includes both **positive tests** (happy path) and **negative tests** (wrong token, no auth, bad payload, wrong HTTP verb, invalid data, etc.).

---

## Test Data

The `testData.csv` file contains sample customer data used when creating jobs:

```
first_name, last_name, mobile_number, email_id
jatin, sharma, 7046563552, jatinvsharma@gmail.com
uday, p, 9321435112, uday@gmail.com
kajal, t, 7046563551, kajal@gmail.com
```

The collection loops through these rows to create multiple jobs during the test run.

---

## Environment (QA)

| Variable | Value |
|---|---|
| `BASE_URL` | `http://64.227.160.186:9000/v1` |
| `fd_token` | Set automatically after Front Desk login |
| `X` | Dynamic value set during the run |

---

## How to Run Locally

Make sure you have Node.js installed, then:

```bash
# Install Newman and the HTML report plugin
npm install -g newman
npm install -g newman-reporter-htmlextra

# Run the collection
newman run "Inwarranty-flow Collection.postman_collection.json" \
  -e "QA.postman_environment.json" \
  -d "testData.csv" \
  -r cli,htmlextra \
  --reporter-htmlextra-export newman/index.html
```

The HTML report will be saved to `newman/index.html`.

---

## How Automation Works (GitHub Actions)

The workflow in `Collection_Workflow.yml` runs tests automatically in three situations:

- **On push to `main`** – every time code is merged
- **On a schedule** – every day at 2:30 AM UTC
- **Manually** – you can trigger it yourself from the GitHub Actions tab

### What the workflow does step by step:

1. Checks out the code
2. Sets up Node.js 22
3. Installs Newman and the HTML reporter
4. Runs the Postman collection with test data
5. Uploads the HTML report as a build artifact
6. Publishes the report to **GitHub Pages**
7. Sends an **email notification** with the report link

### View the latest report

After each run, the report is published at:
**https://abishekh-rajendhran.github.io/Phoenix-Inwarranty-Flow/**

---

## GitHub Secrets Required

You need to add these secrets in your GitHub repo settings before the workflow can send emails:

| Secret | What It Is |
|---|---|
| `EMAIL_USERNAME` | Gmail address used to send the notification |
| `EMAIL_PASSWORD` | Gmail app password (not your regular password) |
| `GITHUB_TOKEN` | Provided automatically by GitHub — no setup needed |

---

## Runner

This workflow uses a **self-hosted runner** (`self-hosted-ubuntu-runner`). Make sure your self-hosted runner is online and connected to the repository before triggering a run.
