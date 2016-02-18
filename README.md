# Jenkins deployment

It's about how to deploy the jenkins server in the openshift platform.

Note: Jenkins docker image built from [openshift/jenkins repo](https://github.com/openshift/jenkins).

## Setup steps

### configure the NFS

export something from the NFS host (e.g. master1.ceyes.os)
```
sudo mkdir -p /home/data/jenkins
sudo chown nfsnobody:nfsnobody /home/data/jenkins
sudo chmod 700 /home/data/jenkins

sudo vi /etc/exports
/home/data/jenkins *(rw,sync,all_squash)
sudo exportfs -r
```

### create the physical volume
```
oc create -f scripts/pv-jenkins.yaml
```

### create the jenkins app using the template
```
oc project ceyes-ci
oc create -f scripts/jenkins-persistent-template.json
```

## config the access token in Jenkins

### grant the access rights
Jenkins server is running in `ceyes-ci` namespace, and the `ceyes-ci:default` service account is used by Jenkins by default.

Now if the job is about the `mytest` project for example, `ceyes-ci:default` needs the `edit` right with `mytest`:
```
oc policy add-role-to-user edit system:serviceaccount:ceyes-ci:default -n mytest
```

### access token
Since we use the `ceyes-ci:default` sa by default to let Jenkins access the Openshift resources, get the access token firstly
```
oc project ceyes-ci
oc describe serviceaccount default
oc describe secret default-token-xxxx
# Using the value from the token field
```

### assign the token in Jenkins job
create a string parameter with the key of `AUTH_TOKEN` is set in the Jenkins Job panel, and leave the `"The authorization token for interacting with OpenShift"` field of job as empty...

also this token can be tested by curl
```
curl --header "Authorization: Bearer xxx" https://master1.ceyes.os:8443/oapi/v1/
```
