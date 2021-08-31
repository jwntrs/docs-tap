# Tanzu Application Platform Tutorial: Install and Deploy the Spring Pet Clinic App 
This topic describes how to use Tanzu Application Platform capabilities to install, configure, and deploy the Spring Pet Clinic app. This procedure includes Cloud Native Runtimes, Application Live View, Application Accelerator, and Tanzu Build Service. 

## Install and Configure the  Tanzu Application Platform Bundle

1. Connect to a Kubernetes cluster. The following example output includes an Amazon Elastic Kubernetes Service (EKS) cluster. 
  ```
  kubectl config get-contexts
  CURRENT   NAME                                                                CLUSTER                                                   AUTHINFO                                                  NAMESPACE
  *         arn:aws:eks:us-east-2:808682851023:cluster/tae-beta-ecs             arn:aws:eks:us-east-2:808682851023:cluster/tae-beta-ecs   arn:aws:eks:us-east-2:808682851023:cluster/tae-beta-ecs
  kubectl config current-context
  arn:aws:eks:us-east-2:808682851023:cluster/tae-beta-ecs
  kubectl cluster-info
  Kubernetes control plane is running at https://94A70B79EF7B1D5E718A4E96B2925F91.gr7.us-east-2.eks.amazonaws.com
  CoreDNS is running at https://94A70B79EF7B1D5E718A4E96B2925F91.gr7.us-east-2.eks.amazonaws.com/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

  To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
  kubectl get pods -A
  NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
  kube-system   aws-node-2fzbz             1/1     Running   0          4d
  kube-system   aws-node-4s2vw             1/1     Running   0          4d
  kube-system   aws-node-jf8vp             1/1     Running   0          4d
  kube-system   aws-node-m4rz7             1/1     Running   0          4d
  kube-system   coredns-5c778788f4-bl47d   1/1     Running   0          4d
  kube-system   coredns-5c778788f4-swf2n   1/1     Running   0          4d
  kube-system   kube-proxy-dcbwg           1/1     Running   0          4d
  kube-system   kube-proxy-rq6gh           1/1     Running   0          4d
  kube-system   kube-proxy-sbsrz           1/1     Running   0          4d
  kube-system   kube-proxy-xl2b5           1/1     Running   0          4d
  kubectl get ns
  NAME              STATUS   AGE
  default           Active   4d
  kube-node-lease   Active   4d
  kube-public       Active   4d
  kube-system       Active   4d
  kubectl get nodes -o wide
  NAME                                          STATUS   ROLES    AGE   VERSION              INTERNAL-IP     EXTERNAL-IP      OS-IMAGE         KERNEL-VERSION                CONTAINER-RUNTIME
  ip-172-31-18-218.us-east-2.compute.internal   Ready    <none>   4d    v1.20.4-eks-6b7464   172.31.18.218   18.118.114.102   Amazon Linux 2   5.4.129-63.229.amzn2.x86_64   docker://19.3.13
  ip-172-31-30-192.us-east-2.compute.internal   Ready    <none>   4d    v1.20.4-eks-6b7464   172.31.30.192   3.143.242.102    Amazon Linux 2   5.4.129-63.229.amzn2.x86_64   docker://19.3.13
  ip-172-31-46-226.us-east-2.compute.internal   Ready    <none>   4d    v1.20.4-eks-6b7464   172.31.46.226   18.223.16.117    Amazon Linux 2   5.4.129-63.229.amzn2.x86_64   docker://19.3.13
  ip-172-31-6-194.us-east-2.compute.internal    Ready    <none>   4d    v1.20.4-eks-6b7464   172.31.6.194    18.222.111.53    Amazon Linux 2   5.4.129-63.229.amzn2.x86_64   docker://19.3.13
  kubectl get service -A
  NAMESPACE     NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
  default       kubernetes   ClusterIP   10.100.0.1    <none>        443/TCP         4d
  kube-system   kube-dns     ClusterIP   10.100.0.10   <none>        53/UDP,53/TCP   4d
  ```
2. Deploy kapp-controller.

  ```
  kapp deploy -a kc -f https://github.com/vmware-tanzu/carvel-kapp-controller/releases/latest/download/release.yml
 
  Target cluster 'https://94A70B79EF7B1D5E718A4E96B2925F91.gr7.us-east-2.eks.amazonaws.com' (nodes: ip-172-31-46-226.us-east-2.compute.internal, 3+)
 
  Changes
 
  Namespace        Name                                                    Kind                      Conds.  Age  Op      Op st.  Wait to    Rs  Ri
  (cluster)        apps.kappctrl.k14s.io                                   CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^                internalpackagemetadatas.internal.packaging.carvel.dev  CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^                internalpackages.internal.packaging.carvel.dev          CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^                kapp-controller                                         Namespace                 -       -    create  -       reconcile  -   -
  ^                kapp-controller-cluster-role                            ClusterRole               -       -    create  -       reconcile  -   -
  ^                kapp-controller-cluster-role-binding                    ClusterRoleBinding        -       -    create  -       reconcile  -   -
  ^                kapp-controller-packaging-global                        Namespace                 -       -    create  -       reconcile  -   -
  ^                packageinstalls.packaging.carvel.dev                    CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^                packagerepositories.packaging.carvel.dev                CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^                pkg-apiserver:system:auth-delegator                     ClusterRoleBinding        -       -    create  -       reconcile  -   -
  ^                v1alpha1.data.packaging.carvel.dev                      APIService                -       -    create  -       reconcile  -   -
  kapp-controller  kapp-controller                                         Deployment                -       -    create  -       reconcile  -   -
  ^                kapp-controller-sa                                      ServiceAccount            -       -    create  -       reconcile  -   -
  ^                packaging-api                                           Service                   -       -    create  -       reconcile  -   -
  kube-system      pkgserver-auth-reader                                   RoleBinding               -       -    create  -       reconcile  -   -
 
  Op:      15 create, 0 delete, 0 update, 0 noop
  Wait to: 15 reconcile, 0 delete, 0 noop
  ```
3. Create an image pull secret. You use the image pull secret to pull Tanzu Application Platform packages.
  ```
  kubectl create ns tap-install
 
  namespace/tap-install created
  kubectl create secret docker-registry tap-registry -n tap-install --docker-server='registry.pivotal.io' --docker-username=$USRFULL --docker-password=$PASS
 
  secret/tap-registry created
  ```
  Where:
  - `$USRFULL` is your username. 
  - `$PASS` is your password. 
4. Install the Tanzu Application Platform bundle.
  ```
  tap-install % more tap-package-repo.yml
  apiVersion: packaging.carvel.dev/v1alpha1
  kind: PackageRepository
  metadata:
    name: tanzu-application-platform-packages
  spec:
    fetch:
      imgpkgBundle:
        image: registry.pivotal.io/tanzu-application-platform/tap-packages:0.1.0
        secretRef:
          name: tap-registry
  ```
  ```
  tap-install % kapp deploy -a tap-package-repo -n tap-install -f ./tap-package-repo.yml -y
 
  Target cluster 'https://94A70B79EF7B1D5E718A4E96B2925F91.gr7.us-east-2.eks.amazonaws.com' (nodes: ip-172-31-46-226.us-east-2.compute.internal, 3+)
 
  Changes
 
  Namespace    Name                                 Kind               Conds.  Age  Op      Op st.  Wait to    Rs  Ri
  tap-install  tanzu-application-platform-packages  PackageRepository  -       -    create  -       reconcile  -   -
 
  Op:      1 create, 0 delete, 0 update, 0 noop
  Wait to: 1 reconcile, 0 delete, 0 noop
  ```
5. Check for schema of the packages and create values.yml files for the following values:
  - `ingress.internal.namespace`
  - `ingress.reuse_crds`
  - `ingress.external.namespace`
  - `local_dns.domain`
  - `local_dns.enable`
  - `pdb.enable`
  - `provider`
  - `registry.password`
  - `registry.server`
  - `registry.username`
  ```
  tap-install % tanzu package available get cnrs.tanzu.vmware.com/1.0.1 --values-schema -n tap-install
  | Retrieving package details for cnrs.tanzu.vmware.com/1.0.1...
    KEY                         DEFAULT  TYPE     DESCRIPTION
    ingress.internal.namespace  <nil>    string   internal namespace
    ingress.reuse_crds          false    boolean  set true to reuse existing Contour instance
    ingress.external.namespace  <nil>    string   external namespace
    local_dns.domain            <nil>    string   domain name
    local_dns.enable            false    boolean  specify true if local DNS needs to be enabled
    pdb.enable                  true     boolean  <nil>
    provider                    <nil>    string   Kubernetes cluster provider
    registry.password           <nil>    string   registry password
    registry.server             <nil>    string   registry server
    registry.username           <nil>    string   registry username
  tap-install % tanzu package available get appliveview.tanzu.vmware.com/0.1.0 --values-schema -n tap-install
  | Retrieving package details for appliveview.tanzu.vmware.com/0.1.0...
    KEY                DEFAULT  TYPE    DESCRIPTION
    registry.password  <nil>    string  Image Registry Password
    registry.server    <nil>    string  Image Registry URL
    registry.username  <nil>    string  Image Registry Username
  tap-install % tanzu package available get accelerator.apps.tanzu.vmware.com/0.2.0 --values-schema -n tap-install
  | Retrieving package details for accelerator.apps.tanzu.vmware.com/0.2.0...
    KEY                           DEFAULT                                                             TYPE    DESCRIPTION
    engine.service_type           ClusterIP                                                           string  The service type for the Service of the engine.
    registry.password                                                                                 string  The password to use for authenticating with the registry where the App-Accelerator images are located.
    registry.server               registry.pivotal.io                                                 string  The hostname for the registry where the App-Accelerator images are located.
    registry.username                                                                                 string  The username to use for authenticating with the registry where the App-Accelerator images are located.
    server.watched_namespace      default                                                             string  The namespace that the server watches for accelerator resources.
    server.engine_invocation_url  http://acc-engine.accelerator-system.svc.cluster.local/invocations  string  The URL the server uses for invoking the accelerator engine.
    server.service_type           LoadBalancer
  tap-install % more values-cnr.yml
  ---
  registry:
   server: registry.pivotal.io
   username: <username>
   Password: <password>
 
 
   provider:
   pdb:
   enable: true
 
 
   ingress:
   reuse_crds:
   external:
   namespace:
   internal:
    namespace:
 
 
  Local_dns:
  tap-install % more values-alv.yml
  ---
  registry:
    server: registry.pivotal.io
    username: <username>
    password: <password>
  tap-install % more values-acc.yml
  registry:
    server: "registry.pivotal.io"
    username: <username>
    password: <password>
  server:
    service_type: "LoadBalancer"
    watched_namespace: "default"
    engine_invocation_url: "http://acc-engine.accelerator-system.svc.cluster.local/invocations"
  engine:
    service_type: "ClusterIP"
  ```
6. Install Cloud Native Runtimes. 
  ```
  tap-install % tanzu package install cloud-native-runtimes -p cnrs.tanzu.vmware.com -v 1.0.1 -n tap-install
  Added installed package 'cloud-native-runtimes' in namespace 'tap-install'
  
  tap-install % kubectl get pods -A
  
  tap-install  cloud-native-runtimes-ctrl                (cluster),contour-external,                                 true  10s
                                                         contour-internal,knative-discovery,knative-eventing,
                                                         knative-serving,knative-sources,triggermesh,vmware-sources
  ^            tanzu-application-platform-packages-ctrl  tap-install                                                 true  8m
  ^            tap-package-repo                          tap-install                                                 true  8m
 
  Lcs: Last Change Successful
  Lca: Last Change Age
 
  4 apps
 
  Succeeded
  tap-install % tanzu package  installed  list -A
  / Retrieving installed packages...
    NAME                   PACKAGE-NAME           PACKAGE-VERSION  STATUS               NAMESPACE
    cloud-native-runtimes  cnrs.tanzu.vmware.com  1.0.1            Reconcile succeeded  tap-install
  ```
7. Instal Flux. Delete the network policies after you install Flux.
  ```
  tap-install % kapp deploy -a flux -f https://github.com/fluxcd/flux2/releases/download/v0.15.0/install.yaml
 
  Target cluster 'https://94A70B79EF7B1D5E718A4E96B2925F91.gr7.us-east-2.eks.amazonaws.com' (nodes: ip-172-31-46-226.us-east-2.compute.internal, 3+)
 
  Changes
 
  Namespace    Name                                            Kind                      Conds.  Age  Op      Op st.  Wait to    Rs  Ri
  (cluster)    alerts.notification.toolkit.fluxcd.io           CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^            buckets.source.toolkit.fluxcd.io                CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^            cluster-reconciler                              ClusterRoleBinding        -       -    create  -       reconcile  -   -
  ^            crd-controller                                  ClusterRole               -       -    create  -       reconcile  -   -
  ^            crd-controller                                  ClusterRoleBinding        -       -    create  -       reconcile  -   -
  ^            flux-system                                     Namespace                 -       -    create  -       reconcile  -   -
  ^            gitrepositories.source.toolkit.fluxcd.io        CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^            helmcharts.source.toolkit.fluxcd.io             CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^            helmreleases.helm.toolkit.fluxcd.io             CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^            helmrepositories.source.toolkit.fluxcd.io       CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^            imagepolicies.image.toolkit.fluxcd.io           CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^            imagerepositories.image.toolkit.fluxcd.io       CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^            imageupdateautomations.image.toolkit.fluxcd.io  CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^            kustomizations.kustomize.toolkit.fluxcd.io      CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^            providers.notification.toolkit.fluxcd.io        CustomResourceDefinition  -       -    create  -       reconcile  -   -
  ^            receivers.notification.toolkit.fluxcd.io        CustomResourceDefinition  -       -    create  -       reconcile  -   -
  flux-system  allow-egress                                    NetworkPolicy             -       -    create  -       reconcile  -   -
  ^            allow-scraping                                  NetworkPolicy             -       -    create  -       reconcile  -   -
  ^            allow-webhooks                                  NetworkPolicy             -       -    create  -       reconcile  -   -
  ^            helm-controller                                 Deployment                -       -    create  -       reconcile  -   -
  ^            helm-controller                                 ServiceAccount            -       -    create  -       reconcile  -   -
  ^            image-automation-controller                     Deployment                -       -    create  -       reconcile  -   -
  ^            image-automation-controller                     ServiceAccount            -       -    create  -       reconcile  -   -
  ^            image-reflector-controller                      Deployment                -       -    create  -       reconcile  -   -
  ^            image-reflector-controller                      ServiceAccount            -       -    create  -       reconcile  -   -
  ^            kustomize-controller                            Deployment                -       -    create  -       reconcile  -   -
  ^            kustomize-controller                            ServiceAccount            -       -    create  -       reconcile  -   -
  ^            notification-controller                         Deployment                -       -    create  -       reconcile  -   -
  ^            notification-controller                         Service                   -       -    create  -       reconcile  -   -
  ^            notification-controller                         ServiceAccount            -       -    create  -       reconcile  -   -
  ^            source-controller                               Deployment                -       -    create  -       reconcile  -   -
  ^            source-controller                               Service                   -       -    create  -       reconcile  -   -
  ^            source-controller                               ServiceAccount            -       -    create  -       reconcile  -   -
  ^            webhook-receiver                                Service                   -       -    create  -       reconcile  -   -
 
  Op:      34 create, 0 delete, 0 update, 0 noop
  Wait to: 34 reconcile, 0 delete, 0 noop
  Succeeded
  
  tap-install % kubectl get pods -A
 
  Apps in all namespaces
 
  Namespace    Name                                      Namespaces                                                  Lcs   Lca
  default      flux                                      (cluster),flux-system                                       true  50s
  ^            kc                                        (cluster),kapp-controller,kube-system                       true  17m
  tap-install  cloud-native-runtimes-ctrl                (cluster),contour-external,                                 true  37s
                                                         contour-internal,knative-discovery,knative-eventing,
                                                         knative-serving,knative-sources,triggermesh,vmware-sources
  ^            tanzu-application-platform-packages-ctrl  tap-install                                                 true  10m
  ^            tap-package-repo                          tap-install                                                 true  10m
 
  Lcs: Last Change Successful
  Lca: Last Change Age
 
  5 apps
 
  Succeeded
  ```
  ```
  tap-install % kubectl delete -n flux-system networkpolicies --all
 
  networkpolicy.networking.k8s.io "allow-egress" deleted
  networkpolicy.networking.k8s.io "allow-scraping" deleted
  networkpolicy.networking.k8s.io "allow-webhooks" deleted
  ```
8. Install Application Accelerator with the Tanzu Application Service package. 
  ```
  tap-install % tanzu package install app-accelerator -p accelerator.apps.tanzu.vmware.com -v 0.2.0 -n tap-install -f values-acc.yml
 
   Added installed package 'app-accelerator' in namespace 'tap-install'
  tap-install % tanzu package  installed  list -A
  - Retrieving installed packages...
    NAME                   PACKAGE-NAME                       PACKAGE-VERSION  STATUS               NAMESPACE
    app-accelerator        accelerator.apps.tanzu.vmware.com  0.2.0            Reconcile succeeded  tap-install
    cloud-native-runtimes  cnrs.tanzu.vmware.com              1.0.1            Reconcile succeeded  tap-install
  tap-install % kubectl get pods -A
 
  Apps in all namespaces
 
  Namespace    Name                                      Namespaces                                                  Lcs   Lca
  default      flux                                      (cluster),flux-system                                       true  4m
  ^            kc                                        (cluster),kapp-controller,kube-system                       true  20m
  tap-install  app-accelerator-ctrl                      (cluster),accelerator-system                                true  1m
  ^            cloud-native-runtimes-ctrl                (cluster),contour-external,                                 true  15s
                                                         contour-internal,knative-discovery,knative-eventing,
                                                         knative-serving,knative-sources,triggermesh,vmware-sources
  ^            tanzu-application-platform-packages-ctrl  tap-install                                                 true  14m
  ^            tap-package-repo                          tap-install                                                 true  14m
 
  Lcs: Last Change Successful
  Lca: Last Change Age
 
  6 apps
 
  Succeeded
  ```
9. Install Application Live View.
  ```
  tap-install % tanzu package install app-live-view -p appliveview.tanzu.vmware.com -v 0.1.0 -n tap-install -f values-alv.yml
  - Installing package 'appliveview.tanzu.vmware.com'
  / Getting package metadata for 'appliveview.tanzu.vmware.com'
  - Creating service account 'app-live-view-tap-install-sa'
  - Creating cluster admin role 'app-live-view-tap-install-cluster-role'
  - Creating cluster role binding 'app-live-view-tap-install-cluster-rolebinding'
  - Creating secret 'app-live-view-tap-install-values'
  - Creating package resource
  / Package install status: Reconciling
 
   Added installed package 'app-live-view' in namespace 'tap-install'
  tap-install % tanzu package  installed  list -A
  / Retrieving installed packages...
    NAME                   PACKAGE-NAME                       PACKAGE-VERSION  STATUS               NAMESPACE
    app-accelerator        accelerator.apps.tanzu.vmware.com  0.2.0            Reconcile succeeded  tap-install
    app-live-view          appliveview.tanzu.vmware.com       0.1.0            Reconcile succeeded  tap-install
    cloud-native-runtimes  cnrs.tanzu.vmware.com              1.0.1            Reconcile succeeded  tap-install
  tap-install % kapp list -A
  Target cluster 'https://94A70B79EF7B1D5E718A4E96B2925F91.gr7.us-east-2.eks.amazonaws.com' (nodes: ip-172-31-46-226.us-east-2.compute.internal, 3+)
 
  Apps in all namespaces
 
  Namespace    Name                                      Namespaces                                                  Lcs   Lca
  default      flux                                      (cluster),flux-system                                       true  6m
  ^            kc                                        (cluster),kapp-controller,kube-system                       true  22m
  tap-install  app-accelerator-ctrl                      (cluster),accelerator-system                                true  10s
  ^            app-live-view-ctrl                        (cluster),tap-install                                       true  36s
  ^            cloud-native-runtimes-ctrl                (cluster),contour-external,                                 true  11s
                                                         contour-internal,knative-discovery,knative-eventing,
                                                         knative-serving,knative-sources,triggermesh,vmware-sources
  ^            tanzu-application-platform-packages-ctrl  tap-install                                                 true  15m
  ^            tap-package-repo                          tap-install                                                 true  16m
 
  Lcs: Last Change Successful
  Lca: Last Change Age
 
  7 apps
 
  Succeeded
  tap-install % kubectl get pods -A
  ```
10. Verify that you can access the Application Accelerator UI and the Application Live View UI using the external IP listed below:
  ```
  tap-install % kubectl get service -A
  NAMESPACE            NAME                         TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)                           AGE
  accelerator-system   acc-engine                   ClusterIP      10.100.253.164   <none>                                                                    80/TCP                            3m24s
  accelerator-system   acc-ui-server                LoadBalancer   10.100.74.90     a3b48dd9f15f746a48b503aee558bd96-309532596.us-east-2.elb.amazonaws.com    80:31689/TCP                      3m24s
  contour-external     contour                      ClusterIP      10.100.10.20     <none>                                                                    8001/TCP                          9m19s
  contour-external     envoy                        LoadBalancer   10.100.29.235    afc0cc5526ef74617a20f89f27433b6f-609822777.us-east-2.elb.amazonaws.com    80:31346/TCP,443:32120/TCP        9m18s
  contour-internal     contour                      ClusterIP      10.100.247.212   <none>                                                                    8001/TCP                          9m18s
  contour-internal     envoy                        ClusterIP      10.100.3.195     <none>                                                                    80/TCP,443/TCP                    9m18s
  default              kubernetes                   ClusterIP      10.100.0.1       <none>                                                                    443/TCP                           4d
  flux-system          notification-controller      ClusterIP      10.100.153.120   <none>                                                                    80/TCP                            6m19s
  flux-system          source-controller            ClusterIP      10.100.214.3     <none>                                                                    80/TCP                            6m20s
  flux-system          webhook-receiver             ClusterIP      10.100.54.115    <none>                                                                    80/TCP                            6m19s
  kapp-controller      packaging-api                ClusterIP      10.100.66.222    <none>                                                                    443/TCP                           22m
  knative-discovery    webhook                      ClusterIP      10.100.204.141   <none>                                                                    9090/TCP,8008/TCP,443/TCP         9m18s
  knative-eventing     broker-filter                ClusterIP      10.100.200.214   <none>                                                                    80/TCP,9092/TCP                   9m18s
  knative-eventing     broker-ingress               ClusterIP      10.100.50.73     <none>                                                                    80/TCP,9092/TCP                   9m18s
  knative-eventing     eventing-webhook             ClusterIP      10.100.188.40    <none>                                                                    443/TCP                           9m18s
  knative-eventing     imc-dispatcher               ClusterIP      10.100.54.100    <none>                                                                    80/TCP                            9m18s
  knative-eventing     inmemorychannel-webhook      ClusterIP      10.100.72.121    <none>                                                                    443/TCP                           9m18s
  knative-serving      activator-service            ClusterIP      10.100.114.196   <none>                                                                    9090/TCP,8008/TCP,80/TCP,81/TCP   9m20s
  knative-serving      autoscaler                   ClusterIP      10.100.2.22      <none>                                                                    9090/TCP,8008/TCP,8080/TCP        9m20s
  knative-serving      autoscaler-bucket-00-of-01   ClusterIP      10.100.3.124     <none>                                                                    8080/TCP                          9m17s
  knative-serving      controller                   ClusterIP      10.100.40.142    <none>                                                                    9090/TCP,8008/TCP                 9m20s
  knative-serving      net-certmanager-webhook      ClusterIP      10.100.168.189   <none>                                                                    9090/TCP,8008/TCP,443/TCP         9m18s
  knative-serving      networking-certmanager       ClusterIP      10.100.168.29    <none>                                                                    9090/TCP,8008/TCP                 9m18s
  knative-serving      webhook                      ClusterIP      10.100.228.52    <none>                                                                    9090/TCP,8008/TCP,443/TCP         9m19s
  knative-sources      rabbitmq-controller          ClusterIP      10.100.150.219   <none>                                                                    443/TCP                           9m18s
  knative-sources      rabbitmq-webhook             ClusterIP      10.100.86.29     <none>                                                                    443/TCP                           9m18s
  kube-system          kube-dns                     ClusterIP      10.100.0.10      <none>                                                                    53/UDP,53/TCP                     4d
  tap-install          application-live-view-5112   LoadBalancer   10.100.176.238   ac229c2c3af2f48be896688d5f176c16-1335129677.us-east-2.elb.amazonaws.com   5112:32387/TCP                    55s
  tap-install          application-live-view-7000   ClusterIP      10.100.29.252    <none>                                                                    7000/TCP                          55s
  vmware-sources       webhook                      ClusterIP      10.100.192.82    <none>                                                                    443/TCP                           9m18s
  ```
  ![Screenshot of page on Application Accelerator where you create new accelerators.](./images/accelerators1.png)
  ![Screenshot of page on Application Accelerator where you create new accelerators.](./images/app-live-view1.png)
## Install Tanzu Build Service 
1. Move the Tanzu Build Service bundle from the `registry.pivotal.io/build-service/bundle:1.2.2` registry to the `dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs` registry. To use image relocation with Tanzu Build Service, pull imgpkg to the local directory from the `dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs` registry.
  ```
  tap-install % imgpkg copy -b "registry.pivotal.io/build-service/bundle:1.2.2" --to-repo dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs
  Succeeded
  
  tap-install % imgpkg pull -b "dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs:1.2.2" -o tmp/bundle
  Pulling bundle 'dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:e03765dbce254a1266a8bba026a71ec908854681bd12bf69cd7d55d407bbca95'
    Extracting layer 'sha256:fe2f85ecc3c64ff1a1e1bf2ada42b179889f06f375fccb64f2f23ed24a331992' (1/1)
 
  Locating image lock file images...
  The bundle repo (dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs) is hosting every image specified in the bundle's Images Lock file (.imgpkg/images.yml)
 
  Succeeded
  ```
2. Install Tanzu Build Service.
  ```
  tap-install % ytt -f tmp/bundle/values.yaml -f tmp/bundle/config/ -v docker_repository=dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs -v docker_username=$USRFULL -v docker_password=$PASS -v tanzunet_username=$USRFULL -v tanzunet_password=$PASS | kbld -f tmp/bundle/.imgpkg/images.yml -f- | kapp deploy -a tanzu-build-service -f- -y
  resolve | final: build-init -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:31e95adee6d59ac46f5f2ec48208cbd154db0f4f8e6c1de1b8edf0cd9418bba8
  resolve | final: completion -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:1c63b9c876b11b7bf5f83095136b690fc07860c80b62a167c41b4c3efd1910bd
  resolve | final: dependency-updater -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:9f71c2fa6f7779924a95d9bcdedc248b4623c4d446ecddf950a21117e1cebd76
  resolve | final: dev.registry.pivotal.io/build-service/pod-webhook@sha256:3d8b31e5fba451bb51ccd586b23c439e0cab293007748c546ce79f698968dab8 -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:3d8b31e5fba451bb51ccd586b23c439e0cab293007748c546ce79f698968dab8
  resolve | final: dev.registry.pivotal.io/build-service/setup-ca-certs@sha256:3f8342b534e3e308188c3d0683c02c941c407a1ddacb086425499ed9cf0888e9 -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:3f8342b534e3e308188c3d0683c02c941c407a1ddacb086425499ed9cf0888e9
  resolve | final: dev.registry.pivotal.io/build-service/stackify@sha256:a40af2d5d569ea8bee8ec1effc43ba0ddf707959b63e7c85587af31f49c4157f -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:a40af2d5d569ea8bee8ec1effc43ba0ddf707959b63e7c85587af31f49c4157f
  resolve | final: dev.registry.pivotal.io/build-service/stacks-operator@sha256:1daa693bd09a1fcae7a2f82859115dc1688823330464e5b47d8b9b709dee89f1 -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:1daa693bd09a1fcae7a2f82859115dc1688823330464e5b47d8b9b709dee89f1
  resolve | final: gcr.io/cf-build-service-public/kpack/build-init-windows@sha256:20758ba22ead903aa4aacaa08a3f89dce0586f938a5d091e6c37bf5b13d632f3 -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:20758ba22ead903aa4aacaa08a3f89dce0586f938a5d091e6c37bf5b13d632f3
  resolve | final: gcr.io/cf-build-service-public/kpack/build-init@sha256:31e95adee6d59ac46f5f2ec48208cbd154db0f4f8e6c1de1b8edf0cd9418bba8 -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:31e95adee6d59ac46f5f2ec48208cbd154db0f4f8e6c1de1b8edf0cd9418bba8
  resolve | final: gcr.io/cf-build-service-public/kpack/completion-windows@sha256:1f8f1d98ea439ba6a25808a29af33259ad926a7054ad8f4b1aea91abf8a8b141 -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:1f8f1d98ea439ba6a25808a29af33259ad926a7054ad8f4b1aea91abf8a8b141
  resolve | final: gcr.io/cf-build-service-public/kpack/completion@sha256:1c63b9c876b11b7bf5f83095136b690fc07860c80b62a167c41b4c3efd1910bd -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:1c63b9c876b11b7bf5f83095136b690fc07860c80b62a167c41b4c3efd1910bd
  resolve | final: gcr.io/cf-build-service-public/kpack/controller@sha256:4b3c825d6fb656f137706738058aab59051d753312e75404fc5cdaf49c352867 -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:4b3c825d6fb656f137706738058aab59051d753312e75404fc5cdaf49c352867
  resolve | final: gcr.io/cf-build-service-public/kpack/lifecycle@sha256:c923a81a1c3908122e29a30bae5886646d6ec26429bad4842c67103636041d93 -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:c923a81a1c3908122e29a30bae5886646d6ec26429bad4842c67103636041d93
  resolve | final: gcr.io/cf-build-service-public/kpack/rebase@sha256:79ae0f103bb39d7ef498202d950391c6ef656e06f937b4be4ec2abb6a37ad40a -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:79ae0f103bb39d7ef498202d950391c6ef656e06f937b4be4ec2abb6a37ad40a
  resolve | final: gcr.io/cf-build-service-public/kpack/webhook@sha256:594fe3525a8bc35f99280e31ebc38a3f1f8e02e0c961c35d27b6397c2ad8fa68 -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:594fe3525a8bc35f99280e31ebc38a3f1f8e02e0c961c35d27b6397c2ad8fa68
  resolve | final: rebase -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:79ae0f103bb39d7ef498202d950391c6ef656e06f937b4be4ec2abb6a37ad40a
  resolve | final: secret-syncer -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:77aecf06753ddca717f63e0a6c8b8602381fef7699856fa4741617b965098d57
  resolve | final: setup-ca-certs -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:3f8342b534e3e308188c3d0683c02c941c407a1ddacb086425499ed9cf0888e9
  resolve | final: sleeper -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:0881284ec39f0b0e00c0cfd2551762f14e43580085dce9d0530717c704ade988
  resolve | final: stackify -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:a40af2d5d569ea8bee8ec1effc43ba0ddf707959b63e7c85587af31f49c4157f
  resolve | final: warmer -> dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs@sha256:4c8627a7f23d84fc25b409b7864930d27acc6454e3cdaa5e3917b5f252ff65ad
  Target cluster 'https://94A70B79EF7B1D5E718A4E96B2925F91.gr7.us-east-2.eks.amazonaws.com' (nodes: ip-172-31-46-226.us-east-2.compute.internal, 3+)
 
  Succeeded
  
  tap-install %
 
  tap-install % kubectl get pods -A
  NAMESPACE                NAME                                                    READY   STATUS    RESTARTS   AGE
  accelerator-system       acc-engine-69f7f7b6b8-pstf9                             1/1     Running   0          49m
  accelerator-system       acc-ui-server-6df7c597bd-fz5hv                          1/1     Running   0          49m
  accelerator-system       accelerator-controller-manager-796f7dff7-pqtvg          1/1     Running   0          49m
  build-service            build-pod-image-fetcher-2zwlg                           5/5     Running   0          12m
  build-service            build-pod-image-fetcher-dpgx7                           5/5     Running   0          12m
  build-service            build-pod-image-fetcher-p7f26                           5/5     Running   0          12m
  build-service            build-pod-image-fetcher-wztmz                           5/5     Running   0          12m
  build-service            cert-injection-webhook-745c5cbcc8-4vnz9                 1/1     Running   0          12m
  build-service            dependency-updater-controller-884fb6f6d-2skrp           1/1     Running   0          12m
  build-service            secret-syncer-controller-5ff66644f5-g5f9b               1/1     Running   0          12m
  build-service            smart-warmer-image-fetcher-45l95                        4/4     Running   0          59s
  build-service            smart-warmer-image-fetcher-lzc9p                        4/4     Running   0          53s
  build-service            smart-warmer-image-fetcher-rfbr4                        4/4     Running   0          52s
  build-service            smart-warmer-image-fetcher-wnkss                        4/4     Running   0          50s
  build-service            warmer-controller-68ff5847df-m2s2t                      1/1     Running   0          12m
  contour-external         contour-7c7795856b-8mqlz                                1/1     Running   0          55m
  contour-external         contour-7c7795856b-8xh6c                                1/1     Running   0          55m
  contour-external         envoy-7fzmk                                             2/2     Running   0          55m
  contour-external         envoy-ccrgj                                             2/2     Running   0          55m
  contour-external         envoy-fndxp                                             2/2     Running   0          55m
  contour-external         envoy-mcbfr                                             2/2     Running   0          55m
  contour-internal         contour-5b6b565d-6sbx4                                  1/1     Running   0          55m
  contour-internal         contour-5b6b565d-lhmvn                                  1/1     Running   0          55m
  contour-internal         envoy-l2tc6                                             2/2     Running   0          55m
  contour-internal         envoy-mv7x2                                             2/2     Running   0          55m
  contour-internal         envoy-rnhtf                                             2/2     Running   0          55m
  contour-internal         envoy-t2llr                                             2/2     Running   0          55m
  flux-system              helm-controller-68996c978c-rlfnb                        1/1     Running   0          52m
  flux-system              image-automation-controller-68d55fccd8-52wwk            1/1     Running   0          52m
  flux-system              image-reflector-controller-7784457d8f-n82r7             1/1     Running   0          52m
  flux-system              kustomize-controller-759f994b-tpbrf                     1/1     Running   0          52m
  flux-system              notification-controller-6fd769cbf4-qn2kk                1/1     Running   0          52m
  flux-system              source-controller-648d7f445d-2h2wc                      1/1     Running   0          52m
  kapp-controller          kapp-controller-ff4656bb-cjrl7                          1/1     Running   0          69m
  knative-discovery        controller-b59bd9449-8l9f4                              1/1     Running   0          55m
  knative-discovery        webhook-54c56fc5b8-8djnb                                1/1     Running   0          55m
  knative-eventing         eventing-controller-6c7fbfdb79-7vr4w                    1/1     Running   0          55m
  knative-eventing         eventing-webhook-5885fcccc9-7b5lh                       1/1     Running   0          55m
  knative-eventing         eventing-webhook-5885fcccc9-vbgzs                       1/1     Running   0          55m
  knative-eventing         imc-controller-bdb84c8ff-q67gj                          1/1     Running   0          55m
  knative-eventing         imc-dispatcher-586bd55496-6j97d                         1/1     Running   0          55m
  knative-eventing         mt-broker-controller-785589dd9d-vlx6v                   1/1     Running   0          55m
  knative-eventing         mt-broker-filter-7dfcf6589-fjhmz                        1/1     Running   0          55m
  knative-eventing         mt-broker-ingress-688cc7f74-l46p9                       1/1     Running   0          55m
  knative-eventing         rabbitmq-broker-controller-85d5b56f8-kwz9x              1/1     Running   0          55m
  knative-serving          activator-5b59f7c699-4cvts                              1/1     Running   0          55m
  knative-serving          activator-5b59f7c699-4mbtl                              1/1     Running   0          55m
  knative-serving          activator-5b59f7c699-sk2m5                              1/1     Running   0          55m
  knative-serving          autoscaler-8f85d46c-g4tnh                               1/1     Running   0          55m
  knative-serving          contour-ingress-controller-8fbc54c-whghl                1/1     Running   0          55m
  knative-serving          controller-645bdbc7d9-z5n5v                             1/1     Running   0          55m
  knative-serving          net-certmanager-webhook-dcdc76d4-qnxzg                  1/1     Running   0          55m
  knative-serving          networking-certmanager-7575777f9f-t4tf9                 1/1     Running   0          55m
  knative-serving          webhook-7cb844897b-sdqrz                                1/1     Running   0          55m
  knative-serving          webhook-7cb844897b-tkszw                                1/1     Running   0          55m
  knative-sources          rabbitmq-controller-manager-5dcb7c8494-t7c8k            1/1     Running   0          55m
  knative-sources          rabbitmq-webhook-bcb77f84f-2d6fw                        1/1     Running   0          55m
  kpack                    kpack-controller-6ffdb5cc6d-wj45x                       1/1     Running   0          12m
  kpack                    kpack-webhook-77567cd6b9-pxzn2                          1/1     Running   0          12m
  kube-system              aws-node-2fzbz                                          1/1     Running   0          4d1h
  kube-system              aws-node-4s2vw                                          1/1     Running   0          4d1h
  kube-system              aws-node-jf8vp                                          1/1     Running   0          4d1h
  kube-system              aws-node-m4rz7                                          1/1     Running   0          4d1h
  kube-system              coredns-5c778788f4-bl47d                                1/1     Running   0          4d1h
  kube-system              coredns-5c778788f4-swf2n                                1/1     Running   0          4d1h
  kube-system              kube-proxy-dcbwg                                        1/1     Running   0          4d1h
  kube-system              kube-proxy-rq6gh                                        1/1     Running   0          4d1h
  kube-system              kube-proxy-sbsrz                                        1/1     Running   0          4d1h
  kube-system              kube-proxy-xl2b5                                        1/1     Running   0          4d1h
  stacks-operator-system   controller-manager-7749db758-dwshw                      1/1     Running   0          12m
  tap-install              application-live-view-connector-57cd7c6c6-pv8dk         1/1     Running   0          47m
  tap-install              application-live-view-crd-controller-5b659f8f57-7zz25   1/1     Running   0          47m
  tap-install              application-live-view-server-79bd874566-qtcbc           1/1     Running   0          47m
  triggermesh              aws-event-sources-controller-7f9dd6d69-6ldbs            1/1     Running   0          55m
  vmware-sources           webhook-c9f67b5cd-tjv4p                                 1/1     Running   0          55m
  tap-install % kapp list -A
  Target cluster 'https://94A70B79EF7B1D5E718A4E96B2925F91.gr7.us-east-2.eks.amazonaws.com' (nodes: ip-172-31-46-226.us-east-2.compute.internal, 3+)
 
  Apps in all namespaces
   
  Namespace    Name                                      Namespaces                                                  Lcs   Lca
  default      flux                                      (cluster),flux-system                                       true  53m
  ^            kc                                        (cluster),kapp-controller,kube-system                       true  1h
  ^            tanzu-build-service                       (cluster),build-service,kpack,                              true  13m
                                                         stacks-operator-system
  tap-install  app-accelerator-ctrl                      (cluster),accelerator-system                                true  33s
  ^            app-live-view-ctrl                        (cluster),tap-install                                       true  47m
  ^            cloud-native-runtimes-ctrl                (cluster),contour-external,                                 true  18s
                                                         contour-internal,knative-discovery,knative-eventing,
                                                         knative-serving,knative-sources,triggermesh,vmware-sources
  ^            tanzu-application-platform-packages-ctrl  tap-install                                                 true  1h
  ^            tap-package-repo                          tap-install                                                 true  1h
   
  Lcs: Last Change Successful
  Lca: Last Change Age
   
  8 apps
   
  Succeeded
  ```

## Deploying the Spring Pet Clinic App
1. Create an Application Accelerator template for the `spring-pet-clinic` app from your git repository.
  1. Create a New Accelerator.
    ```
    tap-install % more new-accelerator.yaml
    apiVersion: accelerator.apps.tanzu.vmware.com/v1alpha1
    kind: Accelerator
    metadata:
      name: new-accelerator
    spec:
      git:
        url: https://github.com/sample-accelerators/new-accelerator
        ref:
          branch: main
          tag: v0.2.x
    tap-install % kubectl apply -f new-accelerator.yaml
    accelerator.accelerator.apps.tanzu.vmware.com/new-accelerator created
    tap-install % kubectl get accelerator                 
    NAME              READY   REASON   AGE
    new-accelerator   True             5s
    ```
    ![Screenshot of page on Application Accelerator that shows a new accelerator is created.](./images/image2021-8-24_9-37-33.png)
  2. Create the `spring-pet-clinic` accelerator, and add the accelerator.yaml to your git repo. You will see the accelerator you created in the Application Accelerator UI.
    ```
    tap-install % cd ../Downloads
    Downloads % ls | grep spring
    spring-petclinic-acc.zip
    Downloads % unzip spring-petclinic-acc.zip
    Archive:  spring-petclinic-acc.zip
       creating: spring-petclinic-acc/
      inflating: spring-petclinic-acc/accelerator.yaml 
      inflating: spring-petclinic-acc/k8s-resource.yaml 
      inflating: spring-petclinic-acc/README.md 
      inflating: spring-petclinic-acc/accelerator-log.md 
    Downloads % cd spring-petclinic-acc
    spring-petclinic-acc % ls
    README.md       accelerator-log.md  accelerator.yaml    k8s-resource.yaml
    spring-petclinic-acc % kubectl apply -f k8s-resource.yaml
    accelerator.accelerator.apps.tanzu.vmware.com/spring-pet-clinic-acc created
    spring-petclinic-acc % kubectl get accelerator             
    NAME                    READY   REASON   AGE
    new-accelerator         True             4m32s
    spring-pet-clinic-acc   True             7s
    spring-petclinic-acc % more accelerator.yaml  
    accelerator:
      displayName: spring-petclinic-acc
      description: spring per clinic accelerator
      iconUrl: https://raw.githubusercontent.com/sample-accelerators/icons/master/icon-tanzu-light.png
      tags: []
      options:
      - name: optionName
        label: Nice Label
        display: true
        defaultValue: ""
    engine:
      include:
      - '**'
    ```
    ![Screenshot of page on Application Accelerator that shows a new accelerator is created.](./images/image2021-8-24_9-48-46.png)
    ![Screenshot of page on Application Accelerator that shows a new accelerator is created.](./images/image2021-8-24_9-44-16.png)
    ![Screenshot of page on Application Accelerator that shows a new accelerator is created.](./images/image2021-8-24_9-45-33.png)
2. Generate a project named `spring-pet-clinic-eks`. Create a new git repo `spring-pet-clinic-eks.git`. Using the new `spring-petclinic-demo-acc`, add the `spring-pet-clinic-eks` project to the `spring-pet-clinic-eks.git` git repo.
  ![Screenshot of page on Application Accelerator that shows a new accelerator is created.](./images/image2021-8-24_9-48-46.png)
  ```
  spring-petclinic-acc % cd ../
  Downloads % ls | grep spring                 
  spring-pet-clinic-eks.zip
  spring-petclinic-acc
  spring-petclinic-acc.zip
  Downloads % unzip spring-pet-clinic-eks.zip
  Archive:  spring-pet-clinic-eks.zip
     creating: spring-pet-clinic-eks/
    inflating: spring-pet-clinic-eks/mvnw 
     creating: spring-pet-clinic-eks/src/
     creating: spring-pet-clinic-eks/src/main/
     creating: spring-pet-clinic-eks/src/main/java/
     creating: spring-pet-clinic-eks/src/main/java/org/
     creating: spring-pet-clinic-eks/src/main/java/org/springframework/
     creating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/
     creating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/
     creating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/vet/
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/vet/VetController.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/vet/Vet.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/vet/Specialty.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/vet/Vets.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/vet/VetRepository.java 
     creating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/model/
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/model/package-info.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/model/NamedEntity.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/model/Person.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/model/BaseEntity.java 
     creating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/owner/
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/owner/Owner.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/owner/OwnerRepository.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/owner/PetType.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/owner/PetValidator.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/owner/PetRepository.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/owner/Pet.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/owner/VisitController.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/owner/PetTypeFormatter.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/owner/PetController.java 
     creating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/system/
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/system/WelcomeController.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/system/CrashController.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/system/CacheConfiguration.java 
     creating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/visit/
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/visit/VisitRepository.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/visit/Visit.java 
    inflating: spring-pet-clinic-eks/src/main/java/org/springframework/samples/petclinic/PetClinicApplication.java 
     creating: spring-pet-clinic-eks/src/main/resources/
     creating: spring-pet-clinic-eks/src/main/resources/templates/
     creating: spring-pet-clinic-eks/src/main/resources/templates/owners/
    inflating: spring-pet-clinic-eks/src/main/resources/templates/owners/ownerDetails.html 
    inflating: spring-pet-clinic-eks/src/main/resources/templates/owners/createOrUpdateOwnerForm.html 
    inflating: spring-pet-clinic-eks/src/main/resources/templates/owners/ownersList.html 
    inflating: spring-pet-clinic-eks/src/main/resources/templates/owners/findOwners.html 
     creating: spring-pet-clinic-eks/src/main/resources/templates/fragments/
    inflating: spring-pet-clinic-eks/src/main/resources/templates/fragments/inputField.html 
    inflating: spring-pet-clinic-eks/src/main/resources/templates/fragments/selectField.html 
    inflating: spring-pet-clinic-eks/src/main/resources/templates/fragments/layout.html 
     creating: spring-pet-clinic-eks/src/main/resources/templates/vets/
    inflating: spring-pet-clinic-eks/src/main/resources/templates/vets/vetList.html 
    inflating: spring-pet-clinic-eks/src/main/resources/templates/error.html 
     creating: spring-pet-clinic-eks/src/main/resources/templates/pets/
    inflating: spring-pet-clinic-eks/src/main/resources/templates/pets/createOrUpdateVisitForm.html 
    inflating: spring-pet-clinic-eks/src/main/resources/templates/pets/createOrUpdatePetForm.html 
    inflating: spring-pet-clinic-eks/src/main/resources/templates/welcome.html 
     creating: spring-pet-clinic-eks/src/main/resources/static/
     creating: spring-pet-clinic-eks/src/main/resources/static/resources/
     creating: spring-pet-clinic-eks/src/main/resources/static/resources/images/
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/images/pets.png 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/images/spring-pivotal-logo.png 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/images/spring-logo-dataflow.png 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/images/favicon.png 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/images/platform-bg.png 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/images/spring-logo-dataflow-mobile.png 
     creating: spring-pet-clinic-eks/src/main/resources/static/resources/fonts/
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/fonts/varela_round-webfont.ttf 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/fonts/varela_round-webfont.eot 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/fonts/montserrat-webfont.ttf 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/fonts/varela_round-webfont.woff 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/fonts/varela_round-webfont.svg 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/fonts/montserrat-webfont.eot 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/fonts/montserrat-webfont.svg 
    inflating: spring-pet-clinic-eks/src/main/resources/static/resources/fonts/montserrat-webfont.woff 
     creating: spring-pet-clinic-eks/src/main/resources/messages/
    inflating: spring-pet-clinic-eks/src/main/resources/messages/messages_de.properties 
    inflating: spring-pet-clinic-eks/src/main/resources/messages/messages_en.properties 
    inflating: spring-pet-clinic-eks/src/main/resources/messages/messages_es.properties 
    inflating: spring-pet-clinic-eks/src/main/resources/messages/messages.properties 
     creating: spring-pet-clinic-eks/src/main/resources/db/
     creating: spring-pet-clinic-eks/src/main/resources/db/h2/
    inflating: spring-pet-clinic-eks/src/main/resources/db/h2/schema.sql 
    inflating: spring-pet-clinic-eks/src/main/resources/db/h2/data.sql 
     creating: spring-pet-clinic-eks/src/main/resources/db/hsqldb/
    inflating: spring-pet-clinic-eks/src/main/resources/db/hsqldb/schema.sql 
    inflating: spring-pet-clinic-eks/src/main/resources/db/hsqldb/data.sql 
     creating: spring-pet-clinic-eks/src/main/resources/db/mysql/
    inflating: spring-pet-clinic-eks/src/main/resources/db/mysql/data.sql 
    inflating: spring-pet-clinic-eks/src/main/resources/db/mysql/user.sql 
    inflating: spring-pet-clinic-eks/src/main/resources/db/mysql/petclinic_db_setup_mysql.txt 
    inflating: spring-pet-clinic-eks/src/main/resources/db/mysql/schema.sql 
    inflating: spring-pet-clinic-eks/src/main/resources/application.properties 
    inflating: spring-pet-clinic-eks/src/main/resources/banner.txt 
    inflating: spring-pet-clinic-eks/src/main/resources/application-mysql.properties 
     creating: spring-pet-clinic-eks/src/main/wro/
    inflating: spring-pet-clinic-eks/src/main/wro/wro.properties 
    inflating: spring-pet-clinic-eks/src/main/wro/wro.xml 
     creating: spring-pet-clinic-eks/src/main/less/
    inflating: spring-pet-clinic-eks/src/main/less/typography.less 
    inflating: spring-pet-clinic-eks/src/main/less/responsive.less 
    inflating: spring-pet-clinic-eks/src/main/less/petclinic.less 
    inflating: spring-pet-clinic-eks/src/main/less/header.less 
     creating: spring-pet-clinic-eks/src/test/
     creating: spring-pet-clinic-eks/src/test/java/
     creating: spring-pet-clinic-eks/src/test/java/org/
     creating: spring-pet-clinic-eks/src/test/java/org/springframework/
     creating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/
     creating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/
     creating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/owner/
    inflating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/owner/PetTypeFormatterTests.java 
    inflating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/owner/VisitControllerTests.java 
    inflating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/owner/OwnerControllerTests.java 
    inflating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/owner/PetControllerTests.java 
     creating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/service/
    inflating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/service/ClinicServiceTests.java 
    inflating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/service/EntityUtils.java 
     creating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/vet/
    inflating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/vet/VetTests.java 
    inflating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/vet/VetControllerTests.java 
     creating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/model/
    inflating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/model/ValidatorTests.java 
     creating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/system/
    inflating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/system/CrashControllerTests.java 
    inflating: spring-pet-clinic-eks/src/test/java/org/springframework/samples/petclinic/PetclinicIntegrationTests.java 
     creating: spring-pet-clinic-eks/src/test/jmeter/
    inflating: spring-pet-clinic-eks/src/test/jmeter/petclinic_test_plan.jmx 
     creating: spring-pet-clinic-eks/src/checkstyle/
    inflating: spring-pet-clinic-eks/src/checkstyle/nohttp-checkstyle-suppressions.xml 
    inflating: spring-pet-clinic-eks/src/checkstyle/nohttp-checkstyle.xml 
    inflating: spring-pet-clinic-eks/mvnw.cmd 
     creating: spring-pet-clinic-eks/.mvn/
     creating: spring-pet-clinic-eks/.mvn/wrapper/
    inflating: spring-pet-clinic-eks/.mvn/wrapper/maven-wrapper.properties 
    inflating: spring-pet-clinic-eks/.mvn/wrapper/MavenWrapperDownloader.java 
    inflating: spring-pet-clinic-eks/.mvn/wrapper/maven-wrapper.jar 
    inflating: spring-pet-clinic-eks/docker-compose.yml 
    inflating: spring-pet-clinic-eks/pom.xml 
    inflating: spring-pet-clinic-eks/.travis.yml 
    inflating: spring-pet-clinic-eks/tes1 
     creating: spring-pet-clinic-eks/.tanzu/
    inflating: spring-pet-clinic-eks/.tanzu/tanzu_develop.py 
    inflating: spring-pet-clinic-eks/readme.md 
    inflating: spring-pet-clinic-eks/.editorconfig 
  replace spring-pet-clinic-eks/README.md? [y]es, [n]o, [A]ll, [N]one, [r]ename: y
    inflating: spring-pet-clinic-eks/README.md 
     creating: spring-pet-clinic-eks/.vscode/
    inflating: spring-pet-clinic-eks/.vscode/launch.json 
    inflating: spring-pet-clinic-eks/.gitignore 
     creating: spring-pet-clinic-eks/config/
    inflating: spring-pet-clinic-eks/config/workload.yaml 
    inflating: spring-pet-clinic-eks/Tiltfile 
    inflating: spring-pet-clinic-eks/accelerator-log.md 
  Downloads % cd spring-pet-clinic-eks
  spring-pet-clinic-eks % ls
  README.md       Tiltfile        accelerator-log.md  config          docker-compose.yml  mvnw            mvnw.cmd        pom.xml         src         tes1
  spring-pet-clinic-eks % git init -b main
  Initialized empty Git repository in /Users/vdesikan/Downloads/spring-pet-clinic-eks/.git/
  spring-pet-clinic-eks % git add .
  spring-pet-clinic-eks % git commit -m "First commit"
  [main (root-commit) a3592d4] First commit
   103 files changed, 14853 insertions(+)
   create mode 100644 .editorconfig
   create mode 100644 .gitignore
   create mode 100644 .mvn/wrapper/MavenWrapperDownloader.java
   create mode 100755 .mvn/wrapper/maven-wrapper.jar
   create mode 100755 .mvn/wrapper/maven-wrapper.properties
   create mode 100644 .tanzu/tanzu_develop.py
   create mode 100644 .travis.yml
   create mode 100644 .vscode/launch.json
   create mode 100644 README.md
   create mode 100644 Tiltfile
   create mode 100644 accelerator-log.md
   create mode 100644 config/workload.yaml
   create mode 100644 docker-compose.yml
   create mode 100755 mvnw
   create mode 100644 mvnw.cmd
   create mode 100644 pom.xml
   create mode 100644 src/checkstyle/nohttp-checkstyle-suppressions.xml
   create mode 100644 src/checkstyle/nohttp-checkstyle.xml
   create mode 100644 src/main/java/org/springframework/samples/petclinic/PetClinicApplication.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/model/BaseEntity.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/model/NamedEntity.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/model/Person.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/model/package-info.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/owner/Owner.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/owner/OwnerRepository.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/owner/Pet.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/owner/PetController.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/owner/PetRepository.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/owner/PetType.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/owner/PetTypeFormatter.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/owner/PetValidator.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/owner/VisitController.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/system/CacheConfiguration.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/system/CrashController.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/system/WelcomeController.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/vet/Specialty.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/vet/Vet.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/vet/VetController.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/vet/VetRepository.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/vet/Vets.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/visit/Visit.java
   create mode 100644 src/main/java/org/springframework/samples/petclinic/visit/VisitRepository.java
   create mode 100644 src/main/less/header.less
   create mode 100644 src/main/less/petclinic.less
   create mode 100644 src/main/less/responsive.less
   create mode 100644 src/main/less/typography.less
   create mode 100644 src/main/resources/application-mysql.properties
   create mode 100644 src/main/resources/application.properties
   create mode 100644 src/main/resources/banner.txt
   create mode 100644 src/main/resources/db/h2/data.sql
   create mode 100644 src/main/resources/db/h2/schema.sql
   create mode 100644 src/main/resources/db/hsqldb/data.sql
   create mode 100644 src/main/resources/db/hsqldb/schema.sql
   create mode 100644 src/main/resources/db/mysql/data.sql
   create mode 100644 src/main/resources/db/mysql/petclinic_db_setup_mysql.txt
   create mode 100644 src/main/resources/db/mysql/schema.sql
   create mode 100644 src/main/resources/db/mysql/user.sql
   create mode 100644 src/main/resources/messages/messages.properties
   create mode 100644 src/main/resources/messages/messages_de.properties
   create mode 100644 src/main/resources/messages/messages_en.properties
   create mode 100644 src/main/resources/messages/messages_es.properties
   create mode 100644 src/main/resources/static/resources/fonts/montserrat-webfont.eot
   create mode 100644 src/main/resources/static/resources/fonts/montserrat-webfont.svg
   create mode 100644 src/main/resources/static/resources/fonts/montserrat-webfont.ttf
   create mode 100644 src/main/resources/static/resources/fonts/montserrat-webfont.woff
   create mode 100644 src/main/resources/static/resources/fonts/varela_round-webfont.eot
   create mode 100644 src/main/resources/static/resources/fonts/varela_round-webfont.svg
   create mode 100644 src/main/resources/static/resources/fonts/varela_round-webfont.ttf
   create mode 100644 src/main/resources/static/resources/fonts/varela_round-webfont.woff
   create mode 100644 src/main/resources/static/resources/images/favicon.png
   create mode 100644 src/main/resources/static/resources/images/pets.png
   create mode 100644 src/main/resources/static/resources/images/platform-bg.png
   create mode 100644 src/main/resources/static/resources/images/spring-logo-dataflow-mobile.png
   create mode 100644 src/main/resources/static/resources/images/spring-logo-dataflow.png
   create mode 100644 src/main/resources/static/resources/images/spring-pivotal-logo.png
   create mode 100644 src/main/resources/templates/error.html
   create mode 100644 src/main/resources/templates/fragments/inputField.html
   create mode 100644 src/main/resources/templates/fragments/layout.html
   create mode 100644 src/main/resources/templates/fragments/selectField.html
   create mode 100644 src/main/resources/templates/owners/createOrUpdateOwnerForm.html
   create mode 100644 src/main/resources/templates/owners/findOwners.html
   create mode 100644 src/main/resources/templates/owners/ownerDetails.html
   create mode 100644 src/main/resources/templates/owners/ownersList.html
   create mode 100644 src/main/resources/templates/pets/createOrUpdatePetForm.html
   create mode 100644 src/main/resources/templates/pets/createOrUpdateVisitForm.html
   create mode 100644 src/main/resources/templates/vets/vetList.html
   create mode 100644 src/main/resources/templates/welcome.html
   create mode 100644 src/main/wro/wro.properties
   create mode 100644 src/main/wro/wro.xml
   create mode 100644 src/test/java/org/springframework/samples/petclinic/PetclinicIntegrationTests.java
   create mode 100644 src/test/java/org/springframework/samples/petclinic/model/ValidatorTests.java
   create mode 100644 src/test/java/org/springframework/samples/petclinic/owner/OwnerControllerTests.java
   create mode 100644 src/test/java/org/springframework/samples/petclinic/owner/PetControllerTests.java
   create mode 100644 src/test/java/org/springframework/samples/petclinic/owner/PetTypeFormatterTests.java
   create mode 100644 src/test/java/org/springframework/samples/petclinic/owner/VisitControllerTests.java
   create mode 100644 src/test/java/org/springframework/samples/petclinic/service/ClinicServiceTests.java
   create mode 100644 src/test/java/org/springframework/samples/petclinic/service/EntityUtils.java
   create mode 100644 src/test/java/org/springframework/samples/petclinic/system/CrashControllerTests.java
   create mode 100644 src/test/java/org/springframework/samples/petclinic/vet/VetControllerTests.java
   create mode 100644 src/test/java/org/springframework/samples/petclinic/vet/VetTests.java
   create mode 100644 src/test/jmeter/petclinic_test_plan.jmx
   create mode 100644 tes1
  spring-pet-clinic-eks % git remote add origin https://github.com/vdesikanvmware/spring-pet-clinic-eks.git
  spring-pet-clinic-eks % git branch -M main
  spring-pet-clinic-eks % git remote set-url origin ssh://git@github.com/vdesikanvmware/spring-pet-clinic-eks.git
  spring-pet-clinic-eks % git push -u origin main                                                               
  Warning: Permanently added the RSA host key for IP address '13.234.210.38' to the list of known hosts.
  Enumerating objects: 149, done.
  Counting objects: 100% (149/149), done.
  Delta compression using up to 12 threads
  Compressing objects: 100% (133/133), done.
  Writing objects: 100% (149/149), 404.15 KiB | 1.99 MiB/s, done.
  Total 149 (delta 24), reused 0 (delta 0), pack-reused 0
  remote: Resolving deltas: 100% (24/24), done.
  To ssh://github.com/vdesikanvmware/spring-pet-clinic-eks.git
   * [new branch]      main -> main
  Branch 'main' set up to track remote branch 'main' from 'origin'.
  ```
3. Use Tanzu Build Service to create an image for the app from the new git repo added using App Accelerator Template.
  1. Create a Tanzu Application Platform service account and a secret for Tanzu Build Service. Patch the secret to the service account's Image Pull secret.
      ```
      tap-install % more tap-sa.yaml
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: tap-service-account
        namespace: tap-install
      ---
      kind: ClusterRole
      apiVersion: rbac.authorization.k8s.io/v1
      metadata:
        name: cluster-admin-cluster-role
      rules:
      - apiGroups: ["*"]
        resources: ["*"]
        verbs: ["*"]
      ---
      kind: ClusterRoleBinding
      apiVersion: rbac.authorization.k8s.io/v1
      metadata:
        name: cluster-admin-cluster-role-binding
      subjects:
      - kind: ServiceAccount
        name: tap-service-account
        namespace: tap-install
      roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: ClusterRole
        name: cluster-admin-cluster-role
      tap-install % kubectl apply -f tap-sa.yaml
      serviceaccount/tap-service-account created
      clusterrole.rbac.authorization.k8s.io/cluster-admin-cluster-role unchanged
      clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-cluster-role-binding configured
      tap-install % kubectl create secret docker-registry tbs-secret -n tap-install --docker-server='dev.registry.pivotal.io' --docker-username=$USRFULL --docker-password=$PASS
      secret/tbs-secret created
      tap-install % kubectl patch serviceaccount default -p "{\"imagePullSecrets\": [{\"name\": \"tbs-secret\"}]}" -n tap-install
      serviceaccount/default patched
      ```
  2. Use Tanzu Build Service to create an image for the git-repo created with Application Accelerator. Specify the `dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks container registry where you can push the image.
    ```
    tap-install % more image.yaml
    apiVersion: kpack.io/v1alpha1
    tap-install % more tap-sa.yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: tap-service-account
      namespace: tap-install
    ---
    kind: ClusterRole
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: cluster-admin-cluster-role
    rules:
    - apiGroups: ["*"]
      resources: ["*"]
      verbs: ["*"]
    ---
    kind: ClusterRoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: cluster-admin-cluster-role-binding
    subjects:
    - kind: ServiceAccount
      name: tap-service-account
      namespace: tap-install
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin-cluster-role
    tap-install % kubectl apply -f tap-sa.yaml
    serviceaccount/tap-service-account created
    clusterrole.rbac.authorization.k8s.io/cluster-admin-cluster-role unchanged
    clusterrolebinding.rbac.authorization.k8s.io/cluster-admin-cluster-role-binding configured
    tap-install % kubectl create secret docker-registry tbs-secret -n tap-install --docker-server='dev.registry.pivotal.io' --docker-username=$USRFULL --docker-password=$PASS
    secret/tbs-secret created
    tap-install % kubectl patch serviceaccount default -p "{\"imagePullSecrets\": [{\"name\": \"tbs-secret\"}]}" -n tap-install
    serviceaccount/default patched
    tap-install % kubectl patch serviceaccount default -p "{\"secrets\": [{\"name\": \"tbs-secret\"}]}" -n tap-install
    serviceaccount/default patched
    ```
2. Use Tanzu Build Service to create an image for the git-repo created with Application Accelerator. Specify a container registry where you can push the image.
  ```
  tap-install % more image.yaml
  apiVersion: kpack.io/v1alpha1
  kind: Image
  metadata:
    name: spring-petclinic-image
  spec:
    tag: dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks
    serviceAccount: default
    builder:
      kind: ClusterBuilder
      name: default
    source:
      git:
        url: https://github.com/vdesikanvmware/spring-pet-clinic-eks
        revision: main
  tap-install % kubectl apply -f image.yaml -n tap-install  
  image.kpack.io/spring-petclinic-image created
  tap-install % kp image list
  Error: no images found
  tap-install % kp image list -n tap-install
  NAME                      READY      LATEST REASON    LATEST IMAGE    NAMESPACE
  spring-petclinic-image    Unknown    CONFIG                           tap-install
   
   
  tap-install % kp build list -n tap-install
  BUILD    STATUS      IMAGE    REASON
  1        BUILDING             CONFIG
   
   
  tap-install % kp image list -n tap-install
  NAME                      READY      LATEST REASON    LATEST IMAGE    NAMESPACE
  spring-petclinic-image    Unknown    CONFIG                           tap-install
   
   
  tap-install % kp build list -n tap-install
  BUILD    STATUS      IMAGE    REASON
  1        BUILDING             CONFIG
   
   
  tap-install % kp build status spring-petclinic-image -n tap-install
  Image:            --
  Status:           BUILDING
  Reason:           CONFIG
                    resources: {}
                    - source: {}
                    + source:
                    +   git:
                    +     revision: a3592d4d1a47ae3e6f6e0143d0369a9476da7620
                    +     url: https://github.com/vdesikanvmware/spring-pet-clinic-eks
  Status Reason:    PodInitializing
   
   
  Started:     2021-08-24 10:09:27
  Finished:    --
   
   
  Pod Name:    spring-petclinic-image-build-1-vl7j5-build-pod
   
   
  Builder:      dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs/default@sha256:c3acd9780a055d9657f702426460d2fbd0b9dcdac3facbaec894646a580d6f6d
  Run Image:    --
   
   
  Source:      GitUrl
  Url:         https://github.com/vdesikanvmware/spring-pet-clinic-eks
  Revision:    a3592d4d1a47ae3e6f6e0143d0369a9476da7620
   
   
  BUILDPACK ID    BUILDPACK VERSION    HOMEPAGE
   
   
  tap-install % kp image list -n tap-install                        
  NAME                      READY      LATEST REASON    LATEST IMAGE    NAMESPACE
  spring-petclinic-image    Unknown    CONFIG                           tap-install
   
   
  tap-install % kp build list -n tap-install                        
  BUILD    STATUS      IMAGE    REASON
  1        BUILDING             CONFIG
   
   
  tap-install % kp image list -n tap-install
  NAME                      READY    LATEST REASON    LATEST IMAGE                                                                                                                                            NAMESPACE
  spring-petclinic-image    True     CONFIG           dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks@sha256:be889cf313016eb4fc168556493c2b1672c8e2af725e33696bf461b8212f9872    tap-install
   
   
  tap-install % kp build list -n tap-install
  BUILD    STATUS     IMAGE                                                                                                                                                   REASON
  1        SUCCESS    dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks@sha256:be889cf313016eb4fc168556493c2b1672c8e2af725e33696bf461b8212f9872    CONFIG
   
   
  tap-install % kp build status spring-petclinic-image -n tap-install
  Image:     dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks@sha256:be889cf313016eb4fc168556493c2b1672c8e2af725e33696bf461b8212f9872
  Status:    SUCCESS
  Reason:    CONFIG
             resources: {}
             - source: {}
             + source:
             +   git:
             +     revision: a3592d4d1a47ae3e6f6e0143d0369a9476da7620
             +     url: https://github.com/vdesikanvmware/spring-pet-clinic-eks
   
   
  Started:     2021-08-24 10:09:27
  Finished:    2021-08-24 10:10:58
   
   
  Pod Name:    spring-petclinic-image-build-1-vl7j5-build-pod
   
   
  Builder:      dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs/default@sha256:c3acd9780a055d9657f702426460d2fbd0b9dcdac3facbaec894646a580d6f6d
  Run Image:    dev.registry.pivotal.io/tanzu-advanced-edition/beta1/tbs/run@sha256:ae65c51e7fb215fa3ed3ffddf9e438a3f9c571e591db71dec7d903ce5bc9bf92
   
   
  Source:      GitUrl
  Url:         https://github.com/vdesikanvmware/spring-pet-clinic-eks
  Revision:    a3592d4d1a47ae3e6f6e0143d0369a9476da7620
   
   
  BUILDPACK ID                           BUILDPACK VERSION    HOMEPAGE
  paketo-buildpacks/ca-certificates      2.3.2                https://github.com/paketo-buildpacks/ca-certificates
  paketo-buildpacks/bellsoft-liberica    8.2.0                https://github.com/paketo-buildpacks/bellsoft-liberica
  paketo-buildpacks/maven                5.3.3                https://github.com/paketo-buildpacks/maven
  paketo-buildpacks/executable-jar       5.1.2                https://github.com/paketo-buildpacks/executable-jar
  paketo-buildpacks/apache-tomcat        6.0.0                https://github.com/paketo-buildpacks/apache-tomcat
  paketo-buildpacks/dist-zip             4.1.2                https://github.com/paketo-buildpacks/dist-zip
  paketo-buildpacks/spring-boot          4.4.2                https://github.com/paketo-buildpacks/spring-boot
  ```
3. Deploy the image you generated as a service with Cloud Native Runtimes. Deploy the image in the namespace where Application Live View is running with the labels `tanzu.app.live.view=true` and `tanzu.app.live.view.application.name=<app_name>`. Add the appropriate DNS entries using `/etc/hosts`.
  ```
  tap-install % more kapp-deploy-spring-petclinic.yaml
      apiVersion: kappctrl.k14s.io/v1alpha1
      kind: App
      metadata:
        name: spring-petclinic
      spec:
        serviceAccountName: tap-service-account
        fetch:
          - inline:
              paths:
                manifest.yml: |
                  ---
                  apiVersion: kapp.k14s.io/v1alpha1
                  kind: Config
                  rebaseRules:
                    - path: [metadata, annotations, serving.knative.dev/creator]
                      type: copy
                      sources: [new, existing]
                      resourceMatchers: &matchers
                        - apiVersionKindMatcher: {apiVersion: serving.knative.dev/v1, kind: Service}
                    - path: [metadata, annotations, serving.knative.dev/lastModifier]
                      type: copy
                      sources: [new, existing]
                      resourceMatchers: *matchers
                  ---
                  apiVersion: serving.knative.dev/v1
                  kind: Service
                  metadata:
                    name: petclinic
                  spec:
                    template:
                      metadata:
                        annotations:
                          client.knative.dev/user-image: ""
                        labels:
                          tanzu.app.live.view: "true"
                          tanzu.app.live.view.application.name: "spring-petclinic"
                      spec:
                        containers:
                        - image: dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks@sha256:be889cf313016eb4fc168556493c2b1672c8e2af725e33696bf461b8212f9872
                          securityContext:
                            runAsUser: 1000
        template:
          - ytt: {}
        deploy:
          - kapp: {}
  tap-install % kubectl apply -f kapp-deploy-spring-petclinic.yaml -n tap-install
  app.kappctrl.k14s.io/spring-petclinic created
  tap-install % kubectl get pods -n tap-install
  NAME                                                    READY   STATUS              RESTARTS   AGE
  application-live-view-connector-57cd7c6c6-pv8dk         1/1     Running             0          17h
  application-live-view-crd-controller-5b659f8f57-7zz25   1/1     Running             0          17h
  application-live-view-server-79bd874566-qtcbc           1/1     Running             0          17h
  petclinic-00001-deployment-5f7b86665f-lqhfs             0/2     ContainerCreating   0          10s
  spring-petclinic-image-build-1-vl7j5-build-pod          0/1     Completed           0          7m56s
  tap-install % kubectl get pods -n tap-install
  NAME                                                    READY   STATUS      RESTARTS   AGE
  application-live-view-connector-57cd7c6c6-pv8dk         1/1     Running     0          17h
  application-live-view-crd-controller-5b659f8f57-7zz25   1/1     Running     0          17h
  application-live-view-server-79bd874566-qtcbc           1/1     Running     0          17h
  petclinic-00001-deployment-5f7b86665f-lqhfs             2/2     Running     0          23s
  spring-petclinic-image-build-1-vl7j5-build-pod          0/1     Completed   0          8m9s
  tap-install % kubectl get service -n tap-install
  NAME                         TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)                                      AGE
  application-live-view-5112   LoadBalancer   10.100.176.238   ac229c2c3af2f48be896688d5f176c16-1335129677.us-east-2.elb.amazonaws.com   5112:32387/TCP                               17h
  application-live-view-7000   ClusterIP      10.100.29.252    <none>                                                                    7000/TCP                                     17h
  petclinic                    ExternalName   <none>           envoy.contour-internal.svc.cluster.local                                  80/TCP                                       19s
  petclinic-00001              ClusterIP      10.100.254.12    <none>                                                                    80/TCP                                       37s
  petclinic-00001-private      ClusterIP      10.100.219.255   <none>                                                                    80/TCP,9090/TCP,9091/TCP,8022/TCP,8012/TCP   37s
    tap-install % kubectl get service -A           
  NAMESPACE                NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)                                      AGE
  accelerator-system       acc-engine                           ClusterIP      10.100.253.164   <none>                                                                    80/TCP                                       17h
  accelerator-system       acc-ui-server                        LoadBalancer   10.100.74.90     a3b48dd9f15f746a48b503aee558bd96-309532596.us-east-2.elb.amazonaws.com    80:31689/TCP                                 17h
  build-service            cert-injection-webhook               ClusterIP      10.100.98.116    <none>                                                                    443/TCP                                      16h
  contour-external         contour                              ClusterIP      10.100.10.20     <none>                                                                    8001/TCP                                     17h
  contour-external         envoy                                LoadBalancer   10.100.29.235    afc0cc5526ef74617a20f89f27433b6f-609822777.us-east-2.elb.amazonaws.com    80:31346/TCP,443:32120/TCP                   17h
  contour-internal         contour                              ClusterIP      10.100.247.212   <none>                                                                    8001/TCP                                     17h
  contour-internal         envoy                                ClusterIP      10.100.3.195     <none>                                                                    80/TCP,443/TCP                               17h
  default                  kubernetes                           ClusterIP      10.100.0.1       <none>                                                                    443/TCP                                      4d18h
  flux-system              notification-controller              ClusterIP      10.100.153.120   <none>                                                                    80/TCP                                       17h
  flux-system              source-controller                    ClusterIP      10.100.214.3     <none>                                                                    80/TCP                                       17h
  flux-system              webhook-receiver                     ClusterIP      10.100.54.115    <none>                                                                    80/TCP                                       17h
  kapp-controller          packaging-api                        ClusterIP      10.100.66.222    <none>                                                                    443/TCP                                      17h
  knative-discovery        webhook                              ClusterIP      10.100.204.141   <none>                                                                    9090/TCP,8008/TCP,443/TCP                    17h
  knative-eventing         broker-filter                        ClusterIP      10.100.200.214   <none>                                                                    80/TCP,9092/TCP                              17h
  knative-eventing         broker-ingress                       ClusterIP      10.100.50.73     <none>                                                                    80/TCP,9092/TCP                              17h
  knative-eventing         eventing-webhook                     ClusterIP      10.100.188.40    <none>                                                                    443/TCP                                      17h
  knative-eventing         imc-dispatcher                       ClusterIP      10.100.54.100    <none>                                                                    80/TCP                                       17h
  knative-eventing         inmemorychannel-webhook              ClusterIP      10.100.72.121    <none>                                                                    443/TCP                                      17h
  knative-serving          activator-service                    ClusterIP      10.100.114.196   <none>                                                                    9090/TCP,8008/TCP,80/TCP,81/TCP              17h
  knative-serving          autoscaler                           ClusterIP      10.100.2.22      <none>                                                                    9090/TCP,8008/TCP,8080/TCP                   17h
  knative-serving          autoscaler-bucket-00-of-01           ClusterIP      10.100.3.124     <none>                                                                    8080/TCP                                     17h
  knative-serving          controller                           ClusterIP      10.100.40.142    <none>                                                                    9090/TCP,8008/TCP                            17h
  knative-serving          net-certmanager-webhook              ClusterIP      10.100.168.189   <none>                                                                    9090/TCP,8008/TCP,443/TCP                    17h
  knative-serving          networking-certmanager               ClusterIP      10.100.168.29    <none>                                                                    9090/TCP,8008/TCP                            17h
  knative-serving          webhook                              ClusterIP      10.100.228.52    <none>                                                                    9090/TCP,8008/TCP,443/TCP                    17h
  knative-sources          rabbitmq-controller                  ClusterIP      10.100.150.219   <none>                                                                    443/TCP                                      17h
  knative-sources          rabbitmq-webhook                     ClusterIP      10.100.86.29     <none>                                                                    443/TCP                                      17h
  kpack                    kpack-webhook                        ClusterIP      10.100.67.127    <none>                                                                    443/TCP                                      16h
  kube-system              kube-dns                             ClusterIP      10.100.0.10      <none>                                                                    53/UDP,53/TCP                                4d18h
  stacks-operator-system   controller-manager-metrics-service   ClusterIP      10.100.118.226   <none>                                                                    8443/TCP                                     16h
  tap-install              application-live-view-5112           LoadBalancer   10.100.176.238   ac229c2c3af2f48be896688d5f176c16-1335129677.us-east-2.elb.amazonaws.com   5112:32387/TCP                               17h
  tap-install              application-live-view-7000           ClusterIP      10.100.29.252    <none>                                                                    7000/TCP                                     17h
  tap-install              petclinic                            ExternalName   <none>           envoy.contour-internal.svc.cluster.local                                  80/TCP                                       35s
  tap-install              petclinic-00001                      ClusterIP      10.100.254.12    <none>                                                                    80/TCP                                       53s
  tap-install              petclinic-00001-private              ClusterIP      10.100.219.255   <none>                                                                    80/TCP,9090/TCP,9091/TCP,8022/TCP,8012/TCP   53s
  vmware-sources           webhook                              ClusterIP      10.100.192.82 <none>                                                                    443/TCP                                      17h
  tap-install % kubectl get ksvc -n tap-install
  NAME        URL                                        LATESTCREATED     LATESTREADY       READY   REASON
  petclinic   http://petclinic.tap-install.example.com   petclinic-00001   petclinic-00001   True   
  tap-install % ping afc0cc5526ef74617a20f89f27433b6f-609822777.us-east-2.elb.amazonaws.com 
  PING afc0cc5526ef74617a20f89f27433b6f-609822777.us-east-2.elb.amazonaws.com (3.19.166.204): 56 data bytes
  Request timeout for icmp_seq 0
  Request timeout for icmp_seq 1
  ^C
  --- afc0cc5526ef74617a20f89f27433b6f-609822777.us-east-2.elb.amazonaws.com ping statistics ---
  3 packets transmitted, 0 packets received, 100.0% packet loss
  tap-install % sudo vi /etc/hosts                                                        
  tap-install % more /etc/hosts
  ##
  # Host Database
  #
  # localhost is used to configure the loopback interface
  # when the system is booting.  Do not change this entry.
  ##
  127.0.0.1       localhost
  255.255.255.255 broadcasthost
  ::1             localhost
  # Added by Docker Desktop
  # To allow the same kube context to work on the host and the container:
  127.0.0.1 kubernetes.docker.internal
  3.19.166.204 petclinic.tap-install.example.com
  # End of section
  tap-install %
  ```
4. Verify that you can access the Spring Pet Clinic app. Ensure that Application Live View is also displaying the Spring Pet Clinic app.
  ![Screenshot of page on Tanzu Network from where you download Tanzu Application Platform packages shows the EULA warning](./images/image2021-8-24_16-6-22.png)
  ![Screenshot of page on Tanzu Network from where you download Tanzu Application Platform packages shows the EULA warning](./images/image2021-8-24_10-44-38.png)
5. Make some code changes in the git repo and commit the change. Verify that the new build is created automatically and the new image is generated.
  ```
  spring-pet-clinic-eks % ls
  README.md       Tiltfile        accelerator-log.md  config          docker-compose.yml  mvnw            mvnw.cmd        pom.xml         src         tes1
  spring-pet-clinic-eks % cd src
  src % ls
  checkstyle  main        test
  src % cd main
  main % ls
  java        less        resources   wro
  main % cd resources
  resources % ls
  application-mysql.properties    application.properties      banner.txt          db              messages            static              templates
  resources % cd templates
  templates % ls
  error.html  fragments   owners      pets        vets        welcome.html
  templates % cd vets
  vets % ls
  vetList.html
  vets % vi vetList.html
  vets % more vetList.html
  <!DOCTYPE html>
   
   
  <html xmlns:th="https://www.thymeleaf.org"
    th:replace="~{fragments/layout :: layout (~{::body},'vets')}">
   
   
  <body>
   
   
    <h2>Veterinarians at VMware</h2>
   
   
    <table id="vets" class="table table-striped">
      <thead>
        <tr>
          <th>Name</th>
          <th>Specialties</th>
        </tr>
      </thead>
      <tbody>
        <tr th:each="vet : ${vets.vetList}">
          <td th:text="${vet.firstName + ' ' + vet.lastName}"></td>
          <td><span th:each="specialty : ${vet.specialties}"
            th:text="${specialty.name + ' '}" /> <span
            th:if="${vet.nrOfSpecialties == 0}">none</span></td>
        </tr>
      </tbody>
    </table>
  </body>
  </html>
  vets % vi vetList.html
  vets % more vetList.html
  <!DOCTYPE html>
   
   
  <html xmlns:th="https://www.thymeleaf.org"
    th:replace="~{fragments/layout :: layout (~{::body},'vets')}">
   
   
  <body>
   
   
    <h2>Veterinarians at VMware - Updated by MAPBU DAP Delivery Team</h2>
   
   
    <table id="vets" class="table table-striped">
      <thead>
        <tr>
          <th>Name</th>
          <th>Specialties</th>
        </tr>
      </thead>
      <tbody>
        <tr th:each="vet : ${vets.vetList}">
          <td th:text="${vet.firstName + ' ' + vet.lastName}"></td>
          <td><span th:each="specialty : ${vet.specialties}"
            th:text="${specialty.name + ' '}" /> <span
            th:if="${vet.nrOfSpecialties == 0}">none</span></td>
        </tr>
      </tbody>
    </table>
  </body>
  </html>
  vets % cd ../../../
  main % cd ../../
  spring-pet-clinic-eks % git add .
  spring-pet-clinic-eks %  git commit -m "Updated Veterinarians Page"
  [main e672afd] Updated Veterinarians Page
   1 file changed, 1 insertion(+), 1 deletion(-)
  spring-pet-clinic-eks % git push -u origin main
  Enumerating objects: 15, done.
  Counting objects: 100% (15/15), done.
  Delta compression using up to 12 threads
  Compressing objects: 100% (7/7), done.
  Writing objects: 100% (8/8), 777 bytes | 777.00 KiB/s, done.
  Total 8 (delta 4), reused 0 (delta 0), pack-reused 0
  remote: Resolving deltas: 100% (4/4), completed with 4 local objects.
  To ssh://github.com/vdesikanvmware/spring-pet-clinic-eks.git
     a3592d4..e672afd  main -> main
  Branch 'main' set up to track remote branch 'main' from 'origin'.
  spring-pet-clinic-eks % kp build list -n tap-install
  BUILD    STATUS      IMAGE                                                                                                                                                   REASON
  1        SUCCESS     dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks@sha256:be889cf313016eb4fc168556493c2b1672c8e2af725e33696bf461b8212f9872    CONFIG
  2        BUILDING                                                                                                                                                            COMMIT
   
  spring-pet-clinic-eks % kp image list -n tap-install
  NAME                      READY      LATEST REASON    LATEST IMAGE                                                                                                                                            NAMESPACE
  spring-petclinic-image    Unknown    COMMIT           dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks@sha256:be889cf313016eb4fc168556493c2b1672c8e2af725e33696bf461b8212f9872    tap-install
   
   
  spring-pet-clinic-eks % kp build list -n tap-install
  BUILD    STATUS      IMAGE                                                                                                                                                   REASON
  1        SUCCESS     dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks@sha256:be889cf313016eb4fc168556493c2b1672c8e2af725e33696bf461b8212f9872    CONFIG
  2        BUILDING                                                                                                                                                            COMMIT
   
   
  spring-pet-clinic-eks % kp image list -n tap-install
  NAME                      READY    LATEST REASON    LATEST IMAGE                                                                                                                                            NAMESPACE
  spring-petclinic-image    True     COMMIT           dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks@sha256:0b627060147d7d80d8aae5a650a70ebca67bbb73001420580b2effa77c4b90cd    tap-install
   
   
  spring-pet-clinic-eks % kp build list -n tap-install
  BUILD    STATUS     IMAGE                                                                                                                                                   REASON
  1        SUCCESS    dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks@sha256:be889cf313016eb4fc168556493c2b1672c8e2af725e33696bf461b8212f9872    CONFIG
  2        SUCCESS    dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks@sha256:0b627060147d7d80d8aae5a650a70ebca67bbb73001420580b2effa77c4b90cd    COMMIT
  ```
6. Update the Cloud Native Runtimes service with the new image.
  ```
  tap-install % more kapp-deploy-spring-petclinic.yaml
      apiVersion: kappctrl.k14s.io/v1alpha1
      kind: App
      metadata:
        name: spring-petclinic
      spec:
        serviceAccountName: tap-service-account
        fetch:
          - inline:
              paths:
                manifest.yml: |
                  ---
                  apiVersion: kapp.k14s.io/v1alpha1
                  kind: Config
                  rebaseRules:
                    - path: [metadata, annotations, serving.knative.dev/creator]
                      type: copy
                      sources: [new, existing]
                      resourceMatchers: &matchers
                        - apiVersionKindMatcher: {apiVersion: serving.knative.dev/v1, kind: Service}
                    - path: [metadata, annotations, serving.knative.dev/lastModifier]
                      type: copy
                      sources: [new, existing]
                      resourceMatchers: *matchers
                  ---
                  apiVersion: serving.knative.dev/v1
                  kind: Service
                  metadata:
                    name: petclinic
                  spec:
                    template:
                      metadata:
                        annotations:
                          client.knative.dev/user-image: ""
                        labels:
                          tanzu.app.live.view: "true"
                          tanzu.app.live.view.application.name: "spring-petclinic"
                      spec:
                        containers:
                        - image: dev.registry.pivotal.io/tanzu-advanced-edition/vdesikan/spring-petclinic-eks@sha256:0b627060147d7d80d8aae5a650a70ebca67bbb73001420580b2effa77c4b90cd
                          securityContext:
                            runAsUser: 1000
        template:
          - ytt: {}
        deploy:
          - kapp: {}
  tap-install % kubectl apply -f kapp-deploy-spring-petclinic.yaml -n tap-install
  app.kappctrl.k14s.io/spring-petclinic configured
  tap-install % kubectl get pods -n tap-install
  NAME                                                    READY   STATUS      RESTARTS   AGE
  application-live-view-crd-controller-5b659f8f57-7zz25   1/1     Running     0          23h
  application-live-view-server-79bd874566-qtcbc           1/1     Running     0          23h
  petclinic-00002-deployment-9bcf8bdc9-mrvf7              1/2     Running     0          15s
  spring-petclinic-image-build-1-vl7j5-build-pod          0/1     Completed   0          5h54m
  spring-petclinic-image-build-2-gthhl-build-pod          0/1     Completed   0          11m
  ```
7. Verify that the code changes are reflected in the app.
  ![Screenshot of page on Tanzu Network from where you download Tanzu Application Platform packages shows the EULA warning](./images/image2021-8-24_16-6-22.png)