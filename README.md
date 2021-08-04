#Quickly try PostgreSQL streaming replication with docker-compose

Create docker-compose file to run postgresql streaming replication.

Installation
--------------------------

1. Initialize the master database directory
Press Ctrl + C to stop when the log of the wind that started listening is displayed.
	docker-compose -f postgres-compose.yaml run --rm master

2. Take a look at pg_hba.conf
	docker-compose -f postgres-compose.yaml run --rm master cat /var/lib/postgresql/data/pg_hba.conf

3. Add line "host replication all all trust" into pg_hba.conf file
	echo "host replication all all trust" | docker-compose -f postgres-compose.yaml run -T --rm master tee -a /var/lib/postgresql/data/pg_hba.conf

4. run  master service
	docker-compose -f postgres-compose.yaml up -d master

5. Create replication user in master
	docker exec -i master_cont su postgres  -c "createuser repl_user --login --replication;"

6. In slave from master use pg_basebackup to init data
	docker-compose -f postgres-compose.yaml run --rm slave pg_basebackup -h master -D /var/lib/postgresql/data -R --progress -U repl_user

7.Run slave service and pgadmin4 service
	docker-compose -f postgres-compose.yaml up -d