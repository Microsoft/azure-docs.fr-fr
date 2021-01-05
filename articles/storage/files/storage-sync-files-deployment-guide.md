---
title: Déployer Azure File Sync | Microsoft Docs
description: Apprenez à déployer Azure File Sync, du début à la fin, à l’aide du portail Azure, de PowerShell ou de l’interface de ligne de commande Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 11/05/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: d39f26d86792214c1ef0300bc39404bf6581826f
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629476"
---
# <a name="deploy-azure-file-sync"></a>Déployer Azure File Sync
Utilisez Azure File Sync pour centraliser les partages de fichiers de votre organisation dans Azure Files tout en conservant la flexibilité, le niveau de performance et la compatibilité d’un serveur de fichiers local. Azure File Sync transforme Windows Server en un cache rapide de votre partage de fichiers Azure. Vous pouvez utiliser tout protocole disponible dans Windows Server pour accéder à vos données localement, notamment SMB, NFS et FTPS. Vous pouvez avoir autant de caches que nécessaire dans le monde entier.

Nous vous recommandons fortement de lire les articles [Planification d’un déploiement Azure Files](storage-files-planning.md) et [Planification d’un déploiement de synchronisation de fichiers Azure](storage-sync-files-planning.md) avant d’effectuer les étapes décrites dans cet article.

## <a name="prerequisites"></a>Prérequis

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Un partage de fichiers Azure situé dans la région où vous souhaitez déployer Azure File Sync. Pour plus d'informations, consultez les pages suivantes :
    - [Disponibilité des régions](storage-sync-files-planning.md#azure-file-sync-region-availability) pour Azure File Sync.
    - [Créer un partage de fichiers](storage-how-to-create-file-share.md) pour obtenir une procédure pas à pas de la création d’un partage de fichiers.
1. Avoir au moins une instance de Windows Server ou d’un cluster Windows Server prise en charge pour la synchronisation avec Azure File Sync. Pour plus d’informations sur les versions prises en charge de Windows Server et les ressources système recommandées, consultez [Considérations relatives à Windows File Server](storage-sync-files-planning.md#windows-file-server-considerations).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Un partage de fichiers Azure situé dans la région où vous souhaitez déployer Azure File Sync. Pour plus d'informations, consultez les pages suivantes :
    - [Disponibilité des régions](storage-sync-files-planning.md#azure-file-sync-region-availability) pour Azure File Sync.
    - [Créer un partage de fichiers](storage-how-to-create-file-share.md) pour obtenir une procédure pas à pas de la création d’un partage de fichiers.
1. Avoir au moins une instance de Windows Server ou d’un cluster Windows Server prise en charge pour la synchronisation avec Azure File Sync. Pour plus d’informations sur les versions prises en charge de Windows Server et les ressources système recommandées, consultez [Considérations relatives à Windows File Server](storage-sync-files-planning.md#windows-file-server-considerations).

1. Le module Az PowerShell peut être utilisé avec PowerShell 5.1 ou PowerShell 6+. Vous pouvez utiliser le module Az PowerShell pour Azure File Sync sur n’importe quel système pris en charge, y compris des systèmes non Windows, mais l’applet de commende d’inscription du serveur doit toujours être exécutée sur l’instance Windows Server que vous inscrivez (cette opération peut être effectuée directement ou via l’accès distant PowerShell). Sur Windows Server 2012 R2, vous pouvez vérifier que vous exécutez au moins PowerShell 5.1.\* en regardant la valeur de la propriété **PSVersion** de l’objet **$PSVersionTable** :

    ```powershell
    $PSVersionTable.PSVersion
    ```

    Si la valeur **PSVersion** est inférieure à 5.1.\*, comme avec la plupart des nouvelles installations de Windows Server 2012 R2, vous pouvez facilement effectuer la mise à niveau en téléchargeant et en installant [Windows Management Framework (WMF) 5.1](https://www.microsoft.com/download/details.aspx?id=54616). Le package correct à télécharger et à installer pour Windows Server 2012 R2 est **Win8.1AndW2K12R2-KB\*\*\*\*\*\*\*-x64.msu**. 

    PowerShell 6+ peut être utilisé avec n’importe quel système pris en charge, et peut être téléchargé via sa page [GitHub](https://github.com/PowerShell/PowerShell#get-powershell). 

    > [!Important]  
    > Si vous prévoyez d’utiliser l’interface d’enregistrement du serveur, plutôt que de vous enregistrer directement depuis PowerShell, vous devez utiliser PowerShell 5.1.

1. Si vous avez choisi d’utiliser PowerShell 5.1, assurez-vous qu’au moins .NET 4.7.2 est installé. En savoir plus sur les [Versions et dépendances .NET Framework](/dotnet/framework/migration-guide/versions-and-dependencies) sur votre système.

    > [!Important]  
    > Si vous installez .NET 4.7.2+ sur Windows Server Core, vous devez effectuer l’installation avec les indicateurs `quiet` et `norestart` ; sinon, l’installation échoue. Par exemple, si vous installez .NET 4.8, la commande se présente comme ceci :
    > ```PowerShell
    > Start-Process -FilePath "ndp48-x86-x64-allos-enu.exe" -ArgumentList "/q /norestart" -Wait
    > ```

1. Le module Az PowerShell, qui peut être installé en suivant les instructions ici : [Installez et configurez Azure PowerShell](/powershell/azure/install-Az-ps).
     
    > [!Note]  
    > Le module Az.StorageSync est maintenant installé automatiquement quand vous installez le module Az PowerShell.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Un partage de fichiers Azure situé dans la région où vous souhaitez déployer Azure File Sync. Pour plus d'informations, consultez les pages suivantes :
    - [Disponibilité des régions](storage-sync-files-planning.md#azure-file-sync-region-availability) pour Azure File Sync.
    - [Créer un partage de fichiers](storage-how-to-create-file-share.md) pour obtenir une procédure pas à pas de la création d’un partage de fichiers.
1. Avoir au moins une instance de Windows Server ou d’un cluster Windows Server prise en charge pour la synchronisation avec Azure File Sync. Pour plus d’informations sur les versions prises en charge de Windows Server et les ressources système recommandées, consultez [Considérations relatives à Windows File Server](storage-sync-files-planning.md#windows-file-server-considerations).

1. [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli)

   Si vous préférez, vous pouvez également utiliser Azure Cloud Shell pour effectuer les étapes décrites dans ce didacticiel.  Azure Cloud Shell est un environnement d’interpréteur de commandes interactif que vous utilisez dans votre navigateur.  Démarrez Cloud Shell à l’aide de l’une des méthodes suivantes :

   - Sélectionnez **Essayer** dans le coin supérieur droit d’un bloc de code. **Essayer** ouvre Azure Cloud Shell, mais n’y copie pas automatiquement le code.

   - Ouvrez Cloud Shell en accédant à [https://shell.azure.com](https://shell.azure.com).

   - Sélectionnez le bouton **Cloud Shell** dans la barre de menus en haut à droite du [portail Azure](https://portal.azure.com).

1. Connectez-vous.

   Connectez-vous à l’aide de la commande [az login](/cli/azure/reference-index#az-login) si vous utilisez une installation locale de l’interface CLI.

   ```azurecli
   az login
   ```

    Suivez les étapes affichées dans votre terminal pour effectuer le processus d’authentification.

1. Installez l’extension Azure CLI [az filesync](/cli/azure/ext/storagesync/storagesync).

   ```azurecli
   az extension add --name storagesync
   ```

   Après l’installation de la référence de l’extension **storagesync**, l’avertissement suivant s’affiche.

   ```output
   The installed extension 'storagesync' is experimental and not covered by customer support. Please use with discretion.
   ```

---

## <a name="prepare-windows-server-to-use-with-azure-file-sync"></a>Préparer Windows Server pour une utilisation avec Azure File Sync
Pour chaque serveur que vous souhaitez utiliser avec Azure File Sync, y compris chaque nœud de serveur dans un cluster de basculement, vous devez désactiver **Configuration de sécurité renforcée d’Internet Explorer**. Cela est nécessaire uniquement pour l’inscription initiale du serveur. Vous pouvez réactiver cette configuration une fois que le serveur a été inscrit.

# <a name="portal"></a>[Portail](#tab/azure-portal)
> [!Note]  
> Vous pouvez ignorer cette étape si vous déployez Azure File Sync sur Windows Server Core.

1. Ouvrez le Gestionnaire de serveurs.
2. Cliquez sur **Serveur Local** :  
    ![« Serveur local » sur le côté gauche de l’interface utilisateur du Gestionnaire de serveur](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-1.PNG)
3. Dans le sous-volet **Propriétés**, sélectionnez le lien vers **Configuration de sécurité renforcée d’Internet Explorer**.  
    ![Volet « Configuration de sécurité renforcée d’Internet Explorer » dans l’interface utilisateur du Gestionnaire de serveur](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-2.PNG)
4. Dans la boîte de dialogue **Configuration de sécurité renforcée d’Internet Explorer**, sélectionnez **Désactivé** pour **Administrateurs** et **Utilisateurs** :  
    ![Fenêtre indépendante Configuration de sécurité renforcée d’Internet Explorer avec l’option « Désactivé » sélectionnée](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-3.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Pour désactiver la configuration de sécurité renforcée d’Internet Explorer, exécutez la commande suivante à partir d’une session PowerShell avec élévation de privilèges :

```powershell
$installType = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\").InstallationType

# This step is not required for Server Core
if ($installType -ne "Server Core") {
    # Disable Internet Explorer Enhanced Security Configuration 
    # for Administrators
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0 -Force
    
    # Disable Internet Explorer Enhanced Security Configuration 
    # for Users
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A8-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0 -Force
    
    # Force Internet Explorer closed, if open. This is required to fully apply the setting.
    # Save any work you have open in the IE browser. This will not affect other browsers,
    # including Microsoft Edge.
    Stop-Process -Name iexplore -ErrorAction SilentlyContinue
}
``` 

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Suivez les instructions pour le portail Azure ou PowerShell.

---

## <a name="deploy-the-storage-sync-service"></a>Déployer le service de synchronisation de stockage 
Le déploiement de Azure File Sync commence par le placement d’une ressource **Service de synchronisation de stockage** dans un groupe de ressources de votre abonnement sélectionné. Nous vous recommandons d’en approvisionner le plus petit nombre possible. Vous allez créer une relation d’approbation entre vos serveurs et cette ressource et un serveur ne peut être inscrit qu’auprès d’un seul service de synchronisation de stockage. Par conséquent, il est recommandé de déployer des services de synchronisation de stockage autant que nécessaire pour séparer les groupes de serveurs. Gardez à l’esprit que les serveurs de services de synchronisation de stockage différents ne peuvent pas être synchronisés entre eux.

> [!Note]
> Le service de synchronisation de stockage hérite des autorisations d’accès de l’abonnement et du groupe de ressources dans lequel il a été déployé. Nous vous recommandons de vérifier soigneusement les personnes qui y ont accès. Les entités avec un accès en écriture peuvent démarrer la synchronisation de nouveaux ensembles de fichiers à partir des serveurs inscrits dans ce service de synchronisation de stockage et permettre le flux de données vers le stockage Azure qui leur est accessible.

# <a name="portal"></a>[Portail](#tab/azure-portal)
Pour déployer un service de synchronisation de stockage, accédez au [Portail Microsoft Azure](https://portal.azure.com/), cliquez sur *Créer une ressource*, puis recherchez Azure File Sync. Dans les résultats de la recherche, sélectionnez **Azure File Sync**, puis sélectionnez **Créer** pour ouvrir l’onglet **Déployer la synchronisation du stockage**.

Dans le volet qui s’ouvre, entrez les informations suivantes :

- **Name** : Nom unique (par région) du service de synchronisation de stockage.
- **Abonnement**: abonnement dans lequel vous voulez créer le service de synchronisation de stockage. Selon la stratégie de configuration de votre organisation, vous avez accès à un ou plusieurs abonnements. Un abonnement Azure est le conteneur de base pour la facturation de chaque service cloud (par exemple, Azure Files).
- **Groupe de ressources** : un groupe de ressources est un groupe logique de ressources Azure, tel qu’un compte de stockage ou un service de synchronisation de stockage. Vous pouvez créer un groupe de ressources ou utiliser un groupe de ressources existant pour Azure File Sync. (Nous vous recommandons d’utiliser des groupes de ressources comme conteneurs pour isoler les ressources logiquement dans votre organisation, par exemple, en regroupant les ressources humaines ou les ressources d’un projet spécifique.)
- **Emplacement** : région dans laquelle vous souhaitez déployer Azure File Sync. Seuls les régions prises en charge sont disponibles dans cette liste.

Quand vous avez terminé, sélectionnez **Créer** pour déployer le service de synchronisation de stockage.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Remplacez `<Az_Region>`, `<RG_Name>` et `<my_storage_sync_service>` par vos propres valeurs, puis utilisez les commandes suivantes pour créer et déployer un service Synchronisation de stockage :

```powershell
$hostType = (Get-Host).Name

if ($installType -eq "Server Core" -or $hostType -eq "ServerRemoteHost") {
    Connect-AzAccount -UseDeviceAuthentication
}
else {
    Connect-AzAccount
}

# this variable holds the Azure region you want to deploy 
# Azure File Sync into
$region = '<Az_Region>'

# Check to ensure Azure File Sync is available in the selected Azure
# region.
$regions = @()
Get-AzLocation | ForEach-Object { 
    if ($_.Providers -contains "Microsoft.StorageSync") { 
        $regions += $_.Location 
    } 
}

if ($regions -notcontains $region) {
    throw [System.Exception]::new("Azure File Sync is either not available in the selected Azure Region or the region is mistyped.")
}

# the resource group to deploy the Storage Sync Service into
$resourceGroup = '<RG_Name>'

# Check to ensure resource group exists and create it if doesn't
$resourceGroups = @()
Get-AzResourceGroup | ForEach-Object { 
    $resourceGroups += $_.ResourceGroupName 
}

if ($resourceGroups -notcontains $resourceGroup) {
    New-AzResourceGroup -Name $resourceGroup -Location $region
}

$storageSyncName = "<my_storage_sync_service>"
$storageSync = New-AzStorageSyncService -ResourceGroupName $resourceGroup -Name $storageSyncName -Location $region
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Suivez les instructions pour le portail Azure ou PowerShell.

---

## <a name="install-the-azure-file-sync-agent"></a>Installer l’agent Azure File Sync
L’agent Azure File Sync est un package téléchargeable qui permet à Windows Server de rester synchronisé avec un partage de fichiers Azure. 

# <a name="portal"></a>[Portail](#tab/azure-portal)
Vous pouvez télécharger l’agent à partir du [Centre de téléchargement Microsoft](https://go.microsoft.com/fwlink/?linkid=858257). À la fin du téléchargement, double-cliquez sur le package MSI pour démarrer l’installation de l’agent Azure File Sync.

> [!Important]  
> Si vous comptez utiliser Azure File Sync avec un cluster de basculement, l’agent Azure File Sync doit être installé sur chaque nœud du cluster. Chaque nœud du cluster doit être inscrit pour fonctionner avec Azure File Sync.

Nous vous recommandons de procéder comme suit :
- Conservez le chemin d’installation par défaut (C:\Program Files\Azure\StorageSyncAgent), pour simplifier le dépannage et la maintenance du serveur.
- Activez Microsoft Update pour garantir la mise à jour d’Azure File Sync. Toutes les mises à jour de l’agent Azure File Sync, y compris les correctifs logiciels et les mises à jour de fonctionnalités, s’effectuent à partir de Microsoft Update. Nous vous recommandons d’installer la dernière mise à jour d’Azure File Sync. Pour plus d’informations, consultez [Stratégie de mise à jour d’Azure File Sync](storage-sync-files-planning.md#azure-file-sync-agent-update-policy).

Au terme de l’installation de l’agent Azure File Sync, l’interface utilisateur d’inscription du serveur s’ouvre automatiquement. Vous devez disposer d’un service de synchronisation de stockage avant de procéder à l’inscription. Consultez la section suivante sur la création d’un service de synchronisation de stockage.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Exécutez le code PowerShell suivant pour télécharger la version appropriée de l’agent Azure File Sync pour votre système d’exploitation et l’installer sur votre système.

> [!Important]  
> Si vous comptez utiliser Azure File Sync avec un cluster de basculement, l’agent Azure File Sync doit être installé sur chaque nœud du cluster. Chaque nœud du cluster doit être inscrit pour fonctionner avec Azure File Sync.

```powershell
# Gather the OS version
$osver = [System.Environment]::OSVersion.Version

# Download the appropriate version of the Azure File Sync agent for your OS.
if ($osver.Equals([System.Version]::new(10, 0, 17763, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2019 `
        -OutFile "StorageSyncAgent.msi" 
} elseif ($osver.Equals([System.Version]::new(10, 0, 14393, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2016 `
        -OutFile "StorageSyncAgent.msi" 
} elseif ($osver.Equals([System.Version]::new(6, 3, 9600, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2012R2 `
        -OutFile "StorageSyncAgent.msi" 
} else {
    throw [System.PlatformNotSupportedException]::new("Azure File Sync is only supported on Windows Server 2012 R2, Windows Server 2016, and Windows Server 2019")
}

# Install the MSI. Start-Process is used to PowerShell blocks until the operation is complete.
# Note that the installer currently forces all PowerShell sessions closed - this is a known issue.
Start-Process -FilePath "StorageSyncAgent.msi" -ArgumentList "/quiet" -Wait

# Note that this cmdlet will need to be run in a new session based on the above comment.
# You may remove the temp folder containing the MSI and the EXE installer
Remove-Item -Path ".\StorageSyncAgent.msi" -Recurse -Force
```
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Suivez les instructions pour le portail Azure ou PowerShell.

---

## <a name="register-windows-server-with-storage-sync-service"></a>Inscrire un serveur Windows Server au service de synchronisation de stockage
L’inscription de votre serveur Windows Server à un service de synchronisation de stockage établit une relation d’approbation entre votre serveur (ou cluster) et le service de synchronisation de stockage. Un serveur ne peut être inscrit qu’auprès d’un seul service de synchronisation du stockage et peut se synchroniser avec d’autres serveurs et avec des partages de fichiers Azure associés avec le même service de synchronisation de stockage.

> [!Note]
> L’inscription du serveur utilise vos informations d’identification Azure pour créer une relation d’approbation entre le service de synchronisation du stockage et votre serveur Windows, toutefois par la suite le serveur crée et utilise sa propre identité qui est valide tant que le serveur reste inscrit et le jeton de signature d’accès partagé actuel (stockage SAS) est valide. Un nouveau jeton SAS ne peut pas être émis sur le serveur une fois le serveur désinscrit, retirant ainsi la capacité du serveur à accéder à vos partages de fichiers Azure, arrêtant toute synchronisation.

L’administrateur qui inscrit le serveur doit être membre du rôle de gestion **Propriétaire** ou **Collaborateur** pour le service de synchronisation du stockage en question. Ce paramètre peut être configuré sous **Access Control (IAM)** sur le Portail Azure pour le service de synchronisation du stockage.

Il est également possible de différencier les administrateurs qui peuvent inscrire des serveurs de ceux qui sont également autorisés à configurer la synchronisation dans un service de synchronisation du stockage. Pour cela, vous devez créer un rôle personnalisé dans lequel vous listez les administrateurs qui ne sont autorisés qu’à inscrire des serveurs et auquel vous attribuez les autorisations suivantes :

* "Microsoft.StorageSync/storageSyncServices/registeredServers/write"
* "Microsoft.StorageSync/storageSyncServices/read"
* "Microsoft.StorageSync/storageSyncServices/workflows/read"
* "Microsoft.StorageSync/storageSyncServices/workflows/operations/read"

# <a name="portal"></a>[Portail](#tab/azure-portal)
L’interface utilisateur d’inscription du serveur s’ouvre normalement automatiquement après l’installation de l’agent Azure File Sync. Si ce n’est pas le cas, vous pouvez l’ouvrir manuellement à partir de son emplacement de fichier : C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe. Quand l’interface utilisateur d’inscription du serveur est ouverte, sélectionnez **Se connecter** pour commencer.

Une fois que vous êtes connecté, vous êtes invité à fournir les informations suivantes :

![Capture d’écran de l’interface utilisateur d’inscription de serveur](media/storage-sync-files-deployment-guide/register-server-scubed-1.png)

- **Abonnement Azure** : abonnement qui contient le service de synchronisation de stockage (consultez [Déployer le service de synchronisation de stockage](#deploy-the-storage-sync-service)). 
- **Groupe de ressources** : groupe de ressources qui contient le service de synchronisation de stockage.
- **Service de synchronisation de stockage** : nom du service de synchronisation de stockage auquel vous souhaitez vous inscrire.

Après avoir sélectionné les informations appropriées, sélectionnez **S’inscrire** pour effectuer l’inscription du serveur. Dans le cadre du processus d’inscription, vous êtes invité à effectuer une nouvelle connexion.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
$registeredServer = Register-AzStorageSyncServer -ParentObject $storageSync
```
# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Suivez les instructions pour le portail Azure ou PowerShell.

---

## <a name="create-a-sync-group-and-a-cloud-endpoint"></a>Créer un groupe de synchronisation et un point de terminaison du cloud
Un groupe de synchronisation définit la topologie de synchronisation d’un ensemble de fichiers. Les points de terminaison dans un groupe de synchronisation sont synchronisés entre eux. Un groupe de synchronisation doit contenir un seul point de terminaison cloud, qui représente un partage de fichiers Azure, et un ou plusieurs points de terminaison de serveur. Un point de terminaison de serveur représente un chemin d’accès vers un serveur inscrit. Un serveur peut avoir des points de terminaison de serveur dans plusieurs groupes de synchronisation. Vous pouvez créer autant de groupes de synchronisation que nécessaire pour décrire de manière appropriée votre topologie de synchronisation.

Un point de terminaison cloud est un pointeur vers un partage de fichiers Azure. Tous les points de terminaison de serveur seront synchronisés avec un point de terminaison cloud, en faisant ainsi un concentrateur. Le compte de stockage pour le partage de fichiers Azure doit se trouver dans la même région que le service de synchronisation de stockage. L’intégralité du partage de fichiers Azure est synchronisée, à une exception près : un dossier spécial, comparable au dossier « System Volume Information » masqué sur un volume NTFS, peut être configuré. Ce répertoire est appelé «.SystemShareInformation». Il contient des métadonnées de synchronisation importantes qui ne seront pas synchronisées avec d’autres points de terminaison. Ne pas l’utiliser ni le supprimer !

> [!Important]  
> Vous pouvez apporter des modifications à un point de terminaison cloud ou un point de terminaison de serveur dans le groupe de synchronisation, et synchroniser vos fichiers avec les autres points de terminaison du groupe de synchronisation. Si vous apportez une modification au point de terminaison cloud (partage de fichiers Azure) directement, cette modification doit être détectée au préalable par un travail de détection des modifications Azure File Sync. Un travail de détection des modifications est lancé pour un point de terminaison cloud toutes les 24 heures uniquement. Pour plus d’informations, consultez [Questions fréquentes (FAQ) sur Azure Files](storage-files-faq.md#afs-change-detection).

L’administrateur qui crée le point de terminaison cloud doit être membre du rôle de gestion **Propriétaire** pour le compte de stockage contenant le partage de fichiers Azure sur lequel pointe le point de terminaison cloud. Ce paramètre peut être configuré sous **Access Control (IAM)** sur le Portail Azure pour le compte de stockage.

# <a name="portal"></a>[Portail](#tab/azure-portal)
Pour créer un groupe de synchronisation, dans le [portail Azure](https://portal.azure.com/), accédez à votre service de synchronisation de stockage, puis sélectionnez **+ Groupe de synchronisation** :

![Créer un groupe de synchronisation dans le portail Azure](media/storage-sync-files-deployment-guide/create-sync-group-1.png)

Dans le volet qui s’ouvre, entrez les informations suivantes pour créer un groupe de synchronisation avec un point de terminaison cloud :

- **Nom du groupe de synchronisation** : nom du groupe de synchronisation à créer. Ce nom doit être unique au sein du service de synchronisation de stockage, mais vous pouvez choisir tout nom ayant un sens pour vous.
- **Abonnement**: abonnement dans lequel vous avez déployé le service de synchronisation de stockage, comme décrit dans [Déployer le service de synchronisation de stockage](#deploy-the-storage-sync-service).
- **Compte de stockage** : si vous sélectionnez **Sélectionner un compte de stockage**, un autre volet s’affiche dans lequel vous pouvez sélectionner le compte de stockage qui contient le partage de fichiers Azure avec lequel effectuer la synchronisation.
- **Partage de fichiers Azure** : nom du partage de fichiers Azure avec lequel vous voulez effectuer la synchronisation.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Pour créer le groupe de synchronisation, exécutez la commande PowerShell suivante. N’oubliez pas de remplacer `<my-sync-group>` par le nom du groupe de synchronisation.

```powershell
$syncGroupName = "<my-sync-group>"
$syncGroup = New-AzStorageSyncGroup -ParentObject $storageSync -Name $syncGroupName
```

Une fois le groupe de synchronisation créé avec succès, vous pouvez créer votre point de terminaison cloud. Veillez à remplacer `<my-storage-account>` et `<my-file-share>` par les valeurs attendues.

```powershell
# Get or create a storage account with desired name
$storageAccountName = "<my-storage-account>"
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroup | Where-Object {
    $_.StorageAccountName -eq $storageAccountName
}

if ($storageAccount -eq $null) {
    $storageAccount = New-AzStorageAccount `
        -Name $storageAccountName `
        -ResourceGroupName $resourceGroup `
        -Location $region `
        -SkuName Standard_LRS `
        -Kind StorageV2 `
        -EnableHttpsTrafficOnly:$true
}

# Get or create an Azure file share within the desired storage account
$fileShareName = "<my-file-share>"
$fileShare = Get-AzStorageShare -Context $storageAccount.Context | Where-Object {
    $_.Name -eq $fileShareName -and $_.IsSnapshot -eq $false
}

if ($fileShare -eq $null) {
    $fileShare = New-AzStorageShare -Context $storageAccount.Context -Name $fileShareName
}

# Create the cloud endpoint
New-AzStorageSyncCloudEndpoint `
    -Name $fileShare.Name `
    -ParentObject $syncGroup `
    -StorageAccountResourceId $storageAccount.Id `
    -AzureFileShareName $fileShare.Name
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Utilisez la commande [az storagesync sync-group](/cli/azure/ext/storagesync/storagesync/sync-group#ext-storagesync-az-storagesync-sync-group-create) pour créer un groupe de synchronisation.  Pour définir un groupe de ressources par défaut pour toutes les commandes CLI, utilisez [az configure](/cli/azure/reference-index#az-configure).

```azurecli
az storagesync sync-group create --resource-group myResourceGroupName \
                                 --name myNewSyncGroupName \
                                 --storage-sync-service myStorageSyncServiceName \
```

Utilisez la commande [az storagesync sync-group cloud-endpoint](/cli/azure/ext/storagesync/storagesync/sync-group/cloud-endpoint#ext-storagesync-az-storagesync-sync-group-cloud-endpoint-create) pour créer un point de terminaison cloud.

```azurecli
az storagesync sync-group cloud-endpoint create --resource-group myResourceGroup \
                                                --storage-sync-service myStorageSyncServiceName \
                                                --sync-group-name mySyncGroupName \
                                                --name myNewCloudEndpointName \
                                                --storage-account mystorageaccountname \
                                                --azure-file-share-name azure-file-share-name
```

---

## <a name="create-a-server-endpoint"></a>Créer un point de terminaison de serveur
Un point de terminaison de serveur représente un emplacement spécifique sur un serveur inscrit, comme un dossier sur un volume de serveur. Un point de terminaison de serveur doit être un chemin d’accès sur un serveur inscrit (plutôt que sur un partage monté), et pour utiliser la hiérarchisation cloud, le chemin d’accès doit être sur un volume non système. Le stockage attaché au réseau (NAS) n’est pas pris en charge.

> [!NOTE]
> La modification du chemin d’accès ou de la lettre de lecteur après l’établissement d’un point de terminaison de serveur sur un volume n’est pas prise en charge. Veillez à utiliser un chemin d’accès final sur votre serveur inscrit.

# <a name="portal"></a>[Portail](#tab/azure-portal)
Pour ajouter un point de terminaison de serveur, accédez au nouveau groupe de synchronisation, puis sélectionnez **Ajouter un point de terminaison de serveur**.

![Ajouter un nouveau point de terminaison de serveur dans le volet Groupe de synchronisation](media/storage-sync-files-deployment-guide/create-sync-group-2.png)

Dans le volet **Ajouter un point de terminaison de serveur**, entrez les informations suivantes pour créer un point de terminaison de serveur :

- **Serveur inscrit** : nom du serveur ou du cluster où vous voulez créer le point de terminaison de serveur.
- **Chemin d’accès** : chemin d’accès Windows Server à synchroniser en tant qu’élément du groupe de synchronisation.
- **Hiérarchisation cloud** : commutateur pour activer ou désactiver la hiérarchisation cloud. La hiérarchisation cloud permet de hiérarchiser les fichiers rarement utilisés dans Azure Files.
- **Espace libre du volume** : quantité d’espace libre à réserver sur le volume sur lequel se trouve le point de terminaison de serveur. Par exemple, si l’espace libre du volume est défini à 50 % sur un volume ayant un seul point de terminaison de serveur, environ la moitié de la quantité de données est hiérarchisée dans Azure Files. Que la hiérarchisation cloud soit activée ou non, le partage de fichiers Azure dispose toujours d’une copie complète des données dans le groupe de synchronisation.
- **Mode de téléchargement initial** : Il s’agit d’une sélection facultative, à partir de la version 11 de l’agent, qui peut être utile lorsque vous avez des fichiers dans le partage de fichiers Azure, mais pas sur le serveur. Une telle situation peut exister, par exemple, si vous créez un point de terminaison de serveur pour ajouter un autre serveur de filiale à un groupe de synchronisation ou lorsque vous récupérez un serveur défaillant en cas de sinistre. Si la hiérarchisation cloud est activée, le comportement par défaut consiste à rappeler uniquement l’espace de noms, pas de contenu de fichier initialement. Cela est utile si vous pensez que les demandes d’accès des utilisateurs doivent décider du contenu du fichier qui est rappelé sur le serveur. Si la hiérarchisation cloud est désactivée, le comportement par défaut est que l’espace de noms sera d’abord téléchargé, puis les fichiers seront rappelés en fonction du timestamp de la dernière modification jusqu’à ce que la capacité locale soit atteinte. Vous pouvez toutefois changer le mode de téléchargement initial sur Espace de noms uniquement. Un troisième mode ne peut être utilisé que si la hiérarchisation cloud est désactivée pour ce point de terminaison de serveur. Ce mode évite de rappeler l’espace de noms d’abord. Les fichiers s’affichent uniquement sur le serveur local s’ils ont pu être téléchargés entièrement. Ce mode est utile si, par exemple, une application nécessite la présence de fichiers complets et ne peut pas tolérer des fichiers hiérarchisés dans son espace de noms.

Pour ajouter le point de terminaison de serveur, sélectionnez **Créer**. Vos fichiers sont maintenant synchronisés entre le partage de fichiers Azure et Windows Server. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Exécutez les commandes PowerShell suivantes pour créer le point de terminaison de serveur, et veillez à remplacer `<your-server-endpoint-path>` et `<your-volume-free-space>` par les valeurs souhaitées et vérifiez le paramètre facultatif pour la stratégie de téléchargement initial facultatif.

```powershell
$serverEndpointPath = "<your-server-endpoint-path>"
$cloudTieringDesired = $true
$volumeFreeSpacePercentage = <your-volume-free-space>
# Optional property. Choose from: [NamespaceOnly] default when cloud tiering is enabled. [NamespaceThenModifiedFiles] default when cloud tiering is disabled. [AvoidTieredFiles] only available when cloud tiering is disabled.
$initialDownloadPolicy = NamespaceOnly

if ($cloudTieringDesired) {
    # Ensure endpoint path is not the system volume
    $directoryRoot = [System.IO.Directory]::GetDirectoryRoot($serverEndpointPath)
    $osVolume = "$($env:SystemDrive)\"
    if ($directoryRoot -eq $osVolume) {
        throw [System.Exception]::new("Cloud tiering cannot be enabled on the system volume")
    }

    # Create server endpoint
    New-AzStorageSyncServerEndpoint `
        -Name $registeredServer.FriendlyName `
        -SyncGroup $syncGroup `
        -ServerResourceId $registeredServer.ResourceId `
        -ServerLocalPath $serverEndpointPath `
        -CloudTiering `
        -VolumeFreeSpacePercent $volumeFreeSpacePercentage `
        -InitialDownloadPolicy $initialDownloadPolicy
} else {
    # Create server endpoint
    New-AzStorageSyncServerEndpoint `
        -Name $registeredServer.FriendlyName `
        -SyncGroup $syncGroup `
        -ServerResourceId $registeredServer.ResourceId `
        -ServerLocalPath $serverEndpointPath `
        -InitialDownloadPolicy $initialDownloadPolicy
}
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Utilisez la commande [az storagesync sync-group server-endpoint](/cli/azure/ext/storagesync/storagesync/sync-group/server-endpoint#ext-storagesync-az-storagesync-sync-group-server-endpoint-create) pour créer un point de terminaison de serveur.

```azurecli
# Create a new sync group server endpoint 
az storagesync sync-group server-endpoint create --resource-group myResourceGroupName \
                                                 --name myNewServerEndpointName
                                                 --registered-server-id 91beed22-7e9e-4bda-9313-fec96c286e0
                                                 --server-local-path d:\myPath
                                                 --storage-sync-service myStorageSyncServiceNAme
                                                 --sync-group-name mySyncGroupName

# Create a new sync group server endpoint with additional optional parameters
az storagesync sync-group server-endpoint create --resource-group myResourceGroupName \
                                                 --storage-sync-service myStorageSyncServiceName \
                                                 --sync-group-name mySyncGroupName \
                                                 --name myNewServerEndpointName \
                                                 --registered-server-id 91beed22-7e9e-4bda-9313-fec96c286e0 \
                                                 --server-local-path d:\myPath \
                                                 --cloud-tiering on \
                                                 --volume-free-space-percent 85 \
                                                 --tier-files-older-than-days 15 \
                                                 --initial-download-policy NamespaceOnly [OR] NamespaceThenModifiedFiles [OR] AvoidTieredFiles
                                                 --offline-data-transfer on \
                                                 --offline-data-transfer-share-name myfilesharename \

```

---

## <a name="configure-firewall-and-virtual-network-settings"></a>Configurer les paramètres de pare-feu et de réseau virtuel

### <a name="portal"></a>Portail
Si vous voulez configurer Azure File Sync pour qu’il fonctionne avec des paramètres de pare-feu et de réseau virtuel, procédez comme suit :

1. Dans le portail Azure, accédez au compte de stockage que vous voulez sécuriser.
1. Sélectionnez le bouton **Pare-feux et réseaux virtuels** dans le menu de gauche.
1. Sélectionnez **Réseaux sélectionnés** sous **Autoriser l’accès depuis**.
1. Vérifiez que l’adresse IP de vos serveurs ou votre réseau virtuel figure sous la section appropriée.
1. Vérifiez que **Autoriser les services Microsoft approuvés à accéder à ce compte de stockage** est coché.
1. Sélectionnez **Save** (Enregistrer) pour enregistrer vos paramètres.

![Configuration des paramètres de pare-feu et de réseau virtuel pour un fonctionnement avec Azure File Sync](media/storage-sync-files-deployment-guide/firewall-and-vnet.png)

## <a name="onboarding-with-azure-file-sync"></a>Intégrer Azure File Sync
Voici les étapes recommandées pour intégrer Azure File Sync sans aucun temps d’arrêt tout en préservant toute la fidélité du fichier et la liste de contrôle d’accès (ACL) :
 
1. Déployez un service de synchronisation du stockage.
1. Créez un groupe de synchronisation.
1. Installez l’agent Azure File Sync sur le serveur comportant le jeu de données complet.
1. Inscrivez ce serveur et créez un point de terminaison de serveur sur le partage. 
1. Laissez la synchronisation effectuer la totalité du chargement sur le partage de fichiers Azure (point de terminaison cloud).  
1. Une fois le chargement initial terminé, installez l’agent Azure File Sync sur chacun des serveurs restants.
1. Créez de nouveaux partages de fichiers sur tous les serveurs restants.
1. Créez des points de terminaison de serveur sur les nouveaux partages de fichiers avec une stratégie de hiérarchisation cloud, si vous le souhaitez. (Cette étape requiert du stockage supplémentaire disponible pour la configuration initiale.)
1. Laissez l’agent Azure File Sync effectuer une restauration rapide de l’espace de noms complet sans transfert de données réel. Après la synchronisation de l’espace de noms complet, le moteur de synchronisation remplira l’espace disque local en fonction de la stratégie de hiérarchisation cloud du point de terminaison de serveur. 
1. Vérifiez la synchronisation est terminée et testez votre topologie comme vous le souhaitez. 
1. Redirigez les utilisateurs et les applications vers ce nouveau partage.
1. Il est possible quoique non obligatoire de supprimer les partages en double sur les serveurs.
 
Si vous n’avez pas de stockage supplémentaire pour l’intégration initiale et que vous souhaitez utiliser les partages existants, vous pouvez préamorcer les données dans les partages de fichiers Azure. Cette approche est proposée si et seulement si vous êtes en mesure d’accepter des temps d’arrêt et de garantir l’absence totale de modification des données sur les partages du serveur pendant le processus d’intégration initial. 
 
1. Vérifiez que les données des différents serveurs ne changent pas pendant le processus d’intégration.
1. Préamorcez les partages de fichiers Azure avec les données du serveur à l’aide d’un outil de transfert de données sur le protocole SMB. Par exemple, Robocopy. Vous pouvez également utiliser AzCopy sur REST. Veillez à utiliser AzCopy avec les commutateurs appropriés pour préserver les horodatages et les attributs des listes de contrôle d’accès.
1. Créez une topologie Azure File Sync avec les points de terminaison de serveur souhaités pointant sur les partages existants.
1. Laissez la synchronisation terminer le processus de rapprochement sur tous les points de terminaison. 
1. Une fois le rapprochement terminé, vous pourrez ouvrir les partages pour les modifier.
 
L’approche par préamorçage a actuellement quelques limitations. 
- Les modifications de données effectuées sur le serveur avant que la topologie de synchronisation ne soit entièrement opérationnelle risquent de provoquer des conflits sur les points de terminaison de serveur.  
- Une fois le point de terminaison cloud créé, Azure File Sync exécute un processus de détection des fichiers dans le cloud avant de démarrer la synchronisation initiale. Le temps nécessaire à ce processus varie en fonction de différents facteurs, comme la vitesse du réseau, la bande passante disponible et le nombre de fichiers et de dossiers. Pour donner une estimation approximative dans la préversion, le processus de détection s’exécute approximativement à une vitesse de 10 fichiers/s. Par conséquent, même si le préamorçage est rapide, le délai global nécessaire pour obtenir un système entièrement opérationnel peut se révéler beaucoup plus long lorsque les données sont préamorcées dans le cloud.

## <a name="self-service-restore-through-previous-versions-and-vss-volume-shadow-copy-service"></a>Restauration libre-service via Versions précédentes et VSS (Volume Shadow Copy Service)

> [!IMPORTANT]
> Les informations suivantes peuvent uniquement être utilisées avec la version 9 (ou ultérieure) de l’agent de synchronisation du stockage. Les versions antérieures à la version 9 ne disposent pas des cmdlets StorageSyncSelfService.

Versions précédentes est une fonctionnalité Windows qui vous offre la possibilité d’utiliser les instantanés VSS côté serveur d’un volume pour présenter les versions restaurables d’un fichier à un client SMB.
Ce scénario puissant, communément appelé restauration libre-service, permet aux professionnels de l’information de ne plus dépendre de la restauration d’un administrateur informatique.

Les instantanés VSS et Versions précédentes fonctionnent indépendamment d'Azure File Sync. Toutefois, la hiérarchisation cloud doit être définie sur un mode compatible. De nombreux points de terminaison de serveur Azure File Sync peuvent être présents sur le même volume. Vous devez effectuer l’appel PowerShell suivant par volume doté d'un point de terminaison de serveur sur lequel vous envisagez d'utiliser ou utilisez la hiérarchisation cloud.

```powershell
Import-Module '<SyncAgentInstallPath>\StorageSync.Management.ServerCmdlets.dll'
Enable-StorageSyncSelfServiceRestore [-DriveLetter] <string> [[-Force]] 
```

Les instantanés VSS portent sur un volume entier. Par défaut, jusqu’à 64 instantanés peuvent être présents pour un volume donné, sous réserve d'espace suffisant pour stocker ces instantanés. VSS gère cela automatiquement. Par défaut, deux instantanés sont prévus quotidiennement, du lundi au vendredi. Cette planification peut être configurée à l’aide d’une tâche planifiée Windows. La cmdlet PowerShell ci-dessus effectue deux opérations :
1. Elle configure la hiérarchisation cloud Azure file Sync sur le volume spécifié à des fins de compatibilité avec les versions précédentes et garantit la restauration d'un fichier à partir d’une version précédente, même si celui-ci faisait l'objet d'une hiérarchisation cloud sur le serveur. 
1. Elle active la planification VSS par défaut. Vous pouvez cependant la modifier ultérieurement. 

> [!Note]  
> Deux points importants sont à prendre en considération :
>- Si vous utilisez le paramètre -Force et que VSS est activé, il remplace la planification d'instantanés VSS par la planification par défaut. Veillez à enregistrer votre configuration personnalisée avant d’exécuter la cmdlet.
> - Si vous utilisez cette cmdlet sur un nœud de cluster, vous devez également l’exécuter sur tous les autres nœuds du cluster. 

Pour savoir si la compatibilité de la restauration libre-service est activée, exécutez la cmdlet suivante.

```powershell
Get-StorageSyncSelfServiceRestore [[-Driveletter] <string>]
```

Elle répertorie tous les volumes présents sur le serveur, ainsi que le nombre de jours compatibles avec la hiérarchisation cloud de chacun. Ce nombre est automatiquement calculé en fonction du nombre maximal d’instantanés possibles par volume et de la planification d'instantanés par défaut. Ainsi, par défaut, toutes les versions précédentes présentées à un professionnel de l’information peuvent être utilisées à des fins de restauration. Il en va de même si vous modifiez la planification par défaut afin de permettre plus d'instantanés.
Cela étant, si vous modifiez la planification de manière à obtenir un instantané disponible sur le volume antérieur à la valeur de jours compatibles, les utilisateurs ne pourront pas utiliser cet instantané antérieur (version précédente) à des fins de restauration.

> [!Note]
> L’activation de la restauration libre-service peut avoir un impact sur votre consommation de stockage Azure et votre facture. Cet impact est limité aux fichiers actuellement hiérarchisés sur le serveur. L’activation de cette fonctionnalité permet de s’assurer qu’une version de fichier disponible dans le cloud peut être référencée via une entrée de versions précédentes (instantané VSS).
>
> Si vous désactivez la fonctionnalité, la consommation du stockage Azure diminue progressivement jusqu'au terme de la fenêtre de jours compatibles. Il n'existe aucun moyen de l'accélérer. 

Le nombre maximal d'instantanés VSS par volume par défaut (64), ainsi que leur planification par défaut, se traduisent par un maximum de 45 jours de versions précédentes pouvant être restaurées par un professionnel de l’information et ce, en fonction du nombre d'instantanés VSS que vous pouvez stocker sur votre volume.

Si le paramètre de 65 instantanés VSS par volume ne vous convient pas, vous pouvez [modifier cette valeur via une clé de Registre](/windows/win32/backup/registry-keys-for-backup-and-restore#maxshadowcopies).
Pour que la nouvelle limite prenne effet, vous devez réexécuter la cmdlet de manière à activer la compatibilité des versions précédentes sur chaque volume sur lequel elle était précédemment activée, avec l'indicateur -Force afin de prendre en compte le nouveau nombre maximal d'instantanés VSS par volume. Un nouveau nombre de jours compatibles est alors calculé. Notez que cette modification prend uniquement effet sur les fichiers nouvellement hiérarchisés et remplace les personnalisations apportées à la planification VSS que vous avez peut-être établie.

<a id="proactive-recall"></a>
## <a name="proactively-recall-new-and-changed-files-from-an-azure-file-share"></a>Rappeler de manière proactive les fichiers nouveaux et modifiés à partir d’un partage de fichiers Azure

Avec la version 11 de l’agent, un nouveau mode devient disponible sur un point de terminaison de serveur. Ce mode permet aux entreprises distribuées à l’échelle globale d’avoir le cache serveur dans une région distante préremplie même avant même que les utilisateurs locaux accèdent à un fichier. Lorsqu’il est activé sur un point de terminaison de serveur, ce mode fait en sorte que ce serveur rappelle les fichiers qui ont été créés ou modifiés dans le partage de fichiers Azure.

### <a name="scenario"></a>Scénario

Une société internationale a des filiales aux États-Unis et en Inde. Le matin (heure des États-Unis), les travailleurs de l’information créent un nouveau dossier et de nouveaux fichiers pour un tout nouveau projet et travaillent dessus toute la journée. Azure File Sync synchronisera le dossier et les fichiers sur le partage de fichiers Azure (point de terminaison cloud). Les travailleurs de l’information en Inde continuent de travailler sur le projet dans leur fuseau horaire. Lorsqu’ils arrivent le matin, le serveur local avec Azure File Sync en Inde doit disposer de ces nouveaux fichiers localement, afin que l’équipe en Inde puisse travailler efficacement à partir d’un cache local. L’activation de ce mode empêche l’accès initial au fichier d’être plus lent en raison du rappel à la demande et permet au serveur de rappeler de manière proactive les fichiers dès qu’ils ont été modifiés ou créés dans le partage de fichiers Azure.

> [!IMPORTANT]
> Il est important de réaliser que le suivi des modifications apportées au partage de fichiers Azure sur le serveur peut augmenter le trafic sortant et la facturation d’Azure. Si les fichiers rappelés sur le serveur ne sont pas réellement nécessaires localement, le rappel inutile au serveur peut avoir des conséquences négatives. Utilisez ce mode lorsque vous savez que le préremplissage du cache sur un serveur avec les modifications récentes dans le cloud aura un effet positif sur les utilisateurs ou les applications qui utilisent les fichiers sur ce serveur.

### <a name="enable-a-server-endpoint-to-proactively-recall-what-changed-in-an-azure-file-share"></a>Activer un point de terminaison de serveur pour rappeler de manière proactive ce qui a changé dans un partage de fichiers Azure

# <a name="portal"></a>[Portail](#tab/proactive-portal)

1. Dans le [portail Azure](https://portal.azure.com/), accédez à votre service de synchronisation de stockage, sélectionnez le groupe de synchronisation approprié, puis identifiez le point de terminaison de serveur pour lequel vous souhaitez suivre attentivement les modifications dans le partage de fichiers Azure (point de terminaison cloud).
1. Dans la section Hiérarchisation cloud, recherchez la rubrique « Téléchargement d’un partage de fichiers Azure ». Le mode actuellement sélectionné s’affiche et vous pouvez le modifier pour suivre les modifications du partage de fichiers Azure plus attentivement et les rappeler de manière proactive sur le serveur.

:::image type="content" source="media/storage-sync-files-deployment-guide/proactive-download.png" alt-text="Image indiquant le comportement de téléchargement d’un partage de fichiers Azure pour un point de terminaison de serveur actuellement appliqué et un bouton permettant d’ouvrir un menu permettant de le modifier.":::

# <a name="powershell"></a>[PowerShell](#tab/proactive-powershell)

Vous pouvez modifier les propriétés de point de terminaison de serveur dans PowerShell par le biais de l’applet de commande [Set-AzStorageSyncServerEndpoint](/powershell/module/az.storagesync/set-azstoragesyncserverendpoint).

```powershell
# Optional parameter. Default: "UpdateLocallyCachedFiles", alternative behavior: "DownloadNewAndModifiedFiles"
$recallBehavior = "DownloadNewAndModifiedFiles"

Set-AzStorageSyncServerEndpoint -InputObject <PSServerEndpoint> -LocalCacheMode $recallBehavior
```

---

## <a name="migrate-a-dfs-replication-dfs-r-deployment-to-azure-file-sync"></a>Migrer un déploiement de la réplication DFS (DFS-R) vers Azure File Sync
Pour migrer un déploiement de DFS-R vers Azure File Sync :

1. Créez un groupe de synchronisation pour représenter la topologie DFS-R que vous remplacez.
2. Démarrez sur le serveur contenant le jeu de données complet de votre topologie DFS-R à migrer. Installez Azure File Sync sur ce serveur.
3. Inscrivez ce serveur et créez un point de terminaison de serveur pour le premier serveur à migrer. N’activez pas la hiérarchisation cloud.
4. Laissez l’ensemble de la synchronisation des données sur votre partage de fichiers Azure (point de terminaison de cloud).
5. Installez et inscrivez l’agent Azure File Sync sur tous les autres serveurs DFS-R.
6. Désactivez DFS-R. 
7. Créez un point de terminaison de serveur sur tous les serveurs DFS-R. N’activez pas la hiérarchisation cloud.
8. Vérifiez la synchronisation est terminée et testez votre topologie comme vous le souhaitez.
9. Mettez DFS-R hors service.
10. La hiérarchisation cloud peut maintenant être activée sur n’importe quel point de terminaison de serveur souhaité.

Pour plus d’informations, consultez [Interopérabilité d’Azure File Sync avec le système de fichiers DFS](storage-sync-files-planning.md#distributed-file-system-dfs).

## <a name="next-steps"></a>Étapes suivantes
- [Ajouter ou supprimer un point de terminaison de serveur pour Azure File Sync](storage-sync-files-server-endpoint.md)
- [Inscrire ou désinscrire un serveur auprès d’Azure File Sync](storage-sync-files-server-registration.md)
- [Superviser Azure File Sync](storage-sync-files-monitoring.md)