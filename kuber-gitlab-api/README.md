cd /Users/alekseydruzik/Gitlab/gitlab4
docker-compose up
http://localhost:8088/root/demo/-/runners/new
docker ps -a
docker exec -it 4b4433a85f4b /bin/bash      (войти в раннер)
gitlab-runner register  --url http://localhost:8088  --token glrt--TiQN_J-DhDBPf6NQKCy --docker-network-mode 'host'


project test
https://www.youtube.com/watch?v=uSTOerrWNaY&t=1s
проект создается с тегами - "api, test"

stages:
- build_and_test
- deploy

variables:
test_file_remote: /tmp/test_job_remote.txt
test_file: /tmp/test_job.txt
git_file: /tmp/test/test.txt

test_job_ci:
stage:  build_and_test
tags:
- test
- api
script:
- echo "Project dir - test_job_ci"
- echo "Project dir - $CI_PROJECT_DIR"
- echo "Project name - $CI_PROJECT_NAME"

test_job_cd:
stage:  deploy
tags:
- api
script:
- echo "Project dir - test_job_cd"
artifacts:
paths:
- build/


project demo
https://www.youtube.com/watch?v=Rvh7OZbDJ_o&t=577s
проект создается с тега - "test"

build:
image: python:alpine
script:
- python --version


project on java

build:
image: maven:3.8.4-openjdk-17
script:
- java --version








My project with multiple images:

stages:
- build_and_test
- deploy

variables:
test_file_remote: /tmp/test_job_remote.txt

test_job_ci:
stage:  build_and_test
tags:
- test
image: python:alpine
script:
- python --version

test_job_cd:
stage:  deploy
tags:
- test
image: ruby:2.7
script:
- ruby -v


https://hub.docker.com/r/lachlanevenson/k8s-kubectl





My project with multiple images and own pipeline: (but you need add this line to 'vi /etc/gitlab-runner/config.toml' inside gitlab-runner (https://stackoverflow.com/questions/76240928/gitlab-ci-cd-error-while-pulling-docker-image) )

stages:
- build_and_test
- deploy

variables:
test_file_remote: /tmp/test_job_remote.txt

test_job_ci:
stage:  build_and_test
tags:
- test
image: alpine:3.18
script:
- apk add openjdk17
- apk add maven
- mvn clean install
test_job_cd:
stage:  deploy
tags:
- test
image: alpine:3.12
script:
- java -version


Пишем Spring Boot микросервис для деплоя в kubernetes с нуля!
https://www.youtube.com/watch?v=KPLJ0i5Ocws&list=LL&index=3
docker build . -t jusaf/myapp:latest
    jusaf: This is the repository name.
    myapp: This is the name of the application or image.
    latest: This is the tag for the image, often used to indicate the most recent version.

https://docs.gitlab.com/ee/user/packages/container_registry/build_and_push_images.html
    script:
        - echo "$CI_REGISTRY_PASSWORD" | docker login $CI_REGISTRY -u $CI_REGISTRY_USER --password-stdin
        - docker build -t $CI_REGISTRY/group/project/image:latest .
        - docker push $CI_REGISTRY/group/project/image:latest


fastplanet version:
    (- docker build -t "$CI_REGISTRY_IMAGE:$IMAGE_VERSION" -t "$CI_REGISTRY_IMAGE:latest" .)

docker build . -t alexd/cats-api:1.0.0

https://stackoverflow.com/questions/54734729/get-back-docker-for-windows-kuberentes-kubeconfig-file-after-deleting-it
kubectl config current-context
kubectl config get-contexts
kubectl config use-context docker-desktop
kubectl get componentstatuses
kubectl cluster-info

The Kubernetes server runs locally within your Docker instance, is not configurable, and is a single-node cluster. It runs within a Docker container on your local system, and is only for local testing.


kubectl apply -f deployment.yaml
kubectl get pods --watch
kubectl logs cats-api-64f848694-nt969


kubectl delete pods cats-api-6cdc879596-rhh5p
kubectl delete pods --all
kubectl delete deployments --all
kubectl apply -f deployment.yaml
kubectl port-forward cats-api-6cdc879596-tzqvh 8899:8080
ifconfig   (эта команда нужна, так как может быть в зависимости от сети разный айпищник)

http://localhost:8899/cats-api/cats


add this line to 'vi /etc/gitlab-runner/config.toml' inside gitlab-runner (https://stackoverflow.com/questions/76240928/gitlab-ci-cd-error-while-pulling-docker-image)
pull_policy = ["if-not-present", "always"]


change this line to 'vi /etc/gitlab-runner/config.toml' inside gitlab-runner (https://forums.docker.com/t/error-error-during-connect-get-http-docker-2375-ping-dial-tcp-lookup-docker-on-10-10-15-22-server-misbehaving/138383)
volumes = ["/var/run/docker.sock:/var/run/docker.sock", "/cache"]

docker-compose up --build --force-recreate

Build With LaL - Setup GitLab Container Registry | Push Docker Images Easily
    https://www.youtube.com/watch?v=4Klbfm4CixY
    https://github.com/BuildWithLal/gitlab-in-docker/tree/master/09.%20setup-container-registry
        - echo "$CI_REGISTRY_PASSWORD" | docker login $CI_REGISTRY -u $CI_REGISTRY_USER --password-stdin
        - docker build -t "$CI_REGISTRY_IMAGE:${CI_COMMIT_SHA:0:8}" .
        - docker push "$CI_REGISTRY_IMAGE:${CI_COMMIT_SHA:0:8}"
        - docker build -t 127.0.0.1:5001/root/build-with-lal:my-local-commit
        - docker push 127.0.0.1:5001/root/build-with-lal




how to connect to my cluster from the command line from gitlab-ci script section?
https://www.youtube.com/watch?v=fwtxi_BRmt0

.gitlab/agents/primary-agent/config.yaml

gitops:
manifest_projects:
- id: "root/kuber-gitlab-api"
paths:
- glob: '/**/*.{yaml,yml,json}'



Agent access token:
    glagent-JUszeWi1TpsEibuoAbZy7s8DRuFJ9YQ4ZKCZuXkS2T4xriPt1A

Install using Helm (recommended)
From a terminal, connect to your cluster and run this command. The token is included in the command. Use a 
Helm version compatible with your Kubernetes version (see Helm version support policy ).

helm repo add gitlab https://charts.gitlab.io
helm repo update
helm upgrade --install primary-agent gitlab/gitlab-agent \
--namespace gitlab-agent-primary-agent \
--create-namespace \
--set image.tag=v17.3.1 \
--set config.token=glagent-ZXQW3EMV4jHnP_FZD-caC7xuk9-fp_81bR6x9ykk6VKqPXE6Tg \
--set config.kasAddress=ws://192.168.0.7:8088/-/kubernetes-agent/


helm uninstall primary-agent --namespace gitlab-agent-primary-agent



kubectl logs -f -l=app.kubernetes.io/name=gitlab-agent -n gitlab-agent-primary-agent

https://docs.gitlab.com/ee/user/clusters/agent/ci_cd_workflow.html
    If you use an environment with KAS and a self-signed certificate, you must configure your Kubernetes client to trust the
        certificate authority (CA) that signed your certificate.
https://gitlab.com/gitlab-org/gitlab/-/issues/352284
    --insecure-skip-tls-verify wouldn't help, it just disables TLS verification, but TLS is still required for it to send credentials
https://forum.gitlab.com/t/gitlab-kas-agent-job-failed-you-must-be-logged-in-to-the-server/65102/3
    Hi everyone, Gitlab Agent for Kubernetes works in free edition. Your gitlab must be hosted from https and also agent
    should be directed to wss address. kubectl does not send authorization header if target is http. Ahmet

# if you installed gitlab-ce with the help of this command - sudo EXTERNAL_URL="https://gitlab.example.com" apt-get install gitlab-ee
# YOU DON'T NEED THIS. THIS YOU NEED ONLY IF YOU INSTALLED THROUGH DOCKER(BUILD WITH LAL VIDEOS):
How to Set Up GitLab with Docker Compose and SSL Certificates - https://www.youtube.com/watch?v=W4omV9jn1QE&list=LL&index=1
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout GitlabSSL.key -out GitlabSSL.crt
openssl req -nodes -x509 -sha256 -newkey rsa:4096 -keyout gitlab.key -out gitlab.crt -days 356 -subj "/C=CC/ST=State/L=City/O=gitlab.my.com/OU=LocalDepartment/CN=gitlab.my.com" -addext "subjectAltName = DNS:localhost,DNS:gitlab.my.com"    
openssl req -nodes -x509 -sha256 -newkey rsa:4096 -keyout gitlab.key -out gitlab.crt -days 356 -subj "/C=CC/ST=State/L=City/O=gitlab.my2.com/OU=LocalDepartment/CN=gitlab.my2.com" -addext "subjectAltName = DNS:localhost,DNS:gitlab.my2.com"
sudo apachectl stop
docker-compose -f docker-compose.yml up -d --build --force-recreate
sudo nano /etc/hosts


root LAP4quMbV42QGvfj password











CREATING A VM INSTANCE IN GCP:
This for disabling switching of ssh console(это надо, что бы у тебя не отвалился коннект при утсановке гитлаба, т е вводишь в нижнем окне, а не кликаешь
всплывающее окно на 'OPEN SSH'):
rm ~/.ssh/google_compute_engine*
gcloud compute ssh --zone "us-central1-a" "instance-20250209-082920" --project "ultra-aloe-443207-k7" --ssh-flag="-ServerAliveInterval=99000"
root qwerty!12345 password
after installing a runner - open in firewall of gcp 5050 port (there is the video)
VIDEO - ~/Gitlab/10-02-2025/videos how to do all/vm-machine.mov
VIDEO - ~/Gitlab/10-02-2025/videos how to do all/ip-addresses.mov
VIDEO - ~/Gitlab/10-02-2025/videos how to do all/gcp dns rule (5050).mov
VIDEO - ~/Gitlab/10-02-2025/videos how to do all/cloud-dns-record.mov



CREATING A DNS ROW IN THE HOSTER.BY:



CONNECTING A DNS ROW IN THE HOSTER.BY WITH GCP:
VIDEO - ~/Gitlab/10-02-2025/videos how to do all/gcp dns rule (5050).mov



INSTALLING GITLAB-CE IN A VM INSTANCE IN GCP + RUNNER:
1. https://www.youtube.com/watch?v=TftSa_xJKXM&ab_channel=ADV-IT
2. https://www.youtube.com/watch?v=xFSLFF2wQDs&t=301s&ab_channel=ADV-IT
VIDEO - ~/Gitlab/10-02-2025/videos how to do all/cloud-dns-record.mov


CREATING A KUBERNET CLUSTER IN GCP https://www.youtube.com/watch?v=ZHv1c64PXO4&list=PLg5SS_4L6LYvN1RqaVesof8KAf-02fJSi&index=4&ab_channel=ADV-IT:
gcloud init
gcloud services enable container.googleapis.com
gcloud container clusters create alex1 --location us-central1-a
gcloud components install gke-gcloud-auth-plugin
gcloud container clusters get-credentials alex1 --location us-central1-a             (команда для переключения между кластерами)
kubectl cluster-info
kubectl get componentstatuses
kubectl get nodes
gcloud container clusters create alex2 --location us-central1-a --num-nodes=1
gcloud container clusters delete alex1 --location us-central1-a
NAME   LOCATION       MASTER_VERSION      MASTER_IP    MACHINE_TYPE  NODE_VERSION        NUM_NODES  STATUS
alex1  us-central1-a  1.31.4-gke.1372000  34.57.7.119  e2-medium     1.31.4-gke.1372000  3          RUNNING
helm repo add gitlab https://charts.gitlab.io
helm repo update
helm upgrade --install k8s-connection gitlab/gitlab-agent \
--namespace gitlab-agent-k8s-connection \
--create-namespace \
--set image.tag=v17.8.1 \
--set config.token=glagent-jQ92LrcsdeNGdTyNVq7yYXXwXt18x3pF2Hz4E84AeqoWNrCqow \
--set config.kasAddress=wss://gitlab-site.online/-/kubernetes-agent/
helm upgrade --install k8s-connection gitlab/gitlab-agent --namespace gitlab-agent-k8s-connection --create-namespace --set image.tag=v17.8.1 --set config.token=glagent-jQ92LrcsdeNGdTyNVq7yYXXwXt18x3pF2Hz4E84AeqoWNrCqow --set config.kasAddress=wss://gitlab-site.online/-/kubernetes-agent/



INSTALLING AGENT https://www.youtube.com/watch?v=fwtxi_BRmt0&ab_channel=be-devops IN THE KUBERNET CLUSTER:
Connecting a Kubernetes cluster with GitLab
https://docs.gitlab.com/ee/user/clusters/agent/
To connect a Kubernetes cluster to GitLab, you must first install an agent in your cluster:
https://docs.gitlab.com/ee/user/clusters/agent/install/index.html
Transcribe:
До 14 минуты важно понять, так как он подукументации делает два репозитория
создал пустой файл .gitlab/agents/k8s-connection/config.yaml в проекте k8s-connection
После 14 и до 30 минуты он показывает, как репозиторий наполнить файлами проекта без всяких агентов, как запушить вручную докер образ в контейнер-регистри, как сдеалть ci-cd пайплайн и автоматом пушить в контейнер-регистри и тд
После 30 минуты - как запушить из контейнер регистри в кубернет-кластер.
До 37 минуты идет теория.
C 37 минуты по 50 он рассказывает, как вручную на кубернет кластер задеплоить вручную докер-образ
kubectl run login-app --image=registry.gitlab.com/d6245/k8s-data/samle:v1 --dry-run=client -o yaml > pod.yaml
kubectl create secret docker-registry app-secret --docker-server=registry.gitlab.com --docker-username='karaminejad' --docker-password='' --dry-run=client -o yaml secret.yaml
kubectl apply -f secret.yaml
kubectl get secret
kubectl apply -f pod.yaml
kubectl get pod
kubectl create service nodeport login-svc --tcp=80:80 --dry-run=client -o yaml loginsvc.yaml
kubectl apply -f loginsvc.yaml
kubectl describe service login-svc
kubectl get pod -o wide
kubectl get node
kubectl get node -o wide
C 50 минуты он рассказывает, как с помощью ci-cd на кубернет кластер задеплоить докер-образ
- добавил новый стейдж в gitlab-ci.yml
- добавил содержимое агента .gitlab/agents/k8s-connection/config.yaml
ci_access:
projects:
- id: root/kuber-gitlab-api













