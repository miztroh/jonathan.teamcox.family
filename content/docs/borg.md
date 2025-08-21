---
{"publish":true,"title":"Borg","created":"2025-08-21T11:59:23.893-05:00","modified":"2025-08-21T12:57:56.317-05:00","published":"2025-08-21T12:57:56.317-05:00","cssclasses":""}
---

A quick overview of how to utilize [Borg](https://www.borgbackup.org/) to back up data over SSH.

```sh
# Define the repo connection string
repo="ssh://<remote username>@<remote hostname>:<remote port>/<remote path>"
# Initialize the repo
borg init --encryption=none $repo
# Create a new archive in the repo
borg create --stats --log-json $repo::$(date +%Y%m%d_%H%M%S) <local path>
# Prune older archives from the repo
borg prune --keep-within=14d --keep-last=14 $repo
```