TP5
I. Préparation du lab
1. Préparation VMs
On paramétre la machine en lui donnant une nouvelle carte "host-only" qui va faire la connexion SSH.
On clone le patron pour avoir les 3 VMs et on lie dans gns3 en précisant qu'il y a 2 interfaces réseau.

2. Préparation Routeurs Cisco
On importe l'image pour mettre en place les 2 routeurs. Le réseau
a utilisé pour les routeurs peut être 10.5.3.0/30 pour utilisé les ip 10.5.12.1
et 10.5.12.2 pour les deux routeurs.

Récapitulation des IPs:
Machines	net1	net2	net3
client1.tp5	X	10.5.2.10	X
client2.tp5	X	10.5.2.11	X
router1.tp5	10.5.1.254	X	10.5.12.1
router2.tp5	X	10.5.2.254	10.5.12.2
server1.tp5	10.5.1.10	X	X

II. Lancement et configuration du lab
Checklist IP VMs
La désactivation de SELinux, les installation et la désactivation de la carte
ont été précédemment. Il faut ensuite définir les IPs statiques, les noms de
domaines et avoir une connexion ssh, pour ce il faut suivre la procédure indiquée
dans les TPs précédents.

Checklist IP Routeurs
Pour la définition des IPs statiques et des noms de domaines des routeurs
router1.tp5.b1 et router2.tp5.b1 il faut suivre la procédure.

Pour le router1:
(config-if)# ip address 10.5.1.254 255.255.255.0
(config-if)# hostname router1.tp5
Checklist routes
 router1:
(config)# ip route 10.5.2.0 255.255.255.0 10.5.12.2
 server1:

Routes
[root@server1 ~]# ip route add 10.2.0.0/24 via 10.2.0.254 dev eth0
Temporaire
[root@server1 ~]# nano /etc/sysconfig/network-scripts/route-eth0
10.2.0.0/24 via 10.2.0.254 dev eth0

Définitif
-Hosts
[root@server1 ~]# nano /etc/hosts
10.5.2.10 client1 client1.tp5
10.5.2.11 client2 client2.tp5

Depuis client2:
[root@client2 ~]# ping client1.tp5 -c 2 && echo &&  ping server1.tp5 -c 2
PING client1 (10.5.2.10) 56(84) bytes of data.
64 bytes from client1 (10.5.2.10): icmp_seq=1 ttl=64 time=5.96 ms
64 bytes from client1 (10.5.2.10): icmp_seq=2 ttl=64 time=7.83 ms

--- client1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1005ms
rtt min/avg/max/mdev = 4.830/5.683/6.537/0.934 ms

PING server1 (10.5.1.10) 56(84) bytes of data.
64 bytes from server1 (10.5.1.10): icmp_seq=1 ttl=62 time=35.7 ms
64 bytes from server1 (10.5.1.10): icmp_seq=2 ttl=62 time=28.9 ms

--- server1 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1004ms
rtt min/avg/max/mdev = 27.911/30.842/33.774/2.923 ms

III. DHCP

1. Mise en place du serveur DHCP
1. Renommer la machine
[root@client2 ~]# hostname dhcp-net2.tp5
[root@client2 ~]# nano /etc/hostname
hostname dhcp-net2.tp5

2. Installer le serveur DHCP
La carte NAT (eth2) est déjà présente pour la réinstallation des VMs sur eve-ng
il faut l'activer avec ifup eth2
[root@client2 ~]# ip a show dev eth2
4: eth2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP
group default qlen 1000
    link/ether 00:50:00:00:05:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.20.17/24 brd 192.168.20.255 scope global noprefixroute dynamic eth2
       valid_lft 7143sec preferred_lft 7143sec
    inet6 fe80::250:ff:fe00:502/64 scope link
       valid_lft forever preferred_lft forever
Ensuite il faut installer les paquets pour le serveur dhccp:

[root@dhcp-net2 ~]# yum install -y dhcp
[root@dhcp-net2 ~]# ifup eth0 && ifdown eth2

5. Démarrer le serveur DHCP
Après avoir modifié le fichier de configuration:
[root@dhcp-net2 ~]# systemctl start dhcpd
Pour le démarrage automatique:
[root@dhcp-net2 ~]# systemctl enable dhcpd
Pour vérifier que le service est démarré:
[root@dhcp-net2 ~]# systemctl status dhcpd -l
● dhcpd.service - DHCPv4 Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
   Active: active (running) since lun. 2019-02-25 21:30:20 CET; 2min 30s ago
     Docs: man:dhcpd(8)
           man:dhcpd.conf(5)
 Main PID: 3881 (dhcpd)
   Status: "Dispatching packets..."
   CGroup: /system.slice/dhcpd.service
           └─3881 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

févr. 25 21:30:20 dhcp-net2.tp5.b1 dhcpd[3881]: Listening on LPF/eth0/00:50:00:00:05:00/10.5.2.0/24
févr. 25 21:30:20 dhcp-net2.tp5.b1 dhcpd[3881]: Sending on   LPF/eth0/00:50:00:00:05:00/10.5.2.0/24
