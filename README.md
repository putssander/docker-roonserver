# docker-roonserver
RoonServer downloading Roon on first run

This little project configures a docker image for running RoonServer.
It downloads RoonServer if not found on an external volume.

Example start:

    docker run -d \
      --net=host \
      -e TZ="Europe/Amsterdam" \
      -v roon-app:/app \
      -v roon-data:/data \
      -v roon-music:/music \
      -v roon-backups:/backup \
      steefdebruijn/docker-roonserver:latest
  
  * You should set `TZ` to your timezone.
  * You can change the volume mappings to local file system paths if you like.
  * You should set up your library root to `/music` and configure backups to `/backup` on first run.
  
Example restart always:

      docker run -d \
        --net=host \
        --name roonserver --restart always \
        -e TZ="Europe/Amsterdam" \
        -v roon-app:/app \
        -v roon-data:/data \
        -v roon-music:/music \
        -v roon-backups:/backup \
        steefdebruijn/docker-roonserver:latest

This how i configured my firewall to get it running:

      firewall-cmd --permanent --zone=trusted --add-interface=docker0
      firewall-cmd --permanent --zone=trusted --add-interface=eno1
      firewall-cmd --permanent --zone=trusted --add-port=9003/udp
      firewall-cmd --permanent --zone=trusted --add-port=3483/udp
      firewall-cmd --permanent --zone=trusted --add-port=3483/tcp
      firewall-cmd --permanent --zone=trusted --add-port=9000-9200/tcp    

Example `systemd` service:

    [Unit]
    Description=Roon
    After=docker.service
    Requires=docker.service
    
    [Service]
    TimeoutStartSec=0
    TimeoutStopSec=180
    ExecStartPre=-/usr/bin/docker kill %n
    ExecStartPre=-/usr/bin/docker rm -f %n
    ExecStartPre=/usr/bin/docker pull steefdebruijn/docker-roonserver
    ExecStart=/usr/bin/docker \
      run --name %n \
      --net=host \
      -e TZ="Europe/Amsterdam" \
      -v roon-app:/app \
      -v roon-data:/data \
      -v roon-music:/music \
      -v roon-backups:/backup \
      steefdebruijn/docker-roonserver
    ExecStop=/usr/bin/docker stop %n
    Restart=always
    RestartSec=10s
    
    [Install]
    WantedBy=multi-user.target


  Don't forget to backup the `roon-backups` *for real* (offsite preferably).

  Have fun!
  
  Steef

## Version history

  * 2019-03-18: Fix example start (thanx @heapxor); add `systemd` example.
  * 2019-01-23: updated base image to `debian-9.6`
  * 2017-08-08: created initial images based on discussion on roonlabs forum

