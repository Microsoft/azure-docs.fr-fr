---
title: Exécuter votre application à partir d’un package ZIP
description: Déployez le package ZIP de votre application avec atomicité. Améliorez la prévisibilité et la fiabilité du comportement de votre application pendant le processus de déploiement ZIP.
ms.topic: article
ms.date: 01/14/2020
ms.openlocfilehash: 3440653455626af4e3705d89349a66d6bf2fbfc0
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97008127"
---
# <a name="run-your-app-in-azure-app-service-directly-from-a-zip-package"></a>Exécuter votre application dans Azure App Service directement à partir d’un package ZIP

Dans [Azure App Service](overview.md), vous pouvez exécuter vos applications directement à partir d’un fichier de package ZIP de déploiement. Cet article explique comment activer cette fonctionnalité dans votre application.

Toutes les autres méthodes de déploiement dans App Service ont un point en commun : vos fichiers sont déployés sur *D:\home\site\wwwroot* dans votre application (ou */home/site/wwwroot* pour les applications Linux). Étant donné que le même répertoire est utilisé par votre application au moment de l’exécution, il est possible que le déploiement échoue en raison de conflits de verrou de fichier et que l’application se comporte de manière imprévisible, car certains fichiers ne sont pas encore mis à jour.

En revanche, lors d’une exécution directe à partir d’un package, les fichiers du package ne sont pas copiés dans le répertoire *wwwroot*. Au lieu de cela, le package ZIP lui-même est monté directement en tant que répertoire *wwwroot* en lecture seule. L’exécution directe à partir d’un package présente plusieurs avantages :

- Élimine les conflits de verrou de fichier entre le déploiement et l’exécution.
- Garantit que seules les applications entièrement déployées sont en cours d’exécution à tout moment.
- Déploiement possible sur une application de production (après redémarrage)
- Améliore les performances des déploiements Azure Resource Manager.
- Peut réduire les temps de démarrage à froid, en particulier pour les fonctions JavaScript avec des arborescences de package npm de grande taille.

> [!NOTE]
> Actuellement, seuls les fichiers de package ZIP sont pris en charge.

[!INCLUDE [Create a project ZIP file](../../includes/app-service-web-deploy-zip-prepare.md)]

## <a name="enable-running-from-package"></a>Activer l’exécution à partir du package

Le paramètre d’application `WEBSITE_RUN_FROM_PACKAGE` active l’exécution à partir d’un package. Pour le définir, exécutez la commande suivante avec l’interface de ligne de commande Azure.

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings WEBSITE_RUN_FROM_PACKAGE="1"
```

`WEBSITE_RUN_FROM_PACKAGE="1"` vous permet d’exécuter votre application à partir d’un package local à votre application. Vous pouvez également [exécuter à partir d’un package distant](#run-from-external-url-instead).

## <a name="run-the-package"></a>Exécution du package

Le moyen le plus simple d’exécuter un package dans App Service consiste à utiliser la commande [az webapp deployment source config-zip](/cli/azure/webapp/deployment/source#az-webapp-deployment-source-config-zip) de l’interface de ligne de commande Azure. Par exemple :

```azurecli-interactive
az webapp deployment source config-zip --resource-group <group-name> --name <app-name> --src <filename>.zip
```

Étant donné que le paramètre d’application `WEBSITE_RUN_FROM_PACKAGE` est défini, cette commande n’extrait pas le contenu du package dans le répertoire *D:\home\site\wwwroot* de votre application. Au lieu de cela, elle charge le fichier ZIP tel quel dans *D:\home\data\SitePackages* et crée un fichier *packagename.txt* dans le même répertoire, qui contient le nom du package ZIP à charger au moment de l’exécution. Si vous chargez votre package ZIP d’une autre façon (comme [FTP](deploy-ftp.md)), vous devez créer le répertoire *D:\home\data\SitePackages* et le fichier *packagename.txt* manuellement.

La commande redémarre également l’application. Étant donné que `WEBSITE_RUN_FROM_PACKAGE` est défini, App Service monte le package chargé en tant que répertoire *wwwroot* en lecture seule et exécute l’application directement à partir de ce répertoire monté.

## <a name="run-from-external-url-instead"></a>Exécuter à partir d’une URL externe

Vous pouvez également exécuter un package à partir d’une URL externe, comme le stockage Blob Azure. Vous pouvez utiliser [l’Explorateur Stockage Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md) pour charger des fichiers de package sur votre compte Stockage Blob. Vous devez utiliser un conteneur de stockage privé avec une [signature d’accès partagé (SAS)](../vs-azure-tools-storage-manage-with-storage-explorer.md#generate-a-sas-in-storage-explorer) pour permettre au runtime App Service d’accéder au package en toute sécurité. 

Une fois que vous avez chargé votre fichier sur le stockage Blob et que vous avez une URL SAS pour le fichier, définissez le paramètre d’application `WEBSITE_RUN_FROM_PACKAGE` sur l’URL. Pour ce faire, l’exemple suivant utilise l’interface de ligne de commande Azure :

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings WEBSITE_RUN_FROM_PACKAGE="https://myblobstorage.blob.core.windows.net/content/SampleCoreMVCApp.zip?st=2018-02-13T09%3A48%3A00Z&se=2044-06-14T09%3A48%3A00Z&sp=rl&sv=2017-04-17&sr=b&sig=bNrVrEFzRHQB17GFJ7boEanetyJ9DGwBSV8OM3Mdh%2FM%3D"
```

Si vous publiez un package mis à jour portant le même nom sur le stockage Blob, vous devez redémarrer votre application afin que le package mis à jour soit chargé dans App Service.

## <a name="troubleshooting"></a>Dépannage

- L’exécution directe à partir d’un package met `wwwroot` en lecture seule. Votre application recevra une erreur si elle tente d’écrire des fichiers dans ce répertoire.
- Les formats TAR et GZIP ne sont pas pris en charge.
- Cette fonctionnalité n’est pas compatible avec le [cache local](overview-local-cache.md).
- Pour améliorer les performances de démarrage à froid, utilisez l’option zip locale (`WEBSITE_RUN_FROM_PACKAGE`= 1).

## <a name="more-resources"></a>Plus de ressources

- [Déploiement continu pour Azure App Service](deploy-continuous-deployment.md)
- [Déployer du code avec un fichier ZIP ou WAR](deploy-zip.md)
