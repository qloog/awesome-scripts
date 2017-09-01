# awesome-scripts
常用脚本记录

## 备份数据库

1. 建立备份用户
```shell
mysql> grant select,lock tables,reload,super,file,show view on *.* to 'mysqlbackup'@'localhost' identified by 'mysql_ritto'; 
mysql> flush privileges;
```
2. 备份脚本
```shell
#!/bin/bash 
 
USERNAME=mysqlbackup 
PASSWORD=mysql_ritto 
 
DATE=`date +%Y-%m-%d` 
OLDDATE=`date +%Y-%m-%d -d '-20 days'` 
FTPOLDDATE=`date +%Y-%m-%d -d '-60 days'` 

 
MYSQL=/usr/local/mysql/bin/mysql 
MYSQLDUMP=/usr/local/mysql/bin/mysqldump 
MYSQLADMIN=/usr/local/mysql/bin/mysqladmin 
SOCKET=/tmp/mysql.sock 
 
BACKDIR=/data/backup/db 
[ -d ${BACKDIR} ] || mkdir -p ${BACKDIR} 
[ -d ${BACKDIR}/${DATE} ] || mkdir ${BACKDIR}/${DATE} 
[ ! -d ${BACKDIR}/${OLDDATE} ] || rm -rf ${BACKDIR}/${OLDDATE} 
 
for DBNAME in mysql db1 db2 db3 
do 
   ${MYSQLDUMP} --opt --master-data=2 --tz-utc=true -u${USERNAME} -p${PASSWORD} -S${SOCKET} ${DBNAME} | gzip > ${BACKDIR}/${DATE}/${DBNAME}-backup-${DATE}.s l.gz 
   logger "${DBNAME} has been backup successful - $DATE" 
   /bin/sleep 5 
done 
 
 
HOST=10.1.2.22 
FTP_USERNAME=db1 
FTP_PASSWORD=db1_ritto 
 
cd ${BACKDIR}/${DATE} 
 
ftp -i -n -v << ! 
open ${HOST} 
user ${FTP_USERNAME} ${FTP_PASSWORD} 
bin 
cd ${FTPOLDDATE} 
mdelete * 
cd .. 
rmdir ${FTPOLDDATE} 
mkdir ${DATE} 
cd ${DATE} 
mput * 
bye 
! 
```
