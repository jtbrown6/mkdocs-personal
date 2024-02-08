# Welcome

For full documentation visit [mkdocs.org](https://www.mkdocs.org).

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

### Add Line Numbers
```py linenums="1"
from fastapi import FastAPI, HTTPException
import requests
import uvicorn

app = FastAPI()

# Define the endpoint for the chat completion API
API_ENDPOINT = "http://localhost:11434/api/chat"

# Define your model name here
MODEL_NAME = "mixtral"

@app.post("/chat/")
async def chat(message: str):
```

### Highlight Code Lines
```py hl_lines="2 3"
    if response.status_code != 200:
        # Handle any errors from that ollama endpoint on my server
        raise HTTPException(status_code=response.status_code, detail="Error from the chat API")
```

## Icons and Emojis

:smile:

:fontawesome-regular-face-laugh-wink: