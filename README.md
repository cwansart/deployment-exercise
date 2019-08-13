# Requirements

You should assign around 6-10 GB of ram to Docker. On macOS open the Docker preferences, click on _Advanced_ and assign enough memory. You can also increase the number of CPUs which will result in a better performance.

# First run

To start all services run 

```
docker-compose up
```

The first run will take a few minutes. Wait until the log messages repeat.

## Configure Jenkins

Find out the initial Jenkins password by running inside the folder where `docker-compose.yml` is stored:

```
cat ./data/jenkins/secrets/initialAdminPassword
```

Copy the printed password, open the browser and login to Jenkins: http://localhost:9082/
Install the recommended plugins and wait...

After finishing you can skip the creation of a user and change the admin password instead on: http://localhost:9082/user/admin/configure

To use Maven during the build we need to add it as a tool: http://localhost:9082/configureTools/
Scroll down to _Maven_ and click on _Add Maven_. Enter the name `maven3.6.1` and select this version in the drop down menu. Then you're done and click on _Save_ on the bottom of the page.

## Configure GitLab

After installing start GitLab via: http://localhost:9080/

First you need to set a password for you admin user. After that login with the user `root` and the password you set before.

Create a new project, click on _Import project_, select _Repo by URL_ and enter: https://github.com/cwansart/deployment-example-project.git

*Attention:* Set the _Visibility Level_ at the bottom of the page to _Visible_.

## Configure Nexus

Find out the initial Nexus admin password by running in the folder where the `docker-compose` is stored:

```
cat ./data/nexus/admin.password
```
_Note: There is no newline at the end of file; try not to copy anything from your command line ;-)_

Head over to http://localhost:9081 and login as _admin_ with the initial admin password.

On the first login the wizard will be run. Change the admin password and enable anonymous access at the end of the wizard.

Click on the gear icon in the upper bar, click on _Repositories_ finally click on _Create repository_ button. Select _maven2 (hosted)_, add a name like _ok-public_, select _Snapshot_ for the _Version policy_ and click on _Create repository_ at the end of the page.

Your new repository URL will look like this: http://localhost:9081/repository/ok-public/ and also replace the port with the original port used by Nexus:
To use it inside the Jenkins container we need to replace `localhost` by the container name: http://nexus:8081/repository/ok-public/
We will use this repository to push artifacts to.

To be done...

 * Create Docker registry to push images to

## Add settings.xml to Jenkins

To be done

## Add build to Jenkins

Open up Jenkins again and create a new item: http://localhost:9082/view/all/newJob

Enter a name for the project (e.g. deployment) and select _Pipeline_ as the type. Then click on _Ok_.

On the next page you configure the build, but we only need to change the _Pipline_ settings at the bottom for now.

Select _Pipeline script from SCM_ on the _Definition_ selection and select _Git_ as the SCM.

Head over to your GitLab repository which you just cloned and get the clone URL, for example: http://daafeb4f6b38/root/deployment-example-project.git
Replace the host (here: `daafeb4f6b38`) with `gitlab`. This works because docker-compose sets the network names inside the containers according to the names in the docker-compose file. So each container can communicate with the others by using those names.

The resulting repository url should look similar to this: http://gitlab/root/deployment-example-project.git

After saving the settings click on the left panel on _Build now_ and your build should work.

# Reset the containers

Simply delete the `data` folder and re-run the steps from _First run_.