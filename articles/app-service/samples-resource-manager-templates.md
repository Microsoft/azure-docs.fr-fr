---
title: Exemples de modèles Azure Resource Manager
description: Recherchez des exemples de modèles Azure Resource Manager pour certains des scénarios App Service les plus répandus. Découvrez comment automatiser vos tâches de déploiement ou de gestion App Service.
author: tfitzmac
tags: azure-service-management
ms.topic: sample
ms.date: 08/26/2020
ms.author: tomfitz
ms.custom: mvc, fasttrack-edit
ms.openlocfilehash: 5129fccceb633991767cdd3b52bbb5b6af067270
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88891043"
---
# <a name="azure-resource-manager-templates-for-app-service"></a>Modèles Azure Resource Manager pour App Service

Le tableau suivant inclut des liens vers des modèles Azure Resource Manager pour Azure App Service. Pour obtenir des recommandations sur la prévention des erreurs courantes lors de la création de modèles d’application, consultez l’article [Guidance on deploying apps with Azure Resource Manager templates](deploy-resource-manager-template.md) (Aide au déploiement d’applications avec des modèles Azure Resource Manager).

Pour découvrir la syntaxe JSON et les propriétés des ressources App Services, consultez [Types de ressources Microsoft.Web](/azure/templates/microsoft.web/allversions).

| Déploiement d’une application | Description |
|-|-|
| [Plan App Service et application Linux de base](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-basic-linux) | Déploie une application App Service configurée pour Linux. |
| [Plan App Service et application Windows de base](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-basic-windows) | Déploie une application App Service configurée pour Windows. |
| [Application liée à un référentiel GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-github-deploy)| Déploie une application App Service qui extrait du code à partir de GitHub. |
| [Application avec des emplacements de déploiement personnalisés](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-custom-deployment-slots)| Déploie une application App Service avec des environnements/emplacements de déploiement personnalisés. |
| [Application avec point de terminaison privé](https://github.com/Azure/azure-quickstart-templates/tree/master/101-private-endpoint-webapp)| Déploie une application App Service avec un point de terminaison privé. |
|**Configuration d’une application**| **Description** |
| [Certificat d’application issu de Key Vault](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-certificate-from-key-vault)| Déploie un certificat d’application App Service issu d’un secret Azure Key Vault et l’utilise pour la liaison TLS/SSL. |
| [Application avec un domaine personnalisé et SSL](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-custom-domain-and-ssl)| Déploie une application App Service avec un nom d’hôte personnalisé et obtient un certificat d’application issu de Key Vault pour la liaison TLS/SSL. |
| [Application avec l’extension GoLang](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-with-golang)| Déploie une application App Service avec l’extension de site GoLang. Vous pouvez ensuite exécuter des applications web développées sur GoLang sur Azure. |
| [Application avec Java 8 et Tomcat 8](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-java-tomcat)| Déploie une application App Service pour laquelle Java 8 et Tomcat 8 sont activés. Vous pouvez ensuite exécuter des applications Java dans Azure. |
| [Application avec intégration au réseau virtuel régional](https://github.com/Azure/azure-quickstart-templates/tree/master/101-app-service-regional-vnet-integration)| Déploie une application App Service avec l’intégration au réseau virtuel régional activée. |
|**Protection d’une application**| **Description** |
| [Application intégrée à Azure Application Gateway](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-with-app-gateway-v2)| Déploie une application App Service et une passerelle d’application, et isole le trafic à l’aide du point de terminaison de service et des restrictions d’accès. |
|**Application Linux avec des ressources connectées**| **Description** |
| [Application sur Linux avec MySQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-mysql) | Déploie une application App Service sur Linux avec Azure Database pour MySQL. |
| [Application sur Linux avec PostgreSQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-linux-managed-postgresql) | Déploie une application App Service sur Linux avec Azure Database pour PostgreSQL. |
|**Application avec des ressources connectées**| **Description** |
| [Application avec MySQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-mysql)| Déploie une application App Service sur Windows avec Azure Database pour MySQL. |
| [Application avec PostgreSQL](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-managed-postgresql)| Déploie une application App Service sur Windows avec Azure Database pour PostgreSQL. |
| [Application avec une base de données dans Azure SQL Database](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-sql-database)| Déploie une application App Service et une base de données dans Azure SQL Database au niveau de service De base. |
| [Application avec une connexion au stockage d’objets blob](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-blob-connection)| Déploie une application App Service avec une chaîne de connexion du stockage d’objets blob Azure. Vous pouvez ensuite utiliser le stockage d’objets blob à partir de l’application. |
| [Application avec un cache Azure pour Redis](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-with-redis-cache)| Déploie une application App Service avec un cache Azure pour Redis. |
| [Application connectée à une application web principale](https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-privateendpoint-vnet-injection)| Déploie deux applications web (front-end et back-end) connectées en toute sécurité avec l’injection de réseaux virtuels et le point de terminaison privé. |
|**Environnement App Service**| **Description** |
| [Créer un environnement App Service Environment v2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-create) | Crée un environnement App Service Environment v2 dans votre réseau virtuel. |
| [Créer un environnement App Service Environment v2 avec une adresse ILB](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-asev2-ilb-create/) | Crée un environnement App Service Environment v2 dans votre réseau virtuel avec une adresse d’équilibreur de charge interne privé. |
| [Configurer le certificat SSL par défaut pour un environnement App Service Environment avec une adresse ILB ou un environnement App Service Environment v2 avec une adresse ILB](https://github.com/Azure/azure-quickstart-templates/tree/master/201-web-app-ase-ilb-configure-default-ssl) | Configure le certificat TLS/SSL par défaut pour un environnement App Service Environment avec une adresse ILB ou un environnement App Service Environment v2 avec une adresse ILB. |
| | |
