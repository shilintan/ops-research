# 目标

代码提交到git

提交代码之后部署到对应环境

# 创建namespace

```
kubectl create ns env
```

# 部署gitlab

https://github.com/shilintan/middleware-research

```
kubectl apply -f gitlab/gitlab.yaml
```

# 部署gitlab-runner

创建runner

http://gitlab-env.your-domain-name.com/admin/runners



配置gitlab-runner

拷贝 --token 的值

修改 `gitlab-runner/gitlab-runner/values.yaml`

```
runnerRegistrationToken: "上面的token值"
```



```
helm install gitlab-runner gitlab-runner/gitlab-runner/ --namespace env --create-namespace -f gitlab-runner/values.yaml
helm upgrade gitlab-runner gitlab-runner/gitlab-runner/ --namespace env --create-namespace -f gitlab-runner/values.yaml
helm uninstall gitlab-runner --namespace env
```

# 配置脚本仓库

将 https://github.com/shilintan/middleware-research/config_gitlab_runner 放到独立的仓库

需要手动执行的文件:

```
gitlab-runner
	config_gitlab_runner
		config
			auth-local
				config-common-env-java.yaml
				config-common-file-java.yaml
				loki-config.yaml
			auth-yidongyun
				config-common-env-java.yaml
				config-common-file-java.yaml
				loki-config.yaml
		manual
			local
				ingress.yaml
			yidongyun
				im-svc.yaml
				ingress.yaml
```

暂时依赖于宿主机的目录

```
mkdir -p /build/cache/config /build/cache/data
mkdir -p /build/cache/config/maven /build/cache/config/container /build/cache/config/k8s/template/java /build/cache/config/k8s/token/test/
mkdir -p /build/cache/data/maven /build/cache/data/container
mkdir -p /build/cache/data/repo_gen
```

创建k8s 专属账号

 k8s-sa-test.yaml

```
echo $(kubectl -n test get secret $(kubectl -n test get secret | grep gitlab-runner | awk '{print $1}') -o go-template='{{.data.token}}' | base64 -d)
```

生产环境需要创建专用openvpn

```
yidongyun-robot-k8s
```



#  java17-group类型项目流水线思路

## 原始的项目目录结构

```
framework
    amqp
        pom.xml
    base
        pom.xml
    elasticsearch
        pom.xml
   logs
        pom.xml
    mybatis
        pom.xml
    sensitive
        pom.xml
    pom.xml
gateway
    pom.xml
lilishop-sdk
    pom.xml
service
	auth-service
    	pom.xml
	broadcast-service
    	pom.xml
	consumer
    	pom.xml
	distribution-service
    	pom.xml
	goods-service
    	pom.xml
	im-service
    	pom.xml
	order-service
    	pom.xml
	payment-service
    	pom.xml
	promotion-service
    	pom.xml
	resource-service
    	pom.xml
	statistics-service
    	pom.xml
	supplier-service
    	pom.xml
	system-service
    	pom.xml
	user-service
    	pom.xml
    pom.xml
pom.xml
```

## ci的项目目录结构

```
framework
    amqp
        pom.xml
        .gitlab-ci.type
    base
        pom.xml
        .gitlab-ci.type
    elasticsearch
        pom.xml
        .gitlab-ci.type
    logs
        pom.xml
        .gitlab-ci.type
    mybatis
        pom.xml
        .gitlab-ci.type
    sensitive
        pom.xml
        .gitlab-ci.type
    pom.xml
    .gitlab-ci.type
gateway
    pom.xml
    .gitlab-ci.type
    .gitlab-ci.appname
sdk
    pom.xml
    .gitlab-ci.type
service
	auth-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	broadcast-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	consumer
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	distribution-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	goods-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	im-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	order-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	payment-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	promotion-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	resource-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	statistics-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	supplier-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	system-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
	user-service
    	pom.xml
    	.gitlab-ci.appname
        .gitlab-ci.type
    pom.xml
    .gitlab-ci.type
pom.xml
.gitlab-ci.yml
.gitlab-ci.type
```

## ci的项目目录结构文件内容

同一层的文件内容相同

### .gitlab-ci.yml

```
stages:
  - build-code
  - build-image
  - deploy-image


build code:
  stage: build-code
  image: maven:3.9-eclipse-temurin-17
  variables:
    GIT_STRATEGY: clone
  script:
    - export ci_type=java17-group
    - /bin/cp -rf /build/cache/config/gitlab-ci-out/"${ci_type}"/build-code.sh ci.sh
    - bash ci.sh

build image:
  stage: build-image
  image: quay.io/buildah/stable:v1.31
  variables:
    GIT_STRATEGY: clone
  script:
    - export ci_type=java17-group
    - /bin/cp -rf /build/cache/config/gitlab-ci-out/"${ci_type}"/build-image.sh ci.sh
    - bash ci.sh

deploy image:
  stage: deploy-image
  image: bitnami/kubectl:1.25
  script:
    - export ci_type=java17-group
    - /bin/cp -rf /build/cache/config/gitlab-ci-out/"${ci_type}"/deploy-image.sh ci.sh
    - bash ci.sh

```

### .gitlab-ci.type

```
group
```

### framework/.gitlab-ci.type

```
group
```

### framework/auth-service/.gitlab-ci.type

```
dependency
```

### sdk/.gitlab-ci.type

```
dependency-out
```

### gateway/.gitlab-ci.type

```
deploy
```

## gateway/.gitlab-ci.appname

```
service-shop-gateway
```

## service/auth-service/.gitlab-ci.type

```
deploy
```

## service/auth-service/.gitlab-ci.appname

```
service-auth-gateway
```

## 当需要添加项目时需要怎么做

如果是依赖项目则添加 `.gitlab-ci.type` 内容为 `dependency`

如果是部署项目则添加 `.gitlab-ci.type` 内容为 `deploy`, 添加 `.gitlab-ci.appname` 内容为 `service-shop-你的服务名称`

# 配置文件管理

特定于java项目, 其他类型的项目也类似

## env方式

```
spring.config.import=optional:file:env/.env[.properties]
```

研发维护:

```
application.properties
application-prod.properties
env/.env
```

运维临时使用:

```
configmap/
config/application-prod.properties
```

运维长期维护

```
configmap
.env
env: JAVA_OPTS
```

## nacos方式

每个环境部署一个nacos, 服务在运行时直接连接nacos

通过jasypt 工具生成密文, 代码集成jasypt包

运维维护jasypt密钥, 服务运行时注入jasypt 密钥

配置文件中通过ENC(密文)保护敏感信息



# gitflow流程

## 环境概览

开发环境

​	研发人员共用服务

测试环境

​	测试人员测试业务

灰度环境

​	部分用户、部分域名长期使用

​	生产环境临时使用

生产环境

​	全部用户使用的环境

## git分支流程

main分支作为起点

test, dev

依次合并dev>test>main

新功能在feature-<功能名>分支开发

修复bug在bugfix-<bug名>分支开发

基于main拉出来的tag作为生产环境发布代码, 生产环境镜像tag以git tag为准, deploy/svc version

