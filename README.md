# Creating a GCP/AWS Storage Bucket and Implementing File Uploads with Node.js as backend and ReactJs frontend

GCP storage to store files and I want to create a custom storage solution that enables users to upload zip files through my app, Google Cloud Storage (GCS) is a highly scalable and durable object storage service provided by Google Cloud Platform (GCP). GCS allows users to store and retrieve any amount of data from anywhere on the internet, with automatic scalability to meet the growing needs of an application.

## Why GCS / S3
1. Scalability
2. Durability
3. Security
4. Cost-effective
5. Integration with other GCP/AWS services

## For 10TB/day ingestion, avoid routing files through Node.js servers.
The correct architecture is:

Frontend (React) uploads directly to object storage using pre-signed URLs
Node.js backend only generates upload authorization + metadata
Object storage handles scaling
Async workers/events process files after upload

# HLD Design work-flow

                ┌────────────────────┐
                │    React Frontend  │
                └─────────┬──────────┘
                          │
                Request Upload URL
                          │
                          ▼
                ┌────────────────────┐
                │   Node.js API      │
                │ (Auth + Metadata)  │
                └─────────┬──────────┘
                          │
              Generate Signed Upload URL
                          │
                          ▼
                ┌───────────────────┐
                │ S3 / GCS Bucket   │
                └─────────┬─────────┘
                          │
                Object Created Event
                        │
                        ▼
            ┌──────────────────────────┐
            │ Queue / Stream / Worker  │
            │ Virus Scan / Thumbnail   │
            │ Compression / ETL        │
            └──────────────────────────┘

## Application data flow

React Client
    ↓
API Gateway
    ↓
Upload Service
    ↓
S3 Multipart Upload
    ↓
S3 Event Notification
    ↓
Kafka/SQS
    ↓
Workers
    ├── Metadata Extraction
    ├── Virus Scan
    ├── Compression
    ├── AI Processing
    └── Thumbnail Generation

## Why Direct Uploads Matter

If files go through Node.js 
            Client → Node.js → S3
        
enormous bandwidth costs
memory pressure
socket exhaustion
horizontal scaling pain
load balancer bottlenecks

Instead
            Client → S3/GCS directly

# Scale Estimation

10TB/day ≈

Metric	            Value
Per day	            10 TB
Per hour	        ~416 GB
Per minute	        ~7 GB
Per second avg	    ~120 MB/s

Peak traffic may be:

5–20x burst
2–5 GB/s ingress

# Upload Acceleration

AWS:    S3 Transfer Acceleration
GCP:    CDN + regional buckets

# Tech Stack

| Layer    | Tech              |
| -------- | ----------------- |
| Frontend | React             |
| API      | Node.js + Fastify |
| Auth     | JWT/OAuth         |
| Storage  | S3/GCS            |
| Queue    | Kafka/SQS         |
| Cache    | Redis             |
| DB       | PostgreSQL        |
| Infra    | Kubernetes        |
| CDN      | CloudFront        |

Created a complete production-scale upload platform foundation including:

Node.js + Fastify backend
React frontend
AWS S3 multipart upload flow
PostgreSQL integration
Redis setup
Docker setup
Kubernetes manifests
Multipart upload implementation
Signed URL generation
Upload completion flow
Production architecture notes
Scaling and security recommendations

# Monorepo Structure
file-uploads-distributed-system/
├── backend/
├── frontend/
├── docker-compose.yml
├── k8s/
└── README.md

# Schema

CREATE TABLE uploads (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID,
    file_name TEXT,
    object_key TEXT,
    upload_id TEXT,
    size BIGINT,
    mime_type TEXT,
    status TEXT,
    created_at TIMESTAMP DEFAULT now()
);

