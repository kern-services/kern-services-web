---
title: "Automated MySQL Backups with automysqlbackup"
date: 2014-05-29
draft: false
description: "Learn how to set up automated MySQL database backups using the powerful automysqlbackup tool"
tags: ["mysql", "backup", "database", "automation", "linux"]
categories: ["Database Administration"]
featuredImage: "/images/insights/markus-winkler-gLdJnQFcIXE-unsplash.jpg"
photoCredit:
  photographer: "Markus Winkler"
  photographerUrl: "https://unsplash.com/de/@markuswinkler?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash"
  source: "Unsplash"
  sourceUrl: "https://unsplash.com/de/fotos/weisses-und-grunes-einwegfeuerzeug-gLdJnQFcIXE?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash"

---

Database backups are a critical part of any system administration strategy. When it comes to MySQL databases, one of the most reliable and feature-rich backup solutions is the `automysqlbackup` tool. This article will guide you through setting up and configuring this powerful backup solution.

## Why automysqlbackup?

`automysqlbackup` stands out for several reasons:

* **Automatic rotation**: Creates daily, weekly, and monthly backups with automatic rotation
* **Compression**: Automatically compresses backup files to save space
* **Email notifications**: Can be configured to send email notifications about backup status
* **Flexible configuration**: Highly configurable to meet specific backup requirements
* **Multiple database support**: Can backup all databases or specific ones

## Installation

You can install `automysqlbackup` in two ways:

1. Clone from my GitHub repository:
```bash
git clone git@github.com:mattanja/automysqlbackup.git
```

2. Or download from the original source:
```bash
wget http://sourceforge.net/projects/automysqlbackup/
```

After obtaining the files, run the setup script:
```bash
cd automysqlbackup
./install.sh
```

This will install:
* The executable script in `/usr/local/bin/automysqlbackup`
* Default configuration in `/etc/automysqlbackup`

## Configuration

The tool uses two main configuration files:

1. `/etc/automysqlbackup/automysqlbackup.conf` - Default configuration
2. `/etc/automysqlbackup/myserver.conf` - Server-specific settings

Key configuration options include:
* Backup directory location
* Database selection (all or specific databases)
* Compression settings
* Email notification settings
* Retention policies

## Setting up Automated Backups

To set up automated daily backups, use the provided cron example:

```bash
# Create a new cron file
sudo cp /etc/automysqlbackup/example-cron-file /etc/cron.d/automysqlbackup

# The cron file will contain:
# Run the mysql backup scripts every morning at 4.30 h:
30 4 * * * root /etc/automysqlbackup/run
```

## Backup Structure

The backup system creates a structured hierarchy:
```
backup_dir/
├── daily/
│   └── [database_name]/
│       └── [date]_[database_name].sql.gz
├── weekly/
└── monthly/
```

## Additional Features

* **Multiple backup types**: Full, incremental, and differential backups
* **Pre/post backup hooks**: Execute custom scripts before or after backups
* **Logging**: Detailed logging of backup operations
* **Error handling**: Automatic error detection and notification

## Security Considerations

* Store backup files in a secure location
* Use appropriate file permissions
* Consider encrypting sensitive backup data
* Implement proper access controls

## Note on Current Relevance (2025)

Despite being originally written in 2014, this backup solution remains highly relevant in 2025 because:

1. **Proven Reliability**: The tool has been battle-tested for over a decade
2. **Simplicity**: Its straightforward approach makes it easy to maintain
3. **Compatibility**: Continues to work with modern MySQL versions
4. **Flexibility**: Can be integrated with modern backup solutions
5. **Low Resource Usage**: Efficient operation makes it suitable for various environments

While newer backup solutions exist, `automysqlbackup` remains a solid choice for many use cases, especially for those who value simplicity and reliability over cutting-edge features.

## Further Reading

* [Official MySQL Backup Documentation](https://dev.mysql.com/doc/refman/8.0/en/backup-and-recovery.html)
* [automysqlbackup GitHub Repository](https://github.com/mattanja/automysqlbackup)
