# Maven Release by Manual Action

[![GitHub release](https://img.shields.io/github/v/release/rosestack/maven-release-by-manual-action)](https://github.com/rosestack/maven-release-by-manual-action/releases)
[![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/rosestack/maven-release-by-manual-action/ci.yml?branch=main)](https://github.com/rosestack/maven-release-by-manual-action/actions/workflows/test.yml)
[![License](https://img.shields.io/github/license/rosestack/maven-release-by-manual-action)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/rosestack/maven-release-by-manual-action)](https://github.com/rosestack/maven-release-by-manual-action/stargazers)

A comprehensive GitHub Actions workflow for automating Maven project releases with complete version management, Maven Central deployment, and documentation publishing.

## 🚀 Features

* 🔄 **Automated Release Workflow** - Complete release process from branch creation to cleanup
* 📦 **Maven Central Deployment** - Deploy artifacts with GPG signing to Maven Central (Sonatype OSSRH)
* 📚 **Documentation Publishing** - Deploy Maven site to GitHub Pages
* 🏷️ **Version Management** - Automatic version updates and branch management
* 🎯 **Milestone Management** - Close current milestone and create next version milestone
* 🧪 **Testing Support** - Optional test execution with configurable skip
* 🔐 **Secure Signing** - GPG signing of all artifacts
* 📁 **Multi-module Support** - Works with submodules via working directory configuration
* ⚙️ **Highly Configurable** - Customize Java version, Maven args, profiles, and more

## 📋 Prerequisites

* Maven project configured for Maven Central deployment
* GPG key for signing artifacts
* Maven Central (OSSRH) account and credentials
* GitHub Actions enabled on your repository
* GitHub token with appropriate permissions

## ⚠️ Important Notes

**This action does NOT include repository checkout**. You must add `actions/checkout` step before using this action:

```yaml
steps:
  - name: Checkout Repository
    uses: actions/checkout@v4
    with:
      fetch-depth: 0  # Required for generating release notes
      token: ${{ secrets.GITHUB_TOKEN }}
  
  - name: Release Maven Project
    uses: rosestack/maven-release-by-manual-action@main
    # ... your configuration
```

**Why checkout is not included?**
- ✅ Allows you to customize checkout behavior (submodules, LFS, etc.)
- ✅ Avoids redundant checkout operations
- ✅ Follows GitHub Actions best practices for composite actions
- ✅ Gives you full control over repository state

## 🎯 Quick Start

### Basic Usage

Create a workflow file `.github/workflows/release.yml`:

```yaml
name: Release

on:
  workflow_dispatch:
    inputs:
      release-version:
        description: 'Release version (e.g., 0.0.1)'
        required: true
      next-version:
        description: 'Next development version (e.g., 0.0.2-SNAPSHOT)'
        required: true

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pages: write
      id-token: write
    
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Required for release notes generation
          token: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Release Maven Project
        uses: rosestack/maven-release-by-manual-action@main
        with:
          release-version: ${{ github.event.inputs.release-version }}
          next-version: ${{ github.event.inputs.next-version }}
          gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
          gpg-passphrase: ${{ secrets.GPG_PASSPHRASE }}
          maven-username: ${{ secrets.MAVEN_USERNAME }}
          maven-password: ${{ secrets.MAVEN_PASSWORD }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Advanced Configuration

```yaml
- name: Checkout Repository
  uses: actions/checkout@v4
  with:
    fetch-depth: 0
    token: ${{ secrets.GITHUB_TOKEN }}

- name: Release Maven Project
  uses: rosestack/maven-release-by-manual-action@main
  with:
    # Version Configuration
    release-version: '0.0.1'
    next-version: '0.0.2-SNAPSHOT'
    
    # Java Configuration
    java-version: '17'
    java-distribution: 'temurin'
    
    # Maven Central Authentication
    maven-username: ${{ secrets.MAVEN_USERNAME }}
    maven-password: ${{ secrets.MAVEN_PASSWORD }}
    
    # GPG Signing
    gpg-private-key: ${{ secrets.GPG_PRIVATE_KEY }}
    gpg-passphrase: ${{ secrets.GPG_PASSPHRASE }}
    
    # GitHub Configuration
    github-token: ${{ secrets.GITHUB_TOKEN }}
    
    # Build Options
    maven-args: '-B -U -ntp -T 1C'
    skip-tests: 'false'
    deploy-pages: 'true'
    
    # Advanced Options
    working-directory: '.'
    maven-profiles: ''
    main-branch: 'main'
```

## 📝 Inputs

### Required Inputs

| Input | Description | Example |
|-------|-------------|---------|
| `release-version` | Release version | `0.0.1` |
| `next-version` | Next development version | `0.0.2-SNAPSHOT` |
| `maven-username` | Maven Central username | From secrets |
| `maven-password` | Maven Central password | From secrets |
| `gpg-private-key` | GPG private key | From secrets |
| `gpg-passphrase` | GPG key passphrase | From secrets |
| `github-token` | GitHub token | `${{ secrets.GITHUB_TOKEN }}` |

### Optional Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `java-version` | Java version to use | `17` |
| `java-distribution` | Java distribution | `temurin` |
| `maven-args` | Additional Maven arguments | `-B -U -ntp` |
| `maven-profiles` | Maven profiles to activate | `release` |
| `maven-server-id` | Maven server ID for authentication | `central` |
| `skip-tests` | Skip running tests | `false` |
| `deploy-pages` | Deploy to GitHub Pages | `true` |
| `working-directory` | Working directory | `.` |

## 🔐 Setting Up Secrets

### 1. GPG Key Setup

```bash
# Generate GPG key (if you don't have one)
gpg --full-generate-key

# Export private key
gpg --armor --export-secret-keys YOUR_KEY_ID > private-key.asc

# Copy the content of private-key.asc to GitHub Secrets
cat private-key.asc

# Upload public key to key servers
gpg --keyserver keyserver.ubuntu.com --send-keys YOUR_KEY_ID
gpg --keyserver keys.openpgp.org --send-keys YOUR_KEY_ID
```

Add to GitHub Secrets:
- `GPG_PRIVATE_KEY`: Content of `private-key.asc`
- `GPG_PASSPHRASE`: Your GPG key passphrase

### 2. Maven Central Setup

1. Create account at https://central.sonatype.com
2. Request namespace verification (e.g., `io.github.yourusername`)
3. Generate User Token from your account

Add to GitHub Secrets:
- `MAVEN_USERNAME`: Your OSSRH username or token username
- `MAVEN_PASSWORD`: Your OSSRH password or token password

### 3. GitHub Token

The default `${{ secrets.GITHUB_TOKEN }}` is usually sufficient. Ensure your workflow has these permissions:

```yaml
permissions:
  contents: write      # For creating releases and pushing commits
  pages: write         # For GitHub Pages deployment
  id-token: write      # For GitHub Pages deployment
```

## 📦 Maven POM Configuration

Your `pom.xml` must include required metadata and plugins. See the [test-project/pom.xml](test-project/pom.xml) for a complete example.

### Essential Configuration

```xml

<project>
    <!-- Project coordinates -->
    <groupId>io.github.yourusername</groupId>
    <artifactId>your-project</artifactId>
    <version>0.0.1</version>

    <!-- Required metadata -->
    <name>Your Project</name>
    <description>Project description</description>
    <url>https://github.com/yourusername/your-project</url>

    <!-- License -->
    <licenses>
        <license>
            <name>Apache License, Version 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0</url>
        </license>
    </licenses>

    <!-- Developers -->
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

    <distributionManagement>
        <!-- After using org.sonatype.central:central-publishing-maven-plugin, it can be ignored. This is only a prompt -->
        <snapshotRepository>
            <id>central</id>
            <url>https://central.sonatype.com/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>

    <!-- Profiles -->
    <profiles>
        <profile>
            <id>release</id>
            <build>
                <plugins>
                    <!-- GPG, Source, and Javadoc plugins -->
                </plugins>
            </build>
        </profile>
    </profiles>
</project>
```

## 🔄 Release Process

The action performs the following steps:

1. **Validate Inputs** - Ensure release version is not a SNAPSHOT
2. **Checkout Repository** - Fetch full git history
3. **Setup Java** - Configure Java environment with Maven and GPG
4. **Create Release Branch** - Create branch named after release version
5. **Update Version** - Set release version in POM files
6. **Build & Deploy** - Build, test, sign, and deploy to Maven Central
7. **Create Tag** - Create git tag and GitHub release
8. **Deploy Docs** - Publish Maven site to GitHub Pages (optional)
9. **Update Main Branch** - Set next development version and merge back
10. **Manage Milestone** - Close current milestone, create next one

## 🎯 Use Cases

### Release from Main Branch

```yaml
on:
  workflow_dispatch:
    inputs:
      release-version:
        required: true
      next-version:
        required: true
```

### Release with Custom Java Version

```yaml
with:
  java-version: '21'
  java-distribution: 'temurin'
```

### Skip Tests for Hotfix

```yaml
with:
  skip-tests: 'true'
```

### Multi-module Project

```yaml
with:
  working-directory: 'my-module'
```

### Custom Maven Profiles

```yaml
with:
  maven-profiles: 'release'
```

## 🔧 Troubleshooting

### Release Fails with GPG Error

- Ensure GPG private key includes `BEGIN` and `END` markers
- Verify passphrase is correct
- Check public key is uploaded to key servers

### Maven Central Deployment Fails

- Verify OSSRH credentials are correct
- Ensure namespace is approved
- Check all required POM metadata is present
- Verify artifacts are signed

### Merge Conflicts

- Ensure main branch is up to date before release
- Resolve any pending PRs that might conflict

## 📄 License

Apache License 2.0 - see [LICENSE](LICENSE) file for details.

## 🤝 Contributing

Contributions are welcome! Please feel free to submit issues and pull requests.

## 🙏 Acknowledgments

- Uses [actions/checkout](https://github.com/actions/checkout)
- Uses [actions/setup-java](https://github.com/actions/setup-java)
- Uses [peaceiris/actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)
- Uses [rosestack/milestone-release-action](https://github.com/rosestack/milestone-release-action)

## 📧 Support

For issues and questions:
- GitHub Issues: https://github.com/rosestack/maven-release-by-manual-action/issues
- Email: ichensoul@gmail.com
