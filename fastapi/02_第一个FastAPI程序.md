```
from fastapi import FastAPI

# 创建Fastapi实例
app = FastAPI()

# 创建路由
@app.get("/")
async def root():
	return {"message": "Hello World"}
	
@app.get("/hello/{name}")
async def say_hello(name: str):
	return {"message": f"Hello {name}"}
```