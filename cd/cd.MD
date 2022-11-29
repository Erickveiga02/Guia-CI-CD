# Deploy contínuo

A seguir será demonstrado passo a passo para montar um deploy contínuo utilizando o Jenkins. Lembrando que o conceito de deploy contínuo é implantar o software sem nenhuma intervenção humana.

### Jenkins Service Account

O Jenkins foi implantando no K8s logo isso facilita o processo de implantação, contudo o Jenkins só tem acesso ao seu próprio namespace. Mas para resolver esse problema é preciso deletar a ClusterRoleBinding chamada "jenkins" e cria-la novamente com as seguintes configurações:

```
$ kubectl create clusterrolebinding jenkins --clusterrole cluster-admin --serviceaccount=jenkins:default -n jenkins
```

Observação: Lembre-se que essa configurações é apenas para o laboratório, em um ambiente produtivo é necessário configurar uma role com apenas a permissões de deploy, ao adicionar "cluster-admin" a conta do Jenkins tem total permissão no Cluster o que para um ambiente real é algo problematico.

### Kaniko

Para nossa etapa de Build será utilizado o Kaniko na pipeline. Necessário configurar uma secret:

```
kubectl create secret docker-registry saas-credentials --docker-username=$REGISTRY_USER --docker-password=$REGISTRY_PASS --docker-server=$REGISTRY_DOCKER --docker-email=$REGISTRY_EMAIL -n jenkins
```
Essa secret é montada no POD através do POD Template no Jenkinsfile:
```
kind: Pod
metadata:
  name: ci
labels:
  component: mvn
spec:
  containers:
  - name: mvn
    image: maven:3-jdk-8-alpine
    command:
    - sleep
    args:
    - 9999999
  - name: kubectl
    image: amaceog/kubectl
    command:
    - sleep
    args:
    - 9999999
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - sleep
    args:
    - 9999999
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
  - name: kaniko-secret
    secret:
       secretName: regcred
       items:
       - key: .dockerconfigjson
         path: config.json
```

Para realizar o push e build da imagem docker será acrescentado o seguinte stage no Jenkinsfile:

```
stage('Push Image'){
            steps{
                container('kaniko'){
                    sh  '/kaniko/executor --context `pwd` --destination erickveiga/hello-world-mvn'
                     
                }
            }
         }
```


### kubectl

Para realizar o deploy será utilizado uma imagem com o "kubectl"  e adicionado o seguinte stage no Jenkinsfile:

```
         stage('deploy'){
            steps{
                container('kubectl'){
                    sh  'kubectl apply -f k8s/deployment.yaml -n app --validate=false '
                    sh  'kubectl apply -f k8s/service.yaml -n app --validate=false '
                     
                }
            }
         }
```

Após a conclusão acesse "http://localhost:30043/" 

### Jenkinsfile

```
pipeline { 
     agent {
    kubernetes {
      label 'spring-petclinic-demo'
      defaultContainer 'jnlp'
      yaml '''
kind: Pod
metadata:
  name: ci
labels:
  component: mvn
spec:
  containers:
  - name: mvn
    image: maven:3-jdk-8-alpine
    command:
    - sleep
    args:
    - 9999999
  - name: kubectl
    image: amaceog/kubectl
    command:
    - sleep
    args:
    - 9999999
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - sleep
    args:
    - 9999999
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker
  restartPolicy: Never
  volumes:
  - name: kaniko-secret
    secret:
       secretName: regcred
       items:
       - key: .dockerconfigjson
         path: config.json
    

'''
    }
  }
 
    stages { 
          
        stage('Build') { 
            steps { 
            container('mvn'){
               sh 'mvn package'
            }
            }
        }
        stage('Unit test'){
            steps{
                container('mvn'){
                    sh 'mvn test'
                }
            }
        }
         stage('Push Image'){
            steps{
                container('kaniko'){
                    sh  '/kaniko/executor --context `pwd` --destination erickveiga/hello-world-mvn'
                     
                }
            }
         }
         stage('deploy'){
            steps{
                container('kubectl'){
                    sh  'kubectl apply -f k8s/deployment.yaml -n app --validate=false '
                    sh  'kubectl apply -f k8s/service.yaml -n app --validate=false '
                     
                }
            }
         }
    }
}

```


