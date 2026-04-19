# Migrating Jenkins to Another Node

How to migrate a Jenkins controller from one virtual machine to another while preserving jobs, plugins, and configurations.

To migrate a Jenkins controller from one virtual machine (Node) to another we need to stop the service, backing up the Jenkins home directory, transferring it to the new Node, and bringing Jenkins back online—preserving all jobs, plugins, and configurations.

## 1. Scenario

* **Source Jenkins controller**\
  IP: `64.227.x.x`\
  Fully configured and running jobs.

* **Target VM**\
  IP: `165.232.x.x`\
  Fresh Jenkins install on Ubuntu with Docker & JDK 17.

## 2. Pre-migration Checklist

Before you begin, make sure both Nodes meet these requirements:

| Requirement      | Source Node           | Target Node           |
| ---------------- | --------------------- | --------------------- |
| Jenkins version  | Match source & target | Match source & target |
| JDK version      | 17                    | 17                    |
| SSH connectivity | Enabled               | Enabled               |

## 3. Back Up Jenkins Home on Source Node

On the **source Node**, `$JENKINS_HOME` is typically `/var/lib/jenkins`. Follow these steps:

1. Stop and disable Jenkins:
   ```bash theme={null}
   sudo systemctl stop jenkins
   sudo systemctl disable jenkins
   ```
2. Verify it’s inactive:
   ```bash theme={null}
   sudo systemctl status jenkins
   # Should show: Active: inactive (dead)
   ```
3. Create a compressed backup:
   ```bash theme={null}
   cd /var/lib
   sudo tar -zcvf jenkins-backup.tar.gz jenkins
   ```

Once `jenkins-backup.tar.gz` is ready, transfer it to the target Node.

## 4. Transfer Backup to Target Node

From the **source Node**, use `scp` to copy the archive:

```bash theme={null}
scp /var/lib/jenkins-backup.tar.gz root@165.232.x.x:/tmp/
```

Enter the root password when prompted. Then SSH into the **target Node** to proceed.

## 5. Prepare the Target Node

1. Stop and disable the existing Jenkins service:
   ```bash theme={null}
   sudo systemctl stop jenkins
   sudo systemctl disable jenkins
   ```
2. Confirm it’s stopped:
   ```bash theme={null}
   sudo systemctl status jenkins
   # Active: inactive (dead)
   ```

If you browse to Jenkins now, you may see the setup screen:

### Optional: Backup Current Jenkins on Target

If you want to keep the default install:

```bash theme={null}
cd /var/lib
sudo tar -zcvf jenkins-original-backup.tar.gz jenkins
sudo rm -rf jenkins
```

## 6. Restore Backup on Target Node

1. Move and extract the backup:
   ```bash theme={null}
   sudo mv /tmp/jenkins-backup.tar.gz /var/lib/
   cd /var/lib
   sudo tar -zxvf jenkins-backup.tar.gz
   ```
2. Fix ownership:
   ```bash theme={null}
   sudo chown -R jenkins:jenkins jenkins
   ```

## 7. Start Jenkins on Target Node

Re-enable and launch Jenkins:

```bash theme={null}
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
# Should show: Active: active (running)
```

Refresh your browser on `http://165.232.x.x:8080/`. All your jobs, plugins, and settings will appear exactly as before.

Do **not** run the same Jenkins instance on two Nodes at once. Always stop the source Jenkins before bringing up the target.

## Links and References

* [Jenkins Official Documentation](https://www.jenkins.io/doc/)
* [Backing Up and Restoring Jenkins](https://www.jenkins.io/doc/book/system-administration/backing-up/)
