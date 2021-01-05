---
title: Amorcer la sauvegarde hors connexion avec le service Azure Import/Export
description: Découvrez comment utiliser Sauvegarde Azure pour envoyer des données en dehors du réseau à l’aide du service Azure Import/Export. Cet article décrit l’amorçage hors connexion des données de sauvegarde initiales à l’aide du service Azure Import/Export.
ms.reviewer: saurse
ms.topic: conceptual
ms.date: 05/17/2018
ms.openlocfilehash: f3cf44a34babab79d135923db040630a1c8e3dfe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88892012"
---
# <a name="offline-backup-workflow-in-azure-backup"></a>Workflow de la sauvegarde hors connexion dans Sauvegarde Azure

La sauvegarde Azure offre plusieurs fonctionnalités intégrées pour réduire les coûts de stockage et de réseau pendant les sauvegardes complètes initiales des données dans Azure. Les sauvegardes complètes initiales transfèrent généralement de grandes quantités de données et requièrent davantage de bande passante, en comparaison avec les sauvegardes suivantes qui transfèrent uniquement les données deltas/incrémentielles. Via le processus d’amorçage hors connexion, la sauvegarde Azure peut utiliser des disques pour charger les données de sauvegarde hors connexion dans Azure.

Le processus d’amorçage hors connexion de Sauvegarde Azure est étroitement intégré au [service Azure Import/Export](../storage/common/storage-import-export-service.md). Vous pouvez utiliser ce service pour transférer des données de sauvegarde initiales vers Azure à l’aide de disques. Si vous devez transférer des téraoctets (To) de données de sauvegarde initiales sur un réseau à latence élevée et à faible bande passante, vous pouvez utiliser le workflow d’amorçage hors connexion pour expédier la copie de sauvegarde initiale sur un ou plusieurs disques durs à un centre de données Azure. L’image suivante présente les étapes du flux de travail.

  ![Présentation du processus de workflow d’importation hors connexion](./media/backup-azure-backup-import-export/offlinebackupworkflowoverview.png)

Le processus de sauvegarde hors connexion inclut les étapes suivantes :

1. Au lieu d’envoyer les données de sauvegarde sur le réseau, écrivez les données de sauvegarde dans un emplacement intermédiaire.
1. À l’aide de l’utilitaire *AzureOfflineBackupDiskPrep*, écrivez les données figurant dans l’emplacement intermédiaire sur un ou plusieurs disques SATA.
1. Dans le cadre de l’étape de préparation, l’utilitaire *AzureOfflineBackupDiskPrep* crée une tâche d’importation Azure. Envoyez les disques durs SATA au centre de données Azure le plus proche et référencez la tâche d’importation pour connecter les activités.
1. Dans le centre de données Azure, les données stockées sur les disques sont copiées vers un compte de stockage Azure.
1. La sauvegarde Azure copie les données de sauvegarde à partir du compte de stockage vers le coffre Recovery Services et des sauvegardes incrémentielles sont planifiées.

## <a name="supported-configurations"></a>Configurations prises en charge

Les charges de travail ou fonctionnalités Sauvegarde Azure suivantes prennent en charge l’utilisation de la sauvegarde hors connexion pour :

> [!div class="checklist"]
>
> * Sauvegarde de fichiers et dossiers avec l’agent Microsoft Azure Recovery Services (MARS), également appelé agent de sauvegarde Azure.
> * Sauvegarde de l’ensemble des charges de travail et fichiers avec Microsoft System Center Data Protection Manager (DPM).
> * Sauvegarde de l’ensemble des charges de travail et fichiers avec le serveur de sauvegarde Microsoft Azure.

   > [!NOTE]
   > La sauvegarde hors connexion n’est pas prise en charge pour les sauvegardes d’état du système effectuées à l’aide de l’agent de sauvegarde Azure.

[!INCLUDE [backup-upgrade-mars-agent.md](../../includes/backup-upgrade-mars-agent.md)]

## <a name="prerequisites"></a>Prérequis

  > [!NOTE]
  > Les prérequis et le workflow suivants s’appliquent uniquement à une sauvegarde hors connexion des fichiers et dossiers à l’aide de [la dernière version de l’agent Microsoft Azure Recovery Services](https://aka.ms/azurebackup_agent). Pour effectuer des sauvegardes hors connexion de charges de travail à l’aide de System Center DPM ou du serveur de sauvegarde Azure, consultez [Workflow de sauvegarde hors connexion pour DPM et le serveur de sauvegarde Azure](backup-azure-backup-server-import-export.md).

Avant de démarrer le workflow de sauvegarde hors connexion, assurez-vous que les prérequis suivants sont respectés :

* Créez un [coffre Recovery Services](backup-azure-recovery-services-vault-overview.md). Pour créer un coffre, suivez les étapes décrites dans [Créer un coffre Recovery Services](tutorial-backup-windows-server-to-azure.md#create-a-recovery-services-vault).
* Vérifiez que seule la [dernière version de l’agent de sauvegarde Azure](https://aka.ms/azurebackup_agent) a été installée sur le client Windows Server ou Windows, le cas échéant, et que l’ordinateur est inscrit auprès du coffre Recovery Services.
* Azure PowerShell version 3.7.0 doit être installé sur l’ordinateur qui exécute l’agent de sauvegarde Azure. Téléchargez et [installez la version 3.7.0 d’Azure PowerShell](https://github.com/Azure/azure-powershell/releases/tag/v3.7.0-March2017).
* Sur l’ordinateur exécutant l’agent de sauvegarde Azure, assurez-vous que Microsoft Edge ou Internet Explorer 11 est installé et que JavaScript est activé.
* Créez un compte de stockage Azure dans le même abonnement que le coffre Recovery Services.
* Assurez-vous de disposer des [autorisations nécessaires](../active-directory/develop/howto-create-service-principal-portal.md) pour créer l’application Azure Active Directory. Le workflow de sauvegarde hors connexion crée une application Azure Active Directory dans l’abonnement associé au compte de stockage Azure. L’objectif de l’application est de fournir à Sauvegarde Azure un accès sécurisé et délimité au service Azure Import/Export, qui est requis pour le workflow de sauvegarde hors connexion.
* Inscrivez le fournisseur de ressources *Microsoft.ImportExport* dans l’abonnement contenant le compte de stockage Azure. Pour inscrire le fournisseur de ressources :
    1. Dans le menu principal, sélectionnez **Abonnements**.
    1. Si vous disposez de plusieurs abonnements, sélectionnez l’abonnement que vous prévoyez d’utiliser pour la sauvegarde hors connexion. Si vous n’utilisez qu’un seul abonnement, celui-ci s’affiche.
    1. Dans le menu de l’abonnement, sélectionnez **Fournisseurs de ressources** pour afficher la liste des fournisseurs.
    1. Faites défiler la liste des fournisseurs jusqu’à *Microsoft.ImportExport*. Si l’**état** est **NotRegistered**, sélectionnez **Inscrire**.

        ![Inscrire le fournisseur de ressources](./media/backup-azure-backup-import-export/registerimportexport.png)

* Un emplacement intermédiaire est créé. Il peut s’agir d’un partage réseau ou de tout lecteur supplémentaire, interne ou externe, sur l’ordinateur offrant suffisamment d’espace disque pour conserver votre copie initiale. Par exemple, si vous souhaitez sauvegarder un serveur de fichiers de 500 Go, assurez-vous que la zone intermédiaire dispose d’au moins 500 Go. (bien qu’une quantité inférieure soit utilisée en raison de la compression).
* Lorsque vous envoyez des disques à Azure, utilisez uniquement des disques SSD de 2,5 pouces ou des disques durs internes SATA II/III de 2,5 ou 3,5 pouces. La capacité maximale par disque dur est de 10 To. Consultez la [documentation sur le service Azure Import/Export](../storage/common/storage-import-export-requirements.md#supported-hardware) pour connaître la dernière série de disques pris en charge par le service.
* Les disques SATA doivent être connectés à un ordinateur (appelé *ordinateur de copie*) à partir duquel est effectuée la copie des données de sauvegarde de l’emplacement intermédiaire vers les disques SATA. Vérifiez que BitLocker est activé sur l’ordinateur de copie.

## <a name="workflow"></a>Workflow

Cette section décrit le workflow de sauvegarde hors connexion qui permet à vos données d’être transmises à un centre de données Azure et chargées dans Stockage Azure. Si vous avez des questions sur le service d’importation ou sur tout autre aspect du processus, consultez la [documentation de présentation du service Azure Import/Export](../storage/common/storage-import-export-service.md).

## <a name="initiate-offline-backup"></a>Lancer la sauvegarde hors connexion

1. Lorsque vous planifiez une sauvegarde sur l’agent Recovery Services, cette page s’affiche.

    ![Page Importer](./media/backup-azure-backup-import-export/offlinebackup_inputs.png)

1. Sélectionnez l’option **Transfert à l’aide de mes propres disques**.

    > [!NOTE]
    > Utilisez l’option Azure Data Box pour transférer des données de sauvegarde initiales hors connexion. Cette option permet d’économiser les efforts nécessaires pour fournir vos propres disques compatibles avec Azure. Le service fournit des appareils Azure Data Box sécurisés, infalsifiables et détenus par Microsoft sur lesquels les données de sauvegarde peuvent être écrites directement par l’agent Recovery Services.

1. Sélectionnez **Suivant** et renseignez les champs avec soin.

    ![Entrer les détails de votre disque](./media/backup-azure-backup-import-export/your-disk-details.png)

   Les champs que vous remplissez sont les suivants :

    * **Emplacement intermédiaire** : Emplacement de stockage temporaire dans lequel la copie de sauvegarde initiale est écrite. L’emplacement intermédiaire peut être sur un partage réseau ou un ordinateur local. Si l’ordinateur de copie et l’ordinateur source sont différents, spécifiez le chemin d’accès réseau complet de l’emplacement intermédiaire.
    * **Compte de stockage Azure Resource Manager** : Nom du compte de stockage de type Resource Manager (usage général v1 ou usage général v2) dans un abonnement Azure quelconque.
    * **Conteneur de stockage Azure** : Nom du conteneur de stockage d’objets blob de destination dans le compte de stockage Azure où les données de sauvegarde sont importées avant d’être copiées dans le coffre Recovery Services.
    * **ID d’abonnement Azure** : ID de l’abonnement Azure dans lequel le compte de stockage Azure est créé.
    * **Nom de la tâche d’importation Azure** : Nom unique utilisé par le service Azure Import/Export et par Sauvegarde Azure pour effectuer le suivi du transfert des données envoyées dans Azure à l’aide de disques.
  
   Après avoir renseigné les champs, sélectionnez **Suivant**. Enregistrez les informations **Emplacement intermédiaire** et **Nom de tâche d’importation Azure**. Elles sont nécessaires pour préparer les disques.

1. Lorsque vous y êtes invité, connectez-vous à votre abonnement Azure. Vous devez vous connecter pour que Sauvegarde Azure puisse créer l’application Azure Active Directory. Entrez les autorisations nécessaires pour accéder au service Azure Import/Export.

    ![Page de connexion à l’abonnement Azure](./media/backup-azure-backup-import-export/azure-login.png)

1. Terminez le workflow. Sur la console de l’agent de sauvegarde Azure, sélectionnez **Sauvegarder maintenant**.

    ![Sauvegarder maintenant](./media/backup-azure-backup-import-export/backupnow.png)

1. Dans la page **Confirmation** de l’Assistant, sélectionnez **Sauvegarder**. La sauvegarde initiale est écrite dans la zone intermédiaire tel que cela a été configuré.

   ![Confirmez que vous êtes prêt à sauvegarder maintenant.](./media/backup-azure-backup-import-export/backupnow-confirmation.png)

    Une fois l’opération terminée, l’emplacement intermédiaire est prêt à être utilisé pour la préparation des disques.

   ![Page Sauvegarder maintenant de l’Assistant](./media/backup-azure-backup-import-export/opbackupnow.png)

## <a name="prepare-sata-drives-and-ship-to-azure"></a>Préparer des disques SATA et envoyer à Azure

L’utilitaire *AzureOfflineBackupDiskPrep* prépare les disques SATA qui sont envoyés au centre de données Azure le plus proche. Cet utilitaire est disponible dans le répertoire d’installation de l’agent de sauvegarde Azure, dont le chemin d’accès est le suivant :

```*\Microsoft Azure Recovery Services Agent\Utils\\*```

1. Accédez au répertoire, puis copiez le répertoire *AzureOfflineBackupDiskPrep* vers un autre ordinateur auquel les disques SATA sont connectés. Sur l’ordinateur auquel les disques SATA sont connectés, vérifiez les points suivants :

   * L’ordinateur de copie peut accéder à l’emplacement intermédiaire spécifié pour le workflow d’amorçage hors connexion via le même chemin d’accès réseau que celui fourni dans le workflow de la section « Lancer la sauvegarde hors connexion ».
   * BitLocker est activé sur l’ordinateur de copie.
   * Azure PowerShell 3.7.0 est installé.
   * Les navigateurs compatibles les plus récents (Microsoft Edge ou Internet Explorer 11) sont installés et JavaScript est activé.
   * L’ordinateur de copie peut accéder au portail Azure. Si nécessaire, l’ordinateur de copie peut être le même que l’ordinateur source.

     > [!IMPORTANT]
     > Si l’ordinateur source est une machine virtuelle, vous devez utiliser un serveur physique ou un ordinateur client distinct comme ordinateur de copie.

1. Ouvrez une invite de commande avec élévation de privilèges sur l’ordinateur de copie en utilisant le répertoire de l’utilitaire *AzureOfflineBackupDiskPrep* comme répertoire actif. Exécutez la commande suivante :

    ```.\AzureOfflineBackupDiskPrep.exe s:<Staging Location Path>```

    | Paramètre | Description |
    | --- | --- |
    | s:&lt;*Staging Location Path*&gt; |Cette entrée obligatoire est utilisée pour fournir le chemin d’accès vers l’emplacement intermédiaire que vous avez entré dans le workflow de la section « Lancer la sauvegarde hors connexion ». |
    | p:&lt;*Path to PublishSettingsFile*&gt; |Cette entrée facultative est utilisée pour fournir le chemin du fichier de paramètres de publication Azure.  |

    Quand vous exécutez la commande, l’utilitaire vous demande de sélectionner la tâche d’importation Azure correspondant aux disques à préparer. S’il existe une seule tâche d’importation associée à l’emplacement intermédiaire spécifié, une page similaire à celle-ci s’affiche.

    ![Entrée de l’outil de préparation des disques Azure](./media/backup-azure-backup-import-export/diskprepconsole0_1.png) <br/>

1. Entrez la lettre du disque, sans les deux-points de fin, correspondant au disque monté que vous souhaitez préparer pour le transfert vers Azure.
1. Confirmez le formatage du disque lorsque vous y êtes invité.
1. Vous êtes invité à vous connecter à votre abonnement Azure. Entrez vos informations d’identification.

    ![Connexion à l’abonnement Azure](./media/backup-azure-backup-import-export/signindiskprep.png) <br/>

    L’outil commence ensuite à préparer le disque et à copier les données de sauvegarde. Vous devrez peut-être ajouter des disques supplémentaires lorsque l’outil vous y invite, si le disque fourni n’a pas suffisamment d’espace pour les données de sauvegarde. <br/>

    Une fois que l’outil a été correctement exécuté, l’invite de commandes fournit trois informations :

   1. Le ou les disques que vous avez fournis sont préparés en vue de leur envoi dans un centre de données Azure.
   1. Vous recevez la confirmation que votre tâche d’importation a été créée. La tâche d’importation utilise le nom que vous avez fourni.
   1. L’outil affiche l’adresse d’expédition correspondant au centre de données Azure.

      ![Préparation des disques Azure terminée](./media/backup-azure-backup-import-export/console2.png)<br/>

1. Une fois la commande exécutée, vous pouvez mettre à jour les informations d’expédition.

1. Expédiez les disques à l’adresse fournie par l’outil. Conservez le numéro de suivi pour référence ultérieure.

   > [!IMPORTANT]
   > Chaque tâche d’importation Azure doit avoir un numéro de suivi unique. Assurez-vous que les disques préparés par l’utilitaire dans le cadre d’une même tâche d’importation Azure sont expédiés ensemble dans un colis unique, et qu’il existe un numéro de suivi unique pour ce colis. Ne combinez pas dans un même colis des disques préparés dans le cadre de tâches d’importation Azure distinctes.

## <a name="update-shipping-details-on-the-azure-import-job"></a>Mettre à jour les informations d’expédition dans la tâche d’importation Azure

La procédure suivante met à jour les informations d’expédition de la tâche d’importation Azure. Ces informations incluent :

* le nom du transporteur qui livre les disques à Azure ;
* les détails de réexpédition pour vos disques.

1. Connectez-vous à votre abonnement Azure.
1. Dans le menu principal, sélectionnez **Tous les services**. Dans la boîte de dialogue **Tous les services**, entrez **Importer**. Lorsque vous voyez **Tâches d’importation/exportation**, sélectionnez-le.

    ![Entrer les informations d’expédition](./media/backup-azure-backup-import-export/search-import-job.png)<br/>

    Le menu **Tâches d’importation/exportation** s’ouvre et affiche la liste de toutes les tâches d’importation/exportation associées à l’abonnement sélectionné.

1. Si vous avez plusieurs abonnements, sélectionnez l’abonnement utilisé pour importer les données de sauvegarde. Sélectionnez ensuite la tâche d’importation que vous venez de créer pour en afficher les informations détaillées.

    ![Réviser les informations d’expédition](./media/backup-azure-backup-import-export/import-job-found.png)<br/>

1. Dans le menu **Paramètres** de la tâche d’importation, sélectionnez **Gérer les informations d’expédition**. Entrez les détails de réexpédition.

    ![Stocker les informations d’expédition](./media/backup-azure-backup-import-export/shipping-info.png)<br/>

1. Une fois que votre transporteur vous a transmis le numéro de suivi, sélectionnez la bannière dans la page de la tâche d’importation Azure et entrez les informations suivantes.

   > [!IMPORTANT]
   > Assurez-vous que les informations relatives au transporteur et au numéro de suivi sont mises à jour dans un délai de deux semaines après la création de la tâche d’importation Azure. Si ces informations ne sont pas confirmées dans un délai de deux semaines, la tâche peut être annulée, annulant de ce fait le traitement des disques.

   ![Alerte de mise à jour des informations de suivi](./media/backup-azure-backup-import-export/joboverview.png)<br/>

   ![Informations sur le transporteur et numéro de suivi](./media/backup-azure-backup-import-export/tracking-info.png)

### <a name="time-to-process-the-drives"></a>Temps de traitement des lecteurs

La durée nécessaire au traitement d’une tâche d’importation Azure varie. Le temps de traitement dépend de facteurs tels que le délai d’expédition, le type de tâche, le type et la taille des données copiées, ainsi que la taille des disques fournis. Le service Azure Import/Export n’est adossé à aucun Contrat de niveau de service. Une fois les disques reçus, le service s’efforce de terminer la copie des données de sauvegarde vers votre compte de stockage Azure dans les 7 à 10 jours.

### <a name="monitor-azure-import-job-status"></a>Surveiller l’état de la tâche d’importation Azure

Vous pouvez suivre l’état de votre tâche d’importation dans le Portail Azure. Accédez à la page **Tâches d’importation/exportation** et sélectionnez votre tâche. Pour en savoir plus sur l’état des tâches d’importation, consultez l’article [Qu’est-ce que le service Azure Import/Export ?](../storage/common/storage-import-export-service.md).

### <a name="finish-the-workflow"></a>Terminer le workflow

Une fois la tâche d’importation terminée, les données de sauvegarde initiale sont disponibles dans votre compte de stockage. Lors de la sauvegarde planifiée suivante, Sauvegarde Azure copie le contenu des données à partir du compte de stockage vers le coffre Recovery Services.

   ![Copier les données vers le coffre Recovery Services](./media/backup-azure-backup-import-export/copyingfromstorageaccounttoazurebackup.png)<br/>

Lors de la sauvegarde planifiée suivante, le service de sauvegarde Azure effectue une sauvegarde incrémentielle.

### <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois la sauvegarde initiale terminée, vous pouvez supprimer sans risque les données importées dans le conteneur Stockage Azure et les données de sauvegarde enregistrées dans l’emplacement intermédiaire.

## <a name="next-steps"></a>Étapes suivantes

* Pour toute question concernant le workflow du service Azure Import/Export, consultez [Utilisation du service Microsoft Azure Import/Export pour transférer des données vers Stockage Blob](../storage/common/storage-import-export-service.md).
