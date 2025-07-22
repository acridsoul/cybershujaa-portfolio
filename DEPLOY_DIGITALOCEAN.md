# Deploying Your Astro Portfolio to DigitalOcean (Ubuntu) with Nginx and HTTPS

This guide will walk you through deploying your Astro static site (portfolio) to a DigitalOcean droplet running Ubuntu, using Nginx as a web server and Certbot for free HTTPS certificates.

---

## 1. Prerequisites
- DigitalOcean account
- Domain name: **portfolio.githinji.dev**
- SSH key (recommended for secure login)
- Basic familiarity with the terminal

---

## 2. Create and Prepare Your Droplet
1. **Create a Droplet**
   - Go to DigitalOcean dashboard → Create → Droplet
   - Choose Ubuntu (latest LTS recommended)
   - Select a plan (the $4/month plan is enough for static sites)
   - Add your SSH key (recommended)
   - Choose a datacenter region close to your audience
   - Create the droplet and note its public IP address

2. **Connect via SSH**
   ```sh
   ssh root@your_droplet_ip
   ```

3. **Update the System**
   ```sh
   sudo apt update && sudo apt upgrade -y
   ```

---

## 3. Install Node.js, npm, and pnpm
1. **Install Node.js (LTS)**
   ```sh
   curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
   sudo apt install -y nodejs
   ```
2. **Install pnpm**
   ```sh
   npm install -g pnpm
   ```
3. **Check versions**
   ```sh
   node -v
   npm -v
   pnpm -v
   ```

---

## 4. Upload or Clone Your Project
1. **Clone from GitHub (recommended):**
   ```sh
   git clone https://github.com/yourusername/your-astro-portfolio.git
   cd your-astro-portfolio
   ```
   *(Or use SFTP/rsync to upload your code)*

2. **Install dependencies and build:**
   ```sh
   pnpm install
   pnpm build
   ```
   The static site will be generated in the `/root/cybershujaa-portfolio/dist` folder.

---

## 5. Install and Configure Nginx
1. **Install Nginx:**
   ```sh
   sudo apt install nginx -y
   ```
2. **Configure Nginx:**
   - Remove the default config:
     ```sh
     sudo rm /etc/nginx/sites-enabled/default
     ```
   - Create a new config file:
     ```sh
     sudo nano /etc/nginx/sites-available/astro-portfolio
     ```
   - Paste the following (replace the root path if your project is elsewhere):
     ```nginx
     server {
         listen 80;
         server_name portfolio.githinji.dev www.portfolio.githinji.dev;
         root /root/cybershujaa-portfolio/dist;
         index index.html;

         location / {
             try_files $uri $uri/ =404;
         }
     }
     ```
   - Enable the config:
     ```sh
     sudo ln -s /etc/nginx/sites-available/astro-portfolio /etc/nginx/sites-enabled/
     sudo nginx -t
     sudo systemctl reload nginx
     ```

---

## 6. Point Your Domain to the Droplet
- In your domain registrar's DNS settings, set an A record for **portfolio.githinji.dev** pointing to your droplet's public IP.
- Wait for DNS propagation (can take a few minutes to a few hours).

---

## 7. Secure with HTTPS (Let's Encrypt & Certbot)
1. **Install Certbot:**
   ```sh
   sudo apt install certbot python3-certbot-nginx -y
   ```
2. **Obtain and install certificate:**
   ```sh
   sudo certbot --nginx -d portfolio.githinji.dev -d www.portfolio.githinji.dev
   ```
   - Follow prompts to allow Certbot to configure HTTPS for you.
   - Test auto-renewal:
     ```sh
     sudo certbot renew --dry-run
     ```

---

## 8. (Optional) Automate Deployment
- Use GitHub Actions or rsync/SFTP to automate pushing new builds to your server.
- For manual updates:
  ```sh
  cd /root/cybershujaa-portfolio
  git pull
  pnpm install
  pnpm build
  sudo systemctl reload nginx
  ```

---

## 9. Best Practices
- **Firewall:**
  ```sh
  sudo ufw allow 'Nginx Full'
  sudo ufw enable
  ```
- **Keep system and dependencies updated**
- **Back up your droplet and site regularly**

---

## 10. Troubleshooting
- Check Nginx status: `sudo systemctl status nginx`
- View logs: `sudo journalctl -u nginx` or `sudo tail -f /var/log/nginx/error.log`
- Check Certbot logs: `/var/log/letsencrypt/`

---

## 11. References
- [Astro Docs: Deployment](https://docs.astro.build/en/guides/deploy/)
- [Nginx Docs](https://nginx.org/en/docs/)
- [Certbot Docs](https://certbot.eff.org/)
- [DigitalOcean Docs](https://www.digitalocean.com/docs/)

---

**You’re done! Your Astro portfolio is now live and secure on your DigitalOcean droplet at https://portfolio.githinji.dev.** 