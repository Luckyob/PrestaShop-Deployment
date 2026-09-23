# 🛒 PrestaShop Deployment on Microsoft Azure

## 🌟 Project Overview

This project demonstrates the deployment of **PrestaShop 9.1.5** on **Microsoft Azure** using a **two-tier cloud architecture**.

The deployment consists of two separate Ubuntu Linux virtual machines:

* 🖥️ **PrestaShop Application VM** — runs Apache, PHP, and hosts the PrestaShop storefront.
* 🗄️ **MariaDB Database VM** — runs MariaDB and stores the PrestaShop database separately from the application server.

The two virtual machines communicate through a **private Azure Virtual Network**, allowing the application server to access the database without exposing MariaDB directly to the public Internet.

The completed deployment provides a publicly accessible PrestaShop storefront while keeping the database on a separate server.

---

## 🏗️ System Architecture

```text
                         🌍 Internet
                             │
                             │ HTTP :80
                             ▼
                ┌─────────────────────────┐
                │ 🖥️ PrestaShop          │
                │    Application VM       │
                │                         │
                │ 🐧 Ubuntu 24.04 LTS    │
                │ 🌐 Apache 2.4           │
                │ 🐘 PHP 8.3             │
                │ 🛒 PrestaShop 9.1.5    │
                │                         │
                │ Public IP: 9.160.105.142│
                │ Private IP: 172.16.0.4  │
                └────────────┬────────────┘
                             │
                             │ 🔐 Private Azure VNet
                             │ TCP :3306
                             ▼
                ┌─────────────────────────┐
                │ 🗄️ MariaDB Database VM │
                │                         │
                │ 🐧 Ubuntu 24.04 LTS    │
                │ 🗄️ MariaDB 10.11       │
                │                         │
                │ Private IP: 172.16.0.5  │
                │ Database: prestashop_db │
                └─────────────────────────┘
```

### 🔄 Request Flow

```text
🌍 User Browser
       │
       │ HTTP :80
       ▼
🌐 Azure Public IP
9.160.105.142
       │
       ▼
🖥️ PrestaShop Application VM
172.16.0.4
       │
       │ TCP :3306
       │ 🔐 Private Network
       ▼
🗄️ MariaDB Database VM
172.16.0.5
```

---

## ☁️ Azure Resources

### 🖥️ PrestaShop Application VM

| ⚙️ Configuration  | 📋 Details              |
| ----------------- | ----------------------- |
| VM Name           | `prestashop-vm`         |
| Operating System  | Ubuntu Server 24.04 LTS |
| VM Size           | Standard B2ats_v2       |
| vCPUs             | 2                       |
| RAM               | 1 GiB                   |
| Region            | Belgium Central         |
| Availability Zone | Zone 2                  |
| Private IP        | `172.16.0.4`            |
| Public IP         | `9.160.105.142`         |

The application VM is responsible for hosting PrestaShop and serving the storefront to users over HTTP.

### 🗄️ Database VM

| ⚙️ Configuration | 📋 Details              |
| ---------------- | ----------------------- |
| VM Name          | `prestashop-db-vm`      |
| Operating System | Ubuntu Server 24.04 LTS |
| VM Size          | Standard B2ats_v2       |
| vCPUs            | 2                       |
| RAM              | 1 GiB                   |
| Region           | Belgium Central         |
| Private IP       | `172.16.0.5`            |
| Database         | MariaDB 10.11           |
| Port             | `3306`                  |

The database VM is dedicated to storing the data used by PrestaShop.

---

## 🌐 Azure Virtual Network

Both virtual machines are connected to the same Azure Virtual Network.

| 🌐 Resource     | 📋 Name                 |
| --------------- | ----------------------- |
| Virtual Network | `vnet-belgiumcentral-1` |
| Subnet          | `snet-belgiumcentral-1` |
| Application VM  | `172.16.0.4`            |
| Database VM     | `172.16.0.5`            |

The private network allows the PrestaShop application server to communicate with MariaDB without sending database traffic over the public Internet.

```text
🖥️ 172.16.0.4
PrestaShop VM
      │
      │ 🔌 TCP 3306
      ▼
🗄️ 172.16.0.5
MariaDB VM
```

---

## 🧰 Software Stack

### 🖥️ Application Server

* 🐧 Ubuntu Server 24.04 LTS
* 🌐 Apache 2.4
* 🐘 PHP 8.3
* 🛒 PrestaShop 9.1.5

Apache receives HTTP requests and serves the PrestaShop application, while PHP provides the runtime environment required by PrestaShop.

### 🗄️ Database Server

* 🐧 Ubuntu Server 24.04 LTS
* 🗄️ MariaDB 10.11
* 💾 Database: `prestashop_db`
* 👤 Database user: `prestashop_user`
* 🔌 Port: `3306`

---

## 🔐 Network Security

Azure **Network Security Groups (NSGs)** were used to control network traffic to the virtual machines.

### 🌐 Application VM

HTTP traffic was allowed through port `80`.

| ⚙️ Setting | 📋 Value |
| ---------- | -------- |
| Protocol   | TCP      |
| Port       | 80       |
| Source     | Any      |
| Action     | Allow    |

This allows users to access the PrestaShop storefront through the public IP address.

### 🗄️ Database VM

MariaDB uses port `3306`.

Access to this port is restricted so that the PrestaShop application VM can communicate with the database server.

```text
📤 Source:       172.16.0.4
📥 Destination:  172.16.0.5
🔌 Protocol:     TCP
🚪 Port:         3306
```

The MariaDB server is not intended to be directly accessible from the public Internet.

🔑 SSH access is also configured for server administration using SSH authentication.

---

## 🗄️ MariaDB Configuration

MariaDB was installed on the dedicated database VM.

A database was created specifically for PrestaShop:

```text
💾 Database: prestashop_db
```

A dedicated database user was also created:

```text
👤 User: prestashop_user
```

PrestaShop was configured with the following database connection details:

```text
🖥️ Database Server: 172.16.0.5
💾 Database Name:   prestashop_db
👤 Database User:   prestashop_user
🔌 Database Port:   3306
```

This allows the application and database layers to operate independently.

---

## 🔗 Database Connectivity Test

Connectivity between the application VM and database VM was tested from the PrestaShop application server.

The following command was used:

```bash
mysql -h 172.16.0.5 -u prestashop_user -p -e "SHOW DATABASES;"
```

### ✅ Successful Result

```text
+--------------------+
| Database           |
+--------------------+
| information_schema |
| prestashop_db      |
+--------------------+
```

This confirmed that:

* ✅ The PrestaShop VM could communicate with the MariaDB VM.
* ✅ Communication was taking place through the private network.
* ✅ The `prestashop_user` account could access the database.
* ✅ The `prestashop_db` database was available to the application server.

---

## 🌐 Apache Web Server

Apache was installed on the PrestaShop application VM.

The Apache service was verified using:

```bash
sudo systemctl status apache2 --no-pager
```

The service reported:

```text
Active: active (running)
```

### ✅ Result

This confirmed that Apache was running successfully and ready to serve the PrestaShop application.

---

## 🛒 PrestaShop Installation

PrestaShop **9.1.5** was installed on the application VM.

During the installation process, the following database configuration was provided:

```text
🖥️ Database server: 172.16.0.5
💾 Database name:   prestashop_db
👤 Database user:   prestashop_user
🔌 Database port:   3306
```

The PrestaShop installation wizard successfully connected to the remote MariaDB server and completed the installation.

The application was then made available through the public IP address of the application VM.

---

## 🌍 Public Website Access

The PrestaShop storefront is accessible through:

```text
http://9.160.105.142
```

The public IP belongs to the PrestaShop application VM.

When a user accesses the address, the request is received by Apache on the application VM. PrestaShop then communicates with the MariaDB database through the database VM's private IP address.

```text
🌍 Internet
     │
     │ HTTP :80
     ▼
🌐 9.160.105.142
     │
     ▼
🖥️ PrestaShop VM
172.16.0.4
     │
     │ 🔐 TCP :3306
     ▼
🗄️ MariaDB VM
172.16.0.5
```

---

## 🧪 Website Verification

The website was tested directly from the application server using:

```bash
curl -I http://9.160.105.142
```

The server returned:

```text
HTTP/1.1 200 OK
```

The response also showed:

```text
Server: Apache/2.4.58 (Ubuntu)
Content-Type: text/html; charset=utf-8
```

### ✅ Verification Result

The `200 OK` response confirmed that the web server successfully responded to the HTTP request and that the PrestaShop application was being served.

The storefront was also opened successfully in a web browser using:

```text
http://9.160.105.142
```

---

## 🔥 Ubuntu Firewall

The Ubuntu firewall was checked using:

```bash
sudo ufw status
```

The result was:

```text
Status: inactive
```

Therefore, UFW was not blocking HTTP traffic on the application server.

Azure Network Security Group rules were used to control external network access to the virtual machine.

---

## 🔒 Security Considerations

Several security measures were implemented as part of the deployment.

### 🔑 SSH Authentication

SSH key authentication is used for server administration.

### 🔐 Private Database Communication

The application connects to MariaDB using the database server's private IP address:

```text
172.16.0.5
```

### 🛡️ Database Network Restriction

MariaDB port `3306` is restricted to communication from the PrestaShop application server.

### 🚫 No Public Database Access

The MariaDB server is not intended to be directly accessible from the public Internet.

### 🌐 Controlled Web Access

HTTP traffic is allowed through port `80` for the public storefront.

> ⚠️ This deployment demonstrates a functional cloud architecture. Additional security controls should be implemented before using the environment as a production e-commerce platform.

---

## 🚀 Future Improvements

The following improvements could be implemented for a production environment:

### 🔒 HTTPS/TLS

Configure HTTPS using a valid TLS certificate instead of serving the website over HTTP.

### 🌐 Custom Domain

Connect the deployment to a domain name instead of accessing the storefront through the public IP address.

### 🔄 HTTP to HTTPS Redirect

Configure Apache to automatically redirect HTTP requests to HTTPS.

### 🛡️ Stronger SSH Restrictions

Restrict SSH access to trusted IP addresses rather than allowing unrestricted SSH access from the Internet.

### 💾 Automated Backups

Configure regular automated backups of the PrestaShop database.

### 📊 Monitoring

Use Azure monitoring and logging services to monitor:

* 📈 Server performance
* 🟢 Application availability
* 🌐 Network activity
* 🔐 Security events
* 💻 Resource utilization

### ⚖️ Load Balancing

For a larger production environment, an Azure load balancer could distribute traffic across multiple application servers.

### 📈 Scalability

Additional application servers and Azure services could be introduced as traffic increases.

### 🔑 Secret Management

Database credentials and other sensitive configuration values should be stored securely using a dedicated secret-management solution rather than being exposed in application configuration files.

---

## 🎯 Final Architecture

The completed environment consists of:

```text
                         🌍 PUBLIC INTERNET
                                │
                                │ HTTP :80
                                ▼
                     🌐 Public IP: 9.160.105.142
                                │
                                ▼
                ┌────────────────────────────┐
                │ 🖥️ PRESTASHOP APPLICATION │
                │          VM                │
                │                            │
                │ 🐧 Ubuntu 24.04 LTS       │
                │ 🌐 Apache 2.4              │
                │ 🐘 PHP 8.3                │
                │ 🛒 PrestaShop 9.1.5       │
                │                            │
                │ 🔐 Private IP: 172.16.0.4 │
                └──────────────┬─────────────┘
                               │
                               │ 🔐 Private Azure VNet
                               │ 🔌 TCP :3306
                               ▼
                ┌────────────────────────────┐
                │ 🗄️ MARIADB DATABASE VM     │
                │                            │
                │ 🐧 Ubuntu 24.04 LTS       │
                │ 🗄️ MariaDB 10.11          │
                │                            │
                │ 💾 Database: prestashop_db │
                │ 👤 User: prestashop_user   │
                │ 🔐 Private IP: 172.16.0.5  │
                │ 🔌 TCP 3306               │
                └────────────────────────────┘
```

---

## ✅ Deployment Verification

The following components were successfully completed and tested:

* ☑️ Azure virtual machines deployed
* ☑️ Application and database servers separated
* ☑️ Azure Virtual Network configured
* ☑️ Private IP communication established
* ☑️ Apache installed and running
* ☑️ PHP configured
* ☑️ PrestaShop 9.1.5 installed
* ☑️ MariaDB configured
* ☑️ `prestashop_db` database created
* ☑️ `prestashop_user` database account created
* ☑️ Database connectivity verified
* ☑️ Private application-to-database communication established
* ☑️ Azure Network Security Group configured
* ☑️ HTTP port 80 configured
* ☑️ PrestaShop storefront successfully loaded
* ☑️ HTTP request returned `200 OK`

---

## 📸 Deployment Screenshots
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/d76fa65f-7239-463f-b69a-9ba456220724" />
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/579d0584-7886-47ed-af32-743b88c8e244" />
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/c28035b5-ec43-4f18-a6f9-84923a9dd1ed" />
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/3ed54f8a-213d-4018-adfb-3fd8f696bfa8" />
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/8a477f94-80d1-47ce-be42-5b6dd58b7c25" />
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/d8e43a38-66eb-4c6b-a964-a262351f0611" />
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/632b46ef-f7b5-4bbf-98b4-c1e82049f4de" />
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/b521c271-9dee-4bed-bc5f-f23f9caf46f2" />
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/c73484fb-bc4e-4c34-9daf-3a8ab47c3a81" />
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/cedcd408-e429-4f2d-aa11-975b07d717d1" />
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/c7399785-df08-4f6a-9c89-9d670064413b" />

<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/2d28472f-29c9-4f31-a047-1163d0616133" />
<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/55c0b85d-896f-410d-9c4a-c8a5af07c71a" />










---

## 👨‍💻 Project Author

**Lucky Samuel**
