---
title: Apache Superset 通过GLUE可视化s3、RDS、RedShift中的数据
tags: 
- AWS
- 大数据
categories:
- AWS
---

## Superset部署
- [Superset部署](#superset部署)
  - [1、启动EC2](#1启动ec2)
    - [1.2、下载Superset Docker文件](#12下载superset-docker文件)
    - [1.3、修改Dockerffile](#13修改dockerffile)
    - [1.4、配置管理员](#14配置管理员)
    - [1.5、结果展示](#15结果展示)
    - [1.6、检查数据库驱动](#16检查数据库驱动)
    - [1.7 、常见错误处理](#17-常见错误处理)
  - [2、Glue（可选参考）](#2glue可选参考)
  - [3、IAM与安全组](#3iam与安全组)
    - [3.1、使用 athena](#31使用-athena)
    - [3.2、使用RedShift或RDS](#32使用redshift或rds)
  - [4、SUPERSET配置数据源并创建DASHBOARD](#4superset配置数据源并创建dashboard)
    - [4.1、连接Athena](#41连接athena)
    - [4.2、连接redshift](#42连接redshift)
  - [5、Superset教程](#5superset教程)


### 1、启动EC2

启动一台Amazon Linux EC2并安装启动docker环境，需要机型为t.xlarge及以上，EBS盘20GB以上。

```powershell
sudo yum update -y

# install python3 gcc
sudo yum install -y python3 libpq-dev python3-dev
sudo yum install -y gcc gcc-c++

# add following into ~/.bashrc
echo "export PATH=/usr/local/bin:$PATH" >> ~/.bashrc
echo "alias python=python3" >> ~/.bashrc
echo "alias pip=pip3" >> ~/.bashrc
source ~/.bashrc
python --version

# install docker
sudo yum -y install docker
sudo usermod -a -G docker ec2-user
sudo systemctl start docker
sudo systemctl status docker
sudo systemctl enable docker
sudo chmod 666 /var/run/docker.sock
docker ps

# install docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
```

####  1.2、下载Superset Docker文件

```powershell
sudo yum install -y git curl
git clone https://github.com/apache/incubator-superset/
```
 
#### 1.3、修改Dockerffile

```powershell
cd incubator-superset
vi Dockerfile
```
在末尾添加

```powershell
RUN pip install PyAthenaJDBC \    #这个是athena连接
        && pip install PyAthena \ #这个是athena连接
        && pip install psycopg2 \
        && pip install sqlalchemy-redshift # 这个是redshift连接
```

构建

```powershell
docker-compose build
docker-compose up
```
#### 1.4、配置管理员
需要配置管理员用户权限，在docker/docker-init.sh中默认创建用户admin（密码也是admin）但权限并没有更新，通过以下命令更新权限

```powershell
#进入docker
docker-compose exec superset bash
superset init
```
#### 1.5、结果展示
配置成功后，Superset默认使用8088端口，使用http://<EC2 公有IP>:8088访问，默认用户名和密码均为admin 可在Dockerfile、docker中命令、管理页面更改
![](https://img-blog.csdnimg.cn/64463456b32a4b909c38bdf6d87a9024.png)
#### 1.6、检查数据库驱动
查看数据库驱动是否安装成功
![](https://img-blog.csdnimg.cn/0c77645858ab48efa8c1f37020799699.png)
![](https://img-blog.csdnimg.cn/d1b4c52c3a294c1ea6ae0dcb8d1ceecd.png)若没有

```powershell

#进入docker
docker-compose exec superset bash
#安装  athena 和redshift驱动
pip install PyAthenaJDBC \
        && pip install PyAthena \
        && pip install psycopg2 \
        && pip install sqlalchemy-redshift
```


#### 1.7 、常见错误处理
build中如果出现error

```powershell
ERROR: Couldn't connect to Docker daemon at http+docker://localunixsocket - is it running?
```
原因：
1、docker 没有启动

```powershell
sudo systemctl start docker
```
2、用户不再docker用户组里面

```powershell
sudo gpasswd -a ${USER} docker
```
如果是第二个，请退出shell 再次登陆

### 2、Glue（可选参考）
[一、创建Glue](https://blog.csdn.net/Inplayable/article/details/126492674)
[二、数据清洗、转换](https://blog.csdn.net/Inplayable/article/details/126492661)

### 3、IAM与安全组
部署Superset的EC2附加的IAM角色需要有 Athena查询和Glue Catalog的权限，为方便起见可以赋予AthenaFullAccess和GlueFullAccess。但实际情况请按照最小权限原则来保障安全。
#### 3.1、使用 athena
如果在 Amazon Athena 中运行查询时，出现 "Access Denied"（拒绝访问）错误

```powershell
Your query has the following errors:Access denied when writing output to url: s3://my-athena-result-bucket/Unsaved/2021/05/07/example_query_ID.csv . Please ensure you are allowed to access the S3 bucket. If you are encrypting query results with KMS key, please ensure you are allowed to access your KMS key
```
向 IAM 用户授予所需的权限。以下 IAM 策略允许源数据存储桶和查询结果存储桶的最低权限：

```xml
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::【数据源的s3名称】"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::【数据源的s3名称】/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetBucketLocation",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:ListBucketMultipartUploads",
                "s3:AbortMultipartUpload",
                "s3:PutObject",
                "s3:ListMultipartUploadParts"
            ],
            "Resource": [
                "arn:aws:s3:::【存储Athena查询结果的s3名称】",
                "arn:aws:s3:::【存储Athena查询结果的s3名称】/*"
            ]
        }
    ]
}
```
请务必在此示例策略中替换【】中的内容
#### 3.2、使用RedShift或RDS
此处以RedShift集群为例
I、查看redshift集群VPC安全组
![](https://img-blog.csdnimg.cn/1883c5d93ead4a9ea7b5aabdb28aedff.png)
II、进入superset所在EC2实例安全组
![](https://img-blog.csdnimg.cn/6f02af100d8e4436a2245840d0a45af9.png)
![](https://img-blog.csdnimg.cn/139eebb147ce4c158adda2ef711dee91.png)
III、添加入站规则
![](https://img-blog.csdnimg.cn/9b23aafa985d4eb7a2163729974c3425.png)
![](https://img-blog.csdnimg.cn/b440d84fba564b2091c24a0eb1a838df.png)
### 4、SUPERSET配置数据源并创建DASHBOARD
此处提供了Athena与RedShift的连接教程，其他JDBC语法请看官方文档
#### 4.1、连接Athena


```powershell
awsathena+rest://@athena.{region}.amazonaws.com.cn/<Glue数据库表>?s3_staging_dir=<用来存储查询结果的S3地址>
```
![](https://img-blog.csdnimg.cn/75fd67f37f18415885013e219cd031a8.png)
Test connection
![](https://img-blog.csdnimg.cn/20738b25cc4e468e9ba72a4859439c4c.png)
#### 4.2、连接redshift

```powershell
redshift+psycopg2://<userName>:<DBPassword>@<AWS End Point>:<port>/<Database Name>
```
![](https://img-blog.csdnimg.cn/2ad642e3a67c4847816acc679df7a2d7.png)
创建连接
![](https://img-blog.csdnimg.cn/97e7912d179e4a4eb855fbd7114eb90e.png)
![](https://img-blog.csdnimg.cn/df0caed9c2194c1cbd2bc8fc8bdfa104.png)
![](https://img-blog.csdnimg.cn/a1bde9579fcb44f0bbf06389d48f9821.png)
### 5、Superset教程

[简单教学](https://blog.csdn.net/m0_37606374/article/details/120386913)
[superset官方文档](https://superset.apache.org/docs/databases/installing-database-drivers)
