---
title: 'Tutoriel : Créer un serveur flexible Azure Database pour MySQL (préversion) et une application web Azure App Service dans le même réseau virtuel'
description: Guide de démarrage rapide pour créer un serveur flexible Azure Database pour MySQL (préversion) avec une application web dans un réseau virtuel
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 9/21/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: a7b673dc8dfeb2ebf86aec5b7449df91c2ffd635
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92534054"
---
# <a name="tutorial-create-an-azure-database-for-mysql---flexible-server-preview-with-app-services-web-app-in-virtual-network"></a>Tutoriel : Créer un serveur flexible Azure Database pour MySQL (préversion) avec une application web App Services dans le même réseau virtuel

> [!IMPORTANT]
> Azure Database pour MySQL – Serveur flexible est actuellement en préversion publique.

Ce didacticiel vous montre comment créer une application web Azure App Service avec un serveur flexible MySQL (préversion) à l’intérieur d’un [réseau virtuel](../../virtual-network/virtual-networks-overview.md).

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

Cet article nécessite que vous exécutiez localement Azure CLI version 2.0 ou ultérieure. Pour afficher la version installée, exécutez la commande `az --version`. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

Vous devrez vous connecter à votre compte à l’aide de la commande [az login](/cli/azure/reference-index#az-login). Notez la propriété **id** depuis la sortie de commande pour le nom d’abonnement correspondant.

```azurecli
az login
```

Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié dans lequel la ressource doit être facturée. Sélectionnez l’ID d’abonnement spécifique sous votre compte à l’aide de la commande [az account set](/cli/azure/account). Remplacez la propriété **ID d’abonnement** de la sortie **az login** pour votre abonnement dans l’espace réservé de l’ID d’abonnement.

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-an-azure-database-for-mysql-flexible-server"></a>Créer un serveur flexible Azure Database pour MySQL

Créez un serveur flexible privé à l’intérieur d’un réseau virtuel à l’aide de la commande suivante :
```azurecli
az mysql flexible-server create --resource-group myresourcegroup --location westus2
```
Cette commande effectue les actions suivantes qui peuvent prendre quelques minutes :

- S'il n'existe pas encore, créez le groupe de ressources.
- Génère un nom de serveur s’il n’est pas fourni.
- Créez un réseau virtuel pour votre nouveau serveur MySQL. Notez le nom du réseau virtuel et le nom du sous-réseau créés pour votre serveur, car vous devez ajouter l’application web au même réseau virtuel.
- Crée un nom d’utilisateur administrateur, un mot de passe pour votre serveur, s’il n’est pas fourni.
- Crée une base de données vide appelée **flexibleserverdb**

> [!NOTE]
> Notez le mot de passe qui sera généré automatiquement s’il n’est pas fourni. Si vous oubliez le mot de passe, vous devrez le réinitialiser à l’aide de la commande ``` az mysql flexible-server update```

## <a name="create-a-web-app"></a>Créer une application web

Dans cette section, vous allez créer l’hôte d’application dans l’application App Service et connecter cette application à la base de données MySQL. Assurez-vous que vous êtes à la racine du dépôt de votre code d’application dans le terminal.

Créez une application App Service (processus hôte) avec la commande az webapp up.

```azurecli
az webapp up --resource-group myresourcegroup --location westus2 --plan testappserviceplan --sku B1 --name mywebapp
```

> [!NOTE]
> - Pour l’argument --location, utilisez le même emplacement que pour la base de données dans la section précédente.
> - Remplacez _&lt;app-name>_ par le nom unique dans Azure (le point de terminaison du serveur est https://\<app-name>.azurewebsites.net). Les caractères autorisés pour <app-name> sont A-Z, 0-9 et -. Un bon modèle consiste à utiliser une combinaison du nom de votre société et d’un identificateur d’application.

Cette commande effectue les actions suivantes qui peuvent prendre quelques minutes :

- S'il n'existe pas encore, créez le groupe de ressources. (dans cette commande, vous utilisez le groupe de ressources dans lequel vous avez créé la base de données précédemment).
- Créez le plan App Service ```testappserviceplan``` au niveau tarifaire De base (B1), le cas échéant. --plan et --sku sont facultatifs.
- Si elle n’existe pas, créez l’application App Service.
- Activez la journalisation par défaut pour l’application, si elle ne l’est pas déjà.
- Chargez le dépôt à l’aide du déploiement ZIP avec l’automatisation de la génération activée.

## <a name="add-the-web-app-to-the-virtual-network"></a>Ajouter l’application web au réseau virtuel

Utilisez la commande **az webapp vnet-integration** pour ajouter une intégration de réseau virtuel régional à une application web. Remplacez _&lt;vnet-name >_ et _&lt;subnet-name_ par le nom du réseau virtuel et le nom du sous-réseau que le serveur flexible utilise.

```azurecli
az webapp vnet-integration add -g myresourcegroup -n  mywebapp --vnet <vnet-name> --subnet <subnet-name>
```

## <a name="configure-environment-variables-to-connect-the-database"></a>Configurer des variables d’environnement pour connecter la base de données

Le code étant maintenant déployé sur App Service, l'étape suivante consiste à connecter l'application au serveur flexible dans Azure. Le code de l’application s’attend à trouver des informations sur la base de données dans plusieurs variables d’environnement. Pour définir des variables d’environnement dans App Service, vous créez des « paramètres d’application » à l’aide de la commande```az webapp config appsettings``` set.

```azurecli
az webapp config appsettings set --settings DBHOST="<mysql-server-name>.mysql.database.azure.com" DBNAME="flexibleserverdb" DBUSER="<username>" DBPASS="<password>"
```

- Remplacez _&lt;mysql-server-name>_ , _&lt;username>_ et _&lt;password>_ pour la commande du serveur flexible nouvellement créé.
- Remplacez _&lt;username>_ et _&lt;password>_ par les informations d’identification que la commande a également générées pour vous.
- Les noms du groupe de ressources et de l’application sont tirés des valeurs mises en cache dans le fichier .azure/config.
- La commande crée des paramètres nommés DBHOST, DBNAME, DBUSER et DBPASS. Si votre code d’application utilise un nom différent pour les informations de base de données, utilisez ces noms pour les paramètres de l’application, comme indiqué dans le code.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Supprimez toutes les ressources que vous avez créées dans le tutoriel à l'aide de la commande suivante. Cette commande supprime toutes les ressources de ce groupe de ressources.

```azurecli
az group delete -n myresourcegroup
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Mapper un nom DNS personnalisé existant à Azure App Service](../../app-service/app-service-web-tutorial-custom-domain.md)