
---

# SSH User Setup for Node.js Development (with restricted access)

This guide provides the steps to create an SSH user who can work on their Node.js project, install and manage local npm packages, and have access to only their own project directory, without the ability to install global npm packages or access other users' directories.

---

## 1. **Create a New SSH User**

Start by creating a new user on your server, e.g., `user1`.

```bash
sudo adduser user1
```

This will prompt you to set a password and some other information for the user.

---

## 2. **Create a Project Directory for the User**

Create a directory under `/var/www` for the user’s project and set the necessary permissions.

```bash
sudo mkdir /var/www/user1
sudo chown user1:user1 /var/www/user1
sudo chmod 755 /var/www/user1
```

This ensures that `user1` can access, create, and modify files inside `/var/www/user1` but cannot access directories belonging to other users.

---

## 3. **Install Node.js and npm**

To allow the user to use Node.js and npm, install these globally on the system:

```bash
sudo apt update
sudo apt install nodejs npm
```

Verify the installation:

```bash
node -v
npm -v
```

---

## 4. **Ensure User Can Use Global npm Packages (but Cannot Install Them)**

By default, npm installs global packages in system directories like `/usr/lib/node_modules` or `/usr/local/lib/node_modules`. These directories should only be writable by the root user, which ensures that regular users, including `user1`, cannot install global npm packages.

To restrict write access to these global directories:

```bash
sudo chown -R root:root /usr/lib/node_modules /usr/local/lib/node_modules
sudo chmod -R 755 /usr/lib/node_modules /usr/local/lib/node_modules
```

This ensures that `user1` can **use** globally installed npm packages, but cannot modify or install new ones.

---

## 5. **Allow User to Install Local npm Packages (Project-specific)**

`user1` can install npm packages for their project (but not globally). To do this, simply allow them to work in their project directory.

### Example:

```bash
cd /var/www/user1
mkdir my_project
cd my_project
npm init -y
npm install lodash  # Install locally for the project
```

This will install `lodash` (or any other package) in the `node_modules` directory within `/var/www/user1/my_project`.

---

## 6. **Restrict User Access to Their Own Directory (Optional: Using Chroot)**

To ensure that `user1` cannot access other users' directories or sensitive system files, you can configure SSH to chroot them to their own directory.

Edit the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Add the following to restrict `user1`:

```bash
Match User user1
    ChrootDirectory /var/www/user1
    ForceCommand /bin/bash
    AllowTcpForwarding no
```

This configuration locks `user1` to their project directory after logging in via SSH and prevents them from accessing other parts of the system.

---

## 7. **Test the Configuration**

1. **Log in as `user1`** via SSH:

```bash
ssh user1@your_server_ip
```

2. **Verify Directory Access:**

Make sure that `user1` can only access their own project directory and cannot access others:

```bash
ls /var/www/user1  # Should work
touch /var/www/user1/my_project/file.js  # Should work
ls /var/www/user2  # Should fail
```

3. **Test npm Global Packages:**

`user1` should be able to **use** globally installed npm packages but not install them:

```bash
npm list -g --depth=0  # This will show globally installed packages
```

4. **Test npm Local Packages:**

`user1` should be able to install local packages for their project:

```bash
cd /var/www/user1/my_project
npm install lodash  # Should work
```

---

## 8. **Optional: Use nvm (Node Version Manager) for Project-Specific Node Versions**

To allow `user1` to manage their own version of Node.js (for their project), you can install **nvm** (Node Version Manager).

1. Switch to `user1`:

```bash
sudo su - user1
```

2. Install nvm:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
source ~/.bashrc
```

3. Install a specific version of Node.js:

```bash
nvm install node
```

Now, `user1` can use a specific version of Node.js for their project without needing root access.

---

## Summary:

1. **User Creation:** You created a restricted user (`user1`) with access to their own project directory.
2. **Node.js & npm Installation:** You installed Node.js and npm globally, and set permissions to prevent the user from installing global npm packages.
3. **Local Project Management:** `user1` can work on their project and install local npm packages.
4. **Access Restrictions:** SSH chroot ensures `user1` is locked to their directory, preventing access to other users' directories or system files.
5. **Optional:** Installing `nvm` allows the user to manage their own version of Node.js for the project.

---
