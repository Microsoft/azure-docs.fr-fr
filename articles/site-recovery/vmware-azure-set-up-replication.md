---
title: Configurer des stratégies de réplication pour la reprise d’activité après sinistre de VMware avec Azure Site Recovery | Microsoft Docs
description: Décrit comment configurer les paramètres de réplication en vue d’une récupération d’urgence VMware sur Azure avec Azure Site Recovery.
author: sujayt
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: sutalasi
ms.openlocfilehash: 45921bdf802a649b7b802f44d2842a543e44f02b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84699598"
---
# <a name="configure-and-manage-replication-policies-for-vmware-disaster-recovery"></a>Configurer et gérer des stratégies de réplication pour la reprise d’activité après sinistre de VMware

Cet article explique comment configurer une stratégie de réplication pour la réplication d’une machine virtuelle VMware sur Azure avec [Azure Site Recovery](site-recovery-overview.md).

## <a name="create-a-policy"></a>Créer une stratégie

1. Sélectionnez **Gérer** > **Infrastructure Site Recovery**.
2. Sélectionnez **Stratégies de réplication** dans **Pour les machines VMware et physiques**.
3. Cliquez sur **+ Stratégie de réplication**, et spécifiez le nom de la stratégie.
4. Dans le champ **Seuil d’objectif de point de récupération**, spécifiez la limite de l’objectif de point de récupération. Des alertes sont générées lorsque la réplication continue dépasse cette limite.
5. Dans **Rétention des points de récupération**, spécifiez la durée (en heures) de la fenêtre de rétention pour chaque point de récupération. Les machines protégées peuvent être récupérées à tout moment pendant cette fenêtre de rétention. Les machines virtuelles répliquées vers le Stockage Premium peuvent prendre en charge jusqu’à 24 heures de rétention. Le stockage standard prend en charge jusqu’à 72 heures de rétention.
6. Dans **Fréquence des captures instantanées de cohérence d’application**, choisissez dans la liste déroulante la fréquence (en heures) à laquelle doivent être créés des points de récupération qui contiennent des instantanés de cohérence d’application. Pour désactiver la génération de points de cohérence d’application, choisissez la valeur « Désactivé » dans la liste déroulante.
7. Cliquez sur **OK**. La création de la stratégie devrait prendre entre 30 et 60 secondes.

Lorsque vous créez une stratégie de réplication, une stratégie de correspondance est automatiquement créée pour la restauration automatique, avec le suffixe « failback ». Une fois la stratégie créée, vous pouvez la modifier en sélectionnant **Modifier les paramètres**.

## <a name="associate-a-configuration-server"></a>Associer un serveur de configuration

Associez la stratégie de réplication à votre serveur de configuration local.

1. Cliquez sur **Associer**, puis sélectionnez le serveur de configuration.

    ![Associer un serveur de configuration](./media/vmware-azure-set-up-replication/associate1.png)
2. Cliquez sur **OK**. L’association du serveur de configuration devrait prendre entre une et deux minutes.

    ![Association du serveur de configuration](./media/vmware-azure-set-up-replication/associate2.png)

## <a name="edit-a-policy"></a>Modifier une stratégie

Vous pouvez modifier une stratégie de réplication après l’avoir créée.

- Les modifications apportées à la stratégie sont appliquées à toutes les machines utilisant la stratégie.
- Si vous souhaitez associer des machines répliquées à une stratégie de réplication différente, vous devez désactiver et réactiver la protection pour les machines concernées.

Modifiez une stratégie comme suit :
1. Sélectionnez **Gérer** > **Infrastructure Site Recovery** > **Stratégies de réplication**.
2. Sélectionnez la stratégie de réplication à modifier.
3. Cliquez sur **Modifier les paramètres** et mettez à jour les champs Seuil d’objectif de point de récupération/Délai de rétention des points de récupération en heures/Fréquence des captures instantanées de cohérence d’application selon vos besoins.
4. Si vous souhaitez désactiver la génération de points de cohérence d’application, choisissez la valeur « Désactivé » dans la liste déroulante du champ **Fréquence des captures instantanées de cohérence d’application**.
5. Cliquez sur **Enregistrer**. La mise à jour de la stratégie devrait prendre entre 30 et 60 secondes.



## <a name="disassociate-or-delete-a-replication-policy"></a>Dissocier ou supprimer une stratégie de réplication

1. Choisissez la stratégie de réplication.
    a. Pour dissocier la stratégie du serveur de configuration, veillez à ce qu’aucune machine répliquée n’utilise la stratégie. Ensuite, cliquez sur **Dissocier**.
    b. Pour supprimer la stratégie, veillez à ce qu’elle ne soit associée à aucun serveur de configuration. Ensuite, cliquez sur **Supprimer**. L’opération de suppression dure entre 30 et 60 secondes.
2. Cliquez sur **OK**.
