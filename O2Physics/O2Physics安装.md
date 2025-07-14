
**此部分教程需要您：**
已完成ALICE账户的激活与虚拟服务的开启（[Getting started · ALICE Analysis Tutorial](https://alice-doc.github.io/alice-analysis-tutorial/start/)）（[Register with/to ALICE VO - JAliEn - ALICE Grid environment](https://jalien.docs.cern.ch/user/register_vo/)）、

已完成证书的下载以及浏览器证书安装部分（[Get a Grid certificate · ALICE Analysis Tutorial](https://alice-doc.github.io/alice-analysis-tutorial/start/cert.html)）、

已拥有可靠的vpn用于访问外网（使用clash进行代理）。

# 1.网络代理设置


（1）将clash主界面各选项按钮设置为如图所示：
![[Pasted image 20250713154113.png]]
注意这里的Port端口不要自己更改，记住自己是多少就行，这里例子为7890。

（2）查询自己电脑的IPv4地址，例如：`10.162.253.177`。

（3）打开`~/.bashrc`文件，添加如下内容，完成后终端运行`source ~/.bashrc`激活更改。
```
export http_proxy="http://10.162.253.177:7890"
export https_proxy="http://10.162.253.177:7890"
```

# 2.容器下载与安装


我们一般在容器内进行O2的安装以保持环境不会互相干扰，组内目前采用singularity容器。

（1）下载可用的.def文件（[kegang02/Tutroial: 关于O2Physics，ROOT，linux的一些常见问题与指南](https://github.com/kegang02/Tutroial)）。

（2）打开.def文件，找到如下片段，将其中的/home/shiqi改为自己的用户路径。
```
# Create a custom .bashrc
    echo 'export PATH=/usr/local/bin:$PATH' >> /root/.bashrc
    echo 'alias ll="ls -la"' >> /root/.bashrc
    echo 'alias o2p="alienv enter O2Physics/latest"' >> /root/.bashrc	
    echo 'alias emacs="emacs -nw"' >> /root/.bashrc

    mkdir -p /WorkSpace
    mkdir -p /WorkSpace2
    mkdir -p /home/shiqi

%environment
    # Ensure the PATH is set correctly
    export PATH=/usr/local/bin:$PATH
    eval "$(alienv shell-helper)"
    export HOME=/home/shiqi
    if [ -f /home/shiqi/.bashrc ]; then
        source /home/shiqi/.bashrc
    fi
```

（3）运行命令进行容器搭建
```
sudo singularity  build  --notest  --sandbox --fix-perms /dockerpath/name /filepath/dockername.def
```
其中name为你给容器取的名字，dockername.def为下载的def文件，路径需自己注意修改。

（4）打开`~/.bashrc`文件，添加：
```
alias start='sudo singularity shell --bind /WorkSpace --bind /home/username --bind /tmp --writable /dockerpath/name'
```
其中start为快捷启动指令（可自定义），--bind是为将对应文件路径挂载进容器中以便读取，按需添加。
完成后终端运行source ~/.bashrc激活更改，往后运行start便可进入容器环境。

# 3.O2Physics安装


**注意：以下所有步骤均在容器环境内进行。**

（1）证书的安装。
将下载的myCertificate.p12文件放置于用户路径下。

创建~/.globus文件夹。

运行如下命令（会要求输入之前创建证书时设置的密码）：
```
openssl pkcs12 -clcerts -nokeys -in ~/Downloads/myCertificate.p12 -out ~/.globus/usercert.pem
```

继续运行：
```
openssl pkcs12 -nocerts -in ~/Downloads/myCertificate.p12 -out ~/.globus/userkey.pem
```
当提示:
```
Enter Import Password:
```
请输入之前创建证书时设置的密码. 下一个问题为:
```
Enter PEM pass phrase:
```
这里会要求你输入一个新的密码用于保护私钥并需要二次确认。

最后：
```
chmod 0400 ~/.globus/userkey.pem
```

至此完成证书的安装，激活需等到O2Physics安装完后。

（2）aliBuild安装
依次逐行运行：
```
sudo apt update -y

sudo apt install -y curl libcurl4-gnutls-dev build-essential gfortran libmysqlclient-dev xorg-dev libglu1-mesa-dev libfftw3-dev libxml2-dev git unzip autoconf automake autopoint texinfo gettext libtool libtool-bin pkg-config bison flex libperl-dev libbz2-dev swig liblzma-dev libnanomsg-dev rsync lsb-release environment-modules libglfw3-dev libtbb-dev python3-dev python3-venv python3-pip graphviz libncurses-dev software-properties-common gtk-doc-tools

sudo add-apt-repository ppa:alisw/ppa  

sudo apt update

sudo apt install python3-alibuild
```

安装完成后打开~/.bashrc文件添加如下内容，并source激活。
```
export ALIBUILD_WORK_DIR="/WorkSpace/shiqi/alice/sw"
eval "$(alienv shell-helper)"
```
此处请自行选择合适的工作路径。

（3）O2Physics安装
cd至此前确定的工作目录下（这里为/WorkSpace/shiqi），创建alice文件夹并进入。
```
mkdir -p alice
cd alice
```

运行：
```
aliBuild init O2Physics@master
```
注意：如果这一步卡住较长时间（超过三分钟），首先尝试`source ~/.bashrc`后再次运行（因为容器内不手动source可能配置会不生效），若还是不行则需检查本文前面的网络代理部分是否正确设置，IP地址是否变更等。

上一步完成后，运行：
```
aliBuild build O2Physics
```
完成最终O2Physics的安装，该过程耗时较长（一天左右），且须注意全程网络稳定，电脑请勿进入睡眠状态。若安装过程中出现git相关报错一般为网络波动所导致，安装进度不会丢失，重新运行`aliBuild build O2Physics`即可。

# 4.最终步骤


安装完成后运行
```
alienv enter O2Physics/latest
```
进入O2Physics环境（需先在容器环境中）。

分别运行
```
alien-token-destroy
alien-token-init YOUR_ALIEN_USERNAME
```
YOUR_ALIEN_USERNAME指你在cern中的用户名。至此证书部分亦完结。