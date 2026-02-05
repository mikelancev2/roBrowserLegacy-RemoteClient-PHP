FROM nginx:alpine

RUN <<'EOF' cat >> /etc/nginx/conf.d/ro-mime-types.conf
types {
    application/ro-action   act;
    application/ro-sprite   spr;
    application/ro-palette  pal;
    application/ro-ground   gnd;
    application/ro-altitude gat;
    application/ro-world    rsw;
    application/ro-model    rsm;
}
EOF

RUN rm -f /etc/nginx/conf.d/default.conf

RUN <<'EOF' cat >> /etc/nginx/conf.d/default.conf
map $http_origin $cors_origin {
    default "";
    "~^https?://localhost" $http_origin;
    "~^https?://127\.0\.0\.1" $http_origin;
}

server {
    listen 80;
    server_name _;

    root /var/www/html;
    index index.php index.html index.htm;

    gzip on;
    gzip_comp_level 5;
    gzip_min_length 256;
    gzip_vary on;
    gzip_proxied any;
    gzip_types
        application/ro-action
        application/ro-sprite
        application/ro-palette
        application/ro-ground
        application/ro-altitude
        application/ro-world
        application/ro-model
        text/plain
        application/xml;

    location / {
        add_header 'Access-Control-Allow-Origin' $cors_origin always;
        add_header 'Access-Control-Allow-Headers' 'X-Application' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;

        if ($request_method = 'OPTIONS') {
            add_header 'Access-Control-Allow-Origin' $cors_origin;
            add_header 'Access-Control-Allow-Headers' 'X-Application';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Max-Age' 86400;
            add_header 'Content-Length' 0;
            return 204;
        }

        try_files $uri $uri/ @php_fallback;
    }

    location @php_fallback {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root/index.php;
        fastcgi_param REDIRECT_STATUS 404;
        fastcgi_pass php:9000;

        fastcgi_hide_header 'Access-Control-Allow-Origin';
        add_header 'Access-Control-Allow-Origin' $cors_origin always;
        add_header 'Access-Control-Allow-Headers' 'X-Application' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        fastcgi_pass php:9000;
        fastcgi_index index.php;

        fastcgi_hide_header 'Access-Control-Allow-Origin';
        add_header 'Access-Control-Allow-Origin' $cors_origin always;
        add_header 'Access-Control-Allow-Headers' 'X-Application' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;
    }

    # block sensitive hidden files
    location ~ /\.(git|svn|hg|env|htaccess|htpasswd)$ {
        deny all;
    }
}
EOF