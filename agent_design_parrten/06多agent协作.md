```python
import os

import sys

from pathlib import Path

from typing import Optional

from dotenv import load_dotenv

from crewai import Agent, Task, Crew, Process, LLM

  

# 适配 Windows 控制台编码

try:

   sys.stdout.reconfigure(encoding="utf-8")

   sys.stderr.reconfigure(encoding="utf-8")

except Exception:

   pass

  

## --- 配置 ---

_PROJECT_ROOT = Path(__file__).resolve().parents[2]

load_dotenv(_PROJECT_ROOT / ".env")

  

_anthropic_key = os.environ.get("ANTHROPIC_AUTH_TOKEN")

_anthropic_base_url = os.environ.get("ANTHROPIC_BASE_URL")

  

# 兼容映射

if _anthropic_key and not os.environ.get("ANTHROPIC_API_KEY"):

   os.environ["ANTHROPIC_API_KEY"] = _anthropic_key

_anthropic_key = os.environ.get("ANTHROPIC_API_KEY") or _anthropic_key

  

## 初始化 LLM

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

  

# ==========================================

# 1. 定义专业 Agent 团队 (员工 - 不包含经理)

# ==========================================

  

market_analyst = Agent(

    role='市场分析师',

    goal='分析初创公司的市场规模、趋势和竞争对手。',

    backstory=(

        "你是一家顶级风险投资公司的资深市场分析师。"

        "你擅长提供基于数据的市场洞察，包括 TAM/SAM/SOM、增长趋势和竞争格局。"

    ),

    verbose=True,

    allow_delegation=False,

    llm=llm

)

  

product_strategist = Agent(

    role='产品战略家',

    goal='定义产品的独特价值主张 (UVP) 和市场定位。',

    backstory=(

        "你曾是 Google 的产品经理，现在专门指导初创公司。"

        "你擅长构建清晰、引人注目的产品叙事，能够引起投资者的共鸣。"

    ),

    verbose=True,

    allow_delegation=False,

    llm=llm

)

  

financial_modeler = Agent(

    role='财务建模师',

    goal='为初创公司创建 realistic 的 3 年财务预测。',

    backstory=(

        "你是一位 CFO 级别的顾问，专注于早期初创公司。"

        "你擅长构建清晰、站得住脚的财务模型，重点关注单位经济效益和资金消耗率。"

    ),

    verbose=True,

    allow_delegation=False,

    llm=llm

)

  

pitch_designer = Agent(

    role='路演幻灯片设计师',

    goal='构建并设计一份引人注目的投资者路演幻灯片 (Pitch Deck)。',

    backstory=(

        "你是一位设计师，曾帮助 50 多家初创公司完成 A 轮融资。"

        "你确切知道投资者想看什么样的幻灯片，以及如何排列顺序以产生最大影响力。"

    ),

    verbose=True,

    allow_delegation=False,

    llm=llm

)

  

# ==========================================

# 2. 定义 经理 Agent (独立于列表之外)

# ==========================================

  

manager = Agent(

    role='项目总经理 (Project Manager)',

    goal='监督整个路演幻灯片的制作流程，拆解任务，分配给合适的专家，并审核最终成果。',

    backstory=(

        "你是一位拥有 15 年经验的风险投资合伙人，曾主导过数百个项目的尽职调查。"

        "你擅长统筹全局，能够精准地将复杂的大目标拆解为具体的执行步骤，并分配给市场、产品、财务和设计专家。"

        "你会严格审核每一份交付物，确保逻辑连贯、数据准确，并最终整合成一份完美的路演材料。"

    ),

    verbose=True,

    allow_delegation=True,

    llm=llm

)

  

# ==========================================

# 3. 定义任务

# ==========================================

  

topic = "FlowTask"

  

market_task = Task(

    description=(

        f"针对 SaaS 初创公司 '{topic}' 进行深入的市场分析。"

        f"{topic} 帮助远程团队利用 AI 管理异步工作流。\n"

        "你需要产出包含 TAM/SAM/SOM、关键趋势、竞争格局和时机论证的详细报告。"

    ),

    expected_output="一份详细的 Markdown 格式市场分析报告，包含 '### 市场分析' 标题。",

    agent=market_analyst,

)

  

product_task = Task(

    description=(

        f"基于市场分析，为 '{topic}' 制定产品定位和独特价值主张 (UVP)。"

        "明确目标客户、痛点、差异化优势和进入市场策略。"

    ),

    expected_output="一份 Markdown 格式的产品定位文档，包含 '### 产品定位与独特价值主张' 标题。",

    agent=product_strategist,

)

  

financial_task = Task(

    description=(

        f"为 '{topic}' 创建 3 年财务预测模型。"

        "假设正在寻求 500 万美元 A 轮融资。需包含收入模型、年度预测 (ARR)、关键 KPI (CAC, Churn, ARPA) 及增长叙述。"

    ),

    expected_output="一份 Markdown 格式的财务预测报告，包含 '### 三年财务预测' 标题。",

    agent=financial_modeler,

)

  

deck_task = Task(

    description=(

        f"整合所有之前的分析（市场、产品、财务），为 '{topic}' 制作一份最终的投资者路演幻灯片草稿。"

        "要求：逐页结构、逻辑流畅的叙事、包含执行摘要，直接可用于演示。"

    ),

    expected_output="一份完整的、结构化的路演幻灯片草稿 (Markdown 格式)。",

    agent=pitch_designer,

)

  

# ==========================================

# 4. 组建 Crew (层级模式) - 修正处在这里

# ==========================================

  

crew = Crew(

    # 【修正点 1】agents 列表中只包含员工，不包含 manager

    agents=[

        market_analyst,

        product_strategist,

        financial_modeler,

        pitch_designer,

        # manager  <-- 删除了这一行

    ],

    tasks=[

        market_task,

        product_task,

        financial_task,

        deck_task

    ],

    process=Process.hierarchical,

    manager_agent=manager,        # 【修正点 2】在这里单独指定经理

    verbose=True,

)

  

print("## 启动层级多智能体工作流 (带经理) ##")

print("经理正在分析任务并分配给团队成员...\n")

  

try:

    result = crew.kickoff()

    print("\n\n########################")

    print("## 最终路演幻灯片 ##")

    print("########################\n")

    print(result)

except Exception as e:

    print(f"\n❌ 执行过程中发生错误: {e}")

    print("提示：如果是 Tool Use 相关错误，可能是 Claude 模型与层级模式的兼容性问题。")
```