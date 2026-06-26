# Fix GPG Key Issues for MongoDB, Puppet, and InfluxDB (Debian Buster)

## Overview

This document resolves GPG key issues for the following repositories:

* MongoDB 4.4
* Puppet 5
* InfluxData

---

# 1. Backup Existing Keyring

```bash
cp /etc/apt/trusted.gpg /root/trusted.gpg.bak.$(date +%F)
```

---

# 2. Fix MongoDB 4.4 Repository Key

### Remove the expired MongoDB key

```bash
apt-key del 656408E390CFB1F5
```

### Import the latest MongoDB 4.4 signing key

```bash
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | apt-key add -
```

### Verify

```bash
apt-key list | grep -A3 MongoDB
```

---

# 3. Fix Puppet 5 Repository Key

### Remove the expired Puppet key

```bash
apt-key del 4528B6CD9E61EF26
```

### Remove the old Puppet keyring

```bash
rm -f /etc/apt/trusted.gpg.d/puppet5-keyring.gpg
```

### Import the latest Puppet signing key (single command)

```bash
curl -fsSL https://apt.puppet.com/DEB-GPG-KEY-future | gpg --dearmor > /etc/apt/trusted.gpg.d/puppet5-keyring.gpg
```

### Verify

```bash
gpg --show-keys /etc/apt/trusted.gpg.d/puppet5-keyring.gpg
```

---

# 4. Fix InfluxData Repository Key

### Import the latest InfluxData signing key

```bash
curl -fsSL https://repos.influxdata.com/influxdata-archive.key | apt-key add -
```

### Verify

```bash
apt-key list | grep -A3 Influx
```

---

# 5. Clean APT Cache

```bash
apt-get clean
rm -rf /var/lib/apt/lists/*
```

---

# 6. Update Package Lists

```bash
apt-get update
```

Expected output:

```text
Reading package lists... Done
```

---

# 7. Verify Puppet

```bash
puppet agent -t
```

or

```bash
puppet agent -td
```

---

# Result

After completing the above steps:

* ✅ MongoDB repository key is updated.
* ✅ Puppet repository key is updated.
* ✅ InfluxData repository key is updated.
* ✅ `apt-get update` completes successfully.
* ✅ Puppet no longer fails with `Exec[apt_update] returned 100`.


Single CDM for above issue

# Fix GPG Keys for MongoDB, Puppet, and InfluxDB (Debian Buster)

## 1. MongoDB 4.4

```bash
apt-key del 656408E390CFB1F5 && wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | apt-key add -
```

---

## 2. Puppet 5

```bash
apt-key del 4528B6CD9E61EF26 && rm -f /etc/apt/trusted.gpg.d/puppet5-keyring.gpg && curl -fsSL https://apt.puppet.com/DEB-GPG-KEY-future | gpg --dearmor > /etc/apt/trusted.gpg.d/puppet5-keyring.gpg
```

---

## 3. InfluxData

```bash
curl -fsSL https://repos.influxdata.com/influxdata-archive.key | apt-key add -
```

---

## Refresh APT Cache

```bash
apt-get clean && rm -rf /var/lib/apt/lists/* && apt-get update
```

---

## Verify

```bash
puppet agent -t
```

Expected result:

* ✅ MongoDB repository key updated.
* ✅ Puppet repository key updated.
* ✅ InfluxData repository key updated.
* ✅ `apt-get update` completes successfully.
* ✅ Puppet runs without `Exec[apt_update] returned 100`.








