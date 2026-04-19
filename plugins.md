# Plugins

## Agents

- Docker Pipeline

## Cloud

- Kubernetes

## Auditing

- Audit Trial (who did what)
- Job Config

## Monitoring

- Prometheus metrics

## Logger

Adding a Logger from the UI

The most straightforward method to configure logging is via the Jenkins web interface:

1. Go to **Manage Jenkins** → **System Log**.
2. Click **Add new log recorder**.
3. Enter a meaningful name, e.g., `k8s-agent-logs`.
4. Under **Log Levels**, select the severity (ALL, FINE, FINEST, etc.).
5. In the **Loggers** section, add one or more package or class names (for example, `io.fabric8.kubernetes.client`).
6. Click **Save**.


## Backup

- ThinBackup (Jenkins plugin helps to backup and restore)

## JCasC

- Configuration as Code
