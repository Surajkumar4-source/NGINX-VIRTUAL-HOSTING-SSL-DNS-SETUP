

# Implementing Nginx with Virtual Hosting and DNS Configuration


<br>


## Introduction

### *In this Implementation, we will set up a web server using Nginx and configure virtual hosting and DNS (Domain Name System) for a custom domain, suraj.com. This includes:*

  - Installing and configuring the Nginx server.
  - Setting up DNS records to resolve the domain suraj.com and its subdomains (e.g., www.suraj.com).
  - Implementing virtual hosting to serve different content for multiple domains.
  - Adding SSL (HTTPS) for secure communication.



## Workflow

1. Install and Configure Nginx: Install the Nginx web server and set up the default web page.
2. Configure DNS: Create DNS zone files to map domain names to IP addresses.
3. Set Up Virtual Hosting: Configure Nginx to serve different content based on domain names.
4. Enable SSL: Generate and configure SSL certificates for secure communication.
5. Testing: Verify that the configurations work by testing DNS resolution, HTTP, and HTTPS.


<br>
<br>
<br>

# Step-by-Step Implementation

### 1. Install Nginx
```yml
# yum install nginx -y   # For RedHat/CentOS
# apt update && apt install nginx bind9 -y  # For Debian/Ubuntu
```
  Explanation:

  - yum install nginx: Installs the Nginx package on RedHat-based systems.
  - apt update: Updates the package index for Debian-based systems.
  - apt install nginx: Installs Nginx on Debian-based systems.
  - named: The daemon for the BIND DNS server that resolves domain names to IP addresses.
  - -y: Automatically confirms the installation prompts.

### 2. Start and Enable Nginx
```yml
# systemctl start nginx
# systemctl enable nginx
```
  Explanation:

  - systemctl start nginx: Starts the Nginx service.
  - systemctl enable nginx: Ensures Nginx starts automatically on system boot.


### 3. Set Up Default Web Page
```yml
# cd /var/www/html   # For Debian/Ubuntu
# cd /usr/share/nginx/html  # For RedHat/CentOS
# echo "<h1>Welcome to Nginx Web Server</h1>" > index.html
```
  - Explanation:

  - cd: Changes the directory to the Nginx document root.
  - echo ... > index.html: Creates an HTML file with a simple "Welcome" message.


### 4. Configure DNS

### Step 1: Update Named Configuration

```yml
# vim /etc/named.conf
```
  - Explanation: Opens the named.conf file to define new DNS zones.

  - Add the following under the zones section:

```
zone "suraj.com" IN {
    type master;
    file "for.suraj.com";
};
```
  - Defines a DNS zone for suraj.com and specifies the file to store its records.

### Step 2: Create Zone File (Bind9)

```yml
# cd /var/named
# cp -av for.example.com for.suraj.com
# vim for.suraj.com
```
- Explanation:

  - cd /var/named: Navigates to the directory storing DNS zone files.
  - cp -av for.example.com for.suraj.com: Copies the template file for the new domain.
  - vim for.suraj.com: Opens the zone file for editing.

      ### Edit for.suraj.com as follows:

```yml
$TTL 1D
@       IN SOA master.suraj.com. admin.suraj.com. (
                0       ; serial
                1D      ; refresh
                1H      ; retry
                1W      ; expire
                3H )    ; minimum
        IN      NS      master.suraj.com.
suraj.com.      IN      A       192.168.1.100
www.suraj.com.  IN      A       192.168.1.100

```
  - Explanation:

  - suraj.com. IN A 192.168.1.100: Maps suraj.com to the server's IP address.
  - www.suraj.com. IN A 192.168.1.100: Maps www.suraj.com to the same IP.

  *Restart the DNS service:*

```yml
# systemctl restart named
```
  - Explanation: Applies the changes to the DNS configuration.

### 5. Virtual Hosting Configuration

Step 1: Create Web Directories

```yml
# mkdir -p /usr/share/nginx/html/suraj.com
```
  - Explanation: Creates a directory to hold the content for suraj.com.

  *Add HTML Content:*
```yml

# echo "<h1>Main Server: Welcome to Suraj.com</h1>" > /usr/share/nginx/html/index.html
# echo "<h1>Virtual Host: Welcome to Suraj.com</h1>" > /usr/share/nginx/html/suraj.com/index.html
```
  - Explanation: Creates a unique HTML file for the main server and virtual host.

Step 2: Configure Virtual Hosts

```yml
# vim /etc/nginx/sites-available/suraj-main.conf
# vim /etc/nginx/sites-available/suraj-vhost.conf
```
  - Explanation: Opens files for defining server configurations.

      *Add the following content to suraj-main.conf:*

```yml
server {
    listen 80 default_server;
    root /usr/share/nginx/html;
    index index.html;
    server_name suraj.com www.suraj.com;
    access_log /var/log/nginx/suraj-main_access.log;
    error_log /var/log/nginx/suraj-main_error.log;
}
```

*And for suraj-vhost.conf:*

```yml
server {
    listen 80;
    root /usr/share/nginx/html/suraj.com;
    index index.html;
    server_name vhost.suraj.com www.vhost.suraj.com;
    access_log /var/log/nginx/suraj-vhost_access.log;
    error_log /var/log/nginx/suraj-vhost_error.log;
}
```

### Step 3: Enable Configurations

```yml
# ln -s /etc/nginx/sites-available/suraj-main.conf /etc/nginx/sites-enabled/
# ln -s /etc/nginx/sites-available/suraj-vhost.conf /etc/nginx/sites-enabled/
# systemctl restart nginx
```
  - Explanation: Links the configuration files to the active directory and restarts Nginx.

### 6. Enable HTTPS/SSL Configuration

Step 1: Generate SSL Certificates

```yml
# mkdir /etc/nginx/certs
# cd /etc/nginx/certs
# openssl req -newkey rsa:2048 -nodes -keyout suraj.key -x509 -days 365 -out suraj.crt
```
  - Generates a private key (suraj.key) and a public certificate (suraj.crt) valid for 365 days.

  - Generates SSL certificates and keys for secure communication.






Step 2: Configure HTTPS

```yml
# vim /etc/nginx/sites-available/suraj-ssl.conf
```
  - Add the following:

```yml
server {
    listen 443 ssl;
    ssl_certificate /etc/nginx/certs/suraj.crt;
    ssl_certificate_key /etc/nginx/certs/suraj.key;

    root /usr/share/nginx/html/suraj.com;
    index index.html;
    server_name suraj.com www.suraj.com;

    access_log /var/log/nginx/suraj-ssl_access.log;
    error_log /var/log/nginx/suraj-ssl_error.log;
}
```

*Enable the configuration:*

```yml
# ln -s /etc/nginx/sites-available/suraj-ssl.conf /etc/nginx/sites-enabled/

# systemctl restart nginx

```




### 7. Testing the Setup
1. Test DNS:

```yml
# host suraj.com
# host www.suraj.com
```
  - Confirms that suraj.com resolves to the correct IP.

2. Test HTTP:

```yml
# curl http://suraj.com      OR  in the browser http://suraj.com
```
 ####  - Display: <h1>Main Server: Welcome to Suraj.com</h1>

3. Test HTTPS:

```yml
# curl -k https://suraj.com      OR  in the browser https://suraj.com
```
 ####  - Display: <h1>Virtual Host: Welcome to Suraj.com</h1>



*Note:  when we open the website in the browser , using SSL for the first time with a self-signed or untrusted certificate, browsers show a "Your connection is not private" warning. In the Advanced options, users can view details like certificate issues (e.g., self-signed or expired) and choose to proceed despite the risks.*

<br>
<br>

## Conclusion

*You have successfully set up Nginx with virtual hosting, DNS configuration, and SSL for secure communication. This setup supports multiple domains, including the main domain suraj.com and a virtual host vhost.suraj.com, with SSL configured for secure HTTPS access.*
