---
title: Compréhension de la hiérarchisation cloud d’Azure File Sync | Microsoft Docs
description: En savoir plus sur la hiérarchisation cloud, une fonctionnalité Azure File Sync facultative. Les fichiers fréquemment utilisés sont mis en cache localement sur le serveur ; les autres sont hiérarchisées sur Azure Files.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 06/15/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: e5aafaa02f503582bd0050f8a6389d78b52eaa76
ms.sourcegitcommit: 541bb46e38ce21829a056da880c1619954678586
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/11/2020
ms.locfileid: "91939151"
---
# <a name="cloud-tiering-overview"></a>Vue d’ensemble de la hiérarchisation cloud
La hiérarchisation cloud est une fonctionnalité facultative d’Azure File Sync, qui met en cache sur le serveur local les fichiers faisant l’objet d’accès fréquents, tous les autres fichiers étant hiérarchisés sur Azure Files en fonction de paramètres de stratégie. Quand un fichier est hiérarchisé, le filtre du système de fichiers Azure File Sync (StorageSync.sys) remplace le fichier local par un pointeur, ou point d’analyse. Le point d’analyse représente une URL vers le fichier dans Azure Files. Un fichier hiérarchisé a l’attribut « offline » (hors connexion), et son attribut FILE_ATTRIBUTE_RECALL_ON_DATA_ACCESS est défini dans le système de fichiers NTFS de façon à ce que des applications tierces puissent identifier sûrement des fichiers hiérarchisés.
 
Lorsqu'un utilisateur ouvre un fichier hiérarchisé, Azure File Sync rappelle les données du fichier à partir d’Azure Files avec fluidité, sans que l’utilisateur ait besoin de savoir que le fichier est stocké dans Azure. 
 
 > [!Important]  
 > La hiérarchisation cloud n’est pas prise en charge sur le volume système Windows.
    
 > [!Important]  
 > Pour rappeler les fichiers qui ont été hiérarchisés, la bande passante réseau doit être d'au moins 1 Mbps. Si la bande passante du réseau est inférieure à 1 Mbps, les fichiers peuvent ne pas se rappeler avec une erreur de délai d'attente.

## <a name="cloud-tiering-faq"></a>FAQ sur la hiérarchisation cloud

<a id="afs-cloud-tiering"></a>
### <a name="how-does-cloud-tiering-work"></a>Comment fonctionne la hiérarchisation cloud ?
Le filtre du système Azure File Sync génère une « carte thermique » de votre espace de noms sur chaque point de terminaison de serveur. Il surveille les accès (opérations de lecture et d’écriture) au fil du temps, puis, selon la fréquence et la nouveauté des accès, attribue un score de chaleur à chaque fichier. Un fichier faisant l’objet d’accès fréquents qui a été récemment ouvert est considéré comme chaud, tandis qu’un fichier à peine touché et qui n’a plus été utilisé depuis un certain temps est considéré comme froid. Quand le volume de fichiers sur un serveur dépasse le seuil d’espace libre du volume que vous définissez, il hiérarchise les fichiers les plus froids sur Azure Files jusqu’à ce que votre pourcentage d’espace libre soit atteint.

En outre, vous pouvez spécifier une stratégie de date sur chaque point de terminaison de serveur. Elle hiérarchise tous les fichiers non ouverts au sein d’un nombre de jours spécifié, peu importe la capacité de stockage locale disponible. Il s’agit d’un bon choix pour libérer de manière proactive l’espace disque local si vous savez que les fichiers de ce point de terminaison de serveur n’ont pas besoin d’être conservés en local après un certain délai. Cela libère une capacité de disque local précieuse pour d’autres points de terminaison sur le même volume, afin de mettre en cache un plus grand nombre de leurs fichiers.

La carte thermique de la hiérarchisation cloud est essentiellement une liste triée de tous les fichiers qui sont synchronisés et se trouvent à un emplacement où la hiérarchisation cloud est activée. Pour déterminer la position relative d’un fichier individuel sur cette carte thermique, le système utilise la valeur maximale de l’un des horodatages suivants, dans cet ordre : MAX (heure du dernier accès, heure de la dernière modification, heure de création). En règle générale, l’heure du dernier accès est suivie et disponible. Toutefois, lorsqu’un nouveau point de terminaison de serveur est créé, avec la hiérarchisation cloud activée, le temps initialement écoulé est consacré à l’observation de l’accès aux fichiers. En l’absence d’heure du dernier accès, l’heure de la dernière modification est utilisée pour évaluer la position relative sur la carte thermique. Le même système de secours est applicable à la stratégie de date. Sans heure de dernier accès, la stratégie de date tiendra compte de l’heure de la dernière modification. Si celle-ci devait être indisponible, la date de création d’un fichier sera la valeur prise en compte. Au fil du temps, le système observera de plus en plus de demandes d’accès aux fichiers et finira par utiliser principalement l’heure de dernier accès suivie automatiquement.

La hiérarchisation cloud ne dépend pas de la fonctionnalité NTFS pour le suivi de l’heure du dernier accès. La fonctionnalité NTFS est désactivée par défaut et, en raison de considérations liées aux performances, nous ne vous recommandons pas d’activer manuellement cette fonctionnalité. La hiérarchisation cloud suit séparément et très efficacement l’heure du dernier accès.

<a id="tiering-minimum-file-size"></a>
### <a name="what-is-the-minimum-file-size-for-a-file-to-tier"></a>Quelle est la taille minimale d’un fichier à hiérarchiser ?

Pour les agents de version 9 et ultérieures, la taille minimale d’un fichier à hiérarchiser basée sur la taille de cluster du système. La taille de fichier minimale éligible pour la hiérarchisation cloud est calculée en multipliant par 2 la taille du cluster et utilise au minimum 8 ko. La table suivante illustre les tailles minimales des fichiers qui peuvent être hiérarchisés, sur la base de la taille du volume de cluster :

|Taille de volume de cluster (en octets) |Les fichiers de cette taille ou plus peuvent être hiérarchisés  |
|----------------------------|---------|
|4 ko ou moins (4096)      | 8 Ko    |
|8 Ko (8192)                 | 16 Ko   |
|16 Ko (16384)               | 32 Ko   |
|32 ko (32 768)               | 64 Ko   |
|64 ko (65 536)    | 128 Ko  |

Les tailles de clusters sont actuellement prises en charge jusqu’à 64 Ko. Pour les tailles supérieures, la hiérarchisation cloud ne fonctionne pas.

Tous les systèmes de fichiers utilisés par Windows organisent votre disque dur en fonction de la taille du cluster (également appelée taille d’unité d’allocation). La taille du cluster représente la plus petite quantité d’espace disque qui peut être utilisée pour contenir un fichier. Lorsque les tailles de fichiers ne correspondent pas à un multiple de la taille du cluster, plus d’espace est nécessaire pour conserver le fichier (jusqu’au multiple supérieur de la taille du cluster).

Azure File Sync est pris en charge sur les volumes NTFS avec Windows Server 2012 R2 et versions ultérieures. La table suivante décrit les tailles de cluster par défaut lorsque vous créez un nouveau volume NTFS. 

|Taille du volume    |Windows Server 2012 R2 et versions plus récentes |
|---------------|---------------|
|7 Mo – 16 To   | 4 Ko          |
|16 To – 32 To   | 8 Ko          |
|32 To – 64 To   | 16 Ko         |
|64 To – 128 To  | 32 Ko         |
|128 To – 256 To | 64 Ko         |
|> 256 To       | Non pris en charge |

Lors de la création du volume, il est possible que vous ayez mis en forme manuellement le volume avec une taille de cluster différente. Si votre volume provient d’une version antérieure de Windows, les tailles de cluster par défaut peuvent également être différentes. [Cet article contient des informations supplémentaires sur les tailles de cluster par défaut.](https://support.microsoft.com/help/140365/default-cluster-size-for-ntfs-fat-and-exfat) Même si vous choisissez une taille de cluster inférieure à 4 ko, la limite de 8 ko comme plus petite taille de fichier pouvant être hiérarchisée s’applique toujours. (Même si, techniquement, le double de la taille du cluster équivaut à moins de 8 ko.)

Ce minimum absolu est dû à la manière dont NTFS stocke les fichiers très petits, de 1 ko à 4 ko. En fonction d’autres paramètres du volume, il est possible que les petits fichiers ne soient pas du tout stockés dans un cluster sur le disque. Il est peut-être plus efficace de stocker ces fichiers directement dans la table de fichiers maîtres ou l’« enregistrement de la table MFT » du volume. Le point d’analyse de la hiérarchisation cloud est toujours stocké sur le disque et occupe exactement un cluster. Hiérarchiser de si petits fichiers pourrait ne pas représenter un gain de place. Les cas extrêmes peuvent même finir par utiliser plus d’espace lorsque la hiérarchisation cloud est activée. Pour éviter cela, la plus petite taille d’un fichier que la hiérarchisation cloud organisera est de 8 ko sur une taille de cluster de 4 ko ou moins.

<a id="afs-volume-free-space"></a>
### <a name="how-does-the-volume-free-space-tiering-policy-work"></a>Comment fonctionne la stratégie de hiérarchisation de l’espace libre du volume ?
L’espace libre du volume est la quantité d’espace libre que vous souhaitez réserver sur le volume sur lequel se trouve un point de terminaison de serveur. Par exemple, si l’espace libre défini du volume est de 20 % sur un volume qui comporte un point de terminaison de serveur, jusqu’à 80 % de l’espace du volume sont occupés par les derniers fichiers utilisés le plus récemment, et tous les fichiers restants ne tenant pas dans cet espace sont hiérarchisés sur Azure. L’espace libre du volume s’applique au niveau du volume plutôt qu’au niveau des répertoires ou des groupes de synchronisation individuels. 

<a id="volume-free-space-fastdr"></a>
### <a name="how-does-the-volume-free-space-tiering-policy-work-with-regards-to-new-server-endpoints"></a>Comment la stratégie de hiérarchisation de l’espace libre du volume fonctionne-t-elle en lien avec les nouveaux points de terminaison de serveur ?
Quand un point de terminaison de serveur vient d’être approvisionné et connecté à un partage de fichiers Azure, le serveur commence par télécharger l’espace de noms, puis télécharge les fichiers réels jusqu’à atteindre son seuil d’espace libre du volume. Ce processus est également appelé reprise après sinistre rapide ou restauration d’espace de noms rapide.

<a id="afs-effective-vfs"></a>
### <a name="how-is-volume-free-space-interpreted-when-i-have-multiple-server-endpoints-on-a-volume"></a>Comment l’espace libre du volume est-il interprété quand il y a plusieurs points de terminaison de serveur sur un volume ?
Quand il y a plusieurs point de terminaison de serveur sur un volume, le seuil d’espace libre de volume effectif est l’espace libre de volume le plus élevé spécifié sur tous les points de terminaison de serveur sur ce volume. Les fichiers sont hiérarchisés en fonction de leurs modèles d’utilisation, quel que soit le point de terminaison de serveur auquel ils appartiennent. Par exemple, si vous avez deux points de terminaison de serveur sur un volume, Point1 et Point2, où Point1 a un seuil d’espace libre de volume de 25 % et Point2 a un seuil d’espace libre de volume de 50 %, le seuil d’espace libre de volume pour les deux points de terminaison de serveur est de 50 %. 

<a id="date-tiering-policy"></a>
### <a name="how-does-the-date-tiering-policy-work-in-conjunction-with-the-volume-free-space-tiering-policy"></a>Comment la stratégie de hiérarchisation de dates fonctionne-t-elle conjointement à la stratégie de hiérarchisation d’espace disponible sur le volume ? 
Lorsque vous activez la hiérarchisation cloud sur un point de terminaison de serveur, vous définissez une stratégie d’espace disponible sur le volume. Elle est toujours prioritaire sur toutes les autres stratégies, y compris la stratégie de date. Si vous le souhaitez, vous pouvez activer une stratégie de date pour chaque point de terminaison de serveur sur ce volume. Cette stratégie prévoit que seuls les fichiers consultés (c’est-à-dire les lectures ou les écritures) dans la plage de jours que cette stratégie décrit sont conservés en local. Les fichiers qui ne sont pas consultés pendant le nombre de jours spécifié sont hiérarchisés. 

La hiérarchisation cloud utilise l’heure du dernier accès pour déterminer les fichiers à hiérarchiser. Le pilote de filtre de la hiérarchisation cloud (storagesync.sys) effectue le suivi de l’heure du dernier accès et enregistre les informations dans le magasin chaud de la hiérarchisation cloud. Vous pouvez récupérer l’accumulateur de chaleur et l’enregistrer dans un fichier CSV à l’aide d’une cmdlet PowerShell serveur-locale.

```powershell
# There is a single heat store for files on a volume / server endpoint / individual file.
# The heat store can get very large. If you only need to retrieve the "coolest" number of items, use -Limit and a number

# Import the PS module:
Import-Module '<SyncAgentInstallPath>\StorageSync.Management.ServerCmdlets.dll'

# VOLUME FREE SPACE: To get the order in which files will be tiered using the volume free space policy:
Get-StorageSyncHeatStoreInformation -VolumePath '<DriveLetter>:\' -ReportDirectoryPath '<FolderPathToStoreResultCSV>' -IndexName LastAccessTimeWithSyncAndTieringOrder

# DATE POLICY: To get the order in which files will be tiered using the date policy:
Get-StorageSyncHeatStoreInformation -VolumePath '<DriveLetter>:\' -ReportDirectoryPath '<FolderPathToStoreResultCSV>' -IndexName LastAccessTimeWithSyncAndTieringOrderV2

# Find the heat store information for a particular file:
Get-StorageSyncHeatStoreInformation -FilePath '<PathToSpecificFile>'
```

> [!IMPORTANT]
> La propriété last-accessed-timestamp n’est pas une propriété suivie par NTFS et, par conséquent, n’est pas visible par défaut dans l’Explorateur de fichiers. N’utilisez pas la propriété last-accessed-timestamp sur un fichier pour vérifier si la stratégie de date fonctionne comme prévu. Ce timestamp suit uniquement les écritures, pas les lectures. Utilisez la cmdlet indiquée pour obtenir la propriété last-accessed-timestamp pour cette évaluation.

> [!WARNING]
> N’activez pas la fonctionnalité NTFS de suivi de la propriété last-accessed-timestamp pour les fichiers et les dossiers. Cette fonctionnalité est désactivée par défaut, car elle a un impact important sur les performances. Azure File Sync suit les heures du dernier accès automatiquement et très efficacement et n’utilise pas cette fonctionnalité NTFS.

N’oubliez pas que la stratégie d’espace disponible sur le volume est toujours prioritaire, et, lorsqu’il n’y a pas suffisamment d’espace libre sur le volume pour conserver les fichiers autant de jours que défini par la stratégie de date, Azure File Sync poursuivra la hiérarchisation des fichiers les plus froids jusqu’à ce que le pourcentage d’espace libre sur le volume requis soit atteint.

Par exemple, si vous avez une stratégie de hiérarchisation par date de 60 jours et une stratégie d’espace disponible sur le volume de 20 % : Si, après avoir appliqué la stratégie de date, il reste moins de 20 % d’espace disponible sur le volume, la stratégie d’espace disponible sur le volume s’active et remplace la stratégie de date. Cela augmentera la hiérarchisation des fichiers, de telle sorte que la durée de conservation des données sur le serveur peut passer de 60 à 45 jours. Inversement, cette stratégie force la hiérarchisation des fichiers qui se situent en dehors de l’intervalle de temps, même si vous n’avez pas atteint votre seuil d’espace libre : un fichier est hiérarchisé au bout de 61 jours, même si votre volume est vide.

<a id="volume-free-space-guidelines"></a>
### <a name="how-do-i-determine-the-appropriate-amount-of-volume-free-space"></a>Comment faire pour déterminer la quantité d’espace libre du volume appropriée ?
La quantité de données à conserver localement est déterminée par la bande passante, le modèle d’accès au jeu de données et le budget. Si vous avez une connexion à bande passante étroite, vous pouvez conserver davantage de vos données localement afin de minimiser la latence pour vos utilisateurs. Autrement, vous pouvez baser la quantité d’espace libre du volume sur le taux de variation sur une période donnée. Par exemple, si vous savez qu’environ 10 % de votre jeu de données de 1 To changent ou sont activement utilisés chaque mois, vous pouvez conserver 100 Go localement de façon à ne pas rappeler fréquemment des fichiers. Si votre volume est de 2 To, vous pouvez conserver 5 % (soit 100 Go) localement, de sorte que les 95 % restants constituent votre pourcentage d’espace libre du volume. Toutefois, nous vous recommandons d’ajouter une mémoire tampon pour prendre en compte des périodes de variations plus importantes, c’est-à-dire en commençant par un pourcentage d’espace libre du volume inférieur, puis en l’ajustant si nécessaire par la suite. 

La conservation de davantage de données localement implique une baisse des coûts de sortie car moins de fichiers sont rappelés à partir d’Azure, mais requiert que vous conserviez davantage de stockage local, ce qui a un coût. Une fois qu’une instance d’Azure File Sync est déployée, vous pouvez examiner la sortie de votre compte de stockage pour évaluer sommairement si vos paramètres d’espace libre sur le volume sont appropriés pour votre utilisation. En supposant que le compte de stockage ne contient que votre point de terminaison cloud Azure File Sync (c’est-à-dire votre partage de synchronisation), une sortie élevée signifie que de nombreux fichiers sont rappelés à partir du cloud et que vous devez envisager d’augmenter votre cache local.

<a id="how-long-until-my-files-tier"></a>
### <a name="ive-added-a-new-server-endpoint-how-long-until-my-files-on-this-server-tier"></a>J’ai ajouté un nouveau point de terminaison de serveur. Au bout de combien de temps surviendra la hiérarchisation sur ce serveur ?

Si les fichiers doivent être hiérarchisés par ensemble, les stratégies sont évaluées une fois par heure. Vous pouvez rencontrer deux situations lors de la création d’un point de terminaison de serveur :

1. Lorsque vous ajoutez un nouveau point de terminaison de serveur, il existe souvent des fichiers à cet emplacement de serveur. Ils doivent être chargés en premier, avant que la hiérarchisation cloud puisse commencer. La stratégie d’espace libre du volume ne commence pas à fonctionner tant que le chargement initial de tous les fichiers n’est pas terminé. Toutefois, la stratégie de date facultative commence à fonctionner sur la base d’un fichier individuel, dès qu’un fichier a été chargé. L’intervalle d’une heure s’applique également dans ce cas de figure. 
2. Lorsque vous ajoutez un nouveau point de terminaison de serveur, il vous est possible de connecter un emplacement de serveur vide à un partage de fichiers Azure contenant vos données. Qu’il s’agisse d’un deuxième serveur ou d’une situation de récupération d’urgence. Si vous choisissez de télécharger l’espace de noms et de rappeler le contenu pendant le téléchargement initial vers votre serveur, une fois l’espace de noms indisponible, les fichiers sont renommés en fonction de l’horodatage de la dernière modification. Seul un grand nombre de fichiers sera rappelé conformément à la stratégie d’espace libre du volume et à la stratégie de date facultative.

<a id="is-my-file-tiered"></a>
### <a name="how-can-i-tell-whether-a-file-has-been-tiered"></a>Comment déterminer si un fichier a été hiérarchisé ?
Vous pouvez déterminer de plusieurs façons si un fichier a été hiérarchisé sur votre partage de fichiers Azure :
    
   *  **Vérifier les attributs du fichier.**
     Cliquez avec le bouton droit sur le fichier, accédez à **Détails**, puis faites défiler vers le bas jusqu’à la propriété **Attributs**. Un fichier hiérarchisé a les attributs suivants définis :     
        
        | Lettre de l’attribut | Attribut | Définition |
        |:----------------:|-----------|------------|
        | Un | Archivage | Indique que le fichier doit être sauvegardé par un logiciel de sauvegarde. Cet attribut est toujours défini, que le fichier soit hiérarchisé ou entièrement stocké sur le disque. |
        | P | Fichier partiellement alloué | Indique qu’il s’agit d’un fichier partiellement alloué. Un fichier partiellement alloué est un type de fichier spécial offert par NTFS, qui est efficace quand le flux sur disque du fichier est pratiquement vide. Azure File Sync utilise des fichiers partiellement alloués, car un fichier est soit entièrement hiérarchisé, soit partiellement rappelé. Dans un fichier entièrement hiérarchisé, le flux de fichier est stocké dans le cloud. Dans un fichier partiellement rappelé, cette partie du fichier est déjà sur le disque. Si un fichier est entièrement rappelé sur le disque, Azure File Sync le convertit d’un fichier partiellement alloué en fichier normal. Cet attribut est défini uniquement sur Windows Server 2016 et les versions antérieures.|
        | M | Rappeler en cas d’accès aux données | Indique que les données du fichier ne sont pas intégralement présentes sur le stockage local. La lecture du fichier entraîne l’extraction d’au moins une partie du contenu du fichier à partir d’un partage de fichiers Azure auquel le point de terminaison de serveur est connecté. Cet attribut est défini uniquement sur Windows Server 2019. |
        | L | Point d’analyse | Indique que le fichier a un point d’analyse. Un point d’analyse est un pointeur spécial destiné à être utilisé par un filtre de système de fichiers. Azure File Sync utilise des points d’analyse pour définir dans le filtre de système de fichiers d’Azure File Sync (StorageSync.sys) à quel endroit du cloud le fichier est stocké. Cela permet de bénéficier d’un accès aisé. Les utilisateurs n’ont pas besoin de savoir qu’Azure File Sync est utilisé, ni comment obtenir l’accès au fichier dans votre partage de fichiers Azure. Quand un fichier est entièrement rappelé, la synchronisation de fichiers Azure supprime le point d’analyse du fichier. |
        | O | Hors connexion | Indique qu’une partie ou la totalité du contenu du fichier n’est pas stockée sur le disque. Quand un fichier est entièrement rappelé, la synchronisation de fichiers Azure supprime cet attribut. |

        ![Boîte de dialogue Propriétés d’un fichier dans laquelle l’onglet Détails est sélectionné](media/storage-files-faq/azure-file-sync-file-attributes.png)
        
        Vous pouvez également voir les attributs de tous les fichiers d’un dossier en ajoutant le champ **Attributs** à l’affichage sous forme de tableau de l’Explorateur de fichiers. Pour ce faire, cliquez avec le bouton droit sur une colonne (par exemple, **Taille**), sélectionnez **Autres**, puis sélectionnez **Attributs** dans la liste déroulante.
        
   * **Utiliser `fsutil` pour rechercher les points d’analyse sur un fichier.**
       Comme indiqué dans l’option précédente, un fichier hiérarchisé a toujours un point d’analyse défini. Un pointeur d’analyse est un pointeur spécial pour le filtre de système de fichiers Azure File Sync (StorageSync.sys). Pour vérifier si un fichier a un point d’analyse, dans une fenêtre d’invite de commandes avec élévation de privilèges ou une fenêtre PowerShell, exécutez l’utilitaire `fsutil` :
    
        ```powershell
        fsutil reparsepoint query <your-file-name>
        ```

        Si le fichier comporte un point d’analyse, vous pouvez vous attendre à voir s’afficher le message **Valeur de la balise d’analyse : 0x8000001e**. Cette valeur hexadécimale est la valeur de point d’analyse détenue par Azure File Sync. La sortie contient également les données d’analyse qui représentent le chemin du fichier dans votre partage de fichiers Azure.

        > [!WARNING]  
        > La commande d’utilitaire `fsutil reparsepoint` permet également de supprimer un point d’analyse. N’exécutez pas cette commande, sauf si l’équipe d’ingénierie Azure File Sync vous le demande. L’exécution de cette commande peut entraîner une perte de données. 

<a id="afs-recall-file"></a>
### <a name="a-file-i-want-to-use-has-been-tiered-how-can-i-recall-the-file-to-disk-to-use-it-locally"></a>Un fichier que je souhaite utiliser a été hiérarchisé. Comment puis-je rappeler le fichier sur le disque pour l’utiliser localement ?
Pour rappeler un fichier sur le disque, le plus simple consiste à l’ouvrir. Le filtre de système de fichiers d’Azure File Sync (StorageSync.sys) télécharge le fichier à partir de votre partage de fichiers Azure de façon fluide, sans aucune intervention de votre part. Pour les types de fichiers qui peuvent être partiellement lus, tels que les fichiers multimédias ou .zip, l’ouverture d’un fichier n’entraîne pas le téléchargement du fichier entier.

Vous pouvez également utiliser PowerShell pour forcer le rappel d’un fichier. Cette option peut être utile si vous souhaitez rappeler plusieurs fichiers en même temps (par exemple tous les fichiers d’un dossier). Ouvrez une session PowerShell sur le nœud de serveur sur lequel Azure File Sync est installé, puis exécutez les commandes PowerShell suivantes :
    
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <path-to-to-your-server-endpoint>
```
Paramètres facultatifs :
* `-Order CloudTieringPolicy` rappellera d’abord les fichiers les plus récemment modifiés ou ayant été consultés, et est autorisée par la stratégie de hiérarchisation actuelle. 
    * Si la stratégie d’espace libre du volume est configurée, les fichiers sont rappelés jusqu’à ce que le paramètre de stratégie d’espace libre du volume soit atteint. Par exemple, si le paramètre de stratégie volume libre est de 20 %, le rappel s’arrête une fois que l’espace libre du volume atteint 20 %.  
    * Si la stratégie d’espace libre et de date du volume est configurée, les fichiers sont rappelés jusqu’à ce que le paramètre de stratégie d’espace libre et de date du volume soit atteint. Par exemple, si le paramètre de stratégie d’espace libre du volume est de 20 % et que la stratégie de date est de 7 jours, le rappel s’arrêtera une fois que l’espace libre du volume atteindra 20 % ou lorsque tous les fichiers consultés ou modifiés dans les 7 jours sont enregistrés en local.
* `-ThreadCount` détermine le nombre de fichiers rappelés en parallèle.
* `-PerFileRetryCount` détermine la fréquence à laquelle un rappel portant sur une fichier actuellement bloqué est tenté.
* `-PerFileRetryDelaySeconds` détermine le délai, en secondes, entre les tentatives de rappel, et doit systématiquement être utilisé avec le paramètre précédent.

Exemple :
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <path-to-to-your-server-endpoint> -ThreadCount 8 -Order CloudTieringPolicy -PerFileRetryCount 3 -PerFileRetryDelaySeconds 10
``` 

> [!Note]  
> - La cmdlet Invoke-StorageSyncFileRecall peut également être utilisée pour améliorer les performances de téléchargement de fichiers lors de l’ajout d’un nouveau point de terminaison de serveur à un groupe de synchronisation existant.  
>- Si le volume local qui héberge le serveur n’a pas assez d’espace libre pour rappeler toutes les données hiérarchisées, l’applet de commande `Invoke-StorageSyncFileRecall` échoue.  

<a id="sizeondisk-versus-size"></a>
### <a name="why-doesnt-the-size-on-disk-property-for-a-file-match-the-size-property-after-using-azure-file-sync"></a>Pourquoi la propriété *Taille sur le disque* pour un fichier ne correspond-elle pas à la propriété *Taille* après l’utilisation d’Azure File Sync ? 
L’Explorateur de fichiers Windows expose deux propriétés pour représenter la taille d’un fichier : **Taille** et **Taille sur le disque**. Ces propriétés ont un sens légèrement différent. La **Taille** représente la taille complète du fichier. La **Taille sur le disque** représente la taille du flux de fichier stocké sur le disque. Les valeurs de ces propriétés peuvent différer pour diverses raisons, telles que la compression, l’utilisation de la déduplication des données ou la hiérarchisation cloud avec Azure File Sync. Si un fichier est hiérarchisé sur un partage de fichiers Azure, la taille sur le disque est égale à zéro, car le flux de fichier est stocké dans votre partage de fichiers Azure, et non sur le disque. Un fichier peut également être partiellement hiérarchisé (ou partiellement rappelé). Dans un fichier partiellement hiérarchisé, une partie du fichier est sur le disque. Cela peut se produire quand des fichiers sont partiellement lus par des applications telles que des lecteurs multimédias ou des utilitaires de compression. 

<a id="afs-force-tiering"></a>
### <a name="how-do-i-force-a-file-or-directory-to-be-tiered"></a>Comment faire pour imposer la hiérarchisation d’un fichier ou répertoire ?

> [!NOTE]
> Lorsque vous sélectionnez un répertoire à hiérarchiser, seuls les fichiers qui se trouvent actuellement dans le répertoire sont hiérarchisés. Les fichiers créés après cette heure ne sont pas automatiquement hiérarchisés.

Quand elle est activée, la fonctionnalité de hiérarchisation cloud hiérarchise automatiquement les fichiers en fonction de la date du dernier accès et de la dernière modification afin d’atteindre le pourcentage d’espace libre du volume spécifié sur le point de terminaison cloud. Parfois, cependant, vous pouvez être amené à imposer une hiérarchisation manuelle d’un fichier. Cette opération peut s’avérer utile si vous enregistrez un fichier volumineux que vous n’envisagez pas de réutiliser pendant un certain temps, et que vous souhaitez dédier dans l’immédiat l’espace libre sur le volume à d’autres fichiers et dossiers. Vous pouvez forcer la hiérarchisation à l’aide des commandes PowerShell suivantes :

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncCloudTiering -Path <file-or-directory-to-be-tiered>
```

<a id="afs-image-thumbnail"></a>
### <a name="why-are-my-tiered-files-not-showing-thumbnails-or-previews-in-windows-explorer"></a>Pourquoi mes fichiers hiérarchisés n’affichent-ils pas de miniatures ou d'aperçus dans l’Explorateur Windows ?
Pour les fichiers hiérarchisés, les miniatures et aperçus ne sont pas visibles au niveau de votre point de terminaison de serveur. Ce comportement est normal puisque la fonctionnalité de cache des miniatures dans Windows ignore intentionnellement la lecture des fichiers avec l’attribut hors connexion. En cas d'activation de la hiérarchisation cloud, la lecture des fichiers hiérarchisés entraînerait leur téléchargement (rappel).

Ce comportement n’est pas spécifique à Azure File Sync. L’Explorateur Windows affiche une « croix grise » pour tous les fichiers dont l’attribut est hors connexion. L’icône X s’affiche lors de l’accès aux fichiers sur SMB. Pour obtenir une explication détaillée de ce comportement, consultez [https://blogs.msdn.microsoft.com/oldnewthing/20170503-00/?p=96105](https://blogs.msdn.microsoft.com/oldnewthing/20170503-00/?p=96105).

<a id="afs-tiering-disabled"></a>
### <a name="i-have-cloud-tiering-disabled-why-are-there-tiered-files-in-the-server-endpoint-location"></a>La hiérarchisation cloud est désactivée. Pourquoi y a-t-il des fichiers hiérarchisés à l’emplacement du point de terminaison de serveur ?

Il existe deux raisons pour lesquelles des fichiers hiérarchisés peuvent se trouver à l’emplacement du point de terminaison de serveur :

- Lorsque vous ajoutez un nouveau point de terminaison de serveur à un groupe de synchronisation existant, les métadonnées sont d’abord synchronisées avec le serveur et les fichiers sont ensuite téléchargés sur le serveur en arrière-plan. Les fichiers s’affichent comme étant hiérarchisés jusqu’à ce qu’ils soient téléchargés en local. Pour améliorer les performances de téléchargement des fichiers lors de l’ajout d’un nouveau serveur à un groupe de synchronisation, utilisez la cmdlet [Invoke-StorageSyncFileRecall](storage-sync-cloud-tiering.md#afs-recall-file).

- Si la hiérarchisation cloud a été activée sur le point de terminaison de serveur, puis désactivée, les fichiers restent hiérarchisés jusqu’à ce qu’ils soient accessibles.


## <a name="next-steps"></a>Étapes suivantes
* [Planification d’un déploiement Azure File Sync](storage-sync-files-planning.md)
