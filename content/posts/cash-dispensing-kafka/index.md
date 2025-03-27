+++
title = "Scalable Cash Dispensing via Kafka Concurrency"
publishDate = 2025-03-27T00:00:00+01:15
lastmod = 2025-03-27T11:24:47+05:45
tags = ["python", "asyncio", "kafka"]
categories = ["system-design", "architecture", "concurrency"]
draft = false
featuredImage = "images/cover-image-cash-dispenser.jpeg"
featuredImagePreview = "./posts/cash-dispensing-kafka/images/designer.jpeg"
+++


We've all experienced that moment of relief when an ATM successfully dispenses our cash. But behind that simple transaction lies a complex dance of systems working together. In this post, I'll walk you through how I tackled the challenge of processing cash dispense requests at scale using Kafka and Python's asyncio.

This approach grew out of a real problem I faced at work recently - how to allow multiple payment transactions to happen simultaneously across different locations while making sure we never had conflicting operations at any single cash machine. The solution I developed turned out to be both elegant and robust, combining message streaming with a clever concurrency control mechanism that I'm excited to share with you.

{{< admonition type=info title="The Challenge" open=true >}}
Imagine you're managing a network of cash machines across different locations, and you need to process dozens of transactions every second. When I started tackling this problem, I quickly realized several thorny challenges:

1. **Concurrent Operations**: How do we process transactions from different locations simultaneously to keep things fast?
2. **Location Safety**: We absolutely can't have multiple operations happening on the same physical machine at once - that's how money goes missing!
3. **Fault Tolerance**: If something crashes mid-transaction, we need to recover without losing track of anyone's money
4. **Resource Efficiency**: These systems need to run efficiently on limited hardware resources
{{< /admonition >}}


## My Journey to a Solution

My first instinct was pretty straightforward - just spin up multiple Hypercorn workers to handle cash transactions concurrently. More workers should mean more throughput, right?

That approach quickly fell apart when I tested it. With multiple workers processing messages independently, I ended up with redundant hardware operations - the same cash machine trying to dispense money twice for the same transaction! Not exactly what you want in a financial system.

So I swung to the opposite extreme. "What if I just make everything strictly synchronous?" Sure, it would be slower, but at least transactions would be recorded with proper balances and we wouldn't have machines dispensing phantom cash.

{{< figure src="images/no-deadlocks.jpeg" caption="*Epic Amateur Programmer Moment* 😭" >}}

But forcing everything to be sequential created a massive bottleneck. Transactions at Location A had to wait for transactions at Location B to complete, even though they had nothing to do with each other. Users were waiting unnecessarily, and the system couldn't scale.

That's when I realized I needed something in between - parallel processing where it's safe, but strict controls where it matters. And that's exactly what led me to the solution I'm about to share.

## Our Solution: Location-Based Semaphores with Asyncio

{{< figure src="images/Cash Microservice - Paralle Processing Per Location Advanced.svg" caption="*Very long Typescript Error* 😭" >}}

The heart of our solution combines Kafka's distributed messaging with Python asyncio's concurrency primitives to create a system that processes transactions in parallel while maintaining safety constraints.

### Key Components

#### 1. Location-Based Concurrency Control

```python
# Dictionary to track location-specific tasks
location_tasks = {}
# Dictionary to track location-specific semaphores
location_semaphores = {}
# Default maximum concurrent transactions per location
MAX_CONCURRENT_PER_LOCATION = 1
```

This approach creates a dedicated semaphore for each location, allowing the system to:
- Process transactions from different locations in parallel
- Ensure only one transaction executes at a time for any given location

#### 2. Dynamic Task Management

```python
# Create a semaphore for this location if it doesn't exist yet
if location not in location_semaphores:
    location_semaphores[location] = asyncio.Semaphore(MAX_CONCURRENT_PER_LOCATION)
    location_tasks[location] = []

# Create and track the task
task = asyncio.create_task(process_with_semaphore(data))
location_tasks[location].append(task)
```

The system dynamically creates and tracks tasks for each location, automatically cleaning up completed tasks to prevent memory leaks.

#### 3. Semaphore-Protected Processing

```python
async def process_with_semaphore(msg_data):
    try:
        async with location_semaphores[location]:
            return await process_message(msg_data)
    finally:
        # Clean up - remove this task from the location's task list when done
        task = asyncio.current_task()
        if location in location_tasks and task in location_tasks[location]:
            location_tasks[location].remove(task)
```

This elegant pattern ensures that:
- Only one transaction per location is processed at a time
- Tasks are automatically removed from tracking when completed
- Resource cleanup happens even if exceptions occur

## Kafka Integration for Reliable Messaging

Our solution leverages Kafka's robust message queuing capabilities:

```python
current_app.kafka_consumer = await create_consumer(group_id="cash-dispense-service-group")

async for message in current_app.kafka_consumer:
    # Process messages here
```

The Kafka consumer includes:
- Retry mechanisms for connection failures
- Proper error handling for malformed messages
- Environment-specific configurations (development vs. production)
- Secure connections with SSL and SASL in production

## Error Recovery and Resilience

To ensure system stability, our implementation includes comprehensive error handling:

```python
except ConsumerStoppedError:
    current_app.logger.warning("Consumer stopped, attempting to restart...")
    await cleanup_consumer(current_app.kafka_consumer)
    current_app.kafka_consumer = None
    await asyncio.sleep(5)
```

This approach provides:
- Automatic restarts after consumer failures
- Proper cleanup of resources during errors
- Graceful handling of JSON parsing errors
- Detailed logging for troubleshooting

## Performance Benefits

This architecture delivers several key benefits:

1. **Horizontal Scaling**: Processing transactions from different locations in parallel
2. **Safety Guarantees**: Preventing conflicting operations at the same location
3. **Resource Efficiency**: Dynamic task management prevents resource leaks
4. **Robustness**: Comprehensive error handling and recovery mechanisms

## Conclusion

By combining Kafka's distributed messaging with asyncio's concurrency controls, we've created a system that efficiently processes cash dispense operations at scale while maintaining the strict safety requirements needed for financial transactions.

This pattern can be applied to many scenarios where you need to balance parallelism with location-specific safety constraints, such as inventory management, resource allocation, or any system where physical resources must be accessed in a controlled manner.

Through intelligent use of semaphores, task tracking, and error handling, we've built a solution that maximizes throughput while maintaining the reliability required for mission-critical financial operations.

