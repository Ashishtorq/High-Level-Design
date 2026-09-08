# Rate Limiting in System Design

This video provides an educational overview of **rate limiting**, explaining its necessity and key algorithms used to manage API traffic and protect server resources.

## What Is Rate Limiting?

Rate limiting is a technique used to control the rate of requests a user can send to a server. It prevents system abuse, such as spamming, and protects server resources from being overwhelmed, ensuring availability for genuine users. When a limit is exceeded, the server typically returns a **429 Too Many Requests** error.

## Rate Limiting Strategies

### 1. Token Bucket Algorithm

- **Concept:** A bucket holds a fixed number of tokens. A refiller adds tokens at a constant rate. Each request requires one token to proceed.
- **Process:** Users consume tokens to make requests. If the bucket is empty, the request is rejected.
- **Drawback:** If users share a single bucket, one user can consume all its tokens quickly, leaving other users unable to make requests until more tokens are added.

### 2. Leaky Bucket Algorithm

- **Concept:** Similar to a funnel, requests enter a bucket at an arbitrary rate but leak out, or are processed, at a constant, steady rate.
- **Process:** The bucket acts as a buffer that smooths out bursty traffic, ensuring the server receives a consistent stream of requests regardless of how quickly users send them.
- **Drawback:** If the bucket reaches capacity, incoming requests are dropped. Buffered requests may experience significant delays, potentially leading to response timeouts.

## Conclusion

The video concludes by noting that other strategies, including **Fixed Window Counter** and **Sliding Window Log**, will be covered in a future part of the series.
