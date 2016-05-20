# Setting up a tmpnb server

 * install git, docker
 * git checkout this repo
 * set up certificates in `/ssl/`, see start script for file names to use
 * configure dns
 * set up automatic updates: dpkg-reconfigure --priority=low unattended-upgrades
 * execute start script after docker starts using systemd
   * copy `tmpnb.service.example` to /lib/systemd/system/tmpnb.service
   * sudo systemctl daemon-reload
   * sudo systemctl start tmpnb.service


