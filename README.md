Here’s a step-by-step Standard Operating Procedure (SOP) for deploying a Node.js application using **Nginx** as a reverse proxy, **Certbot** for SSL certificates, and **PM2** for process management. This SOP assumes your app is deployed on a Linux server.

### Prerequisites
1. **Nginx** installed.
2. **Certbot** installed (for automatic SSL certificate management).
3. **PM2** installed globally (for managing Node.js processes).

---

### Step 1: SSH into the Server
Start by accessing your server via SSH:

```bash
ssh your_user@your_server_ip
```

### Step 2: Set Up PM2 to Run Your Node.js App
1. **Navigate to your project directory**:
   ```bash
   cd /path/to/your/project
   ```

2. **Start the Node.js application using PM2**:
   Run PM2 with the environment-specific start command:
   ```bash
   pm2 start npm --name "app_name" -- start
   ```

   This command runs `npm start` and assigns the process a name (`app_name`).

3. **Save the PM2 process list** (to ensure it restarts on server reboot):
   ```bash
   pm2 save
   ```

4. **Set PM2 to auto-start on reboot**:
   ```bash
   pm2 startup
   ```

   Follow any additional instructions provided by PM2 after running this command.

---

### Step 3: Configure Nginx as a Reverse Proxy
1. **Create a new Nginx configuration file** for your app:
   ```bash
   sudo nano /etc/nginx/sites-available/app_name
   ```

2. **Add the following configuration** (adjust based on your app’s port, typically 3000):

   ```nginx
   server {
       listen 80;
       server_name yourdomain.com www.yourdomain.com;

       location / {
           proxy_pass http://localhost:3000;  # Adjust the port if necessary
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```

3. **Enable the configuration** by creating a symbolic link:
   ```bash
   sudo ln -s /etc/nginx/sites-available/app_name /etc/nginx/sites-enabled/
   ```

4. **Test the Nginx configuration** for any syntax errors:
   ```bash
   sudo nginx -t
   ```

5. **Reload Nginx** to apply the changes:
   ```bash
   sudo systemctl reload nginx
   ```

---

### Step 4: Secure the Application with SSL using Certbot
1. **Run Certbot** to obtain an SSL certificate for your domain:
   ```bash
   sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
   ```

   Certbot will automatically configure SSL in your Nginx file if the setup is successful.

2. **Verify SSL renewal**:
   Certbot should set up a cron job to auto-renew your SSL certificate. You can verify renewal by running:

   ```bash
   sudo certbot renew --dry-run
   ```

---

### Step 5: Verify Deployment and PM2 Process Monitoring
1. **Check the PM2 process list** to ensure your app is running:
   ```bash
   pm2 list
   ```

2. **Restart Nginx and PM2 if needed**:
   ```bash
   sudo systemctl restart nginx
   pm2 restart all
   ```

3. **Access the application** via your domain (https://yourdomain.com) to verify everything is functioning as expected.

---

### Summary of Commands
Here’s a quick command summary for deployment:

```bash
# SSH into the server
ssh your_user@your_server_ip

# Start Node.js app with PM2
cd /path/to/your/project
pm2 start npm --name "app_name" -- start
pm2 save
pm2 startup

# Configure Nginx reverse proxy
sudo nano /etc/nginx/sites-available/app_name
sudo ln -s /etc/nginx/sites-available/app_name /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# Set up SSL with Certbot
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
sudo certbot renew --dry-run

# Verify PM2 and Nginx
pm2 list
sudo systemctl restart nginx
pm2 restart all
```

This SOP should provide you with a streamlined, reliable deployment process for your Node.js app. Let me know if you need any adjustments or additions!
