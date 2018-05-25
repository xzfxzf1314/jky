# Maven部署步骤

1. 外层根目录命令

```
mvn -U clean install
```

2. 主工程目录

```
mvn -e tomcat7:redeploy
```