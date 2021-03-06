# jenkins-tutorial

## Install Jenkins

Install Jenkins on the AWS EC2 Amazon Linux instance, as root.
```
$ sudo -i
# wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
# rpm --import http://pkg.jenkins-ci.org/redhat-stable/jenkins-ci.org.key
# yum install jenkins
# service jenkins start
# chkconfig jenkins on
```
Access Jenkins UI on the browser at `http://IP-ADDRESS:8080`. When prompted for the password, get that from following file on the EC2 host:
```
$ cat /var/lib/jenkins/secrets/initialAdminPassword
```

Create a new admin user after logged in to Jenkins.

## Create a simple build job.

Click on `New Item`

Enter `simple-build` as the job name and select `Freestyle project`.

Under `Source Code Management` tab:
- Select Git and 
   - for `Repositories -> Repository URL` enter `https://github.com/kurianinc/docker-java-sample.git`
   - for `Branches to build -> Branch Specifier` replace `master` with name of your branch you created during Git tutorial.

Under `Build Triggers` tab:
- Check `Poll SCM` and in the `Schedule` field enter  `* * * * *`. This means Jenkins is scheduled to poll the Git repo/branch every minute for a changes. If there would be any change Jenkins will kick off a build.

Under `Build` tab:
- `Add build step` -> `Execute shell` and in the `Command` field enter:
```
mvn package
mvn exec:java
```

`Save` the job. In a minute you will see that Jenkins would trigger a build. You can also kick off a build manually by clicking on the `Build Now` link.

Check in changes to `docker-java-sample` repo, as you have learned in the Git tutorial, and verify that Jenkins picks that up and builds.

## Install Artifactory plugin

Click on `Jenkins` -> `Manage Jenkins` -> `Manage Plugins`

Select `Available` tab and filter by `Artifactory`. Select `Artifactory` and click the button `Install without Restart`.

Check `Restart Jenkins when installation is complete and no jobs are running`. This will trigger restart of Jenkins instance. Log back in when the instance is back online again.

## Add Artifactory to Jenkins

Click `Jenkins` -> `Manage Jenkins` -> `Configure System`.

Scroll down to the section `Artifactory`.

In `Artifactory servers`:
- Enter `bootcamp-artifactory` for `Server ID`
- Enter `http://34.212.154.26:8081/artifactory` for `URL`.
- For `Default Deployer Credentials` enter Artifactory admin user creds.
- Click `Test Connection` button. You will see a message `Found Artifactory 6.0.1`.

## Create job with Artifactory integration

Click on `New Item`

Enter `artifactory-integration` as the job name and select `Freestyle project`.

In `Copy from` enter `simple-build` and click `OK` button.

Check `Build Environment` -> `Generic-Artifactory Integration`.
- Make sure `Artifactory upload server` field is filled with the Artifactory URL.
- Pick `Job configuration` for `Upload spec source`
- In `Spec` field enter following configuration:
```
{
    "files": [
        {
            "pattern": "target/*.jar",
            "target": "bootcamp/builds/libs/"
        }
    ]
}
```
Click `Save`.

## Enable build notification

Click `Manage Jenkins` -> `Configure System`.

Scroll down to `Jenkins Location`.
- Update `System Admin e-mail address` to a valid email address. Example: `Thomas TK <thomas123@gmail.com>`

Click `Save` button.

Make sure that `sendmail` service is running on the host:
```
$ sudo service sendmail status
sendmail (pid  2750) is running...
sm-client (pid  2759) is running...
```

Click on one of the build jobs - `simple-build` or `artifactory-integration`.

Click `Configure` on the left pane.

Scroll down to `Post-build Actions`. Pick `E-Mail notification`.
- Enter a valid e-mail address in the `Recipients` field.
- Make sure `Send e-mail for every unstable build` is checked.

Click `Save` button.

Add some bug by making a syntax error in `src/main/java/org/examples/java/App.java` and check that change into your branch.

Wait for the build to get triggered and the email notification be sent. The email might be in the Spam folder.

Fix the syntax error and repeat the check in step and check if you get a `back to normal` notification.

E-mail notification format can be customized by changing the configuration info at `Manage Jenkins` -> `System Configuration` -> `Extended E-mail Notification`.



