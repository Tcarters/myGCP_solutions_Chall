
- sudo apt install postgresql-13-pglogical

```bash
	sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/pg_hba_append.conf ."
sudo su - postgres -c "gsutil cp gs://cloud-training/gsp918/postgresql_append.conf ."
sudo su - postgres -c "cat pg_hba_append.conf >> /etc/postgresql/13/main/pg_hba.conf"
sudo su - postgres -c "cat postgresql_append.conf >> /etc/postgresql/13/main/postgresql.conf"
sudo systemctl restart postgresql@13-main

```

- sudo su - postgres
psql


```bash
	CREATE USER import_admin PASSWORD 'DMS_1s_cool!';
ALTER DATABASE orders OWNER TO import_admin;
ALTER ROLE import_admin WITH REPLICATION;
```

- Set Permissions: 

```bash
	\c postgres;
GRANT USAGE ON SCHEMA pglogical TO import_admin;
GRANT ALL ON SCHEMA pglogical TO import_admin;
GRANT SELECT ON pglogical.tables TO import_admin;
GRANT SELECT ON pglogical.depend TO import_admin;
GRANT SELECT ON pglogical.local_node TO import_admin;
GRANT SELECT ON pglogical.local_sync_status TO import_admin;
GRANT SELECT ON pglogical.node TO import_admin;
GRANT SELECT ON pglogical.node_interface TO import_admin;
GRANT SELECT ON pglogical.queue TO import_admin;
GRANT SELECT ON pglogical.replication_set TO import_admin;
GRANT SELECT ON pglogical.replication_set_seq TO import_admin;
GRANT SELECT ON pglogical.replication_set_table TO import_admin;
GRANT SELECT ON pglogical.sequence_state TO import_admin;
GRANT SELECT ON pglogical.subscription TO import_admin;
```

```bash
	\c orders;
GRANT USAGE ON SCHEMA pglogical TO import_admin;
GRANT ALL ON SCHEMA pglogical TO import_admin;
GRANT SELECT ON pglogical.tables TO import_admin;
GRANT SELECT ON pglogical.depend TO import_admin;
GRANT SELECT ON pglogical.local_node TO import_admin;
GRANT SELECT ON pglogical.local_sync_status TO import_admin;
GRANT SELECT ON pglogical.node TO import_admin;
GRANT SELECT ON pglogical.node_interface TO import_admin;
GRANT SELECT ON pglogical.queue TO import_admin;
GRANT SELECT ON pglogical.replication_set TO import_admin;
GRANT SELECT ON pglogical.replication_set_seq TO import_admin;
GRANT SELECT ON pglogical.replication_set_table TO import_admin;
GRANT SELECT ON pglogical.sequence_state TO import_admin;
GRANT SELECT ON pglogical.subscription TO import_admin;

```

```bash
	GRANT USAGE ON SCHEMA public TO import_admin;
GRANT ALL ON SCHEMA public TO import_admin;
GRANT SELECT ON public.distribution_centers TO import_admin;
GRANT SELECT ON public.inventory_items TO import_admin;
GRANT SELECT ON public.order_items TO import_admin;
GRANT SELECT ON public.products TO import_admin;
GRANT SELECT ON public.users TO import_admin;

```

```bash
	\c orders;
\dt
ALTER TABLE public.distribution_centers OWNER TO import_admin;
ALTER TABLE public.inventory_items OWNER TO import_admin;
ALTER TABLE public.order_items OWNER TO import_admin;
ALTER TABLE public.products OWNER TO import_admin;
ALTER TABLE public.users OWNER TO import_admin;
\dt
```

- - -
## Create Database in Cloud Shell

```bash
	export PROJECT_ID=$(gcloud config get-value project)
gcloud config set project $PROJECT_ID
	gcloud auth login --no-launch-browser
```

- Connet to the SQL instance:

```sql
	gcloud sql connect my-demo --user=root --quiet	
```

- Create DB:

```SQL
	CREATE DATABASE bike;
```

- Create a table in Cloud Shell

```SQL
	USE bike;
CREATE TABLE london1 (start_station_name VARCHAR(255), num INT);

	USE bike;
CREATE TABLE london2 (end_station_name VARCHAR(255), num INT);
```
vm-rdp-pass: #62pyZ%%l~9ajIQ

new: Bea[@a1f|G3<^7m


- - -
key: student-03-5ddb90db2bb1
pass: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDUFeNpbTXy4bbnKQ0/05jnhSK6khaJU0rwVMQKcOHyAkzd26n3OPmDh9mskRDwQcutFCoG8TE4ofS1rpwzzCr7K0ayGK4mYU8u9x8wRgL5Nma/LCoD77dtM2okmxKIsuPMmaAYlryQ8DQT+fmSYWztCG/9muQNlqPaUmxyT6sQv11lywTbgdXHH4wqCtwfFbmpmPef8kBGFT5XiuvFFKZ3nG0G4zgY3KYNJQNRczSZbmOYqPVncnPMOTN7D2CpDvQlFW6+LlnBEMbsHrLDXWby75JU1G45KDOyE7tZ4SJfUS7NrgWdMJzbD/y4QgOXYJDCZ8dCwsPJEPLfRneS0kkH student-03-5ddb90db2bb1@qwiklabs.net
