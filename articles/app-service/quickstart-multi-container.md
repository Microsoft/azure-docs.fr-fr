---
title: 'Démarrage rapide : Créer une application multiconteneur'
description: Découvrez comment bien démarrer avec les applications multiconteneurs sur Azure App Service en déployant votre première application multiconteneur.
keywords: azure app service, application web, Linux, Docker, Compose, multiconteneur, multi-conteneurs, Web App pour conteneurs, plusieurs conteneurs, conteneurs, WordPress, base de données Azure pour MySQL, base de données de production avec des conteneurs
author: msangapu-msft
ms.topic: quickstart
ms.date: 08/23/2019
ms.author: msangapu
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: 2ba42e5e800ae607631e00aee50954bf2638ae43
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007125"
---
# <a name="create-a-multi-container-preview-app-using-a-docker-compose-configuration"></a>Créer une application à plusieurs conteneurs (préversion) à l’aide d’une configuration Docker Compose

> [!NOTE]
> La fonctionnalité multiconteneur est en préversion.

[Web App pour conteneurs](overview.md#app-service-on-linux) fournit une solution souple d’utilisation des images Docker. Ce guide de démarrage rapide montre comment déployer une application multiconteneur (préversion) sur Web App pour conteneurs dans [Cloud Shell](../cloud-shell/overview.md) à l’aide d’une configuration Docker Compose.

![Exemple d’application à plusieurs conteneurs sur Web App pour conteneurs][1]

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

Cet article nécessite la version 2.0.32 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="download-the-sample"></a>Télécharger l’exemple

Pour ce guide de démarrage rapide, vous utilisez le fichier Composer de [Docker](https://docs.docker.com/compose/wordpress/#define-the-project). Le fichier de configuration se trouve dans les [exemples Azure](https://github.com/Azure-Samples/multicontainerwordpress).

[!code-yml[Main](../../azure-app-service-multi-container/docker-compose-wordpress.yml)]

Dans Cloud Shell, créez un répertoire de démarrage rapide, puis utilisez-le.

```bash
mkdir quickstart

cd $HOME/quickstart
```

Exécutez ensuite la commande suivante pour cloner le référentiel de l’exemple d’application sur votre répertoire de démarrage rapide. Accédez ensuite au répertoire `multicontainerwordpress`.

```bash
git clone https://github.com/Azure-Samples/multicontainerwordpress

cd multicontainerwordpress
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

[!INCLUDE [resource group intro text](../../includes/resource-group.md)]

Dans Cloud Shell, créez un groupe de ressources avec la commande [`az group create`](/cli/azure/group#az-group-create). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *USA Centre Sud*. Pour afficher tous les emplacements pris en charge pour App Service sous Linux au niveau **Standard**, exécutez la commande [`az appservice list-locations --sku S1 --linux-workers-enabled`](/cli/azure/appservice#az-appservice-list-locations).

```azurecli-interactive
az group create --name myResourceGroup --location "South Central US"
```

Vous créez généralement votre groupe de ressources et les ressources dans une région proche de chez vous.

Une fois la commande terminée, une sortie JSON affiche les propriétés du groupe de ressources.

## <a name="create-an-azure-app-service-plan"></a>Créer un plan Azure App Service

Dans Cloud Shell, créez un plan App Service dans le groupe de ressources avec la commande [`az appservice plan create`](/cli/azure/appservice/plan#az-appservice-plan-create).

L’exemple suivant crée un plan App Service nommé `myAppServicePlan` dans le niveau tarifaire **Standard** (`--sku S1`) et un conteneur Linux (`--is-linux`).

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux
```

Lorsque le plan App Service est créé, l’interface Azure CLI affiche des informations similaires à l’exemple suivant :

<pre>
{
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "South Central US",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "linux",
  "location": "South Central US",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  &lt; JSON data removed for brevity. &gt;
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
}
</pre>

## <a name="create-a-docker-compose-app"></a>Création d’une application Docker Compose

> [!NOTE]
> Docker Compose sur Azure App Services a actuellement une limite de 4000 caractères.

Dans Cloud Shell, créez une [application web](overview.md#app-service-on-linux) multiconteneur dans le plan App Service `myAppServicePlan` avec la commande [az webapp create](/cli/azure/webapp#az-webapp-create). N’oubliez pas de remplacer _\<app_name>_ par un nom d’application unique (les caractères valides sont `a-z`, `0-9` et `-`).

```azurecli
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --multicontainer-config-type compose --multicontainer-config-file compose-wordpress.yml
```

Une fois l’application web créée, Azure CLI affiche une sortie similaire à l’exemple suivant :

<pre>
{
  "additionalProperties": {},
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "&lt;app_name&gt;.azurewebsites.net",
  "enabled": true,
  &lt; JSON data removed for brevity. &gt;
}
</pre>

### <a name="browse-to-the-app"></a>Accéder à l’application

Accédez à l’application déployée dans (`http://<app_name>.azurewebsites.net`). Le chargement de l’application peut prendre plusieurs minutes. Si vous recevez une erreur, attendez quelques minutes supplémentaires, puis actualisez le navigateur.

![Exemple d’application à plusieurs conteneurs sur Web App pour conteneurs][1]

**Félicitations**, vous avez créé une application à plusieurs conteneurs dans Web App pour conteneurs.

[!INCLUDE [Clean-up section](../../includes/cli-script-clean-up.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Application WordPress avec plusieurs conteneurs](tutorial-multi-container-app.md)

> [!div class="nextstepaction"]
> [Configurer un conteneur personnalisé](configure-custom-container.md)

<!--Image references-->
[1]: media/tutorial-multi-container-app/azure-multi-container-wordpress-install.png
