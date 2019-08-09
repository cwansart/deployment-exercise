# Requirements

You should assign around 6-10 GB of ram to Docker. On macOS open the Docker preferences, click on _Advanced_ and assign enough memory. You can also increase the number of CPUs which will result in a better performance.

# First run

To start all services run 

```
docker-compose up
```

The first run will take a few minutes. Wait until the log messages repeat.

## Configure Jenkins

Find out the initial Jenkins password by running:

```
docker exec -it $(docker ps | grep "jenkins/jenkins:2.189-alpine" | awk '{print $1}') cat /var/jenkins_home/secrets/initialAdminPassword
```

Copy the printed password, open the browser and login to Jenkins: http://localhost:9082/
Install the recommended plugins and wait...

After finishing you can skip the creation of a user and change the admin password instead on: http://localhost:9082/user/admin/configure

## Configure GitLab

To be done...

 * Initial setup
 * Add user
 * Add ssh key
 * Push example project (tbd) to GitLab

## Configure Nexus

To be done...

 * Create public maven repo to push artifacts to
 * Create Docker registry to push images to

# Reset the containers

Simply delete the `data` folder and re-run the steps from _First run_.