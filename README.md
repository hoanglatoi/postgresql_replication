# postgresql_replication
Quickly try PostgreSQL streaming replication with docker-compose

# Initialize the master database directory
# Press Ctrl + C to stop when the log of the wind that started listening is displayed.
docker-compose -f postgres-compose.yaml run --rm master

# Take a look at pg_hba.conf
docker-compose -f postgres-compose.yaml run --rm master cat /var/lib/postgresql/data/pg_hba.conf

# Add line "host replication all all trust" into pg_hba.conf file
echo "host replication all all trust" | docker-compose -f postgres-compose.yaml run -T --rm master tee -a /var/lib/postgresql/data/pg_hba.conf

# run  master service
docker-compose -f postgres-compose.yaml up -d master

# Create replication user in master
docker exec -i master_cont su postgres  -c "createuser repl_user --login --replication;"

# In slave from master use pg_basebackup to init data
docker-compose -f postgres-compose.yaml run --rm slave pg_basebackup -h master -D /var/lib/postgresql/data -R --progress -U repl_user

# Run slave service and pgadmin4 service
docker-compose -f postgres-compose.yaml up -d