---
title: Déployer du code avec un fichier ZIP ou WAR
description: Découvrez comment déployer votre application sur Azure App Service avec un fichier ZIP (ou un fichier WAR pour les développeurs Java).
ms.topic: article
ms.date: 08/12/2019
ms.reviewer: sisirap
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 150f6b57f1dec0b6d925ef53b4a7001ae9f23607
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007906"
---
# <a name="deploy-your-app-to-azure-app-service-with-a-zip-or-war-file"></a>Déployer votre application sur Azure App Service avec un fichier ZIP ou WAR

Cet article montre comment utiliser un fichier ZIP ou WAR pour déployer votre application web sur [Azure App Service](overview.md). 

Ce déploiement de fichier ZIP utilise le même service Kudu que celui qui pilote les déploiements continus basés sur l’intégration. Kudu prend en charge les fonctionnalités suivantes pour le déploiement de fichier ZIP : 

- Suppression des fichiers laissés par un déploiement précédent
- Option pour activer le processus de génération par défaut, qui inclut la restauration de package
- Personnalisation du déploiement, notamment exécution de scripts de déploiement  
- Journaux d’activité de déploiement 
- La taille de fichier ne doit pas dépasser 2 048 Mo.

Pour plus d’informations, consultez la [documentation Kudu](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file).

Le fichier [WAR](https://wikipedia.org/wiki/WAR_(file_format)) est déployé sur App Service pour exécuter votre application web Java. Consultez [Déployer un fichier WAR](#deploy-war-file).

> [!NOTE]
> Quand vous utilisez `ZipDeploy`, les fichiers sont copiés uniquement si leurs timestamps ne correspondent pas à ce qui est déjà déployé. La génération d’un fichier zip selon un processus de build qui met en cache les sorties contribue à accélérer les déploiements. Pour plus d’informations, consultez [Déploiement à partir d’un fichier zip ou d’une URL](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url).

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de cet article, [créez une application App Service](./index.yml) ou utilisez une application créée pour un autre didacticiel.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [Create a project ZIP file](../../includes/app-service-web-deploy-zip-prepare.md)]

[!INCLUDE [Deploy ZIP file](../../includes/app-service-web-deploy-zip.md)]
Le point de terminaison ci-dessus ne fonctionne pas pour App Services Linux. Envisagez plutôt d’utiliser FTP ou l’[API de déploiement ZIP](faq-app-service-linux.md#continuous-integration-and-deployment).

## <a name="deploy-zip-file-with-azure-cli"></a>Déployer le fichier ZIP avec Azure CLI

Déployez le fichier ZIP chargé sur votre application web à l’aide de la commande [az webapp deployment source config-zip](/cli/azure/webapp/deployment/source#az-webapp-deployment-source-config-zip).  

L’exemple suivant déploie le fichier ZIP que vous avez chargé. Quand vous utilisez une installation locale d’Azure CLI, spécifiez le chemin de votre fichier ZIP local pour `--src`.

```azurecli-interactive
az webapp deployment source config-zip --resource-group <group-name> --name <app-name> --src clouddrive/<filename>.zip
```

Cette commande déploie les fichiers et répertoires du fichier ZIP vers votre dossier d’applications App Service par défaut (`\home\site\wwwroot`), puis redémarre l’application.

Par défaut, le moteur de déploiement suppose qu’un fichier ZIP est prêt à s’exécuter en l’état et n’effectue aucune automatisation de build. Pour permettre la même automatisation de build que dans un [déploiement Git](deploy-local-git.md), définissez le paramètre d’application `SCM_DO_BUILD_DURING_DEPLOYMENT` en exécutant la commande suivante dans [Azure Cloud Shell](https://shell.azure.com) :

```azurecli-interactive
az webapp config appsettings set --resource-group <group-name> --name <app-name> --settings SCM_DO_BUILD_DURING_DEPLOYMENT=true
```

Pour plus d’informations, consultez la [documentation Kudu](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url).

[!INCLUDE [app-service-deploy-zip-push-rest](../../includes/app-service-deploy-zip-push-rest.md)]  

## <a name="deploy-war-file"></a>Déployer un fichier WAR

Pour déployer un fichier WAR sur App Service, envoyez une requête POST à `https://<app-name>.scm.azurewebsites.net/api/wardeploy`. La requête POST doit contenir le fichier .war dans le corps du message. Les informations d’identification de déploiement pour votre application sont fournies dans la demande avec l’authentification de base HTTP.

Utilisez toujours `/api/wardeploy` lors du déploiement de fichiers WAR. Cette API développera votre fichier WAR et le placera sur le lecteur de fichiers partagés. L’utilisation d’autres API de déploiement peut entraîner un comportement incohérent. 

Pour l’authentification HTTP BASIC, vous avez besoin de vos informations d’identification de déploiement App Service. Pour découvrir comment définir les informations d’identification de votre déploiement, consultez [Définir et réinitialiser les informations d’identification de niveau utilisateur](deploy-configure-credentials.md#userscope).

### <a name="with-curl"></a>Avec cURL

L’exemple suivant utilise l’outil cURL pour déployer un fichier .war. Remplacez les espaces réservés `<username>`, `<war-file-path>` et `<app-name>`. Quand vous y êtes invité par cURL, tapez le mot de passe.

```bash
curl -X POST -u <username> --data-binary @"<war-file-path>" https://<app-name>.scm.azurewebsites.net/api/wardeploy
```

### <a name="with-powershell"></a>Avec PowerShell

L’exemple suivant utilise [Publish-AzWebapp](/powershell/module/az.websites/publish-azwebapp) pour charger le fichier .war. Remplacez les espaces réservés `<group-name>`, `<app-name>` et `<war-file-path>`.

```powershell
Publish-AzWebapp -ResourceGroupName <group-name> -Name <app-name> -ArchivePath <war-file-path>
```

[!INCLUDE [What happens to my app during deployment?](../../includes/app-service-deploy-atomicity.md)]

## <a name="next-steps"></a>Étapes suivantes

Pour des scénarios de déploiement plus avancés, consultez [Déploiement Git local vers Azure App Service](deploy-local-git.md). Le déploiement GIT vers Azure autorise le contrôle de version, la restauration du package, MSBuild et bien plus encore.

## <a name="more-resources"></a>Plus de ressources

* [Kudu : Deploying from a zip file (Déploiement à partir d’un fichier zip)](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file)
* [Informations d’identification du déploiement d’Azure App Service](deploy-ftp.md)
