# Multi-agent Conversation Framework

AutoGen offers a unified multi-agent conversation framework as a high-level abstraction of using foundation models. It features capable, customizable and conversable agents which integrate LLMs, tools, and humans via automated agent chat. By automating chat among multiple capable agents, one can easily make them collectively perform tasks autonomously or with human feedback, including tasks that require using tools via code.

This framework simplifies the orchestration, automation and optimization of a complex LLM workflow. It maximizes the performance of LLM models and overcome their weaknesses. It enables building next-gen LLM applications based on multi-agent conversations with minimal effort.

### Agents[​](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#agents "Direct link to Agents")

AutoGen abstracts and implements conversable agents designed to solve tasks through inter-agent conversations. Specifically, the agents in AutoGen have the following notable features:

- Conversable: Agents in AutoGen are conversable, which means that any agent can send and receive messages from other agents to initiate or continue a conversation
    
- Customizable: Agents in AutoGen can be customized to integrate LLMs, humans, tools, or a combination of them.
    

The figure below shows the built-in agents in AutoGen. ![Agent Chat Example](https://microsoft.github.io/autogen/assets/images/autogen_agents-b80434bcb15d46da0c6cbeed28115f38.png)

We have designed a generic [`ConversableAgent`](https://microsoft.github.io/autogen/docs/reference/agentchat/conversable_agent#conversableagent-objects) class for Agents that are capable of conversing with each other through the exchange of messages to jointly finish a task. An agent can communicate with other agents and perform actions. Different agents can differ in what actions they perform after receiving messages. Two representative subclasses are [`AssistantAgent`](https://microsoft.github.io/autogen/docs/reference/agentchat/assistant_agent#assistantagent-objects) and [`UserProxyAgent`](https://microsoft.github.io/autogen/docs/reference/agentchat/user_proxy_agent#userproxyagent-objects)

- The [`AssistantAgent`](https://microsoft.github.io/autogen/docs/reference/agentchat/assistant_agent#assistantagent-objects) is designed to act as an AI assistant, using LLMs by default but not requiring human input or code execution. It could write Python code (in a Python coding block) for a user to execute when a message (typically a description of a task that needs to be solved) is received. Under the hood, the Python code is written by LLM (e.g., GPT-4). It can also receive the execution results and suggest corrections or bug fixes. Its behavior can be altered by passing a new system message. The LLM [inference](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#enhanced-inference) configuration can be configured via [`llm_config`].
    
- The [`UserProxyAgent`](https://microsoft.github.io/autogen/docs/reference/agentchat/user_proxy_agent#userproxyagent-objects) is conceptually a proxy agent for humans, soliciting human input as the agent's reply at each interaction turn by default and also having the capability to execute code and call functions or tools. The [`UserProxyAgent`](https://microsoft.github.io/autogen/docs/reference/agentchat/user_proxy_agent#userproxyagent-objects) triggers code execution automatically when it detects an executable code block in the received message and no human user input is provided. Code execution can be disabled by setting the `code_execution_config` parameter to False. LLM-based response is disabled by default. It can be enabled by setting `llm_config` to a dict corresponding to the [inference](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference) configuration. When `llm_config` is set as a dictionary, [`UserProxyAgent`](https://microsoft.github.io/autogen/docs/reference/agentchat/user_proxy_agent#userproxyagent-objects) can generate replies using an LLM when code execution is not performed.
    

The auto-reply capability of [`ConversableAgent`](https://microsoft.github.io/autogen/docs/reference/agentchat/conversable_agent#conversableagent-objects) allows for more autonomous multi-agent communication while retaining the possibility of human intervention. One can also easily extend it by registering reply functions with the [`register_reply()`](https://microsoft.github.io/autogen/docs/reference/agentchat/conversable_agent#register_reply) method.

In the following code, we create an [`AssistantAgent`](https://microsoft.github.io/autogen/docs/reference/agentchat/assistant_agent#assistantagent-objects) named "assistant" to serve as the assistant and a [`UserProxyAgent`](https://microsoft.github.io/autogen/docs/reference/agentchat/user_proxy_agent#userproxyagent-objects) named "user_proxy" to serve as a proxy for the human user. We will later employ these two agents to solve a task.

```
from autogen import AssistantAgent, UserProxyAgent# create an AssistantAgent instance named "assistant"assistant = AssistantAgent(name="assistant")# create a UserProxyAgent instance named "user_proxy"user_proxy = UserProxyAgent(name="user_proxy")
```

#### Tool calling[​](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#tool-calling "Direct link to Tool calling")

Tool calling enables agents to interact with external tools and APIs more efficiently. This feature allows the AI model to intelligently choose to output a JSON object containing arguments to call specific tools based on the user's input. A tool to be called is specified with a JSON schema describing its parameters and their types. Writing such JSON schema is complex and error-prone and that is why AutoGen framework provides two high level function decorators for automatically generating such schema using type hints on standard Python datatypes or Pydantic models:

1. [`ConversableAgent.register_for_llm`](https://microsoft.github.io/autogen/docs/reference/agentchat/conversable_agent#register_for_llm) is used to register the function as a Tool in the `llm_config` of a ConversableAgent. The ConversableAgent agent can propose execution of a registered Tool, but the actual execution will be performed by a UserProxy agent.
    
2. [`ConversableAgent.register_for_execution`](https://microsoft.github.io/autogen/docs/reference/agentchat/conversable_agent#register_for_execution) is used to register the function in the `function_map` of a UserProxy agent.
    

The following examples illustrates the process of registering a custom function for currency exchange calculation that uses type hints and standard Python datatypes:

1. First, we import necessary libraries and configure models using [`autogen.config_list_from_json`](https://microsoft.github.io/autogen/docs/FAQ#set-your-api-endpoints) function:

```
from typing import Literalfrom pydantic import BaseModel, Fieldfrom typing_extensions import Annotatedimport autogenconfig_list = autogen.config_list_from_json(    "OAI_CONFIG_LIST",    filter_dict={        "model": ["gpt-4", "gpt-3.5-turbo", "gpt-3.5-turbo-16k"],    },)
```

2. We create an assistant agent and user proxy. The assistant will be responsible for suggesting which functions to call and the user proxy for the actual execution of a proposed function:

```
llm_config = {    "config_list": config_list,    "timeout": 120,}chatbot = autogen.AssistantAgent(    name="chatbot",    system_message="For currency exchange tasks, only use the functions you have been provided with. Reply TERMINATE when the task is done.",    llm_config=llm_config,)# create a UserProxyAgent instance named "user_proxy"user_proxy = autogen.UserProxyAgent(    name="user_proxy",    is_termination_msg=lambda x: x.get("content", "") and x.get("content", "").rstrip().endswith("TERMINATE"),    human_input_mode="NEVER",    max_consecutive_auto_reply=10,)
```

3. We define the function `currency_calculator` below as follows and decorate it with two decorators:
    - [`@user_proxy.register_for_execution()`](https://microsoft.github.io/autogen/docs/reference/agentchat/conversable_agent#register_for_execution) adding the function `currency_calculator` to `user_proxy.function_map`, and
    - [`@chatbot.register_for_llm`](https://microsoft.github.io/autogen/docs/reference/agentchat/conversable_agent#register_for_llm) adding a generated JSON schema of the function to `llm_config` of `chatbot`.

```
CurrencySymbol = Literal["USD", "EUR"]def exchange_rate(base_currency: CurrencySymbol, quote_currency: CurrencySymbol) -> float:    if base_currency == quote_currency:        return 1.0    elif base_currency == "USD" and quote_currency == "EUR":        return 1 / 1.1    elif base_currency == "EUR" and quote_currency == "USD":        return 1.1    else:        raise ValueError(f"Unknown currencies {base_currency}, {quote_currency}")# NOTE: for Azure OpenAI, please use API version 2023-12-01-preview or later as# support for earlier versions will be deprecated.# For API versions 2023-10-01-preview or earlier you may# need to set `api_style="function"` in the decorator if the default value does not work:# `register_for_llm(description=..., api_style="function")`.@user_proxy.register_for_execution()@chatbot.register_for_llm(description="Currency exchange calculator.")def currency_calculator(    base_amount: Annotated[float, "Amount of currency in base_currency"],    base_currency: Annotated[CurrencySymbol, "Base currency"] = "USD",    quote_currency: Annotated[CurrencySymbol, "Quote currency"] = "EUR",) -> str:    quote_amount = exchange_rate(base_currency, quote_currency) * base_amount    return f"{quote_amount} {quote_currency}"
```

Notice the use of [Annotated](https://docs.python.org/3/library/typing.html?highlight=annotated#typing.Annotated) to specify the type and the description of each parameter. The return value of the function must be either string or serializable to string using the [`json.dumps()`](https://docs.python.org/3/library/json.html#json.dumps) or [`Pydantic` model dump to JSON](https://docs.pydantic.dev/latest/concepts/serialization/#modelmodel_dump_json) (both version 1.x and 2.x are supported).

You can check the JSON schema generated by the decorator `chatbot.llm_config["tools"]`:

```
[{'type': 'function', 'function': {'description': 'Currency exchange calculator.',  'name': 'currency_calculator',  'parameters': {'type': 'object',   'properties': {'base_amount': {'type': 'number',     'description': 'Amount of currency in base_currency'},    'base_currency': {'enum': ['USD', 'EUR'],     'type': 'string',     'default': 'USD',     'description': 'Base currency'},    'quote_currency': {'enum': ['USD', 'EUR'],     'type': 'string',     'default': 'EUR',     'description': 'Quote currency'}},   'required': ['base_amount']}}}]
```

Python decorators are functions themselves. If you do not want to use the `@chatbot.register...` decorator syntax, you can call the decorators as functions:

```
# Register the function with the chatbot's llm_config.currency_calculator = chatbot.register_for_llm(description="Currency exchange calculator.")(currency_calculator)# Register the function with the user_proxy's function_map.user_proxy.register_for_execution()(currency_calculator)
```

Alternatevely, you can also use `autogen.agentchat.register_function()` instead as follows:

```
def currency_calculator(    base_amount: Annotated[float, "Amount of currency in base_currency"],    base_currency: Annotated[CurrencySymbol, "Base currency"] = "USD",    quote_currency: Annotated[CurrencySymbol, "Quote currency"] = "EUR",) -> str:    quote_amount = exchange_rate(base_currency, quote_currency) * base_amount    return f"{quote_amount} {quote_currency}"autogen.agentchat.register_function(    currency_calculator,    agent=chatbot,    executor=user_proxy,    description="Currency exchange calculator.",)
```

4. Agents can now use the function as follows:

```
user_proxy.initiate_chat(    chatbot,    message="How much is 123.45 USD in EUR?",)
```

Output:

```
user_proxy (to chatbot):How much is 123.45 USD in EUR?--------------------------------------------------------------------------------chatbot (to user_proxy):***** Suggested tool Call: currency_calculator *****Arguments:{"base_amount":123.45,"base_currency":"USD","quote_currency":"EUR"}********************************************************-------------------------------------------------------------------------------->>>>>>>> EXECUTING FUNCTION currency_calculator...user_proxy (to chatbot):***** Response from calling function "currency_calculator" *****112.22727272727272 EUR****************************************************************--------------------------------------------------------------------------------chatbot (to user_proxy):123.45 USD is equivalent to approximately 112.23 EUR....TERMINATE
```

Use of Pydantic models further simplifies writing of such functions. Pydantic models can be used for both the parameters of a function and for its return type. Parameters of such functions will be constructed from JSON provided by an AI model, while the output will be serialized as JSON encoded string automatically.

The following example shows how we could rewrite our currency exchange calculator example:

```
# defines a Pydantic modelclass Currency(BaseModel):  # parameter of type CurrencySymbol  currency: Annotated[CurrencySymbol, Field(..., description="Currency symbol")]  # parameter of type float, must be greater or equal to 0 with default value 0  amount: Annotated[float, Field(0, description="Amount of currency", ge=0)]def currency_calculator(  base: Annotated[Currency, "Base currency: amount and currency symbol"],  quote_currency: Annotated[CurrencySymbol, "Quote currency symbol"] = "USD",) -> Currency:  quote_amount = exchange_rate(base.currency, quote_currency) * base.amount  return Currency(amount=quote_amount, currency=quote_currency)autogen.agentchat.register_function(    currency_calculator,    agent=chatbot,    executor=user_proxy,    description="Currency exchange calculator.",)
```

The generated JSON schema has additional properties such as minimum value encoded:

```
[{'type': 'function', 'function': {'description': 'Currency exchange calculator.',  'name': 'currency_calculator',  'parameters': {'type': 'object',   'properties': {'base': {'properties': {'currency': {'description': 'Currency symbol',       'enum': ['USD', 'EUR'],       'title': 'Currency',       'type': 'string'},      'amount': {'default': 0,       'description': 'Amount of currency',       'minimum': 0.0,       'title': 'Amount',       'type': 'number'}},     'required': ['currency'],     'title': 'Currency',     'type': 'object',     'description': 'Base currency: amount and currency symbol'},    'quote_currency': {'enum': ['USD', 'EUR'],     'type': 'string',     'default': 'USD',     'description': 'Quote currency symbol'}},   'required': ['base']}}}]
```

For more in-depth examples, please check the following:

- Currency calculator examples - [View Notebook](https://github.com/microsoft/autogen/blob/main/notebook/agentchat_function_call_currency_calculator.ipynb)
    
- Use Provided Tools as Functions - [View Notebook](https://github.com/microsoft/autogen/blob/main/notebook/agentchat_function_call.ipynb)
    
- Use Tools via Sync and Async Function Calling - [View Notebook](https://github.com/microsoft/autogen/blob/main/notebook/agentchat_function_call_async.ipynb)
    

## Multi-agent Conversations[​](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#multi-agent-conversations "Direct link to Multi-agent Conversations")

### A Basic Two-Agent Conversation Example[​](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#a-basic-two-agent-conversation-example "Direct link to A Basic Two-Agent Conversation Example")

Once the participating agents are constructed properly, one can start a multi-agent conversation session by an initialization step as shown in the following code:

```
# the assistant receives a message from the user, which contains the task descriptionuser_proxy.initiate_chat(    assistant,    message="""What date is today? Which big tech stock has the largest year-to-date gain this year? How much is the gain?""",)
```

After the initialization step, the conversation could proceed automatically. Find a visual illustration of how the user_proxy and assistant collaboratively solve the above task autonomously below: ![Agent Chat Example](https://microsoft.github.io/autogen/assets/images/agent_example-a965f253ce7d8e1548ff819e19edc5e4.png)

1. The assistant receives a message from the user_proxy, which contains the task description.
2. The assistant then tries to write Python code to solve the task and sends the response to the user_proxy.
3. Once the user_proxy receives a response from the assistant, it tries to reply by either soliciting human input or preparing an automatically generated reply. If no human input is provided, the user_proxy executes the code and uses the result as the auto-reply.
4. The assistant then generates a further response for the user_proxy. The user_proxy can then decide whether to terminate the conversation. If not, steps 3 and 4 are repeated.

### Supporting Diverse Conversation Patterns[​](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#supporting-diverse-conversation-patterns "Direct link to Supporting Diverse Conversation Patterns")

#### Conversations with different levels of autonomy, and human-involvement patterns[​](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#conversations-with-different-levels-of-autonomy-and-human-involvement-patterns "Direct link to Conversations with different levels of autonomy, and human-involvement patterns")

On the one hand, one can achieve fully autonomous conversations after an initialization step. On the other hand, AutoGen can be used to implement human-in-the-loop problem-solving by configuring human involvement levels and patterns (e.g., setting the `human_input_mode` to `ALWAYS`), as human involvement is expected and/or desired in many applications.

#### Static and dynamic conversations[​](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#static-and-dynamic-conversations "Direct link to Static and dynamic conversations")

By adopting the conversation-driven control with both programming language and natural language, AutoGen inherently allows dynamic conversation. Dynamic conversation allows the agent topology to change depending on the actual flow of conversation under different input problem instances, while the flow of a static conversation always follows a pre-defined topology. The dynamic conversation pattern is useful in complex applications where the patterns of interaction cannot be predetermined in advance. AutoGen provides two general approaches to achieving dynamic conversation:

- Registered auto-reply. With the pluggable auto-reply function, one can choose to invoke conversations with other agents depending on the content of the current message and context. A working system demonstrating this type of dynamic conversation can be found in this code example, demonstrating a [dynamic group chat](https://github.com/microsoft/autogen/blob/main/notebook/agentchat_groupchat.ipynb). In the system, we register an auto-reply function in the group chat manager, which lets LLM decide who the next speaker will be in a group chat setting.
    
- LLM-based function call. In this approach, LLM decides whether or not to call a particular function depending on the conversation status in each inference call. By messaging additional agents in the called functions, the LLM can drive dynamic multi-agent conversation. A working system showcasing this type of dynamic conversation can be found in the [multi-user math problem solving scenario](https://github.com/microsoft/autogen/blob/main/notebook/agentchat_two_users.ipynb), where a student assistant would automatically resort to an expert using function calls.
    

### LLM Caching[​](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#llm-caching "Direct link to LLM Caching")

Since version 0.2.8, a configurable context manager allows you to easily configure LLM cache, using either DiskCache or Redis. All agents inside the context manager will use the same cache.

```
from autogen import Cache# Use Redis as cachewith Cache.redis(redis_url="redis://localhost:6379/0") as cache:    user.initiate_chat(assistant, message=coding_task, cache=cache)# Use DiskCache as cachewith Cache.disk() as cache:    user.initiate_chat(assistant, message=coding_task, cache=cache)
```

You can vary the `cache_seed` parameter to get different LLM output while still using cache.

```
# Setting the cache_seed to 1 will use a different cache from the default one# and you will see different output.with Cache.disk(cache_seed=1) as cache:    user.initiate_chat(assistant, message=coding_task, cache=cache)
```

By default DiskCache uses `.cache` for storage. To change the cache directory, set `cache_path_root`:

```
with Cache.disk(cache_path_root="/tmp/autogen_cache") as cache:    user.initiate_chat(assistant, message=coding_task, cache=cache)
```

For backward compatibility, DiskCache is on by default with `cache_seed` set to 41. To disable caching completely, set `cache_seed` to `None` in the `llm_config` of the agent.

```
assistant = AssistantAgent(    "coding_agent",    llm_config={        "cache_seed": None,        "config_list": OAI_CONFIG_LIST,        "max_tokens": 1024,    },)
```

### Diverse Applications Implemented with AutoGen[​](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#diverse-applications-implemented-with-autogen "Direct link to Diverse Applications Implemented with AutoGen")

The figure below shows six examples of applications built using AutoGen. ![Applications](https://microsoft.github.io/autogen/assets/images/app-c414cd164ef912e5e8b40f61042143ad.png)

Find a list of examples in this page: [Automated Agent Chat Examples](https://microsoft.github.io/autogen/docs/Examples#automated-multi-agent-chat)

## For Further Reading[​](https://microsoft.github.io/autogen/docs/Use-Cases/agent_chat#for-further-reading "Direct link to For Further Reading")

_Interested in the research that leads to this package? Please check the following papers._

- [AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation Framework](https://arxiv.org/abs/2308.08155). Qingyun Wu, Gagan Bansal, Jieyu Zhang, Yiran Wu, Shaokun Zhang, Erkang Zhu, Beibin Li, Li Jiang, Xiaoyun Zhang and Chi Wang. ArXiv 2023.
    
- [An Empirical Study on Challenging Math Problem Solving with GPT-4](https://arxiv.org/abs/2306.01337). Yiran Wu, Feiran Jia, Shaokun Zhang, Hangyu Li, Erkang Zhu, Yue Wang, Yin Tat Lee, Richard Peng, Qingyun Wu, Chi Wang. ArXiv preprint arXiv:2306.01337 (2023).

---

# Enhanced Inference

`autogen.OpenAIWrapper` provides enhanced LLM inference for `openai>=1`. `autogen.Completion` is a drop-in replacement of `openai.Completion` and `openai.ChatCompletion` for enhanced LLM inference using `openai<1`. There are a number of benefits of using `autogen` to perform inference: performance tuning, API unification, caching, error handling, multi-config inference, result filtering, templating and so on.

## Tune Inference Parameters (for openai<1)[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#tune-inference-parameters-for-openai1 "Direct link to Tune Inference Parameters (for openai<1)")

Find a list of examples in this page: [Tune Inference Parameters Examples](https://microsoft.github.io/autogen/docs/Examples#inference-hyperparameters-tuning)

### Choices to optimize[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#choices-to-optimize "Direct link to Choices to optimize")

The cost of using foundation models for text generation is typically measured in terms of the number of tokens in the input and output combined. From the perspective of an application builder using foundation models, the use case is to maximize the utility of the generated text under an inference budget constraint (e.g., measured by the average dollar cost needed to solve a coding problem). This can be achieved by optimizing the hyperparameters of the inference, which can significantly affect both the utility and the cost of the generated text.

The tunable hyperparameters include:

1. model - this is a required input, specifying the model ID to use.
2. prompt/messages - the input prompt/messages to the model, which provides the context for the text generation task.
3. max_tokens - the maximum number of tokens (words or word pieces) to generate in the output.
4. temperature - a value between 0 and 1 that controls the randomness of the generated text. A higher temperature will result in more random and diverse text, while a lower temperature will result in more predictable text.
5. top_p - a value between 0 and 1 that controls the sampling probability mass for each token generation. A lower top_p value will make it more likely to generate text based on the most likely tokens, while a higher value will allow the model to explore a wider range of possible tokens.
6. n - the number of responses to generate for a given prompt. Generating multiple responses can provide more diverse and potentially more useful output, but it also increases the cost of the request.
7. stop - a list of strings that, when encountered in the generated text, will cause the generation to stop. This can be used to control the length or the validity of the output.
8. presence_penalty, frequency_penalty - values that control the relative importance of the presence and frequency of certain words or phrases in the generated text.
9. best_of - the number of responses to generate server-side when selecting the "best" (the one with the highest log probability per token) response for a given prompt.

The cost and utility of text generation are intertwined with the joint effect of these hyperparameters. There are also complex interactions among subsets of the hyperparameters. For example, the temperature and top_p are not recommended to be altered from their default values together because they both control the randomness of the generated text, and changing both at the same time can result in conflicting effects; n and best_of are rarely tuned together because if the application can process multiple outputs, filtering on the server side causes unnecessary information loss; both n and max_tokens will affect the total number of tokens generated, which in turn will affect the cost of the request. These interactions and trade-offs make it difficult to manually determine the optimal hyperparameter settings for a given text generation task.

_Do the choices matter? Check this [blogpost](https://microsoft.github.io/autogen/blog/2023/04/21/LLM-tuning-math) to find example tuning results about gpt-3.5-turbo and gpt-4._

With AutoGen, the tuning can be performed with the following information:

1. Validation data.
2. Evaluation function.
3. Metric to optimize.
4. Search space.
5. Budgets: inference and optimization respectively.

### Validation data[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#validation-data "Direct link to Validation data")

Collect a diverse set of instances. They can be stored in an iterable of dicts. For example, each instance dict can contain "problem" as a key and the description str of a math problem as the value; and "solution" as a key and the solution str as the value.

### Evaluation function[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#evaluation-function "Direct link to Evaluation function")

The evaluation function should take a list of responses, and other keyword arguments corresponding to the keys in each validation data instance as input, and output a dict of metrics. For example,

```
def eval_math_responses(responses: List[str], solution: str, **args) -> Dict:    # select a response from the list of responses    answer = voted_answer(responses)    # check whether the answer is correct    return {"success": is_equivalent(answer, solution)}
```

`autogen.code_utils` and `autogen.math_utils` offer some example evaluation functions for code generation and math problem solving.

### Metric to optimize[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#metric-to-optimize "Direct link to Metric to optimize")

The metric to optimize is usually an aggregated metric over all the tuning data instances. For example, users can specify "success" as the metric and "max" as the optimization mode. By default, the aggregation function is taking the average. Users can provide a customized aggregation function if needed.

### Search space[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#search-space "Direct link to Search space")

Users can specify the (optional) search range for each hyperparameter.

1. model. Either a constant str, or multiple choices specified by `flaml.tune.choice`.
2. prompt/messages. Prompt is either a str or a list of strs, of the prompt templates. messages is a list of dicts or a list of lists, of the message templates. Each prompt/message template will be formatted with each data instance. For example, the prompt template can be: "{problem} Solve the problem carefully. Simplify your answer as much as possible. Put the final answer in \boxed{{}}." And `{problem}` will be replaced by the "problem" field of each data instance.
3. max_tokens, n, best_of. They can be constants, or specified by `flaml.tune.randint`, `flaml.tune.qrandint`, `flaml.tune.lograndint` or `flaml.qlograndint`. By default, max_tokens is searched in [50, 1000); n is searched in [1, 100); and best_of is fixed to 1.
4. stop. It can be a str or a list of strs, or a list of lists of strs or None. Default is None.
5. temperature or top_p. One of them can be specified as a constant or by `flaml.tune.uniform` or `flaml.tune.loguniform` etc. Please don't provide both. By default, each configuration will choose either a temperature or a top_p in [0, 1] uniformly.
6. presence_penalty, frequency_penalty. They can be constants or specified by `flaml.tune.uniform` etc. Not tuned by default.

### Budgets[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#budgets "Direct link to Budgets")

One can specify an inference budget and an optimization budget. The inference budget refers to the average inference cost per data instance. The optimization budget refers to the total budget allowed in the tuning process. Both are measured by dollars and follow the price per 1000 tokens.

### Perform tuning[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#perform-tuning "Direct link to Perform tuning")

Now, you can use `autogen.Completion.tune` for tuning. For example,

```
import autogenconfig, analysis = autogen.Completion.tune(    data=tune_data,    metric="success",    mode="max",    eval_func=eval_func,    inference_budget=0.05,    optimization_budget=3,    num_samples=-1,)
```

`num_samples` is the number of configurations to sample. -1 means unlimited (until optimization budget is exhausted). The returned `config` contains the optimized configuration and `analysis` contains an ExperimentAnalysis object for all the tried configurations and results.

The tuned config can be used to perform inference.

## API unification[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#api-unification "Direct link to API unification")

`autogen.OpenAIWrapper.create()` can be used to create completions for both chat and non-chat models, and both OpenAI API and Azure OpenAI API.

```
from autogen import OpenAIWrapper# OpenAI endpointclient = OpenAIWrapper()# ChatCompletionresponse = client.create(messages=[{"role": "user", "content": "2+2="}], model="gpt-3.5-turbo")# extract the response textprint(client.extract_text_or_completion_object(response))# get cost of this completionprint(response.cost)# Azure OpenAI endpointclient = OpenAIWrapper(api_key=..., base_url=..., api_version=..., api_type="azure")# Completionresponse = client.create(prompt="2+2=", model="gpt-3.5-turbo-instruct")# extract the response textprint(client.extract_text_or_completion_object(response))
```

For local LLMs, one can spin up an endpoint using a package like [FastChat](https://github.com/lm-sys/FastChat), and then use the same API to send a request. See [here](https://microsoft.github.io/autogen/blog/2023/07/14/Local-LLMs) for examples on how to make inference with local LLMs.

## Usage Summary[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#usage-summary "Direct link to Usage Summary")

The `OpenAIWrapper` from `autogen` tracks token counts and costs of your API calls. Use the `create()` method to initiate requests and `print_usage_summary()` to retrieve a detailed usage report, including total cost and token usage for both cached and actual requests.

- `mode=["actual", "total"]` (default): print usage summary for all completions and non-caching completions.
- `mode='actual'`: only print non-cached usage.
- `mode='total'`: only print all usage (including cache).

Reset your session's usage data with `clear_usage_summary()` when needed. [View Notebook](https://github.com/microsoft/autogen/blob/main/notebook/oai_client_cost.ipynb)

Example usage:

```
from autogen import OpenAIWrapperclient = OpenAIWrapper()client.create(messages=[{"role": "user", "content": "Python learning tips."}], model="gpt-3.5-turbo")client.print_usage_summary()  # Display usageclient.clear_usage_summary()  # Reset usage data
```

Sample output:

```
Usage summary excluding cached usage:Total cost: 0.00015* Model 'gpt-3.5-turbo': cost: 0.00015, prompt_tokens: 25, completion_tokens: 58, total_tokens: 83Usage summary including cached usage:Total cost: 0.00027* Model 'gpt-3.5-turbo': cost: 0.00027, prompt_tokens: 50, completion_tokens: 100, total_tokens: 150
```

## Caching[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#caching "Direct link to Caching")

API call results are cached locally and reused when the same request is issued. This is useful when repeating or continuing experiments for reproducibility and cost saving.

Starting version 0.2.8, a configurable context manager allows you to easily configure the cache, using either DiskCache or Redis. All `OpenAIWrapper` created inside the context manager can use the same cache through the constructor.

```
from autogen import Cachewith Cache.redis(redis_url="redis://localhost:6379/0") as cache:    client = OpenAIWrapper(..., cache=cache)    client.create(...)with Cache.disk() as cache:    client = OpenAIWrapper(..., cache=cache)    client.create(...)
```

You can also set a cache directly in the `create()` method.

```
client = OpenAIWrapper(...)with Cache.disk() as cache:    client.create(..., cache=cache)
```

You can vary the `cache_seed` parameter to get different LLM output while still using cache.

```
# Setting the cache_seed to 1 will use a different cache from the default one# and you will see different output.with Cache.disk(cache_seed=1) as cache:    client.create(..., cache=cache)
```

By default DiskCache uses `.cache` for storage. To change the cache directory, set `cache_path_root`:

```
with Cache.disk(cache_path_root="/tmp/autogen_cache") as cache:    client.create(..., cache=cache)
```

### Turnning off cache[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#turnning-off-cache "Direct link to Turnning off cache")

For backward compatibility, DiskCache is always enabled by default with `cache_seed` set to 41. To fully disable it, set `cache_seed` to None.

```
# Turn off cache in constructor,client = OpenAIWrapper(..., cache_seed=None)# or directly in create().client.create(..., cache_seed=None)
```

### Difference between `cache_seed` and openai's `seed` parameter[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#difference-between-cache_seed-and-openais-seed-parameter "Direct link to difference-between-cache_seed-and-openais-seed-parameter")

openai v1.1 introduces a new param `seed`. The differences between autogen's `cache_seed` and openai's `seed`: - autogen uses local disk cache to guarantee the exactly same output is produced for the same input and when cache is hit, no openai api call will be made. - openai's `seed` is a best-effort deterministic sampling with no guarantee of determinism. When using openai's `seed` with `cache_seed` set to None, even for the same input, an openai api call will be made and there is no guarantee for getting exactly the same output.

## Error handling[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#error-handling "Direct link to Error handling")

### Runtime error[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#runtime-error "Direct link to Runtime error")

One can pass a list of configurations of different models/endpoints to mitigate the rate limits and other runtime error. For example,

```
client = OpenAIWrapper(    config_list=[        {            "model": "gpt-4",            "api_key": os.environ.get("AZURE_OPENAI_API_KEY"),            "api_type": "azure",            "base_url": os.environ.get("AZURE_OPENAI_API_BASE"),            "api_version": "2023-08-01-preview",        },        {            "model": "gpt-3.5-turbo",            "api_key": os.environ.get("OPENAI_API_KEY"),            "base_url": "https://api.openai.com/v1",        },        {            "model": "llama2-chat-7B",            "base_url": "http://127.0.0.1:8080",        }    ],)
```

`client.create()` will try querying Azure OpenAI gpt-4, OpenAI gpt-3.5-turbo, and a locally hosted llama2-chat-7B one by one, until a valid result is returned. This can speed up the development process where the rate limit is a bottleneck. An error will be raised if the last choice fails. So make sure the last choice in the list has the best availability.

For convenience, we provide a number of utility functions to load config lists.

- `get_config_list`: Generates configurations for API calls, primarily from provided API keys.
- `config_list_openai_aoai`: Constructs a list of configurations using both Azure OpenAI and OpenAI endpoints, sourcing API keys from environment variables or local files.
- `config_list_from_json`: Loads configurations from a JSON structure, either from an environment variable or a local JSON file, with the flexibility of filtering configurations based on given criteria.
- `config_list_from_models`: Creates configurations based on a provided list of models, useful when targeting specific models without manually specifying each configuration.
- `config_list_from_dotenv`: Constructs a configuration list from a `.env` file, offering a consolidated way to manage multiple API configurations and keys from a single file.

We suggest that you take a look at this [notebook](https://github.com/microsoft/autogen/blob/main/notebook/oai_openai_utils.ipynb) for full code examples of the different methods to configure your model endpoints.

### Logic error[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#logic-error "Direct link to Logic error")

Another type of error is that the returned response does not satisfy a requirement. For example, if the response is required to be a valid json string, one would like to filter the responses that are not. This can be achieved by providing a list of configurations and a filter function. For example,

```
def valid_json_filter(response, **_):    for text in OpenAIWrapper.extract_text_or_completion_object(response):        try:            json.loads(text)            return True        except ValueError:            pass    return Falseclient = OpenAIWrapper(    config_list=[{"model": "text-ada-001"}, {"model": "gpt-3.5-turbo-instruct"}, {"model": "text-davinci-003"}],)response = client.create(    prompt="How to construct a json request to Bing API to search for 'latest AI news'? Return the JSON request.",    filter_func=valid_json_filter,)
```

The example above will try to use text-ada-001, gpt-3.5-turbo-instruct, and text-davinci-003 iteratively, until a valid json string is returned or the last config is used. One can also repeat the same model in the list for multiple times (with different seeds) to try one model multiple times for increasing the robustness of the final response.

_Advanced use case: Check this [blogpost](https://microsoft.github.io/autogen/blog/2023/05/18/GPT-adaptive-humaneval) to find how to improve GPT-4's coding performance from 68% to 90% while reducing the inference cost._

## Templating[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#templating "Direct link to Templating")

If the provided prompt or message is a template, it will be automatically materialized with a given context. For example,

```
response = client.create(    context={"problem": "How many positive integers, not exceeding 100, are multiples of 2 or 3 but not 4?"},    prompt="{problem} Solve the problem carefully.",    allow_format_str_template=True,    **config)
```

A template is either a format str, like the example above, or a function which produces a str from several input fields, like the example below.

```
def content(turn, context):    return "\n".join(        [            context[f"user_message_{turn}"],            context[f"external_info_{turn}"]        ]    )messages = [    {        "role": "system",        "content": "You are a teaching assistant of math.",    },    {        "role": "user",        "content": partial(content, turn=0),    },]context = {    "user_message_0": "Could you explain the solution to Problem 1?",    "external_info_0": "Problem 1: ...",}response = client.create(context=context, messages=messages, **config)messages.append(    {        "role": "assistant",        "content": client.extract_text(response)[0]    })messages.append(    {        "role": "user",        "content": partial(content, turn=1),    },)context.append(    {        "user_message_1": "Why can't we apply Theorem 1 to Equation (2)?",        "external_info_1": "Theorem 1: ...",    })response = client.create(context=context, messages=messages, **config)
```

## Logging (for openai<1)[​](https://microsoft.github.io/autogen/docs/Use-Cases/enhanced_inference#logging-for-openai1 "Direct link to Logging (for openai<1)")

When debugging or diagnosing an LLM-based system, it is often convenient to log the API calls and analyze them. `autogen.Completion` and `autogen.ChatCompletion` offer an easy way to collect the API call histories. For example, to log the chat histories, simply run:

```
autogen.ChatCompletion.start_logging()
```

The API calls made after this will be automatically logged. They can be retrieved at any time by:

```
autogen.ChatCompletion.logged_history
```

There is a function that can be used to print usage summary (total cost, and token count usage from each model):

```
autogen.ChatCompletion.print_usage_summary()
```

To stop logging, use

```
autogen.ChatCompletion.stop_logging()
```

If one would like to append the history to an existing dict, pass the dict like:

```
autogen.ChatCompletion.start_logging(history_dict=existing_history_dict)
```

By default, the counter of API calls will be reset at `start_logging()`. If no reset is desired, set `reset_counter=False`.

There are two types of logging formats: compact logging and individual API call logging. The default format is compact. Set `compact=False` in `start_logging()` to switch.

- Example of a history dict with compact logging.

```
{    """    [        {            'role': 'system',            'content': system_message,        },        {            'role': 'user',            'content': user_message_1,        },        {            'role': 'assistant',            'content': assistant_message_1,        },        {            'role': 'user',            'content': user_message_2,        },        {            'role': 'assistant',            'content': assistant_message_2,        },    ]""": {        "created_at": [0, 1],        "cost": [0.1, 0.2],    }}
```

- Example of a history dict with individual API call logging.

```
{    0: {        "request": {            "messages": [                {                    "role": "system",                    "content": system_message,                },                {                    "role": "user",                    "content": user_message_1,                }            ],            ... # other parameters in the request        },        "response": {            "choices": [                "messages": {                    "role": "assistant",                    "content": assistant_message_1,                },            ],            ... # other fields in the response        }    },    1: {        "request": {            "messages": [                {                    "role": "system",                    "content": system_message,                },                {                    "role": "user",                    "content": user_message_1,                },                {                    "role": "assistant",                    "content": assistant_message_1,                },                {                    "role": "user",                    "content": user_message_2,                },            ],            ... # other parameters in the request        },        "response": {            "choices": [                "messages": {                    "role": "assistant",                    "content": assistant_message_2,                },            ],            ... # other fields in the response        }    },}
```

- Example of printing for usage summary

```
Total cost: <cost>Token count summary for model <model>: prompt_tokens: <count 1>, completion_tokens: <count 2>, total_tokens: <count 3>
```

It can be seen that the individual API call history contains redundant information of the conversation. For a long conversation the degree of redundancy is high. The compact history is more efficient and the individual API call history contains more details.

---

