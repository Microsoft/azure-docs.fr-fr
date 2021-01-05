---
title: Sauvegarde hors connexion avec Azure Data Box pour DPM et MABS
description: Vous pouvez utiliser Azure Data Box pour amorcer des données de sauvegarde initiales hors connexion à partir de DPM et MABS.
ms.topic: conceptual
ms.date: 08/12/2020
ms.openlocfilehash: 1cfd9131099ad6a8ccd3d43e93f3d97641514f03
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96752547"
---
# <a name="offline-seeding-using-azure-data-box-for-dpm-and-mabs-preview"></a>Amorçage hors connexion à l’aide d’Azure Data Box pour DPM et MABS (préversion)

> [!NOTE]
> Cette fonctionnalité s’applique à Data Protection Manager (DPM) 2019 UR2 et versions ultérieures.<br><br>
> Cette fonctionnalité est actuellement disponible en préversion pour Serveur de sauvegarde Microsoft Azure (MABS). Si vous souhaitez utiliser Azure Data Box pour l’amorçage hors connexion avec MABS, contactez-nous à l’adresse [systemcenterfeedback@microsoft.com](mailto:systemcenterfeedback@microsoft.com).

Cet article explique comment vous pouvez utiliser Azure Data Box pour amorcer des données de sauvegarde initiales hors connexion de DPM et MABS dans un coffre Azure Recovery Services.

Vous pouvez utiliser [Azure Data Box](../databox/data-box-overview.md) pour amorcer vos sauvegardes DPM/MABS initiales volumineuses hors connexion (sans utiliser de réseau) dans un coffre Recovery Services. Ce processus permet de gagner du temps et d’économiser de la bande passante réseau, qui serait sinon consommée pour déplacer de grandes quantités de données de sauvegarde en ligne sur un réseau à latence élevée. Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire.

La sauvegarde hors connexion basée sur Azure Data Box offre deux avantages distincts par rapport à la [sauvegarde hors connexion basée sur le service Azure Import/Export](backup-azure-backup-server-import-export.md) :

- Vous n’avez pas besoin de fournir vos propres disques et connecteurs compatibles avec Azure. Azure Data Box fournit les disques associés à la [référence SKU Data Box](https://azure.microsoft.com/services/databox/data/) sélectionnée.

- Le service Sauvegarde Azure (agent MARS) peut écrire directement des données de sauvegarde sur les références SKU prises en charge d’Azure Data Box. Grâce à cette capacité, vous n’avez plus besoin d’approvisionner un emplacement intermédiaire pour vos données de sauvegarde initiales. Vous n’avez pas non plus besoin d’utilitaires pour formater et copier ces données sur les disques.

## <a name="supported-platforms"></a>Plateformes prises en charge

Les plateformes suivantes sont prises en charge :

- Windows Server 2019 64 bits (Standard, Datacenter, Essentials)
- Windows Server 2016 64 bits (Standard, Datacenter, Essentials)

## <a name="backup-data-size-and-supported-data-box-skus"></a>Taille des données de sauvegarde et SKU Data Box prises en charge

Les SKU Data Box suivants sont pris en charge :

| Taille des données de sauvegarde (après compression par MARS)\* par serveur | SKU Azure Data Box pris en charge |
| --- | --- |
| \<= 7,2 To | [Azure Data Box Disk](../databox/data-box-disk-overview.md) |
| > 7,2 To et <= 80 To\*\* | [Azure Data Box (100 To)](../databox/data-box-overview.md) |

\*Les taux de compression habituels varient entre 10 et 20 % <br>
\*\*Contactez [SystemCenterFeedback@microsoft.com](mailto:SystemCenterFeedback@microsoft.com) si vous prévoyez plus de 80 To de données de sauvegarde initiales pour une seule source de données.

> [!IMPORTANT]
> Les données de sauvegarde initiales provenant d’une source de données unique doivent se trouver sur un seul appareil Azure Data Box ou Azure Data Box Disk. Il n’est pas possible de les partager entre plusieurs appareils de SKU identiques ou différents. En revanche, Azure Data Box peut contenir des sauvegardes initiales provenant de plusieurs sources de données.

## <a name="before-you-begin"></a>Avant de commencer

L’agent MARS exécuté sur DPM/MABS doit être mis à niveau vers la [dernière version](https://aka.ms/azurebackup_agent) (2.0.9171.0 ou version ultérieure).

Vérifiez les points suivants :

### <a name="azure-subscription-and-required-permissions"></a>Abonnement Azure et autorisations nécessaires

- Un abonnement Azure valide.
- L’utilisateur désigné pour exécuter la stratégie de sauvegarde hors connexion doit être propriétaire de l’abonnement Azure.
- Le travail Data Box et le coffre Recovery Services (dans lequel les données doivent être amorcées) doivent obligatoirement être disponibles dans le même abonnement.
    > [!NOTE]
    > Nous vous recommandons de situer le compte de stockage cible et le coffre Recovery Services dans la même région. Cependant, cela n’est pas obligatoire.

### <a name="order-and-receive-the-data-box-device"></a>Commander et recevoir l’appareil Data Box

Assurez-vous que les appareils Data Box requis sont dans l’état *Livré* avant de déclencher la sauvegarde hors connexion. Pour commander le SKU qui répond le mieux à vos besoins, reportez-vous à [Taille des données de sauvegarde et SKU Data Box pris en charge](#backup-data-size-and-supported-data-box-skus). Suivez les étapes décrites dans [cet article](../databox/data-box-disk-deploy-ordered.md) pour commander et recevoir vos appareils Data Box.

> [!IMPORTANT]
> Ne sélectionnez pas *BlobStorage* pour le **type de compte**. Le serveur DPM/MABS exige un compte qui prend en charge les objets blob de pages, ce qui n’est pas possible quand *BlobStorage* est sélectionné. Sélectionnez le **type de compte** **Stockage V2 (v2 universel)** lors de la création du compte de stockage cible pour votre travail Azure Data Box.

![Configurer Azure Data Box](./media/offline-backup-azure-data-box-dpm-mabs/setup-azure-databox.png)

## <a name="setup-azure-data-box-devices"></a>Configurer un ou plusieurs appareils Azure Data Box

Quand vous recevez l’appareil Azure Data Box, selon la référence SKU Azure Data Box que vous avez commandée, effectuez les étapes décrites dans les sections appropriées ci-dessous pour configurer et préparer les appareils Data Box afin de permettre au serveur DPM/MABS d’identifier et de transférer les données de sauvegarde initiales.

### <a name="setup-azure-data-box-disk"></a>Configurer Azure Data Box Disk

Si vous avez commandé un ou plusieurs disques Azure Data Box (jusqu’à 8 To chacun), suivez les étapes mentionnées ici pour [déballer, connecter et déverrouiller votre disque Data Box Disk](../databox/data-box-disk-deploy-set-up.md).

> [!NOTE]
> Il est possible que le serveur DPM/MABS ne dispose pas d’un port USB. Dans cette situation, vous pouvez connecter votre disque Azure Data Box Disk à un autre serveur/client et exposer la racine de l’appareil en tant que partage réseau.

## <a name="setup-azure-data-box"></a>Configurer Azure Data Box

Si vous avez commandé un disque Azure Data Box (jusqu’à 100 To), suivez les étapes mentionnées ici [pour configurer votre disque Data Box](../databox/data-box-deploy-set-up.md).

### <a name="mount-your-azure-data-box-as-local-system"></a>Monter votre disque Azure Data Box en tant que système local

Le serveur DPM/MABS fonctionne dans le contexte du système local et nécessite donc de fournir le même niveau de privilège au chemin de montage où le disque Azure Data Box est connecté. Suivez les étapes ci-dessous pour vérifier que vous pouvez monter votre appareil Data Box en tant que système local à l’aide du protocole NFS.

1. Activez la fonctionnalité Client pour NFS sur le serveur DPM/MABS.
Spécifiez une autre source : *WIM:D:\Sources\Install.wim:4*
2. Téléchargez **PSExec** à partir de [https://download.sysinternals.com/files/PSTools.zip](https://download.sysinternals.com/files/PSTools.zip) sur le serveur DPM/MABS.
3. Ouvrez une invite de commandes avec élévation de privilèges et exécutez la commande suivante avec le répertoire contenant *PSExec.exe* comme répertoire actif.

   ```cmd
   psexec.exe  -s  -i  cmd.exe
   ```

4. La fenêtre de commande qui s’ouvre à l’issue de la commande ci-dessus se trouve dans le contexte du système local. Utilisez cette fenêtre de commande pour exécuter les étapes de montage du partage d’objets blob de pages Azure en tant que lecteur réseau sur votre serveur Windows.
5. Suivez les étapes décrites [ici](../databox/data-box-deploy-copy-data-via-nfs.md#connect-to-data-box) pour connecter votre serveur DPM/MABS à l’appareil Data Box par le biais de NFS, puis exécutez la commande suivante à l’invite de commandes du système local pour monter le partage d’objets blob de pages Azure :

    ```cmd
    mount -o nolock \\<DeviceIPAddres>\<StorageAccountName_PageBlob X:
    ```

6. Une fois monté, vérifiez si vous pouvez accéder à X: à partir de votre serveur. Si vous le pouvez, passez à la section suivante de cet article.

## <a name="transfer-initial-backup-data-to-azure-data-box-devices"></a>Transférer les données de sauvegarde initiales aux appareils Azure Data Box

1. Sur votre serveur DPM/MABS, suivez les étapes pour [créer un groupe de protection](/system-center/dpm/create-dpm-protection-groups). Si vous ajoutez une protection en ligne au groupe de protection existant, cliquez avec le bouton droit sur le groupe de protection existant, puis sélectionnez **Ajouter une de protection en ligne** et commencez à partir de **l’étape 8**.
2. Dans la page **Sélectionner les membres du groupe**, spécifiez les ordinateurs et sources que vous souhaitez sauvegarder.
3. Dans la page **Sélectionner la méthode de protection des données**, spécifiez la façon dont vous souhaitez gérer les sauvegardes à court terme et à long terme. Veillez à sélectionner **Je voudrais une protection en ligne.**

   ![Création d'un nouveau groupe de protection](./media/offline-backup-azure-data-box-dpm-mabs/create-new-protection-group.png)

4. Dans la page **Sélectionner les objectifs à court terme**, spécifiez la manière dont vous voulez sauvegarder dans le stockage à court terme sur disque.
5. Dans la page **Vérifier l’allocation de disque**, vérifiez l’espace disque de pool de stockage alloué pour le groupe de protection.
6. Dans la page **Choisir la méthode de création de réplica**, sélectionnez **Automatiquement sur le réseau.**
7. Dans la page **Sélectionner les options de vérification de cohérence**, indiquez comment vous voulez automatiser les vérifications de cohérence.
8. Dans la page **Indiquer les données de protection en ligne**, sélectionnez le membre pour lequel vous souhaitez activer la protection en ligne.

    ![Spécifier les données de protection en ligne](./media/offline-backup-azure-data-box-dpm-mabs/specify-online-protection-data.png)

9. Dans la page **Spécifier la planification de sauvegarde en ligne**, indiquez la fréquence à laquelle doivent avoir lieu les sauvegardes incrémentielles dans Azure.
10. Dans la page **Spécifier une stratégie de rétention en ligne**, vous pouvez préciser la façon dont les points de récupération créés à partir des sauvegardes quotidiennes, hebdomadaires, mensuelles et annuelles sont conservés dans Azure.
11. Dans l’écran **Choisir la réplication en ligne** de l’assistant, choisissez l’option **Transférer à l’aide de disques appartenant à Microsoft**, puis sélectionnez **Suivant**.

    ![Choisir la réplication en ligne initiale](./media/offline-backup-azure-data-box-dpm-mabs/choose-initial-online-replication.png)

    >[!NOTE]
    > L’option permettant de sélectionner **Transférer à l’aide de disques appartenant à Microsoft** n’est pas disponible pour MABS v3, car la fonctionnalité est en préversion. Contactez-nous à l’adresse [systemcenterfeedback@microsoft.com](mailto:systemcenterfeedback@microsoft.com) si vous souhaitez utiliser cette fonctionnalité pour MABS v3.

12. Connectez-vous à Azure lorsque vous y êtes invité à l’aide des informations d’identification d’utilisateur qui disposent d’un accès propriétaire à l’abonnement Azure. Une fois la connexion établie, l’écran suivant s’affiche :

    ![Une fois la connexion établie](./media/offline-backup-azure-data-box-dpm-mabs/after-successful-login.png)

     Le serveur DPM/MABS récupère ensuite les travaux Data Box inclus dans l’abonnement dont l’état est *Livré*.

     > [!NOTE]
     > La première connexion prend plus de temps que la normale. Le module Azure PowerShell est installé en arrière-plan, et l’application Azure AD est également inscrite.
     >
     >  - Les modules PowerShell suivants sont installés :<br>
          - AzureRM.Profile     *5.8.3*<br>
          - AzureRM.Resources   *6.7.3*<br>
          - AzureRM.Storage     *5.2.0*<br>
          - Azure.Storage       *4.6.1*<br>
     >  - L’application Azure AD est inscrite en tant que *AzureOfflineBackup_\<object GUID of the user>* .

13. Sélectionnez la commande Data Box appropriée pour laquelle vous avez déballé, connecté et déverrouillé votre disque Data Box. Sélectionnez **Suivant**.

    ![Sélectionner le Data Box](./media/offline-backup-azure-data-box-dpm-mabs/select-databox.png)

14. Dans l’écran **Détecter le Data Box**, entrez le chemin d’accès de votre appareil Data Box, puis sélectionnez **Détecter l’appareil**.

    ![Entrer le chemin d’accès réseau](./media/offline-backup-azure-data-box-dpm-mabs/enter-network-path.png)

    > [!IMPORTANT]
    > Indiquez le chemin réseau du répertoire racine du disque Azure Data Box. Ce répertoire doit contenir un répertoire portant le nom *PageBlob* comme illustré ci-dessous :
    >
    > ![Lecteur USB](./media/offline-backup-azure-data-box-dpm-mabs/usb-drive.png)
    >
    > Par exemple, si le chemin du disque est `\\mydomain\myserver\disk1\` et si *Disk1* contient un répertoire appelé *PageBlob*, le chemin à fournir dans le serveur DPM/MABS est `\\mydomain\myserver\disk1\`.
    > Si vous [configurez un appareil Azure Data Box de 100 To](./offline-backup-azure-data-box.md#set-up-azure-data-box), indiquez le chemin réseau d’appareil `\\<DeviceIPAddress>\<StorageAccountName>_PageBlob` suivant.

15. Sélectionnez **Suivant**. Dans la page **Résumé**, vérifiez vos paramètres puis sélectionnez **Créer un groupe**.

    ![Détecter le Data Box](./media/offline-backup-azure-data-box-dpm-mabs/detect-databox.png)

    L’écran suivant confirme que le groupe de protection a été créé avec succès.

    ![Groupe de protection créé](./media/offline-backup-azure-data-box-dpm-mabs/protection-group-created.png)

16. Sélectionnez **Fermer** dans l’écran ci-dessus.

    Après cela, la réplication initiale des données se produit sur le disque DPM/MABS. Une fois la protection terminée, l’état du groupe indique l’état de protection **OK** sur la page **Protection**.

17. Pour lancer la copie de sauvegarde hors connexion sur votre appareil Azure Data Box, faites un clic droit sur le **groupe de protection**, puis choisissez l’option **Créer un point de récupération**. Sélectionnez ensuite l’option **Protection en ligne** .

    ![Créer un point de récupération](./media/offline-backup-azure-data-box-dpm-mabs/create-recovery-point.png)

    ![Point de récupération](./media/offline-backup-azure-data-box-dpm-mabs/recovery-point.png)

   Le serveur DPM/MABS commence à sauvegarder les données que vous avez sélectionnées sur l’appareil Azure Data Box. Cette opération peut prendre quelques heures ou quelques jours selon le volume de données et la vitesse de connexion entre le serveur sur lequel est installé le serveur DPM/MABS et le disque Azure Data Box Disk.

   Vous pouvez surveiller l’état du travail dans le volet **Surveillance**. Une fois la sauvegarde des données terminée, un écran semblable à celui-ci apparaît :

   ![Console d'administration](./media/offline-backup-azure-data-box-dpm-mabs/administrator-console.png)

## <a name="post-backup-steps"></a>Étapes à l’issue de la sauvegarde

Procédez comme suit une fois que la sauvegarde des données sur Azure Data Box Disk est réussie.

- Suivez les étapes décrites dans cet article pour [expédier le disque Azure Data Box à Azure](../databox/data-box-disk-deploy-picked-up.md). Si vous avez utilisé un appareil Azure Data Box de 100 To, effectuez ces étapes pour [expédier le disque Azure Data Box à Azure](../databox/data-box-deploy-picked-up.md).
- [Supervisez le travail Data Box](../databox/data-box-disk-deploy-upload-verify.md) dans le portail Azure. Une fois le travail Azure Data Box *effectué*, le serveur DPM/MABS déplace automatiquement les données du compte de stockage vers le coffre Recovery Services au moment de la sauvegarde planifiée suivante. Le travail de sauvegarde est alors marqué comme *Travail effectué* si un point de récupération a correctement été créé.

  > [!NOTE]
  > Le serveur DPM/MABS déclenche les sauvegardes aux heures planifiées pendant la création du groupe de protection. Toutefois, ces travaux indiquent *En attente de la fin du travail Azure Data Box* tant que le travail n’est pas terminé.

- Une fois que le serveur DPM/MABS a réussi à créer un point de récupération correspondant à la sauvegarde initiale, vous pouvez supprimer le compte de stockage (ou son contenu spécifique) associé au travail Azure Data Box.

## <a name="troubleshooting"></a>Dépannage

L’agent Sauvegarde Microsoft Azure sur le serveur DPM crée une application Azure AD pour vous dans votre locataire. Cette application nécessite un certificat à des fins d’authentification. Celui-ci est créé et chargé lors de la configuration de la stratégie d’amorçage hors connexion.

Nous utilisons Azure PowerShell pour créer et charger le certificat sur l’application Azure AD.

### <a name="issue"></a>Problème

Au moment de la configuration de la sauvegarde hors connexion, en raison d’un bogue dans l’applet de commande Azure PowerShell, vous ne pouvez pas ajouter plusieurs certificats à la même application Azure AD que celle créée par l’agent Sauvegarde Microsoft Azure. Vous êtes concerné si vous avez configuré une stratégie d’amorçage hors connexion pour le même serveur ou un autre serveur.

### <a name="verify-if-the-issue-is-caused-by-this-specific-root-cause"></a>Vérifier si le problème est provoqué par cette cause racine spécifique

Pour vérifier que cette défaillance est due au [problème](#issue) ci-dessus, effectuez l’une des étapes suivantes :

#### <a name="step-1"></a>Étape 1

Vérifiez si vous voyez le message d’erreur suivant dans la console DPM/MABS au moment de la configuration de la sauvegarde hors connexion :

![Agent Azure Recovery Services](./media/offline-backup-azure-data-box-dpm-mabs/azure-recovery-services-agent.png)

#### <a name="step-2"></a>Étape 2

1. Ouvrez le dossier **temp** dans le chemin d’installation (le chemin du dossier temporaire par défaut est *C:\Program Files\Microsoft Azure Recovery Services Agent\Temp*. Recherchez le fichier *CBUICurr* et ouvrez-le.
2. Dans le fichier *CBUICurr*, faites défiler jusqu’à la dernière ligne et vérifiez si l’échec est dû à l’erreur « Impossible de créer des informations d’identification d’application Azure AD dans le compte du client. Exception : La mise à jour des informations d’identification existantes avec le KeyId \<some guid> n’est pas autorisée ».

### <a name="workaround"></a>Solution de contournement

Pour résoudre ce problème, effectuez les étapes suivantes et réessayez de configurer la stratégie.

1. Connectez-vous sur la page de connexion Azure qui apparaît dans l’interface utilisateur DPM/MABS avec un autre compte doté d’un accès administrateur à l’abonnement pour lequel le travail Data Box est créé.
2. Si aucun autre serveur n’a d’amorçage hors connexion configuré et ne dépend de l’application `AzureOfflineBackup_<Azure User Id>`, supprimez cette application depuis **Portail Azure > Azure Active Directory > Inscriptions d’applications**.

   > [!NOTE]
   > Vérifiez si l’application `AzureOfflineBackup_<Azure User Id>` n’a pas d’autre amorçage hors connexion configuré et si aucun autre serveur ne dépend de cette application. Accédez à **Paramètres >Clés** sous la section Clés publiques. Elle ne doit pas avoir d’autres **clés publiques** ajoutées. Examinez la capture d’écran suivante à des fins de référence :
   >
   > ![Clés publiques](./media/offline-backup-azure-data-box-dpm-mabs/public-keys.png)

#### <a name="step-3"></a>Étape 3 :

À partir du serveur DPM/MABS sur lequel vous essayez de configurer la sauvegarde hors connexion, réalisez les actions suivantes :

1. Ouvrez l’onglet **Gérer l’application de certificat d’ordinateur** > **Personnel** et recherchez le certificat portant le nom `CB_AzureADCertforOfflineSeeding_<ResourceId>`.
2. Sélectionnez le certificat ci-dessus, cliquez avec le bouton droit sur **Toutes les tâches** et **exportez**-le sans clé privée, au format .cer.
3. Accédez à l’application de sauvegarde hors connexion Azure mentionnée au **point 2**. Dans **Paramètres** > **Clés** > **Charger une clé publique**, chargez le certificat exporté à l’étape précédente.

   ![Charger des clés publiques](./media/offline-backup-azure-data-box-dpm-mabs/upload-public-keys.png)

4. Sur le serveur, ouvrez le Registre en tapant **regedit** dans la fenêtre **Exécuter**.
5. Accédez au Registre *Computer\HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Config\CloudBackupProvider*. Cliquez avec le bouton droit sur **CloudBackupProvider** et ajoutez une nouvelle valeur de chaîne portant le nom `AzureADAppCertThumbprint_<Azure User Id>`.

    >[!NOTE]
    > Pour récupérer l’identifiant utilisateur Azure, effectuez l’une des actions suivantes :
    >
    >- À partir du PowerShell connecté à Azure, exécutez la commande `Get-AzureRmADUser -UserPrincipalName "Account Holder's email as defined in the portal"`.
    > - Accédez au chemin du Registre `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\DbgSettings\OnlineBackup` portant le nom *CurrentUserId*.

6. Cliquez avec le bouton droit sur la chaîne ajoutée à l’étape ci-dessus, puis sélectionnez **Modifier**. Dans la valeur, fournissez l’empreinte du certificat que vous avez exporté au **point 2**, puis sélectionnez **OK**.
7. Pour obtenir la valeur de l’empreinte, double-cliquez sur le certificat, sélectionnez **Détails** et faites défiler le contenu jusqu’à ce que le champ de l’empreinte apparaisse. Sélectionnez **Empreinte** et copiez la valeur.

   ![Valeur de l’empreinte](./media/offline-backup-azure-data-box-dpm-mabs/certificate.png)

## <a name="next-steps"></a>Étapes suivantes

- [Amorçage hors connexion à l’aide de votre propre disque (à l’aide du service Azure Import/Export)](backup-azure-backup-server-import-export.md)
