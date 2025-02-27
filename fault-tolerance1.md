# Fault-Tolerant and Self-Recovering System Using Circuit Breaker and Retry Mechanisms

Ensuring fault tolerance and graceful recovery requires implementing techniques like:

1. **Circuit Breaker Pattern** – Prevents system overload when a service is failing.
2. **Retry Mechanisms** – Automatically retries failed requests.
3. **Graceful Degradation** – Uses fallback responses if a service is unavailable.
4. **Auto-Healing & Failover** – Detects failures and automatically restarts services.

Lets see a simple example.

## Scenario
We have a **microservices-based architecture** where:
- A **User Service** calls an **Order Service**.
- If the **Order Service** fails, the system **does not overload it**.
- The system **retries failed requests** using an **exponential backoff**.
- If the **Order Service remains down**, a **fallback response** is provided.

Next, I show how we can implement it.

## 1. Implementing the Circuit Breaker Pattern
We'll use **FastAPI (Python)** with **Tenacity (retry library)** to prevent system overload.

### User Service (Client) - Calls Order Service
This service will **retry failed requests** and **trigger a circuit breaker** if the Order Service is down.

```python
import requests
import time
from fastapi import FastAPI
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type

app = FastAPI()

ORDER_SERVICE_URL = "http://order-service:8001/orders"

class OrderServiceUnavailable(Exception):
    """ Custom exception for Order Service failures """

@retry(
    stop=stop_after_attempt(3),  # Stop after 3 attempts
    wait=wait_exponential(multiplier=1, min=2, max=10),  # Exponential backoff
    retry=retry_if_exception_type(OrderServiceUnavailable)
)
def fetch_orders():
    try:
        response = requests.get(ORDER_SERVICE_URL, timeout=5)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException:
        raise OrderServiceUnavailable()

@app.get("/user/orders")
def get_user_orders():
    try:
        return fetch_orders()
    except OrderServiceUnavailable:
        return {"message": "Order service is down, please try again later."}

```

### 2. Order Service (Backend) - Simulating a Failing Service
This is a **mock service** that randomly fails to demonstrate failure recovery.

```python
import random
import time
from fastapi import FastAPI

app = FastAPI()

@app.get("/orders")
def get_orders():
    """Simulates an order service that fails randomly"""
    if random.choice([True, False]):  # 50% chance of failure
        time.sleep(6)  # Simulate a slow response
        return {"error": "Order Service is overloaded!"}, 503
    return {"orders": ["Order 1", "Order 2"]}
```

### 3. Using Kubernetes for Auto-Healing & Failover
Kubernetes can **automatically restart failing services** using **Liveness and Readiness Probes**.

**Deployment with Auto-Healing in Kubernetes**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3  # Ensures high availability
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: my-order-service
        ports:
        - containerPort: 8001
        livenessProbe:
          httpGet:
            path: /health
            port: 8001
          initialDelaySeconds: 3
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: 8001
          initialDelaySeconds: 3
          periodSeconds: 5
```

This ensures that:
- **Liveness Probe**: Restarts the service if it becomes unresponsive.
- **Readiness Probe**: Ensures traffic is routed only to healthy instances.


## How This Ensures Fault Tolerance
- **Automatic Retries with Backoff** – Prevents overwhelming the failing service.  
- **Circuit Breaker** – Stops retrying after multiple failures and provides a fallback.  
- **Load Balancing & Auto-Healing** – Ensures traffic is redirected to healthy instances.  
- **Graceful Degradation** – Users get a response even when a service is down.  


# Fault-Tolerant System Without Kubernetes
If you don't want to use Kubernetes, we can still build a **fault-tolerant, self-recovering system** using:

1. **Load Balancing (NGINX)** – Distributes traffic across multiple instances.
2. **Service Auto-Restart (Systemd / Docker Health Check)** – Automatically restarts failed services.
3. **Circuit Breaker & Retry Mechanism** – Prevents overloading a failing service.
4. **Graceful Fallback Responses** – Ensures continued functionality during failures.
5. **Database Replication & Auto-Failover** – Ensures continuous database availability.

## Scenario
We have a **User Service** that calls an **Order Service**. If **Order Service** fails:
- The **User Service retries with exponential backoff**.
- A **Circuit Breaker prevents overload**.
- A **Failover Mechanism automatically restarts the Order Service**.
- A **Load Balancer (NGINX) routes requests to healthy instances**.

Following are the main steps to implement it:

## 1. Circuit Breaker & Retry Logic in User Service
This is the **User Service** that calls the **Order Service** with **fault tolerance mechanisms**.

```python
import requests
from flask import Flask, jsonify
from tenacity import retry, stop_after_attempt, wait_exponential, retry_if_exception_type

app = Flask(__name__)

ORDER_SERVICE_URL = "http://order-service:8001/orders"

class OrderServiceUnavailable(Exception):
    """ Custom exception for Order Service failures """

@retry(
    stop=stop_after_attempt(3),  # Retry up to 3 times
    wait=wait_exponential(multiplier=1, min=2, max=10),  # Exponential backoff
    retry=retry_if_exception_type(OrderServiceUnavailable)
)
def fetch_orders():
    try:
        response = requests.get(ORDER_SERVICE_URL, timeout=5)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.RequestException:
        raise OrderServiceUnavailable()

@app.route('/user/orders')
def get_user_orders():
    try:
        return fetch_orders()
    except OrderServiceUnavailable:
        return {"message": "Order service is temporarily unavailable. Please try again later."}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### What This Does
- **Retries failed requests up to 3 times**  
- **Uses exponential backoff (2s, 4s, 8s) to avoid overloading**  
- **Provides a graceful fallback response if retries fail**  

## 2. Order Service (Backend) with Automatic Restart
This **Order Service** randomly fails and **recovers automatically** using **Systemd or Docker health checks**.

```python
import random
import time
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/orders")
def get_orders():
    if random.choice([True, False]):  # 50% chance of failure
        time.sleep(6)  # Simulate a slow response
        return jsonify({"error": "Order Service is overloaded!"}), 503
    return jsonify({"orders": ["Order 1", "Order 2"]})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port