---
title: Exemple de conteneur d’exécution de la commande d’exécution du docker
titleSuffix: Azure Cognitive Services
description: Commande docker run pour le conteneur de santé
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 11/12/2020
ms.author: aahi
ms.openlocfilehash: b19fb3f86be46a5db60fb87f9c7f5c3e28ac6428
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965144"
---
## <a name="install-the-container"></a>Installer le conteneur

Il existe plusieurs façons d’installer et d’exécuter le conteneur Analyse de texte pour la santé. 

- Utilisez le [Portail Azure](../how-tos/text-analytics-how-to-install-containers.md?tabs=healthcare) pour créer une ressource d’Analyse de texte, et utilisez Docker pour obtenir votre conteneur.
- Utilisez les scripts PowerShell et Azure CLI suivants pour automatiser le déploiement des ressources et la configuration du conteneur.

### <a name="run-the-container-locally"></a>Exécutez localement le conteneur

Pour exécuter le conteneur dans votre propre environnement après avoir téléchargé l’image conteneur, recherchez son ID d’image :
 
```bash
docker images --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```

Exécutez la commande `docker run` suivante. Remplacez les espaces réservés suivants par vos valeurs :

| Espace réservé | Valeur | Format ou exemple |
|-------------|-------|---|
| **{API_KEY}** | Clé de votre ressource Analyse de texte. Cette information est disponible dans le portail Azure, sur la page **Key and endpoint** (Clé et point de terminaison) de votre ressource. |`xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`|
| **{ENDPOINT_URI}** | Point de terminaison pour accéder à l’API Analyse de texte. Cette information est disponible dans le portail Azure, sur la page **Key and endpoint** (Clé et point de terminaison) de votre ressource. | `https://<your-custom-subdomain>.cognitiveservices.azure.com` |
| **{IMAGE_ID}** | ID d’image du conteneur. | `1.1.011300001-amd64-preview` |
| **{INPUT_DIR}** | Répertoire d’entrée du conteneur. | Windows : `C:\healthcareMount` <br> Linux/macOS : `/home/username/input` |

```bash
docker run --rm -it -p 5000:5000 --cpus 6 --memory 12g \
--mount type=bind,src={INPUT_DIR},target=/output {IMAGE_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Disk:Format=json
```

Cette commande :

- Suppose que le répertoire d’entrée existe sur l’ordinateur hôte
- Exécute un conteneur Analyse de texte pour la santé à partir de l’image conteneur
- Alloue 6 cœurs de processeur et 12 gigaoctets (Go) de mémoire
- Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur
- Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.

### <a name="demo-ui-to-visualize-output"></a>Interface utilisateur de démonstration pour visualiser la sortie

> [!NOTE]
> La démonstration est disponible uniquement avec le conteneur Analyse de texte pour la santé.

Le conteneur fournit des API de point de terminaison de prédiction de requête basées sur REST.  Nous avons également fourni un outil de visualisation dans le conteneur qui est accessible en ajoutant `/demo` au point de terminaison du conteneur. Par exemple :

```
http://<serverURL>:5000/demo
```

Utilisez l’exemple de requête cURL ci-dessous pour envoyer une requête au conteneur que vous avez déployé en remplaçant la variable `serverURL` par la valeur appropriée.

```bash
curl -X POST 'http://<serverURL>:5000/text/analytics/v3.2-preview.1/entities/health' --header 'Content-Type: application/json' --header 'accept: application/json' --data-binary @example.json

```

### <a name="install-the-container-using-azure-web-app-for-containers"></a>Installer le conteneur à l’aide d’Azure Web App pour conteneurs

Azure [Web App pour conteneurs](https://azure.microsoft.com/services/app-service/containers/) est une ressource Azure dédiée à l’exécution de conteneurs dans le Cloud. Il offre des fonctionnalités prêtes à l’emploi, telles que la mise à l’échelle automatique, la prise en charge des conteneurs Docker et la composition Docker, la prise en charge de HTTPS et bien plus encore.

> [!NOTE]
> À l’aide d’Azure Web App, vous obtiendrez automatiquement un domaine sous la forme `<appservice_name>.azurewebsites.net`

Exécutez ce script PowerShell au moyen de la Azure CLI pour créer un Web App pour conteneurs, à l’aide de votre abonnement et de l’image de conteneur sur HTTPS. Attendez la fin du script (environ 25 à 30 minutes) avant d’envoyer la première requête.

```bash
$subscription_name = ""                    # THe name of the subscription you want you resource to be created on.
$resource_group_name = ""                  # The name of the resource group you want the AppServicePlan
                                           #    and AppSerivce to be attached to.
$resources_location = ""                   # This is the location you wish the AppServicePlan to be deployed to.
                                           #    You can use the "az account list-locations -o table" command to
                                           #    get the list of available locations and location code names.
$appservice_plan_name = ""                 # This is the AppServicePlan name you wish to have.
$appservice_name = ""                      # This is the AppService resource name you wish to have.
$TEXT_ANALYTICS_RESOURCE_API_KEY = ""      # This should be taken from the Text Analytics resource.
$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT = "" # This should be taken from the Text Analytics resource.
$DOCKER_REGISTRY_SERVER_PASSWORD = ""      # This will be provided separately.
$DOCKER_REGISTRY_SERVER_USERNAME = ""      # This will be provided separately.
$DOCKER_IMAGE_NAME = "containerpreview.azurecr.io/microsoft/cognitive-services-healthcare:latest"

az login
az account set -s $subscription_name
az appservice plan create -n $appservice_plan_name -g $resource_group_name --is-linux -l $resources_location --sku P3V2
az webapp create -g $resource_group_name -p $appservice_plan_name -n $appservice_name -i $DOCKER_IMAGE_NAME -s $DOCKER_REGISTRY_SERVER_USERNAME -w $DOCKER_REGISTRY_SERVER_PASSWORD
az webapp config appsettings set -g $resource_group_name -n $appservice_name --settings Eula=accept Billing=$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT ApiKey=$TEXT_ANALYTICS_RESOURCE_API_KEY

# Once deployment complete, the resource should be available at: https://<appservice_name>.azurewebsites.net
```

### <a name="install-the-container-using-azure-container-instance"></a>Installer le conteneur à l’aide d’Azure Container Instance

Vous pouvez également utiliser Azure Container Instance (ACI) pour faciliter le déploiement. ACI est une ressource qui vous permet d’exécuter des conteneurs Docker à la demande dans un environnement Azure serverless géré. 

Consultez [Comment utiliser Azure Container Instances](../how-tos/text-analytics-how-to-use-container-instances.md) pour connaître les étapes de déploiement d’une ressource ACI à l’aide du Portail Azure. Vous pouvez également utiliser le script PowerShell ci-dessous avec Azure CLI, ce qui crée une ACI sur votre abonnement à l’aide de l’image conteneur.  Attendez la fin du script (environ 25 à 30 minutes) avant d’envoyer la première requête.  En raison de la limite du nombre maximal de processeurs par ressource ACI, ne sélectionnez pas cette option si vous envisagez de soumettre plus de 5 documents volumineux (environ 5 000 caractères chacun) par requête.
Pour plus d’informations sur la disponibilité, consultez l’article [Support régional ACI](../../../container-instances/container-instances-region-availability.md). 

> [!NOTE] 
> Azure Container Instances n’inclue pas la prise en charge HTTPS pour les domaines intégrés. Si vous avez besoin de HTTPS, vous devez le configurer manuellement, y compris créer un certificat et l’inscription d’un domaine. Vous trouverez des instructions pour effectuer cette opération avec NGINX ci-dessous.

```bash
$subscription_name = ""                    # The name of the subscription you want you resource to be created on.
$resource_group_name = ""                  # The name of the resource group you want the AppServicePlan
                                           # and AppService to be attached to.
$resources_location = ""                   # This is the location you wish the web app to be deployed to.
                                           # You can use the "az account list-locations -o table" command to
                                           # Get the list of available locations and location code names.
$azure_container_instance_name = ""        # This is the AzureContainerInstance name you wish to have.
$TEXT_ANALYTICS_RESOURCE_API_KEY = ""      # This should be taken from the Text Analytics resource.
$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT = "" # This should be taken from the Text Analytics resource.
$DOCKER_REGISTRY_SERVER_PASSWORD = ""      # This will be provided separately.
$DOCKER_REGISTRY_SERVER_USERNAME = ""      # This will be provided separately.
$DNS_LABEL = ""                            # This is the DNS label name you wish your ACI will have
$DOCKER_REGISTRY_LOGIN_SERVER = "containerpreview.azurecr.io"
$DOCKER_IMAGE_NAME = "containerpreview.azurecr.io/microsoft/cognitive-services-healthcare:latest"

az login
az account set -s $subscription_name
az container create --resource-group $resource_group_name --name $azure_container_instance_name --image $DOCKER_IMAGE_NAME --cpu 4 --memory 12 --registry-login-server $DOCKER_REGISTRY_LOGIN_SERVER --registry-username $DOCKER_REGISTRY_SERVER_USERNAME --registry-password $DOCKER_REGISTRY_SERVER_PASSWORD --port 5000 --dns-name-label $DNS_LABEL --environment-variables Eula=accept Billing=$TEXT_ANALYTICS_RESOURCE_API_ENDPOINT ApiKey=$TEXT_ANALYTICS_RESOURCE_API_KEY

# Once deployment complete, the resource should be available at: http://<unique_dns_label>.<resource_group_region>.azurecontainer.io:5000
```

### <a name="secure-aci-connectivity"></a>Sécuriser la connectivité ACI

Par défaut, aucune sécurité n’est fournie lors de l’utilisation d’ACI avec l’API de conteneur. Cela est dû au fait que les conteneurs s’exécute généralement dans le cadre d’un pod qui est protégé de l’extérieur par un pont réseau. Vous pouvez toutefois modifier un conteneur avec un composant frontal, en gardant le point de terminaison de conteneur privé. Les exemples suivants utilisent [NGINX](https://www.nginx.com) en tant que passerelle d’entrée pour prendre en charge le protocole HTTPS/SSL et l’authentification client-certificat.

> [!NOTE]
> NGINX est un serveur et un proxy HTTP open source à hautes performances. Un conteneur NGINX peut être utilisé pour mettre fin à une connexion TLS pour un seul conteneur. Des solutions de terminaison TLS basées sur l’entrée NGINX plus complexes sont également possibles.

#### <a name="set-up-nginx-as-an-ingress-gateway"></a>Configurer NGINX en tant que passerelle d’entrée

NGINX utilise des [fichiers config](https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/) pour activer les fonctionnalités lors du runtime. Pour activer la terminaison TLS pour un autre service, vous devez spécifier un certificat SSL pour mettre fin à la connexion TLS et `proxy_pass` pour spécifier une adresse pour le service. Un exemple est fourni ci-dessous.


> [!NOTE]
> `ssl_certificate` attend la spécification d’un chemin d’accès dans le système de fichiers local du conteneur NGINX. L’adresse spécifiée pour `proxy_pass` doit être disponible dans le réseau du conteneur NGINX.

Le conteneur NGINX chargera tous les fichiers dans `_.conf_` qui sont montés sous `/etc/nginx/conf.d/` dans le chemin de configuration HTTP.

```nginx
server {
  listen              80;
  return 301 https://$host$request_uri;
}
server {
  listen              443 ssl;
  # replace with .crt and .key paths
  ssl_certificate     /cert/Local.crt;
  ssl_certificate_key /cert/Local.key;

  location / {
    proxy_pass http://cognitive-service:5000;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Real-IP  $remote_addr;
  }
}
```

#### <a name="example-docker-compose-file"></a>Exemple de fichier Docker Compose

L’exemple ci-dessous montre comment créer un fichier [Docker Compose](https://docs.docker.com/compose/reference/overview) pour déployer les conteneurs NGINX et d’Analyse de texte pour la santé :

```yaml
version: "3.7"
services:
  cognitive-service:
    image: {IMAGE_ID}
    ports:
      - 5000:5000
    environment:
      - eula=accept
      - billing={ENDPOINT_URI}
      - apikey={API_KEY}
      - Logging:Disk:Format=json
    volumes:
        # replace with path to logs folder
      - <path-to-logs-folder>:/output
  nginx:
    image: nginx
    ports:
      - 443:443
    volumes:
        # replace with paths for certs and conf folders
      - <path-to-certs-folder>:/cert
      - <path-to-conf-folder>:/etc/nginx/conf.d/
```

Pour initier ce fichier Docker Compose, exécutez la commande suivante à partir d’une console au niveau racine du fichier :

```bash
docker-compose up
```

Pour plus d’informations, consultez la documentation de NGINX sur la [terminaison SSL NGINX](https://docs.nginx.com/nginx/admin-guide/security-controls/terminating-ssl-http/).

