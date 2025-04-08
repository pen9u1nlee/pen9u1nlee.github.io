---
title: Semantic Kernel入门把玩
tags: AI
pageview: false
modify_date: 2025-04-08
aside:
  toc: true
---

<!--more-->

> 前言：鉴于当下主流的国内外大语言模型厂商均支持OpenAI格式的API接入，即便你未曾向OpenAI付费，也能在API中稍作配置，接入国内的模型。具体的实现方式可参考本文给出的示例代码。

在生成式人工智能的时代浪潮下，人机对话的模式发生了显著变革。如今，我们只需运用自然语言，便能轻松与机器展开对话。

当前的设想是，我们不仅要借助自然语言与机器交流，更要通过下达自然语言指令，让机器为我们完成各类任务。

以下是几个具体的用户故事：

1. 你向小爱同学下达开关电器设备的指令。小爱同学会依据自然语言指令，精准调用指定的回调函数，从而实现电器的操作。
2. 你正基于近期公司可观测性平台的指标，撰写双月OKR。这类工作可由大模型分解为查询指标和撰写文档两个步骤。其中，查询指标这一环节需要调用公司内部的数据库接口。
3. 你同时与三位暧昧对象聊天，分别采用了三家不同的大模型供应商，为每个对象设定了独特的人设，并配备了对应的提示词模板。从技术层面来看，你需要在代码中配置各家的key、endpoint、模型、温度、max token等超参数。而且，与不同暧昧对象的聊天可能涉及不同的业务场景和风格（例如订酒店、买药、买花，或者采用舔狗、高冷、文艺的聊天路线），这就需要不同的模型进行针对性处理。

面对如此繁杂的需求，我们需要一种可插拔式的方式，将各种工具和设备集成起来，以实现灵活调用。

在这种情况下，Semantic Kernel的价值便得以凸显。

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

内核是 Semantic Kernel 的核心组件，负责管理运行 AI 应用程序所需的所有服务和插件。

参考上文示例，我们分别创建了一个豆包客户端对象（服务）和一组灯的开关逻辑（插件），并将它们交由内核进行管理。

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

* **服务**包括AI服务（比如大模型聊天客户端）和其他的服务（比如日志服务、HTTP客户端等）。
* **插件**是可供 AI 服务调用的功能模块。例如，你的插件可以实现数据库接口，或者调用外部 API 来执行特定指令（如开关灯）。

Semantic Kernel 会在恰当的时机调用这些服务和插件。当你在 Semantic Kernel 中运行任何提示或代码时，内核会自动检索所需的服务和插件并进行调用。
对于开发者来说，内核是管理和监控 AI 应用的理想工具。

例如，当你向内核输入一句提示词时，内核将执行以下操作：
1. 选择合适的 AI 服务，如豆包、DeepSeek 或 Hugging Face 的模型。
2. 根据提示词模板整合提示词。
3. 向 AI 服务发送整合后的提示词。
4. 接收并解析 AI 服务的回复。
5. 返回最终结果。

Semantic Kernel 的不同客户端目前支持以下AI服务：
![alt text](/img/2025-04-08-SemanticKernel/image-1.png)

## semantic kernel的组件

Semantic Kernel 提供了众多不同的组件，这些组件既可以单独使用，也可以组合使用。

**AI 服务连接器**、

AI 服务连接器提供一个抽象层，实现对多种 AI 服务的接入。支持接入的服务包括聊天补全、文本生成、嵌入生成、文本转图像、图像转文本、文本转音频、音频转文本。

当向 Semantic Kernel 注册一个 AI 服务时，对内核的任何方法调用都将默认路由到聊天服务，其他服务不会被自动调用，除非当前的大模型服务选择了其他的服务。

**向量存储（内存）连接器**

向量存储连接器主要用于将各家的历史数据和接口描述进行向量化降维并存储。在需要时（如模型生成回答或调用插件 API），可以快速检索这些数据。

Semantic Kernel 的向量存储连接器提供了一个抽象层，通过公共接口将不同提供商的向量存储暴露出来。

**函数和插件**

插件是函数的集合，每个插件都有一个名称。例如，某个插件的名称为 “灯光控制组件”，集成了若干个灯的开关函数逻辑，它可以控制所有物理灯光的开关（由于制造厂商不同，每个灯的开关控制由不同的函数实现）。

![alt text](/img/2025-04-08-SemanticKernel/image-2.png)

插件可以注册到内核中，其主要功能如下：

1. 供 AI 调用。
2. 在模板中被调用。

> 这里所谓的“在模板中被调用”指的是在提示词模板中调用插件里的函数。当内核处理提示词模板时，会自动调用相应的插件函数来完成某些任务。
> 结合前面的例子，我们要根据当前灯光状态生成描述信息。提示词模板就可以写为：`当前灯光状态如下：{{Lights.get_lights}}，请生成一段关于灯光状态的描述。`
> 这里的`{{Lights.get_lights}}`就是在模板里调用`LightsPlugin`插件的`get_lights`函数。内核在处理该模板时，会先调用`get_lights`函数获取灯光状态信息，接着把信息填充到模板中，再将最终提示词传递给AI模型处理。

插件中的函数形式多样，可以是自定义的回调函数，可以是包裹了提示词的大模型，也可以是基于 ElasticSearch 的 RAG 等。

**提示词模板**

模板中可以包含给大模型的指令、用户输入占位符，以及在调用大模型之前对插件的调用（例如，在进行数据分析前需要先获取数据，可以通过插件来实现）。
可以通过以下两种方式使用提示模板：
1. 内核会对模板进行处理，填充其中的各种元素（如占位符），然后将最终的提示传递给 AI 模型进行处理。
2. 像调用普通函数一样调用模板。这样可以使用自然语言而非硬编码来描述功能。这种将功能分离到插件中的方式，允许模型进行分步骤推理，由于模型可以一次专注于一个问题，从而可能提高推理的成功率。

![alt text](/img/2025-04-08-SemanticKernel/image-3.png)

**过滤器**

在任务执行期间，需要使用过滤器为特定事件前后自定义操作方法。比如对输入输出做一些风控过滤监测，避免涉h、摄政回答；或者打几条日志实现可观测性。

过滤器需要在semantic kernel中进行注册，这样才能在聊天补全流程中被调用。只有经过注册，内核才能在相应的事件发生时找到并执行对应的过滤器逻辑。

![alt text](/img/2025-04-08-SemanticKernel/image-4.png)
