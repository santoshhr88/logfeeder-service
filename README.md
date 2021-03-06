#### An Ambari Stack for Logfeeder
Ambari stack for easily installing and managing Logfeeder on HDP cluster

Limitations:

- This is not an officially supported service and *is not meant to be deployed in production systems*. It is only meant for testing demo/purposes
- It does not support Ambari/HDP upgrade process and will cause upgrade problems if not removed prior to upgrade
- Not tested on secured clusters

##### Steps

- Download HDP 2.2 sandbox VM image (Sandbox_HDP_2.2_VMware.ova) from [Hortonworks website](http://hortonworks.com/products/hortonworks-sandbox/)
- Import Sandbox_HDP_2.2_VMware.ova into VMWare and set the VM memory size to 8GB
- Now start the VM
- After it boots up, find the IP address of the VM and add an entry into your machines hosts file e.g.
```
192.168.191.241 sandbox.hortonworks.com sandbox    
```
- Connect to the VM via SSH (password hadoop) and start Ambari server
```
ssh root@sandbox.hortonworks.com
/root/start_ambari.sh
```
- Make sure SolrCloud and Logsearch are installed and started already

- To deploy the Logfeeder stack, run below
```
VERSION=`hdp-select status hadoop-client | sed 's/hadoop-client - \([0-9]\.[0-9]\).*/\1/'`
git clone https://github.com/abajwa-hw/Logfeeder-service.git /var/lib/ambari-server/resources/stacks/HDP/$VERSION/services/LOGFEEDER
```

- Restart Ambari
```
#on sandbox
sudo service ambari restart

#on non-sandbox
sudo service ambari-server restart

```
- Then you can click on 'Add Service' from the 'Actions' dropdown menu in the bottom left of the Ambari dashboard:

On bottom left -> Actions -> Add service -> check Logfeeder service -> Next -> Next -> Next -> Deploy

- Also ensure that the install location you are choosing (/opt/Logfeeder by default) does not exist

- On successful deployment you will see the Logfeeder service as part of Ambari stack and will be able to start/stop the service from here:
![Image](../master/screenshots/1.png?raw=true)

- You can see the parameters you configured under 'Configs' tab
![Image](../master/screenshots/2.png?raw=true)


#### Use Logfeeder 

- Lauch the Logfeeder webapp via navigating to http://sandbox.hortonworks.com:8888/

- Alternatively, you can launch it from Ambari via [iFrame view](https://github.com/abajwa-hw/iframe-view)
![Image](../master/screenshots/3.png?raw=true)



- One benefit to wrapping the component in Ambari service is that you can now monitor/manage this service remotely via REST API
```
export SERVICE=LOGFEEDER
export PASSWORD=admin
export AMBARI_HOST=sandbox.hortonworks.com
export CLUSTER=Sandbox

#get service status
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X GET http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE

#start service
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Start $SERVICE via REST"}, "Body": {"ServiceInfo": {"state": "STARTED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE

#stop service
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Stop $SERVICE via REST"}, "Body": {"ServiceInfo": {"state": "INSTALLED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE
```
#### Remove Logfeeder service

- To remove the Logfeeder service: 
  - Stop the service via Ambari
  - Delete the service
  
```
export SERVICE=LOGFEEDER
export PASSWORD=admin
export AMBARI_HOST=sandbox.hortonworks.com
export CLUSTER=Sandbox    
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X DELETE http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE
```
  - Remove artifacts 
  
    ```
    rm -rf /var/lib/ambari-server/resources/stacks/HDP/2.2/services/Logfeeder
    rm -rf /opt/Logfeeder
    ```
  - Restart Ambari
    ```
    service ambari restart
    ```