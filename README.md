<img width="709" height="231" alt="image" src="https://github.com/user-attachments/assets/1804d3b0-b094-4a74-9221-e81b402227e3" />

# 🕸️ Trustpilot Scraper Workflow (n8n + Bright Data + Google Sheets)

This workflow automates the scraping of Trustpilot company reviews using **Bright Data**, processes the data in **n8n**, and stores it in **Google Sheets**. It includes manual URL input, data extraction, job monitoring, and automated storage.

---

## 🧱 Workflow Architecture

### 1. 📝 **Form Trigger Node**
- **Type:** `n8n-nodes-base.formTrigger`
- **Purpose:** Accepts a Trustpilot URL from the user to start the scraping process.
- **Field:** `Trustpilot Website URL`

---

### 2. 🌐 **HTTP Request: Trigger Scraping**
- **Type:** `n8n-nodes-base.httpRequest`
- **Method:** `POST`
- **Endpoint:** `https://api.brightdata.com/datasets/v3/trigger`
- **Authentication:** Bearer token
- **Body:** JSON payload containing the input URL and over 35 custom fields.

#### 🏷️ Extracted Fields:
- **Company Info:** `company_name`, `company_logo`, `company_overall_rating`, `company_total_reviews`, `company_about`, `company_email`, `company_phone`, `company_location`, `company_country`, `company_category`, `company_id`, `company_website`
- **Review Data:** `review_id`, `review_date`, `review_rating`, `review_title`, `review_content`, `review_date_of_experience`, `review_url`, `date_posted`
- **Reviewer Info:** `reviewer_name`, `reviewer_location`, `reviews_posted_overall`
- **Metadata:** `is_verified_review`, `review_replies`, `review_useful_count`
- **Rating Distribution:** `5_star`, `4_star`, `3_star`, `2_star`, `1_star`
- **Other Fields:** `url`, `company_rating_name`, `is_verified_company`, `breadcrumbs`, `company_other_categories`

---

### 3. ⌛ **Snapshot Progress Check**
- **Type:** `n8n-nodes-base.httpRequest`
- **Method:** `GET`
- **Endpoint:** `https://api.brightdata.com/datasets/v3/progress/{{ $json.snapshot_id }}`
- **Purpose:** Checks if the scraping job has completed.

---

### 4. ✅ **IF Node (Status Check)**
- **Type:** `n8n-nodes-base.if`
- **Condition:** `$json.status === "ready"`
- **Purpose:** Determines whether to continue or wait.

---

### 5. 🕒 **Wait Node**
- **Type:** `n8n-nodes-base.wait`
- **Duration:** `1 minute`
- **Purpose:** Prevents API overuse by pausing before rechecking status.

---

### 6. 🔄 **Loop Logic**
- **Flow:** Wait → Check Status → Evaluate → Loop or Proceed
- **Purpose:** Ensures continuous monitoring until data is ready.

---

### 7. 📥 **Snapshot Download**
- **Type:** `n8n-nodes-base.httpRequest`
- **Method:** `GET`
- **Endpoint:** `https://api.brightdata.com/datasets/v3/snapshot/{{ $json.snapshot_id }}`
- **Purpose:** Downloads the completed scraped dataset in JSON format.

---

### 8. 📊 **Google Sheets Integration**
- **Type:** `n8n-nodes-base.googleSheets`
- **Operation:** `Append`
- **Document ID:** `1yQ10Q2qSjm-hhafHF2sXu-hohurW5_KD8fIv4IXEA3I`
- **Sheet Name:** `Trustpilot`
- **Mapping:** Auto-map all 35+ fields
- **Authentication:** Google OAuth2

---

## 🔄 Data Flow Summary

