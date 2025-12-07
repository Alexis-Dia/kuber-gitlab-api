THIS READ-ME IS WRITTEN ONLY FOR LOCAL SERVER USAGE WITH PUBLIC IPV4 ADDRESS

1. Open on your router 22, 80, 443, 5050 ports

2. gitlab https://docs.gitlab.com/install/package/ubuntu/
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

3. runner
    ADV-IT - https://www.youtube.com/watch?v=WflvuVPvCL8 - GitLab CI/CD - Как работают Runners, Установка своего SHELL GitLab Runner на Linux и Windows
    Mihail Kozlov - https://www.youtube.com/watch?v=uSTOerrWNaY&t=1s - GitLab CI/CD Runner простой проект
    Build With LaL - https://www.youtube.com/watch?v=Rvh7OZbDJ_o&t=577s - Register Docker Runner/Executor with GitLab Server to Run Pipelines
    gitlab-runner register  --url http://93.84.96.137 --token glrt-IzO2G358ecpJnAlIztTWGG86MQpwOjMKdDozCnU6MQ8.01.1707nra3l

    scp trump@93.84.96.137:/etc/gitlab-runner/config.toml C:/test (на самом деле надо скопировать в /home/trump/ и затем - scp trump@93.84.96.137:/home/trump/config.toml C:/test)
    sudo gitlab-ctl reconfigure
    sudo gitlab-ctl restart

    scp trump@93.84.96.137:/home/trump/gitlab.rb C:/test (на самом деле надо скопировать в /home/trump/ и затем - scp trump@93.84.96.137:/etc/gitlab/gitlab.rb C:/test)
    sudo systemctl restart gitlab-runner

    скопируй полностью содержимое трех файлов с files-to-change/for-local-server-hp-proliant-dl180 - .gitlab-ci.yml, gitlab.rb, config.toml
        .gitlab-ci.yml - полностью замени, а gitlab.rb, config.toml - скопируй настройки

    Install + start Docker
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

4. CREATING A KUBERNET CLUSTER IN GCP
    be-devops https://www.youtube.com/watch?v=fwtxi_BRmt0&ab_channel=be-devops How to Build and Deploy an app on Kubernetes by GitLab ci cd pipeline:


