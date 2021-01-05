---
title: Résoudre les erreurs de transfert
description: Résolvez les problèmes liés au déplacement de ressources vers un nouveau groupe de ressource ou un nouvel abonnement.
ms.topic: conceptual
ms.date: 08/27/2019
ms.openlocfilehash: 41b1e2435caf9874f3582a3394664c7b7f5a8d29
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90054160"
---
# <a name="troubleshoot-moving-azure-resources-to-new-resource-group-or-subscription"></a>Résoudre les problèmes liés au déplacement de ressources vers un nouveau groupe de ressource ou abonnement

Cet article propose des suggestions pour résoudre les problèmes liés au déplacement de ressources.

## <a name="upgrade-a-subscription"></a>Mettre à niveau un abonnement

Vous devez convertir votre abonnement Azure si vous souhaitez le mettre à niveau (par exemple, passer d’une offre gratuite au paiement à l’utilisation).

* Pour mettre à niveau votre essai gratuit, consultez [Passer d’un essai gratuit ou d’un abonnement Azure Microsoft Imagine au paiement à l’utilisation](../../cost-management-billing/manage/upgrade-azure-subscription.md).
* Pour changer un compte de paiement à l’utilisation, consultez [Remplacer votre abonnement Paiement à l’utilisation Azure par une autre offre](../../cost-management-billing/manage/switch-azure-offer.md).

Si vous ne pouvez pas convertir l’abonnement, [créez une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). Sélectionnez **Gestion des abonnements** comme type de problème.

## <a name="service-limitations"></a>Limitations de service

Certains services doivent faire l'objet de considérations supplémentaires en termes de déplacement de ressources. Si vous déplacez les services suivants, assurez-vous d'avoir vérifié les conseils et limitations.

* [Services d’application](./move-limitations/app-service-move-limitations.md)
* [Azure DevOps Services](/azure/devops/organizations/billing/change-azure-subscription?toc=/azure/azure-resource-manager/toc.json)
* [Modèle de déploiement classique](./move-limitations/classic-model-move-limitations.md)
* [Mise en réseau](./move-limitations/networking-move-limitations.md)
* [Recovery Services](../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json)
* [Machines virtuelles](./move-limitations/virtual-machines-move-limitations.md)

## <a name="large-requests"></a>Requêtes volumineuses

Quand c’est possible, divisez les grands déplacements en opérations de déplacement distinctes. Resource Manager retourne immédiatement une erreur en cas de tentative de déplacement de plus de 800 ressources en une seule opération. Cependant, un déplacement de moins de 800 ressources peut également échouer en raison d’un dépassement du délai d’expiration.

## <a name="resource-not-in-succeeded-state"></a>Ressource qui n’est pas dans un état de réussite

Quand vous obtenez un message d’erreur indiquant qu’une ressource ne peut pas être déplacée parce qu’elle n’est pas dans un état de réussite, le problème peut en fait être lié à une ressource dépendante qui bloque le déplacement. En général, le code d’erreur est **MoveCannotProceedWithResourcesNotInSucceededState**.

Si le groupe de ressources source ou cible contient un réseau virtuel, les états de toutes les ressources dépendantes pour le réseau virtuel sont vérifiés au cours du déplacement. La vérification comprend les ressources qui dépendent directement et indirectement du réseau virtuel. Si l’une de ces ressources est en état d’échec, le déplacement est bloqué. Par exemple, si une machine virtuelle qui utilise le réseau virtuel a échoué, le déplacement est bloqué. Le déplacement est bloqué même lorsque la machine virtuelle n’est pas l’une des ressources déplacées et ne se trouve pas dans l’un des groupes de ressources à déplacer.

Lorsque vous recevez cette erreur, deux options s’offrent à vous. Déplacez vos ressources vers un groupe de ressources qui n’a pas de réseau virtuel ou [contactez le support technique](../../azure-portal/supportability/how-to-create-azure-support-request.md).

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir les commandes permettant de déplacer des ressources, consultez [Déplacer des ressources vers un nouveau groupe de ressources ou un nouvel abonnement](move-resource-group-and-subscription.md).
