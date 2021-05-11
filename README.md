# OpenShift Service Mesh - Book Info App

This will walk through how to install the bookinfo app described [here](https://istio.io/latest/docs/examples/bookinfo/) with the sample code [here](https://github.com/istio/istio/tree/master/samples/bookinfo).


## Prerequisites
1.  OCP 4.7
2.  `oc` cli logged into the cluster assuming with cluster-admin


## Service Mesh Installation
All docs can be found [here](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.7/html-single/service_mesh/index#installing-ossm).

1.  Install OpenShift Elasticsearch via Operator
2.  Install Red Hat OpenShift Jaeger via Operator
3.  Install Kiali Operator by Red Hat via Operator
4.  Install OpenShift Service Mesh via Operator
5.  Install Service Mesh Control Plane
```shell
oc apply -f ./istio_installation.yaml
```

6.  Verify Installation
```shell
# wait for STATUS == ComponentsReady
oc get smcp -n istio-system
```

7.  Install Service Mesh Member Role which lists the project belonging to the control plane
```shell
oc apply -f ./servicemeshmemberroll-default.yaml
```

8.  If you want to give users access to modify the projects that are part of service mesh, see the section named [1.5.7.3. Creating the Red Hat OpenShift Service Mesh members](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.7/html-single/service_mesh/index#ossm-member-roll-create_installing-ossm)

9.  If you want to create control plane profiles that act as reusable service mesh control plane profiles, then see the section named [1.9.2. Creating control plane profiles](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.7/html-single/service_mesh/index#ossm-control-plane-profiles_deploying-applications-ossm)


## Bookinfo Deployment
The instructions here are how to install the bookinfo app. Docs can be found [here](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.7/html-single/service_mesh/index#ossm-tutorial-bookinfo-overview_deploying-applications-ossm).

1.  Create project
```shell
oc new-project bookinfo
```

2.  Install Bookinfo project
```shell
oc apply -n bookinfo -f https://raw.githubusercontent.com/Maistra/istio/maistra-2.0/samples/bookinfo/platform/kube/bookinfo.yaml
```

3.  Install Ingress Gateway
```shell
oc apply -n bookinfo -f https://raw.githubusercontent.com/Maistra/istio/maistra-2.0/samples/bookinfo/networking/bookinfo-gateway.yaml
```

4.  Set GATEWAY_URL
```shell
export GATEWAY_URL=$(oc -n istio-system get route istio-ingressgateway -o jsonpath='{.spec.host}')
```

5.  Enable Routing Rules
```shell
oc apply -n bookinfo -f https://raw.githubusercontent.com/Maistra/istio/maistra-2.0/samples/bookinfo/networking/destination-rule-all.yaml
```

6.  Verify Endpoint
```shell
# should get back 200
curl -o /dev/null -s -w "%{http_code}\n" http://$GATEWAY_URL/productpage

# also navigate to this URL in your browser
echo "http://$GATEWAY_URL/productpage"
```

7.  Verify Pods
```shell
oc get pods -n bookinfo
```

Output should look like:
```
NAME                             READY   STATUS    RESTARTS   AGE
details-v1-86dfdc4b95-nvrl9      2/2     Running   0          4m9s
productpage-v1-658849bb5-zbmcg   2/2     Running   0          4m8s
ratings-v1-76b8c9cbf9-hwg68      2/2     Running   0          4m9s
reviews-v1-58b8568645-t6wj2      2/2     Running   0          4m9s
reviews-v2-5d8f8b6775-65klj      2/2     Running   0          4m9s
reviews-v3-666b89cfdf-kglbh      2/2     Running   0          4m9s
```
