# Jenkins monitoring

## Monitoring Jenkins With prometheus 

### LAB on GCP 

| Host | Ferramentas| Tipo | 
| -------- | -------- | -------- |
| 34.67.153.94    | Jenkins   | Docker     |
| 34.67.153.94    | Prometheus   | Docker     |
| 34.67.153.94    | Grafana   | Docker     |


## Configuring Jenkins 

Jenkins: https://hub.docker.com/_/jenkins

This will store the jenkins data in /your/home on the host. Ensure that /your/home is accessible by the jenkins user in container (jenkins user - uid 1000) or use -u some_other_user parameter with docker run.

You can also use a volume container:

```bash
docker run -d -u root --name jenkins -p 8080:8080 -p 50000:50000 -v /var/jenkins:/var/jenkins_home jenkins/jenkins
```

```bash
[root@otrs-ansible is_otrs_az]# docker run -d -u root --name jenkins -p 8080:8080 -p 50000:50000 -v /var/jenkins:/var/jenkins_home jenkins/jenkins
1e61d87865dcd05a9c259f97f4d4742de365badff037092cb8def0b3963f5407
[root@otrs-ansible is_otrs_az]# docker ps
CONTAINER ID        IMAGE                                          COMMAND                  CREATED             STATUS                         PORTS                                              NAMES
1e61d87865dc        jenkins/jenkins                                "/sbin/tini -- /us..."   2 seconds ago       Up 2 seconds                   0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   jenkins
08f6e8bd0217        ocsinventory/ocsinventory-docker-image:2.8.1   "/usr/bin/docker-e..."   9 days ago          Restarting (0) 3 minutes ago                                                      ocsinventory-server
eae9e938a74e        mysql:5.7                                      "docker-entrypoint..."   9 days ago          Up 10 minutes                  0.0.0.0:3306->3306/tcp, 33060/tcp                  ocsinventory-db
```

Access you jenkins on your IP and port 8080 

http://34.67.153.94:8080 

![](https://i.imgur.com/j65qCzc.png)

Unlok your jenkins, get you password 


```bash
[root@otrs-ansible is_otrs_az]# cat  /var/jenkins/secrets/initialAdminPassword 
ea08621fe48d43b6a04110e110bcf7a6
```

After put your password, click on install suggested plugins 

![](https://i.imgur.com/0bu4qHh.png)

Now, create a admin user: 

![](https://i.imgur.com/JT0FSZf.png)

Now, you are logged on your jenkins lab. Let's create our frist project, click on create a job. 

![](https://i.imgur.com/bMDyTF0.png)

![](https://i.imgur.com/IPrOaUq.png)

We will execute a shell command for test our job

echo "This is test, for monitoring jenkins jobs"

Like this:

![](https://i.imgur.com/2zteL2l.png)

Save and execute our frist build:
![](https://i.imgur.com/8drOhKh.png)


## Configure Prometheus

Prometheus: https://hub.docker.com/r/prom/prometheus

### Architecture overview

![Prom](https://cdn.rawgit.com/prometheus/prometheus/e761f0d/documentation/images/architecture.svg)

```bash
docker run -d -u root --name prometheus -p 9090:9090 -v /var/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus


docker rm 
537b40ac35f9c9206f5d66727bebaf1982538a98ec99797055aaa9abc384c065

docker run -d -u root --name prometheus -p 9090:9090 -v prometheus_config:/etc/prometheus/ prom/prometheus

[root@otrs-ansible is_otrs_az]# docker ps
CONTAINER ID        IMAGE                                          COMMAND                  CREATED             STATUS                         PORTS                                              NAMES
515fced9aa52        prom/prometheus                                "/bin/prometheus -..."   3 seconds ago       Up 2 seconds                   0.0.0.0:9090->9090/tcp                             prometheus
1e61d87865dc        jenkins/jenkins                                "/sbin/tini -- /us..."   24 minutes ago      Up 24 minutes                  0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   jenkins

```
Then, access you prometheus on browser: 
http://34.67.153.94:9090/


![](https://i.imgur.com/jGpIsFj.png)

Our configure prometheus file: 

```bash
$vim /var/lib/docker/volumes/prometheus_config/_data/prometheus.yml 
```

Let's configure our jenkins for send metrics for prometheus: 

On context targets we can see the targets that prometheus is monitoring:
http://34.67.153.94:9090/targets

![](https://i.imgur.com/JFGml4c.png)

Now, install the prometheus plugin:
https://plugins.jenkins.io/prometheus/

![](https://i.imgur.com/UOS7AAj.png)

After that, go to configure Jenkins: http://34.67.153.94:8080/configure
find for prometheus, 

![](https://i.imgur.com/2fJJk0g.png)


Then access you jenkins URL, you will see the metrics; 

http://34.67.153.94:8080/prometheus/
![](https://i.imgur.com/oDgYx6e.png)

Create a new JOB monitoring on you prometheus file: 

```yaml
 32   - job_name: 'jenkins'
 33 
 34     metrics_path: /promehteus
 35    
 36     static_configs:
 37     - targets: ['http://34.67.153.94:8080']
 ```
 After that, restart the prometheus container
 
 ```bash
 [root@otrs-ansible is_otrs_az]# docker ps
CONTAINER ID        IMAGE                                          COMMAND                  CREATED             STATUS                         PORTS                                              NAMES
515fced9aa52        prom/prometheus                                "/bin/prometheus -..."   23 minutes ago      Up 23 minutes                  0.0.0.0:9090->9090/tcp                             prometheus
1e61d87865dc        jenkins/jenkins                                "/sbin/tini -- /us..."   48 minutes ago      Up 48 minutes                  0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   jenkins
08f6e8bd0217        ocsinventory/ocsinventory-docker-image:2.8.1   "/usr/bin/docker-e..."   9 days ago          Restarting (0) 3 minutes ago                                                      ocsinventory-server
eae9e938a74e        mysql:5.7                                      "docker-entrypoint..."   9 days ago          Up 58 minutes                  0.0.0.0:3306->3306/tcp, 33060/tcp                  ocsinventory-db
[root@otrs-ansible is_otrs_az]# docker restart 515fced9aa52        
515fced9aa52
```

Accessing our targets on prometheus again: http://34.67.153.94:9090/classic/targets

![](https://i.imgur.com/sMBMGQo.png)


## Run Grafana

Grafana: https://grafana.com/grafana/download
Documentation: https://grafana.com/docs/grafana/latest/installation/docker/

```bash
docker run -d --name=grafana -p 3000:3000 -v grafana_config:/etc/grafana -v grafana_data:/var/lib/grafana grafana/grafana
```


```bash
[root@otrs-ansible is_otrs_az]# docker run -d --name=grafana -p 3000:3000 -v grafana_config:/etc/grafana -v grafana_data:/var/lib/grafana grafana/grafana
Unable to find image 'grafana/grafana:latest' locally
Trying to pull repository docker.io/grafana/grafana ... 
latest: Pulling from docker.io/grafana/grafana
801bfaa63ef2: Pull complete 
bfa9705a3cb2: Pull complete 
12c11a7e9d94: Pull complete 
377c2dc21544: Pull complete 
4a20d1f981fb: Pull complete 
4f4fb700ef54: Pull complete 
5d9743dc37f2: Pull complete 
ec2035efdb39: Pull complete 
Digest: sha256:29e4e68a557fac7ead72496acea16a9b89626f3311ba7c4a9e39f7fb99f8f68f
Status: Downloaded newer image for docker.io/grafana/grafana:latest
adc8418dcfcbae2f2da30c97c5281b1f3719ed476c878d3b4b0776883b6ae421
[root@otrs-ansible is_otrs_az]# docker ps
CONTAINER ID        IMAGE                                          COMMAND                  CREATED              STATUS                          PORTS                                              NAMES
adc8418dcfcb        grafana/grafana                                "/run.sh"                About a minute ago   Up About a minute               0.0.0.0:3000->3000/tcp                             grafana
515fced9aa52        prom/prometheus                                "/bin/prometheus -..."   56 minutes ago       Up 24 minutes                   0.0.0.0:9090->9090/tcp                             prometheus
1e61d87865dc        jenkins/jenkins                                "/sbin/tini -- /us..."   About an hour ago    Up About an hour                0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   jenkins
08f6e8bd0217        ocsinventory/ocsinventory-docker-image:2.8.1   "/usr/bin/docker-e..."   9 days ago           Restarting (0) 36 minutes ago                                                      ocsinventory-server
eae9e938a74e        mysql:5.7                                      "docker-entrypoint..."   9 days ago           Up About an hour                0.0.0.0:3306->3306/tcp, 33060/tcp                  ocsinventory-db

```

Now we have 3 conteiners; 

Access you grafana on 3000 port, user: admin, password: admin 

![](https://i.imgur.com/7ETeGWD.png)

Add a new Data Source for prometheus
![](https://i.imgur.com/U0iwc6G.png)


Save and then, create your own dashboards or download one ready in https://grafana.com/grafana/dashboards 

![](https://i.imgur.com/KSzHQXp.png)

https://www.youtube.com/watch?v=N8P9ZLMA2xY&ab_channel=Thetips4you
