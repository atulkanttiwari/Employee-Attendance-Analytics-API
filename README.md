# Employee Attendance & Analytics API

Backend service for the HROne attendance system, built with FastAPI and MongoDB.

## How to Run

1. Ensure you have MongoDB 6.0+ running (locally or via Docker).
2. Set your environment variables in a `.env` file:
   ```text
   MONGO_URI=mongodb://localhost:27017
   MONGO_DB=attendance_db

3. Install Dependencies
* pip install -r requirements.txt

4. Start the Server
* uvicorn app.main:app --port 8000

## Notes for the Reviewer
* All required database indexes are created idempotently on application startup.
* All endpoints, including the MongoDB aggregation pipelines and the admin explain endpoint, are fully implemented.