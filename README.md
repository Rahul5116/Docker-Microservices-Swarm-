# ⚓ Microservices Architecture with Docker Swarm

This project demonstrates how to deploy a simple **microservices architecture** using **Docker Swarm**, featuring:

- 🧩 A **Backend Service** (Flask)
- 🌐 An **API Gateway** (Flask + Requests)

---

## 📁 Project Structure

```
microservices-docker-swarm/
│── backend-service/
│   ├── backend.py
│   ├── Dockerfile
│
│── api-gateway/
│   ├── api_gateway.py
│   ├── Dockerfile
│
│── docker-compose.yml
│── README.md
```

---

## 🚀 Overview

- The **API Gateway** receives the client request and forwards it to the **Backend Service**
- Backend Service responds with `"Rahul"`, and the API Gateway prepends `"API Gateway: "` to the response
- Built using **Flask** for both services and containerized using **Docker**

---

## ⚙️ Prerequisites

- [Docker](https://www.docker.com/get-started) installed
- Docker Desktop **running**
- [Visual Studio Code](https://code.visualstudio.com/)
- Docker Swarm initialized:

```bash
docker swarm init
```

---

## 🔧 Step-by-Step Setup (VS Code)

### Step 1: Clone or Create the Project

```bash
git clone https://github.com/your-username/microservices-docker-swarm.git
cd microservices-docker-swarm
```

Or manually create the structure using:

```powershell
mkdir microservices-docker-swarm
cd microservices-docker-swarm

mkdir backend-service, api-gateway

New-Item backend-service\backend.py -ItemType File
New-Item backend-service\Dockerfile -ItemType File

New-Item api-gateway\api_gateway.py -ItemType File
New-Item api-gateway\Dockerfile -ItemType File

New-Item docker-compose.yml -ItemType File
New-Item README.md -ItemType File
```

---

### Step 2: Write the Services

#### ✅ backend-service/backend.py

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return "Rahul"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)
```

#### ✅ backend-service/Dockerfile

```Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY backend.py /app
RUN pip install flask
CMD ["python", "backend.py"]
```

#### ✅ api-gateway/api_gateway.py

```python
from flask import Flask
import requests

app = Flask(__name__)

@app.route('/')
def hello():
    backend_response = requests.get('http://backend-service:5000')
    return f"API Gateway: {backend_response.text}"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080)
```

#### ✅ api-gateway/Dockerfile

```Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY api_gateway.py /app
RUN pip install flask requests
CMD ["python", "api_gateway.py"]
```

---

### Step 3: Define Services in `docker-compose.yml`

```yaml
version: '3.7'

services:
  backend-service:
    image: backend-service
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure
    networks:
      - app-network
    ports:
      - "5000:5000"

  api-gateway:
    image: api-gateway
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure
    networks:
      - app-network
    ports:
      - "8080:8080"
    depends_on:
      - backend-service

networks:
  app-network:
    driver: overlay
```

---

### Step 4: Build the Docker Images

```bash
docker build -t backend-service ./backend-service
docker build -t api-gateway ./api-gateway
```

---

### Step 5: Deploy to Docker Swarm

```bash
docker stack deploy -c docker-compose.yml my_microservices
```

---

### Step 6: Access the Microservices

Visit: [http://localhost:8080](http://localhost:8080)  
Expected output:

```
API Gateway: Rahul
```

---

### Step 7: Scale the Backend Service

```bash
docker service scale my_microservices_backend-service=5
```

---

## 🧪 Verify & Manage

- List running services:

```bash
docker stack services my_microservices
```

- View running containers:

```bash
docker ps
```

---

## 🔄 Updating Services

1. Modify code in `backend.py`
2. Rebuild the image:

```bash
docker build -t backend-service ./backend-service
```

3. Update the running service:

```bash
docker service update --image backend-service:latest my_microservices_backend-service
```

---

## 🧹 Remove Stack and Leave Swarm

```bash
docker stack rm my_microservices
docker swarm leave --force
```

---

## 🛠️ Common Errors & Fixes

| Error | Cause | Solution |
|------|-------|----------|
| `Error response from daemon: Swarm not initialized` | You didn't run `docker swarm init` | Run `docker swarm init` first |
| `Cannot connect to the Docker daemon` | Docker Desktop is not running | Start Docker Desktop |
| `Connection refused at localhost:8080` | Stack not deployed or port not exposed | Verify with `docker stack services my_microservices` |
| `Name resolution error for backend-service` | Network issue in Swarm | Ensure both services are on the same `overlay` network |

---

## 👨‍💻 Author

**Rahul**  
Microservices using Docker Swarm with Flask  
Feel free to fork, customize, or contribute to this project!

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).
