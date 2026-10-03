# Enterprise Use Cases â€” api-oss-queue

## Overview

api-oss-queue is a local-first message queue replacing AWS SQS, Azure Service Bus, and similar managed queue services. It supports Redis Streams, RabbitMQ, and in-process queues, with SQS-compatible API surface for easy migration.

---

## Use Case 1: LLM Inference Request Queue

**Scenario:** High-traffic application queues inference requests to a local vLLM server, smoothing burst traffic and preventing OOM on the GPU.

```python
from api_oss_queue import Queue
from aioss import Ledger

ledger = Ledger.open("./queue_ledger.aioss")
q = Queue.connect("redis://localhost:6379", queue_name="llm-requests")

# Producer
def enqueue_inference(prompt: str, request_id: str):
    msg_id = q.send({"prompt": prompt, "request_id": request_id})
    ledger.append(
        entry_type="message_enqueued",
        actor="api-oss-queue",
        content={"tokens_in": len(prompt.split()), "tokens_out": 0,
                 "wall_time_ms": 2, "cost_if_cloud_microcents": 0}
    )
    return msg_id

# Consumer (runs on GPU node)
for msg in q.consume(batch_size=4):
    result = vllm_client.generate(msg["prompt"])
    q.ack(msg["id"])
```

**ROI:** AWS SQS: $0.40/million requests. At 10M requests/month: $4/month. But SQS triggers API gateway costs at scale; managed queue SaaS $500â€“2,000/month. Local Redis: **$6,000â€“24,000/year saved**.

---

## Use Case 2: Event-Driven AI Pipeline Orchestration

**Scenario:** Data pipeline triggers ML model retraining when new data arrives, coordinating ingest â†’ preprocess â†’ train â†’ evaluate stages via queues.

```python
from api_oss_queue import Queue, DeadLetterQueue

pipeline_stages = {
    "ingest": Queue.connect("redis://localhost:6379", "pipeline-ingest"),
    "preprocess": Queue.connect("redis://localhost:6379", "pipeline-preprocess"),
    "train": Queue.connect("redis://localhost:6379", "pipeline-train"),
}
dlq = DeadLetterQueue.connect("redis://localhost:6379", "pipeline-dlq")

# Stage router with AIOSS audit
def route_stage(event: dict, from_stage: str, to_stage: str):
    pipeline_stages[to_stage].send(event)
    ledger.append(entry_type="stage_routed", actor="api-oss-queue",
                  content={"tokens_in": 0, "tokens_out": 0,
                           "wall_time_ms": 1, "cost_if_cloud_microcents": 0})
```

**Deployment:**
```
[Data Source] â”€â”€â–º [ingest queue] â”€â”€â–º [preprocess queue] â”€â”€â–º [train queue]
                                              â”‚
                                    [DLQ for failed messages]
                                              â”‚
                                    [AIOSS Ledger: full pipeline trace]
```

**ROI:** Eliminates AWS Step Functions + SQS combination: ~$800/month for comparable throughput. **$9,600/year saved**.

---

## Use Case 3: Distributed Agent Task Distribution

**Scenario:** Multi-agent system distributes subtasks across worker nodes via queues, with result aggregation and lineage tracking.

```python
from api_oss_queue import Queue, FifoQueue

task_queue = FifoQueue.connect("redis://localhost:6379", "agent-tasks")
result_queue = Queue.connect("redis://localhost:6379", "agent-results")

# Distribute tasks to agents
for task in decompose_goal("Analyse Q3 sales data"):
    task_queue.send({"task": task, "agent_id": assign_agent()})

# Collect results with AIOSS lineage
for result in result_queue.consume(timeout=30):
    ledger.append(entry_type="task_completed", actor="api-oss-queue",
                  content={"tokens_in": result["input_tokens"],
                           "tokens_out": result["output_tokens"],
                           "wall_time_ms": result["duration_ms"],
                           "cost_if_cloud_microcents": 0})
```

**ROI:** Cloud-hosted agent orchestration (e.g., LangSmith): $100â€“500/month. Local queue-based orchestration: $0. **$1,200â€“6,000/year saved**.
