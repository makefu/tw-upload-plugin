# Tiddly Wiki Upload Plugin

This is a patched version of the tiddly wiki upload plugin
(http://tiddlywiki.tiddlyspace.com/UploadPlugin).

The original version has not been maintained for a long time and the original
version contained multiple directory-traversal vulnerabilities.

In addition the store.php file is configured via ini file, you do not have to
touch the code anymore.

# Configuration

ENV['twconf'] contains a path to an ini file which looks like this:

    [users]
    user1 = pass1
    user2 = pass2
    userN = passN

    [directories]
    storedir = /path/to/wiki/store
    backupdir = /path/to/wiki/backup


# PHP-FPM

An FPM Pool may look like this:

    [wiki]
    user =  www-data
    group =  www-data
    listen = /var/run/php5-fpm.sock
    listen.owner = www-data
    listen.group = www-data
    env[twconf] = /var/www/wiki/twconf.ini;
    pm = dynamic
    pm.max_children = 5
    pm.start_servers = 2
    pm.min_spare_servers = 1
    pm.max_spare_servers = 3
    chdir = /
    # errors to journal
    php_admin_value[error_log] = 'stderr'
    php_admin_flag[log_errors] = on
    catch_workers_output = yes

# nginx config

    server {
        listen 443 ssl;
        server_name mywiki.localhost;
        location / {
            root /var/www/wiki/store/;
        }
        location /store.php {
            root /path/to/tw-upload-plugin/;
            client_max_body_size 200M;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php5-fpm.sock;
            include fastcgi_params;
            include fastcgi.conf;
        }
    }

