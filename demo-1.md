## Install helm CLI client on your system
### On MacOS using Homebrew
```sh
$ brew install kubernetes-helm
```
### From script
```sh
$ curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

## Create a service account and a cluster role binding for Tiller
```sh
$ kubectl -n kube-system create sa tiller

$ kubectl create clusterrolebinding tiller-cluster-rule \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:tiller 
```

## Deploy tiller (Helm server) on your cluster 
```sh
$ helm init --skip-refresh --upgrade --service-account tiller
```

## Create OpenFaaS namespaces
```sh
$ kubectl apply -f https://raw.githubusercontent.com/openfaas/faas-netes/master/namespaces.yml
```

## Generate basic authentication credentials secret
```sh
$ password=$(head -c 12 /dev/urandom | shasum | cut -d' ' -f1)

$ kubectl -n openfaas create secret generic basic-auth \
   --from-literal=basic-auth-user=admin \
   --from-literal=basic-auth-password=$password
```

## Install OpenFaaS with Helm
```sh
$ helm repo add openfaas https://openfaas.github.io/faas-netes/

$ helm upgrade openfaas --install openfaas/openfaas \
    --namespace openfaas  \
    --set functionNamespace=openfaas-fn \
    --set serviceType=LoadBalancer \
    --set basic_auth=true \
    --set operator.create=true
```

## Verify the installation
```sh
$ kubectl --namespace=openfaas get deployments -l "release=openfaas, app=openfaas"
```

## List CRD installed on the cluster
```sh
$ kubectl get crd
```

Note: Deploy a function from store using OpenFaaS gateway UI

## List all deployed function using `kubectl`
```sh
$ kubectl get functions -n openfaas-fn
```

## Get a function information
```sh
$ kubectl get function/figlet -n openfaas-fn
```

## Delete a function using `kubectl` 
```sh
$ kubectl delete function/figlet -n openfaas-fn
```

## Deploy a function using `kubectl`
```sh
$ faas-cli generate -f operator-demo.yml | kubectl apply -f - 
```

## Delete OpenFaaS installation
```sh
$ helm delete --purge openfaas
```
