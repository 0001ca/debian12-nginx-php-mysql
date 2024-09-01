# Installation Script for Nginx, PHP, and MySQL on Debian 12

This guide provides a script that automates the installation of Nginx, PHP, and MySQL on a Debian 12 system. The script will update the system, install the required packages, and configure basic setups for each service.

## Bash Script

```bash
#!/bin/bash

# Update the package list and upgrade the system
echo "Updating the system..."
sudo apt update && sudo apt upgrade -y

# Install Nginx
echo "Installing Nginx..."
sudo apt install -y nginx

# Start and enable Nginx
echo "Starting and enabling Nginx..."
sudo systemctl start nginx
sudo systemctl enable nginx

# Install MySQL Server
echo "Installing MySQL Server..."
sudo apt install -y mysql-server

# Secure MySQL installation (automated)
echo "Securing MySQL installation..."
sudo mysql_secure_installation <<EOF

y
0
y
y
y
y
EOF

# Start and enable MySQL
echo "Starting and enabling MySQL..."
sudo systemctl start mysql
sudo systemctl enable mysql

# Install PHP and necessary extensions
echo "Installing PHP and extensions..."
sudo apt install -y php-fpm php-mysql php-cli php-curl php-gd php-mbstring php-xml php-zip

# Configure PHP with Nginx
echo "Configuring PHP with Nginx..."
sudo tee /etc/nginx/sites-available/default > /dev/null <<EOL
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /var/www/html;
    index index.php index.html index.htm;

    server_name _;

    location / {
        try_files \$uri \$uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
EOL

# Restart Nginx and PHP services
echo "Restarting Nginx and PHP services..."
sudo systemctl restart nginx
sudo systemctl restart php*-fpm

echo "Installation and setup of Nginx, PHP, and MySQL are complete!"
