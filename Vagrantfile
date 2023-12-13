# -*- mode: ruby -*-
# vi: set ft=ruby :

##
## Recommended plugins
##
# - vagrant-disksize : https://github.com/sprotheroe/vagrant-disksize
# - vagrant-vbguest : https://github.com/dotless-de/vagrant-vbguest
##
##


MEMORY = "4096"
CPUS = "2"
DISK_SIZE = "100GB"
NODES = 3

system("
  if [ #{ARGV[0]} = 'up' ]; then
    echo 'Creating SSH keys...'
    ssh-keygen -t rsa -N '' -f id_rsa

    cat id_rsa.pub > authorized_keys
  fi
")

Vagrant.configure("2") do |config|
  
  config.vm.box = "bento/ubuntu-22.04"

  username = 'iabd'
  user_pass = 'Password0'
  user_home = '/home/iabd/'
  user_ssh_folder = '/home/iabd/.ssh/'
  user_bashrc = '/home/iabd/.bashrc'

  hadoop_download_url = 'https://dlcdn.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz'


  ##
  ## Set disk size manually
  ##
  if Vagrant.has_plugin?("vagrant-disksize") then
    config.disksize.size = DISK_SIZE
  end


  ##
  ## Ensure that disk size is fully used
  ##
  config.vm.provision "shell", inline: <<-SHELL
    echo -e "Fix\n" | parted ---pretend-input-tty /dev/sda print
    parted /dev/sda print free
    parted /dev/sda resizepart 3 100%
    pvresize /dev/sda3
    lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
    resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv

    df -h
  SHELL


  ## 
  ## Copying keys to temp folder
  ##
  config.vm.provision "file", source: "id_rsa", destination: '/tmp/ssh/'
  config.vm.provision "file", source: "id_rsa.pub", destination: '/tmp/ssh/'
  config.vm.provision "file", source: "authorized_keys", destination: '/tmp/ssh/'


  ##
  ## Prepare user and SSH Keys
  ##
  config.vm.provision "shell", inline: <<-SHELL
    echo 'Create user for VM...'
    useradd -m -p $(openssl passwd -1 "#{user_pass}") -s /bin/bash #{username}

    echo 'Create SSH Folder'
    sudo -u '#{username}' mkdir -p '#{user_ssh_folder}'

    echo 'Copying files...'
    mv /tmp/ssh/* '#{user_ssh_folder}'
    
    echo 'Setting permissions...'
    sudo -u '#{username}' chmod 700 '#{user_ssh_folder}'
    chown -R '#{username}': '#{user_ssh_folder}'
  SHELL


  ##
  ## Update and install apt packages...
  ##
  config.vm.provision "shell", inline: <<-SHELL
    echo 'Updating and installing dependencies...'
    apt-get update --fix-missing
    apt-get -y install openssh-server
    apt-get -y install default-jdk
  SHELL


  ##
  ## Check Java version...
  ##
  config.vm.provision "shell", inline: <<-SHELL
    echo 'Java Version Check ....'
    echo $(java -version)
    echo $(dirname $(dirname $(readlink -f $(which java))))
  SHELL


  ##
  ## Download Hadoop
  ##
  config.vm.provision "shell", inline: <<-SHELL
    echo 'Downloading Hadoop..'

    cd /opt
    curl '#{hadoop_download_url}' | tar zx
    mv hadoop* hadoop
    chown -R '#{username}': hadoop
  SHELL
  

  ##
  ## Setting .bashrc variables
  ##
  config.vm.provision "shell", inline: <<-SHELL
    echo 'Setting envs @ .bashrc'
    echo export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java)))) >> '#{user_bashrc}'
    echo 'export HADOOP_HOME=/opt/hadoop' >> '#{user_bashrc}'
    echo 'export HADOOP_INSTALL=$HADOOP_HOME' >> '#{user_bashrc}'
    echo 'export HADOOP_MAPRED_HOME=$HADOOP_HOME' >> '#{user_bashrc}'
    echo 'export HADOOP_COMMON_HOME=$HADOOP_HOME' >> '#{user_bashrc}'
    echo 'export HADOOP_HDFS_HOME=$HADOOP_HOME' >> '#{user_bashrc}'
    echo 'export HADOOP_YARN_HOME=$HADOOP_HOME' >> '#{user_bashrc}'
    echo 'export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native' >> '#{user_bashrc}'
    echo 'export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin' >> '#{user_bashrc}'
    echo 'export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"' >> '#{user_bashrc}'
  SHELL

  
  ##  
  ## Setting JAVA_HOME to HADOOP
  ##
  config.vm.provision "shell", inline: <<-SHELL
    echo 'Setting JAVA_HOME @ hadoop-env.sh'
    echo export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java)))) >> /opt/hadoop/etc/hadoop/hadoop-env.sh
  SHELL

  
  ## 
  ## Reload variables
  ##
  config.vm.provision "shell", inline: "source /home/iabd/.bashrc"


  ##
  ## Prepare data folder
  ##
  config.vm.provision "shell", inline: <<-SHELL
    echo 'Preparing data folder'
    mkdir -p /datos/{name,data}node
    chown -R '#{username}': /datos
  SHELL


  ##
  ## Copy config files
  ##
  ["core-site.xml", "hdfs-site.xml", "mapred-site.xml", "yarn-site.xml"].each do |file|
    config.vm.provision "file", 
      source: "./hadoop/#{file}",
      destination: "/tmp/hadoop/#{file}"

    config.vm.provision "shell", inline: "mv /tmp/hadoop/'#{file}' /opt/hadoop/etc/hadoop/'#{file}'"
  end


  ##
  ## Setting /etc/hosts
  ##
  config.vm.provision "shell", inline: <<-SHELL
    sed -i -E "s/(127.0.2.*)/#\\1/g" /etc/hosts
    echo '192.0.2.5   master' >> /etc/hosts
  SHELL
  
  (1..NODES).each do |i|  
    config.vm.provision "shell", inline: <<-SHELL
      sed -E "s/(127.0.2.*)/#\\1/g" /etc/hosts
      echo '192.0.2.#{i}0   nodo#{i}' >> /etc/hosts
    SHELL
  end


  ##
  ## Setting /opt/hadoop/etc/hadoop/workers
  ##
  (1..NODES).each do |i|  
    config.vm.provision "shell", inline: "echo 'nodo#{i}' >> /opt/hadoop/etc/hadoop/workers"
  end


  ##
  # Configure VirtualBox
  ##
  config.vm.provider "virtualbox" do |v|
    v.memory = MEMORY
    v.cpus = CPUS  
  end


  ##
  ## Set Virtual Box Guest Additions
  if Vagrant.has_plugin?("vagrant-vbguest") then
    config.vbguest.auto_update = false
  end


  ##
  ## Nodes
  ##
  (1..NODES).each do |i|  
    config.vm.define "nodo#{i}" do |node|
      node.vm.network "private_network", ip: "192.0.2.#{i}0"
      node.vm.hostname = "nodo#{i}"

      node.vm.provider "virtualbox" do |v|
        v.name = "SBD-Nodo#{i}"
      end
    end
  end


  ##
  ## Master
  ##
  config.vm.define "master" do |master|
    master.vm.hostname = "master"
    master.vm.network "private_network", ip: "192.0.2.5"
    
    master.vm.network "forwarded_port", guest: 8088, host: 8088
    master.vm.network "forwarded_port", guest: 9087, host: 9087

    master.vm.provider "virtualbox" do |v|
      v.name = "SBD-Master"
    end

    ## Initialize Hadoop FS
    master.vm.provision "shell", inline: <<-SHELL
      echo 'Setting HDFS Format'
      sudo -u '#{username}' /opt/hadoop/bin/hdfs namenode -format
    SHELL

    master.vm.provision "shell", 
      inline: "sudo -u '#{username}' /opt/hadoop/sbin/start-all.sh",
      run: "always"
    
  end

end