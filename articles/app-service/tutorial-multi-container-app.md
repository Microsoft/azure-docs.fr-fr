---
title: 'Tutoriel : Créer une application multiconteneur'
description: Découvrez comment créer et utiliser une application multiconteneur dans Azure App Service qui contient une application WordPress et un conteneur MySQL, et comment configurer l’application WordPress.
keywords: azure app service, application web, Linux, Docker, Compose, multiconteneur, multi-conteneurs, Web App pour conteneurs, plusieurs conteneurs, conteneurs, WordPress, base de données Azure pour MySQL, base de données de production avec des conteneurs
author: msangapu-msft
ms.topic: tutorial
ms.date: 10/31/2020
ms.author: msangapu
ms.custom: cli-validate, devx-track-azurecli
ms.openlocfilehash: e73d681cadaddd32290cec40ae7a9c8f6e8ac758
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97005645"
---
# <a name="tutorial-create-a-multi-container-preview-app-in-web-app-for-containers"></a>Tutoriel : Créer une application multiconteneur (préversion) dans Web App pour conteneurs

> [!NOTE]
> La fonctionnalité multiconteneur est en préversion.

[Web App pour conteneurs](overview.md#app-service-on-linux) fournit une solution souple d’utilisation des images Docker. Dans ce didacticiel, vous allez apprendre à créer une application à plusieurs conteneurs à l’aide de WordPress et de MySQL. Vous allez effectuer ce tutoriel dans Cloud Shell, mais vous pouvez également exécuter ces commandes localement avec l’outil en ligne de commande [Azure CLI](/cli/azure/install-azure-cli) (2.0.32 ou ultérieur).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Convertir une configuration Docker Compose pour travailler avec Web App pour conteneurs
> * Déployer une application à plusieurs conteneurs vers Azure
> * Ajouter des paramètres d’application
> * Utiliser le stockage persistant pour vos conteneurs
> * Se connecter à la base de données Azure pour MySQL
> * Résoudre les erreurs

[!INCLUDE [Free trial note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez connaître [Docker Compose](https://docs.docker.com/compose/).

## <a name="download-the-sample"></a>Télécharger l’exemple

Pour ce tutoriel, vous utilisez le fichier de composition de [Docker](https://docs.docker.com/compose/wordpress/#define-the-project), mais vous le modifiez pour inclure Azure Database pour MySQL, le stockage persistant et Redis. Le fichier de configuration se trouve dans les [exemples Azure](https://github.com/Azure-Samples/multicontainerwordpress). Pour connaître les options de configuration prises en charge, consultez [Options Docker Compose](configure-custom-container.md#docker-compose-options).

[!code-yml[Main](../../azure-app-service-multi-container/docker-compose-wordpress.yml)]

Dans Cloud Shell, créez un répertoire pour le tutoriel, puis accédez à ce répertoire.

```bash
mkdir tutorial

cd tutorial
```

Exécutez ensuite la commande suivante pour cloner le dépôt de l’exemple d’application dans le répertoire de votre tutoriel. Accédez ensuite au répertoire `multicontainerwordpress`.

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

<!-- [!INCLUDE [app-service-plan](app-service-plan-linux.md)] -->

L’exemple suivant crée un plan App Service nommé `myAppServicePlan` dans le niveau tarifaire **Standard** (`--sku S1`) et un conteneur Linux (`--is-linux`).

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux
```

Une fois le plan App Service créé, Cloud Shell affiche des informations similaires à l’exemple suivant :

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

### <a name="docker-compose-with-wordpress-and-mysql-containers"></a>Docker Compose avec conteneurs WordPress et MySQL

## <a name="create-a-docker-compose-app"></a>Création d’une application Docker Compose

Dans Cloud Shell, créez une [application web](overview.md) multiconteneur dans le plan App Service `myAppServicePlan` avec la commande [az webapp create](/cli/azure/webapp#az-webapp-create). N’oubliez pas de remplacer _\<app-name>_ par un nom d’application unique.

```azurecli-interactive
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

Une fois l’application web créée, Cloud Shell affiche une sortie similaire à l’exemple suivant :

<pre>
{
  "additionalProperties": {},
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "&lt;app-name&gt;.azurewebsites.net",
  "enabled": true,
  &lt; JSON data removed for brevity. &gt;
}
</pre>

### <a name="browse-to-the-app"></a>Accéder à l’application

Accédez à l’application déployée dans (`http://<app-name>.azurewebsites.net`). Le chargement de l’application peut prendre plusieurs minutes. Si vous recevez une erreur, attendez quelques minutes supplémentaires, puis actualisez le navigateur. Si vous rencontrez des difficultés et souhaitez résoudre les problèmes, consultez les [journaux d’activité du conteneur](#find-docker-container-logs).

![Exemple d’application à plusieurs conteneurs sur Web App pour conteneurs][1]

**Félicitations**, vous avez créé une application à plusieurs conteneurs dans Web App pour conteneurs. Ensuite, vous devez configurer votre application pour utiliser Azure Database pour MySQL. N’installez pas WordPress pour le moment.

## <a name="connect-to-production-database"></a>Connexion à la base de données de production

Il n’est pas recommandé d’utiliser des conteneurs de base de données dans un environnement de production. Les conteneurs locaux ne sont pas évolutifs. Au lieu de cela, vous allez utiliser Azure Database pour MySQL qui peut être mis à l’échelle.

### <a name="create-an-azure-database-for-mysql-server"></a>Création d’un serveur Azure Database pour MySQL

Créez un serveur Azure Database pour MySQL avec la commande [`az mysql server create`](/cli/azure/mysql/server#az-mysql-server-create).

Dans la commande suivante, indiquez le nom unique de votre propre serveur MySQL là où se trouve l’espace réservé _&lt;mysql-server-name>_ (les caractères valides sont `a-z`, `0-9` et `-`). Ce nom fait partie du nom d’hôte du serveur MySQL (`<mysql-server-name>.database.windows.net`) et doit donc être globalement unique.

```azurecli-interactive
az mysql server create --resource-group myResourceGroup --name <mysql-server-name>  --location "South Central US" --admin-user adminuser --admin-password My5up3rStr0ngPaSw0rd! --sku-name B_Gen5_1 --version 5.7
```

La création du serveur peut prendre quelques minutes. Une fois le serveur MySQL créé, Cloud Shell affiche des informations similaires à l’exemple suivant :

<pre>
{
  "administratorLogin": "adminuser",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "&lt;mysql-server-name&gt;.database.windows.net",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/&lt;mysql-server-name&gt;",
  "location": "southcentralus",
  "name": "&lt;mysql-server-name&gt;",
  "resourceGroup": "myResourceGroup",
  ...
}
</pre>

### <a name="configure-server-firewall"></a>Configuration d’un pare-feu de serveur

Créez une règle de pare-feu pour votre serveur MySQL afin d’autoriser les connexions client à l’aide de la commande [`az mysql server firewall-rule create`](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-create). Lorsque les adresses IP de début et de fin sont définies sur 0.0.0.0, le pare-feu est ouvert uniquement pour les autres ressources Azure.

```azurecli-interactive
az mysql server firewall-rule create --name allAzureIPs --server <mysql-server-name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP]
> Vous pouvez être encore plus restrictif dans votre règle de pare-feu en [choisissant uniquement les adresses IP sortantes que votre application utilise](overview-inbound-outbound-ips.md#find-outbound-ips).
>

### <a name="create-the-wordpress-database"></a>Création de la base de données WordPress

```azurecli-interactive
az mysql db create --resource-group myResourceGroup --server-name <mysql-server-name> --name wordpress
```

Une fois la base de données créée, Cloud Shell affiche des informations similaires à l’exemple suivant :

<pre>
{
  "additionalProperties": {},
  "charset": "latin1",
  "collation": "latin1_swedish_ci",
  "id": "/subscriptions/12db1644-4b12-4cab-ba54-8ba2f2822c1f/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/&lt;mysql-server-name&gt;/databases/wordpress",
  "name": "wordpress",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DBforMySQL/servers/databases"
}
</pre>

### <a name="configure-database-variables-in-wordpress"></a>Configuration des variables de la base de données dans WordPress

Pour connecter l’application WordPress à ce nouveau serveur MySQL, vous devez configurer quelques variables d’environnement spécifiques à WordPress, y compris le chemin d’accès de l’autorité de certification SSL défini par `MYSQL_SSL_CA`. Le [certificat racine CyberTrust Baltimore](https://www.digicert.com/digicert-root-certificates.htm) de [DigiCert](https://www.digicert.com/) est fourni dans [l’image personnalisée](#use-a-custom-image-for-mysql-ssl-and-other-configurations) ci-dessous.

Pour effectuer ces modifications, utilisez la commande [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set) dans Cloud Shell. Les paramètres d’application respectent la casse et sont séparés par des espaces.

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WORDPRESS_DB_HOST="<mysql-server-name>.mysql.database.azure.com" WORDPRESS_DB_USER="adminuser@<mysql-server-name>" WORDPRESS_DB_PASSWORD="My5up3rStr0ngPaSw0rd!" WORDPRESS_DB_NAME="wordpress" MYSQL_SSL_CA="BaltimoreCyberTrustroot.crt.pem"
```

Une fois le paramètre d’application créé, Cloud Shell affiche des informations similaires à l’exemple suivant :

<pre>
[
  {
    "name": "WORDPRESS_DB_HOST",
    "slotSetting": false,
    "value": "&lt;mysql-server-name&gt;.mysql.database.azure.com"
  },
  {
    "name": "WORDPRESS_DB_USER",
    "slotSetting": false,
    "value": "adminuser@&lt;mysql-server-name&gt;"
  },
  {
    "name": "WORDPRESS_DB_NAME",
    "slotSetting": false,
    "value": "wordpress"
  },
  {
    "name": "WORDPRESS_DB_PASSWORD",
    "slotSetting": false,
    "value": "My5up3rStr0ngPaSw0rd!"
  },
  {
    "name": "MYSQL_SSL_CA",
    "slotSetting": false,
    "value": "BaltimoreCyberTrustroot.crt.pem"
  }
]
</pre>

Pour plus d’informations sur les variables d’environnement, consultez [Configurer des variables d’environnement](configure-custom-container.md#configure-environment-variables).

### <a name="use-a-custom-image-for-mysql-ssl-and-other-configurations"></a>Utilisation d’une image personnalisée pour le protocole SSL de MySQL et d’autres configurations

Par défaut, SSL est utilisé par Azure Database pour MySQL. WordPress nécessite une configuration supplémentaire pour utiliser SSL avec MySQL. « L’image officielle » de WordPress ne fournit pas la configuration supplémentaire, mais une [image personnalisée](https://github.com/Azure-Samples/multicontainerwordpress) a été préparée pour vous simplifier la tâche. En pratique, vous devez ajouter les modifications souhaitées à votre propre image.

L’image personnalisée est basée sur « l’image officielle » de [WordPress provenant du Hub Docker](https://hub.docker.com/_/wordpress/). Les modifications suivantes ont été apportées dans cette image personnalisée pour Azure Database pour MySQL :

* [Ajoute le fichier de certificat racine de CyberTrust Baltimore pour SSL à MySQL.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L61)
* [Utilise le paramètre d’application pour le certificat d’autorité de certification SSL de MySQL dans le fichier WordPress wp-config.php.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L163)
* [Ajoute une définition WordPress pour MYSQL_CLIENT_FLAGS (requise pour SSL de MySQL).](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L164)

Les modifications suivantes ont été apportées pour Redis (à utiliser dans une section ultérieure) :

* [Ajoute une extension PHP pour Redis v4.0.2.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/Dockerfile#L35)
* [Ajoute la décompression requise pour l’extraction du fichier.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L71)
* [Ajoute le plug-in WordPress Redis Object Cache 1.3.8.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L74)
* [Utilise le paramètre d’application pour le nom d’hôte Redis dans le fichier WordPress wp-config.php.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L162)

Pour utiliser l’image personnalisée, vous devez mettre à jour votre fichier docker-compose-wordpress.yml. Dans Cloud Shell, tapez `nano docker-compose-wordpress.yml` pour ouvrir l’éditeur de texte nano. Remplacez `image: wordpress` par `image: mcr.microsoft.com/azuredocs/multicontainerwordpress`. Vous n’avez plus besoin du conteneur de base de données. Supprimer les sections `db`, `environment`, `depends_on` et `volumes` du fichier de configuration. Votre fichier doit ressembler au code suivant :

```yaml
version: '3.3'

services:
   wordpress:
     image: mcr.microsoft.com/azuredocs/multicontainerwordpress
     ports:
       - "8000:80"
     restart: always
```

Enregistrez vos modifications et quittez nano. Utilisez la commande `^O` pour enregistrer et `^X` pour quitter.

### <a name="update-app-with-new-configuration"></a>Mise à jour avec la nouvelle configuration

Dans Cloud Shell, reconfigurez votre [application web](overview.md) multiconteneur avec la commande [az webapp config container set](/cli/azure/webapp/config/container#az-webapp-config-container-set). N’oubliez pas de remplacer _\<app-name>_ par le nom de l’application web créée précédemment.

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

Une fois l’application reconfigurée, Cloud Shell affiche des informations similaires à celles de l’exemple suivant :

<pre>
[
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICB3b3JkcHJlc3M6CiAgICAgaW1hZ2U6IG1zYW5nYXB1L3dvcmRwcmVzcwogICAgIHBvcnRzOgogICAgICAgLSAiODAwMDo4MCIKICAgICByZXN0YXJ0OiBhbHdheXM="
  }
]
</pre>

### <a name="browse-to-the-app"></a>Accéder à l’application

Accédez à l’application déployée dans (`http://<app-name>.azurewebsites.net`). L’application utilise maintenant Azure Database pour MySQL.

![Exemple d’application multiconteneur sur Web App pour conteneurs][1]

## <a name="add-persistent-storage"></a>Ajout de stockage persistant

Votre conteneur à plusieurs applications s’exécute maintenant dans Web App pour conteneurs. Toutefois, si vous installez WordPress maintenant et redémarrez votre application plus tard, vous verrez que l’installation de WordPress a disparu. Cela se produit parce que votre configuration Docker Compose pointe actuellement vers un emplacement de stockage à l’intérieur de votre conteneur. Les fichiers installés dans le conteneur ne sont pas conservés après le redémarrage de l’application. Dans cette section, vous allez [ajouter du stockage persistant](configure-custom-container.md#use-persistent-shared-storage) à votre conteneur WordPress.

### <a name="configure-environment-variables"></a>Configuration des variables d’environnement

Pour utiliser le stockage persistant, vous devez activer ce paramètre dans App Service. Pour faire cette modification, utilisez la commande [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set) dans Cloud Shell. Les paramètres d’application respectent la casse et sont séparés par des espaces.

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=TRUE
```

Une fois le paramètre d’application créé, Cloud Shell affiche des informations similaires à l’exemple suivant :

<pre>
[
  &lt; JSON data removed for brevity. &gt;
  {
    "name": "WORDPRESS_DB_NAME",
    "slotSetting": false,
    "value": "wordpress"
  },
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "TRUE"
  }
]
</pre>

### <a name="modify-configuration-file"></a>Modification du fichier de configuration

Dans Cloud Shell, tapez `nano docker-compose-wordpress.yml` pour ouvrir l’éditeur de texte nano.

L’option `volumes` mappe le système de fichiers sur un répertoire dans le conteneur. `${WEBAPP_STORAGE_HOME}` est une variable d’environnement d’App Service mappée sur le stockage persistant de votre application. Vous allez utiliser cette variable d’environnement dans l’option des volumes pour que les fichiers WordPress soient installés dans le stockage persistant plutôt que dans le conteneur. Apportez les modifications suivantes au fichier :

Dans la section `wordpress`, ajoutez une option `volumes` afin d’obtenir le code suivant :

```yaml
version: '3.3'

services:
   wordpress:
     image: mcr.microsoft.com/azuredocs/multicontainerwordpress
     volumes:
      - ${WEBAPP_STORAGE_HOME}/site/wwwroot:/var/www/html
     ports:
       - "8000:80"
     restart: always
```

### <a name="update-app-with-new-configuration"></a>Mise à jour avec la nouvelle configuration

Dans Cloud Shell, reconfigurez votre [application web](overview.md) multiconteneur avec la commande [az webapp config container set](/cli/azure/webapp/config/container#az-webapp-config-container-set). N’oubliez pas de remplacer _\<app-name>_ par un nom d’application unique.

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

Après son exécution, la commande retourne un résultat semblable à l’exemple suivant :

<pre>
[
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "TRUE"
  },
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICBteXNxbDoKICAgICBpbWFnZTogbXlzcWw6NS43CiAgICAgdm9sdW1lczoKICAgICAgIC0gZGJfZGF0YTovdmFyL2xpYi9teXNxbAogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgTVlTUUxfUk9PVF9QQVNTV09SRDogZXhhbXBsZXBhc3MKCiAgIHdvcmRwcmVzczoKICAgICBkZXBlbmRzX29uOgogICAgICAgLSBteXNxbAogICAgIGltYWdlOiB3b3JkcHJlc3M6bGF0ZXN0CiAgICAgcG9ydHM6CiAgICAgICAtICI4MDAwOjgwIgogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgV09SRFBSRVNTX0RCX1BBU1NXT1JEOiBleGFtcGxlcGFzcwp2b2x1bWVzOgogICAgZGJfZGF0YTo="
  }
]
</pre>

### <a name="browse-to-the-app"></a>Accéder à l’application

Accédez à l’application déployée dans (`http://<app-name>.azurewebsites.net`).

Le conteneur WordPress utilise maintenant Azure Database pour MySQL et le stockage persistant.

## <a name="add-redis-container"></a>Ajout d’un conteneur Redis

 « L’image officielle » de WordPress n’inclut pas les dépendances pour Redis. Ces dépendances et la configuration supplémentaire requise pour utiliser Redis avec WordPress ont été préparées pour vous dans cette [image personnalisée](https://github.com/Azure-Samples/multicontainerwordpress). En pratique, vous devez ajouter les modifications souhaitées à votre propre image.

L’image personnalisée est basée sur « l’image officielle » de [WordPress provenant du Hub Docker](https://hub.docker.com/_/wordpress/). Les modifications suivantes ont été apportées dans cette image personnalisée pour Redis :

* [Ajoute une extension PHP pour Redis v4.0.2.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/Dockerfile#L35)
* [Ajoute la décompression requise pour l’extraction du fichier.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L71)
* [Ajoute le plug-in WordPress Redis Object Cache 1.3.8.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L74)
* [Utilise le paramètre d’application pour le nom d’hôte Redis dans le fichier WordPress wp-config.php.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L162)

Ajoutez le conteneur redis en bas du fichier de configuration afin qu’il ressemble à l’exemple suivant :

```yaml
version: '3.3'

services:
   wordpress:
     image: mcr.microsoft.com/azuredocs/multicontainerwordpress
     ports:
       - "8000:80"
     restart: always

   redis:
     image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
     environment: 
      - ALLOW_EMPTY_PASSWORD=yes
     restart: always
```

### <a name="configure-environment-variables"></a>Configuration des variables d’environnement

Pour utiliser Redis, vous devez allez devoir activer ce paramètre, `WP_REDIS_HOST`, dans App Service. Pour que WordPress communique avec l’hôte Redis, ce *paramètre est obligatoire*. Pour faire cette modification, utilisez la commande [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set) dans Cloud Shell. Les paramètres d’application respectent la casse et sont séparés par des espaces.

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WP_REDIS_HOST="redis"
```

Une fois le paramètre d’application créé, Cloud Shell affiche des informations similaires à l’exemple suivant :

<pre>
[
  &lt; JSON data removed for brevity. &gt;
  {
    "name": "WORDPRESS_DB_USER",
    "slotSetting": false,
    "value": "adminuser@&lt;mysql-server-name&gt;"
  },
  {
    "name": "WP_REDIS_HOST",
    "slotSetting": false,
    "value": "redis"
  }
]
</pre>

### <a name="update-app-with-new-configuration"></a>Mise à jour avec la nouvelle configuration

Dans Cloud Shell, reconfigurez votre [application web](overview.md) multiconteneur avec la commande [az webapp config container set](/cli/azure/webapp/config/container#az-webapp-config-container-set). N’oubliez pas de remplacer _\<app-name>_ par un nom d’application unique.

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file compose-wordpress.yml
```

Après son exécution, la commande retourne un résultat semblable à l’exemple suivant :

<pre>
[
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICBteXNxbDoKICAgICBpbWFnZTogbXlzcWw6NS43CiAgICAgdm9sdW1lczoKICAgICAgIC0gZGJfZGF0YTovdmFyL2xpYi9teXNxbAogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgTVlTUUxfUk9PVF9QQVNTV09SRDogZXhhbXBsZXBhc3MKCiAgIHdvcmRwcmVzczoKICAgICBkZXBlbmRzX29uOgogICAgICAgLSBteXNxbAogICAgIGltYWdlOiB3b3JkcHJlc3M6bGF0ZXN0CiAgICAgcG9ydHM6CiAgICAgICAtICI4MDAwOjgwIgogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgV09SRFBSRVNTX0RCX1BBU1NXT1JEOiBleGFtcGxlcGFzcwp2b2x1bWVzOgogICAgZGJfZGF0YTo="
  }
]
</pre>

### <a name="browse-to-the-app"></a>Accéder à l’application

Accédez à l’application déployée dans (`http://<app-name>.azurewebsites.net`).

Suivez la procédure et installez WordPress.

### <a name="connect-wordpress-to-redis"></a>Connexion de WordPress à Redis

Connectez-vous à l’interface d’administration de WordPress. Dans le volet de navigation gauche, sélectionnez **Plugins** (Extensions), puis sélectionnez **Installed Plugins** (Extensions installées).

![Sélection des extensions WordPress][2]

Afficher toutes les extensions ici

Dans la page des extensions, recherchez **Redis Object Cache** et cliquez sur **Activate** (Activer).

![Activation de Redis][3]

Cliquez sur **Paramètres**.

![Cliquez sur Settings (Paramètres)][4]

Cliquez sur le bouton **Enable Object Cache** (Activer Object Cache).

![Cliquez sur le bouton Enable Object Cache (Activer Object Cache).][5]

WordPress se connecte au serveur Redis. **L’état** de la connexion apparaît sur la même page.

![WordPress se connecte au serveur Redis. **L’état** de la connexion apparaît sur la même page.][6]

**Félicitations**, vous avez connecté WordPress à Redis. L’application prête pour la production utilise maintenant **Azure Database pour MySQL, le stockage persistant et Redis**. Vous pouvez maintenant effectuer un scale-out de votre plan App Service à plusieurs instances.

## <a name="find-docker-container-logs"></a>Recherche des journaux d’activité de conteneur Docker

Si vous rencontrez des problèmes avec l’utilisation de plusieurs conteneurs, vous pouvez accéder aux journaux d’activité dans : `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.

Le résultat ressemble à l’exemple suivant :

<pre>
[
   {
      "machineName":"RD00XYZYZE567A",
      "lastUpdated":"2018-05-10T04:11:45Z",
      "size":25125,
      "href":"https://&lt;app-name&gt;.scm.azurewebsites.net/api/vfs/LogFiles/2018_05_10_RD00XYZYZE567A_docker.log",
      "path":"/home/LogFiles/2018_05_10_RD00XYZYZE567A_docker.log"
   }
]
</pre>

Vous voyez un journal pour chaque conteneur et un journal supplémentaire pour le processus parent. Copiez la valeur `href` respective dans le navigateur pour afficher le journal.

[!INCLUDE [Clean-up section](../../includes/cli-script-clean-up.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :
> [!div class="checklist"]
> * Convertir une configuration Docker Compose pour travailler avec Web App pour conteneurs
> * Déployer une application à plusieurs conteneurs vers Azure
> * Ajouter des paramètres d’application
> * Utiliser le stockage persistant pour vos conteneurs
> * Se connecter à la base de données Azure pour MySQL
> * Résoudre les erreurs

Passez au didacticiel suivant pour découvrir comment mapper un nom DNS personnalisé à votre application.

> [!div class="nextstepaction"]
> [Tutoriel : Mappage d’un nom DNS personnalisé à une application](app-service-web-tutorial-custom-domain.md)

Ou consultez les autres ressources :

> [!div class="nextstepaction"]
> [Configurer un conteneur personnalisé](configure-custom-container.md)

<!--Image references-->
[1]: ./media/tutorial-multi-container-app/azure-multi-container-wordpress-install.png
[2]: ./media/tutorial-multi-container-app/wordpress-plugins.png
[3]: ./media/tutorial-multi-container-app/activate-redis.png
[4]: ./media/tutorial-multi-container-app/redis-settings.png
[5]: ./media/tutorial-multi-container-app/enable-object-cache.png
[6]: ./media/tutorial-multi-container-app/redis-connected.png
