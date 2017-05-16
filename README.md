# shrine-extras
Atlanta Clinical and Translational Science Institute, Emory University, Atlanta, GA

## What is it?
This package contains scripts for installing SHRINE as a service on Red Hat and CentOS Linux systems. These instructions will make your SHRINE installation independent of the account of the user who installed SHRINE, and they will make SHRINE start automatically on boot.

Below the instructions are some extra tips we have found useful when deploying SHRINE.

## Instructions
These instructions assume you have already followed the automated install instructions on the SHRINE website and your user account on your SHRINE server has sudo access. `$SHRINE_HOME` is the path to your SHRINE installation, usually `/opt/shrine`. These instructions have been tested with SHRINE version 1.19.1 on Red Hat Enterprise Linux version 6.5.

1. Move the `common.rc` and `shrine.rc` files that the SHRINE installer placed in your home directory to `$SHRINE_HOME`.
2. Copy the `runshrine` script to `$SHRINE_HOME`. Make sure this script has execute permissions. 
3. Create a `shrine` user and `shrine` group on your server. The user account should be a service account with no home directory: `sudo useradd -M shrine`. The user is assigned $SHRINE_HOME as the home directory: `sudo usermod -d $SHRINE_HOME shrine’ 
4. Change the owner and group of your SHRINE installation to the shrine user and group: `sudo chown -R shrine:shrine $SHRINE_HOME`. If you run shrine directly through tomcat as a user other than shrine after this point, there might be new files created(e.g.:log files) that shrine would not own. You need to run the chown command again. 
5. Change the `SHRINE_HOME` variable in the shrine.rc script to your `$SHRINE_HOME`.
6. Copy the `shrine` script to `/etc/init.d`, and make its owner root: `sudo chown root:root /etc/init.d/shrine`.
7. You may now start SHRINE with the following command: `sudo service shrine start`. Similarly, stop SHRINE by invoking `sudo service shrine stop`. When starting and stopping SHRINE, you may see the following error: `Starting/Stopping Shrine... eth0: error fetching interface information: Device not found`. This appears to be a bug in SHRINE's configuration files, but it is harmless as SHRINE still starts normally.
8. To make SHRINE start automatically on boot, run `sudo chkconfig shrine on`. This command makes SHRINE start automatically in runlevel 3, which is the default runlevel for Red Hat and CentOS server installations.

## Extra tips
### Securing Tomcat
The SHRINE installation installs all of the default webapps that come with tomcat. These are not needed and can impose a security risk if not secured properly. It's easier just to remove them. The default webapps that are safe to remove are `ROOT`, `docs`, `examples`, `host-manager`, and `manager`. For the risk-averse, create a directory in `$SHRINE_HOME/tomcat` named something like `webapps-disabled` and move the default webapps there. Shutdown SHRINE while making these changes.Also remove host-manager.xml and manager.xml from conf/Catalina/host directory. To be safe, create a directory within ‘webapps-disabled’ named ‘disabled-conf’ and move the two .xml files there.

Restrict the set of ciphers that SHRINE will use for accepting HTTPS requests. SHRINE depends on default tomcat behavior, and tomcat accepts ciphers by default that are known insecure. See https://wiki.apache.org/tomcat/Security/Ciphers for instructions. The cipher list to use depends on the version of Java that you are using, and it goes in the ciphers attribute of the SSL connector in `$SHRINE_HOME/tomcat/conf/server.xml`. The Java system property described on that page can be set in `$SHRINE_HOME/tomcat/bin/setenv.sh`. Include a line in that file with `CATALINA_OPTS="put_the_system property_here"`. You will need to restart SHRINE for these changes to take effect.

### Making SHRINE's webclient the ROOT webapp
Make the `shrine-webclient` the `ROOT` webapp (just rename the `$SHRINE_HOME/tomcat/webapps/shrine-webclient` directory `ROOT`). Shutdown SHRINE while making this change.

### Starting the SHRINE database on boot
Make the database management system that your SHRINE is using start automatically on boot. For example, for MySQL installed using the default packages from CentOS or Red Hat, the command is `sudo chkconfig --level 2345 mysqld on`.

### "No route to host" error troubleshooting
The SHRINE installer does not setup the i2b2 services URL correctly for us. We get a "No route to host" exception when doing the happy SHRINE test. Analysis of SHRINE's log files (in `$SHRINE_HOME/tomcat/logs/shrine.log`) indicated that SHRINE was using an invalid URL to access our i2b2 server. To fix this, we went into `$SHRINE_HOME/tomcat/lib/shrine.conf` and edited the pmEndpoint and ontEndpoint sections at the top of the file. With a default i2b2 installation, the pmEndpoint should look like `http://<your_hostname>:9090/i2b2/services/PMService/getServices`, and your ontEndpoint should look like `http://<your_hostname>:9090/i2b2/services/OntologyService/`.

### Certificate exchange with a SHRINE hub
If you are joining a SHRINE hub, here are instructions for the certificate exchange: https://open.med.harvard.edu/wiki/plugins/servlet/mobile#content/view/23462012

Here is our adaptation of that process:
1) Ensure the keystore at $SHRINE_HOME/shrine.keystore is empty, or delete it.
2) In your own account on the SHRINE node, run `source $SHRINE_HOME/shrine.rc`.
3) Set JAVA_HOME to your Java installation, and set SHRINE_HOME to the root of your SHRINE installation.
4) Create a private key (also creates the keystore, if it does not already exist):
```
sudo $JAVA_HOME/bin/keytool -genkeypair -keysize 2048 -alias $KEYSTORE_ALIAS -dname "CN=$KEYSTORE_ALIAS, OU=$KEYSTORE_HUMAN, O=SHRINE Network, L=$KEYSTORE_CITY, S=$KEYSTORE_STATE, C=$KEYSTORE_COUNTRY" -keyalg RSA -keypass $KEYSTORE_PASSWORD -storepass $KEYSTORE_PASSWORD -keystore $KEYSTORE_FILE -validity 7300
```
5) Create a certificate signing request:
```
$JAVA_HOME/bin/keytool -certreq -alias $KEYSTORE_ALIAS -keyalg RSA -file shrine-client.csr -keypass $KEYSTORE_PASSWORD -storepass $KEYSTORE_PASSWORD -keystore $KEYSTORE_FILE 
```
6) Send the resulting file in your working directory, `shrine-client.csr`, to the hub.
