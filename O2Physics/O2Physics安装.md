
**此部分教程需要您：**
已完成ALICE账户的激活与虚拟服务的开启（[Getting started · ALICE Analysis Tutorial](https://alice-doc.github.io/alice-analysis-tutorial/start/)）（[Register with/to ALICE VO - JAliEn - ALICE Grid environment](https://jalien.docs.cern.ch/user/register_vo/)）、
已完成证书的下载（[Get a Grid certificate · ALICE Analysis Tutorial](https://alice-doc.github.io/alice-analysis-tutorial/start/cert.html)）、
已拥有可靠的vpn用于访问外网（使用clash进行代理）。


# 1.容器下载与安装

我们一般在容器内进行O2的安装以保持环境不会互相干扰，组内目前采用singularity容器。

（1）下载可用的.def文件（）
sudo singularity  build  --notest  --sandbox --fix-perms  name  alidockSingularity_o2p.def