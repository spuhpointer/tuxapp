# tuxapp
Tuxedo to Enduro/X migration demo application


This is sample project which demonstrates Oracle Tuxedo to Enduro/X migration process

* Tuxedo install path: /opt/tuxedo12.2.2.0.0 (see tux.env)
* Oracle DB setting (common.env)
* Prapre DB users according to src/txlogsv/README
* Demo is prepared on Centos 8 Linux, Oracle 18 XE database used

# Building

## Tuxedo version

To build Tuxedo version:

* Install Tuxedo accordingly
* Then:

```
    $ . tux.env
    $ cd src
    $ make clean
    $ make
```

## Enduro/X version

To build Enduro/X version:
* Install Enduro/X accordingly:
*   Download RPM from https://www.mavimax.com/downloads 
*   Install RPM e.g. $ sudo rpm -i endurox-8.0.4-1.oracleserver8_0_GNU_epoll.x86_64.rpm
*   Configure system limits according to https://www.endurox.org/dokuwiki/doku.php?id=endurox:v8.0.x:guides:ex_adminman chapter "2.2. Linux setup"
* Then:

```
    $ . ex.env
    $ cd src
    $ make clean
    $ make
```

# Running 

## Tuxedo version

```
    $ . tux.env
    $ cd conf
    $ tmloadcf -b 30000 ubbconfig -y
    -- For first run TLOG & myspace shall be created, use script:
    $ ./tux_first.sh
    $ tmboot -y
    $ txgencl
    -- Check databases:
    $ sqlplus1
    SQL> select count(*) from txlog;
    $ sqlplus2
    SQL> select count(*) from txlog;
```

## Enduro/X version

This includes migration step - ubb2ex for converting Tuxedo configuration to Enduro/X format:

```
    $ . ex.env
    $ cd conf
    -- Convert Tuxedo config to Enduro/X, first time (this generates setmatch1, app.mach1.ini, ndrxconfig.mach1.xml):
    $ ubb2ex ubbconfig
    $ . setmach1
    $ xadmin start -y
    $ txgencl
    -- Check databases:
    $ sqlplus1
    SQL> select count(*) from txlog;
    $ sqlplus2
    SQL> select count(*) from txlog;
```

# Application

Given application demonstrates XATMI client which calls XATMI server, which enqueues message to persistent queue. The persistent queue is forwarded to "TXLOG" service. "TXLOG" service adds records in Oracle DB table "TXLOG". There are two servers txlogsv where each is connected to Oracle DB to different schemas. The target server is selected by DDR route. Do not confuse, in this example "TXLOG" queue/service/table refer to business transactions log and is not related to Tuxedo internal TLOG which is used for XA transaction tracking. 

![Application diagram](doc/program.drawio.png?raw=true "Application diagram")
