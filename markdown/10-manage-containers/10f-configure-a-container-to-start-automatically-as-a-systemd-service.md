# 10.f Configure a container to start automatically as a systemd service

+ [RHEL > 8 > Building, running, and managing containers > Chapter 8. Porting containers to systemd using Podman](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/porting-containers-to-systemd-using-podman_building-running-and-managing-containers)
+ [Running containers with Podman and shareable systemd services](https://www.redhat.com/sysadmin/podman-shareable-systemd-services)
+ [How to run systemd in a container](https://developers.redhat.com/blog/2019/04/24/how-to-run-systemd-in-a-container/)

**Commands:**
- podman-generate-systemd (1) - Generate systemd unit file(s) for a container or pod
- loginctl (1)         - Control the systemd login manager
- systemctl (1)        - Control the systemd system and service manager

## Creating and Enabling a System Service

We can use `podman generate systemd` to create a systemd unit file.  

Change into the Systemd unit files folder

    # cd /etc/systemd/system

**📝 NOTE:** _The systemd unit files folder can be `/etc/systemd/system` or `/usr/lib/systemd/system`._

Create the systemd unit file

    # podman generate systemd -f -n -t 2 naughty_albattani  
    /etc/systemd/system/container-naughty_albattani.service

The new systemd unit file looks like this

    # container-naughty_albattani.service
    # autogenerated by Podman 2.0.5
    # Fri Dec  4 12:04:22 EST 2020

    [Unit]
    Description=Podman container-naughty_albattani.service
    Documentation=man:podman-generate-systemd(1)
    Wants=network.target
    After=network-online.target

    [Service]
    Environment=PODMAN_SYSTEMD_UNIT=%n
    Restart=on-failure
    ExecStart=/usr/bin/podman start naughty_albattani
    ExecStop=/usr/bin/podman stop -t 2 naughty_albattani
    ExecStopPost=/usr/bin/podman stop -t 2 naughty_albattani
    PIDFile=/var/run/containers/storage/overlay-containers/435cb8153beaae5d92668bd83965d9169f8718fe0849bf398661f340d998e5cc/userdata/conmon.pid
    KillMode=none
    Type=forking

    [Install]
    WantedBy=multi-user.target default.target

Start and enable the service

    # systemctl enable –-now container-naughty_albattani.service

Make sure that it's running

    # systemctl status container-naughty_albattani.service
    ● container-naughty_albattani.service - Podman container-naughty_albattani.service
       Loaded: loaded (/etc/systemd/system/container-httpd_systemd.service; enabled; vendor preset: disabled)
       Active: active (running) since Sat 2020-12-05 18:12:39 EST; 34min ago
         Docs: man:podman-generate-systemd(1)
      Process: 55913 ExecStart=/usr/bin/podman start naughty_albattani (code=exited, status=0/SUCCESS)
    Main PID: 56006 (conmon)
        Tasks: 2 (limit: 12285)
       Memory: 2.0M
       CGroup: /system.slice/container-naughty_albattani.service
               └─56006 /usr/bin/conmon --api-version 1 -c 443cbcf1ec10662140c904417ea36520418f6f8c02817c0f05746b07a3dea84b -u 443cbcf1ec10662140c904417ea36520418f6f8c02817c0f05746b07a3dea84b -r /usr/bin/runc -b /va>
    Dec 05 18:12:38 rhel8-lab systemd[1]: Starting Podman container-naughty_albattani.service...
    Dec 05 18:12:39 rhel8-lab podman[55913]: naughty_albattani
    Dec 05 18:12:39 rhel8-lab systemd[1]: Started Podman container-naughty_albattani.service.

And double check with 'podman ps'

    # podman ps -a
    CONTAINER ID  IMAGE                                 COMMAND               CREATED       STATUS                  PORTS                 NAMES
    435cb8153bea  docker.io/library/httpd:latest        httpd-foreground      46 hours ago  Up 24 hours ago         0.0.0.0:8080->80/tcp  naughty_albattani

## Creating and Enabling a User Service

It's always a good approach to run rootless containers. This will provide another layer of security by downgrading the possible access that the container could have.  

**📝 IMPORTANT NOTES:**
- Users run the Systemd commands with `systemctl –user`
- User unit files are stored at `~/.config/systemd/user/`
- User Systemd services by default only run when a user logs in (they do not start with the server)
- When creating a new account, make sure to create a non-system account, otherwise you will not be able to start the services
- You cannot run `systemctl` as user after changing into the user with `sudo` or `su`. You will need to `ssh` or fully login as the user

### Enabling User Service to Start with Server

First let's enable the Systemd service to start with the server

    # loginctl enable-linger [user]

### Creating the Service

Login as the user

    # ssh [user]@localhost

Now let's create the folder

    $ mkdir -p ~/.config/systemd/user  

    $ cd !$

Create the unit file

    $ podman generate systemd –f –n [container]

Enable the service

    $ systemctl –-user enable –now container-[container].service

Check that the service is up

    $ systemctl –-user status container-[container].service

Make sure that the container it's running

    $ podman ps

**⚠️ WARNING:** _After creating the service file, you should not use `podman` to control the container._

---
[⬅️ Back](10-manage-containers.md)