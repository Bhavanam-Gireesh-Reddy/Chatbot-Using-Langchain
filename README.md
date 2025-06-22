# Chatbot-Using-Langchain
Chat combination of Weather and General Questions

# Explanation

import os: This library is used to interact with the operating system, specifically to access environment variables where you'll store your secret API keys.
import requests: This is a standard library for making HTTP requests to APIs, which is essential for the weather tool to fetch data from OpenWeatherMap.
from dotenv import load_dotenv: This function from the python-dotenv library is used to load variables from a .env file into your environment. This is a best practice for managing secret keys without hardcoding them in your script.
from langchain.chains import ConversationalRetrievalChain: Although imported, this specific chain is not used in the final agent setup. It's typically used for question-answering over documents.
from langchain.memory import ConversationBufferMemory: This is a key component for making the chatbot conversational. It provides a "memory" for the agent to remember past interactions in the conversation.
from langchain_groq import ChatGroq: This is the LangChain integration for the Groq API. It allows you to use models hosted on Groq (like Llama) as the "brain" of your agent. Groq is known for its very high-speed inference.
from langchain_core.prompts import PromptTemplate: This is used to create a structured set of instructions (a prompt) for the language model. The prompt guides the model on how to behave, how to reason, and how to use the available tools.
from langchain.agents import AgentExecutor, Tool, create_react_agent: These are the core components for building the agent.
Tool: A standard wrapper for any function you want the agent to be able to use.
create_react_agent: A function that assembles the language model (LLM), the tools, and the prompt into a coherent "reasoning engine" or agent. The "ReAct" framework stands for "Reasoning and Acting."
AgentExecutor: This is the runtime that actually executes the agent. It takes a user query, passes it to the agent, and manages the loop of "thought -> action -> observation" until a final answer is found.
from langchain_core.messages import AIMessage, HumanMessage: These are data structures used to represent messages from the AI and the human, which is essential for managing the conversation history in memory.
from langchain_community.utilities import GoogleSerperAPIWrapper: This is a convenient pre-built wrapper from LangChain that simplifies making calls to the Google Serper search API.

Defines a standard Python function get_current_weather.
The if not OPENWEATHER_API_KEY: check ensures that the program doesn't crash if the API key is missing, instead returning a helpful error message.
It constructs the API request URL for OpenWeatherMap, including the location, your appid (API key), and the desired units.
The try...except block is for robust error handling.
response = requests.get(...): This sends the actual GET request to the OpenWeatherMap server.
response.raise_for_status(): This will automatically raise an error if the HTTP response indicates a failure (like a 404 Not Found or 401 Unauthorized).
data = response.json(): This parses the JSON response from the server into a Python dictionary.
The rest of the function extracts the relevant weather details (temperature, description, humidity, etc.) from the dictionary and formats them into a clean, human-readable string to be returned.

code turns your Python function into a Tool that the LangChain agent can understand and use.
name="get_current_weather": This is the name the agent will use internally when it decides to call this tool.
func=get_current_weather: This links the tool to the actual Python function you just defined.
description="...": This is the most important part. The language model reads this description to understand what the tool does and when it should be used. A good, clear description is critical for the agent to make correct decisions.

serper_search = GoogleSerperAPIWrapper(): This creates an instance of the helper class that knows how to talk to the Serper API. It will automatically use the SERPER_API_KEY from your environment.
search_tool = Tool(...): Just like the weather tool, this wraps the search functionality into a LangChain Tool.
name="Google Search": A human-readable name for the search tool.
func=serper_search.run: The .run method of the GoogleSerperAPIWrapper is the function that takes a query string and returns the search results.
description="...": Again, this is the crucial description that tells the agent that if it needs to answer questions about general knowledge or current events, it should use this tool.

This block initializes the language model (LLM) that will power the agent.
The try...except block makes the code resilient. If the GROQ_API_KEY is missing or something else goes wrong, it prints an error and falls back to a dummy FakeListLLM.
llm = ChatGroq(...): This creates the LLM object.
temperature=0.7: This parameter controls the "creativity" of the model. A value of 0 is very deterministic, while a value closer to 1 is more creative and random. 0.7 is a good balance.
model_name=LLAMA_MODEL_ID: Specifies which model to use on Groq.
groq_api_key=GROQ_API_KEY: Passes your API key for authentication.

memory = ...: This creates the memory object.
memory_key="chat_history": This key is used in the prompt to tell the agent where to insert the conversation history.
return_messages=True: This ensures the history is stored as a structured list of HumanMessage and AIMessage objects.
prompt = ...: This defines the master instruction set for the agent. Let's break down the template string:
{tools} and {tool_names}: These are placeholders where LangChain will automatically insert the list of available tools and their names.
Thought: ... Action: ... Action Input: ... Observation: ...: This defines the "ReAct" format. It instructs the LLM to "think" out loud about what it needs to do, choose a tool (Action), provide the input for that tool (Action Input), and then wait for the result (Observation). It can repeat this cycle many times.
{chat_history}: The placeholder for the conversation memory.
{input}: The placeholder for the user's latest question.
{agent_scratchpad}: A temporary working area for the agent to keep track of its thought/action/observation steps for the current query.

agent = create_react_agent(...): This function takes the LLM, the list of tools you created, and the prompt, and combines them into the agent's reasoning logic.
agent_executor = AgentExecutor(...): This creates the runtime environment for the agent.
agent=agent: The reasoning logic to use.
tools=[weather_tool, search_tool]: The list of tools the executor can provide to the agent.
verbose=True: This is a fantastic debugging tool. It prints the agent's internal "thoughts" to the console, so you can see its entire reasoning process.
memory=memory: Links the executor to the conversation memory object.
handle_parsing_errors=True: Makes the agent more robust. If the LLM generates a malformed response, the executor will try to handle it gracefully instead of crashing.

This final cell contains the code to actually interact with the chatbot.
if __name__ == "__main__":: This is a standard Python construct that ensures the chat_with_bot() function is called only when the script is run directly.
while True:: This creates an infinite loop so you can have an ongoing conversation.
user_input = input("You: "): Prompts you for input from the keyboard.
agent_executor.invoke({"input": user_input}): This is the command that triggers the entire agent system. It takes your input, passes it to the agent, which then goes through its "ReAct" cycle (thinking, using tools) until it arrives at a final answer.
print(f"Chatbot: {response['output']}"): The invoke method returns a dictionary, and the final answer is stored in the output key, which is then printed.
