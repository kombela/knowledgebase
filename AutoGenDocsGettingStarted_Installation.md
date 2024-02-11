# Getting Started

AutoGen is a framework that enables development of LLM applications using multiple agents that can converse with each other to solve tasks. AutoGen agents are customizable, conversable, and seamlessly allow human participation. They can operate in various modes that employ combinations of LLMs, human inputs, and tools.

![AutoGen Overview](https://microsoft.github.io/autogen/assets/images/autogen_agentchat-250ca64b77b87e70d34766a080bf6ba8.png)

### Main Features[​](https://microsoft.github.io/autogen/docs/Getting-Started#main-features "Direct link to Main Features")

- AutoGen enables building next-gen LLM applications based on [multi-agent conversations](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat) with minimal effort. It simplifies the orchestration, automation, and optimization of a complex LLM workflow. It maximizes the performance of LLM models and overcomes their weaknesses.
- It supports [diverse conversation patterns](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#supporting-diverse-conversation-patterns) for complex workflows. With customizable and conversable agents, developers can use AutoGen to build a wide range of conversation patterns concerning conversation autonomy, the number of agents, and agent conversation topology.
- It provides a collection of working systems with different complexities. These systems span a [wide range of applications](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#diverse-applications-implemented-with-autogen) from various domains and complexities. This demonstrates how AutoGen can easily support diverse conversation patterns.
- AutoGen provides [enhanced LLM inference](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#api-unification). It offers utilities like API unification and caching, and advanced usage patterns, such as error handling, multi-config inference, context programming, etc.

AutoGen is powered by collaborative [research studies](https://microsoft.github.io/autogen/docs/Research) from Microsoft, Penn State University, and University of Washington.

### Quickstart[​](https://microsoft.github.io/autogen/docs/Getting-Started#quickstart "Direct link to Quickstart")

Install from pip: `pip install pyautogen`. Find more options in [Installation](https://microsoft.github.io/autogen/docs/Installation). For [code execution](https://microsoft.github.io/autogen/docs/FAQ#code-execution), we strongly recommend installing the python docker package, and using docker.

#### Multi-Agent Conversation Framework[​](https://microsoft.github.io/autogen/docs/Getting-Started#multi-agent-conversation-framework "Direct link to Multi-Agent Conversation Framework")

Autogen enables the next-gen LLM applications with a generic multi-agent conversation framework. It offers customizable and conversable agents which integrate LLMs, tools, and humans. By automating chat among multiple capable agents, one can easily make them collectively perform tasks autonomously or with human feedback, including tasks that require using tools via code. For [example](https://github.com/microsoft/autogen/blob/main/test/twoagent.py),

```
from autogen import AssistantAgent, UserProxyAgent, config_list_from_json# Load LLM inference endpoints from an env variable or a file# See https://microsoft.github.io/autogen/docs/FAQ#set-your-api-endpoints# and OAI_CONFIG_LIST_sample.jsonconfig_list = config_list_from_json(env_or_file="OAI_CONFIG_LIST")assistant = AssistantAgent("assistant", llm_config={"config_list": config_list})user_proxy = UserProxyAgent("user_proxy", code_execution_config={"work_dir": "coding", "use_docker": False}) # IMPORTANT: set to True to run code in docker, recommendeduser_proxy.initiate_chat(assistant, message="Plot a chart of NVDA and TESLA stock price change YTD.")# This initiates an automated chat between the two agents to solve the task
```

The figure below shows an example conversation flow with AutoGen. ![Agent Chat Example](https://microsoft.github.io/autogen/assets/images/chat_example-da70a7420ebc817ef9826fa4b1e80951.png)

- [Code examples](https://microsoft.github.io/autogen/docs/Examples).
- [Documentation](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat).

#### Enhanced LLM Inferences[​](https://microsoft.github.io/autogen/docs/Getting-Started#enhanced-llm-inferences "Direct link to Enhanced LLM Inferences")

Autogen also helps maximize the utility out of the expensive LLMs such as ChatGPT and GPT-4. It offers enhanced LLM inference with powerful functionalities like tuning, caching, error handling, templating. For example, you can optimize generations by LLM with your own tuning data, success metrics and budgets.

```
# perform tuning for openai<1config, analysis = autogen.Completion.tune(    data=tune_data,    metric="success",    mode="max",    eval_func=eval_func,    inference_budget=0.05,    optimization_budget=3,    num_samples=-1,)# perform inference for a test instanceresponse = autogen.Completion.create(context=test_instance, **config)
```

- [Code examples](https://microsoft.github.io/autogen/docs/Examples).
- [Documentation](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference).

### Where to Go Next ?[​](https://microsoft.github.io/autogen/docs/Getting-Started#where-to-go-next- "Direct link to Where to Go Next ?")

- Understand the use cases for [multi-agent conversation](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat) and [enhanced LLM inference](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference).
- Find [code examples](https://microsoft.github.io/autogen/docs/Examples).
- Read [SDK](https://microsoft.github.io/autogen/docs/reference/agentchat/conversable_agent/).
- Learn about [research](https://microsoft.github.io/autogen/docs/Research) around AutoGen.
- [Roadmap](https://github.com/orgs/microsoft/projects/989/views/3)
- Chat on [Discord](https://discord.gg/pAbnFJrkgZ).
- Follow on [Twitter](https://twitter.com/pyautogen).

If you like our project, please give it a [star](https://github.com/microsoft/autogen/stargazers) on GitHub. If you are interested in contributing, please read [Contributor's Guide](https://microsoft.github.io/autogen/docs/Contribute).

---

# Installation

## Create a virtual environment (optional)[​](https://microsoft.github.io/autogen/docs/installation/#create-a-virtual-environment-optional "Direct link to Create a virtual environment (optional)")

When installing AutoGen locally, we recommend using a virtual environment for the installation. This will ensure that the dependencies for AutoGen are isolated from the rest of your system.

- venv
- Conda
- Poetry

Create and activate:

```
python3 -m venv pyautogensource pyautogen/bin/activate
```

To deactivate later, run:

```
deactivate
```

## Install AutoGen[​](https://microsoft.github.io/autogen/docs/installation/#install-autogen "Direct link to Install AutoGen")

AutoGen requires **Python version >= 3.8, < 3.13**. It can be installed from pip:

```
pip install pyautogen
```

INFO

`pyautogen<0.2` required `openai<1`. Starting from pyautogen v0.2, `openai>=1` is required.

## Code execution with Docker (default)[​](https://microsoft.github.io/autogen/docs/installation/#code-execution-with-docker-default "Direct link to Code execution with Docker (default)")

Even if you install AutoGen locally, we highly recommend using Docker for [code execution](https://microsoft.github.io/autogen/docs/FAQ#code-execution).

The default behaviour for code-execution agents is for code execution to be performed in a docker container.

**To turn this off**: if you want to run the code locally (not recommended) then `use_docker` can be set to `False` in `code_execution_config` for each code-execution agent, or set `AUTOGEN_USE_DOCKER` to `False` as an environment variable.

You might want to override the default docker image used for code execution. To do that set `use_docker` key of `code_execution_config` property to the name of the image. E.g.:

```
user_proxy = autogen.UserProxyAgent(    name="agent",    human_input_mode="TERMINATE",    max_consecutive_auto_reply=10,    code_execution_config={"work_dir":"_output", "use_docker":"python:3"},    llm_config=llm_config,    system_message=""""Reply TERMINATE if the task has been solved at full satisfaction.Otherwise, reply CONTINUE, or the reason why the task is not solved yet.""")
```

**Turn off code execution entirely**: if you want to turn off code execution entirely, set `code_execution_config` to `False`. E.g.:

```
user_proxy = autogen.UserProxyAgent(    name="agent",    llm_config=llm_config,    code_execution_config=False,)
```

---

# Docker

Docker, an indispensable tool in modern software development, offers a compelling solution for AutoGen's setup. Docker allows you to create consistent environments that are portable and isolated from the host OS. With Docker, everything AutoGen needs to run, from the operating system to specific libraries, is encapsulated in a container, ensuring uniform functionality across different systems. The Dockerfiles necessary for AutoGen are conveniently located in the project's GitHub repository at [https://github.com/microsoft/autogen/tree/main/samples/dockers](https://github.com/microsoft/autogen/tree/main/samples/dockers).

**Pre-configured DockerFiles**: The AutoGen Project offers pre-configured Dockerfiles for your use. These Dockerfiles will run as is, however they can be modified to suit your development needs. Please see the README.md file in autogen/samples/dockers

- **autogen_base_img**: For a basic setup, you can use the `autogen_base_img` to run simple scripts or applications. This is ideal for general users or those new to AutoGen.
- **autogen_full_img**: Advanced users or those requiring more features can use `autogen_full_img`. Be aware that this version loads ALL THE THINGS and thus is very large. Take this into consideration if you build your application off of it.

## Step 1: Install Docker[​](https://microsoft.github.io/autogen/docs/installation/Docker#step-1-install-docker "Direct link to Step 1: Install Docker")

- **General Installation**: Follow the [official Docker installation instructions](https://docs.docker.com/get-docker/). This is your first step towards a containerized environment, ensuring a consistent and isolated workspace for AutoGen.
    
- **For Mac Users**: If you encounter issues with the Docker daemon, consider using [colima](https://smallsharpsoftwaretools.com/tutorials/use-colima-to-run-docker-containers-on-macos/). Colima offers a lightweight alternative to manage Docker containers efficiently on macOS.
    

## Step 2: Build a Docker Image[​](https://microsoft.github.io/autogen/docs/installation/Docker#step-2-build-a-docker-image "Direct link to Step 2: Build a Docker Image")

AutoGen now provides updated Dockerfiles tailored for different needs. Building a Docker image is akin to setting the foundation for your project's environment:

- **Autogen Basic**: Ideal for general use, this setup includes common Python libraries and essential dependencies. Perfect for those just starting with AutoGen.
    
    ```
    docker build -f .devcontainer/base/Dockerfile -t autogen_base_img https://github.com/microsoft/autogen.git
    ```
    
- **Autogen Advanced**: Advanced users or those requiring all the things that AutoGen has to offer `autogen_full_img`
    
    ```
    docker build -f .devcontainer/full/Dockerfile -t autogen_full_img https://github.com/microsoft/autogen.git
    ```
    

## Step 3: Run AutoGen Applications from Docker Image[​](https://microsoft.github.io/autogen/docs/installation/Docker#step-3-run-autogen-applications-from-docker-image "Direct link to Step 3: Run AutoGen Applications from Docker Image")

Here's how you can run an application built with AutoGen, using the Docker image:

1. **Mount Your Directory**: Use the Docker `-v` flag to mount your local application directory to the Docker container. This allows you to develop on your local machine while running the code in a consistent Docker environment. For example:
    
    ```
    docker run -it -v $(pwd)/myapp:/home/autogen/autogen/myapp autogen_base_img:latest python /home/autogen/autogen/myapp/main.py
    ```
    
    Here, `$(pwd)/myapp` is your local directory, and `/home/autogen/autogen/myapp` is the path in the Docker container where your code will be located.
    
2. **Mount your code:** Now suppose you have your application built with AutoGen in a main script named `twoagent.py` ([example](https://github.com/microsoft/autogen/blob/main/test/twoagent.py)) in a folder named `myapp`. With the command line below, you can mount your folder and run the application in Docker.
    
    ```
    # Mount the local folder `myapp` into docker image and run the script named "twoagent.py" in the docker.docker run -it -v `pwd`/myapp:/myapp autogen_img:latest python /myapp/main_twoagent.py
    ```
    
3. **Port Mapping**: If your application requires a specific port, use the `-p` flag to map the container's port to your host. For instance, if your app runs on port 3000 inside Docker and you want it accessible on port 8080 on your host machine:
    
    ```
    docker run -it -p 8080:3000 -v $(pwd)/myapp:/myapp autogen_base_img:latest python /myapp
    ```
    
    In this command, `-p 8080:3000` maps port 3000 from the container to port 8080 on your local machine.
    
4. **Examples of Running Different Applications**: Here is the basic format of the docker run command.
    

```
docker run -it -p {WorkstationPortNum}:{DockerPortNum} -v {WorkStation_Dir}:{Docker_DIR} {name_of_the_image} {bash/python} {Docker_path_to_script_to_execute}
```

- _Simple Script_: Run a Python script located in your local `myapp` directory.
    
    ```
    docker run -it -v `pwd`/myapp:/myapp autogen_base_img:latest python /myapp/my_script.py
    ```
    
- _Web Application_: If your application includes a web server running on port 5000.
    
    ```
    docker run -it -p 8080:5000 -v $(pwd)/myapp:/myapp autogen_base_img:latest
    ```
    
- _Data Processing_: For tasks that involve processing data stored in a local directory.
    
    ```
    docker run -it -v $(pwd)/data:/data autogen_base_img:latest python /myapp/process_data.py
    ```
    

## Additional Resources[​](https://microsoft.github.io/autogen/docs/installation/Docker#additional-resources "Direct link to Additional Resources")

- Details on all the Dockerfile options can be found in the [Dockerfile](https://github.com/microsoft/autogen/.devcontainer/README.md) README.
- For more information on Docker usage and best practices, refer to the [official Docker documentation](https://docs.docker.com/).
- Details on how to use the Dockerfile dev version can be found on the [Contributing](https://microsoft.github.io/autogen/docs/Contribute#docker)

---

# Optional Dependencies

## LLM Caching[​](https://microsoft.github.io/autogen/docs/installation/Optional-Dependencies#llm-caching "Direct link to LLM Caching")

To use LLM caching with Redis, you need to install the Python package with the option `redis`:

```
pip install "pyautogen[redis]"
```

See [LLM Caching](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#llm-caching) for details.

## Docker[​](https://microsoft.github.io/autogen/docs/installation/Optional-Dependencies#docker "Direct link to Docker")

Even if you install AutoGen locally, we highly recommend using Docker for [code execution](https://microsoft.github.io/autogen/docs/FAQ#enable-python-3-docker-image).

To use docker for code execution, you also need to install the python package `docker`:

```
pip install docker
```

You might want to override the default docker image used for code execution. To do that set `use_docker` key of `code_execution_config` property to the name of the image. E.g.:

```
user_proxy = autogen.UserProxyAgent(    name="agent",    human_input_mode="TERMINATE",    max_consecutive_auto_reply=10,    code_execution_config={"work_dir":"_output", "use_docker":"python:3"},    llm_config=llm_config,    system_message=""""Reply TERMINATE if the task has been solved at full satisfaction.Otherwise, reply CONTINUE, or the reason why the task is not solved yet.""")
```

## blendsearch[​](https://microsoft.github.io/autogen/docs/installation/Optional-Dependencies#blendsearch "Direct link to blendsearch")

`pyautogen<0.2` offers a cost-effective hyperparameter optimization technique [EcoOptiGen](https://arxiv.org/abs/2303.04673) for tuning Large Language Models. Please install with the [blendsearch] option to use it.

```
pip install "pyautogen[blendsearch]<0.2"
```

Example notebooks:

[Optimize for Code Generation](https://github.com/microsoft/autogen/blob/main/notebook/oai_completion.ipynb)

[Optimize for Math](https://github.com/microsoft/autogen/blob/main/notebook/oai_chatgpt_gpt4.ipynb)

## retrievechat[​](https://microsoft.github.io/autogen/docs/installation/Optional-Dependencies#retrievechat "Direct link to retrievechat")

`pyautogen` supports retrieval-augmented generation tasks such as question answering and code generation with RAG agents. Please install with the [retrievechat] option to use it.

```
pip install "pyautogen[retrievechat]"
```

RetrieveChat can handle various types of documents. By default, it can process plain text and PDF files, including formats such as 'txt', 'json', 'csv', 'tsv', 'md', 'html', 'htm', 'rtf', 'rst', 'jsonl', 'log', 'xml', 'yaml', 'yml' and 'pdf'. If you install [unstructured](https://unstructured-io.github.io/unstructured/installation/full_installation.html) (`pip install "unstructured[all-docs]"`), additional document types such as 'docx', 'doc', 'odt', 'pptx', 'ppt', 'xlsx', 'eml', 'msg', 'epub' will also be supported.

You can find a list of all supported document types by using `autogen.retrieve_utils.TEXT_FORMATS`.

Example notebooks:

[Automated Code Generation and Question Answering with Retrieval Augmented Agents](https://github.com/microsoft/autogen/blob/main/notebook/agentchat_RetrieveChat.ipynb)

[Group Chat with Retrieval Augmented Generation (with 5 group member agents and 1 manager agent)](https://github.com/microsoft/autogen/blob/main/notebook/agentchat_groupchat_RAG.ipynb)

[Automated Code Generation and Question Answering with Qdrant based Retrieval Augmented Agents](https://github.com/microsoft/autogen/blob/main/notebook/agentchat_qdrant_RetrieveChat.ipynb)

## Teachability[​](https://microsoft.github.io/autogen/docs/installation/Optional-Dependencies#teachability "Direct link to Teachability")

To use Teachability, please install AutoGen with the [teachable] option.

```
pip install "pyautogen[teachable]"
```

Example notebook: [Chatting with a teachable agent](https://github.com/microsoft/autogen/blob/main/notebook/agentchat_teachability.ipynb)

## Large Multimodal Model (LMM) Agents[​](https://microsoft.github.io/autogen/docs/installation/Optional-Dependencies#large-multimodal-model-lmm-agents "Direct link to Large Multimodal Model (LMM) Agents")

We offered Multimodal Conversable Agent and LLaVA Agent. Please install with the [lmm] option to use it.

```
pip install "pyautogen[lmm]"
```

Example notebooks:

[LLaVA Agent](https://github.com/microsoft/autogen/blob/main/notebook/agentchat_lmm_llava.ipynb)

## mathchat[​](https://microsoft.github.io/autogen/docs/installation/Optional-Dependencies#mathchat "Direct link to mathchat")

`pyautogen<0.2` offers an experimental agent for math problem solving. Please install with the [mathchat] option to use it.

```
pip install "pyautogen[mathchat]<0.2"
```

Example notebooks:

[Using MathChat to Solve Math Problems](https://github.com/microsoft/autogen/blob/main/notebook/agentchat_MathChat.ipynb)

---


# LLM Endpoint Configuration

## TL;DR[​](https://microsoft.github.io/autogen/docs/llm_endpoint_configuration#tldr "Direct link to TL;DR")

For just getting started with AutoGen you can use the following to define your LLM endpoint configuration:

```
import autogenconfig_list = autogen.get_config_list(["YOUR_OPENAI_API_KEY"], api_type="openai")
```

DANGER

Never commit secrets into your code. Before committing, change the code to use a different way of providing your API keys as described below.

## In-depth[​](https://microsoft.github.io/autogen/docs/llm_endpoint_configuration#in-depth "Direct link to In-depth")

Managing API configurations can be tricky, especially when dealing with multiple models and API versions. The provided utility functions assist users in managing these configurations effectively. Ensure your API keys and other sensitive data are stored securely. You might store keys in `.txt` or `.env` files or environment variables for local development. Never expose your API keys publicly. If you insist on storing your key files locally on your repo (you shouldn’t), ensure the key file path is added to the `.gitignore` file.

### Steps:[​](https://microsoft.github.io/autogen/docs/llm_endpoint_configuration#steps "Direct link to Steps:")

1. Obtain API keys from OpenAI and optionally from Azure OpenAI (or other provider).
2. Store them securely using either:
    - Environment Variables: `export OPENAI_API_KEY='your-key'` in your shell.
    - Text File: Save the key in a `key_openai.txt` file.
    - Env File: Save the key to a `.env` file eg: `OPENAI_API_KEY=sk-********************`
3. [Ensure `pyautogen` is installed](https://microsoft.github.io/autogen/docs/installation/)

There are many ways to generate a `config_list` depending on your use case:

- `get_config_list`: Generates configurations for API calls, primarily from provided API keys.
- `config_list_openai_aoai`: Constructs a list of configurations using both Azure OpenAI and OpenAI endpoints, sourcing API keys from environment variables or local files.
- `config_list_from_json`: Loads configurations from a JSON structure, either from an environment variable or a local JSON file, with the flexibility of filtering configurations based on given criteria.
- `config_list_from_models`: Creates configurations based on a provided list of models, useful when targeting specific models without manually specifying each configuration.
- `config_list_from_dotenv`: Constructs a configuration list from a `.env` file, offering a consolidated way to manage multiple API configurations and keys from a single file.

If multiple models are provided, the Autogen client (`OpenAIWrapper`) and agents don’t choose the “best model” on any criteria - inference is done through the very first model and the next one is used only if the current model fails (e.g. API throttling by the provider or a filter condition is unsatisfied).

### What is a `config_list`?[​](https://microsoft.github.io/autogen/docs/llm_endpoint_configuration#what-is-a-config_list "Direct link to what-is-a-config_list")

When instantiating an assistant, such as the example below, it is passed a `config_list`. This is used to tell the `AssistantAgent` which models or configurations it has access to:

```
assistant = AssistantAgent(    name="assistant",    llm_config={        "timeout": 600,        "cache_seed": 42,        "config_list": config_list,        "temperature": 0,    },)
```

Consider an intelligent assistant that utilizes OpenAI’s GPT models. Depending on user requests, it might need to:

- Generate creative content (using gpt-4).
- Answer general queries (using gpt-3.5-turbo).

Different tasks may require different models, and the `config_list` aids in dynamically selecting the appropriate model configuration, managing API keys, endpoints, and versions for efficient operation of the intelligent assistant. In summary, the `config_list` helps the agents work efficiently, reliably, and optimally by managing various configurations and interactions with the OpenAI API - enhancing the adaptability and functionality of the agents.

## get_config_list[​](https://microsoft.github.io/autogen/docs/llm_endpoint_configuration#get_config_list "Direct link to get_config_list")

Used to generate configurations for API calls.

```
api_keys = ["YOUR_OPENAI_API_KEY"]base_urls = None  # You can specify API base URLs if needed. eg: localhost:8000api_type = "openai"  # Type of API, e.g., "openai" or "aoai".api_version = None  # Specify API version if needed.config_list = autogen.get_config_list(api_keys, base_urls=base_urls, api_type=api_type, api_version=api_version)print(config_list)
```

```
[{'api_key': 'YOUR_OPENAI_API_KEY', 'api_type': 'openai'}]
```

## config_list_openai_aoai[​](https://microsoft.github.io/autogen/docs/llm_endpoint_configuration#config_list_openai_aoai "Direct link to config_list_openai_aoai")

This method creates a list of configurations using Azure OpenAI endpoints and OpenAI endpoints. It tries to extract API keys and bases from environment variables or local text files.

Steps: - Store OpenAI API key in: - Environment variable: `OPENAI_API_KEY` - or Local file: `key_openai.txt` - Store Azure OpenAI API key in: - Environment variable: `AZURE_OPENAI_API_KEY` - or Local file: `key_aoai.txt` (Supports multiple keys, one per line) - Store Azure OpenAI API base in: - Environment variable: `AZURE_OPENAI_API_BASE` - or Local file: `base_aoai.txt` (Supports multiple bases, one per line)

```
config_list = autogen.config_list_openai_aoai(    key_file_path=".",    openai_api_key_file="key_openai.txt",    aoai_api_key_file="key_aoai.txt",    aoai_api_base_file="base_aoai.txt",    exclude=None,  # The API type to exclude, eg: "openai" or "aoai".)
```

## config_list_from_json[​](https://microsoft.github.io/autogen/docs/llm_endpoint_configuration#config_list_from_json "Direct link to config_list_from_json")

This method loads configurations from an environment variable or a JSON file. It provides flexibility by allowing users to filter configurations based on certain criteria.

Steps: - Setup the JSON Configuration: 1. Store configurations in an environment variable named `OAI_CONFIG_LIST` as a valid JSON string. 2. Alternatively, save configurations in a local JSON file named `OAI_CONFIG_LIST.json` 3. Add `OAI_CONFIG_LIST` to your `.gitignore` file on your local repository.

Your JSON structure should look something like this:

```
# OAI_CONFIG_LIST file example[    {        "model": "gpt-4",        "api_key": "YOUR_OPENAI_API_KEY"    },    {        "model": "gpt-3.5-turbo",        "api_key": "YOUR_OPENAI_API_KEY",        "api_version": "2023-03-01-preview"    }]
```

```
config_list = autogen.config_list_from_json(    env_or_file="OAI_CONFIG_LIST",  # or OAI_CONFIG_LIST.json if file extension is added    filter_dict={        "model": {            "gpt-4",            "gpt-3.5-turbo",        }    },)
```

#### What is `filter_dict`?[​](https://microsoft.github.io/autogen/docs/llm_endpoint_configuration#what-is-filter_dict "Direct link to what-is-filter_dict")

The z parameter in `autogen.config_list_from_json` function is used to selectively filter the configurations loaded from the environment variable or JSON file based on specified criteria. It allows you to define criteria to select only those configurations that match the defined conditions.

let’s say you want to configure an assistant agent to only LLM type. Take the below example: even though we have “gpt-3.5-turbo” and “gpt-4” in our `OAI_CONFIG_LIST`, this agent would only be configured to use

```
cheap_config_list = autogen.config_list_from_json(    env_or_file="OAI_CONFIG_LIST",    filter_dict={        "model": {            "gpt-3.5-turbo",        }    },)costly_config_list = autogen.config_list_from_json(    env_or_file="OAI_CONFIG_LIST",    filter_dict={        "model": {            "gpt-4",        }    },)# Assistant using GPT 3.5 Turboassistant_one = autogen.AssistantAgent(    name="3.5-assistant",    llm_config={        "timeout": 600,        "cache_seed": 42,        "config_list": cheap_config_list,        "temperature": 0,    },)# Assistant using GPT 4assistant_two = autogen.AssistantAgent(    name="4-assistant",    llm_config={        "timeout": 600,        "cache_seed": 42,        "config_list": costly_config_list,        "temperature": 0,    },)
```

With the `OAI_CONFIG_LIST` we set earlier, there isn’t much to filter on. But when the complexity of a project grows and you’re managing multiple models for various purposes, you can see how `filter_dict` can be useful.

A more complex filtering criteria could be the following: Assuming we have an `OAI_CONFIG_LIST` several models used to create various agents - Let’s say we want to load configurations for `gpt-4` using API version `"2023-03-01-preview"` and we want the `api_type` to be `aoai`, we can set up `filter_dict` as follows:

```
config_list = autogen.config_list_from_json(    env_or_file="OAI_CONFIG_LIST",    filter_dict={"model": {"gpt-4"}, "api_version": {"2023-03-01-preview"}, "api_type": ["aoai"]},)
```

## config_list_from_models[​](https://microsoft.github.io/autogen/docs/llm_endpoint_configuration#config_list_from_models "Direct link to config_list_from_models")

This method creates configurations based on a provided list of models. It’s useful when you have specific models in mind and don’t want to manually specify each configuration. The [`config_list_from_models`](https://microsoft.github.io/autogen/docs/reference/oai/openai_utils#config_list_from_models) function tries to create a list of configurations using Azure OpenAI endpoints and OpenAI endpoints for the provided list of models. It assumes the api keys and api bases are stored in the corresponding environment variables or local txt files. It’s okay to only have the OpenAI API key, OR only the Azure OpenAI API key + base. For Azure the model name refers to the OpenAI Studio deployment name.

Steps: - Similar to method 1, store API keys and bases either in environment variables or `.txt` files.

```
config_list = autogen.config_list_from_models(    key_file_path=".",    openai_api_key_file="key_openai.txt",    aoai_api_key_file="key_aoai.txt",    aoai_api_base_file="base_aoai.txt",    exclude="aoai",    model_list=["gpt-4", "gpt-3.5-turbo", "gpt-3.5-turbo-16k"],)
```

## config_list_from_dotenv[​](https://microsoft.github.io/autogen/docs/llm_endpoint_configuration#config_list_from_dotenv "Direct link to config_list_from_dotenv")

If you are interested in keeping all of your keys in a single location like a `.env` file rather than using a configuration specifically for OpenAI, you can use `config_list_from_dotenv`. This allows you to conveniently create a config list without creating a complex `OAI_CONFIG_LIST` file.

The `model_api_key_map` parameter is a dictionary that maps model names to the environment variable names in the `.env` file where their respective API keys are stored. It lets the code know which API key to use for each model.

If not provided, it defaults to using `OPENAI_API_KEY` for `gpt-4` and `OPENAI_API_KEY` for `gpt-3.5-turbo`.

```
    # default key map    model_api_key_map = {        "gpt-4": "OPENAI_API_KEY",        "gpt-3.5-turbo": "OPENAI_API_KEY",    }
```

Here is an example `.env` file:

```
OPENAI_API_KEY=sk-*********************HUGGING_FACE_API_KEY=**************************ANOTHER_API_KEY=1234567890234567890
```

```
config_list = autogen.config_list_from_dotenv(    dotenv_file_path=".env",  # If None the function will try to find in the working directory    filter_dict={        "model": {            "gpt-4",            "gpt-3.5-turbo",        }    },)config_list
```

```
[{'api_key': 'sk-*********************', 'model': 'gpt-4'}, {'api_key': 'sk-*********************', 'model': 'gpt-3.5-turbo'}]
```

```
# gpt-3.5-turbo will default to OPENAI_API_KEYconfig_list = autogen.config_list_from_dotenv(    dotenv_file_path=".env",  # If None the function will try to find in the working directory    model_api_key_map={        "gpt-4": "ANOTHER_API_KEY",  # String or dict accepted    },    filter_dict={        "model": {            "gpt-4",            "gpt-3.5-turbo",        }    },)config_list
```

```
[{'api_key': '1234567890234567890', 'model': 'gpt-4'}, {'api_key': 'sk-*********************', 'model': 'gpt-3.5-turbo'}]
```

```
# example using different environment variable namesconfig_list = autogen.config_list_from_dotenv(    dotenv_file_path=".env",    model_api_key_map={        "gpt-4": "OPENAI_API_KEY",        "vicuna": "HUGGING_FACE_API_KEY",    },    filter_dict={        "model": {            "gpt-4",            "vicuna",        }    },)config_list
```

```
[{'api_key': 'sk-*********************', 'model': 'gpt-4'}, {'api_key': '**************************', 'model': 'vicuna'}]
```

You can also provide additional configurations for APIs, simply by replacing the string value with a dictionary expanding on the configurations. See the example below showing the example of using `gpt-4` on `openai` by default, and using `gpt-3.5-turbo` with additional configurations for `aoai`.

```
config_list = autogen.config_list_from_dotenv(    dotenv_file_path=".env",    model_api_key_map={        "gpt-4": "OPENAI_API_KEY",        "gpt-3.5-turbo": {            "api_key_env_var": "ANOTHER_API_KEY",            "api_type": "aoai",            "api_version": "v2",            "base_url": "https://api.someotherapi.com",        },    },    filter_dict={        "model": {            "gpt-4",            "gpt-3.5-turbo",        }    },)config_list
```

```
[{'api_key': 'sk-*********************', 'model': 'gpt-4'}, {'api_key': '1234567890234567890',  'base_url': 'https://api.someotherapi.com',  'api_type': 'aoai',  'api_version': 'v2',  'model': 'gpt-3.5-turbo'}]
```

---

