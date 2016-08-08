chef-wowza cookbook for Wowza 4.5.0 on Ubuntu 16.04
===================================================
This cookbook automates the installation of the Wowza Media Server 4.4.1, including their installer, which contains a very long EULA, and 7 interactive prompts.

![chef-wowza](https://github.com/msergiy87/chef-wowza/blob/master/chef-wowza.jpg)

Requirements
------------
- Java jdk - openjdk will be just fine.
- Wowza license - this you will get when you sign up for a free trial, or you can request a developer license.
- Wowza binary - the Wowza binary (4.5.0) is now wgetted as part of this role.

Attributes
------------
```
default['wowza_file'] = "WowzaStreamingEngine-4.5.0-linux-x64-installer.run"
default['wowza_download_path'] = "http://www.wowza.com/downloads/WowzaStreamingEngine-4-5-0"
default['user_name'] = "admin"
default['password'] = "admin"
default['license_key'] = "insert key between quotations"
```

Distros tested
----------
Currently, this is only tested on Centos 6.7 as a client machine and Ubuntu 16.04 as a server machime.

Wait a minute, what the hell is Wowza?
------------
From the Wowza site:

_Wowza Streaming Engine™ is robust, customizable media server software that powers reliable streaming of high-quality video and audio to any device, anywhere._

_Wowza software is platform-agnostic, multi-format, and multi-screen. It takes in any video format, transcodes it once, and reliably delivers it in multiple formats and with the highest possible quality_

Usage
------------
##### 1) Install knife-solo
```shell
apt-get update
apt-get install git -y
curl -L https://www.opscode.com/chef/install.sh | sudo bash
/opt/chef/embedded/bin/gem install knife-solo --no-ri --no-rdoc
```

##### 2) Clone repository
```shell
mkdir ~/my_deploy_code
cd ~/my_deploy_code
knife solo init .
cd site-cookbooks/
git clone https://github.com/msergiy87/chef-wowza.git
```

##### 3) Prepare host
```shell
cd ~/my_deploy_code
knife solo prepare root@HOST -P 'PASS'
```

##### 4) Include `chef-wowza` in your node's `run_list`:
```shell
vim nodes/192.168.0.103.json
```
```json
{
  "run_list": [
    "recipe[chef-wowza]"
  ],
  "automatic": {
    "ipaddress": "192.168.0.103"
  }
}
```
##### 5) Add license key and change username and password
```shell
vim site-cookbooks/chef-wowza/attributes/default.rb

default['user_name'] = "admin"
default['password'] = "admin"
default['license_key'] = "insert key between quotations"
```

##### 6) Cook
```shell
knife solo cook root@HOST -P 'PASS'
```

##### 7) Add iptables rules on server
```shell
iptables -I INPUT -p tcp --dport 1935 -j ACCEPT
iptables -I INPUT -p tcp --dport 8088 -j ACCEPT
```

##### 8) Run web interface
```
http://192.168.0.103:8088/enginemanager
```

Wowza needs several user actions on the interactive prompt portion of the installer:
- an acknowledgement of acceptance of their terms
- an administrative user added for using the Wowza console
- a password for the administrative user
- confirmation of that password
- Wowza license key
- an acknowledgement of whether or not you want Wowza to start at boot

Those values can be edited in the template/script.exp.erb file.
