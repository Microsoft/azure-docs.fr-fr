---
title: Sauvegarde hors connexion avec Azure Data Box
description: Découvrez comment vous pouvez utiliser Azure Data Box pour amorcer des données de sauvegarde initiales volumineuses hors connexion à partir de l’agent MARS dans un coffre Recovery Services.
ms.topic: conceptual
ms.date: 1/27/2020
ms.openlocfilehash: e789b6c9f4ff2e8cd168e6b5c138d423911d4743
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752581"
---
# <a name="azure-backup-offline-backup-by-using-azure-data-box"></a>Sauvegarde hors connexion Sauvegarde Microsoft Azure avec Azure Data Box

Vous pouvez utiliser [Azure Data Box](../databox/data-box-overview.md) pour amorcer vos sauvegardes Microsoft Azure Recovery Services (MARS) initiales volumineuses hors connexion (sans utiliser de réseau) dans un coffre Recovery Services. Ce processus permet de gagner du temps et d’économiser de la bande passante réseau, qui serait sinon consommée pour déplacer de grandes quantités de données de sauvegarde en ligne sur un réseau à latence élevée. Cette amélioration est actuellement en préversion. La sauvegarde hors connexion basée sur Azure Data Box offre deux avantages distincts par rapport à la [sauvegarde hors connexion basée sur le service Azure Import/Export](./backup-azure-backup-import-export.md) :

- Vous n’avez pas besoin de fournir vos propres disques et connecteurs compatibles avec Azure. Azure Data Box fournit les disques associés à la [référence SKU Data Box](https://azure.microsoft.com/services/databox/data/) sélectionnée.
- Le service Sauvegarde Azure (agent MARS) peut écrire directement des données de sauvegarde sur les références SKU prises en charge d’Azure Data Box. Grâce à cette capacité, vous n’avez plus besoin d’approvisionner un emplacement intermédiaire pour vos données de sauvegarde initiales. Vous n’avez pas non plus besoin d’utilitaires pour formater et copier ces données sur les disques.

## <a name="azure-data-box-with-the-mars-agent"></a>Azure Data Box avec l’agent MARS

Cet article explique comment vous pouvez utiliser Azure Data Box pour amorcer des données de sauvegarde initiales volumineuses hors connexion à partir de l’agent MARS dans un coffre Recovery Services.

## <a name="supported-platforms"></a>Plateformes prises en charge

Le processus d’amorçage des données à partir de l’agent MARS avec Azure Data Box est pris en charge sur les références SKU Windows suivantes.

| **SE**                                 | **Référence (SKU)**                                                      |
| -------------------------------------- | ------------------------------------------------------------ |
| **Station de travail**                        |                                                              |
| Windows 10 64 bits                     | Entreprise, Professionnel, Familiale                                       |
| Windows 8.1 64 bits                    | Entreprise, Professionnel                                             |
| Windows 8 64 bits                      | Entreprise, Professionnel                                             |
| Windows 7 64 bits                      | Édition Intégrale, Entreprise, Professionnel, Édition Familiale Premium, Édition Familiale Basique, Édition Starter |
| **Serveur**                             |                                                              |
| Windows Server 2019 64 bits            | Standard, Datacenter, Essentials                            |
| Windows Server 2016 64 bits            | Standard, Datacenter, Essentials                            |
| Windows Server 2012 R2 64 bits         | Standard, Datacenter, Foundation                            |
| Windows Server 2012 64 bits            | Datacenter, Foundation, Standard                            |
| Windows Storage Server 2016 64 bits    | Standard, Workgroup                                         |
| Windows Storage Server 2012 R2 64 bits | Standard, Workgroup, Essential                              |
| Windows Storage Server 2012 64 bits    | Standard, Workgroup                                         |
| Windows Server 2008 R2 SP1 64 bits     | Standard, Entreprise, Datacenter, Foundation                |
| Windows Server 2008 SP2 64 bits        | Standard, Entreprise, Datacenter                            |

## <a name="backup-data-size-and-supported-data-box-skus"></a>Taille des données de sauvegarde et SKU Data Box prises en charge

| Taille des données de sauvegarde (après compression par MARS)* par serveur | SKU Azure Data Box pris en charge                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <= 7,2 To                                                    | [Disque Azure Data Box](../databox/data-box-disk-overview.md) |
| > 7,2 To et <= 80 To**                                      | [Azure Data Box (100 To)](../databox/data-box-overview.md) |

* Les taux de compression habituels varient entre 10 et 20 %. <br>
** Contactez [AskAzureBackupTeam@microsoft.com](mailto:AskAzureBackupTeam@microsoft.com) si vous prévoyez plus de 80 To de données de sauvegarde initiales pour un seul serveur MARS.

>[!IMPORTANT]
>Les données de sauvegarde initiales provenant d’un serveur unique doivent se trouver sur une seule instance Azure Data Box ou sur un seul disque Azure Data Box. Il n’est pas possible de les partager entre plusieurs appareils, que leur SKU soit identique ou différente. En revanche, un appareil Azure Data Box peut contenir des sauvegardes initiales provenant de plusieurs serveurs.

## <a name="prerequisites"></a>Prérequis

### <a name="azure-subscription-and-required-permissions"></a>Abonnement Azure et autorisations nécessaires

- Le processus nécessite un abonnement Azure.
- Le processus exige que l’utilisateur désigné pour exécuter la stratégie de sauvegarde hors connexion soit propriétaire de l’abonnement Azure.
- Le travail Data Box et le coffre Recovery Services (dans lequel les données doivent être amorcées) ont l’obligation de se trouver dans le même abonnement.
- Nous vous recommandons de situer le compte de stockage cible associé au travail Azure Data Box et le coffre Recovery Services dans la même région. Toutefois, cela n’est pas une obligation.

### <a name="get-azure-powershell-370"></a>Obtenir Azure PowerShell 3.7.0

*Il s’agit du prérequis le plus important pour le processus*. Avant d’installer Azure PowerShell (version 3.7.0), effectuez les vérifications suivantes.

#### <a name="step-1-check-the-powershell-version"></a>Étape 1 : Vérifier la version de PowerShell

1. Ouvrez Windows PowerShell et exécutez la commande suivante :

    ```powershell
    Get-Module -ListAvailable AzureRM*
    ```

1. Si la sortie indique une version supérieure à 3.7.0, effectuez l’étape 2. Sinon, passez à l’étape 3.

#### <a name="step-2-uninstall-the-powershell-version"></a>Étape 2 : Désinstaller la version de PowerShell

Désinstallez la version actuelle de PowerShell.

1. Supprimez les modules dépendants en exécutant la commande suivante dans PowerShell :

    ```powershell
    foreach ($module in (Get-Module -ListAvailable AzureRM*).Name |Get-Unique)  { write-host "Removing Module $module" Uninstall-module $module }
    ```

2. Pour garantir la suppression correcte de tous les modules dépendants, exécutez la commande suivante :

    ```powershell
    Get-Module -ListAvailable AzureRM*
    ```

#### <a name="step-3-install-powershell-version-370"></a>Étape 3 : Installer PowerShell version 3.7.0

Après avoir vérifié qu’aucun module AzureRM n’est présent, installez la version 3.7.0 à l’aide de l’une des méthodes suivantes :

- À partir de GitHub, utilisez [ce lien](https://github.com/Azure/azure-powershell/releases/tag/v3.7.0-March2017).

Vous pouvez également :

- Exécutez la commande suivante dans la fenêtre PowerShell :

    ```powershell
    Install-Module -Name AzureRM -RequiredVersion 3.7.0
    ```

Azure PowerShell peut également avoir été installé à l’aide d’un fichier msi. Pour le supprimer, désinstallez-le à l’aide de l’option **Désinstallation des programmes** dans le Panneau de configuration.

### <a name="order-and-receive-the-data-box-device"></a>Commander et recevoir l’appareil Data Box

Le processus de sauvegarde hors connexion avec MARS et Azure Data Box nécessite que l’état des appareils Data Box soit « Livré » pour pouvoir déclencher la sauvegarde hors connexion à l’aide de l’agent MARS. Pour commander la SKU qui répond le mieux à vos besoins, reportez-vous à [Taille des données de sauvegarde et SKU Data Box prises en charge](#backup-data-size-and-supported-data-box-skus). Suivez les étapes décrites dans [Tutoriel : Commander un disque Azure Data Box Disk](../databox/data-box-disk-deploy-ordered.md) pour commander et recevoir vos appareils Data Box.

> [!IMPORTANT]
> Ne sélectionnez pas *BlobStorage* pour le **type de compte**. L’agent MARS exige un compte qui prend en charge les objets blob de pages, ce qui n’est pas possible quand *BlobStorage* est sélectionné. Sélectionnez le **type de compte** **Stockage V2 (v2 universel)** lors de la création du compte de stockage cible pour votre travail Azure Data Box.

![Choisir un type de compte dans les détails de l’instance](./media/offline-backup-azure-data-box/instance-details.png)

## <a name="install-and-set-up-the-mars-agent"></a>Installer et configurer l’agent MARS

1. Veillez à désinstaller toutes les installations précédentes de l’agent MARS.
1. Téléchargez l’agent MARS le plus récent sur [ce site](https://aka.ms/azurebackup_agent).
1. Exécutez *MARSAgentInstaller.exe* et effectuez *uniquement* les étapes nécessaires pour [installer et inscrire l’agent](./install-mars-agent.md#install-and-register-the-agent) dans le coffre Recovery Services dans lequel vous voulez stocker vos sauvegardes.

   > [!NOTE]
   > Le coffre Recovery Services doit se trouver dans le même abonnement que le travail Azure Data Box.

   Une fois que l’agent est inscrit dans le coffre Recovery Services, suivez les étapes décrites dans les sections suivantes.

## <a name="set-up-azure-data-box-devices"></a>Configurer des appareils Azure Data Box

Selon la référence SKU Azure Data Box que vous avez commandée, effectuez les étapes décrites dans les sections appropriées qui suivent. Les étapes vous montrent comment configurer et préparer les appareils Data Box pour l’agent MARS afin d’identifier et de transférer les données de sauvegarde initiales.

### <a name="set-up-azure-data-box-disks"></a>Configurer des disques Azure Data Box

Si vous avez commandé un ou plusieurs disques Azure Data Box (jusqu’à 8 To chacun), suivez les étapes mentionnées ici pour [déballer, connecter et déverrouiller votre disque Data Box Disk](../databox/data-box-disk-deploy-set-up.md).

>[!NOTE]
>Il est possible que le serveur doté de l’agent MARS ne dispose d’aucun port USB. Dans ce cas, vous pouvez connecter votre disque Azure Data Box Disk à un autre serveur ou client et exposer la racine de l’appareil en tant que partage réseau.

### <a name="set-up-azure-data-box"></a>Configurer Azure Data Box

Si vous avez commandé une instance Azure Data Box (jusqu’à 100 To), suivez les étapes mentionnées ici [pour configurer votre instance Data Box](../databox/data-box-deploy-set-up.md).

#### <a name="mount-your-azure-data-box-instance-as-a-local-system"></a>Monter votre instance Azure Data Box en tant que système local

L’agent MARS fonctionne dans le contexte du système local : il est donc nécessaire de fournir le même niveau de privilège au chemin de montage où l’instance Azure Data Box est connectée.

Pour vous assurer de pouvoir monter votre appareil Data Box en tant que système local à l’aide du protocole NFS :

1. Activez la fonctionnalité Client pour NFS sur le serveur Windows sur lequel l’agent MARS est installé. Spécifiez l’autre source *WIM:D:\Sources\Install.wim:4*.
1. Téléchargez PsExec à partir de la page [Sysinternals](/sysinternals/downloads/psexec) sur le serveur sur lequel l’agent MARS est installé.
1. Ouvrez une invite de commandes avec élévation de privilèges et exécutez la commande suivante avec le répertoire contenant *PSExec.exe* comme répertoire actif.

    ```cmd
    psexec.exe  -s  -i  cmd.exe
    ```

   La fenêtre de commande qui s’ouvre en raison de la commande précédente se trouve dans le contexte du système local. Utilisez cette fenêtre de commande pour exécuter les étapes de montage du partage d’objets blob de pages Azure en tant que lecteur réseau sur votre serveur Windows.
1. Suivez les étapes décrites dans [Se connecter à Data Box](../databox/data-box-deploy-copy-data-via-nfs.md#connect-to-data-box) pour connecter votre serveur doté de l’agent MARS à l’appareil Data Box via NFS. Exécutez la commande suivante dans l’invite de commandes du système local pour monter le partage d’objets blob de pages Azure.

    ```cmd
    mount -o nolock \\<DeviceIPAddress>\<StorageAccountName_PageBlob X:  
    ```

   Une fois le partage monté, vérifiez si vous pouvez accéder à X: à partir de votre serveur. Si vous le pouvez, passez à la section suivante de cet article.

## <a name="transfer-initial-backup-data-to-azure-data-box-devices"></a>Transférer les données de sauvegarde initiales aux appareils Azure Data Box

1. Ouvrez l’application **Sauvegarde Microsoft Azure** sur votre serveur.
1. Dans le volet **Actions**, sélectionnez **Planifier la sauvegarde**.

    ![Sélectionner Planifier la sauvegarde](./media/offline-backup-azure-data-box/schedule-backup.png)

1. Suivez les étapes de l’**Assistant Planifier la sauvegarde**.

1. Ajoutez des éléments en sélectionnant le bouton **Ajouter des éléments**. Maintenez la taille totale des éléments dans les [limites de taille prises en charge par la SKU Azure Data Box](#backup-data-size-and-supported-data-box-skus) que vous avez commandée et reçue.

    ![Ajouter des éléments à sauvegarder](./media/offline-backup-azure-data-box/add-items.png)

1. Sélectionnez la planification de sauvegarde et la stratégie de rétention appropriées pour **Fichiers et dossiers** et **État du système**. L’état du système est applicable uniquement pour les serveurs Windows et non pour les clients Windows.
1. Dans la page **Choisir le type de sauvegarde initiale (Fichiers et dossiers)** de l’Assistant, sélectionnez l’option **Transférer à l’aide de disques Microsoft Azure Data Box**, puis sélectionnez **Suivant**.

    ![Choisir un type de sauvegarde initiale](./media/offline-backup-azure-data-box/initial-backup-type.png)

1. Connectez-vous à Azure lorsque vous y êtes invité à l’aide des informations d’identification d’utilisateur qui disposent d’un accès Propriétaire à l’abonnement Azure. Une fois que vous avez effectué cette opération, une page semblable à celle-ci doit s’afficher.

    ![Créer des ressources et appliquer les autorisations requises](./media/offline-backup-azure-data-box/creating-resources.png)

   L’agent MARS récupère ensuite les travaux Data Box inclus dans l’abonnement dont l’état est Livré.

    ![Récupérer les travaux Data Box de l’ID d’abonnement](./media/offline-backup-azure-data-box/fetching-databox-jobs.png)

1. Sélectionnez la commande Data Box appropriée pour laquelle vous avez déballé, connecté et déverrouillé votre disque Data Box. Sélectionnez **Suivant**.

    ![Sélectionner les commandes Data Box](./media/offline-backup-azure-data-box/select-databox-order.png)

1. Sélectionnez **Détecter l’appareil** sur la page **Détection de l’appareil Data Box**. Cette action permet à l’agent MARS d’analyser les disques Azure Data Box attachés localement et de les détecter.

    ![Détection des appareils Data Box](./media/offline-backup-azure-data-box/databox-device-detection.png)

    Si vous avez connecté l’instance Azure Data Box en tant que partage réseau (en raison de l’indisponibilité des ports USB ou parce que vous avez commandé et monté l’appareil Data Box de 100 To), la détection échoue dans un premier temps. Vous avez la possibilité d’entrer le chemin réseau de l’appareil Data Box.

    ![Entrer le chemin réseau](./media/offline-backup-azure-data-box/enter-network-path.png)

    >[!IMPORTANT]
    > Indiquez le chemin réseau du répertoire racine du disque Azure Data Box. Ce répertoire doit contenir un répertoire portant le nom *PageBlob*.
    >
    >![Répertoire racine du disque Azure Data Box](./media/offline-backup-azure-data-box/root-directory.png)
    >
    >Par exemple, si le chemin du disque est `\\mydomain\myserver\disk1\` et si *disk1* contient un répertoire appelé *PageBlob*, le chemin à entrer dans l’Assistant de l’agent MARS est `\\mydomain\myserver\disk1\`.
    >
    >Si vous [configurez un appareil Azure Data Box de 100 To](#set-up-azure-data-box-devices), entrez `\\<DeviceIPAddress>\<StorageAccountName>_PageBlob` comme chemin réseau de l’appareil.

1. Sélectionnez **Suivant**, puis **Terminer** sur la page suivante pour enregistrer la stratégie de sauvegarde et de rétention avec la configuration de sauvegarde hors connexion à l’aide d’Azure Data Box.

   La page suivante confirme que la stratégie est correctement enregistrée.

    ![La stratégie est correctement enregistrée](./media/offline-backup-azure-data-box/policy-saved.png)

1. Sélectionnez **Fermer** sur la page précédente.

1. Sélectionnez **Sauvegarder maintenant** dans le volet **Actions** de la console de l’agent MARS. Sélectionnez **Sauvegarder** sur la page de l’Assistant.

    ![Assistant Sauvegarder maintenant](./media/offline-backup-azure-data-box/backup-now.png)

L’agent MARS commence à sauvegarder les données que vous avez sélectionnées sur l’appareil Azure Data Box. Ce processus peut prendre de quelques heures à quelques jours. La durée dépend du nombre de fichiers et de la vitesse de connexion entre le serveur sur lequel est installé l’agent MARS et le disque Azure Data Box.

Une fois la sauvegarde des données terminée, vous verrez une page sur l’agent MARS qui ressemble à celle-ci.

![Progression de la sauvegarde indiquée](./media/offline-backup-azure-data-box/backup-progress.png)

## <a name="post-backup-steps"></a>Étapes à l’issue de la sauvegarde

Cette section explique les étapes à suivre une fois que la sauvegarde des données a été correctement effectuée sur le disque Azure Data Box.

- Suivez les étapes décrites dans cet article pour [expédier le disque Azure Data Box à Azure](../databox/data-box-disk-deploy-picked-up.md). Si vous avez utilisé un appareil Azure Data Box de 100 To, effectuez ces étapes pour [expédier l’appareil Azure Data Box à Azure](../databox/data-box-deploy-picked-up.md).

- [Supervisez le travail Data Box](../databox/data-box-disk-deploy-upload-verify.md) dans le portail Azure. Une fois le travail Azure Data Box terminé, l’agent MARS déplace automatiquement les données du compte de stockage vers le coffre Recovery Services au moment de la sauvegarde planifiée suivante. Le travail de sauvegarde est alors marqué *Travail effectué* si un point de récupération a correctement été créé.

    >[!NOTE]
    >L’agent MARS déclenche des sauvegardes aux heures planifiées pendant la création de la stratégie. Ces travaux indiquent « En attente de la fin du travail Azure Data Box » tant que le travail à l’heure prévue n’est pas terminé.

- Une fois que l’agent MARS a réussi à créer un point de récupération correspondant à la sauvegarde initiale, vous pouvez supprimer le compte de stockage ou son contenu spécifique associé au travail Azure Data Box.

## <a name="troubleshooting"></a>Dépannage

L’agent Microsoft Azure Recovery Services (MARS) crée une application Azure Active Directory (Azure AD) pour vous dans votre locataire. Cette application nécessite un certificat à des fins d’authentification. Celui-ci est créé et chargé lors de la configuration d’une stratégie d’amorçage hors connexion. Nous utilisons Azure PowerShell pour créer et charger le certificat sur l’application Azure AD.

### <a name="problem"></a>Problème

Lorsque vous configurez la sauvegarde hors connexion, vous risquez de rencontrer un problème en raison d’un bogue dans la cmdlet Azure PowerShell. Il se peut que vous ne puissiez pas ajouter plusieurs certificats à la même application Azure AD créée par l’agent MAB. Ce problème vous concerne si vous avez configuré une stratégie d’amorçage hors connexion pour le même serveur ou pour un autre serveur.

### <a name="verify-if-the-problem-is-caused-by-this-specific-root-cause"></a>Vérifier si le problème est due à cette cause racine spécifique

Pour voir si votre problème est le même que celui décrit précédemment, effectuez l’une des étapes suivantes.

#### <a name="step-1-of-verification"></a>Étape 1 de la vérification

Vérifiez si le message d’erreur suivant s’affiche dans la console MAB lorsque vous avez configuré la sauvegarde hors connexion.

![Impossible de créer une stratégie de sauvegarde hors connexion pour le compte Azure actuel](./media/offline-backup-azure-data-box/unable-to-create-policy.png)

#### <a name="step-2-of-verification"></a>Étape 2 de la vérification

1. Ouvrez le dossier **Temp** dans le chemin d’installation. Le chemin d’accès au dossier temporaire par défaut est *C:\Program Files\Microsoft Azure Recovery Services Agent\Temp*. Recherchez le fichier *CBUICurr* et ouvrez-le.

1. Dans le fichier *CBUICurr*, faites défiler jusqu’à la dernière ligne et vérifiez si le problème est le même que celui du message d’erreur suivant : `Unable to create an Azure AD application credential in customer's account. Exception: Update to existing credential with KeyId <some guid> is not allowed`.

### <a name="workaround"></a>Solution de contournement

Pour résoudre ce problème, effectuez les étapes suivantes et réessayez de configurer la stratégie.

#### <a name="step-1-of-workaround"></a>Étape 1 de la solution de contournement

Connectez-vous à PowerShell qui apparaît dans l’interface utilisateur de Sauvegarde Microsoft Azure avec un autre compte doté d’un accès administrateur à l’abonnement pour lequel le travail Data Box est créé.

#### <a name="step-2-of-workaround"></a>Étape 2 de la solution de contournement

Si aucun autre serveur n’a d’amorçage hors connexion configuré et si aucun autre serveur ne dépend de l’application `AzureOfflineBackup_<Azure User Id>`, supprimez-la. Sélectionnez **Portail Azure** > **Azure Active Directory** > **Inscriptions d’applications**.

>[!NOTE]
> Vérifiez si l’application `AzureOfflineBackup_<Azure User Id>` n’a pas d’autre amorçage hors connexion configuré et si aucun autre serveur ne dépend de cette application. Accédez à **Paramètres** > **Clés** sous la section **Clés publiques**. Elle ne doit pas avoir d’autres clés publiques ajoutées. Examinez la capture d’écran suivante à des fins de référence.
>
>![Clés publiques](./media/offline-backup-azure-data-box/public-keys.png)

#### <a name="step-3"></a>Étape 3 :

À partir du serveur sur lequel vous essayez de configurer la sauvegarde hors connexion, réalisez les actions suivantes.

1. Accédez à l’onglet **Gérer l’application de certificat d’ordinateur** > **Personnel** et recherchez le certificat portant le nom `CB_AzureADCertforOfflineSeeding_<ResourceId>`.

2. Sélectionnez le certificat, cliquez avec le bouton droit sur **Toutes les tâches** et sélectionnez **Exporter** sans clé privée au format .cer.

3. Accédez à l’application de sauvegarde hors connexion Azure mentionnée à l’étape 2. Sélectionnez **Paramètres** > **Clés** > **Charger la clé publique**. Chargez le certificat que vous avez exporté à l’étape précédente.

    ![Charger une clé publique](./media/offline-backup-azure-data-box/upload-public-key.png)

4. Sur le serveur, ouvrez le registre en entrant **regedit** dans la fenêtre d’exécution.

5. Accédez au Registre *Ordinateur\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config\CloudBackupProvider.* Cliquez avec le bouton droit sur **CloudBackupProvider** et ajoutez une nouvelle valeur de chaîne portant le nom `AzureADAppCertThumbprint_<Azure User Id>`.

    >[!NOTE]
    > Pour récupérer l’identifiant utilisateur Azure, effectuez l’une des actions suivantes :
    >
    >- À partir du PowerShell connecté à Azure, exécutez la commande `Get-AzureRmADUser -UserPrincipalName "Account Holder's email as defined in the portal"`.
    > - Accédez au chemin du Registre `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\DbgSettings\OnlineBackup` portant le nom *CurrentUserId*.

6. Cliquez avec le bouton droit sur la chaîne ajoutée à l’étape précédente, puis sélectionnez **Modifier**. Dans la valeur, fournissez l’empreinte du certificat que vous avez exporté à l’étape 2. Sélectionnez **OK**.

7. Pour afficher la valeur de l’empreinte, double-cliquez sur le certificat. Sélectionnez l’onglet **Détails**, puis faites défiler jusqu’à ce que la champ Empreinte s’affiche. Sélectionnez **Empreinte** et copiez la valeur.

    ![Champ Empreinte du certificat](./media/offline-backup-azure-data-box/thumbprint-field.png)

## <a name="questions"></a>Questions

Pour toute question ou demande de clarification concernant les problèmes auxquels vous êtes confronté, contactez [AskAzureBackupTeam@microsoft.com](mailto:AskAzureBackupTeam@microsoft.com).
