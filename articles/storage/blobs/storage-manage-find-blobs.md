---
title: Gérer et rechercher des données Azure Blob à l’aide de balises d’index de blob (préversion)
description: Découvrez comment utiliser des balises d’index de blobs pour catégoriser, gérer et interroger afin de découvrir des objets blob.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 10/19/2020
ms.service: storage
ms.subservice: common
ms.topic: conceptual
ms.reviewer: klaasl
ms.custom: references_regions
ms.openlocfilehash: 4f84c3c2f6fc671a8cb6ac70313361540e3dd815
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95523278"
---
# <a name="manage-and-find-azure-blob-data-with-blob-index-tags-preview"></a>Gérer et rechercher des données Azure Blob à l’aide de balises d’index de blob (préversion)

Plus les jeux de données croissent, plus la recherche d’un objet spécifique dans une mer de données peut s’avérer difficile. Les balises d’index de blob fournissent des capacités de gestion et de découverte de données à l’aide d’attributs de balise d’index clé-valeur. Vous pouvez classer et rechercher des objets dans un seul conteneur ou dans tous les conteneurs de votre compte de stockage. À mesure que les exigences en matière de données changent, les objets peuvent être catégorisés dynamiquement en mettant à jour leurs balises d’index. Les objets peuvent rester à leur emplacement avec leur organisation de conteneur actuelle.

Les balises d’index de blob vous permettent d’effectuer les actions suivantes :

- Classer dynamiquement vos blobs à l’aide de balises d’index clé-valeur
- Rechercher rapidement des blobs spécifiques balisés dans un compte de stockage entier
- Spécifier des comportements conditionnels pour les API d’objets blob en fonction de l’évaluation des étiquettes d’index
- Utiliser des balises d’index pour les contrôles avancés sur les fonctionnalités telles que la [gestion du cycle de vie des blobs](storage-lifecycle-management-concepts.md)

Imaginez un scénario dans lequel vous avez des millions de blobs dans votre compte de stockage, accessibles par de nombreuses applications différentes. Vous souhaitez rechercher toutes les données associées à un projet. Vous n’êtes pas certain de l’étendue concernée, car les données peuvent être réparties sur plusieurs conteneurs avec des conventions d’affectation de noms différentes. Toutefois, vos applications chargent toutes les données avec des balises en fonction de leur projet. Au lieu de rechercher parmi des millions de blobs et de comparer les noms et les propriétés, vous pouvez utiliser `Project = Contoso` comme critère de découverte. Un index de blob filtrera tous les conteneurs sur l’ensemble de votre compte de stockage pour rechercher et renvoyer rapidement un simple ensemble de 50 blobs à partir de `Project = Contoso`.

Pour bien démarrer, consultez des exemples d’utilisation d’un index de blob dans [Utiliser des balises d’index de blob pour gérer et rechercher des données](storage-blob-index-how-to.md).

## <a name="blob-index-tags-and-data-management"></a>Étiquettes d’index d’objet blob et gestion des données

Les préfixes de nom de blob et de conteneur constituent des catégorisations unidimensionnelles. Les balises d’index de blob permettent une catégorisation multidimensionnelle pour les [types de données blob (bloc, ajout ou page)](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs). La catégorisation multidimensionnelle est indexée en mode natif par Stockage Blob Azure, ce qui vous permet de trouver rapidement vos données.

Considérez les cinq objets blob suivants dans votre compte de stockage :

- *container1/transaction.csv*
- *container2/campaign.docx*
- *photos/bannerphoto.png*
- *archives/completed/2019review.pdf*
- *logs/2020/01/01/logfile.txt*


Ces blobs sont séparés à l’aide d’un préfixe de *nom de conteneur/dossier virtuel/blob*. Vous pouvez définir un attribut de balise d’index `Project = Contoso` sur ces cinq blobs pour les classer ensemble tout en maintenant leur organisation de préfixe actuelle. L’ajout de balises d’index évite d’avoir à déplacer les données en exposant la capacité de filtrer et de rechercher des données à l’aide de l’index.

## <a name="setting-blob-index-tags"></a>Définition des étiquettes d’un index d’objet blob

Les étiquettes d’un index d’objet blob sont des attributs clé-valeur pouvant être appliqués à des objets nouveaux ou existants dans votre compte de stockage. Vous pouvez spécifier des balises d’index pendant le processus de chargement en utilisant les opérations [Put Blob](/rest/api/storageservices/put-blob), [Put Block List](/rest/api/storageservices/put-block-list) ou [Copy Blob](/rest/api/storageservices/copy-blob) et l’en-tête `x-ms-tags` facultatif. Si vous avez déjà des blobs dans votre compte de stockage, appelez [Set Blob Tags](/rest/api/storageservices/set-blob-tags) en transmettant un document XML mis en forme avec les balises d’index de blob dans le corps de la demande.

> [!IMPORTANT]
> La définition des balises d’index de blob peut être effectuée par le [propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) et par toute personne disposant d’une signature d’accès partagé qui a l’autorisation d’accéder aux balises du blob (l’autorisation SAP `t`).
>
> En outre, les utilisateurs RBAC dotés de l’autorisation `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write` peuvent effectuer cette opération.

Vous pouvez appliquer une étiquette individuelle à votre objet blob pour décrire le moment où vos données finissent d’être traitées.

> "processedDate" = '2020-01-01'

Vous pouvez appliquer plusieurs étiquettes à votre objet blob pour mieux décrire les données.

> "Project" = 'Contoso'  
> "Classified" = 'True'  
> "Status" = 'Unprocessed'  
> "Priority" = '01'

Pour modifier les attributs de balise d’index existants, récupérez les attributs de balise existants, modifiez-les et remplacez-les à l’aide d’une opération [Set Blob Tags](/rest/api/storageservices/set-blob-tags). Pour supprimer toutes les balises d’index du blob, appelez l’opération `Set Blob Tags` sans spécifier d’attributs de balise. Comme les balises d’index de blob sont une sous-ressource du contenu des données blob, `Set Blob Tags` ne modifie pas le contenu sous-jacent ni la propriété Last-Modified-Time ou l’ETag du blob. Vous pouvez créer ou modifier des balises d’index pour tous les blobs de base actuels et les versions précédentes. Toutefois, les balises sur les instantanés ou les blobs supprimés de manière réversible ne peuvent pas être modifiées.

Les limites suivantes s’appliquent aux étiquettes d’un index d’objet blob :

- Chaque objet blob peut avoir jusqu’à 10 étiquettes d’index d’objet blob.
- Les clés de balise doivent comporter entre 1 et 128 caractères.
- Les valeurs de balise doivent comporter entre 0 et 256 caractères.
- Les clés et les valeurs d’étiquette sont sensibles à la casse.
- Les clés et les valeurs de balise prennent uniquement en charge les données de type chaîne. Les nombres, les dates, les heures ou les caractères spéciaux sont enregistrés sous forme de chaîne.
- Les clés et les valeurs d’étiquette doivent respecter les règles de nommage suivantes :
  - Caractères alphanumériques :
    - **a** à **z** (lettres minuscules)
    - **A** à **Z** (lettres majuscules)
    - **0** à **9** (chiffres)
  - Caractères spéciaux valides : espace, plus, moins, point, deux-points, égal, trait de soulignement, barre oblique (` +-.:=_/`)

## <a name="getting-and-listing-blob-index-tags"></a>Obtention et affichage de la liste des étiquettes d’index d’objet blob

Les balises d’index de blob sont stockées en tant que sous-ressource avec les données blob et peuvent être récupérées indépendamment du contenu sous-jacent des données blob. Les balises d’index de blob pour un seul blob peuvent être récupérées avec l’opération [Get Blob Tags](/rest/api/storageservices/get-blob-tags). L’opération [List Blobs](/rest/api/storageservices/list-blobs) avec le paramètre `include:tags` renverra également tous les blobs dans un conteneur avec leurs balises d’index de blob.

> [!IMPORTANT]
> L’obtention et l’affichage de la liste des balises d’index de blob peuvent être effectués par le [propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) et par toute personne disposant d’une signature d’accès partagé qui a l’autorisation d’accéder aux balises du blob (l’autorisation SAP `t`).
>
> En outre, les utilisateurs RBAC dotés de l’autorisation `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read` peuvent effectuer cette opération.

Pour tous les blobs dotés d’au moins une balise d’index de blob, la valeur `x-ms-tag-count` est retournée dans les opérations [List Blobs](/rest/api/storageservices/list-blobs), [Get Blob](/rest/api/storageservices/get-blob) et [Get Blob Properties](/rest/api/storageservices/get-blob-properties), indiquant le nombre de balises d’index de blob sur le blob.

## <a name="finding-data-using-blob-index-tags"></a>Recherche de données à l’aide d’étiquettes d’index d’objet blob

Le moteur d’indexation expose ces attributs clé-valeur dans un index multidimensionnel. Une fois que vous avez défini vos balises d’index, elles existent sur le blob et peuvent être récupérées immédiatement. La mise à jour de l’index de blob peut prendre un certain temps. Une fois l’index de blob mis à jour, vous pouvez utiliser des capacités natives de requête et de découverte offertes par Stockage Blob.

L’opération [Find Blobs By Tags](/rest/api/storageservices/find-blobs-by-tags) vous permet d’obtenir un ensemble filtré de blobs retournés dont les balises d’index correspondent à une expression de requête donnée. `Find Blobs by Tags` prend en charge le filtrage sur tous les conteneurs de votre compte de stockage, ou vous pouvez limiter le filtrage à un seul conteneur. Comme toutes les clés et valeurs des balises d’index sont des chaînes, les opérateurs de relation utilisent un tri lexicographique.

> [!IMPORTANT]
> La recherche de données à l’aide de balises d’index de blob peut être effectuée par le [propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) et par toute personne disposant d’une signature d’accès partagé qui a l’autorisation de rechercher des blobs par balise (l’autorisation SAP `f`).
>
> En outre, les utilisateurs RBAC dotés de l’autorisation `Microsoft.Storage/storageAccounts/blobServices/containers/blobs/filter/action` peuvent effectuer cette opération.

Les critères suivants s’appliquent au filtrage de l’index d’objet blob :

- Les clés des étiquettes doivent être placées entre guillemets doubles (").
- Les valeurs des étiquettes et les noms des conteneurs doivent être placés entre guillemets simples (').
- Le caractère @ est autorisé uniquement pour le filtrage sur un nom de conteneur spécifique (p. ex., `@container = 'ContainerName'`).
- Les filtres sont appliqués avec un tri lexicographique sur les chaînes.
- Des opérations de comparaison dans un même sens sur la même clé ne sont pas valides (p. ex., `"Rank" > '10' AND "Rank" >= '15'`).
- Lorsque vous utilisez REST pour créer une expression de filtre, les caractères doivent être encodés sous forme d’URI.

Le tableau ci-dessous montre tous les opérateurs valides pour `Find Blobs by Tags` :

|  Opérateur  |  Description  | Exemple |
|------------|---------------|---------|
|     =      |     Égal à     | `"Status" = 'In Progress'` |
|     >      |  Supérieur à | `"Date" > '2018-06-18'` |
|     >=     |  Supérieur ou égal à | `"Priority" >= '5'` |
|     <      |  Inférieur à   | `"Age" < '32'` |
|     <=     |  Inférieur ou égal à  | `"Company" <= 'Contoso'` |
|    AND     |  ET logique  | `"Rank" >= '010' AND "Rank" < '100'` |
| @container | Limiter l’étendue à un conteneur spécifique | `@container = 'videofiles' AND "status" = 'done'` |

> [!NOTE]
> Familiarisez-vous avec l'ordre lexicographique lors de la définition et de l'interrogation des étiquettes.
>
> - Les nombres sont triés avant les lettres. Les nombres sont triés sur la base du premier chiffre.
> - Les lettres majuscules sont triées avant les lettres minuscules.
> - Les symboles ne sont pas standard. Certains symboles sont triés avant les valeurs numériques. Les autres symboles sont triés avant ou après les lettres.

## <a name="conditional-blob-operations-with-blob-index-tags"></a>Opérations d’objet blob conditionnelles avec étiquettes d’index d’objet blob

Dans les versions REST 2019-10-10 et ultérieures, la plupart des [API de service blob](/rest/api/storageservices/operations-on-blobs) prennent désormais en charge un en-tête conditionnel, `x-ms-if-tags`, de sorte que l’opération réussira uniquement si la condition d’index de blob spécifiée est remplie. Si la condition n’est pas remplie, vous obtiendrez `error 412: The condition specified using HTTP conditional header(s) is not met`.

L’en-tête `x-ms-if-tags` peut être combiné avec les autres en-têtes conditionnels HTTP existants (If-Match, If-None-Match, etc.). Si plusieurs en-têtes conditionnels sont fournis dans une demande, ils doivent tous équivaloir à true pour que l’opération réussisse. Tous les en-têtes conditionnels sont combinés de manière efficace avec un ET logique.

Le tableau ci-dessous montre les opérateurs valides pour les opérations conditionnelles :

|  Opérateur  |  Description  | Exemple |
|------------|---------------|---------|
|     =      |     Égal à     | `"Status" = 'In Progress'` |
|     <>     |   Différent de   | `"Status" <> 'Done'` |
|     >      |  Supérieur à | `"Date" > '2018-06-18'` |
|     >=     |  Supérieur ou égal à | `"Priority" >= '5'` |
|     <      |  Inférieur à   | `"Age" < '32'` |
|     <=     |  Inférieur ou égal à  | `"Company" <= 'Contoso'` |
|    AND     |  ET logique  | `"Rank" >= '010' AND "Rank" < '100'` |
|     OR     | OU logique   | `"Status" = 'Done' OR "Priority" >= '05'` |

> [!NOTE]
> Il existe deux opérateurs supplémentaires, « Différent de » et « OU logique », qui sont autorisés dans l’en-tête `x-ms-if-tags` conditionnel pour les opération de blob, mais qui n’existent pas dans l’opération `Find Blobs by Tags`.

## <a name="platform-integrations-with-blob-index-tags"></a>Intégrations de plateforme avec des étiquettes d’index d’objet blob

Les balises d’index de blob vous permettent non seulement de classer, de gérer et de rechercher vos données blob, mais également d’intégrer d’autres fonctionnalités de Stockage Blob, telles que la [gestion du cycle de vie](storage-lifecycle-management-concepts.md).

### <a name="lifecycle-management"></a>Gestion du cycle de vie

L’utilisation de `blobIndexMatch` en tant que filtre de règle dans la gestion du cycle de vie vous permet de déplacer les données vers des niveaux plus froids ou de supprimer des données en fonction des balises d’index appliquées à vos blobs. Vous pouvez être plus précis dans vos règles et déplacer ou supprimer uniquement les blobs qui correspondent aux critères d’étiquette spécifiés.

Vous pouvez définir une correspondance d’index d’objet blob comme filtre autonome défini dans une règle de cycle de vie pour appliquer des actions aux données avec étiquettes. Vous pouvez également combiner un préfixe et un index de blob pour établir une correspondance avec des jeux de données plus spécifiques. La spécification de plusieurs filtres dans une règle de cycle de vie applique une opération « ET logique ». L’action s’appliquera uniquement si *tous* les critères de filtrage correspondent.

L’exemple de règle de gestion du cycle de vie suivant s’applique aux objets blob de blocs dans un conteneur appelé *videofiles*. La règle hiérarchise les blobs pour archiver le stockage uniquement si les données correspondent aux critères de balise d’index de blob `"Status" == 'Processed' AND "Source" == 'RAW'`.

# <a name="portal"></a>[Portail](#tab/azure-portal)

![Exemple de règle de correspondance d’index d’objet blob pour la gestion du cycle de vie dans le portail Azure](media/storage-blob-index-concepts/blob-index-lifecycle-management-example.png)

# <a name="json"></a>[JSON](#tab/json)

```json
{
    "rules": [
        {
            "enabled": true,
            "name": "ArchiveProcessedSourceVideos",
            "type": "Lifecycle",
            "definition": {
                "actions": {
                    "baseBlob": {
                        "tierToArchive": {
                            "daysAfterModificationGreaterThan": 0
                        }
                    }
                },
                "filters": {
                    "blobIndexMatch": [
                        {
                            "name": "Status",
                            "op": "==",
                            "value": "Processed"
                        },
                        {
                            "name": "Source",
                            "op": "==",
                            "value": "RAW"
                        }
                    ],
                    "blobTypes": [
                        "blockBlob"
                    ],
                    "prefixMatch": [
                        "videofiles/"
                    ]
                }
            }
        }
    ]
}
```

---

## <a name="permissions-and-authorization"></a>Autorisation et permissions

Vous pouvez autoriser l’accès aux balises d’index de blob à l’aide de l’une des approches suivantes :

- En utilisant le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour accorder des autorisations à un principal de sécurité Azure Active Directory (Azure AD). Utilisez Azure AD pour une meilleure sécurité et une plus grande facilité d’utilisation. Pour plus d’informations sur l’utilisation d’Azure AD avec les opérations d’objet blob, consultez [Autoriser l’accès aux objets blob et aux files d’attente avec Azure Active Directory](../common/storage-auth-aad.md).
- En utilisant une signature d’accès partagé (SAP) pour déléguer l’accès à l’index de blob. Pour plus d’informations sur les signatures d’accès partagé, consultez [Accorder un accès limité aux ressources du Stockage Azure à l’aide des signatures d’accès partagé (SAP)](../common/storage-sas-overview.md).
- En utilisant les clés d’accès au compte pour autoriser les opérations avec une clé partagée. Pour plus d’informations, consultez [Autoriser avec une clé partagée](/rest/api/storageservices/authorize-with-shared-key).

Les balises d’index de blob sont une sous-ressource des données blob. Un utilisateur disposant d’autorisations ou d’un jeton SAS pour lire ou écrire des objets blob peut ne pas avoir accès aux étiquettes d’index d’objet blob.

### <a name="role-based-access-control"></a>Contrôle d’accès en fonction du rôle

Les appelants utilisant une [identité Azure AD](../common/storage-auth-aad.md) peuvent se voir accorder les autorisations suivantes pour effectuer des opérations sur les étiquettes d’index d’objet blob.

| Opérations sur les balises d’index de blob                                          | Action Azure RBAC                                                             |
|--------------------------------------------------------------------|-------------------------------------------------------------------------------|
| [Définir des étiquettes d’objet blob](/rest/api/storageservices/set-blob-tags)           | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/write    |
| [Obtenir les étiquettes d’objet blob](/rest/api/storageservices/get-blob-tags)           | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/tags/read     |
| [Rechercher des objets blob par étiquettes](/rest/api/storageservices/find-blobs-by-tags) | Microsoft.Storage/storageAccounts/blobServices/containers/blobs/filter/action |

Des autorisations supplémentaires, distinctes des données blob sous-jacentes, sont requises pour effectuer des opérations sur les balises d’index. Le rôle [Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) dispose des autorisations pour les trois opérations de balises d’index de blob. Le [lecteur des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader) reçoit uniquement des autorisations pour les opérations `Find Blobs by Tags` et `Get Blob Tags`.

### <a name="sas-permissions"></a>Autorisations SAS

Les appelants utilisant une [signature d’accès partagé (SAP)](../common/storage-sas-overview.md) peuvent se voir accorder des autorisations délimitées pour effectuer des opérations sur les balises d’index de blob.

#### <a name="blob-sas"></a>Signature d’accès partagé d’objet blob

Les autorisations suivantes peuvent être accordées dans une SAP de blob pour permettre l’accès aux balises d’index de blob. Les autorisations de lecture et d’écriture de blob seules ne sont pas suffisantes pour permettre la lecture ou l’écriture de ses balises d’index.

| Autorisation | Symbole d'URI | Opérations autorisées                |
|------------|------------|-----------------------------------|
| Étiquettes d’index |     t      | Obtenir et définir des balises d’index pour un blob |

#### <a name="container-sas"></a>SAP de conteneur

Les autorisations suivantes peuvent être accordées dans une SAP de conteneur pour permettre le filtrage sur les balises de blob. L’autorisation `Blob List` n’est pas suffisante pour permettre le filtrage des blobs par leurs balises d’index.

| Autorisation | Symbole d'URI | Opérations autorisées         |
|------------|------------|----------------------------|
| Étiquettes d’index |     f      | Rechercher des blobs avec des balises d’index |

## <a name="choosing-between-metadata-and-blob-index-tags"></a>Choisir entre les métadonnées et les étiquettes d’index d’objet blob

Les métadonnées et les balises d’index de blob permettent de stocker des propriétés clé-valeur arbitraires définies par l’utilisateur en même temps qu’une ressource de blob. Les deux peuvent être récupérés et définis directement, sans retourner ni modifier le contenu du blob. Il est possible d’utiliser à la fois des balises d’index et des métadonnées.

Seules les balises d’index sont indexées automatiquement et peuvent être interrogées par le service Stockage Blob natif. Les métadonnées ne peuvent pas être indexées ou recherchées en mode natif. Vous devez utiliser un service distinct, par exemple [Recherche Azure](../../search/search-blob-ai-integration.md). Les balises d’index de blob disposent d’autorisations supplémentaires pour la lecture, le filtrage et l’écriture qui sont distinctes des données blob sous-jacentes. Les métadonnées utilisent les mêmes autorisations que le blob et sont retournées sous forme d’en-têtes HTTP dans les opérations [Get Blob](/rest/api/storageservices/get-blob) et [Get Blob Properties](/rest/api/storageservices/get-blob-properties). Les balises d’index de blob sont chiffrées au repos à l’aide d’une [clé gérée par Microsoft](../common/storage-service-encryption.md). Les métadonnées sont chiffrées au repos à l’aide de la même clé de chiffrement spécifiée pour les données blob.

Le tableau suivant récapitule les différences entre les métadonnées et les étiquettes d’index d’objet blob :

|              |   Métadonnées   |   Étiquettes d’index d’objet blob  |
|--------------|--------------|--------------------|
| **Limites**      | Aucune limite numérique ; 8 ko au total ; ne respectent pas la casse | Dix balises par blob max. ; 768 octets par balise ; respectent la casse |
| **Mises à jour**    | Non autorisées sur le niveau Archive ; `Set Blob Metadata` remplace toutes les métadonnées existantes ; `Set Blob Metadata` modifie la propriété Last-Modified-Time du blob | Autorisées pour tous les niveaux d’accès ; `Set Blob Tags` remplace toutes les balises existantes ; `Set Blob Tags` ne modifie pas la propriété Last-Modified-Time du blob |
| **Stockage**     | Stockées avec les données blob | Sous-ressource des données blob |
| **Indexation et interrogation** | Doit utiliser un service distinct comme Recherche Azure | Capacités d’indexation et d’interrogation intégrées à Stockage Blob |
| **Chiffrement** | Chiffrées au repos avec la même clé de chiffrement que celle utilisée pour les données blob | Chiffrées au repos avec une clé de chiffrement gérée par Microsoft |
| **Tarification** | La taille des métadonnées est incluse dans les coûts de stockage d’un objet blob | Coût fixe par étiquette d’index |
| **Réponse d’en-tête** | Métadonnées retournées en tant qu’en-têtes dans `Get Blob` et `Get Blob Properties` | Nombre de balises retourné par `Get Blob` ou `Get Blob Properties` ; balises retournées uniquement par `Get Blob Tags` et `List Blobs` |
| **autorisations**  | Les autorisations de lecture ou d’écriture sur les données blob s’étendent aux métadonnées | Des autorisations supplémentaires sont requises pour lire, filtrer ou écrire des balises d’index |
| **Dénomination** | Les noms de métadonnées doivent respecter les règles d'affectation de noms des identificateurs C# | Les étiquettes d’index d’objet blob prennent en charge un plus grand nombre de caractères alphanumériques |

## <a name="pricing"></a>Tarifs

Les tarifs d’index de blob sont en préversion publique et sujets à modification en ce qui concerne la disponibilité générale. Vous êtes facturé pour le nombre moyen mensuel de balises d’index dans un compte de stockage. Aucun coût ne s’applique au moteur d’indexation. Les demandes adressées à `Set Blob Tags`, `Get Blob Tags` et `Find Blobs by Tags` sont facturées en fonction de leurs types d’opérations respectifs. Consultez [Tarification d’objet blob de blocs](https://azure.microsoft.com/pricing/details/storage/blobs/) pour en savoir plus.

## <a name="regional-availability-and-storage-account-support"></a>Disponibilité régionale et prise en charge des comptes de stockage

Les balises d’index de blob sont disponibles uniquement sur les comptes Usage général v2 (GPv2) sur lesquels l’espace de noms hiérarchique est désactivé. Les comptes Usage général v1 (GPv1) ne sont pas pris en charge, mais vous pouvez mettre à niveau n’importe quel compte GPv1 pour le transformer en compte GPv2.

Les balises d’index ne sont pas prises en charge sur les comptes de stockage Premium. Pour plus d’informations sur les comptes de stockage, consultez [Vue d’ensemble des comptes de stockage Azure](../common/storage-account-overview.md).

En préversion publique, les balises d’index de blob sont uniquement disponible dans les régions suivantes :

- Centre du Canada
- Est du Canada
- France Centre
- France Sud

Pour commencer, consultez [Utiliser des balises d’index de blob pour gérer et rechercher des données](storage-blob-index-how-to.md).

> [!IMPORTANT]
> Vous devez inscrire votre abonnement avant de pouvoir utiliser la préversion de l’index de blob sur vos comptes de stockage. Consultez la section [Conditions et problèmes connus](#conditions-and-known-issues) de cet article.

### <a name="register-your-subscription-preview"></a>Inscrire votre abonnement (préversion)

Étant donné que les balises d’index de blob sont uniquement disponibles en préversion publique, vous devez inscrire votre abonnement avant de pouvoir utiliser la fonctionnalité. Pour soumettre une requête, exécutez les commandes PowerShell ou CLI suivantes.

#### <a name="register-by-using-powershell"></a>S’inscrire à l’aide de PowerShell

```powershell
Register-AzProviderFeature -FeatureName BlobIndex -ProviderNamespace Microsoft.Storage
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

#### <a name="register-by-using-azure-cli"></a>S’inscrire à l’aide d’Azure CLI

```azurecli
az feature register --namespace Microsoft.Storage --name BlobIndex
az provider register --namespace 'Microsoft.Storage'
```

## <a name="conditions-and-known-issues"></a>Conditions et problèmes connus

Cette section décrit les problèmes connus et les conditions de la préversion publique des balises d’index de blob. Cette fonctionnalité ne doit pas être utilisée pour les charges de travail de production avant d’avoir atteint la disponibilité générale (GA), car le comportement peut changer.

- Pour la préversion, vous devez d’abord inscrire votre abonnement avant de pouvoir utiliser l’index d’objet blob pour votre compte de stockage dans les régions en préversion.
- Seuls les comptes GPv2 sont pris en charge dans la préversion. Les comptes de blob, BlockBlobStorage et DataLake Gen2 avec HNS activé ne sont pas pris en charge. Les comptes GPv1 ne seront pas pris en charge.
- Le chargement d’objets blob de pages avec des balises d’index ne conserve pas les balises. Définissez les balises après le chargement d’un objet blob de pages.
- Lorsque le filtrage est limité à un conteneur individuel, il est possible de passer `@container` seulement si toutes les étiquettes d’index dans l’expression de filtre sont des contrôles d’égalité (clé=valeur).
- Lorsque vous utilisez l’opérateur de comparaison avec la condition `AND`, vous pouvez uniquement spécifier le même nom de clé de balise d’index (`"Age" > '013' AND "Age" < '100'`).
- Le contrôle de version et l’index de blob ne sont pas pris en charge. Les balises d’index de blob sont conservées pour les versions, mais ne sont pas transmises au moteur d’index de blob.
- Il n’existe aucune API permettant de déterminer si les balises d’index sont indexées.
- Le basculement de compte n’est pas pris en charge. L’index d’objet blob peut ne pas être mis à jour correctement après le basculement.
- La gestion du cycle de vie prend en charge uniquement les contrôles d’égalité avec correspondance d’index de blob.
- `Copy Blob` ne copie pas les balises d’index de blob du blob source dans le nouveau blob de destination. Vous pouvez spécifier les étiquettes que vous souhaitez appliquer à l’objet blob de destination pendant l’opération de copie.
- En cas d’utilisation de `Copy Blob` (copie asynchrone) à partir d’un autre compte de stockage avec des balises appliquées sur le blob de destination, le moteur d’index de blob ne retourne pas le blob ni ses balises dans le jeu de filtres. Utilisez `Copy Blob` à partir de l’URL (copie de synchronisation).
- Les balises sont conservées lors de la création d’instantanés. Toutefois, la promotion d’un instantané n’est pas prise en charge et peut aboutir à un jeu de balises vide.

## <a name="faq"></a>Questions fréquentes (FAQ)

**L’index d’objet blob peut-il m’aider à filtrer et à interroger le contenu de mes objets blob ?**

Non, si vous devez effectuer une recherche dans vos données blob, utilisez l’accélération des requêtes ou Recherche Azure.

**Existe-t-il des exigences concernant les valeurs de balise d’index ?**

Les étiquettes d'index d'objet blob ne prennent en charge que les types de données de chaîne, et l'interrogation renvoie les résultats dans l'ordre lexicographique. Pour les nombres, ajoutez un zéro au chiffre. Pour les dates et les heures, enregistrez-les dans un format conforme à la norme ISO 8601.

**Les étiquettes d’index d’objet blob et les étiquettes Azure Resource Manager sont-elles liées ?**

Non, les étiquettes Resource Manager aident à organiser les ressources de plan de contrôle telles que les abonnements, les groupes de ressources et les comptes de stockage. Les balises d’index fournissent la gestion et la découverte des blobs sur le plan de données.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir un exemple d’utilisation d’un index de blob, consultez [Utiliser un index de blob pour gérer et rechercher des données](storage-blob-index-how-to.md).

En savoir plus sur la [gestion de cycle de vie](storage-lifecycle-management-concepts.md) et définir une règle avec correspondance d’index de blob.