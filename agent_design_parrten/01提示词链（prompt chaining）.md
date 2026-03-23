![](assets/01提示词链（prompt%20chaining）/file-20260323184402523.png)
![675](assets/01提示词链（prompt%20chaining）/file-20260323184413379.png)![](assets/01提示词链（prompt%20chaining）/file-20260323185036533.png)
```
import os
// 导入openai模块
from langchain_openai import ChatOpenAI
// 导入提示词模板
from langchain_core.prompts import ChatPromptTemplate
// 导入输出解析器
from langchain_core.output_parsers import StrOutputParser

## For better security, load environment variables from a .env file
## from dotenv import load_dotenv
## load_dotenv()
## Make sure your OPENAI_API_KEY is set in the .env file

## Initialize the Language Model (using ChatOpenAI is recommended)
// 初始化模型
llm = ChatOpenAI(temperature=0)

## --- Prompt 1: Extract Information ---
// 定义第一轮提示词，text_input作为用户的提问
prompt_extract = ChatPromptTemplate.from_template(
    "Extract the technical specifications from the following text:\n\n{text_input}"
)

## --- Prompt 2: Transform to JSON ---
// 定义第二轮提示词，specifications是上一个链的输出
prompt_transform = ChatPromptTemplate.from_template(
    "Transform the following specifications into a JSON object with 'cpu', 'memory', and 'storage' as keys:\n\n{specifications}"
)

## --- Build the Chain using LCEL ---
## The StrOutputParser() converts the LLM's message output to a simple string.
## 构造第一条提示词链
extraction_chain = prompt_extract | llm | StrOutputParser()

## The full chain passes the output of the extraction chain into the 'specifications'
## variable for the transformation prompt.
## 构造完整的链，把第一条链的输出与后面的链连接起来
full_chain = (
    {"specifications": extraction_chain}
    | prompt_transform
    | llm
    | StrOutputParser()
)

## --- Run the Chain ---
input_text = "The new laptop model features a 3.5 GHz octa-core processor, 16GB of RAM, and a 1TB NVMe SSD."

## Execute the chain with the input text dictionary.
## invoke执行链得到结果
final_result = full_chain.invoke({"text_input": input_text})

print("\n--- Final JSON Output ---")
print(final_result)
```

1.`RunnablePassthrough()` 接收的参数，就是**流向它的那一步的输入数据**，并原封不动的把原来数据输出出来

2.`RunnableBranch`（分支） 接收一个列表，列表里包含多个 **`(条件函数, 执行链条)`** 的元组，最后跟一个 **默认链条**。
它的执行逻辑如下：
1. **拿到数据**：接收上一步传来的数据（比如包含 `decision` 和 `request` 的字典）。
2. **依次检查**：从上到下，依次运行每个元组里的**条件函数**。
3. **命中即停**：
    - 如果条件函数返回 `True` 👉 **立即执行**对应的链条，并**忽略**后面所有的分支。
    - 如果条件函数返回 `False` 👉 **跳过**该分支，检查下一个。
4. **默认兜底**：如果所有条件都不满足，则执行最后一个**默认链条**。

3.
	**`RunnablePassthrough`**：保证原始输入数据原封不动地传递下去。
	**`.assign(...)`**：在原始数据的基础上，计算并添加新的字段。
	![](assets/01提示词链（prompt%20chaining）/file-20260323183027220.png)