---
title: 【openshift】在Openshift上通过yaml部署应用
tags: [openshift]
date: 2019-8-20
---

# 在Openshift上通过yaml部署应用

## 1.通过直接执行yaml

通过如下命令直接执行

```bash
oc create -f nginx.yml
```

**nginx.yml**
```yml
apiVersion: v1
items:
- apiVersion: apps.openshift.io/v1
  # okd 部署配置(dc)，与 k8s Deployment 资源对象类似，以启动多个容器的方式生成 pod
  kind: DeploymentConfig
  metadata:
    # 标签，在查询时具体资源对象时非常重要，如：>oc get dc -l app=nginx
    labels:
      app: nginx
    name: nginx
    # 选定项目空间
    namespace: test
  spec:
    # 副本数，即 nginx app 部署的实例数
    replicas: 1
    # pod 的计算资源配额
    resources:
      # pod 能分配的最大计算资源
      limits:
        cpu: 300m
        memory: 1024Mi
      # pod 分配的最少计算资源
      requests:
        cpu: 100m
        memory: 200Mi
    # 选择器 Service 根据此项来绑定到 dc
    selector:
      name: nginx
    strategy:
    #   type: Recreate
      type: Rolling
    # dc 根据模板里内容创建 pod
    template:
      metadata:
        labels:
          app: nginx
          name: nginx
          deploymentconfig: nginx
      spec:
        # 容器集合
        containers:
        - capabilities: {}
          # 容器内环境变量，下文给这个容器设置了时区和语言的环境变量
          env:
          - name: TZ
            value: Asia/Shanghai
          - name: LANG
            value: en_US.UTF-8
          # 容器使用什么镜像部署，在创建时需要替换成实际要部署的镜像
          image: nginx:1.16
          # 镜像下载策略，总是下载最新的(Always)
          imagePullPolicy: IfNotPresent
          ports: 
            - containerPort: 80
              protocol: TCP          

          # 健康检查-pod 是否存活
          livenessProbe:
            failureThreshold: 2
            # http get 请求的方式验证 pod-ip:80/
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 60
            periodSeconds: 60
            timeoutSeconds: 5
          name: nginx
          # 健康检查-pod 是否就绪
          readinessProbe:
            httpGet:
              path: /
              port: 80
            initialDelaySeconds: 3
            timeoutSeconds: 5
          # 容器计算资源配额，与 pod 配额类似
          resources:
            limits:
              cpu: 300m
              memory: 1024Mi
            requests:
              cpu: 100m
              memory: 200Mi
          securityContext:
            capabilities: {}
            # privileged: true
          terminationMessagePath: /dev/termination-log
          # 将 pod 中配置的卷挂载到容器内
        #   volumeMounts:
        #   - mountPath: /data
        #     name: nginx
        dnsPolicy: ClusterFirst
        # 使用节点选择器将应用固定部署到 node1 计算节点上
        # nodeSelector:
        #   kubernetes.io/hostname: node1.app.com
        restartPolicy: Always
        # 配置和定义使用实际计算节点主机文件夹地址这种类型的卷
        # volumes:
        # - persistentVolumeClaim:
        #     claimName: nginx
        #   name: nginx
    # pod 触发器--配置变动触发更新
    triggers:
    - type: ConfigChange
- apiVersion: v1
  # 服务，与 k8s 中 service 一样，将 dc 上部署的应用暴露给内部(多)或外部(少)
  kind: Service
  metadata:
    labels:
      app: nginx
    name: nginx
  spec:
    # 应用中要暴露的端口信息
    ports:
    - name: http
      # 对外暴露的端口
      port: 80
      protocol: TCP
      # 应用实际端口
      targetPort: 80
    # 通过选择器选择 dc
    selector:
      name: nginx
    sessionAffinity: None
    type: ClusterIP
    
- apiVersion: route.openshift.io/v1
  # OKD 平台特有资源，与 k8s 中 ingress 类似，用于将 service 正真暴露给外部使用，但只能使用域名访问
  kind: Route
  metadata:
    labels:
      app: nginx
    name: nginx
  spec:
    # 配置域名
    host: xinchen.app.com
    to:
      kind: Service
      name: nginx
kind: List
metadata: {}
```

---

## 2. 通过创建template

参考文档: https://docs.okd.io/latest/dev_guide/templates.html#writing-templates

相关指令

```bash

# 上传模板
oc create -f <filename> -n <project>

# 使用模板
oc process -f nginx-template -p IMAGE_NAME=nginx:1.6 -p REPLICA_COUNT=1

# 编辑模板
oc edit template <templateName>

# 导出模板
oc get -o yaml --export all > <yaml_filename>

```

**nginx-template.yml**
```yml
kind: Template
apiVersion: v1
metadata:
  name: nginx-template
objects: 
  - kind: DeploymentConfig
    apiVersion: v1
    metadata: 
      name: nginx
      namespace: xinchen
      labels: 
        app: nginx
    spec:
      replicas: ${REPLICA_COUNT}
      selector:
        name: nginx
        
      resources:
        limits:
          cpu: 300m
          memory: 1024Mi
        requests:
          cpu: 100m
          memory: 200Mi
    
      strategy:
        type: Rolling
    
      template:
        metadata:
          labels:
            app: nginx
            name: nginx
            deploymentconfig: nginx
        spec:
        
          containers:
          - capabilities: {}
          
            env:
            - name: TZ
              value: Asia/Shanghai
            - name: LANG
              value: en_US.UTF-8

            image: ${IMAGE_NAME}
  
            imagePullPolicy: IfNotPresent

            ports: 
              - containerPort: 80
                protocol: TCP          
          
            livenessProbe:
              failureThreshold: 2
              httpGet:
                path: /
                port: 80
              initialDelaySeconds: 60
              periodSeconds: 60
              timeoutSeconds: 5
            name: nginx
          
            readinessProbe:
              httpGet:
                path: /
                port: 80
              initialDelaySeconds: 3
              timeoutSeconds: 5
          
            resources:
              limits:
                cpu: 300m
                memory: 1024Mi
              requests:
                cpu: 100m
                memory: 200Mi

            securityContext:
              capabilities: {}
              # privileged: true
            terminationMessagePath: /dev/termination-log
          
          #   volumeMounts:
          #   - mountPath: /data
          #     name: nginx
          dnsPolicy: ClusterFirst
        
          restartPolicy: Always
        
          # volumes:
          # - persistentVolumeClaim:
          #     claimName: nginx
          #   name: nginx
    # pod 触发器--配置变动触发更新
      triggers:
      - type: ConfigChange  

  - kind: Service
    apiVersion: v1
    metadata:
      name: nginx
      labels:
        app: nginx    
    spec:
      ports: 
      - name: 80-tcp
        port : 80
        protocol: TCP
        targetPort: 80
      selector:
        name: nginx
      sessionAffinity: None
      type: ClusterIP

- apiVersion: route.openshift.io/v1
  kind: Route
  metadata:
    labels:
      app: nginx
    name: nginx
  spec:
    host: ${HOST_NAME}
    port:
      targetPort: 80-tcp
    to:
      kind: Service
      name: nginx
      weight: 100

parameters:
  - name: HOST_NAME
    displayName: Host Name
    required: true  
  - name: IMAGE_NAME
    displayName: Image Name
    required: true
  - name: REPLICA_COUNT
    displayName: Replica Count
    value: "1"
    required: true
```

---

## 3. 进入运行的容器

参考: https://docs.okd.io/latest/dev_guide/ssh_environment.html

```bash

# 查看pod名
oc get pods

# 进入容器
oc rsh <pod>

# 或者直接进入dc
oc rsh dc/nginx
```

## 4. 删除所有

```bash
# 删除所有
oc delete all -l app=nginx
```