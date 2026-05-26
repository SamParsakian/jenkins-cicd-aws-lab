# Jenkins CI/CD on AWS — Lab Overview

This repository records how a **Jenkins CI/CD lab** was set up on **AWS EC2** for a **Java Maven** application hosted on GitHub. The Jenkins controller was used to run builds and tests; **SonarQube** was added for code quality checks; **Nexus** was used to store snapshot artifacts.

**Sample application:** [jenkins-ci-sample-app](https://github.com/SamParsakian/jenkins-ci-sample-app)  
**Full walkthrough (Steps 1–69 and screenshots):** [SETUP-REPORT.md](SETUP-REPORT.md)

---

## What was built

| Component | Role |
|-----------|------|
| `ci-jenkins-controller` | Jenkins LTS, Freestyle job, Pipeline job (port 8080) |
| `sonarqube-ci-server` | SonarQube Community Edition (port 9000) |
| `nexus-ci-server` | Nexus Repository OSS (port 8081) |

All three servers were configured with **Ubuntu** in the same **AWS VPC** (`eu-north-1`). Jenkins was set up to reach SonarQube and Nexus over **private IPs**; **public IPs** were used from a browser for the web UIs.

```text
GitHub (jenkins-ci-sample-app)
        |
        v
+---------------------+
| Jenkins controller  |
+----------+----------+
           |
     +-----+-----+
     |           |
     v           v
+------------+ +------------+
| SonarQube  | | Nexus OSS  |
+------------+ +------------+
```

---

## Jenkins jobs

| Job | Type | Purpose |
|-----|------|---------|
| `build-maven-sample-app` | Freestyle | `mvn clean package`, archive `target/*.jar` |
| `pipeline-maven-sample-app` | Pipeline | Root `Jenkinsfile` from GitHub — full CI flow |

**Global tools:** `Maven-3.9`, `JDK-21`, `SonarQube-Scanner`  
**Credential IDs (values stored only in Jenkins):** `sonarqube-token`, `nexus-admin-creds`

---

## Pipeline stages

The **`pipeline-maven-sample-app`** job was wired to the root **`Jenkinsfile`** in the sample app repository. The pipeline was defined with six stages:

1. **Checkout** — the repository was checked out from GitHub  
2. **Build and Test** — `mvn clean package` was run  
3. **Run App Smoke Test** — the packaged JAR was executed with `java -jar` (CLI output in the build log)  
4. **SonarQube Analysis** — `sonar-scanner` was run with main and test sources kept separate  
5. **Archive Artifact** — `target/*.jar` was stored in Jenkins  
6. **Upload to Nexus** — `mvn deploy:deploy-file` published to `jenkins-maven-snapshots`

```text
GitHub → Jenkins → Maven build/test → smoke test → SonarQube → archive → Nexus
```

SonarQube was configured with project key **`jenkins-ci-sample-app`**. Snapshot artifacts were published to **`jenkins-maven-snapshots`** at `com/mycompany/app/jenkins-ci-sample-app/1.0-SNAPSHOT`.

---

## Repository contents

| Path | Description |
|------|-------------|
| [SETUP-REPORT.md](SETUP-REPORT.md) | Step-by-step setup report (69 steps) with screenshots from EC2, Jenkins, SonarQube, and Nexus |
| [screenshots/](screenshots/) | 30 images referenced in the report (sensitive values redacted where needed) |

---

## Outcomes

- End-to-end CI was verified on the CLI Maven app (build, test, quality gate, artifact storage).  
- The SonarQube Quality Gate **Passed** after separate main/test source paths and Java libraries were configured for the scanner.  
- Nexus snapshot upload was corrected by replacing REST `curl` with **`mvn deploy:deploy-file`**.

Commands, console settings, and screenshots for each phase are documented in **[SETUP-REPORT.md](SETUP-REPORT.md)**.
