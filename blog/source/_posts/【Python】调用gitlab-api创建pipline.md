---
title: 【Python】调用gitlab-api创建pipline
tags: [python]
date: 2018-02-27
---

## 调用gitlab-api创建pipline

官网api:

```bash

POST /projects/:id/pipeline

curl --request POST --header "PRIVATE-TOKEN: 9koXpg98eAheJpvBs5tK" "https://gitlab.example.com/api/v4/projects/1/pipeline?ref=master"
```

### python 3.X

执行脚本： 

```python
# -*- coding: utf-8 -*-
import requests

url = "https://gitlab.com/api/v4/projects/4096120/pipeline?ref=master"
header_dict = {'PRIVATE-TOKEN': '9koXpg98eAheJpvBs5tK'}

res = requests.post(url=url, headers=header_dict)
print(res)
# 201 成功 
print(res.status_code)
```