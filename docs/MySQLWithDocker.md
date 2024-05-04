# MySQL with Docker:Simple Guide
## Prerquisite
Befor starting, make sure you have [mysql-workbench-community](https://snapcraft.io/mysql-workbench-community) installed.

## Step 1: Pull [mysql:8.0.37](https://hub.docker.com/_/mysql)
First, we need to download the MySQL Docker image. Run the following command:
```
docker pull mysql:8.0.37
```
**Note** :  It's important to use mysql:8.0.37 as mysql:latest doesn't work well with MySQL Workbench.
## Step 2: Run MySQL Container
Now, let's run a MySQL container with the downloaded image. Use the command below:
```
docker run -d \   
 --name mysql_test \                         
 -p 3306:3306 \
 -e MYSQL_ROOT_PASSWORD=give_root_password \
 -e MYSQL_USER=your_name \
 -e MYSQL_PASSWORD=your_password \
 mysql:8.0.37
```
In this command :
 - `--name` flag is for give a name to our running container
 - `-d` flag is for running the container in detached mode
 - `-p` flag is for port mapping which expose the container's `3306` port to `localhost:3306` port
 - `-e` flag is for set the environment variables in the container 
## Step 3: Check Container Logs
You can check the logs of the MySQL container by running:
```
docker logs mysql_test
```
This will show information about the container's initialization.

![mysql_test_logs](https://github.com/REZ-OAN/Docker_Verse/blob/mysql_with_docker/docs/images/logs.png)
## Step 4: Connect to MySQL Server and Create Database
Open MySQL Workbench and log in as the root user using the password you provided. Create a new connection and
fill in the details. Test the connection, and if successful, save it.
Now, create a new database using this connection:

![root_testdb](https://github.com/REZ-OAN/Docker_Verse/blob/mysql_with_docker/docs/images/root_testdb.png)

Create A new connection

![root_testdb](https://github.com/REZ-OAN/Docker_Verse/blob/mysql_with_docker/docs/images/new_connection.png)

Now add the connection after filling the fields

![root_testdb](https://github.com/REZ-OAN/Docker_Verse/blob/mysql_with_docker/docs/images/add_connection.png)

Then click the `Test Connection` , you will see this dialog

![success_dialog](https://github.com/REZ-OAN/Docker_Verse/blob/mysql_with_docker/docs/images/success.png)

Then click ok to save the connection, now try to create a new database to this new connection using this sql command.
```
create database test_Data;
```
You may encounter an error due to lack of privileges. We'll fix this in the next step.

```
Error Code: 1044. Access denied for user 'abir'@'%' to database 'test_Data'
```
This error describes that the user `abir` from any host ( `'%'`) he does not have the `PRIVILEGES` to create
database or query. 
## Step 5: Grant Privileges
We have to give privileges to this user from the root user.
```
GRANT ALL PRIVILEGES ON *.* TO 'username'@'host';
```
Replace `username` with your MySQL username.
Now, the user should be able to create and access databases.

ex : 
```
GRANT ALL PRIVILEGES ON *.* TO 'abir'@'%';
```
Then the user will get all privileges alike root user.

Now  user `abir` can create and access the database on root.

![user_created_database](https://github.com/REZ-OAN/Docker_Verse/blob/mysql_with_docker/docs/images/created_database.png)
## Step 6: Verify Persistence
Stop the docker container
```
docker stop mysql_test
```
Then remove the stopped container
```
docker rm mysql_test
```
Then, run the container again and recreate the connection in MySQL Workbench. Check if the database you created earlier still exists. You'll notice that the data is lost. We'll fix this using Docker volumes.
```
docker run -d \   
 --name mysql_test \                         
 -p 3306:3306 \
 -e MYSQL_ROOT_PASSWORD=give_root_password \
 -e MYSQL_USER=your_name \
 -e MYSQL_PASSWORD=your_password \
 mysql:8.0.37
 ```
**Note** : In docker every time you run an image, it creates a new container out of the image, so previous data are lost.

## Step 7: Persist Data with Volumes

In docker it's solution is given using docker volumes. In Docker host binding method allow you to link a directory or file from your host machine directly into a Docker container. This means that any changes made to the files or directories within the container will directly affect the files on your host machine, and vice versa.

Create a directory on your host machine to store MySQL data:
```
mkdir -p /path/to/data_dir
```
For mysql it saves data to `/var/lib/mysql` in linux machines, Run the MySQL container again with volume
mounting:
```
docker run -d \   
 --name mysql_test \                         
 -p 3306:3306 \
 -v "/path/to/data_dir:/var/lib/mysql" \
 -e MYSQL_ROOT_PASSWORD=give_root_password \
 -e MYSQL_USER=your_name \
 -e MYSQL_PASSWORD=your_password \
 mysql:8.0.37
```
This command mounts the `/path/to/data_dir` directory on your host machine to the MySQL data directory in the container. Here, `/path/to/data_dir` this path does not need to be an absolute path, but it is better to use absolute path.

## Step 8: Create Table and Insert Data to Database

![data_table](https://github.com/REZ-OAN/Docker_Verse/blob/mysql_with_docker/docs/images/data_table.png)

Now again stop and remove the container.

## Step 9: Verify Persistence Again
Repeat the process of stopping and removing the container.
Run the container again and recreate the connection in MySQL Workbench.
Run a docker container again with the command :
```
docker run -d \   
 --name mysql_test \                         
 -p 3306:3306 \
 -v "/path/to/data_dir:/var/lib/mysql" \
 -e MYSQL_ROOT_PASSWORD=give_root_password \
 -e MYSQL_USER=your_name \
 -e MYSQL_PASSWORD=your_password \
 mysql:8.0.37
```
Check if the database you created earlier still exists. You'll see that the data persists even after recreating the container. That's the power of volume binding in Docker!
