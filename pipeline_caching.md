
## Stash and Unstash

`stash` and `unstash` directives to preserve files between stages in the same pipeline run. By default, stashed files are discarded at the end of the run. However, with the Declarative Pipeline `preserveStashes` option or the Pipeline: Durable Task plugin, you can retain them across a restart. Once stashed, files can be unstashed by name in any other stage even on a different node.

```groovy
pipeline {
  agent any
  options {
    timestamps()
  }
  stages {
    stage('Installing Dependencies') {
      steps {
        sh 'node -v'
        sh 'npm install --no-audit'
        stash includes: 'node_modules/', name: 'solar-system-node-modules'          // adding node modules to stash
      }
    }
    stage('Dependency Scanning') {
      steps {
        // ...
      }
    }
  }
}
```

Using unstash in a Downstream Stage

In any later stage regardless of agent or node you can restore the stashed files instead of reinstalling:

```groovy
pipeline {
  agent any
  stages {
    stage('NodeJS 20') {
      agent {
        // e.g. run in a Docker container or Kubernetes pod
      }
      stages {
        stage('Install Dependencies') {
          options {
            retry(2)
          }
          steps {
            sh 'node -v'
            // Restore the stash instead of npm install
            unstash 'solar-system-node-modules'
          }
        }
        stage('Testing') {
          steps {
            // Run tests against the restored node_modules
          }
        }
      }
    }
  }
}
```


# Pipeline Caching

Efficient dependency caching in a CI/CD pipeline can dramatically reduce build times by reusing previously downloaded libraries and generated artifacts. In Jenkins, the Job Cacher Plugin enables you to store and restore cache items such as node_modules across pipeline runs, even in ephemeral environments like containers.

Caching avoids repeated downloads and installations, leading to faster feedback loops and lower resource usage. It’s especially helpful for languages and frameworks with large dependency trees (e.g., Node.js, Python).

| Field                       | Description                                           | Example             |
| --------------------------- | ----------------------------------------------------- | ------------------- |
| `path`                      | Directory to cache                                    | `node_modules`      |
| `cacheValidityDecidingFile` | File that triggers cache invalidation when it changes | `package-lock.json` |
| `includes`                  | Glob pattern to include in cache                      | `**/*`              |
| `excludes`                  | Glob pattern to exclude from cache                    | `**/*.generated`    |

## Installing the Job Cacher Plugin

1. Navigate to **Manage Jenkins** > **Manage Plugins** > **Available**.
2. Search for **Job Cacher Plugin** and install it.
3. Restart Jenkins to activate the plugin.

Generated snippet:

```groovy theme={null}
cache(
  caches: [
    arbitraryFileCache(
      cacheName: 'npm-dependency-cache',
      cacheValidityDecidingFile: 'package-lock.json',
      includes: '**/*',
      path: 'node_modules'
    )
  ],
  maxCacheSize: 550
) {
}
```

## Integrating Caching into Your Jenkinsfile

Insert the `cache` block around your dependency installation stage. Here’s a streamlined `Jenkinsfile` example:

```groovy theme={null}
pipeline {
  agent any

  stages {
    stage('Installing Dependencies') {
      options { timestamps() }
      steps {
        cache(
          maxCacheSize: 550,
          caches: [
            arbitraryFileCache(
              cacheName: 'npm-dependency-cache',
              cacheValidityDecidingFile: 'package-lock.json',
              includes: '**/*',
              path: 'node_modules'
            )
          ]
        ) {
          sh 'node -v'
          sh 'npm install --no-audit'
          stash(includes: 'node_modules', name: 'npm-node-modules')
        }
      }
    }

    stage('Dependency Scanning') {
      steps {
        // Add your scanning logic here
      }
    }

    stage('Unit Testing') {
      parallel {
        stage('NodeJS 18') {
          options { retry(2) }
          steps {
            unstash 'npm-node-modules'
            sh 'node -v'
            sh 'npm test'
          }
        }
        stage('NodeJS 19') {
          options { retry(2) }
          steps {
            container('node-19') {
              unstash 'npm-node-modules'
              sh 'node -v'
              sh 'npm test'
            }
          }
        }
      }
    }

    stage('Code Coverage') {
      steps {
        catchError(buildResult: 'SUCCESS', message: 'Coverage step failed', stageResult: 'FAILURE') {
          unstash 'npm-node-modules'
          sh 'node -v'
          sh 'npm run coverage'
        }
      }
    }
  }
}
```
