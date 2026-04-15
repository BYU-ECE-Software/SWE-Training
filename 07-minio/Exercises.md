# 07 — MinIO: Exercises & Mini Project

## Prerequisites

MinIO should be running from module 01:
```bash
cd task-tracker && docker compose up -d minio
```

Visit the MinIO console at http://localhost:9001 (username: `minioadmin`, password: `minioadmin`).

---

## Exercise 1: Create a Bucket via the Console

1. Open http://localhost:9001 and log in
2. Click **Buckets** → **Create Bucket**
3. Name it `task-attachments` and click **Create**
4. Explore the UI — try uploading a file manually by clicking into the bucket

**Checkpoint:** You can create a bucket and upload files via the MinIO console.

---

## Exercise 2: Use the SDK to Upload and Download

Create a standalone script to practice the SDK outside of Next.js.

```bash
mkdir minio-demo && cd minio-demo
npm init -y
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

**`minio-demo/index.mjs`**
```js
import {
  S3Client,
  PutObjectCommand,
  GetObjectCommand,
  ListObjectsV2Command,
} from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";

const s3 = new S3Client({
  endpoint: "http://localhost:9000",
  region: "us-east-1",
  credentials: {
    accessKeyId: "minioadmin",
    secretAccessKey: "minioadmin",
  },
  forcePathStyle: true,
});

const BUCKET = "task-attachments";

// 1. Upload a text file
await s3.send(new PutObjectCommand({
  Bucket: BUCKET,
  Key: "task-1/notes.txt",
  Body: "These are my notes for task 1.",
  ContentType: "text/plain",
}));
console.log("Uploaded notes.txt");

// 2. List objects in the bucket
const list = await s3.send(new ListObjectsV2Command({ Bucket: BUCKET }));
console.log("Objects in bucket:", list.Contents?.map(o => o.Key));

// 3. Generate a presigned download URL
const url = await getSignedUrl(
  s3,
  new GetObjectCommand({ Bucket: BUCKET, Key: "task-1/notes.txt" }),
  { expiresIn: 60 }
);
console.log("Presigned URL (valid 60s):", url);
```

```bash
node index.mjs
```

Open the presigned URL in your browser — you should see the file contents. Wait 60 seconds and try again — it should fail with an expired signature error.

**Checkpoint:** You can upload, list, and generate presigned URLs using the S3 SDK.

---

## Exercise 3: Presigned Upload URL

Simulate the flow of a browser uploading directly to MinIO:

**`minio-demo/presigned-upload.mjs`**
```js
import { S3Client, PutObjectCommand, GetObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
import { writeFileSync } from "fs";

const s3 = new S3Client({
  endpoint: "http://localhost:9000",
  region: "us-east-1",
  credentials: { accessKeyId: "minioadmin", secretAccessKey: "minioadmin" },
  forcePathStyle: true,
});

// Step 1: Server generates an upload URL
const uploadUrl = await getSignedUrl(
  s3,
  new PutObjectCommand({
    Bucket: "task-attachments",
    Key: "task-2/upload.txt",
    ContentType: "text/plain",
  }),
  { expiresIn: 300 }
);

console.log("Upload URL generated:", uploadUrl.slice(0, 80) + "...");

// Step 2: Client uploads directly (simulated with fetch)
const uploadResponse = await fetch(uploadUrl, {
  method: "PUT",
  body: "Content uploaded directly from the client!",
  headers: { "Content-Type": "text/plain" },
});
console.log("Upload status:", uploadResponse.status);

// Step 3: Server generates a download URL
const downloadUrl = await getSignedUrl(
  s3,
  new GetObjectCommand({ Bucket: "task-attachments", Key: "task-2/upload.txt" }),
  { expiresIn: 60 }
);
console.log("Download URL:", downloadUrl.slice(0, 80) + "...");

// Fetch and print the content
const content = await fetch(downloadUrl).then(r => r.text());
console.log("File content:", content);
```

```bash
node presigned-upload.mjs
```

**Checkpoint:** You understand the presigned URL flow — the server never handles the file bytes.

---

## Mini Project: Task Attachments

Add file attachment support to the Task Tracker.

### 1. Install the SDK

```bash
cd task-tracker/app
npm install @aws-sdk/client-s3 @aws-sdk/s3-request-presigner
```

### 2. Add env variables

Add to `task-tracker/.env`:
```
MINIO_ENDPOINT=minio
MINIO_PORT=9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin
MINIO_BUCKET=task-attachments
```

### 3. Create the storage client

**`app/lib/storage.ts`**
```ts
import { S3Client } from "@aws-sdk/client-s3";

export const s3 = new S3Client({
  endpoint: `http://${process.env.MINIO_ENDPOINT}:${process.env.MINIO_PORT}`,
  region: "us-east-1",
  credentials: {
    accessKeyId: process.env.MINIO_ACCESS_KEY!,
    secretAccessKey: process.env.MINIO_SECRET_KEY!,
  },
  forcePathStyle: true,
});

export const BUCKET = process.env.MINIO_BUCKET ?? "task-attachments";
```

### 4. Create a bucket initializer

MinIO needs the bucket to exist before you can upload to it. Add a startup script:

**`app/lib/init-storage.ts`**
```ts
import { CreateBucketCommand, HeadBucketCommand } from "@aws-sdk/client-s3";
import { s3, BUCKET } from "./storage";

export async function ensureBucketExists() {
  try {
    await s3.send(new HeadBucketCommand({ Bucket: BUCKET }));
    console.log(`Bucket "${BUCKET}" already exists`);
  } catch {
    await s3.send(new CreateBucketCommand({ Bucket: BUCKET }));
    console.log(`Bucket "${BUCKET}" created`);
  }
}
```

### 5. API route: generate presigned upload URL

**`app/app/api/tasks/[id]/attachments/route.ts`**
```ts
import { NextResponse } from "next/server";
import { PutObjectCommand, ListObjectsV2Command, GetObjectCommand } from "@aws-sdk/client-s3";
import { getSignedUrl } from "@aws-sdk/s3-request-presigner";
import { s3, BUCKET } from "@/lib/storage";

// GET: list attachments for a task
export async function GET(
  _req: Request,
  { params }: { params: { id: string } }
) {
  const result = await s3.send(
    new ListObjectsV2Command({ Bucket: BUCKET, Prefix: `task-${params.id}/` })
  );

  const files = await Promise.all(
    (result.Contents ?? []).map(async (obj) => ({
      key: obj.Key,
      size: obj.Size,
      downloadUrl: await getSignedUrl(
        s3,
        new GetObjectCommand({ Bucket: BUCKET, Key: obj.Key }),
        { expiresIn: 3600 }
      ),
    }))
  );

  return NextResponse.json(files);
}

// POST: generate a presigned upload URL
export async function POST(
  req: Request,
  { params }: { params: { id: string } }
) {
  const { filename, contentType } = await req.json();

  if (!filename) {
    return NextResponse.json({ error: "filename required" }, { status: 400 });
  }

  const key = `task-${params.id}/${Date.now()}-${filename}`;

  const uploadUrl = await getSignedUrl(
    s3,
    new PutObjectCommand({ Bucket: BUCKET, Key: key, ContentType: contentType }),
    { expiresIn: 300 }
  );

  return NextResponse.json({ uploadUrl, key });
}
```

### 6. Test it

```bash
docker compose up --build

# Get a presigned upload URL
curl -X POST http://localhost/api/tasks/1/attachments \
  -H "Content-Type: application/json" \
  -d '{"filename":"notes.txt","contentType":"text/plain"}'

# Use the returned uploadUrl to upload:
curl -X PUT "<uploadUrl>" \
  -H "Content-Type: text/plain" \
  -d "My task notes"

# List attachments
curl http://localhost/api/tasks/1/attachments
```

---

## Reflection Questions

1. What's the difference between object storage and a regular filesystem?
2. Why use presigned URLs instead of uploading through your server?
3. What happens if a presigned URL expires before the upload is complete?
4. Why do we set `forcePathStyle: true` for MinIO but not for real AWS S3?

---

Move on to [08 — GitHub](../08-github/README.md).