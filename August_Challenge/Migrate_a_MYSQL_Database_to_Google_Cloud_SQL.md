# Migrate a MySQL Database to Google Cloud SQL

Your WordPress blog is running on a server that is no longer suitable. As the first part of a complete migration exercise, you are migrating the locally hosted database used by the blog to Cloud SQL.

The existing WordPress installation is installed in the /var/www/html/wordpress directory in the instance called blog that is already running in the lab. You can access the blog by opening a web browser and pointing to the external IP address of the blog instance.

The existing database for the blog is provided by MySQL running on the same server. The existing MySQL database is called wordpress and the user called blogadmin with password Password1*, which provides full access to that database.


## MY CHALLENGE

1.You need to create a new Cloud SQL instance to host the migrated database.
 
2. Once you have created the new database and configured it, you can then create a database dump of the existing database and import it into Cloud SQL.


3. When the data has been migrated, you will then reconfigure the blog software to use the migrated database.




##  You need to create a new Cloud SQL instance to host the migrated database.

```bash
    export ZONE=us-central1-a

    gcloud sql instances create wordpress --tier=db-n1-standard-1 --activation-policy=ALWAYS --zone $ZONE

```

- set root user password

```bash

    gcloud sql users set-password --host % root --instance wordpress --password Password1*


```

- Add Blog instance IP address to database connection

```bash
    export IP=<External_IP_of_Blog_Vm>

    gcloud sql instances patch wordpress --authorized-networks $IP --quiet

```

##  Create database in Instance CLoud Instance by sshing in Blog Vm

- ssh to Blog Vm instance , create Db 

```bash
    MYSQLIP=$(gcloud sql instances describe wordpress --format="value(ipAddresses.ipAddress)")

    mysql --host=$MYSQLIP --user=root --password

    ## Now inside Db
    CREATE DATABASE wordpress; 
    CREATE USER 'blogadmin'@'%' IDENTIFIED BY 'Password1*';
    GRANT ALL PRIVILEGES ON wordpress.* TO 'blogadmin'@'%';
    FLUSH PRIVILEGES;

    exit

```

- Create a dump of database

```bash
    sudo mysqldump -u root -pPassword1* wordpress > wordpress_backup.sql

    sudo service apache2 restart

    cd /var/www/html/wordpress

    sudo nano wp-config.php


```

- Check that wp-config points to the Cloud SQL intance by changing `DB_HOST` to the Ip of Cloud Instance

 >     sudo nano wp-config.php

```sql
    // ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'blogadmin');

/** MySQL database password */
define('DB_PASSWORD', 'Password1*');

/** MySQL hostname */
define('DB_HOST', '35.192.63.187');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8mb4');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');

```



-----------------------------------------------------END_OF_LAB-----------------------------------------------------------------------------------------