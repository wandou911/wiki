# CentOS安装HomeAssistant

## 安装 Python环境

* 安装软件集

```text
sudo yum install centos-release-scl
sudo yum-config-manager --enable centos-sclo-rh-testing
sudo yum install -y scl-utils
```

* 安装依赖

`sudo yum install gcc gcc-c++ systemd-devel`

* 安装Python3.6

`sudo yum install rh-python36`

* 开始使用软件集

`scl enable rh-python36 bash`

查看python版本

```text
$ python --version
Python 3.6.3
```

## 安装 Home Assistant

* 在当前目录创建虚拟环境:

`python3 -m venv homeassistant`

* 进入虚拟环境目录:

`cd homeassistant`

* 激活虚拟环境:

`source bin/activate`

* 安装 Home Assistant:

`python3 -m pip install homeassistant`

* 运行 Home Assistant:

`hass --open-ui`

* 访问web页面 `http://ipaddress:8123/`

### 参考链接

[Installing Home Assistant Core on CentOS/RHEL](https://community.home-assistant.io/t/installing-home-assistant-core-on-centos-rhel/199512)

[Installation in Python virtual environment](https://www.home-assistant.io/docs/installation/virtualenv/#step-4-set-up-the-virtualenv)

