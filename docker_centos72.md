# install docker in centos 7.2


## issue: can't start the docker deamo
[zhangdingshui@dataplatform-gulfstream-offline03 ~]$ sudo systemctl status docker -l
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; disabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/docker.service.d
           └─cleanup_docker0_bridge.conf
   Active: failed (Result: exit-code) since Mon 2017-03-27 22:03:52 CST; 3 days ago
     Docs: http://docs.docker.com
 Main PID: 25748 (code=exited, status=1/FAILURE)

Mar 27 22:03:51 dataplatform-gulfstream-offline03.xg01 docker-current[25748]: time="2017-03-27T22:03:51.995563350+08:00" level=info msg="[graphdriver] using prior storage driver \"devicemapper\""
Mar 27 22:03:51 dataplatform-gulfstream-offline03.xg01 docker-current[25748]: time="2017-03-27T22:03:51.996062958+08:00" level=warning msg="Docker could not enable SELinux on the host system"
Mar 27 22:03:51 dataplatform-gulfstream-offline03.xg01 docker-current[25748]: time="2017-03-27T22:03:51.996475340+08:00" level=info msg="Graph migration to content-addressability took 0.00 seconds"
Mar 27 22:03:52 dataplatform-gulfstream-offline03.xg01 docker-current[25748]: time="2017-03-27T22:03:52.000155803+08:00" level=info msg="Firewalld running: false"
Mar 27 22:03:52 dataplatform-gulfstream-offline03.xg01 docker-current[25748]: time="2017-03-27T22:03:52.036279418+08:00" level=info msg="Default bridge (docker0) is assigned with an IP address 172.17.0.0/16. Daemon option --bip can be used to set a preferred IP address"
Mar 27 22:03:52 dataplatform-gulfstream-offline03.xg01 docker-current[25748]: time="2017-03-27T22:03:52.061601026+08:00" level=fatal msg="Error starting daemon: Error initializing network controller: Error creating default \"bridge\" network: Failed to setup IP tables, cannot acquire Interface address: Interface docker0 has no IPv4 addresses"
Mar 27 22:03:52 dataplatform-gulfstream-offline03.xg01 systemd[1]: docker.service: main process exited, code=exited, status=1/FAILURE
Mar 27 22:03:52 dataplatform-gulfstream-offline03.xg01 systemd[1]: Failed to start Docker Application Container Engine.
Mar 27 22:03:52 dataplatform-gulfstream-offline03.xg01 systemd[1]: Unit docker.service entered failed state.
Mar 27 22:03:52 dataplatform-gulfstream-offline03.xg01 systemd[1]: docker.service failed.


## solution:
https://github.com/docker/libnetwork/issues/892

We had a similar issue on a centos like system Scientific Linux release 7.2, with disable_ipv6=1 and using iptables instead of firewalld. The docker0 interface was left behind, and the service failed to start with a docker0 has no ipv4 addresses error.

Docker version 1.10.3

I added a systemd override file to manually clean up the "orphaned" docker0 interface whenever the service is stopped or fails to start.

Add an override file (or configure the ExecStopPost command elsewhere) /etc/systemd/system/docker.service.d/cleanup_docker0_bridge.conf:

[Service]
ExecStopPost=/bin/bash -c "/sbin/ip link show docker0 && /sbin/ip link delete docker0"
Then update systemd with:

/usr/bin/systemctl daemon-reload

thanks
