---
title: Support étendu du cycle de vie Red Hat Enterprise Linux
description: Découvrez le module complémentaire de support du cycle de vie étendu de Red Hat Enterprise
author: mathapli
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.date: 04/16/2020
ms.author: mathapli
ms.reviewer: cynthn
ms.openlocfilehash: 1f31cc982e5e7f92cfe7e8adf588dd96628f2c6f
ms.sourcegitcommit: 1bf144dc5d7c496c4abeb95fc2f473cfa0bbed43
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95744039"
---
# <a name="red-hat-enterprise-linux-rhel-extended-lifecycle-support"></a>Support du cycle de vie étendu de Red Hat Enterprise Linux (RHEL)
Cet article fournit des informations sur le support du cycle de vie étendu des images Red Hat Enterprise :
* Red Hat Enterprise Linux 6  

## <a name="red-hat-enterprise-linux-6-lifecycle"></a>Cycle de vie de Red Hat Enterprise Linux 6
À partir du 30 novembre 2020, Red Hat Enterprise Linux 6 atteindra la fin de la phase de maintenance. La phase de maintenance est suivie de la phase de vie étendue. Lorsque Red Hat Enterprise Linux 6 approche de la fin des phases complète/de maintenance, il est fortement recommandé de procéder à une mise à niveau vers Red Hat Enterprise Linux 7 ou 8. Si des clients doivent rester sur Red Hat Enterprise Linux 6, il est recommandé d’ajouter le module complémentaire de support du cycle de vie étendu (ELS) de Red Hat Enterprise Linux.

## <a name="steps-to-add-extended-lifecycle-support-on-marketplace-pay-as-you-go-vms"></a>Procédure d’ajout de la prise en charge du cycle de vie étendu sur des machines virtuelles Place de marché avec paiement à l’utilisation
1. Renseignez le [formulaire ELS disponible ici](https://aka.ms/els-form) en indiquant vos coordonnées ainsi que les informations d’abonnement des machines virtuelles pour lesquelles vous souhaitez ajouter la prise en charge ELS. Les tarifs détaillés du module complémentaire sont également disponibles dans le formulaire.
1. L’équipe Azure Red Hat Enterprise Linux vous transmettra la liste des machines virtuelles acceptant le module complémentaire de prise en charge ELS d’ici 1 à 2 jours ouvrables. Veuillez consulter la liste et répondre en acceptant le tarif du module complémentaire.
1. L’équipe Azure Red Hat Enterprise Linux communiquera les étapes permettant d’ajouter le package client ELS aux machines virtuelles. Suivez les étapes qui seront fournies dans l’e-mail, pour continuer à bénéficier de la maintenance logicielle (correctifs de bogues et de sécurité) et du support pour Red Hat Enterprise Linux 6.

> [!Note]
> Ne partagez pas les étapes de l’utilisation du module complémentaire RHEL ELS avec une personne extérieure à votre organisation. Contactez AzureRedHatELS@microsoft.com pour obtenir de l’aide ou pour toute autre question.

## <a name="frequently-asked-questions"></a>Forum Aux Questions (FAQ)

#### <a name="im-running-red-hat-enterprise-linux-6-and-cant-migrate-to-a-later-version-at-this-time-what-options-do-i-have"></a>J’exécute Red Hat Enterprise Linux 6 et ne peut pas migrer vers une version ultérieure pour l’instant. Quelles sont les options dont je dispose ?
* Continuez à exécuter Red Hat Enterprise Linux 6 et achetez les référentiels du module complémentaire de support du cycle de vie étendu (ELS) pour continuer à bénéficier d’une maintenance logicielle et d’un support technique limités (voir le processus de mise à niveau et les détails de tarification ci-dessous).
* Migrez vers Red Hat Enterprise Linux 7 ou 8 dès que possible.

#### <a name="what-is-the-additional-charge-for-using-red-hat-enterprise-linux-extended-life-cycle-support-els-add-on"></a>Quels sont les frais supplémentaires liés à l’utilisation du module complémentaire de support du cycle de vie étendu (ELS) de Red Hat Enterprise Linux ?
Les coûts liés à la prise en charge du cycle de vie étendu sont disponibles dans le [formulaire ELS](https://aka.ms/els-form)


## <a name="next-steps"></a>Étapes suivantes

* Pour voir la liste complète des images RHEL dans Azure, consultez [Images RHEL (Red Hat Enterprise Linux) disponibles dans Azure](./redhat-imagelist.md).
* Pour en savoir plus sur Azure Red Hat Update Infrastructure, consultez [Red Hat Update Infrastructure pour les machines virtuelles RHEL à la demande dans Azure](./redhat-rhui.md).
* Pour en savoir plus sur l’offre RHEL BYOS, consultez [Images Gold BYOS (Apportez votre propre abonnement) de Red Hat Enterprise Linux dans Azure](./byos.md).
* Pour obtenir des informations sur les stratégies de prise en charge de Red Hat pour toutes les versions de RHEL, consultez la page [Cycle de vie de Red Hat Enterprise Linux](https://access.redhat.com/support/policy/updates/errata).