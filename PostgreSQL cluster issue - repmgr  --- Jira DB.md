 Let’s assume:

jira-db2 → real primary

jira-db1 → will become standby again

Step 2: Stop PostgreSQL on the old primary (DB1)
sudo systemctl stop postgresql
Step 3: Re-clone DB1 from DB2 (the current primary)
As the postgres user on DB1:

repmgr -h jira-db2 -U repmgr -d repmgr -f /etc/repmgr.conf standby clone --force
This wipes the old data directory and copies fresh data from DB2.

Step 4: Start PostgreSQL on DB1 again
sudo systemctl start postgresql
Step 5: Re-register DB1 as standby
repmgr -f /etc/repmgr.conf standby register
Step 6: Verify cluster state
On either node:

repmgr cluster show
✅ You should now see:

ID | Name     | Role    | Status
---+----------+---------+---------
2  | jira-db2 | primary | * running
1  | jira-db1 | standby |   running
