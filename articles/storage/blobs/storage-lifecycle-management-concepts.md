---
title: Optimiser les coûts en automatisant les niveaux d’accès au stockage Blob Azure
description: Créez des règles automatisées pour déplacer des données entre les niveaux chaud, froid et archive.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 10/29/2020
ms.service: storage
ms.subservice: common
ms.topic: conceptual
ms.reviewer: yzheng
ms.custom: devx-track-azurepowershell, references_regions
ms.openlocfilehash: 1b568687ffe646a91544c1bb75d26d552a23f49c
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96005280"
---
# <a name="optimize-costs-by-automating-azure-blob-storage-access-tiers"></a>Optimiser les coûts en automatisant les niveaux d’accès au stockage Blob Azure

Les jeux de données ont des cycles de vie différents. Tôt dans le cycle de vie, les utilisateurs accèdent souvent à certaines données. Mais à mesure que les données vieillissent, le nombre d’accès diminue considérablement. Certaines données restent inactives dans le cloud et sont rarement sollicitées une fois stockées. Certaines expirent plusieurs jours ou mois après leur création, tandis que d’autres jeux de données sont lus et modifiés de manière active tout au long de leur vie. La gestion du cycle de vie du stockage Blob Azure offre une stratégie complète basée sur des règles pour les comptes de stockage Blob et GPv2. Utilisez la stratégie pour effectuer la transition de vos données vers les niveaux d’accès appropriés ou pour faire en sorte qu’elles expirent à la fin de leur cycle de vie.

La gestion du cycle de vie vous permet de :

- Transférer des objets blob du niveau froid au niveau chaud dès qu’ils sont consultés pour optimiser les performances 
- Transférer les objets blob, les versions d’objets blob et les instantanés d’objets blob vers un niveau de stockage plus froid (des niveaux chaud à froid, chaud à archive ou froid à archive) s’ils ne sont pas consultés ni modifiés pendant une certaine période afin d’optimiser le coût
- Supprimer les objets blob, les versions d’objets blob et les instantanés d’objets blob à la fin de leur cycle de vie
- Définir des règles à exécuter une fois par jour au niveau du compte de stockage.
- Appliquer des règles à des conteneurs ou à un sous-ensemble d’objets blob (en utilisant des préfixes de noms ou des [balises d’index d’objets blob](storage-manage-find-blobs.md) en tant que filtres)

Considérez un scénario où des données sont sollicitées fréquemment durant les premières étapes du cycle de vie, mais seulement occasionnellement après deux semaines. Au-delà du premier mois, le jeu de données est rarement sollicité. Dans ce scénario, le stockage chaud est préférable durant les premiers temps. Un stockage froid est plus approprié pour un accès occasionnel. L’option Stockage archive est la meilleure une fois que les données ont plus d’un mois. En ajustant les niveaux de stockage en fonction de l’ancienneté des données, vous pouvez concevoir les options de stockage les moins coûteuses par rapport à vos besoins. Pour effectuer cette transition, les règles de stratégie de gestion du cycle de vie permettent de déplacer les données vieillissantes vers des niveaux plus froids.

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

>[!NOTE]
>Si vous avez besoin que les données restent lisibles, par exemple, quand elles sont utilisées par StorSimple, ne définissez pas de stratégie pour déplacer des objets blob vers le niveau Archive.

## <a name="availability-and-pricing"></a>Disponibilité et tarification

La fonctionnalité de gestion du cycle de vie est disponible dans toutes les régions Azure pour les comptes v2 universels (GPv2), les comptes de stockage Blob, les comptes de stockage Blob de blocs Premium et les comptes Azure Data Lake Storage Gen2. Dans le portail Azure, vous pouvez mettre à niveau un compte de stockage universel (GPv1) existant en compte GPv2. Pour plus d’informations sur les comptes de stockage, consultez [Vue d’ensemble des comptes de stockage Azure](../common/storage-account-overview.md).

La fonctionnalité de gestion du cycle de vie est gratuite. Les clients sont facturés au coût de fonctionnement normal pour les appels d’API [Définir le niveau d’objet blob](/rest/api/storageservices/set-blob-tier). L’opération de suppression est gratuite. Pour plus d’informations sur les prix, consultez [Tarification Objets blob de blocs](https://azure.microsoft.com/pricing/details/storage/blobs/).

## <a name="add-or-remove-a-policy"></a>Ajouter ou supprimer une stratégie

Vous pouvez ajouter, modifier ou supprimer une stratégie à l’aide de l’une des méthodes suivantes :

* [Azure portal](https://portal.azure.com)
* [Azure PowerShell](https://github.com/Azure/azure-powershell/releases)
* [Azure CLI](/cli/azure/install-azure-cli)
* [API REST](/rest/api/storagerp/managementpolicies)

Une stratégie peut être lue ou écrite dans son intégralité. Les mises à jour partielles ne sont pas prises en charge. 

> [!NOTE]
> Si vous activez les règles de pare-feu de votre compte de stockage, les requêtes de gestion du cycle de vie peuvent être bloquées. Vous pouvez débloquer ces requêtes en fournissant des exceptions pour les services Microsoft approuvés. Pour plus d’informations, consultez la section Exceptions dans [Configurer des pare-feu et des réseaux virtuels](../common/storage-network-security.md#exceptions).

Cet article explique comment gérer une stratégie en utilisant le portail et des méthodes PowerShell.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Il existe deux façons d’ajouter une stratégie à l’aide du Portail Microsoft Azure. 

* [Mode Liste du Portail Microsoft Azure](#azure-portal-list-view)
* [Mode Code du Portail Microsoft Azure](#azure-portal-code-view)

#### <a name="azure-portal-list-view"></a>Mode Liste du Portail Microsoft Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans le Portail Azure, recherchez et sélectionnez votre compte de stockage. 

1. Sous **Service Blob**, sélectionnez **Gestion du cycle de vie** pour afficher ou modifier vos règles.

1. Sélectionnez l’onglet **Mode Liste**.

1. Sélectionnez **Ajouter une règle** et nommez votre règle dans le formulaire **Détails**. Vous pouvez également définir les l’**Étendue de la règle**, le **Type d’objet blob** et le **Sous-type d’objet blob**. L’exemple suivant définit l’étendue pour filtrer les objets blob. Cela entraîne l’ajout de l’onglet **Jeu de filtres**.

   :::image type="content" source="media/storage-lifecycle-management-concepts/lifecycle-management-details.png" alt-text="Gestion du cycle de vie - Ajouter une page Détails de la règle dans le Portail Azure":::

1. Sélectionnez **Objets blob de base** pour définir les conditions de votre règle. Dans l’exemple suivant, les objets BLOB sont déplacés vers le stockage froid s’ils n’ont pas été modifiés depuis 30 jours.

   :::image type="content" source="media/storage-lifecycle-management-concepts/lifecycle-management-base-blobs.png" alt-text="Page Gestion de cycle de vie des objets blob de base du Portail Microsoft Azure":::

   L’option **Dernier accès** est disponible en préversion dans les régions suivantes :

    - France Centre
    - Est du Canada
    - Centre du Canada

   > [!IMPORTANT]
   > La dernière préversion du suivi de l’heure d’accès concerne uniquement l’utilisation en dehors de la production. Les contrats SLA (contrats de niveau de service) de production ne sont actuellement pas disponibles.
   
   Pour utiliser l’option **Dernier accès**, sélectionnez **Duivi d’accès activé** sur la page **Gestion du cycle de vie** du Portail Azure. Pour plus d’informations sur l’option **Dernier accès**, consultez [Déplacer des données en fonction de la date du dernier accès (préversion)](#move-data-based-on-last-accessed-date-preview).

1. Si vous avez sélectionné **Limiter les objets blob avec des filtres** dans la page **Détails**, sélectionnez **Jeu de filtres** pour ajouter un filtre facultatif. L’exemple suivant filtre sur les objets blob dans le conteneur *mylifecyclecontainer* qui commencent par « log ».

   :::image type="content" source="media/storage-lifecycle-management-concepts/lifecycle-management-filter-set.png" alt-text="Page Lifecycle management action set (Jeu de filtres de gestion du cycle de vie) du Portail Microsoft Azure":::

1. Sélectionnez **Ajouter** pour ajouter la stratégie.

#### <a name="azure-portal-code-view"></a>Mode Code du Portail Microsoft Azure
1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans le Portail Azure, recherchez et sélectionnez votre compte de stockage.

1. Sous **Service Blob**, sélectionnez **Gestion du cycle de vie** pour afficher ou modifier votre stratégie.

1. Le code JSON suivant est un exemple de stratégie que vous pouvez coller dans l’onglet **Mode Code**.

   ```json
   {
     "rules": [
       {
         "enabled": true,
         "name": "move-to-cool",
         "type": "Lifecycle",
         "definition": {
           "actions": {
             "baseBlob": {
               "tierToCool": {
                 "daysAfterModificationGreaterThan": 30
               }
             }
           },
           "filters": {
             "blobTypes": [
               "blockBlob"
             ],
             "prefixMatch": [
               "mylifecyclecontainer/log"
             ]
           }
         }
       }
     ]
   }
   ```

1. Sélectionnez **Enregistrer**.

1. Pour plus d’informations sur cet exemple de JSON, voir les sections [Stratégie](#policy) et [Règles](#rules).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Le script PowerShell suivant permet d’ajouter une stratégie à votre compte de stockage. La variable `$rgname` doit être initialisée avec le nom de votre groupe de ressources. La variable `$accountName` doit être initialisée avec le nom de votre compte de stockage.

```powershell
#Install the latest module
Install-Module -Name Az -Repository PSGallery

#Initialize the following with your resource group and storage account names
$rgname = ""
$accountName = ""

#Create a new action object
$action = Add-AzStorageAccountManagementPolicyAction -BaseBlobAction Delete -daysAfterModificationGreaterThan 2555
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -BaseBlobAction TierToArchive -daysAfterModificationGreaterThan 90
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -BaseBlobAction TierToCool -daysAfterModificationGreaterThan 30
$action = Add-AzStorageAccountManagementPolicyAction -InputObject $action -SnapshotAction Delete -daysAfterCreationGreaterThan 90

# Create a new filter object
# PowerShell automatically sets BlobType as “blockblob” because it is the only available option currently
$filter = New-AzStorageAccountManagementPolicyFilter -PrefixMatch ab,cd

#Create a new rule object
#PowerShell automatically sets Type as “Lifecycle” because it is the only available option currently
$rule1 = New-AzStorageAccountManagementPolicyRule -Name Test -Action $action -Filter $filter

#Set the policy
Set-AzStorageAccountManagementPolicy -ResourceGroupName $rgname -StorageAccountName $accountName -Rule $rule1
```

# <a name="template"></a>[Modèle](#tab/template)

Vous pouvez définir une gestion du cycle de vie à l’aide de modèles Azure Resource Manager. Voici un exemple de modèle pour déployer un compte de stockage RA-GRS GPv2 avec une stratégie de gestion du cycle de vie.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "variables": {
    "storageAccountName": "[uniqueString(resourceGroup().id)]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "name": "[variables('storageAccountName')]",
      "location": "[resourceGroup().location]",
      "apiVersion": "2019-04-01",
      "sku": {
        "name": "Standard_RAGRS"
      },
      "kind": "StorageV2",
      "properties": {
        "networkAcls": {}
      }
    },
    {
      "name": "[concat(variables('storageAccountName'), '/default')]",
      "type": "Microsoft.Storage/storageAccounts/managementPolicies",
      "apiVersion": "2019-04-01",
      "dependsOn": [
        "[variables('storageAccountName')]"
      ],
      "properties": {
        "policy": {...}
      }
    }
  ],
  "outputs": {}
}
```

---

## <a name="policy"></a>Stratégie

Une stratégie de gestion du cycle de vie est un ensemble de règles dans un document JSON :

```json
{
  "rules": [
    {
      "name": "rule1",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {...}
    },
    {
      "name": "rule2",
      "type": "Lifecycle",
      "definition": {...}
    }
  ]
}
```

Une stratégie est un ensemble de règles :

| Nom du paramètre | Type de paramètre | Notes |
|----------------|----------------|-------|
| `rules`        | Un ensemble d’objets de règle | Une stratégie requiert au moins une règle. Vous pouvez définir jusqu’à 100 règles dans une stratégie.|

Chaque règle au sein de la stratégie a plusieurs paramètres :

| Nom du paramètre | Type de paramètre | Notes | Obligatoire |
|----------------|----------------|-------|----------|
| `name`         | String |Un nom de règle peut compter jusqu’à 256 caractères alphanumériques. Les noms de règle respectent la casse. Ils doivent être uniques dans la stratégie. | True |
| `enabled`      | Boolean | Valeur booléenne facultative pour permettre la désactivation temporaire d’une règle. La valeur par défaut est true. | False | 
| `type`         | Une valeur enum | Le type valide actuel est `Lifecycle`. | True |
| `definition`   | Un objet qui définit la règle du cycle de vie | Chaque définition se compose d’un jeu de filtres et d’un jeu d’actions. | True |

## <a name="rules"></a>Règles

Chaque définition de règle se compose d’un jeu de filtres et d’un jeu d’actions. Le [jeu de filtres](#rule-filters) limite les actions de règle à un certain ensemble d’objets dans un conteneur ou des noms d’objets. L’[ensemble d’actions](#rule-actions) applique les actions de niveau ou de suppression à l’ensemble d’objets filtré.

### <a name="sample-rule"></a>Exemple de règle

L’exemple de règle suivant filtre le compte pour exécuter les actions sur des objets existant à l’intérieur de `container1` et commençant par `foo`.

>[!NOTE]
>- La gestion du cycle de vie prend en charge les types objets blob de bloc et d’ajout.<br>
>- La gestion du cycle de vie n’affecte pas les conteneurs système comme $logs et $web.

- Niveau objet blob sur accès froid 30 jours après la dernière modification
- Niveau objet blob sur accès archive 90 jours après la dernière modification
- Supprimer l’objet blob 2 555 jours (sept ans) après la dernière modification
- Supprimer les versions précédentes des objets blob 90 jours après leur création

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "rulefoo",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "version": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          },
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 2555
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "container1/foo"
          ]
        }
      }
    }
  ]
}
```

### <a name="rule-filters"></a>Filtres de règle

Les filtres limitent les actions des règles à un sous-ensemble d’objets blob dans le compte de stockage. Si plusieurs filtres sont définis, une opération logique `AND` est exécutée sur tous les filtres.

Les filtres sont les suivants :

| Nom du filtre | Type de filtre | Notes | Est obligatoire |
|-------------|-------------|-------|-------------|
| blobTypes   | Un ensemble de valeurs enum prédéfinies. | La version actuelle prend en charge `blockBlob` et `appendBlob`. Seule la suppression est prise en charge pour `appendBlob`, le niveau défini n’est pas pris en charge. | Oui |
| prefixMatch | Un ensemble de chaînes pour les préfixes à mettre en correspondance. Chaque règle peut définir jusqu’à 10 préfixes. Une chaîne de préfixe doit commencer par un nom de conteneur. Par exemple, si vous souhaitez faire correspondre tous les objets blob sous `https://myaccount.blob.core.windows.net/container1/foo/...` pour une règle, prefixMatch est `container1/foo`. | Si vous ne définissez pas prefixMatch, la règle s’applique à tous les objets blob au sein du compte de stockage. | Non |
| blobIndexMatch | Un ensemble de valeurs de dictionnaire constitué d’une clé de balise d’index d’objets blob et de conditions de valeur à mettre en correspondance. Chaque règle peut définir jusqu’à 10 conditions de balise d’index d’objets blob. Par exemple, si vous souhaitez mettre en correspondre tous les objets blob avec `Project = Contoso` sous `https://myaccount.blob.core.windows.net/` pour une règle, le blobIndexMatch est `{"name": "Project","op": "==","value": "Contoso"}`. | Si vous ne définissez pas blobIndexMatch, la règle s’applique à tous les objets blob au sein du compte de stockage. | Non |

> [!NOTE]
> L’index d’objets blob, actuellement en préversion publique, est disponible dans les régions **Canada Centre**, **Canada Est**, **France Centre** et **France Sud**. Pour en savoir plus sur cette fonctionnalité ainsi que sur les problèmes et limitations connus, consultez [Gérer et rechercher des données sur le Stockage Blob Azure avec un index d’objets blob (préversion)](storage-manage-find-blobs.md).

### <a name="rule-actions"></a>Actions de règle

Des actions sont appliquées aux objets blob filtrés lorsque la condition d’exécution est remplie.

La gestion du cycle de vie prend en charge la hiérarchisation et la suppression des objets blob, des versions d’objets blob précédentes et des instantanés d’objets blob. Définissez au moins une action pour chaque règle sur les objets blob de base, les versions d’objets blob précédentes ou les instantanés d’objets blob.

| Action                      | Objet blob de base                                  | Instantané      | Version
|-----------------------------|--------------------------------------------|---------------|---------------|
| tierToCool                  | Prise en charge pour `blockBlob`                  | Prise en charge     | Prise en charge     |
| enableAutoTierToHotFromCool | Prise en charge pour `blockBlob`                  | Non pris en charge | Non pris en charge |
| tierToArchive               | Prise en charge pour `blockBlob`                  | Prise en charge     | Prise en charge     |
| supprimer                      | Pris en charge pour `blockBlob` et `appendBlob` | Prise en charge     | Prise en charge     |

>[!NOTE]
>Si vous définissez plusieurs actions sur le même objet blob, la gestion du cycle de vie applique l’action la moins coûteuse à l’objet blob. Par exemple, l’action `delete` est moins coûteuse que l’action `tierToArchive`. L’action `tierToArchive` est moins coûteuse que l’action `tierToCool`.

Les conditions d’exécution sont basées sur l’âge. Les objets blob de base utilisent l’heure de dernière modification, les versions d’objets blob utilisent l’heure de création de la version et les instantanés d’objets blob utilisent l’heure de création des instantanés pour suivre l’ancienneté.

| Condition d’exécution d’action               | Valeur de la condition                          | Description                                                                      |
|------------------------------------|------------------------------------------|----------------------------------------------------------------------------------|
| daysAfterModificationGreaterThan   | Nombre entier indiquant l’âge en jours | Condition pour les actions des objets blob de base                                              |
| daysAfterCreationGreaterThan       | Nombre entier indiquant l’âge en jours | Condition pour les actions de versions d’objets blob et d’instantanés d’objets blob                         |
| daysAfterLastAccessTimeGreaterThan | Nombre entier indiquant l’âge en jours | (préversion) La condition pour les actions de base de l’objet blob lorsque l’heure du dernier accès est activée |

## <a name="examples"></a>Exemples

Les exemples suivants expliquent comment résoudre des scénarios courants avec les règles de stratégie du cycle de vie.

### <a name="move-aging-data-to-a-cooler-tier"></a>Déplacer les données vieillissantes vers un niveau plus froid

Cet exemple montre comment déplacer des objets blob de blocs ayant le préfixe `container1/foo` ou `container2/bar`. La stratégie déplace les objets blob qui n’ont pas été modifiés depuis plus de 30 jours vers le stockage froid et les objets blob non modifiés depuis 90 jours vers le niveau archive :

```json
{
  "rules": [
    {
      "name": "agingRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "container1/foo", "container2/bar" ]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": { "daysAfterModificationGreaterThan": 30 },
            "tierToArchive": { "daysAfterModificationGreaterThan": 90 }
          }
        }
      }
    }
  ]
}
```

### <a name="move-data-based-on-last-accessed-date-preview"></a>Déplacer les données en fonction de la date du dernier accès (préversion)

Vous pouvez activer le suivi de l’heure du dernier accès pour conserver un enregistrement de la dernière lecture ou écriture de votre objet blob. Vous pouvez utiliser l’heure de dernier accès comme filtre pour gérer la hiérarchisation et la rétention de vos données d’objet blob.

L’option **Dernier accès** est disponible en préversion dans les régions suivantes :

 - France Centre
 - Est du Canada
 - Centre du Canada

> [!IMPORTANT]
> La dernière préversion du suivi de l’heure d’accès concerne uniquement l’utilisation en dehors de la production. Les contrats SLA (contrats de niveau de service) de production ne sont actuellement pas disponibles.

Pour utiliser l’option **Dernier accès**, sélectionnez **Duivi d’accès activé** sur la page **Gestion du cycle de vie** du Portail Azure.

#### <a name="how-last-access-time-tracking-works"></a>Fonctionnement du suivi de l’heure du dernier accès

Lorsque le suivi de l’heure du dernier accès est activé, la propriété d’objet blob appelée `LastAccessTime` est mise à jour lors de la lecture ou de l’écriture d’un objet blob. Une opération [Obtenir un objet blob](/rest/api/storageservices/get-blob) est considérée comme une opération d’accès. [Obtenir les propriétés d’objets blob](/rest/api/storageservices/get-blob-properties), [Obtenir des métadonnées d’objets blob](/rest/api/storageservices/get-blob-metadata) et [Obtenir des étiquettes d’objet blob](/rest/api/storageservices/get-blob-tags) ne sont pas des opérations d’accès et ne mettent donc pas à jour l’heure du dernier accès.

Pour réduire l’impact sur la latence d’accès en lecture, seule la première lecture des dernières 24 heures met à jour l’heure du dernier accès. Les lectures suivantes dans la même période de 24 heures ne mettent pas à jour l’heure du dernier accès. Si un objet blob est modifié entre des lectures, l’heure du dernier accès est la plus récente des deux valeurs.

Dans l’exemple suivant, les objets BLOB sont déplacés vers le stockage froid s’ils n’ont pas fait l’objet d’accès depuis 30 jours. La propriété `enableAutoTierToHotFromCool` est une valeur booléenne qui indique si un objet blob doit être automatiquement hiérarchisé de froid à chaud s’il fait l’objet d’un accès à nouveau après avoir été hiérarchisé en froid.

```json
{
  "enabled": true,
  "name": "last-accessed-thirty-days-ago",
  "type": "Lifecycle",
  "definition": {
    "actions": {
      "baseBlob": {
        "enableAutoTierToHotFromCool": true,
        "tierToCool": {
          "daysAfterLastAccessTimeGreaterThan": 30
        }
      }
    },
    "filters": {
      "blobTypes": [
        "blockBlob"
      ],
      "prefixMatch": [
        "mylifecyclecontainer/log"
      ]
    }
  }
}
```

#### <a name="storage-account-support"></a>Prise en charge du compte de stockage

Le suivi de l’heure du dernier accès est disponible pour les types de comptes de stockage suivants :

 - Comptes de stockage universel v2
 - Comptes de stockage d’objets blob de blocs
 - Comptes de stockage d’objets blob

Si votre compte de stockage est un compte v1 à usage général, utilisez le Portail Azure pour effectuer une mise à niveau vers un compte v2 à usage général.

Les comptes de stockage avec espace de noms hiérarchique activé pour une utilisation avec Azure Data Lake Storage Gen2 sont désormais pris en charge.

#### <a name="pricing-and-billing"></a>Tarification et facturation

Chaque mise à jour de l’heure du dernier accès est considérée comme une [opération différente](https://azure.microsoft.com/pricing/details/storage/blobs/).

### <a name="archive-data-after-ingest"></a>Archiver les données après leur ingestion

D’autres sont inactives dans le cloud dès le départ et sont peu, voire pas sollicitées une fois stockées. La stratégie du cycle de vie suivante est configurée pour archiver les données peu après leur ingestion. Cet exemple déplace les objets blob de blocs du compte de stockage au sein du conteneur `archivecontainer` dans un niveau archive. Le déplacement est accompli en agissant sur les objets blob 0 jour après l’heure de dernière modification :

> [!NOTE] 
> Il est recommandé de charger vos objets BLOB directement au niveau Archive pour gagner en efficacité. Vous pouvez utiliser l’en-tête x-ms-access-tier pour [PutBlob](/rest/api/storageservices/put-blob) ou [PutBlockList](/rest/api/storageservices/put-block-list) avec REST 2018-11-09 et les versions ultérieures ou nos dernières bibliothèques clientes de stockage d’objets BLOB. 

```json
{
  "rules": [
    {
      "name": "archiveRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ],
          "prefixMatch": [ "archivecontainer" ]
        },
        "actions": {
          "baseBlob": {
              "tierToArchive": { "daysAfterModificationGreaterThan": 0 }
          }
        }
      }
    }
  ]
}

```

### <a name="expire-data-based-on-age"></a>Faire expirer les données selon l’âge

Certaines données sont supposées expirer un certain nombre de jours ou de mois après leur création. Vous pouvez configurer une stratégie de gestion du cycle de vie afin de faire expirer les données en les supprimant en fonction de leur ancienneté. L’exemple suivant présente une stratégie qui supprime tous les objets blob de bloc datant de plus de 365 jours.

```json
{
  "rules": [
    {
      "name": "expirationRule",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": [ "blockBlob" ]
        },
        "actions": {
          "baseBlob": {
            "delete": { "daysAfterModificationGreaterThan": 365 }
          }
        }
      }
    }
  ]
}
```

### <a name="delete-data-with-blob-index-tags"></a>Supprimer des données avec des balises d’index d’objets blob
Certaines données ne doivent expirer que si elles sont marquées explicitement pour suppression. Vous pouvez configurer une stratégie de gestion du cycle de vie pour faire expirer les données qui sont marquées avec des attributs clé/valeur d’index d’objets blob. L’exemple suivant présente une stratégie qui supprime tous les objets blob de bloc balisés avec `Project = Contoso`. Pour en savoir plus sur l’index d’objets blob, consultez [Gérer et rechercher des données sur le Stockage Blob Azure avec un index d’objets blob (préversion)](storage-manage-find-blobs.md).

```json
{
    "rules": [
        {
            "enabled": true,
            "name": "DeleteContosoData",
            "type": "Lifecycle",
            "definition": {
                "actions": {
                    "baseBlob": {
                        "delete": {
                            "daysAfterModificationGreaterThan": 0
                        }
                    }
                },
                "filters": {
                    "blobIndexMatch": [
                        {
                            "name": "Project",
                            "op": "==",
                            "value": "Contoso"
                        }
                    ],
                    "blobTypes": [
                        "blockBlob"
                    ]
                }
            }
        }
    ]
}
```

### <a name="manage-versions"></a>Gérer les versions

Pour les données modifiées et consultées régulièrement pendant toute leur durée de vie, vous pouvez activer la gestion de versions du stockage Blob afin de gérer automatiquement les versions précédentes d’un objet. Vous pouvez créer une stratégie pour hiérarchiser ou supprimer les versions précédentes. L’âge de la version est déterminé en regardant l’heure de création de cette dernière. Cette règle de stratégie hiérarchise les versions précédentes du conteneur `activedata` qui datent de 90 jours ou plus après la création de la version vers le niveau Froid, et supprime les versions antérieures datant de 365 jours ou plus.

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "versionrule",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "version": {
            "tierToCool": {
              "daysAfterCreationGreaterThan": 90
            },
            "delete": {
              "daysAfterCreationGreaterThan": 365
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "activedata"
          ]
        }
      }
    }
  ]
}
```

## <a name="faq"></a>Questions fréquentes (FAQ)

**J’ai créé une stratégie. Pourquoi les actions ne s’exécutent-elles pas immédiatement ?**

La plateforme exécute la stratégie de cycle de vie une fois par jour. Une fois que vous avez configuré une stratégie, jusqu’à 24 heures peuvent s’écouler avant que certaines actions s’exécutent pour la première fois.

**Si je mets à jour une stratégie existante, combien de temps dois-je attendre avant que les actions soient effectuées ?**

Cela peut prendre jusqu’à 24 heures avant que la stratégie mise à jour ne soit appliquée. Une fois la stratégie en vigueur, cela peut prendre jusqu’à 24 heures pour que les actions s’exécutent. Par conséquent, l’exécution des actions de la stratégie peut prendre jusqu’à 48 heures.

**J’ai réactivé manuellement un blob archivé. Comment puis-je empêcher son renvoi temporaire au niveau Archives ?**

Quand un objet blob est déplacé d’un niveau d’accès vers un autre, l’heure de sa dernière modification ne change pas. Si vous réactivez manuellement un blob archivé à un chaud, il est renvoyé au niveau archive par le moteur de gestion du cycle de vie. Désactivez la règle qui affecte temporairement cet objet BLOB pour empêcher son archivage. Réactivez la règle lorsque le BLOB peut être renvoyé en toute sécurité au niveau archive. Vous pouvez aussi copier le blob vers un autre emplacement s’il doit rester en permanence au niveau chaud ou froid.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment récupérer des données après une suppression accidentelle :

- [Suppression réversible pour les objets blob de Stockage Azure](./soft-delete-blob-overview.md)

Apprenez à gérer et à rechercher des données avec un index d’objets blob :

- [Gérer et rechercher des données dans le Stockage Blob Azure avec un index d’objets blob](storage-manage-find-blobs.md)