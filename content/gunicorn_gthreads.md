+++
title = "TIL: Gunicorn's gthread Worker Timeout Doesn't Work as Expected"
date = 2024-11-20

[taxonomies]
tags = [
    "python",
    "TIL",
]
+++

When using Gunicorn to run our Django application in production, I had a simple assumption: setting a timeout value (or using the default 30-second timeout) would make sure that any request taking longer than that would be stopped. This is important for keeping the system stable and preventing resource problems.

# The Unexpected Discovery

While looking into issues in our system, I created a simple Django server similar to our application to test how it works.

What I found was surprising: when using Gunicorn with the `gthread` worker class and testing a delayed response, the worker didn't stop after the expected 30-second timeout. Instead, it kept waiting for the response.

---

*Note: This behavior may change in future versions of Gunicorn. If you're reading this in the future, check the latest documentation and test the behavior in your system.*

# Digging Deeper

After some research, I found [an existing issue](https://github.com/benoitc/gunicorn/issues/2695) describing similar behavior. The key point: Gunicorn's timeout setting doesn't work as described in the docs when using the `gthread` worker class.

# Reproducing the Issue

To show this behavior, I created a simple test:

### Dockerfile
```dockerfile
FROM python:3.13-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8080
CMD ["gunicorn", "--bind", "0.0.0.0:8080", "project.wsgi:application", "-k", "gthread", "-t", "30"]
```

### A Simple View That Sleeps
```python
def long_sleep(request) -> HttpResponseBase:
    logger.info(f"Going to sleep!")
    time.sleep(30)
    logger.info(f"30 seconds passed! Am I still alive?")
    time.sleep(30)
    logger.info(f"Isn't my timeout suposed to kill me? Weird...")
    return JsonResponse({"message": "ok"})
```

### Server Logs
```
server-1  | [2025-03-06 14:26:06 +0000] [1] [INFO] Starting gunicorn 23.0.0
server-1  | [2025-03-06 14:26:06 +0000] [1] [INFO] Listening at: http://0.0.0.0:8080 (1)
server-1  | [2025-03-06 14:26:06 +0000] [1] [INFO] Using worker: gthread
server-1  | [2025-03-06 14:26:06 +0000] [7] [INFO] Booting worker with pid: 7
server-1  | INFO 2025-03-06 14:26:10,175: Going to sleep!
server-1  | INFO 2025-03-06 14:26:40,176: 30 seconds passed! Am I still alive?
server-1  | INFO 2025-03-06 14:27:10,176: Isn't my timeout suposed to kill me? Weird...
```

The worker simply kept running without any timeout, which goes against what the [Gunicorn's documentation](https://docs.gunicorn.org/en/stable/settings.html#timeout) says:

> Workers silent for more than this many seconds are killed and restarted.
>
> Default: 30

# Conclusion
If you're using Gunicorn with the `gthread` worker in production, I strongly suggest testing your timeout behavior to make sure it works as you expect.

# References

[Gunicorn's documentation](https://docs.gunicorn.org/en/stable/settings.html)