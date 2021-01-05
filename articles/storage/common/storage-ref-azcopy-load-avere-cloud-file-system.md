---
title: azcopy load clfs | Microsoft Docs
titleSuffix: Azure Storage
description: Cet article fournit des informations de référence sur la commande azcopy load clfs.
author: normesta
ms.service: storage
ms.topic: reference
ms.date: 07/24/2020
ms.author: normesta
ms.subservice: common
ms.reviewer: zezha-msft
ms.openlocfilehash: 8f0a8903f90b134c35d9adb35a493d989d414b56
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87293732"
---
# <a name="azcopy-load-clfs"></a>azcopy load clfs

Transfère les données locales dans un conteneur et les stocke au format CLFS (Cloud FileSystem) Avere de Microsoft.

## <a name="synopsis"></a>Synopsis

La commande load copie des données dans des conteneurs de stockage Blob Azure, puis stocke ces données au format CLFS (Cloud FileSystem) Avere de Microsoft. Le format CLFS propriétaire est utilisé par les produits Azure HPC Cache et Avere vFXT pour Azure.

Pour tirer parti de cette commande, installez l’extension nécessaire à l’aide de : pip3 install clfsload~=1.0.23. Vérifiez que CLFSLoad.py se trouve dans votre chemin. Pour plus d’informations sur cette étape, consultez [https://aka.ms/azcopy/clfs](https://aka.ms/azcopy/clfs).

Cette commande est une option simple permettant de déplacer des données existantes vers le stockage cloud en vue de les utiliser avec des produits de cache informatique haute performances Microsoft spécifiques. 

Étant donné que ces produits utilisent un format de système de fichiers cloud propriétaire pour gérer les données, ces données ne peuvent pas être chargées par le biais de la commande de copie native. 

Au lieu de cela, les données doivent être chargées par le biais du produit de cache lui-même ou à l’aide de cette commande de chargement, qui utilise le bon format propriétaire.
Cette commande vous permet de transférer des données sans utiliser le cache, par exemple pour préremplir le stockage ou ajouter des fichiers à une plage de travail sans augmenter la charge du cache.

La destination est un conteneur de stockage Azure vide. Une fois le transfert terminé, le conteneur de destination peut être utilisé avec une instance Azure HPC Cache ou un cluster Avere vFXT pour Azure.

> [!NOTE] 
> Il s’agit d’une préversion de la commande load. N’hésitez pas à signaler tout problème dans le dépôt GitHub AzCopy.

```
azcopy load clfs [local dir] [container URL] [flags]
```

## <a name="related-conceptual-articles"></a>Articles conceptuels associés

- [Bien démarrer avec AzCopy](storage-use-azcopy-v10.md)
- [Transférer des données avec AzCopy et le Stockage Blob](storage-use-azcopy-blobs.md)
- [Transférer des données avec AzCopy et le stockage de fichiers](storage-use-azcopy-files.md)
- [Configurer, optimiser et dépanner AzCopy](storage-use-azcopy-configure.md)

## <a name="examples"></a>Exemples

Charger un répertoire entier dans un conteneur avec une signature d’accès partagé au format CLFS :

```azcopy
azcopy load clfs "/path/to/dir" "https://[account].blob.core.windows.net/[container]?[SAS]" --state-path="/path/to/state/path"
```

## <a name="options"></a>Options

**--compression-type** chaîne   Spécifier le type de compression à utiliser pour les transferts. Les valeurs disponibles sont : `DISABLED`,`LZ4`. (valeur par défaut `LZ4`)

**--help**    Aide pour la commande `azcopy load clfs`.

**--log-level** chaîne   Définir le niveau de détail pour le fichier journal. Niveaux disponibles : `DEBUG`, `INFO`, `WARNING`, `ERROR`. (valeur par défaut `INFO`)

**--max-errors** uint32   Spécifier le nombre maximal d’échecs de transfert à tolérer. Si suffisamment d’erreurs se produisent, le travail est arrêté immédiatement.

**--new-session**   Démarrer un nouveau travail au lieu de poursuivre un travail existant dont les informations de suivi sont conservées à `--state-path`. (par défaut : true)

**--preserve-hardlinks**    Conserver les relations de liens physiques.

**--state-path** chaîne   Chemin obligatoire vers un répertoire local pour le suivi de l’état du travail. Le chemin doit pointer vers un répertoire existant afin de reprendre un travail. Il doit être vide pour un nouveau travail.

## <a name="options-inherited-from-parent-commands"></a>Options héritées des commandes parentes

|Option|Description|
|---|---|
|--cap-mbps float|Limite la vitesse de transfert, en mégabits par seconde. Par moment, le débit peut dépasser légèrement cette limite. Si cette option est définie sur zéro ou si elle est omise, le débit n’est pas limité.|
|--output-type (chaîne)|Met en forme la sortie de la commande. Les formats possibles sont « text » et « JSON ». La valeur par défaut est « text ».|
|--trusted-microsoft-suffixes (chaîne)   | Spécifie des suffixes de domaine supplémentaires où des jetons de connexion Azure Active Directory peuvent être envoyés.  La valeur par défaut est «  *.core.windows.net;* .core.chinacloudapi.cn; *.core.cloudapi.de;* .core.usgovcloudapi.net ». Tous les éléments répertoriés ici sont ajoutés à la valeur par défaut. Pour la sécurité, vous devez placer uniquement des domaines Microsoft Azure ici. Séparez plusieurs entrées par des points-virgules.|

## <a name="see-also"></a>Voir aussi

- [azcopy](storage-ref-azcopy.md)
