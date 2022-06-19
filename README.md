# 自动构建部署腾讯云函数

## 创建子账户获取认证凭证

> <https://cloud.tencent.com/document/product/583/44786>

```shell
# .env
TENCENT_SECRET_ID=xxxxxxxxxx #您账号的 SecretId
TENCENT_SECRET_KEY=xxxxxxxx #您账号的 SecretKey
```

### 配置 `Github secrets` 凭证

- `APPNAME` 应用名称 `hello`
- `INSTANCENAME` 实例名称 `web-server`
- `REGION` 地域 `ap-guangzhou`
- `RUNTIME` 运行时 `Go1`
- `SECRETID` 凭证ID `xxxx`
- `SECRETKEY` 凭证Key `xxxx`

## 自动部署

> <https://cloud.tencent.com/document/product/1154/47290>

### 1、构建二进制

```shell
docker build -t scf_app . -f ./SCF/CustomRuntime/Dockerfile.build
docker create --name extract scf_app
docker cp extract:/app app
```

### 2、自定义运行时

```shell
touch ./app/scf_bootstrap && chmod +x ./app/scf_bootstrap
cat > ./app/scf_bootstrap<<EOF
#!/usr/bin/env bash
# export LD_LIBRARY_PATH=/opt/swift/usr/lib:${LD_LIBRARY_PATH}
./Run serve --env production --hostname 0.0.0.0 --port 9000
EOF
```

### 3、部署

```shell
npm install -g serverless
npm install -g @slsplus/cli

slsJson=$(cat ./SCF/CustomRuntime/sls.json)
slsplus parse --output --auto-create --sls-options="$slsJson" && cat serverless.yml

sls deploy --force --debug
```

- 配置

```yml
# serverless.yml

#应用组织信息
app: '' # 应用名称。留空则默认取当前组件的实例名称为app名称。
stage: '' # 环境名称。默认值是 dev。建议使用${env.STAGE}变量定义环境名称

#组件信息
component: scf # (必选) 组件名称，在该实例中为scf
name: scfdemo # (必选) 组件实例名称。

#组件参数配置
inputs:
  name: scfdemo # 云函数名称，默认为 ${name}-${stage}-${app}
  namespace: default
  role: exRole # 云函数执行角色
  # 1. 默认写法，新建特定命名的 cos bucket 并上传
  src: ./src
```

> 详情参考 `.github/workflows/deploy-sls.yml`
> <https://github.com/serverless-components/tencent-scf/blob/master/docs/configure.md>
