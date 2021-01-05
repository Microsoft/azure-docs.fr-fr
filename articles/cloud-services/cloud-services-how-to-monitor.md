---
title: Surveillance d’Azure Cloud Service | Microsoft Docs
description: Décrit ce que la surveillance d’Azure Cloud Service implique et certaines des options à votre disposition.
services: cloud-services
documentationcenter: ''
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 01/29/2018
ms.author: tagore
ms.openlocfilehash: 61c794ba03934ae1828ba310f3f776bfb61b652b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85847248"
---
# <a name="introduction-to-cloud-service-monitoring"></a>Introduction à la surveillance des services cloud

Vous pouvez surveiller les principales mesures de performances pour n’importe quel service cloud. Chaque rôle de service cloud collecte des données de base : utilisation du processeur, utilisation du réseau et utilisation du disque. Si le service cloud a l’extension `Microsoft.Azure.Diagnostics` appliquée à un rôle, ce rôle peut collecter des points de données supplémentaires. Cet article vous fournit une présentation de Diagnostics Azure pour Cloud Services.

Avec la surveillance de base, les données des compteurs de performances des instances de rôle sont échantillonnées et collectées toutes les 3 minutes. Ces données de surveillance de base ne sont pas stockées dans votre compte de stockage et ne représentent aucun coût supplémentaire associé.

Avec la surveillance avancée, des mesures supplémentaires sont prélevées et collectées par intervalles de 5 minutes, 1 heure et 12 heures. Les données consolidées sont stockées dans un compte de stockage et purgées tous les 10 jours. Le compte de stockage utilisé est configuré par rôle. Vous pouvez utiliser différents comptes de stockage pour différents rôles. Vous configurez cela avec une chaîne de connexion dans les fichiers [.csdef](cloud-services-model-and-package.md#servicedefinitioncsdef) et [.cscfg](cloud-services-model-and-package.md#serviceconfigurationcscfg).


## <a name="basic-monitoring"></a>Surveillance de base

Comme indiqué dans l’introduction, un service cloud recueille automatiquement des données de surveillance de base à partir de la machine virtuelle hôte. Ces données incluent le pourcentage d’utilisation du processeur, les entrées/sorties du réseau et les accès disque en lecture/écriture. Les données d’analyse collectées s’affichent automatiquement dans les pages de vue d’ensemble et de métriques du service cloud, dans le portail Azure. 

La surveillance de base ne nécessite pas de compte de stockage. 

![vignettes de surveillance de service cloud de base](media/cloud-services-how-to-monitor/basic-tiles.png)

## <a name="advanced-monitoring"></a>Surveillance avancée

La surveillance avancée implique l’utilisation de l’extension **Diagnostics Azure** (et éventuellement le SDK d’Application Insights) sur le rôle que vous souhaitez analyser. L’extension Diagnostics utilise un fichier de configuration (par rôle) nommé **diagnostics.wadcfgx** pour configurer les mesures de diagnostic analysées. L’extension Azure diagnostics collecte et stocke des données dans un compte de stockage Azure. Ces paramètres sont configurés dans les fichiers **.wadcfgx**, [.csdef](cloud-services-model-and-package.md#servicedefinitioncsdef) et [.cscfg](cloud-services-model-and-package.md#serviceconfigurationcscfg). Cela signifie qu’il y a un coût supplémentaire associé à la surveillance avancée.

Lorsque chaque rôle est créé, Visual Studio lui ajoute l’extension Diagnostics Azure. Cette extension des diagnostics peut collecter les types d’informations suivants :

* Compteurs de performances personnalisés
* Journaux d’activité d’application
* Journaux des événements Windows
* Source d’événement .NET
* Journaux d’activité IIS
* ETW basé sur les manifestes
* Vidages sur incident
* Journaux d’activité d’erreurs client

> [!IMPORTANT]
> Alors que toutes ces données sont regroupées dans le compte de stockage, le portail ne fournit **pas** un moyen natif pour les représenter visuellement. Il est fortement recommandé d’intégrer un autre service, comme Application Insights, dans votre application.

## <a name="setup-diagnostics-extension"></a>Configurer l’extension de diagnostic

Pour commencer, si vous ne disposez pas d'un compte de stockage **classique**, [créez-en un](../storage/common/storage-account-create.md). Assurez-vous que le compte de stockage est créé avec le **modèle de déploiement classique** spécifié.

Ensuite, accédez à la ressource **Compte de stockage (classique)** . Sélectionnez **Paramètres** > **Clés d’accès** et copiez la valeur de **Chaîne de connexion principale**. Vous avez besoin de cette valeur pour le service cloud. 

Pour activer les diagnostics avancés, vous devez modifier deux fichiers de configuration, **ServiceDefinition.csdef** et **ServiceConfiguration.cscfg**.

### <a name="servicedefinitioncsdef"></a>ServiceDefinition.csdef

Dans le fichier **ServiceDefinition.csdef**, ajoutez un nouveau paramètre nommé `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString` pour chaque rôle qui utilise des diagnostics avancés. Visual Studio ajoute cette valeur dans le fichier lorsque vous créez un nouveau projet. Au cas où elle manquerait, vous pouvez l’ajouter maintenant. 

```xml
<ServiceDefinition name="AnsurCloudService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition" schemaVersion="2015-04.2.6">
  <WorkerRole name="WorkerRoleWithSBQueue1" vmsize="Small">
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" />
```

Cela définit un nouveau paramètre qui doit être ajouté à chaque fichier **ServiceConfiguration.cscfg**. 

Vous disposez probablement de deux fichiers **.cscfg**, un nommé **ServiceConfiguration.cloud.cscfg** pour le déploiement sur Azure et l’autre nommé **ServiceConfiguration.local.cscfg** qui est utilisé pour les déploiements locaux dans l’environnement émulé. Ouvrez et modifiez chaque fichier **.cscfg**. Ajoutez un paramètre nommé `Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString`. Définissez la valeur sur la **chaîne de connexion principale** du compte de stockage classique. Si vous souhaitez utiliser le stockage local sur votre machine de développement, utilisez `UseDevelopmentStorage=true`.

```xml
<ServiceConfiguration serviceName="AnsurCloudService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceConfiguration" osFamily="4" osVersion="*" schemaVersion="2015-04.2.6">
  <Role name="WorkerRoleWithSBQueue1">
    <Instances count="1" />
    <ConfigurationSettings>
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="DefaultEndpointsProtocol=https;AccountName=mystorage;AccountKey=KWwkdfmskOIS240jnBOeeXVGHT9QgKS4kIQ3wWVKzOYkfjdsjfkjdsaf+sddfwwfw+sdffsdafda/w==" />
      
      <!-- or use the local development machine for storage
      <Setting name="Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString" value="UseDevelopmentStorage=true" />
      -->
```

## <a name="use-application-insights"></a>Utiliser Application Insights

Lorsque vous publiez le service cloud à partir de Visual Studio, vous disposez de la possibilité d’envoyer les données de diagnostic à Application Insights. Vous pouvez créer la ressource Azure Application Insights à ce moment-là ou envoyer les données à une ressource Azure existante. La disponibilité, les performances, les échecs et l’utilisation de votre service cloud peuvent être surveillés par Application Insights. Vous pouvez ajouter des graphiques personnalisés à Application Insights afin d’afficher les données qui vous intéressent le plus. Les données d’instance de rôle peuvent être collectées à l’aide du SDK d’Application Insights dans votre projet de service cloud. Pour plus d’informations sur l’intégration d’Application Insights, consultez [Application Insights avec Cloud Services](../azure-monitor/app/cloudservices.md).

Notez que, si vous pouvez utiliser Application Insights pour afficher les compteurs de performances (et les autres paramètres) que vous avez spécifiés via l’extension Diagnostics Azure pour Windows, vous n’obtenez une expérience plus riche qu’en intégrant le kit SDK Application Insights dans vos rôles web et de travail.


## <a name="next-steps"></a>Étapes suivantes

- [En savoir plus sur Application Insights avec Cloud Services](../azure-monitor/app/cloudservices.md)
- [Configurer les compteurs de performances](diagnostics-performance-counters.md)




