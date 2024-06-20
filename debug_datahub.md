



# Commands

```bash
docker run --entrypoint "/bin/sh" --mount type=bind,src=/var/lib/docker/volumes/quickstart_mysqldata/_data/mysql/,dst=/var/lib/mysql --mount type=bind,src=/home/ec2-user/mysql_vol_dumps/quickstart_mysqldata/,dst=/data/backups --rm container-registry.oracle.com/mysql/community-server:8.0 -c "mysqldump -udatahub --password='datahub' --all-databases > /data/backups/all-databases.sql"

docker run --entrypoint "/bin/sh" --mount type=bind,src=/var/lib/docker/volumes/datahub_mysqldata/_data/mysql/,dst=/var/lib/mysql --mount type=bind,src=/home/ec2-user/mysql_vol_dumps/datahub_mysqldata/,dst=/data/backups --rm container-registry.oracle.com/mysql/community-server:8.0 -c "mysqldump -uadmin --password='datahub' --all-databases > /data/backups/all-databases.sql"

docker run --rm -v /var/lib/docker/volumes/datahub_mysqldata/_data/mysql/:/var/lib/mysql -v /home/ec2-user/mysql_vol_dumps/datahub_mysqldata/:/data/backups -it mysql:latest bash

docker run --entrypoint "/bin/sh"  -e MYSQL_DATABASE=datahub -e MYSQL_USER=datahub -e MYSQL_PASSWORD=datahub -e MYSQL_ROOT_PASSWORD=datahub --mount type=bind,src=/var/lib/docker/volumes/datahub_mysqldata/_data/mysql/,dst=/var/lib/mysql --mount type=bind,src=/home/ec2-user/mysql_vol_dumps/datahub_mysqldata/,dst=/data/backups --rm container-registry.oracle.com/mysql/community-server:8.0 -c "mysqldump -uadmin --password='datahub' --all-databases > /data/backups/all-databases.sql"

# creating bak and removing files
sudo mv /var/lib/docker/volumes/datahub_mysqldata /var/lib/docker/volumes/datahub_mysqldata.bak && sudo mv /var/lib/docker/volumes/quickstart_mysqldata /var/lib/docker/volumes/quickstart_mysqldata.bak

sudo rm /var/lib/docker/volumes/datahub_mysqldata/_data/ib_logfile0 && sudo rm /var/lib/docker/volumes/datahub_mysqldata/_data/ib_logfile1 && sudo rm /var/lib/docker/volumes/datahub_mysqldata/_data/ibdata1

# install datahub
sudo python3 -m pip install --upgrade pip wheel setuptools 
sudo python3 -m pip install --upgrade acryl-datahub
# start datahub (with version)
sudo python3 -m datahub docker quickstart --version=v0.12.1
# stop datahub
sudo python3 -m datahub docker quickstart --stop
# reset datahub
sudo python3 -m datahub docker nuke
# backup datahub
sudo python3 -m datahub docker quickstart --backup

# create/update user.props file
sudo -i                     #to peek into root folder
ls -a                       #get the list of dicts in root
/root/.datahub/user_conf    #path to user.props file mount the same path in docker compose file




# create dump
docker run -d -e MYSQL_ROOT_PASSWORD=datahub -v /var/lib/docker/volumes/datahub_mysqldata/_data/mysql:/var/lib/mysql --name mrestore mysql:5.7
docker exec -it mrestore bash
- mysql -u root -p
- mysql> SET GLOBAL innodb_fast_shutdown = 1;
- mysql_upgrade -u root -p
- mysqldump -u root -p databasename > databasename.sql
docker container stop mrestore && docker container rm mrestore
```


# Update on 2024-04-29

Hi @Sushant Mishrikotkar @Arun Nadaraj @Suprakash Nandy, we tried multiple things to solve these issues, some of them are as follows:
- Tried with latest datahub CLI version to see if these bugs are fixed in latest version ( We can only upgrade it to v12 because of dependency issues )
- sql minor version upgrade to force the schema check and fix the corrupted schema.
- quickstart docker container with specific version, quickstart was taking the latest docker compose file prior to it which can lead to version mismatch.
- removed log files and schema data ( backed up before removing and restored after testing ), Can crash when version mismatch.

We connected with Natarajan regarding this issue on Friday and followed up with him today, but he is still researching on it and yet to find a solution.

We created a dump file that contains the data from datahub sql db. Our last plan if nothing works is to nuke the current datahub instance and freshly install the datahub and restore the current data.
