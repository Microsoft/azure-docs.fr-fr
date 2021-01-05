---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/04/2020
ms.author: glenga
ms.openlocfilehash: 826fccad043b067ce86d5f56eaebc6ee48b532d1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96027927"
---
| Ressource |[Plan Consommation](../articles/azure-functions/functions-scale.md#consumption-plan)|[Plan Premium](../articles/azure-functions/functions-scale.md#premium-plan)|[Plan dédié](../articles/azure-functions/functions-scale.md#app-service-plan)|[ASE](../articles/app-service/environment/intro.md)| [Kubernetes](../articles/aks/quotas-skus-regions.md) |
| --- | --- | --- | --- | --- | --- |
|[Durée du délai d’expiration](../articles/azure-functions/functions-scale.md#timeout) (min) par défaut |5 | 30 |30<sup>1</sup> | 30 | 30 |
|[Durée du délai d’expiration](../articles/azure-functions/functions-scale.md#timeout) maximum (min) |10 | illimité<sup>7</sup> | illimité<sup>2</sup> | illimité | illimité |
| Nbre max. de connexions sortantes (par instance) | 600 actives (1 200 au total) | illimité | illimité | illimité | illimité |
| Taille de requête max. (Mo)<sup>3</sup> | 100 | 100 | 100 | 100 | Dépend du cluster |
| Longueur de chaîne de requête max.<sup>3</sup> | 4096 | 4096 | 4096 | 4096 | Dépend du cluster |
| Longueur d’URL de requête max.<sup>3</sup> | 8 192 | 8 192 | 8 192 | 8 192 | Dépend du cluster |
|[ACU](../articles/virtual-machines/acu.md) par instance | 100 | 210-840 | 100-840 | 210-250<sup>8</sup> | [Tarification d’AKS](https://azure.microsoft.com/pricing/details/container-service/) |
| Mémoire max. (en Go par instance) | 1.5 | 3,5-14 | 1,75-14 | 3.5 - 14 | Tous les nœuds sont pris en charge |
| Applications de fonction par plan |100 |100 |illimité<sup>4</sup> | illimité | illimité |
| [Plans App Service](../articles/app-service/overview-hosting-plans.md) | 100 par [région](https://azure.microsoft.com/global-infrastructure/regions/) |100 par groupe de ressources |100 par groupe de ressources | - | - |
| Stockage<sup>5</sup> |5 To |250 Go |50-1 000 Go | 1 To | n/a |
| Domaines personnalisés par application</a> |500<sup>6</sup> |500 |500 | 500 | n/a |
| domaines personnalisés [Prise en charge SSL](../articles/app-service/configure-ssl-bindings.md) |connexion SNI SSL illimitée incluse | connexions 1 IP SSL et SNI SSL illimitées incluses |connexions 1 IP SSL et SNI SSL illimitées incluses | connexions 1 IP SSL et SNI SSL illimitées incluses | n/a |

<sup>1</sup> Par défaut, le délai d’attente du runtime de Functions 1.x dans un plan App Service est illimité.  
<sup>2</sup> Nécessite que le plan App Service soit défini sur [Always On](../articles/azure-functions/functions-scale.md#always-on). Facturation aux [tarifs](https://azure.microsoft.com/pricing/details/app-service/) standard.  
<sup>3</sup> Ces limites sont [définies dans l’hôte](https://github.com/Azure/azure-functions-host/blob/dev/src/WebJobs.Script.WebHost/web.config).  
<sup>4</sup> Le nombre réel d’applications de fonction qui peuvent être hébergées dépend de l’activité des applications, de la taille des instances de machine et de l’utilisation de ressources correspondante.  
<sup>5</sup> La limite de stockage est la taille totale du contenu dans le stockage temporaire de toutes les applications du même plan App Service. Le plan Consommation utilise Azure Files pour le stockage temporaire.  
<sup>6</sup> Lorsque votre application de fonction est hébergée dans un [Plan Consommation](../articles/azure-functions/functions-scale.md#consumption-plan), seule l’option CNAME est prise en charge. Pour les applications de fonction présentes dans un [plan Premium](../articles/azure-functions/functions-scale.md#premium-plan) ou un [plan App Service](../articles/azure-functions/functions-scale.md#app-service-plan), vous pouvez mapper un domaine personnalisé en utilisant l’un ou l’autre des enregistrements : CNAME ou A.  
<sup>7</sup> Garanti pour une durée maximale de 60 minutes.  
<sup>8</sup>Les workers sont des rôles qui hébergent des applications clientes. Ils sont disponibles dans trois tailles fixes : Un processeur virtuel/3,5 Go de RAM ; Deux processeurs virtuels/7 Go de RAM ; Quatre processeurs virtuels/14 Go de RAM.