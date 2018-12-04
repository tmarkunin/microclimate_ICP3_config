# microclimate_ICP3_config


https://github.com/IBM/charts/blob/master/stable/ibm-microclimate/README.md

1. Create namespace for deployments kubectl create namespace microclimate-pipeline-deployments

2. Create namespace for Microclimate installation Create namespace for deployments kubectl create namespace devops

3. Change kubectl to run all commands in installation namespace:  
kubectl config set-context $(kubectl config current-context) --namespace devops

4. Get available cluster policies: kubectl get clusterimagepolicy

5. Edit cluster policy to be able access docker repos 
kubectl edit clusterimagepolicy   

add repos to the list

  - name: <your cluster name e.g. mycluster.icp>:8500:*
  - name: docker.io/maven:*
  - name: docker.io/lachlanevenson/k8s-helm:*
  - name: docker.io/jenkins/*

6. Create the Microclimate registry secret:
kubectl create secret docker-registry microclimate-registry-secret --docker-server=mycluster.icp:8500  --docker-username=admin --docker-password=admin  --docker-email=tmarkunin@gmail.com


secret/microclimate-registry-secret created

7. Create a secret so Microclimate can securely use Helm

cd ~/.helm/

ls -la

rw-------  1 root root 2.0K Nov 24 17:44 ca.pem
drwxr-xr-x  3 root root 4.0K Nov  2 15:29 cache
-rw-------  1 root root 1.4K Nov 24 17:44 cert.pem
-rw-------  1 root root 1.7K Nov 24 17:44 key.pem




root@icp31-master:~/.helm# kubectl create secret generic microclimate-helm-secret --from-file=cert.pem=$HELM_HOME/cert.pem --from-file=ca.pem=$HELM_HOME/ca.pem --from-file=key.pem=$HELM_HOME/key.pem


secret/microclimate-helm-secret created

8. Create the Microclimate pipeline secret in the microclimate-pipeline-deployments namespace

root@icp31-master:~/.helm# kubectl create secret docker-registry microclimate-pipeline-secret --docker-server=mycluster.icp:8500 --docker-username=admin --docker-password=admin --docker-email=tmarkunin@gmail.com --namespace=microclimate-pipeline-deployments


secret/microclimate-pipeline-secret created


9. Patch service account 

kubectl describe serviceaccount default --namespace microclimate-pipeline-deployments

If it does not contain any other secrets, patch the service account by using the following command:

kubectl patch serviceaccount default --namespace microclimate-pipeline-deployments -p "{\"imagePullSecrets\": [{\"name\": \"microclimate-pipeline-secret\"}]}"








