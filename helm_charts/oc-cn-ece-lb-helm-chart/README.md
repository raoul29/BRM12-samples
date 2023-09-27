
# CN ECE PATCHSET 12.0.0.8.0

## Jconsole connection to management enabled ecs pod
From CN ECE PS8 onward, ecs1 deployment has been deprecated.
user have option to make 1 or few replicas of ecs deployment to be management enabled using field
```managementEnabledReplicas``` in override file. Value of ```managementEnabledReplicas``` attribute 
must be at least 1 and less than ```replicas``` attribute of ecs.

use below command to label ecs pod for jconsole connection
```shell
kubectl -n namespace label po ecs-0 ece-jmx=ece-jmx-external
```

## Upgrade to CN ECE PS8
use below command to upgrade to PS8 from CN ECE PS7 patchset or interim patch

```shell
cd <path_to_ece_ps8_helmcharts>
upgradeECE_12.0.0.8.0.sh [-o overrideValues_ECE.yaml] [-n namespace] [-r helm_release]
```

Note:
1. From this release onward, configloader job has been replaced with single replica configloader 
   deployment.
2. Make sure to update override file with newly introduced attributes by comparing PS8 values.yaml 
   file.
3. Make sure to use only PS8 helm charts templates before installing/upgrading to CN ECE PS8 
   release.

## CN ECE performance tuning and resiliency Guidelines
1. For predicable resource usage, each replica of ECE statefulsets or deployments should be bounded 
   with heap, GC parameters and container memory, CPU resource request/limit.
   example: 
   heap parameters can be set as ```jvmOpts: "-Xms1024m -Xmx1536m"``` in override file.
   GC parameters can be set using ```jvmGCOpts```.
   container resources can be configured as 
   ```shell
    resources:
      limits:
        cpu: "500m"
        memory: "4096Mi"
      requests:
        cpu: "200m"
        memory: "1024Mi"
   ```

   if container resources are configured, make sure to allocate memory limit high enough to 
   accommodate pod ```RSS``` and ```WSS``` memory.
   
2. Heap size must be tuned according to TPS and subscribers base.
3. For HA, Anti-affinity in emGateway, httpGateway replicas can be set similar to ecs pod.
4. In case of ECE Active-Active set up, it is desirable to have separate Load balancers for 
   federation, remote site BRS requests and HttpGateway signaling traffic.
   Load balancers must be able to support bandwidth required for federation traffic.
5. Disruption to multiple ecs pods voluntarily must not be done. for example, terminating ecs pods 
   manually at the same time.
6. For monitoring ECE, sample grafana dashboard jsons are provided in 
   ```docker_files/samples/monitoring``` folder of ```oc-cn-ece-docker-files-release.tgz```
