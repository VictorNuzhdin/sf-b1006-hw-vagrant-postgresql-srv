# -*- mode: ruby -*-
# vi: set ft=ruby :

## Set Variables
$VM_BASE_BOX = "ubuntu/bionic64"     ## defines vm base box/image: Ubuntu 18.04.6 LTS (Bionic Beaver);
#$VM_NAME = "userver1804"            ## defines vm names of: gui display name, vagrant vm name, hostname
$VM_NAME = "ubuntu1804-pgsql84-srv"
$VM_CPUS = 2                         ## defines vm number of CPUs/Cores;
$VM_RAMS = 2048                      ## defines vm number of RAM in MB;
$VM_LAN_IP = "192.168.0.70"          ## defines vm ip-address in your local network;
##


## Begin of Vagrantfile configuration (syntax version is v2)
Vagrant.configure("2") do |config|

  config.vm.box = $VM_BASE_BOX                              ## base box/image of vm;

  config.vm.box_check_update = false                        ## disable box updates from Vagrant Cloud;
  config.vm.define $VM_NAME                                 ## vm name (default name is "default");
  config.vm.hostname = $VM_NAME                             ## network hostname;

  ## Network configuration
  config.vm.network "public_network", ip: $VM_LAN_IP        ## public accessible ip-address of VM interface (from internal LAN address range);

  ## Provider-specific configuration (Oracle VirtualBox)
  config.vm.provider "virtualbox" do |vb|
    vb.name = $VM_NAME                                      ## vm display name in VirtualBox GUI;
    vb.gui = false                                          ## disable VM console in VirtualBox GUI when VM is creating;
    vb.check_guest_additions=false                          ## disabled is "Guest Additions Tools" is installed on VM (this tools is not required on servers);
    vb.cpus = $VM_CPUS                                      ## number of CPUs/Cores;
    vb.memory = $VM_RAMS                                    ## number of RAM in MB;
  end


  ## Provisioning: Installing PostgreSQL 8.4 on Guest host
  ## https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04
  ## https://askubuntu.com/questions/149304/how-do-i-install-older-postgresql-8-4-in-ubuntu-12-04
  ## https://github.com/ansible/ansible/issues/2763
  config.vm.provision "DeployPostgres84", type: "shell", inline:<<-SHELL

    ## STEP
    echo
    echo "--STEP10: Creating special sudo user.."
    useradd -p '$6$y8aQ4FxWTNrDzVSa$sG1j1s7dfTu3PlKwrmv7NLDuH7ADnY6rB26aZ9HJPpE3ucqwbeLAQiHj81xE3Z8BoDlzfWm1LhNktYsL07/E7.' -G sudo --create-home --shell /bin/bash devops
    touch /etc/sudoers.d/devops
    #echo "devops  ALL=(ALL:ALL) ALL" >> /etc/sudoers.d/devops
    echo "devops  ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/devops
    id devops
    echo

    ## STEP
    echo "--STEP11: Generating ssh key pair and configuring ssh serviice for user connection.."
    sudo -u devops /bin/bash -c "mkdir -p /home/devops/.ssh"
    chmod u=rwx,g=,o= /home/devops/.ssh
    sudo -u devops /bin/bash -c "echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIBfGokGoaKzkCofNwwTPVko198HhAuwHeD+F6xO5Khk5 devops@userver1804' >> /home/devops/.ssh/authorized_keys"    
    chmod u=rw,g=,o= /home/devops/.ssh/authorized_keys
    echo

    ## STEP
    echo "--STEP20: Installing some utils.."
    apt install -y wget ca-certificates net-tools 2>/dev/null
    echo

    ## STEP
    echo "--STEP21: Adding PostgreSQL repository.."
    echo "deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main" >> /etc/apt/sources.list.d/pgdg.list
    tail -n1 /etc/apt/sources.list.d/pgdg.list
    wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    echo
    
    ## STEP
    echo "--STEP22: Updating package list.."
    apt update -y 2>/dev/null
    echo

    ## STEP
    echo "--STEP23: Installing PostgreSQL v8.4 and components.."
    #apt install -y postgresql-8.4 postgresql-contrib-8.4 2>/dev/null
    #apt install -y postgresql-8.4
    #apt install -y postgresql-contrib-8.4   ## strange behavior: step "Setting up postgresql-common (248.pgdg18.04+1)" is never finished
    #apt install -y postgresql-contrib       ## same problem with: "Setting up postgresql-common (248.pgdg18.04+1)"
    #
    # REASON: during package installation of "postgresql-common" stupid interactive "window" requests for press "OK" !?!?
    # FIX: supress interactive mode for apt:
    DEBIAN_FRONTEND=noninteractive DEBIAN_PRIORITY=critical apt-get --option Dpkg::Options::=--force-confold -q -y install postgresql-8.4 postgresql-contrib-8.4
    echo

    ## STEP
    echo "--STEP24: Starting PostgreSQL service.."
    systemctl start postgresql.service
    echo

    ## STEP
    echo "--STEP25: Checking PostgreSQL service status.."
    systemctl status postgresql.service
    echo

    ## STEP
    echo "--STEP26: Checking PostgreSQL path and version.."
    find / -wholename '*/bin/postgres' 2>&- | xargs -i xargs -t '{}' -V
    echo

    ## STEP
    echo "--STEP27: Executing select query (show PostgreSQL version).."
    sudo -u postgres psql -c "select version()::varchar(100);" | grep PostgreSQL | awk '{print $1" "$2}'
    echo

    ## STEP
    echo "--STEP30: Configuring PostgreSQL (creating db user, user database, grant access).."
    sudo -u postgres psql -c "CREATE DATABASE vagrant;"
    sudo -u postgres psql -c "CREATE USER vagrant WITH PASSWORD 'vagrant@pass';"
    sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE vagrant to vagrant;"
    echo

    ## STEP
    echo "--STEP31: Configuring PostgreSQL (creating sample data).."
    sudo -u vagrant psql -c "CREATE TABLE greetings (lang VARCHAR (3) UNIQUE NOT NULL, word VARCHAR (50) NOT NULL);"
    sudo -u vagrant psql -c "INSERT INTO greetings (lang, word) values ('ENG', 'hello'), ('FRA', 'bonjour'), ('ESP', 'hola'), ('ITA', 'ciao'), ('DEU', 'hallo'), ('UKR', 'вітаю'), ('BLR', 'прывітанне'), ('RUS', 'привет');"
    echo

    ## STEP
    echo "--STEP32: Configuring PostgreSQL (adding some connection permissions).."
    echo "listen_addresses='*'" >> /etc/postgresql/8.4/main/postgresql.conf
    echo "# CUSTOM SETTINGS" >> /etc/postgresql/8.4/main/pg_hba.conf
    #echo "host    vagrant     vagrant     192.168.0.100/32      md5" >> /etc/postgresql/8.4/main/pg_hba.conf
    echo "host    vagrant     vagrant     0.0.0.0/0             md5" >> /etc/postgresql/8.4/main/pg_hba.conf
    echo

    ## STEP
    echo "--STEP33: Restarting PostgreSQL service for apply new configuration.."
    systemctl restart postgresql.service
    echo

    ## STEP
    echo "--STEP34: Checking PostgreSQL service status.."
    systemctl status postgresql | grep Active
    systemctl status postgresql@8.4-main.service | grep Active
    echo "PostgreSQL tcp port status:"
    netstat  -tunelp | grep 5432
    echo

    echo "--FINISHED;"

  SHELL

end
