# 3. Report on FastAPI

## Introduction to FastAPI

FastAPI is a contemporary web framework for building APIs with Python 3.7 and later, known for its high performance and ease of use. It utilizes modern Python features like asynchronous programming and type hints, making it a robust choice for developing APIs. This framework is designed for high speed and rapid development, with added emphasis on quick and easy code management.

### Key Features of FastAPI

#### Fast
Leveraging Starlette for the web parts and Pydantic for the data handling, FastAPI offers performance that rivals NodeJS and Go thanks to its asynchronous request handling. This makes it an excellent choice for applications that require high throughput and low latency, such as real-time data processing applications.

#### Easy to Use
FastAPI simplifies the development process by reducing boilerplate code and providing a clean yet powerful API design structure. The framework's design promotes maintainable and modular code, essential for long-term projects and large development teams. FastAPI's detailed error handling further aids in quick debugging and development.

#### Automatic Documentation
FastAPI's support for OpenAPI and JSON Schema automatically generates detailed documentation for the API. This documentation is interactive, allowing developers to test endpoints directly from their browsers using platforms like Swagger UI and ReDoc, thus simplifying the initial stages of front-end integration.

#### Type Hints
The framework's reliance on Python type hints boosts development speed by catching errors at development time rather than during production. This strong, static typing helps prevent common type errors and improves IDE support with more robust autocompletion, real-time type checks, and refactoring tools.

#### Dependency Injection
Dependency injection in FastAPI is implemented to provide a way to supply external or shared components to a component. This feature supports various configurations and cross-cutting concerns like data access configurations, user authentication, and background tasks management, making the system easier to manage and scale.

#### Security and Authentication
FastAPI comes with built-in security and authentication features, supporting standards like OAuth2 with password hashing, JWT for secure tokens, and integration of more complex security policies, ensuring robust security mechanisms are easy to implement and manage.

#### Extensibility
The modular nature of FastAPI makes it highly extensible; developers can plug in any ORM, request parsing libraries, or other tools as required. The flexibility here is key to building systems that need to evolve over time with minimal hassle.

### Core Components of FastAPI

#### APIRouter
`APIRouter` allows developers to organize and manage different parts of the API in a modular and scalable way. This feature helps in separating the routing logic into different files, making the codebase cleaner and easier to maintain. Example usage:

```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/items/{item_id}", tags=["items"])
async def read_item(item_id: str):
    return {"name": "Foo", "item_id": item_id}

```

#### Pydantic Models
FastAPI uses Pydantic for data validation and settings management through Python type annotations. Pydantic models enforce type hints at runtime and provide user-friendly errors when data is invalid. Example definition:
```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

```
Pydantic's integration allows for data validation through Python type annotations. This ensures that the data received and processed within the API adheres strictly to the defined schemas, providing automatic error handling and standardization of data formats.

#### Dependency Injection
FastAPI's dependency injection system allows for reusable components and utilities to be easily integrated into your application's API endpoints. It can manage sophisticated user authentication and authorization schemes, database connections, and much more. Example of a simple dependency:
```python
from fastapi import Depends, FastAPI

def get_query_token(token: str = Depends()):
    return token

@app.get("/items/")
async def read_items(token: str = Depends(get_query_token)):
    return {"token": token}

```
How to use dependency injection to manage database sessions throughout the lifecycle of a web request, ensuring efficient and safe handling of database connections.
```python
from fastapi import Depends, FastAPI
from sqlalchemy.orm import Session
from .database import SessionLocal

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/items/")
async def read_items(db: Session = Depends(get_db)):
    items = db.query(models.Item).all()
    return items

```
## Building an API with FastAPI
Setting Up a New FastAPI Project
Start by setting up your project environment and installing necessary packages:
```bash
pip install fastapi uvicorn
```
Create a new file, for example, main.py, and set up the first API endpoints:
```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}

```
#### Running the FastAPI Application
To run your FastAPI application, use Uvicorn as the ASGI server. Execute the following command in your terminal:
```bash
uvicorn main:app --reload  # The `--reload` flag enables auto-reloading for development.

```
Visit http://127.0.0.1:8000 in your browser to see your API in action.

### Integrating with Databases and External Services
FastAPI can be easily integrated with databases and external services. Here is a basic example of integrating with a SQL database using SQLAlchemy:
```python
from fastapi import FastAPI, Depends
from sqlalchemy.orm import Session
from . import crud, models, schemas
from .database import SessionLocal, engine

models.Base.metadata.create_all(bind=engine)

app = FastAPI()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.post("/users/", response_model=schemas.User)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    return crud.create_user(db=db, user=user)

```

## Advanced Features of FastAPI

### Background Tasks
FastAPI allows you to define background tasks that are executed after a request has been returned. This is useful for operations like sending emails, processing data asynchronously, or making calls to external APIs without delaying the response to the client.

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()

def write_log(message: str):
    with open("log.txt", "a") as file:
        file.write(message)

@app.post("/send-notification/")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_log, f"notification sent to {email}")
    return {"message": "Notification sent in the background"}

```
Background tasks in FastAPI allow operations like email notifications, external API calls, or data processing to run asynchronously without blocking server responses. This is crucial for maintaining the responsiveness of the application.

### Middleware
Middleware in FastAPI can be used to run some code before each request is processed and after each response is sent. This allows for tasks such as request logging, CORS handling, or security checks.
```python
from fastapi import FastAPI
from starlette.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

```
Middleware can be used for a variety of cross-cutting concerns, such as session management, rate limiting, or server-side analytics.
```python
from fastapi import FastAPI
from starlette.middleware.base import BaseHTTPMiddleware

class CustomMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)
        response.headers['X-Custom-Header'] = 'Value'
        return response

app = FastAPI()
app.add_middleware(CustomMiddleware)
```

### WebSockets
FastAPI supports WebSockets out of the box, enabling real-time two-way communication between the client and server. This is ideal for features like chat applications or real-time updates.
```python
from fastapi import FastAPI, WebSocket

app = FastAPI()

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")

```

## Security Features
Authentication
FastAPI provides several tools to handle authentication securely and easily. Here is how you can implement OAuth2 with Password (and hashing), using JWT tokens for secure transmission of information between parties.
```python
from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext

```
FastAPI's security utilities simplify the implementation of various authentication protocols, which can be customized extensively to fit the needs of any application.
```python

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)

def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)

def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user or not verify_password(password, user.hashed_password):
        return False
    return user

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(fake_fake_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}

```

### Future Directions
#### Upcoming Features
GraphQL Support: FastAPI is planning to integrate more seamless support for GraphQL, broadening its applicability for more complex APIs.
Enhanced Dependency Injection: Further improvements are expected in the dependency injection mechanism, making it even more flexible and easier to use in large-scale applications.
#### Community Contributions
FastAPI has a growing community that contributes a wide range of third-party extensions, plugins, and integrations, continually enriching the ecosystem. Future updates and community contributions can be tracked through FastAPI's GitHub repository and community forums.
