# Maven Central Publish Action

A GitHub Action for publishing libraries to Maven Central using the Vanniktech Maven Publish Plugin.

This action automates:
- Publishing artifacts to Maven Central
- Generating and publishing documentation with Dokka
- Creating GitHub releases
- Updating version in gradle.properties

## Usage

```yaml
name: Publish Release

on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to publish'
        required: true

jobs:
  publish:
    runs-on: macos-latest
    permissions:
      contents: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Publish to Maven Central
        uses: yourusername/maven-central-publish@v1
        with:
          version: ${{ github.event.inputs.version }}
          maven-central-username: ${{ secrets.MAVEN_CENTRAL_USERNAME }}
          maven-central-password: ${{ secrets.MAVEN_CENTRAL_PASSWORD }}
          gpg-key-id: ${{ secrets.GPG_SIGNING_KEY_ID }}
          gpg-key: ${{ secrets.GPG_SIGNING_KEY }}
          gpg-passphrase: ${{ secrets.GPG_PASSPHRASE }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

## Prerequisites

1. [Central Portal account](https://central.sonatype.org/register/central-portal/#create-an-account)
2. [Registered namespace](https://central.sonatype.org/register/namespace/)
3. GPG key for signing artifacts
4. [Vanniktech Maven Publish Plugin](https://vanniktech.github.io/gradle-maven-publish-plugin/central/) configured in your project

## Required Secrets

Configure these secrets in your repository:

- `MAVEN_CENTRAL_USERNAME`: Your Maven Central username
- `MAVEN_CENTRAL_PASSWORD`: Your Maven Central password/token
- `GPG_SIGNING_KEY_ID`: Your GPG key ID (last 8 characters of key fingerprint)
- `GPG_SIGNING_KEY`: Your ASCII-armored GPG private key
- `GPG_PASSPHRASE`: Your GPG key passphrase (if applicable)
- `GITHUB_TOKEN`: GitHub automatically provides this secret in workflows

These secrets must be passed to the action explicitly in your workflow as shown in the example above.

> **Note**: You'll need to set the `permissions: contents: write` in your job if creating GitHub releases and publishing documentation.

To export your GPG key in ASCII-armored format:
```bash
gpg --export-secret-keys --armor KEY_ID
```

## Inputs

| Input                      | Description                      | Required | Default                                          |
|----------------------------|----------------------------------|----------|--------------------------------------------------|
| `version`                  | Version to publish               | Yes      | N/A                                              |
| `maven-central-username`   | Maven Central username           | Yes      | N/A                                              |
| `maven-central-password`   | Maven Central password/token     | Yes      | N/A                                              |
| `gpg-key-id`               | GPG signing key ID               | Yes      | N/A                                              |
| `gpg-key`                  | GPG signing key (ASCII-armored)  | Yes      | N/A                                              |
| `gpg-passphrase`           | GPG key passphrase               | Yes      | `''`                                             |
| `github-token`             | GitHub token for releases & docs | Yes      | N/A                                              |
| `java-version`             | Java version                     | No       | `17`                                             |
| `java-distribution`        | Java distribution                | No       | `zulu`                                           |
| `publish-dokka-docs`       | Publish Dokka docs               | No       | `true`                                           |
| `create-github-release`    | Create GitHub release            | No       | `true`                                           |
| `update-gradle-properties` | Update gradle.properties         | No       | `true`                                           |
| `working-directory`        | Project directory                | No       | `.`                                              |
| `gradle-publish-task`      | Gradle publish task              | No       | `publishAllPublicationsToMavenCentralRepository` |
| `dokka-html-task`          | Documentation task               | No       | `dokkaHtml`                                      |
| `docs-folder`              | Docs output folder               | No       | `build/dokka/html`                               |
| `sonatype-host`            | Sonatype host                    | No       | `CENTRAL_PORTAL`                                 |
| `automatic-release`        | Auto-release from staging        | No       | `true`                                           |

## Plugin Configuration

Add the Vanniktech Maven Publish Plugin to your project:

```kotlin
// build.gradle.kts
plugins {
    id("com.vanniktech.maven.publish") version "0.25.3"
}

// Configure the plugin
mavenPublishing {
    publishToMavenCentral(SonatypeHost.CENTRAL_PORTAL)
    signAllPublications()
    
    pom {
        // Your POM configuration
    }
}
```

Or using `gradle.properties`:

```properties
GROUP=com.example.mylibrary
POM_ARTIFACT_ID=mylibrary-runtime
# VERSION_NAME will be set by the GitHub Action
POM_NAME=My Library
POM_DESCRIPTION=A description of the library
POM_URL=https://github.com/username/mylibrary/
# License information
POM_LICENSE_NAME=The Apache Software License, Version 2.0
POM_LICENSE_URL=https://www.apache.org/licenses/LICENSE-2.0.txt
POM_LICENSE_DIST=repo
# SCM information
POM_SCM_URL=https://github.com/username/mylibrary/
POM_SCM_CONNECTION=scm:git:git://github.com/username/mylibrary.git
POM_SCM_DEV_CONNECTION=scm:git:ssh://git@github.com/username/mylibrary.git
# Developer information
POM_DEVELOPER_ID=username
POM_DEVELOPER_NAME=Your Name
POM_DEVELOPER_URL=https://github.com/username/
```

## Publishing Snapshots

To publish a snapshot version, use a version ending with `-SNAPSHOT`. The action will publish it to the snapshot repository.

## License

MIT License

---

> \[!Info] For detailed information about all configuration options,
> please visit the [Vanniktech Maven Publish Plugin documentation](https://vanniktech.github.io/gradle-maven-publish-plugin/central/).
> This documentation includes advanced settings for POM configuration, publication targets, and signing requirements.