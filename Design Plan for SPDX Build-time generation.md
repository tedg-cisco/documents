### Design Plan for Integrating Build-Time Generation of SPDX SBOMs in BES Build Pipeline

#### Objectives
- Add ability to generate Software Bill of Materials (SBOM) in SPDX format during BES Jenkins build process.
- Ensure the SBOM is generated, archived, and accessible post-build.
   - *TODO: Push to trusted repository of SBOMs?*
- Automate the process to ensure it runs with every build.

#### Prerequisites
- Jenkins installed and configured.
- A Jenkins Pipeline (either Declarative or Scripted) for the ```bes``` and ```requirements``` repositories.
- Syft installed on the Jenkins server or as a Docker image.
- Access to the source code repository.

#### Steps

1. **Install Syft on Jenkins Server:**
   - Ensure Syft is available on the Jenkins server. 
     - Install it directly or use it via a Docker container.
   - For direct installation, follow the [Syft installation instructions](https://github.com/anchore/syft#installation).

2. **Modify Jenkins Pipeline to Include Syft SBOM Generation:**
   - Update the bes `Jenkinsfile` to include steps for generating and archiving the SBOM.

##### Declarative Pipeline Example:

```groovy
pipeline {
    agent any

    stages {

        stage('Information') {
            steps {
                echo "=== syft SPDX ==="
                syft --version
            }
        }

        stage('Checkout') {
            steps {
                // Need clone of repositories to scan source
                checkout scm   // Replace with your checkout/clone command
            }
        }
        
        stage('Build') {
            steps {
                sh 'make build'
            }
        }

        stage('Generate SBOM') {
            steps {
                script {
                    // Generate SPDX v2.3 for both bes and requirements repositories
                    //    (Example pseudo-code, -not- tested or optimized)
                    def dirnames = ['bes-main', 'requirements-main']
                    for (dirname in dirnames) {
                        // Add $BUILD_NUMBER to SPDX JSON filename
                        sh "syft scan dir:${dirname} -o spdx-json@2.3 > ${dirname}.$BUILD_NUMBER.spdx.json"
                    }
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                // Generate SPDX v2.3 for both bes and requirements repositories
                //    (Example pseudo-code, -not- tested or optimized)
                def dirnames = ['bes-main', 'requirements-main']
                for (dirname in dirnames) {
                    // Add $BUILD_NUMBER to SPDX JSON filename
                    archiveArtifacts artifacts: '${dirname}.$BUILD_NUMBER.spdx.json', allowEmptyArchive: true
                }
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
            // Need clone of repositories to scan source
            checkout scm   // Replace with your checkout/clone command
        }

        stage('Build') {
            sh 'make build' // Replace with your build command
        }

        stage('Generate SBOM') {
            // Generate SPDX v2.3 for both bes and requirements repositories
            //    (Example pseudo-code, -not- tested or optimized)
            def dirnames = ['bes-main', 'requirements-main']
            for (dirname in dirnames) {
                // Add $BUILD_NUMBER to SPDX JSON filename
                sh "syft scan dir:${dirname} -o spdx-json@2.3 > ${dirname}.$BUILD_NUMBER.spdx.json"
            }
        }

        stage('Archive Artifacts') {
            //    (Example pseudo-code, -not- tested or optimized)
            def dirnames = ['bes-main', 'requirements-main']
            for (dirname in dirnames) {
                // Add $BUILD_NUMBER to SPDX JSON filename
                archiveArtifacts artifacts: '${dirname}.$BUILD_NUMBER.spdx.json', allowEmptyArchive: true
            }
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
            //    (Example pseudo-code, -not- tested or optimized)
            def dirnames = ['bes-main', 'requirements-main']
            for (dirname in dirnames) {
                // Add $BUILD_NUMBER to SPDX JSON filename
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v \$(pwd):/tmp anchore/syft:latest ${imageName} -o spdx-json@2.3 > ${dirname}.$BUILD_NUMBER.spdx.json"
            }
        }
    }
}
```

4. **Configuring Jenkins Job:**
   - Ensure the Jenkins job has the necessary permissions and access to Docker if running Syft in a container.
   - Install any required plugins, such as Docker Pipeline plugin, if you are using Docker.
   - *TBD: BES configuration already does this*

5. **Testing the Integration:**
   - Run a build and verify the SBOM is generated and archived.
   - Check the Jenkins console output for any errors or issues during the SBOM generation.

6. **Post-Build Actions:**
   - Consider adding notifications or other post-build actions if the SBOM generation fails.
      - *TODO: Add Try-Catch to display any error results to console/log?*
   - Ensure the SBOM is accessible from the Jenkins build artifacts for auditing and compliance purposes.

### Additional Considerations
- **Security:** Ensure that the Jenkins server and Docker daemon are secured to prevent unauthorized access.
   - *TBD: BES configuration already does this*
- **Performance:** Monitor the build times to ensure that the SBOM generation does not significantly impact build performance.
- **Maintenance:** Regularly update Syft to its latest version to benefit from improvements and security fixes.

By following this detailed plan, BES can successfully integrate build-time generation of Syft SPDX SBOMs into the Jenkins build pipeline, generating and archiving industry standard SPDX SBOMs for every build.


