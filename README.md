# devops-demo-app
Playing with this app in Katacoda (https://www.katacoda.com/openshift/courses/playgrounds/openshift311)

curl -LO https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem

oc login -u developer -p developer https://master:8443 --certificate-authority=lets-encrypt-x3-cross-signed.pem --insecure-skip-tls-verify=true

oc new-project my-project

oc new-app https://github.com/mikakatua/devops-demo-app --name frontend

oc expose service frontend

oc get route frontend -o jsonpath='{"http://"}{.spec.host}{"\n"}'

oc new-app --template=mariadb-persistent -p MYSQL_USER=devops -p MYSQL_PASSWORD=devops#2018 -p MYSQL_DATABASE=devopsdemo -p DATABASE_SERVICE_NAME=mariadb

oc set env dc/frontend ENVVAR=TEST
