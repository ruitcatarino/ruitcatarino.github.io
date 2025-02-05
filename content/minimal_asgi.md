+++
title = "Building a Minimal ASGI Web Framework in Python"
date = 2025-01-05

[taxonomies]
tags = [
    "python"
]
+++

ASGI (Asynchronous Server Gateway Interface) is the modern standard for asynchronous Python web applications, extending WSGI to support WebSockets, long-lived connections, and background tasks.

This post walks through building a minimal ASGI web framework from scratch for educational purposes, designed to help you understand how web frameworks work under the hood.

# Understanding ASGI
ASGI applications are Python callables that accept three arguments:
```python
async def app(scope, receive, send):
    ...
```
- **scope**: Connection information (type, path, headers).
- **receive**: Async function to receive events (for WebSockets or HTTP requests).
- **send**: Async function to send responses.

# Creating the Core Framework
A minimal ASGI framework needs:
- **Routing** to match incoming requests to handlers.
- **Request & Response abstraction** for easier handling.

## Defining Request and Response Classes
```python
from urllib.parse import parse_qs

class Request:
    def __init__(self, scope, receive):
        self.scope = scope
        self.receive = receive
        self.method = scope["method"]
        self.path = scope["path"]
        self.query_params = parse_qs(scope.get("query_string", b"").decode())

class Response:
    def __init__(self, body, status=200, headers=None):
        self.body = body.encode("utf-8")
        self.status = status
        self.headers = headers or [("content-type", "text/plain")]

    async def send(self, send):
        await send({
            "type": "http.response.start",
            "status": self.status,
            "headers": [(k.encode(), v.encode()) for k, v in self.headers],
        })
        await send({
            "type": "http.response.body",
            "body": self.body,
        })
```
## Adding Routing
```python
class App:
    def __init__(self):
        self.routes = {}

    def route(self, path):
        def wrapper(handler):
            self.routes[path] = handler
            return handler
        return wrapper

    async def __call__(self, scope, receive, send):
        if scope["type"] != "http":
            return

        request = Request(scope, receive)
        handler = self.routes.get(request.path)
        
        if handler:
            response = await handler(request)
        else:
            response = Response("Not Found", status=404)

        await response.send(send)
```
# Running the Application

## Define a Simple ASGI App Using the Framework
Once the framework is set up, create an instance of `App` and define some routes:
```python
app = App()

@app.route("/")
async def homepage(request):
    return Response("Hello, ASGI!")

@app.route("/echo")
async def echo(request):
    message = request.query_params.get("message", [""])[0]
    return Response(f"You said: {message}")
```

## Running the Server Locally
To start the ASGI application locally, use **Uvicorn**, a high-performance ASGI server:

First, install Uvicorn if you haven't:
```bash
pip install uvicorn
```

Then, run the server:
```bash
uvicorn myapp:app --host 127.0.0.1 --port 8000
```

You can test it in a browser or with `curl`:
```bash
curl http://127.0.0.1:8000/
curl http://127.0.0.1:8000/echo?message=Hello
```

## Deploying to Production
For production, use **Uvicorn with multiple workers** behind a process manager like `systemd` or a reverse proxy like `nginx`:
```bash
uvicorn myapp:app --host 0.0.0.0 --port 8000 --workers 4
```

Alternatively, use **Daphne**:
```bash
pip install daphne
daphne -b 0.0.0.0 -p 8000 myapp:app
```
### Note on Kubernetes, Docker Swarm, and Load Balancers
If deploying in **Kubernetes**, **Docker Swarm**, or behind a **load balancer**, the general best practice is to run **only one worker per container or pod**. The load balancer distributes traffic across multiple instances, making additional workers inside a single container unnecessary and potentially counterproductive.

# Handling JSON Requests and Responses
Extend `Response` to handle JSON:
```python
import json

class JSONResponse(Response):
    def __init__(self, data, status=200, headers=None):
        body = json.dumps(data)
        headers = headers or [("content-type", "application/json")]
        super().__init__(body, status, headers)
```

Example usage:
```python
@app.route("/json")
async def json_route(request):
    return JSONResponse({"message": "Hello, JSON!"})
```
# Conclusion
This minimal ASGI framework demonstrates:
- Request and response abstraction.
- Simple routing.
- JSON response handling.

However, this framework is not meant for production use. It's a basic implementation designed to help you understand how web frameworks work. For a production-ready alternative, consider [**Starlette**](https://github.com/encode/starlette) or [**FastAPI**](https://github.com/fastapi/fastapi), which offer more robust features, security, and scalability.

Keep calm and `import this`.

# References

[ASGI](https://asgi.readthedocs.io/en/latest/)

[Uvicorn](https://www.uvicorn.org/)