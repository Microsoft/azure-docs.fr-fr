---
title: Sauvegarder des ordinateurs Windows en utilisant l’agent MARS
description: Utilisez l’agent Microsoft Azure Recovery Services (MARS) pour sauvegarder des ordinateurs Windows.
ms.topic: conceptual
ms.date: 03/03/2020
ms.openlocfilehash: d2cdd5d1fa98462a70d72fd9f8723685952b665a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90052220"
---
# <a name="back-up-windows-server-files-and-folders-to-azure"></a>Sauvegarder les fichiers et dossiers de Windows Server sur Azure

Cet article explique comment sauvegarder des ordinateurs Windows à l’aide du service [Sauvegarde Azure](backup-overview.md) et de l’agent MARS (Microsoft Azure Recovery Services). MARS est également connu sous le nom d’agent de Sauvegarde Azure.

Dans cet article, vous allez apprendre à :

> [!div class="checklist"]
>
> * Vérifier les prérequis
> * Créez une stratégie et une planification de la sauvegarde.
> * Effectuez une sauvegarde à la demande.

## <a name="before-you-start"></a>Avant de commencer

* Découvrez comment le service [Sauvegarde Azure utilise l’agent MARS pour sauvegarder les ordinateurs Windows](backup-architecture.md#architecture-direct-backup-of-on-premises-windows-server-machines-or-azure-vm-files-or-folders).
* Découvrez l’[architecture de sauvegarde](backup-architecture.md#architecture-back-up-to-dpmmabs) qui exécute l’agent MARS sur un serveur MABS ou Data Protection Manager secondaire.
* Examinez [ce qui est pris en charge et ce qui peut être sauvegardé](backup-support-matrix-mars-agent.md) par l’agent MARS.
* [Vérifiez l’accès Internet](install-mars-agent.md#verify-internet-access) sur les ordinateurs à sauvegarder.
* Si l’agent MARS n’est pas installé, découvrez comment l’installer [ici](install-mars-agent.md).

## <a name="create-a-backup-policy"></a>Créer une stratégie de sauvegarde

La stratégie de sauvegarde spécifie à quel moment pendre des captures d’instantanées des données pour créer des points de récupération. Elle spécifie aussi la durée de conservation des points de récupération. Pour configurer une stratégie de sauvegarde, vous devez utiliser l’agent MARS.

Le service Sauvegarde Azure ne prend pas automatiquement en compte l’heure d’été. Un décalage entre l’heure réelle et l’heure de sauvegarde planifiée peut résulter de cette configuration par défaut.

Pour créer une stratégie de sauvegarde :

1. Après avoir téléchargé et inscrit l’agent MARS, ouvrez la console de l’agent. Vous pouvez le trouver en recherchant **Sauvegarde Microsoft Azure** sur votre ordinateur.  

1. En dessous de **Actions** sélectionnez **Planifier la sauvegarde**.

    ![Planifier une sauvegarde de Windows Server](./media/backup-configure-vault/schedule-first-backup.png)
1. Dans l’Assistant Planifier la sauvegarde, sélectionnez **Mise en route** > **Suivant**.
1. En dessous de **Sélectionner les éléments à sauvegarder**, sélectionnez **Ajouter des éléments**.

    ![Ajouter les éléments à sauvegarder](./media/backup-azure-manage-mars/select-item-to-backup.png)

1. Dans la zone **Sélectionner des éléments**, sélectionnez les éléments à sauvegarder, puis **OK**.

    ![Sélectionner les éléments à sauvegarder](./media/backup-azure-manage-mars/selected-items-to-backup.png)

1. Dans la page **Sélectionner les éléments à sauvegarder**, sélectionnez **Suivant**.
1. Dans la page **Spécifier une planification de la sauvegarde**, indiquez à quel moment effectuer les sauvegardes quotidiennes ou hebdomadaires. Sélectionnez ensuite **Suivant**.

    * Un point de récupération est créé lorsqu’une sauvegarde est effectuée.
    * Le nombre de points de récupération créés dans votre environnement dépend de votre planification de sauvegarde.
    * Vous pouvez planifier jusqu’à trois sauvegardes quotidiennes. Dans l’exemple suivant, deux sauvegardes quotidiennes se produisent : une à minuit et l’autre à 18h.

        ![Configurer une planification de sauvegarde quotidienne](./media/backup-configure-vault/day-schedule.png)

    * Vous pouvez également exécuter des sauvegardes hebdomadaires. Dans l’exemple suivant, les sauvegardes sont effectuées à tour de rôle le dimanche à 9h30 et le mercredi à 1h.

        ![Configurer une planification de sauvegarde hebdomadaire](./media/backup-configure-vault/week-schedule.png)

1. Dans la page **Sélectionner une stratégie de conservation**, spécifiez le mode de stockage des copies historiques de vos données. Sélectionnez ensuite **Suivant**.

    * Les paramètres de conservation précisent les points de récupération à stocker ainsi que leur durée de stockage.
    * Quand vous définissez une conservation quotidienne, vous indiquez qu’à l’heure spécifiée pour la conservation quotidienne, le dernier point de récupération sera conservé pendant le nombre de jours spécifié. Vous pouvez aussi spécifier une stratégie de conservation mensuelle pour indiquer que le point de récupération créé le 30 de chaque mois doit être stocké pendant 12 mois.
    * La conservation des points de récupération quotidiens et hebdomadaires coïncide généralement avec la planification de sauvegarde. Ainsi, quand la planification déclenche une sauvegarde, le point de récupération créé par cette dernière est stocké pendant la durée spécifiée dans la stratégie de conservation quotidienne ou hebdomadaire.
    * Dans l’exemple suivant :

        * Les sauvegardes quotidiennes effectuées à minuit et 18h sont conservées pendant sept jours.
        * Les sauvegardes effectuées le samedi à minuit et 18h sont conservées pendant quatre semaines.
        * Les sauvegardes effectuées le dernier samedi du mois à minuit et à 18h sont conservées pendant 12 mois.
        * Les sauvegardes effectuées le dernier samedi de mars sont conservées pendant 10 ans.

        ![Exemple de stratégie de conservation](./media/backup-configure-vault/retention-example.png)

1. Dans la page **Choisir un type de sauvegarde initiale**, indiquez si vous souhaitez effectuer la sauvegarde initiale sur le réseau ou utiliser la sauvegarde hors connexion. Pour effectuer la sauvegarde initiale sur le réseau, sélectionnez **Automatiquement sur le réseau** > **Suivant**.

    Pour plus d’informations sur la sauvegarde hors connexion, consultez [Utiliser Azure Data Box pour la sauvegarde hors connexion](offline-backup-azure-data-box.md).

    ![Choisir un type de sauvegarde initiale](./media/backup-azure-manage-mars/choose-initial-backup-type.png)

1. Dans la page **Confirmation**, passez en revue les informations, puis sélectionnez **Terminer**.

    ![Confirmer le type de sauvegarde](./media/backup-azure-manage-mars/confirm-backup-type.png)

1. Une fois que l’Assistant a créé la planification de sauvegarde, sélectionnez **Fermer**.

    ![Examiner la progression de la planification de sauvegarde](./media/backup-azure-manage-mars/confirm-modify-backup-process.png)

Créez une stratégie sur chaque ordinateur où l’agent est installé.

### <a name="do-the-initial-backup-offline"></a>Effectuer la sauvegarde initiale hors connexion

Vous pouvez exécuter une sauvegarde initiale automatiquement sur le réseau ou vous pouvez sauvegarder en mode hors connexion. L’amorçage hors connexion d’une sauvegarde initiale s’avère utile si vous avez de grandes quantités de données à transférer, ce qui nécessite beaucoup de bande passante réseau.

Pour effectuer un transfert en mode hors connexion :

1. Écrivez les données de sauvegarde dans un emplacement intermédiaire.
1. Utilisez l’outil AzureOfflineBackupDiskPrep pour copier les données de l’emplacement intermédiaire sur un ou plusieurs disques SATA.

    L’outil crée une tâche d’importation Azure. Pour plus d’informations, consultez [Qu’est-ce que le service Azure Import/Export](../storage/common/storage-import-export-service.md).
1. Envoyez les disques SATA à un centre de données Azure.

    Dans le centre de données, les données de disque sont copiées vers un compte de stockage Azure. Sauvegarde Azure copie les données de sauvegarde à partir du compte de stockage vers le coffre, et des sauvegardes incrémentielles sont planifiées.

Pour plus d'informations sur l’amorçage hors connexion, consultez [Utiliser Azure Data Box pour la sauvegarde hors connexion](offline-backup-azure-data-box.md).

### <a name="enable-network-throttling"></a>Activation de la limitation du réseau

Vous pouvez contrôler la façon dont l’agent MARS utilise la bande passante réseau en activant la limitation de bande passante. Cette limitation s’avère utile si vous avez besoin de sauvegarder des données pendant les heures de travail, mais que vous souhaitez contrôler la quantité de bande passante utilisée par l’activité de sauvegarde et de restauration.

La fonctionnalité de limitation de bande passante de Sauvegarde Azure utilise la [Qualité de service (QoS)](/windows-server/networking/technologies/qos/qos-policy-top) sur le système d’exploitation local.

La limitation de bande passante pour les sauvegardes est disponible sur Windows Server version 2012 et ultérieures et sur Windows version 8 et ultérieures. Les systèmes d’exploitation doivent exécuter les derniers Service Packs.

Pour activer la limitation de bande passante :

1. Dans l’agent MARS, sélectionnez **Modifier les propriétés**.
1. Sous l’onglet **Limitation**, sélectionnez la case **Activer la limitation de la bande passante sur Internet pour les opérations de sauvegarde**.

    ![Configurer la limitation de bande passante pour les opérations de sauvegarde](./media/backup-configure-vault/throttling-dialog.png)
1. Spécifiez la bande passante autorisée pendant et en dehors des heures de travail. Les valeurs de bande passante s'échelonnent de 512 kbit/s à 1 023 Mbit/s. Sélectionnez ensuite **OK**.

## <a name="run-an-on-demand-backup"></a>Exécuter une sauvegarde à la demande

1. Dans l’agent MARS, sélectionnez **Sauvegarder maintenant**.

    ![Sauvegarder maintenant dans Windows Server](./media/backup-configure-vault/backup-now.png)

1. Si la version de l’agent MARS utilisée est la version 2.0.9169.0 ou plus récente, vous pouvez définir une date de conservation personnalisée. Dans la section **Conserver la sauvegarde jusqu’au**, choisissez une date dans le calendrier.

   ![Utiliser le calendrier pour personnaliser une date de conservation](./media/backup-configure-vault/mars-ondemand.png)

1. Dans la page **Confirmation**, examinez les paramètres, puis sélectionnez **Sauvegarder**.
1. Sélectionnez **Fermer** pour fermer l’Assistant. Si vous fermez l’Assistant avant la fin de la sauvegarde, il continue de s’exécuter en arrière-plan.

Une fois la sauvegarde initiale terminée, l’état **Tâche terminée** apparaît dans la console de sauvegarde.

## <a name="set-up-on-demand-backup-policy-retention-behavior"></a>Configurer le comportement de conservation de la stratégie de sauvegarde à la demande

> [!NOTE]
> Ces informations s’appliquent uniquement aux version de l’agent MARS antérieures à la version 2.0.9169.0.
>

| Option de planification de sauvegarde | Durée de conservation des données
| -- | --
| jour | **Conservation par défaut** : correspond à la « conservation en jours pour les sauvegardes quotidiennes ». <br/><br/> **Exception** : en cas d’échec d’une sauvegarde planifiée quotidienne définie pour une conservation à long terme (semaines, mois ou années), une sauvegarde à la demande déclenchée juste après l’échec est prise en compte pour la conservation à long terme. Dans le cas contraire, la sauvegarde planifiée suivante est prise en compte pour la conservation à long terme.<br/><br/> **Exemple de scénario** : la sauvegarde planifiée du jeudi à 8h a échoué. Cette sauvegarde devait être prise en compte pour une conservation hebdomadaire, mensuelle ou annuelle. De ce fait, la première sauvegarde à la demande déclenchée avant la prochaine sauvegarde planifiée du vendredi à 8h est automatiquement marquée pour une conservation hebdomadaire, mensuelle ou annuelle. Cette sauvegarde remplace la sauvegarde du jeudi à 8h.
| Week | **Conservation par défaut** : un jour. Les sauvegardes à la demandes exécutées pour une source de données associée à une stratégie de sauvegarde hebdomadaire sont supprimées le jour suivant. Elles le sont même s’il s’agit des sauvegardes les plus récentes de la source de données. <br/><br/> **Exception** : en cas d’échec d’une sauvegarde planifiée hebdomadaire définie pour une conservation à long terme (semaines, mois ou années), une sauvegarde à la demande déclenchée juste après l’échec est prise en compte pour la conservation à long terme. Dans le cas contraire, la sauvegarde planifiée suivante est prise en compte pour la conservation à long terme. <br/><br/> **Exemple de scénario** : la sauvegarde planifiée du jeudi à 8h a échoué. Cette sauvegarde devait être prise en compte pour une conservation mensuelle ou annuelle. De ce fait, la première sauvegarde à la demande déclenchée avant la prochaine sauvegarde planifiée du jeudi à 8h est automatiquement marquée pour une conservation mensuelle ou annuelle. Cette sauvegarde remplace la sauvegarde du jeudi à 8h.

Pour plus d’informations, consultez [Créer une stratégie de sauvegarde](#create-a-backup-policy).

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [restaurer des fichiers dans Azure](backup-azure-restore-windows-server.md).
* Consultez les [questions courantes sur la sauvegarde des fichiers et des dossiers](backup-azure-file-folder-backup-faq.md).
