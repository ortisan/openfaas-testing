## Configure Kubenetes Cluster

### Install [K3d](https://k3d.io/)

```sh
curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash
```

### Creating cluster:

```sh
k3d cluster create ortisan-cluster
k3d kubeconfig merge ortisan-cluster --kubeconfig-switch-context

kubectl cluster-info
kubectl get pods --all-namespaces
```

## Installing [OpenFaas](https://docs.openfaas.com/)

### Install [arkade](https://github.com/alexellis/arkade) (Devops marketplace)

```sh
curl -SLsf https://dl.get-arkade.dev/ | sudo sh
```

### Install Openfaas with Arkade:

```sh
arkade install Openfaas
```

### Install OpenFaas Cli:

```sh
curl -sLSf https://cli.openfaas.com | sudo sh

faas-cli help
faas-cli version
```

## Configuring OpenFaas:

### Forward the gateway to your machine

```sh
kubectl rollout status -n openfaas deploy/gateway
kubectl port-forward -n openfaas svc/gateway 8080:8080 &
```

### If basic auth is enabled, you can now log into your gateway:

```sh
PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
echo -n $PASSWORD | faas-cli login --username admin --password-stdin
```

You can login with:

User: admin
Password: echo $PASSWORD

### Configuring secret for dockerhub deploying:

Each function in OpenFaas is wrap into container image. The default provider is Dockerhub.

[Kubernetes doc](https://kubernetes.io/docs/concepts/configuration/secret/)

```sh
kubectl create secret generic regcred \
 --from-file=.dockerconfigjson=~/.docker/config.json \
 --type=kubernetes.io/dockerconfigjson
```

or [OpenFaas doc](https://docs.openfaas.com/deployment/kubernetes/)

```sh
export DOCKER_USERNAME=<your_docker_username>
export DOCKER_PASSWORD=<your_docker_password>
export DOCKER_EMAIL=<your_docker_email>

kubectl create secret docker-registry dockerhub \
    -n openfaas-fn \
    --docker-username=$DOCKER_USERNAME \
    --docker-password=$DOCKER_PASSWORD \
    --docker-email=$DOCKER_EMAIL
```

## Deploying on OpenFaas

### Deploy from store:

```sh
faas-cli store deploy figlet
```

### Own function:

#### Create function:

```sh
faas-cli new --lang go hello-openfaas --prefix=tentativafc
mv hello-openfaas.yml stack.yml
```

if you are using specific dockerhub secret, you need to configure this on stack.yml

```yml
secrets:
  - dockerhub
```

#### Deploy function

```sh
faas-cli up
```

#### Invoke function

```sh
faas-cli invoke hello-openfaas
```
