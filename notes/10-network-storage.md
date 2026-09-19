# Network Storage & Data Lifecycle

## EFS (Elastic File System) Architecture

- an implementation of NFSv4
- shared filesystem, can be mounted in **linux only**, across many EC2
    - right now, for A4L demo, the media is stored locally on EC2. Using EFS will make the instances stateless, and make the data permanent
- EFS is file storage (vs block storage of EBS)
    - but linux systems can mount EFS filesystems as though they are connected directly to the system
- private service, via "mount targets" inside a VPC
    - can be accessed outside via VPN etc

***todo: diagram***

- general purpose and max i/o performance modes
- burst and provisioned throughput modes
- 2 storage classes — infrequent access, and standard

## AWS Backup

- fully managed data protection (backup/restore) service
- consolidate management into one place — across accounts and regions
- handles everything — ebs, db, ec2, efs, s3 etc — central mgmt for all data/backups
- Vaults, vault lock, WORM, 72 hr cool off period
- point in time recovery
