# Serverless Face Recognition Attendance System

AWS serverless starter for enrolling consented face images from S3 and recording attendance through a Cognito-authenticated API. Rekognition performs face matching; DynamoDB stores face metadata and attendance events.

> **Biometric privacy warning:** Face data is sensitive personal data. Use only with informed consent, a lawful basis, strict access and retention controls, and a tested non-biometric alternative. Do not make consequential decisions based only on automated matching. Never commit real face images or personal data.

## Architecture
Enrollment: S3 `enrollment/<person-id>/photo.jpg` → IndexFace Lambda → Rekognition collection → DynamoDB FaceIndex.
Attendance: Cognito JWT → API Gateway `POST /attendance` → MarkAttendance Lambda → Rekognition search → DynamoDB Attendance.

## Prerequisites
AWS CLI credentials, AWS SAM CLI, Python 3.12, Docker and permissions to create resources in `template.yaml`. Deploy all resources in one AWS Region.

## Deploy
1. Create a Rekognition collection:
   ```bash
   chmod +x scripts/create-collection.sh
   ./scripts/create-collection.sh face-attendance-collection
   ```
2. Build and deploy:
   ```bash
   sam build
   sam deploy --guided
   ```
   Set `RekognitionCollectionId` to the collection created above. Review the change set and costs before confirming.
3. Upload one clear, consented image per person to `enrollment/<person-id>/photo.jpg`:
   ```bash
   aws s3 cp ./consented-sample.jpg s3://YOUR_BUCKET/enrollment/student-001/photo.jpg
   ```
4. Create a Cognito user, authenticate to obtain an ID token, and call:
   ```bash
   curl -X POST "$ATTENDANCE_API_URL/attendance" \
     -H "Authorization: Bearer $COGNITO_ID_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"image_base64":"BASE64_ENCODED_JPEG"}'
   ```

## API
`POST /attendance` requires Cognito authorization. Request body:
```json
{"image_base64":"BASE64_ENCODED_IMAGE_BYTES"}
```
Successful match returns person ID, similarity and UTC timestamp. No match returns HTTP 404; invalid input returns HTTP 400.

## DynamoDB
- FaceIndex: partition key `person_id`; stores Rekognition face ID, S3 key and creation timestamp.
- Attendance: partition key `attendance_id`; stores person ID, timestamp, similarity and status.

## Local checks
```bash
python -m pip install -r requirements.txt
python -m compileall src
sam validate
```
Compilation does not test live AWS service integrations.

## Limitations and production work
The sample indexes up to one face per enrollment image and scans FaceIndex to map a Rekognition FaceId to a person. For scale, add a `face_id` GSI. Production needs duplicate-enrollment handling, deletion workflows, attendance windows, idempotency, rate limiting, audit trails, retention policies, monitoring and a user interface. The Rekognition collection is created separately and must exist in the deployment Region.

Keep S3 private, encrypt data, restrict enrollment/API access, document consent and retention, delete S3 images/Rekognition faces/DynamoDB metadata when required, avoid logging images or tokens, review CloudTrail, and provide human review plus a non-biometric option.
# serverless-face-recognition-attendance
