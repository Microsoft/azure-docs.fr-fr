---
title: Mettre hors service des API de déploiement pour les métriques et la mise à l'échelle Azure Monitor
description: Les API classiques de métriques et de mise à l’échelle automatique, également appelées Azure Service Management (ASM) ou modèle de déploiement RDFE, sont cours de suppression
ms.subservice: ''
ms.topic: conceptual
ms.date: 11/19/2018
ms.openlocfilehash: a8900d238e7542864e3dc53f9d192e579e2c96d1
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95536691"
---
# <a name="azure-monitor-retirement-of-classic-deployment-model-apis-for-metrics-and-autoscale"></a>Suppression par Azure Monitor d’API du modèle de déploiement classique pour les métriques et la mise à l’échelle automatique

Azure Monitor (anciennement Azure Insights lors de son premier lancement) offre actuellement la possibilité de créer et gérer les paramètres de mise à l’échelle automatique et de consommer des métriques à partir de machines virtuelles classiques et de services cloud classiques. Le jeu initial d’API de déploiement classique basées sur modèle sera **supprimé après le 30 juin 2019** dans tous les clouds publics et privés Azure dans toutes les régions.   

Les mêmes opérations sont prises en charge via un ensemble d’API Azure Resource Manager depuis plus d’un an. Le portail Azure utilise les nouvelles API REST pour la mise à l’échelle automatique et les métriques. Un kit SDK, PowerShell et CLI nouveaux basés sur ces API Resource Manager sont également disponibles. Nos services partenaires pour surveiller consomment les nouvelles API REST Resource Manager dans Azure Monitor.  

## <a name="who-is-not-affected"></a>Qui n’est pas affecté

Si vous gérez la mise à l’échelle automatique via le portail Azure, le [nouveau SDK Azure Monitor](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/), PowerShell, CLI ou les modèles Resource Manager, aucune action n’est nécessaire.  

Si vous consommez des métriques via le portail Azure ou divers [services partenaires de surveillance](./partners.md), aucune action n’est nécessaire. Microsoft collabore avec les partenaires de surveillance pour migrer vers les nouvelles API.

## <a name="who-is-affected"></a>Qui est affecté

Cet article vous concerne si vous utilisez les composants suivants :

- **SDK Azure Insights classique** - Si vous utilisez le [SDK Azure Insights classique](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/), passez au nouveau SDK Azure Monitor pour [.NET](https://github.com/azure/azure-libraries-for-net#download) ou [Java](https://github.com/azure/azure-libraries-for-java#download). Téléchargez le [package NuGet de SDK Azure Monitor](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/).

- **Mise à l’échelle automatique classique** - Si vous appelez les [API de paramètres de mise à l’échelle automatique classique](/previous-versions/azure/reference/mt348562(v=azure.100)) à partir de vos outils personnalisés ou à l’aide du [SDK Azure Insights classique](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/), vous devriez adopter l’utilisation de [l’API REST Azure Monitor Resource Manager](/rest/api/monitor/autoscalesettings).

- **Métriques classiques** - Si vous consommez des métriques à l’aide des [API REST classiques](/previous-versions/azure/reference/dn510374(v=azure.100)) ou du [SDK Azure Insights classique](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/) à partir d’outils personnalisés, vous devriez adopter l’utilisation de [l’API Azure Monitor Resource Manager](/rest/api/monitor/autoscalesettings). 

Si vous ne savez pas si votre code ou vos outils personnalisés appellent les API classiques, examinez les éléments suivants :

- Passez en revue l’URI référencé dans votre code ou outil. Les API classiques utilisent l’URI https://management.core.windows.net. Vous devriez utiliser l’URI plus récent pour les API Resource Manager qui commence par `https://management.azure.com/`.

- Comparez le nom de l’assembly sur votre ordinateur. L’assembly classique plus ancien se trouve dans https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/.

- Si vous utilisez l’authentification par certificat pour accéder aux API de métriques ou de mise à l’échelle automatique, vous utilisez un point de terminaison et une bibliothèque classiques. Les API Resource Manager plus récentes requièrent l’authentification Azure Active Directory via un principal de service ou un principal de l’utilisateur.

- Si vous utilisez des appels référencés dans la documentation sur les liens suivants, vous utilisez les API classiques plus anciennes.

  - [Bibliothèque de classes Windows.Azure.Management.Monitoring](/previous-versions/azure/dn510414(v=azure.100))

  - [Surveillance (classique) .NET](/previous-versions/azure/reference/mt348562(v%3dazure.100))

  - [Interface IMetricOperations](/previous-versions/azure/reference/dn802395(v%3dazure.100))

## <a name="why-you-should-switch"></a>Pourquoi basculer

Toutes les fonctionnalités existantes pour la mise à l’échelle automatique et les métriques continueront de fonctionner via les nouvelles API.  

La migration vers des API plus récentes donne accès à des fonctionnalités Resource Manager, comme la prise en charge du contrôle d’accès en fonction du rôle Azure (Azure RBAC) cohérent sur tous vos services de surveillance. Vous bénéficiez également de fonctionnalités supplémentaires pour les métriques : 

- prise en charge des dimensions
- granularité métrique 1 minute cohérente sur tous les services 
- interrogation optimisée
- conservation des données plus élevée (93 jours de métriques au lieu de 30 jours) 

Dans l’ensemble, comme tous les autres services dans Azure, les API Azure Monitor basées sur Resource Manager bénéficient de performances, d’une évolutivité et d’une fiabilité meilleures. 

## <a name="what-happens-if-you-do-not-migrate"></a>Que se passe-t-il si vous ne migrez pas

### <a name="before-retirement"></a>Avant la suppression

Il n’y aura aucun impact direct sur vos services Azure ou leurs charges de travail.  

### <a name="after-retirement"></a>Après la suppression

Tous les appels aux API classiques répertoriées précédemment échoueront et retourneront des messages d’erreur semblables à ce qui suit :

Pour la mise à l’échelle automatique : *Cette API est obsolète. Utilisez le portail Azure, le SDK Azure Monitor, PowerShell, CLI ou les modèles Resource Manager pour gérer les paramètres de mise à l’échelle automatique*.  

Pour les métriques : *Cette API est obsolète. Utilisez le portail Azure, le SDK Azure Monitor, PowerShell, CLI pour interroger les métriques*.

## <a name="email-notifications"></a>Notifications par e-mail

Une notification de suppression a été envoyée à des adresses e-mail pour les rôles de compte suivants : 

- Administrateurs de compte et de service
- Coadministrateurs  

Si vous avez des questions, contactez-nous MonitorClassicAPIhelp@microsoft.com.  

## <a name="references"></a>References

- [API REST plus récentes pour Azure Monitor](/rest/api/monitor/) 
- [SDK Azure Monitor plus récent](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/)

