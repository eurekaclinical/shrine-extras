# shrine-extras

This package contains scripts for installing SHRINE as a service on Red Hat and CentOS Linux systems. These instructions will make your SHRINE installation independent of the account of the user who installed SHRINE, and they will make SHRINE start automatically on boot.

Below the instructions are some extra tips we have found useful when deploying SHRINE.

## Instructions

These instructions assume you have already followed the automated install instructions on the SHRINE website and your user account on your SHRINE server has sudo access. `$SHRINE_HOME` is the path to your SHRINE installation, usually `/opt/shrine`. These instructions have been tested with SHRINE version 1.19.1 on Red Hat Enterprise Linux version 6.5.

1. Copy the `common.rc` and `shrine.rc` files that the installer placed in your home directory to `$SHRINE_HOME`.
2. Copy the `runshrine` script to `$SHRINE_HOME`.
3. Create a `shrine` user and `shrine` group on your server. The user account should be a service account with no home directory.
4. Change the owner and group of your SHRINE installation to the shrine user and group: `sudo chown -R shrine:shrine $SHRINE_HOME`.
5. Change the `HOME` variable in the shrine script to your `$SHRINE_HOME`.
6. Copy the `shrine` script to `/etc/init.d`, and make its owner root: `sudo chown root:root /etc/init.d/shrine`.
7. You may now start SHRINE with the following command: `sudo service shrine start`. Similarly, stop SHRINE by invoking `sudo service shrine stop`. When starting and stopping SHRINE, you may see the following error: `Starting/Stopping Shrine... eth0: error fetching interface information: Device not found`. This appears to be a bug in SHRINE's configuration files, but it is harmless as SHRINE still starts normally.
8. To make SHRINE start automatically on boot, run `sudo chkconfig --level 3 shrine on`. This command makes SHRINE start automatically in runlevel 3, which is the default runlevel for Red Hat and CentOS server installations.

## Extra tips

You likely will want your database management system to start automatically. For example, for MySQL installed using the default packages from CentOS or Red Hat, the command is `sudo chkconfig --level 2345 mysqld on`.

You also likely will want to restrict the set of ciphers that SHRINE will use for accepting HTTPS requests. SHRINE depends on default tomcat behavior, and tomcat accepts ciphers by default that are known insecure. See https://wiki.apache.org/tomcat/Security/Ciphers for instructions. The cipher list to use depends on the version of Java that you are using, and it goes in the ciphers attribute of the SSL connector in `$SHRINE_HOME/tomcat/conf/server.xml`. The Java system property described on that page can be set in `$SHRINE_HOME/tomcat/bin/setenv.sh`. Include a line in that file with `CATALINA_OPTS="put_the_system property_here"`. You will need to restart SHRINE for these changes to take effect.

The SHRINE installer does not setup the i2b2 services URL correctly for us. We get a "No route to host" exception when doing the happy SHRINE test. Analysis of SHRINE's log files (in `$SHRINE_HOME/tomcat/logs/shrine.log`) indicated that SHRINE was using an invalid URL to access our i2b2 server. To fix this, we went into `$SHRINE_HOME/tomcat/lib/shrine.conf` and edited the pmEndpoint and ontEndpoint sections at the top of the file. With a default i2b2 installation, the pmEndpoint should look like `http://<your_hostname>:9090/i2b2/services/PMService/getServices`, and your ontEndpoint should look like `http://<your_hostname>:9090/i2b2/services/OntologyService/`.
