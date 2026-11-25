# Connection Polling

Connection polling is something which is always initiated by the client. The client periodically sends requests to the server to check if any new data or updates are available.

Some questions that you might ask:

1. Client periodically sends requests to server, what does that mean?
2. Why, when and how does the client periodically send requests to the server?

All these questions will be answered later in this article.

Before knowing the answers to these questions, tell me: how do backend and frontend connect to each other?

You might say through HTTP or WebSockets, right? You are correct! But how do these protocols (HTTP, WebSockets) connect backend and frontend to each other?

## Types of Connection Polling

### 1. Short Polling

Client asks the server for updates and new data, and the server responds (either "there is no data/update" or the new data/update). The connection terminates, and after a fixed interval of time the client asks for an update/data again, the server responds again, and this cycle goes on.

### 2. Long Polling

Client asks the server for updates and new data, and the server waits (if there is no data/update). Whenever the server has data it gives it to the client and the connection terminates; instantly the client asks for data/update again, and the server again waits.

The point to be noted here is that the connection does not terminate if there is no data. The server waits, and when the data is ready to move to the client it gives it, then the connection terminates and the client asks again. This causes a long-waited connection between client and server.

### 3. WebSockets

The updated version of HTTP.

**How a WebSocket connection is established**

1. First, a TCP connection is established with a 3-way handshake. After this handshake the client requests the server to upgrade their connection from HTTP to WebSockets.
2. Client sends a GET request with `Upgrade: websocket`.
3. Server responds with `101 Switching Protocols`.

With the help of WebSockets the connection between client and server stays alive (no connection termination happens), which means whenever required the client can send data to the server and the server can send data to the client. The client first asks for connection establishment to the server, and when the HTTP protocol is upgraded to WebSockets no connection termination happens between them.

### 4. SSE (Server-Sent Events)

Client first asks for the connection with the server, and when the connection is established the client does not send any request to the server. Only the server transmits data to the client. This is a one-way data transmission pipeline between the client and the server.

### 5. HTTP Streaming

Client requests the server for connection establishment. After the connection is established, the server sends the data in chunks or streams to the client. That is HTTP Streaming. But what if all the data has been sent? Then closing the connection or keeping it alive depends solely upon the application architecture.

## How is the connection established between client and server?

Client requests the server and if the request is successful then the connection is established. What happens underneath is not your concern; these are already designed, that's how the protocol works. Your concern is to understand connection polling and which type of connection polling is used in what type of applications.

Now, after understanding all the types of connection polling, you will get the answer to the question of how the client periodically sends the connection request to the server. As for _how_ it sends, that is the coding part, which I will not cover in this article. You will find code and related stuff on GitHub, other people's codebases, or you can just ask GPT for that.

## Why and When does the client periodically send the connection request to the server?

### Why

Let us take an example. Suppose you need to build a chat application, and for that both client and server need to connect with each other in real time. Client sends messages to the server and the server is supposed to process them. For that you need WebSockets, where client and server keep their connection open.

Let me elaborate on this example: when a user sends a message, the client passes that message to the server, the server processes it and notifies the client that the message is received.

If we are using short polling, then again and again the client will ask the server for confirmation of whether the message is received or not. (We cannot say that if the client has sent the message to the server then the message is 100% received, as we cannot rely on networks, they might fail, so the client asks the server directly whether the message is received or not.) If the server does not reply, then after a fixed interval of time the client will ask again. This will cause many calls to the server, which is highly inefficient.

You can say: use long polling, because when the server receives it, it will notify the client. Yes, you are right! In fact this is one of the great questions: long polling can be used. In terms of using a connection you can use short polling as well, because ultimately it is also doing the work, but it is inefficient.

Similarly, ask yourself whether long polling is effective or not. No: when we talk about scaling, long polling is very complex to scale. You can do it to some extent, but after that WebSockets win. Ask yourself: you are opening the connection again and again, and whenever the server receives something it notifies, the connection terminates and you open the connection again. So it's better to go for a keep-alive connection, that is WebSockets.

So hopefully you get the answer to why the client periodically sends the connection request to the server: for confirmation that the message was successfully received or not.

### When

When should we use which one? That totally depends upon the architecture of your application and the application's demand.

_(Table/image: examples of which one to use when)_

If you are new to this you might find it difficult to understand, but as you grow you will find it easy.

## Advantages and Disadvantages

There are some advantages and disadvantages of each connection polling type by which you can decide which one to use.

### Short Polling

- **Advantages:** Simple to implement, universally supported, good for infrequent updates.
- **Disadvantages:** Many unnecessary requests, higher bandwidth usage, increased server load, not suitable for real-time applications.

### Long Polling

- **Advantages:** Near real-time updates, fewer unnecessary requests, more efficient than short polling.
- **Disadvantages:** Open connection consumes server resources, more complex than short polling, less scalable than WebSockets for very high concurrency.

### WebSockets

- **Advantages:** True real-time connection, bidirectional (full-duplex), low latency, efficient use of bandwidth.
- **Disadvantages:** More complex implementation, stateful connection requires careful scaling, some network proxies/firewalls may require additional configuration.

### Server-Sent Events (SSE)

- **Advantages:** Simple server-to-client streaming, automatic reconnection support, uses standard HTTP.
- **Disadvantages:** One-way connection only, not ideal when the client also needs to send frequent updates.

### HTTP Streaming

- **Advantages:** Efficient for continuously generated data, works over HTTP, good for streaming content.
- **Disadvantages:** Primarily one-way, requires handling of streamed/chunked responses.

---

> **NOTE:** If you are not comfortable with the coding part you can surely skip this portion.

## Axios with Short/Long Polling

At last, one important thing: Axios connection with short/long polling. I will be using JavaScript in this code (you can use GPT models or any other AI to convert that code into your domain-specific language).

### What is Axios?

It is a popular JavaScript library used to make HTTP requests (such as GET, POST, PUT and DELETE) to communicate with a server or an API. It helps send data to the backend and receive data from the backend in a simpler and more convenient way than the built-in `fetch()` API.

### Short Polling with Axios

```javascript
// NOTE: Short Polling with Axios
// client code
setInterval(async () => {
  try {
    const response = axios.get("http://localhost:3000/api/v1/books");
    return response;
  } catch (err) {
    console.error(err);
    throw err;
  }
}, 5000); // every 5 seconds client sends the request to server
```

### Long Polling with Axios

```javascript
// NOTE: Long polling with Axios
// client code
const longPolling = async () => {
  try {
    const response = axios.get("http://localhost:3000/api/v1/books");
    console.log(response);
    longPolling();
  } catch (err) {
    console.error(`Error: ${err}`);
    setTimeout(longPolling, 5000); // retry after 5 seconds if error occurs
  }
};
```
