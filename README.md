# 🧾 Automated Receipt Processing Pipeline — AWS Serverless

A fully serverless receipt processing system built on AWS that automatically extracts data from receipt images, stores it in a database, and sends email notifications — with zero manual intervention.

---

## 🏗️ Architecture Overview

```
Receipt Image
     │
     ▼
┌─────────────┐     S3 Event      ┌──────────────┐     AnalyzeExpense     ┌─────────────┐
│  Amazon S3  │ ────────────────► │ AWS Lambda   │ ───────────────────── ► │  Textract   │
│  (Storage)  │                   │  (Processor) │                         │  (OCR/AI)   │
└─────────────┘                   └──────┬───────┘                         └─────────────┘
                                         │
                          ┌──────────────┼──────────────┐
                          ▼                             ▼
                  ┌──────────────┐            ┌──────────────┐
                  │   DynamoDB   │            │  Amazon SES  │
                  │  (Database)  │            │   (Email)    │
                  └──────────────┘            └──────────────┘
```

---

## ✨ Features

- **Automatic trigger** — Upload a receipt to S3 and the entire pipeline runs instantly
- **AI-powered extraction** — Uses AWS Textract `AnalyzeExpense` to detect vendor, date, total, and line items
- **Persistent storage** — All receipt data stored in DynamoDB with a unique ID and timestamp
- **Email notifications** — Instant email summary sent via Amazon SES after every processing
- **Error handling** — Object verification, URL decoding for special characters, and graceful failure logging

---

## 🛠️ AWS Services Used

| Service | Purpose |
|---|---|
| **Amazon S3** | Stores uploaded receipt images |
| **AWS Lambda** | Core processing engine (Python 3.9) |
| **Amazon Textract** | AI-based receipt data extraction |
| **Amazon DynamoDB** | Stores structured receipt data |
| **Amazon SES** | Sends email notifications |
| **AWS IAM** | Role-based access control |
| **Amazon CloudWatch** | Logging and monitoring |

---

## 📁 Repository Structure

```
receipt-processing-pipeline/
│
├── lambda_function.py       # Main Lambda handler code
├── IMPLEMENTATION.md        # Step-by-step setup guide with screenshots
└── README.md                # Project overview (this file)
```

---

## ⚙️ How It Works

1. Upload a receipt image (JPG/PNG/PDF) to the `incoming/` folder in S3
2. S3 fires an event notification to Lambda automatically
3. Lambda calls **Textract AnalyzeExpense** on the image
4. Textract returns structured data: vendor name, date, total amount, line items
5. Lambda stores all extracted data in **DynamoDB** with a UUID and timestamp
6. Lambda sends an **HTML email** via SES with the full receipt summary

---

## 📊 DynamoDB Schema

```json
{
  "receipt_id": "uuid-string",
  "date": "2026-03-29",
  "vendor": "Store Name",
  "total": "149.99",
  "items": [
    { "name": "Item 1", "price": "99.99", "quantity": "1" }
  ],
  "s3_path": "s3://bucket-name/incoming/receipt.jpg",
  "processed_timestamp": "2026-03-29T00:10:00.000000"
}
```

---

## 🧪 Testing

Use this test event in **Lambda → Test tab** to trigger manually:

```json
{
  "Records": [
    {
      "s3": {
        "bucket": { "name": "your-bucket-name" },
        "object": { "key": "incoming/sample-receipt.jpg" }
      }
    }
  ]
}
```

---

## 🐛 Troubleshooting

| Issue | Likely Cause | Fix |
|---|---|---|
| No email received | SES sandbox — recipient not verified | Verify recipient email in SES console |
| DynamoDB write fails | Table name mismatch | Ensure `DYNAMODB_TABLE` env var matches exactly |
| Lambda not triggering | S3 prefix wrong | Set prefix to `incoming/` with trailing slash |
| Textract fails | File format unsupported | Use JPG, PNG, or PDF only |
| Timeout error | Default timeout too short | Increase Lambda timeout to 2 minutes |

---

## 🔐 Security Notes

- Lambda uses a least-privilege IAM role
- S3 bucket is private — Lambda accesses it via IAM, not a public URL
- SES operates in sandbox mode by default

---

## 👨‍💻 Author

**Pravesh Gangwar**
📬 [LinkedIn](https://www.linkedin.com/in/pravesh22) · [GitHub](https://github.com/pravesh2201)

---

## 📄 License

MIT License — free to use and modify.
