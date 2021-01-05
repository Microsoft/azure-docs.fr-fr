---
title: Gérer et surveiller les sauvegardes de l’agent MARS
description: Découvrez comment gérer et surveiller les sauvegardes de l’agent Microsoft Azure Recovery Services (MARS) à l’aide du service Sauvegarde Azure.
ms.reviewer: srinathv
ms.topic: conceptual
ms.date: 10/07/2019
ms.openlocfilehash: 25f0c41b535f9403d0a7027687cc5261cd437275
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368594"
---
# <a name="manage-microsoft-azure-recovery-services-mars-agent-backups-by-using-the-azure-backup-service"></a>Gérer les sauvegardes de l’agent Microsoft Azure Recovery Services (MARS) à l’aide du service Sauvegarde Azure

Cet article explique comment gérer les fichiers et dossiers qui sont sauvegardés avec l’agent Microsoft Azure Recovery Services.

## <a name="modify-a-backup-policy"></a>Modifier une stratégie de sauvegarde

Lorsque vous modifiez une stratégie de sauvegarde, vous pouvez ajouter de nouveaux éléments, supprimer des éléments existants de la sauvegarde ou exclure des fichiers de la sauvegarde à l’aide des paramètres d’exclusion.

- **Ajouter des éléments** : utilisez cette option uniquement pour ajouter de nouveaux éléments à sauvegarder. Pour supprimer des éléments existants, utilisez l’option **Supprimer des éléments** ou **Paramètres d’exclusion**.  
- **Supprimer des éléments** : utilisez cette option pour supprimer des éléments de la sauvegarde.
  - Utilisez **Paramètres d’exclusion** pour supprimer tous les éléments d’un volume au lieu de **Supprimer des éléments**.
  - L’effacement de toutes les sélections d’un volume entraîne la conservation des anciennes sauvegardes des éléments conformément aux paramètres de rétention au moment de la dernière sauvegarde, sans possibilité de modification.
  - Si vous resélectionnez ces éléments, une première sauvegarde complète est effectuée, et les modifications apportées à la nouvelle stratégie ne sont pas appliquées aux anciennes sauvegardes.
  - La désélection d’un volume entier conserve la sauvegarde précédente sans possibilité de modifier la stratégie de conservation.
- **Paramètres d’exclusion** : utilisez cette option pour exclure des éléments spécifiques de la sauvegarde.

### <a name="add-new-items-to-existing-policy"></a>Ajouter de nouveaux éléments à une stratégie existante

1. Dans **Actions**, sélectionnez **Planifier la sauvegarde**.

    ![Planifier une sauvegarde de Windows Server](./media/backup-configure-vault/schedule-first-backup.png)

2. Dans l’onglet **Sélectionner un élément de stratégie**, sélectionnez **Modifier la planification de sauvegarde pour vos fichiers et dossiers**, puis **Suivant**.

    ![Sélectionner des éléments de stratégie](./media/backup-azure-manage-mars/select-policy-items.png)

3. Dans l’onglet **Modifier ou arrêter la planification de sauvegarde**, sélectionnez **Apporter des modifications aux éléments ou aux heures de la sauvegarde**, puis **Suivant**.

    ![Modifier ou planifier une sauvegarde](./media/backup-azure-manage-mars/modify-schedule-backup.png)

4. Dans l’onglet **Sélectionner les éléments à sauvegarder**, sélectionnez **Ajouter des éléments** pour ajouter les éléments à sauvegarder.

    ![Modifier ou planifier une sauvegarde, Ajouter des éléments](./media/backup-azure-manage-mars/modify-schedule-backup-add-items.png)

5. Dans la fenêtre **Sélectionner les éléments**, sélectionnez les fichiers ou dossiers à ajouter, puis **OK**.

    ![Sélectionner les éléments](./media/backup-azure-manage-mars/select-item.png)

6. Effectuez les étapes suivantes, puis sélectionnez **Terminer** pour terminer l’opération.

### <a name="add-exclusion-rules-to-existing-policy"></a>Ajouter des règles d’exclusion à une stratégie existante

Vous pouvez ajouter des règles d’exclusion pour ignorer les fichiers et les dossiers que vous ne souhaitez pas sauvegarder. Vous pouvez faire cela pendant la définition d’une nouvelle stratégie ou la modification d’une stratégie existante.

1. Dans le volet Actions, sélectionnez **Planifier la sauvegarde**. Accédez à **Sélectionner les éléments à sauvegarder** et sélectionnez **Paramètres d’exclusion**.

    ![Paramètres d’exclusion](./media/backup-azure-manage-mars/select-exclusion-settings.png)

2. Dans **Paramètres d’exclusion**, sélectionnez **Ajouter une exclusion**.

    ![Ajouter une exclusion](./media/backup-azure-manage-mars/add-exclusion.png)

3. Dans **Sélectionner les éléments à exclure**, parcourez les fichiers et les dossiers et sélectionnez les éléments à exclure, puis sélectionnez **OK**.

    ![Sélectionner les éléments à exclure](./media/backup-azure-manage-mars/select-items-exclude.png)

4. Par défaut, tous les **sous-dossiers** dans les dossiers sélectionnés sont exclus. Vous pouvez modifier cela en sélectionnant **Oui** ou **Non**. Vous pouvez modifier et spécifier les types de fichiers à exclure, comme indiqué ci-dessous :

    ![Sélectionner les types de sous-dossiers](./media/backup-azure-manage-mars/subfolders-type.png)

5. Effectuez les étapes suivantes, puis sélectionnez **Terminer** pour terminer l’opération.

### <a name="remove-items-from-existing-policy"></a>Supprimer les éléments d’une stratégie existante

1. Dans le volet Actions, sélectionnez **Planifier la sauvegarde**. Accédez à **Sélectionner les éléments à sauvegarder**. Dans la liste, sélectionnez les fichiers et les dossiers à supprimer de la planification de sauvegarde, puis sélectionnez **Supprimer les éléments**.

    ![Sélectionner les éléments à supprimer](./media/backup-azure-manage-mars/select-items-remove.png)

    > [!NOTE]
    > Prenez garde lorsque vous supprimez complètement un volume de la stratégie.  Si vous devez l’ajouter à nouveau, il sera traité comme un nouveau volume. La prochaine sauvegarde planifiée effectuera une sauvegarde initiale (sauvegarde complète) au lieu d’une sauvegarde incrémentielle. Si vous avez besoin de supprimer temporairement et d’ajouter ultérieurement des éléments, nous vous recommandons d’utiliser **Paramètres d’exclusions** au lieu de **Supprimer des éléments** pour garantir l’exécution une sauvegarde incrémentielle au lieu d’une sauvegarde complète.

2. Effectuez les étapes suivantes, puis sélectionnez **Terminer** pour terminer l’opération.

## <a name="stop-protecting-files-and-folder-backup"></a>Arrêter la protection de la sauvegarde de fichiers et de dossiers

Il existe deux façons d’arrêter la protection de la sauvegarde de fichiers et de dossiers :

- **Arrêter la protection et conserver les données de sauvegarde**.
  - Cette option arrête la protection sur toutes les futures tâches de sauvegarde.
  - Le service Sauvegarde Azure conservera tous les points de récupération existants.  
  - Vous pourrez restaurer les données sauvegardées pour les points de récupération non expirés.
  - Si vous décidez de reprendre la protection, vous pouvez utiliser l’option *Réactiver la planification de sauvegarde*. Après cela, les données sont conservées en fonction de la nouvelle stratégie de rétention.
- **Arrêter la protection et supprimer les données de sauvegarde**.
  - Cette option empêche toutes les futures tâches de sauvegarde de protéger vos données et supprime tous les points de récupération.
  - Vous recevrez un e-mail d’alerte de suppression des données de sauvegarde comportant le message *Vos données relatives à cet élément de sauvegarde ont été supprimées. Ces données seront temporairement disponibles pendant 14 jours, après quoi elles seront définitivement supprimées* et l’action recommandée *Protégez à nouveau l’élément de sauvegarde dans un délai de 14 jours pour récupérer vos données.*
  - Pour reprendre la protection, réactivez la protection dans un délai de 14 jours après l’opération de suppression.

### <a name="stop-protection-and-retain-backup-data"></a>Arrêter la protection et conserver les données de sauvegarde

1. Ouvrez la console de gestion de MARS, accédez au **volet Actions** et **sélectionnez Planifier la sauvegarde**.

    ![Sélectionner la planification de la sauvegarde](./media/backup-azure-manage-mars/mars-actions.png)
1. Sur la page **Sélectionner un élément de stratégie**, sélectionnez **Modifier la planification de sauvegarde des fichiers et dossiers**, puis **Suivant**.

    ![Sélectionner l’élément de stratégie](./media/backup-azure-manage-mars/select-policy-item-retain-data.png)
1. Dans la page **Modifier ou arrêter une sauvegarde planifiée**, sélectionnez **Ne plus utiliser cette panification de sauvegarde, mais conserver les sauvegardes stockées jusqu’à la réactivation de la planification**. Ensuite, sélectionnez **Suivant**.

    ![Arrêtez une sauvegarde planifiée.](./media/backup-azure-manage-mars/stop-schedule-backup.png)
1. Dans **Suspendre la sauvegarde planifiée**, vérifiez les informations, puis sélectionnez **Terminer**.

    ![Suspendre la sauvegarde planifiée.](./media/backup-azure-manage-mars/pause-schedule-backup.png)
1. Dans **Modifier la progression de la sauvegarde**, vérifiez que l’état de la suspension de la sauvegarde planifiée est Réussite, puis sélectionnez **Fermer** pour terminer.

### <a name="stop-protection-and-delete-backup-data"></a>Arrêter la protection et supprimer les données de sauvegarde

1. Ouvrez la console de gestion de MARS, accédez au volet **Actions** et sélectionnez **Planifier la sauvegarde**.
2. Dans la page **Modifier ou arrêter une sauvegarde planifiée**, sélectionnez **Ne plus utiliser cette panification de sauvegarde et supprimer toutes les sauvegardes stockées**. Ensuite, sélectionnez **Suivant**.

    ![Modifiez ou arrêtez une sauvegarde planifiée.](./media/backup-azure-delete-vault/modify-schedule-backup.png)

3. Dans la page **Arrêter une sauvegarde planifiée**, sélectionnez **Terminer**.

    ![Arrêter une sauvegarde planifiée et sélectionner terminer](./media/backup-azure-delete-vault/stop-schedule-backup.png)
4. Il vous est demandé d’entrer un code PIN (Personal Identification Number) de sécurité, que vous devez générer manuellement. Pour cela, commencez par vous connecter au portail Azure.
5. Accédez à **Coffre Recovery Services** > **Paramètres** > **Propriétés**.
6. Sous **Code PIN de sécurité**, sélectionnez **Générer**. Copiez ce code PIN. Il n’est valide que pendant cinq minutes.
7. Dans la console de gestion, collez le code PIN, puis sélectionnez **OK**.

    ![Générez un code PIN de sécurité.](./media/backup-azure-delete-vault/security-pin.png)

8. Dans la page **Modifier la progression de la sauvegarde**, le message suivant s’affiche : *Les données supprimées sont conservées pendant 14 jours. Passé ce délai, les données de sauvegarde sont définitivement supprimées.*  

    ![Modifier la progression de la sauvegarde](./media/backup-azure-delete-vault/deleted-backup-data.png)

Une fois les éléments de sauvegarde locaux supprimés, effectuez les étapes suivantes à partir du portail.

## <a name="re-enable-protection"></a>Réactiver la protection

Si vous avez arrêté la protection tout en conservant les données et avez décidé de reprendre la protection, vous pouvez réactiver la planification de sauvegarde en modifiant la stratégie de sauvegarde.

1. Dans **Actions** sélectionnez **Planifier la sauvegarde**.
1. Sélectionnez **Réactiver la planification de sauvegarde. Vous pouvez également modifier les éléments de sauvegarde ou les heures**, puis sélectionner **Suivant**.<br>

    ![Réactiver la planification de sauvegarde](./media/backup-azure-manage-mars/re-enable-policy-next.png)
1. Dans **Sélectionner les éléments à sauvegarder**, sélectionnez **Suivant**.

    ![Sélectionner les éléments à sauvegarder](./media/backup-azure-manage-mars/re-enable-next.png)
1. Dans **Spécifier la planification de sauvegarde**, spécifiez la planification de sauvegarde, puis sélectionnez **Suivant**.
1. Dans **Sélectionner une stratégie de conservation**, spécifiez la durée de conservation, puis sélectionnez **Suivant**.
1. Enfin, dans l’écran **Confirmation**, vérifiez les détails de la stratégie, puis sélectionnez **Terminer**.

## <a name="re-generate-passphrase"></a>Régénérer la phrase secrète

Une phrase secrète est utilisée pour chiffrer et déchiffrer les données lors de la sauvegarde ou de la restauration de votre ordinateur local à l’aide de l’agent MARS sur ou à partir d’Azure. Si vous avez perdu ou oublié la phrase secrète, vous pouvez la régénérer (à condition que votre machine soit toujours inscrite auprès du coffre Recovery Services et que la sauvegarde soit configurée) en procédant comme suit :

1. Dans la console de l’agent MARS, accédez à **volet Actions** > **Modifier les propriétés** >. Accédez ensuite à l’**onglet Chiffrement**.<br>
1. Cochez la case **Modifier la phrase secrète**.<br>
1. Entrez une nouvelle phrase secrète ou sélectionnez **Générer une phrase secrète**.
1. Sélectionnez **Parcourir** pour enregistrer la nouvelle phrase secrète.

    ![Générez une phrase secrète.](./media/backup-azure-manage-mars/passphrase.png)

1. Sélectionnez **OK** pour appliquer les modifications.  Si la [Fonctionnalité de sécurité](./backup-azure-security-feature.md#enable-security-features) est activée sur le Portail Azure pour le coffre Recovery Services, il vous est demandé d’entrer le code PIN de sécurité. Pour recevoir le code PIN, effectuez les étapes décrites dans cet [article](./backup-azure-security-feature.md#authentication-to-perform-critical-operations).<br>
1. Collez le code PIN de sécurité à partir du portail, puis sélectionnez **OK** pour appliquer les modifications.<br>

    ![Coller le code PIN de sécurité](./media/backup-azure-manage-mars/passphrase2.png)
1. Assurez-vous que la phrase secrète est enregistrée en toute sécurité à un autre emplacement (autre que la machine source), de préférence dans Azure Key Vault. Suivez toutes les phrases secrètes si vous avez plusieurs machines sauvegardées avec les agents MARS.

## <a name="managing-backup-data-for-unavailable-machines"></a>Gestion des données de sauvegarde pour les machines non disponibles

Cette section décrit un scénario dans lequel votre machine source qui a été protégée avec MARS n’est plus disponible, car elle a été supprimée, endommagée, infectée par un programme malveillant/ransomware ou a été mise hors service.

Pour ces machines, le service Sauvegarde Azure vérifie que le point de récupération le plus récent n’expire pas (c’est-à-dire qu’il n’est pas nettoyé) en fonction des règles de conservation spécifiées dans la stratégie de sauvegarde. Ainsi, vous pouvez restaurer la machine de manière sécurisée.  Envisagez les scénarios suivants que vous pouvez effectuer sur les données sauvegardées :

### <a name="scenario-1-the-source-machine-is-unavailable-and-you-no-longer-need-to-retain-backup-data"></a>Scénario 1 : La machine source n’est pas disponible et vous n’avez plus besoin de conserver les données de sauvegarde

- Vous pouvez supprimer les données sauvegardées du portail Azure à l’aide de la procédure décrite dans [cet article](backup-azure-delete-vault.md#delete-protected-items-on-premises).

### <a name="scenario-2-the-source-machine-is-unavailable-and-you-need-to-retain-backup-data"></a>Scénario 2 : La machine source n’est pas disponible et vous devez conserver les données de sauvegarde

La gestion de la stratégie de sauvegarde pour MARS s’effectue par le biais de la console MARS et non du portail. Si vous devez étendre les paramètres de conservation pour les points de récupération existants avant qu’ils n’expirent, vous devez restaurer la machine, installer la console MARS et étendre la stratégie.

- Pour restaurer la machine, effectuez les étapes suivantes :
  1. [Restaurer la machine virtuelle sur une autre machine cible](backup-azure-restore-windows-server.md#use-instant-restore-to-restore-data-to-an-alternate-machine)
  1. Recréer la machine cible avec le même nom d’hôte que la machine source
  1. Installer l’agent et refaire l’inscription dans le même coffre et avec la même phrase secrète
  1. Lancer le client MARS pour prolonger la durée de conservation en fonction de vos besoins
- Votre machine nouvellement restaurée, protégée par MARS, continuera à effectuer des sauvegardes.  

## <a name="configuring-antivirus-for-the-mars-agent"></a>Configuration de l’antivirus pour l’agent MARS

Nous vous recommandons de configurer votre logiciel antivirus comme suit pour éviter les conflits avec les opérations de l’agent MARS.

1. **Ajouter des exclusions de chemin** : Pour éviter une dégradation des performances et des conflits éventuels, excluez les chemins suivants de la supervision en temps réel assurée par le logiciel antivirus :
    1. `%ProgramFiles%\Microsoft Azure Recovery Services Agent` et les sous-dossiers
    1. **Dossier temporaire** : Si le dossier temporaire n’est pas à l’emplacement normal, ajoutez-le également aux exclusions.  [Découvrez ici les étapes](backup-azure-file-folder-backup-faq.md#how-to-check-if-scratch-folder-is-valid-and-accessible) pour déterminer l’emplacement du dossier temporaire.
1. **Ajouter des exclusions binaires** : Pour éviter la dégradation des activités de sauvegarde et de console, excluez les processus de la supervision en temps réel du logiciel antivirus pour les binaires suivants :
    1. `%ProgramFiles%\Microsoft Azure Recovery Services Agent\bin\cbengine.exe`

>[!NOTE]
>Bien que l’exclusion de ces chemins soit suffisante pour la plupart des logiciels antivirus, certains peuvent continuer d’interférer avec les opérations de l’agent MARS. Si vous constatez des défaillances inattendues, désinstallez temporairement le logiciel antivirus et vérifiez si le problème disparaît. Si cela résout le problème, contactez l’éditeur de votre logiciel antivirus pour savoir comment configurer convenablement son produit.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les scénarios pris en charge et les limitations, reportez-vous à la [Matrice de prise en charge pour l’agent MARS](./backup-support-matrix-mars-agent.md).
- En savoir plus sur le [comportement de conservation de la stratégie de sauvegarde à la demande](backup-windows-with-mars-agent.md#set-up-on-demand-backup-policy-retention-behavior).
- Pour connaître les réponses à d’autres questions courantes, consultez la [FAQ sur l’agent MARS](backup-azure-file-folder-backup-faq.md).
