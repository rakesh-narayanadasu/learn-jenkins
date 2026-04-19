# Backing upRestoring Jenkins

Regular backups of your Jenkins instance are critical for disaster recovery, data integrity, and accidental-deletion protection. In this guide, we’ll cover the most reliable backup strategies and detail exactly which files and directories you should preserve.

## Backup Approaches

| Approach              | Description                                                         | Tools / Plugins                                      |
| --------------------- | ------------------------------------------------------------------- | ---------------------------------------------------- |
| File-system snapshots | Capture a point-in-time, consistent copy of the entire file system. | Linux LVM, AWS EBS snapshots, GCP persistent disks   |
| Jenkins plugins       | Export Jenkins configuration and jobs via plugin interfaces.        | [ThinBackup](https://plugins.jenkins.io/thinBackup/) |
| Custom scripts        | Copy only essential directories and files to a backup location.     | Bash, PowerShell, rsync                              |
| Hybrid strategy       | Use local snapshots for speed, then replicate to remote storage.    | Combination of above                                 |

The **ThinBackup** plugin is the only actively maintained backup plugin for Jenkins. Other plugins may be outdated or lack features.

By combining snapshots, plugins, and scripts, you can assemble a resilient backup plan tailored to your infrastructure and RPO/RTO requirements.

## What to Backup

Your `$JENKINS_HOME` directory (typically `/var/lib/jenkins` on Linux) holds all your Jenkins data. While full-directory backups are the simplest, selective backups can save storage and speed up operations.

```bash theme={null}
$ tree /var/lib/jenkins
.
├── config.xml            # Main Jenkins configuration file
├── *.xml                 # Site-wide config files (e.g., credentials.xml)
├── builds/               # Build results: build.xml, changelog.xml
├── fingerprints/         # Artifact fingerprint records
├── identity.key.enc      # Encrypted RSA key pair for this instance
├── jobs/                 # Job definitions and history
│   └── [JOBNAME]/
│       └── config.xml    # Individual job configuration
├── nodes/                # Configuration for build agents
├── plugins/              # Installed plugins (.jpi, .hpi)
├── secrets/              # Encrypted credentials and keys
├── userContent/          # Static files served at /userContent/
├── users/                # User account configurations
└── workspace/            # SCM workspace directories
```

### Essential Inclusions

| Directory / File               | Purpose                                        | Priority   |
| ------------------------------ | ---------------------------------------------- | ---------- |
| `config.xml`, `*.xml`          | Global Jenkins and system configurations       | High       |
| `jobs/[JOBNAME]/config.xml`    | Job definitions and parameters                 | High       |
| `secrets/`, `identity.key.enc` | Credentials and master keys                    | High       |
| `fingerprints/`                | Artifact tracking records                      | Medium     |
| `nodes/`                       | Build agent configurations                     | Medium     |
| `plugins/`                     | Installed plugin binaries (can be reinstalled) | Low–Medium |

### Optional Exclusions

* `workspace/` directories (recreated on build)
* Build artifacts and logs (exclude if storing centrally)
* Caches and temporary files
* Plugin data that can be fetched again from Update Center

> Always encrypt your backups at rest and in transit. Store credentials and master keys securely to prevent unauthorized access.

By exporting these files via snapshots, plugins, or scripts—and replicating them to secure local and remote targets, you ensure a solid Jenkins disaster-recovery strategy.

## Links and References

* [Jenkins Backup and Restore](https://www.jenkins.io/doc/book/system-administration/backing-up/)
* [ThinBackup Plugin](https://plugins.jenkins.io/thinBackup/)
* [Linux LVM Snapshots](https://linux.die.net/man/8/lvcreate)
* [AWS EBS Snapshots](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html)
