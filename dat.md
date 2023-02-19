# **DAT BRIEF 7 - Automatisation de l'Application**

## **Contexte du projet**

La pipeline devra automatiquement déployer la mise a jour de l'application remplacement de l'ancienne version par la nouvelle. L'application devra être mise à jour toutes les heures. 


## **Besoins fonctionnels**

Nous utilisons **Aks** (**Kubernetes**) **l'application vote** dans lequel ils avaient développé l'application de vote, l'application de vote consiste à permettre à l'utilisateur de pouvoir choisir entre *Windows* et *Linux* ou de réinitialiser le compteur. Pour deployer la machine virtuelle Jenkins j'ai utilisé *Terraform*.



## **La représentation technique** 

### DÉPLOYMENT 

1. *Développé le script pour créer la mv*
2. *Installer Java et Jenkins dans la mv*
3. *Créer le ClusterAKS 

## LES OUTILS UTILISÉS: 

1. *AZURE*
2. *GIT*
3. *TERRAFORM*
4. *JENKINS PIPELINE*
5. *DOCKER*

Sur la mv j'ai installer: kubectl, git, jq, azure CLI.

Ajout des identifiants de kubeconfig pour pouvoir accéder au cluster AKS:

```consol
az aks get-credentials --name AKSCluster --resource-group Brief7-RC

```

- Afin de déverrouiller Jenkins nécessite un mot de passe pour pouvoir le récupérer, il est nécessaire d'effectuer les opérations suivantes sur le terminal:

            sudo cat /var/lib/jenkins/secrets/initialAdminPassword
       
![](https://i.imgur.com/RtSI7G7.png)

Une fois que nous obtenons le mot de passe, nous pouvons accéder à Jenkins, après la connexion, j'ai installé des plugins:  
- **Pipeline,**
- **Kubernetes CLI,** 
- **Workspace Cleanup**
  
Pour s'assurer qu'il y a une nouvelle mise à jour de version toutes les heures il faut aller sur build Triggers -> Construire périodiquement et mettre  **H******  (https://crontab.guru/#1_*_*_*_*)

### Script Pipeline 

``` consol 

pipeline {
    agent any 
    stages {
        stage('Stage 1') {
            steps {
                withKubeConfig([credentialsId: 'aks']) {
                    sh('''
                    git clone https://github.com/simplon-choukriraja/brief7-votinapp.git app
                    TAG=\044(curl -sSf https://registry.hub.docker.com/v2/repositories/simplonasa/azure_voting_app/tags |jq '."results"[0]["name"]'| tr -d '"')
                    sed -i "s/TAG/\044{TAG}/" ./app/vote.yml
                    kubectl apply -f ./app
                    ''')
                }
            }
        }
    }
    post {
        always {
            // Nettoyage de l'espace de travail Jenkins
            step([$class: 'WsCleanup'])
        }
    }
}

```

La variable TAG remplacer la version sur le script de vote. 



## **La représentation fonctionnelle** 

- Image Docker: https://hub.docker.com/r/simplonasa/azure_voting_app
Est l'image qui a été envisagée pour créer l'application de vote entre Windows et Linux et qui sera envisagée pour automatiser une nouvelle mise à jour de version toutes les heures.

- la machine virtuelle jenkins a deux règles de sécurité : le port SSH (22) et le port HTTP (8080).

## **Topologie**

![](https://i.imgur.com/5VNPS8M.png)
