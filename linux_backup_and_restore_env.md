[TOC]

### 记录一次linux从发现异常到直接重装整机再安装环境的过程

#### 起因

linux莫名其妙崩溃，型号是dell 7920，具体表现是界面卡死，键盘卡死，皆无响应，即`ctrl+alt+f` 系列不管用，并且安全重启也不起作用，在强制关机后，dell主机电源开关显示灯闪烁情况是2黄4白，屏幕一点都不亮，怀疑操作系统都没打开；另外还有就是往往需要重启n次才会随机进入系统，但是进入系统后又会卡住。

#### 问题定位

首先经过简单的排查后推测是内核错误，遂更新过内核，不起作用，但是重启后发现内核并没有完全更新，于是不确定这条路是否正确，其二是推测温度太高，导致硬盘宕机，系统卡死，后续将空调调低也没有改变现状，遂排除。

通过打售后电话后，了解到可能是内存条氧化问题，需要拔除内存条进行擦拭铜芯，以及上面的灰，为了排除是哪个内存条出了问题，于是还需逐一擦拭内存条并且开机测试，所幸，在擦完第一根内存条后就成功解决问题，系统可以正常进入。于是遂打算先将所有东西备份再做处理

#### 重装系统

没有遇到什么问题，下载镜像，这里选择的是20版本，因为22版本实在太高，官方并没有进行测试，保险选择20，不选18是因为原系统就是18，存在内核问题，需要升级，综合考虑20最佳，选择rufus烧录，然后关机，再开机狂摁F12进入BIOS，选择U盘启动，正常安装

#### 重装之后bug

不知道是dell的原因，还是原来磁盘的另一个ubuntu18引导没有删干净的原因，在开机的时候会进入grub引导程序，如果在这里不操作启动默认的ubuntu版本的话，在这里会浪费接近10s的时间，于是想删去ubuntu18的引导程序，并且让grub直接选择最新安装的ubuntu。节省开机时间

但是有点失败，没有找到合适的解决方案，只是成功将等待10s变为等待1s

方案为

#### 新建用户

一开始bug在一开始找了一个教程，似乎是很早之前的帖子被新拿出来了，整个过程极其复杂，需要新建用户，修改权限，新建用户home目录，绑定用户目录，赋权等一系列操作，还不包含ssh登录等一系列配置

但是后面找了几个操作的流程，如下：

> sudo adduser csdn # 次数的用户名需要符合一般命名规范，开始不能以数字，不包含大写字母，之后会让直接输入密码
>
>   **useradd**：需要使用参数选项指定上述基本设置，如果不使用任何参数，则创建的用户无密码、无主目录、没有指定shell版本
>
>  **adduser**： 会自动为创建的用户指定主目录、系统shell版本，会在创建时输入用户密码。因此建议使用这个命令。
>
> | --home | 指定创建主目录的路径，默认是在/home目录下创建用户名同名的目录，这里可以指定；如果主目录同名目录存在，则不再创建，仅在登录时进入主目录。 |
> | ------ | ------------------------------------------------------------ |
> |        |                                                              |

添加sudo 权限

> su
>
> chmod u+w /etc/sudoers
>
> vim /etc/sudoers 
>
> csdn    ALL=(ALL:ALL) ALL  # 在root ALL=(ALL:ALL) ALL 后添加

删除用户

> deluser
>
> root@user:~# userdel -rf 20testuser #一次性删除用户以及主目录
>
> root@user:~#userdel 20testuser 不删除主目录

#### 解决ssh远程登录问题

#### 安装cuda环境

[refer](https://segmentfault.com/a/1190000040322236)

首先高版本的驱动兼容低版本的cuda，所以不用纠结驱动提示cuda和实际安装cuda是否一致

##### 驱动

1. 下载驱动
   [website](https://link.segmentfault.com/?enc=O1qf%2BsP1st%2BtXy5KX%2Fh0YQ%3D%3D.hZ9h%2FhEttwki%2F7KNeD6Dj%2BIhZYlCrEoKeJKSiYB3nJbiE%2FsiMbKAYmBOuJaUBnALGYpH%2F41WON%2BQZMwwrHXlLQKR%2Fv96mWh6uqtltQggHSj4uRLXZG0FxM4XUzPHnLMAOgaLwNhr%2FIlPChB4TPRXhw%3D%3D)

2. 禁用nouveau

   > sudo gedit /etc/modprobe.d/blacklist.conf 
   >
   > 在blacklist.conf文件末尾加上这两行，并保存
   >
   > blacklist nouveau
   >
   > sudo update-initramfs -u  //应用更改 重启电脑，验证是否禁用nouveau，这一条是用来禁用nouveau驱动，之后也不需要改回来。
   >
   > lsmod | grep nouveau

3. 安装NVIDIA驱动

   > 开terminal卸载旧版本NVIDIA驱动：
   > sudo apt-get remove --purge nvidia*
   > sudo chmod  a+x NVIDIA-Linux-x86_64-460.84.run //对应自己下载的驱动名称 可执行权力
   >
   > sudo ./NVIDIA-Linux-x86_64-460.84.run -no-x-check -no-nouveau-check -no-opengl-files
   > // 注意：后面三个选项的前面都是：减号“-”
   >
   > ```
   > The distribution-provided pre-install script failed! Are you sure you want to continue? 选择 yes 继续。
   > 
   > 
   > Would you like to register the kernel module souces with DKMS? This will allow DKMS to automatically build a new module, if you install a different kernel later?  选择 No 继续。
   > 
   > 
   > 问题没记住，选项是：install without signing
   > 
   > 
   > 问题大概是：Nvidia's 32-bit compatibility libraries? 选择 No 继续。
   > 
   > 
   > Would you like to run the nvidia-xconfigutility to automatically update your x configuration so that the NVIDIA x driver will be used when you restart x? Any pre-existing x confile will be backed up.  选择 Yes
   > ```

   ##### cuda

   [website](https://link.segmentfault.com/?enc=B%2F9NC6w9mYt2HwsLdkKMCw%3D%3D.VU9GQWsu5lBrEdP2Q3rUNlja5CeUhiqgkkEVkCGIZzptFZ0J0p%2BnCH2HB91QsrP%2BPUpl8bDc49iBcfUHMiTkFsuTOGxSR%2BlnVvCCIiyICkMT8yyp9wu0T63rEvh%2F0XQS)

   1. 下载

      最后一项`Installer Type`建议选择`runfile [local]`，因为命令行少，更方便。

   2. > sudo sh cuda_11.4.0_470.42.01_linux.run
      > 输入accept
      > 回车这个地方不要下载Driver，因为之前已经安装完了，一定要选择CUDA Toolkit 10.2。
      >
      > > 取消选择的方法是：光标停留在`Driver`那一行上，然后回车，**使`[]`里的`X`消失**
      >
      > 选择Install回车,
      >
      > 检查cat /usr/local/cuda/version.txt
      >
      > 如果显示没有这个文件，就到该文件夹下去看看有没有一个`version.txt`，里面如果有`version.json`，且json中有版本信息，也可以

   3. 环境变量设置

      ```shell
      gedit ~/.bashrc
      
      加入如下环境变量(注意修改为自己的路径)：
      export PATH=/usr/local/cuda-11.4/bin\${PATH:+:\${PATH}}
      export LD_LIBRARY_PATH=/usr/local/cuda-11.4/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
      
      保存退出，终端运行：source ~/.bashrc
      
      测试:nvcc -V
      ```
   
   ##### cudnn
   
   选择合适版本
   第一种：Debian File形式的安装
   
   1. 选择ubuntu20的这个三文件下载到本地，
   
   2. 依次执行
   
      > sudo dpkg -i libcudnn8_8.2.2.26-1+cuda11.4_amd64.deb
      > sudo dpkg -i libcudnn8-dev_8.2.2.26-1+cuda11.4_amd64.deb
      > sudo dpkg -i libcudnn8-samples_8.2.2.26-1+cuda11.4_amd64.deb
   
   3. 验证
   
      当选择Debian File进行安装时会在`/usr/src/cudnn_samples_v8`有一些cudnn的例子
   
      在任意目录下展开终端，运行以下命令，通过编译`mnistCUDNN sample`进行验证
   
      > cp -r /usr/src/cudnn_samples_v8/ $HOME
      > cd  $HOME/cudnn_samples_v8/mnistCUDNN
      > make clean && make
      > ./mnistCUDNN
   
      **如果在执行sudo make”时报以下编译错误 ：fatal error: FreeImage.h**
   
      则执行：`sudo apt-get install libfreeimage3 libfreeimage-dev`，然后重新验证。通过有test past
   
   第二种 第二种：Tar File形式的安装
   
   1. 选择cudnn lib for liunx （x86)
   
   2. 解压 > tar -xzvf cudnn-11.4-linux-x64-v8.2.2.26.tgz
   
   3. > sudo cp cuda/include/cudnn.h /usr/local/cuda/include
      > sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
      > sudo chmod a+r /usr/local/cuda/include/cudnn.h 
      > sudo chmod a+r /usr/local/cuda/lib64/libcudnn*
   
      完成。解压操作会在当前目录生成一个`cuda`文件夹，删除即可。

##### **多个用户同个cuda**

本人作为超级用户安装完cuda和cudnn，希望其他用户也能正常调用我的程序。
只需要打开我的目录下.bashrc文件，将我的cuda的环境变量拷贝到其他用户的.bashrc文件下：
export PATH="/home/zhongjia/cuda9.0/bin:\$PATH"
export LD_LIBRARY_PATH="/home/zhongjia/cuda9.0/lib64:$LD_LIBRARY_PATH"
同时需要修改我的相应目录对其他用户有可读取的权限，所以chmod 755 "相应目录下文件名称“，具体可参考
https://www.cnblogs.com/shangzekai/p/5822907.html修改文件权限。
修改完成后，其他用户就可以读和执行相应路径下的程序。

##### **Unable to find the development tool cc**

sudo apt install build-essential

##### 设置向日葵开机自启动

```
1. gnome-session-properties
2. 点击add
3. 使用dpkg -L命令查看应用程序安装路径

4. 将第3步搜索结果的最后一条写入如下地方，Name自己定义


```

**配置Anaconda**

设置 Anaconda 的环境变量，打开 `/etc/profile` 文件添加：

```shell
# Anaconda environment
export ANACONDA_HOME=/usr/local/anaconda3
export PATH=$ANACONDA_HOME/bin:$PATH
# 刷新
source /etc/profile

```

或者

```shell
sudo vim /etc/profile
export PATH=/your_install_path/anaconda3/bin:$PATH # your_install_path为你安装anaconda3的路径，由上述步骤可知我的anaconda安装路径为/home/student/anaconda3，
export PATH=/home/student/anaconda3/bin:$PATH

# 刷新
source /etc/profile
source /home/student/anaconda3/etc/profile.d/conda.sh

```



##### **共用cuda**

在自己的当前环境/usr/local/anaconda3/bin/conda init bash

source .bashrc

##### **不进入grub界面，或者减少选择时间**

>  sudo gedit /etc/default/grub

find the line that says GRUB_HIDDEN_TIMEOUT=0 or GRUB_TIMEOUT_STYLE=hidden put # at the start of this line to comment it out

#GRUB_HIDDEN_TIMEOUT=0
or

#GRUB_TIMEOUT_STYLE=hidden
and make sure GRUB_TIMEOUT=10 or some other number bigger than zero. When done exit nano saving changes and run

>  sudo update-grub
> 

##### **配置jupyter虚拟环境并设置密码。**

让jupyter可以包含多个虚拟环境

> ```text
> conda install nb_conda # 在目标环境中安装，重启
> ```

设置密码

> 1. jupyter notebook --generate-config
> 2. 在生成的.jupyter\jupyter_notebook_config.py中  搜索 NotebookApp.allow_password_change，改为：NotebookApp.allow_password_change=False ，记得去掉注释的#
> 3. jupyter notebook password
> 4. 在生成的\.jupyter\jupyter_notebook_config.json 中拿到哈希码
> 5. 在第一个jupyter_notebook_config.py配置文件中找到“c.NotebookApp.password“，等于，刚生成的那个密码sha1，效果如下：去掉前面的”#
> 6. c.NotebookApp.password = u'码'
> 7. 重启
>
> 第三步骤可以换成在ipython中
>
> ```
> from notebook.auth import passwd
> passwd()
> ```

#

##### **/usr/bin/xauth: file /root/.Xauthority does not exist**

这个不算错误，重新登录即可

##### **文件后带* 或者文件夹是一种被包裹的选中的状态**

表示这个文件是可执行文件

修改文件权限

> chmod (-R) 755 xxx

修改文件归属

> chown (-R) owner:owner_group xxx

##### **pycharm 添加快捷方式**

> tools - create desktop entry

##### **DL framework 对应cuda关系**

tensorflow2.8 对应cuda11.4

torch 1.11或者1.10对应cuda11.3 类似11.4

##### **ubuntu 弹出硬盘**

 首先使用umount解除所有挂载

> sudo udisksctl power-off -b /dev/sdb

##### **ubuntu 18.04.5 LTS 安装 NVIDIA 显卡驱动时报错：An NVIDIA kernel module ‘nvidia-drm‘ appears to already be load**

1. 通过另一台主机的终端工具 ssh 登录 Ubuntu 系统，依次执行如下两条命令（设置系统默认进入终端命令模式，然后重启系统）

> sudo systemctl set-default multi-user.target
> sudo reboot 0

1. 待系统重启后，通过其他主机的终端工具 ssh 登录 Ubuntu 系统，依次执行如下三条命令，卸载已安装的 NVIDIA 驱动后重启

> sudo apt-get purge nvidia*
> sudo apt-get autoremove
> sudo reboot

3. 待系统重启后，通过其他主机的终端工具 ssh 登录 Ubuntu 系统，执行如下两条命令（先进入 NVIDIA 驱动安装文件所在的目录，再安装驱动

> cd NVIDIA驱动安装文件所在的目录
> sudo sh ./NVIDIA驱动安装文件.run

4. 等待 NVIDIA 驱动安装完成并测试显卡正常识别和运行后，再在终端执行如下两条命令（设置系统默认进入图形化界面模式，重启系统）

> sudo systemctl set-default graphical.target
> sudo reboot 0

**ubuntu 禁用自带显卡驱动**

```shell
# 1. 在终端中通过 nano 创建 blacklist-nouveau.conf 文件，终端命令如下：
nano /etc/modprobe.d/blacklist-nouveau.conf
# 2. 在该文件中添加如下内容：
blacklist nouveau
options nouveau modeset=0
# 3. 按 Ctrl + X，再按 Y，回车保存该文件
# 4. 重新生成 kernel initramfs，终端命令如下：
update-initramfs -u
# 5. 等待 kernel initramfs 重新生成完毕后，重启系统，终端命令如下：
reboot
# 6. 进入系统中即可正常安装 NVIDIA 驱动

```

##### **CondaEnvException: Pip failed**

在-pip : 前面加上`-pip`

##### **vim 替换**

> ```undefined
> :{作用范围}s/{目标}/{替换}/{替换标志}
> ```

> ```ruby
> :%s/foo/bar/g # 全局 替换所有
> ```

[refer](https://www.jianshu.com/p/b3b2f04c1897)

##### http: //cn.archive.ubuntu.com/ubuntu/dists/focal/InRelease 连接失败 IP: 91.189.91.38 80

1. 这是因为[ubuntu](https://so.csdn.net/so/search?q=ubuntu&spm=1001.2101.3001.7020)的服务器在国外，使用国内的软件源在下载时更新受到限制，将服务地址修改成国内的地址。

输入：以下命令找到软件源文件

>  cd /etc/apt

2.备份

输入这个命令进行备份\

>  sudo tar -zcvf sources.list.tar.gz sources.list\

输入以下命令：sudo vim sources.list

将地址cn.archive.ubuntu.com 更改为 mirrors.aliyun.com

3.修改源文件地址

输入以下命令：sudo vim sources.list

将地址cn.archive.ubuntu.com 更改为 mirrors.aliyun.com

4. 保存退出 sudo apt-get update

##### **下载段错误**

解决，由于文件太大引起的中断

> wget -c url

##### **shell 没有颜色**

在`~/.bashrc`中其实有了颜色的设置，查看代码可以发现

最下面有一个`PS1`的赋值，它就是控制颜色的设置，当终端是某种类型或者满足某些特定条件的时候，它就会显示颜色。

这里可以看到有个`force_color_prompt`变量默认被注释了，而如果这个变量的值为`yes`的时候，下面的`color_prompt`就也会是`yes`，然后的颜色设置也就会开启。

所以可以手动把这个注释去掉，使`force_color_prompt`的值等于`yes`，然后`source ~/.bashrc`就能看到效果了：

##### **解决linux系统 不在sudoers 文件中，此事将被报告**

[refer](https://zhuanlan.zhihu.com/p/143388819)

> 在`root ALL=(ALL) ALL` 的下一行添加代码：`86god ALL=(ALL) ALL`

> chmod u+w /etc/sudoers
>
> "vim /etc/sudoers"
>
> 在`root ALL=(ALL) ALL` 的下一行添加代码：`86god ALL=(ALL) ALL`
>
> chmod 440 /etc/sudoers