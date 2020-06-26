# devops-demo-app
Playing with this app in [Katacoda](https://www.katacoda.com/openshift/courses/playgrounds/openshift311)...

Use the commands below to avoid a [bug](https://github.com/openshift-labs/learn-katacoda/issues/278) with `oc rsh` :
```
curl -LO https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem

oc login -u developer -p developer https://master:8443 --certificate-authority=lets-encrypt-x3-cross-signed.pem --insecure-skip-tls-verify=true
```

Create the project and deploy the app using S2I:
```
oc new-project my-project

oc new-app https://github.com/mikakatua/devops-demo-app --name frontend
```

Create a route for external access:
```
oc expose service frontend

oc get route frontend -o jsonpath='{"http://"}{.spec.host}{"\n"}'
oc get route frontend -o jsonpath='{"http://"}{.spec.host}{"/prefs.php\n"}'
```

The app shows some errors because it needs the following environment var. This command triggers a new deployment:
```
oc set env dc/frontend ENVVAR=TEST
```

Deploy the database from a template. The connection settings are in the conf directory of the repo:
```
oc new-app --template=mariadb-persistent -p MYSQL_USER=devops -p MYSQL_PASSWORD=devops#2018 -p MYSQL_DATABASE=devopsdemo -p DATABASE_SERVICE_NAME=mariadb
```

Populate the database with the data in the repo:
```
git clone https://github.com/mikakatua/devops-demo-app.git

cat devops-demo-app/data/* | oc exec -i $(oc get pod -l name=mariadb -o=jsonpath='{.items[0].metadata.name}') -- mysql -u root devopsdemo
```

Storing database credentials in the repo is a very bad idea. One solution will be to create a configmap:
```
oc create cm my-config --from-file=devops-demo-app/conf

oc set volumes dc/frontend --add --configmap-name=my-config --mount-path=/opt/app-root/src/conf
```

Another advantage of using a configmap is that configuration changes do not require a new build or deployment. Try it editing the configmap `my-config` (it may take a while to see the changes).
