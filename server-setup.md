# Setting up a tmpnb server

 * install git, docker (follow instructions at https://docs.docker.com/engine/installation/linux/)
 * git checkout this repo into `/opt/`
 * configure dns
 * set up certificates in `/ssl/`, named `tmpnb.ca` (the full chain), `tmpnb.crt` (the certificate), and `tmpnb.key` (the private key)
 * set up automatic updates: `dpkg-reconfigure --priority=low unattended-upgrades`
 * execute start script after docker starts using systemd
   * copy `tmpnb.service.example` to `/lib/systemd/system/tmpnb.service`
   * adjust paths to fit your installation, if necessary
   * `sudo systemctl daemon-reload`
   * `sudo systemctl start tmpnb.service`
   * `sudo systemctl enable tmpnb.service`


