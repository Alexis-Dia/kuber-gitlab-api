



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
    https://elinext-ai-gitlab.tech/


CONNECTING A DNS ROW IN THE HOSTER.BY WITH GCP:
    VIDEO - ~/Gitlab/10-02-2025/videos how to do all/gcp dns rule (5050).mov



INSTALLING GITLAB-CE IN A VM INSTANCE IN GCP + RUNNER:
    1. https://www.youtube.com/watch?v=TftSa_xJKXM&ab_channel=ADV-IT (у меня все ok работало на - GitLab Community Edition v17.8.1)
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



    kubectl delete svc login-svc
    kubectl delete deploy postgres-db
    kubectl delete svc postgres-service
    kubectl delete pod login-app
    kubectl delete svc login-svc
    kubectl apply -f postgres-pvc.yaml
    kubectl apply -f postgres-deployment.yaml
    kubectl apply -f postgres-service.yaml
    kubectl apply -f pod.yaml
    kubectl apply -f loginsvc-loadbalancer.yaml


CHECKING THAT EVRTH IS WORKING AS EXPECTED:
    - connect to kluster in 'Google Cloud SDK Shell' and run this command - 'kubectl get svc login-svc' to know IP and use     {external-ip}/cats or go to .gitlab-ci.yml file and then check IP in pipeline



