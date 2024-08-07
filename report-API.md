# Overview of an API Concept

## 1. What is an API?

- An API (Application Programming Interface) is a set of rules and protocols that allows different software applications to communicate with each other. APIs enable the integration of various systems, making it easier for developers to use specific functionalities of an application or service without knowing its internal workings.

## 2. Key Components of an API

General Diagram of How API Works:
```plaintext

+------------------+    +--------------------+    +------------------+
|                  |    |                    |    |                  |
|      Log in      |--->|     Front End      |--->|    Back End      |
|                  | ^  |                    |  ^ |                  |
+------------------+ |  +--------------------+  | +------------------+
                     |        passing data      |
                     UI  --------------------> API

```

- In most ot the cases, API have many pieces that we can connect to. You might get many part of one API.

For example:
```plaintext
                         +------------------+
                         |                  |
                         |      Part 2      |
                         |                  |
                         +------------------+
                                  |
+------------------+    +--------------------+    +------------------+
|     (Part 1)     |    |                    |    |                  |
|      Log in      |--->|     Back  End      |<---|    Feature 1     |
|                  |    |                    |    |                  |
+------------------+    +--------------------+    +------------------+

                  All these parts are called Endpoints.

```
1. **Endpoints**: Specific URLs where the API can be accessed. Each endpoint corresponds to a particular function or data resource.
2. **Requests**: The calls made by a client to the API, asking for data or services. Requests usually contain HTTP methods like GET, POST, PUT, DELETE.
3. **Responses**: The data or message returned by the API after processing a request. Responses often include a status code indicating the success or failure of the request.
4. **Authentication**: Mechanisms to ensure that only authorized clients can access the API.
5. **Rate Limiting**: Controls to limit the number of requests a client can make to the API in a given time period.

More Detailed Diagram of How API Works:
```plaintext

+---------+             +------------+             +------------+
|         |  HTTP       |            |  Process    |            |
|  Client <------------->     API    <------------->   Server   |
|         |  Request    |            |  Request    |            |
+---------+             +------------+             +------------+
                       /               \
                      /                 \
              +--------+             +--------+
              | Auth   |             | Rate   |
              | Layer  |             | Limit  |
              +--------+             +--------+

```
1. Client sends an HTTP request to the API.
2. API processes the request.
3. Authentication layer verifies client credentials.
4. Rate limiting checks the request quota.
5. Server processes the request and sends back a response.
   
```plaintext

+------------------+    +--------------------+    +------------------+
|                  |    |                    |    |                  |
|  Client Machine  |--->|    FastAPI App     |--->|   LangServe      |
|                  |    |                    |    |                  |
+------------------+    +--------------------+    +------------------+

```
Usually, API are come with special API keys and documentations for security reasons and also good for developers to manage their calls.
However, such API operations might sound like a lots of work.
But we could use tools and platforms such as zapier, FastAPI.

## 3. Report on FastAPI

### Introduction to FastAPI

FastAPI is a modern, fast (high-performance), web framework for building APIs with Python 3.7+ based on standard Python type hints. Its key features include:

- **Fast**: As the name suggests, FastAPI is designed to be one of the fastest Python frameworks available. This performance is achieved by using asynchronous programming, which allows handling multiple requests concurrently.
- **Easy to Use**: FastAPI is designed to be easy to use and learn. It significantly reduces the amount of boilerplate code needed to create APIs.
- **Automatic Documentation**: FastAPI automatically generates interactive API documentation using OpenAPI (formerly known as Swagger) and JSON Schema. This makes it easier for developers to understand and use the API endpoints.
- **Type Hints**: FastAPI uses Python type hints for parameter validation, serialization, and documentation. This ensures that data is handled correctly and reduces runtime errors.
- **Dependency Injection**: FastAPI supports dependency injection, allowing for more modular and testable code.

### Integration of FastAPI with LangServe

FastAPI’s integration into LangServe is seamless, leveraging its type hints and async capabilities to create robust API endpoints. FastAPI’s automatic generation of OpenAPI and JSON Schema documentation enhances the usability and maintainability of LangServe.

#### Step-by-Step Instructions

1. **Install FastAPI and Uvicorn**:
   To get started, you need to install FastAPI and an ASGI server, Uvicorn, which will serve your FastAPI application.

   ```bash
   pip install "fastapi[standard]"
   ```
2. **Set Up the FastAPI Application**:
   Create a new Python file, like main.py
   ```python
   from typing import Union
   from fastapi import FastAPI

   app = FastAPI()
    
   @app.get("/")
   def read_root():
     return {"Hello": "World"}
    
   @app.get("/items/{item_id}")
   def read_item(item_id: int, q: Union[str, None] = None):
     return {"item_id": item_id, "q": q}

   
   #If you are building with LangServe
   from fastapi import FastAPI
   from langserve import ModelManager
   
   # Initialize the FastAPI app
   app = FastAPI()
    
   # Create an instance of the ModelManager and load your model
   model_manager = ModelManager(model_path="path/to/model")
    
   # Define a POST endpoint for predictions
   @app.post("/predict/")
   async def predict(text: str):
   # Use the model manager to predict based on the input text
   return model_manager.predict(text)
   ```
   
4. **Run the Server**:
   ```bash
   fastapi dev main.py
   ```
   Please go to https://fastapi.tiangolo.com/#installation to see with more detail.
