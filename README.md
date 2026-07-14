> [!IMPORTANT]
> This repository is intended as a technical portfolio and learning resource. All domain concepts, workflows, and architectural examples are generalized and anonymized. No proprietary source code, confidential information, or internal documentation from any employer or client is included.

# 1. nats-cluster-showcase

This repository showcases a small scalable messaging infrastructure pattern based on my real-world experience implementing NATS in corporate environments. It serves as the foundational core for multi-region connectivity and edge computing.

While this repository focuses on infrastructure, additional repositories will demonstrate practical applications built on top of this architecture, such as ETL pipelines, remote device management, observability platforms, and industrial automation services.

## 1.1 NATS

[NATS](https://nats.io/) is an open-source, highly performant, lightweight, and secure messaging system designed for modern distributed systems. Delivered as a single binary, it acts as a connective tissue for microservices, edge computing, and IoT devices, completely decoupling producers from consumers.

### 1.1.1 Capabilities

NATS supports multiple communication patterns, allowing for flexible architecture designs:

- **Publish/Subscribe:** 1-to-N communication. Publishers send messages to a subject, and any active subscriber listening receives them instantly.
- **Request/Reply:** 1-to-1 or 1-to-N communication simulating an asynchronous API call
- **Point-to-Point (queue groups):** Distributed queueing where only one subscriber in a designated group processes a message, providing automatic load balancing.

### 1.1.2 JetStream (persistence)

- **Persistent streams** with at-least-once, at-most-once, or exactly-once delivery semantics.
- **KV & Object Stores** built natively into the cluster, eliminating the need for external caching or storage dependencies like Redis or S3 for many use cases.
- **Stream replication** across multiple regions for disaster recovery and high availability.

##  1.2 Infrastructure

This repository demonstrates a distributed NATS topology composed of:

- A central NATS cluster
- Multiple geographically distributed Leaf Nodes
- Local JetStream persistence at the edge
- Clients and services connected to regional infrastructure

Leaf nodes act as local bridges that securely connect isolated regions, private networks, or edge devices to the central global cluster. This allows local traffic to stay local (reducing latency and bandwidth costs) while forwarding necessary messages to the broader network, even supporting offline operations if the connection to the central cluster drops.

```
                      ┌────────────────────────────────────────────┐                               
                      │                                            │                               
                      │               NATS Cluster                 │                               
                      │                                            │                               
                      │                                            │                               
                      │    ┌────────┐    ┌────────┐    ┌─────────┐ │                               
                      │    │ node 1 │    │ node 2 │    │  node 3 │ │                               
                      │    └────────┘    └────────┘    └─────────┘ │                               
                      │                                            │                               
                      └────────────────────────────────────────────┘                               
                                 ▲                 ▲            ▲                                  
                                 │                 │            │                                  
                                 │                 │            │                                  
                ┌────────────────┘                 │            │                                  
                │                                  │            │          REGION 3                
  REGION 1      │                                  │            │         ┌─────────────────────┐  
  ┌─────────────┼───────┐                          │            │         │ Leafnode N          │  
  │ Leafnode 1          │               REGION 2   │            └─────────┤                     │  
  │                     │               ┌──────────┴──────────┐           │    ┌───────────┐    │  
  │    ┌───────────┐    │               │ Leafnode 2          │           │    │ JetStream │    │  
  │    │ JetStream │    │               │                     │           │    │           │    │  
  │    │           │    │               │    ┌───────────┐    │           │    └───────────┘    │  
  │    └───────────┘    │               │    │ JetStream │    │           └─────────────────────┘  
  └─────────────────────┘               │    │           │    │                          ▲         
            ▲                           │    └───────────┘    │                          │         
            │                           └─────────────────────┘                     ┌────┴────┐    
      ┌─────┴───┐                                  ▲                                │ Client  │    
      │ Client  │                                  │                                └─────────┘    
      └─────────┘                            ┌─────┴───┐                                           
                                             │ Client  │                                           
                                             └─────────┘                                           
```

## 1.3 Example use cases

Although this repository only provides the infrastructure layer, it can serve as the foundation for many different systems.

```
                                                       ┌─────────┐                                                                     
                                                       │         │                                                                     
                                                       │ Edge    │                                                                     
                                                       │ Device  │                                                                     
                                                       │         │                                                                     
                                                       └─────────┘                                                                     
                                                            ▲                                                                           
                                                            │                                                                           
                                                            │                                                                           
                                                            │                                                                           
                                                            │                                                                           
                                              xxxxxxxxxxxxxx│xxxxxxxxxxxxxx                                                             
       ┌───────┐                             xx                           xx                                                            
       │       │                            xx                             x                                                            
       │  User │      Remote commands       x                              x                                                            
       │       ┼───   to ´Edge Device´ ───► x      NATS Cloud Cluster      x                         ┌───────────────────────────────┐  
       └───────┘                            x                              x │                       │                               │  
                                             x                            x  └──── ETL Pipeline ────►│  Central PostgreSQL Database  │  
                                             xxx                         xx                          │                               │  
                                               xxxxxxxx xxxxxxxxxxxxxxxxx                            └──────────────────┬────────────┘  
                                                                │                                                       │               
                                                         ▲      │                                                       │               
                                                         │      │                                              Historical data          
                                                         │      │                                                       │               
                                                         │      │                                                       ▼               
                                                         │      │                                                  ┌────────┐           
                                                         │      │                                                  │        │           
                                                         │      │                                                  │  User  │           
                                                         │      │         ┌────────────────────┐                   │        │           
  ┌──────────────────┐                                   │      │         │                    │                   └────────┘           
  │                  │              ┌───────────────┐    │      │         │  IoT Observability │                                                
  │                  │              │  Persistency  │    │      └────────►│                    │                                                
  │    IoT devices   ┼────  Data ──►│  (JetStream)  ┼────┘                └───────────┬────────┘                                                
  │                  │              └───────────────┘                                 ▼                                                         
  │                  │                                                           ┌─────────┐                                                     
  └──────────────────┘                                                           │         │                                                     
                                                                                 │ Grafana │                                                     
                                                                                 │         │                                                     
                                                                                 └────┬────┘                                                     
                                                                                      │                                                          
                                                                                  Dashboard                                                      
                                                                                      │                                                          
                                                                                      │                                                          
                                                                                      ▼                                                          
                                                                                  ┌────────┐                                                     
                                                                                  │        │                                                     
                                                                                  │  User  │                                                     
                                                                                  │        │                                                     
                                                                                  └────────┘
```

### 1.3.1 Remote device management

Send commands from a centralized cloud platform to edge devices deployed anywhere in the world.

`User → NATS Cloud Cluster → Edge Device`

Examples:

Device commissioning
Firmware deployment
Remote diagnostics
Service orchestration
IoT data collection

Devices publish telemetry to local JetStream streams.

`IoT Devices → JetStream → Consumers`

Benefits:

- Reliable delivery
- Offline buffering
- Replay capabilities
- Historical data retention

### 1.3.2 ETL pipelines

NATS can act as a transport layer between data producers and processing services.

`IoT Devices → JetStream → ETL Services → PostgreSQL`

Typical use cases:

- Time-series ingestion
- Data normalization
- Aggregations
- Historical storage

### 1.3.3 Observability Platforms

Telemetry can be consumed directly from NATS and visualized through monitoring tools.

`IoT Devices → JetStream → Observability Services → Grafana`

Possible metrics:

- Device health
- Connectivity status
- Operational KPIs
- Alarm monitoring
- Production metrics




The main architecture consists of a NATS cluster running in the cloud, with 3 nodes, connecting each plant to a single environment to be able to exchange NATS messages between servers.

Each plant functions as a client, connecting the necessary services to a local NATS server called a Leaf Node, which in turn routes the traffic to the cluster, serving as a single communication bridge between the local services and the cloud infrastructure.

From a security standpoint, all clients (including the Leaf Nodes) must authenticate to the main NATS server. This authentication is done using a credentials file that contains the client's private key (which confirms its identity) and a Jason Web Token (JWT) that contains the claims defining each client's capabilities, signed by the Account that issued it.

In addition to authentication via credentials, the external communication between the cloud and local services/plants must be encrypted with TLS. This requires the issuance of certificates and the existence of a Certificate Authority (CA) to sign them.

Usage of `nsc` is explained in the `nsc usage.md` file.


## 1.4 Usage

The `docker-compose.yaml` initiates three individual NATS servers, which are configured to compose a three-node cluster.

Cluster:

```conf

services:
  cluster-node-1:
    container_name: "n1.nats.cluster"
    image: nats:2.12.1
    volumes:
      - ./nats-server-1.conf:/etc/nats/nats-server.conf
    command: ["-c", "/etc/nats/nats-server.conf"]
    networks: 
      - nats_network

  cluster-node-2:
    container_name: "n2.nats.cluster"
    image: nats:2.12.1
    volumes:
      - ./nats-server-2.conf:/etc/nats/nats-server.conf
    command: ["-c", "/etc/nats/nats-server.conf"]
    networks: 
      - nats_network

  cluster-node-3:
    container_name: "n3.nats.cluster"
    image: nats:2.12.1
    volumes:
      - ./nats-server-3.conf:/etc/nats/nats-server.conf
    command: ["-c", "/etc/nats/nats-server.conf"]
    networks: 
      - nats_network

networks:
  nats_network:
    name: nats-network
    driver: bridge
    attachable: true
```

The folders' structure:

```
.
├── cluster
│   ├── docker-compose.yml
│   ├── nats-server-1.conf
│   ├── nats-server-2.conf
│   └── nats-server-3.conf
├── leafnodes
│   ├── docker-compose.yml
│   ├── leafnode-a.conf
│   └── leafnode-b.conf
├── nats-tools
│   └── docker-compose.yaml
└── README.md
```

Start the core cluster:

`docker compose -f ./cluster/docker-compose.yml up -d`

Start the leaf nodes:

`docker compose -f ./leafnodes/docker-compose.yml up -d`

The nats-client is used as a container to NATS commands, to avoid having to install NATS in the OS. To use it, start the container and access it:

```
docker compose -f ./nats-tools/docker-compose.yml up -d
docker exec -it nats-cli sh
```

Inside the nats-client, subscribe to a topic:

`nats sub -s nats://leafnode-a:4111 telemetry.>`

Publish a message in another instance:

`nats pub -s nats://leafnode-b telemetry.test "hello"`

