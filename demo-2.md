## Deploy OpenFaaS on docker swarm using deploy script. 
Use `--no-auth` if you want to disable basic authentication

```sh
./deploy_stack.sh --no-auth
```

## Deploy local docker registry.
```sh
docker service rm registry
docker service create --network func_functions \
  --name registry \
  --detach=true -p 5000:5000 registry:latest
```

## Deploy buildkit service
### For using root in the container
```sh
docker rm -f of-buildkit
docker run -d --net func_functions -d --privileged \
--restart always \
--name of-buildkit alexellis2/buildkit:2018-04-17 --addr tcp://0.0.0.0:1234
```

### For Rootless option
```sh
docker rm -f of-buildkit
docker run -d --net func_functions -d --privileged \
--restart always \
--name of-buildkit akihirosuda/buildkit-rootless:20180605 --addr tcp://0.0.0.0:1234
```

## Deploy builder service
```sh
docker rm -f of-builder
export TAG=0.4.1
docker service create --network func_functions --name of-builder openfaas/of-builder:$TAG
```

## Create `github-webhook-secret` secret 
```sh
docker secret create github-webhook-secret <file_path_to_github_secret>
```

## Create `private-key` secret to provide your github app secret
```sh
docker secret create private-key <path/to/private-key.pem>
```
### Update the `private_key_filename` and `github_app_id` in `github.yml`
```yaml
...
   github_app_id: "your_github_app_id"
   private_key_filename: "private_key"
...
```

### Update `gateway_config.yml` file as below

```yaml
environment:
  gateway_url:  http://gateway:8080/
  gateway_public_url: https://cloud.o6s.io/
  audit_url: http://gateway:8080/function/audit-event
  repository_url: docker.io/ofcommunity/
  push_repository_url: docker.io/ofcommunity/
  basic_auth: false
  secret_mount_path: /var/openfaas/secrets
  builder_url: http://of-builder:8080/

# To use a shared Docker Hub account.
# repository_url: docker.io/ofcommunity/
# push_repository_url: docker.io/ofcommunity/

# Private repo config
  repository_url: 127.0.0.1:5000
  push_repository_url: registry:5000
```

### Update `builder_url` variable and `environment_file` for `buildshiprun` function in `stack.yml`

```yaml
 buildshiprun:
    ...
    environment:
      ...
      builder_url: http://of-builder:8080/
      ...
    environment_file:
      ...
      - buildshiprun_limits_swarm.yml
      ...
```

Note: Update the prefix for function image names in `stack.yml` if you wish to push images to you own registry.

## If you would like to use docker hub to push function docker images. 

### Update the `gateway_config.yml` file as below.

```yaml
environment:
  gateway_url:  http://gateway:8080/
  gateway_public_url: https://cloud.o6s.io/
  audit_url: http://gateway:8080/function/audit-event
  repository_url: docker.io/<docker_hub_id>/
  push_repository_url: docker.io/<docker_hub_id>/
  basic_auth: false
  secret_mount_path: /var/openfaas/secrets
  builder_url: http://of-builder:8080/
```

### Create registry authentication secret

```sh
chmod 777 $HOME/.docker/config.json
cat $HOME/.docker/config.json | docker secret create registry-secret -
```

### Deploy `of-builder` service
```sh
export OF_BUILDER_TAG=0.4.2
docker service create --constraint="node.role==manager" \
 --name of-builder \
 --env insecure=false --detach=true --network func_functions \
 --secret src=registry-secret,target="/home/app/.docker/config.json" \
 --env enable_lchown=false \
openfaas/of-builder:$OF_BUILDER_TAG
```
