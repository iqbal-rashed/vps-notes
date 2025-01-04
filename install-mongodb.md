# Install MongoDB Community Edition

Follow these steps to install MongoDB Community Edition using the apt package manager.

### Install gnupg and curl if they are not already available:

```bash
sudo apt install gnupg curl
```

### To import the MongoDB public GPG key, run the following command:

```bash
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg \
   --dearmor
```

### Create the list file /etc/apt/sources.list.d/mongodb-org-8.0.list for your version of Ubuntu.

```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```

### Follow next steps to Install Mongodb

```bash
sudo apt update
```

```bash
sudo apt upgrade
```

```bash
sudo apt install mongodb-org
```

### Verify MongoDb Installed or Not

```bash
mongod --version
```

### Enable MongoDB to start at system startup

```bash
sudo systemctl enable mongod
```

### Start MongoDB

```bash
sudo service mongod start
```

### Check MongoDB Status

```bash
sudo service mongod status
```
