---
title: Questions fréquentes (FAQ) concernant les problèmes de déploiement pour Microsoft Azure Cloud Services | Microsoft Docs
description: Cet article répertorie les questions fréquentes sur le déploiement pour Microsoft Azure Cloud Services.
services: cloud-services
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue
ms.assetid: 84985660-2cfd-483a-8378-50eef6a0151d
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: 63a219078927e9001a8eb4085c722e7ec8d2fac9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75980627"
---
# <a name="deployment-issues-for-azure-cloud-services-frequently-asked-questions-faqs"></a>Problèmes de déploiement pour les services cloud Azure : Forum Aux Questions (FAQ)

Cet article comprend des questions fréquentes sur les problèmes de déploiement pour [Microsoft Azure Cloud Services](https://azure.microsoft.com/services/cloud-services). Vous pouvez également consulter la page [Taille de services cloud](cloud-services-sizes-specs.md) pour obtenir des informations sur la taille.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="why-does-deploying-a-cloud-service-to-the-staging-slot-sometimes-fail-with-a-resource-allocation-error-if-there-is-already-an-existing-deployment-in-the-production-slot"></a>Pourquoi le déploiement d’un service cloud dans l’emplacement intermédiaire échoue-t-il parfois avec une erreur d’allocation de ressources si l’emplacement de production contient déjà un déploiement ?
Si un service cloud est déployé à chaque emplacement, l’ensemble du service cloud est épinglé à un cluster spécifique. Par conséquent, si l’emplacement de production contient déjà un déploiement, un nouveau déploiement intermédiaire peut être affecté uniquement dans le même cluster que l’emplacement de production.

Des échecs d’allocation se produisent quand le cluster dans lequel se situe votre service cloud n’a pas suffisamment de ressources de calcul physiques pour satisfaire votre demande de déploiement.

Pour savoir comment limiter les effets de ces échecs d’allocation, consultez [Échec d’allocation d’un service cloud : Solutions](cloud-services-allocation-failures.md#solutions).

## <a name="why-does-scaling-up-or-scaling-out-a-cloud-service-deployment-sometimes-result-in-allocation-failure"></a>Pourquoi une montée en puissance ou une augmentation de la taille des instances dans le déploiement d’un service cloud génère-t-elle parfois un échec d’allocation ?
Quand un service cloud est déployé, il est généralement épinglé à un cluster spécifique. Autrement dit, quand un service cloud existant fait l’objet d’une montée en puissance/augmentation de la taille des instances, les nouvelles instances doivent être allouées au même cluster. Si le cluster est proche de sa capacité maximale ou que la taille/type de machine virtuelle souhaité n’est pas disponible, la demande peut échouer.

Pour savoir comment limiter les effets de ces échecs d’allocation, consultez [Échec d’allocation d’un service cloud : Solutions](cloud-services-allocation-failures.md#solutions).

## <a name="why-does-deploying-a-cloud-service-into-an-affinity-group-sometimes-result-in-allocation-failure"></a>Pourquoi le déploiement d’un service cloud dans un groupe d’affinités provoque-t-il parfois un échec d’allocation ?
Un nouveau déploiement dans un service cloud vide peut être alloué par l’infrastructure d’un cluster de cette région, sauf si le service cloud est épinglé à un groupe d’affinités. Des déploiements vers le même groupe d'affinités seront tentés sur le même cluster. Si le cluster est presque plein, la demande peut échouer.

Pour savoir comment limiter les effets de ces échecs d’allocation, consultez [Échec d’allocation d’un service cloud : Solutions](cloud-services-allocation-failures.md#solutions).

## <a name="why-does-changing-vm-size-or-adding-a-new-vm-to-an-existing-cloud-service-sometimes-result-in-allocation-failure"></a>Pourquoi le fait de modifier la taille des machines virtuelles ou d’ajouter une nouvelle machine virtuelle à un service cloud existant occasionne-t-il parfois un échec d’allocation ?
Les clusters d’un centre de données peuvent présenter différentes configurations de type de machine (par exemple, série A, série Av2, série D, série Dv2, série G, série H, etc.). Mais les clusters n’ont pas nécessairement besoin d’être tous dotés de tous les types de machines virtuelles. Par exemple, si vous essayez d’ajouter une machine virtuelle de série D à un service cloud qui est déjà déployé dans un cluster constitué uniquement de machines de série A, vous rencontrerez un échec d’allocation. Il en est de même si vous essayez de modifier la taille des références de machines virtuelles (par exemple, en passant d’une série A à une série D).

Pour savoir comment limiter les effets de ces échecs d’allocation, consultez [Échec d’allocation d’un service cloud : Solutions](cloud-services-allocation-failures.md#solutions).

Pour vérifier les tailles disponibles dans votre région, consultez [Microsoft Azure : Disponibilité des produits par région](https://azure.microsoft.com/regions/services).

## <a name="why-does-deploying-a-cloud-service-sometime-fail-due-to-limitsquotasconstraints-on-my-subscription-or-service"></a>Pourquoi le déploiement d’un service cloud échoue-t-il parfois en raison des limites/quotas/contraintes de mon abonnement ou service ?
Le déploiement d’un service cloud peut échouer si les ressources qui doivent être allouées dépassent la valeur par défaut ou le quota maximal autorisé pour votre service au niveau de la région et/ou du centre de données. Pour plus d’informations, consultez [Limites de services cloud](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-cloud-services-limits).

Vous pouvez aussi suivre l’utilisation et/ou le quota actuel de votre abonnement sur le portail : Portail Azure => Abonnements => \<appropriate subscription> => « Utilisation + quota ».

Vous pouvez aussi obtenir des informations sur l’utilisation/consommation de ressources via les API de facturation Azure. Consultez [API Azure Resource Usage (version préliminaire)](../cost-management-billing/manage/usage-rate-card-overview.md#azure-resource-usage-api-preview).

## <a name="how-can-i-change-the-size-of-a-deployed-cloud-service-vm-without-redeploying-it"></a>Comment modifier la taille de machine virtuelle d’un service cloud déployé sans le redéployer ?
Vous ne pouvez pas modifier la taille de machine virtuelle d’un service cloud déployé sans le redéployer. La taille de machine virtuelle est intégrée au fichier CSDEF, dont la mise à jour n’est possible que moyennant un redéploiement.

Pour plus d’informations, consultez [Comment mettre à jour un service clou](cloud-services-update-azure-service.md).

## <a name="why-am-i-not-able-to-deploy-cloud-services-through-service-management-apis-or-powershell-when-using-azure-resource-manager-storage-account"></a>Pourquoi ne puis-je pas déployer de services cloud via les API Gestion des services ou PowerShell en utilisant le compte de stockage Azure Resource Manager ? 

Étant donné que le service cloud est une ressource Classic qui n’est pas directement compatible avec le modèle Azure Resource Manager, vous ne pouvez pas l’associer aux comptes de stockage Azure Resource Manager. Voici quelques solutions alternatives possibles : 

- Déploiement via l’API REST.

    Quand vous déployez via l’API REST Gestion des services, vous pouvez obtenir la limitation en spécifiant une URL SAS pour le stockage d’objets blob, qui fonctionnera à la fois avec un compte de stockage Azure Resource Manager et classique. Pour en savoir plus sur la propriété 'PackageUrl', cliquez [ici](/previous-versions/azure/reference/ee460813(v=azure.100)).

- Déploiement via le [portail Azure](https://portal.azure.com).

    Le déploiement est possible à partir du [portail Azure](https://portal.azure.com), car l’appel passe par un proxy/shim qui permet la communication entre les ressources Azure Resource Manager et Classic. 

## <a name="why-does-azure-portal-require-me-to-provide-a-storage-account-for-deployment"></a>Pourquoi le portail Azure me demande-t-il de fournir un compte de stockage pour le déploiement ?

Dans le portail classique, le package a été chargé directement sur la couche API de gestion, après quoi la couche API place temporairement le package dans un compte de stockage interne.  Ce processus entraîne des problèmes de performances et d’extensibilité, car la couche API n’a pas été conçue pour être utilisée comme un service de téléchargement de fichiers.  Dans le portail Azure (modèle de déploiement Resource Manager), nous avons contourné l’étape intermédiaire de téléchargement préalable sur la couche API, ce qui permet des déploiements plus rapides et plus fiables.

En ce qui concerne le coût, celui-ci est très faible et vous pouvez réutiliser le même compte de stockage pour tous les déploiements. Vous pouvez utiliser la [calculatrice de prix pour le stockage](https://azure.microsoft.com/pricing/calculator/#storage1) pour déterminer le coût pour charger le package de services (CSPKG), le télécharger, puis le supprimer.
