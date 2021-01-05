---
title: Déplacement des images de machines virtuelles de la Place de marché Azure vers un stockage sur disque managé
description: Afin de fournir un stockage et une prise en charge plus rapides et plus fiables des nouvelles fonctionnalités et capacités de la Place de marché, nous allons déplacer les images de machines virtuelles de la Place de marché vers un stockage sur disque managé.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 11/22/2019
author: iqshahmicrosoft
ms.author: iqshah
ms.openlocfilehash: 814a4012ace95e84e75bcb4071c549822b2b0633
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89182426"
---
# <a name="were-moving-virtual-machine-vm-images-on-azure-marketplace-to-managed-disk-storage"></a>Déplacement des images de machines virtuelles de la Place de marché Azure vers un stockage sur disque managé

Afin de fournir un stockage et une prise en charge plus rapides et plus fiables des nouvelles fonctionnalités et capacités de la Place de marché, nous allons déplacer les images de machines virtuelles de la Place de marché vers un stockage sur disque managé.

À compter du 2 janvier 2020, nous allons progressivement déplacer les images de machines virtuelles vers un stockage sur disque managé. Durant la première phase, nous déplacerons uniquement les images qui ne contiennent pas de nouveaux déploiements ni de machines virtuelles ayant été exécutées au cours des 90 derniers jours. Avant de déplacer des images, nous enverrons un e-mail aux éditeurs pour leur indiquer quelles images seront déplacées et à quelle date.

Aucune action n’est requise de la part des éditeurs ou des consommateurs, et il n’y aura aucun impact pour les utilisateurs. Les offres de la Place de marché resteront disponibles, et les clients pourront continuer à déployer des machines virtuelles managées à partir des images déplacées, pendant et après leur déplacement.

Si vous avez des questions, n’hésitez pas à [nous contacter](https://support.microsoft.com/supportforbusiness/productselection?sapId=48734891-ee9a-5d77-bf29-82bf8d8111ff).

## <a name="faqs"></a>FAQ

### <a name="would-the-users-of-my-vm-images-experience-an-outage"></a>Les utilisateurs de mes images de machines virtuelles subiront-ils des temps d’arrêt ?

Les utilisateurs des images de machines virtuelles ne subiront aucun temps d’arrêt. 

Durant la première phase, nous déplacerons uniquement les images de machines virtuelles où aucune machine virtuelle n’est en cours d’exécution. Comme ces images ne seront utilisées par personne, il n’y aura aucun impact pour les utilisateurs. Durant les phases suivantes, il n’y aura pas d’impact non plus pour les utilisateurs.

### <a name="how-long-does-it-take-for-the-process-to-complete"></a>Combien de temps durera le processus au total ?

La migration pourra prendre jusqu’à 24 heures.

### <a name="do-i-need-to-take-any-action"></a>Aurai-je quelque chose à faire de mon côté ?

Non. Aucune action n’est requise de la part des éditeurs ou des consommateurs.

### <a name="do-i-have-to-update-my-system-to-call-the-cloud-portal-apis-in-a-different-way-after-they-are-moved-to-managed-disk-storage"></a>Aurai-je besoin de mettre à jour mon système pour appeler les API du portail cloud d’une autre façon une fois qu’elles auront été déplacées vers le stockage sur disque managé ?

Non. Tous vos appels d’API continueront de fonctionner comme avant.

### <a name="would-all-my-vm-images-be-moved-to-managed-disk-at-the-same-time"></a>Mes images de machines virtuelles seront-elles toutes déplacées en même temps vers le disque managé ?

Oui. Nous déplacerons toutes vos images de machines virtuelles le même jour. Nous vous avertirons quand leur déplacement sera terminé.

### <a name="can-i-request-to-schedule-the-move-of-my-vm-images-to-a-later-time"></a>Puis-je demander un report du déplacement planifié de mes images de machines virtuelles ?

Nous vous recommandons d’accepter le déplacement des images à la date planifiée. Toutefois, si cela vous pose des problèmes, n’hésitez à nous contacter pour changer la date du déplacement.

### <a name="can-i-publish-updates-to-my-vm-images-during-the-move"></a>Pourrai-je publier des mises à jour sur mes images de machines virtuelles durant le déplacement ?

Aucune mise à jour ne pourra être apportée aux images de machines virtuelles pendant le déplacement.

### <a name="will-the-publishing-process-change-after-my-vm-image-is-moved-to-managed-disk"></a>Le processus de publication changera-t-il après le déplacement de mon image de machine virtuelle vers le disque managé ?

Non, le processus de publication restera le même. 

### <a name="can-the-publisher-move-their-offers-to-managed-disk"></a>Est-ce que l’éditeur peut déplacer ses offres vers le disque managé ?

Non, les éditeurs ne peuvent pas déplacer leurs offres vers le disque managé. Ils devront attendre et leurs images seront déplacées automatiquement. Nous enverrons des notifications à l’éditeur avant d’apporter des modifications.
