# Agents and Nodes in Jenkins

## Agents in Jobs

```groovy
pipeline {
    agent any

    stages {
        stage('S1 – Any Agent') {
            steps {
                sh 'cat /etc/os-release'
                sh 'node -v'
                sh 'npm -v'
            }
        }
        stage('S2 – Ubuntu Agent') {
            agent { label 'ubuntu-docker-jdk17-node20' }
            steps {
                sh 'cat /etc/os-release'
                sh 'node -v'
                sh 'npm -v'
            }
        }
    }
}
```

## Docker Image Agent

Navigate to Pipeline Syntax → Agent to generate the DSL snippet.

```groovy
pipeline {
    agent any
    stages {
        stage('S1-Any Agent') {
            steps {
                sh 'cat /etc/os-release'
                sh 'node -v'
                sh 'npm -v'
            }
        }
        stage('S2-Ubuntu Agent') {
            agent { label 'ubuntu-docker-jdk17-node20' }
            steps {
                sh 'cat /etc/os-release'
                sh 'node -v'
                sh 'npm -v'
            }
        }
        stage('S3-Docker Image Agent') {
            agent {
                docker {
                    alwaysPull true         // Optional
                    image 'node:18-alpine'
                    label 'ubuntu-docker-jdk17-node20'  // runs on this node
                }
            }
            steps {
                sh 'cat /etc/os-release'
                sh 'node -v'
                sh 'npm -v'
            }
        }
    }
}
```

## Dockerfile Agent

Create a Custom Dockerfile

```
# Dockerfile.cowsay

FROM node:18-alpine

RUN apk update && \
    apk add --no-cache git perl && \
    git clone https://github.com/jasonm23/cowsay.git /tmp/cowsay && \
    cd /tmp/cowsay && \
    ./install.sh /usr/local && \
    rm -rf /tmp/cowsay
```

```groovy
pipeline {
    agent any

    stages {
        stage('S1 – Any Agent') {
            steps {
                sh 'cat /etc/os-release'
                sh 'node -v'
                sh 'npm -v'
            }
        }
        stage('S2 – Ubuntu Agent') {
            agent { label 'ubuntu-docker-jdk17-node20' }
            steps {
                sh 'cat /etc/os-release'
                sh 'node -v'
                sh 'npm -v'
            }
        }
    }
}
```

## Docker Image Agent

Navigate to Pipeline Syntax → Agent to generate the DSL snippet.

```groovy
pipeline {
    agent any
    stages {
        stage('S1-Any Agent') {
            steps {
                sh 'cat /etc/os-release'
                sh 'node -v'
                sh 'npm -v'
            }
        }
        stage('S2-Ubuntu Agent') {
            agent { label 'ubuntu-docker-jdk17-node20' }
            steps {
                sh 'cat /etc/os-release'
                sh 'node -v'
                sh 'npm -v'
            }
        }
        stage('S3-Docker Image Agent') {
            agent {
                docker {
                    alwaysPull true         // Optional
                    image 'node:18-alpine'
                    label 'ubuntu-docker-jdk17-node20'  // runs on this node
                }
            }
            steps {
                sh 'cat /etc/os-release'
                sh 'node -v'
                sh 'npm -v'
            }
        }
        stages {
            stage('S4 - Dockerfile Agent') {
                agent {
                    dockerfile {
                        filename 'Dockerfile.cowsay'
                        label 'ubuntu-docker-jdk17-node20'
                    }
                }
                steps {
                    sh 'node -v'
                    sh 'npm -v'
                    sh 'cowsay -f dragon "This is running on Docker Container"'
                }
            }
        }        
    }
}
```

## newContainerPerStage

By default, Jenkins runs all stages in a single container, sharing the workspace and any generated artifacts.

```groovy
pipeline {
  agent {
    dockerfile {
      filename 'Dockerfile.cowsay'
      label 'ubuntu-docker-jdk17-node20'
    }
  }
  stages {
    stage('Stage-1') {
      steps {
        sh 'cat /etc/os-release'
        sh 'node -v'
        sh 'npm -v'
        echo '#############################'
        sh "echo $((RANDOM)) > /tmp/imp-file-$BUILD_ID"
        sh 'ls -ltr /tmp/imp-file-$BUILD_ID'
        sh 'cat /tmp/imp-file-$BUILD_ID'
        echo '#############################'
      }
    }
    stage('Stage-2') {
      steps {
        sh 'cat /etc/os-release'
        sh 'node -v'
        sh 'npm -v'
        echo 'Reading file generated in Stage-1:'
        sh 'cat /tmp/imp-file-$BUILD_ID'
      }
    }
    stage('Stage-3') {
      steps {
        sh 'cat /etc/os-release'
        sh 'node -v'
        sh 'npm -v'
      }
    }
    stage('Stage-4') {
      steps {
        sh 'node -v'
        sh 'npm -v'
        sh 'cowsay -f dragon This is running on Docker Container'
        echo 'Final file check:'
        sh 'cat /tmp/imp-file-$BUILD_ID'
        sh 'sleep 120s'
      }
    }
  }
}
```

## Enforcing Stage Isolation with newContainerPerStage

To run each stage in its own container so artifacts from one stage aren’t carried over enable the `newContainerPerStage()` option.

```groovy
pipeline {
  agent {
    dockerfile {
      filename 'Dockerfile.cowsay'
      label 'ubuntu-docker-jdk17-node20'
    }
  }
  options {
    newContainerPerStage()
  }
  stages {
    stage('Stage-1') {
      steps {
        sh 'cat /etc/os-release'
        sh 'node -v'
        sh 'npm -v'
        echo '********************'
        sh "echo $((RANDOM)) > /tmp/imp-file-$BUILD_ID"
        sh 'ls -ltr /tmp/imp-file-$BUILD_ID'
        sh 'cat /tmp/imp-file-$BUILD_ID'
        echo '********************'
      }
    }
    stage('Stage-2') {
      steps {
        sh 'cat /etc/os-release'
        sh 'node -v'
        sh 'npm -v'
        echo 'Trying to read file from Stage-1:'
        sh 'ls -ltr /tmp/imp-file-$BUILD_ID'
        sh 'cat /tmp/imp-file-$BUILD_ID'
      }
    }
  }
}
```

## Kubernetes Pod as Agent

```groovy
pipeline {
  agent {
    kubernetes {
      yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: ubuntu-container
    image: ubuntu
    command:
    - sleep
    args:
    - infinity
'''
    }
  }
  stages {
    stage('Print Hostname') {
      steps {
        sh 'hostname'
      }
    }
  }
}
```

## Multiple Containers

Define multiple containers in the same Pod to run different tools per stage:

```groovy
pipeline {
  agent {
    kubernetes {
      cloud 'dasher-prod-k8s-us-east'
      yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: ubuntu-container
    image: ubuntu
    command:
    - sleep
    args:
    - infinity
  - name: node-container
    image: node:18-alpine
    command:
    - cat
    tty: true
'''
        defaultContainer 'ubuntu-container'
    }
  }
  stages {
    stage('Print Hostname') {
      steps {
        sh 'hostname'
      }
    }
    stage('Print Node Version') {
      steps {
        container('node-container') {
          sh 'node -v'
          sh 'npm -v'
        }
      }
    }
  }
}
```

## Sharing a File between Containers

```groovy
pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: ubuntu-container
    image: ubuntu
    command: ['sleep']
    args: ['infinity']
  - name: node-container
    image: node:18-alpine
    command: ['cat']
    tty: true
"""
      defaultContainer 'ubuntu-container'
    }
  }

  stages {
    stage('Create File on Ubuntu') {
      steps {
        sh '''
          # Display the hostname
          hostname

          # Write data to a file
          echo important_UBUNTU_data > ubuntu-$BUILD_ID.txt

          # Confirm the file was created
          ls -ltr ubuntu-$BUILD_ID.txt
        '''
      }
    }

    stage('Access File on Node') {
      steps {
        container('node-container') {
          sh '''
            # Verify Node.js and npm versions
            node -v
            npm -v

            # List and read the file created earlier
            ls -ltr ubuntu-$BUILD_ID.txt
            cat ubuntu-$BUILD_ID.txt
          '''
        }
      }
    }
  }
}
```

## Sequential (Nested) Stages

To break down the work, nest a stages block inside your NodeJS 20 stage, creating two ordered steps: Install Dependencies and Testing.

```groovy
pipeline {
  agent {
    kubernetes {
      cloud 'dasher-prod-k8s-us-east'
      yamlFile 'k8s-agent.yaml'
      defaultContainer 'node-18'
    }
  }
  tools {
    nodejs 'nodejs-22-6-0'
  }
  environment {
    MONGO_URI          = "mongodb+srv://supercluster.d83jj.mongodb.net/superData"
    MONGO_DB_CREDS     = credentials('mongo-db-credentials')
    MONGO_USERNAME     = credentials('mongo-db-username')
    MONGO_PASSWORD     = credentials('mongo-db-password')
    SONAR_SCANNER_HOME = tool 'sonarqube-scanner-610'
    GITEA_TOKEN        = credentials('gitea-api-token')
  }
  options {
    disableResume()
    disableConcurrentBuilds(abortPrevious: true)
  }
  stages {
    stage('Installing Dependencies') {
      options { timestamps() }
      steps {
        sh 'node -v'
        sh 'npm install --no-audit'
      }
    }

    stage('Dependency Scanning') {
      parallel {
        // e.g., Trivy, Snyk, OWASP Dependency Check
      }
    }

    stage('Unit Testing') {
      parallel {
        stage('NodeJS 18') {
          options { retry(2) }
          steps {
            sh 'node -v'
            sh 'npm test'
          }
        }

        stage('NodeJS 19') {
          options { retry(2) }
          steps {
            container('node-19') {
              // Sleep to avoid port conflicts on shared volumes
              sh 'sleep 10s'
              sh 'node -v'
              sh 'npm test'
            }
          }
        }

        stage('NodeJS 20') {
          agent {
            docker { image 'node:20-alpine' }
          }
          stages {
            stage ('Install Dependencies') {
                options { retry(2) }
                steps {
                    sh 'node -v'
                    sh 'npm install --no-audit'
                }
            }

            stage ('Testing') {
                options { retry(2) }
                steps {
                    sh 'node -v'
                    sh 'npm test'
                }
            }            
          }
        }
      }
    }

    stage('Code Coverage') {
      steps {
        catchError(buildResult: 'SUCCESS',
                   message: 'Coverage will be fixed in future releases',
                   stageResult: 'UNSTABLE') {
          sh 'node -v'
          sh 'npm run coverage'
        }
      }
    }
  }
}
```
