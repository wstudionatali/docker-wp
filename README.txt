# Docker Compose and WordPress (roots/bedrocket) 
  (php composer + nginx + mariabd + phpmyadmin + mailhog)
If you do not use ssl sertificate, then correct your /nginx/default.conf.conf file
---------------------------------
server {
    listen 80;

    root /var/www/html/web;
    index index.php;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    client_max_body_size 100M;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass wordpress:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
----------------------------------------
SSl sertificate for Windows.
----------------------------------------
1. Set up open-ssl for windows and input settings See steps here https://thesecmaster.com/procedure-to-install-openssl-on-the-windows-platform/
2. Open windows Shell for Administrator and Install Chocolatey for Individual Use
  -- Change executive policy from Restricted to Bypass
   At first check it:   Get-ExecutionPolicy
   Then change: Set-ExecutionPolicy Bypass -Scope Process
  -- Install Chocolatey 
   Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))   
3. Run in Shell
   choco install mkcert
4. Go to progect folder and make dir /certs
    mkdir certs 
5. Go to directory and create sertificates:
   cd certs	&& mkcert -install mydomain    - where  mydomain - domain name for which certificates was released
Check - in folder /certs must appear 2 files: mydomain-key.pem and mydomain.pem , where  mydomain - domain name for which certificates was released
  //// For Mac OC use file cli/create-cert.sh /////
------------------------------------------- 

6. run container for composer image and  command for creating project in the current dir(html), use dot for creating project in current dir. If you use name of dir, then the dir will be created in current dir.
   docker-compose run composer create-project roots/bedrock .
  
7. Edit file .env in folder src. Use values DB_NAME and DB_PASSWORD from file .env of root project folder
----progect_folder/.env--------------------
DB_NAME=myapp
DB_ROOT_PASSWORD=password
DOMAIN=myapp.local

---progect_folder/src/.env-----------------
DB_NAME='myapp'
DB_USER='root'
DB_PASSWORD='password'  
DB_HOST='mysql'  /* name of datebase service from docker-composer.yml*/
WP_ENV='development'
WP_HOME='https://myapp.local'  /* https if you activated ssl sert (https:/DOMAIN)  */
WP_SITEURL="${WP_HOME}/wp"
WP_DEBUG_LOG=/var/www/html/debug.log

https://roots.io/salts.html geterate key values here
-----------------------------------------------

8. Check file hosts  C:\Windows\System32\drivers\etc It must have a record:
   127.0.0.1 myapp.local
   
9. Launch image building and containers up 
  docker-compose up -d 
  
11. Now pun the link https://myapp.local/ in you browser and pass wp installation 

12.Installation additional wp plugins and themes. 
  You have to add lines in composer.json in the section "require" with plugins and themes refferenses from  WPackagist.com
  For example:
"require": { 
    ...
    "wpackagist-plugin/advanced-custom-fields":"6.1.7",
    "wpackagist-plugin/woocommerce": "^7.9"
}
After updating composer.json launch the cmd in the shell in project_folder
  docker-compose run composer update
  
13. Use WP-CLI
Login to the container.   Use exit for exiting 
   docker exec -it myapp-php bash

Run a wp-cli command  (for example)
    wp search-replace https://olddomain.com https://newdomain.com --allow-root

