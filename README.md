# To add Kubernetes Clusters as runners to Gitlab CICD and deploy app to k8 pods

## Step -1: To setup Kubernetes Cluster

To launch one t3.medium ubuntu-20 machine for Kubernetes Cluster.

### To install below commands on both kubernetes master and worker node.

    sudo apt-get update 
    sudo apt-get install -y docker.io
    sudo usermod –aG docker ubuntu
    newgrp docker
    sudo chmod 777 /var/run/docker.sock
    sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF
    deb https://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    sudo apt-get update
    sudo apt-get install -y kubelet kubeadm kubectl
    sudo snap install kube-apiserver

## Step -2: To setup Gitlab Account

Signup with github account or create new account to work on it.

Create two repository as K8S-connection and K8S-Data in Gitlab.

## Step -3 Configure Agent in Gitlab

To create a new agent file in K8S-connection under below directory.

K8S-connection/.gitlab/agents/k8s-connection

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/10595143-54a3-4051-a59d-8ed2ec0db52d)

### To configure Kubernetes cluster in repo

Navigate to K8S-connection repo -> Operate -> Kubernetes cluster

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/586aefd8-7b61-4ad7-8c24-d3aabcaabe50)

Connect a cluster -> k8s-connection ->Register

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/aabd08e4-5d0a-476b-9df0-4b0e413c1d90)

#### Install below commands on Kubernetes machine to make agent connection

Now, agent token will show on the screen and copy it for later use. Its one time visible

    sudo snap install helm --classic
    helm repo add gitlab https://charts.gitlab.io
    helm repo update
    helm upgrade --install k8s-connection gitlab/gitlab-agent \
        --namespace gitlab-agent-k8s-connection \
        --create-namespace \
        --set image.tag=v16.7.0-rc2 \
        --set config.token=<agent-token> \
        --set config.kasAddress=wss://kas.gitlab.com

Once its installed, now the agent has been connected.

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/f8326b1e-e088-4cca-a7be-30881b50b8e1)

## Step -4: To push the code to k8s-data repo

Use below link to download and push the code to k8s-data repo

    https://gitlab.com/kohlidevops1/k8s-data.git

## Step -5: To configure Container Registry

Navigate to k8s-data repo -> Deploy -> Container Registry

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/654b8be2-3b31-49c6-b4ac-b4a5486be0a1)

To execute the below commands to build and push image

    docker login registry.gitlab.com
    docker build -t registry.gitlab.com/kohlidevops1/k8s-data .
    docker push registry.gitlab.com/kohlidevops1/k8s-data

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/8cf871b6-66fc-4c47-bfbd-cc7f6dc5c0cb)

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/6302daef-e859-4271-ac57-8b4f0a248420)

Now check the Container registry in Gitlab k8s-data repo

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/3c4c992c-bd8e-460b-9e76-b1b36ced080e)

Image is available in Gitlab repo. Now I would like to automare this step using Gitlab CICD. As of now im going to delete this image from container registry.

Below files should be available in the k8s-data repo

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/afb8949d-ef79-470c-b416-2698cc47f757)

You can use the below link to place these files.

    https://gitlab.com/kohlidevops1/k8s-data.git

### To push the files to Gitlab repo

Now im going to push this files to gitlab

    git config --global user.email "latchudevops1@gmail.com"
    git config --global user.name "kohlidevops"
    git add .
    git status
    git commit -am "My Project files"
    git push

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/e227cd43-59b6-4c8d-b232-6bde6996745f)

My app files are available in Gitlab k8s-data repo

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/a4996f3d-7754-46f9-b295-55b79733e30f)

### Create a Gitlab-cicd yaml file

To create a .gitlab-ci.yml file in k8s-data repo and update your pipeline code like below

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/00d89054-4eb3-4cdc-adb8-79f24780dfc7)

Save this file and start the Pipeline using -> Build -> Pipelines

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/325d1ed7-389a-4512-afbe-f56129f1cf74)

Now my build has been succeeded.

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/5ff36ab0-c793-463b-afda-e6d41734a1e2)

You can see the build logs

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/c10246d3-30c4-448b-899c-812063539c46)

To check the container registry using, Project directory -> Deploy -> Container Registry

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/f7a35f6d-3f34-4457-ad54-74f5cd6c2e04)

## Step -6: Create Manifest files

To create a manifest file and copy to pod.yaml file using below command

     kubectl run login-app --image=registry.gitlab.com/kohlidevops1/k8s-data/sample:v1 --dry-run=client -o yaml > pod.yaml

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/b1bf28db-35b8-4816-b067-234bba1b95d1)

Edit the pod.yaml and update as below

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/8991b76a-2e91-4f35-91b4-89ce35ea1361)

To create a manifest for secret

    kubectl create secret docker-registry app-secret --docker-server=registry.gitlab.com --docker-username='
    kohlidevops' --docker-password='<your-password>' --dry-run=client -o yaml > secret.yaml
    sudo cat secret.yaml

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/1de547ba-dd65-4de2-842f-61fd72bf2819)

    kubectl apply -f secret.yaml
    kubectl get secret

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/7255096c-f3f9-455f-bfd7-2528b7454171)

Then copy the secret image name and place it inside pod.yaml like below.

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/fa02569b-71f4-4700-a5d7-7e8198715704)

To create a manifest file for service using below command

    kubectl create svc nodeport login-svc --tcp=80:80 --dry-run=client -o yaml

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/bb996c3e-f150-4b6c-897a-06ad3c6590a5)

To copy and paste the service file to loginsvc.yaml

    kubectl create svc nodeport login-svc --tcp=80:80 --dry-run=client -o yaml > loginsvc.yaml

Now, update the loginsvc.yaml like below -( just remove app: login-svc and add run: login-app) You can refer pod.yaml file to get the login-app name.

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/48f93941-a927-4d06-9201-6ee96187ebe8)

    kubectl apply -f loginsvc.yaml

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/e614ce0f-ca6b-4bf3-83a6-f13934b797fd)

Perfect! My app is running in K8 cluster

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/48c377f4-a0ba-43f8-8c50-90d35d5493b7)

To delete the secret, pods, and service which are manually created just now. Because i will do automate this using gitlab-ci.yml file.

    kubectl delete svc login-svc
    kubectl delete pod login-app
    kubectl delete secret app-secret

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/754a3fab-9e4c-4a39-99fd-4d996fb15ded)

I create one folder k8s-files inside k8s-data and copy the loginsvc.yaml, pod.yaml, and secret.yaml files to k8s-files.

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/11aa2265-3701-4af6-8caa-ffa682efbf1d)

To update the .gitlab-ci.yml file

In your project -> Build -> Pipeline Editor -> To update the cicd yaml file

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/85784119-4513-4310-9f0b-edf947ab5cde)

Navigate to K8S-connection repo and go to below path and edit the config.yaml and update it.

    .gitlab/agents/k8s-connection/config.yaml

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/31d1d12e-8151-4eea-a7de-92d02a46ad26)

Now start the build and check the status. Awesome! My build has been succeeded.

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/ae1295cf-7887-4dde-b391-d9b6a5fabf90)

Below are my .gitlab-ci.yml file complete contents

    variables:
      KUBE_CONTEXT: kohlidevops1/k8s-connection:k8s-connection
    stages:
        - build
        - test
        - deploy
    build_image:
      image: docker
      stage: build
      services:
        - docker:dind
      script:
        - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
        # registry.gitlab.com/kohlidevops1/k8s-data/sample:v1 <we have used in docker build>
        # $CI_REGISTRY -> registry.gitlab.com
        # group -> kohlidevops1
        # project -> k8s-data
        # image:latest -> sample:v1
        # - docker build -t $CI_REGISTRY/group/project/image:latest .
        - docker build -t $CI_REGISTRY/kohlidevops1/k8s-data/sample:v1 .
        - docker push $CI_REGISTRY/kohlidevops1/k8s-data/sample:v1
        - echo "Image Build"
    test_project:
      stage: test
      script:
        - echo "Testing the build!"
    
    deploy_project:
      stage: deploy
      image:
        name: bitnami/kubectl:latest
        entrypoint: ['']
      script:
        - kubectl config use-context $KUBE_CONTEXT
        - kubectl get pods
        - kubectl get nodes -o wide
        - echo "Deploy the Webapp to Kubernetes Cluster"
        - ls $CI_PROJECT_DIR/k8s-files/
        - kubectl apply -f $CI_PROJECT_DIR/k8s-files/.
        - kubectl get pods
        - kubectl get svc

Now i can able to access my application through kubernetes cluster.

![image](https://github.com/kohlidevops/gitlab-cicd-add-cluster-as-runners/assets/100069489/20d71370-d091-4a65-acca-436faed9ecb3)

That's it!

### This is the way that we have to add Kubernetes cluster as a runner to Gitlab and deploy app on pods using Gitlab CICD

You can refer below link too

    https://www.youtube.com/watch?v=fwtxi_BRmt0&t=626s

