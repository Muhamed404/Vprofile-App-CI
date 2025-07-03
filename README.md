Vprofile App: Continuous Integration (CI) Pipeline
This README outlines the Continuous Integration (CI) pipeline for the Vprofile application, based on the provided Jenkinsfile. The pipeline automates the process from code commit to artifact readiness and notification, ensuring code quality and build integrity.

🚀 CI Pipeline Flow Overview
The CI pipeline for the Vprofile App is designed to provide rapid feedback on code changes, ensure adherence to coding standards, perform comprehensive quality and security checks, and manage build artifacts. The process is triggered by developer code pushes and culminates in a validated artifact stored in Nexus, with real-time notifications provided via Slack.

🛠️ Pipeline Configuration and Stages
Global Tools and Environment Variables
The pipeline is configured with the following global tools and environment variables, ensuring consistent execution across stages:

JDK: JDK17

Maven: MAVEN3.9

SonarScanner: sonarscanner

SonarQube Server: sonarserver

Nexus Repository Details:

NEXUSIP: IP address of the Nexus server (e.g., 54.152.99.51)

NEXUSPORT: Port for Nexus (e.g., 8081)

SNAP_REPO: Snapshot repository name (e.g., vprofile-snapshot)

RELEASE_REPO: Release repository name (e.g., vprofile-release)

CENTRAL_REPO: Maven Central proxy repository name (e.g., vprofile-maven-central)

NEXUS_GRP_REPO: Nexus group repository name (e.g., vprofile-maven-group)

NEXUS_USER: Nexus deployment username (e.g., admin)

NEXUS_PASS: Nexus deployment password (e.g., admin123)

NEXUS_LOGIN: Jenkins credential ID for Nexus login (e.g., nexuslogin-ID)

Key Stages and Components
The pipeline executes through a series of sequential stages, each performing a specific set of tasks:

Build

Description: Compiles the Vprofile application source code and packages it into a .war artifact. Tests are skipped at this stage to ensure a fast build.

Command: mvn -s settings.xml -DskipTests install

Post-Stage Action: Upon successful build, the generated **/*.war artifact is archived by Jenkins, making it accessible for later steps or for download.

Test

Description: Executes the unit and integration tests defined for the Vprofile application. Test results are typically generated in Surefire report format.

Command: mvn -s settings.xml test

Checkstyle Analysis

Description: Performs static code analysis using Checkstyle to enforce coding standards and identify style violations.

Command: mvn -s settings.xml checkstyle:checkstyle

Sonar Analysis

Description: Integrates with SonarQube to perform a comprehensive static code analysis, identifying bugs, vulnerabilities, code smells, and technical debt. It also incorporates JUnit and JaCoCo reports for test coverage.

Tool: SonarScanner, connecting to the configured sonarserver.

Command: sonar-scanner with various project-specific properties (sonar.projectKey, sonar.projectName, sonar.sources, sonar.java.binaries, sonar.junit.reportsPath, sonar.jacoco.reportsPath, sonar.java.checkstyle.reportPaths).

Quality Gate

Description: Pauses the pipeline execution and queries the SonarQube server for the project's Quality Gate status. This ensures that only code meeting predefined quality standards proceeds.

Action: If the Quality Gate fails within a 10-minute timeout, the pipeline is aborted.

UploadArtifact

Description: Uploads the successfully built and validated .war artifact to the Nexus Repository Manager. The artifact is versioned using the Jenkins BUILD_ID and BUILD_TIMESTAMP for unique identification.

Tool: Nexus Artifact Uploader plugin.

Repository: vprofile-release repository on Nexus.

Artifact Details: artifactId: 'vproapp', file: 'target/vprofile-v2.war', type: 'war'.

Post-Pipeline Actions
Regardless of the outcome of the stages, the pipeline performs a final action:

Always: Slack Notification

Description: Sends a notification to a designated Slack channel (#all-devops) with the build's status (SUCCESS, FAILURE, ABORTED, etc.), job name, build number, and a direct link to the Jenkins build URL.

Coloring: The message color is dynamically set based on the build result using the COLOR_MAP ('good' for success, 'danger' for failure).

📊 Visual Flow Diagram
For a visual representation of this CI pipeline, please refer to the ci-vprofile.drawio.pdf diagram. It visually depicts the sequence of steps and the interaction between different tools.

⚙️ Technologies Used
Version Control: Git

CI Server: Jenkins

Build Automation: Apache Maven

Java Development Kit: JDK 17

Code Style Analysis: Checkstyle

Static Code Analysis: SonarQube

Artifact Repository: Nexus Repository Manager

Notifications: Slack

This pipeline ensures that every code change undergoes rigorous automated checks, leading to a high-quality and reliable Vprofile application artifact ready for further deployment
