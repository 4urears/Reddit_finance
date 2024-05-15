# Reddit Scraper and Query Engine

This repository contains a Jupyter Notebook designed to scrape Reddit posts from the `stocks` subreddit and analyze them using LangChain and Llama-Index. The application is deployed as a FastAPI service.

## Prerequisites

Make sure you have the following dependencies installed:

- Python 3.7+
- Jupyter Notebook
- Google Colab (if running in the cloud)

## Installation

The following libraries need to be installed. Run these commands in your Jupyter Notebook:

```python
!pip install praw
!pip install llama-index llama-hub
!pip install llama-index-embeddings-huggingface
!pip install llama-index-embeddings-instructor
!pip install accelerate
!pip install git+https://github.com/huggingface/accelerate
!pip install fastapi nest-asyncio pyngrok uvicorn
Configuration
Environment Variables
Ensure the following environment variables are set:

LANGCHAIN_TRACING_V2: Set to "true"
LANGCHAIN_PROJECT: Set to "Reddit_finance"
LANGCHAIN_ENDPOINT: Set to "https://api.smith.langchain.com"
LANGCHAIN_API_KEY: Your Langsmith API key
fast: Your ngrok authentication token
Reddit_client: Your Reddit client ID
Reddi_secret: Your Reddit client secret
from google.colab import userdata
import os
from uuid import uuid4

unique_id = uuid4().hex[0:8]
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = f"Reddit_finance"
os.environ["LANGCHAIN_ENDPOINT"] = "https://api.smith.langchain.com"
os.environ["LANGCHAIN_API_KEY"] =  userdata.get('Langsmith')
Usage
Scrape Reddit Data

Initialize the Reddit instance and scrape the stocks subreddit:

import praw
import datetime

reddit = praw.Reddit(client_id=userdata.get('Reddit_client'),
                     client_secret=userdata.get('Reddi_secret'),
                     user_agent=True)

subreddit = reddit.subreddit('stocks')
start_date = datetime.datetime.utcnow() - datetime.timedelta(days=180)
start_date_timestamp = int(start_date.timestamp())

with open('reddit_data.txt', 'w', encoding='utf-8') as file:
    for submission in subreddit.new(limit=None):
        if submission.created_utc > start_date_timestamp:
            file.write("Title: " + submission.title + "\n")
            file.write("Body: " + submission.selftext + "\n")
            file.write("\n")
Load Data and Initialize Query Engine
python
Copy code
from llama_index.core import SimpleDirectoryReader
from llama_index.core.llama_pack import download_llama_pack

reader = SimpleDirectoryReader(input_files=["/content/reddit_data.txt"])
documents = reader.load_data()

ZephyrQueryEnginePack = download_llama_pack("ZephyrQueryEnginePack", "./zephyr_pack")
zephyr_pack = ZephyrQueryEnginePack(documents)
Query Data
python
Copy code
response = zephyr_pack.run("most talked about stocks?", similarity_top_k=2)
print(str(response))
