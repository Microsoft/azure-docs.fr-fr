---
title: Exemples CLI
description: Recherchez des exemples Azure CLI pour certains des scénarios App Service les plus répandus. Découvrez comment automatiser vos tâches de déploiement ou de gestion App Service.
tags: azure-service-management
ms.assetid: 53e6a15a-370a-48df-8618-c6737e26acec
ms.topic: sample
ms.date: 07/07/2020
ms.custom: mvc
ms.openlocfilehash: beab87618b97da4e61b0525c0c5a6bdd134fb7f8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86169440"
---
# <a name="cli-samples-for-azure-app-service"></a>Exemples d’interface CLI pour Azure App Service

Le tableau suivant contient des liens vers des scripts Bash créés à l’aide de l’interface de ligne de commande Azure.

| Script | Description |
|-|-|
|**Créer une application**||
| [Créer une application et déployer des fichiers par FTP](./scripts/cli-deploy-ftp.md?toc=%2fcli%2fazure%2ftoc.json)| Crée une application App Service et y déploie un fichier à l’aide de FTP. |
| [Créer une application et déployer le code à partir de GitHub](./scripts/cli-deploy-github.md?toc=%2fcli%2fazure%2ftoc.json)| Crée une application App Service et déploie le code à partir d’un dépôt GitHub public. |
| [Créer une application avec un déploiement continu à partir de GitHub](./scripts/cli-continuous-deployment-github.md?toc=%2fcli%2fazure%2ftoc.json)| Crée une application App Service avec la publication continue à partir d’un dépôt GitHub dont vous êtes le propriétaire. |
| [Créer une application et déployer le code à partir d’un dépôt Git local](./scripts/cli-deploy-local-git.md?toc=%2fcli%2fazure%2ftoc.json) | Crée une application App Service et configure la transmission de code de type push à partir d’un dépôt Git local. |
| [Créer une application et déployer le code dans un environnement de préproduction](./scripts/cli-deploy-staging-environment.md?toc=%2fcli%2fazure%2ftoc.json) | Crée une application App Service avec un emplacement de déploiement pour les modifications de code intermédiaires. |
| [Créer une application ASP.NET Core dans un conteneur Docker](./scripts/cli-linux-docker-aspnetcore.md?toc=%2fcli%2fazure%2ftoc.json) | Crée une application App Service sur Linux et charge une image Docker à partir de Docker Hub. |
| [Créer et exposer une application avec un point de terminaison privé](./scripts/cli-deploy-privateendpoint.md?toc=%2fcli%2fazure%2ftoc.json) | Crée une application App Service et un point de terminaison privé. |
|**Configurer l’application**||
| [Mapper un domaine personnalisé à une application](./scripts/cli-configure-custom-domain.md?toc=%2fcli%2fazure%2ftoc.json)| Crée une application App Service et mappe un nom de domaine personnalisé à celle-ci. |
| [Lier un certificat TLS/SSL personnalisé à une application](./scripts/cli-configure-ssl-certificate.md?toc=%2fcli%2fazure%2ftoc.json)| Crée une application App Service et lie le certificat TLS/SSL d’un nom de domaine personnalisé à celle-ci. |
|**Mettre à l’échelle une application**||
| [Mettre à l’échelle une application manuellement](./scripts/cli-scale-manual.md?toc=%2fcli%2fazure%2ftoc.json) | Crée une application App Service et la met à l’échelle entre deux instances. |
| [Mettre à l’échelle une application dans le monde entier avec une architecture haute disponibilité](./scripts/cli-scale-high-availability.md?toc=%2fcli%2fazure%2ftoc.json) | Crée deux applications App Service dans deux régions géographiques différentes et les rend disponibles par le biais d’un point de terminaison unique à l’aide d’Azure Traffic Manager. |
|**Protéger l’application**||
| [Intégration à Azure Application Gateway](./scripts/cli-integrate-app-service-with-application-gateway.md?toc=%2fcli%2fazure%2ftoc.json) | Crée une application App Service et l’intègre à Application Gateway à l’aide du point de terminaison de service et des restrictions d’accès. |
|**Connecter l’application aux ressources**||
| [Connecter une application à une instance SQL Database](./scripts/cli-connect-to-sql.md?toc=%2fcli%2fazure%2ftoc.json)| Crée une application App Service et une base de données dans Azure SQL Database, puis ajoute la chaîne de connexion de base de données aux paramètres d’application. |
| [Connecter une application à un compte de stockage](./scripts/cli-connect-to-storage.md?toc=%2fcli%2fazure%2ftoc.json)| Crée une application App Service et un compte de stockage, puis ajoute la chaîne de connexion de stockage aux paramètres d’application. |
| [Connecter une application à un Cache Azure pour Redis](./scripts/cli-connect-to-redis.md?toc=%2fcli%2fazure%2ftoc.json) | Crée une application App Service et un Cache Azure pour Redis, puis ajoute les détails de connexion redis aux paramètres d’application. |
| [Connecter une application à Cosmos DB](./scripts/cli-connect-to-documentdb.md?toc=%2fcli%2fazure%2ftoc.json) | Crée une application App Service et une base de données Cosmos DB, puis ajoute les détails de connexion Cosmos DB aux paramètres d’application. |
|**Sauvegarder et restaurer une app**||
| [Sauvegarder une application](./scripts/cli-backup-onetime.md?toc=%2fcli%2fazure%2ftoc.json) | Crée une application App Service et une sauvegarde unique pour celle-ci. |
| [Créer une sauvegarde planifiée pour une application](./scripts/cli-backup-scheduled.md?toc=%2fcli%2fazure%2ftoc.json) | Crée une application App Service et une sauvegarde planifiée pour celle-ci. |
| [Restaurer une application à partir d’une sauvegarde](./scripts/cli-backup-restore.md?toc=%2fcli%2fazure%2ftoc.json) | Restaure une application App Service à partir d’une sauvegarde. |
|**Surveiller l’application**||
| [Superviser une application avec les journaux d’activité de serveur web](./scripts/cli-monitor.md?toc=%2fcli%2fazure%2ftoc.json) | Crée une application App Service, active sa journalisation et télécharge les journaux d’activité sur votre ordinateur local. |
| | |
