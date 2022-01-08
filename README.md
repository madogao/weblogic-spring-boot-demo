##### 编译
```
mvn clean package
```
##### 构建镜像
1. 下载官方最新版本 imagetool 工具：
https://github.com/oracle/weblogic-image-tool/releases
2. 下载最新版本 weblogic-deploy-tooling：
https://github.com/oracle/weblogic-deploy-tooling/releases/download/weblogic-deploy-tooling-1.8.1/weblogic-deploy.zip
3. 将 weblogic-deploy-tool 添加至缓存:
```
imagetool cache addInstaller --type wdt --version 0.22 --path /home/acmeuser/cache/weblogic-deploy.zip
```
4. 组织archive目录，并压缩为 domain.zip：
├── model
│   └── model.yaml
└── wlsdeploy
    └── applications
        └── app.war
其中app.war即编译后的war包，model.yaml内容：
```
domainInfo:
  AdminUserName: 'weblogic'
  AdminPassword: 'Welcome1'
  ServerStartMode: 'dev'

topology:
  Name: 'domain1'
  AdminServerName: "admin-server"
  Cluster:
    "cluster-1":
      DynamicServers:
        ServerTemplate:  "cluster-1-template"
        ServerNamePrefix: "managed-server"
        DynamicClusterSize: 2
        MaxDynamicClusterSize: 2
        CalculatedListenPorts: false
  Server:
    "admin-server":
      ListenPort: 7001
  ServerTemplate:
    "cluster-1-template":
      Cluster: "cluster-1"
      ListenPort: 8001
appDeployments:
  Application:
    demo:
      SourcePath: wlsdeploy/applications/app.war
      Target: 'cluster-1'
      ModuleType: war
```
4. 构建最终镜像：
```
imagetool create --fromImage dwgao/weblogic:12.2.1.4 --tag wls:12.2.1.4  --version 12.2.1.4 --wdtVersion 1.8.1 --wdtArchive domain.zip --wdtDomainHome /u01/oracle/user_projects/domains/domain1
```
