```python
import os

import asyncio

from pathlib import Path

from typing import List, Optional

import sys

import nest_asyncio

from dotenv import load_dotenv

import os

from dotenv import load_dotenv

from crewai import Agent, Task, Crew, Process, LLM

  

# 适配 Windows 控制台编码：避免 CrewAI 日志/模型输出包含 emoji 或特殊字符时触发 UnicodeEncodeError

try:

   sys.stdout.reconfigure(encoding="utf-8")

   sys.stderr.reconfigure(encoding="utf-8")

except Exception:

   pass

  

## --- 配置 ---

## 从项目根目录的 .env 文件加载环境变量

_PROJECT_ROOT = Path(__file__).resolve().parents[2]

load_dotenv(_PROJECT_ROOT / ".env")

  

_anthropic_key = os.environ.get("ANTHROPIC_AUTH_TOKEN")

_anthropic_base_url = os.environ.get("ANTHROPIC_BASE_URL")

## CrewAI Anthropic 原生实现默认读取环境变量名为 ANTHROPIC_API_KEY。

## 你的 .env 使用的是 ANTHROPIC_AUTH_TOKEN，这里做一次兼容映射，避免初始化失败。

if _anthropic_key and not os.environ.get("ANTHROPIC_API_KEY"):

   os.environ["ANTHROPIC_API_KEY"] = _anthropic_key

_anthropic_key = os.environ.get("ANTHROPIC_API_KEY") or _anthropic_key

  

## 初始化 Chat LLM（通过兼容网关调用 Anthropic）。

## 如果初始化失败，则设置 llm = None，并跳过后续反思循环。

_DEFAULT_MODEL = "claude-3-5-haiku-20241022"

try:

   _anthropic_model = (os.environ.get("ANTHROPIC_MODEL") or _DEFAULT_MODEL).strip()

   llm: Optional[LLM] = LLM(

      model=_anthropic_model,

      provider="anthropic",

      api_key=_anthropic_key,

      base_url=_anthropic_base_url,

      temperature=0.7,

      max_tokens=4096,

   )

except Exception as e:

   print(f"Error initializing language model: {e}")

   llm = None

  
  

## 2. Define a clear and focused agent

planner_writer_agent = Agent(

   role='Article Planner and Writer',

   goal='Plan and then write a concise, engaging summary on a specified topic.',

   backstory=(

       'You are an expert technical writer and content strategist. '

       'Your strength lies in creating a clear, actionable plan before writing, '

       'ensuring the final summary is both informative and easy to digest.'

   ),

   verbose=True,

   allow_delegation=False,

   llm=llm # Assign the specific LLM to the agent

)

  

## 3. Define a task with a more structured and specific expected output

topic = "The importance of Reinforcement Learning in AI"

high_level_task = Task(

   description=(

       f"1. Create a bullet-point plan for a summary on the topic: '{topic}'.\n"

       f"2. Write the summary based on your plan, keeping it around 200 words."

   ),

   expected_output=(

       "A final report containing two distinct sections:\n\n"

       "### Plan\n"

       "- A bulleted list outlining the main points of the summary.\n\n"

       "### Summary\n"

       "- A concise and well-structured summary of the topic."

   ),

   agent=planner_writer_agent,

)

  

## Create the crew with a clear process

crew = Crew(

   agents=[planner_writer_agent],

   tasks=[high_level_task],

   ## 执行流程按照顺序执行

   process=Process.sequential,

)

  

## Execute the task

print("## Running the planning and writing task ##")

## 执行任务

result = crew.kickoff()

print("\n\n---\n## Task Result ##\n---")

print(result)
```