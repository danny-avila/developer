
I would like you to help me build a langchain-serve API. The following after the first set of "###" is a good majority of their docs to assist you in developing this. In particular, agents, HITL, and pandas integrations are of importance for me. You should also dockerize the API. For any libraries you are unfamiliar with, like langchain, you can just code up the langchain-serve templates for me to fill.

###

**langchain-serve** helps you deploy your LangChain apps on Jina AI Cloud in just a matter of seconds. You can now benefit from the scalability and serverless architecture of the cloud without sacrificing the ease and convenience of local development.

> Give us a :star: and tell us what more you'd like to see! 

# LLM Apps as-a-service

langchain-serve currently wraps following apps as a service to be deployed on Jina AI Cloud with one command.

# Usage

Let's first install `langchain-serve` using pip.

```bash
pip install langchain-serve
```

## REST APIs using `@serving` decorator

Let's build & deploy a custom agent using this example taken from [LangChain documentation](https://python.langchain.com/en/latest/modules/agents/agents/custom_agent.html). 

agent code (app.py)
```python
# app.py
from langchain.agents import ZeroShotAgent, Tool, AgentExecutor
from langchain import OpenAI, SerpAPIWrapper, LLMChain

search = SerpAPIWrapper()
tools = [
    Tool(
        name = "Search",
        func=search.run,
        description="useful for when you need to answer questions about current events"
    )
]

prefix = """Answer the following questions as best you can, but speaking as a pirate might speak. You have access to the following tools:"""
suffix = """Begin! Remember to speak as a pirate when giving your final answer. Use lots of "Args"

Question: {input}
{agent_scratchpad}"""

prompt = ZeroShotAgent.create_prompt(
    tools, 
    prefix=prefix, 
    suffix=suffix, 
    input_variables=["input", "agent_scratchpad"]
)

llm_chain = LLMChain(llm=OpenAI(temperature=0), prompt=prompt)
tool_names = [tool.name for tool in tools]
agent = ZeroShotAgent(llm_chain=llm_chain, allowed_tools=tool_names)
agent_executor = AgentExecutor.from_agent_and_tools(agent=agent, tools=tools, verbose=True)
agent_executor.run("How many people live in canada as of 2023?")
```

#### Output


```text
> Entering new AgentExecutor chain...
Thought: I need to find out the population of Canada
Action: Search
Action Input: Population of Canada 2023
Observation: The current population of Canada is 38,610,447 as of Saturday, February 18, 2023, based on Worldometer elaboration of the latest United Nations data. Canada 2020 population is estimated at 37,742,154 people at mid year according to UN data.
Thought: I now know the final answer
Final Answer: Arrr, Canada be havin' 38,610,447 scallywags livin' there as of 2023!

> Finished chain.
```

### Step 1: 

**Refactor your code to function(s) that should be served with `@serving` decorator**


updated agent code (app.py)

```python
# app.py
from langchain import LLMChain, OpenAI, SerpAPIWrapper
from langchain.agents import AgentExecutor, Tool, ZeroShotAgent

from lcserve import serving


@serving
def ask(input: str) -> str:
    search = SerpAPIWrapper()
    tools = [
        Tool(
            name="Search",
            func=search.run,
            description="useful for when you need to answer questions about current events",
        )
    ]
    prefix = """Answer the following questions as best you can, but speaking as a pirate might speak. You have access to the following tools:"""
    suffix = """Begin! Remember to speak as a pirate when giving your final answer. Use lots of "Args"

    Question: {input}
    {agent_scratchpad}"""

    prompt = ZeroShotAgent.create_prompt(
        tools,
        prefix=prefix,
        suffix=suffix,
        input_variables=["input", "agent_scratchpad"],
    )

    print(prompt.template)

    llm_chain = LLMChain(llm=OpenAI(temperature=0), prompt=prompt)
    tool_names = [tool.name for tool in tools]
    agent = ZeroShotAgent(llm_chain=llm_chain, allowed_tools=tool_names)

    agent_executor = AgentExecutor.from_agent_and_tools(
        agent=agent, tools=tools, verbose=True
    )

    return agent_executor.run(input)

if __name__ == "__main__":
    ask('How many people live in canada as of 2023?')
```

</details>


##### What changed?

- We moved our code to an `ask` function.
- Added type hints to the function parameters (input and output), so API definition can be generated.
- Imported `from lcserve import serving` and added `@serving` decorator to the `ask` function.
- Added `if __name__ == "__main__":` block to test the function locally.

---

### Step 2:

**Create a `requirements.txt` file in your app directory to ensure all necessary dependencies are installed.**
requirements.txt

```text
# requirements.txt
openai
google-search-results
```

--- 

### Step 3:

**Run `lc-serve deploy local app` to test your API locally.**

> `app` is the name of the module that contains the `ask` function.

```bash
lc-serve deploy local app
```

Let's test our local API with `How many people live in canada as of 2023?` input with a cURL command.

```bash
curl -X 'POST' \
  'http://localhost:8080/ask' \
  -H 'accept: application/json' \
  -H 'Content-Type: application/json' \
  -d '{
  "input": "How many people live in canada as of 2023?",
  "envs": {
    "OPENAI_API_KEY": "'"${OPENAI_API_KEY}"'",
    "SERPAPI_API_KEY": "'"${SERPAPI_API_KEY}"'"
  }
}'
```

```json
{
  "result": "Arrr, there be 38,645,670 people livin' in Canada as of 2023!",
  "error": "",
  "stdout": "Answer the following questions as best you can, but speaking as a pirate might speak. You have access to the following tools:\n\nSearch: useful for when you need to answer questions about current events\n\nUse the following format:\n\nQuestion: the input question you must answer\nThought: you should always think about what to do\nAction: the action to take, should be one of [Search]\nAction Input: the input to the action\nObservation: the result of the action\n... (this Thought/Action/Action Input/Observation can repeat N times)\nThought: I now know the final answer\nFinal Answer: the final answer to the original input question\n\nBegin! Remember to speak as a pirate when giving your final answer. Use lots of \"Args\"\n\n    Question: {input}\n    {agent_scratchpad}\n\n\n\u001b[1m> Entering new AgentExecutor chain...\u001b[0m\n\u001b[32;1m\u001b[1;3m\nThought: I need to find out how many people live in Canada\nAction: Search\nAction Input: How many people live in Canada as of 2023\u001b[0m\nObservation: \u001b[36;1m\u001b[1;3mThe current population of Canada is 38,645,670 as of Wednesday, March 29, 2023, based on Worldometer elaboration of the latest United Nations data.\u001b[0m\nThought:\u001b[32;1m\u001b[1;3m I now know the final answer\nFinal Answer: Arrr, there be 38,645,670 people livin' in Canada as of 2023!\u001b[0m\n\n\u001b[1m> Finished chain.\u001b[0m"
}
```

##### What happened?

- `POST /ask` is generated from `ask` function defined in `app.py`.
- `input` is an argrment defined in `ask` function. 
- `envs` is a dictionary of environment variables that will be passed to all the functions decorated with `@serving` decorator.
- return type of `ask` function is `str`. So, `result` would carry the return value of `ask` function.
- If there is an error, `error` would carry the error message.
- `stdout` would carry the output of the function decorated with `@serving` decorator.


## Human in the Loop

Here is a simple example of how to use websockets to create a human in the loop (HITL) system. This demo uses `ChatOpenAI` in streaming mode to send responses to the client as soon as they are available. The client has the context available to it. When `human` action is needed, client asks the user for the input and sends it back to the server. The server then uses the context to continue the conversation.

### Setup

This directory contains 3 files

#### `hitl.py` 

1. Defines a function `hitl` decorated with `@serving` with `websocket=True`.
2. Gets `streaming_handler` from kwargs and passes it to `ChatOpenAI` and `OpenAI` callback managers. This handler is responsible to stream the response to the client.
3. Returns `agent.run` output which is finally streamed to the client.
---

#### `requirements.txt`

Contains the dependencies for the `hitl` endpoint.

---

#### Deploy using `lc-serve`

```bash
lc-serve deploy jcloud hitl
```

---

#### `hitl_client.py`

A simple python client that connects to the websocket server and sends the question to the `hitl` endpoint. It then listens to the stream of responses and prints it to the console. When it receives a response in the following format, it asks the prompt to the user using the client and waits for the user to input the answer. (This is how human is brought into the loop). Next, this answer is then sent to the server.

This can be implemented in any language that supports websockets.


1. Connects to the websocket server and sends the following to the `hitl` endpoint.

    ```json
    {
        "question": "${question}", 
        "envs": {
            "OPENAI_API_KEY": "${OPENAI_API_KEY}"
        }
    }
    ```

2. Listens to the stream of responses and prints it to the console

3. When it receives a response in the following format, it asks the prompt to the user using the client and waits for the user to input the answer. (This is how human is brought into the loop). Next, this answer is then sent to the server.

    ```json
    {
        "prompt": "$prompt"
    }
    ```

    https://github.com/jina-ai/langchain-serve/blob/c0af0de263c4494ce457faa4c9b82cab61992c5a/examples/websockets/hitl/hitl_client.py#L42-L44

4. Finally, the client is disconnected from the server automatically when the `hitl` function is done executing.

---


### Example run on localhost

```bash
python hitl_client.py
```

```text
Connected to ws://localhost:8080/hitl.
I don't know Eric Zhu's birthday, so I need to ask a human.
Action: Human
Action Input: "Do you know Eric Zhu and his birthday?"
Yes
Great, now I can ask for Eric Zhu's birthday.
Action: Human
Action Input: "What is Eric Zhu's birthday?"
29th Feb
I need to make sure this is a valid date.
Action: Calculator
Action Input: Check if 29th Feb is a valid date
```

```python
import datetime

try:
    datetime.datetime(2020, 2, 29)
    print("Valid date")
except ValueError:
    print("Invalid date")
```

```text
I now have a valid birth date, but I need to know the year for Eric's age.
Action: Human
Action Input: "Do you know Eric Zhu's birth year?"
1990
Now I can calculate Eric Zhu's age.
Action: Calculator
Action Input: Current year minus 1990
```

```python
import datetime
print(datetime.datetime.now().year - 1990)
```

```text
I now know Eric Zhu's age.
Final Answer: Eric Zhu's birthday is February 29th, 1990 and he is currently 33 years old.Eric Zhu's birthday is February 29th, 1990 and he is currently 33 years old.% 
```
# examples/websockets/hitl/hitl.py

import os

from langchain.agents import initialize_agent, load_tools

# In order to be compatible with different langchain versions
try:
    from langchain.callbacks.manager import CallbackManager
except ImportError:
    from langchain.callbacks import CallbackManager

from langchain.chat_models import ChatOpenAI
from langchain.llms import OpenAI

from lcserve import serving


@serving(websocket=True)
def hitl(question: str, **kwargs) -> str:
    # Get the `streaming_handler` from `kwargs`. This is used to stream data to the client.
    streaming_handler = kwargs.get('streaming_handler')

    llm = ChatOpenAI(
        temperature=0.0,
        verbose=True,
        streaming=True,  # Pass `streaming=True` to make sure the client receives the data.
        callback_manager=CallbackManager(
            [streaming_handler]
        ),  # Pass the callback handler
    )
    math_llm = OpenAI(
        temperature=0.0,
        verbose=True,
        streaming=True,  # Pass `streaming=True` to make sure the client receives the data.
        callback_manager=CallbackManager(
            [streaming_handler]
        ),  # Pass the callback handler
    )
    tools = load_tools(
        ["human"],
        llm=math_llm,
    )

    agent_chain = initialize_agent(
        tools,
        llm,
        agent="zero-shot-react-description",
        verbose=True,
    )
    return agent_chain.run(question)

# examples/websockets/hitl/hitl_client.py

import asyncio
import os
from typing import Dict

import aiohttp
from pydantic import BaseModel, ValidationError


class Response(BaseModel):
    result: str
    error: str
    stdout: str


class HumanPrompt(BaseModel):
    prompt: str


async def hitl_client(url: str, name: str, question: str, envs: Dict = {}):
    async with aiohttp.ClientSession() as session:
        async with session.ws_connect(f'{url}/{name}') as ws:
            print(f'Connected to {url}/{name}.')

            await ws.send_json(
                {
                    "question": question,
                    "envs": envs if envs else {},
                }
            )

            async for msg in ws:
                if msg.type == aiohttp.WSMsgType.TEXT:
                    if msg.data == 'close cmd':
                        await ws.close()
                        break
                    else:
                        try:
                            response = Response.parse_raw(msg.data)
                            print(response.result, end='')
                        except ValidationError:
                            try:
                                prompt = HumanPrompt.parse_raw(msg.data)
                                answer = input(prompt.prompt + '\n')
                                await ws.send_str(answer)
                            except ValidationError:
                                print(f'Unknown message: {msg.data}')

                elif msg.type == aiohttp.WSMsgType.ERROR:
                    print('ws connection closed with exception %s' % ws.exception())
                else:
                    print(msg)


asyncio.run(
    hitl_client(
        url='wss://langchain-72aff35874.wolf.jina.ai',
        name='hitl',
        question='What is Eric Zhu\'s birthday?',
        envs={
            'OPENAI_API_KEY': os.environ['OPENAI_API_KEY'],
        },
    )
)