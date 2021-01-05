---
title: Résoudre les problèmes liés à votre Azure Data Box et Azure Data Box Heavy
description: Explique comment résoudre les problèmes rencontrés dans Azure Data Box et Azure Data Box Heavy lors de la copie de données sur ces appareils.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: troubleshooting
ms.date: 09/10/2020
ms.author: alkohli
ms.openlocfilehash: bb70946fda4fad7a42fd885a2515cb0d82698eca
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92124673"
---
# <a name="troubleshoot-issues-related-to-azure-data-box-and-azure-data-box-heavy"></a>Résoudre les problèmes liés à Azure Data Box et Azure Data Box Heavy

Cet article fournit des informations détaillées sur la résolution des problèmes que vous pouvez rencontrer lors de l’utilisation d’Azure Data Box ou d’Azure Data Box Heavy pour les commandes d’importation. Il inclut la liste des erreurs possibles quand des données sont copiées vers Data Box ou quand des données sont chargées à partir de Data Box pour une commande d’importation.

Les informations contenues dans cet article ne s’appliquent pas aux ordres d’exportation créés pour Data Box.

## <a name="error-classes"></a>Classes d’erreur

Dans Data Box et Data Box Heavy, les erreurs sont récapitulées comme suit :

| Catégorie d’erreur*        | Description        | Action recommandée    |
|----------------------------------------------|---------|--------------------------------------|
| Noms de conteneurs ou de partages | Les noms de conteneurs ou de partages ne suivent pas les règles de nommage Azure.  |Téléchargez les listes d’erreurs. <br> Renommez les conteneurs ou les partages. [Plus d’informations](#container-or-share-name-errors)  |
| Limite de taille des conteneurs ou des partages | La taille totale des données dans les conteneurs ou les partages dépasse la limite Azure.   |Téléchargez les listes d’erreurs. <br> Réduisez le volume global des données dans le conteneur ou le partage. [Plus d’informations](#container-or-share-size-limit-errors)|
| Limite de taille des objets ou des fichiers | L’objet ou les fichiers présents dans les conteneurs ou les partages dépassent la limite Azure.|Téléchargez les listes d’erreurs. <br> Réduisez la taille de fichier dans le conteneur ou le partage. [Plus d’informations](#object-or-file-size-limit-errors) |    
| Type de fichier ou de données | Le format de données ou le type de fichier n’est pas pris en charge. |Téléchargez les listes d’erreurs. <br> Pour les objets blob de pages ou les disques managés, vérifiez que les données sont de alignées par éléments de 512 octets et copiées vers les dossiers créés au préalable. [Plus d’informations](#data-or-file-type-errors) |
| Erreurs non critiques liées aux objets blob ou aux fichiers  | Les noms d’objets blob ou de fichiers ne suivent pas les règles de nommage Azure ou le type de fichier n’est pas pris en charge. | Ces objets blob ou fichiers ne peuvent pas être copiés ou ils peuvent être changés. [Découvrez comment résoudre ces erreurs](#non-critical-blob-or-file-errors). |

\* Les quatre premières catégories d’erreur sont des erreurs critiques qui doivent être corrigées pour que vous puissiez continuer à préparer l’expédition.


## <a name="container-or-share-name-errors"></a>Erreurs liées aux noms de conteneurs ou de partages

Il s’agit d’erreurs liées aux noms de conteneurs et de partages.

### <a name="error_container_or_share_name_length"></a>ERROR_CONTAINER_OR_SHARE_NAME_LENGTH     

**Description de l’erreur :** Le nom du conteneur ou du partage doit comporter entre 3 et 63 caractères. 

**Résolution suggérée :** Le dossier sous le partage Data Box or Data Box Heavy (SMB/NFS) dans lequel vous avez copié des données devient un conteneur Azure dans votre compte de stockage. 

- Dans la page **Connexion et copie** de l’interface utilisateur web locale de l’appareil, téléchargez et consultez les fichiers d’erreur pour identifier les noms de dossiers comportant des problèmes.
- Changez le nom du dossier sous le partage Data Box ou Data Box Heavy pour vérifier les points suivants :

    - Le nom comprend entre 3 et 63 caractères.
    - Les noms peuvent uniquement contenir des lettres, des chiffres et des traits d’union.
    - Les noms ne peuvent pas commencer ou se terminer par des traits d’union.
    - Les noms ne peuvent pas contenir deux traits d’union consécutifs.
    - Exemples de noms valides : `my-folder-1`, `my-really-extra-long-folder-111`
    - Exemples de noms qui ne sont pas valides : `my-folder_1`, `my`, `--myfolder`, `myfolder--`, `myfolder!`

    Pour plus d’informations, consultez les conventions de nommage Azure pour les [noms de conteneurs](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names) et les [noms de partages](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#share-names).


### <a name="error_container_or_share_name_alpha_numeric_dash"></a>ERROR_CONTAINER_OR_SHARE_NAME_ALPHA_NUMERIC_DASH

**Description de l’erreur :** Le nom du conteneur ou du partage doit uniquement comporter des lettres, des chiffres ou des traits d’union.

**Résolution suggérée :** Le dossier sous le partage Data Box or Data Box Heavy (SMB/NFS) dans lequel vous avez copié des données devient un conteneur Azure dans votre compte de stockage. 

- Dans la page **Connexion et copie** de l’interface utilisateur web locale de l’appareil, téléchargez et consultez les fichiers d’erreur pour identifier les noms de dossiers comportant des problèmes.
- Changez le nom du dossier sous le partage Data Box ou Data Box Heavy pour vérifier les points suivants :

    - Le nom comprend entre 3 et 63 caractères.
    - Les noms peuvent uniquement contenir des lettres, des chiffres et des traits d’union.
    - Les noms ne peuvent pas commencer ou se terminer par des traits d’union.
    - Les noms ne peuvent pas contenir deux traits d’union consécutifs.
    - Exemples de noms valides : `my-folder-1`, `my-really-extra-long-folder-111`
    - Exemples de noms qui ne sont pas valides : `my-folder_1`, `my`, `--myfolder`, `myfolder--`, `myfolder!`

    Pour plus d’informations, consultez les conventions de nommage Azure pour les [noms de conteneurs](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names) et les [noms de partages](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#share-names).

### <a name="error_container_or_share_name_improper_dash"></a>ERROR_CONTAINER_OR_SHARE_NAME_IMPROPER_DASH

**Description de l’erreur :** Les noms de conteneurs et les noms de partages ne peuvent pas commencer ou se terminer par des traits d’union et ne peuvent pas contenir deux traits d’union consécutifs.

**Résolution suggérée :** Le dossier sous le partage Data Box or Data Box Heavy (SMB/NFS) dans lequel vous avez copié des données devient un conteneur Azure dans votre compte de stockage. 

- Dans la page **Connexion et copie** de l’interface utilisateur web locale de l’appareil, téléchargez et consultez les fichiers d’erreur pour identifier les noms de dossiers comportant des problèmes.
- Changez le nom du dossier sous le partage Data Box ou Data Box Heavy pour vérifier les points suivants :

    - Le nom comprend entre 3 et 63 caractères.
    - Les noms peuvent uniquement contenir des lettres, des chiffres et des traits d’union.
    - Les noms ne peuvent pas commencer ou se terminer par des traits d’union.
    - Les noms ne peuvent pas contenir deux traits d’union consécutifs.
    - Exemples de noms valides : `my-folder-1`, `my-really-extra-long-folder-111`
    - Exemples de noms qui ne sont pas valides : `my-folder_1`, `my`, `--myfolder`, `myfolder--`, `myfolder!`

    Pour plus d’informations, consultez les conventions de nommage Azure pour les [noms de conteneurs](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names) et les [noms de partages](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#share-names).
    
### <a name="error_file_or_directory_name_illegal"></a>ERROR_FILE_OR_DIRECTORY_NAME_ILLEGAL

**Description de l’erreur** : Les noms de répertoire ou de conteneur contiennent des caractères non conformes.

**Résolution suggérée :**  Les noms de répertoire ou de conteneur que vous avez copiés contiennent des caractères non pris en charge.

- Dans la page Connexion et copie de l’interface utilisateur web locale de l’appareil, téléchargez et consultez les fichiers d’erreur pour identifier les noms de dossiers problématiques. 
- Renommez le répertoire ou les conteneurs pour vous assurer qu’ils sont conformes aux conventions d’affectation de noms Azure.

Pour plus d’informations, consultez les conventions d’affectation de noms Azure pour les  [répertoires](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names) et les [conteneurs](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata#container-names).

## <a name="container-or-share-size-limit-errors"></a>Erreurs liées à la limite de taille des conteneurs ou des partages

Il s’agit d’erreurs liées à des données dont la taille dépasse la taille autorisée dans un conteneur ou un partage.

### <a name="error_container_or_share_capacity_exceeded"></a>ERROR_CONTAINER_OR_SHARE_CAPACITY_EXCEEDED

**Description de l’erreur :** Le partage de fichiers Azure limite un partage à 5 Tio de données, et les partages de fichiers volumineux ne sont pas activés sur le compte de stockage. Cette limite a été dépassée pour certains partages.

**Résolution suggérée :** Dans la page **Connexion et copie** de l’interface utilisateur web locale, téléchargez et consultez les fichiers d’erreur.

- Identifiez les dossiers qui rencontrent ce problème dans les journaux d'erreur et vérifiez que la taille des fichiers présents dans ce dossier est inférieure à 5 Tio.
- La limite de 5 Tio ne s'applique pas à un compte de stockage qui autorise les partages de fichiers volumineux. Cependant, des partages de fichiers volumineux doivent être configurés lorsque vous passez votre commande. 
  - Contactez le [Support Microsoft](data-box-disk-contact-microsoft-support.md) et demandez une nouvelle étiquette d'expédition.
  - [Activez les partages de fichiers volumineux sur le compte de stockage.](../storage/files/storage-files-how-to-create-large-file-share.md#enable-large-files-shares-on-an-existing-account)
  - [Développez les partages de fichiers sur le compte de stockage](../storage/files/storage-files-how-to-create-large-file-share.md#expand-existing-file-shares) et définissez le quota sur 100 Tio.
  
  
## <a name="object-or-file-size-limit-errors"></a>Erreurs liées à la limite de taille des objets ou des fichiers

Il s’agit d’erreurs liées à des données dépassant la taille maximale d’objet ou de fichier autorisée dans Azure. 

### <a name="error_blob_or_file_size_limit"></a>ERROR_BLOB_OR_FILE_SIZE_LIMIT

**Description de l’erreur :** La taille du fichier dépasse la taille de fichier maximale autorisée pour le chargement.

**Résolution suggérée :** Les tailles d’objets blob ou de fichiers dépassent la limite maximale autorisée pour le chargement.

- Dans la page **Connexion et copie** de l’interface utilisateur web locale, téléchargez et consultez les fichiers d’erreur.
- Vérifiez que les tailles d’objets blob et de fichiers ne dépassent pas les limites de taille d’objet Azure.

## <a name="data-or-file-type-errors"></a>Erreurs liées aux types de fichiers ou de données

Il s’agit d’erreurs liées au type de fichier ou type de données non pris en charge trouvé dans le conteneur ou le partage. 

### <a name="error_blob_or_file_size_alignment"></a>ERROR_BLOB_OR_FILE_SIZE_ALIGNMENT

**Description de l’erreur :** L’alignement du blob ou du fichier est incorrect.

**Résolution suggérée :** Sur Data Box ou Data Box Heavy, le partage d’objets blob de pages prend uniquement en charge les fichiers qui sont alignés par éléments de 512 octets (par exemple, VHD/VHDX). Toutes les données copiées vers le partage d’objets blob de pages sont chargées vers Azure sous la forme d’objets blob de pages.

Supprimez toutes les données non VHD/VHDX du partage d’objets blob de pages. Vous pouvez utiliser des partages pour les objets blob de blocs ou les fichiers Azure pour les données génériques.

Pour plus d’informations, consultez [Vue d’ensemble des objets blob de pages](../storage/blobs/storage-blob-pageblob-overview.md).

### <a name="error_blob_or_file_type_unsupported"></a>ERROR_BLOB_OR_FILE_TYPE_UNSUPPORTED

**Description de l’erreur :** Un type de fichier non pris en charge est présent dans un partage de disque managé. Seuls les disques durs virtuels fixes sont autorisés.

**Résolution suggérée :**

- Prenez soin de charger uniquement les disques durs virtuels fixes pour créer des disques managés.
- Les fichiers VHDX ou les disques durs virtuels **dynamiques** et de **différenciation** ne sont pas pris en charge.

### <a name="error_directory_disallowed_for_type"></a>ERROR_DIRECTORY_DISALLOWED_FOR_TYPE

**Description de l’erreur :** Un répertoire n’est autorisé dans aucun des dossiers préexistants pour les disques managés. Seuls les disques durs virtuels fixes sont autorisés dans ces dossiers.

**Résolution suggérée :** Pour les disques managés, dans chaque partage, les trois dossiers suivants, qui correspondent aux conteneurs de votre compte de stockage, sont créés : SSD Premium, HDD Standard et SSD Standard. Ces dossiers correspondent au niveau de performance pour le disque managé.

- Vérifiez que vous copiez vos données d’objet blob de pages (disques durs virtuels) dans l’un de ces dossiers existants.
- Un dossier ou un répertoire n’est pas autorisé dans ces dossiers existants. Supprimez tous les dossiers que vous avez créés dans les dossiers préexistants.

Pour plus d’informations, consultez [Copier sur des disques managés](data-box-deploy-copy-data-from-vhds.md#connect-to-data-box).

### <a name="reparse_point_error"></a>REPARSE_POINT_ERROR

**Description de l’erreur :** Les liens symboliques ne sont pas autorisés dans Linux. 

**Résolution suggérée :** Les liens symboliques sont généralement des liens, des canaux et d’autres fichiers de ce type. Retirez les liens, ou résolvez les liens et copiez les données.


## <a name="non-critical-blob-or-file-errors"></a>Erreurs non critiques liées aux objets blob ou aux fichiers

Toutes les erreurs non critiques liées aux noms des objets blob, des fichiers ou des conteneurs qui sont affichés pendant la copie des données sont récapitulées dans la section suivante. Si ces erreurs sont présentes, les noms sont modifiés pour être conformes aux conventions de nommage Azure. L’état de la commande correspondante pour le chargement des données s’est **terminé avec des avertissements**.  

### <a name="error_blob_or_file_name_character_control"></a>ERROR_BLOB_OR_FILE_NAME_CHARACTER_CONTROL

**Description de l’erreur :** Les noms d’objets blob ou de fichiers contiennent des caractères de contrôle non pris en charge.

**Résolution suggérée :** Les objets blob ou les fichiers que vous avez copiés contiennent des noms avec des caractères non pris en charge.

Dans la page **Connexion et copie** de l’interface utilisateur web locale, téléchargez et consultez les fichiers d’erreur.
Retirez ou renommez les fichiers pour supprimer les caractères non pris en charge.

Pour plus d’informations, consultez les conventions de nommage Azure pour les [noms d’objets blob](/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata#blob-names) et les [noms de fichiers](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names).

### <a name="error_blob_or_file_name_character_illegal"></a>ERROR_BLOB_OR_FILE_NAME_CHARACTER_ILLEGAL

**Description de l’erreur :** Les noms d’objets blob ou de fichiers contiennent des caractères non autorisés.

**Résolution suggérée :** Les objets blob ou les fichiers que vous avez copiés contiennent des noms avec des caractères non pris en charge.

Dans la page **Connexion et copie** de l’interface utilisateur web locale, téléchargez et consultez les fichiers d’erreur.
Retirez ou renommez les fichiers pour supprimer les caractères non pris en charge.

Pour plus d’informations, consultez les conventions de nommage Azure pour les [noms d’objets blob](/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata#blob-names) et les [noms de fichiers](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names).


### <a name="error_blob_or_file_name_ending"></a>ERROR_BLOB_OR_FILE_NAME_ENDING

**Description de l’erreur :** Les noms d’objets blob ou de fichiers se terminent par des caractères non valides.

**Résolution suggérée :** Les objets blob ou les fichiers que vous avez copiés contiennent des noms avec des caractères non pris en charge.

Dans la page **Connexion et copie** de l’interface utilisateur web locale, téléchargez et consultez les fichiers d’erreur.
Retirez ou renommez les fichiers pour supprimer les caractères non pris en charge.

Pour plus d’informations, consultez les conventions de nommage Azure pour les [noms d’objets blob](/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata#blob-names) et les [noms de fichiers](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names).


### <a name="error_blob_or_file_name_segment_count"></a>ERROR_BLOB_OR_FILE_NAME_SEGMENT_COUNT

**Description de l’erreur :** Le nom d’objet blob ou de fichier contient trop de segments de chemin.

**Résolution suggérée :** Les objets BLOB ou les fichiers que vous avez copiés dépassent le nombre maximal de segments de chemin. Un segment de chemin représente la chaîne située entre deux caractères délimiteurs consécutifs ; par exemple, la barre oblique (/).

- Dans la page **Connexion et copie** de l’interface utilisateur web locale, téléchargez et consultez les fichiers d’erreur.
- Vérifiez que les [noms d’objets blob](/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata#blob-names) et les [noms de fichiers](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names) sont conformes aux conventions de nommage Azure.

### <a name="error_blob_or_file_name_aggregate_length"></a>ERROR_BLOB_OR_FILE_NAME_AGGREGATE_LENGTH

**Description de l’erreur :** Le nom du blob ou du fichier est trop long.

**Résolution suggérée :** Les noms d’objets blob ou de fichiers dépassent la longueur maximale.

- Dans la page **Connexion et copie** de l’interface utilisateur web locale, téléchargez et consultez les fichiers d’erreur.
- Le nom d’objet blob ne doit pas comporter plus de 1 024 caractères.
- Supprimez ou renommez les objets blob ou les fichiers afin que les noms ne dépassent pas 1 024 caractères.

Pour plus d’informations, consultez les conventions de nommage Azure pour les noms d’objets blob et les noms de fichiers.

### <a name="error_blob_or_file_name_component_length"></a>ERROR_BLOB_OR_FILE_NAME_COMPONENT_LENGTH

**Description de l’erreur :** L’un des segments du nom de blob ou de fichier est trop long.

**Résolution suggérée :** L’un des segments de chemin dans le nom d’objet blob ou de fichier dépasse le nombre maximal de caractères. Un segment de chemin représente la chaîne située entre deux caractères délimiteurs consécutifs ; par exemple, la barre oblique (/).

- Dans la page **Connexion et copie** de l’interface utilisateur web locale, téléchargez et consultez les fichiers d’erreur.
- Vérifiez que les [noms d’objets blob](/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata#blob-names) et les [noms de fichiers](/rest/api/storageservices/naming-and-referencing-shares--directories--files--and-metadata#directory-and-file-names) sont conformes aux conventions de nommage Azure.


### <a name="error_container_or_share_name_disallowed_for_type"></a>ERROR_CONTAINER_OR_SHARE_NAME_DISALLOWED_FOR_TYPE

**Description de l’erreur :** Des noms de conteneurs non valides sont spécifiés pour les partages de disque managé.

**Résolution suggérée :** Pour les disques managés, dans chaque partage, les dossiers suivants, qui correspondent aux conteneurs de votre compte de stockage, sont créés : SSD Premium, HDD Standard et SSD Standard. Ces dossiers correspondent au niveau de performance pour le disque managé.

- Vérifiez que vous copiez vos données d’objet blob de pages (disques durs virtuels) dans l’un de ces dossiers existants. Seules les données provenant de ces conteneurs existants sont chargées dans Azure.
- Tout autre dossier qui est créé au même niveau que le disque SSD Premium, HDD Standard et SSD Standard ne correspond pas à un niveau de performance valide et ne peut pas être utilisé.
- Supprimez les fichiers ou dossiers créés hors des niveaux de performance.

Pour plus d’informations, consultez [Copier sur des disques managés](data-box-deploy-copy-data-from-vhds.md#connect-to-data-box).


## <a name="next-steps"></a>Étapes suivantes

- Découvrez la [configuration système requise du Stockage Blob Data Box](data-box-system-requirements-rest.md).