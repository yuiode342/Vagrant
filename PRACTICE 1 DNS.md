
# TIERRA CONFIGURATION

1. In the first step you have to go to the /etc/network/interfaces document and add the conexion for the enp0s8 and assign a fixed ip address (192.168.57.102).

``` 
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
iface enp0s3 inet dhcp

allow-hotplug enp0s8
iface enp0s8 inet static
address 192.168.57.102

```

2. Next we have to install the package bind9 with the order sudo apt-get install bind9 bind9utils bind9-doc

3. Now we start configuring the server, first we enterr in the file /etc/default/named, and que change the next paramater --> OPTIONS = "-u bind -4".
```
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-u bind -4"

```

Next we have to chenge the file /etc/resolve.conf and put this:
```
nameserver 192.168.57.103
nameserver 192.168.57.102
```
Before we continue, we do a backup of the file --> /etc/bind/named.conf.options /etc/bind/named.conf.options.backup

4. When we had that done, the first file we have to touch is /etc/bind/named.conf.options. The file haves  to look like this.
```
acl confiables {
        192.168.57.0/24;
        127.0.0.0/8;
};

options {
        directory "/var/lib/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
                208.67.222.222;
        };

        allow-transfer { 192.168.57.103; };

        listen-on port 53 { 192.168.57.102; };

        recursion yes;
        allow-recursion { confiables; };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation yes;

        // listen-on-v6 { any; };
};


```

Then we check the configuration with the next command --> named-checkconf /etc/bind/named.conf.options
If all is okay, we do systemctl restart named systemctl status named 

5. The next file we have to modify is /etc/bind/named.conf.local. It has to look like that
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "sistema.sol" {
        type master;
        file "/var/lib/bind/sistema.sol.dns";
};

zone "57.168.192.in-addr.arpa" {
        type master;
        file "/var/lib/bind/sistema.sol.rev";
};
```

6. Then we create the file /var/lib/bind/sistema.sol.dns
```
;
; sistema.sol
;
$TTL    86400
@       IN      SOA     tierra.sistema.sol.     root.sistema.sol. (
                        1       ;  Serial
                        3600    ;  Refresh
                        1800    ;  Retry
                        604800  ;  Expire
                        86400)  ;  Negative Cache TTL
;
@       IN      NS      tierra.sistema.sol.
tierra.sistema.sol.     IN      A       192.168.57.102
venus.sistema.sol.      IN      A       192.168.57.103
marte.sistema.sol.      IN      A       192.168.57.104
ns1     IN      CNAME   tierra.sistema.sol.
ns2     IN      CNAME   venus.sistema.sol.
mail.sistema.sol.       IN      CNAME   marte.sistema.sol.
@       IN      MX 10   marte.sistema.sol.



```

7. Then we create the file /var/lib/bind/sistema.sol.rev
```
;
; 57.168.192
;
$TTL    86400
@       IN      SOA     tierra.sistema.sol.     admin.deaw.es. (
                                1       ;  Serial
                                3600    ;  Refresh
                                1800    ;  Retry
                                604800  ;  Expire
                                86400 ) ;  Negative Cache TTL
;
@       IN      NS      tierra.sistema.sol.
102     IN      PTR     tierra.sistema.sol.
103     IN      PTR     venus.sistema.sol.
104     IN      PTR     marte.sistema.sol.

```

8. Comprobamos las configuraciones con los siguientes comando.
named-checkzone sistema.sol. /var/lib/bind/sistema.sol.dns
named-checkzone 57.168.192.in-addr.arpa. /var/lib/bind/sistema.sol.rev

# VENUS CONFIGURATON

1. In the first step you have to go to the /etc/network/interfaces document and add the conexion for the enp0s8 and assign a fixed ip address (192.168.57.103).
```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp0s3
iface enp0s3 inet dhcp

allow-hotplug enp0s8
iface enp0s8 inet static
address 192.168.57.103
```

2. Next we have to install the package bind9 with the order sudo apt-get install bind9 bind9utils bind9-doc

3. Now we start configuring the server, first we enterr in the file /etc/default/named, and que change the next paramater --> OPTIONS = "-u bind -4".
```
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-u bind -4"
```

Before we continue, we do a backup of the file --> /etc/bind/named.conf.options /etc/bind/named.conf.options.backup

4. When we had that done, the first file we have to touch is /etc/bind/named.conf.options. The file haves  to look like this.
```
acl confiables {
        192.168.57.0/24;
        127.0.0.0/8;
};

options {
        directory "/var/lib/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

         forwarders {
                208.67.222.222;
         };

        listen-on port 53 { 192.168.57.103; };

        recursion yes;
        allow-recursion { confiables; };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation yes;

        // listen-on-v6 { any; };
};


```

Then we check the configuration with the next command --> named-checkconf /etc/bind/named.conf.options
If all is okay, we do systemctl restart named systemctl status named 

5. The next file we have to modify is /etc/bind/named.conf.local. It has to look like that
```
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "sistema.sol" {
        type slave;
        file "/var/lib/bind/sistema.sol.dns";
        masters {
                192.168.57.102;
        };
};

zone "57.168.192.in-addr.arpa" {
        type slave;
        file "/var/lib/bind/sistema.sol.rev";
        masters {
                192.168.57.102;
        };
};

```

6. Once we have done this, if we did ir all correctly, we do a systemctl restart named, and if we do a status, the logs show us that the configuration had been copied from the ip 192.168.57.102

# CHECKS

8. We do the command dig from venus to tierra and that has te be showed.
```
usuario@debian:~$ dig tierra.sistema.sol

; <<>> DiG 9.18.19-1~deb12u1-Debian <<>> tierra.sistema.sol
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8216
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 441fa9f0b50e830901000000655f4b79e6142d71c74771ba (good)
;; QUESTION SECTION:
;tierra.sistema.sol.            IN      A

;; ANSWER SECTION:
tierra.sistema.sol.     86400   IN      A       192.168.57.102

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Thu Nov 23 13:54:17 CET 2023
;; MSG SIZE  rcvd: 91

```

Then dig to venus
```
usuario@debian:~$ dig venus.sistema.sol

; <<>> DiG 9.18.19-1~deb12u1-Debian <<>> venus.sistema.sol
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18489
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 6eb65bb078f92e4301000000655f4c0a7f8d5e9bfd8bf0f7 (good)
;; QUESTION SECTION:
;venus.sistema.sol.             IN      A

;; ANSWER SECTION:
venus.sistema.sol.      86400   IN      A       192.168.57.103

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Thu Nov 23 13:56:42 CET 2023
;; MSG SIZE  rcvd: 90

```

Then dig to marte
```
usuario@debian:~$ dig marte.sistema.sol

; <<>> DiG 9.18.19-1~deb12u1-Debian <<>> marte.sistema.sol
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12726
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: dc5a83ae58d8f87801000000655f4e97cc758c809c795550 (good)
;; QUESTION SECTION:
;marte.sistema.sol.             IN      A

;; ANSWER SECTION:
marte.sistema.sol.      86400   IN      A       192.168.57.104

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Thu Nov 23 14:07:35 CET 2023
;; MSG SIZE  rcvd: 90

usuario@debian:~$

```

Then dig to ns1
```
usuario@debian:~$ dig ns1.sistema.sol

; <<>> DiG 9.18.19-1~deb12u1-Debian <<>> ns1.sistema.sol
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34667
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: a63635db25b6c27e01000000655f4c42f22422080f55f998 (good)
;; QUESTION SECTION:
;ns1.sistema.sol.               IN      A

;; ANSWER SECTION:
ns1.sistema.sol.        86400   IN      CNAME   tierra.sistema.sol.
tierra.sistema.sol.     86400   IN      A       192.168.57.102

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Thu Nov 23 13:57:38 CET 2023
;; MSG SIZE  rcvd: 109

```

And to ns2
```
usuario@debian:~$ dig ns2.sistema.sol

; <<>> DiG 9.18.19-1~deb12u1-Debian <<>> ns2.sistema.sol
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 5
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 78db1fd03f4e877901000000655f4c7ef672e41ac808df40 (good)
;; QUESTION SECTION:
;ns2.sistema.sol.               IN      A

;; ANSWER SECTION:
ns2.sistema.sol.        86400   IN      CNAME   venus.sistema.sol.
venus.sistema.sol.      86400   IN      A       192.168.57.103

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Thu Nov 23 13:58:38 CET 2023
;; MSG SIZE  rcvd: 108

```

We check the resolves with the inverse, with NS, and with MX
```
usuario@debian:~$ dig -x 192.168.57.102

; <<>> DiG 9.18.19-1~deb12u1-Debian <<>> -x 192.168.57.102
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 24116
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 146d6a33dc5fee0701000000655f4e348f0cb63e6b40994c (good)
;; QUESTION SECTION:
;102.57.168.192.in-addr.arpa.   IN      PTR

;; ANSWER SECTION:
102.57.168.192.in-addr.arpa. 86400 IN   PTR     tierra.sistema.sol.

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Thu Nov 23 14:05:56 CET 2023
;; MSG SIZE  rcvd: 116

usuario@debian:~$

```

```
usuario@debian:~$ dig -x 192.168.57.103

; <<>> DiG 9.18.19-1~deb12u1-Debian <<>> -x 192.168.57.103
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44006
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 075a901323ef877d01000000655f4e5133b6c9a485ee72d8 (good)
;; QUESTION SECTION:
;103.57.168.192.in-addr.arpa.   IN      PTR

;; ANSWER SECTION:
103.57.168.192.in-addr.arpa. 86400 IN   PTR     venus.sistema.sol.

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Thu Nov 23 14:06:25 CET 2023
;; MSG SIZE  rcvd: 115

usuario@debian:~$

```

```
usuario@debian:~$ dig -x 192.168.57.104

; <<>> DiG 9.18.19-1~deb12u1-Debian <<>> -x 192.168.57.104
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51221
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: cec28b070468c4bf01000000655f4e6aba1f3464f158055a (good)
;; QUESTION SECTION:
;104.57.168.192.in-addr.arpa.   IN      PTR

;; ANSWER SECTION:
104.57.168.192.in-addr.arpa. 86400 IN   PTR     marte.sistema.sol.

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Thu Nov 23 14:06:50 CET 2023
;; MSG SIZE  rcvd: 115

usuario@debian:~$

```

```
usuario@debian:~$ dig NS sistema.sol

; <<>> DiG 9.18.19-1~deb12u1-Debian <<>> NS sistema.sol
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 40685
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: c11d1dd9c389ec3901000000655f528ae2363a0e336865e8 (good)
;; QUESTION SECTION:
;sistema.sol.                   IN      NS

;; ANSWER SECTION:
sistema.sol.            86400   IN      NS      venus.sistema.sol.
sistema.sol.            86400   IN      NS      tierra.sistema.sol.

;; ADDITIONAL SECTION:
venus.sistema.sol.      86400   IN      A       192.168.57.103
tierra.sistema.sol.     86400   IN      A       192.168.57.102

;; Query time: 0 msec
;; SERVER: 192.168.57.102#53(192.168.57.102) (UDP)
;; WHEN: Thu Nov 23 14:24:26 CET 2023
;; MSG SIZE  rcvd: 141

usuario@debian:~$


```

```
usuario@debian:~$ dig MX sistema.sol

; <<>> DiG 9.18.19-1~deb12u1-Debian <<>> MX sistema.sol
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18719
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: b6b7c9c4bd96e59f01000000655f4fd633aa9f310db4ac44 (good)
;; QUESTION SECTION:
;sistema.sol.                   IN      MX

;; ANSWER SECTION:
sistema.sol.            86400   IN      MX      10 marte.sistema.sol.

;; ADDITIONAL SECTION:
marte.sistema.sol.      86400   IN      A       192.168.57.104

;; Query time: 0 msec
;; SERVER: 192.168.57.103#53(192.168.57.103) (UDP)
;; WHEN: Thu Nov 23 14:12:54 CET 2023
;; MSG SIZE  rcvd: 106

usuario@debian:~$

```

We check that the transfer was succesfull in the zone between the  master server DNS and the slave.
```
nov 23 12:44:38 debian systemd[1]: Started named.service - BIND Domain Name Server.
nov 23 12:44:38 debian named[1178]: zone sistema.sol/IN: Transfer started.
nov 23 12:44:38 debian named[1178]: transfer of 'sistema.sol/IN' from 192.168.57.102#53: connected using 192.168.57.102#53
nov 23 12:44:38 debian named[1178]: zone sistema.sol/IN: transferred serial 1
nov 23 12:44:38 debian named[1178]: transfer of 'sistema.sol/IN' from 192.168.57.102#53: Transfer status: success
nov 23 12:44:38 debian named[1178]: transfer of 'sistema.sol/IN' from 192.168.57.102#53: Transfer completed: 1 messages, 10 records, 258 bytes, 0.001 secs (258000 bytes/sec) (serial 1)
nov 23 12:44:39 debian named[1178]: zone 57.168.192.in-addr.arpa/IN: Transfer started.
nov 23 12:44:39 debian named[1178]: transfer of '57.168.192.in-addr.arpa/IN' from 192.168.57.102#53: connected using 192.168.57.102#53
nov 23 12:44:39 debian named[1178]: zone 57.168.192.in-addr.arpa/IN: transferred serial 1
nov 23 12:44:39 debian named[1178]: transfer of '57.168.192.in-addr.arpa/IN' from 192.168.57.102#53: Transfer status: success
nov 23 12:44:39 debian named[1178]: transfer of '57.168.192.in-addr.arpa/IN' from 192.168.57.102#53: Transfer completed: 1 messages, 6 records, 224 bytes, 0.001 secs (224000 bytes/sec) (serial 1)
nov 23 12:47:02 debian systemd[1]: Stopping named.service - BIND Domain Name Server...

```
