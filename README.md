# cp4i-argocd

A) Install ArgoCD:

1. Create and configure the namespace:
   ```
   oc create ns openshift-gitops-operator
   oc label namespace openshift-gitops-operator openshift.io/cluster-monitoring=true
   ```
2. Create operator group:
   ```
   oc apply -f resources/00-gitops-operatorgroup.yaml
   ```
3. Create subscription:
   ```
   oc apply -f resources/00-gitops-subscription.yaml
   ```
   Confirm the operator has been deployed successfully before moving to the next step running the following commands:
   ```
   oc get pods -n openshift-gitops-operator
   oc get pods -n openshift-gitops
   ```
   You should receive a response like this for each command respectively.
   ```
   NAME                                                            READY   STATUS    RESTARTS   AGE
   openshift-gitops-operator-controller-manager-7859c4ddd4-j2g8z   2/2     Running   0          98s
   ```
   ```
   NAME                                                          READY   STATUS    RESTARTS   AGE
   cluster-6b66cd5687-h5fhm                                      1/1     Running   0          2m50s
   kam-868f97bd49-2mchk                                          1/1     Running   0          2m50s
   openshift-gitops-application-controller-0                     1/1     Running   0          2m46s
   openshift-gitops-applicationset-controller-7d9dcdf769-s7ssn   1/1     Running   0          2m46s
   openshift-gitops-dex-server-5c66897994-f4wrq                  1/1     Running   0          2m46s
   openshift-gitops-redis-5684c6fc5b-456nt                       1/1     Running   0          2m46s
   openshift-gitops-repo-server-dcf86f4c7-d6x28                  1/1     Running   0          2m46s
   openshift-gitops-server-55dbf6b78b-m4mhw                      1/1     Running   0          2m46s
   ```
4. Enable kustomize:
   ```
   oc patch argocd -n openshift-gitops openshift-gitops --patch '{"spec":{"kustomizeBuildOptions":"--enable-alpha-plugins=true --enable-exec"}}' --type=merge
   ```
5. Once ArgoCD is up and running get the access info:
   ```
   scripts/00c-argocd-access-info.sh
   ```

B) Set a default storage class for your cluster:

1. If you have provisioned your OCP cluster in Tech Zone you can use the following script to set the proper default storage class:
   ```
   scripts/99-odf-tkz-set-scs.sh
   ```

c) Create and configure CP4I namespace:

1. Create namespace for CP4I:
   ```
   oc create namespace tools
   ```

2. Grant access to ArgoCD to CP4I ns:
   ```
   oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n tools 
   ```

3. Set your entitlement key:
   ```
   export ENT_KEY=<my-key>
   ```

4. Create secret with entitlement key:
   ```
   oc create secret docker-registry ibm-entitlement-key \
        --docker-username=cp \
        --docker-password=$ENT_KEY \
        --docker-server=cp.icr.io \
        --namespace=tools
   ```