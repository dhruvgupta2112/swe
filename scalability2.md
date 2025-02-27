# Scalable Event-Driven Microservices Architecture with Message Queue (RabbitMQ)

## Scenario
A web service receives **image upload requests**. Instead of processing images synchronously (which is slow and blocking), it sends jobs to a **message queue (RabbitMQ)**. A separate **worker service** consumes these messages, processes the images, and stores them.


### 1. Producer (API Service) - Sends Jobs to Queue
This is the **fast, non-blocking** API that **receives an image upload request** and enqueues it for background processing.

```python
import pika
import json
from flask import Flask, request, jsonify

app = Flask(__name__)

# Connect to RabbitMQ
connection = pika.BlockingConnection(pika.ConnectionParameters(host='rabbitmq'))
channel = connection.channel()
channel.queue_declare(queue='image_tasks')

@app.route('/upload', methods=['POST'])
def upload_image():
    image_data = request.files['image'].read()
    
    # Simulate storing image and enqueue processing task
    message = json.dumps({"image": image_data.hex()})
    channel.basic_publish(exchange='', routing_key='image_tasks', body=message)
    
    return jsonify({"message": "Image upload successful. Processing in background."})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### 2. Consumer (Worker Service) - Processes Jobs Asynchronously
This is a **separate worker service** that **processes images asynchronously** without blocking the API.

```python
import pika
import json
import time

# Connect to RabbitMQ
connection = pika.BlockingConnection(pika.ConnectionParameters(host='rabbitmq'))
channel = connection.channel()
channel.queue_declare(queue='image_tasks')

def process_image(image_hex):
    """ Simulate image processing """
    time.sleep(3)  # Simulate long-running image processing
    print("Processed image successfully")

def callback(ch, method, properties, body):
    message = json.loads(body)
    process_image(message["image"])
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(queue='image_tasks', on_message_callback=callback)

print("Worker is waiting for tasks...")
channel.start_consuming()
```

### 3. Docker Compose (Scaling Microservices)
We use **Docker Compose** to run **multiple workers** for handling more load.

```yaml
version: '3'
services:
  api:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - rabbitmq
  worker1:
    build: .
    command: ["python", "worker.py"]
    depends_on:
      - rabbitmq
  worker2:
    build: .
    command: ["python", "worker.py"]
    depends_on:
      - rabbitmq
  rabbitmq:
    image: "rabbitmq:management"
    ports:
      - "5672:5672"
      - "15672:15672"  # RabbitMQ dashboard
```


## How This Implements Scalability
- **Decoupled Microservices** – The API service and worker service are independent. Scaling one does not affect the other.  
- **Asynchronous Processing** – The API is **fast** because it does not wait for image processing to complete.  
- **Auto-Scalability** – Multiple worker instances (`worker1`, `worker2`) process jobs in parallel. More workers can be added dynamically.  
- **Message Queue Reliability** – RabbitMQ **persists jobs**, so tasks won’t be lost if a worker crashes.  
