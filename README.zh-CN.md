# Maven 手动发布 Action

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

[English](README.md) | 简体中文

一个全面的 GitHub Actions 工作流，用于自动化 Maven 项目发布，包含完整的版本管理、Maven Central 部署和文档发布。

## 🚀 功能特性

* 🔄 **自动化发布流程** - 从创建分支到清理的完整发布过程
* 📦 **Maven Central 部署** - 使用 GPG 签名部署到 Maven Central (Sonatype OSSRH)
* 📚 **文档发布** - 部署 Maven 站点到 GitHub Pages
* 🏷️ **版本管理** - 自动更新版本和分支管理
* 🎯 **里程碑管理** - 关闭当前里程碑并创建下一个版本里程碑
* 🧪 **测试支持** - 可选的测试执行，支持跳过配置
* 🔐 **安全签名** - 对所有构件进行 GPG 签名
* 📁 **多模块支持** - 通过工作目录配置支持子模块
* ⚙️ **高度可配置** - 自定义 Java 版本、Maven 参数、profiles 等

## 📋 前置条件

* 配置好 Maven Central 部署的 Maven 项目
* 用于签名构件的 GPG 密钥
* Maven Central (OSSRH) 账号和凭证
* 仓库启用 GitHub Actions
* 具有适当权限的 GitHub token

## ⚠️ 重要说明

**此 Action 不包含代码检出步骤**。您必须在使用此 Action 之前添加 `actions/checkout` 步骤：

```yaml
steps:
  - name: 检出代码
    uses: actions/checkout@v4
    with:
      fetch-depth: 0  # 生成发布说明所需
      token: ${{ secrets.GITHUB_TOKEN }}
  
  - name: 发布 Maven 项目
    uses: chensoul/maven-release-by-manual-action@main
    # ... 您的配置
```

**为什么不包含 checkout？**
- ✅ 允许您自定义检出行为（submodules、LFS 等）
- ✅ 避免重复的检出操作
- ✅ 遵循 GitHub Actions 复合 Action 的最佳实践
- ✅ 让您完全控制仓库状态

## 🎯 快速开始

### 基本用法

创建工作流文件 `.github/workflows/release.yml`：

```yaml
name: Release

on:
  workflow_dispatch:
    inputs:
      release-version:
        description: '发布版本 (如 1.0.0)'
        required: true
      next-version:
        description: '下一个开发版本 (如 1.0.1-SNAPSHOT)'
        required: true

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pages: write
      id-token: write
    
    steps:
      - name: 检出代码
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 生成发布说明所需
          token: ${{ secrets.GITHUB_TOKEN }}
      
      - name: 发布 Maven 项目
        uses: chensoul/maven-release-by-manual-action@main
        with:
          release-version: ${{ github.event.inputs.release-version }}
          next-version: ${{ github.event.inputs.next-version }}
          gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
          gpg-passphrase: ${{ secrets.GPG_PASSPHRASE }}
          maven-username: ${{ secrets.MAVEN_USERNAME }}
          maven-password: ${{ secrets.MAVEN_PASSWORD }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 高级配置

```yaml
- name: 发布 Maven 项目
  uses: chensoul/maven-release-by-manual-action@main
  with:
    # 版本配置
    release-version: '1.0.0'
    next-version: '1.0.1-SNAPSHOT'
    
    # Java 配置
    java-version: '17'
    java-distribution: 'temurin'
    
    # Maven Central 认证
    maven-username: ${{ secrets.MAVEN_USERNAME }}
    maven-password: ${{ secrets.MAVEN_PASSWORD }}
    
    # GPG 签名
    gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
    gpg-passphrase: ${{ secrets.GPG_PASSPHRASE }}
    
    # GitHub 配置
    github-token: ${{ secrets.GITHUB_TOKEN }}
    
    # 构建选项
    maven-args: '-B -U -ntp -T 1C'
    skip-tests: 'false'
    deploy-pages: 'true'
    
    # 高级选项
    working-directory: '.'
    maven-profiles: ''
    main-branch: 'main'
```

## 📝 输入参数

### 必需参数

| 参数 | 说明 | 示例 |
|------|------|------|
| `release-version` | 发布版本 | `1.0.0` |
| `next-version` | 下一个开发版本 | `1.0.1-SNAPSHOT` |
| `maven-username` | Maven Central 用户名 | 来自 secrets |
| `maven-password` | Maven Central 密码 | 来自 secrets |
| `gpg-private-key` | GPG 私钥 | 来自 secrets |
| `gpg-passphrase` | GPG 密钥密码 | 来自 secrets |
| `github-token` | GitHub token | `${{ secrets.GITHUB_TOKEN }}` |

### 可选参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `java-version` | 使用的 Java 版本 | `8` |
| `java-distribution` | Java 发行版 | `temurin` |
| `maven-args` | 额外的 Maven 参数 | `-B -U -ntp` |
| `maven-profiles` | 激活的 Maven profiles | `central` |
| `maven-server-id` | Maven 服务器 ID | `central` |
| `skip-tests` | 跳过测试 | `false` |
| `deploy-pages` | 部署到 GitHub Pages | `true` |
| `working-directory` | 工作目录 | `.` |

## 🔐 配置 Secrets

### 1. GPG 密钥设置

```bash
# 生成 GPG 密钥（如果还没有）
gpg --full-generate-key

# 导出私钥
gpg --armor --export-secret-keys YOUR_KEY_ID > private-key.asc

# 复制 private-key.asc 的内容到 GitHub Secrets
cat private-key.asc

# 上传公钥到密钥服务器
gpg --keyserver keyserver.ubuntu.com --send-keys YOUR_KEY_ID
gpg --keyserver keys.openpgp.org --send-keys YOUR_KEY_ID
```

添加到 GitHub Secrets：
- `GPG_PRIVATE_KEY`: `private-key.asc` 的内容
- `GPG_PASSPHRASE`: GPG 密钥的密码

### 2. Maven Central 设置

1. 在 https://central.sonatype.com 创建账号
2. 请求命名空间验证（如 `io.github.yourusername`）
3. 从账号中生成 User Token

添加到 GitHub Secrets：
- `MAVEN_USERNAME`: OSSRH 用户名或 token 用户名
- `MAVEN_PASSWORD`: OSSRH 密码或 token 密码

### 3. GitHub Token

默认的 `${{ secrets.GITHUB_TOKEN }}` 通常就足够了。确保工作流有以下权限：

```yaml
permissions:
  contents: write      # 用于创建 release 和推送提交
  pages: write         # 用于 GitHub Pages 部署
  id-token: write      # 用于 GitHub Pages 部署
```

## 📦 Maven POM 配置

您的 `pom.xml` 必须包含必需的元数据和插件。参见 [test-project/pom.xml](test-project/pom.xml) 获取完整示例。

### 基本配置

```xml
<project>
  <!-- 项目坐标 -->
  <groupId>io.github.yourusername</groupId>
  <artifactId>your-project</artifactId>
  <version>1.0.0</version>
  
  <!-- 必需元数据 -->
  <name>Your Project</name>
  <description>项目描述</description>
  <url>https://github.com/yourusername/your-project</url>
  
  <!-- 许可证 -->
  <licenses>
    <license>
      <name>Apache License, Version 2.0</name>
      <url>https://www.apache.org/licenses/LICENSE-2.0</url>
    </license>
  </licenses>
  
  <!-- 开发者 -->
  <developers>
    <developer>
      <name>Your Name</name>
      <email>your.email@example.com</email>
    </developer>
  </developers>
  
  <!-- SCM -->
  <scm>
    <connection>scm:git:git://github.com/yourusername/your-project.git</connection>
    <developerConnection>scm:git:ssh://github.com:yourusername/your-project.git</developerConnection>
    <url>https://github.com/yourusername/your-project</url>
  </scm>
  
  <!-- 分发管理 -->
  <distributionManagement>
    <repository>
      <id>central</id>
      <url>https://s01.oss.sonatype.org/service/local/staging/deploy/maven2/</url>
    </repository>
  </distributionManagement>
  
  <!-- Profiles -->
  <profiles>
    <profile>
      <id>central</id>
      <build>
        <plugins>
          <!-- GPG、Source 和 Javadoc 插件 -->
        </plugins>
      </build>
    </profile>
  </profiles>
</project>
```

## 🔄 发布流程

该 Action 执行以下步骤：

1. **验证输入** - 确保发布版本不是 SNAPSHOT
2. **检出仓库** - 获取完整的 git 历史
3. **设置 Java** - 配置带有 Maven 和 GPG 的 Java 环境
4. **创建发布分支** - 创建以发布版本命名的分支
5. **更新版本** - 在 POM 文件中设置发布版本
6. **构建和部署** - 构建、测试、签名并部署到 Maven Central
7. **创建标签** - 创建 git 标签和 GitHub release
8. **部署文档** - 发布 Maven 站点到 GitHub Pages（可选）
9. **更新主分支** - 设置下一个开发版本并合并回主分支
10. **管理里程碑** - 关闭当前里程碑，创建下一个

## 🎯 使用场景

### 从主分支发布

```yaml
on:
  workflow_dispatch:
    inputs:
      release-version:
        required: true
      next-version:
        required: true
```

### 使用自定义 Java 版本发布

```yaml
with:
  java-version: '21'
  java-distribution: 'temurin'
```

### 紧急修复跳过测试

```yaml
with:
  skip-tests: 'true'
```

### 多模块项目

```yaml
with:
  working-directory: 'my-module'
```

### 自定义 Maven Profiles

```yaml
with:
  maven-profiles: ',sign'
```

## 🔧 故障排除

### GPG 错误导致发布失败

- 确保 GPG 私钥包含 `BEGIN` 和 `END` 标记
- 验证密码正确
- 检查公钥是否已上传到密钥服务器

### Maven Central 部署失败

- 验证 OSSRH 凭证正确
- 确保命名空间已批准
- 检查所有必需的 POM 元数据是否存在
- 验证构件已签名

### 合并冲突

- 确保发布前主分支是最新的
- 解决任何可能冲突的待处理 PR

## 📊 与 maven-deploy-action 的比较

该 Action 基于 [maven-deploy-action](https://github.com/rosestack/maven-deploy-action) 的概念，但增加了：

- ✅ 发布分支管理
- ✅ 自动版本更新
- ✅ 下一个开发版本处理
- ✅ 发布分支合并和清理
- ✅ 里程碑管理集成

## 📄 许可证

Apache License 2.0 - 详见 [LICENSE](LICENSE) 文件。

## 🤝 贡献

欢迎贡献！请随时提交 issue 和 pull request。

## 🙏 致谢

- 灵感来自 [maven-deploy-action](https://github.com/rosestack/maven-deploy-action)
- 使用 [actions/checkout](https://github.com/actions/checkout)
- 使用 [actions/setup-java](https://github.com/actions/setup-java)
- 使用 [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)
- 使用 [rosestack/milestone-release-action](https://github.com/rosestack/milestone-release-action)

## 📧 支持

问题和咨询：
- GitHub Issues: https://github.com/chensoul/maven-release-by-manual-action/issues
- 电子邮件: ichensoul@gmail.com

