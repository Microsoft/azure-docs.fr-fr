---
title: Gérer le contenu public dans un registre de conteneurs privé
description: Pratiques et workflows dans Azure Container Registry pour gérer les dépendances sur les images publiques à partir de Docker Hub et d’autres contenus publics
author: dlepow
ms.topic: article
ms.author: danlep
ms.date: 11/20/2020
ms.openlocfilehash: 0c92899528d417f9c91f8f8930ca4932dc74e850
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024736"
---
# <a name="manage-public-content-with-azure-container-registry"></a>Gérer le contenu public à l’aide d’Azure Container Registry

Cet article est une vue d’ensemble des pratiques et des workflows permettant d’utiliser un registre local, tel qu’un [registre de conteneurs Azure](container-registry-intro.md), afin de conserver des copies de contenu public, telles que les images conteneur dans Docker Hub. 


## <a name="risks-with-public-content"></a>Risques liés au contenu public

Votre environnement peut avoir des dépendances vis-à-vis de contenus publics, par exemple des images conteneur publiques, des [charts Helm](https://helm.sh/), des stratégies [Open Policy agent](https://www.openpolicyagent.org/) (OPA) ou d’autres artefacts. Par exemple, vous pouvez exécuter [nginx](https://hub.docker.com/_/nginx) pour le routage du service ou `docker build FROM alpine` en extrayant les images directement à partir de Docker Hub ou d’un autre registre public. 

En l’absence de contrôles appropriés, le fait d’être dépendant du contenu d’un registre public peut introduire des risques pour vos workflows de développement et de déploiement d’images. Pour atténuer les risques, conservez des copies locales du contenu public lorsque cela est possible. Pour plus d’informations, consultez le [blog d’Open Container Initiative](https://opencontainers.org/posts/blog/2020-10-30-consuming-public-content/). 

## <a name="authenticate-with-docker-hub"></a>S’authentifier avec Docker Hub

Dans un premier temps, si vous extrayez des images publiques de Docker Hub dans le cadre d’un workflow de création ou de déploiement, nous vous recommandons de [vous authentifier à l’aide d’un compte Docker Hub](https://docs.docker.com/docker-hub/download-rate-limit/#how-do-i-authenticate-pull-requests) au lieu de créer une demande de tirage (pull request) anonyme.

Lorsque vous effectuez des demande de tirage (pull request) anonymes fréquentes, vous pouvez voir des erreurs Docker similaires à `ERROR: toomanyrequests: Too Many Requests.` ou `You have reached your pull rate limit.` Authentifiez-vous auprès de Docker Hub pour éviter ces erreurs.

> [!NOTE]
> À compter du 2 novembre 2020, des [limites de taux de téléchargement](https://docs.docker.com/docker-hub/download-rate-limit) s’appliquent aux requêtes anonymes et authentifiées qui sont envoyées à Docker Hub à partir de comptes Docker du plan Gratuit. Ces limites sont appliquées par adresse IP et ID Docker, respectivement. 
>
> Lorsque vous estimez le nombre de demandes de tirage (pull request), prenez en compte le fait que, lorsque vous utilisez les services d’un fournisseur de cloud ou que vous travaillez derrière une NAT d’entreprise, plusieurs utilisateurs sont présentés à Docker Hub de manière agrégée comme un sous-ensemble d’adresses IP. L’ajout de l’authentification d’un compte Docker payant aux demandes adressées à Docker Hub évite les éventuelles interruptions de service dues à la limitation du débit.
>
> Pour plus d’informations, consultez [la tarification et les abonnements de Docker](https://www.docker.com/pricing) et les [conditions d’utilisation du service de Docker](https://www.docker.com/legal/docker-terms-service).

### <a name="docker-hub-access-token"></a>Jeton d’accès Docker Hub

Docker Hub prend en charge [les jetons d’accès personnels](https://docs.docker.com/docker-hub/access-tokens/) comme alternative à un mot de passe Docker lors de l’authentification auprès de Docker Hub. Les jetons sont recommandés pour les services automatisés qui extraient des images de Docker Hub. Vous pouvez générer plusieurs jetons pour différents utilisateurs ou services et révoquer les jetons quand vous n’en avez plus besoin.

Pour vous authentifier auprès de `docker login` à l’aide d’un jeton, omettez le mot de passe sur la ligne de commande. Lorsque vous êtes invité à saisir un mot de passe, entrez le jeton à la place. Si vous avez activé l’authentification à 2 facteurs pour votre compte Docker Hub, vous devez utiliser un jeton d’accès personnel lors de la connexion à partir de l’interface de commande de Docker.

### <a name="authenticate-from-azure-services"></a>S’authentifier à partir des services Azure

Plusieurs services Azure, y compris App Service et Azure Container Instances, prennent en charge l’extraction d’images à partir de registres publics tels que Docker Hub pour les déploiements de conteneurs. Si vous avez besoin de déployer une image à partir de Docker Hub, nous vous recommandons de configurer les paramètres pour vous authentifier à l’aide d’un compte Docker Hub. Exemples :

**App Service**

* **Source d’image** : Hub Docker
* **Accès au référentiel** : Privé
* **Identifiant** : \<Docker Hub username>
* **Mot de passe** : \<Docker Hub token>

Pour plus d’informations, consultez [Extractions authentifiées Docker Hub sur App Service](https://azure.github.io/AppService/2020/10/15/Docker-Hub-authenticated-pulls-on-App-Service.html).

**Azure Container Instances**

* **Source d’image** : Docker Hub ou un autre registre
* **Type d’image** : Privé
* **Serveur de connexion du registre d’images** : docker.io
* **Nom d’utilisateur du registre d’images** : \<Docker Hub username>
* **Mot de passe du registre d’images** : \<Docker Hub token>
* **Image** : docker.io/\<repo name\>:\<tag>

## <a name="import-images-to-an-azure-container-registry"></a>Importer des images dans un registre de conteneurs Azure
 
Pour commencer à gérer des copies d’images publiques, vous pouvez créer un registre de conteneurs Azure si vous n’en avez pas déjà un. Créez un registre à l’aide d’[Azure CLI](container-registry-get-started-azure-cli.md), de [Portail Azure](container-registry-get-started-portal.md), d’[Azure PowerShell](container-registry-get-started-powershell.md) ou d’autres outils. 

Comme étape unique recommandée, [importez](container-registry-import-images.md) des images de base et d’autres contenus publics dans votre registre de conteneurs Azure. La commande [az acr import](/cli/azure/acr#az_acr_import) dans Azure CLI prend en charge l’importation d’images à partir de registres publics, tels que Docker Hub et Microsoft Container Registry, et à partir d’autres registres de conteneurs privés. 

`az acr import` ne nécessite pas d’installation locale de Docker. Vous pouvez l’exécuter avec une installation locale d’Azure CLI ou directement dans Azure Cloud Shell. Elle prend en charge des images de n’importe quel type de système d’exploitation, des images à architectures multiples ou des artefacts OCI tels que les charts Helm.

Exemple :

```azurecli-interactive
az acr import \
  --name myregistry \
  --source docker.io/library/hello-world:latest \
  --image hello-world:latest \
  --username <Docker Hub username> \
  --password <Docker Hub token>
```

En fonction des besoins de votre organisation, vous pouvez importer vers un registre dédié ou vers un référentiel dans un registre partagé.

## <a name="automate-application-image-updates"></a>Automatiser les mises à jour des images d’application

Les développeurs d’images d’application doivent s’assurer que leur code fait référence au contenu local sous leur contrôle. Par exemple, une instruction `Docker FROM` dans un Dockerfile doit faire référence à une image dans un registre privé d’images de base au lieu d’un registre public. 

En plus de l’importation d’images, configurez une [tâche Azure Container Registry](container-registry-tasks-overview.md) pour automatiser les versions d’image d’application lorsque les images de base sont mises à jour. Une tâche de génération automatisée peut effectuer le suivi des [mises à jour d’image de base](container-registry-tasks-base-images.md) et des [mises à jour du code source](container-registry-tasks-overview.md#trigger-task-on-source-code-update).

Pour obtenir un exemple détaillé, consultez [Comment utiliser et gérer le contenu public à l’aide d’Azure Container Registry Tasks](tasks-consume-public-content.md). 

> [!NOTE]
> Une seule tâche préconfigurée peut régénérer automatiquement chaque image d’application qui fait référence à une image de base dépendante. 
 
## <a name="next-steps"></a>Étapes suivantes
 
* En savoir plus sur [ACR Tasks](container-registry-tasks-overview.md) pour générer, exécuter, envoi (push) et corriger des images conteneur dans Azure.
* Consultez [Comment utiliser et gérer le contenu public à l’aide d’Azure Container Registry Tasks](tasks-consume-public-content.md) pour obtenir un workflow de contrôle automatisé afin de mettre à jour les images de base dans votre environnement. 
* Pour plus d’exemples sur l’automatisation des versions d’images et des mises à jour, consultez les [tutoriels relatifs à ACR Tasks](container-registry-tutorial-quick-task.md).
