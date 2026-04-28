# Proxmox User Creation and Firewall Rule Configuration

## 1. User Creation in Proxmox

This guide explains how to create a user in Proxmox and assign the required role.

---

## Step 1: Create the User

### For Proxmox Internal Authentication (`pve`)

```bash
pveum useradd username@pve --password StrongPassword
```

### Example

```bash
pveum useradd yashupadhyay@pve --password H23lVyHA
```

---

## Step 2: Assign Role to the User

### Example: Assign `PVEAdmin`

```bash
pveum aclmod / -user yashupadhyay@pve -role PVEAdmin
```

### Example: Assign Custom Role (`edrsec`)

```bash
pveum aclmod /pool/edrsec -user yashupadhyay@pve -role edrsec
```

---

## Step 3: Delete User (If Needed)

### Remove User from Proxmox

```bash
pveum user delete yashupadhyay@pve
```

### Example

```bash
pveum user delete username@pve
```

> Note: If the user has existing ACLs or permissions, remove them first if required.

---

## Step 4: Verify User and Permissions

### Check Existing User

```bash
pveum user list | grep yashupadhyay
```

### Check Assigned Permissions

```bash
pveum acl list | grep yashupadhyay
```

---

# 2. Add Firewall Rule Using `cluster.fw`

This method is used when Proxmox Firewall is enabled and you want to allow a specific IP address.

---

## Step 1: Edit Firewall Configuration File

```bash
vi /etc/pve/firewall/cluster.fw
```

---

## Step 2: Add the Rule

Add the following under the `[RULES]` section:

```text
[RULES]
IN ACCEPT -source 162.253.157.165 -p tcp --dport 8006
```

### Explanation

* `IN` → Incoming traffic
* `ACCEPT` → Allow traffic
* `-source` → Allowed source IP
* `-p tcp` → TCP protocol
* `--dport 8006` → Proxmox Web UI port

---

## Step 3: Restart Proxmox Firewall

```bash
pve-firewall restart
```

---

## Step 4: Verify Firewall Status

```bash
pve-firewall status
```

---

## Optional: Test Access

Open in browser:

```text
https://your-proxmox-server:8006
```

Example:

```text
https://pve5-ewr1.belugacdn.com:8006
```

---

## Notes

* If firewall rules are changed after login, logout and login again.
* Ensure no conflicting DROP rules exist above the ACCEPT rule.
* If using Security Groups, verify the same IP is allowed there as well.

---

## Useful Commands

### Check iptables Rules

```bash
iptables -L -n -v
```

### Check Proxmox ACLs

```bash
pveum acl list
```

### Check Pools

```bash
pvesh get /pools
```
