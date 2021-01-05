---
title: Gestion des données d’objets blob avec l’inventaire du Stockage Azure (préversion)
description: L’inventaire du Stockage Azure est un outil qui permet d’obtenir une vue d’ensemble de toutes les données blob d’un compte de stockage.
services: storage
author: mhopkins-msft
ms.service: storage
ms.date: 12/03/2020
ms.topic: conceptual
ms.author: mhopkins
ms.reviewer: yzheng
ms.subservice: blobs
ms.custom: references_regions
ms.openlocfilehash: 86ded3dea819702631b1fa04dbc56f727566fc98
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602680"
---
# <a name="use-azure-storage-blob-inventory-to-manage-blob-data-preview"></a>Gestion des données d’objets blob avec l’inventaire des objets blob du Stockage Azure (préversion)

La fonctionnalité d’inventaire des objets blob du Stockage Azure offre une vue d’ensemble des données blob d’un compte de stockage. Utilisez le rapport d’inventaire pour connaître la taille totale des données, l’âge, l’état du chiffrement, etc. Il vous fournit une synthèse de vos données pour les besoins de l’entreprise et ses exigences de conformité. Après activation, un rapport d’inventaire est créé automatiquement tous les jours.

## <a name="availability"></a>Disponibilité

L’inventaire d’objets blob est pris en charge pour les comptes de stockage universels version 2 (GPv2) et les comptes de stockage d’objets blobs de blocs Premium. Cette fonctionnalité est prise en charge avec ou sans la fonctionnalité d’[espace de noms hiérarchique](data-lake-storage-namespace.md) activée.

### <a name="preview-regions"></a>Régions de la préversion

La préversion de l’inventaire des objets blob est disponible dans les comptes de stockage des régions suivantes :

- France Centre
- Centre du Canada
- Est du Canada

### <a name="pricing-and-billing"></a>Tarification et facturation

Les rapports d’inventaire ne génèrent pas de frais pendant la période de préversion. La tarification sera déterminée une fois cette fonctionnalité en disponibilité générale.

## <a name="enable-inventory-reports"></a>Activation des rapports d’inventaire

Activez les rapports d’inventaire des objets blob en ajoutant une stratégie à votre compte de stockage. Ajoutez, modifiez ou supprimez une stratégie à l’aide du [Portail Azure](https://portal.azure.com/).

1. Accédez au [portail Azure](https://portal.azure.com/).
1. Sélectionnez l’un de vos comptes de stockage.
1. Sous **Service blob**, sélectionnez **Inventaire des objets blob**.
1. Vérifiez que l’option **Inventaire des objets blob activé** est sélectionnée.
1. Sélectionnez **Ajouter une règle**.
1. Donnez un nom à votre nouvelle règle.
1. Sélectionner les **Types d’objets blob** de votre rapport d’inventaire.
1. Ajouter une correspondance de préfixe pour filtrer votre rapport d’inventaire.
1. Indiquez si vous souhaitez **Inclure les versions d’objets blob** et **Inclure les instantanés** dans votre rapport d’inventaire. Vous devez activer les versions et les instantanés sur le compte pour pouvoir enregistrer une nouvelle règle avec l’option correspondante activée.
1. Sélectionnez **Enregistrer**.

:::image type="content" source="./media/blob-inventory/portal-blob-inventory.png" alt-text="Capture d’écran montrant comment ajouter une règle d’inventaire des objets blob à l’aide du Portail Azure":::

Les stratégies d’inventaire sont lues ou écrites en entier. Les mises à jour partielles ne sont pas prises en charge.

> [!IMPORTANT]
> Si vous activez les règles de pare-feu sur votre compte de stockage, les demandes d’inventaire risquent d’être bloquées. Vous pouvez débloquer ces requêtes en fournissant des exceptions pour les services Microsoft approuvés. Pour plus d’informations, consultez la section Exceptions dans [Configurer des pare-feu et des réseaux virtuels](../common/storage-network-security.md#exceptions).

L’exécution d’un inventaire des objets blob est planifiée automatiquement tous les jours. Elle peut prendre jusqu’à 24 heures. La configuration d’un rapport d’inventaire s’effectue en ajoutant une stratégie d’inventaire avec une ou plusieurs règles.

## <a name="inventory-policy"></a>Stratégie d’inventaire

Une stratégie d’inventaire est un ensemble de règles dans un document JSON.

```json
{
    "destination": "destinationContainer",
    "enabled": true,
    "rules": [
    {
        "enabled": true,
        "name": "inventoryrule1",
        "definition": {. . .}
    },
    {
        "enabled": true,
        "name": "inventoryrule2",
        "definition": {. . .}
    }]
}
```

Pour afficher le code JSON d’une stratégie d’inventaire, sélectionnez l’onglet **Mode Code** dans la section **Inventaire des objets blob** du Portail Azure.

| Nom du paramètre | Type de paramètre        | Notes | Requis ? |
|----------------|-----------------------|-------|-----------|
| destination    | String                | Conteneur de destination dans lequel tous les fichiers d’inventaire seront générés. Il doit déjà exister. | Yes |
| enabled        | Booléen               | Paramètre permettant de désactiver l’intégralité de la stratégie. Lorsqu’il a la valeur **true**, le champ de niveau règle activé remplace ce paramètre. Quand il est désactivé, l’inventaire est désactivé pour toutes les règles. | Yes |
| rules          | Tableau d’objets de règle | Une stratégie requiert au moins une règle. Le nombre de règles prises en charge est de 10 maximum. | Yes |

## <a name="inventory-rules"></a>Règles d’inventaire

Une règle capture les conditions de filtrage et les paramètres de sortie associés à la génération d’un rapport d’inventaire. Chaque règle crée un rapport d’inventaire. Les règles peuvent comporter des préfixes qui se chevauchent. Il peut arriver qu’un objet blob apparaisse dans plusieurs inventaires en fonction des définitions de règles.

Chaque règle au sein de la stratégie a plusieurs paramètres :

| Nom du paramètre | Type de paramètre                 | Notes | Requis ? |
|----------------|--------------------------------|-------|-----------|
| name           | String                         | Un nom de règle peut comporter jusqu’à 256 caractères alphanumériques sensibles à la casse. Il doit être unique dans une stratégie. | Yes |
| enabled        | Booléen                        | Indicateur permettant l’activation et la désactivation d’une règle. La valeur par défaut est **true**. | Yes |
| Définition     | Définition de règle d’inventaire JSON | Chaque définition se compose d’un ensemble de filtres de règle. | Yes |

L’indicateur global **Inventaire des objets blob activé** est prioritaire sur le paramètre *activé* dans une règle.

### <a name="rule-filters"></a>Filtres de règle

Plusieurs filtres sont disponibles pour personnaliser un rapport d’inventaire d’objets blob :

| Nom du filtre         | Type de filtre                     | Notes | Requis ? |
|---------------------|---------------------------------|-------|-----------|
| blobTypes           | Tableau de valeurs enum prédéfinies | Les valeurs valides sont `blockBlob` et `appendBlob` pour les comptes avec espace de noms hiérarchique, et `blockBlob`, `appendBlob` et `pageBlob` pour les autres. | Oui |
| prefixMatch         | Tableau pouvant comporter jusqu’à 10 chaînes pour les préfixes à mettre en correspondance (qui doivent commencer par un nom de conteneur, par exemple « conteneur1/foo ») | Si vous ne définissez pas *prefixMatch* ou que vous indiquez un préfixe vide, la règle s’applique à tous les objets blob du compte de stockage. | No |
| includeSnapshots    | Booléen                         | Spécifie si l’inventaire doit inclure les instantanés. La valeur par défaut est **false**. | No |
| includeBlobVersions | Booléen                         | Spécifie si l’inventaire doit inclure les versions d’objets blob. La valeur par défaut est **false**. | No |

Pour afficher le code JSON de règles d’inventaire, sélectionnez l’onglet **Mode Code** dans la section **Inventaire des objets blob** du Portail Azure. Les filtres sont spécifiés au sein d’une définition de règle.

```json
{
    "destination": "destinationContainer",
    "enabled": true,
    "rules": [
    {
        "enabled": true,
        "name": "inventoryrule1",
        "definition":
        {
            "filters":
            {
                "blobTypes": ["blockBlob", "appendBlob", "pageBlob"],
                "prefixMatch": ["inventorycontainer1", "inventorycontainer2/abcd", "etc"]
            }
        }
    },
    {
        "enabled": true,
        "name": "inventoryrule2",
        "definition":
        {
            "filters":
            {
                "blobTypes": ["pageBlob"],
                "prefixMatch": ["inventorycontainer-disks-", "inventorycontainer4/"],
                "includeSnapshots": true,
                "includeBlobVersions": true
            }
        }
    }]
}
```

## <a name="inventory-output"></a>Sortie de l’inventaire

Chaque exécution d’inventaire génère un ensemble de fichiers au format CSV dans le conteneur de destination des inventaires spécifié. La sortie d’inventaire est générée dans le chemin `https://<accountName>.blob.core.windows.net/<inventory-destination-container>/YYYY/MM/DD/HH-MM-SS/`, où :

- *accountName* est le nom du compte Stockage Blob Azure ;
- *inventory-destination-container* est le conteneur de destination que vous avez spécifié dans la stratégie d’inventaire ;
- *YYYY/MM/DD/HH-MM-SS* est l’heure à laquelle l’inventaire a commencé à s’exécuter.

### <a name="inventory-files"></a>Fichiers d’inventaire

Chaque exécution d’inventaire génère les fichiers suivants :

- **Fichier CSV d’inventaire** : fichier de valeurs séparées par des virgules (CSV) pour chaque règle d’inventaire. Chacun contient les objets mis en correspondance et leurs métadonnées. La première ligne de chaque fichier est toujours la ligne de schéma. Voici un fichier CSV d’inventaire ouvert dans Microsoft Excel.

   :::image type="content" source="./media/blob-inventory/csv-file-excel.png" alt-text="Capture d’écran d’un fichier CSV d’inventaire ouvert dans Microsoft Excel":::

- **Fichier manifeste** : fichier manifest.json contenant les détails des fichiers d’inventaire générés pour chaque règle de cette exécution. Il capture également la définition de règle fournie par l’utilisateur et le chemin de l’inventaire de cette règle.

- **Fichier de somme de contrôle** : fichier manifest.checksum contenant la somme de contrôle MD5 du contenu du fichier manifest.json. La génération du fichier manifest.checksum marque la fin de l’exécution d’un inventaire.

## <a name="inventory-completed-event"></a>Événement d’inventaire terminé

Abonnez-vous à l’événement d’inventaire terminé pour recevoir une notification à la fin de l’exécution de l’inventaire. Cet événement est généré lors de la création du fichier manifest.checksum. L’événement d’inventaire terminé se produit également si l’exécution de l’inventaire rencontre une erreur utilisateur avant de commencer (par exemple une erreur de type Stratégie non valide ou Conteneur de destination manquant). L’événement est publié dans la rubrique d’inventaire des objets blob.

Exemple d’événement :

```json
{
  "topic": "/subscriptions/3000151d-7a84-4120-b71c-336feab0b0f0/resourceGroups/BlobInventory/providers/Microsoft.EventGrid/topics/BlobInventoryTopic",
  "subject": "BlobDataManagement/BlobInventory",
  "eventType": "Microsoft.Storage.BlobInventoryPolicyCompleted",
  "id": "c99f7962-ef9d-403e-9522-dbe7443667fe",
  "data": {
    "scheduleDateTime": "2020-10-13T15:37:33Z",
    "accountName": "inventoryaccountname",
    "policyRunStatus": "Succeeded",
    "policyRunStatusMessage": "Inventory run succeeded, refer manifest file for inventory details.",
    "policyRunId": "b5e1d4cc-ee23-4ed5-b039-897376a84f79",
    "manifestBlobUrl": "https://inventoryaccountname.blob.core.windows.net/inventory-destination-container/2020/10/13/15-37-33/manifest.json"
  },
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-10-13T15:47:54Z"
}
```

## <a name="next-steps"></a>Étapes suivantes

[Gérer le cycle de vie de Stockage Blob Azure](storage-lifecycle-management-concepts.md)
