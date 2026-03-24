# Installing MySQL

## Overview

MySQL is available for all major operating systems. This guide covers three installation paths — macOS, Ubuntu/Debian Linux, and Docker — and includes post-installation verification steps.

---

## macOS (via Homebrew)

[Homebrew](https://brew.sh) is the easiest way to install MySQL on macOS.

### Install

```bash
# 1. Update Homebrew
brew update

# 2. Install MySQL 8
brew install mysql

# 3. Start MySQL as a background service (auto-starts on login)
brew services start mysql

# 4. Verify the version
mysql --version
# Expected: mysql  Ver 8.x.x ...
```

### Connect After Install

By default, the Homebrew install has no root password:

```bash
mysql -u root
```

### Harden the Installation

```bash
mysql_secure_installation
```

This interactive wizard will:

- Set (or change) the `root` password
- Remove anonymous users
- Disable remote `root` login
- Remove the test database
- Reload privilege tables

---

## Ubuntu / Debian Linux

```bash
# 1. Update the package index
sudo apt update

# 2. Install MySQL Server
sudo apt install mysql-server -y

# 3. Check the service is running
sudo systemctl status mysql

# 4. Enable auto-start on boot
sudo systemctl enable mysql

# 5. Harden the installation
sudo mysql_secure_installation
```

### Connect as Root on Ubuntu

Ubuntu installs MySQL using the `auth_socket` plugin for root — meaning the OS user `root` authenticates automatically:

```bash
sudo mysql
# or
sudo mysql -u root
```

To switch to password-based auth for root:

```sql
ALTER USER 'root'@'localhost'
  IDENTIFIED WITH mysql_native_password BY 'your_strong_password';
FLUSH PRIVILEGES;
```

---

## Docker (Recommended for Development)

Docker is the cleanest option for local development — isolated, reproducible, no system-level installation required.

### Quick Start

```bash
# Pull and run MySQL 8
docker run --name mysql-dev \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=bookstore \
  -p 3306:3306 \
  -d mysql:8

# Connect from your host machine
mysql -u root -p -h 127.0.0.1 -P 3306
```

### Docker Compose (Recommended — with Persistent Storage)

```yaml
# docker-compose.yml
services:
  db:
    image: mysql:8
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: bookstore
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

```bash
# Start in background
docker compose up -d

# Connect
mysql -u root -p -h 127.0.0.1 -P 3306
```

---

## Verifying Your Installation

After any installation method, connect to MySQL and run these checks:

```sql
-- Check the server version
SELECT VERSION();

-- List all built-in system databases
SHOW DATABASES;
-- Expected: information_schema, mysql, performance_schema, sys

-- Confirm InnoDB is the default storage engine
SHOW ENGINES;
-- InnoDB should show: Support = DEFAULT

-- Show server status
STATUS;
```

---

## MySQL Service Management

| Action | macOS (Homebrew) | Linux (systemd) | Docker |
|--------|-----------------|-----------------|--------|
| Start | `brew services start mysql` | `sudo systemctl start mysql` | `docker start mysql-dev` |
| Stop | `brew services stop mysql` | `sudo systemctl stop mysql` | `docker stop mysql-dev` |
| Restart | `brew services restart mysql` | `sudo systemctl restart mysql` | `docker restart mysql-dev` |
| Status | `brew services list` | `sudo systemctl status mysql` | `docker ps` |

---

## Default Configuration Reference

| Setting | Default Value |
|---------|--------------|
| Port | `3306` |
| Default host | `localhost` |
| Default storage engine | `InnoDB` |
| Config file (Ubuntu) | `/etc/mysql/my.cnf` |
| Config file (macOS Homebrew) | `/opt/homebrew/etc/my.cnf` |
| Data directory (Ubuntu) | `/var/lib/mysql/` |

---

## Best Practices

- **Never** use `root` for application connections — create a dedicated user with minimal privileges.
- Always run `mysql_secure_installation` after any server installation.
- Use **Docker** for local development to keep your OS clean and easily reset state.
- Keep MySQL updated to receive security patches (`brew upgrade mysql` / `apt upgrade mysql-server`).

## Common Mistakes

- Forgetting to start the MySQL service before connecting — causes `Can't connect to MySQL server` errors.
- Connecting with `-h localhost` when MySQL is set to use a Unix socket (use `-h 127.0.0.1` to force TCP).
- Using the same weak password across all environments (dev, staging, production).

## Next Step

Continue to [5-database.md](5-database.md).

