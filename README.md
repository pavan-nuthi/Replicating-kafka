# Mini Kafka

A simplified Python implementation of a distributed message queuing system with 3 brokers, leader election, and message replication.

## How It Works

This system replicates Kafka's core functionality:

- **3 Broker Cluster**: Brokers run on ports 3125, 3126, 3127
- **Leader Election**: One broker acts as leader, handles all writes
- **Message Partitioning**: Messages distributed across 3 partitions per topic
- **Replication**: Leader replicates data to all other brokers
- **ZooKeeper Coordinator**: Monitors brokers and manages leader election

## 🏗️ System Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   ZooKeeper     │    │    Producer     │    │    Consumer     │
│   Coordinator   │    │                 │    │                 │
│   (Port: N/A)   │    │                 │    │  (Dynamic Port) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                    Broker Cluster                              │
├─────────────────┬─────────────────┬─────────────────────────────┤
│   Broker 1      │   Broker 2      │   Broker 3                  │
│   (Port: 3125)  │   (Port: 3126)  │   (Port: 3127)              │
│   ./server1/    │   ./server2/    │   ./server3/                │
└─────────────────┴─────────────────┴─────────────────────────────┘
```

## Key Components

- **ZooKeeper** (`zookeeper.py`): Monitors brokers and elects leader
- **3 Brokers** (`broker.py`, `broker1.py`, `broker2.py`): Handle messages on ports 3125-3127
- **Producer** (`kafka-producer.py`): Sends messages to current leader
- **Consumer** (`kafka-consumer.py`): Subscribes to topics and receives messages

## Quick Start

1. **Setup**:
   ```bash
   echo "3125" > global.txt
   echo "3180" > global1.txt
   mkdir -p server1 server2 server3
   ```

2. **Start system** (4 terminals):
   ```bash
   python3 zookeeper.py    # Terminal 1
   python3 broker.py       # Terminal 2  
   python3 broker1.py      # Terminal 3
   python3 broker2.py      # Terminal 4
   ```

3. **Produce messages**:
   ```bash
   python3 kafka-producer.py
   ```

4. **Consume messages**:
   ```bash
   python3 kafka-consumer.py
   ```

## Core Workflow

1. **Leader Election**: ZooKeeper monitors broker ports and selects leader
2. **Message Production**: Producer sends messages to current leader (stored in `global.txt`)
3. **Partitioning**: Messages distributed across 3 partitions using round-robin
4. **Replication**: Leader copies all data to other brokers after each message
5. **Consumption**: Consumers register with leader and receive real-time notifications

## Data Storage

- Messages stored in `./server{1,2,3}/topic/partition{1,2,3}/` directories
- Each message saved as numbered `.txt` files (1.txt, 2.txt, etc.)
- Full directory replication keeps all brokers synchronized

## Limitations

- Simple port-based leader election (no sophisticated consensus)
- Full directory replication after each message (inefficient)
- Fixed 3-broker, 3-partition architecture
- No fault tolerance or authentication

---
