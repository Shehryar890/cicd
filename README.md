This project was taken from a GitHub repository and used to build and demonstrate a complete Jenkins CI/CD pipeline.

Whenever a change is committed and pushed to the main branch, GitHub triggers the Jenkins pipeline through a webhook.

The first step of the pipeline is to fetch the latest source code from GitHub.

After getting the source code, Jenkins builds the project using Maven. The Maven build contains three main steps: `clean`, `compile`, and `test`. The required Java and Maven versions are configured in Jenkins according to the versions used to build this project.

After the build and tests are completed successfully, Jenkins runs a SonarQube analysis on the source code. SonarQube is used to check the code quality, bugs, code smells, vulnerabilities, and other issues in the project.

A SonarQube quality gate is also configured so that the pipeline only continues when the required quality conditions are satisfied.

After passing the quality checks, Jenkins builds a Docker image of the application.

The Docker image is then started as a container so that the application can be tested in a real running environment.

For dynamic security testing, a DAST scan is performed against the running application using the configured DAST tool. This allows the pipeline to test the application from the outside and identify possible security issues in the running application.

If the required build, testing, quality, and security checks pass successfully, the Docker image is pushed to Docker Hub.

Docker Hub is being used initially as the container registry, but the same pipeline can later be configured to use a private container registry.

The final part of the pipeline will be the deployment stage. The verified Docker image from the container registry will be taken and deployed to the required environment.

The deployment environment can later be a Docker server, virtual machine, Kubernetes cluster, or any other infrastructure depending on the deployment architecture.

The overall flow of the pipeline is:

GitHub → Jenkins → Maven Build & Test → SonarQube Analysis → Quality Gate → Docker Image Build → Run Container → DAST Security Test → Security Gate → Push Docker Image → Deployment

The deployment part will be implemented separately after completing the CI and security pipeline.
