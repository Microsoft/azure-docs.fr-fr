---
title: Utilisation d’un partage de fichiers Azure avec Windows | Microsoft Docs
description: Apprenez à utiliser des partages de fichiers Azure avec Windows et Windows Server. Utilisez des partages de fichiers Azure avec SMB 3.0 sur des installations Windows exécutées localement ou sur des machines virtuelles Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 06/22/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e64b7efdd430287a7a3a969c5bf62b0c0e2aec9c
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94626892"
---
# <a name="use-an-azure-file-share-with-windows"></a>Utiliser un partage de fichiers Azure avec Windows
[Azure Files](storage-files-introduction.md) est le système de fichiers cloud facile à utiliser de Microsoft. Il est possible d’utiliser sans problème le partage de fichiers Azure dans Windows et Windows Server. Cet article décrit les considérations concernant l’utilisation d’un partage de fichiers Azure avec Windows et Windows Server.

Pour utiliser un partage de fichiers Azure en dehors de la région Azure sur laquelle il est hébergé, par exemple localement ou dans une région Azure différente, le système d’exploitation doit prendre en charge SMB 3.0. 

Vous pouvez utiliser des partages de fichiers Azure sur une installation Windows en cours d’exécution dans une machine virtuelle Azure ou localement. Le tableau suivant illustre les versions du système d’exploitation qui prennent en charge l’accès aux partages de fichiers dans un environnement donné :

| Version de Windows        | Version SMB | Version montable dans une machine virtuelle Azure | Montable en local |
|------------------------|-------------|-----------------------|-----------------------|
| Windows Server 2019 | SMB 3.0 | Oui | Oui |
| Windows 10<sup>1</sup> | SMB 3.0 | Oui | Oui |
| Canal semestriel Windows Server<sup>2</sup> | SMB 3.0 | Oui | Oui |
| Windows Server 2016 | SMB 3.0 | Oui | Oui |
| Windows 8.1 | SMB 3.0 | Oui | Oui |
| Windows Server 2012 R2 | SMB 3.0 | Oui | Oui |
| Windows Server 2012 | SMB 3.0 | Oui | Oui |
| Windows 7<sup>3</sup> | SMB 2.1 | Oui | Non |
| Windows Server 2008 R2<sup>3</sup> | SMB 2.1 | Oui | Non |

<sup>1</sup>Windows 10, versions 1507, 1607, 1803, 1809, 1903, 1909 et 2004.  
<sup>2</sup>Windows Server, versions 1809, 1903, 1909, 2004.  
<sup>3</sup>Le support Microsoft standard pour Windows 7 et Windows Server 2008 R2 a pris fin. Il est possible d’acheter du support supplémentaire pour les mises à jour de sécurité par l’intermédiaire du [Programme des correctifs de sécurité étendus (ESU)](https://support.microsoft.com/help/4497181/lifecycle-faq-extended-security-updates) uniquement. Nous vous recommandons vivement de procéder à la migration de ces systèmes d’exploitation.

> [!Note]  
> Nous vous conseillons de prendre la base de connaissances la plus récente pour votre version de Windows.

## <a name="prerequisites"></a>Prérequis 

Vérifiez que le port 445 est ouvert : Le protocole SMB nécessite que le port TCP 445 soit ouvert, les connexions échoueront si ce port est bloqué. Vous pouvez vérifier si votre pare-feu bloque le port 445 avec l’applet de commande `Test-NetConnection`. Pour découvrir comment contourner un port 445 bloqué, consultez la section [Cause 1 : Le port 445 est bloqué](storage-troubleshoot-windows-file-connection-problems.md#cause-1-port-445-is-blocked) de notre guide de dépannage de Windows.

## <a name="using-an-azure-file-share-with-windows"></a>Utiliser un partage de fichiers Azure avec Windows
Pour utiliser un partage de fichiers Azure avec Windows, vous devez soit le monter (lui affecter un chemin de point de montage ou une lettre de lecteur), soit y accéder via son [chemin d’accès UNC](/windows/win32/fileio/naming-a-file). 

Cet article utilise la clé de compte de stockage pour accéder au partage de fichiers. Une clé de compte de stockage est une clé administrateur pour un compte de stockage (y compris les autorisations d’administrateur sur tous les fichiers et dossiers dans le partage de fichiers auquel vous accédez), ainsi que pour tous les partages de fichiers et autres ressources de stockage (objets blob, files d’attente, tables, etc.) contenus dans votre compte de stockage. Si cela ne suffit pas pour votre charge de travail, vous pouvez utiliser [Azure File Sync](storage-sync-files-planning.md) ou une [authentification basée sur l’identité sur SMB](storage-files-active-directory-overview.md).

Un modèle commun pour délester et déplacer les applications métier (LOB) qui attendent un partage de fichiers SMB vers Azure consiste à utiliser un partage de fichiers Azure comme alternative pour l’exécution d’un serveur de fichiers Windows dédié dans une machine virtuelle Azure. Pour réussir la migration d’une application métier afin qu’elle utilise un partage de fichiers Azure, il est important de considérer que de nombreuses applications métier s’exécutent dans le contexte d’un compte de service dédié avec des autorisations système limitées, plutôt que le compte administrateur de la machine virtuelle. Par conséquent, vous devez vous assurer que vous montez/enregistrez les informations d’identification pour le partage de fichiers Azure à partir du contexte du compte de service plutôt que de votre compte d’administration.

### <a name="mount-the-azure-file-share"></a>Monter le partage de fichiers Azure

Le portail Azure vous fournit un script que vous pouvez utiliser pour monter votre partage de fichiers directement sur un hôte. Nous vous recommandons d’utiliser ce script fourni.

Pour obtenir ce script :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Accédez au compte de stockage contenant le partage de fichiers que vous souhaitez monter.
1. Sélectionner **Partages de fichiers**.
1. Sélectionnez le fichier que vous voulez monter.

    :::image type="content" source="media/storage-how-to-use-files-windows/select-file-shares.png" alt-text="Exemple":::

1. Sélectionnez **Connecter**.

    :::image type="content" source="media/storage-how-to-use-files-windows/file-share-connect-icon.png" alt-text="Capture d’écran de l’icône de connexion pour votre partage de fichiers.":::

1. Sélectionnez la lettre de lecteur sur laquelle monter le partage.
1. Copiez le script fourni.

    :::image type="content" source="media/storage-how-to-use-files-windows/files-portal-mounting-cmdlet-resize.png" alt-text="Exemple de texte":::

1. Collez le script dans un shell sur l’hôte sur lequel vous voulez monter le partage de fichiers, puis exécutez-le.

Vous avez maintenant monté votre partage de fichiers Azure.

### <a name="mount-the-azure-file-share-with-file-explorer"></a>Montage du partage de fichiers Azure avec l’Explorateur de fichiers
> [!Note]  
> Notez que les instructions ci-après sont affichées sur Windows 10 et peuvent différer légèrement sur les versions plus anciennes. 

1. Ouvrez l’Explorateur de fichiers. Pour cela, accédez au menu Démarrer ou appuyez sur les touches Win+E.

1. Accédez à **Ce PC** sur le côté gauche de la fenêtre. Cela modifie les menus disponibles dans le ruban. Dans le menu Ordinateur, sélectionnez **Connecter un lecteur réseau**.
    
    ![Une capture d’écran du menu déroulant « Connecter un lecteur réseau »](./media/storage-how-to-use-files-windows/1_MountOnWindows10.png)

1. Sélectionnez la lettre de lecteur et entrez le chemin d’accès UNC, le format du chemin d’accès UNC étant `\\<storageAccountName>.file.core.windows.net\<fileShareName>`. Par exemple : `\\anexampleaccountname.file.core.windows.net\example-share-name`.
    
    ![Une capture d’écran de la boîte de dialogue « Connecter un lecteur réseau »](./media/storage-how-to-use-files-windows/2_MountOnWindows10.png)

1. Utilisez le nom du compte de stockage avec le préfixe `AZURE\` comme nom d’utilisateur et une clé de compte de stockage comme mot de passe.
    
    ![Une capture d’écran de la boîte de dialogue des identifiants réseau](./media/storage-how-to-use-files-windows/3_MountOnWindows10.png)

1. Utilisez le partage de fichiers Azure à votre guise.
    
    ![Le partage de fichiers Azure est désormais monté](./media/storage-how-to-use-files-windows/4_MountOnWindows10.png)

1. Lorsque vous êtes prêt à démonter le partage de fichiers Azure, il vous suffit de cliquer avec le bouton droit de la souris sur l’entrée du partage, sous **Emplacements réseau**, dans l’Explorateur de fichiers et de sélectionner **Déconnecter**.

### <a name="accessing-share-snapshots-from-windows"></a>Accès aux instantanés de partage à partir de Windows
Si vous avez utilisé un instantané de partage, soit manuellement, soit automatiquement par le biais d’un script ou d’un service tel que Sauvegarde Microsoft Azure, vous pouvez visualiser les versions précédentes d’un partage, d’un répertoire ou d’un fichier spécifique d’un partage de fichiers sur Windows. Vous pouvez prendre un instantané du partage en utilisant [Azure PowerShell](storage-how-to-use-files-powershell.md), [Azure CLI](storage-how-to-use-files-cli.md) ou le [Portail Azure](storage-how-to-use-files-portal.md).

#### <a name="list-previous-versions"></a>Répertorier les versions précédentes
Accédez à l’élément ou à l’élément parent à restaurer. Double-cliquez pour accéder au répertoire souhaité. Cliquez avec le bouton droit et sélectionnez **Propriétés** dans le menu.

![Menu contextuel pour un répertoire sélectionné](./media/storage-how-to-use-files-windows/snapshot-windows-previous-versions.png)

Sélectionnez **Versions précédentes** pour afficher la liste des instantanés de partage pour ce répertoire. Le chargement de la liste peut prendre quelques secondes, selon la vitesse du réseau et le nombre d’instantanés de partage du répertoire.

![Onglet Versions précédentes](./media/storage-how-to-use-files-windows/snapshot-windows-list.png)

Vous pouvez sélectionner **Ouvrir** pour ouvrir un instantané particulier. 

![Instantané ouvert](./media/storage-how-to-use-files-windows/snapshot-browse-windows.png)

#### <a name="restore-from-a-previous-version"></a>Restaurer à partir d’une version précédente
Sélectionnez **Restaurer** pour copier à l’emplacement d’origine le contenu de l’ensemble du répertoire de façon récursive à l’heure de création de l’instantané de partage.

 ![Bouton Restaurer dans un message d’avertissement](./media/storage-how-to-use-files-windows/snapshot-windows-restore.png) 

## <a name="securing-windowswindows-server"></a>Sécurisation de Windows/Windows Server
Pour monter un partage de fichiers Azure sur Windows, le port 445 doit être accessible. De nombreuses organisations bloquent le port 445 en raison des risques de sécurité inhérents à SMB 1. SMB 1, également appelé CIFS (Common Internet File System), est un protocole de système de fichiers hérités inclus avec Windows et Windows Server. SMB 1 est un protocole obsolète, inefficace et qui pose surtout des problèmes de sécurité. La bonne nouvelle est que Azure Files ne prend pas en charge SMB 1, et que toutes les versions prises en charge de Windows et Windows Server permettent de supprimer ou désactiver SMB 1. Nous [recommandons vivement](https://aka.ms/stopusingsmb1) de toujours supprimer ou désactiver le client et le serveur SMB 1 dans Windows avant d’utiliser des partages de fichiers Azure en production.

Le tableau suivant fournit des informations détaillées sur l’état de SMB 1 à chaque version de Windows :

| Version de Windows                           | État par défaut de SMB 1 | Méthode de désactivation/suppression       | 
|-------------------------------------------|----------------------|-----------------------------|
| Windows Server 2019                       | Désactivé             | Supprimer avec la fonctionnalité Windows |
| Windows Server, versions 1709 et ultérieures            | Désactivé             | Supprimer avec la fonctionnalité Windows |
| Windows 10, versions 1709 et ultérieures                | Désactivé             | Supprimer avec la fonctionnalité Windows |
| Windows Server 2016                       | activé              | Supprimer avec la fonctionnalité Windows |
| Windows 10, versions 1507, 1607 et 1703 | activé              | Supprimer avec la fonctionnalité Windows |
| Windows Server 2012 R2                    | activé              | Supprimer avec la fonctionnalité Windows | 
| Windows 8.1                               | activé              | Supprimer avec la fonctionnalité Windows | 
| Windows Server 2012                       | activé              | Désactiver grâce au Registre       | 
| Windows Server 2008 R2                    | activé              | Désactiver grâce au Registre       |
| Windows 7                                 | activé              | Désactiver grâce au Registre       | 

### <a name="auditing-smb-1-usage"></a>Audit de l’utilisation de SMB 1
> S’applique à Windows Server 2019, canal semestriel Windows Server (versions 1709 et 1803), Windows Server 2016, Windows 10 (versions 1507, 1607, 1703, 1709 et 1803), Windows Server 2012 R2 et Windows 8.1

Avant de supprimer SMB 1 de votre environnement, vous pouvez auditer l’utilisation de SMB 1 pour voir si un client risque d’être coupé par la modification. Si une requête est effectuée par rapport à des partages SMB avec SMB 1, un événement d’audit sera consigné dans le journal des événements sous `Applications and Services Logs > Microsoft > Windows > SMBServer > Audit`. 

> [!Note]  
> Pour activer la prise en charge de l’audit sur Windows Server 2012 R2 et Windows 8.1, installez au moins [KB4022720](https://support.microsoft.com/help/4022720/windows-8-1-windows-server-2012-r2-update-kb4022720).

Pour activer l’audit, exécutez l’applet de commande suivante à partir d’une session PowerShell avec élévation des privilèges :

```powershell
Set-SmbServerConfiguration –AuditSmb1Access $true
```

### <a name="removing-smb-1-from-windows-server"></a>Suppression de SMB 1 de Windows Server
> S’applique à Windows Server 2019, canal semestriel Windows Server (versions 1709 et 1803), Windows Server 2016, Windows Server 2012 R2

Pour supprimer SMB 1 d’une instance Windows Server, exécutez l’applet de commande suivante à partir d’une session PowerShell avec élévation de privilèges :

```powershell
Remove-WindowsFeature -Name FS-SMB1
```

Pour terminer le processus de suppression, redémarrez votre serveur. 

> [!Note]  
> À partir de Windows 10 et Windows Server version 1709, SMB 1 n’est pas installé par défaut et dispose de fonctionnalités Windows distinctes pour le client SMB 1 et le serveur SMB 1. Nous vous recommandons de ne pas installer le serveur SMB 1 (`FS-SMB1-SERVER`) ni le client SMB 1 (`FS-SMB1-CLIENT`).

### <a name="removing-smb-1-from-windows-client"></a>Suppression de SMB 1 du client Windows
> S’applique à Windows 10 (versions 1507, 1607, 1703, 1709 et 1803) et Windows 8.1

Pour supprimer SMB 1 d’un client Windows, exécutez l’applet de commande suivante à partir d’une session PowerShell avec élévation de privilèges :

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

Pour terminer le processus de suppression, redémarrez votre PC.

### <a name="disabling-smb-1-on-legacy-versions-of-windowswindows-server"></a>Désactivation de SMB 1 sur des versions héritées de Windows/Windows Server
> S’applique à Windows Server 2012, Windows Server 2008 R2 et Windows Server 7

SMB 1 ne peut pas être complètement supprimé sur les versions héritées de Windows/Windows Server, mais il peut être désactivé via le Registre. Pour désactiver SMB 1, créez une nouvelle clé de Registre `SMB1` de type `DWORD` avec la valeur `0` sous `HKEY_LOCAL_MACHINE > SYSTEM > CurrentControlSet > Services > LanmanServer > Parameters`.

Vous pouvez le faire facilement avec la cmdlet PowerShell suivante :

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" SMB1 -Type DWORD -Value 0 –Force
```

Après avoir créé cette clé de Registre, vous devez redémarrer votre serveur pour désactiver SMB 1.

### <a name="smb-resources"></a>Ressources SMB
- [Stop using SMB 1](https://blogs.technet.microsoft.com/filecab/2016/09/16/stop-using-smb1/) (Arrêtez d’utiliser SMB 1)
- [SMB 1 Product Clearinghouse](https://blogs.technet.microsoft.com/filecab/2017/06/01/smb1-product-clearinghouse/)
- [Discover SMB 1 in your environment with DSCEA](/archive/blogs/ralphkyttle/discover-smb1-in-your-environment-with-dscea) (Recherchez SMB 1 dans votre environnement à l’aide de DSCEA)
- [Disabling SMB 1 through Group Policy](/archive/blogs/secguide/disabling-smbv1-through-group-policy) (Désactivation de SMB 1 à l’aide de la stratégie de groupe)

## <a name="next-steps"></a>Étapes suivantes
Consultez ces liens pour en savoir plus sur Azure Files :
- [Planification d’un déploiement Azure Files](storage-files-planning.md)
- [FORUM AUX QUESTIONS](./storage-files-faq.md)
- [Résolution des problèmes sur Windows](storage-troubleshoot-windows-file-connection-problems.md)