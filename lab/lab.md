# Requisitos para o Lab
- Instalar o [Rancher Desktop](https://docs.rancherdesktop.io/getting-started/installation/)
- Realizar o deploy do Jenkins 
  <hl>
    1. helm repo add jenkins https://charts.jenkins.io
    2. helm repo update
    3. helm upgrade --install myjenkins jenkins/jenkins
  </hl>
- Criar uma conta em algum SCM de sua preferência como: Gitlab ou Github
    
