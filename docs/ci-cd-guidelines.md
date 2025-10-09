# CI/CD Guidelines for Java Projects on GitHub

This guide outlines a practical, secure CI/CD pipeline for Java projects hosted on GitHub. It follows four principal stages requested in the issue: Build and test; Unit test + Sonar analysis; Package or Docker image; and Upload to GitHub Registry.

Key goals
- Fast feedback (incremental builds, caching)
- Reproducible builds (pinned actions, deterministic JDK)
- Security by default (least-privilege tokens, minimal base images)
- Vendor-agnostic (supports Maven and Gradle)

Pipeline overview (GitHub Actions)
1) Build and Test
   - Trigger: pull_request and push
   - JDK 21 (adjust as needed)
   - Matrix for Maven/Gradle (optional)
   - Cache dependencies (actions/setup-java cache)
   - Run unit tests with coverage

2) Unit Tests + Sonar
   - If SonarCloud or SonarQube is configured, run analysis after tests
   - Requires secrets: SONAR_TOKEN and either SONAR_ORGANIZATION (SonarCloud) or SONAR_HOST_URL (SonarQube)
   - Upload coverage to Sonar (Jacoco by default)

3) Package or Docker Image
   - Produce JAR/WAR and upload as a workflow artifact
   - Optionally build a Docker image with Buildx (multi-arch optional) using a small JRE base image

4) Upload to GitHub Registry (GHCR)
   - Log in with GITHUB_TOKEN (packages: write)
   - Push image to ghcr.io/<owner>/<repo>:<sha> and optionally :latest on default branch

Repository prerequisites
- Java build: Maven (pom.xml) or Gradle (gradle.*)
- Tests produce Jacoco coverage (recommended defaults)
- Optional: sonar-project.properties at the repo root (template below)
- Permissions: In the workflow, set permissions for contents: read and packages: write

Secrets & variables
- SONAR_TOKEN: Generated from SonarCloud/SonarQube, least scopes
- SONAR_ORGANIZATION: For SonarCloud only
- SONAR_HOST_URL: For self-hosted SonarQube, e.g. https://sonar.example.com
- REGISTRY_IMAGE_NAME (optional): Override image name if different from repo

Quality gates and PR decoration
- For SonarCloud, enable PR decoration to block merges on failing quality gate
- Use branch protection in GitHub to require workflow checks and code scanning

Tips and recommendations
- Pin action versions with SHAs in long-lived enterprises; tags are used here for readability
- Use Dependabot for actions and Maven/Gradle updates
- Keep Dockerfiles minimal; prefer distroless or alpine JRE where compatible
- Publish SBOMs (cyclonedx-maven-plugin or gradle-cyclonedx-plugin) and attach as artifact
- If building multi-module, consider separate jobs with needs and artifact passing

Sonar template (optional minimal)
```
# sonar-project.properties (place at repository root)
sonar.projectKey=<org>_<repo>
sonar.organization=<your-sonarcloud-org>
sonar.java.binaries=**/target/classes,**/build/classes
sonar.sources=.
sonar.exclusions=**/target/**,**/build/**
sonar.junit.reportPaths=**/target/surefire-reports,**/build/test-results/test
sonar.coverage.jacoco.xmlReportPaths=**/target/site/jacoco/jacoco.xml,**/build/reports/jacoco/test/jacocoTestReport.xml
```

How to use
- Commit the workflow file from this guide to .github/workflows/ci.yml
- Add required secrets in GitHub: Settings → Secrets and variables → Actions
- If pushing images, enable packages: write for GITHUB_TOKEN in the workflow
- Inspect run logs and adjust Java version/build tool as needed
