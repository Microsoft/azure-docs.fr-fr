---
title: Récupérer des données depuis un serveur de sauvegarde Azure
description: Récupérez les données que vous avez protégées dans un coffre Recovery Services depuis n’importe quel serveur de sauvegarde Azure inscrit auprès de ce coffre.
ms.topic: conceptual
ms.date: 07/09/2019
ms.openlocfilehash: 0cce45fbaaedad4cfcee0cda768aae2fa294312b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96002968"
---
# <a name="recover-data-from-azure-backup-server"></a>Récupérer des données depuis Azure Backup Server

Vous pouvez utiliser Azure Backup Server pour récupérer les données sauvegardées dans un coffre Recovery Services. Le processus pour cette opération est intégré dans la console de gestion Azure Backup Server et est similaire au flux de travail de récupération pour les autres composants Azure Backup.

> [!NOTE]
> Cet article est applicable à [System Center Data Protection Manager 2012 R2 avec UR7 ou ultérieur](https://support.microsoft.com/kb/3065246), combiné à l’agent [Sauvegarde Azure](https://aka.ms/azurebackup_agent) le plus récent.
>
>

Pour récupérer des données depuis un serveur de sauvegarde Azure :

1. Sous l’onglet **Récupération** de la console de gestion du serveur de sauvegarde Azure, sélectionnez **« Ajouter un DPM externe »** (dans la partie supérieure gauche de l’écran).

    ![Ajouter un serveur DPM externe](./media/backup-azure-alternate-dpm-server/add-external-dpm.png)
2. Téléchargez les nouvelles **informations d’identification du coffre** depuis le coffre associé au **serveur de sauvegarde Azure** où les données ont été récupérées, choisissez le serveur de sauvegarde Azure dans la liste des serveurs de sauvegarde Azure inscrits auprès du coffre Recovery Services et saisissez la **phrase secrète de chiffrement** associée au serveur pour lequel les données sont en cours de récupération.

    ![Informations d'identification d’un serveur DPM externe](./media/backup-azure-alternate-dpm-server/external-dpm-credentials.png)

   > [!NOTE]
   > Seuls les serveurs de sauvegarde Azure associés au même coffre d’inscription peuvent récupérer des données entre eux.
   >
   >

    Une fois le serveur de sauvegarde Azure externe ajouté, vous pouvez parcourir les données du serveur externe et du serveur de sauvegarde Azure local dans l’onglet **Récupération**.
3. Parcourez la liste des serveurs de production protégés par le serveur de sauvegarde Azure externe et sélectionnez la source de données appropriée.

    ![Parcourir un serveur DPM externe](./media/backup-azure-alternate-dpm-server/browse-external-dpm.png)
4. Sélectionnez **le mois et l’année** dans la liste déroulante **Points de récupération**. Sélectionnez la **date de récupération** souhaitée correspondant au moment de la création du point de récupération, puis sélectionnez **l’Heure de récupération**.

    Une liste de fichiers et de dossiers s'affiche dans le volet inférieur, qui peut être parcourue et récupérée sur n'importe quel emplacement.

    ![Points de récupération d’un serveur DPM externe](./media/backup-azure-alternate-dpm-server/external-dpm-recoverypoint.png)
5. Cliquez avec le bouton droit sur l’élément approprié, puis sélectionnez **Récupérer**.

    ![Récupération d’un serveur DPM externe](./media/backup-azure-alternate-dpm-server/recover.png)
6. Vérifiez la **sélection à récupérer**. Vérifiez les données et l’heure de la copie de sauvegarde à récupérer, ainsi que la source à partir de laquelle la copie de sauvegarde a été créée. Si la sélection est incorrecte, sélectionnez **Annuler** pour revenir à l’onglet Récupération et sélectionnez le point de récupération approprié. Si la sélection est correcte, sélectionnez **Suivant**.

    ![Résumé de la récupération d’un serveur DPM externe](./media/backup-azure-alternate-dpm-server/external-dpm-recovery-summary.png)
7. Sélectionnez **Récupérer à un autre emplacement**. **Rechercher** l'emplacement correct pour la récupération.

    ![Emplacement de remplacement de la récupération d’un serveur DPM externe](./media/backup-azure-alternate-dpm-server/external-dpm-recovery-alternate-location.png)
8. Choisissez entre les options **Créer une copie**, **Ignorer** et **Remplacer**.

   * **Créer une copie** : crée une copie du fichier en cas de conflit de noms.
   * **Ignorer** : s’il existe un conflit de noms, ne récupère pas le fichier, ce qui laisse le fichier d’origine en place.
   * **Remplacer** : s’il existe un conflit de noms, remplace la copie existante du fichier.

     Choisissez l'option appropriée pour **Restaurer la sécurité**. Vous pouvez appliquer les paramètres de sécurité de l'ordinateur de destination sur lequel les données sont récupérées ou les paramètres de sécurité qui étaient applicables au produit au moment de la création du point de récupération.

     Indiquez si une **Notification** est envoyée une fois la récupération terminée avec succès.

     ![Notifications de la récupération d’un serveur DPM externe](./media/backup-azure-alternate-dpm-server/external-dpm-recovery-notifications.png)
9. L’écran **Résumé** répertorie les options sélectionnées jusqu'à présent. Une fois que vous avez sélectionné **Récupérer**, les données sont récupérées à l’emplacement local approprié.

    ![Résumé des options de la récupération d’un serveur DPM externe](./media/backup-azure-alternate-dpm-server/external-dpm-recovery-options-summary.png)

   > [!NOTE]
   > Le travail de récupération peut être surveillé dans l’onglet **Analyse** du serveur de sauvegarde Azure.
   >
   >

    ![Surveillance de la récupération](./media/backup-azure-alternate-dpm-server/monitoring-recovery.png)
10. Vous pouvez sélectionner **Effacer le DPM externe** sous l’onglet **Récupération** du serveur DPM pour supprimer l’affichage du serveur DPM externe.

    ![Effacer un serveur DPM externe](./media/backup-azure-alternate-dpm-server/clear-external-dpm.png)

## <a name="troubleshooting-error-messages"></a>Dépannage des messages d'erreur

| Non. | Message d’erreur | Étapes de dépannage |
|:---:|:--- |:--- |
| 1. |Ce serveur n'est pas enregistré dans le coffre spécifié par les informations d'identification de coffre. |**Cause :** cette erreur apparaît lorsque le fichier des informations d’identification du coffre sélectionné n’appartient pas au coffre Recovery Services associé au serveur de sauvegarde Azure sur lequel la récupération est tentée. <br> **Résolution :** téléchargez le fichier des informations d’identification du coffre Recovery Services pour lequel le serveur de sauvegarde Azure est inscrit. |
| 2. |Les données récupérables ne sont pas disponibles ou le serveur sélectionné n'est pas un serveur DPM. |**Cause :** aucun autre serveur de sauvegarde Azure n’est inscrit auprès du coffre Recovery Services ou les serveurs n’ont pas encore chargé les métadonnées ou le serveur sélectionné n’est pas un serveur de sauvegarde Azure (utilisant Windows Server ou le Client Windows). <br> **Résolution :** s’il existe d’autres serveurs de sauvegarde Azure inscrits auprès du coffre Recovery Services, assurez-vous que le dernier agent Sauvegarde Azure est installé. <br>S’il existe d’autres serveurs Azure Backup inscrits auprès du coffre Recovery Services, patientez un jour après l’installation pour lancer le processus de récupération. Le travail nocturne chargera les métadonnées de toutes les sauvegardes protégées dans le cloud. Les données seront disponibles pour la récupération. |
| 3. |Aucun autre serveur DPM n'est inscrit auprès de ce coffre. |**Cause :** aucun autre serveur de sauvegarde Azure n’est enregistré dans le coffre à partir duquel la récupération est tentée.<br>**Résolution :** s’il existe d’autres serveurs de sauvegarde Azure inscrits auprès du coffre Recovery Services, assurez-vous que le dernier agent Sauvegarde Azure est installé.<br>S’il existe d’autres serveurs Azure Backup inscrits auprès du coffre Recovery Services, patientez un jour après l’installation pour lancer le processus de récupération. Le travail nocturne charge les métadonnées de toutes les sauvegardes protégées dans le cloud. Les données seront disponibles pour la récupération. |
| 4. |La phrase secrète de chiffrement fournie ne correspond pas à la phrase secrète associée au serveur suivant : **\<server name>** |**Cause :** la phrase secrète de chiffrement utilisée dans le processus de chiffrement des données à partir des données du serveur de sauvegarde Azure en cours de récupération ne correspond pas à la phrase secrète de chiffrement fournie. L’agent ne peut pas déchiffrer les données, de sorte que la récupération échoue.<br>**Résolution :** fournissez la phrase secrète de chiffrement associée au serveur de sauvegarde Azure pour lequel les données sont récupérées. |

## <a name="next-steps"></a>Étapes suivantes

Lire les autres forums aux questions :

* [Questions courantes](backup-azure-vm-backup-faq.md) sur la sauvegarde des machines virtuelles Azure
* [Questions courantes](backup-azure-file-folder-backup-faq.md) sur l’agent Sauvegarde Azure.
