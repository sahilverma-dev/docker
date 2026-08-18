# Pinecone Local Docker Setup

This directory contains a Docker Compose configuration for running **Pinecone Local** (`ghcr.io/pinecone-io/pinecone-local:latest`) for local vector database development and testing.

---

## 🛠 Services & Architecture

| Service | Image | Host Port | Container Port | Volume / Storage |
| :--- | :--- | :--- | :--- | :--- |
| `pinecone` | `ghcr.io/pinecone-io/pinecone-local:latest` | `5080` | `5080` | `pinecone_data` (`/data`) |

### Key Configuration
- **Container Name**: `pinecone-local`
- **Environment Variables**:
  - `PORT`: `5080`
  - `PINECONE_HOST`: `0.0.0.0` (Binds server host to receive external connections)
- **Persistence**: Named volume `pinecone_data` mapped to `/data`.
- **Restart Policy**: `unless-stopped`

---

## 🚀 Quick Start

### 1. Start the Container
Run the service in detached mode:
```bash
docker compose up -d
```

### 2. Verify Running Container
Check if the `pinecone-local` container is running:
```bash
docker compose ps
```

View logs to ensure Pinecone initialized cleanly:
```bash
docker compose logs -f
```

### 3. Stop the Service
Stop the container without removing volume data:
```bash
docker compose down
```

To stop and wipe persisted data volume:
```bash
docker compose down -v
```

---

## ⚡ Client Connection Examples

When connecting using Pinecone SDKs locally:
- **Host / Endpoint**: `http://localhost:5080` (or `http://127.0.0.1:5080`)
- **API Key**: `"pinecone-local"` (or any non-empty string placeholder)

---

### 🐍 Python Example

#### Installation
```bash
pip install pinecone
```

#### Usage Script (`example.py`)
```python
from pinecone import Pinecone, ServerlessSpec

# 1. Initialize Pinecone client targeting the local instance
pc = Pinecone(
    api_key="pinecone-local",
    host="http://localhost:5080"
)

index_name = "demo-index"

# 2. Create an index if it doesn't already exist
existing_indexes = [idx.name for idx in pc.list_indexes()]
if index_name not in existing_indexes:
    pc.create_index(
        name=index_name,
        dimension=3,
        metric="cosine",
        spec=ServerlessSpec(cloud="aws", region="us-east-1")
    )
    print(f"Index '{index_name}' created.")

# 3. Connect to the index
index = pc.Index(index_name)

# 4. Upsert sample vector embeddings
index.upsert(
    vectors=[
        {"id": "vec1", "values": [0.1, 0.2, 0.3], "metadata": {"category": "tech"}},
        {"id": "vec2", "values": [0.4, 0.5, 0.6], "metadata": {"category": "finance"}},
    ]
)
print("Vectors upserted successfully.")

# 5. Query the index
query_response = index.query(
    vector=[0.1, 0.2, 0.3],
    top_k=2,
    include_metadata=True
)

print("Query Results:")
print(query_response)
```

---

### 🟨 JavaScript / TypeScript Example

#### Installation
```bash
npm install @pinecone-database/pinecone
```

#### Usage Script (`example.ts` / `example.js`)
```typescript
import { Pinecone } from '@pinecone-database/pinecone';

async function run() {
  // 1. Initialize Pinecone client targeting the local instance
  const pc = new Pinecone({
    apiKey: 'pinecone-local',
    host: 'http://localhost:5080',
  });

  const indexName = 'demo-index';

  // 2. Create an index if it doesn't exist
  const existingIndexes = await pc.listIndexes();
  const indexExists = existingIndexes.indexes?.some((idx) => idx.name === indexName);

  if (!indexExists) {
    await pc.createIndex({
      name: indexName,
      dimension: 3,
      metric: 'cosine',
      spec: {
        serverless: {
          cloud: 'aws',
          region: 'us-east-1',
        },
      },
    });
    console.log(`Index '${indexName}' created.`);
  }

  // 3. Target the index
  const index = pc.index(indexName);

  // 4. Upsert vector data
  await index.upsert([
    {
      id: 'vec1',
      values: [0.1, 0.2, 0.3],
      metadata: { category: 'tech' },
    },
    {
      id: 'vec2',
      values: [0.4, 0.5, 0.6],
      metadata: { category: 'finance' },
    },
  ]);
  console.log('Vectors upserted successfully.');

  // 5. Query vectors
  const queryResult = await index.query({
    vector: [0.1, 0.2, 0.3],
    topK: 2,
    includeMetadata: true,
  });

  console.log('Query Results:', JSON.stringify(queryResult, null, 2));
}

run().catch(console.error);
```

---

## 📋 Docker Compose File Review Summary

- **Correctness**: 🟢 Excellent. Uses official GitHub Container Registry image `ghcr.io/pinecone-io/pinecone-local:latest`.
- **Port Mapping**: 🟢 Port `5080` properly exposed and mapped.
- **Persistence**: 🟢 Includes persistent named volume (`pinecone_data`) so data is preserved across container restarts.
- **Restart Policy**: 🟢 `unless-stopped` ensures resilience against accidental crashes or system reboots.
