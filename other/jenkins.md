# Jenkins

The core features of Jenkins are `Pipeline` and the Blue Ocean UI.  

## Installation

Make sure you've installed and switch to the correct Java version such as OpenJDK 11. Then: 

```bash
sudo apt-get install jenkins
```

To start Jenkins: 

1. Use `dpkg -L jenkins` to find the location of `jenkins.war`. 
2. Run:

```bash
java -jar jenkins.war --httpPort=8000
```

3. Unlock Jenkins with the password provided in the terminal.
4. Install plugins.
5. Create user (or skip and continue as admin).
6. Set up the Jenkins URL.

## Jenkins Pipeline

https://www.jenkins.io/doc/book/pipeline/

Basic workflow: the root of the repository contains a `Jenkinsfile`. This defines the stages of the pipeline. Whenever you updated the designated repository, a new build is triggered. 

Pipeline is a suite of plugins which supports implementing and integrating *continuous delivery pipelines* into Jenkins.

The definition of a Jenkins Pipeline is typically written into a text file (called a `Jenkinsfile`) which in turn is checked into a project’s source control repository.

Pipeline provides an extensible set of tools for modeling simple-to-complex delivery pipelines "as code" via the [Pipeline domain-specific language (DSL) syntax](https://www.jenkins.io/doc/book/pipeline/syntax). [[1](https://www.jenkins.io/doc/book/pipeline/#_footnotedef_1)]

The definition of a Jenkins Pipeline is written into a text file (called a [`Jenkinsfile`](https://www.jenkins.io/doc/book/pipeline/jenkinsfile)) which in turn can be committed to a project’s source control repository. This is the foundation of "Pipeline-as-code"; treating the CD pipeline a part of the application to be versioned and reviewed like any other code.

Creating a `Jenkinsfile` and committing it to source control provides a number of immediate benefits:

- Automatically creates a Pipeline build process for all branches and pull requests.
- Code review/iteration on the Pipeline (along with the remaining source code).
- Audit trail for the Pipeline.
- Single source of truth [[3](https://www.jenkins.io/doc/book/pipeline/#_footnotedef_3)] for the Pipeline, which can be viewed and edited by multiple members of the project.

**Declarative pipeline fundamentals**. The `pipeline` block defines all the work done throughout your entire Pipeline:

```bash
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any 
    stages {
        stage('Build') { 
            steps {
                // 
            }
        }
        stage('Test') { 
            steps {
                // 
            }
        }
        stage('Deploy') { 
            steps {
                // 
            }
        }
    }
}
```

Declarative pipeline example:

```bash
Jenkinsfile (Declarative Pipeline)
pipeline { 
    agent any 
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Build') { 
            steps { 
                sh 'make' 
            }
        }
        stage('Test'){
            steps {
                sh 'make check'
                junit 'reports/**/*.xml' 
            }
        }
        stage('Deploy') {
            steps {
                sh 'make publish'
            }
        }
    }
}
```

Typical stages in a pipeline are Build, Test and Deploy.

1. Build. Source code is assembled, compiled or packaged. This could be done e.g. through a `make` file. 
2. Test. If the tests fail, the pipeline is marked as "unstable".
3. Deploy. Pushing code to a production system. 

Assuming everything has executed successfully in the example Jenkins Pipeline, each successful Pipeline run will have associated build artifacts archived, test results reported upon and the full console output all in Jenkins.