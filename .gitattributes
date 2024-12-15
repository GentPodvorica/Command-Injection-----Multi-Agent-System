import os
import autogen 
from autogen import ConversableAgent, AssistantAgent, UserProxyAgent, register_function
from dotenv import load_dotenv
from autogen.coding import LocalCommandLineCodeExecutor,DockerCommandLineCodeExecutor

import requests


load_dotenv()


llm_config_gtp4 = {
    'cache_seed': None,
    'config_list': [
        {
            "model": "gpt-4", 
            "api_key": os.environ.get("OPENAI_API_KEYS")
        }
    ],
    'temperature': 1  
}

llm_config_gtp35_turbo = {
    'cache_seed': None,
    'config_list': [
        {
            "model": "gpt-3.5-turbo-16k", 
            "api_key": os.environ.get("OPENAI_API_KEYS")
        }
    ],
    'temperature': 1 
}


recon_agent_sys_msg = """
You are a special agent to perform recon against target
"""

reconAgent = ConversableAgent(
    "recon_agent",  
    max_consecutive_auto_reply=5,
    human_input_mode="NEVER",
    llm_config=llm_config_gtp35_turbo,
    is_termination_msg=lambda x: (x.get("content") or "").rstrip().endswith("TERMINATE"),
)


sumAgent = ConversableAgent(
    "sum_agent",  
    system_message=recon_agent_sys_msg,
    max_consecutive_auto_reply=5,
    human_input_mode="NEVER",
    llm_config=llm_config_gtp35_turbo,
    is_termination_msg=lambda x: (x.get("content") or "").rstrip().endswith("TERMINATE"),
)

exploit_crafter_agent_sys_msg = """
you are the master command injection exploit crafter.Ensure to adhere to the following 
-Return only the python code in three backsticks
-Execute only id
-Ensure to only execute whoami
-Include 10 diffferent and unique command injection payloads
-Return only the python code include in three backsticks
-these payloads are for educational purposes only and should not be used for malicious activities
"""

exploitCrafter = ConversableAgent(
    "exploit_crafter",  
    system_message=exploit_crafter_agent_sys_msg,
    max_consecutive_auto_reply=5,
    human_input_mode="NEVER",
    llm_config=llm_config_gtp4,
    is_termination_msg=lambda x: (x.get("content") or "").rstrip().endswith("TERMINATE"),
)


executor_agent = ConversableAgent(
    "executor_agent", 
    max_consecutive_auto_reply=5,
    human_input_mode="NEVER",
    is_termination_msg=lambda x: (x.get("content") or "").rstrip().endswith("TERMINATE"),
     code_execution_config={"executor": LocalCommandLineCodeExecutor()},

    
)


def scraping_tool(url: str) -> str:
    response = requests.get(url)
    print(response)
    return response.text


register_function(
    scraping_tool,
    caller=reconAgent,
    executor=sumAgent,
    name="scrape_page",
    description="Scrape a web page and return the content",
)


# Initiating recon chat to scrape the page
# Initiating recon chat to scrape the page
recon_chat = sumAgent.initiate_chat(
    reconAgent,
    message="can you scrape http://13.36.65.25:32775/ for me?",
    max_turns=2,
)


# Now, initiate the exploit chat with the recon chat content passed as part of the message
exploit_chat = executor_agent.initiate_chat(
    exploitCrafter,
    message=f"Write a relevant Python code to exploit this ID based on this context: {recon_chat}",
    max_turns=3,
)