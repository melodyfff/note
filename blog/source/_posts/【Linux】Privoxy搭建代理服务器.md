---
title: 【Linux】Privoxy搭建代理服务器
tags: [Linux]
date: 2019-10-30
---
# Privoxy搭建代理服务器
[Docker Hub镜像地址](https://hub.docker.com/r/splazit/privoxy-alpine)

## Dockerfile
```dockerfile
FROM alpine

EXPOSE 8118

RUN apk --no-cache --update add privoxy wget ca-certificates bash p7zip && \
    wget https://s3.amazonaws.com/ab2p/ab2p.all_rus.7z && \
    mkdir temp && \
    7za e ab2p.all_rus.7z -y -otemp && \
    cp temp/ab2p.system.action temp/ab2p.action temp/ab2p.system.filter temp/ab2p.filter /etc/privoxy && \
    sed -i'' 's/127\.0\.0\.1:8118/0\.0\.0\.0:8118/' /etc/privoxy/config && \
    sed -i'' 's/enable-edit-actions\ 0/enable-edit-actions\ 1/' /etc/privoxy/config && \
    sed -i'' 's/#max-client-connections/max-client-connections/' /etc/privoxy/config && \
    sed -i'' 's/accept-intercepted-requests\ 0/accept-intercepted-requests\ 1/' /etc/privoxy/config && \    
    sed -i'' 's/http/https/g' /etc/privoxy/ab2p.system.filter && \
    echo 'actionsfile ab2p.system.action' >> /etc/privoxy/config && \
    echo 'actionsfile ab2p.action' >> /etc/privoxy/config && \
    echo 'filterfile ab2p.system.filter' >> /etc/privoxy/config && \
    echo 'filterfile ab2p.filter' >> /etc/privoxy/config && \
    rm -Rf temp ab2p.all_rus.7z && \
    apk del bash p7zip
RUN chown privoxy.privoxy /etc/privoxy/*
ENTRYPOINT ["privoxy"]
CMD ["--no-daemon","--user","privoxy","/etc/privoxy/config"]
```
---

## RUN
```bash
docker run -d --restart unless-stopped --name privoxy -p 8118:8118 splazit/privoxy-alpine
```
---

## TEST
```bash
curl -vv https://www.bing.com --proxy localhost:8118
```
