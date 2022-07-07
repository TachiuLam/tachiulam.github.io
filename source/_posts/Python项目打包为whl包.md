---
title: Python项目打包为whl包
index_img: /img/py1.png
banner_img: /img/py3.png
tags:
  - Python
categories:
  - Python
date: 2022-07-07 23:18:36
---
### Python项目打包为whl包
####  1、进入要打包的项目根目录下，创建setup.py：
```python
from setuptools import setup, find_packages
 
setup(
    name = "driver",
    version = "0.1",
    packages = find_packages(),
    #目标文件
    py_modeles = 'driver_classification.py',
    #excel文件
    data_file = 'driver_model_data.xlsx',
    include_package_data = True,
    #包含所有.xlsx文件
    package_data = {
)
```
##### setup函数各参数详解：
```
--name              包名称
  --version (-V)      包版本
  --author            程序的作者
  --author_email      程序的作者的邮箱地址
  --maintainer        维护者
  --maintainer_email  维护者的邮箱地址
  --url               程序的官网地址
  --license           程序的授权信息
  --description       程序的简单描述
  --long_description  程序的详细描述
  --platforms         程序适用的软件平台列表
  --classifiers       程序的所属分类列表
  --keywords          程序的关键字列表
  --packages  需要打包的目录列表
  --py_modules  需要打包的python文件列表
  --download_url  程序的下载地址
  --cmdclass  
  --data_files  打包时需要打包的数据文件，如图片，配置文件等
  --scripts  安装时需要执行的脚步列表
```
#### 2、打包文件
```python
# 打包为egg文件
python setup.py bdist_egg
# 打包为whl文件
python setup.py bdist_wheel
```
##### 运行后会在当前目录多出3个文件夹：build、dist、driver.egg-info

##### 打包好后的whl文件在dist文件夹内，进入dist文件夹中安装whl文件：
```python
pip install driver-0.1-py3-none-any.whl
```

