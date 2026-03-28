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

<img width="979" height="399" alt="s3-bucket" src="https://github.com/user-attachments/assets/543f72c5-6dfa-4baa-99e3-da8bff43ecce" />

<img width="1462" height="479" alt="s3-object" src="https://github.com/user-attachments/assets/084952c0-20b9-43f2-89ac-50dda271cb50" />


---

## 2️⃣ Create a DynamoDB Table

### Steps:
1. Go to the **DynamoDB Console** → Click **Create Table**
2. Table name: `Receipts-table`
3. Partition key: `receipt_id` (String)
4. Sort key: `date` (String)
5. Click **Create**

<img width="1456" height="721" alt="dynamodb-table" src="https://github.com/user-attachments/assets/2150f424-48a6-497e-af1a-a980df800b81" />


<img width="1448" height="756" alt="dynamodb-table-update" src="https://github.com/user-attachments/assets/b08f0b12-449c-4d43-9d1b-6cdb6788594c" />


---

## 3️⃣ Set Up Amazon SES

### Steps:
1. Go to **Amazon SES Console** → **Verified Identities**
2. Click **Create Identity** → choose **Email address**
3. Verify your **sender** email (check inbox and click the verification link)
4. Repeat and verify your **recipient** email as well
5. Both must show status ✅ **Verified** before proceeding

> ⚠️ **Note:** New AWS accounts are in SES sandbox mode. Both sender AND recipient emails must be verified in sandbox.

<img width="1454" height="659" alt="ses" src="https://github.com/user-attachments/assets/d013c4b9-5120-49a3-9470-5eaa48b464bc" />

<img width="1461" height="734" alt="ses1" src="https://github.com/user-attachments/assets/34f58f41-34a5-46b4-b102-9ea354943bb4" />

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

<img width="1335" height="761" alt="IAM-role" src="https://github.com/user-attachments/assets/a66cfb32-eefe-43a6-84e5-ab8e1eaf666f" />

<img width="1463" height="718" alt="Role-permission" src="https://github.com/user-attachments/assets/711f471c-3820-45b6-9c81-77b836df5c0b" />

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

<img width="1326" height="746" alt="lambdafn" src="https://github.com/user-attachments/assets/66ac80be-411e-4880-bad2-29c433a19139" />

<img width="1458" height="754" alt="lambda-py-fn" src="https://github.com/user-attachments/assets/fb19bbb2-20f5-4f1c-b0f3-d4226323047e" />

<img width="1090" height="282" alt="labda-env" src="https://github.com/user-attachments/assets/a260b870-4d71-42c5-8e53-6f2e12eac864" />

<img width="1348" height="761" alt="lambda-basic-setting" src="https://github.com/user-attachments/assets/e9ac1463-05c0-4939-b1f0-7ef429acb808" />


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

<img width="1299" height="735" alt="s3-bucket-event" src="https://github.com/user-attachments/assets/9f9432b1-6c43-4d73-bb92-f0aa390a86cd" />

<img width="1444" height="675" alt="s3-bucket-event1" src="https://github.com/user-attachments/assets/e08252fe-f63b-41a6-b49e-5f29d27bb0cd" />


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

<img width="1451" height="757" alt="lambda-cloudwatch" src="https://github.com/user-attachments/assets/ba384fea-f1e5-4ea5-b8f0-54561fb79675" />


<img width="1448" height="756" alt="dynamodb-table-update" src="https://github.com/user-attachments/assets/f3d848c2-8e12-41d1-9d33-58ec43c8e87d" />


<img width="1089" height="684" alt="gmail-received -mail" src="https://github.com/user-attachments/assets/88e1f13e-86ce-440f-a032-85d1ebc49716" />

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

<img width="1424" height="755" alt="cloudwatch-log" src="https://github.com/user-attachments/assets/f285aa86-54ee-4b42-a1ba-ad8f714bb1db" />


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

*✍️ Author: **Pravesh Kumar***
