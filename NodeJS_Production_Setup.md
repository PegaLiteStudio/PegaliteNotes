# Deploying a Node.js Application with Nginx and PM2

This guide outlines the steps required to deploy your Node.js application on an Ubuntu/Debian server using Nginx as a reverse proxy, nvm for Node.js version management, and PM2 for process management. It also covers setting up SSL with Let’s Encrypt and configuring the UFW firewall.

---

## 1. Install Nginx

Update your package list and install Nginx:

```bash
sudo apt update && sudo apt install nginx -y
```

---

## 2. Install Node.js with nvm

Using **nvm** allows you to easily manage multiple versions of Node.js.

1. **Install nvm:**

   ```sh
   curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
   ```

2. **Reload your shell environment:**

   ```sh
   source ~/.bashrc
   ```

3. **Install the latest Node.js version:**

   ```sh
   nvm install node
   ```

4. **Use the latest Node.js version:**

   ```sh
   nvm use node
   ```

5. **Verify the installation:**

   ```bash
   node -v
   ```

---

## 3. Prepare Your Application

### a. Create Application Directory

Create a directory for your website:

```bash
sudo mkdir -p /var/www/web/
```

### b. Deploy Application Files

- **If uploading locally:** Use SFTP or scp:

  ```bash
  scp web.zip root@<YOUR_VPS_IP>:/var/www/web
  ```

- **If cloning from a repository:**

  ```bash
  git clone <YOUR_REPO_URL> /var/www/web
  ```

---

## 4. Install Application Dependencies

Navigate to your application directory and install dependencies:

```bash
cd /var/www/web
npm install
```

If your application requires environment variables, create a `.env` file:

```bash
nano .env
```

Then add your variables (example):

```
PORT=3000
MONGO_URI=mongodb://localhost:27017/mydb
```

---

## 5. Start the Node.js Application with PM2

### a. Install PM2 Globally

```bash
sudo npm install -g pm2
```

### b. (If Applicable) Build Your Application

For frameworks like React, Vue.js, Angular, Svelte, Next.js, or if using TypeScript:

```bash
npm run build
```

### c. Start Your Application

- **For npm-based start scripts:**

  ```bash
  pm2 start npm --name "myapp" -- start
  ```

- **For a specific entry file (e.g., `src/index.js`):**

  ```bash
  pm2 start src/index.js --name "web"
  ```

### d. Enable PM2 to Auto-Start on Reboot

```bash
pm2 startup
pm2 save
```

### e. Monitor PM2 Logs

Check the logs to verify your application is running correctly:

```bash
pm2 logs
```

For more detailed logs (last 1000 lines):

```bash
pm2 logs --lines 1000
```

---

## 6. Configure Nginx Server Blocks

### a. Create a Server Block for Your Domain

Create a configuration file for your domain (e.g., `web1.com`):

```bash
sudo nano /etc/nginx/sites-available/web1.com
```

### b. Configuration Examples

#### For Applications Using WebSockets

```nginx
server {
    listen 80;
    server_name web1.com;

    location /.well-known/acme-challenge/ {
        root /var/www/html;
        allow all;
    }

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api/ws {
        proxy_pass http://127.0.0.1:3000/api/ws;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
    }
}
```

#### For Standard HTTP Reverse Proxy

```nginx
server {
    listen 80;
    server_name web1.com www.web1.com;

    location /.well-known/acme-challenge/ {
        root /var/www/html;
        allow all;
    }

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### c. Enable the Server Block

Create a symbolic link to enable your site:

```bash
sudo ln -s /etc/nginx/sites-available/web1.com /etc/nginx/sites-enabled/
```

---

## 7. Test and Restart Nginx

1. **Test the configuration for syntax errors:**

   ```bash
   sudo nginx -t
   ```

2. **Restart Nginx:**

   ```bash
   sudo systemctl restart nginx
   ```

---

## 8. Secure Your Site with SSL (Let’s Encrypt)

### a. Install Certbot and the Nginx Plugin

```bash
sudo apt install certbot python3-certbot-nginx -y
```

### b. Obtain and Install an SSL Certificate

Run the Certbot command to automatically configure SSL for your domain:

```bash
sudo certbot --nginx
```

### c. Set Up Automatic Renewal

Test the renewal process with:

```bash
sudo certbot renew --dry-run
```

---

## 9. Configure the Firewall with UFW

Ensure only essential services (SSH, HTTP, HTTPS) are allowed:

```bash
sudo apt install ufw -y
sudo ufw allow OpenSSH
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

---

Your Node.js application should now be running securely behind Nginx on your specified domain. Should you encounter any issues, verify the logs for both PM2 and Nginx and adjust configurations as necessary.

Happy deploying!
