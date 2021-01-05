---
title: FAQ sur le calcul confidentiel Azure
description: Réponses aux questions fréquemment posées sur le calcul confidentiel Azure.
author: JBCook
ms.topic: troubleshooting
ms.workload: infrastructure
ms.service: virtual-machines
ms.subservice: workloads
ms.date: 4/17/2020
ms.author: jencook
ms.openlocfilehash: 9df3d9771029e6d72e9d0092a129cddc27be6cd7
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94564104"
---
# <a name="frequently-asked-questions-for-azure-confidential-computing"></a>Forum aux questions (FAQ) sur le calcul confidentiel Azure

Cet article fournit des réponses à certaines des questions les plus courantes concernant l'exécution de [charges de travail d'informatique confidentielle sur des machines virtuelles Azure](overview.md).

Si le problème lié à Azure n’est pas traité dans cet article, parcourez les forums Azure sur [MSDN et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez publier votre problème sur ces forums ou [@AzureSupport sur Twitter](https://twitter.com/AzureSupport). Vous pouvez également soumettre une demande de support Azure. Pour soumettre une demande de support, sur la [page de support Azure](https://azure.microsoft.com/support/options/), sélectionnez Obtenir de l’aide.

## <a name="confidential-computing-virtual-machines"></a>Machines virtuelles de calcul confidentiel <a id="vm-faq"></a>

**Comment déployer des machines virtuelles de la série DCsv2 sur Azure ?**

Voici quelques méthodes permettant de déployer une machine virtuelle DCsv2 :
   - En utilisant un [modèle Azure Resource Manager](../virtual-machines/windows/template-description.md).
   - à partir du [Portail Azure](https://portal.azure.com/#create/hub)
   - Dans le modèle de solution de la Place de marché [Azure Confidential Computing (Virtual Machine)](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-compute.acc-virtual-machine-v2?tab=overview). Le modèle de solution de la Place de marché permet de contraindre un client aux scénarios pris en charge (régions, images, disponibilité, chiffrement de disque). 

**Toutes les images de système d’exploitation fonctionnent-elles avec le calcul confidentiel Azure ?**

Non. Les machines virtuelles peuvent uniquement être déployées sur des machines de deuxième génération avec Ubuntu Server 18.04, Ubuntu Server 16.04, Windows Server 2019 Datacenter et Windows Server 2016 Datacenter. En savoir plus sur les machines virtuelles de la génération 2 sur [Linux](../virtual-machines/generation-2.md) et [Windows](../virtual-machines/generation-2.md)

**Les machines virtuelles DCsv2 sont grisées dans le portail et je ne peux en sélectionner aucune**

Selon l’infobulle en regard de la machine virtuelle, différentes actions sont possibles :
   -    **UnsupportedGeneration** : Modifiez la génération de l’image de machine virtuelle en « Gen2 ».
   -    **NotAvailableForSubscription** : La région n’est pas encore disponible pour votre abonnement. Sélectionnez une région disponible.
   -    **InsufficientQuota** : [Créez une demande de support pour augmenter votre quota](../azure-portal/supportability/per-vm-quota-requests.md). Les abonnements d’évaluation gratuite n’ont pas de quota pour les machines virtuelles de calcul confidentiel. 

**Les machines virtuelles DCsv2 ne s’affichent pas lorsque je les recherche dans le sélecteur de taille du portail**

Vérifiez que vous avez sélectionné une [région disponible](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines). Veillez également à activer l’option « Effacer tous les filtres » dans le sélecteur de taille. 

**J’obtiens une erreur m’informant de l’échec du déploiement avec un modèle Azure Resource Manager : « Impossible d’effectuer l’opération, car elle entraîne un dépassement du quota approuvé de cœurs pour la famille DcsV2 standard »**

[Créez une demande de support pour augmenter votre quota](../azure-portal/supportability/per-vm-quota-requests.md). Les abonnements d’évaluation gratuite n’ont pas de quota pour les machines virtuelles de calcul confidentiel. 

**Quelle est la différence entre les machines virtuelles des séries DCsv2 et DC ?**

Les machines virtuelles de la série DC s’exécutent sur des processeurs Intel à 6 cœurs plus anciens avec Intel SGX et ont moins de mémoire totale, ont moins de mémoire EPC (Enclave Page Cache) et ne sont disponibles que dans deux régions (USA Est et Europe Ouest dans les tailles Standard_DC2s et Standard_DC4s). Ces machines virtuelles ne seront pas mises à la disposition générale et ne sont pas recommandées en production. Pour déployer ces machines virtuelles, utilisez l’instance de la Place de marché [Confidential Compute DC-Series VM [Preview]](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-compute.confidentialcompute?tab=Overview).

**Les machines virtuelles DCsv2 sont-elles disponibles dans le monde entier ?**

Non. Pour l’instant, ces machines virtuelles ne sont disponibles que dans certaines régions. Pour connaître les dernières régions disponibles, consultez la [page des produits par région](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines). 

**L’hyper-threading est-il désactivé sur ces ordinateurs ?**

L’hyper-threading est désactivé pour tous les clusters informatiques confidentiels Azure.

**Comment installer le SDK Open enclave sur les machines virtuelles DCsv2 ?**
   
Pour obtenir des instructions sur l’installation du SDK Open Enclave sur une machine Azure ou locale, suivez les instructions de la [page GitHub du SDK Open Enclave](https://github.com/openenclave/openenclave).
     
Vous pouvez également accéder à la page GitHub du SDK Open Enclave pour obtenir des instructions d’installation spécifiques au système d’exploitation :
   - [Installer le kit de développement logiciel (SDK) Open Enclave sur Windows](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Windows.md)
   - [Installer le kit de développement logiciel (SDK) Open Enclave sur Ubuntu 18.04](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Ubuntu_18.04.md)
   - [Installer le kit de développement logiciel (SDK) Open Enclave sur Ubuntu 16.04](https://github.com/openenclave/openenclave/blob/master/docs/GettingStartedDocs/install_oe_sdk-Ubuntu_16.04.md)