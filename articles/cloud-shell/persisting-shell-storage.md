---
title: Rendre persistants des fichiers dans Azure Cloud Shell | Microsoft Docs
description: Procédure pas à pas de conservation des fichiers avec Azure Cloud Shell.
services: azure
documentationcenter: ''
author: maertendMSFT
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 02/24/2020
ms.author: damaerte
ms.openlocfilehash: f1846c126e81ca5851cfbb1d782e5315ae10a82a
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92152272"
---
# <a name="persist-files-in-azure-cloud-shell"></a>Conserver des fichiers dans Azure Cloud Shell
Cloud Shell utilise le stockage de fichiers Azure pour conserver les fichiers entre les sessions. Lors du premier démarrage, Cloud Shell vous invite à associer un partage de fichiers nouveau ou existant afin de conserver les fichiers entre les sessions.

> [!NOTE]
> Bash et PowerShell utilisent le même partage de fichiers. Un seul partage de fichiers peut être associé à un montage automatique dans Cloud Shell.

> [!NOTE]
> Le pare-feu de stockage Azure n’est pas pris en charge pour les comptes de stockage Cloud Shell.

## <a name="create-new-storage"></a>Créer un stockage

Lorsque vous utilisez des paramètres de base et sélectionnez uniquement un abonnement, Cloud Shell crée trois ressources pour vous dans la région prise en charge la plus proche de vous :
* Groupe de ressources : `cloud-shell-storage-<region>`
* Compte de stockage : `cs<uniqueGuid>`
* Partage de fichiers : `cs-<user>-<domain>-com-<uniqueGuid>`

![Paramètre d’abonnement](media/persisting-shell-storage/basic-storage.png)

Le partage de fichiers est monté comme un `clouddrive` dans votre répertoire `$Home`. Cette opération n’a lieu qu’une seule fois, car le partage de fichiers est automatiquement monté dans les sessions suivantes. 

Le partage de fichiers contient également une image de 5 Go. Automatiquement créée pour vous, celle-ci conserve les données dans votre répertoire `$Home`. Cela vaut pour Bash et PowerShell.

## <a name="use-existing-resources"></a>Utiliser les ressources existantes

L’option Avancé vous permet d’associer des ressources existantes. Lorsque vous sélectionnez une région Cloud Shell, vous devez également choisir un compte de stockage de sauvegarde dans cette région. Par exemple, si votre région affectée est USA Ouest, vous devez alors associer un partage de fichiers qui se trouve également dans USA Ouest.

Lorsque l’invite de configuration du stockage s’affiche, sélectionnez **Afficher les paramètres avancés** pour visualiser des options supplémentaires. Les options de stockage renseignées filtrent les comptes de stockage localement redondant (LRS), de stockage géoredondant (GRS) et de stockage redondant interzone (ZRS). 

> [!NOTE]
> Il est recommandé d’utiliser des comptes de stockage GRS ou ZRS afin d’offrir une résilience supplémentaire pour votre partage de fichiers de sauvegarde. Le type de redondance dépend de vos objectifs et de votre budget. [En savoir plus sur les options de réplication pour les comptes de Stockage Azure](../storage/common/storage-redundancy.md).

![Paramètre Groupe de ressources](media/persisting-shell-storage/advanced-storage.png)

## <a name="securing-storage-access"></a>Sécurisation de l’accès au stockage
Pour la sécurité, chaque utilisateur doit approvisionner son propre compte de stockage.  Pour le contrôle d’accès en fonction du rôle Azure (Azure RBAC), les utilisateurs doivent disposer d’un accès contributeur ou supérieur au niveau du compte de stockage.

Cloud Shell utilise un partage de fichiers Azure dans un compte de stockage, au sein d’un abonnement spécifié. En raison des autorisations héritées, les utilisateurs disposant de droits d’accès suffisants à l’abonnement peuvent accéder à tous les comptes de stockage, ainsi qu'aux partages de fichiers contenus dans l’abonnement.

Les utilisateurs doivent verrouiller l’accès à leurs fichiers en définissant des autorisations au niveau du compte de stockage ou de l’abonnement.

## <a name="supported-storage-regions"></a>Régions de stockage prises en charge
Pour déterminer votre région actuelle, vous pouvez exécuter `env` dans Bash et localiser la variable `ACC_LOCATION`, ou exécuter `$env:ACC_LOCATION` à partir de PowerShell. Les partages de fichiers reçoivent une image de 5 Go créée pour conserver votre répertoire `$Home`.

Les machines Cloud Shell existent dans les régions suivantes :

|Domaine|Région|
|---|---|
|Amérique|USA Est, USA Centre Sud, USA Ouest|
|Europe|Europe Nord, Europe Ouest|
|Asie-Pacifique|Inde Centre, Asie Sud-Est|

Les clients doivent choisir une région primaire, sauf s’ils demandent que leurs données au repos soient stockées dans une région particulière. Dans ce cas, ils doivent utiliser une région de stockage secondaire.

### <a name="secondary-storage-regions"></a>Régions de stockage secondaires
Si une région de stockage secondaire est utilisée, le compte de stockage Azure associé réside dans une autre région que la machine Cloud Shell sur laquelle vous les montez. Par exemple, Jane peut choisir d’installer son compte de stockage dans la région secondaire Canada Est, alors que la machine sur laquelle elle est montée est restée dans une région primaire. Ses données au repos sont stockées au Canada, mais elles sont traitées aux États-Unis.

> [!NOTE]
> Si une région secondaire est utilisée, l’accès aux fichiers et le démarrage de Cloud Shell peuvent être ralentis.

Un utilisateur peut exécuter `(Get-CloudDrive | Get-AzStorageAccount).Location` dans PowerShell pour voir l’emplacement des partages de fichiers.

## <a name="restrict-resource-creation-with-an-azure-resource-policy"></a>Restreindre la création de ressources avec une stratégie de ressource Azure
Les comptes de stockage que vous créez dans Cloud Shell sont identifiés à l’aide de la balise `ms-resource-usage:azure-cloud-shell`. Si vous souhaitez interdire aux utilisateurs de créer des comptes de stockage par le biais de Cloud Shell, créez une [stratégie de ressource Azure pour les balises](../governance/policy/samples/index.md) déclenchée par cette balise spécifique.

## <a name="how-cloud-shell-storage-works"></a>Fonctionnement du stockage Cloud Shell 
Cloud Shell conserve les fichiers à l’aide des deux méthodes suivantes : 
* Création d’une image disque de votre répertoire `$Home` pour conserver la totalité du contenu figurant dans le répertoire. Cette image disque est enregistrée dans votre partage de fichiers spécifié en tant que `acc_<User>.img` à l’emplacement `fileshare.storage.windows.net/fileshare/.cloudconsole/acc_<User>.img` et synchronise automatiquement les modifications. 
* Montage du partage de fichiers spécifié en tant que `clouddrive` dans votre répertoire `$Home` pour l’interaction directe avec le partage de fichiers. `/Home/<User>/clouddrive` est mappé à `fileshare.storage.windows.net/fileshare`.
 
> [!NOTE]
> Tous les fichiers figurant dans votre répertoire `$Home`, tels que les clés SSH, sont conservés dans l’image disque utilisateur qui est stockée dans votre partage de fichiers monté. Appliquez les bonnes pratiques lors de la conservation d’informations dans votre répertoire `$Home` et votre partage de fichiers monté.

## <a name="clouddrive-commands"></a>Commandes CloudDrive

### <a name="use-the-clouddrive-command"></a>Utiliser la commande `clouddrive`
Dans Cloud Shell, vous pouvez exécuter une commande appelée `clouddrive`, qui vous permet de mettre à jour manuellement le partage de fichiers qui est monté dans Cloud Shell.

![Utilisation de la commande clouddrive](media/persisting-shell-storage/clouddrive-h.png)

### <a name="list-clouddrive"></a>Liste `clouddrive`
Pour détecter le partage de fichiers monté comme `clouddrive`, exécutez la commande `df`. 

Le chemin de fichier vers clouddrive affiche le nom de votre compte de stockage et le partage de fichiers dans l’URL. Par exemple : `//storageaccountname.file.core.windows.net/filesharename`

```
justin@Azure:~$ df
Filesystem                                          1K-blocks   Used  Available Use% Mounted on
overlay                                             29711408 5577940   24117084  19% /
tmpfs                                                 986716       0     986716   0% /dev
tmpfs                                                 986716       0     986716   0% /sys/fs/cgroup
/dev/sda1                                           29711408 5577940   24117084  19% /etc/hosts
shm                                                    65536       0      65536   0% /dev/shm
//mystoragename.file.core.windows.net/fileshareName 5368709120    64 5368709056   1% /home/justin/clouddrive
justin@Azure:~$
```

### <a name="mount-a-new-clouddrive"></a>Monter un nouveau clouddrive

#### <a name="prerequisites-for-manual-mounting"></a>Prérequis pour le montage manuel
Vous pouvez mettre à jour le partage de fichiers qui est associé à Cloud Shell à l’aide de la commande `clouddrive mount`.

Si vous montez un partage de fichiers existant, les comptes de stockage doivent se trouver dans la région Cloud Shell de votre choix. Récupérez l’emplacement en exécutant la commande `env` et en vérifiant `ACC_LOCATION`.

#### <a name="the-clouddrive-mount-command"></a>La commande `clouddrive mount`

> [!NOTE]
> Si vous montez un nouveau partage de fichiers, une nouvelle image utilisateur est créée pour votre répertoire `$Home`. L’image précédente `$Home` est conservée dans le partage de fichier précédent.

Exécutez la commande `clouddrive mount` avec les paramètres suivants :

```
clouddrive mount -s mySubscription -g myRG -n storageAccountName -f fileShareName
```

Pour afficher plus de détails, exécutez `clouddrive mount -h`, comme illustré ici :

![Exécution de la commande clouddrive mount](media/persisting-shell-storage/mount-h.png)

### <a name="unmount-clouddrive"></a>Démontage de clouddrive
Vous pouvez démonter un partage de fichiers monté sur Cloud Shell à tout moment. Dans la mesure où Cloud Shell requiert l’utilisation d’un partage de fichiers monté, vous serez invité à créer et monter un autre partage de fichiers lors de la prochaine session.

1. Exécutez `clouddrive unmount`.
2. Acceptez et confirmez les invites.

Votre partage de fichiers continue d’exister, tant que vous ne le supprimez pas manuellement. Cloud Shell ne fera plus de recherche pour ce partage de fichiers lors des sessions ultérieures. Pour afficher plus de détails, exécutez `clouddrive unmount -h`, comme illustré ici :

![Exécution de la commande clouddrive unmount](media/persisting-shell-storage/unmount-h.png)

> [!WARNING]
> L’exécution de cette commande ne va pas supprimer de ressources. Toutefois, la suppression manuelle d’un groupe de ressources, d’un compte de stockage ou d’un partage de fichiers mappé à Cloud Shell efface votre image disque du répertoire `$Home`, ainsi que tous les autres fichiers présents dans votre partage de fichiers. Il est impossible d’annuler cette opération.
## <a name="powershell-specific-commands"></a>Commandes PowerShell spécifiques

### <a name="list-clouddrive-azure-file-shares"></a>Répertorier les partages de fichiers Azure `clouddrive`
La cmdlet `Get-CloudDrive` récupère les informations du partage de fichiers Azure actuellement monté par le `clouddrive` dans Cloud Shell. <br>
![Exécution de Get-CloudDrive](media/persisting-shell-storage-powershell/Get-Clouddrive.png)

### <a name="unmount-clouddrive"></a>Démonter `clouddrive`
Vous pouvez démonter un partage de fichiers Azure monté sur Cloud Shell à tout moment. En cas de suppression du partage de fichiers Azure, vous serez invité à créer et à monter un nouveau partage de fichiers Azure lors de la prochaine session.

La cmdlet `Dismount-CloudDrive` démonte un partage de fichiers Azure à partir du compte de stockage actuel. Le démontage du `clouddrive` met fin à la session active. L’utilisateur sera invité à créer et monter un nouveau partage de fichiers Azure lors de la prochaine session.
![Exécution de Dismount-CloudDrive](media/persisting-shell-storage-powershell/Dismount-Clouddrive.png)

[!INCLUDE [PersistingStorage-endblock](../../includes/cloud-shell-persisting-shell-storage-endblock.md)]

Remarque : Si vous devez définir une fonction dans un fichier et l'appeler à partir des cmdlets PowerShell, l’opérateur point doit être inclus. Par exemple : . .\MyFunctions.ps1

## <a name="next-steps"></a>Étapes suivantes
[Démarrage rapide de Cloud Shell](quickstart.md) <br>
[En savoir plus sur le stockage de fichiers Microsoft Azure](../storage/files/storage-files-introduction.md) <br>
[En savoir plus sur les balises de stockage](../azure-resource-manager/management/tag-resources.md) <br>