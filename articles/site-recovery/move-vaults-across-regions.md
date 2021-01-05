---
title: Déplacer un coffre Azure Site Recovery vers une autre région
description: Explique comment déplacer un coffre Recovery Services (Azure Site Recovery) vers une autre région Azure
services: site-recovery
author: Sharmistha-Rai
ms.service: site-recovery
ms.topic: tutorial
ms.date: 07/31/2019
ms.author: sharrai
ms.custom: MVC
ms.openlocfilehash: a8cb5ec782b5932c13e321b2ba2d6513597fef52
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87422638"
---
# <a name="move-a-recovery-services-vault-and-azure-site-recovery-configuration-to-another-azure-region"></a>Déplacer un coffre Recovery Services et une configuration Azure Site Recovery vers une autre région Azure

Il existe différents scénarios dans lesquels vous pouvez avoir besoin de déplacer vos ressources Azure existantes d’une région à une autre. Par exemple, cela peut être pour faciliter la gestion, pour des impératifs de gouvernance ou en raison d’une fusion ou acquisition d’entreprise. L’une des ressources associées que vous aimerez peut-être conserver lors du déplacement de vos machines virtuelles Azure est la configuration de la reprise d’activité. 

Il n’existe pas de méthode idéale permettant de déplacer une configuration de reprise d’activité existante d’une région à une autre. Cela est dû au fait que vous avez configuré votre région cible en fonction de votre région source pour les machines virtuelles. Si vous décidez de changer la région source, les précédentes configurations existantes de la région cible ne sont pas réutilisables et doivent être redéfinies. Cet article décrit le processus pas à pas de la reconfiguration de la reprise d’activité, et de son déplacement vers une autre région.

Dans ce document, vous allez :

> [!div class="checklist"]
> * Vérifier les prérequis du déplacement.
> * Identifier les ressources qui ont été utilisées par Azure Site Recovery.
> * Désactiver la réplication.
> * Supprimer les ressources.
> * Configurer Site Recovery en fonction de la nouvelle région source pour les machines virtuelles.

> [!IMPORTANT]
> Actuellement, il n’existe pas de méthode idéale pour déplacer en l’état un coffre Recovery Services et la configuration de la reprise d’activité vers une autre région. Cet article vous guide tout au long du processus de désactivation de la réplication et de sa configuration dans la nouvelle région.

## <a name="prerequisites"></a>Conditions préalables requises

- Veillez à bien retirer et supprimer la configuration de la reprise d’activité avant d’essayer de déplacer les machines virtuelles Azure vers une autre région. 

  > [!NOTE]
  > Si votre nouvelle région cible pour la machine virtuelle Azure est la même que la région cible de la reprise d’activité, vous pouvez utiliser votre configuration de réplication existante et la déplacer. Effectuez les étapes décrites dans [Déplacer des machines virtuelles Azure IaaS vers une autre région Azure](azure-to-azure-tutorial-migrate.md).

- Veillez à prendre une décision fondée et à en informer les parties prenantes. Votre machine virtuelle n’est pas protégée contre les sinistres tant que son déplacement n’est pas terminé.

## <a name="identify-the-resources-that-were-used-by-azure-site-recovery"></a>Identifier les ressources qui ont été utilisées par Azure Site Recovery
Nous vous recommandons d’effectuer cette étape avant de passer à la suivante. Il est plus facile d’identifier les ressources pertinentes pendant la réplication des machines virtuelles.

Pour chaque machine virtuelle Azure répliquée, accédez à **Éléments protégés** > **Éléments répliqués** > **Propriétés** et identifiez les ressources suivantes :

- Groupe de ressources cible
- Compte de stockage de cache
- Compte de stockage cible (dans le cas d’une machine virtuelle Azure sur disque non managé) 
- Réseau cible


## <a name="disable-the-existing-disaster-recovery-configuration"></a>Désactiver la configuration de reprise d’activité existante

1. Accédez au coffre Recovery Services.
2. Dans **Éléments protégés** > **Éléments répliqués**, cliquez avec le bouton droit sur la machine et sélectionnez **Désactiver la réplication**.
3. Répétez cette étape pour toutes les machines virtuelles à déplacer.

> [!NOTE]
> Le service Mobilité n’est pas désinstallé sur les serveurs protégés. Vous devez le désinstaller manuellement. Si vous envisagez de reprotéger le serveur, vous pouvez ignorer la désinstallation du service Mobilité.

## <a name="delete-the-resources"></a>Supprimer les ressources

1. Accédez au coffre Recovery Services.
2. Sélectionnez **Supprimer**.
3. Supprimez toutes les autres ressources que vous avez [identifiées précédemment](#identify-the-resources-that-were-used-by-azure-site-recovery).
 
## <a name="move-azure-vms-to-the-new-target-region"></a>Déplacer les machines virtuelles Azure vers la nouvelle région cible

Effectuez les étapes indiquées dans ces articles, en fonction de vos besoins, pour déplacer les machines virtuelles Azure vers la région cible :

- [Déplacer des machines virtuelles Azure vers une autre région](azure-to-azure-tutorial-migrate.md)
- [Déplacer des machines virtuelles Azure vers des zones de disponibilité](move-azure-VMs-AVset-Azone.md)

## <a name="set-up-site-recovery-based-on-the-new-source-region-for-the-vms"></a>Configurer Site Recovery en fonction de la nouvelle région source pour les machines virtuelles

Configurez la reprise d’activité pour les machines virtuelles Azure ayant été déplacées vers la nouvelle région en effectuant les étapes décrites dans [Configurer la reprise d’activité pour les machines virtuelles Azure](azure-to-azure-tutorial-enable-replication.md).
