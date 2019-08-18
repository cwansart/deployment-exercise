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

Another tool we'll need is Docker. To add it open up http://localhost:9082/configureTools/ again, scroll down to _Docker_ and click on _Add Docker_. Enter `docker` as the name and select the _Install automatically_ field. Click on _Add installer_ and select _Download from docker.com_. Then click _Save_.

We also need to add the _Config File Provider_ plugin to Jenkins. Open http://localhost:9082/pluginManager/available, click on the _Available_ tab and search for `config file provider`. Check the checkbox and click on the _Download now and install after restart_ button. Click on the _Restart Jenkins when installation is complete and no jobs are running_ checkbox to restart Jenkins after the installation.

At last we need some credentials for Nexus. You'll create a user called `publisher` with the password `publisher` in the next steps to publish Docker images to the Nexus. We'll take a step ahead and add those credentials in Jenkins now. To do that open http://localhost:9082/credentials/store/system/domain/_/newCredentials and type `publisher` into the _Username_ and the _Password_ field. In the _ID_ field enter `docker-publisher`. (Check the Jenkinsfile in the example project where it is used.)

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

Click on the gear icon in the upper bar, click on _Repositories_ finally click on _Create repository_ button. Select _maven2 (hosted)_, use the name _ok-public_, select _Snapshot_ for the _Version policy_ and click on _Create repository_ at the end of the page.

Your new repository URL will look like this: http://localhost:9081/repository/ok-public/ and also replace the port with the original port used by Nexus:
To use it inside the Jenkins container we need to replace `localhost` by the container name: http://nexus:8081/repository/ok-public/
We will use this repository to push artifacts to.

Create another repository and select _docker (hosted)_ this time. Enter a name like _ok-docker_, check the checkbox for _HTTP_ and enter the port _9999_ and create the repository.

In order to push artifacts to Nexus we create a new user called `publisher`. Head over to http://localhost:9081/#admin/security/users and create a new user with the ID `publisher` and the password `publisher`. Set the status to `Active` and move `nx-admin` to the `Granted` list. This, of course, should be restricted in a production environment.
The other data is up to you.

## Add settings.xml to Jenkins

To use the created repositories we need to add a settings.xml to Jenkins. In order to provide the `settings.xml` to the Jenkinsfile you need to open http://localhost:9082/configfiles/ and click on `Add new config file` on the left side. Select `Maven settings.xml` and enter the ID _deployment-settings_. This ID is used inside the Jenkinsfile to reference it.

Enter the following as the config content:

```
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" 
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>nexus</id>
      <username>publisher</username>
      <password>publisher</password>
    </server>
  </servers>
</settings>
```

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