
###### **DNS Entry for Kuberneetes**


        Service Name.Namespace.Service.Domain 
        example:db-service.dev.svc.cluster.local
  
###### **Switching default Namespce in K8s** 

        kubectl config set-context $(kubectl config curren-context) --namespace=dev
        
        
###### **Manual Scheduling**

      In order to manually map a pood to node we have to create a binding object in k8s
      because we can only specify the "nodeName" property at the creation time
  Example Yaml for Pod-bind-definition.yaml  
      
     apiVersion: v1
     kind: Binding
     metadata:
       name: nginx
     target:
       apiVersion: v1
       kind: Node
       name: node02       
  
  Curl
  
        curl --header "Content-Tyoe: application/json" --request POST --data '{"apiVersion":"v1","kind": "Binding" ....}'
        http://$SERVER/api/v1/namespaces/default/pods/$PODNAME/binding/
        
###### **Labels&Selectors**


<img src="/Users/ankitsahi/Learning/CKA/images/Labels\&Selectors.png"<br>     alt="Markdown Monster icon"<br>     style="float: left; margin-right: 10px;" />

  [test](/Users/ankitsahi/Learning/CKA/images/Labels&Selectors.png)

###### **Taints&Tolerations**

###### **Node Selector**

###### **Node Affinity**

###### **Taints&Tolerations VS Node Affinity**

###### **Resource Requirements & Limits**

###### **DaemonSets**


###### **StaticPods**

###### **StaticPods**



        /etc/kubernetes/manifests
        
        -- Only Pods can be created by placing a pod definiation file in this directory 
           (--pod-manifest-pth=/etc/kubernetes/manifests) but we cannot create replicaSet or Deployments via this method
        -- KubeApi Servcer is aware of the static pods as kubelet creates a mirror image in the Kube-ApiServer
           - You can view the information of the POD but you cannot edit or delete the POD    
![kubelet](/Users/ankitsahi/Learning/CKA/images/StaticPod.png)
![kubelet](/Users/ankitsahi/Learning/CKA/images/KubeAdmCofigFileForManifestLocation.png)

        
###### **Multiple Schedulers & Configuring Kubernetes Scheduler**

### **Logging And Monitoring**

###### **Monitor Cluster Components**
    
        To capture the number of Node and how many are healthy as well as capture Performance parameters such as CPU,
        Memory and disk utilization it also capture the number of pods and performance parameters such as CPU,Memory etc
        
       -- Advance Metrics Server Outside Kubernetes: Prometheus , Elastic Stack, DataDog, dynatrace
       
       -- Metrics Server: We can have one metrics server per Kubernetes cluster the metric server retrieves metrics from
          each of the k8s nodes and pods, aggregates them and stores them in memory, as its a in-memory  solution it 
          does not store data in disk hence we have to rely on one of Advance Metrics Server for that.
          
          -- The kubelet contains a subcomponent known as cAdvisor or Container Advisor , which is responsible for
             retrieving performance metrics from pods and exposing them through the kubelet API
           
          -- Kubectl top node, Kubectl top pod 
             
             INSTALL
          -- minikube ddons enable metrics-server
             https://computingforgeeks.com/how-to-deploy-metrics-server-to-kubernetes-cluster
             
###### **Managing Application Logs**          
      
      
      -- kubectl logs -f event-simulator-pod 
      -- If there are multiple containers in the pod then you need to specify the container name to view the logs
         eg. kubectl logs -f <pod name> <container name>  
       

 
**Docker Storage**

        -- Storage Drivers
             /var/lib/Docker
             /var/aufs (aufs is for Ubuntu only)
             /var/containers  (Container Layer: Read&Write till Contiainer is exits )
             /var/Users/ankitsahi/Learning/CKA/images   (Image Layer: Read only )
             /var/volumes
             
             
             --> COPY-On-Write
             

              
      
        Storage Driver responsibilty is to maitaing the layered docker image build architecture, creating writble layer 
         , Moving files across layers to enable copy and write 
        -- Storage Driver: AUFS,ZFS,BTRFS,DEVICE MAPPER, OVERPLAY
  ![](/Users/ankitsahi/Learning/CKA/images/StorageDrivers.png)

**Volume Drivers: When continer exits the data is save in the Volume**

                Volume are Handeled by Volume driver plugins
                
               --Volume Drivers (local driver) : docker volume create data_volume  
                 /var/lib/docker
                 /var/volume
                 /var/volume/data_volume     
                 
                 --> docker run -v data_volume:/var/lib/mysql mysql   
                     [ Type: Volume mount | Volume mouning of existing docker volume]
                     
                 --> docker run -v data_volume2:/var/lib/mysql mysql  
                     [ Type: Volume mount | Volume mouning at run time ]
                     
                 --> docker run -v /data/mysql:/var/lib/mysql mysql     
                     [ Type: Bind mount | Mounting of fileSystem path ]
                 
                 example of Volume driver: Local| Azure Filke Storge | RexRay | NetApp | DigitalOcean etc 
                 
                 --> docker run -it \
                         --name mysql
                         --volume-driver rexry/ebs
                         --mount src=ebs-vol,target=/var/lib/mysql
                 
                 PS:
                   - docker run -v : is old style
                   - docker run \
                     --mount type=bind,source=/dat/mysql,trget=/vr/lib/mysql mysql 

**Container Storage Interface(CSI)**

![](/Users/ankitsahi/Learning/CKA/images/StandardsForK8s.png)
![](/Users/ankitsahi/Learning/CKA/images/CSI.png)
[==> Storage Deck](file:///Users/ankitsahi/Learning/CKA/007_Storage.pdf)

                Reason why we hve Persistent Volume claim is beacuse without this functionlity user / developer will 
                need to define the volume defintion in each Pod defintion file  & to manage Storage Centerally in more configurable and
                managed way
                
                PresistentVolumes
                   - accessModes [ ReadOnlyMany, ReadWriteOnce, ReadWriteMany , ReadWriteOncePod ]
                   - capacity [Storage: 1Gi]
                   - volumeType [hostPath , awsElasticBlockstore , azureDisk, azureFile etc]
                 
                PersistentVolumeClaims
                  - Every persistent volume claim is bond to  single PersistentVolume and k8s does the binding based on
                   the detiails in the PVC defination such as AccessMode,Volume Mode,Storge Class,Sufficient Capcity etc
                   
                  - Labels and Selector can be used to bind the PVC to a prticular PV.
                   
                  - There is a possibility that a smaller claim might bind to a PV with larger capcity and there are no 
                   beter options there in 1-to-1 b/w Claim and volume.
                   
                  - If there is no suitable volume to claim the PVC k8s object will be in pending stge to be scheduled
                  
                  - Once the PVC k8s object is deleted based on the vaule defined in the persistentVolumeReclaimPolicy
                    the respective action will be done the PV
                    
                    - Reatin : (Default Option) To be retained and to be manually deleted 
                    - Recyle : Data is scrapped and PV is available to be claimed
                    - Delete : Data PV k8s object is deleted.

**Storage Classes**

            

### **Networking**

        - Switching 1 & Routing 
