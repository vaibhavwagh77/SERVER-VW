# 🛠️ LDAP Setup Guide (Master + Client)

This guide walks you through setting up an LDAP (Lightweight Directory Access Protocol) Server (Master) and integrating a Client node for centralized authentication.

---

## 🧠 What is LDAP?

LDAP is an open, vendor-neutral, industry-standard protocol for accessing and maintaining distributed directory information services over an IP network. It is commonly used for authentication, authorization, and user management.

---

## 🖥️ Server Environment (Lab Setup)

* **LDAP Master Server**: Ubuntu 22.04 LTS (or similar)
* **LDAP Client Node**: Ubuntu 22.04 LTS (or similar)
* **Hostname**: `ldap-master`
* **Domain Name**: `ldap.example.com`
* **IP Address (Server)**: `192.168.1.10`
* **IP Address (Client)**: `192.168.1.20`

---

## 🔧 Master LDAP Server Setup (ldap-master)

### Step 1: Set Hostname and Host Mapping

```bash
sudo hostnamectl set-hostname ldap-master
echo "192.168.1.10 ldap-master" | sudo tee -a /etc/hosts
```

### Step 2: Install Required Packages

```bash
sudo apt update
sudo apt install slapd ldap-utils -y
```

### Step 3: Configure slapd

Run the configuration tool:

```bash
sudo dpkg-reconfigure slapd
```

Follow these prompts:

* Omit OpenLDAP server configuration? → No
* DNS domain name: `example.com`
* Organization name: `Example Inc.`
* Admin password: `admin@123`
* Database backend: MDB (default)
* Remove database when slapd is purged? → No
* Move old database? → Yes

### Step 4: Verify LDAP Service

```bash
sudo systemctl status slapd
```

---

## 📁 Base LDAP Directory Structure

### Step 5: Create Base LDIF File

Create a file named `base.ldif`:

```ldif
# base.ldif

dn: ou=People,dc=example,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Groups,dc=example,dc=com
objectClass: organizationalUnit
ou: Groups
```

### Step 6: Add Base Structure to LDAP

```bash
sudo ldapadd -x -D cn=admin,dc=example,dc=com -W -f base.ldif
```

---

## 👥 Add Users and Groups

### Step 7: Create User LDIF File

Create `user.ldif`:

```ldif
# user.ldif

dn: uid=john,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: John Doe
sn: Doe
uid: john
uidNumber: 10000
gidNumber: 10000
homeDirectory: /home/john
loginShell: /bin/bash
userPassword: {SSHA}hashed-password
```

Generate password hash:

```bash
slappasswd
```

### Step 8: Add User

```bash
sudo ldapadd -x -D cn=admin,dc=example,dc=com -W -f user.ldif
```

### Step 9: Create Group LDIF

```ldif
# group.ldif

dn: cn=users,ou=Groups,dc=example,dc=com
objectClass: posixGroup
cn: users
gidNumber: 10000
memberUid: john
```

### Step 10: Add Group

```bash
sudo ldapadd -x -D cn=admin,dc=example,dc=com -W -f group.ldif
```

---

## 🌐 Web Interface (phpLDAPadmin - Optional)

### Step 11: Install phpLDAPadmin

```bash
sudo apt install phpldapadmin -y
```

### Step 12: Configure phpLDAPadmin

Edit `/etc/phpldapadmin/config.php`:

```php
$servers->setValue('server','host','127.0.0.1');
$servers->setValue('login','bind_id','cn=admin,dc=example,dc=com');
```

Allow access in Apache if necessary.

### Step 13: Access phpLDAPadmin

Go to: `http://<your-ip>/phpldapadmin`

---

## 💻 LDAP Client Configuration

### Step 1: Set Hostname and Map Server IP

```bash
sudo hostnamectl set-hostname ldap-client
echo "192.168.1.10 ldap-master" | sudo tee -a /etc/hosts
```

### Step 2: Install Client Packages

```bash
sudo apt update
sudo apt install libnss-ldap libpam-ldap ldap-utils nscd -y
```

### Step 3: Configuration Prompts

You’ll be asked:

* LDAP URI: `ldap://ldap-master`
* Base DN: `dc=example,dc=com`
* LDAP Version: 3
* Make local root Database admin: Yes
* LDAP account for root: `cn=admin,dc=example,dc=com`
* Password: admin password

### Step 4: Configure NSS and PAM

Check `/etc/nsswitch.conf`:

```conf
passwd:         files ldap
group:          files ldap
shadow:         files ldap
```

Enable home directory creation:

```bash
sudo pam-auth-update
```

Check: \[✓] Create home directory on login

### Step 5: Restart NSS & Test

```bash
sudo systemctl restart nscd
getent passwd john
```

### Step 6: Login Test

Try switching to the LDAP user:

```bash
su - john
```

---

## 🧪 Testing & Troubleshooting

* `ldapsearch -x`: Simple LDAP search
* `getent passwd <username>`: User fetched correctly?
* `tail -f /var/log/syslog`: Check for login/auth errors

---

## ✅ Summary

* Master LDAP server installed with slapd and base structure
* Users and groups created using `.ldif` files
* phpLDAPadmin for web-based LDAP management
* Clients configured with PAM + NSS to authenticate against LDAP

---

## 📚 References

* [https://ubuntu.com/server/docs/service-ldap](https://ubuntu.com/server/docs/service-ldap)
* [https://wiki.debian.org/LDAP](https://wiki.debian.org/LDAP)
* [https://www.openldap.org/doc](https://www.openldap.org/doc)

---

