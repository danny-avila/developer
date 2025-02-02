Develop a RESTful FastAPI in Python to interface with various AI utility libraries.

**FastAPI Application**
Create a new file named main.py and initialize a FastAPI application. Structure your application for modularity, maintainability, and scalability.

**Environment Setup**
Include FastAPI, Uvicorn (for serving your application), and all necessary AI libraries or dependencies in a requirements.txt file. Note: all FastAPI dependencies can be installed with `pip install "fastapi[all]"`. Note: environment variables won't be necessary for interacting with external APIs, as the necessary credentials will be supplied by the request payload.

**Data Handling**
Implement handling of expected payloads. This API will serve as a service, so no database operations are necessary. You will receive requests as follows: 
```bash
curl -X 'POST' \
'http://localhost:8080/ask' \
-H 'accept: application/json' \
-H 'Content-Type: application/json' \
-d '{
"service": "q&a",
"input": "How many people live in canada as of 2023?",
"envs": {
    "OPENAI_API_KEY": "'"${OPENAI_API_KEY}"'",
}
}'
```

**Organization**
To distribute routes, create a `routes` directory and separate route files for each feature. Define routes in each file using FastAPI's `Router`. In your main application, import and include these routers. Your AI service files should also be organized in a modular manner.

**API Endpoints**
Define endpoints corresponding to functions provided by the AI utility libraries. Use appropriate HTTP methods for each endpoint. For instance, an endpoint like POST /sentiment_analysis could accept text data and return the sentiment score.

**Endpoint Implementation**
Implement the functionality for each endpoint using the corresponding AI library. Handle errors, validate and sanitize user input.

**Response Formatting**
Structure your response data consistently. Typically, return JSON data from an API, with a status code, a message, and the data itself. The expected data format is:
```json
{
"result": "Arrr, there be 38,645,670 people livin' in Canada as of 2023!",
"error": "",
"stdout": "Answer the following questions as best you can, but speaking as a pirate might speak...Final Answer: Arrr, there be 38,645,670 people livin' in Canada as of 2023!"
}
```

**Testing**
Write tests for your endpoints to ensure they work as expected. Consider edge cases and invalid inputs.

**Error Handling**
An AI service should return an HttpException at the function level; in other words, an error message from an external AI API should not result in a 200 status code through this API.

**Authentication**
Your API should have simple middleware for authentication, solely verifying one master api key. No user handling or other authentication is required. Keep this part of your code as simple and straightforward as possible.

**Dockerization**
Write a Dockerfile with all the setup steps and a corresponding docker-compose.yaml file for convenience.

**Documentation**
Document your API thoroughly. For each endpoint, explain the URL, HTTP method, expected input, possible error messages, and example output.

Consistency and clarity are key to a good API. Users should be able to predict endpoint URLs and understand responses based on their knowledge of the rest of the API.
