# BGP Simulation with BIRD and Docker - SIT716 Assignment

This repository contains a customized BGP simulation using **BIRD** (BGP Internet Routing Daemon) for the **SIT716 - Computer Networks and Security** assignment. The goal is to demonstrate a **normal operation of BGP** and simulate a **BGP prefix hijacking attack** using a three-router topology.

## Topology: R3 → R1 → R2

- **Router 3 (R3 - AS 64514)** connects to **Router 1 (R1 - AS 64512)** via `peeringbr1 (10.0.0.0/24)`
- **Router 1 (R1 - AS 64512)** connects to **Router 2 (R2 - AS 64513)** via `peeringbr1 (10.0.0.0/24)`
- No direct connection between R3 and R2

## Role of Routers:
- Router 1 (R1): Peer with both Router 2 and Router 3.
- Router 2 (R2): Advertise the legitimate prefix 10.0.0.0/24.
- Router 3 (R3): Initially advertise a fake route (192.168.100.0/24), followed by hijacking the legitimate prefix 10.0.0.0/24 from Router 2.

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
- **Wireshark** (for packet capture)

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
docker exec -it bgp-routing-sim_router1_1 birdc show protocols all
docker exec -it bgp-routing-sim_router2_1 birdc show protocols all
docker exec -it bgp-routing-sim_router3_1 birdc show protocols all

```

### 5. Checking Routing Table
```sh
docker exec -it bgp-routing-sim_router1_1 birdc show route
```

### 6. Capture and Analyze BGP Traffic
Run Wireshark and capture packets on the Docker bridge interface (`peeringbr1`). Filter the captured traffic by BGP.

## Expected Results
- **BGP Sessions Established**: Routers exchange `OPEN`, `UPDATE`, and `KEEPALIVE` messages.
- **Routing Table Population**: Each router learns the routes advertised by its BGP peers.
  

# BGP Prefix Hijacking Simulation
Router3 will simulate a prefix hijack by advertising both a non-owned prefix (192.168.100.0/24) and later hijacking a legitimate prefix (10.0.0.0/24) from Router2. This simulates a real-world BGP hijacking attack.

### 1. Simulate the BGP Hijack
Hijack with an Unused Prefix (192.168.100.0/24): Configure Router 3 to advertise a fake prefix (192.168.100.0/24) using the following configuration in bird.conf:
```sh
protocol bgp router1 {
    local as 64514;
    neighbor 10.0.0.10 as 64512;
    multihop;
    import all;
    export where net ~ [ 192.168.100.0/24 ];
}
```
Reload the configuration:
```sh
birdc configure
```

Hijack the Legitimate Prefix (10.0.0.0/24): Configure Router 3 to advertise the legitimate prefix (10.0.0.0/24) using the following configuration in bird.conf:
```sh
protocol static {
    route 10.0.0.0/24 via "eth0";
}

protocol bgp router1 {
    local as 64514;
    neighbor 10.0.0.10 as 64512;
    import all;
    export where net = 10.0.0.0/24;
}
```
Reload the configuration:
```sh
birdc configure
```
Verify the BGP session status:
```sh
birdc show protocols all
```
### 2. Confirm Hijacked Route on Router 1
```sh
docker exec -it bgp-routing-sim_router1_1 birdc show route
```
### 3. Capture the Hijacked UPDATE Message in Wireshark
Ensure Wireshark is still capturing traffic. You should now see the following BGP messages:
- BGP UPDATE messages sent from Router 3 (10.0.0.12) to Router 1 (10.0.0.10), containing the hijacked prefix 10.0.0.0/24.
- KEEPALIVE messages and ACKs in response to maintain the session.
- The hijacked route (10.0.0.0/24) in the NLRI field of the UPDATE message.

## Expected Results
- **BGP Sessions Established**: Routers exchange `OPEN`, `UPDATE`, and `KEEPALIVE` messages.
- **Routing Table Population**: Each router learns the routes advertised by its BGP peers, but Router 1 incorrectly accepts the hijacked route from Router 3.

## SIT716 Assignment Context
This setup is designed to demonstrate both **normal BGP operation** and **BGP prefix hijacking** as required for SIT716 Tasks 2.2C, 2.3D, and 2.4HD. The experiment is conducted using Docker and BIRD to simulate the BGP routers, and Wireshark was used to capture and analyze the traffic. The results show the vulnerability of BGP to prefix hijacking attacks, and the experiment highlights the importance of implementing BGP security measures like RPKI and prefix filtering.

## References
- [BIRD Documentation](https://bird.network.cz/)
- [Docker Networking](https://docs.docker.com/network/)
- [Wireshark BGP Analysis](https://wiki.wireshark.org/BGP)
- [BGP version 4 - RFC 4271](https://datatracker.ietf.org/doc/html/rfc4271)
- [ROA and RPKI - RFC 6480](https://datatracker.ietf.org/doc/html/rfc6480)
- [BGP Prefix Hijacking - BGPMon](https://www.bgpmon.net/)
