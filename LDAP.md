# LDAP Setup from Scratch - Full Guide

## 🏢 What is LDAP?

LDAP (Lightweight Directory Access Protocol) is a protocol used for accessing and maintaining distributed directory information services over an IP network. It is commonly used for:

* Centralized authentication
* User and group management
* Authorization services

## 📃 LDAP Components

| Component      | Purpose                                      |
| -------------- | -------------------------------------------- |
| `slapd`        | LDAP server daemon                           |
| `ldap-utils`   | CLI tools for LDAP                           |
| `phpldapadmin` | Web-based GUI for LDAP management (optional) |
| `libnss-ldap`  | NSS module for LDAP integration              |
| `libpam-ldap`  | PAM module for LDAP-based authentication     |

---

## 🛠️ Step 1: Install LDAP Server (Ubuntu)

### On the LDAP Server:

```bash
sudo apt update
sudo apt install slapd ldap-utils -y
```

To reconfigure:

```bash
sudo dpkg-reconfigure slapd
```

* Provide Admin password
* Set base DN: `dc=example,dc=com`

---

## 🧰 Step 2: LDAP Directory Tree Structure

```
dc=example,dc=com
├── ou=People
│   ├── uid=vaibhav
├── ou=Groups
    ├── cn=developers
```

* `dc=example,dc=com`: Base domain
* `ou=People`: Organizational unit for users
* `ou=Groups`: Organizational unit for groups

---

## 🏗️ Step 3: Add Base Structure

### Create `base.ldif`

```ldif
dn: ou=People,dc=example,dc=com
objectClass: organizationalUnit
ou: People

dn: ou=Groups,dc=example,dc=com
objectClass: organizationalUnit
ou: Groups
```

### Apply LDIF:

```bash
ldapadd -x -D cn=admin,dc=example,dc=com -W -f base.ldif
```

---

## 👤 Step 4: Add LDAP Users

### Create `user.ldif`

```ldif
dn: uid=vaibhav,ou=People,dc=example,dc=com
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: Vaibhav Wagh
sn: Wagh
uid: vaibhav
uidNumber: 10000
gidNumber: 10000
homeDirectory: /home/vaibhav
loginShell: /bin/bash
userPassword: {SSHA}HASHED_PASSWORD
```

To generate password hash:

```bash
slappasswd
```

### Add user:

```bash
ldapadd -x -D cn=admin,dc=example,dc=com -W -f user.ldif
```

---

## 👥 Step 5: Add LDAP Groups

### Create `group.ldif`

```ldif
dn: cn=developers,ou=Groups,dc=example,dc=com
objectClass: posixGroup
cn: developers
gidNumber: 10000
memberUid: vaibhav
```

### Add group:

```bash
ldapadd -x -D cn=admin,dc=example,dc=com -W -f group.ldif
```

---

## 🤔 Step 6: Test LDAP Server

```bash
ldapsearch -x -b dc=example,dc=com
```

---

## 💻 Step 7: Configure LDAP Client

### On the Client Machine:

```bash
sudo apt install libnss-ldap libpam-ldap ldap-utils nscd -y
```

### During Configuration Prompts:

* LDAP URI: `ldap://<ldap-server-ip>`
* Base DN: `dc=example,dc=com`
* LDAP Version: 3
* Make local root Database admin: Yes
* LDAP login required: No/Yes based on security

---

## ⚙️ Step 8: NSS and PAM Configuration

### Edit `/etc/nsswitch.conf`:

```text
passwd:         files ldap
group:          files ldap
shadow:         files ldap
```

Check PAM files under `/etc/pam.d/`:

* common-auth
* common-account
* common-password
* common-session

---

## 🔄 Step 9: Restart Services and Test

```bash
sudo systemctl restart nscd
getent passwd vaibhav
```

---

## 🌐 Optional: Web Interface using phpLDAPadmin

```bash
sudo apt install phpldapadmin -y
sudo nano /etc/phpldapadmin/config.php
```

Update config:

```php
$servers->setValue('server','base',array('dc=example,dc=com'));
$servers->setValue('login','bind_id','cn=admin,dc=example,dc=com');
```

Access: `http://<server-ip>/phpldapadmin`

---

## ✅ Final Directory Layout

```
dc=example,dc=com
├── ou=People
│   └── uid=vaibhav
├── ou=Groups
    └── cn=developers
```

---

## ⚠️ Troubleshooting

| Issue                | Fix                                              |
| -------------------- | ------------------------------------------------ |
| Bind errors          | Check slapd status, verify admin DN/password     |
| Users not visible    | Validate LDIF structure and successful ldapadd   |
| Authentication fails | Review NSS/PAM config and restart `nscd` service |
| Firewall issues      | Ensure port 389 is open on server                |

---


