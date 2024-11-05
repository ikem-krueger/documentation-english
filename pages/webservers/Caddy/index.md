# Caddy

<!-- position: 2 -->

Bludit supports [Caddy]([https://nginx.org/en/](https://caddyserver.com/)), and we actually recommend it as a web server.

Caddy has its own `router` which handles all requests and responses. The idea is to redirect all requests to the `index.php` file.

Considerations:

- The webserver is running PHP-FPM as CGI Process Manager.
- PHP-FPM is listening on Unix socket at `unix:/run/php/php-fpm.sock`.

## HTTP set up

In order to set up a new server block for Bludit, generate a new file with the configuration in `/etc/caddy/Caddyfile`.

For security reasons, don't forget to forbid access to PHP files inside the `/bl-kernel/` folder, as well as the `/bl-content/databases`, `/bl-content/pages`, and `/bl-content/workspaces` folders. Otherwise it's possible that users would have direct access to some files in these directories.

```
server {
    root * /var/www/bludit
    file_server
    php_fastcgi unix//run/php/php-fpm.sock

    @staticFiles {
        path *.jpg *.jpeg *.gif *.png *.css *.js *.ico *.svg *.eot *.ttf *.woff *.woff2 *.otf
    }
    handle @staticFiles {
        header Cache-Control "public, max-age=2592000"  # 30 days
    }

    handle / {
        try_files {path} {path}/ /index.php?{query}
    }

    @denyDirs {
        path /bl-content/databases/* /bl-content/workspaces/* /bl-content/pages/* /bl-kernel/*.php
    }
    handle @denyDirs {
        respond "Access Denied" 403
    }
}
```

## HTTPS set up

No extra setup needed.
