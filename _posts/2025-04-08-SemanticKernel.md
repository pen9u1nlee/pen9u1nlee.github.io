---
title: Semantic Kernel入门把玩
tags: AI
pageview: false
modify_date: 2025-04-08
aside:
  toc: true
---

<!--more-->

> 写在前面的话：由于现在的主流国内外大语言模型厂商都支持openai格式的API接入，所以即使你没有给openai掏钱，也可以在API里改几个配置接入国内的模型。具体实现可以参照本文给出的示例代码。

在生成式人工智能时代，人机对话的方式有了很大的改变，我们用自然语言就可以完成与机器的对话。

现在的想法是，我们不光要用自然语言和机器对话，还要下达自然语言指令，让机器为我们干活。

于是有这么几个用户故事：

1. 你让小爱同学开关电器设备。小爱同学根据自然语言指令，选择调用指定的回调函数，实现电器操作。
2. 你还在基于近期公司可观测性平台指标，写双月OKR。这种活可以被大模型分解为查指标和写文档。查指标这种事情，需要调用公司内部的数据库接口。
3. 你同时还在和三个暧昧对象聊天，分别有三家不同的大模型供应商、三个不同的人设、对应的提示词模板。从技术上讲，你还要代码中写上各家的key、endpoint、模型、温度、max token等超参。而且，你和暧昧对象聊天中还可能包含不同的业务和风格（比如订酒店/买药/买花，或者走舔狗/高冷/文艺路线），还需要不同的模型进行针对性处理。

你手里有这么多需求，需要可插拔式地把工具和设备集成起来，供你做灵活调用。

这个时候，Semantic Kernel 就体现出价值了。

![alt text](/img/2025-04-08-SemanticKernel/image.png)

## Hello world

下面是一段非常简单的semantic kernel官网示例代码，支持你用自然语言控制三台虚拟台灯的开关。

> 由于我没有给openai掏钱，因此基于字节方舟大模型进行了适配。

```python
import asyncio
import logging
from semantic_kernel import Kernel
from semantic_kernel.utils.logging import setup_logging
from semantic_kernel.functions import kernel_function
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion
from semantic_kernel.connectors.ai.function_choice_behavior import FunctionChoiceBehavior
from semantic_kernel.connectors.ai.chat_completion_client_base import ChatCompletionClientBase
from semantic_kernel.functions.kernel_arguments import KernelArguments
from semantic_kernel.connectors.ai.open_ai.prompt_execution_settings.azure_chat_prompt_execution_settings import (
    AzureChatPromptExecutionSettings,
)

from openai import AsyncOpenAI

from semantic_kernel.contents import ChatHistory
from semantic_kernel.connectors.ai.open_ai import (
    OpenAIChatCompletion,
    OpenAIChatPromptExecutionSettings,
)

DEEPSEEK_API_KEY = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" # 这里写你自己的API-KEY

from typing import Annotated
from semantic_kernel.functions import kernel_function

class LightsPlugin:
    lights = [
        {"id": 1, "name": "Table Lamp", "is_on": False},
        {"id": 2, "name": "Porch light", "is_on": False},
        {"id": 3, "name": "Chandelier", "is_on": True},
    ]

    @kernel_function(
        name="get_lights",
        description="Gets a list of lights and their current state",
    )
    def get_state(
        self,
    ) -> str:
        """Gets a list of lights and their current state."""
        return self.lights

    @kernel_function(
        name="change_state",
        description="Changes the state of the light",
    )
    def change_state(
        self,
        id: int,
        is_on: bool,
    ) -> str:
        """Changes the state of the light."""
        for light in self.lights:
            if light["id"] == id:
                light["is_on"] = is_on
                return light
        return None

async def main():
    # Initialize the kernel
    kernel = Kernel()

    # Add Azure OpenAI chat completion
    chat_completion = OpenAIChatCompletion(
        ai_model_id="doubao-1-5-lite-32k-250115", # "deepseek-r1-250120",    # or "deepseek-reasoner"
        async_client=AsyncOpenAI(
            api_key=DEEPSEEK_API_KEY,
            base_url="https://ark.cn-beijing.volces.com/api/v3",
        ),
    )
    kernel.add_service(chat_completion)

    # Set the logging level for  semantic_kernel.kernel to DEBUG.
    setup_logging()
    logging.getLogger("kernel").setLevel(logging.DEBUG)

    # Add a plugin (the LightsPlugin class is defined below)
    kernel.add_plugin(
        LightsPlugin(),
        plugin_name="Lights",
    )

    # Enable planning
    execution_settings = OpenAIChatPromptExecutionSettings()
    execution_settings.function_choice_behavior = FunctionChoiceBehavior.Auto()

    # Create a history of the conversation
    history = ChatHistory()

    # Initiate a back-and-forth chat
    userInput = None
    while True:
        # Collect user input
        userInput = input("User > ")

        # Terminate the loop if the user says "exit"
        if userInput == "exit":
            break

        # Add user input to the history
        history.add_user_message(userInput)

        # Get the response from the AI
        result = await chat_completion.get_chat_message_content(
            chat_history=history,
            settings=execution_settings,
            kernel=kernel,
        )

        # Print the results
        print("Assistant > " + str(result))

        # Add the message from the agent to the chat history
        history.add_message(result)

# Run the main function
if __name__ == "__main__":
    asyncio.run(main())
```

## 内核

内核是semantic kernel的核心组件，用于管理运行 AI 应用程序所需的所有服务和插件。 

参考上文示例，我们分别实现了一个豆包客户端对象（服务）和一组灯的开关逻辑（插件），并将其交给内核进行托管。

```python
# 添加服务
kernel.add_service(chat_completion)
...

# 添加插件
kernel.add_plugin(
        LightsPlugin(),
        plugin_name="Lights",
    )
```

具体地，

* **服务**包括AI服务（比如大模型聊天客户端）和其他的服务（比如日志服务、HTTP客户端）
* **插件**是可以被你的AI服务调用的功能。比如，你的插件实现了数据库接口，或者调用外部的API来执行一些指令（比如开关灯）。

semantic kernel会在合适的时候调用这些服务和插件。如果你在semantic kernel中运行任何提示或代码，内核将为你检索必要的服务和插件并调用之。

于是，作为开发者，这个内核是你管理和监控AI应用的绝佳对象。

举个例子，当你给这个内核喂了一句提示词后，内核将：
1. 选择合适的AI服务，比如豆包、deepseek或者huggingface的模型
2. 基于提示词模板对提示词进行整合
3. 向AI服务发送提示词
4. 接受并且解析回复
5. 返回结果

既然这些东西是分步骤的，于是埋个日志点，于是就有了可观性；让小爱同学给你远程关灯也可以实现。

semantic kernel的不同客户端目前支持这些服务：
![alt text](/img/2025-04-08-SemanticKernel/image-1.png)

## semantic kernel的组件

semantic kernel提供了许多不同的组件，这些组件可以单独或一起使用。

**AI 服务连接器**：提供一个抽象层，实现多种AI服务的接入。支持接入的服务包括Chat Completion, Text Generation, Embedding Generation, Text to Image, Image to Text, Text to Audio, Audio to Text.

当向semantic kernel注册了一个AI服务时，对内核的任何方法调用都将默认路由到聊天服务，其他服务都不会被自动调用。

**向量存储（内存）连接器**

这个向量存储连接器，主要是为了将各家的历史数据和接口描述进行向量化降维和存储，在需要的时候（比如模型生成回答或者调用插件API）时，可以进行快速检索。

semantic kernel向量存储连接器提供一个抽象层，通过公共接口把来自不同提供商的向量存储暴露出来。

**函数和插件**

插件是函数的集合，插件具有名称。比如，某个插件的名称定义为“灯光控制组件”，可以控制物理所有灯光的开关（由于制造厂商不同，每个灯分别由不同的函数实现开关控制）。

![alt text](/img/2025-04-08-SemanticKernel/image-2.png)

插件可以注册到内核中，功能如下：
1. 用于给AI做调用
2. 在模板里被调用（？）

插件中函数的形式是多样化的，可以是自己写的回调，可以是包了一层提示词的大模型，也可以是基于ElasticSearch的RAG等。

**提示词模板**

模板中可以包含给大模型的指令、用户输入占位符，以及在调用大模型之前对插件的调用（比如在数据分析前需要先有数据，通过插件获取）。

可以通过两种方式使用提示模板：

1. 内核对模板进行处理，把其中的各种元素（如占位符填充等）处理好后，将最终的提示传递给 AI 模型进行处理。
2. 像任何函数调用一样调用模板。这样可以使用自然语言而不是硬编码来描述功能。这种将功能分离到插件中的方式，允许模型的分步骤推理。并且由于模型可以一次专注于一个问题，从而可能提高成功率 。

![alt text](/img/2025-04-08-SemanticKernel/image-3.png)

**过滤器**

在任务执行期间，需要使用过滤器为特定事件前后自定义操作方法。比如对输入输出做一些风控过滤监测，避免涉h、摄政回答；或者打几条日志实现可观测性。

过滤器需要在semantic kernel中进行注册，这样才能在聊天补全流程中被调用。只有经过注册，内核才能在相应的事件发生时找到并执行对应的过滤器逻辑。

![alt text](/img/2025-04-08-SemanticKernel/image-4.png)
