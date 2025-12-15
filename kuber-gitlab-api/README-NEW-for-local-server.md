THIS READ-ME IS WRITTEN ONLY FOR LOCAL SERVER USAGE WITH PUBLIC IPV4 ADDRESS

1. Set up DNS name as shown at the dns-settings folder and screenshots: 
      Screenshot 2025-12-08 201436.jpg 
      Screenshot 2025-12-08 201456.jpg

2. Open on your router 22, 80, 443, 5050 ports

3. gitlab https://docs.gitlab.com/install/package/ubuntu/
    To open the needed firewall ports (80, 443, 22) and be able to access GitLab:
    Enable and start the OpenSSH server daemon:
        shell
            sudo systemctl enable --now ssh
    With ufw installed, open the firewall ports:
        shell
            sudo ufw allow 22/tcp
            sudo ufw allow 80/tcp
            sudo ufw allow 443/tcp
            sudo ufw enable
            (6443 для кубера возможно надо)
            (Container Registry по HTTP возможно надо :5050)
    sudo EXTERNAL_URL="http://93.84.96.137" GITLAB_ROOT_PASSWORD="Yx/jHR4vtZaFUXAaH6dncc8wx3r2GDfkoYxeLc+A2S7=" apt install gitlab-ce=17.8.1
        cat /etc/gitlab/initial_root_password - Yx/jHR4vtZaFUXAaH6dncc8wx3r2GDfkoYxeLc+A2S4=

4. Создай два проекта:
    - kuber-gitlab-api (be-devops так именно не назвал(он назвал k8s-connection), но это не важно, так как важно как он назвал второй проект k8s-connection для соединения с кубернет агентом)
        склонируй его и наполни содержимым kuber-gitlab-api.
        !!! у меня не получалось сперва скопировать, т к гитлаб при клонировании предлагал левые айпи, поэтому проверь что бы был публичный ipv4  
            если не получается склонировать - есть урок у ADV-IT - https://www.youtube.com/watch?v=-j4hfcXSdws но я в итоге на токене настроил
    - k8s-connection (можно потом, но так именно назвал be-devops https://www.youtube.com/watch?v=fwtxi_BRmt0&ab_channel=be-devops когда кубер кластер ставил - может можно и по своему, но я скопировал что бы все работало как у него)

5. runner + registry
    ADV-IT - https://www.youtube.com/watch?v=WflvuVPvCL8 - GitLab CI/CD - Как работают Runners, Установка своего SHELL GitLab Runner на Linux и Windows
    Mihail Kozlov - https://www.youtube.com/watch?v=uSTOerrWNaY&t=1s - GitLab CI/CD Runner простой проект
    Build With LaL - https://www.youtube.com/watch?v=Rvh7OZbDJ_o&t=577s - Register Docker Runner/Executor with GitLab Server to Run Pipelines
    
    - настрой примерное содержание /etc/hosts - как в примере - files-to-change/hosts

    - создай раннер в созданном проекте - kuber-gitlab-api
        Install GitLab Runner будет - в самом гитлабе инфа:
            # Download the binary for your system
                sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
            # Give it permission to execute
                sudo chmod +x /usr/local/bin/gitlab-runner
            # Create a GitLab Runner user
                sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
            # Install and run as a service
                sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
                sudo gitlab-runner start
        gitlab-runner register  --url http://93.84.96.137 --token glrt-IzO2G358ecpJnAlIztTWGG86MQpwOjMKdDozCnU6MQ8.01.1707nra3l

    - скопируй полностью содержимое трех файлов с files-to-change/for-local-server-hp-proliant-dl180 - .gitlab-ci.yml, gitlab.rb, config.toml
        .gitlab-ci.yml - полностью замени, а gitlab.rb, config.toml - скопируй настройки

    - после изменения .gitlab-ci.yml, gitlab.rb, config.toml примени:
        sudo gitlab-ctl reconfigure
        sudo gitlab-ctl restart
        sudo systemctl restart gitlab-runner

    - т к на роутере не настроен NAT loopback (hairpin NAT), то нужно прописать у раннера в config.toml:
        extra_hosts = ["elinext-ai-gitlab.tech:192.168.1.3"]

    - когда не с первого раза настроил, то полезно в бд гитлаба прописать флаги, которые слетают:

        sudo gitlab-rails runner -e production - <<'RUBY'
        r = Ci::Runner.find(6)
        r.active = true
        r.locked = false
        r.run_untagged = true
        # ключевое: разрешить protected ветки
        r.access_level = :ref_protected
        # ключевое: теги должны быть в GitLab, а не только в config.toml
        r.tag_list = "test"
        r.save!
        puts "Runner fixed: id=#{r.id} active=#{r.active} access_level=#{r.access_level} tags=#{r.tags.map(&:name).join(",")}"
        RUBY

    - Container Registry (актуально если ты настраиваешь без dns, т е если у тебя Https и норм днс имя с хостера, то не надо это делать)
        ты настроил уже если добавил в gitlab.rb вот эти настройки:
            registry_external_url 'http://192.168.1.3:5050'
            gitlab_rails['registry_enabled'] = true
            registry['enable'] = true
            registry_nginx['enable'] = true
            registry_nginx['listen_port'] = 5050
            registry_nginx['listen_https'] = false
            registry['token_realm'] = 'http://192.168.1.3'

    - Install + start Docker
            sudo apt update
            sudo apt install -y docker.io
            sudo systemctl enable --now docker
            sudo systemctl status docker --no-pager
        Give gitlab-runner access to the Docker socket
            sudo usermod -aG docker gitlab-runner
            sudo systemctl restart gitlab-runner
        Verify
            sudo -u gitlab-runner docker info
            ls -l /var/run/docker.sock`
   
    - PS:
       этими командами я скачал gitlab.rb, config.toml, тебе они не нужны, но для истории оставлю:
           scp trump@93.84.96.137:/etc/gitlab-runner/config.toml C:/test (на самом деле надо скопировать в /home/trump/ и затем - scp trump@93.84.96.137:/home/trump/config.toml C:/test)
           scp trump@93.84.96.137:/home/trump/gitlab.rb C:/test (на самом деле надо скопировать в /home/trump/ и затем - scp trump@93.84.96.137:/etc/gitlab/gitlab.rb C:/test)

6. CREATING A KUBERNET CLUSTER IN GCP
    be-devops https://www.youtube.com/watch?v=fwtxi_BRmt0&ab_channel=be-devops How to Build and Deploy an app on Kubernetes by GitLab ci cd pipeline:

    helm repo add gitlab https://charts.gitlab.io
    helm repo update
    helm upgrade --install k8s-connection gitlab/gitlab-agent \
    --namespace gitlab-agent-k8s-connection \
    --create-namespace \
    --set image.tag=v18.6.0 \
    --set config.token=glagent-Zz8vmcTqo2dwhuNX79KBt286MQpwOjMH.01.0w04gvu12 \
    --set config.kasAddress=wss://elinext-ai-gitlab.tech/-/kubernetes-agent/
    
    т к на роутере не настроен NAT loopback (hairpin NAT), то нужно прописать:
    
    helm upgrade --install k8s-connection gitlab/gitlab-agent \
    -n gitlab-agent-k8s-connection --reuse-values \
    --set hostAliases[0].ip=192.168.1.3 \
    --set hostAliases[0].hostnames[0]=elinext-ai-gitlab.tech
    
    kubectl -n gitlab-agent-k8s-connection rollout restart deploy/k8s-connection-gitlab-agent-v2
    kubectl -n gitlab-agent-k8s-connection logs -f deploy/k8s-connection-gitlab-agent-v2 --tail=200

7. настройка кубернет инстансов

    Откуда брать креды в GitLab:
        В GitLab обычно делают Deploy Token или Project Access Token с правом read_registry (и при необходимости write_registry). Эти значения вы и подставляете в команду выше.
            kuber_gitlab_api_access_tokens glpat-OUJlaIxkXQ_82ReqQknBam86MQp1OjMH.01.0w1pyvbum
        И важный момент в вашем случае: раз образ у вас на http://...:5050, k3s/containerd нужно настроить как insecure registry, иначе pull будет ломаться даже при правильном secret.
    Способ 1 (самый простой): kubectl сам всё сгенерирует

    Вы даёте GitLab-овские креды (user+token), а Kubernetes формирует .dockerconfigjson внутри secret:
        sudo kubectl -n default create secret docker-registry app-secret \
            --docker-server="elinext-ai-gitlab.tech:5050" \
            --docker-username="kuber_gitlab_api_access_tokens" \
            --docker-password="glpat-OUJlaIxkXQ_82ReqQknBam86MQp1OjMH.01.0w1pyvbum"

{"auths":{"elinext-ai-gitlab.tech:5050":{"username":"kuber_gitlab_api_access_tokens","password":"glpat-OUJlaIxkXQ_82ReqQknBam86MQp1OjMH.01.0w1pyvbum","auth":"a3ViZXJfZ2l0bGFiX2FwaV9hY2Nlc3NfdG9rZW5zOmdscGF0LU9VSmxhSXhrWFFfODJSZXFRa25CYW04Nk1RcDFPak1ILjAxLjB3MXB5dmJ1bQ=="}}}
{"auths":{"elinext-ai-gitlab.tech:5050":{"username":"kuber_gitlab_api_access_tokens","password":"***","auth":"***"}}}

7.  у меня дома установлен hp proliant dl180 gen9. провайдер выделил мне статический Ipv4. мой роутер не поддерживает NAT loopback (hairpin NAT). я заказал на этот статический айпи днс запись. еще я поставил гитлаб на эту запись, открыл порты 443, 80, 5050 и 22 на роутере. в гитлабе я настроил раннер, что бы он выполнял пайплайны. затем я поставил кубернет через k3s и настроил соединение между кубернет кластером и гитлабом через агента.
    у меня такой вопрос. следующим этапом я буду настраивать сущности кубернета. у меня будет несколько нэймспйэсов - dev, test, stage. каждый стэйдж будет состоять из своей базы данных, приложения на джаве и тд. соответсвенно этим приложением я хочу обращаться по днс имени с https. смогу ли я как то перенаправлять на другие порты, т к гитлаб если я правильно понимаю занял 80 порт и 443?
    GitLab у меня не в k3s. и роутере не настроен или не поддерживается NAT loopback (hairpin NAT)
    я пришлю все файлы в папке k8s-files, секрет уже есть - app-secret

    ты имеешь ввиду, что т к  у меня гитлаб установлен не через кубернет, а отдельно, то я не смогу просто поставить ингрес на кубернете, что бы он и гитлаб обслуживал, по этому отдельно ставить надо на сервер что-то типа nginx и уже в нем прописывать мэтчинг между днс айпи и проброшеными портами сервисами кубернета обявленными с параметром LoadBalancer и внутренним айпи сервера с портом на котором развернут гитлаб?

    я пытался настроить по HTTP, но неполучилось. раннеры работали, но не удавалась что бы через агенты кубернет кластер был связан с гитлабом

    у мен hp proliant dl180 и один ssd. я хочу сделать правки в системе Linux. но не уверен что смогу отатить назад. могу ли я подключить второй обычный не ссд диск, сделать на него зеркальную копию системы и затем откоючить первый ссд, затем сделать изменения в системе необратимые и если не получится то вернуть назад пеервый ссд?

    у меня несколько вопросов, касаемо твоего предложения:
    1. если гитлаб будет работать по https://elinext-ai-gitlab.tech/ через внешний nginx, то как генерировать сертификат, где он будет находиться
    2. ресурс для дев энвайрамента будет например по  https://my-app1-vvv357-dev.tech/, для тест энвайрамента, например - https://my-app1-vvv357-test.tech/ и тд. То как генерировать сертификаты для этих ресурсов, где они будут находиться?
    3. т е сертификаты самостоятельно нужно будет генерить для гитлаба, для https://my-app1-vvv357-dev.tech/, для https://my-app1-vvv357-test.tech/ и тд? и как нужно будет генерить эти сертификаты?
    4. а если я недавно ставил гитлаб где указывал к примеру EXTERNAL_URL="https://elinext-ai-gitlab.tech/" и скорее всего сертификат был сгенерен, то мне придется заново генерить или найти сушествуюший и положить в nginx? 

helm repo add gitlab https://charts.gitlab.io
helm repo update
helm upgrade --install k8s-connection gitlab/gitlab-agent \
--namespace gitlab-agent-k8s-connection \
--create-namespace \
--set image.tag=v18.6.0 \
--set config.token=glagent-Zz8vmcTqo2dwhuNX79KBt286MQpwOjMH.01.0w04gvu12 \
--set config.kasAddress=wss://elinext-ai-gitlab.tech/-/kubernetes-agent/

helm upgrade --install k8s-connection gitlab/gitlab-agent \
-n gitlab-agent-k8s-connection --reuse-values \
--set hostAliases[0].ip=192.168.1.3 \
--set hostAliases[0].hostnames[0]=elinext-ai-gitlab.tech

kubectl -n gitlab-agent-k8s-connection rollout restart deploy/k8s-connection-gitlab-agent-v2
kubectl -n gitlab-agent-k8s-connection logs -f deploy/k8s-connection-gitlab-agent-v2 --tail=200


#for manual setting up right parameters in the inner gitlab db:

sudo gitlab-rails runner -e production - <<'RUBY'
r = Ci::Runner.find(6)
r.active = true
r.locked = false
r.run_untagged = true
# ключевое: разрешить protected ветки
r.access_level = :ref_protected
# ключевое: теги должны быть в GitLab, а не только в config.toml
r.tag_list = "test"
r.save!
puts "Runner fixed: id=#{r.id} active=#{r.active} access_level=#{r.access_level} tags=#{r.tags.map(&:name).join(",")}"
RUBY




Нажмите комбинацию клавиш: Ctrl+Alt+F2 (или F3, F4 — одна из них должна сработать).

bash
sudo dd if=/dev/sda of=/dev/sdb bs=4M status=progress
Use code with caution.

if=/dev/sda: Источник (SSD).
of=/dev/sdb: Цель (HDD).
bs=4M: Размер блока 4 мегабайта (для скорости).
status=progress: Показывает, сколько уже скопировано.




sudo ip addr add 192.168.1.3/24 dev eno2



