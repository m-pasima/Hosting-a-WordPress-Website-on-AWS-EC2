# Hosting a WordPress Website on AWS EC2

This guide provides detailed instructions for hosting a WordPress website on an AWS EC2 instance using a traditional method. We'll set up an Apache web server, a MySQL database, and WordPress. We will also configure a domain name and SSL certificate.

## Prerequisites

- AWS account
- SSH client (e.g., Mobaxterm)

## Steps

### 1. Launch an EC2 Instance

#### Create an EC2 Instance
1. **Sign in to AWS Management Console.**
2. **Navigate to the EC2 Dashboard** and click on "Launch Instance."
   - Pic below:
   ![EC2 Dashboard](https://github.com/m-pasima/Hosting-a-WordPress-Website-on-AWS-EC2/assets/170426323/1f60dbc9-a1cf-4d31-9bc3-eb5207efca1c)

3. **Choose an Amazon Machine Image (AMI).** Select an Ubuntu Server AMI.
4. **Select an instance type** (e.g., t2.micro for free tier).
5. **Configure instance details, storage, and tags** (use default settings).
6. **Configure security group** to allow HTTP (port 80) and HTTPS (port 443) traffic.
   - Pic below:
   ![Security Group Settings](https://github.com/m-pasima/Hosting-a-WordPress-Website-on-AWS-EC2/assets/170426323/40896180-af68-4567-a56e-81cef4892298)

7. **Review and launch the instance.**
8. **Create and download a new key pair** (.pem file). Save this key pair securely as it will be used to connect to your instance.
   - Pic below:
   ![Key Pair](https://github.com/m-pasima/Hosting-a-WordPress-Website-on-AWS-EC2/assets/170426323/6e785084-d0e0-4406-a380-eb97116ad3b1)

### 2. Allocate an Elastic IP
Allocating an Elastic IP ensures your server retains the same IP address even if it's restarted.

1. **Navigate to the "Elastic IPs"** section in the EC2 Dashboard.
2. **Allocate a new Elastic IP** and associate it with your EC2 instance.
   - Pic below:
   ![Elastic IP Allocation](https://github.com/m-pasima/Hosting-a-WordPress-Website-on-AWS-EC2/assets/170426323/3be981ab-7fbc-41d8-b6fa-98cb12be6189)

### 3. Connect to Your EC2 Instance via SSH

1. **Open Mobaxterm.**
2. **Go to "Session"** and choose "SSH."
3. **Enter your Elastic IP** in the "Remote host" section.
4. **Set the username** to `ubuntu` (default for Ubuntu servers).
5. **Under "Advanced SSH settings," upload your .pem key.**
6. **Click "OK"** to connect.
   - Pic below:
   ![Mobaxterm SSH Settings](https://github.com/m-pasima/Hosting-a-WordPress-Website-on-AWS-EC2/assets/170426323/b60ab635-6d64-4b45-a896-fe5997d8472f)

### 4. Install Apache Web Server

1. **Update package lists:**
   ```sh
   sudo apt update
   ```
2. **Install Apache:**
   ```sh
   sudo apt install apache2 -y
   ```
3. **Verify the installation** by entering your Elastic IP in a browser. You should see the Apache2 default page.
   - Pic below:
   ![Apache Default Page](https://github.com/m-pasima/Hosting-a-WordPress-Website-on-AWS-EC2/assets/170426323/1fdc5c31-6666-4cb1-bc1e-0056eade4403)

### 5. Install PHP and MySQL

1. **Install PHP and MySQL connector:**
   ```sh
   sudo apt install php libapache2-mod-php php-mysql -y
   ```
2. **Install MySQL server:**
   ```sh
   sudo apt install mysql-server -y
   ```

### 6. Configure MySQL

1. **Log in to MySQL as root:**
   ```sh
   sudo mysql -u root
   ```
2. **Change the authentication plugin and set a strong password:**
   ```sql
   ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourStrongPassword';
   ```
3. **Create a new MySQL user for WordPress:**
   ```sql
   CREATE USER 'wp_user'@'localhost' IDENTIFIED BY 'YourStrongPassword';
   ```
4. **Create a new database for WordPress:**
   ```sql
   CREATE DATABASE wp;
   ```
5. **Grant all privileges on the `wp` database to the new user:**
   ```sql
   GRANT ALL PRIVILEGES ON wp.* TO 'wp_user'@'localhost';
   ```
6. **Exit MySQL:**
   ```sh
   exit
   ```

### 7. Install WordPress

1. **Download WordPress:**
   ```sh
   cd /tmp
   wget https://wordpress.org/latest.tar.gz
   ```
2. **Extract the downloaded file:**
   ```sh
   tar -xvf latest.tar.gz
   ```
3. **Move the WordPress files to the Apache document root:**
   ```sh
   sudo mv wordpress/ /var/www/html
   ```

### 8. Configure Apache for WordPress

1. **Edit the Apache configuration file:**
   ```sh
   sudo vi /etc/apache2/sites-available/000-default.conf
   ```
2. **Change the `DocumentRoot` to point to the WordPress directory:**
   ```sh
   DocumentRoot /var/www/html/wordpress
   ```
3. **Save and exit the editor.**

### 9. Restart Apache

1. **Restart Apache to apply the changes:**
   ```sh
   sudo systemctl restart apache2
   ```

### 10. Complete WordPress Setup

1. **Open your browser** and navigate to your Elastic IP. You should see the WordPress installation page.
   - Pic below:
   ![WordPress Installation](https://github.com/m-pasima/Hosting-a-WordPress-Website-on-AWS-EC2/assets/170426323/cc191581-030f-447c-ab53-751e18e24e4e)

2. **Follow the on-screen instructions** to configure WordPress with the database details (`wp_user`, `wp`, `YourStrongPassword`).
   - Pic below:
   ![WordPress Configuration](https://github.com/m-pasima/Hosting-a-WordPress-Website-on-AWS-EC2/assets/170426323/3f468c2b-e445-492a-a251-1e1174d036bf)

3. **Once successful, use the username and password** to log in to the admin page and proceed with website configurations.
   - Pic below:
   ![WordPress Admin](https://github.com/m-pasima/Hosting-a-WordPress-Website-on-AWS-EC2/assets/170426323/3603e77c-b210-466a-9c76-8bcd39d980e2)

### 11. Configure Domain Name

1. **Go to Route 53** in the AWS Management Console.
2. **Create a hosted zone** for your domain.
3. **Add an A record** with your Elastic IP.
   - Pic below:
   ![Route 53 A Record](https://github.com/m-pasima/Hosting-a-WordPress-Website-on-AWS-EC2/assets/170426323/b2575baa-15a5-4522-be49-3deafea840ce)

4. **Back on the terminal**, edit `/etc/apache2/sites-available/000-default.conf` and modify the file with:
   ```apache
   ServerName demo.example.com
   ServerAlias www.example.com
   ```

### 12. Enable SSL with Certbot

1. **Install Certbot:**
   ```sh
   sudo apt-get update
   sudo apt install certbot python3-certbot-apache -y
   ```
2. **Request and install an SSL certificate:**
   ```sh
   sudo certbot --apache
   ```
3. **Follow the Certbot instructions** to configure SSL for your domain.

## Conclusion

You have successfully hosted a WordPress website on an AWS EC2 instance with Apache, MySQL, and PHP. Additionally, you configured a domain name and secured your site with SSL.
