
$$
\bigstar \textcolor{red}{Installing ~Junit ~with ~K8s ~cluster }
$$


# Requirements

To run this tutorial you will need:

-   Docker installed
-   Docker compose installed

>We need to create two dockers images, the **_jmeter-master_** and **_jmeter-slave_**.

## Step 1

Create a folder named **jmeter-base** and **save this Dockerfile inside**, the **execute**:

The  [Dockerfile](https://docs.docker.com/engine/reference/builder/)  bellow is base JMeter image.
```Dockerfile
FROM java:8

RUN wget http://mirror.nbtelecom.com.br/apache//jmeter/binaries/apache-jmeter-3.2.tgz

RUN tar -xvzf apache-jmeter-3.2.tgz

RUN rm apache-jmeter-3.2.tgz

RUN mv apache-jmeter-3.2 /jmeter

ENV JMETER_HOME /jmeter

# Add Jmeter to the Path

ENV PATH $JMETER_HOME/bin:$PATH
```
## Step 2
Create a folder named **_jmeter-master_** and **save this Dockerfile inside**, the **execute**:
```Dockerfile
FROM java:8

RUN wget http://mirror.nbtelecom.com.br/apache//jmeter/binaries/apache-jmeter-3.2.tgz

RUN tar -xvzf apache-jmeter-3.2.tgz

RUN rm apache-jmeter-3.2.tgz

RUN mv apache-jmeter-3.2 /jmeter

ENV JMETER_HOME /jmeter

# Add Jmeter to the Path

ENV PATH $JMETER_HOME/bin:$PATH
```
## Step 3
Create a folder named **_jmeter-slave_** and **save this Dockerfile inside**, the **execute**:
```Dockerfile
FROM jmeter-base

WORKDIR $JMETER_HOME

# Ports to be exposed from the container for JMeter Master

RUN mkdir scripts

EXPOSE 60000
```

## Step 4
Create a folder named **_docker-compose.yml_** 

```yaml
version: '2'

services:

master:

build: ./jmeter-master

container_name: master

tty: true # tty true make the docker image stay up even with nothing running

networks:

- jmeter-network

slave:

build: ./jmeter-slave

networks:

- jmeter-network

networks:

jmeter-network:

driver: bridge
```

## Step 5
Kubernetes deployments:

```yaml
apiVersion: apps/v1
kind: Deployment 
metadata:
 name: jmeter-slaves
 labels:
   jmeter_mode: slave
spec:
 replicas: 2 
 selector:
   matchlabels:
     jmeter_mode: slave
 template:
   metadata: 
    labels:
      jmeter_mode: slave
   spec :
    containers:
    - name: jmeter-slave
      image: jmeter-kubectl_slave:latest
      imagePullPolicy: Never
      ports: 
      -  containerPort: 1099
      -  containerPort: 50000
      resources:
        limits:
         cpu: 500m 
         memory: 512Mi
        requests:
         cpu: 250m 
         memory: 256Mi
      command: [ "/bin/bash", "-c", "--" ]
      args: [ "while true; do sleep 30; done;" ]
```

### Commands to be executed:
```bash
docker build -t jmeter-base ./jmeter-base
docker-compose build
docker-compose up -d
docker images #checking
kubectl create namespace jmeter
kubectl get namespaces #checking
kubectl create -n jmeter -f Jmeter-kubedeploy.yaml
kubectl config set-context --current --namespace=jmeter

kubectl get pods #Copy the name
kubectl exec -ti PODNAME -- /bin/bash
```

## DELETING Resources
```bash
kubectl delete deployment jmeter-slaves
kubectl delete namespace jmeter
```

[Source Article](https://vepo.medium.com/dockerized-jmeter-84228733e306)

[github Jmeter Docker](https://github.com/pedrocesar-ti/distributed-jmeter-docker)

[HELM Chart backdated](https://github.com/helm/charts/tree/master/stable/distributed-jmeter)

[Datree Jmeter](https://datree.io/helm-chart/jmeter-service-keptn)
