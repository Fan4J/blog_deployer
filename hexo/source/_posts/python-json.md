---
title: python json
date: 2017-08-18 11:05:37
tags: 
	- Python
---
>python世界里，json和dict是天生一对，他们之间的转换是必须要熟练的

# 1.dumps/loads 
```
dict1 = {"fan":123456,"gaga":"4j"}
json1 = json.dumps(dict1)
print(json1)
print(type(json1))

dict2 = json.loads(str(json1))
print(dict2)
print(type(dict2))
```
输出如下
```
{"gaga": "4j", "fan": 123456}
<class 'str'>
{'fan': 123456, 'gaga': '4j'}
<class 'dict'>
```
# 2.dump/load
```
dict = {"fan":1123,"gaga":"12312"}
with open("test.txt","w") as f:
     json.dump(dict,f)

with open("test.txt","r") as f:
     dict1 = json.load(f)
print(dict1)
print(type(dict1))
```
输出如下
```
{'gaga': '12312', 'fan': 1123}
<class 'dict'>
```
以上，后面会继续补充