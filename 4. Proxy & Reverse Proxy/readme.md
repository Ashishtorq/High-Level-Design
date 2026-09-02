# Proxy, Reverse Proxy, VPN, and Load Balancer

This note summarizes how forward proxies and reverse proxies work, why they are useful, and how they differ from VPNs and load balancers.

## The core idea

Both kinds of proxy are intermediaries. The difference is **which side they represent and hide**:

| Component | Sits in front of | Primarily represents/hides | Typical uses |
|---|---|---|---|
| Forward proxy | Clients | Client network and IP | Access control, privacy, filtering, caching |
| Reverse proxy | Servers | Origin servers and their topology | Routing, TLS termination, caching, security |
| Load balancer | Server pool | A group of equivalent service instances | Traffic distribution, health checks, failover |
| VPN | Client device or network | Client traffic from the local network and public destination | Encrypted transport to a VPN endpoint, private-network access |

> **Memory shortcut:** A forward proxy acts on behalf of clients. A reverse proxy acts on behalf of servers.

## 1. Forward proxy

![Forward proxy flow: clients connect through a proxy to internet services](forward-proxy.png)

A **forward proxy** sits between one or more clients and destinations on the internet. The client is configured to send requests to the proxy; the proxy then sends those requests onward.

From the destination server's point of view, the network connection usually comes from the proxy's public IP address rather than directly from the client.

### Request flow

1. The client sends a request to the forward proxy.
2. The proxy applies policy, such as authentication or URL filtering.
3. It may return a cached response when allowed and still fresh.
4. Otherwise, it contacts the destination using its own network connection.
5. The response returns through the proxy to the client.

### Common benefits

- **IP masking:** The destination sees the proxy's IP, though application data, cookies, logins, browser fingerprinting, and other signals can still identify the user.
- **Access control:** A company or school can allow, block, or audit destinations.
- **Caching:** Reusable responses can be served locally, reducing latency and outbound traffic.
- **Connection efficiency:** Some proxies pool connections or deduplicate identical work. They do not generally combine arbitrary user requests into one request.

Relevant video sections: definition at **1:22**, request flow at **3:17–4:56**, access control at **7:14–9:23**, request grouping at **10:00–11:00**, and caching at **12:28–13:41**.

## 2. Reverse proxy

![Reverse proxy flow: internet clients reach hidden origin servers through one public endpoint](reverse-proxy.png)

A **reverse proxy** is the public entry point in front of one or more origin servers. Clients connect to it without needing to know which internal server will handle the request.

### Request flow

1. A client connects to the service's public hostname.
2. DNS leads the client to the reverse proxy or an upstream load balancer.
3. The reverse proxy can terminate TLS, enforce security rules, log the request, or serve cached content.
4. It selects an internal service or server and forwards the request.
5. It returns the backend response as though it came from the public endpoint.

### Common benefits

- **Origin hiding:** Internal server addresses and topology are not directly exposed.
- **Routing:** Requests can be routed by hostname, path, headers, or other rules.
- **TLS termination:** The proxy handles HTTPS certificates and encryption at the edge. Traffic from proxy to backend must still be protected when the network is not trusted.
- **Caching and compression:** Static or reusable content can be returned without contacting the origin every time.
- **Security control point:** Rate limiting, web application firewall rules, authentication, and request-size limits can be applied centrally.
- **Load balancing:** Many reverse proxies can distribute requests across healthy backend servers.

A reverse proxy can reduce direct exposure to attacks, but it is **not complete DDoS protection by itself**. Capacity, upstream filtering, rate limits, and a distributed edge network are often needed for large attacks.

Relevant video sections: definition at **13:51**, origin hiding at **15:04**, load balancing at **15:56–16:42**, security at **16:52–17:36**, and TLS termination at **23:40**.

## 3. Load balancer

![Load balancer flow: traffic is distributed only to healthy servers](load-balancer.png)

A **load balancer** distributes traffic across a pool of servers that provide the same service. It tracks server health and avoids instances that are unavailable.

### Typical routing strategies

- **Round robin:** Send each new request to the next server in sequence.
- **Least connections:** Prefer the server currently handling fewer active connections.
- **Weighted routing:** Give more traffic to higher-capacity servers.
- **Hash-based routing:** Use a value such as client IP or a cookie to select a server consistently.

### Reverse proxy vs. load balancer

The categories overlap. A reverse proxy describes the **position and role** of an intermediary in front of servers. Load balancing describes a **traffic-distribution capability**.

- A reverse proxy may serve only one backend and perform no load balancing.
- A load balancer is often implemented as a reverse proxy, especially at the HTTP layer.
- A dedicated load balancer may operate at Layer 4 using IP addresses and TCP/UDP ports, while an HTTP reverse proxy usually operates at Layer 7 and understands URLs, headers, and cookies.
- Large systems can use both: an external or global load balancer first, followed by reverse proxies or ingress gateways closer to the application.

Relevant video comparison: **22:15–25:57**.

## 4. Proxy vs. VPN

| Question | Forward proxy | VPN |
|---|---|---|
| What traffic is covered? | Usually selected applications or protocols | Usually most or all traffic routed by the device, depending on configuration |
| What does the destination see? | The proxy's IP | The VPN endpoint's IP |
| Where is traffic encrypted? | Not guaranteed by the proxy itself; HTTPS still encrypts application traffic | Encrypted from the client to the VPN endpoint |
| What happens after the intermediary? | Proxy connects onward to the destination | VPN provider routes traffic onward to the destination |
| Typical purpose | Filtering, application-level routing, caching, IP masking | Secure access over untrusted networks, remote private-network access, IP masking |

Important nuance: a VPN does **not automatically provide end-to-end encryption all the way to the destination server**. It encrypts the tunnel from the device to the VPN endpoint. HTTPS or another end-to-end protocol is still needed to protect traffic between the VPN endpoint and the destination.

Relevant video comparison: **19:10–21:11**.

## 5. Example architecture

For a public web application, the path might look like this:

```text
User -> Global load balancer -> Reverse proxy / ingress -> Application instances -> Database
```

- The **global load balancer** sends the user to a healthy region or cluster.
- The **reverse proxy** terminates TLS and routes `/api`, `/images`, and other paths to the correct services.
- A **local load-balancing rule** spreads `/api` requests among healthy application instances.
- The **database** remains on a private network and is not exposed through the proxy.

## Final takeaway

- Use a **forward proxy** to control or mediate clients going out to the internet.
- Use a **reverse proxy** to control or mediate traffic coming into servers.
- Use a **load balancer** when traffic must be spread across multiple healthy service instances.
- Use a **VPN** when traffic needs an encrypted tunnel to another network endpoint.

These components are complementary. A production system may use all four at different points in the request path.
