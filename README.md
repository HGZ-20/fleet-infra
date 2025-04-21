本仓库用于验证华为云CCE集群中使用fluxcd将Git仓库的应用程序自动化部署到集群中，对应应用仓库为[podinfo](https://github.com/HGZ-20/podinfo)

# 1 背景介绍
[开源for Huawei](developer.huaweicloud.com/programs/opensource/contributing/)通过和公司、高校、社区的开发者合作，完成鲲鹏、昇腾、欧拉、鸿蒙、高斯、云服务等与开源软件的适配开发，帮助繁荣Huawei的基础生态，同时让开源软件能够更加简单、高效的运行于华为云上。
开始之前，开发者可以下载[开源for Huawei Wiki](gitcode.com/HuaweiCloudDeveloper/OpenSourceForHuaweiWiki/overview)了解详细的开发步骤，技术准备，以及开发过程需要的各种资源。

# 2 需求分析
## 2.1 开源软件基本情况
相对于 DevOps 而言，GitOps 是一种全新的理念，它将 Git 仓库作为一切事实的唯一来源，fluxcd 项目就是这一理念的践行者。Flux 是一种工具，用于使 Kubernetes 集群与配置（如 Git 资料档案库和 OCI 构件）， 以及在有新代码要部署时自动更新配置。Flux 版本 2（“v2”）是从头开始构建的，以使用 Kubernetes 的 API 扩展系统，并与 Prometheus 等核心集成 Kubernetes 生态系统的组件。在版本 2 中，Flux 支持多租户并支持同步任意数量的 Git repositories 以及其他长期请求的功能。

源码地址：github.com/fluxcd/flux2

官网主页：fluxcd.io/

主要开发语言：Go

LICENSE：Apache 2.0

维护者：CNCF Fluxcd-开源社区

项目数据：Fork 612, Star 6.6K, Contributor 155，最近一次提交：2024-11-14
star历史（star-history.com/）：
![image](https://github.com/user-attachments/assets/42ab8142-677f-46e4-a856-38fbd63560ba)

## 2.2 任务目的和范围
本次任务的主要目的是验证如何利用 fluxcd 将 Git 仓库的应用程序自动部署到华为云CCE集群，让集群中的工作负载随着 Git 代码库的更新而自动更新，从而降低运维成本。

# 3 华为云CCE使用fluxcd自动部署podinfo

## 3.1 购买CCE集群和配置鲲鹏节点资源
购买CCE集群（因为要验证鲲鹏云，因此控制节点选择鲲鹏）
![image](https://github.com/user-attachments/assets/83e1dd4d-def7-4694-9ef4-db84c68fc96b)

创建节点池并购买节点资源（鲲鹏服务器）
![image](https://github.com/user-attachments/assets/65bd884d-9625-49bb-84c8-878db61ce3a2)

## 3.2 在个人仓库fork podinfo仓库
Podinfo 是一个用 Go 语言编写的微型 Web 应用程序，展示了在 Kubernetes 中运行微服务的最佳实践。Podinfo 被如 Flux 和 Flagger 等 CNCF 项目用于端到端测试和工作坊。因此我们使用podinfo作为自动化部署应用来验证华为云的CCE集群是否可以使用Fluxcd通过git自动化部署应用。

为了能自行控制podinfo，首先需要在Github上Fork [Podinfo](github.com/stefanprodan/podinfo.git)仓库

## 3.3 连接CCE集群
可以直接通过华为云的CCE控制台右上角的`命令行工具`连接
![image](https://github.com/user-attachments/assets/3a30a3db-b547-4492-8e90-14b37ee375ec)
也可以在自己机器上通过`kubectl`连接集群，参考[官方文档](support.huaweicloud.com/usermanual-cce/cce_10_0107.html)

## 3.4 安装flux
安装过程参考[flux官方文档](fluxcd.io/flux/installation/#install-the-flux-cli)

以linux为例，运行下面命令安装：
``` bash
curl -s https://fluxcd.io/install.sh | sudo bash
```
安装完成后，运行下面版本查看命令检查是否正确安装：
```bash
flux -v
```
输出类似于：
```
# flux -v
flux version 2.5.1
```

## 3.5 获取Github个人访问令牌
参考Github官方文档[管理个人访问令牌](docs.github.com/zh/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)获得账户的访问令牌，如果是组织或企业账户同样可以参考Flux bootstrap for GitHub(fluxcd .io/flux/installation/bootstrap/github/)

获取到令牌后，将令牌和账户名导入到系统环境中，供后续使用：
``` bash
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>
```

## 3.6 检查Kubernetes集群环境
运行检查命令查看集群环境是否符合要求
```bash
flux check --pre
```

输出类似于：
```
► checking prerequisites
✔ Kubernetes 1.30.4-r4-30.0.14.11 >=1.30.0-0
✔ prerequisites checks passed
```

## 3.7 在集群中安装flux
运行[bootstrap](fluxcd.io/flux/installation/bootstrap/)命令
```bash
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=fleet-infra \
  --branch=main \
  --path=./clusters/my-cluster \
  --private=false \
  --token-auth \
  --personal
```
输出类似于：
```
► connecting to github.com
► cloning branch "main" from Git repository "https://github.com/HGZ-20/fleet-infra.git"
✔ cloned repository
► generating component manifests
✔ generated component manifests
✔ component manifests are up to date
✔ reconciled components
► determining if source secret "flux-system/flux-system" exists
► generating source secret
► applying source secret "flux-system/flux-system"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ sync manifests are up to date
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for GitRepository "flux-system/flux-system" to be reconciled
✔ GitRepository reconciled successfully
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
```
上述引导命令执行以下操作：
- 在您的 GitHub 账户中创建一个 git 仓库 fleet-infra
- 将 Flux 组件清单添加到仓库中
- 将 Flux 组件部署到您的 Kubernetes 集群中
- 配置 Flux 组件以跟踪仓库中的路径 /clusters/my-cluster/
- 启用`token-auth`后会使得使用个人访问令牌代替 SSH 部署密钥

## 3.8 克隆fleet-infra仓库
将fleet-infra仓库克隆到本地
```bash
git clone https://github.com/$GITHUB_USER/fleet-infra
cd fleet-infra
```

## 3.9 将podinfo仓库添加到Flux
使用[podinfo](github.com/stefanprodan/podinfo)作为部署示例，**注意一定要使用自己fork后的地址，不然无法通过修改git仓库内容来查看自动化部署的变化结果。**
1. 创建指向 podinfo 仓库 master 分支的 GitRepository 清单：
```bash
  flux create source git podinfo \
    --url=https://github.com/$GITHUB_USER/podinfo \
    --branch=master \
    --interval=1m \
    --export > ./clusters/my-cluster/podinfo-source.yaml
```
输出类似于：
```
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: podinfo
  namespace: flux-system
spec:
  interval: 1m
  ref:
    branch: master
  url: <Your GitHub repository URL>
```
2. 将 `podinfo-source.yaml` 文件提交并推送到 `fleet-infra` 仓库：
```bash
git add -A && git commit -m "Add podinfo GitRepository"
git push
```

## 3.10 修改podinfo应用部署配置
在部署应用前先修改fork的podinfo应用的配置内容
```bash
git clone https://github.com/$GITHUB_USER/podinfo.git
cd podinfo/kustomize
ls
```
可以看到四个配置文件如下：
```
deployment.yaml
hpa.yaml
kustomization.yaml
service.yaml
```
1. 首先，删除`hpa.yaml`，这个配置能够根据 CPU 使用率自动缩放 Pod 数量，但是我们后续观察集群是否根据git更新进行了自动化部署就是通过修改pod副本数量来观察的，为了避免收到影响，因此将改配置删除。
2. 修改`service.yaml`内容如下，使用负载均衡的访问类型，使应用服务暴露出去
```
apiVersion: v1
kind: Service
metadata:
  name: podinfo
spec:
  type: LoadBalancer
  selector:
    app: podinfo
  ports:
    - name: http
      port: 9898
      protocol: TCP
      targetPort: http
    - name: grpc
      port: 9999
      protocol: TCP
      targetPort: grpc
```
3. 修改`deployment.yaml`，在配置中添加具体的pod数量，添加`replicas: 2`配置，修改后文件内容如下：
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: podinfo
spec:
  replicas: 2
  minReadySeconds: 3
  revisionHistoryLimit: 5
  progressDeadlineSeconds: 60
  strategy:
    rollingUpdate:
      maxUnavailable: 0
    type: RollingUpdate
  selector:
    matchLabels:
      app: podinfo
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9797"
      labels:
        app: podinfo
    spec:
      containers:
      - name: podinfod
        image: ghcr.io/stefanprodan/podinfo:6.8.0
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 9898
          protocol: TCP
        - name: http-metrics
          containerPort: 9797
          protocol: TCP
        - name: grpc
          containerPort: 9999
          protocol: TCP
        command:
        - ./podinfo
        - --port=9898
        - --port-metrics=9797
        - --grpc-port=9999
        - --grpc-service-name=podinfo
        - --level=info
        - --random-delay=false
        - --random-error=false
        env:
        - name: PODINFO_UI_COLOR
          value: "#34577c"
        livenessProbe:
          exec:
            command:
            - podcli
            - check
            - http
            - localhost:9898/healthz
          initialDelaySeconds: 5
          timeoutSeconds: 5
        readinessProbe:
          exec:
            command:
            - podcli
            - check
            - http
            - localhost:9898/readyz
          initialDelaySeconds: 5
          timeoutSeconds: 5
        resources:
          limits:
            cpu: 2000m
            memory: 512Mi
          requests:
            cpu: 100m
            memory: 64Mi
        volumeMounts:
          - name: data
            mountPath: /data
      volumes:
        - name: data
          emptyDir: {}
```
4. 修改`kustomization.yaml`如下：
```
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
```
5. 修改后，添加相关的变动提交信息，并将修改的内容push到远程仓库中，注意一定要Push到远程仓库，flux是跟踪远程仓库地址来自动化部署的
```bash
git add -A && git commit -m "Modify application deployment configuration"
git push
```
## 3.11 部署podinfo应用
配置 Flux 以构建并应用位于 podinfo 仓库中的 kustomize 目录
1. 使用 flux create 命令创建应用 podinfo 部署的 Kustomization
```bash
# 注意，在fleet-infra仓库目录下，运行下面命令
flux create kustomization podinfo \
  --target-namespace=default \
  --source=podinfo \
  --path="./kustomize" \
  --prune=true \
  --wait=true \
  --interval=1m \
  --retry-interval=1m \
  --health-check-timeout=1m \
  --export > ./clusters/my-cluster/podinfo-kustomization.yaml
```

2. 将`Kustomization` 清单提交并推送到仓库：
```bash
git add -A && git commit -m "Add podinfo Kustomization"
git push
```

## 3.12 观察flux是否同步部署应用程序
1. 使用 `flux get` 命令监视 `podinfo` 应用
```bash
flux get kustomizations --watch
```
输出类似于：
```
NAME       	REVISION          	SUSPENDED	READY	MESSAGE                              
flux-system	main@sha1:42ea91f7	False    	True 	Applied revision: main@sha1:42ea91f7	
podinfo	master@sha1:30f508f0	False	True	Applied revision: master@sha1:30f508f0
```
2. 检查 podinfo 是否已在您的集群中部署：
```
kubectl -n default get deployments,services
```
输出类似于：
```
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/podinfo      2/2     2            2           18h

NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP                   PORT(S)                         AGE
service/podinfo         LoadBalancer   10.247.39.71    121.36.10.205,192.168.0.181   9898:32587/TCP,9999:32567/TCP   18h
```

同时也可以在集群控制台查看相关负载和服务是否正确运行，注意如果服务中podinfo的负载均衡访问没有自动化配置成功，需要手动更新一下，购买相关的资源后系统方可自动化配置，自动化部署结果如下：
![image](https://github.com/user-attachments/assets/9498a8f2-beb5-4937-bfef-6a5b840b9a2f)
![image](https://github.com/user-attachments/assets/f7ef03fd-a661-4896-86df-a94b1d12f07f)


## 3.13 验证flux根据git自动更新
上面我们已经实现了flux在华为云鲲鹏CCE集群上的自动部署，那么下面要验证下flux能根据git仓库自动更新部署。

首先，我们根据上面的部署可知，目前podinfo的pod数量为2，并且根据`flux get kustomizations --watch`的结果可以看到当前拉取的git仓库版本为`master@sha1:30f508f0`。

下面我们在本地修改`podinfo`仓库中`kustomize`文件夹中的`deployment.yaml`文件，将副本数量修改为`replicas: 3`，修改后内容如下：
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: podinfo
spec:
  replicas: 3
  minReadySeconds: 3
  revisionHistoryLimit: 5
  progressDeadlineSeconds: 60
  strategy:
    rollingUpdate:
      maxUnavailable: 0
    type: RollingUpdate
  selector:
    matchLabels:
      app: podinfo
  template:
    metadata:
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "9797"
      labels:
        app: podinfo
    spec:
      containers:
      - name: podinfod
        image: ghcr.io/stefanprodan/podinfo:6.8.0
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 9898
          protocol: TCP
        - name: http-metrics
          containerPort: 9797
          protocol: TCP
        - name: grpc
          containerPort: 9999
          protocol: TCP
        command:
        - ./podinfo
        - --port=9898
        - --port-metrics=9797
        - --grpc-port=9999
        - --grpc-service-name=podinfo
        - --level=info
        - --random-delay=false
        - --random-error=false
        env:
        - name: PODINFO_UI_COLOR
          value: "#34577c"
        livenessProbe:
          exec:
            command:
            - podcli
            - check
            - http
            - localhost:9898/healthz
          initialDelaySeconds: 5
          timeoutSeconds: 5
        readinessProbe:
          exec:
            command:
            - podcli
            - check
            - http
            - localhost:9898/readyz
          initialDelaySeconds: 5
          timeoutSeconds: 5
        resources:
          limits:
            cpu: 2000m
            memory: 512Mi
          requests:
            cpu: 100m
            memory: 64Mi
        volumeMounts:
          - name: data
            mountPath: /data
      volumes:
        - name: data
          emptyDir: {}
```
将本地修改内容提交到远程仓库
```
git add -A && git commit -m "update replica count from 2 to 3"
git push
```
继续监视podinfo应用状态：
```nash
flux get kustomizations --watch
```
发现结果为：
```
NAME       	REVISION          	SUSPENDED	READY	MESSAGE                              
flux-system	main@sha1:42ea91f7	False    	True 	Applied revision: main@sha1:42ea91f7	
podinfo	master@sha1:5d29acc2	False	True	Applied revision: master@sha1:5d29acc2
```
运行下面命令
```
kubectl -n default get deployments,services
```
结果为：
```
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/podinfo      3/3     3            3           18h

NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP                   PORT(S)                         AGE
service/podinfo         LoadBalancer   10.247.39.71    121.36.10.205,192.168.0.181   9898:32587/TCP,9999:32567/TCP   18h
```

可以发现，podinfo的应用pod数量已经根据我们的配置修改变更为了3，并且目前拉取的最新git代码已经从`master@sha1:30f508f0`变更为`master@sha1:5d29acc2`。**因此在华为云鲲鹏CCE集群中成功验证了flux能够自动化根据git仓库部署更新应用**
![image](https://github.com/user-attachments/assets/5bf6dad7-438a-4da7-90ca-8f435b863bde)

# 4 参考资料
1. [开源for Huawei介绍、环境搭建、示例项目、开发和部署指南](gitcode.com/HuaweiCloudDeveloper/OpenSourceForHuaweiWiki/overview)
3. [SPIRE适配GaussDB开源开发任务](https://bbs.huaweicloud.com/blogs/446798)
