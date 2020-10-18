# tensquare十次方文档

# 1、部署

## 1.1、基本环境安装

### 1.1.1、git的安装

```
安装git
	vim /etc/apt/source.list 
	修改源中的内容参考https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/
	apt-get update
	apt-get install git
	git clone https://xxx.git
```

### 1.1.2、虚拟环境的安装

```
pip install virtualenv
pip install virtualenvwrapper
编辑~/.bashrc文件，在文件原本内容的末尾追加如下文字:
	export WORKON_HOME=$HOME/.virtualenvs
	export PROJECT_HOME=$HOME/workspace
	source /usr/local/bin/virtualenvwrapper.sh
然后退出vim
执行命令：source ~/.bashrc
创建python3的虚拟环境
mkvirtualenv tensquare -p python3
```

### 1.1.3、项目依赖包安装

```
pip install -r requirements.txt
在安装的过程中出现mysqlclient错误：
apt-get install libmysqlclient-dev
apt-get install python3-dev
然后再执行pip install -r requirements.txt
进入到 tensquare/backend/tensquare/utils目录下，执行：
pip install fdfs_client-py-master.zip
```

## 1.2、MySQL的安装

```
安装mysql的数据库
apt-get install mysql-server
mysql -uroot -pmysql
create database tensquare charset utf8;
					
执行数据库的迁移,进入到tensquare/backend目录下：
	python manager.py makemigrations
	python manager.py migrate 
```

## 1.3、Redis的安装

```
apt-get install redis-server
```

## 1.4、mongo db的安装

```
mongo db 阿里云的系统就带了，只需要保证 /var/lib/mongodb/文件夹是干净的即可
重启mongo db服务
service mongo stop
service mongo start
```



## 1.5、Nginx的安装

```
安装nginx
apt-get install nginx
修改配置文件
	vim /etc/nginx/sites-available/default
	配置如下：
	server {
        listen 80 default_server;
        listen [::]:80 default_server;
        server_name _;
        location / {
             root   /root/tensquare/code/tensquare/frontend/;
             index  index.html index.htm;
        }
	}

重启服务
	/etc/init.d/nginx stop #启动
	/etc/init.d/nginx start #停止
```

## 1.6、Docker的安装

### 1.6.1、FDFS的安装

```
docker load -i 文件路径/fastdfs_docker.tar
运行tracker
docker run -dti --network=host --name tracker -v /var/fdfs/tracker:/var/fdfs delron/fastdfs tracker

运行storage
docker run -dti --network=host --name storage -e TRACKER_SERVER=172.16.2.198:22122 -v /var/fdfs/storage:/var/fdfs delron/fastdfs storage

```

### 1.6.2、Elasticsearch的安装

```
docker load -i 文件路径/elasticsearch-ik-2.4.6_docker.tar
修改elasticsearch的配置文件 /root/tensquare/code/tensquare/backend/tensquare/utils/elasticsearch/config/elasticsearch.yml第54行，更改ip地址为本机ip地址
network.host: 172.16.2.198
docker run -dti --network=host --name=elasticsearch -v /root/tensquare/code/tensquare/backend/tensquare/utils/elasticsearch/config:/usr/share/elasticsearch/config delron/elasticsearch-ik:2.4.6-1.0
```

## 1.7、相关配置文件的修改

dev.py

```
ALLOWED_HOSTS = ['47.92.144.35',"*"]
HAYSTACK_CONNECTIONS = {
    'default': {
        'URL': 'http://172.16.2.198:9200/'
    },
}
FDFS_URL = 'http://47.92.144.35:8888/'
CORS_ORIGIN_WHITELIST = (
    'http://47.92.144.35',
)

```

/root/tensquare/code/tensquare/backend/tensquare/utils/fdfs/client.conf

```
base_path=/root/tensquare/code/tensquare/backend/logs/

tracker_server=172.16.2.198:22122
```

/root/tensquare/code/tensquare/frontend/js/host.js

```
var host = 'http://47.92.144.35:8000/';
```

/root/tensquare/code/tensquare/frontend/plugins/config.js

```
config.filebrowserImageUploadUrl = "http://47.92.144.35:8000/upload/common/";
```

## 1.8、运行项目:

screen -S django 

进入到 /root/tensquare/code/tensquare/backend

```
workon tensquare
python manage.py runserver 0.0.0.0:8000
先按键 ctrl+a,再按 d
```

screen -S celery

进入到 /root/tensquare/code/tensquare/backend

```
workon tensquare
celery -A celery_tasks.main worker --loglevel=info
先按键 ctrl+a,再按 d
```

创建超级管理员账号：

python manage.py createsuperuser

admin

admin11111111

# 2、其他相关

## 2.1、screen相关命令

screen -ls     查看screen

screen -S screen1

在进入screen的前提下,

先按键 ctrl+a,再按 d  即可挂起(Detached)

screen -r screen1  进入挂起(Detached)的screen

进入后exit直接退出,相当于删除

## 2.2、git相关命令

**git config --global credential.helper store** 