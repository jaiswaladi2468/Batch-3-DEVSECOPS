# NEXUS3

Nexus 3 is a popular artifact repository manager used in software development and DevOps workflows. It is designed to store and manage binary artifacts such as JAR files, Docker images, npm packages, and more. Nexus 3 is developed by Sonatype and provides a centralized location for teams to store, retrieve, and manage dependencies and artifacts in a secure and efficient manner. Below, I'll explain the key features and components of Nexus 3 in detail:

1. **Artifact Repository Manager**:
   - Nexus 3 primarily functions as an artifact repository manager, allowing organizations to store and manage various types of binary artifacts. These artifacts can include libraries, dependencies, and build outputs required for software development projects.

2. **Support for Multiple Repository Formats**:
   - Nexus 3 supports a wide range of repository formats, including:
     - **Maven**: For Java-based projects, Maven Central compatibility.
     - **Docker**: Container images and registries.
     - **npm**: Node.js package manager for JavaScript and Node.js projects.
     - **NuGet**: Package manager for .NET applications.
     - **Raw**: Generic storage for any binary format.
     - **Yum**: For RPM package management in Linux environments.
     - **PyPI**: Python package manager repository support.

3. **Proxy and Caching**:
   - Nexus 3 allows you to set up proxy repositories that cache external repositories like Maven Central or Docker Hub. This feature improves build performance by reducing the need to repeatedly download dependencies from external sources.

4. **Hosting Private Repositories**:
   - Nexus 3 enables organizations to host private repositories to store their proprietary or custom artifacts. This is essential for managing internal libraries and ensuring data security.

5. **Search and Indexing**:
   - Nexus 3 provides robust search capabilities, making it easy to find and retrieve artifacts quickly. It indexes repositories, allowing you to search for artifacts by name, version, and other metadata.

6. **Security and Access Control**:
   - Nexus 3 includes fine-grained access control features. You can define roles and permissions to restrict or grant access to specific users or groups, ensuring that sensitive artifacts are protected.

7. **Integration with CI/CD Tools**:
   - Nexus 3 integrates seamlessly with popular CI/CD (Continuous Integration/Continuous Deployment) tools like Jenkins, Travis CI, and CircleCI. This integration ensures that artifacts are automatically published and retrieved during the build and deployment processes.

8. **Lifecycle Management**:
   - Nexus 3 supports the management of artifact lifecycles. You can define retention policies, promote artifacts through different stages (e.g., from development to production), and track the history of changes.

9. **Repository Health and Monitoring**:
   - It provides monitoring and reporting features to track repository health, performance, and usage. You can identify storage issues and optimize your repository manager accordingly.

10. **RESTful API**:
    - Nexus 3 offers a RESTful API that allows you to automate various tasks, such as uploading and downloading artifacts, configuring repositories, and managing permissions.

11. **High Availability and Scalability**:
    - Nexus 3 can be set up in a high-availability configuration for improved reliability. It can scale horizontally to handle increased artifact storage and retrieval demands.

12. **User-Friendly Web Interface**:
    - Nexus 3 includes a web-based user interface that makes it easy to manage repositories, configure settings, and perform administrative tasks.

13. **Plugin Ecosystem**:
    - Nexus 3 has a plugin system that allows you to extend its functionality to meet specific requirements or integrate with other tools and systems.

In summary, Nexus 3 is a powerful artifact repository manager that plays a crucial role in modern software development and DevOps pipelines. It offers features for artifact storage, caching, security, and integration with various development tools, making it an essential component of many software development environments.


# STEPS

```markdown
# NEXUS3 INSTALLATION

sudo apt install openjdk-8-jdk -y

cd /opt
wget https://download.sonatype.com/nexus/3/nexus-3.59.0-01-unix.tar.gz
tar -xvf nexus-3.59.0-01-unix.tar.gz

# Create user nexus

adduser nexus

chown -R nexus:nexus nexus-3.59.0-01/
chown -R nexus:nexus sonatype-work/

vi nexus-3.59.0-01/bin/nexus.rc
in ""  put nexus  --->"nexus"

/opt/nexus-3.59.0-01/bin/nexus start
```

## ADD in POM

```xml
<project>
    <!-- ... other project information ... -->
    
    <distributionManagement>
        <repository>
            <id>maven-releases</id>
            <url>NEXUS-URL/repository/maven-releases/</url>
        </repository>
        <snapshotRepository>
            <id>maven-snapshots</id>
            <url>NEXUS-URL/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>

    <!-- ... other project configuration ... -->
</project>
```

# PIPELINE

```groovy
pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('git-checkout') {
            steps {
                git 'https://github.com/jaiswaladi2468/BoardgameListingWebApp.git'
            }
        }

        stage('Code-Compile') {
            steps {
               sh "mvn clean compile"
            }
        }
        
        stage('Unit-Test') {
            steps {
               sh "mvn clean test"
            }
        }
        
        stage('Trivy Scan') {
            steps {
               sh "trivy fs ."
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
               dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=BoardGame '''
               }
            }
        }
        
        stage('Code-Build') {
            steps {
               sh "mvn clean package"
            }
        }
        
        stage('Deploy To Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'e7838703-298a-44a7-b080-a9ac14fa0a5e') {
                    sh "mvn deploy"
               }
            }
        }
    }
}
```

## Pipeline for downloading the Artifact


```groovy
pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('git-checkout') {
            steps {
                git 'https://github.com/jaiswaladi2468/BoardgameListingWebApp.git'
            }
        }

        stage('Code-Compile') {
            steps {
               sh "mvn clean compile"
            }
        }
        
        stage('Unit-Test') {
            steps {
               sh "mvn clean test"
            }
        }
        
        stage('Trivy Scan') {
            steps {
               sh "trivy fs ."
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
               dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=BoardGame '''
               }
            }
        }
        
        stage('Download JAR with Credentials') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'your-credentials-id', usernameVariable: 'user', passwordVariable: 'pass')]) {
                        def jarUrl = 'https://example.com/path/to/your.jar'

                        sh "curl -u $user:$pass -O $jarUrl"
                    }
                }
            }
        }
        
        stage('Build & Deploy To Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'e7838703-298a-44a7-b080-a9ac14fa0a5e') {
                    sh "mvn deploy"
               }
            }
        }
    }
}
```
