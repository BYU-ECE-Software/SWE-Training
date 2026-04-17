# 07 — MinIO

## What is Object Storage?

Traditional file systems store files in a hierarchy of folders. Object storage is different — it stores files (called **objects**) in flat containers called **buckets**. Each object has a key (basically a filename), the file data itself, and metadata.

Amazon S3 is the most well-known object storage service. MinIO is an open-source, S3-compatible alternative that we run locally in Docker. The same code that talks to MinIO will talk to S3 in production — you just change the endpoint URL.

We use object storage for anything that's a file: user uploads, attachments, generated reports, images.

---

## Core Concepts

| Term | Meaning |
|------|---------|
| **Bucket** | A named container for objects. Like a top-level folder. |
| **Object** | A file stored in a bucket, identified by its key. |
| **Key** | The object's name/path within the bucket (e.g., `attachments/task-1/report.pdf`) |
| **Presigned URL** | A temporary URL that allows a browser to upload/download an object directly without going through your server |

### Buckets

Bucket names must be globally unique within a MinIO instance. You create them once — they're like the top-level directories. A common pattern is one bucket per type of content:

```
task-attachments/
  task-1/report.pdf
  task-1/screenshot.png
  task-7/notes.txt

user-avatars/
  user-1.jpg
  user-2.jpg
```

---

## The S3 SDK

MinIO is S3-compatible, so we use the official AWS S3 SDK to talk to it:

```bash
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

### Client Setup

```ts
// lib/storage.ts
import { S3Client } from "@aws-sdk/client-s3";

export const s3 = new S3Client({
  endpoint: `http://${process.env.MINIO_ENDPOINT}:${process.env.MINIO_PORT}`,
  region: "us-east-1", // required by SDK, value doesn't matter for MinIO
  credentials: {
    accessKeyId: process.env.MINIO_ACCESS_KEY!,
    secretAccessKey: process.env.MINIO_SECRET_KEY!,
  },
  forcePathStyle: true, // required for MinIO
});

export const BUCKET = "task-attachments";
```

### Common Operations

```ts
import {
  PutObjectCommand,
  GetObjectCommand,
  DeleteObjectCommand,
  ListObjectsV2Command,
} from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

// Upload a file
await s3.send(new PutObjectCommand({
  Bucket: BUCKET,
  Key: `task-${taskId}/${filename}`,
  Body: fileBuffer,
  ContentType: "application/pdf",
}));

// Generate a presigned download URL (valid for 1 hour)
const url = await getSignedUrl(
  s3,
  new GetObjectCommand({ Bucket: BUCKET, Key: objectKey }),
  { expiresIn: 3600 }
);

// Delete an object
await s3.send(new DeleteObjectCommand({
  Bucket: BUCKET,
  Key: objectKey,
}));

// List objects in a bucket (optionally filter by prefix)
const result = await s3.send(new ListObjectsV2Command({
  Bucket: BUCKET,
  Prefix: `task-${taskId}/`,
}));
const files = result.Contents ?? [];
```

### Presigned Upload URLs

Instead of uploading a file through your server (wasting bandwidth), generate a presigned URL that lets the browser upload directly to MinIO:

```ts
import { PutObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

// In your API route:
const key = `task-${taskId}/${filename}`;
const uploadUrl = await getSignedUrl(
  s3,
  new PutObjectCommand({ Bucket: BUCKET, Key: key }),
  { expiresIn: 300 } // 5 minutes to complete the upload
);

// Return uploadUrl to the client
// The client does: fetch(uploadUrl, { method: "PUT", body: file })
```

---

## MinIO Console

MinIO ships with a web UI. In our Docker setup it's on port 9001:

```
http://localhost:9001
Username: minioadmin
Password: minioadmin
```

Use it to browse buckets, view objects, and create buckets manually. Useful for debugging.

---

## Environment Variables

Add to your `.env`:
```
MINIO_ENDPOINT=localhost     # Use "minio" inside Docker containers
MINIO_PORT=9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
```

---

## Helpful (optional, but encouraged) videos
[MinIO](https://www.youtube.com/watch?v=FjvDOGcTYyQ)

## Next Steps

Head to [EXERCISES.md](./EXERCISES.md) to try it out, as well as your mini-project directions

Editor's Note: All the MinIO memes searches auto-redirected me to Minion memes, and I was not about to put them here.