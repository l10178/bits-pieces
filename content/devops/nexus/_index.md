---
title: 'Nexus3 批量导入'
date: 2021-03-19T23:54:37+08:00
draft: false
---

应用场景：

1. 从一个可联网镜像仓库迁移到另一个无法联网的镜像仓库，涉及 Java 和 JavaScript 依赖包。
2. 源镜像仓库比较大（10T），新镜像仓库只需要部分依赖，不需要全量导入。

## Nexus 配置 Repository

在 nexus 上创建 Java 和 NPM Repository。

注意：

1. 创建时类型选择 hosted。
2. 对于 Java Repository 可根据实际情况将 Version Policy 选择 mixed。
3. Deployment Policy 选择 Allow redeploy，便于后续重复导入。

## 导入 Java 依赖包

1. 准备 Java 依赖包：将本地 maven 或 gradle 的存储目录(默认$USER/.m2/repository)清空，重新执行命令，获取干净的 repository。
2. 从 [nexus-repository-import-scripts][] 获取 mavenimport.sh 文件。
3. 将 mavenimport.sh copy 到 repository 文件夹内
4. 执行以下命令开始导入，注意换成自己的 admin 密码，修改端口和 Repository。

   ```bash
     cd repository && chmod +x mavenimport.sh
     ./mavenimport.sh -u admin -p admin123 -r http://127.0.0.1:8081/repository/your-repo-name/
   ```

5. 以上命令开始导入，可能时间较长，可以去界面上刷新下 jdf 的 repository 看看有没有导进去。

## 导入 NPM 依赖包

导入前端 npm 包就是先下载好各个包的 tgz 文件，然后根据 tgz 文件执行 npm publish。

1. 准备前端依赖包：使用这个工具 [node-tgz-downloader](https://github.com/Meir017/node-tgz-downloader)，根据他的说明，先下载好各个 js 的 `.tgz`文件，默认下载到一个叫 tarballs 的文件夹内，下载成功后大致结果如下，如果 nodejs 版本比较低可能下载失败，可以尝试使用低版本的 node-tgz-downloader。

   ```console
   tarballs
   ├── @babel
   │   ├── code-frame
   │   │   ├── code-frame-7.12.11.tgz
   │   │   └── code-frame-7.16.0.tgz
   │   ├── compat-data
   │   │   └── compat-data-7.16.4.tgz
   │   ├── core
   │   │   └── core-7.16.0.tgz
   │   ├── generator
   │   │   └── generator-7.16.0.tgz
   │   ├── helper-annotate-as-pure
   │   │   └── helper-annotate-as-pure-7.16.0.tgz
   ```

2. 从 [nexus-repository-import-scripts][] 获取 npmimport.sh 文件。
3. 将 npmimport.sh copy 到 tarballs 文件夹内。
4. 登录 npm 账户，开始执行导入。

   ```bash
    npm config set registry=http://127.0.0.1:8081/repository/your-npm-repo/
    npm adduser --registry=http://127.0.0.1:8081/repository/your-npm-repo/
    cd tarballs && chmod +x npmimport.sh
    ./npmimport.sh -r http://127.0.0.1:8081/repository/your-npm-repo/
   ```

5. 以上命令开始导入，可能时间较长，可以去界面上刷新下 npm 的 repository 看看有没有导进去。

## 客户端使用

Maven 修改 setting.xml，使用私有仓库。

另外因为部分包不受自己控制，可能缺少 metadata，执行 `mvn package` 等命令时，注意增加 `-c` 参数忽略 checksum。

如：`mvn -c clean package -Dmaven.test.skip=true`

[nexus-repository-import-scripts]: https://github.com/sonatype-nexus-community/nexus-repository-import-scripts
