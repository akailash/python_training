**Basic FastAPI Tutorial**

Welcome to the Basic FastAPI Tutorial! In this tutorial, you will learn how to build a simple API using FastAPI, a modern web framework for building APIs with Python. FastAPI is known for its fast performance, automatic validation, and interactive documentation. Let's dive in!

**1. Installation:**

First, you need to install FastAPI and Uvicorn, the ASGI server. You can install them using pip:

```bash
pip install fastapi uvicorn
```

**2. Hello World with FastAPI:**

Create a file named `main.py` and add the following code:

```python
from fastapi import FastAPI

# Create an instance of FastAPI
app = FastAPI()

# Define a route and its handler
@app.get("/")
def read_root():
    return {"message": "Hello, World!"}
```

**3. Running the FastAPI Application:**

You can start the FastAPI application using the Uvicorn server. Run the following command in your terminal:

```bash
uvicorn main:app --reload
```

This command starts the server and reloads it automatically whenever you make changes to your code.

**4. Testing the API:**

Open your web browser or any API testing tool and navigate to `http://127.0.0.1:8000`. You should see the message "Hello, World!".

**5. Path Parameters:**

You can define dynamic routes with path parameters using FastAPI. Modify `main.py` as follows:

```python
@app.get("/items/{item_id}")
def read_item(item_id: int):
    return {"item_id": item_id}
```

Now you can access paths like `/items/1`, `/items/2`, etc., and you'll receive the respective `item_id` in the response.

**6. Request Body and JSON Response:**

You can also handle request bodies and return JSON responses. Update `main.py` as follows:

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

@app.post("/items/")
def create_item(item: Item):
    return item
```

With this change, you can send POST requests to `/items/` with JSON data containing `name`, `description`, `price`, and optionally `tax`, and FastAPI will automatically validate the request body and return it in the response.

**7. Interactive Documentation:**

FastAPI generates interactive API documentation automatically using Swagger UI. You can access it by navigating to `http://127.0.0.1:8000/docs` in your web browser. You can explore the available endpoints, send requests, and view responses directly from the documentation.

Congratulations! You've completed the Basic FastAPI Tutorial. You've learned how to create a simple API, handle path parameters, request bodies, and generate interactive documentation using FastAPI. Experiment with different features and build more complex APIs to further explore FastAPI's capabilities. If you have any questions or need further clarification, feel free to ask. Happy coding!