

````md
# Lendsqr Assessment – API Test Collection

This repository contains an automated API test collection for the Lendsqr adjutor APIs, built and executed using Postman / Newman.

The collection covers:

- **BVN Verification**
- **Credit Bureau reports (CRC & FirstCentral)**
- **Bank directory & account verification endpoints** :contentReference[oaicite:0]{index=0}

---

## 1. Repository Structure

```text
.
├─ Lendsqr_Assessment_collection.postman_collection.json
├─ README.md
└─ newman-reports/
   ├─ lendsqr-report.html      # Generated after running tests with Newman (optional)
   └─ lendsqr-report.json      # Machine-readable report (optional)
````

> ⚠️ **Important:** Before pushing to GitHub, remove/replace any real secrets (e.g. `sk_live_...`) with placeholders and use environment variables instead. 

---

## 2. Tools & Prerequisites

* [Postman](https://www.postman.com/downloads/)
* [Node.js](https://nodejs.org/) (for Newman)
* [Newman](https://github.com/postmanlabs/newman) – Postman CLI runner

Install Newman globally:

```bash
npm install -g newman
```

(Optional) Install HTML reporter:

```bash
npm install -g newman-reporter-htmlextra
```

---

## 3. Collection Overview

The collection file is: `Lendsqr_Assessment_collection.postman_collection.json`. 

### 3.1. Global/Collection Variables

The collection uses the following variables:

* `base_url` – Base URL for the API, set to
  `https://adjutor.lendsqr.com/v2/` in a collection-level pre-request script.
* `access_token` – Secret token for Bearer auth (must be set securely).
* `BVN` – Sample Bank Verification Number.
* `contactNumber` – Phone number used for BVN Consent.
* `access_token2` – Additional token if needed for some endpoints. 

You should **override these via an environment file** or Postman environment, not hard-code real production secrets in the collection.

---

## 4. How to Set Up

### 4.1. Clone the Repository

```bash
git clone https://github.com/<your-username>/lendsqr-assessment.git
cd lendsqr-assessment
```

### 4.2. Import into Postman

1. Open Postman.
2. Click **Import** → **Upload Files**.
3. Select `Lendsqr_Assessment_collection.postman_collection.json`.

   | Variable        | Initial Value                             |
   | --------------- | ----------------------------------------- |
   | `base_url`      | `https://adjutor.lendsqr.com/v2/`         |
   | `access_token`  | `sk_live_xxx` (or test/placeholder token) |
   | `BVN`           | Valid BVN for test (e.g. `"22293381111"`) |
   | `contactNumber` | Test phone (e.g. `"0704xxxxxxx"`)         |

> Note: In the uploaded collection, a pre-request script already sets `base_url`, `access_token`, `BVN`, and `contactNumber`. For security, replace that logic with environment variables before publishing. 

### 4.3. How to get your api keys 
1. Visitt https://app.adjutor.io/login
2. Create an account and complete verification 
3. Navigate to app and create a new app (Give it access to every service)
4. Copy the access token/api key 
5. On postman, go to collection variable and paste inside the value field for the access token 

---

## 5. How to Run the Test Scripts

### 5.1. Running via Postman (Collection Runner)

1. Select the imported **Lendsqr_Assessment_collection**.
2. Click **Run** (Collection Runner).
3. Choose your environment (e.g. `Lendsqr-Adjutor-Env`).
4. Click **Start Run**.

Postman will show:

* The status of each request.
* Each test result (pass/fail) per request.

### 5.2. Running via Newman (CLI)

From the project root:

```bash
newman run Lendsqr_Asessment_collection.postman_collection.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export newman-reports/lendsqr-report.html
```

If you have a Postman environment file exported (e.g. `lendsqr-env.postman_environment.json`):

```bash
newman run Lendsqr_Asessment_collection.postman_collection.json \
  -e lendsqr-env.postman_environment.json \
  --reporters cli,htmlextra \
  --reporter-htmlextra-export newman-reports/lendsqr-report.html
```

This will generate:

* Console output of each request & test.
* A detailed HTML report in `newman-reports/lendsqr-report.html`.

---

## 6. Test Coverage Details

Below is a brief description of what’s being validated in each request, based on the test scripts embedded in the collection. 

### 6.1. Bank Verification Number (BVN)

#### **1) Initialize BVN Consent**

**Method:** `POST` `{{base_url}}verification/bvn/:bvn`

Tests:

* `Status code is 200` – ensures request is successful.
* `Validating Response Message` – `message` should be `"Please provide OTP sent to contact"`.
* `Response time is less than 500ms` – checks performance constraint.

#### **2) Complete Consent and get BVN Details**

**Method:** `PUT` `{{base_url}}verification/bvn/:bvn`

Tests:

* `Status code is 200`.
* `Message Validation` – `message === "Successful"`.
* `Status Validation` – `status === "success"`.
* `The 'reference' field is present and is a positive integer`.
* `The 'bvn' field must be present and a non-empty string`.
* `The first_name field is present and is a non-empty string`.
* `The last_name field must be present and be a non-empty string`.
* `The 'dob' field must be present and in 'YYYY-MM-DD' format`.

---

### 6.2. Credit Bureaus 🇳🇬

#### **3) Get Credit Report from CRC Credit Bureau**

**Method:** `GET` `{{base_url}}creditbureaus/crc/:bvn`

Tests:

* `Status code is 200`.
* `Message Validation` – `message === "Successful"`.
* `Status Validation` – `status === "success"`.
* `Consumer details object has all expected keys` – verifies shape of `nano_consumer_profile.consumer_details` (name, ruid, gender, last_name, first_name, citizenship, date_of_birth, identification).

#### **4) Get Credit Report from FirstCentral Credit Bureau**

**Method:** `GET` `{{base_url}}creditbureaus/firstcentral/:bvn`

Tests:

* `Status code is 200`.
* `Response Status Validation` – message for test mode (`"This is a test mode response. Your app is in test mode. Complete your KYC to access live data."`).
* `Response Status Validation` – `status === "success"`.
* `Response time is less than 500ms`.

---

### 6.3. Banks

#### **5) Get All Banks**

**Method:** `GET` `{{base_url}}direct-debit/banks?limit=100&page=1`

Tests:

* `Status code is 200`.
* `Response Status Validation` – `message === "This is a test mode response. Your app is in test mode. Complete your KYC to access live data."`.
* `Response Status Validation` – `status === "success"`.
* `Response time is less than 500ms`.

#### **6) Get Details of a Bank**

**Method:** `GET` `{{base_url}}direct-debit/banks/:bank_id`

Tests:

* `Status code is 200`.
* `Response Status Validation` – same test-mode message as above.
* `Response Status Validation` – `status === "success"`.
* `Response time is less than 500ms`.

#### **7) Verify Bank Account Number**

**Method:** `POST` `{{base_url}}direct-debit/banks/account-lookup`

Body:

```json
{
  "account_number": "220000000099",
  "bank_code": "057"
}
```

The collection includes this request with example response that returns `account_name`, masked `bvn`, and `session_id`. You can extend this request with additional assertions (e.g. account name not empty, BVN masked to expected pattern). 

---

## 7. Test Results Summary

> ✅ Replace the “Result” column with your actual Pass/Fail after running the collection in your environment (Postman or Newman).

### 7.1. High-Level Summary

* **Total requests with tests:** 6 (plus 1 request without explicit tests).
* **Total assertions:** ~25+ across status codes, messages, schema, and performance.
* **Expected behaviour:** with correct credentials and a valid BVN, all scripted tests **should pass**.

### 7.2. Per-Request Result Table

| # | Folder                | Request Name                                      | Purpose                                                | Result | Notes                            |
| - | --------------------- | ------------------------------------------------- | ------------------------------------------------------ | ------ | -------------------------------- |
| 1 | Bank Verification No. | Initialize BVN Consent                            | Start BVN verification & send OTP                      | TODO   | Expect status 200 & OTP message  |
| 2 | Bank Verification No. | Complete Consent and get BVN Details              | Complete BVN flow and fetch full BVN profile           | TODO   | Validates many profile fields    |
| 3 | Credit Bureaus        | Get Credit Report from CRC Credit Bureau          | Fetch credit profile from CRC                          | TODO   | Checks consumer details shape    |
| 4 | Credit Bureaus        | Get Credit Report from FirstCentral Credit Bureau | Fetch credit profile from FirstCentral (test-mode)     | TODO   | Test-mode message & performance  |
| 5 | Banks                 | Get All Banks                                     | Get list of banks available for direct debit           | TODO   | Test-mode message & performance  |
| 6 | Banks                 | Get Details of a Bank                             | Get metadata for a specific bank                       | TODO   | Test-mode message & performance  |
| 7 | Banks                 | Verify Bank Account Number                        | Validate account number and retrieve linked BVN & name | TODO   | Can extend with extra assertions |

Suggested values after you run:

* Use `✅ Pass` or `❌ Fail` in the **Result** column.
* If any test fails, note **why** in the **Notes** column (e.g. “invalid BVN used”, “expired token”, “sandbox config changed”, etc.).

```
