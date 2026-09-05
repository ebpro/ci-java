# ci-java

Shared GitHub Actions workflows for the Java/Maven sample repos of the `ebpro` org
(lecture sample code used by the `notebook-*` / `lecture-*` repos).

Consumers pin a **tag** (e.g. `@v1.0.0`) — bump the tag in the wrapper to upgrade.

## Reusable workflows

### `sonarqube-maven.yml`

Builds, tests and analyzes a Maven project against the org SonarQube, with JaCoCo
coverage through the project's `-Psonar` Maven profile.

Caller repo requirements:

- Maven wrapper (`./mvnw`) at the repo root;
- a `sonar` Maven profile running JaCoCo (`prepare-agent` + XML `report`) and
  `sonar-maven-plugin` (see `references/starter-quarkus-rest/pom.xml`);
- a `sonar-project.properties` at the repo root (`sonar.projectKey` = Maven GA).

Wrapper example (`.github/workflows/sonarqube.yml` in the caller repo):

```yaml
name: SonarQube

on:
  workflow_dispatch:
  pull_request:
    branches: ['**']
  push:
    branches: ['**']

permissions:
  contents: read

concurrency:
  group: sonarqube-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

jobs:
  sonarqube:
    uses: ebpro/ci-java/.github/workflows/sonarqube-maven.yml@v1.0.0
    with:
      java-version: '21'
      sonar-main-branch: ${{ vars.SONAR_MAIN_BRANCH || 'develop' }}
      sonar-host-url: ${{ vars.SONAR_HOST_URL }}
      sonar-public-url: ${{ vars.SONAR_PUBLIC_URL }}
      sonar-runner: ${{ vars.SONAR_RUNNER }}
    secrets:
      SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      SONAR_URL: ${{ secrets.SONAR_URL }}
```

#### Inputs

| input | default | purpose |
| --- | --- | --- |
| `java-version` | `21` | Temurin JDK used to build and analyze |
| `sonar-main-branch` | `develop` | branch scanned as mainline (others go to `sonar.branch.name=…`) |
| `sonar-host-url` | — | cluster-local SonarQube URL (used when `sonar-runner` is set) |
| `sonar-public-url` | — | public SonarQube URL (fallback for hosted runners) |
| `sonar-runner` | — | `runs-on` label; empty = `ubuntu-latest` |
| `build-command` | `./mvnw -B clean verify sonar:sonar -Psonar` | build + analysis command |

#### Secrets

| secret | purpose |
| --- | --- |
| `SONAR_TOKEN` | SonarQube analysis token (org secret) |
| `SONAR_URL` | SonarQube base URL consumed by `sonar.host.url` in the Maven profile (org secret) |

## Org plumbing

- Org secrets (`--visibility all`): `SONAR_TOKEN`, `SONAR_URL`;
- Org vars: `SONAR_HOST_URL` (cluster-local), `SONAR_PUBLIC_URL`, `SONAR_RUNNER`
  (in-cluster self-hosted runner);
- Optional per-repo var `SONAR_MAIN_BRANCH` (overrides the `develop` default when
  the teaching mainline is `main`/`master`).
