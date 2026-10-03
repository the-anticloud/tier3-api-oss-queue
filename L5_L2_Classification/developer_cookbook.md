# Developer Cookbook — api-oss-queue
**Stack:** Python 3.11, asyncio, SQLite (queue backend), AIOSS_FORMAT
**Domain:** Sovereign job queue: async task scheduling for long-running Anticloud operations
**License:** Apache-2.0 | **IP:** USPTO pending 2026, Anticloud FZ LLE

## Core Usage

```python
from api_oss_queue import JobQueue
queue = JobQueue('./queue.db', aioss_chain='./queue.aioss')

# Enqueue jobs
job_id = await queue.enqueue('inference', payload={'prompt': '...', 'priority': 'high'})
job_id2 = await queue.enqueue('batch_embed', payload={'files': ['a.md','b.md']})

# Worker
@queue.worker('inference')
async def inference_worker(job):
    result = pax.infer(job.payload['prompt'])
    return result

await queue.run_workers(concurrency=2)
```

## AIOSS Chain Append

```python
import hashlib, time

def aioss_append(chain_path, payload: bytes, module_id: str):
    entry_hash = hashlib.sha3_256(payload).digest()
    ts = int(time.time_ns()).to_bytes(8, 'big')
    with open(chain_path, 'rb') as f:
        f.seek(-32, 2); prev_hash = f.read(32)
    new_hash = hashlib.sha3_256(prev_hash + entry_hash + ts).digest()
    with open(chain_path, 'ab') as f:
        f.write(ts + entry_hash + new_hash)
    return new_hash.hex()

# After every api-oss-queue output:
chain_hash = aioss_append("./api_oss_queue.aioss",
                           result_bytes, "api-oss-queue")
```

## Performance & Integration

Performance: profile with api-oss-devtools. Benchmark with api-oss-analytics. Integration: all api-oss-queue operations are logged to api-oss-logging and audited by api-oss-compliance.
