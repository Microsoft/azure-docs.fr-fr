---
title: Résoudre les problèmes d’Azure File Sync | Microsoft Docs
description: Résolvez les problèmes courants d’un déploiement sur Azure File Sync, que vous pouvez utiliser pour transformer Windows Server en un cache rapide de votre partage de fichiers Azure.
author: jeffpatt24
ms.service: storage
ms.topic: troubleshooting
ms.date: 6/12/2020
ms.author: jeffpatt
ms.subservice: files
ms.openlocfilehash: c7405ada800bd5fb9161e9d96bd4c8b0484be620
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005315"
---
# <a name="troubleshoot-azure-file-sync"></a>Résoudre les problèmes de synchronisation de fichiers Azure
Utilisez Azure File Sync pour centraliser les partages de fichiers de votre organisation dans Azure Files tout en conservant la flexibilité, le niveau de performance et la compatibilité d’un serveur de fichiers local. Azure File Sync transforme Windows Server en un cache rapide de votre partage de fichiers Azure. Vous pouvez utiliser tout protocole disponible dans Windows Server pour accéder à vos données localement, notamment SMB, NFS et FTPS. Vous pouvez avoir autant de caches que nécessaire dans le monde entier.

Cet article est destiné à vous aider à dépanner et à résoudre les problèmes que vous pouvez rencontrer avec le déploiement d’Azure File Sync. Nous vous y expliquons également comment collecter des journaux d’activité du système qui sont utiles pour analyser les problèmes rencontrés de manière plus approfondie. Si vous ne trouvez pas de réponse à votre question ici, vous pouvez nous joindre par le biais des méthodes suivantes (par ordre de priorité) :

1. [Page de questions Microsoft Q&A sur le Stockage Azure](/answers/products/azure?product=storage).
2. [Azure Files UserVoice](https://feedback.azure.com/forums/217298-storage/category/180670-files)
3. Support Microsoft Pour créer une demande de support, dans le portail Azure, sous l’onglet **Aide**, sélectionnez le bouton **Aide et support**, puis **Nouvelle demande de support**.

## <a name="im-having-an-issue-with-azure-file-sync-on-my-server-sync-cloud-tiering-etc-should-i-remove-and-recreate-my-server-endpoint"></a>Je rencontre un problème avec Azure File Sync sur mon serveur (synchronisation, hiérarchisation cloud, etc.). Dois-je supprimer et recréer mon point de terminaison de serveur ?
[!INCLUDE [storage-sync-files-remove-server-endpoint](../../../includes/storage-sync-files-remove-server-endpoint.md)]

## <a name="agent-installation-and-server-registration"></a>Installation de l’agent et inscription du serveur
<a id="agent-installation-failures"></a>**Résoudre les problèmes d’installation de l’agent**  
Si l’installation de l’agent Azure File Sync échoue, à partir d’une invite de commandes avec élévation de privilèges, exécutez la commande suivante pour activer la journalisation durant l’installation de l’agent :

```
StorageSyncAgent.msi /l*v AFSInstaller.log
```

Examinez le fichier installer.log pour déterminer la cause de l’échec de l’installation.

<a id="agent-installation-on-DC"></a>**L’installation de l’agent échoue sur le contrôleur de domaine Active Directory**  
Si vous essayez d’installer l’agent de synchronisation sur un contrôleur de domaine Active Directory où le propriétaire du rôle de contrôleur de domaine principal est sur Windows Server 2008 R2 ou un système d’exploitation antérieur, l’agent de synchronisation peut échouer.

Pour résoudre cela, transférez le rôle de contrôleur de domaine principal à un autre contrôleur de domaine avec Windows Server 2012 R2 ou une version plus récente, puis synchronisez.

<a id="parameter-is-incorrect"></a>**L’accès à un volume sur Windows Server 2012 R2 échoue en générant l’erreur suivante : Le paramètre est incorrect**  
Après la création d’un point de terminaison de serveur sur Windows Server 2012 R2, l’erreur suivante se produit lors de l’accès au volume :

lettre_lecteur :\ n’est pas accessible.  
Le paramètre est incorrect.

Pour résoudre ce problème, installez [KB2919355](https://support.microsoft.com/help/2919355/windows-rt-8-1-windows-8-1-windows-server-2012-r2-update-april-2014) et redémarrez le serveur. Si cette mise à jour ne s’installe pas car une mise à jour ultérieure est déjà installée, accédez à Windows Update, installez les dernières mises à jour pour Windows Server 2012 R2 et redémarrez le serveur.

<a id="server-registration-missing-subscriptions"></a>**L’inscription du serveur ne répertorie pas tous les abonnements Azure**  
Lors de l’inscription d’un serveur à l’aide de ServerRegistration.exe, des abonnements sont manquants dans la liste déroulante Abonnement Azure.

Ce problème se produit parce que ServerRegistration. exe ne prend pas actuellement en charge les environnements mutualisés. Ce problème sera résolu dans une prochaine mise à jour de l’agent Azure File Sync.

Pour contourner ce problème, utilisez les commandes PowerShell suivantes pour inscrire le serveur :

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.PowerShell.Cmdlets.dll"
Login-AzureRmStorageSync -SubscriptionID "<guid>" -TenantID "<guid>"
Register-AzureRmStorageSyncServer -SubscriptionId "<guid>" -ResourceGroupName "<string>" -StorageSyncServiceName "<string>"
```

<a id="server-registration-prerequisites"></a>**L’inscription du serveur affiche le message suivant : « Les conditions préalables sont manquantes »**  
Ce message apparaît si le module Az ou AzureRM PowerShell n’est pas installé sur PowerShell 5.1. 

> [!Note]  
> ServerRegistration.exe ne prend pas en charge PowerShell 6.x. Vous pouvez utiliser la cmdlet Register-AzStorageSyncServer sur PowerShell 6.x pour inscrire le serveur.

Pour installer le module AZ ou AzureRM sur PowerShell 5.1, procédez comme suit :

1. Tapez **PowerShell** à partir d’une invite de commandes avec élévation de privilèges et appuyez sur entrée.
2. Installez le dernier module AZ ou AzureRM en suivant la documentation :
    - [Module Az (requiert .NET 4.7.2)](/powershell/azure/install-az-ps?viewFallbackFrom=azps-1.1.0)
    - [Module AzureRM]( https://go.microsoft.com/fwlink/?linkid=856959)
3. Exécutez ServerRegistration.exe, puis effectuez l’Assistant pour inscrire le serveur auprès d’un service de synchronisation de stockage.

<a id="server-already-registered"></a>**L’inscription du serveur affiche le message suivant : « Ce serveur est déjà inscrit »** 

![Capture d’écran de la boîte de dialogue d’inscription du serveur avec le message d’erreur « Ce serveur est déjà inscrit »](media/storage-sync-files-troubleshoot/server-registration-1.png)

Ce message s’affiche si le serveur a déjà été inscrit auprès d’un service de synchronisation de stockage. Pour désinscrire le serveur du service de synchronisation de stockage actuel et l’inscrire ensuite auprès d’un nouveau service de synchronisation de stockage, suivez les étapes décrites dans [Désinscrire un serveur d’Azure File Sync](storage-sync-files-server-registration.md#unregister-the-server-with-storage-sync-service).

Si le serveur n’est pas répertorié sous **Serveurs inscrits** dans le service de synchronisation de stockage, sur le serveur à désinscrire, exécutez les commandes PowerShell suivantes :

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Reset-StorageSyncServer
```

> [!Note]  
> Si le serveur fait partie d’un cluster, vous pouvez utiliser le paramètre facultatif *Reset-StorageSyncServer -CleanClusterRegistration* pour annuler aussi l’inscription du cluster.

<a id="web-site-not-trusted"></a>**Quand j’inscris un serveur, je vois de nombreuses réponses indiquant que le site web n’est pas approuvé. Pourquoi ?**  
Ce problème se produit quand la stratégie **Sécurité renforcée d’Internet Explorer** est activée pendant l’inscription du serveur. Pour plus d’informations sur la façon de désactiver correctement la stratégie de **Sécurité renforcée d’Internet Explorer**, consultez [Préparer Windows Server pour une utilisation avec Azure File Sync](storage-sync-files-deployment-guide.md#prepare-windows-server-to-use-with-azure-file-sync) et [Déployer Azure File Sync](storage-sync-files-deployment-guide.md).

<a id="server-registration-missing"></a>**Le serveur n’est pas listé sous Serveurs inscrits sur le Portail Azure**  
Si un serveur n’est pas listé sous **Serveurs inscrits** pour un service de synchronisation de stockage :
1. Connectez-vous au serveur que vous souhaitez inscrire.
2. Ouvrez l’Explorateur de fichiers, puis accédez au répertoire d’installation de l’agent de synchronisation de stockage (l’emplacement par défaut est C:\Program Files\Azure\StorageSyncAgent). 
3. Exécutez ServerRegistration.exe, puis effectuez l’Assistant pour inscrire le serveur auprès d’un service de synchronisation de stockage.

## <a name="sync-group-management"></a>Gestion du groupe de synchronisation

### <a name="cloud-endpoint-creation-errors"></a>Erreurs de création de point de terminaison cloud

<a id="cloud-endpoint-using-share"></a>**La création du point de terminaison cloud échoue, avec cette erreur : « Le partage de fichiers Azure spécifié est déjà en cours d’utilisation par un autre point de terminaison cloud »**  
Cette erreur se produit si le partage de fichiers Azure est déjà en cours d’utilisation par un autre point de terminaison cloud. 

Si vous voyez ce message et que le partage de fichiers Azure n’est pas en cours d’utilisation par un point de terminaison cloud, effectuez les étapes suivantes pour supprimer les métadonnées Azure File Sync sur le partage de fichiers Azure :

> [!Warning]  
> La suppression des métadonnées sur un partage de fichiers Azure en cours d’utilisation par un point de terminaison cloud entraîne l’échec des opérations Azure File Sync. 

1. Dans le portail Azure, accédez au partage de fichiers Azure.  
2. Cliquez avec le bouton droit sur le partage de fichiers Azure, puis sélectionnez **Modifier les métadonnées**.
3. Cliquez avec le bouton droit sur **SyncService**, puis sélectionnez **Supprimer**.

<a id="cloud-endpoint-authfailed"></a>**La création du point de terminaison cloud échoue, avec cette erreur : « AuthorizationFailed »**  
Cette erreur se produit si votre compte d’utilisateur ne dispose pas des droits suffisants pour créer un point de terminaison cloud. 

Pour créer un point de terminaison cloud, votre compte d’utilisateur doit disposer des autorisations Microsoft suivantes :  
* Lecture : Obtenir la définition de rôle
* Écriture : Créer ou mettre à jour une définition de rôle personnalisée
* Lecture : Obtenir l’affectation de rôle
* Écriture : Créer une attribution de rôle

Les rôles intégrés suivants ont les autorisations Microsoft nécessaires :  
* Propriétaire
* Administrateur de l'accès utilisateur

Pour déterminer si votre rôle de compte d’utilisateur a les autorisations nécessaires :  
1. Dans le portail Azure, sélectionnez **Groupes de ressources**.
2. Sélectionnez le groupe de ressources dans lequel se trouve le compte de stockage, puis sélectionnez **Contrôle d’accès (IAM)** .
3. Sélectionnez l’onglet **Attributions de rôles**.
4. Sélectionnez le **Rôle** (par exemple, Propriétaire ou Contributeur) pour votre compte d’utilisateur.
5. Dans la liste **Fournisseur de ressources**, sélectionnez **Autorisation Microsoft**. 
    * **Attribution de rôle** doit avoir les autorisations **Lecture** et **Écriture**.
    * **Définition de rôle** doit avoir les autorisations **Lecture** et **Écriture**.

### <a name="server-endpoint-creation-and-deletion-errors"></a>Erreurs de création et de suppression de point de terminaison de serveur

<a id="-2134375898"></a>**La création du point de terminaison de serveur échoue, avec cette erreur : « MgmtServerJobFailed » (Code d'erreur : -2134375898 ou 0x80c80226)**  
Cette erreur se produit si le chemin du point de terminaison de serveur se trouve sur le volume système et que la hiérarchisation cloud est activée. La hiérarchisation cloud n’est pas prise en charge sur le volume système. Pour créer un point de terminaison de serveur sur le volume système, désactivez la hiérarchisation cloud quand vous créez le point de terminaison de serveur.

<a id="-2147024894"></a>**La création du point de terminaison de serveur échoue, avec cette erreur : « MgmtServerJobFailed » (Code d'erreur : -2147024894 ou 0x80070002)**  
Cette erreur se produit si le chemin d’accès au point de terminaison du serveur spécifié n’est pas valide. Vérifiez que le chemin d’accès au point de terminaison du serveur spécifié est un volume NTFS attaché localement. Notez que Azure File Sync ne prend pas en charge les lecteurs mappés comme un chemin de point de terminaison de serveur.

<a id="-2134375640"></a>**La création du point de terminaison de serveur échoue, avec cette erreur : « MgmtServerJobFailed » (Code d’erreur : -2134375640 ou 0x80c80328)**  
Cette erreur se produit si le chemin du point de terminaison du serveur spécifié n’est pas un volume NTFS. Vérifiez que le chemin d’accès au point de terminaison du serveur spécifié est un volume NTFS attaché localement. Notez que Azure File Sync ne prend pas en charge les lecteurs mappés comme un chemin de point de terminaison de serveur.

<a id="-2134347507"></a>**La création du point de terminaison de serveur échoue, avec cette erreur : « MgmtServerJobFailed » (Code d'erreur : -2134347507 ou 0x80c8710d)**  
Cette erreur se produit parce que Azure File Sync ne prend pas en charge les points de terminaison de serveur sur les volumes qui comportent un dossier System Volume Information compressé. Pour résoudre ce problème, décompressez le dossier System Volume Information. Si le dossier System Volume Information est le seul dossier compressé sur le volume, procédez comme suit :

1. Téléchargez l’outil [PsExec](/sysinternals/downloads/psexec).
2. Exécutez la commande suivante à partir d’une invite de commandes avec élévation de privilèges pour lancer une invite de commandes qui s’exécute sous le compte système : **PsExec.exe -i -s -d cmd**
3. À partir de l’invite de commandes qui s’exécute sous le compte système, tapez les commandes suivantes et appuyez sur entrée :   
    **cd /d "drive letter:\System Volume Information"**  
    **compact /u /s**

<a id="-2134376345"></a>**La création du point de terminaison de serveur échoue, avec cette erreur : « MgmtServerJobFailed » (Code d'erreur : -2134376345 ou 0x80C80067)**  
Cette erreur se produit si la limite des points de terminaison de serveur par serveur est atteinte. Azure File Sync prend actuellement en charge jusqu’à 30 points de terminaison de serveur par serveur. Pour plus d’informations, consultez la page sur la [de tarification Azure File Sync](./storage-files-scale-targets.md#azure-file-sync-scale-targets).

<a id="-2134376427"></a>**La création du point de terminaison de serveur échoue, avec cette erreur : « MgmtServerJobFailed » (Code d’erreur : -2134376427 or 0x80c80015)**  
Cette erreur se produit si un autre nœud final de serveur est déjà en train de synchroniser le chemin du nœud final de serveur spécifié. Azure File Sync ne prend pas en charge plusieurs points de terminaison de serveur qui synchronisent le même répertoire ou volume.

<a id="-2160590967"></a>**La création du point de terminaison de serveur échoue, avec cette erreur : « MgmtServerJobFailed » (Code d’erreur : -2160590967 ou 0x80c80077)**  
Cette erreur se produit si le chemin du point de terminaison de serveur contient des fichiers hiérarchisés orphelins. Si un point de terminaison de serveur a été récemment supprimé, patientez jusqu’à la fin du nettoyage des fichiers hiérarchisés orphelins. Un ID d’événement 6662 est enregistré dans le journal des événements de télémétrie une fois que le nettoyage des fichiers hiérarchisés orphelins a démarré. Un ID d’événement 6661 est enregistré une fois que le nettoyage des fichiers hiérarchisés orphelins est terminé et qu’un point de terminaison de serveur peut être recréé à l’aide du chemin. Si la création du point de terminaison de serveur échoue après le nettoyage des fichiers hiérarchisés ou si l’ID d’événement 6661 est introuvable dans le journal des événements de télémétrie en raison de la substitution de ce journal, supprimez les fichiers hiérarchisés orphelins en suivant les étapes décrites dans la section [Les fichiers hiérarchisés ne sont pas accessibles sur le serveur après la suppression d’un point de terminaison de serveur](?tabs=portal1%252cazure-portal#tiered-files-are-not-accessible-on-the-server-after-deleting-a-server-endpoint).

<a id="-2134347757"></a>**La suppression du point de terminaison de serveur échoue, avec cette erreur : « MgmtServerJobExpired » (Code d'erreur : -2134347757 ou 0x80c87013)**  
Cette erreur se produit si le serveur est hors connexion ou n’a pas de connectivité réseau. Si le serveur n’est plus disponible, désinscrivez le serveur dans le portail pour supprimer les points de terminaison de serveur. Pour supprimer les points de terminaison de serveur, suivez les étapes décrites dans [Désinscrire un serveur dans Azure File Sync](storage-sync-files-server-registration.md#unregister-the-server-with-storage-sync-service).

### <a name="server-endpoint-health"></a>Intégrité du point de terminaison de serveur

<a id="server-endpoint-provisioningfailed"></a>**Impossible d’ouvrir la page de propriétés du point de terminaison serveur ou de mettre à jour de la stratégie de hiérarchisation du cloud**  
Ce problème peut se produire si une opération de gestion sur le point de terminaison serveur échoue. Si la page de propriétés de point de terminaison serveur ne s’ouvre pas dans le portail Azure, la mise à jour du point de terminaison serveur à l’aide des commandes PowerShell à partir du serveur peut résoudre ce problème. 

```powershell
# Get the server endpoint id based on the server endpoint DisplayName property
Get-AzStorageSyncServerEndpoint `
    -ResourceGroupName myrgname `
    -StorageSyncServiceName storagesvcname `
    -SyncGroupName mysyncgroup | `
Tee-Object -Variable serverEndpoint

# Update the free space percent policy for the server endpoint
Set-AzStorageSyncServerEndpoint `
    -InputObject $serverEndpoint
    -CloudTiering `
    -VolumeFreeSpacePercent 60
```
<a id="server-endpoint-noactivity"></a>**Le point de terminaison de serveur a un état d’intégrité « Aucune activité » ou « En attente », et l’état du serveur sur le panneau des serveurs inscrits est « Apparaît hors connexion »**  

Ce problème peut se produire si le processus de supervision de la synchronisation du stockage (AzureStorageSyncMonitor.exe) ne s’exécute pas ou que le serveur ne peut pas accéder au service Azure File Sync.

Sur le serveur qui porte la mention « Apparaît hors connexion » dans le portail, examinez l’ID d’événement 9301 dans le journal des événements de télémétrie (situé sous Applications et services\Microsoft\FileSync\Agent dans l’Observateur d’événements) pour déterminer la raison pour laquelle le serveur ne peut pas accéder au service Azure File Sync. 

- Si l’événement **GetNextJob s’est terminé avec l’état : 0** est enregistré, le serveur peut communiquer avec le service Azure File Sync. 
    - Ouvrez le gestionnaire des tâches sur le serveur et vérifiez que le processus de surveillance de la synchronisation du stockage (AzureStorageSyncMonitor.exe) est en cours d’exécution. Si le processus ne s’exécute pas, essayez d’abord de redémarrer le serveur. Si le redémarrage du serveur ne résout pas le problème, procédez à une mise à niveau vers la dernière [version de l'agent](./storage-files-release-notes.md) Azure File Sync. 

- Si l’événement **GetNextJob s’est terminé avec l’état : -2134347756** est enregistré, le serveur ne peut pas communiquer avec le service Azure File Sync à cause d’un pare-feu ou d’un proxy. 
    - Si le serveur se trouve derrière un pare-feu, vérifiez que le port 443 sortant est autorisé. Si le pare-feu restreint le trafic à des domaines spécifiques, vérifiez que les domaines répertoriés dans la [documentation](./storage-sync-files-firewall-and-proxy.md#firewall) du pare-feu sont accessibles.
    - Si le serveur se trouve derrière un proxy, configurez les paramètres de proxy au niveau de l’ordinateur ou de l’application en suivant la procédure de la [documentation](./storage-sync-files-firewall-and-proxy.md#proxy) du proxy.
    - Utilisez l’applet de commande Test-StorageSyncNetworkConnectivity pour vérifier la connectivité réseau aux points de terminaison de service. Pour plus d’informations, consultez [Tester la connectivité réseau aux points de terminaison de service](./storage-sync-files-firewall-and-proxy.md#test-network-connectivity-to-service-endpoints).

- Si l’événement **GetNextJob s’est terminé avec l’état : -2134347764** est enregistré, le serveur ne peut pas communiquer avec le service Azure File Sync à cause de l’expiration ou de la suppression d’un certificat.  
    - Exécutez la commande PowerShell suivante sur le serveur pour réinitialiser le certificat utilisé pour l’authentification :
    ```powershell
    Reset-AzStorageSyncServerCertificate -ResourceGroupName <string> -StorageSyncServiceName <string>
    ```
<a id="endpoint-noactivity-sync"></a>**Le point de terminaison du serveur a l’état d’intégrité « Aucune activité », et l’état du serveur sur le panneau des serveurs inscrits est « En ligne »**  

Si un point de terminaison de serveur a l’état d’intégrité « Aucune activité », cela signifie qu’il n’a journalisé aucune activité de synchronisation au cours des deux dernières heures.

Pour vérifier l’activité de synchronisation active sur un serveur, consultez [Comment suivre la progression d’une session de synchronisation active ?](#how-do-i-monitor-the-progress-of-a-current-sync-session).

Un point de terminaison de serveur risque de ne pas journaliser l’activité de synchronisation pendant plusieurs heures en raison d’un bogue ou de ressources système insuffisantes. Vérifiez que la dernière [version de l’agent](./storage-files-release-notes.md) Azure File Sync est installée. Si le problème persiste, ouvrez une demande de support.

> [!Note]  
> Si l’état du serveur sur le panneau des serveurs inscrits est « Apparaît hors connexion », suivez les étapes décrites dans la section [Le point de terminaison de serveur a un état d’intégrité « Aucune activité » ou « En attente » et l’état du serveur dans le panneau des serveurs inscrits est « Apparaît hors connexion »](#server-endpoint-noactivity).

## <a name="sync"></a>Synchronisation
<a id="afs-change-detection"></a>**Après avoir créé un fichier directement dans mon partage de fichiers Azure sur SMB ou par le biais du portail, combien de temps faut-il pour synchroniser le fichier sur les serveurs du groupe de synchronisation ?**  
[!INCLUDE [storage-sync-files-change-detection](../../../includes/storage-sync-files-change-detection.md)]

<a id="serverendpoint-pending"></a>**L’intégrité du point de terminaison de serveur est en état d’attente pendant plusieurs heures**  
Ce problème peut se produire si vous créez un point de terminaison cloud et que vous utilisez un partage de fichiers Azure contenant des données. Le travail d’énumération de modifications qui analyse les modifications dans le partage de fichiers Azure doit être terminé avant que les fichiers puissent être synchronisés entre le cloud et les points de terminaison serveur. La durée d’exécution du travail dépend de la taille de l’espace de noms dans le partage de fichiers Azure. L’intégrité du point de terminaison de serveur doit se mettre à jour une fois que le travail d’énumération des modifications est terminé.

### <a name="how-do-i-monitor-sync-health"></a><a id="broken-sync"></a>Comment surveiller l’intégrité de synchronisation ?
# <a name="portal"></a>[Portail](#tab/portal1)
Au sein de chaque groupe de synchronisation, vous pouvez zoomer sur ses points de terminaison serveur individuel pour afficher l’état des dernières sessions de synchronisation terminées. Une colonne de contrôle d’intégrité verte et des fichiers non synchronisés à la valeur 0 indiquent que la synchronisation fonctionne comme prévu. Si ce n’est pas le cas, voir ci-dessous pour obtenir la liste des erreurs de synchronisation courantes et savoir comment gérer les fichiers qui ne synchronisent pas. 

![Capture d’écran du portail Azure](media/storage-sync-files-troubleshoot/portal-sync-health.png)

# <a name="server"></a>[Serveur](#tab/server)
Accédez aux enregistrements de données de télémétrie du serveur, qui se trouve dans l’observateur d’événements à `Applications and Services Logs\Microsoft\FileSync\Agent\Telemetry`. Événement 9102 correspond à une session de synchronisation terminée; Pour obtenir le dernier état de synchronisation, recherchez l’événement le plus récent avec ID 9102. SyncDirection vous indique si cette session a été chargée ou téléchargée. Si la valeur HResult est 0, la session de synchronisation a réussi. Une valeur HResult différente de zéro indique une erreur pendant la synchronisation; voir ci-dessous pour obtenir la liste d’erreurs courantes. Si le PerItemErrorCount est supérieur à 0, cela signifie que certains fichiers ou dossiers non pas synchronisés correctement. Il est possible d’avoir une valeur HResult égale à 0, mais un PerItemErrorCount supérieur à 0.

Vous trouverez ci-dessous un exemple d’un chargement réussi. Par souci de concision, uniquement certaines des valeurs contenues dans chaque événement 9102 sont répertoriées ci-dessous. 

```
Replica Sync session completed.
SyncDirection: Upload,
HResult: 0, 
SyncFileCount: 2, SyncDirectoryCount: 0,
AppliedFileCount: 2, AppliedDirCount: 0, AppliedTombstoneCount 0, AppliedSizeBytes: 0.
PerItemErrorCount: 0,
TransferredFiles: 2, TransferredBytes: 0, FailedToTransferFiles: 0, FailedToTransferBytes: 0.
```

À l’inverse, un téléchargement échoué peut ressembler à ceci :

```
Replica Sync session completed.
SyncDirection: Upload,
HResult: -2134364065,
SyncFileCount: 0, SyncDirectoryCount: 0, 
AppliedFileCount: 0, AppliedDirCount: 0, AppliedTombstoneCount 0, AppliedSizeBytes: 0.
PerItemErrorCount: 0, 
TransferredFiles: 0, TransferredBytes: 0, FailedToTransferFiles: 0, FailedToTransferBytes: 0.
```

Parfois, les sessions de synchronisation échouent globalement ou ont un PerItemErrorCount différent de zéro, mais progressent toujours, avec des synchronisations de fichiers réussies. Ceci peut être observé dans les champs Applied* (AppliedFileCount AppliedDirCount, AppliedTombstoneCount et AppliedSizeBytes), qui vous indiquent le niveau d’avancement de la session. Si vous voyez plusieurs sessions de synchronisation consécutives échouer tout en ayant un nombre d’Applied* croissant, alors vous devriez laisser à la synchronisation le temps de réessayer avant d’ouvrir un ticket de support.

---

### <a name="how-do-i-monitor-the-progress-of-a-current-sync-session"></a>Comment surveiller la progression d’une session en cours de synchronisation ?
# <a name="portal"></a>[Portail](#tab/portal1)
Au sein de votre groupe de synchronisation, accédez au point de terminaison de serveur en question et examinez la section Activité de synchronisation pour voir le nombre de fichiers chargés ou téléchargés dans la session de synchronisation en cours. Notez que cet état sera retardé d’environ 5 minutes, et si votre session de synchronisation est suffisamment petite pour être effectuée pendant cette période, il peut ne pas être signalé dans le portail. 

# <a name="server"></a>[Serveur](#tab/server)
Recherchez les événements 9302 les plus récents dans les enregistrements de données de télémétrie sur le serveur (dans l’observateur d’événements, accédez aux enregistrements des Applications and Services\Microsoft\FileSync\Agent\Telemetry). Cet événement indique l’état de la session de synchronisation en cours. TotalItemCount indique combien de fichiers doivent être synchronisés, AppliedItemCount le nombre de fichiers qui ont été synchronisés jusqu’ici et PerItemErrorCount le nombre de fichiers qui ne parviennent pas à se synchroniser (voir ci-dessous pour savoir comment gérer cette opération).

```
Replica Sync Progress. 
ServerEndpointName: <CI>sename</CI>, SyncGroupName: <CI>sgname</CI>, ReplicaName: <CI>rname</CI>, 
SyncDirection: Upload, CorrelationId: {AB4BA07D-5B5C-461D-AAE6-4ED724762B65}. 
AppliedItemCount: 172473, TotalItemCount: 624196. AppliedBytes: 51473711577, 
TotalBytes: 293363829906. 
AreTotalCountsFinal: true. 
PerItemErrorCount: 1006.
```
---

### <a name="how-do-i-know-if-my-servers-are-in-sync-with-each-other"></a>Comment savoir si mes serveurs sont synchronisés entre eux ?
# <a name="portal"></a>[Portail](#tab/portal1)
Pour chaque serveur dans un groupe de synchronisation donné, vérifiez que :
- Les horodatages de la dernière tentative de synchronisation pour le chargement téléchargement sont récents.
- L’état est affiché en vert pour le chargement et le téléchargement.
- Le champ de l’activité de synchronisation présente très peu ou pas de fichiers restant à synchroniser.
- Le champ des fichiers non synchronisés est à 0 pour le chargement et le téléchargement.

# <a name="server"></a>[Serveur](#tab/server)
Examinez les sessions de synchronisation terminées, qui sont marquées par des événements 9102 dans les données de télémétrie de l’événement pour chaque serveur (dans l’observateur d’événements, accédez à `Applications and Services Logs\Microsoft\FileSync\Agent\Telemetry`). 

1. Sur un serveur donné, vous souhaitez vous assurer que les dernières sessions de chargement et téléchargement se sont terminées correctement. Pour ce faire, vérifiez que le HResult et PerItemErrorCount sont à 0 pour le chargement et le téléchargement (le champ SyncDirection indique si une session donnée est une session de chargement ou téléchargement). Notez que si vous ne voyez pas une session de synchronisation effectuée récemment, il est probable qu’une session de synchronisation soit en cours, ce qui est normal si vous venez d’ajouter ou de modifier une grande quantité de données.
2. Lorsqu’un serveur est complètement à jour avec le cloud et ne comporte aucune modification pour la synchronisation dans les deux sens, vous verrez des sessions de synchronisation vide. Ceux-ci sont indiqués par des événements de chargement et de téléchargement dans lesquels tous les champs Sync* (SyncFileCount, SyncDirCount, SyncTombstoneCount, SyncTombstoneCount et SyncSizeBytes) sont à zéro, ce qui signifie qu’il n’y avait rien à synchroniser. Notez que ces sessions de synchronisation vide ne peuvent pas survenir sur les serveurs à forte activité car il y a toujours quelque chose de nouveau à synchroniser. S’il n’existe aucune activité de synchronisation, elles doivent se produire toutes les 30 minutes. 
3. Si tous les serveurs sont à jour avec le cloud, ce qui signifie que leur chargement récent et les sessions de téléchargement sont des sessions de synchronisation vide, vous pouvez affirmer avec une certitude raisonnable que l’ensemble du système est synchronisé. 
    
Notez que si vous avez effectué des modifications directement dans votre partage de fichiers Azure, Azure File Sync ne détectera pas cette modification jusqu’à ce que les énumérations de modifications s’exécutent, ce qui se produit une fois toutes les 24 heures. Il est possible qu’un serveur indique qu’il est à jour avec le cloud alors qu’il lui manque en fait des modifications récentes apportées directement dans le partage de fichiers Azure. 

---

### <a name="how-do-i-see-if-there-are-specific-files-or-folders-that-are-not-syncing"></a>Comment puis-je voir s’il existe des fichiers ou dossiers qui ne sont pas synchronisés ?
Si votre PerItemErrorCount sur le serveur ou le nombre de fichiers non synchronisés dans le portail est supérieur à 0 pour une session de synchronisation donnée, cela signifie que certains éléments ne sont pas synchronisés. Les fichiers et les dossiers peuvent avoir des caractéristiques qui empêchent leur synchronisation. Ces caractéristiques peuvent être persistantes et nécessitent une action explicite pour reprendre la synchronisation, par exemple la suppression des caractères non pris en charge à partir du nom du fichier ou du dossier. Elles peuvent aussi être temporaires, ce qui signifie que le fichier ou le dossier reprendra automatiquement la synchronisation ; par exemple, des fichiers avec des handles ouverts qui reprennent automatiquement la synchronisation quand le fichier est fermé. Lorsque le moteur Azure File Sync détecte ce type de problème, un journal des erreurs est généré, il peut être analysé pour répertorier les éléments qui ne sont pas en train de se synchroniser correctement.

Pour afficher ces erreurs, exécutez le script PowerShell **FileSyncErrorsReport.ps1** (situé dans le répertoire d’installation de l’agent Azure File Sync) pour identifier les fichiers qui ont échoué lors de la synchronisation en raison de descripteurs ouverts, de caractères non pris en charge, ou d’autres problèmes. Le champ ItemPath vous indique l’emplacement du fichier en relation avec le répertoire racine de synchronisation. Consultez la liste des erreurs de synchronisation courantes ci-dessous pour les étapes de correction.

> [!Note]  
> Si le script FileSyncErrorsReport.ps1 retourne « Aucune erreur de fichier n’a été trouvée » ou ne contient pas d’erreurs par élément pour le groupe de synchronisation, la cause est l’une des suivantes :
>
>- Cause 1 : La dernière session de synchronisation terminée n’a pas d’erreurs par élément. Le portail doit être mis à jour bientôt pour afficher « 0 Fichiers non synchronisés ». 
>    - Vérifiez l’[ID d’événement 9102](?tabs=server%252cazure-portal#broken-sync) dans le journal des événements de télémétrie pour confirmer que PerItemErrorCount a la valeur 0. 
>
>- Cause 2 : Le journal des événements ItemResults sur le serveur a été enveloppé (wrapped) car les erreurs par élément sont trop nombreuses et le journal des événements ne contient plus d’erreurs pour ce groupe de synchronisation.
>    - Pour éviter ce problème, augmentez la taille du journal des événements ItemResults. Le journal des événements ItemResults se trouve sous « Journaux des applications et des services\Microsoft\FileSync\Agent » dans l’observateur d’événements. 

#### <a name="troubleshooting-per-filedirectory-sync-errors"></a>Résolution des problèmes par les erreurs de synchronisation de fichier/répertoire
**Journal ItemResults : erreurs de synchronisation par élément**  

| HRESULT | HRESULT (décimal) | Chaîne d’erreur | Problème | Correction |
|---------|-------------------|--------------|-------|-------------|
| 0x80070043 | -2147942467 | ERROR_BAD_NET_NAME | Le fichier hiérarchisé sur le serveur n’est pas accessible. Ce problème se produit si le fichier hiérarchisé n’a pas été rappelé avant la suppression d’un point de terminaison de serveur. | Pour résoudre ce problème, consultez [Fichiers hiérarchisés non accessibles sur le serveur après la suppression d’un point de terminaison de serveur](?tabs=portal1%252cazure-portal#tiered-files-are-not-accessible-on-the-server-after-deleting-a-server-endpoint). |
| 0x80c80207 | -2134375929 | ECS_E_SYNC_CONSTRAINT_CONFLICT | La modification de fichier ou de répertoire ne peut pas encore être synchronisée, car un dossier dépendant n’est pas encore synchronisé. Cet élément sera synchronisé une fois que les modifications dépendantes seront synchronisées. | Aucune action requise. Si l’erreur persiste pendant plusieurs jours, utilisez le script PowerShell FileSyncErrorsReport.ps1 pour déterminer la raison pour laquelle le dossier dépendant n’est pas encore synchronisé. |
| 0x80C8028A | -2134375798 | ECS_E_SYNC_CONSTRAINT_CONFLICT_ON_FAILED_DEPENDEE | La modification de fichier ou de répertoire ne peut pas encore être synchronisée, car un dossier dépendant n’est pas encore synchronisé. Cet élément sera synchronisé une fois que les modifications dépendantes seront synchronisées. | Aucune action requise. Si l’erreur persiste pendant plusieurs jours, utilisez le script PowerShell FileSyncErrorsReport.ps1 pour déterminer la raison pour laquelle le dossier dépendant n’est pas encore synchronisé. |
| 0x80c80284 | -2134375804 | ECS_E_SYNC_CONSTRAINT_CONFLICT_SESSION_FAILED | La modification de fichier ou de répertoire ne peut pas encore être synchronisée, car un dossier dépendant n’est pas encore synchronisé et la session de synchronisation a échoué. Cet élément sera synchronisé une fois que les modifications dépendantes seront synchronisées. | Aucune action requise. Si l’erreur persiste, recherchez la cause de l’échec de la session de synchronisation. |
| 0x8007007b | -2147024773 | ERROR_INVALID_NAME | Le nom de répertoire est non valide. | Renommez le fichier ou le répertoire en question. Pour plus d’informations, voir [Gestion des caractères non pris en charge](?tabs=portal1%252cazure-portal#handling-unsupported-characters). |
| 0x80c80255 | -2134375851 | ECS_E_XSMB_REST_INCOMPATIBILITY | Le nom de répertoire est non valide. | Renommez le fichier ou le répertoire en question. Pour plus d’informations, voir [Gestion des caractères non pris en charge](?tabs=portal1%252cazure-portal#handling-unsupported-characters). |
| 0x80c80018 | -2134376424 | ECS_E_SYNC_FILE_IN_USE | Le fichier ne peut pas être synchronisé, car il est en cours d’utilisation. Le fichier sera synchronisé lorsqu’il ne sera plus en cours d’utilisation. | Aucune action requise. Azure File Sync crée une capture instantanée VSS temporaire une fois par jour sur le serveur pour synchroniser les fichiers qui ont des descripteurs ouverts. |
| 0x80c8031d | -2134375651 | ECS_E_CONCURRENCY_CHECK_FAILED | Le fichier a changé, mais la modification n’a pas encore été détectée par la synchronisation. La synchronisation sera rétablie une fois cette modification détectée. | Aucune action requise. |
| 0x80070002 | -2147024894 | ERROR_FILE_NOT_FOUND | Le fichier a été supprimé et la synchronisation n’est pas au courant de la modification. | Aucune action requise. La synchronisation arrête la journalisation de cette erreur une fois que la détection des modifications a détecté que le fichier a été supprimé. |
| 0x80070003 | -2147942403 | ERROR_PATH_NOT_FOUND | La suppression d’un fichier ou d’un répertoire ne peut pas être synchronisée, car l’élément a déjà été supprimé dans la destination et la synchronisation n’a pas connaissance de la modification. | Aucune action requise. La synchronisation arrête la journalisation de cette erreur une fois que la détection des modifications s’est exécutée sur la destination et que la synchronisation a détecté que l’élément a été supprimé. |
| 0x80c80205 | -2134375931 | ECS_E_SYNC_ITEM_SKIP | Le fichier ou le répertoire a été ignoré, mais sera synchronisé au cours de la prochaine session de synchronisation. Si cette erreur est signalée lors du téléchargement de l’élément, il est plus que probable que le nom du fichier ou du répertoire n’est pas valide. | Aucune action n’est requise si cette erreur est signalée lors du chargement du fichier. Si l’erreur est signalée lors du téléchargement du fichier, renommez le fichier ou le répertoire en question. Pour plus d’informations, voir [Gestion des caractères non pris en charge](?tabs=portal1%252cazure-portal#handling-unsupported-characters). |
| 0x800700B7 | -2147024713 | ERROR_ALREADY_EXISTS | La création d’un fichier ou d’un répertoire ne peut pas être synchronisée, car l’élément existe déjà dans la destination et la synchronisation n’a pas connaissance de la modification. | Aucune action requise. La synchronisation arrête la journalisation de cette erreur une fois que la détection des modifications s’est exécutée sur la destination et que la synchronisation a connaissance de ce nouvel élément. |
| 0x80c8603e | -2134351810 | ECS_E_AZURE_STORAGE_SHARE_SIZE_LIMIT_REACHED | Le fichier ne peut pas être synchronisé parce que la limite de partage de fichiers Azure est atteinte. | Pour résoudre ce problème, voir la section [Vous avez atteint la limite de stockage du partage de fichiers Azure](?tabs=portal1%252cazure-portal#-2134351810) dans le guide de dépannage. |
| 0x80c8027C | -2134375812 | ECS_E_ACCESS_DENIED_EFS | Le fichier est chiffré par une solution non prise en charge (comme NTFS EFS). | Déchiffrez le fichier et utilisez une solution de chiffrement prise en charge. Pour obtenir la liste des solutions prises en charge, voir la section [Solutions de chiffrement](./storage-sync-files-planning.md#encryption) du guide de planification. |
| 0x80c80283 | -2160591491 | ECS_E_ACCESS_DENIED_DFSRRO | Le fichier se trouve dans un dossier de réplication DFS-R en lecture seule. | Le fichier se trouve dans un dossier de réplication DFS-R en lecture seule. Azure Files Sync ne prend pas en charge les points de terminaison de serveur sur les dossiers de réplication en lecture seule DFS-R. Consultez le [guide de planification](./storage-sync-files-planning.md#distributed-file-system-dfs) pour plus d’informations. |
| 0x80070005 | -2147024891 | ERROR_ACCESS_DENIED | Le fichier est en attente de suppression. | Aucune action requise. Le fichier est supprimé une fois que tous les descripteurs de fichiers ouverts sont fermés. |
| 0x80c86044 | -2134351804 | ECS_E_AZURE_AUTHORIZATION_FAILED | Le fichier ne peut pas être synchronisé, car les paramètres du pare-feu et du réseau virtuel sur le compte de stockage sont activés et le serveur n’a pas accès au compte de stockage. | Ajoutez l’adresse IP ou le réseau virtuel du serveur en suivant les étapes décrites dans la section [Configurer les paramètres de pare-feu et de réseau virtuel](./storage-sync-files-deployment-guide.md?tabs=azure-portal#configure-firewall-and-virtual-network-settings) du Guide de déploiement. |
| 0x80c80243 | -2134375869 | ECS_E_SECURITY_DESCRIPTOR_SIZE_TOO_LARGE | Le fichier ne peut pas être synchronisé car la taille du descripteur de sécurité dépasse la limite de 64 Ko. | Pour résoudre ce problème, supprimez les entrées du contrôle d’accès (ACE) du fichier afin de réduire la taille du descripteur de sécurité. |
| 0x8000FFFF | -2147418113 | E_UNEXPECTED | Le fichier ne peut pas être synchronisé en raison d’une erreur inattendue. | Si l’erreur persiste pendant plusieurs jours, veuillez ouvrir un dossier de support. |
| 0x80070020 | -2147024864 | ERROR_SHARING_VIOLATION | Le fichier ne peut pas être synchronisé, car il est en cours d’utilisation. Le fichier sera synchronisé lorsqu’il ne sera plus en cours d’utilisation. | Aucune action requise. |
| 0x80c80017 | -2134376425 | ECS_E_SYNC_OPLOCK_BROKEN | Le fichier a été modifié pendant la synchronisation, par conséquent, il doit être synchronisé à nouveau. | Aucune action requise. |
| 0x80070017 | -2147024873 | ERROR_CRC | Impossible de synchroniser le fichier en raison d’une erreur CRC. Cette erreur peut se produire si un fichier hiérarchisé n’a pas été rappelé avant la suppression d’un point de terminaison de serveur ou si le fichier est endommagé. | Pour résoudre ce problème, voir [Les fichiers hiérarchisés ne sont pas accessibles sur le serveur après la suppression d’un point de terminaison de serveur](?tabs=portal1%252cazure-portal#tiered-files-are-not-accessible-on-the-server-after-deleting-a-server-endpoint) afin de supprimer les fichiers hiérarchisés orphelins. Si l’erreur persiste après la suppression des fichiers hiérarchisés orphelins, exécutez la commande [chkdsk](/windows-server/administration/windows-commands/chkdsk) sur le volume. |
| 0x80c80200 | -2134375936 | ECS_E_SYNC_CONFLICT_NAME_EXISTS | Impossible de synchroniser le fichier, car le nombre maximal de fichiers conflictuels a été atteint. Azure File Sync prend en charge 100 fichiers conflictuels par fichier. Pour en savoir plus sur les conflits de fichiers, consultez la [FAQ](./storage-files-faq.md#afs-conflict-resolution) d’Azure File Sync. | Pour résoudre ce problème, réduisez le nombre de fichiers conflictuels. Le fichier est synchronisé une fois que le nombre de fichiers conflictuels est inférieur à 100. |

#### <a name="handling-unsupported-characters"></a>Gestion des caractères non pris en charge
Si le script PowerShell **FileSyncErrorsReport.ps1** montre des erreurs de synchronisation par élément dues à des caractères non pris en charge (code d’erreur 0x8007007b ou 0x80c80255), supprimez les caractères en cause des noms de fichiers respectifs, ou renommez-les. PowerShell imprimera probablement ces caractères en tant que points d’interrogation ou rectangles vides dans la mesure où la plupart de ces caractères n’ont aucun codage visuel standard. 
> [!Note]  
> [L’outil d’évaluation](storage-sync-files-planning.md#evaluation-cmdlet) peut servir à identifier les caractères qui ne sont pas pris en charge. Si votre jeu de données contient plusieurs fichiers avec des caractères non valides, utilisez le script [ScanUnsupportedChars](https://github.com/Azure-Samples/azure-files-samples/tree/master/ScanUnsupportedChars) pour renommer les fichiers qui contiennent des caractères non pris en charge.

Le tableau ci-dessous contient tous les caractères unicode qu’Azure File Sync ne prend pas en charge.

| Jeu de caractères | Nombre de caractères |
|---------------|-----------------|
| <ul><li>0x0000009D (osc commande de système d’exploitation)</li><li>0x00000090 (dcs chaîne de commande d’appareils)</li><li>0x0000008F (ss3 remplacement unique trois)</li><li>0x00000081 (préréglage haut octet)</li><li>0x0000007F (suppr Supprimer)</li><li>0x0000008D (ri interligne inversé)</li></ul> | 6 |
| 0x0000FDD0 - 0x0000FDEF (formulaire de présentation arabe-a) | 32 |
| 0x0000FFF0 - 0x0000FFFF (caractères spéciaux) | 16 |
| <ul><li>0x0001FFFE - 0x0001FFFF = 2 (type non caractère)</li><li>0x0002FFFE - 0x0002FFFF = 2 (type non caractère)</li><li>0x0003FFFE - 0x0003FFFF = 2 (type non caractère)</li><li>0x0004FFFE - 0x0004FFFF = 2 (type non caractère)</li><li>0x0005FFFE - 0x0005FFFF = 2 (type non caractère)</li><li>0x0006FFFE - 0x0006FFFF = 2 (type non caractère)</li><li>0x0007FFFE - 0x0007FFFF = 2 (type non caractère)</li><li>0x0008FFFE - 0x0008FFFF = 2 (type non caractère)</li><li>0x0009FFFE - 0x0009FFFF = 2 (type non caractère)</li><li>0x000AFFFE - 0x000AFFFF = 2 (type non caractère)</li><li>0x000BFFFE - 0x000BFFFF = 2 (type non caractère)</li><li>0x000CFFFE - 0x000CFFFF = 2 (type non caractère)</li><li>0x000DFFFE - 0x000DFFFF = 2 (type non caractère)</li><li>0x000EFFFE - 0x000EFFFF = 2 (type non caractère)</li><li>0x000FFFFE - 0x000FFFFF = 2 (zone d’utilisation privée supplémentaire)</li></ul> | 30 |
| 0x0010FFFE, 0x0010FFFF | 2 |

### <a name="common-sync-errors"></a>Erreurs de synchronisation courantes
<a id="-2147023673"></a>**La session de synchronisation a été annulée.**  

| | |
|-|-|
| **HRESULT** | 0x800704c7 |
| **HRESULT (décimal)** | -2147023673 | 
| **Chaîne d’erreur** | ERROR_CANCELLED |
| **Correction requise** | Non |

Les sessions de synchronisation peuvent échouer pour diverses raisons, comme la mise à jour ou le redémarrage en cours du serveur, des captures instantanées VSS, etc. Bien que cette erreur semble nécessiter un suivi, il est possible d’ignorer cette erreur, sauf si elle persiste pendant une période de plusieurs heures.

<a id="-2147012889"></a>**Impossible d’établir une connexion avec le service.**    

| | |
|-|-|
| **HRESULT** | 0x80072ee7 |
| **HRESULT (décimal)** | -2147012889 | 
| **Chaîne d’erreur** | WININET_E_NAME_NOT_RESOLVED |
| **Correction requise** | Oui |

[!INCLUDE [storage-sync-files-bad-connection](../../../includes/storage-sync-files-bad-connection.md)]

<a id="-2134376372"></a>**La requête de l’utilisateur a été limitée par le service.**  

| | |
|-|-|
| **HRESULT** | 0x80c8004c |
| **HRESULT (décimal)** | -2134376372 |
| **Chaîne d’erreur** | ECS_E_USER_REQUEST_THROTTLED |
| **Correction requise** | Non |

Aucune action n’est requise; le serveur essayera à nouveau. Si cette erreur persiste pendant plusieurs heures, créez une demande de support.

<a id="-2134364043"></a>**La synchronisation est bloquée jusqu’à ce que la détection des modifications soit terminée après la restauration**  

| | |
|-|-|
| **HRESULT** | 0x80c83075 |
| **HRESULT (décimal)** | -2134364043 |
| **Chaîne d’erreur** | ECS_E_SYNC_BLOCKED_ON_CHANGE_DETECTION_POST_RESTORE |
| **Correction requise** | Non |

Aucune action n'est requise. Quand un fichier ou un partage de fichiers (point de terminaison cloud) est restauré à l’aide de la Sauvegarde Azure, la synchronisation est bloquée jusqu’à ce que la détection des modifications soit terminée sur le partage de fichiers Azure. La détection des modifications s’exécute immédiatement une fois la restauration terminée, et sa durée dépend du nombre de fichiers contenus dans le partage de fichiers.

<a id="-2147216747"></a>**La synchronisation a échoué car la base de données de synchronisation a été déchargée.**  

| | |
|-|-|
| **HRESULT** | 0x80041295 |
| **HRESULT (décimal)** | -2147216747 |
| **Chaîne d’erreur** | SYNC_E_METADATA_INVALID_OPERATION |
| **Correction requise** | Non |

Cette erreur se produit généralement lorsqu’une application de sauvegarde crée une capture instantanée VSS et que la base de données de synchronisation est déchargée. Si cette erreur persiste pendant plusieurs heures, créez une demande de support.

<a id="-2134364065"></a>**La synchronisation ne peut pas accéder au partage de fichiers Azure spécifié dans le point de terminaison de cloud.**  

| | |
|-|-|
| **HRESULT** | 0x80c8305f |
| **HRESULT (décimal)** | -2134364065 |
| **Chaîne d’erreur** | ECS_E_EXTERNAL_STORAGE_ACCOUNT_AUTHORIZATION_FAILED |
| **Correction requise** | Oui |

Cette erreur se produit parce que l’agent Azure File Sync ne peut pas accéder au partage de fichiers Azure, ce qui peut être dû au fait que le partage de fichiers Azure ou le compte de stockage qui l’héberge n’existent plus. Vous pouvez résoudre cette erreur en parcourant les étapes suivantes :

1. [Vérifiez l’existence du compte de stockage.](#troubleshoot-storage-account)
2. [Assurez-vous de l’existence du partage de fichiers Azure.](#troubleshoot-azure-file-share)
3. [Vérifiez qu’Azure File Sync a accès au compte de stockage.](#troubleshoot-rbac)
4. [Vérifiez que les paramètres de pare-feu et de réseau virtuel sur le compte de stockage sont configurés correctement (si activés)](./storage-sync-files-deployment-guide.md?tabs=azure-portal#configure-firewall-and-virtual-network-settings)

<a id="-2134351804"></a>**La synchronisation a échoué, car la demande n’est pas autorisée à effectuer cette opération.**  

| | |
|-|-|
| **HRESULT** | 0x80c86044 |
| **HRESULT (décimal)** | -2134351804 |
| **Chaîne d’erreur** | ECS_E_AZURE_AUTHORIZATION_FAILED |
| **Correction requise** | Oui |

Cette erreur se produit car l’agent Azure File Sync n’est pas autorisé à accéder au partage de fichiers Azure. Vous pouvez résoudre cette erreur en parcourant les étapes suivantes :

1. [Vérifiez l’existence du compte de stockage.](#troubleshoot-storage-account)
2. [Assurez-vous de l’existence du partage de fichiers Azure.](#troubleshoot-azure-file-share)
3. [Vérifiez que les paramètres de pare-feu et de réseau virtuel sur le compte de stockage sont configurés correctement (si activés)](./storage-sync-files-deployment-guide.md?tabs=azure-portal#configure-firewall-and-virtual-network-settings)
4. [Vérifiez qu’Azure File Sync a accès au compte de stockage.](#troubleshoot-rbac)

<a id="-2134364064"></a><a id="cannot-resolve-storage"></a>**Le nom du compte de stockage utilisé n’a pas pu être résolu.**  

| | |
|-|-|
| **HRESULT** | 0x80C83060 |
| **HRESULT (décimal)** | -2134364064 |
| **Chaîne d’erreur** | ECS_E_STORAGE_ACCOUNT_NAME_UNRESOLVED |
| **Correction requise** | Oui |

1. Vérifiez que vous pouvez résoudre le nom DNS de stockage à partir du serveur.

    ```powershell
    Test-NetConnection -ComputerName <storage-account-name>.file.core.windows.net -Port 443
    ```
2. [Vérifiez l’existence du compte de stockage.](#troubleshoot-storage-account)
3. [Vérifiez que les paramètres de pare-feu et de réseau virtuel sur le compte de stockage sont configurés correctement (si activés)](./storage-sync-files-deployment-guide.md?tabs=azure-portal#configure-firewall-and-virtual-network-settings)

<a id="-2134364022"></a><a id="storage-unknown-error"></a>**Une erreur inconnue s’est produite lors de l’accès au compte de stockage.**  

| | |
|-|-|
| **HRESULT** | 0x80c8308a |
| **HRESULT (décimal)** | -2134364022 |
| **Chaîne d’erreur** | ECS_E_STORAGE_ACCOUNT_UNKNOWN_ERROR |
| **Correction requise** | Oui |

1. [Vérifiez l’existence du compte de stockage.](#troubleshoot-storage-account)
2. [Vérifiez que les paramètres de pare-feu et de réseau virtuel sur le compte de stockage sont configurés correctement (si activés)](./storage-sync-files-deployment-guide.md?tabs=azure-portal#configure-firewall-and-virtual-network-settings)

<a id="-2134364014"></a>**La synchronisation a échoué en raison du verrouillage du compte de stockage.**  

| | |
|-|-|
| **HRESULT** | 0x80c83092 |
| **HRESULT (décimal)** | -2134364014 |
| **Chaîne d’erreur** | ECS_E_STORAGE_ACCOUNT_LOCKED |
| **Correction requise** | Oui |

Cette erreur se produit parce que le compte de stockage a un [verrou de ressource](../../azure-resource-manager/management/lock-resources.md) en lecture seule. Pour résoudre ce problème, supprimez le verrou de ressource en lecture seule sur le compte de stockage. 

<a id="-1906441138"></a>**La synchronisation a échoué en raison d’un problème avec la base de données de synchronisation.**  

| | |
|-|-|
| **HRESULT** | 0x8e5e044e |
| **HRESULT (décimal)** | -1906441138 |
| **Chaîne d’erreur** | JET_errWriteConflict |
| **Correction requise** | Oui |

Cette erreur se produit lorsqu’il existe un problème avec la base de données interne utilisée par Azure File Sync. Lorsque ce problème se produit créez une demande de support et nous vous contacterons pour vous aider à résoudre ce problème.

<a id="-2134364053"></a>**La version de l’agent Azure File Sync installé sur le serveur n’est pas prise en charge.**  

| | |
|-|-|
| **HRESULT** | 0x80C8306B |
| **HRESULT (décimal)** | -2134364053 |
| **Chaîne d’erreur** | ECS_E_AGENT_VERSION_BLOCKED |
| **Correction requise** | Oui |

Cette erreur se produit si la version de l’agent Azure File Sync installé sur le serveur n’est pas prise en charge. Pour résoudre ce problème, [mettez à niveau]( https://docs.microsoft.com/azure/storage/files/storage-files-release-notes#upgrade-paths) l’agent avec une [version prise en charge]( https://docs.microsoft.com/azure/storage/files/storage-files-release-notes#supported-versions).

<a id="-2134351810"></a>**La limite de stockage de partage de fichiers Azure a été atteinte.**  

| | |
|-|-|
| **HRESULT** | 0x80c8603e |
| **HRESULT (décimal)** | -2134351810 |
| **Chaîne d’erreur** | ECS_E_AZURE_STORAGE_SHARE_SIZE_LIMIT_REACHED |
| **Correction requise** | Oui |

Cette erreur se produit lorsque la limite de stockage de partage de fichiers Azure a été atteinte, ce qui peut se produire si un quota est appliqué pour un partage de fichiers Azure ou si l’utilisation dépasse les limites d’un partage de fichiers Azure. Pour plus d’informations, consultez les [limites actuelles pour un partage de fichiers Azure](storage-files-scale-targets.md).

1. Accédez au groupe de synchronisation au sein du service de synchronisation de stockage.
2. Sélectionnez le point de terminaison de cloud au sein du groupe de synchronisation.
3. Notez le nom de partage de fichiers Azure dans le volet ouvert.
4. Sélectionnez le compte de stockage associé. Si ce lien échoue, le compte de stockage référencé a été supprimé.

    ![Une capture d’écran montrant le volet d’informations de point de terminaison de cloud avec un lien vers le compte de stockage.](media/storage-sync-files-troubleshoot/file-share-inaccessible-1.png)

5. Sélectionnez **Fichiers** pour afficher la liste des partages de fichiers.
6. Cliquez sur les points de suspension à la fin de la ligne pour le partage de fichiers Azure référencé par le point de terminaison de cloud.
7. Vérifiez que l’**Utilisation** est inférieure au **Quota**. Remarque : à moins qu’un autre quota n’ait été spécifié, le quota correspondra à la [taille maximale du partage de fichiers Azure](storage-files-scale-targets.md).

    ![Une capture d’écran des propriétés de partage de fichiers Azure.](media/storage-sync-files-troubleshoot/file-share-limit-reached-1.png)

Si le partage est plein et si un quota n’est pas défini, il est possible de corriger ce problème en faisant de chaque sous-dossier du point de terminaison de serveur en cours son propre point de terminaison de serveur dans leurs propres groupes de synchronisation distincts. De cette manière, chaque sous-dossier se synchronisera avec des partages de fichiers Azure individuels.

<a id="-2134351824"></a>**Impossible de trouver le partage de fichiers Azure.**  

| | |
|-|-|
| **HRESULT** | 0x80c86030 |
| **HRESULT (décimal)** | -2134351824 |
| **Chaîne d’erreur** | ECS_E_AZURE_FILE_SHARE_NOT_FOUND |
| **Correction requise** | Oui |

Cette erreur se produit lorsque le partage de fichiers Azure n’est pas accessible. Pour résoudre des problèmes :

1. [Vérifiez l’existence du compte de stockage.](#troubleshoot-storage-account)
2. [Assurez-vous de l’existence du partage de fichiers Azure.](#troubleshoot-azure-file-share)

Si le partage de fichiers Azure a été supprimé, vous devez créer un nouveau partage de fichiers, puis recréer le groupe de synchronisation. 

<a id="-2134364042"></a>**La synchronisation est en pause pendant la suspension de cet abonnement Azure.**  

| | |
|-|-|
| **HRESULT** | 0x80C83076 |
| **HRESULT (décimal)** | -2134364042 |
| **Chaîne d’erreur** | ECS_E_SYNC_BLOCKED_ON_SUSPENDED_SUBSCRIPTION |
| **Correction requise** | Oui |

Cette erreur se produit lorsque l’abonnement Azure est suspendu. La synchronisation est réactivée lors de la restauration de l’abonnement Azure. Consultez [Pourquoi mon abonnement Azure est-il désactivé et comment puis-je le réactiver ?](../../cost-management-billing/manage/subscription-disabled.md) pour plus d’informations.

<a id="-2134375618"></a>**Le compte de stockage comporte un pare-feu ou des réseaux virtuels configurés.**  

| | |
|-|-|
| **HRESULT** | 0x80c8033e |
| **HRESULT (décimal)** | -2134375618 |
| **Chaîne d’erreur** | ECS_E_SERVER_BLOCKED_BY_NETWORK_ACL |
| **Correction requise** | Oui |

Cette erreur se produit lorsque le partage de fichiers Azure est inaccessible en raison d’un pare-feu de compte de stockage ou parce que le compte de stockage appartient à un réseau virtuel. Vérifiez que les paramètres de pare-feu et de réseau virtuel sur le compte de stockage sont configurés correctement. Pour plus d’informations, consultez [Configurer les paramètres du pare-feu et du réseau virtuel](./storage-sync-files-deployment-guide.md?tabs=azure-portal#configure-firewall-and-virtual-network-settings). 

<a id="-2134375911"></a>**La synchronisation a échoué en raison d’un problème avec la base de données de synchronisation.**  

| | |
|-|-|
| **HRESULT** | 0x80c80219 |
| **HRESULT (décimal)** | -2134375911 |
| **Chaîne d’erreur** | ECS_E_SYNC_METADATA_WRITE_LOCK_TIMEOUT |
| **Correction requise** | Non |

Cette erreur se résout généralement d’elle-même et peut se produire s’il existe :

* Un nombre élevé de changements apportés aux fichiers sur les serveurs du groupe de synchronisation.
* Un grand nombre d’erreurs sur les fichiers et répertoires individuels.

Si cette erreur persiste pendant plusieurs heures, créez une demande de support et nous vous contacterons pour vous aider à résoudre ce problème.

<a id="-2146762487"></a>**Échec de connexion sécurisée au serveur. Le service de cloud a reçu un certificat inattendu.**  

| | |
|-|-|
| **HRESULT** | 0x800b0109 |
| **HRESULT (décimal)** | -2146762487 |
| **Chaîne d’erreur** | CERT_E_UNTRUSTEDROOT |
| **Correction requise** | Oui |

Cette erreur peut se produire si votre organisation utilise un proxy de terminaison TLS ou si une entité malveillante intercepte le trafic entre votre serveur et le service Azure File Sync. Si vous êtes certain que cela est prévu (parce que votre organisation utilise un proxy de terminaison TLS), vous pouvez ignorer la vérification du certificat avec un remplacement du registre.

1. Créez la valeur de Registre SkipVerifyingPinnedRootCertificate.

    ```powershell
    New-ItemProperty -Path HKLM:\SOFTWARE\Microsoft\Azure\StorageSync -Name SkipVerifyingPinnedRootCertificate -PropertyType DWORD -Value 1
    ```

2. Redémarrez le service de synchronisation sur le serveur inscrit.

    ```powershell
    Restart-Service -Name FileSyncSvc -Force
    ```

En définissant cette valeur de Registre, l'agent Azure File Sync accepte n'importe quel certificat TLS/SSL approuvé localement lors du transfert de données entre le serveur et le service cloud.

<a id="-2147012894"></a>**Impossible d’établir une connexion avec le service.**  

| | |
|-|-|
| **HRESULT** | 0x80072ee2 |
| **HRESULT (décimal)** | -2147012894 |
| **Chaîne d’erreur** | WININET_E_TIMEOUT |
| **Correction requise** | Oui |

[!INCLUDE [storage-sync-files-bad-connection](../../../includes/storage-sync-files-bad-connection.md)]

<a id="-2134375680"></a>**La synchronisation a échoué en raison d’un problème d’authentification.**  

| | |
|-|-|
| **HRESULT** | 0x80c80300 |
| **HRESULT (décimal)** | -2134375680 |
| **Chaîne d’erreur** | ECS_E_SERVER_CREDENTIAL_NEEDED |
| **Correction requise** | Oui |

Cette erreur se produit généralement lorsque l’heure du serveur est incorrecte. Si le serveur s’exécute sur une machine virtuelle, vérifiez que l’heure de l’hôte est correcte.

<a id="-2134364040"></a>**La synchronisation a échoué en raison de l’expiration du certificat.**  

| | |
|-|-|
| **HRESULT** | 0x80c83078 |
| **HRESULT (décimal)** | -2134364040 |
| **Chaîne d’erreur** | ECS_E_AUTH_SRV_CERT_EXPIRED |
| **Correction requise** | Oui |

Cette erreur se produit parce que le certificat utilisé pour l’authentification a expiré.

Pour vérifier si le certificat a expiré, effectuez les étapes suivantes :  
1. Ouvrez le composant logiciel enfichable MMC de certificats, sélectionnez Compte d’ordinateur et accédez à Certificates (Ordinateur Local)\Personal\Certificates.
2. Vérifiez si le certificat d’authentification client est arrivé à expiration.

Si le certificat d’authentification client a expiré, procédez comme suit pour résoudre le problème :

1. Vérifiez que la version 4.0.1.0 (ou ultérieure) de l'agent Azure File Sync est installée.
2. Exécutez la commande PowerShell suivante :

    ```powershell
    Reset-AzStorageSyncServerCertificate -ResourceGroupName <string> -StorageSyncServiceName <string>
    ```

<a id="-2134375896"></a>**La synchronisation car le certificat d’authentification est introuvable.**  

| | |
|-|-|
| **HRESULT** | 0x80c80228 |
| **HRESULT (décimal)** | -2134375896 |
| **Chaîne d’erreur** | ECS_E_AUTH_SRV_CERT_NOT_FOUND |
| **Correction requise** | Oui |

Cette erreur se produit parce que le certificat utilisé pour l’authentification est introuvable.

Pour résoudre ce problème, procédez comme suit :

1. Vérifiez que la version 4.0.1.0 (ou ultérieure) de l'agent Azure File Sync est installée.
2. Exécutez la commande PowerShell suivante :

    ```powershell
    Reset-AzStorageSyncServerCertificate -ResourceGroupName <string> -StorageSyncServiceName <string>
    ```

<a id="-2134364039"></a>**La synchronisation car l’identité d’authentification est introuvable.**  

| | |
|-|-|
| **HRESULT** | 0x80c83079 |
| **HRESULT (décimal)** | -2134364039 |
| **Chaîne d’erreur** | ECS_E_AUTH_IDENTITY_NOT_FOUND |
| **Correction requise** | Oui |

Cette erreur se produit en raison de l’échec de la suppression du point de terminaison de serveur et du point de terminaison dans un état partiellement supprimé. Pour résoudre ce problème, réessayez de supprimer le point de terminaison de serveur.

<a id="-1906441711"></a><a id="-2134375654"></a><a id="doesnt-have-enough-free-space"></a>**Le volume où se trouve le point de terminaison de serveur est faible sur l’espace disque.**  

| | |
|-|-|
| **HRESULT** | 0x8e5e0211 |
| **HRESULT (décimal)** | -1906441711 |
| **Chaîne d’erreur** | JET_errLogDiskFull |
| **Correction requise** | Oui |

| | |
|-|-|
| **HRESULT** | 0x80c8031a |
| **HRESULT (décimal)** | -2134375654 |
| **Chaîne d’erreur** | ECS_E_NOT_ENOUGH_LOCAL_STORAGE |
| **Correction requise** | Oui |

Cette erreur se produit parce que le volume est saturé. Cette erreur se produit souvent parce que les fichiers situés à l’extérieur du point de terminaison de serveur occupent de l’espace sur le volume. Libérer de l’espace sur le volume en ajoutant des points de terminaison serveur supplémentaires, déplacer les fichiers vers un autre volume ou augmenter la taille du volume sur lequel se trouve le point d’extrémité du serveur.

<a id="-2134364145"></a><a id="replica-not-ready"></a>**Le service n’est pas encore prêt pour la synchronisation avec ce point de terminaison de serveur.**  

| | |
|-|-|
| **HRESULT** | 0x80c8300f |
| **HRESULT (décimal)** | -2134364145 |
| **Chaîne d’erreur** | ECS_E_REPLICA_NOT_READY |
| **Correction requise** | Non |

Cette erreur se produit parce que le point de terminaison cloud a été créé avec du contenu déjà existant sur le partage de fichiers Azure. Azure File Sync doit analyser le partage de fichiers Azure pour déceler toute présence de contenu avant d’autoriser le point de terminaison de serveur à poursuivre sa synchronisation initiale.

<a id="-2134375877"></a><a id="-2134375908"></a><a id="-2134375853"></a>**La synchronisation a échoué en raison de problèmes avec de nombreux fichiers individuels.**  

| | |
|-|-|
| **HRESULT** | 0x80c8023b |
| **HRESULT (décimal)** | -2134375877 |
| **Chaîne d’erreur** | ECS_E_SYNC_METADATA_KNOWLEDGE_SOFT_LIMIT_REACHED |
| **Correction requise** | Oui |

| | |
|-|-|
| **HRESULT** | 0x80c8021c |
| **HRESULT (décimal)** | -2134375908 |
| **Chaîne d’erreur** | ECS_E_SYNC_METADATA_KNOWLEDGE_LIMIT_REACHED |
| **Correction requise** | Oui |

| | |
|-|-|
| **HRESULT** | 0x80c80253 |
| **HRESULT (décimal)** | -2134375853 |
| **Chaîne d’erreur** | ECS_E_TOO_MANY_PER_ITEM_ERRORS |
| **Correction requise** | Oui |

Les sessions de synchronisation échouent avec l’une de ces erreurs quand de nombreux fichiers ne parviennent pas à se synchroniser avec des erreurs par élément. Pour résoudre les erreurs par élément, effectuez les étapes décrites dans la section [Comment puis-je voir s’il existe des fichiers ou dossiers qui ne sont pas synchronisés ?](?tabs=portal1%252cazure-portal#how-do-i-see-if-there-are-specific-files-or-folders-that-are-not-syncing) Pour l’erreur de synchronisation ECS_E_SYNC_METADATA_KNOWLEDGE_LIMIT_REACHED, ouvrez une demande de support.

> [!NOTE]
> Azure File Sync crée une capture instantanée VSS temporaire une fois par jour sur le serveur pour synchroniser les fichiers qui ont des descripteurs ouverts.

<a id="-2134376423"></a>**La synchronisation a échoué en raison d’un problème avec le chemin du point de terminaison de serveur.**  

| | |
|-|-|
| **HRESULT** | 0x80c80019 |
| **HRESULT (décimal)** | -2134376423 |
| **Chaîne d’erreur** | ECS_E_SYNC_INVALID_PATH |
| **Correction requise** | Oui |

Vérifiez que le chemin d’accès existe, qu’il se trouve sur un volume NTFS local et n’est pas un point d’analyse ou un point de terminaison de serveur existant.

<a id="-2134375817"></a>**La synchronisation a échoué parce que la version du pilote de filtre n’est pas compatible avec la version de l’agent**  

| | |
|-|-|
| **HRESULT** | 0x80C80277 |
| **HRESULT (décimal)** | -2134375817 |
| **Chaîne d’erreur** | ECS_E_INCOMPATIBLE_FILTER_VERSION |
| **Correction requise** | Oui |

Cette erreur se produit parce que la version du pilote de filtre de hiérarchisation cloud (StorageSync.sys) chargée n’est pas compatible avec l’Agent de synchronisation de stockage (FileSyncSvc). Si l’agent Azure File Sync a été mis à niveau, redémarrez le serveur pour achever l’installation. Si l’erreur persiste, désinstallez l’agent, redémarrez le serveur, puis réinstallez l’agent Azure File Sync.

<a id="-2134376373"></a>**Le service est actuellement indisponible.**  

| | |
|-|-|
| **HRESULT** | 0x80c8004b |
| **HRESULT (décimal)** | -2134376373 |
| **Chaîne d’erreur** | ECS_E_SERVICE_UNAVAILABLE |
| **Correction requise** | Non |

Cette erreur se produit parce que le service Azure File Sync n’est pas disponible. Cette erreur se résoudra automatiquement lorsque le service Azure File Sync sera à nouveau disponible.

<a id="-2146233088"></a>**La synchronisation a échoué en raison d’une exception.**  

| | |
|-|-|
| **HRESULT** | 0x80131500 |
| **HRESULT (décimal)** | -2146233088 |
| **Chaîne d’erreur** | COR_E_EXCEPTION |
| **Correction requise** | Non |

Cette erreur se produit car la synchronisation a échoué en raison d’une exception. Si cette erreur persiste pendant plusieurs heures, créez une demande de support.

<a id="-2134364045"></a>**La synchronisation a échoué, car le compte de stockage a basculé vers une autre région.**  

| | |
|-|-|
| **HRESULT** | 0x80c83073 |
| **HRESULT (décimal)** | -2134364045 |
| **Chaîne d’erreur** | ECS_E_STORAGE_ACCOUNT_FAILED_OVER |
| **Correction requise** | Oui |

Cette erreur se produit car le compte de stockage a basculé vers une autre région. Azure File Sync ne prend pas en charge la fonctionnalité de basculement de compte de stockage. Les comptes de stockage contenant des partages de fichiers Azure utilisés en tant que points de terminaison cloud dans Azure File Sync ne doivent pas être basculés. Cela provoquera en effet un arrêt de la synchronisation et pourra entraîner une perte inattendue de données dans le cas de fichiers nouvellement hiérarchisés. Pour résoudre ce problème, déplacez le compte de stockage vers la région primaire.

<a id="-2134375922"></a>**La synchronisation a échoué en raison d’un problème temporaire avec la base de données de synchronisation.**  

| | |
|-|-|
| **HRESULT** | 0x80c8020e |
| **HRESULT (décimal)** | -2134375922 |
| **Chaîne d’erreur** | ECS_E_SYNC_METADATA_WRITE_LEASE_LOST |
| **Correction requise** | Non |

Cette erreur se produit en raison d’un problème interne avec la base de données de synchronisation. Cette erreur se résoudra automatiquement lors de la prochaine tentative de synchronisation. Si cette erreur persiste pendant une période prolongée, créez une demande de support et nous vous contacterons pour vous aider à résoudre ce problème.

<a id="-2134364024"></a>**La synchronisation a échoué en raison d’un changement dans le client Azure Active Directory**  

| | |
|-|-|
| **HRESULT** | 0x80c83088 |
| **HRESULT (décimal)** | -2134364024 | 
| **Chaîne d’erreur** | ECS_E_INVALID_AAD_TENANT |
| **Correction requise** | Oui |

Assurez-vous de disposer du dernier agent Azure File Sync. Depuis l’agent v10, Azure File Sync prend en charge le déplacement d’abonnement vers un autre locataire Azure Active Directory.
 
Lorsque vous disposez de la dernière version de l’agent, vous devez accorder à l’application Microsoft.StorageSync l’accès au compte de stockage (voir [Vérifiez qu’Azure File Sync a accès au compte de stockage](#troubleshoot-rbac)).

<a id="-2134364010"></a>**La synchronisation a échoué en raison d’une exception de pare-feu et de réseau virtuel non configurée**  

| | |
|-|-|
| **HRESULT** | 0x80c83096 |
| **HRESULT (décimal)** | -2134364010 | 
| **Chaîne d’erreur** | ECS_E_MGMT_STORAGEACLSBYPASSNOTSET |
| **Correction requise** | Oui |

Cette erreur se produit si les paramètres de pare-feu et de réseau virtuel sont activés sur le compte de stockage et que l’exception « Autoriser les services Microsoft approuvés à accéder à ce compte de stockage » n’est pas activée. Pour résoudre ce problème, suivez les étapes décrites dans la section [Configurer les paramètres de pare-feu et de réseau virtuel](./storage-sync-files-deployment-guide.md?tabs=azure-portal#configure-firewall-and-virtual-network-settings) du Guide de déploiement.

<a id="-2147024891"></a>**La synchronisation a échoué car les autorisations sur le dossier System Volume Information sont incorrectes.**  

| | |
|-|-|
| **HRESULT** | 0x80070005 |
| **HRESULT (décimal)** | -2147024891 |
| **Chaîne d’erreur** | ERROR_ACCESS_DENIED |
| **Correction requise** | Oui |

Cette erreur peut se produire si le compte NT AUTHORITY\SYSTEM n’a pas d’autorisation sur le dossier System Volume Information sur le volume où se trouve le point de terminaison de serveur. Remarque : si des fichiers individuels ne sont pas synchronisés avec ERROR_ACCESS_DENIED, suivez les étapes décrites dans la section [Résolution des erreurs de synchronisation par fichier/répertoire](?tabs=portal1%252cazure-portal#troubleshooting-per-filedirectory-sync-errors).

Pour résoudre ce problème, procédez comme suit :

1. Téléchargez l’outil [Psexec](/sysinternals/downloads/psexec).
2. Exécutez la commande suivante à partir d’une invite de commandes avec élévation de privilèges pour lancer une invite de commandes utilisant le compte système : **PsExec.exe -i -s -d cmd** 
3. À partir de l’invite de commandes qui s’exécute sous le compte système, exécutez la commande suivante pour confirmer que le compte NT AUTHORITY\SYSTEM n’a pas accès au dossier System Volume Information : **cacls "drive letter:\system volume information" /T /C**
4. Si le compte NT AUTHORITY\SYSTEM n’a pas accès au dossier System Volume Information, exécutez la commande suivante: **cacls  "drive letter:\system volume information" /T /E /G "NT AUTHORITY\SYSTEM:F"**
    - Si l’étape n°4 échoue avec l’accès refusé, exécutez la commande suivante pour prendre possession du dossier System Volume Information, puis répétez l'étape n°4 : **takeown /A /R /F "drive letter:\System Volume Information"**

<a id="-2134375810"></a>**La synchronisation a échoué car le partage de fichiers Azure a été supprimé et recréé.**  

| | |
|-|-|
| **HRESULT** | 0x80c8027e |
| **HRESULT (décimal)** | -2134375810 |
| **Chaîne d’erreur** | ECS_E_SYNC_REPLICA_ROOT_CHANGED |
| **Correction requise** | Oui |

Cette erreur se produit parce que Azure File Sync ne prend pas en charge la suppression et la recréation d’un partage de fichiers Azure dans le même groupe de synchronisation. 

Pour résoudre ce problème, supprimez et recréez le groupe de synchronisation en procédant comme suit :

1. Supprimez tous les points de terminaison de serveur dans le groupe de synchronisation.
2. Supprimez le point de terminaison cloud. 
3. Supprimez le groupe de synchronisation.
4. Si la hiérarchisation cloud a été activée sur un point de terminaison de serveur, supprimez les fichiers hiérarchisés orphelins sur le serveur en effectuant les étapes décrites dans la section [Les fichiers hiérarchisés ne sont pas accessibles sur le serveur après la suppression d’une section de point de terminaison de serveur](?tabs=portal1%252cazure-portal#tiered-files-are-not-accessible-on-the-server-after-deleting-a-server-endpoint).
5. Recréez le groupe de synchronisation.

<a id="-2145844941"></a>**La synchronisation a échoué car la requête HTTP a été redirigée**  

| | |
|-|-|
| **HRESULT** | 0x80190133 |
| **HRESULT (décimal)** | -2145844941 |
| **Chaîne d’erreur** | HTTP_E_STATUS_REDIRECT_KEEP_VERB |
| **Correction requise** | Oui |

Cette erreur se produit car Azure File Sync ne prend pas en charge la redirection HTTP (code d’état 3xx). Pour résoudre ce problème, désactivez la redirection HTTP sur votre serveur proxy ou périphérique réseau.

<a id="-2134364027"></a>**Un dépassement de délai s’est produit lors du transfert de données hors connexion, mais il est toujours en cours.**  

| | |
|-|-|
| **HRESULT** | 0x80c83085 |
| **HRESULT (décimal)** | -2134364027 |
| **Chaîne d’erreur** | ECS_E_DATA_INGESTION_WAIT_TIMEOUT |
| **Correction requise** | Non |

Cette erreur se produit quand une opération d’ingestion de données dépasse le délai d’attente. Cette erreur peut être ignorée si la synchronisation progresse (AppliedItemCount est supérieur à 0). Voir [Comment surveiller la progression d’une session en cours de synchronisation ?](#how-do-i-monitor-the-progress-of-a-current-sync-session).

### <a name="common-troubleshooting-steps"></a>Ouvrir les étapes de résolution des problèmes
<a id="troubleshoot-storage-account"></a>**Vérifiez l’existence du compte de stockage.**  
# <a name="portal"></a>[Portail](#tab/azure-portal)
1. Accédez au groupe de synchronisation au sein du service de synchronisation de stockage.
2. Sélectionnez le point de terminaison de cloud au sein du groupe de synchronisation.
3. Notez le nom de partage de fichiers Azure dans le volet ouvert.
4. Sélectionnez le compte de stockage associé. Si ce lien échoue, le compte de stockage référencé a été supprimé.
    ![Une capture d’écran montrant le volet d’informations de point de terminaison de cloud avec un lien vers le compte de stockage.](media/storage-sync-files-troubleshoot/file-share-inaccessible-1.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
# Variables for you to populate based on your configuration
$region = "<Az_Region>"
$resourceGroup = "<RG_Name>"
$syncService = "<storage-sync-service>"
$syncGroup = "<sync-group>"

# Log into the Azure account
Connect-AzAccount

# Check to ensure Azure File Sync is available in the selected Azure
# region.
$regions = [System.String[]]@()
Get-AzLocation | ForEach-Object { 
    if ($_.Providers -contains "Microsoft.StorageSync") { 
        $regions += $_.Location 
    } 
}

if ($regions -notcontains $region) {
    throw [System.Exception]::new("Azure File Sync is either not available in the " + `
        " selected Azure Region or the region is mistyped.")
}

# Check to ensure resource group exists
$resourceGroups = [System.String[]]@()
Get-AzResourceGroup | ForEach-Object { 
    $resourceGroups += $_.ResourceGroupName 
}

if ($resourceGroups -notcontains $resourceGroup) {
    throw [System.Exception]::new("The provided resource group $resourceGroup does not exist.")
}

# Check to make sure the provided Storage Sync Service
# exists.
$syncServices = [System.String[]]@()

Get-AzStorageSyncService -ResourceGroupName $resourceGroup | ForEach-Object {
    $syncServices += $_.StorageSyncServiceName
}

if ($syncServices -notcontains $syncService) {
    throw [System.Exception]::new("The provided Storage Sync Service $syncService does not exist.")
}

# Check to make sure the provided Sync Group exists
$syncGroups = [System.String[]]@()

Get-AzStorageSyncGroup -ResourceGroupName $resourceGroup -StorageSyncServiceName $syncService | ForEach-Object {
    $syncGroups += $_.SyncGroupName
}

if ($syncGroups -notcontains $syncGroup) {
    throw [System.Exception]::new("The provided sync group $syncGroup does not exist.")
}

# Get reference to cloud endpoint
$cloudEndpoint = Get-AzStorageSyncCloudEndpoint `
    -ResourceGroupName $resourceGroup `
    -StorageSyncServiceName $syncService `
    -SyncGroupName $syncGroup

# Get reference to storage account
$storageAccount = Get-AzStorageAccount | Where-Object { 
    $_.Id -eq $cloudEndpoint.StorageAccountResourceId
}

if ($storageAccount -eq $null) {
    throw [System.Exception]::new("The storage account referenced in the cloud endpoint does not exist.")
}
```
---

<a id="troubleshoot-azure-file-share"></a>**Assurez vous de l’existence du partage de fichiers Azure.**  
# <a name="portal"></a>[Portail](#tab/azure-portal)
1. Cliquez sur **Aperçu** sur la table des matières de gauche pour revenir à la page principale de compte de stockage.
2. Sélectionnez **Fichiers** pour afficher la liste des partages de fichiers.
3. Vérifiez que le partage de fichiers référencé par le point de terminaison de du cloud apparaît dans la liste des partages de fichiers (vous auriez dû le noter à l’étape 1 ci-dessus).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
$fileShare = Get-AzStorageShare -Context $storageAccount.Context | Where-Object {
    $_.Name -eq $cloudEndpoint.AzureFileShareName -and
    $_.IsSnapshot -eq $false
}

if ($fileShare -eq $null) {
    throw [System.Exception]::new("The Azure file share referenced by the cloud endpoint does not exist")
}
```
---

<a id="troubleshoot-rbac"></a>**Vérifiez qu’Azure File Sync a accès au compte de stockage.**  
# <a name="portal"></a>[Portail](#tab/azure-portal)
1. Cliquez sur **Contrôle d’accès (IAM)** sur la table des matières de gauche.
1. Cliquez sur l'onglet **Affectations de rôles** pour accéder à la liste des utilisations et applications (*principaux de service*) ayant accès à votre compte de stockage.
1. Vérifiez que **Microsoft.StorageSync** ou **Hybrid File Sync Service** (ancien nom de l’application) apparaît dans la liste avec le rôle **Lecteur et accès aux données**. 

    ![Capture d’écran du principal de service Hybrid File Sync Service dans l’onglet Contrôle d’accès du compte de stockage](media/storage-sync-files-troubleshoot/file-share-inaccessible-3.png)

    Si **Microsoft.StorageSync** ou **Hybrid File Sync Service** n’apparaît pas dans la liste, effectuez les étapes suivantes :

    - Cliquez sur **Add**.
    - Dans le champ **Rôle**, sélectionnez **Lecteur et accès aux données**.
    - Dans le champ **Sélectionner**, tapez **Microsoft.StorageSync**, sélectionnez le rôle, puis cliquez sur **Enregistrer**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell    
$role = Get-AzRoleAssignment -Scope $storageAccount.Id | Where-Object { $_.DisplayName -eq "Microsoft.StorageSync" }

if ($role -eq $null) {
    throw [System.Exception]::new("The storage account does not have the Azure File Sync " + `
                "service principal authorized to access the data within the " + ` 
                "referenced Azure file share.")
}
```
---

### <a name="how-do-i-prevent-users-from-creating-files-containing-unsupported-characters-on-the-server"></a>Comment empêcher les utilisateurs de créer des fichiers contenant des caractères non pris en charge sur le serveur ?
Vous pouvez utiliser [File Server Resource Manager (FSRM) File Screens](/windows-server/storage/fsrm/file-screening-management) pour empêcher les fichiers contenant dans leurs noms des caractères non pris en charge d’être créés sur le serveur. Vous pouvez avoir à le faire en utilisant PowerShell car la plupart des caractères non pris en charge ne sont pas imprimables et vous devez donc caster leurs représentations hexadécimales en premier lieu comme caractères.

Tout d’abord créez un groupe de fichiers FSRM à l’aide de la [cmdlet New-FsrmFileGroup](/powershell/module/fileserverresourcemanager/new-fsrmfilegroup). Cet exemple définit le groupe comme ne contenant que deux des caractères non pris en charge, mais vous pouvez inclure autant de caractères que nécessaire dans votre groupe de fichiers.

```powershell
New-FsrmFileGroup -Name "Unsupported characters" -IncludePattern @(("*"+[char]0x00000090+"*"),("*"+[char]0x0000008F+"*"))
```

Une fois que vous avez défini un groupe de fichiers FSRM, vous pourrez créer un filtre de fichiers FSRM à l’aide de la cmdlet New-FsrmFileScreen.

```powershell
New-FsrmFileScreen -Path "E:\AFSdataset" -Description "Filter unsupported characters" -IncludeGroup "Unsupported characters"
```

> [!Important]  
> Notez que les écrans de fichier doivent uniquement servir à bloquer la création de caractères non pris en charge par Azure File Sync. Si les filtres de fichiers sont utilisés dans d’autres scénarios, la synchronisation tentera en permanence de télécharger les fichiers à partir du partage de fichiers Azure sur le serveur et sera bloquée en raison de l’écran de fichier, ce qui entraînera une grande suite de données. 

## <a name="cloud-tiering"></a>Hiérarchisation cloud 
Il existe deux chemins d’accès dédiés aux défaillances dans la hiérarchisation cloud :

- La hiérarchisation des fichiers peut être mise en échec, auquel cas la tentative d’Azure File Sync de hiérarchiser un fichier sur Azure Files est elle aussi avortée.
- Le rappel des fichiers peut être mis en échec, ce qui signifie que le filtre du système de fichiers Azure File Sync (StorageSync.sys) ne parvient pas à télécharger des données quand un utilisateur tente d’accéder à un fichier hiérarchisé.

Il existe deux classes principales de défaillances pouvant se produire par le biais des deux chemins d’accès dédiés :

- Défaillances de stockage cloud
    - *Problèmes de disponibilité du service de stockage temporaire*. Pour plus d’informations, consultez [Contrat de niveau de service (SLA) pour le stockage Azure](https://azure.microsoft.com/support/legal/sla/storage/v1_2/).
    - *Inaccessibilité du partage de fichiers Azure*. Cet échec se produit généralement lorsque vous supprimez un partage de fichiers Azure qui est toujours un point de terminaison de cloud d’un groupe de synchronisation.
    - *Inaccessibilité d’un compte de stockage*. Cet échec se produit généralement lorsque vous supprimez un compte de stockage comportant un partage de fichiers Azure qui est un point de terminaison cloud dans un groupe de synchronisation. 
- Défaillances de serveur 
  - *Défaut de chargement du filtre du système de fichiers Azure File Sync (StorageSync.sys)* . Pour répondre aux requêtes de hiérarchisation/de rappel, il est nécessaire que le filtre du système de fichiers Azure File Sync soit chargé. Ce défaut de chargement peut s’expliquer de différentes manières. La raison la plus courante est le déchargement manuel par un administrateur. Le filtre du système de fichiers Azure File Sync doit être chargé à tout moment pour qu’un bon fonctionnement d’Azure File Sync soit garanti.
  - *Défaut, corruption ou endommagement de point d’analyse*. Un point d’analyse est une structure de données spécifique d’un fichier qui est composée de deux parties distinctes :
    1. Une balise d’analyse, qui indique au système d’exploitation que le filtre du système de fichiers Azure File Sync (StorageSync.sys) doit éventuellement effectuer une action sur les E/S du fichier. 
    2. Les données d’analyse, qui indiquent au filtre du système de fichiers l’URI du fichier sur le point de terminaison associé du cloud (le partage de fichiers Azure). 
        
       La raison la plus courante de la corruption d’un point d’analyse est la tentative par un administrateur de modifier la balise ou ses données. 
  - *Problèmes de connectivité réseau*. Pour hiérarchiser ou rappeler un fichier, le serveur doit disposer d’une connectivité Internet.

Les sections suivantes vous indiquent comment résoudre les problèmes de hiérarchisation cloud et déterminer si un problème est lié au stockage cloud ou au serveur.

### <a name="how-to-monitor-tiering-activity-on-a-server"></a>Comment surveiller l’activité de hiérarchisation sur un serveur  
Pour surveiller l’activité de hiérarchisation sur un serveur, utilisez les ID d’événement 9003, 9016 et 9029 dans le journal des événements de télémétrie (situé sous Applications and Services\Microsoft\FileSync\Agent in Event Viewer).

- L’ID d’événement 9003 fournit la distribution des erreurs de distribution pour un point de terminaison de serveur. Par exemple, le nombre total d’erreurs, le code d’erreur, etc. Remarque : un événement est enregistré par code d’erreur.
- L’ID d’événement 9016 fournit des résultats de dédoublement pour un volume. Par exemple, le pourcentage d’espace libre, le nombre de fichiers dédoublé dans la session, le nombre d’échec de dédoublement de fichiers, etc.
- L’ID d’événement 9029 fournit des informations sur les sessions de duplication d’un point de terminaison de serveur. Par exemple, le nombre de fichiers tentés dans la session, le nombre de fichiers hiérarchisés dans la session, le nombre de fichiers déjà hiérarchisés, etc.

### <a name="how-to-monitor-recall-activity-on-a-server"></a>Comment surveiller l’activité de rappel sur un serveur
Pour surveiller l’activité de rappel sur un serveur, utilisez les ID d’événement 9005, 9006, 9009 et 9059 dans le journal des événements de télémétrie (situé sous Applications and Services\Microsoft\FileSync\Agent in Event Viewer).

- L’ID d’événement 9005 fournit une fiabilité de rappel pour un point de terminaison de serveur. Par exemple, le nombre de total des fichiers uniques consultés, le nombre total des fichiers uniques dont l’accès a échoué, etc.
- L’ID d’événement 9006 fournit la distribution des erreurs de rappel pour un point de terminaison de serveur. Par exemple, le nombre total d’échecs de requête, le code d’erreur, etc. Remarque : un événement est enregistré par code d’erreur.
- L’ID d’événement 9009 fournit des informations sur les sessions de rappel d’un point de terminaison de serveur. Par exemple, DurationSeconds, CountFilesRecallSucceeded, CountFilesRecallFailed, etc.
- L’ID d’événement 9059 fournit la distribution des rappels d’application pour un point de terminaison de serveur. Par exemple, ShareId, Application Name et TotalEgressNetworkBytes.

### <a name="how-to-troubleshoot-files-that-fail-to-tier"></a>Résoudre les problèmes de hiérarchisation de fichiers
Si la hiérarchisation des fichiers dans Azure Files échoue :

1. Dans l’observateur d’événements, examinez les journaux d’événements de télémétrie, des opérations et de diagnostic, situés sous Applications and Services\Microsoft\FileSync\Agent. 
   1. Vérifiez que le ou les fichiers se trouvent dans le partage de fichiers Azure.

      > [!NOTE]
      > Un fichier doit être synchronisé dans un partage de fichiers Azure pour pouvoir ensuite être hiérarchisé.

   2. Vérifiez la connectivité Internet du serveur. 
   3. Vérifiez que les pilotes de filtre Azure File Sync (StorageSync.sys et StorageSyncGuard.sys) sont en cours d’exécution :
       - À partir d’une invite de commandes avec élévation de privilèges, exécutez `fltmc`. Vérifiez que les pilotes de filtre du système de fichiers StorageSync.sys et StorageSyncGuard.sys sont répertoriés.

> [!NOTE]
> Un d’ID d’événement 9003 est enregistré une fois par heure dans le journal d’événements de télémétrie si un fichier ne parvient pas à hiérarchiser (un événement est enregistré par code d’erreur). Consultez la section [Erreurs de hiérarchisation et correction](#tiering-errors-and-remediation) pour vérifier si des étapes de correction sont fournies pour le code d’erreur.

### <a name="tiering-errors-and-remediation"></a>Erreurs de hiérarchisation et correction

| HRESULT | HRESULT (décimal) | Chaîne d’erreur | Problème | Correction |
|---------|-------------------|--------------|-------|-------------|
| 0x80c86045 | -2134351803 | ECS_E_INITIAL_UPLOAD_PENDING | Le fichier n’a pas pu être hiérarchisé, car le chargement initial est en cours. | Aucune action requise. Le fichier sera hiérarchisé une fois le chargement initial terminé. |
| 0x80c86043 | -2134351805 | ECS_E_GHOSTING_FILE_IN_USE | Le fichier n’a pas pu être hiérarchisé car il est en cours d’utilisation. | Aucune action requise. Le fichier sera hiérarchisé quand il ne sera plus en cours d’utilisation. |
| 0x80c80241 | -2134375871 | ECS_E_GHOSTING_EXCLUDED_BY_SYNC | Le fichier n’a pas pu être hiérarchisé car il est exclu par synchronisation. | Aucune action requise. Les fichiers de la liste d’exclusion de synchronisation ne peuvent pas être hiérarchisés. |
| 0x80c86042 | -2134351806 | ECS_E_GHOSTING_FILE_NOT_FOUND | Le fichier n’a pas pu être hiérarchisé car il est introuvable sur le serveur. | Aucune action requise. Si l’erreur persiste, vérifiez si le fichier existe sur le serveur. |
| 0x80c83053 | -2134364077 | ECS_E_CREATE_SV_FILE_DELETED | Le fichier n’a pas pu être hiérarchisé car il a été supprimé dans le partage de fichiers Azure. | Aucune action requise. Le fichier doit être supprimé sur le serveur lors de l’exécution de la session de synchronisation de téléchargement suivante. |
| 0x80c8600e | -2134351858 | ECS_E_AZURE_SERVER_BUSY | La hiérarchisation du fichier a échoué à cause d’un problème réseau. | Aucune action requise. Si l’erreur persiste, vérifiez la connectivité réseau vers le partage de fichiers Azure. |
| 0x80072ee7 | -2147012889 | WININET_E_NAME_NOT_RESOLVED | La hiérarchisation du fichier a échoué à cause d’un problème réseau. | Aucune action requise. Si l’erreur persiste, vérifiez la connectivité réseau vers le partage de fichiers Azure. |
| 0x80070005 | -2147024891 | ERROR_ACCESS_DENIED | La hiérarchisation du fichier a échoué à cause d’une erreur d’accès refusé. Ce problème peut se produire si le fichier se trouve dans un dossier de réplication DFS-R en lecture seule. | Azure Files Sync ne prend pas en charge les points de terminaison de serveur sur les dossiers de réplication en lecture seule DFS-R. Consultez le [guide de planification](./storage-sync-files-planning.md#distributed-file-system-dfs) pour plus d’informations. |
| 0x80072efe | -2147012866 | WININET_E_CONNECTION_ABORTED | La hiérarchisation du fichier a échoué à cause d’un problème réseau. | Aucune action requise. Si l’erreur persiste, vérifiez la connectivité réseau vers le partage de fichiers Azure. |
| 0x80c80261 | -2134375839 | ECS_E_GHOSTING_MIN_FILE_SIZE | La hiérarchisation du fichier a échoué car la taille du fichier est inférieure à la taille prise en charge. | Si la version de l’agent est inférieure à 9.0, la taille de fichier minimale prise en charge est de 64 Ko. Si la version de l’agent est 9.0 ou plus récente, la taille de fichier minimale prise en charge dépend de la taille de cluster du système de fichiers (deux fois la taille de cluster de système de fichiers). Par exemple, si la taille du cluster de système de fichiers est de 4 Ko, la taille de fichier minimale est de 8 Ko. |
| 0x80c83007 | -2134364153 | ECS_E_STORAGE_ERROR | La hiérarchisation du fichier a échoué en raison d’un problème de stockage Azure. | Si l’erreur persiste, ouvrez une demande de support. |
| 0x800703e3 | -2147023901 | ERROR_OPERATION_ABORTED | Le fichier n’a pas pu être hiérarchisé car il a été rappelé en même temps. | Aucune action requise. Le fichier sera hiérarchisé quand le rappel sera terminé et que le fichier ne sera plus utilisé. |
| 0x80c80264 | -2134375836 | ECS_E_GHOSTING_FILE_NOT_SYNCED | Le fichier n’a pas pu être hiérarchisé car il n’a pas été synchronisé avec le partage de fichiers Azure. | Aucune action requise. Le fichier sera hiérarchisé une fois qu’il aura été synchronisé avec le partage de fichiers Azure. |
| 0x80070001 | -2147942401 | ERROR_INVALID_FUNCTION | Le fichier n’a pas pu être hiérarchisé car le pilote de filtre de hiérarchisation cloud (storagesync.sys) n’est pas en cours d’exécution. | Pour résoudre ce problème, ouvrez une invite de commandes avec élévation de privilèges et exécutez la commande suivante : `fltmc load storagesync`<br>Si le chargement du pilote de filtre storagesync échoue lors de l’exécution de la commande fltmc, désinstallez l’agent Azure File Sync, redémarrez le serveur et réinstallez l’agent Azure File Sync. |
| 0x80070070 | -2147024784 | ERROR_DISK_FULL | La hiérarchisation du fichier a échoué car l’espace disque est insuffisant sur le volume où se trouve le point de terminaison de serveur. | Pour résoudre ce problème, libérez au moins 100 Mo d’espace disque sur le volume où se trouve le point de terminaison de serveur. |
| 0x80070490 | -2147023728 | ERROR_NOT_FOUND | Le fichier n’a pas pu être hiérarchisé car il n’a pas été synchronisé avec le partage de fichiers Azure. | Aucune action requise. Le fichier sera hiérarchisé une fois qu’il aura été synchronisé avec le partage de fichiers Azure. |
| 0x80c80262 | -2134375838 | ECS_E_GHOSTING_UNSUPPORTED_RP | La hiérarchisation du fichier a échoué car il s’agit d’un point d’analyse non pris en charge. | Si le fichier est un point d’analyse de déduplication des données, effectuez les étapes décrites dans le [guide de planification](./storage-sync-files-planning.md#data-deduplication) pour activer la prise en charge de la déduplication des données. Les fichiers avec des points d’analyse autres que la déduplication des données ne sont pas pris en charge et ne seront pas hiérarchisés.  |
| 0x80c83052 | -2134364078 | ECS_E_CREATE_SV_STREAM_ID_MISMATCH | Le fichier n’a pas pu être hiérarchisé car il a été modifié. | Aucune action requise. Le fichier sera hiérarchisé une fois que le fichier modifié aura été synchronisé avec le partage de fichiers Azure. |
| 0x80c80269 | -2134375831 | ECS_E_GHOSTING_REPLICA_NOT_FOUND | Le fichier n’a pas pu être hiérarchisé car il n’a pas été synchronisé avec le partage de fichiers Azure. | Aucune action requise. Le fichier sera hiérarchisé une fois qu’il aura été synchronisé avec le partage de fichiers Azure. |
| 0x80072ee2 | -2147012894 | WININET_E_TIMEOUT | La hiérarchisation du fichier a échoué à cause d’un problème réseau. | Aucune action requise. Si l’erreur persiste, vérifiez la connectivité réseau vers le partage de fichiers Azure. |
| 0x80c80017 | -2134376425 | ECS_E_SYNC_OPLOCK_BROKEN | Le fichier n’a pas pu être hiérarchisé car il a été modifié. | Aucune action requise. Le fichier sera hiérarchisé une fois que le fichier modifié aura été synchronisé avec le partage de fichiers Azure. |
| 0x800705aa | -2147023446 | ERROR_NO_SYSTEM_RESOURCES | La hiérarchisation du fichier a échoué car les ressources système sont insuffisantes. | Si l’erreur persiste, essayez d’identifier l’application ou le pilote en mode noyau qui épuise les ressources système. |
| 0x8e5e03fe | -1906441218 | JET_errDiskIO | Le fichier n’a pas pu être hiérarchisé en raison d’une erreur d’E/S lors de l’écriture dans la base de données de hiérarchisation cloud. | Si l’erreur persiste, exécutez chkdsk sur le volume et vérifiez le matériel de stockage. |
| 0x8e5e0442 | -1906441150 | JET_errInstanceUnavailable | Le fichier n’a pas pu être hiérarchisé car la base de données de hiérarchisation cloud n’est pas en cours d’exécution. | Pour résoudre ce problème, redémarrez le service ou le serveur FileSyncSvc. Si l’erreur persiste, exécutez chkdsk sur le volume et vérifiez le matériel de stockage. |



### <a name="how-to-troubleshoot-files-that-fail-to-be-recalled"></a>Résoudre les problèmes de rappel de fichiers  
Si le rappel de fichiers échoue :
1. Dans l’observateur d’événements, examinez les journaux d’événements de télémétrie, des opérations et de diagnostic, situés sous Applications and Services\Microsoft\FileSync\Agent.
    1. Vérifiez que le ou les fichiers se trouvent dans le partage de fichiers Azure.
    2. Vérifiez la connectivité Internet du serveur. 
    3. Ouvrez le composant logiciel enfichable MMC des services et vérifiez que le service de l’Agent de synchronisation du stockage (FileSyncSvc) est en cours d’exécution.
    4. Vérifiez que les pilotes de filtre Azure File Sync (StorageSync.sys et StorageSyncGuard.sys) sont en cours d’exécution :
        - À partir d’une invite de commandes avec élévation de privilèges, exécutez `fltmc`. Vérifiez que les pilotes de filtre du système de fichiers StorageSync.sys et StorageSyncGuard.sys sont répertoriés.

> [!NOTE]
> Un d’ID d’événement 9006 est enregistré une fois par heure dans le journal d’événements de télémétrie si un fichier ne parvient pas à rappeler (un événement est enregistré par code d’erreur). Consultez la section [Erreurs de rappel et correction](#recall-errors-and-remediation) pour vérifier si des étapes de correction sont fournies pour le code d’erreur.

### <a name="recall-errors-and-remediation"></a>Erreurs de rappel et correction

| HRESULT | HRESULT (décimal) | Chaîne d’erreur | Problème | Correction |
|---------|-------------------|--------------|-------|-------------|
| 0x80070079 | -2147942521 | ERROR_SEM_TIMEOUT | Le rappel du fichier a échoué à cause d’un délai d’expiration E/S. Ce problème peut se produire pour plusieurs raisons : contraintes de ressources du serveur, connectivité réseau médiocre ou problème de stockage Azure (par exemple une limitation). | Aucune action requise. Si l’erreur persiste pendant plusieurs heures, veuillez ouvrir un cas de support. |
| 0x80070036 | -2147024842 | ERROR_NETWORK_BUSY | Le rappel du fichier a échoué à cause d’un problème réseau.  | Si l’erreur persiste, vérifiez la connectivité réseau vers le partage de fichiers Azure. |
| 0x80c80037 | -2134376393 | ECS_E_SYNC_SHARE_NOT_FOUND | Le rappel du fichier a échoué car le point de terminaison de serveur a été supprimé. | Pour résoudre ce problème, consultez [Fichiers hiérarchisés non accessibles sur le serveur après la suppression d’un point de terminaison de serveur](?tabs=portal1%252cazure-portal#tiered-files-are-not-accessible-on-the-server-after-deleting-a-server-endpoint). |
| 0x80070005 | -2147024891 | ERROR_ACCESS_DENIED | Le rappel du fichier a échoué à cause d’une erreur de refus d’accès. Ce problème peut se produire si les paramètres du pare-feu et du réseau virtuel sur le compte de stockage sont activés et que le serveur n’a pas accès au compte de stockage. | Pour résoudre ce problème, ajoutez le réseau virtuel ou l’adresse IP du serveur en suivant les étapes décrites dans la section [Configurer les paramètres de pare-feu et de réseau virtuel](./storage-sync-files-deployment-guide.md?tabs=azure-portal#configure-firewall-and-virtual-network-settings) du Guide de déploiement. |
| 0x80c86002 | -2134351870 | ECS_E_AZURE_RESOURCE_NOT_FOUND | Le rappel du fichier a échoué car il n’est pas accessible dans le partage de fichiers Azure. | Pour résoudre ce problème, vérifiez que le fichier existe dans le partage de fichiers Azure. Si le fichier existe dans le partage de fichiers Azure, effectuez une mise à niveau vers la dernière [version de l’agent](./storage-files-release-notes.md#supported-versions) Azure File Sync. |
| 0x80c8305f | -2134364065 | ECS_E_EXTERNAL_STORAGE_ACCOUNT_AUTHORIZATION_FAILED | Le rappel du fichier a échoué en raison d’un échec d’autorisation auprès du compte de stockage. | Pour résoudre ce problème, vérifiez qu’[Azure File Sync a accès au compte de stockage](?tabs=portal1%252cazure-portal#troubleshoot-rbac). |
| 0x80c86030 | -2134351824 | ECS_E_AZURE_FILE_SHARE_NOT_FOUND | Le rappel du fichier a échoué car le partage de fichiers Azure n’est pas accessible. | Vérifiez que le partage de fichiers existe et qu’il est accessible. Si le partage de fichiers a été supprimé et recréé, suivez les étapes décrites dans la section [La synchronisation a échoué car le partage de fichiers Azure a été supprimé et recréé](?tabs=portal1%252cazure-portal#-2134375810) pour supprimer et recréer le groupe de synchronisation. |
| 0x800705aa | -2147023446 | ERROR_NO_SYSTEM_RESOURCES | Le rappel du fichier a échoué car les ressources système sont insuffisantes. | Si l’erreur persiste, essayez d’identifier l’application ou le pilote en mode noyau qui épuise les ressources système. |
| 0x8007000e | -2147024882 | ERROR_OUTOFMEMORY | Le rappel du fichier a échoué parce que la mémoire est insuffisante. | Si l’erreur persiste, essayez d’identifier l’application ou le pilote en mode noyau à l’origine de l’insuffisance de mémoire. |
| 0x80070070 | -2147024784 | ERROR_DISK_FULL | Le rappel du fichier a échoué car l’espace disque est insuffisant. | Pour résoudre ce problème, libérez de l’espace sur le volume en déplaçant des fichiers vers un autre volume, augmentez la taille du volume ou forcez la hiérarchisation des fichiers à l’aide de l’applet de commande Invoke-StorageSyncCloudTiering. |

### <a name="tiered-files-are-not-accessible-on-the-server-after-deleting-a-server-endpoint"></a>Les fichiers hiérarchisés ne sont pas accessibles sur le serveur après la suppression d’un point de terminaison de serveur
Les fichiers hiérarchisés sur un serveur sont inaccessibles si les fichiers ne sont pas rappelés avant la suppression du point de terminaison.

Erreurs consignées si les fichiers hiérarchisés ne sont pas accessibles
- Lors de la synchronisation d’un fichier, le code d’erreur-2147942467 (0x80070043-ERROR_BAD_NET_NAME) est enregistré dans le journal des événements ItemResults
- Lors du rappel d’un fichier, le code d’erreur-2134376393 (0x80c80037-ECS_E_SYNC_SHARE_NOT_FOUND) est enregistré dans le journal des événements RecallResults

La restauration de l’accès à vos fichiers hiérarchisés est possible si les conditions suivantes sont remplies :
- Le point de terminaison de serveur a été supprimé au cours des 30 derniers jours
- Le point de terminaison Cloud n’a pas été supprimé 
- Le partage de fichiers n’a pas été supprimé
- Le groupe de synchronisation n’a pas été supprimé

Si les conditions ci-dessus sont remplies, vous pouvez restaurer l’accès aux fichiers sur le serveur en recréant le point de terminaison de serveur sur le même chemin d’accès sur le serveur au sein du même groupe de synchronisation dans les 30 jours. 

Si les conditions ci-dessus ne sont pas remplies, la restauration de l’accès n’est pas possible, car ces fichiers hiérarchisés sur le serveur sont désormais orphelins. Suivez les instructions ci-dessous pour supprimer les fichiers hiérarchisés orphelins.

**Remarques**
- Lorsque les fichiers hiérarchisés ne sont pas accessibles sur le serveur, le fichier complet doit toujours être accessible si vous accédez directement au partage de fichiers Azure.
- Pour empêcher les fichiers hiérarchisés orphelins à l’avenir, suivez les étapes décrites dans [Supprimer un point de terminaison de serveur](./storage-sync-files-server-endpoint.md#remove-a-server-endpoint) lors de la suppression d’un point de terminaison de serveur.

<a id="get-orphaned"></a>**Obtention de la liste des fichiers hiérarchisés orphelins** 

1. Vérifiez que la version v5.1 (ou ultérieure) de l’agent Azure File Sync est installée.
2. Exécutez les commandes PowerShell suivantes pour répertorier les fichiers hiérarchisés orphelins :
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
$orphanFiles = Get-StorageSyncOrphanedTieredFiles -path <server endpoint path>
$orphanFiles.OrphanedTieredFiles > OrphanTieredFiles.txt
```
3. Enregistrez le fichier de sortie OrphanTieredFiles.txt au cas où les fichiers doivent être restaurés à partir d’une sauvegarde après avoir été supprimés.

<a id="remove-orphaned"></a>**Comment supprimer des fichiers hiérarchisés orphelins** 

*Option n°1 : supprimer les fichiers hiérarchisés orphelins*

Cette option supprime les fichiers hiérarchisés orphelins sur le serveur Windows, mais nécessite de supprimer le point de terminaison de serveur s’il existe en raison de la récréation après 30 jours ou s’il est connecté à un autre groupe de synchronisation. Des conflits de fichiers se produisent si des fichiers sont mis à jour sur le partage de fichiers Windows Server ou Azure avant la recréation du point de terminaison de serveur.

1. Vérifiez que la version v5.1 (ou ultérieure) de l’agent Azure File Sync est installée.
2. Sauvegardez le partage de fichiers Azure et l’emplacement du point de terminaison de serveur.
3. Supprimez le point de terminaison de serveur dans le groupe de synchronisation (s’il existe) en suivant les étapes décrites dans [Supprimer un point de terminaison de serveur](./storage-sync-files-server-endpoint.md#remove-a-server-endpoint).

> [!Warning]  
> Si le point de terminaison de serveur n’est pas supprimé avant d’utiliser l’applet de commande Remove-StorageSyncOrphanedTieredFiles, la suppression du fichier hiérarchisé orphelin sur le serveur supprimera le fichier complet dans le partage de fichiers Azure. 

4. Exécutez les commandes PowerShell suivantes pour répertorier les fichiers hiérarchisés orphelins :

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
$orphanFiles = Get-StorageSyncOrphanedTieredFiles -path <server endpoint path>
$orphanFiles.OrphanedTieredFiles > OrphanTieredFiles.txt
```
5. Enregistrez le fichier de sortie OrphanTieredFiles.txt au cas où les fichiers doivent être restaurés à partir d’une sauvegarde après avoir été supprimés.
6. Exécutez les commandes PowerShell suivantes pour supprimer les fichiers hiérarchisés orphelins :

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
$orphanFilesRemoved = Remove-StorageSyncOrphanedTieredFiles -Path <folder path containing orphaned tiered files> -Verbose
$orphanFilesRemoved.OrphanedTieredFiles > DeletedOrphanFiles.txt
```
**Remarques** 
- Les fichiers hiérarchisés modifiés sur le serveur qui ne sont pas synchronisés avec le partage de fichiers Azure seront supprimés.
- Les fichiers hiérarchisés qui sont accessibles (non orphelins) ne seront pas supprimés.
- Les fichiers non hiérarchisés sont conservés sur le serveur.

7. Facultatif : Recréez le point de terminaison de serveur si vous l’avez supprimé à l’étape 3.

*Option n°2 : Montez le partage de fichiers Azure et copiez les fichiers localement qui sont orphelins sur le serveur*

Cette option ne nécessite pas la suppression du point de terminaison de serveur, mais nécessite un espace disque suffisant pour copier les fichiers complets localement.

1. [Montez](./storage-how-to-use-files-windows.md) le partage de fichiers Azure sur le serveur Windows qui contient des fichiers hiérarchisés orphelins.
2. Exécutez les commandes PowerShell suivantes pour répertorier les fichiers hiérarchisés orphelins :
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
$orphanFiles = Get-StorageSyncOrphanedTieredFiles -path <server endpoint path>
$orphanFiles.OrphanedTieredFiles > OrphanTieredFiles.txt
```
3. Utilisez le fichier de sortie OrphanTieredFiles.txt pour identifiez les fichiers hiérarchisés orphelins sur le serveur.
4. Remplacez les fichiers hiérarchisés orphelins en copiant le fichier complet du partage de fichiers Azure sur le serveur Windows.

### <a name="how-to-troubleshoot-files-unexpectedly-recalled-on-a-server"></a>Comment résoudre les problèmes de rappel de fichiers inattendu sur un serveur  
Les antivirus, applications de sauvegarde et autres applications qui lisent un grand nombre de fichiers provoquent des rappels inattendus, sauf s’ils ont été configurés pour ignorer la lecture du contenu des fichiers hors connexion. Ignorer les fichiers hors connexion pour les produits qui prennent en charge cette option permet d’éviter des rappels inattendus pendant les opérations telles que les analyses antivirus ou les travaux de sauvegarde.

Contactez votre éditeur de logiciel pour savoir comment configurer la solution de façon à ignorer la lecture des fichiers hors connexion.

Des rappels inattendus peuvent également se produire dans d’autres scénarios, par exemple, quand vous parcourez des fichiers dans l’Explorateur de fichiers. L’ouverture d’un dossier contenant des fichiers cloud hiérarchisés dans l’Explorateur de fichiers sur le serveur peut provoquer des rappels inattendus. Le risque est d’autant plus grand si une solution antivirus est activée sur le serveur.

> [!NOTE]
>Utilisez l’ID d’événement 9059 dans le journal d’événements de télémétrie pour déterminer quelles sont les applications à l’origine des rappels. Cet événement fournit la distribution des rappels d’application pour un point de terminaison de serveur, et est enregistré une fois par heure.

### <a name="tls-12-required-for-azure-file-sync"></a>Protocole TLS 1.2 requis pour Azure File Sync

Vous pouvez afficher les paramètres TLS sur votre serveur en examinant les [paramètres du Registre](/windows-server/security/tls/tls-registry-settings). 

Si vous utilisez un proxy, consultez la documentation de votre proxy et assurez-vous qu’il est configuré pour utiliser le protocole TLS 1.2.

## <a name="general-troubleshooting"></a>Résolution générale des problèmes
Si vous rencontrez des problèmes avec Azure File Sync sur un serveur, commencez par effectuer les étapes suivantes :
1. Dans l’observateur d’événements, examinez les journaux d’événements de télémétrie, des opérations et de diagnostic.
    - Les problèmes de synchronisation, de hiérarchisation et de rappel sont enregistrés dans les journaux d’événements de télémétrie, de diagnostic et d’exploitation sous Applications and services\Microsoft\FileSync\Agent.
    - Les problèmes liés à la gestion d’un serveur (par exemple, les paramètres de configuration) sont enregistrés dans ces journaux des événements opérationnels et de diagnostic sous Applications et Services\Microsoft\FileSync\Management.
2. Vérifiez que le service Azure File Sync est en cours d’exécution sur le serveur :
    - Ouvrez le composant logiciel enfichable MMC des services et vérifiez que le service Storage Sync Agent (FileSyncSvc) est en cours d’exécution.
3. Vérifiez que les pilotes de filtre Azure File Sync (StorageSync.sys et StorageSyncGuard.sys) sont en cours d’exécution :
    - À partir d’une invite de commandes avec élévation de privilèges, exécutez `fltmc`. Vérifiez que les pilotes de filtre du système de fichiers StorageSync.sys et StorageSyncGuard.sys sont répertoriés.

Si le problème n’est pas résolu, exécutez l’outil AFSDiag et envoyez sa sortie au format de fichier .zip à l’ingénieur de support affecté à votre cas pour lui permettre d’effectuer un diagnostic plus approfondi.

Pour exécuter AFSDiag, procédez comme suit.

Pour les versions v11 et ultérieures de l’agent :
1. Ouvrez une fenêtre PowerShell avec élévation de privilèges, puis exécutez les commandes suivantes (appuyez sur Entrée après chaque commande) :

    > [!NOTE]
    >AFSDiag crée le répertoire de sortie et un dossier temporaire au sein de celui-ci avant de collecter les journaux, puis supprime le dossier temporaire une fois l’opération terminée. Spécifiez un emplacement de sortie qui ne contient pas de données.
    
    ```powershell
    cd "c:\Program Files\Azure\StorageSyncAgent"
    Import-Module .\afsdiag.ps1
    Debug-AFS -OutputDirectory C:\output -KernelModeTraceLevel Verbose -UserModeTraceLevel Verbose
    ```

2. Reproduisez le problème. Quand vous avez terminé, entrez **D**.
3. Un fichier .zip contenant les journaux d’activité et les fichiers de trace est enregistré dans le répertoire de sortie que vous avez spécifié. 

Pour les versions v10 et antérieures de l’agent :
1. Créez un répertoire où la sortie AFSDiag doit être enregistrée (par exemple, C:\Output).
    > [!NOTE]
    >AFSDiag supprimera tout le contenu du répertoire de sortie avant la collecte des journaux. Spécifiez un emplacement de sortie qui ne contient pas de données.
2. Ouvrez une fenêtre PowerShell avec élévation de privilèges, puis exécutez les commandes suivantes (appuyez sur Entrée après chaque commande) :

    ```powershell
    cd "c:\Program Files\Azure\StorageSyncAgent"
    Import-Module .\afsdiag.ps1
    Debug-Afs c:\output # Note: Use the path created in step 1.
    ```

3. Pour le niveau de trace du mode noyau d’Azure File Sync, entrez **1** (sauf indication contraire, pour créer des traces plus détaillées), puis appuyez sur Entrée.
4. Pour le niveau de trace du mode utilisateur d’Azure File Sync, entrez **1** (sauf indication contraire, pour créer des traces plus détaillées), puis appuyez sur Entrée.
5. Reproduisez le problème. Quand vous avez terminé, entrez **D**.
6. Un fichier .zip contenant les journaux d’activité et les fichiers de trace est enregistré dans le répertoire de sortie que vous avez spécifié.


## <a name="see-also"></a>Voir aussi
- [Superviser Azure File Sync](storage-sync-files-monitoring.md)
- [Forum Aux Questions Azure Files](storage-files-faq.md)
- [Résoudre les problèmes liés à Azure Files sous Windows](storage-troubleshoot-windows-file-connection-problems.md)
- [Résoudre les problèmes liés à Azure Files dans Linux](storage-troubleshoot-linux-file-connection-problems.md)
