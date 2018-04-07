# Senior Project
Welcome, this is the GitHub repository for Victor Zuniga's senior CS project. The aim of this project is to create a network of Raspberry Pis operating as IoT hubs, connecting them in a blockchain network using Hyperledger Fabric, and implementing logic to allow for fault tolerance amongst the hubs.

## Setup
These are the steps to create the base system on each RPi. I have had a number of issues pop up while installing everything.
Sometimes the same process on different RPis has different results and requires trying again. Note that Zabbix was installed
with the original builds but is no longer used.

* #### Credits

  * Network Communication Program:

    * http://blog.whaleygeek.co.uk/raspberry-pi-network-chat-in-python/


  * Install Postgres on RPi:

    * https://opensource.com/article/17/10/set-postgres-database-your-raspberry-pi


  * Install Zabbix on RPi:

    * http://devopsish.blogspot.com/2016/05/installing-zabbix-3-on-raspberry-pi.html

    * https://www.zabbix.com/documentation/3.0/manual/appendix/install/db_scripts

    * https://www.zabbix.com/documentation/1.8/manual/quickstart


  * Install Hyperledger Fabric on RPi:

    * http://www.joemotacek.com/hyperledger-fabric-v1-0-on-a-raspberry-pi-docker-swarm-part-1/

    * http://www.joemotacek.com/hyperledger-fabric-v1-0-on-a-raspberry-pi-docker-swarm-part-2/


  * Potentially useful links for Hyperledger install:

    * https://www.admfactory.com/how-to-install-golang-on-raspberry-pi/

    * https://stackoverflow.com/questions/45800167/hyperledger-fabric-on-raspberry-pi-3


1. #### Set RPi configuration based on locale

1. #### Install PostgreSQL
    `sudo apt install postgresql libpq-dev postgresql-client postgresql-client-common -y`

1. #### Install Zabbix (note: no longer needed)
    * Install Packages
    
    ```
    sudo apt-get install php5 apache2 php5-gd php5-pgsql php5-ldap snmpd libiksemel3 libodbc1 libopenipmi0 fping ttf-dejavu-core ttf-japanese-gothic
    ```

    * Unzip Files
    
    ```
    wget https://github.com/imkebe/zabbix3-rpi/archive/master.zip
    unzip master.zip
    cd zabbix3-rpi-master
    ```

    * Install Packages
  
    ```
    sudo dpkg -i zabbix-server-pqsql_3.0.*+jessie_armhf.deb
    sudo dpkg -i zabbix-frontend-php_3.0.*+jessie_all.deb
    sudo dpkg -i zabbix-agent_3.0.*+jessie_armhf.deb
    sudo service apache2 reload
    ```

    * Setup for DB
  
    ```
    sudo -u postgres createuser --pwprompt zabbix
    sudo -u postgres createdb -O zabbix zabbix
    zcat /usr/share/doc/zabbix-server-pgsql/create.sql.gz | sudo -u zabbix psql zabbix
    ```

    * Uncomment line for `DBPassword` and enter password
  
    ```
    sudo nano /etc/zabbix/zabbix_server.conf
    ```

    * Uncomment line for timezone and enter correct timezone
  
    ```
    sudo nano /etc/apache2/conf-enabled/zabbix.conf
    ```

    * Start Services
  
    ```
    sudo service apache2 restart
    sudo service zabbix-server start
    sudo service zabbix-agent start
    ```

    * Zabbix can now be reached at localhost/zabbix
  
	  * To ensure services start on reboot edit `/etc/rc.local` and before the `exit 0` enter:

    ```
    sudo systemctl daemon-reload
    sudo service apache2 start
    sudo service zabbix-server start
    sudo service zabbix-agent start
    ```

1. #### Install Hyperledger Fabric

    * Update
    
    ```
    sudo apt-get update
    sudo apt-get dist-upgrade
    ```

    * Install Go
    
    ```
    wget https://dl.google.com/go/go1.7.5.linux-armv6l.tar.gz
    sudo tar -C /usr/local -xzf go1.7.5.linux-armv6l.tar.gz
    sudo echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.profile
    sudo echo 'export GOPATH=$HOME/go' >> ~/.profile
    ```
    * Install Docker
    
    ```
    curl -sSL https://get.docker.com | sh
    ```
    * Install Docker Compose
    
    ```
    curl -s https://packagecloud.io/install/repositories/Hypriot/Schatzkiste/script.deb.sh | sudo bash
    ```
    
    * Install Packages
    
    ```
    sudo apt-get install git python-pip python-dev docker-compose build-essential libtool libltdl-dev libssl-dev libevent-dev libffi-dev 
    ```
    
    * Install Python Libs
    
    ```
    sudo pip install --upgrade pip
    sudo pip install --upgrade setuptools
    sudo pip install behave nose docker-compose
    sudo pip install -I flask==0.10.1 python-dateutil==2.2 pytz==2014.3 pyyaml==3.10 couchdb==1.0 flask-cors==2.0.1 requests==2.4.3 pyOpenSSL==16.2.0 pysha3==1.0b1 grpcio==1.0.4
    ```
    
    * Reboot & Verify Everything Installed
    
    ```
    go version
    docker -v
    docker-compose -v
    pip -V
    git --version
    echo $GOPATH
    ```
    
    * Pull Premade Images
    
    ```
    docker pull jmotacek/fabric-baseos:armv7l-0.3.2
    docker pull jmotacek/fabric-basejvm:armv7l-0.3.2
    docker pull jmotacek/fabric-baseimage:armv7l-0.3.2
    docker pull jmotacek/fabric-ccenv:armv7l-1.0.7
    docker pull jmotacek/fabric-javaenv:armv7l-1.0.7
    docker pull jmotacek/fabric-peer:armv7l-1.0.7
    docker pull jmotacek/fabric-orderer:armv7l-1.0.7
    docker pull jmotacek/fabric-buildenv:armv7l-1.0.7
    docker pull jmotacek/fabric-testenv:armv7l-1.0.7
    docker pull jmotacek/fabric-zookeeper:armv7l-1.0.7
    docker pull jmotacek/fabric-kafka:armv7l-1.0.7
    docker pull jmotacek/fabric-couchdb:armv7l-1.0.7
    docker pull jmotacek/fabric-tools:armv7l-1.0.7
    ```
    
    *  Build Hyperledger
    
    ```
    make native docker license spelling linter unit-test BASE_DOCKER_NS=jmotacek ARCH=armv7l BASEIMAGE_RELEASE=0.3.2
    ```
    
    * 
