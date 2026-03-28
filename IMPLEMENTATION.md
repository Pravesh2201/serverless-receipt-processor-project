# 🛠️ Implementation Guide — Automated Receipt Processing Using AWS

This guide walks you through the complete setup of the serverless receipt processing pipeline with screenshots at every step.

---

## 1️⃣ Create the S3 Bucket

### Steps:
1. Go to the **S3 Console** → Click **Create Bucket**
2. Name your bucket (e.g., `storage-receipt-yourname`)
3. Choose a region (e.g., `ap-south-1`)
4. Click **Create bucket**
5. Inside the bucket, create a folder named **`incoming/`** — this is where you will upload receipt files

### Screenshots:
![alt text](s3-bucket.png)

> 📸 *Add your `incoming/` folder screenshot here*

---

## 2️⃣ Create a DynamoDB Table

### Steps:
1. Go to the **DynamoDB Console** → Click **Create Table**
2. Table name: `Receipts-table`
3. Partition key: `receipt_id` (String)
4. Sort key: `date` (String)
5. Click **Create**

### Screenshots:
> 📸 *Add your DynamoDB table creation screenshot here*

> 📸 *Add your DynamoDB table overview screenshot here*

---

## 3️⃣ Set Up Amazon SES

### Steps:
1. Go to **Amazon SES Console** → **Verified Identities**
2. Click **Create Identity** → choose **Email address**
3. Verify your **sender** email (check inbox and click the verification link)
4. Repeat and verify your **recipient** email as well
5. Both must show status ✅ **Verified** before proceeding

> ⚠️ **Note:** New AWS accounts are in SES sandbox mode. Both sender AND recipient emails must be verified in sandbox.

### Screenshots:
> 📸 *Add your SES Verified Identities screenshot here (showing both emails as Verified)*

---

## 4️⃣ Create IAM Role for Lambda

### Steps:
1. Go to **IAM Console** → **Roles** → **Create Role**
2. Trusted entity: **AWS Service** → Use case: **Lambda**
3. Attach the following 5 policies:
   - `AmazonS3ReadOnlyAccess`
   - `AmazonTextractFullAccess`
   - `AmazonDynamoDBFullAccess`
   - `AmazonSESFullAccess`
   - `AWSLambdaBasicExecutionRole`
4. Name the role: `LambdaReceiptProcessingRole`
5. Click **Create role**

### Screenshots:
> 📸 *Add your IAM role permissions screenshot here (showing all 5 policies)*

---

## 5️⃣ Create the Lambda Function

### Steps:
1. Go to **AWS Lambda Console** → Click **Create Function**
2. Name: `ProcessReceiptFunction`
3. Runtime: **Python 3.9**
4. Execution role: **Use an existing role** → Select `LambdaReceiptProcessingRole`
5. Click **Create function**
6. Go to the **Code tab** → paste the code from `lambda_function.py` → click **Deploy**

### Set Environment Variables:
Go to **Configuration → Environment Variables → Edit** and add:

| Key | Value |
|---|---|
| `DYNAMODB_TABLE` | `Receipts-table` |
| `SES_SENDER_EMAIL` | `your-verified-sender@email.com` |
| `SES_RECIPIENT_EMAIL` | `your-verified-recipient@email.com` |

### Increase Timeout:
Go to **Configuration → General configuration → Edit**
- Change timeout from `3 sec` → **`2 min 0 sec`**
- Click **Save**

### Screenshots:
> 📸 *Add your Lambda function overview screenshot here*

> 📸 *Add your environment variables screenshot here*

> 📸 *Add your timeout configuration screenshot here*

---

## 6️⃣ Add S3 Event Trigger

### Steps:
1. Go back to your **S3 Bucket → Properties tab**
2. Scroll down to **Event Notifications** → Click **Create event notification**
3. Fill in:
   - Event name: `ReceiptUploadTrigger`
   - Prefix: `incoming/`
   - Event types: ✅ **All object create events**
   - Destination: **Lambda function** → Select `ProcessReceiptFunction`
4. Click **Save changes**

### Screenshots:
> 📸 *Add your S3 event notification configuration screenshot here*

> 📸 *Add your S3 event notification saved confirmation screenshot here*

---

## 7️⃣ Test the Pipeline

### Option A — Manual Test in Lambda:
Go to **Lambda → Test tab**, create a new test event with:

```json
{
  "Records": [
    {
      "s3": {
        "bucket": { "name": "storage-receipt-yourname" },
        "object": { "key": "incoming/sample-receipt.jpg" }
      }
    }
  ]
}
```

Click **Test** — you should see `"Receipt processed successfully!"` in the response.

### Option B — Real End-to-End Test:
1. Upload any receipt image (JPG/PNG) to the `incoming/` folder in S3
2. Wait 30–60 seconds
3. Check your recipient email inbox (also check spam folder)
4. Check **DynamoDB → Receipts-table → Explore items** for the stored record

### Screenshots:
> 📸 *Add your Lambda test success screenshot here*

> 📸 *Add your DynamoDB stored record screenshot here*

> 📸 *Add your received email screenshot here*

---

## 8️⃣ Monitor with CloudWatch

If something doesn't work, check the logs:

1. Go to **Lambda → Monitor tab**
2. Click **View CloudWatch logs**
3. Open the **latest log stream**
4. Look for print statements like:
   - `Processing receipt from bucket/key` ✅
   - `Textract analyze_expense call successful` ✅
   - `Receipt data stored in DynamoDB` ✅
   - `Email notification sent to ...` ✅

### Screenshots:
> 📸 *Add your CloudWatch logs screenshot here*

---

## ✅ Verification Checklist

- [ ] S3 bucket created with `incoming/` folder
- [ ] DynamoDB table `Receipts-table` created with correct keys
- [ ] Both sender and recipient emails verified in SES
- [ ] IAM role created with all 5 policies
- [ ] Lambda function deployed with correct code
- [ ] All 3 environment variables set
- [ ] Lambda timeout increased to 2 minutes
- [ ] S3 event trigger configured with `incoming/` prefix
- [ ] Test event returns `statusCode: 200`
- [ ] Email received in inbox
- [ ] Record visible in DynamoDB

---

*✍️ Author: **Pravesh Gangwar***
