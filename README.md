# HDPdocker


```
docker-compose -f examples/compose/multi-container.yml build
```

##Running HDP 2.5:
```
docker-compose -f examples/compose/multi-container.yml up --no-recreate 
```

After a minute or so, you can access Ambari's Web UI at localhost:8080. Default User/PW is admin/admin.

##Using Ambari Blueprints:
To snapshot your cluster's configuration into a blueprint:

# You can extract a blueprint as soon as you click Deploy. No need to wait for install to complete.
```
curl --user admin:admin -H 'X-Requested-By:admin' localhost:8080/api/v1/clusters/dev?format=blueprint > examples/blueprints/multi-container.json 
```


# Can swap "single-container" for multi-container, or any type saved in examples/blueprints and examples/hostgroups
```
sh submit-blueprint.sh HDP_Cluster examples/blueprints/HDP_Cluster.json
```

There are additional blueprints for common test-beds in examples/blueprints, including Hive-LLAP and HBase-Phoenix.

##Notes:
1. Ambari, Hive, and Ranger dbs have been pre-created in the postgres database running at postgres.dev. To configure them in Ambari, set Postgres as the DB type and change the Database URL to point at postgres.dev (as depicted in screenshot below) and leave everything else as the default options. The password for the dbs are all "dev":
![hive-setup](/screenshots/hive-setup.png?raw=true)
2. The "node" container can be used for master, worker, or both types of services. The ambari-agent is configured to register with ambari-server.dev automatically, thus no SSH key setup is necessary. Use dn0.dev (and master0.dev if using multi-container):
![cluster-hosts](/screenshots/cluster-hosts.png?raw=true)
3. Yum packages for all HDP services have been pre-installed in the "node" container. This lets cluster install take place much faster at the expense of a spurious warning from Ambari during Host-Checks.
4. All Ambari and HDP repositories are downloaded at buildtime. The versions and URLs are specified in .env in the project's root
5. Docker for Linux is more restrictive about "su" use, which Ambari relies on heavily, thus examples/compose/single-container.yml and multi-container.yml images are marked "privileged:true". Read up on the implications.

##Helpful Hints:
If you HDFS having issues starting up/not leaving SafeMode, it's probably because docker-compose is re-using containers from a previous run.

To start with fresh containers, before each run do:
```
docker-compose -f examples/compose/multi-container.yml rm
Going to remove compose_ambari-server.dev_1, compose_dn0.dev_1, compose_master0.dev_1, compose_postgres.dev_1
Are you sure? [yN] y
Removing compose_ambari-server.dev_1 ... done
Removing compose_dn0.dev_1 ... done
Removing compose_master0.dev_1 ... done
Removing compose_postgres.dev_1 ... done
```

Docker for Mac sometimes has storage space problems. I recommend adding the following to your ~/.bash_profile and restarting terminal:
```
function docker-cleanup(){
 # remove untagged images  
 docker rmi $(docker images | grep none | awk '{ print $3}')
 # remove unused volumes  
 docker volume rm $(docker volume ls -q )  
 # `shotgun` remove unused networks
 docker network rm $(docker network ls | grep "_default")   
 # remove stopped + exited containers, I skip Exit 0 as I have old scripts using data containers.
 docker rm -v $(docker ps -a | grep "Exit [0-255]" | awk '{ print $1 }')
}
```

Run "docker-cleanup" if you run into Docker errors or "No space left on device" issues inside containers.

Since Hadoop UIs often link to hostnames, add the following to your hosts file:
```
echo "127.0.0.1 ambari-server ambari-server.dev" >> /etc/hosts
echo "127.0.0.1 master0 master0.dev" >> /etc/hosts
echo "127.0.0.1 dn0 dn0.dev" >> /etc/hosts
echo "127.0.0.1 dn1 dn1.dev" >> /etc/hosts
```


##TODO:
For hbase to work you need to complete /etc/hosts with lines that goes "EXT.ERN.AL.IP host01.domain host01", repeated properly for each host in the cluster every node should know about (including itself, more importantly), must exist.

