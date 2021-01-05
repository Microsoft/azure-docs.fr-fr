---
title: Sauvegarde hors connexion pour DPM et le serveur de sauvegarde Azure
description: Grâce à Sauvegarde Azure, vous pouvez envoyer des données en dehors du réseau à l’aide du service Azure Import/Export. Cet article explique le workflow de sauvegarde hors connexion pour DPM et le serveur de sauvegarde Azure.
ms.topic: conceptual
ms.date: 05/24/2020
ms.openlocfilehash: 368ae846a24ec04ee4b7da9b5971c00180be611d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89378455"
---
# <a name="offline-backup-workflow-for-dpm-and-azure-backup-server-mabs"></a>Flux de travail de la sauvegarde hors connexion pour DPM et le serveur de sauvegarde Azure (MABS)

>[!IMPORTANT]
> Ces étapes s’appliquent à DPM 2019 UR1 (ou version ultérieure) et à MABS v3 UR1 (ou version ultérieure).

System Center Data Protection Manager et le serveur de sauvegarde Azure (MABS) sont intégrés au service Sauvegarde Azure et utilisent plusieurs fonctionnalités intégrées qui permettent d’économiser les coûts de stockage et de réseau pendant les sauvegardes initiales complètes des données dans Azure. Les sauvegardes complètes initiales transfèrent généralement de grandes quantités de données et requièrent davantage de bande passante, en comparaison avec les sauvegardes suivantes qui transfèrent uniquement les données deltas/incrémentielles. La sauvegarde Azure compresse les sauvegardes initiales. Via le processus d’amorçage hors connexion, la sauvegarde Azure peut utiliser des disques pour charger les données de sauvegarde initiale compressées hors connexion dans Azure.

Le processus d’amorçage hors connexion de Sauvegarde Azure est étroitement intégré au [service Azure Import/Export](../storage/common/storage-import-export-service.md). Vous pouvez utiliser ce service pour transférer des données vers Azure à l’aide de disques. Si vous devez transférer des téraoctets (To) de données de sauvegarde initiales sur un réseau à latence élevée et à faible bande passante, vous pouvez utiliser le workflow d’amorçage hors connexion pour expédier la copie de sauvegarde initiale sur un ou plusieurs disques durs à un centre de données Azure. Cet article donne une vue d’ensemble et des étapes supplémentaires qui terminent ce workflow pour Microsoft System Center Data Protection Manager (DPM) et le serveur de sauvegarde Microsoft Azure (MABS).

> [!NOTE]
> Le processus de sauvegarde hors connexion pour l’agent Microsoft Azure Recovery Services (MARS) est distinct de celui de DPM et de MABS. Pour plus d’informations sur l’utilisation de la sauvegarde hors connexion avec l’agent MARS, consultez [Workflow de la sauvegarde hors connexion dans Sauvegarde Azure](backup-azure-backup-import-export.md). La sauvegarde hors connexion n’est pas prise en charge pour les sauvegardes d’état du système effectuées à l’aide de l’agent de sauvegarde Azure.
>
> La mise à jour MABS UR1 affiche également la version préliminaire pour le mode sauvegarde hors connexion avec Azure Data Box dans MABS. Pour en savoir plus, contactez [SystemCenterFeedback@microsoft.com](mailto:SystemCenterFeedback@microsoft.com).

## <a name="overview"></a>Vue d’ensemble

Grâce à la capacité d’amorçage hors connexion de Sauvegarde Azure et au service Azure Import/Export, vous pouvez charger facilement les données dans Azure hors connexion à l’aide de disques. Le processus de sauvegarde hors connexion inclut les étapes suivantes :

> [!div class="checklist"]
>
> * Les données de sauvegarde sont écrites dans un emplacement intermédiaire au lieu d’être envoyées sur le réseau.
> * Les données figurant dans l’emplacement intermédiaire sont ensuite écrites sur un ou plusieurs disques SATA à l’aide de l’utilitaire *AzureOfflineBackupDiskPrep*.
> * L’utilitaire crée automatiquement une tâche d’importation Azure.
> * Les disques SATA sont ensuite envoyés au centre de données Azure le plus proche.
> * Une fois les données de sauvegarde chargées dans Azure, la sauvegarde Azure copie les données de sauvegarde dans le coffre de sauvegarde, et des sauvegardes incrémentielles sont planifiées.

## <a name="prerequisites"></a>Prérequis

Assurez-vous que les prérequis suivants sont remplis avant de démarrer le workflow de sauvegarde hors connexion :

* Un [coffre Recovery Services](backup-azure-recovery-services-vault-overview.md) a été créé. Pour en créer un, suivez les étapes décrites dans [Créer un coffre Recovery Services](tutorial-backup-windows-server-to-azure.md#create-a-recovery-services-vault)tutorial-backup-windows-server-to-azure#create-a-recovery-services-vault).
* Vérifiez que seule la [dernière version de l’agent Microsoft Azure Recovery Services](https://aka.ms/azurebackup_agent) est installée sur SC DPM ou MABS, et qu’elle est inscrite auprès d’un coffre Recovery Services.
* Le correctif cumulatif 1 est installé sur SC DPM 2019 ou MABS v3.

  > [!NOTE]
  > Avec DPM 2019 UR1 et MABS v3 UR1 l’amorçage hors connexion s’authentifie à l’aide d’Azure Active Directory.

* Sur le serveur DPM ou MABS, assurez-vous que soit installé Microsoft Edge ou Internet Explorer 11 et que JavaScript soit activé.
* Créez un compte de stockage Azure dans le même abonnement que le coffre Recovery Services.
* Assurez-vous de disposer des [autorisations nécessaires](../active-directory/develop/howto-create-service-principal-portal.md) pour créer l’application Azure Active Directory. Le flux de travail de sauvegarde hors connexion crée une application Azure Active Directory dans l’abonnement associé au compte de stockage Azure. L’objectif de l’application est de fournir au service de sauvegarde Azure un accès sécurisé et délimité au service d’importation Azure qui est requis pour le flux de travail de sauvegarde hors connexion.
* Inscrivez le fournisseur de ressources Microsoft.ImportExport dans l’abonnement contenant le compte de stockage Azure. Pour inscrire le fournisseur de ressources :
    1. Dans le menu principal, sélectionnez **Abonnements**.
    2. Si vous disposez de plusieurs abonnements, sélectionnez celui que vous utilisez pour la sauvegarde hors connexion. Si vous n’utilisez qu’un seul abonnement, celui-ci s’affiche.
    3. Dans le menu de l’abonnement, sélectionnez **Fournisseurs de ressources** pour afficher la liste des fournisseurs.
    4. Faites défiler la liste des fournisseurs jusqu’à Microsoft.ImportExport. Si l’état est NotRegistered, sélectionnez **Inscrire**.

       ![Inscription du fournisseur de ressources](./media/backup-azure-backup-server-import-export/register-import-export.png)

* Un emplacement intermédiaire est créé. Il peut s’agir d’un partage réseau ou de tout lecteur supplémentaire, interne ou externe, sur l’ordinateur offrant suffisamment d’espace disque pour conserver votre copie initiale. Par exemple, si vous souhaitez sauvegarder un serveur de fichiers de 500 Go, assurez-vous que la zone intermédiaire dispose d’au moins 500 Go. (bien qu’une quantité inférieure soit utilisée en raison de la compression).
* Pour les disques envoyés à Azure, assurez-vous que seuls des disques SSD de 2,5 pouces ou des disques durs internes SATA II/III de 2,5 ou 3,5 pouces sont utilisés. La capacité maximale par disque dur est de 10 To. Consultez la [documentation sur le service Azure Import/Export](../storage/common/storage-import-export-requirements.md#supported-hardware) pour connaître la dernière série de disques pris en charge par le service.
* Les disques SATA doivent être connectés à un ordinateur (appelé *ordinateur de copie*) à partir duquel est effectuée la copie des données de sauvegarde de l’emplacement intermédiaire vers les disques SATA. Vérifiez que BitLocker est activé sur l’ordinateur de copie.

## <a name="workflow"></a>Workflow

Les informations de cette section vous permettent d’effectuer le flux de travail de sauvegarde hors connexion, afin que vos données puissent être remises à un centre de données Azure et chargées dans le stockage Azure. Si vous avez des questions sur le service Import ou sur un aspect du processus, consultez la documentation sur la [vue d’ensemble du service Import](../storage/common/storage-import-export-service.md) susmentionnée.

## <a name="initiate-offline-backup"></a>Lancer la sauvegarde hors connexion

1. Lorsque vous créez un nouveau groupe de protection avec la protection en ligne ou ajoutez une protection en ligne à un groupe de protection existant, l’écran suivant s’affiche. Pour sélectionner la méthode de réplication en ligne initiale, sélectionnez **Transfert depuis mon propre disque** puis sélectionnez **Suivant**.

    ![Écran d’importation](./media/backup-azure-backup-server-import-export/create-new-protection-group.png)

2. La page de connexion à Azure s’ouvrira. Connectez-vous avec votre compte d’utilisateur Azure, qui dispose de l’autorisation de rôle *propriétaire* sur l’abonnement Azure.

    ![Page de connexion à Azure](./media/backup-azure-backup-server-import-export/choose-initial-online-replication.png)

3. Renseignez les champs sur la page **Utiliser votre propre disque**.

   ![Entrées pour la page Utiliser votre propre disque](./media/backup-azure-backup-server-import-export/use-your-own-disk.png)

   Les champs sont décrits ci-dessous :

   * **Emplacement intermédiaire** : Emplacement de stockage temporaire dans lequel la copie de sauvegarde initiale est écrite. L’emplacement intermédiaire peut être sur un partage réseau ou un ordinateur local. Si l’ordinateur de copie et l’ordinateur source sont différents, spécifiez le chemin d’accès réseau complet de l’emplacement intermédiaire.
   * **Compte de stockage Azure Resource Manager** : Nom du compte de stockage de type Resource Manager (usage général v1 ou usage général v2) dans un abonnement Azure quelconque.
   * **Conteneur de stockage Azure** : Nom du conteneur de stockage d’objet blob cible dans le compte de stockage Azure où les données de sauvegarde sont importées.
   * **ID d’abonnement Azure** : ID de l’abonnement Azure dans lequel le compte de stockage Azure est créé.
   * **Nom de la tâche d’importation Azure** : Nom unique utilisé par le service Azure Import et Sauvegarde Azure pour effectuer le suivi du transfert des données envoyées dans Azure à l’aide de disques.

    Enregistrez les informations **Emplacement intermédiaire** et **Nom de tâche d’importation Azure** que vous avez fournies. Elles sont nécessaires pour préparer les disques.

4. Terminez le flux de travail pour créer ou mettre à jour la protection. Pour lancer la copie de sauvegarde hors connexion, faites un clic droit sur le **groupe de protection**, puis choisissez l’option **Créer un point de récupération**. Sélectionnez ensuite l’option **Protection en ligne** .

   ![Créer un point de récupération](./media/backup-azure-backup-server-import-export/create-recovery-point.png)

5. Surveillez le travail de création de réplica en ligne dans le volet Analyse. Le travail se termine correctement et affiche l’avertissement *En attente de la fin de la tâche d’importation Azure*.

   ![Terminer le point de récupération](./media/backup-azure-backup-server-import-export/complete-recovery-point.png)

6. Une fois l’opération terminée, l’emplacement intermédiaire est prêt à être utilisé pour la préparation des disques.

## <a name="prepare-sata-drives-and-ship-to-azure"></a>Préparer des disques SATA et envoyer à Azure

L’utilitaire *AzureOfflineBackupDiskPrep* prépare les disques SATA qui sont envoyés au centre de données Azure le plus proche. Cet utilitaire est disponible dans le répertoire d’installation de l’agent Sauvegarde Azure (dont le chemin d’accès est le suivant) : `*\Microsoft Azure Recovery Services Agent\Utils\\*`

1. Accédez au répertoire, puis copiez le répertoire **AzureOfflineBackupDiskPrep** vers un autre ordinateur auquel les disques SATA sont connectés. Sur l’ordinateur auquel les disques SATA sont connectés, vérifiez les points suivants :

   * L’ordinateur de copie peut accéder à l’emplacement intermédiaire spécifié pour le workflow d’amorçage hors connexion via le même chemin d’accès réseau que celui fourni dans le workflow de la section « Lancer la sauvegarde hors connexion ».
   * BitLocker est activé sur l’ordinateur de copie.
   * Azure PowerShell 3.7.0 est installé sur l’ordinateur de copie (pas nécessaire si vous exécutez l’utilitaire AzureOfflineBackupDiskPrep sur le serveur DPM ou MABS).
   * Les navigateurs compatibles les plus récents (Microsoft Edge ou Internet Explorer 11) sont installés et JavaScript est activé.
   * L’ordinateur de copie peut accéder au portail Azure. Si nécessaire, l’ordinateur de copie peut être le même que l’ordinateur source.

     > [!IMPORTANT]
     > Si l’ordinateur source est une machine virtuelle, il est obligatoire d’utiliser un autre serveur physique ou un ordinateur client comme ordinateur de copie.

1. Ouvrez une invite de commande avec élévation de privilèges sur l’ordinateur de copie en utilisant le répertoire de l’utilitaire *AzureOfflineBackupDiskPrep* comme répertoire actif. Exécutez la commande suivante :

    ```console
    .\AzureOfflineBackupDiskPrep.exe s:<Staging Location Path>
    ```

    | Paramètre | Description |
    | --- | --- |
    | s:&lt;*Staging Location Path*&gt; |Cette entrée obligatoire est utilisée pour fournir le chemin d’accès vers l’emplacement intermédiaire que vous avez entré dans le workflow de la section « Lancer la sauvegarde hors connexion ». |
    | p:&lt;*Path to PublishSettingsFile*&gt; |Cette entrée facultative est utilisée pour fournir le chemin du fichier de paramètres de publication Azure. |

    Quand vous exécutez la commande, l’utilitaire vous demande de sélectionner la tâche d’importation Azure correspondant aux disques à préparer. S’il existe un seul travail d’importation associé à l’emplacement intermédiaire spécifié, un écran similaire à l’écran suivant s’affiche.

      ![Console de préparation de disque](./media/backup-azure-backup-server-import-export/disk-prep-console.png)

1. Entrez la lettre du disque, sans les deux-points de fin, correspondant au disque monté que vous souhaitez préparer pour le transfert vers Azure.
1. Confirmez le formatage du disque lorsque vous y êtes invité.
1. Vous êtes invité à vous connecter à votre abonnement Azure. Indiquez vos informations d’identification.

    ![Écran de connexion Azure](./media/backup-azure-backup-server-import-export/signin-disk-prep.png)

    L’outil commence ensuite à préparer le disque et à copier les données de sauvegarde. Vous devrez peut-être ajouter des disques supplémentaires lorsque l’outil vous y invite, si le disque fourni n’a pas suffisamment d’espace pour les données de sauvegarde. <br/>

    Une fois que l’outil a été correctement exécuté, l’invite de commandes fournit trois informations :
    * Le ou les disques que vous avez fournis sont préparés en vue de leur envoi dans un centre de données Azure.
    * Vous recevez la confirmation que votre tâche d’importation a été créée. La tâche d’importation utilise le nom que vous avez fourni.
    * L’outil affiche l’adresse d’expédition correspondant au centre de données Azure.

     ![Préparation des disques Azure terminée](./media/backup-azure-backup-server-import-export/console.png)

1. À la fin de l’exécution de la commande, vous voyez également s’afficher l’option permettant de mettre à jour les informations d’expédition.

1. Expédiez les disques à l’adresse indiquée par l’outil et conservez le numéro de suivi pour référence.

   > [!IMPORTANT]
   > Chaque tâche d’importation Azure doit avoir un numéro de suivi unique. Assurez-vous que les disques préparés par l’utilitaire dans le cadre d’une même tâche d’importation Azure sont expédiés ensemble dans un colis unique, et qu’il existe un numéro de suivi unique pour ce colis. Ne combinez pas dans un même package des lecteurs préparés dans le cadre de tâches d’importation Azure différentes.

## <a name="update-shipping-details-on-the-azure-import-job"></a>Mettre à jour les informations d’expédition dans la tâche d’importation Azure

La procédure suivante met à jour les informations d’expédition de la tâche d’importation Azure. Ces informations incluent :

* le nom du transporteur qui livre les disques à Azure
* les détails de retour d’expédition pour vos disques

   1. Connectez-vous à votre abonnement Azure.
   2. Dans le menu principal, sélectionnez **Tous les services**, puis, dans la boîte de dialogue Tous les services, saisissez Importer. Lorsque vous voyez **Tâches d’importation/d’exportation**, sélectionnez-le.
       ![Saisie des informations d’expédition](./media/backup-azure-backup-server-import-export/search-import-job.png)

       Le menu **Tâches d’importation/exportation** s’ouvre et affiche la liste de toutes les tâches d’importation/exportation associées à l’abonnement sélectionné.

   3. Si vous avez plusieurs abonnements, veillez à sélectionner l’abonnement utilisé pour importer les données de sauvegarde. Sélectionnez ensuite la tâche d’importation que vous venez de créer pour en afficher les informations détaillées.

       ![Réviser les informations d’expédition](./media/backup-azure-backup-server-import-export/import-job-found.png)

   4. Dans le menu Paramètres de la tâche d’importation, sélectionnez **Manage Shipping Info** (Gérer les informations d’expédition) et saisissez les informations de retour d’expédition.

       ![Stockage des informations d’expédition](./media/backup-azure-backup-server-import-export/shipping-info.png)

   5. Une fois que votre transporteur vous a transmis le numéro de suivi, sélectionnez la bannière dans la pageTâche d’importation d’Azure et entrez les informations suivantes :

      > [!IMPORTANT]
      > Assurez-vous que les informations relatives au transporteur et au numéro de suivi sont mises à jour dans un délai de deux semaines après la création de la tâche d’importation Azure. Si ces informations ne sont pas confirmées dans un délai de deux semaines, la tâche peut être annulée, annulant de ce fait le traitement des disques.

      ![Vue d’ensemble de la tâche](./media/backup-azure-backup-server-import-export/job-overview.png)

      ![Informations de suivi](./media/backup-azure-backup-server-import-export/tracking-info.png)

### <a name="time-to-process-the-drives"></a>Temps de traitement des lecteurs

La durée nécessaire au traitement d’une tâche d’importation Azure varie. Le temps de traitement dépend de facteurs tels que le délai d’expédition, le type de tâche, le type et la taille des données copiées, ainsi que la taille des disques fournis. Le service Azure Import/Export n’est adossé à aucun Contrat de niveau de service. Une fois les disques reçus, le service s’efforce de terminer la copie des données de sauvegarde vers votre compte de stockage Azure dans les 7 à 10 jours. La section suivante décrit la façon dont vous pouvez surveiller l’état de la tâche d’importation Azure.

### <a name="monitor-azure-import-job-status"></a>Surveiller l’état de la tâche d’importation Azure

Vous pouvez suivre l’état de votre tâche d’importation à partir du portail Azure en accédant à la page **Tâches d’importation/exportation** et en sélectionnant votre tâche. Pour en savoir plus sur l’état des tâches d’importation, consultez l’article [Storage Import Export service](../storage/common/storage-import-export-service.md) (Service d’importation/exportation du stockage).

### <a name="complete-the-workflow"></a>Terminer le flux de travail

Une fois le travail d’importation terminé, les données de sauvegarde initiale sont disponibles dans votre compte de stockage. Lors de la sauvegarde planifiée suivante, Sauvegarde Azure copie le contenu des données à partir du compte de stockage vers le coffre Recovery Services.

Lors du prochain travail de création de réplica planifié, Data Protection Manager effectue une sauvegarde incrémentielle par-dessus la copie de sauvegarde initiale.

## <a name="next-steps"></a>Étapes suivantes

* Pour toute question concernant le workflow du service Azure Import/Export, consultez [Utilisation du service Microsoft Azure Import/Export pour transférer des données vers Stockage Blob](../storage/common/storage-import-export-service.md).
