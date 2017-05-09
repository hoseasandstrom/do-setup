# NGINX Set up on Digital Ocean

# NGINX Set up on Digital Ocean

# NGINX SET UP
# following : https:#www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-12-04-lts-precise-pangolin

#ssh into droplet
ssh root@174.138.80.13

#installed nginx
sudo apt-get install nginx

#started nginx
sudo service nginx start

#confirm nginx has started
ifconfig eth0 | grep inet | awk '{ print $2 }'

#ensure nginx will be up after reboot
update-rc.d nginx defaults

#  How To Host Multiple Node.js Applications On a Single VPS with nginx, forever, and crontab
#  https:#www.digitalocean.com/community/tutorials/how-to-host-multiple-node-js-applications-on-a-single-vps-with-nginx-forever-and-crontab
#  setting up nginx and node were prereqs to this

# Reason for a reverse proxy:
  Reverse proxies can operate wherever multiple web-servers must be accessible via a single public IP address. The web servers listen on different ports in the same machine, with the same local IP address or, possibly, on different machines and different local IP addresses altogether. The reverse proxy analyzes each incoming request and delivers it to the right server within the local area network.

# create a file for your desired domain config
  nano /etc/nginx/conf.d/sweetgrassconstruction.com.conf

# copy the following into the conf file:
server {
  listen 80;

  server_name sweetgrassconstruction.com;

  location / {
      proxy_pass http:#localhost:3000;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;
  }
}

#  to be able to reference multiple domains for one Node.js app (like www.example.com and example.com)
  nano /etc/nginx/nginx.conf

# add to http block:
  server_names_hash_bucket_size 64;

# OPEN A NEW TERMINAL WINDOW FOR THE FOLLOWING :

# create cronjob to restart application if VPS gets rebooted
# navigate to root folder of sweetgrass
  touch starter.sh
# add to starter.sh, pointing to location of our server script:
  #!/bin/sh

  if [ $(ps -e -o uid,cmd | grep $UID | grep node | grep -v grep | wc -l | tr -s "\n") -eq 0 ]
  then
          export PATH=/usr/local/bin:$PATH
          forever start --sourceDir /opt/live/sweetgrass server.js >> /path/to/log.txt 2>&1
  fi
# add, commit and push to master branch of github

# push master to DO
  git push live master

# BACK TO SSH TERMINAL WINDOW :

# To start cron script at each reboot you need to edit the crontab with this command:
  crontab -e
# select nano editor
# append the following code :
  @reboot /opt/live/sweetgrass/starter.sh
# restart the nginx webserver
  service nginx restart
# check to see if it restarted
  service nginx status
