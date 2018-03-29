---
title: 【docker】清理空间
tags: [docker]
date: 2018-3-29
---

>  Docker清理空间

```bash
docker volume rm $(docker volume ls -f dangling=true -q)

docker system prune --volumes

```