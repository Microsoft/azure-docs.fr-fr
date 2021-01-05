---
title: Configurer la récupération d’urgence Azure VM vers une région secondaire avec Azure Site Recovery
description: Configurez rapidement la récupération d’urgence vers une autre région Azure pour une machine virtuelle Azure à l'aide du service Azure Site Recovery.
ms.topic: quickstart
ms.date: 03/27/2020
ms.custom: mvc
ms.openlocfilehash: 8681ccaac30402927005e5e8802d7da2bddc2c5c
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "86135704"
---
# <a name="quickstart-set-up-disaster-recovery-to-a-secondary-azure-region-for-an-azure-vm"></a>Démarrage rapide : Configurer la récupération d’urgence vers une région Azure secondaire pour une machine virtuelle Azure

Le service [Azure Site Recovery](site-recovery-overview.md) contribue à votre stratégie de continuité d’activité et de reprise d’activité en maintenant vos applications métier en ligne pendant les interruptions planifiées et non planifiées. Site Recovery gère et orchestre la reprise d’activité des machines locales et des machines virtuelles Azure, notamment la réplication, le basculement et la récupération.

Ce guide de démarrage rapide explique comment configurer la reprise d’activité pour une machine virtuelle Azure en la répliquant vers une région Azure secondaire. En général, les paramètres par défaut sont utilisés pour activer la réplication.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous avez besoin d’un abonnement Azure et d’une machine virtuelle.

- Si vous n’avez pas de compte Azure avec un abonnement actif, vous pouvez [créer un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Nous recommandons une machine virtuelle avec au moins 1 Go de RAM. [Découvrez-en plus](../virtual-machines/windows/quick-create-portal.md) sur la création d’une machine virtuelle.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="enable-replication-for-the-azure-vm"></a>Activer la réplication des machines virtuelles Azure

Effectuez les étapes suivantes pour répliquer une machine virtuelle dans un emplacement secondaire.

1. Dans le portail Azure, accédez au menu **Accueil** > **Machines virtuelles**, puis sélectionnez une machine virtuelle à répliquer.
1. Dans **Opérations**, sélectionnez **Récupération d’urgence**.
1. Dans **De base** > **Région cible**, sélectionnez la région cible.
1. Pour voir les paramètres de réplication, sélectionnez **Examiner + démarrer la réplication**. Si vous avez besoin de changer des valeurs par défaut, sélectionnez **Paramètres avancés**.
1. Pour démarrer le travail qui active la réplication de la machine virtuelle, sélectionnez **Démarrer la réplication**.

   :::image type="content" source="media/azure-to-azure-quickstart/enable-replication1.png" alt-text="Activez la réplication.":::

## <a name="verify-settings"></a>Vérifier les paramètres

Une fois le travail de réplication terminé, vous pouvez vérifier l’état de la réplication, modifier les paramètres de réplication et tester le déploiement.

1. Dans le menu du portail Azure, sélectionnez **Machines virtuelles**, puis la machine virtuelle que vous avez répliquée.
1. Dans **Opérations**, sélectionnez **Récupération d’urgence**.
1. Pour voir les détails de réplication à partir de la **Vue d’ensemble**, sélectionnez **Bases**. Vous trouverez plus de détails dans les sections **Intégrité et état** et **Disponibilité du basculement** ainsi que dans la carte **Affichage de l’infrastructure**.

   :::image type="content" source="media/azure-to-azure-quickstart/replication-status.png" alt-text="Activez la réplication.":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour arrêter la réplication de la machine virtuelle dans la région primaire, vous devez désactiver la réplication :

- Les paramètres de réplication source sont automatiquement nettoyés.
- L’extension Site Recovery installée sur la machine virtuelle pendant la réplication n’est pas supprimée.
- La facturation Site Recovery pour la machine virtuelle cesse.

Pour désactiver la réplication, effectuez les étapes suivantes :

1. Dans le menu du portail Azure, sélectionnez **Machines virtuelles**, puis la machine virtuelle que vous avez répliquée.
1. Dans **Opérations**, sélectionnez **Récupération d’urgence**.
1. Dans la **Vue d’ensemble**, sélectionnez **Désactiver la réplication**.
1. Pour désinstaller l’extension Site Recovery, accédez à la machine virtuelle et sélectionnez **Paramètres** > **Extensions**.

   :::image type="content" source="media/azure-to-azure-quickstart/disable2-replication.png" alt-text="Activez la réplication.":::.

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez répliqué une seule machine virtuelle vers une région secondaire. Ensuite, configurez la réplication pour plusieurs machines virtuelles Azure.

> [!div class="nextstepaction"]
> [Configurer la récupération d’urgence pour les machines virtuelles Azure](azure-to-azure-tutorial-enable-replication.md)
