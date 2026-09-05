# Networking Protocols — Lecture Notes

## Overview

Networking protocols are standardized rules that allow devices and software to communicate across a network. This lecture explains how communication works through three layers of the OSI model: the network, transport, and application layers.

## The Communication Flow

When one application sends data to another device:

1. The **application layer** defines the format and rules of the communication.
2. The **transport layer** decides how the data should be delivered between applications.
3. The **network layer** identifies the destination device and routes the data to it.

At the receiving device, the process is reversed and the data is delivered to the intended application.

## 1. Network Layer

The network layer is responsible for:

- Identifying devices on a network
- Determining where the destination device is located
- Routing packets across one or more networks

### IP Addresses

An **IP address** is a logical address used to identify a device and route data to it.

#### IPv4

- Uses 32-bit addresses
- Usually written as four decimal numbers separated by dots
- Example: `192.168.1.10`
- Has a limited address space

#### IPv6

- Uses 128-bit addresses
- Usually written as hexadecimal groups separated by colons
- Example: `2001:db8::1`
- Provides a much larger address space than IPv4

### Core Idea

The network layer answers: **Which device should receive the data, and how can the data reach it?**

## 2. Transport Layer

The transport layer manages communication between applications running on different devices. Its responsibilities can include:

- Establishing and maintaining a connection
- Splitting data into smaller units
- Controlling delivery order
- Detecting loss or errors
- Delivering data to the correct application

The two main transport protocols covered are TCP and UDP.

### TCP: Transmission Control Protocol

TCP is a **connection-oriented** protocol. It establishes a connection before application data is exchanged.

#### Characteristics

- Reliable delivery
- Ordered delivery
- Detection and retransmission of lost data
- More protocol overhead and generally higher latency than UDP

TCP is useful when correctness and completeness matter more than minimum delay, such as web pages, file transfers, and email.

### TCP Three-Way Handshake

Before sending application data, TCP establishes a connection through three messages:

1. **SYN:** The client asks to start a connection.
2. **SYN-ACK:** The server acknowledges the request and agrees to connect.
3. **ACK:** The client acknowledges the server's response.

After these steps, the TCP connection is established and data transfer can begin.

```text
Client                          Server
  | -------- SYN ------------> |
  | <------ SYN-ACK ----------- |
  | -------- ACK ------------> |
  |     Connection ready       |
```

### UDP: User Datagram Protocol

UDP is a **connectionless** protocol. It sends data without first creating a connection or confirming successful delivery.

#### Characteristics

- Low overhead
- Fast transmission
- No delivery guarantee
- No built-in retransmission of lost data
- No guarantee that packets arrive in order

UDP is useful for real-time applications where late data may be less valuable than missing data.

#### Common Use Cases

- Video calls
- Live streaming
- Online gaming
- Other latency-sensitive communication

### TCP vs. UDP

| Feature | TCP | UDP |
| --- | --- | --- |
| Connection | Connection-oriented | Connectionless |
| Reliability | Guaranteed through acknowledgements and retransmission | No built-in guarantee |
| Packet order | Preserved | Not guaranteed |
| Overhead | Higher | Lower |
| Typical priority | Accuracy and completeness | Speed and low latency |
| Common uses | Web pages, files, email | Calls, streaming, gaming |

## 3. Application Layer

The application layer defines how software applications exchange messages. It specifies details such as request formats, response formats, and the meaning of exchanged data.

### Client–Server Communication

In the client–server model:

- A **client** initiates a request.
- A **server** receives the request, processes it, and returns a response.

#### HTTP

HTTP is commonly used for communication between web clients and servers. It follows a request–response model:

1. The client sends an HTTP request.
2. The server processes it.
3. The server sends an HTTP response.

#### HTTPS and Encryption

HTTPS is HTTP protected by **TLS** (historically called SSL). TLS encrypts data in transit so that someone intercepting the traffic cannot easily read or modify it.

HTTPS mainly provides:

- **Confidentiality:** Data is encrypted.
- **Integrity:** Tampering can be detected.
- **Authentication:** Certificates help verify the server's identity.

> HTTPS protects data while it travels between endpoints. It does not automatically make an application, server, or client secure in every other respect.

#### WebSockets

WebSockets provide a long-lived, bidirectional communication channel between a client and server. Once the connection is established, either side can send messages without waiting for a new HTTP request.

They are useful for:

- Chat and messaging applications
- Live notifications
- Collaborative applications
- Real-time dashboards

### Peer-to-Peer Communication

In a **peer-to-peer (P2P)** model, devices can communicate directly with one another. Each participant can act as both a client and a server instead of relying on one central server for every data exchange.

Possible advantages include:

- Direct data transfer
- Distribution of workload across peers
- Reduced dependence on a central server

P2P systems may still use coordinating services for discovery, authentication, or connection setup.

## Key Comparisons

| Concept | Main purpose |
| --- | --- |
| IP | Identifies devices and routes packets |
| TCP | Provides reliable, ordered transport |
| UDP | Provides fast, low-overhead transport |
| HTTP | Defines web request–response communication |
| HTTPS | Protects HTTP traffic using TLS |
| WebSocket | Enables persistent, bidirectional client–server communication |
| P2P | Enables peers to communicate directly |

## Quick Revision

- The **network layer** finds the destination and routes packets using IP addresses.
- The **transport layer** controls end-to-end delivery using TCP or UDP.
- **TCP** prioritizes reliable, ordered delivery and begins with a three-way handshake.
- **UDP** prioritizes speed and low latency but does not guarantee delivery or ordering.
- The **application layer** defines how applications communicate through protocols such as HTTP, HTTPS, and WebSockets.
- **HTTPS** uses TLS to provide encryption, integrity, and server authentication.
- **P2P** enables direct communication between participating devices.

## Check Your Understanding

1. What responsibility does the network layer have?
2. Why does TCP use a three-way handshake?
3. When would UDP be preferable to TCP?
4. How does HTTPS differ from HTTP?
5. Why are WebSockets useful for chat applications?
6. How does peer-to-peer communication differ from the client–server model?
