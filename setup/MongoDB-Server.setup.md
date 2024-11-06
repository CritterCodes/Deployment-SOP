### **SOP: Setting Up a MongoDB Server on a VPS**

#### **1. Install MongoDB Community Edition**

1. **Update the Package Database**:
   ```bash
   sudo apt update
   ```

2. **Install MongoDB**:
   Use the official MongoDB package for your operating system. For example, on Ubuntu:

   ```bash
   wget -qO - https://www.mongodb.org/static/pgp/server-8.0.asc | sudo apt-key add -
   echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
   sudo apt update
   sudo apt install -y mongodb-org
   ```

3. **Start MongoDB**:
   ```bash
   sudo systemctl start mongod
   sudo systemctl enable mongod
   ```

#### **2. Verify MongoDB Installation**

Check if MongoDB is running:

```bash
sudo systemctl status mongod
```

You should see `Active: active (running)` if MongoDB started successfully.

#### **3. Set Up Admin Credentials**

1. **Open MongoDB Shell**:
   Connect to MongoDB with `mongosh`:

   ```bash
   mongosh
   ```

2. **Switch to the Admin Database**:
   ```javascript
   use admin
   ```

3. **Create an Admin User**:
   Run the following command to set up an admin user with `root` privileges:

   ```javascript
   db.createUser({
     user: "adminUser",
     pwd: "strongPassword",
     roles: [ { role: "root", db: "admin" } ]
   })
   ```

   Replace `"adminUser"` and `"strongPassword"` with your preferred username and a strong password.

4. **Exit the MongoDB Shell**:
   Type `exit` to leave `mongosh`.

#### **4. Update MongoDB Configuration to Require Authentication**

1. **Open the MongoDB Configuration File**:
   Edit the MongoDB config file, usually located at `/etc/mongod.conf`:

   ```bash
   sudo nano /etc/mongod.conf
   ```

2. **Set the Network Interface**:
   Under the `net` section, set `bindIp` to `0.0.0.0` to allow remote access, or `127.0.0.1` to limit it to local access:

   ```yaml
   net:
     port: 27017
     bindIp: 0.0.0.0
   ```

3. **Enable Authorization**:
   Uncomment or add the `security` section to require user authentication:

   ```yaml
   security:
     authorization: "enabled"
   ```

4. **Save and Exit**:
   Save changes and exit the editor (for nano, press `CTRL+X`, then `Y`, then `Enter`).

5. **Restart MongoDB**:
   Apply the configuration changes by restarting MongoDB:

   ```bash
   sudo systemctl restart mongod
   ```

#### **5. Verify the Setup**

1. **Connect with Authentication**:
   Now, try connecting to MongoDB using the admin credentials:

   ```bash
   mongosh --username adminUser --password strongPassword --authenticationDatabase admin
   ```

2. **Check Database Access**:
   Once authenticated, you should be able to perform administrative tasks. Use `show dbs` to verify access:

   ```javascript
   show dbs
   ```

   If you see the list of databases, authentication is working correctly.

#### **6. (Optional) Configure Firewall**

To allow external access, configure your firewall to permit connections to MongoDB on port `27017`. For example, using UFW:

```bash
sudo ufw allow from <your-ip-address> to any port 27017
```

Replace `<your-ip-address>` with your IP or `0.0.0.0/0` for all addresses (only if secure).

#### **7. Final Check**

Try connecting from a remote client (like MongoDB Compass) using the following format:

```plaintext
mongodb://adminUser:strongPassword@your-vps-ip-or-domain:27017/admin
```

### **Notes**
- Avoid using `0.0.0.0` for `bindIp` and open firewall access to `27017` for all unless necessary.
- If you later need to configure SSL, follow additional MongoDB documentation for SSL/TLS setup.
