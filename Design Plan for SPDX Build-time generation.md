### Design Plan for Integrating Build-Time Generation of Syft SPDX SBOMs in Jenkins Build Pipeline

#### Objectives
- Integrate Syft to generate Software Bill of Materials (SBOM) in SPDX format during the Jenkins build process.
- Ensure the SBOM is generated, archived, and accessible post-build.
- Automate the process to ensure it runs with every build.

#### Prerequisites
- Jenkins installed and configured.
- A Jenkins Pipeline (either Declarative or Scripted) for the web server project.
- Syft installed on the Jenkins server or as a Docker image.
- Access to the source code repository.

#### Steps

1. **Install Syft on Jenkins Server:**
   - Ensure Syft is available on the Jenkins server. You can either install it directly or use it via a Docker container.
   - For direct installation, follow the [Syft installation instructions](https://github.com/anchore/syft#installation).

2. **Modify Jenkins Pipeline to Include Syft SBOM Generation:**
   - Update your Jenkinsfile to include steps for generating and archiving the SBOM.

##### Declarative Pipeline Example:

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'make build' // Replace with your build command
            }
        }

        stage('Generate SBOM') {
            steps {
                script {
                    def imageName = "my-webserver:latest" // Replace with your image name
                    sh "syft ${imageName} -o spdx-json > sbom.spdx.json"
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'sbom.spdx.json', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
```

##### Scripted Pipeline Example:

```groovy
node {
    try {
        stage('Checkout') {
            checkout scm
        }

        stage('Build') {
            sh 'make build' // Replace with your build command
        }

        stage('Generate SBOM') {
            def imageName = "my-webserver:latest" // Replace with your image name
            sh "syft ${imageName} -o spdx-json > sbom.spdx.json"
        }

        stage('Archive Artifacts') {
            archiveArtifacts artifacts: 'sbom.spdx.json', allowEmptyArchive: true
        }
    } finally {
        cleanWs()
    }
}
```

3. **Running Syft in Docker:**
   - If Syft is not installed directly on the Jenkins server, use it via a Docker container.
   - Modify the SBOM generation stage to use Docker.

```groovy
stage('Generate SBOM') {
    steps {
        script {
            def imageName = "my-webserver:latest" // Replace with your image name
            sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v \$(pwd):/tmp anchore/syft:latest ${imageName} -o spdx-json > sbom.spdx.json"
        }
    }
}
```

4. **Configuring Jenkins Job:**
   - Ensure the Jenkins job has the necessary permissions and access to Docker if running Syft in a container.
   - Install any required plugins, such as Docker Pipeline plugin, if you are using Docker.

5. **Testing the Integration:**
   - Run a build and verify the SBOM is generated and archived.
   - Check the Jenkins console output for any errors or issues during the SBOM generation.

6. **Post-Build Actions:**
   - Consider adding notifications or other post-build actions if the SBOM generation fails.
   - Ensure the SBOM is accessible from the Jenkins build artifacts for auditing and compliance purposes.

### Additional Considerations
- **Security:** Ensure that the Jenkins server and Docker daemon are secured to prevent unauthorized access.
- **Performance:** Monitor the build times to ensure that the SBOM generation does not significantly impact build performance.
- **Maintenance:** Regularly update Syft to its latest version to benefit from improvements and security fixes.

By following this detailed plan, you can successfully integrate build-time generation of Syft SPDX SBOMs into your Jenkins build pipeline, ensuring comprehensive software composition analysis and compliance with industry standards.
