+++
title = "Scalable Transaction Processing via Kafka Concurrency"
publishDate = 2025-04-25T00:00:00+01:15
lastmod = 2025-04-25T11:24:47+05:45
tags = ["python", "asyncio", "kafka"]
categories = ["system-design", "architecture", "concurrency"]
draft = true
featuredImage = "images/cover-image-transaction-processing.jpeg"
featuredImagePreview = "./posts/cash-dispensing-kafka/images/Designer.jpeg"
+++

We've all experienced that moment of waiting for a transaction to complete. Behind that simple operation lies a complex dance of systems working together. In this post, I'll walk you through how I tackled the challenge of processing financial transactions at scale using [Apache Kafka](https://kafka.apache.org/intro#intro_nutshell) and Python's [asyncio](https://docs.python.org/3/library/asyncio.html).

This approach grew out of a real problem I faced at work recently - **how to allow multiple transactions to happen simultaneously across different locations while making sure we never had conflicting operations at any single processing point**. The solution I developed combined message streaming with a simple concurrency control mechanism that I'll explain in this blog post.

{{< admonition type=bug title="The Challenge" open=true >}}
Imagine you're managing a network of transaction processing systems across different locations, and you need to handle dozens of operations every second. When I started tackling this problem, I quickly realized several thorny challenges:

1. **Concurrent Operations**: How do we process transactions from different locations simultaneously to keep things fast?
2. **Location Safety**: We absolutely can't have multiple operations happening on the same physical system at once - that's how inconsistencies occur!
3. **Fault Tolerance**: If something crashes mid-transaction, we need to recover gracefully
{{< /admonition >}}

## My Journey to a Solution

My first approach was implementing multi-threading in Python, but knowing about the [Global Interpreter Lock (GIL)](https://leimao.github.io/blog/Python-GIL/ "Lei's awesome blog") deemed it not a good solution. The GIL prevents [true parallel execution](https://stackoverflow.com/a/74936772/6154579) of Python threads.

{{< figure src="images/python-gil-meme.png" caption="*Python GIL Sucks* 😭" >}}

I then explored spawning multiple [Hypercorn workers](https://hypercorn.readthedocs.io/en/latest/discussion/workers.html#workers) to handle transactions concurrently. More workers should mean more throughput, right?

That approach quickly fell apart when I tested it. With multiple workers processing messages independently, I ended up with redundant operations - the same physical system trying to process the same transaction twice! Not exactly what you want in a financial system.

So I swung to the opposite extreme. *"What if I just make everything strictly synchronous?"* Sure, it would be slower, but at least transactions would be recorded properly and we wouldn't have inconsistent states.

But forcing everything to be sequential created a massive bottleneck. Transactions at Location A had to wait for transactions at Location B to complete, even though they had nothing to do with each other. Users were waiting unnecessarily, and the system couldn't scale.

{{< figure src="images/waiting-for-transaction.jpeg" caption="" >}}

That's when I realized I needed something in between - parallel processing where it's safe, but strict controls where it matters.

## My Solution: Location-Based Semaphores with Asyncio

{{< mermaid >}}
flowchart TD
    subgraph KafkaBroker["Apache Kafka Broker"]
        topic["Topic: transactions\nWith Multiple Partitions"]
        partitioning["Messages Partitioned by Location\n(Hash of location key)"]
        partition0["Partition 0\nLocation A,D,G"]
        partition1["Partition 1\nLocation B,E,H"]
        partition2["Partition 2\nLocation C,F,I"]

        topic --> partitioning
        partitioning --> partition0
        partitioning --> partition1
        partitioning --> partition2
    end

    consumer["Python Consumer Service"]
    processing["Message Processing"]
    semaphores["Location-Based Semaphores (Control\nconcurrent processing)"]

    partition0 --> consumer
    partition1 --> consumer
    partition2 --> consumer
    consumer --> processing
    processing --> semaphores

    locA["Location A\nProcessing"]
    locB["Location B\nProcessing"]
    locC["Location C\nProcessing"]

    semaphores --> locA
    semaphores --> locB
    semaphores --> locC

    sysA["System\nLocation A"]
    sysB["System\nLocation B"]
    sysC["System\nLocation C"]

    locA --> sysA
    locB --> sysB
    locC --> sysC

    compA["Complete Transaction\nLocation A"]
    compB["Complete Transaction\nLocation B"]
    compC["Complete Transaction\nLocation C"]

    sysA --> compA
    sysB --> compB
    sysC --> compC

    style partition0 fill:#ffccff
    style partition1 fill:#ffccff
    style partition2 fill:#ffccff
    style locA fill:#99ffcc
    style locB fill:#99ffcc
    style locC fill:#99ffcc
    style semaphores fill:#ffffcc
{{< /mermaid >}}

The heart of our solution combines Kafka's distributed messaging with Python asyncio's concurrency primitives to create a system that processes transactions in parallel while maintaining safety constraints.

{{< admonition type=tip title="What's a Semaphore?" open=true >}}
Think of a [semaphore](https://docs.python.org/3/library/threading.html#semaphore-objects) like a bouncer at a nightclub with a strict capacity limit. If the club can only safely hold 50 people, the bouncer keeps count and only lets new people in when others leave. In programming terms, a semaphore is a variable that controls access to a shared resource by multiple processes - in our case, limiting access to each location's processing system.
{{< /admonition >}}

In simpler terms, we built a system that can handle many transactions at once (like a multi-lane highway) but makes sure that no two transactions try to access the same system at the same time (no lane crossing). This gives us speed without sacrificing safety.

Let me walk through the key components:

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
This approach creates a dedicated semaphore for each location, allowing the system to process transactions from different locations in parallel while ensuring only one transaction executes at a time for any given location. Like having independent checkout lines at different momo stalls, customers can get tasty momos served simultaneously across the network, but each individual stall carefully manages its own line to prevent confusion.

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
Think of this part as an organised filing system that automatically creates a new folder the first time we encounter a new location. Then, whenever a customer creates a new transaction, their request gets placed in the appropriate location’s folder and tracked until completion. Once their transaction is done, we remove their paperwork from the folder to keep things tidy — no clutter, no lost transactions.

#### 3. Semaphore-Protected Processing

```python
async def process_with_semaphore(msg_data):
    try:
        async with location_semaphores[location]:
            return await process_message(msg_data)
    finally:
        # Clean up when done
        task = asyncio.current_task()
        if location in location_tasks and task in location_tasks[location]:
            location_tasks[location].remove(task)
```

Here’s where the magic happens!🪄Think of this like a security gate that only allows one person through at a time. When a transaction arrives, it waits its turn until it gets the green light to proceed. Once it’s done, it cleans up after itself by removing its record from our tracking system.

The benefit of this approach? Transactions from different locations run simultaneously, but transactions for the same location politely wait their turn. And if something goes wrong, we still clean up properly — no loose ends!

## Kafka Integration and Resilience
Our solution leverages [Kafka's robust message queuing capabilities](https://www.svix.com/resources/guides/kafka-message-queue/#what-makes-kafka-a-unique-message-queue), which forms the backbone of our transaction processing architecture. When a transaction comes in, Kafka ensures it's not lost and can be properly tracked even if something goes wrong. I implemented a consumer group that processes these messages asynchronously:

```python
async def consume_messages():
    while True:
        try:
            async with app_context():
                consumer = await create_consumer(group_id="transaction-service-group")

                async for message in consumer:
                    # Process message with location-based semaphore protection
                    data = parse_message(message)
                    location = str(data["location"])

                    # Create task with semaphore protection
                    if location not in location_semaphores:
                        location_semaphores[location] = asyncio.Semaphore(MAX_CONCURRENT_PER_LOCATION)
                        location_tasks[location] = []

                    task = asyncio.create_task(process_with_semaphore(data))
                    location_tasks[location].append(task)
        except ConnectionError:
            logger.warning("Connection lost, attempting to reconnect...")
            await asyncio.sleep(5)  # Wait before trying again
```

The system handles connection failures with retry mechanisms and includes robust error handling. If something breaks mid-transaction, our error recovery ensures proper cleanup and logging.

{{< admonition type=success title="Performance Benefits💪" open=true >}}
The real win here wasn’t just creating a technical solution — it actually solved real business problems for our client. Our Transaction Processing system went from handling just a handful of transactions to processing dozens per second across all our locations simultaneously, while still keeping each individual machine safe/transaction records safe from conflicts.

When we first deployed this to production, I was surprised by how much throughput improved. Transactions that used to get backed up during peak hours now sailed through without a hitch. And the best part? We didn’t need to add any extra hardware to make it happen. You’re welcome, DevOps team! 😉

I was particularly nervous about failure scenarios (there’s nothing worse than a dispenser incorrectly processing a transaction!), but after a few weeks in production, the error recovery has been rock solid. We’ve had network blips and even a few server restarts, but transactions either complete correctly or roll back safely.
{{< /admonition >}}

## Conclusion

So there you have it - by combining Kafka's distributed messaging with asyncio's concurrency controls, I built a system that handles transactions at scale while keeping everything consistent and reliable.

{{< figure src="images/kafka-complexity.png" caption="*Managing Kafka is complex* 😖">}}

Could I have solved this with a simpler approach? Possibly. But since Apache Kafka was already the backbone of our microservice architecture, it made sense to leverage what we had rather than introduce yet another technology. Does Kafka add complexity? Absolutely - the learning curve is steep. But for our needs, this complexity was a necessary trade-off when weighed against the performance and scalability benefits.

This pattern has already proven useful beyond just transaction processing. We've applied similar techniques to other systems where we needed that careful balance between parallelism and safety constraints.

I'm quite satisfied with how this turned out. Through use of semaphores, careful task tracking, and robust error handling, I built something that's both fast and reliable.

Let me know your thoughts in the comments - how would you approach this problem? Would you go with a similar Kafka + asyncio solution, or take a completely different path? 🤔