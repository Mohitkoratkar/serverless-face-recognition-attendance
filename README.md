# Serverless Face Recognition Attendance System

An AWS-based serverless application that uses **Amazon Rekognition** for face matching and **Amazon DynamoDB** to store attendance records. The system uses AWS Lambda, Amazon S3, Amazon API Gateway, and Amazon Cognito to implement a cloud-based attendance workflow.

## 🚀 Project Overview

The Serverless Face Recognition Attendance System demonstrates how AWS serverless services can be integrated to build an attendance management solution.

The system supports two main operations:

- **Face Enrollment:** Stores consented face images in Amazon S3 and uses Amazon Rekognition to index faces.
- **Attendance Marking:** Authenticated users submit an image through an API. AWS Lambda invokes Rekognition to search for a matching face and stores the attendance event in DynamoDB.

## 🛠️ Technologies Used

- **AWS Lambda** – Serverless application logic
- **Amazon Rekognition** – Face detection and matching
- **Amazon S3** – Face image storage
- **Amazon DynamoDB** – Face metadata and attendance records
- **Amazon API Gateway** – REST API endpoint
- **Amazon Cognito** – User authentication
- **Python** – Lambda function development
- **AWS SAM** – Infrastructure deployment and management
- **AWS CLI** – AWS resource management
- **Boto3** – AWS SDK for Python

## 🏗️ AWS Architecture

### 1. Face Enrollment Workflow

1. A consented face image is uploaded to an Amazon S3 bucket.
2. An S3 event triggers the Index Face Lambda function.
3. Lambda sends the image to Amazon Rekognition.
4. Rekognition indexes the detected face in a face collection.
5. Face metadata is stored in DynamoDB.

### 2. Attendance Marking Workflow

1. An authenticated user submits a face image through the API.
2. Amazon Cognito authenticates the request.
3. API Gateway invokes the Mark Attendance Lambda function.
4. Lambda sends the image to Amazon Rekognition for face matching.
5. If a matching face is found, attendance details are stored in DynamoDB.
6. The API returns the matching result and timestamp.

## 📂 Project Structure

```text
serverless-face-recognition-attendance/
│
├── src/
│   ├── index_face/
│   │   └── app.py
│   │
│   └── mark_attendance/
│       └── app.py
│
├── scripts/
│   └── create-collection.sh
│
├── template.yaml
├── requirements.txt
├── .gitignore
└── README.md
```

## ☁️ AWS Services Configuration

| AWS Service | Purpose |
|---|---|
| AWS Lambda | Executes face enrollment and attendance logic |
| Amazon Rekognition | Indexes and matches faces |
| Amazon S3 | Stores enrollment images |
| Amazon DynamoDB | Stores face metadata and attendance |
| Amazon API Gateway | Exposes attendance API |
| Amazon Cognito | Provides authentication |

## ⚙️ Prerequisites

Before deploying the project, install and configure:

- AWS CLI
- AWS SAM CLI
- Python 3.12
- Docker
- AWS credentials with the required permissions

Use the same AWS Region for the Rekognition collection and the deployed resources.

## 🚀 Deployment Steps

**1. Clone the repository**

```bash
git clone https://github.com/Mohitkoratkar/serverless-face-recognition-attendance.git
cd serverless-face-recognition-attendance
```

**2. Create a Rekognition collection**

```bash
chmod +x scripts/create-collection.sh
./scripts/create-collection.sh face-attendance-collection
```

**3. Build the application**

```bash
sam build
```

**4. Deploy AWS resources**

```bash
sam deploy --guided
```

During deployment, provide the Rekognition collection ID created in the previous step. Review the deployment configuration and estimated AWS costs before confirming.

**5. Upload a consented enrollment image**

```bash
aws s3 cp ./consented-sample.jpg s3://YOUR_BUCKET/enrollment/student-001/photo.jpg
```

Replace `YOUR_BUCKET` with the deployed S3 bucket name.

**6. Mark attendance**

After creating and authenticating a Cognito user, use the Cognito ID token to call the attendance API:

```bash
curl -X POST "$ATTENDANCE_API_URL/attendance" \
  -H "Authorization: Bearer $COGNITO_ID_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"image_base64":"BASE64_ENCODED_JPEG"}'
```

## 🗄️ DynamoDB Tables

**FaceIndex**

Stores face enrollment metadata, including:

- Person ID
- Rekognition Face ID
- S3 image key
- Creation timestamp

**Attendance**

Stores attendance events, including:

- Attendance ID
- Person ID
- Timestamp
- Face similarity score
- Attendance status

## 🧪 Validation Commands

Install Python dependencies:

```bash
python -m pip install -r requirements.txt
```

Check Python files:

```bash
python -m compileall src
```

Validate the AWS SAM template:

```bash
sam validate
```

## 🎯 Key Learning Outcomes

- Building serverless applications using AWS Lambda.
- Integrating Amazon Rekognition for face matching.
- Working with Amazon S3 event notifications.
- Storing application data in DynamoDB.
- Creating REST APIs using Amazon API Gateway.
- Implementing authentication with Amazon Cognito.
- Deploying infrastructure using AWS SAM.
- Integrating multiple AWS services into a serverless architecture.

## 🔒 Security & Privacy Considerations

Face images and biometric identifiers are sensitive data.

- Use face images only with informed consent and an appropriate lawful basis.
- Keep S3 buckets private and enable encryption.
- Restrict access using least-privilege IAM policies.
- Define data retention and deletion policies.
- Never commit real face images, credentials, or personal data to GitHub.
- Provide a non-biometric attendance alternative and human review.
- Do not make consequential decisions based solely on automated face matching.

## 👨‍💻 Author

**Mohit Koratkar**

- GitHub: https://github.com/Mohitkoratkar
- LinkedIn: https://www.linkedin.com/in/mohit-koratkar-7a8603239/
- Portfolio: https://mohitkoratkar.github.io/
