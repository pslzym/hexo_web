---
title: (Untitled)
permalink: untitled
id: 21
updated: '2016-09-29 16:23:27'
tags:
---

sudo docker save busybox-1 > /home/save.tar
docker ps –a	
docker commit 3a09b2588478 mynewimage
docker save mynewimage > /tmp/mynewimage.tar
docker load < /tmp/mynewimage.tar
docker run -v /u01/my3306:/u01/my3306 -v /u02/my3306:/u02/my3306 -itd --net=host benchmark-mysql
docker exec -it eaf4dff92055b26dbbc2431552ef8d3d119f06552465ce3942c1090e4ccc0651  /bin/bash

docker绑定cpu：
sudo docker update --cpuset-cpus=0-63
sudo docker update --cpuset-cpus=0-63  73a7b6d0c85d



初始化数据步骤：
1.my进去数据库
2. set global read_only=0;
3.CREATE DATABASE sbtest;
4.GRANT USAGE ON *.* TO 'sbtest'@'%' IDENTIFIED BY PASSWORD '*2AFD99E79E4AA23DE141540F4179F64FFB3AC521';
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER,INDEX  ON `sbtest`.* TO 'sbtest'@'%';
5.\q退出
6.nohup /u01/benchmark/tools/sysbench/sysbench/sysbench --mysql-user=sbtest --mysql-host=10.7.11.46 --mysql-table-engine=innodb --oltp-range-size=20 --mysql-dbname=sbtest --max-requests=0 --oltp-tables-count=1000 --test=/u01/benchmark/tools/sysbench/sysbench/tests/db/oltp.lua --oltp-point-selects=0 --max-time=10 --mysql-password=sbtest --oltp-table-size=1000000 --mysql-db=sbtest prepare &

http://shangliuyan.github.io/2015/07/04/celery%E6%9C%89%E4%BB%80%E4%B9%88%E9%9A%BE%E7%90%86%E8%A7%A3%E7%9A%84/


sourceinsight :  SI3US-296724-96469
