# Cost Optimization Strategies in Cloud Computing using AWS Services

Serverless AWS projects built to do useful work at near-zero idle cost: no servers to keep running, pay only per event.

## Automatic Receipt Manager — [`Automatic recipt manager/`](Automatic%20recipt%20manager)

Upload a photo or PDF of a receipt to S3 and it is read, stored and emailed to you automatically.

```
S3 upload ──► Lambda ──► Textract AnalyzeExpense ──► DynamoDB ──► SES email
```

1. An **S3** `ObjectCreated` event triggers the **Lambda** function (Python, `boto3`).
2. **Textract `AnalyzeExpense`** extracts the vendor, date, total and line items (name, price, quantity).
3. The result is written to a **DynamoDB** table with a generated receipt ID and the S3 path.
4. **SES** emails a formatted summary.

| File | Contents |
|---|---|
| `code.txt` | Lambda handler source |
| `Lambda Function Explanation.pdf` | Walkthrough of the function |

### Deploy

1. Create the S3 bucket, a DynamoDB table (partition key `receipt_id`) and verify the sender/recipient in SES.
2. Create a Python Lambda with the code from `code.txt` and these environment variables:

   | Variable | Default |
   |---|---|
   | `DYNAMODB_TABLE` | `Receipts` |
   | `SES_SENDER_EMAIL` | verified sender |
   | `SES_RECIPIENT_EMAIL` | where summaries go |

3. Give the Lambda role `s3:GetObject`, `textract:AnalyzeExpense`, `dynamodb:PutItem` and `ses:SendEmail`.
4. Add the S3 bucket as the trigger.

### Why it is cheap

Every piece is pay-per-use: Lambda bills per invocation, Textract per page, DynamoDB on-demand per write and SES per
email. With no traffic the stack costs nothing beyond S3 storage.
