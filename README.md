# Docker-Wordpress

Before building and running
- Change the _gateway_ network in docker compose with the one where your Traefik is running on. Else created a new one bridged with the host to expose only required containers
- Update the file nginx/conf/nginx-wp.conf to replace the server name
- Update the .env file with wanted values, in particular the FQDN of your server if you use Traefik
- Optional : change the exposed port by Nginx in the docker-compose file (default 80 is mapped to 8080
- Optional : change the challenge used by Traefik (default is myhttpchallenge)