# BGP Simulation with BIRD and Docker - SIT716 Assignment

This repository contains a customized BGP simulation using **BIRD** (BGP Internet Routing Daemon) for the **SIT716 - Computer Networks and Security** assignment. The goal is to demonstrate the **normal operation of BGP** using a three-router topology.

## Topology: R3 → R1 → R2

- **Router 3 (R3 - AS 64514)** connects to **Router 1 (R1 - AS 64512)** via `peeringbr2 (10.0.100.0/24)`
- **Router 1 (R1 - AS 64512)** connects to **Router 2 (R2 - AS 64513)** via `peeringbr1 (10.0.0.0/24)`
- No direct connection between R3 and R2

## Components

- **Docker Compose**: Used to create a virtual network of BGP routers.
- **BIRD**: A lightweight routing daemon implementing BGP.
- **Wireshark**: Used for capturing and analyzing BGP messages.
- **Debian-based containers**: Running BIRD for simulating BGP peering.

## Setup Instructions

### 1. Prerequisites
Ensure you have the following installed:
- **Docker**
- **Docker Compose**
- **Wireshark** (optional, for packet capture)

### 2. Clone the Repository
```sh
 git clone <repo_url>
 cd <repo_name>
```

### 3. Build and Start Containers
```sh
docker-compose up --build -d
```

### 4. Verify BGP Peering
#### Checking BGP Sessions
Log into any router and run:
```sh
docker exec -it router1 birdc show protocols all
```

#### Checking Routing Table
```sh
docker exec -it router1 birdc show route
```

### 5. Capture and Analyze BGP Traffic
Run Wireshark and capture packets on the Docker bridge interfaces (`peeringbr1`, `peeringbr2`).

## Expected Results
- **BGP Sessions Established**: Routers exchange `OPEN`, `UPDATE`, and `KEEPALIVE` messages.
- **Routing Table Population**: Each router learns the routes advertised by its BGP peers.

## SIT716 Assignment Context
This setup is designed to demonstrate **normal BGP operation** as required for **SIT716 Task 2.3D**. The next step will involve **analyzing captured BGP traffic** and documenting its behavior.

## References
- [BIRD Documentation](https://bird.network.cz/)
- [Docker Networking](https://docs.docker.com/network/)
- [Wireshark BGP Analysis](https://wiki.wireshark.org/BGP)