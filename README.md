# Letsencrypt for WordPress in a docker image
Modded official Wordpress docker image with ability to run letsencrypt

This repo is the source for the docker hub image https://hub.docker.com/r/vitr/letsencrypt-wordpress/

You may use this image for quick obtaining certificates from https://letsencrypt.org/ and use them somewhere else or incorporate these ideas into your production images. 
I haven't changed much, just added these two features:

* ability to restart apache, otherwise the container stops everytime letsencrypt tries to restart apache, note the `tail` command in `apache/docker-entrypoint.sh`
* preinstall letsencrypt client, note `# install letsencrypt` section in `apache/Dockerfile`

### how to build image
    git clone https://github.com/vitr/docker-letsencrypt-wordpress.git
    cd docker-letsencrypt-wordpress/apache
    docker build -t vitr/letsencrypt-wordpress .

or you can use my pre-built image from the docker hub https://hub.docker.com/r/vitr/letsencrypt-wordpress/

### how to run letsencrypt
copy `docker-compose.yml` from this repo into `wp` folder in your homedir (~) (the reason for this is that docker compose requires absolute paths for volumes, e.g. `~/wp/letsencrypt`). Then run the compose

    docker-compose up -d
    docker exec -ti wp_wordpress_1 bash
    
now run inside the wordpress container

    cd /opt/letsencrypt
    ./letsencrypt-auto --apache -d yourdomain.com -m youremail@domain.com --agree-tos --redirect
    
You may also try dry-run to test your system before issuing a real certificate (I messed up with containers a lot and hit their rate limit 5 certs in 7 days)

    ./letsencrypt-auto certonly --dry-run --apache -d yourdomain.com -m youremail@domain.com --agree-tos --redirect
    
the certificate files will be stotred in `~/wp/letsencrypt/live/yourdomain.com` and you can reinstall with
 
    ./letsencrypt-auto install --apache -d yourdomain.com -m youremail@domain.com --agree-tos --redirect \
    --cert-path /etc/letsencrypt/live/yourdomain.com/cert.pem \
    --key-path /etc/letsencrypt/live/yourdomain.com/privkey.pem \
    --fullchain-path /etc/letsencrypt/live/yourdomain.com/fullchain.pem 
    
    
    
@TODO: add auto renewal

more about auto renewal process here
https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-14-04
    
@TODO test with fpm
    
    
set correct locale

    apt-get install locales
    locale-gen "en_US.UTF-8"
    localedef -v -c -i en_US -f UTF-8 en_US.UTF-8
    export LC_ALL="en_US.UTF-8"
    export NCURSES_NO_UTF8_ACS=1
    

./letsencrypt-auto --help all

-n, --non-interactive, --noninteractive
                        Run without ever asking for user input. This may
                        require additional command line flags; the client will
                        try to explain which ones are required if it finds one
                        missing (default: False)
    
