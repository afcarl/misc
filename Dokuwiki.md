# Dokuwiki installation on Arch Linux with Nginx

The guides on arch were extremely confusing. So at the end I condensed the steps and did this:

1. Install everything.

  ```
  pacman -S nginx-mainline php php-fpm dokuwiki
  ```


2. Add this block on `/etc/nginx/nginx.conf` to make nginx use the php processor.

  ```
  location ~ \.php$ {
      fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
      fastcgi_index  index.php;
      fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
      include        fastcgi.conf;
  }
  ```

3. Reboot services.

  ```
  systemctl restart php-fpm nginx
  ```

4. By default nginx uses `/usr/share/nginx/html` to store files. Add a file there to test php works.

  ```
  echo "<?php phpinfo() ?>" > /usr/share/nginx/html/fuuk.php
  ```

5. Go to that [page](http://localhost/fuuk.php) and if php works then add dokuwiki's and other relevant directories to `/etc/php/php.ini`.

  ```
  open_basedir = /srv/http/:/tmp/:/usr/share/pear/:/usr/share/webapps/:/etc/webapps/dokuwiki/:/var/lib/dokuwiki/
  ```

6. Also uncomment `extension=gd.so` in that same file as apparently dokuwiki uses it for resizing images.

7. Add blocks some blocks on `/etc/nginx/nginx.conf` to forbid access to configuration files.

  ```
  location ~ /(data|conf|bin|inc)/ {
              deny all;
  }
  location ~ /\.ht {
      deny  all;
  }
  ```

8. I'll use nginx only for dokuwiki so I set a global root folder pointing to `/usr/share/webapps/dokuwiki`. The final server block on `/etc/nginx/nginx.conf` looks like this:

  ```
  server {
          listen       80;
          server_name  localhost;
          root    /usr/share/webapps/dokuwiki;

          #charset koi8-r;

          #access_log  logs/host.access.log  main;

          location / {
              #root   /usr/share/nginx/html;
              index  index.html index.htm index.php doku.php;
          }

          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
              root   /usr/share/nginx/html;
          }

          location ~ /(data|conf|bin|inc)/ {
              deny all;
          }
          location ~ /\.ht {
              deny  all;
          }

          location ~ \.php$ {
              fastcgi_pass   unix:/var/run/php-fpm/php-fpm.sock;
              fastcgi_index  index.php;
              fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
              include        fastcgi.conf;
          }
  }
  ```

9. Restart services.

  ```
  systemctl restart php-fpm nginx
  ```

10. Register services so they automatically start.

  ```
  systemctl enable php-fpm nginx
  ```

11. Finally go to [http://localhost/install.php](http://localhost/install.php) to configure Dokuwiki
