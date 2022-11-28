# Requisitos para o Lab
- Instalar o [Rancher Desktop](https://docs.rancherdesktop.io/getting-started/installation/)
- Realizar o deploy do Jenkins 
  <hl>
    1. helm repo add jenkins https://charts.jenkins.io
    2. helm repo update
    3. kubectl create namespace jenkins
    4. helm upgrade --install myjenkins jenkins/jenkins -n jenkins
  </hl>
- Após instalar o Jenkins
  <hl> 
    1. kubectl get secrets jenkins -o yaml -n jenkins
    2. Pegue a entrada de Jenkins Password e use o decode do site https://www.base64decode.org/ ou pelo próprio bash
    3. Para acessar o Jenkins:  kubectl --namespace jenkins port-forward svc/jenkins 8080:8080
    4. Usuário default "admin"
  </hl>
- Criar uma conta em algum SCM de sua preferência como: Gitlab ou Github
    
