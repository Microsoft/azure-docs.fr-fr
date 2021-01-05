---
title: Sauvegarde en mode hors connexion pour Data Protection Manager (DPM) et le serveur de sauvegarde Microsoft Azure (MABS) – versions précédentes
description: Grâce à Sauvegarde Azure, vous pouvez envoyer des données en dehors du réseau à l’aide du service Azure Import/Export. Cet article explique le workflow de sauvegarde hors connexion pour les versions précédentes de DPM et du serveur de sauvegarde Azure.
ms.topic: conceptual
ms.date: 06/08/2020
ms.openlocfilehash: b747fd3c682dc1caf7312ba7279470a1e6b38bd5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88890091"
---
# <a name="offline-backup-workflow-for-dpm-and-azure-backup-server-previous-versions"></a>Workflow de sauvegarde en mode hors connexion pour DPM et le serveur de sauvegarde Azure (versions précédentes)

>[!IMPORTANT]
>Ces étapes s’appliquent à DPM 2019 RTM et versions antérieures ainsi qu’à MABS v3 RTM et versions antérieures.

La sauvegarde Azure offre plusieurs fonctionnalités intégrées pour réduire les coûts de stockage et de réseau pendant les sauvegardes complètes initiales des données dans Azure. Les sauvegardes complètes initiales transfèrent généralement de grandes quantités de données et requièrent davantage de bande passante, en comparaison avec les sauvegardes suivantes qui transfèrent uniquement les données deltas/incrémentielles. La sauvegarde Azure compresse les sauvegardes initiales. Via le processus d’amorçage hors connexion, la sauvegarde Azure peut utiliser des disques pour charger les données de sauvegarde initiale compressées hors connexion dans Azure.

Le processus d’amorçage hors connexion de Sauvegarde Azure est étroitement intégré au [service Azure Import/Export](../storage/common/storage-import-export-service.md). Vous pouvez utiliser ce service pour transférer des données vers Azure à l’aide de disques. Si vous devez transférer des téraoctets (To) de données de sauvegarde initiales sur un réseau à latence élevée et à faible bande passante, vous pouvez utiliser le workflow d’amorçage hors connexion pour expédier la copie de sauvegarde initiale sur un ou plusieurs disques durs à un centre de données Azure. Cet article donne une vue d’ensemble et des étapes supplémentaires qui terminent ce workflow pour Microsoft System Center Data Protection Manager (DPM) et le serveur de sauvegarde Microsoft Azure (MABS).

> [!NOTE]
> Le processus de sauvegarde hors connexion pour l’agent Microsoft Azure Recovery Services (MARS) est distinct de celui de DPM et de MABS. Pour plus d’informations sur l’utilisation de la sauvegarde hors connexion avec l’agent MARS, consultez [Workflow de la sauvegarde hors connexion dans Sauvegarde Azure](backup-azure-backup-import-export.md). La sauvegarde hors connexion n’est pas prise en charge pour les sauvegardes d’état du système effectuées à l’aide de l’agent de sauvegarde Azure.
>

## <a name="overview"></a>Vue d’ensemble

Grâce à la capacité d’amorçage hors connexion de Sauvegarde Azure et au service Azure Import/Export, vous pouvez charger facilement les données dans Azure hors connexion à l’aide de disques. Le processus de sauvegarde hors connexion inclut les étapes suivantes :

> [!div class="checklist"]
>
> * Les données de sauvegarde sont écrites dans un emplacement intermédiaire au lieu d’être envoyées sur le réseau.
> * Les données figurant dans l’emplacement intermédiaire sont ensuite écrites sur un ou plusieurs disques SATA à l’aide de l’utilitaire *AzureOfflineBackupDiskPrep*.
> * L’utilitaire crée automatiquement une tâche d’importation Azure.
> * Les disques SATA sont ensuite envoyés au centre de données Azure le plus proche.
> * Une fois les données de sauvegarde chargées dans Azure, la sauvegarde Azure copie les données de sauvegarde dans le coffre de sauvegarde, et des sauvegardes incrémentielles sont planifiées.

## <a name="supported-configurations"></a>Configurations prises en charge

La sauvegarde hors connexion est prise en charge pour tous les modèles de déploiement de Sauvegarde Azure qui sauvegardent des données locales dans le cloud Microsoft. Ces modèles comprennent les éléments suivants :

> [!div class="checklist"]
>
> * Sauvegarde de fichiers et de dossiers avec l’agent MARS ou l’agent de sauvegarde Azure.
> * Sauvegarde de toutes les charges de travail et fichiers avec DPM.
> * Sauvegarde de toutes les charges de travail et fichiers avec MABS.

>[!NOTE]
>Les abonnements Azure CSP ne sont pas pris en charge dans le cadre d’une utilisation avec l’amorçage hors connexion pour DPM 2019 RTM et versions antérieures et MABS v3 RTM et versions antérieures. Les sauvegardes en ligne sur le réseau sont toujours prises en charge.

## <a name="prerequisites"></a>Prérequis

Assurez-vous que les prérequis suivants sont remplis avant de démarrer le workflow de sauvegarde hors connexion :

* Un [coffre Recovery Services](backup-azure-recovery-services-vault-overview.md) a été créé. Pour en créer un, suivez les étapes décrites dans [Créer un coffre Recovery Services](tutorial-backup-windows-server-to-azure.md#create-a-recovery-services-vault).
* L’agent de sauvegarde Azure, MABS ou DPM a été installé sur Windows Server ou un client Windows, selon le besoin, et l’ordinateur est inscrit auprès du coffre Recovery Services. Assurez-vous que seule la [version la plus récente de Sauvegarde Azure](https://go.microsoft.com/fwlink/?linkid=229525) est utilisée.
* [Téléchargez le fichier des paramètres de publication Azure](https://portal.azure.com/#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade) sur l’ordinateur à partir duquel vous prévoyez de sauvegarder vos données. L’abonnement à partir duquel vous téléchargez le fichier des paramètres de publication peut être différent de celui contenant le coffre Recovery Services. Si votre abonnement figure sur un cloud Azure souverain, utilisez les liens appropriés suivants pour télécharger le fichier des paramètres de publication Azure.

    | Région cloud souveraine | Lien du fichier des paramètres de publication Azure |
    | --- | --- |
    | États-Unis | [Lien](https://portal.azure.us#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade) |
    | Chine | [Lien](https://portal.azure.cn/#blade/Microsoft_Azure_ClassicResources/PublishingProfileBlade) |

* Un compte de stockage Azure avec un modèle de déploiement Resource Manager a été créé dans l’abonnement à partir duquel vous avez téléchargé le fichier des paramètres de publication. Dans le compte de stockage, créez un conteneur de blobs qui servira de destination.

  ![Créer un compte de stockage avec développement Resource Manager](./media/offline-backup-dpm-mabs-previous-versions/storage-account-resource-manager.png)

* Un emplacement intermédiaire est créé. Il peut s’agir d’un partage réseau ou de tout lecteur supplémentaire, interne ou externe, sur l’ordinateur offrant suffisamment d’espace disque pour conserver votre copie initiale. Par exemple, si vous souhaitez sauvegarder un serveur de fichiers de 500 Go, assurez-vous que la zone intermédiaire dispose d’au moins 500 Go. (bien qu’une quantité inférieure soit utilisée en raison de la compression).
* Pour les disques envoyés à Azure, assurez-vous que seuls des disques SSD de 2,5 pouces ou des disques durs internes SATA II/III de 2,5 ou 3,5 pouces sont utilisés. La capacité maximale par disque dur est de 10 To. Consultez la [documentation sur le service Azure Import/Export](../storage/common/storage-import-export-requirements.md#supported-hardware) pour connaître la dernière série de disques pris en charge par le service.
* Les disques SATA doivent être connectés à un ordinateur (appelé *ordinateur de copie*) à partir duquel est effectuée la copie des données de sauvegarde de l’emplacement intermédiaire vers les disques SATA. Vérifiez que BitLocker est activé sur l’ordinateur de copie.

## <a name="prepare-the-server-for-the-offline-backup-process"></a>Préparer le serveur au processus de sauvegarde hors connexion

>[!NOTE]
> Si vous ne trouvez pas les utilitaires listés, par exemple *AzureOfflineBackupCertGen.exe*, dans votre installation de l’agent MARS, écrivez à AskAzureBackupTeam@microsoft.com pour pouvoir y accéder.

* Ouvrez une invite de commandes avec élévation de privilèges sur le serveur, puis exécutez la commande suivante :

    ```cmd
    AzureOfflineBackupCertGen.exe CreateNewApplication SubscriptionId:<Subs ID>
    ```

    L’outil crée une application Azure Active Directory de sauvegarde hors connexion s’il n’en existe pas.

    Si une application existe déjà, cet exécutable vous invite à charger manuellement le certificat sur l’application dans le locataire. Suivez les étapes décrites dans [cette section](#manually-upload-an-offline-backup-certificate) pour charger le certificat manuellement sur l’application.

* L’outil *AzureOfflineBackupCertGen.exe* génère un fichier *OfflineApplicationParams.xml*. Copiez ce fichier sur le serveur avec MABS ou DPM.
* Installez l’[agent MARS le plus récent](https://aka.ms/azurebackup_agent) sur l’instance DPM ou le serveur de sauvegarde Azure.
* Inscrivez le serveur sur Azure.
* Exécutez la commande suivante :

    ```cmd
    AzureOfflineBackupCertGen.exe AddRegistryEntries SubscriptionId:<subscriptionid> xmlfilepath:<path of the OfflineApplicationParams.xml file>  storageaccountname:<storageaccountname to be used for offline backup>
    ```

* La commande précédente crée le fichier `C:\Program Files\Microsoft Azure Recovery Services Agent\Scratch\MicrosoftBackupProvider\OfflineApplicationParams_<Storageaccountname>.xml`.

## <a name="manually-upload-an-offline-backup-certificate"></a>Charger manuellement un certificat de sauvegarde hors connexion

Procédez comme suit pour charger manuellement le certificat de sauvegarde hors connexion sur une application Azure Active Directory précédemment créée et destinée à la sauvegarde hors connexion.

1. Connectez-vous au portail Azure.
1. Accédez à **Azure Active Directory** > **Inscriptions des applications**.
1. Dans l’onglet **Applications détenues**, recherchez une application au format de nom d’affichage `AzureOfflineBackup _<Azure User Id`.

    ![Localiser l’application sous l’onglet Applications détenues](./media/offline-backup-dpm-mabs-previous-versions/owned-applications.png)

1. Sélectionnez l’application. Sous **Gérer** dans le volet gauche, accédez à **Certificats et secrets**.
1. Vérifiez s’il existe déjà des certificats ou des clés publiques. S’il n’en existe pas, vous pouvez supprimer l’application en toute sécurité en sélectionnant le bouton **Supprimer** dans la page **Vue d’ensemble** de l’application. Ensuite, vous pouvez réessayer les étapes permettant de [préparer le serveur à la sauvegarde hors connexion](#prepare-the-server-for-the-offline-backup-process) et ignorer les étapes suivantes. Dans le cas contraire, continuez à suivre ces étapes à partir de l’instance DPM ou du serveur de sauvegarde Azure sur lequel vous souhaitez configurer la sauvegarde hors connexion.
1. À partir de **Démarrer** – **Exécuter**, tapez *Certlm.msc*. Dans la fenêtre **Certificats – Ordinateur local**, sélectionnez l’onglet **Certificats – Ordinateur local** > **Personnel**. Recherchez le certificat portant le nom `CB_AzureADCertforOfflineSeeding_<ResourceId>`.
1. Sélectionnez le certificat, cliquez avec le bouton droit sur **Toutes les tâches**, puis sélectionnez **Exporter**, sans clé privée, au format .cer.
1. Accédez à l’application de sauvegarde hors connexion Azure dans le Portail Azure.
1. Sélectionnez **Gérer** > **Certificats et secrets** > **Charger le certificat**. Chargez le certificat exporté à l’étape précédente.

    ![Téléchargement du certificat](./media/offline-backup-dpm-mabs-previous-versions/upload-certificate.png)

1. Sur le serveur, ouvrez le registre en entrant **regedit** dans la fenêtre d’exécution.
1. Accédez à l’entrée de Registre *Ordinateur\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config\CloudBackupProvider*.
1. Cliquez avec le bouton droit sur **CloudBackupProvider** et ajoutez une nouvelle valeur de chaîne portant le nom `AzureADAppCertThumbprint_<Azure User Id>`.

    >[!NOTE]
    > Pour trouver l’identifiant utilisateur Azure, effectuez l’une des étapes suivantes :
    >
    >* À partir du PowerShell connecté à Azure, exécutez la commande `Get-AzureRmADUser -UserPrincipalName "Account Holder's email as appears in the portal"`.
    >* Accédez au chemin d’accès du registre `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\DbgSettings\OnlineBackup; Name: CurrentUserId;`.

1. Cliquez avec le bouton droit sur la chaîne ajoutée à l’étape précédente, puis sélectionnez **Modifier**. Dans la valeur, fournissez l’empreinte du certificat que vous avez exporté à l’étape 7. Sélectionnez ensuite **OK**.
1. Pour afficher la valeur de l’empreinte, double-cliquez sur le certificat. Sélectionnez l’onglet **Détails**, puis faites défiler jusqu’à ce que la champ Empreinte s’affiche. Sélectionnez **Empreinte** et copiez la valeur.

    ![Copier la valeur du champ de l’empreinte](./media/offline-backup-dpm-mabs-previous-versions/thumbprint-field.png)

1. Passez à la section [Workflow](#workflow) pour poursuivre le processus de sauvegarde hors connexion.

## <a name="workflow"></a>Workflow

Les informations de cette section vous permettent de terminer le workflow de sauvegarde hors connexion, afin que vos données puissent être remises à un centre de données Azure et chargées dans Stockage Azure. Si vous avez des questions sur le service d’importation ou sur un aspect du processus, consultez la [documentation de présentation du service d’importation](../storage/common/storage-import-export-service.md) susmentionnée.

### <a name="initiate-offline-backup"></a>Lancer la sauvegarde hors connexion

1. Lorsque vous planifiez une sauvegarde, la page suivante s’affiche dans Windows Server, un client Windows ou DPM.

    ![Page Importer](./media/offline-backup-dpm-mabs-previous-versions/offline-backup-screen-inputs.png)

    Voici la page correspondante dans DPM. <br/>

    ![Page d’importation de DPM et du serveur de sauvegarde Azure](./media/offline-backup-dpm-mabs-previous-versions/dpm-offline.png)

    Les champs que vous remplissez sont les suivants :

   * **Emplacement intermédiaire** : Emplacement de stockage temporaire dans lequel la copie de sauvegarde initiale est écrite. L’emplacement intermédiaire peut être sur un partage réseau ou un ordinateur local. Si l’ordinateur de copie et l’ordinateur source sont différents, spécifiez le chemin d’accès réseau complet de l’emplacement intermédiaire.
   * **Paramètres de publication Azure** : Chemin d’accès local au fichier des paramètres de publication.
   * **Nom de la tâche d’importation Azure** : Nom unique utilisé par le service Azure Import/Export et par Sauvegarde Azure pour effectuer le suivi du transfert des données envoyées dans Azure à l’aide de disques.
   * **ID d’abonnement Azure** : ID de l’abonnement Azure à partir duquel vous avez téléchargé le fichier des paramètres de publication Azure.
   * **Compte de stockage Azure** : Nom du compte de stockage de l’abonnement Azure associé au fichier des paramètres de publication Azure.
   * **Conteneur de stockage Azure** : nom de l'objet blob de stockage cible dans le compte de stockage Azure où les données de sauvegarde sont importées.

   Enregistrez les informations **Emplacement intermédiaire** et **Nom de tâche d’importation Azure** que vous avez fournies. Elles sont nécessaires pour préparer les disques.

1. Terminez le workflow. Pour lancer la copie de sauvegarde hors connexion, sélectionnez **Sauvegarder maintenant** dans la console de gestion de l’agent de sauvegarde Azure. La sauvegarde initiale est écrite dans la zone intermédiaire à cette étape.

    ![Sauvegarder maintenant](./media/offline-backup-dpm-mabs-previous-versions/backup-now.png)

    Pour terminer le workflow correspondant dans DPM ou le serveur de sauvegarde Azure, cliquez avec le bouton droit sur le **groupe de protection**. Sélectionnez l’option **Créer un point de récupération**. Sélectionnez ensuite l’option **Protection en ligne**.

    ![Sauvegarder maintenant dans DPM et MABS](./media/offline-backup-dpm-mabs-previous-versions/dpm-backup-now.png)

    Une fois l’opération terminée, l’emplacement intermédiaire est prêt à être utilisé pour la préparation des disques.

    ![Progression de la sauvegarde](./media/offline-backup-dpm-mabs-previous-versions/op-backup-now.png)

### <a name="prepare-sata-drives-and-ship-to-azure"></a>Préparer des disques SATA et envoyer à Azure

L’utilitaire *AzureOfflineBackupDiskPrep* sert à préparer les disques SATA qui sont envoyés au centre de données Azure le plus proche. Cet utilitaire est disponible dans le répertoire d’installation de l’agent Recovery Services dont le chemin d’accès est le suivant :

`*\\Microsoft Azure Recovery Services Agent\Utils\*`

1. Accédez au répertoire, puis copiez le répertoire *AzureOfflineBackupDiskPrep* vers un ordinateur de copie auquel les disques SATA à préparer sont connectés. Assurez-vous que :

   * L’ordinateur de copie peut accéder à l’emplacement intermédiaire spécifié pour le workflow d’amorçage hors connexion via le même chemin d’accès réseau que celui fourni dans le workflow de la section « Lancer la sauvegarde hors connexion ».
   * BitLocker est activé sur l’ordinateur de copie.
   * L’ordinateur de copie peut accéder au portail Azure. Si nécessaire, l’ordinateur de copie peut être le même que l’ordinateur source.

     > [!IMPORTANT]
     > Si l’ordinateur source est une machine virtuelle, il est obligatoire d’utiliser un autre serveur physique ou un ordinateur client comme ordinateur de copie.

1. Ouvrez une invite de commande avec élévation de privilèges sur l’ordinateur de copie en utilisant le répertoire de l’utilitaire *AzureOfflineBackupDiskPrep* comme répertoire actif. Exécutez la commande suivante :

    `*.\AzureOfflineBackupDiskPrep.exe*   s:<*Staging Location Path*>   [p:<*Path to AzurePublishSettingsFile*>]`

    | Paramètre | Description |
    | --- | --- |
    | s:&lt;*Staging Location Path*&gt; |Cette entrée obligatoire est utilisée pour fournir le chemin d’accès vers l’emplacement intermédiaire que vous avez entré dans le workflow de la section « Lancer la sauvegarde hors connexion ». |
    | p:&lt;*Path to PublishSettingsFile*&gt; |Cette entrée optionnelle est utilisée pour fournir le chemin d’accès vers le fichier des paramètres de publication Azure que vous avez entré dans le workflow de la section « Lancer la sauvegarde hors connexion ». |

    > [!NOTE]
    > La valeur &lt;Chemin d’accès à AzurePublishSettingFile&gt; est obligatoire quand l’ordinateur de copie et l’ordinateur source sont différents.
    >
    >

    Quand vous exécutez la commande, l’utilitaire vous demande de sélectionner la tâche d’importation Azure correspondant aux disques à préparer. S’il existe une seule tâche d’importation associée à l’emplacement intermédiaire spécifié, une page similaire à celle-ci s’affiche.

    ![Entrée de l’outil de préparation des disques Azure](./media/offline-backup-dpm-mabs-previous-versions/azure-disk-preparation-tool-drive-input.png) <br/>

1. Entrez la lettre du disque, sans les deux-points de fin, correspondant au disque monté que vous souhaitez préparer pour le transfert vers Azure. Lorsque vous y êtes invité, confirmez le formatage du disque.

    L’outil commence ensuite à préparer le disque et à copier les données de sauvegarde. Vous devrez peut-être joindre des disques supplémentaires lorsque l’outil vous y invite, si le disque fourni n’a pas suffisamment d’espace pour les données de sauvegarde. <br/>

    À la fin de l’exécution correcte de l’outil, un ou plusieurs disques que vous avez fournis sont préparés pour l’expédition à Azure. Une tâche d’importation du nom que vous avez fourni lors du workflow de la section « Lancer la sauvegarde hors connexion » est également créée dans Azure. Enfin, l’outil affiche l’adresse d’expédition du centre de données Azure auquel les disques doivent être envoyés.

    ![Préparation des disques Azure terminée](./media/offline-backup-dpm-mabs-previous-versions/azure-disk-preparation-tool-success.png)<br/>

1. À la fin de l’exécution de la commande, vous voyez également s’afficher l’option permettant de mettre à jour les informations d’expédition.

    ![Option de mise à jour des informations d’expédition](./media/offline-backup-dpm-mabs-previous-versions/update-shipping-utility.png)<br/>

1. Vous pouvez entrer les détails immédiatement. L’outil vous guide tout au long du processus impliquant une série d’entrées. Si vous n’avez pas d’informations telles que le numéro de suivi ou d’autres détails concernant l’expédition, vous pouvez mettre fin à la session. La procédure de mise à jour des détails de l’expédition est décrite plus loin dans cet article.

1. Expédiez les disques à l’adresse fournie par l’outil. Conservez le numéro de suivi pour référence ultérieure.

   > [!IMPORTANT]
   > Chaque tâche d’importation Azure doit avoir un numéro de suivi unique. Assurez-vous que les disques préparés par l’utilitaire dans le cadre d’une même tâche d’importation Azure sont expédiés ensemble dans un colis unique, et qu’il existe un numéro de suivi unique pour ce colis. Ne combinez pas dans un même package des lecteurs préparés dans le cadre de tâches d’importation Azure différentes.

1. Lorsque vous disposez du numéro de suivi, accédez à l’ordinateur source, qui attend la fin de la tâche d’importation. Exécutez la commande suivante dans une invite de commande avec élévation de privilèges en utilisant le répertoire de l’utilitaire *AzureOfflineBackupDiskPrep* comme répertoire actif.

   `*.\AzureOfflineBackupDiskPrep.exe*  u:`

   Vous pouvez éventuellement exécuter la commande suivante à partir d’un autre ordinateur, tel que l’ordinateur de copie, en utilisant le répertoire de l’utilitaire *AzureOfflineBackupDiskPrep* en tant que répertoire actif.

   `*.\AzureOfflineBackupDiskPrep.exe*  u:  s:<*Staging Location Path*>   p:<*Path to AzurePublishSettingsFile*>`

    | Paramètre | Description |
    | --- | --- |
    | u: | Cette entrée obligatoire est utilisée pour mettre à jour les détails de l’expédition d’une tâche d’importation Azure. |
    | s:&lt;*Staging Location Path*&gt; | Cette entrée obligatoire est utilisée si la commande n’est pas exécutée sur l’ordinateur source. Elle est utilisée pour fournir le chemin d’accès vers l’emplacement intermédiaire que vous avez entré dans le workflow de la section « Lancer la sauvegarde hors connexion ». |
    | p:&lt;*Path to PublishSettingsFile*&gt; | Cette entrée obligatoire est utilisée si la commande n’est pas exécutée sur l’ordinateur source. Elle est utilisée pour fournir le chemin d’accès vers le fichier des paramètres de publication Azure que vous avez saisi dans le workflow de la section « Lancer la sauvegarde hors connexion ». |

    L’utilitaire détecte automatiquement la tâche d’importation que l’ordinateur source attend ou les tâches d’importation associées à l’emplacement intermédiaire lorsque la commande est exécutée sur un autre ordinateur. Il propose ensuite l’option de mise à jour des informations d’expédition au travers d’une série d’entrées.

    ![Entrer les informations d’expédition](./media/offline-backup-dpm-mabs-previous-versions/shipping-inputs.png)<br/>

1. Une fois toutes les informations entrées, révisez attentivement leurs détails et validez-les en entrant **oui**.

    ![Réviser les informations d’expédition](./media/offline-backup-dpm-mabs-previous-versions/review-shipping-information.png)<br/>

1. Une fois la mise à jour des informations d’expédition effectuée, l’utilitaire fournit un emplacement local où stocker les informations d’expédition que vous avez entrées.

    ![Stocker les informations d’expédition](./media/offline-backup-dpm-mabs-previous-versions/storing-shipping-information.png)<br/>

   > [!IMPORTANT]
   > Assurez-vous que les lecteurs parviennent au centre de données Azure dans les deux semaines suivant la fourniture des informations d’expédition à l’aide de l’utilitaire *AzureOfflineBackupDiskPrep*. À défaut, les lecteurs pourraient ne pas être traités.

Une fois les étapes précédentes terminées, le centre de données Azure est prêt à recevoir les lecteurs et à les traiter afin de transférer les données de sauvegarde des lecteurs vers le compte de stockage Azure de type classique que vous avez créé.

### <a name="time-to-process-the-drives"></a>Temps de traitement des lecteurs

La durée nécessaire au traitement d’une tâche d’importation Azure varie. Le temps de traitement dépend de facteurs tels que le délai d’expédition, le type de tâche, le type et la taille des données copiées, ainsi que la taille des disques fournis. Le service Azure Import/Export n’est adossé à aucun Contrat de niveau de service. Une fois les disques reçus, le service s’efforce de terminer la copie des données de sauvegarde vers votre compte de stockage Azure dans les 7 à 10 jours. La section suivante décrit la façon dont vous pouvez surveiller l’état de la tâche d’importation Azure.

### <a name="monitor-azure-import-job-status"></a>Surveiller l’état de la tâche d’importation Azure

Quand vos lecteurs sont en transit ou se trouvent dans le centre de données Azure en vue de leurs copie vers le compte de stockage, l’agent Sauvegarde Azure, DPM ou la console MABS sur l’ordinateur source affichent l’état de la tâche suivante pour vos sauvegardes planifiées :

  `Waiting for Azure Import Job to complete. Please check on Azure Management portal for more information on job status`

Pour vérifier l’état de la tâche d’importation :

1. Ouvrez une invite de commandes avec élévation de privilèges sur l’ordinateur source, puis exécutez la commande suivante :

     `AzureOfflineBackupDiskPrep.exe u:`

1. La sortie indique l’état actuel de la tâche d’importation.

    ![Vérifier l’état de la tâche d’importation](./media/offline-backup-dpm-mabs-previous-versions/import-job-status-reporting.png)<br/>

Pour plus d’informations sur les différents états de la tâche d’importation Azure, consultez [Voir l’état des tâches Azure Import/Export](../storage/common/storage-import-export-view-drive-status.md).

### <a name="finish-the-workflow"></a>Terminer le workflow

Une fois le travail d’importation terminé, les données de sauvegarde initiale sont disponibles dans votre compte de stockage. Lors de la sauvegarde planifiée suivante, Sauvegarde Azure copie le contenu des données à partir du compte de stockage vers le coffre Recovery Services.

   ![Copier les données vers le coffre Recovery Services](./media/offline-backup-dpm-mabs-previous-versions/copying-from-storage-account-to-azure-backup.png)<br/>

Lors de la sauvegarde planifiée suivante, Sauvegarde Azure effectue une sauvegarde incrémentielle par-dessus la copie de sauvegarde initiale.

## <a name="next-steps"></a>Étapes suivantes

* Pour toute question concernant le workflow du service Azure Import/Export, consultez [Utilisation du service Microsoft Azure Import/Export pour transférer des données vers Stockage Blob](../storage/common/storage-import-export-service.md).
