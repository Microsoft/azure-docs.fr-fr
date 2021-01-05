---
title: azcopy remove | Microsoft Docs
description: Cet article fournit des informations de référence sur la commande azcopy remove.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: d79b647d216fe28241e5891def574ab598304828
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89078218"
---
# <a name="azcopy-remove"></a>azcopy remove

Supprimez des objets BLOB ou des fichiers d’un compte de stockage Azure.

## <a name="synopsis"></a>Synopsis

```azcopy
azcopy remove [resourceURL] [flags]
```

## <a name="related-conceptual-articles"></a>Articles conceptuels associés

- [Bien démarrer avec AzCopy](storage-use-azcopy-v10.md)
- [Transférer des données avec AzCopy et le Stockage Blob](storage-use-azcopy-blobs.md)
- [Transférer des données avec AzCopy et le stockage de fichiers](storage-use-azcopy-files.md)
- [Configurer, optimiser et dépanner AzCopy](storage-use-azcopy-configure.md)

## <a name="examples"></a>Exemples

Suppression d’un seul objet BLOB à l’aide d’un jeton SAS :

```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/blob]?[SAS]"
```

Suppression d’un répertoire virtuel entier à l’aide d’un jeton SAS :
 
```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/directory]?[SAS]" --recursive=true
```

Supprimer uniquement les objets BLOB dans un répertoire virtuel, mais ne pas supprimer les sous-répertoires ou les objets BLOB de ces sous-répertoires :

```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/virtual/dir]" --recursive=false
```

Supprimer un sous-ensemble d’objets blob dans un répertoire virtuel (par exemple, supprimer uniquement les fichiers .jpg et PDF, ou uniquement les objets blob dont le nom est « `exactName` ») :

```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/directory]?[SAS]" --recursive=true --include-pattern="*.jpg;*.pdf;exactName"
```

Supprimer l’intégralité d’un répertoire virtuel, mais exclure certains objets blob (par exemple, tous ceux dont le nom commence par « foo » ou se termine par « bar ») :

```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/directory]?[SAS]" --recursive=true --exclude-pattern="foo*;*bar"
```

Supprimez des objets BLOB et des répertoires virtuels spécifiques en plaçant leurs chemins d’accès relatifs (NON codés URL) dans un fichier :

```azcopy
azcopy rm "https://[account].blob.core.windows.net/[container]/[path/to/parent/dir]" --recursive=true --list-of-files=/usr/bar/list.txt
- file content:
    dir1/dir2
    blob1
    blob2
```
Supprimez un seul fichier d’un compte de stockage d’objets BLOB qui a un espace de noms hiérarchique (inclure/exclure non pris en charge) :

```azcopy
azcopy rm "https://[account].dfs.core.windows.net/[container]/[path/to/file]?[SAS]"
```

Supprimez un seul répertoire d’un compte de stockage d’objets BLOB qui a un espace de noms hiérarchique (inclure/exclure non pris en charge) :

```azcopy
azcopy rm "https://[account].dfs.core.windows.net/[container]/[path/to/directory]?[SAS]"
```

## <a name="options"></a>Options

Chaîne **--delete-snapshots**   Par défaut, l’opération de suppression échoue si un objet blob comporte des captures instantanées. Spécifiez « `include` » pour supprimer l’objet blob racine et tous ses instantanés. Vous pouvez également spécifier « `only` » pour supprimer uniquement les captures instantanées et conserver l’objet blob racine.

Chaîne **--exclude-path**   Exclut ces chemins d’accès lors de la suppression. Cette option ne prend pas en charge les caractères génériques (*). Vérifie le préfixe du chemin d’accès relatif. Par exemple : `myFolder;myFolder/subDirName/file.pdf`

Chaîne **--exclude-pattern** Exclut les fichiers dont le nom correspond à la liste de caractères génériques. Par exemple : `*.jpg`;`*.pdf`;`exactName`

**--force-if-read-only**   Lors de la suppression d’un fichier ou d’un dossier Azure Files, force la suppression même si l’objet existant a son attribut en lecture seule défini.

**--help**   Aide pour la suppression.

Chaîne **--include-path**   Inclut uniquement ces chemins lors de la suppression. Cette option ne prend pas en charge les caractères génériques (*). Vérifie le préfixe du chemin d’accès relatif. Par exemple : `myFolder;myFolder/subDirName/file.pdf`

**--include-pattern** string  Inclut uniquement les fichiers dont le nom correspond à la liste de caractères génériques. Par exemple : *`.jpg`;* `.pdf`;`exactName`

Chaîne **--list-of-files**  Définit l’emplacement d’un fichier qui contient la liste des fichiers et répertoires à supprimer. Les chemins d’accès relatifs doivent être délimités par des sauts de ligne, et les chemins d’accès ne doivent pas être codés URL. 

Chaîne **--list-of-versions**   Spécifie un fichier où chaque ID de version est listé sur une ligne distincte. Assurez-vous que la source pointe vers un objet blob unique et que tous les ID de version spécifiés dans le fichier à l’aide de cet indicateur appartiennent uniquement à l’objet blob source. Les ID de version spécifiés de l’objet blob donné seront supprimés du stockage Azure. 

Chaîne **--log-level**   Définit la verbosité du journal pour le fichier journal. Les niveaux disponibles sont les suivants : `INFO` (toutes les demandes/réponses), `WARNING` (réponses lentes), `ERROR` (uniquement les demandes ayant échoué) et `NONE` (aucun journal de sortie). (par défaut `INFO`) (par défaut `INFO`)

**--recursive**    Examine les sous-répertoires de manière récursive lors de la synchronisation des répertoires.

## <a name="options-inherited-from-parent-commands"></a>Options héritées des commandes parentes

|Option|Description|
|---|---|
|--cap-mbps float|Limite la vitesse de transfert, en mégabits par seconde. Par moment, le débit peut dépasser légèrement cette limite. Si cette option est définie sur zéro ou si elle est omise, le débit n’est pas limité.|
|--output-type (chaîne)|Met en forme la sortie de la commande. Les formats possibles sont « text » et « JSON ». La valeur par défaut est « text ».|
|--trusted-microsoft-suffixes (chaîne)   |Spécifie des suffixes de domaine supplémentaires où des jetons de connexion Azure Active Directory peuvent être envoyés.  La valeur par défaut est «  *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net ». Tous les éléments répertoriés ici sont ajoutés à la valeur par défaut. Pour la sécurité, vous devez placer uniquement des domaines Microsoft Azure ici. Séparez plusieurs entrées par des points-virgules.|

## <a name="see-also"></a>Voir aussi

- [azcopy](storage-ref-azcopy.md)
