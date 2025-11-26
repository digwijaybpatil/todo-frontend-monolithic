# Readme for Todo App 📝

### Installation 🚀

1. **🚀 Frontend VM Setup**
   - Run the following commands inside the Frontend VM:
   
     ```bash

      # 1️⃣ Verify that Frontend VM can reach Backend VM using private IP
      curl -v http://10.10.2.4:8000/tasks

      # 2️⃣ Install Node.js 16.x and npm
      curl -s https://deb.nodesource.com/setup_16.x | sudo bash
      sudo apt install -y nodejs

      # Check versions
      node -v
      npm -v

      # 3️⃣ Install NGINX
      sudo apt update
      sudo apt install -y nginx

      # 4️⃣ Prepare directory for frontend build
      sudo rm -rf /var/www/html
      sudo mkdir -p /var/www/html

      # Replace 'adminuser' with your VM username
      sudo chown -R adminuser:adminuser /var/www/html

     ```



2. **⚙️ Configure Backend Private IP in React**
   - Open the `src/TodoApp.js` file.
   - Locate the variable storing the backend URL and update it with the appropriate value. (* See Below for PrivateIp Configuration)
   - Update the Backend URL by replacing the existing line with the following:

   ```javascript
   const API_BASE_URL = 'http://<BackendVM private IP>:8000'; # const API_BASE_URL = 'http://10.10.2.4:8000';
   ```

   - Replace `<backendVM private IP>` with the actual private IP address of your Backend VM.
   - This ensures your frontend talks directly to backend using private IP inside Azure VNet.


3. **🏗️ Install & Build the Frontend**
   - Run the following command to install project dependencies: Should be in the pipeline or agent/runner
     ```bash
     npm install
     ```
   - Execute the following command to build the project:
     ```bash
     npm run build
     ```


4. **🚀 NGINX Setup on Frontend VM**

   Open the NGINX configuration file:

   ```bash
   sudo nano /etc/nginx/sites-available/default
   ```


   Delete everything, then paste this:
   ```nginx
   server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;

    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ /index.html;
    }
   }

   ```

   ## Check the configuration and restart the service

   ```bash
   sudo nginx -t
   sudo systemctl restart nginx
   ```

## Important Note 📌

   Make sure to restart NGINX on the VM after making the changes:

   ```bash
   sudo service nginx restart
   ```

   These configurations enable communication between the Frontend and Backend using Private IP on the Backend VM. Ensure that the IPs and ports are correctly set to match your environment.

5. **🔧 CI/CD – GitHub Actions Pipeline (Frontend Deployment to Azure VM)**

   Create this file in your repo:
   ```bash
   .github/workflows/deploy-frontend.yml
   ```
   Paste the following ready-to-use pipeline:
   ```yml
   name: Deploy Frontend to Azure VM

   on:
      push:
         branches:
         - main

   jobs:
      deploy:
         runs-on: ubuntu-latest

         steps:
         - name: Checkout Code
           uses: actions/checkout@v3

         - name: Setup Node
           uses: actions/setup-node@v3
           with:
            node-version: "16"

         - name: Install Dependencies
           run: npm install

         - name: Build Frontend
           run: npm run build

         # Copy only build CONTENTS into /var/www/html
         - name: Copy Build Contents to VM
           uses: appleboy/scp-action@v0.1.7
           with:
            host: ${{ secrets.SSH_HOST }}       # Frontend VM Public IP
            username: ${{ secrets.SSH_USER }}   # e.g., adminuser
            key: ${{ secrets.SSH_KEY }}         # SSH private key (PEM)
            port: 22
            source: "build/*"
            target: "/var/www/html/"

         - name: Restart Nginx
        uses: appleboy/ssh-action@v0.1.6
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            sudo systemctl restart nginx

   ```

## Final Note- Final Verification 📌
   From Frontend VM:
   ```bash

   curl http://10.10.2.4:8000/tasks

   ```

   From browser:Visit:
   ```cpp

   http://<Frontend-VM-Public-IP>
   

   ```
