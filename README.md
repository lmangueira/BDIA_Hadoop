## Requirements

[![Vagrant](https://img.shields.io/badge/-Vagrant-1868F2?logo=Vagrant&style=for-the-badge&logoColor=white)](https://www.vagrantup.com/)
[![VirtualBox](https://img.shields.io/badge/-VirtualBox-183A61?logo=VirtualBox&style=for-the-badge&logoColor=white)](https://www.virtualbox.org/)

### Recommended Plugins for Vagrant

- vagrant-disksize : https://github.com/sprotheroe/vagrant-disksize
  
  ``vagrant plugin install vagrant-disksize``

- vagrant-vbguest : https://github.com/dotless-de/vagrant-vbguest

  ``vagrant plugin install vagrant-vbguest``

## Usage

Clone repo and ``vagrant up``

Will create key-pair and copy to each machine, so _sshing_ will be easier

Hadoop configuration files are in hadoop folder. Will be copied to each VM.

## Customization

|   Value   |  Default Value  |
|-----------|-----------------|
| MEMORY    |       4096      |
| CPU       |          2      |
| DISK_SIZE |      100GB      |
| NODES     |          3      |
