---
title: Vagrant必知必会
subtitle: 10分钟Vagrant入门
description: 零基础vagrant入门,vagrant中文教程,vagrant例子,vagrant基础知识
layout: default
date: 2018-11-15
category: 工具
---

### 一、初识Vagrant

[Vagrant](https://www.vagrantup.com)是指[HashiCorp](https://www.hashicorp.com/)开源的开箱即用，快速配置开发环境的命令行工具。官宣Sologan: Development Environments Made Easy。旨在一键配置开发环境，是团队成员间同步开发环境的绝佳助手，比如：前端开发可以凭着与服务端共享的*Vagrantfile*一键在本地配置出后端所有的服务，后续同步版本也非常方便，前端不需要关心/掌握服务端的任何知识。

Vagrant可以让你用简单的命令行在一分钟内就完成：

* 创建一个你所指定的操作系统(Ubuntu/CentOS/etc…)虚拟机（以下简称VM:*virtual machines* ）。
* 配置VM的物理属性（比如*config.vm.provider*中配置内存，CPU核数)。
* 配置VM的网络配置，让你的宿主机器/同一网络的其它机器都能访问。
* 配置宿主与VM的同步文件夹。
* 设置VM的Hostname。
* 一键配置VM的SSH。

Vagrant内部依赖已有成熟的VM技术(VritualBox/VMware/etc)。让vagrant结构简单但功能强大。

![vgrant_workflow](https://user-images.githubusercontent.com/3116225/48494500-f9ccee00-e868-11e8-885f-7edd43be1117.jpg)
HashiCorp官方提供主流的操作系统的各种版本镜像(在vagrant中都称为Box)，[可供下载](https://app.vagrantup.com/boxes/search)。丰富的boxes镜像也是Vagrant近几年迅速在各大公司内风靡的原因之一。

| 开发人员  | 第一个Release版本 | 开发语言 | 系统要求                     |
| --------- | ----------------- | -------- | ---------------------------- |
| HashiCorp | 2010年            | Ruby     | Linux/FreeBSD/OS X/Microsoft |

总之，Vagrant操作简单但功能强大，一键就可以创建出所需沙箱(sandbox)环境。在正式开始前，你需要花几分钟(主要是下载耗时)在官网上并下载安装[virtualbox](https://www.virtualbox.org/wiki/Downloads)和[vagrant](https://www.vagrantup.com/docs/installation/)。

### 二、基本流程(BasicFlow)

Vagrant为了维护VM的三个状态（初始化，运行中，停止），使用的关键命令有6个命令。频繁使用的只有2个命令。接下来是一个从零构建Nginx HTTP服务的完整例子，一步步照着操作，就可以理解Vagrant的基本流程。

![vagrant_command](https://user-images.githubusercontent.com/3116225/48494499-f9345780-e868-11e8-963e-8128d91cb6c1.jpg)
#### 2.1 项目设置

```shell
mkdir vagrant_paradise
cd vagrant_paradise
vagrant init
```

`vagrant init` 初始化项目，会在当前的目录下生成一个[Vagrantfile](https://www.vagrantup.com/docs/vagrantfile/)的文件，它关键的两个作用：

* 确定项目的根目录。很多的配置选项都与这个根目录有关。
* 指定VM的具体系统版本，想要预装的软件及如何与这个系统交互(SSH)。

团队间需要保持相同环境，只需要用版本管理工具同步此文件(不需要加入根目录下的.vagrant文件夹)。

#### 2.2 Boxes

vagrant官方提供了各种操作系统版本的[镜像文件box](https://vagrantcloud.com/boxes/search)。比如安装CentOS7操作系统。

```shell
vagrant box add centos/7
```

默认会从[HashiCorp's Vagrant Cloud box](https://vagrantcloud.com/boxes/search)使用HTTPS下载，你可以在这上面找到各种版本的box。

如果这些定制都不满足你的需求(这种情况很少出现)，你也可以指定URL下载自制的box。

```
vagrant box add centos/7 https://github.com/CommanderK5/packer-centos-template/releases/download/0.7.2/vagrant-centos-7.2.box
```

boxes下载后是在全局保存`~/.vagrant.d/`，每个项目只是clone此基础镜像，无法改变它。

下载镜像需要几分钟（时间取决于你的网络状态），完成时可见*Vagrantfile*中的配置会变成：

```ruby
Vagrant.configure("2") do |config|
  
  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centOS/7"
```

#### 2.3 启动并SSH进入

启动操作系统：

```shell
vagrant up
```

此命令可以在1分钟内启动一个配置centos7系统的VM，vagrant默认是命令行式的，并没有像直接很virtualbox启动时对应的UI。正常启动后网络和SSH都是已设置好的状态。

💡: 如何判断VM正常启动呢？

此命令返回提示和`vagrant status`可以判断出是否卡在了致命的错误上，但是最终是否正常应该看VM能不能正常被SSH进入。如果不能，最直接的方法就是：打开virtualbox运行界面，使用它直接启动镜像，这样就能看到系统提示有什么。比如：我在宿主机器是windows10中就卡在配置ssh上。

```shell
default: SSH address: 127.0.0.1:2222
default: SSH username: vagrant
default: SSH auth method: private key
```

然后用virtualbox启动时发现报错：

```shell
this kernel requires an x86-64 cpu but only detected an i686 cpu
```

明明是64位机器，结果却只能使用32位的配置，所以直接打开vritualbox界面右键设置，在常规/基本中Linux中吸有32位机型选择，没有32位。这是我机器上默认的BIOS的*virtualization*默认为disabled。需要重启windows然后启动时进入BIOS界面(不断按F10)，*Advanced / VT-x virtualization*设置为enabled（前提：Virtualization Extension插件已安装)。

修复后，通过看是否可以ssh来检查VM启动是否正常。

```shell
vagrant ssh
```

此命令相当于你使用`ssh vagrant@127.0.0.1 -p 2222`直接进入到VM中。你可以使用`Ctrl+D`退出ssh会话。

你可以使用`vagrant ssh-config`查看当前的ssh配置情况。默认使用的用户是vagrant用户，如果需要改成root用户配置如下：

```ruby
Vagrant.configure("2") do |config|  
  config.ssh.username = "root"
  config.ssh.private_key_path="/xyz/.vagrant/machines/default/virtualbox/private_key" 
```

最好使用openssl重新生成RSA Key，默认的是不安全的。其它高级选项可见[文档](https://www.vagrantup.com/docs/vagrantfile/ssh_settings.html)。

#### 2.4 停止

你可以通过`halt`停止VM：

```shell
vagrant halt
```

vagrant首先会使用guest用户执行`shutdown`尝试优雅地关闭VM。如果关闭失败，就会直接关闭VM的电源（`vagrant halt —force`也可直接关闭电源）。

#### 2.5 销毁

你可以通过`destory`彻底销毁VM：

```shell
vagrant destory
```

它会删除与除了共享文件夹外的所有资源，他不会释放box(全局的)，如果你需要删除box可以使用

```shell
vagrant box remove centos/7
```

### 三、同步文件夹(SyncedFolds)

默认的同步文件夹是宿主的项目根目录(*Vagrantfile*所在目录)与VM的`/vagrant/`目录，可以看到两个目录下的*Vagrantfile*是同步的。这个同步的文件夹就是宿主与VM的桥梁，当VM IO负担很重时，同步大量文件夹会对IO影响很大，所以一般只把代码都放在此文件夹中，那些频繁改动的大文件（数据库运行产生的文件）直接放在VM内部。

🙇‍♂️如何修改/更新/禁止此同步文件夹？

直接修改*Vagrantfile*中的*config.vm.synced_folder*，然后执行`vagrant reload`。

```ruby
Vagrant.configure("2") do |config|
  # other config here
  config.vm.synced_folder "src/", "/srv/website"
end
```

* 第一个参数为宿主机器的目录，如果使用相对路径，相对的是项目的根目录。
* 第二个参数是VM中的路径，必须是绝对路径，如果不存在，就会递归创建。共享文件夹默认的*owner/group*权限是和ssh的用户一致，此文件夹的父文件夹*owner/group*被设置root。

如果你想修改上面的默认权限或禁止使用共享文件夹，[👉查看官方文档](https://www.vagrantup.com/docs/synced-folders/basic_usage.html#options)。

💡: 最好不要把共享文件夹指定为符号链接。大多数情况下可以工作，少数情况下会莫名出错。

### 四、初始化脚本(Provisioning)

使用`vagrant box add`下载的box只是别人打包过后的基础镜像，我们需要在基础镜像上根据个性化需求再次初始化。比如需要安装Nginx，以前我们会直接通过ssh后使用命令行安装它，不便之处是团队中每个成员都必须按照各种指引手动去安装软件。vagrant把这些前期准备步骤称为provision，可以通过`vagrant provision`或`vagrant reload —provision`时来完成。

1. 在根目录创建启动脚本`bootstrap.sh`。																	

   ```shell
   #!/usr/bin/env bash
   yum install -y nginx
   ```

   💡: 脚本中不允许出现与需要用户输入确认(交互)的行为，所以`yum`加了一个`-y`选项。

2. 在*Vagrantfile*文件中指定脚本路径。

   ```ruby
   Vagrant.configure("2") do |config|
     config.vm.box = "centos/7"
     config.vm.provision :shell, path: "bootstrap.sh"
   end
   ```

3. 执行provision。

   ``` shell
   vagrant provision
   ```

   💡: `vagrant up`并不会每次都执行provision，执行的时机是provision从来没有执行过，或你明确告诉它。

4. 在VM中验证nginx是否可用。

   ```shell
   curl 127.0.0.1
   ```

### 五、网络设置(Network)

在上节中只是在VM中使用`curl`验证，在宿主机器上是不行的，因为我们并没有把VM的80端口映射到宿主机器上。那么我们现在把宿主机器的8080端口转发到VM的80端口，以便能在VM外部访问。

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.provision :shell, path: "bootstrap.sh"
  config.vm.network :forwarded_port, guest: 80, host: 8080
end
```

此时使用宿主机器的浏览器打开`http://127.0.0.1:8080`就可以看到Nginx的欢迎界面。
PS:如果还是无法访问，需要把你的防火墙关闭(VM的root密码默认为*vagrant*)。

```shell
sudo systemctl stop firewalld
```

💡: 如果需要转发多个端口，可以写多行。

```ruby
config.vm.network :forwarded_port, guest: 80, host: 8080
config.vm.network :forwarded_port, guest: 81, host: 8081
```

网络设置中`config.vm.network`默认为*public_network*，如果你需要设置为*private_network*或想搞清楚这两者的具体区别，可以[👉这些高级设置](https://www.vagrantup.com/docs/networking/private_network.html)。

### 六、清理(Teardown)

Vagrant停止有3种方式(suspend,halt,destroy)，退出时清理的程度级级加深。

``` shell
vagrant suspend
```

挂起（supending)，会保存当前运行的所有状态的快照（snapshot），当你再次使用`vagrant up`启动时，它会还原到上次挂起时的状态。这样的好处是启动和关闭都非常快(5~10秒)，缺点就是VM会大量占用你的磁盘空间。

```shell
vagrant halt
```

停止(halting)，首先使用guest用户尝试使用`shutdown`关闭，如果无法关闭，就直接关闭VM的电源。它的好处是不在硬盘上保存当前内存信息，占用空间较小，缺点是这种冷启动方式会比suspend慢，并且VM冷启动成功后需手动再启动你的服务。

```shell
vagrant destroy
```

销毁(destroying)，它会移除除同步目录外VM其它痕迹。好处是停止不占硬盘空间。缺点是当再次`vagrant up`，会重新进行provision。

### 七、参考资料(Reference)

* [Vagrant官方网站](https://www.vagrantup.com)。
* [Vagrantfile所有配置参数详解](https://www.vagrantup.com/docs/vagrantfile/)。
* [Vagrant Boxes官方搜索](https://app.vagrantup.com/boxes/search)。
* [Vagrant Boxes非官方搜索](http://www.vagrantbox.es/)。
