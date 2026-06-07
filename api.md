Here is a clean teaching flow for your FastAPI class.

# Python FastAPI Teaching Content

## 1. What are APIs?

API means **Application Programming Interface**.

Simple explanation:

> An API allows two software systems to talk to each other.

Example:

A frontend website wants to create a Kubernetes namespace.

The frontend should not directly run `kubectl`.

Instead:

```text
Frontend  --->  FastAPI Backend  --->  Kubernetes Cluster
```

The frontend sends a request to the API, and the API performs the action.

---

## 2. How we worked before APIs existed

Before APIs became common, systems were often connected using:

```text
Manual work
Direct database access
File sharing
Shell scripts
Human-to-human communication
```

Example:

Earlier process:

```text
Student asks admin for namespace
Admin logs into server
Admin runs kubectl create namespace student1
Admin confirms manually
```

With API:

```text
Student enters namespace name in UI
FastAPI receives POST request
FastAPI creates namespace automatically
```

---

## 3. Benefits of API

APIs provide:

```text
Automation
Security
Standard communication
Frontend-backend separation
Integration with other tools
Faster operations
Less manual work
Reusable backend logic
```

---

## 4. HTTP Request Types

Two common types for beginners:

### GET

Used to retrieve data.

Example:

```text
GET /hello
```

### POST

Used to send data to the server.

Example:

```text
POST /create-namespace
```

---

# Install Requirements

```bash
pip install fastapi uvicorn kubernetes
```

Run FastAPI:

```bash
uvicorn main:app --reload
```

---

# 1. Simple API

Create `main.py`:

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
def home():
    return {
        "message": "Welcome to FastAPI"
    }


@app.get("/hello")
def hello():
    return {
        "message": "Hello from my first API"
    }
```

Test in browser:

```text
http://127.0.0.1:8000/
http://127.0.0.1:8000/hello
```

Swagger UI:

```text
http://127.0.0.1:8000/docs
```

---

# 2. API Which Takes a POST Request

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Student(BaseModel):
    name: str
    course: str


@app.post("/student")
def create_student(student: Student):
    return {
        "message": "Student data received",
        "student_name": student.name,
        "course": student.course
    }
```

Example POST body:

```json
{
  "name": "Aryan",
  "course": "DevOps"
}
```

---

# 3. API Which Creates Kubernetes Namespace Using POST

Create `main.py`:

```python
from fastapi import FastAPI
from pydantic import BaseModel
from kubernetes import client, config

app = FastAPI()


class NamespaceRequest(BaseModel):
    namespace: str


@app.post("/create-namespace")
def create_namespace(data: NamespaceRequest):
    try:
        config.load_kube_config()

        v1 = client.CoreV1Api()

        namespace_body = client.V1Namespace(
            metadata=client.V1ObjectMeta(
                name=data.namespace
            )
        )

        v1.create_namespace(body=namespace_body)

        return {
            "status": "success",
            "message": f"Namespace {data.namespace} created successfully"
        }

    except Exception as e:
        return {
            "status": "failed",
            "message": str(e)
        }
```

Test POST body:

```json
{
  "namespace": "student1"
}
```

Verify:

```bash
kubectl get ns
```

---

# 4. Simple Frontend for Namespace API

Create `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Kubernetes Namespace Creator</title>
</head>
<body>

    <h2>Create Kubernetes Namespace</h2>

    <input type="text" id="namespace" placeholder="Enter namespace name">

    <button onclick="createNamespace()">Create Namespace</button>

    <p id="result"></p>

    <script>
        async function createNamespace() {
            const namespace = document.getElementById("namespace").value;

            const response = await fetch("http://127.0.0.1:8000/create-namespace", {
                method: "POST",
                headers: {
                    "Content-Type": "application/json"
                },
                body: JSON.stringify({
                    namespace: namespace
                })
            });

            const data = await response.json();

            document.getElementById("result").innerText = data.message;
        }
    </script>

</body>
</html>
```

---

# Important CORS Fix

Because HTML frontend and FastAPI backend are different origins, add CORS in `main.py`:

```python
from fastapi import FastAPI
from pydantic import BaseModel
from kubernetes import client, config
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


class NamespaceRequest(BaseModel):
    namespace: str


@app.post("/create-namespace")
def create_namespace(data: NamespaceRequest):
    try:
        config.load_kube_config()

        v1 = client.CoreV1Api()

        namespace_body = client.V1Namespace(
            metadata=client.V1ObjectMeta(
                name=data.namespace
            )
        )

        v1.create_namespace(body=namespace_body)

        return {
            "status": "success",
            "message": f"Namespace {data.namespace} created successfully"
        }

    except Exception as e:
        return {
            "status": "failed",
            "message": str(e)
        }
```

---

# Final Classroom Flow

Teach in this order:

```text
1. What is API?
2. Why APIs are needed
3. GET request
4. POST request
5. FastAPI basics
6. Simple API
7. POST API
8. Kubernetes namespace API
9. Frontend calling backend API
10. Real DevOps use case discussion
```
