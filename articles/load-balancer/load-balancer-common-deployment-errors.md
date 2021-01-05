---
title: Résoudre les erreurs de déploiement courantes
titleSuffix: Azure Load Balancer
description: Explique comment résoudre les erreurs courantes lors du déploiement d’équilibreurs de charge Azure
services: load-balancer
documentationcenter: na
tags: top-support-issue
author: anavinahar
ms.service: load-balancer
ms.topic: troubleshooting
ms.date: 04/27/2020
ms.author: anavin
ms.openlocfilehash: ae3899a61e942695fed1e3da5fc543b298a42e24
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94695552"
---
# <a name="troubleshoot-common-azure-deployment-errors-with-azure-load-balancer"></a>Résoudre les erreurs courantes dans des déploiements Azure avec Azure Load Balancer

Cet article décrit certaines erreurs courantes liées au déploiement Azure Load Balancer et fournit des informations pour les résoudre. Si vous recherchez des informations sur un code d’erreur et que ces informations ne sont pas fournies dans cet article, faites-le nous savoir. En bas de cette page, vous pouvez laisser des commentaires. Ces commentaires sont suivis avec les problèmes GitHub.

## <a name="error-codes"></a>Codes d’erreur

| Code d'erreur | Détails et atténuation |
| ------- | ---------- |
|DifferentSkuLoadBalancersAndPublicIPAddressNotAllowed| La référence SKU d’adresse IP publique et la référence SKU Load Balancer doivent correspondre. Assurez-vous que les références SKU Azure Load Balancer et d’adresse IP publique correspondent. La référence SKU standard est recommandée pour les charges de travail de production. Découvrez-en plus sur les [différences entre les références SKU](./skus.md).  |
|DifferentSkuLoadBalancerAndPublicIPAddressNotAllowedInVMSS | Les groupes de machines virtuelles identiques sont par défaut des équilibreurs de charge de base quand la référence SKU n’est pas spécifiée ou est déployée sans adresses IP publiques standard. Redéployez le groupe de machines virtuelles identiques avec des adresses IP publiques standard sur les différentes instances pour vous assurer que Standard Load Balancer est sélectionné ou sélectionnez simplement un équilibreur de charge standard lors du déploiement du groupe de machines virtuelles identiques à partir du portail Azure. |
|MaxAvailabilitySetsInLoadBalancerReached | Le pool de back-ends d’un équilibreur de charge peut contenir au maximum 150 groupes à haute disponibilité. Si vous n’avez pas de groupes à haute disponibilité définis explicitement pour vos machines virtuelles dans le pool de back-ends, chaque machine virtuelle se retrouve dans son propre groupe à haute disponibilité. Ainsi, le déploiement de 150 machines virtuelles autonomes peut impliquer l’existence de 150 groupes à haute disponibilité, la limite étant alors atteinte. Vous pouvez déployer un groupe à haute disponibilité et y ajouter des machines virtuelles comme solution de contournement. |
|NetworkInterfaceAndLoadBalancerAreInDifferentAvailabilitySets | Pour l’équilibreur de charge de référence SKU de base, l’interface réseau et l’équilibreur de charge doivent se trouver dans le même groupe à haute disponibilité. |
|RulesOfSameLoadBalancerTypeUseSameBackendPortProtocolAndIPConfig| Vous ne pouvez pas avoir plus d’une règle sur un type d’équilibreur de charge donné (interne, public) avec les mêmes port back-end et protocole référencés par le même groupe de machines virtuelles identiques. Mettez à jour votre règle pour changer cette création de règle en double. |
|RulesOfSameLoadBalancerTypeUseSameBackendPortProtocolAndVmssIPConfig| Vous ne pouvez pas avoir plus d’une règle sur un type d’équilibreur de charge donné (interne, public) avec les mêmes port back-end et protocole référencés par le même groupe de machines virtuelles identiques. Mettez à jour les paramètres de votre règle pour changer cette création de règle en double. |
|AnotherInternalLoadBalancerExists| Un seul équilibreur de charge de type interne peut référencer le même ensemble de machines virtuelles/interfaces réseau dans le back-end de l’équilibreur de charge. Mettez à jour votre déploiement pour ne créer qu’un seul équilibreur de charge du même type. |
|CannotUseInactiveHealthProbe| Vous ne pouvez pas avoir une sonde qui n’est utilisée par aucune règle configurée pour l’intégrité du groupe de machines virtuelles identiques. Assurez-vous que la sonde qui est configurée est effectivement utilisée. |
|VMScaleSetCannotUseMultipleLoadBalancersOfSameType| Vous ne pouvez pas avoir plusieurs équilibreurs de charge du même type (interne, public). Vous pouvez avoir au maximum un équilibreur de charge interne et un équilibreur de charge public. |
|VMScaleSetCannotReferenceLoadbalancerWhenLargeScaleOrCrossAZ | Basic Load Balancer n’est pas pris en charge pour les groupes de machines virtuelles identiques de groupe multiplacement ou pour un groupe de machines virtuelles identiques inter-zones de disponibilité. Utilisez Standard Load Balancer à la place. |
|MarketplacePurchaseEligibilityFailed | Basculez vers le compte d’administration approprié pour autoriser les achats en lien avec le caractère EA de l’abonnement. Vous trouverez plus d’informations [ici](../marketplace/marketplace-faq-publisher-guide.md#what-could-block-a-customer-from-completing-a-purchase). |
|ResourceDeploymentFailure| Si votre équilibreur de charge est dans un état d’échec, procédez comme suit pour le rétablir :<ol><li>Accédez au site https://resources.azure.com et connectez-vous avec vos informations d’identification pour le Portail Azure.</li><li>Sélectionnez **Lecture/écriture**.</li><li>Sur la gauche, développez **Abonnements**, puis l’abonnement dont le Load Balancer doit être mis à jour.</li><li>Développez **ResourceGroups**, puis le groupe de ressources dont le Load Balancer doit être mis à jour.</li><li>Sélectionnez **Microsoft.Network** > **LoadBalancers**, puis le Load Balancer à mettre à jour, **LoadBalancer_1**.</li><li>Sur la page d’affichage de **LoadBalancer_1**, sélectionnez **GET** > **Modifier**.</li><li>Mettez à jour la valeur **ProvisioningState** de **Échec** à **Opération réussie**.</li><li>Sélectionnez **PUT**.</li></ol>|
|  |  |

## <a name="next-steps"></a>Étapes suivantes

* Consulter le [tableau de comparaison des références SKU](./skus.md) Azure Load Balancer
* Découvrir les [limites d’Azure Load Balancer](../azure-resource-manager/management/azure-subscription-service-limits.md#load-balancer)