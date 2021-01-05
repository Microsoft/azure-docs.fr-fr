---
title: Surveiller les journaux de diagnostic de Media Services via Azure Monitor | Microsoft Docs
description: Cet article explique comment router et voir les journaux de diagnostic via Azure Monitor.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-azurecli
ms.openlocfilehash: e756ff0133e3d3f0f86ea2f592005f2f2986488e
ms.sourcegitcommit: 19dce034650c654b656f44aab44de0c7a8bd7efe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/04/2020
ms.locfileid: "91707759"
---
# <a name="monitor-media-services-diagnostic-logs"></a>Surveillez les journaux de diagnostic de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

[Azure Monitor](../../azure-monitor/overview.md) vous permet de surveiller les métriques et journaux de diagnostic pour vous aider à comprendre le fonctionnement de vos applications. Pour une description détaillée de cette fonctionnalité et pour comprendre pourquoi utiliser les mesures et les journaux de diagnostic Azure Media Services, consultez la section [Surveiller les mesures et les journaux de diagnostic Media Services](media-services-metrics-diagnostic-logs.md).

Cet article vous montre comment acheminer les données vers le compte de stockage, puis comment afficher les données.

## <a name="prerequisites"></a>Prérequis

- [Créer un compte Media Services](./create-account-howto.md).
- Évaluation [Surveiller les mesures et les journaux de diagnostic Media Services](media-services-metrics-diagnostic-logs.md).

## <a name="route-data-to-the-storage-account-using-the-portal"></a>Acheminez les données vers le compte de stockage à l’aide du portail

1. Connectez-vous au portail Azure sur https://portal.azure.com.
1. Accédez à votre compte Media Services dans et cliquez sur **Paramètres de diagnostic** sous **Surveiller**. Vous trouverez la liste de toutes les ressources dans votre abonnement, qui produisent des données de surveillance dans Azure Monitor.

    ![Capture d’écran qui met en évidence Paramètres de diagnostic dans la section Supervision.](media/media-services-diagnostic-logs/logs01.png)

1. Cliquez sur **Ajouter le paramètre de diagnostic**.

   Un paramètre de diagnostic de ressource définit *quelles* données de surveillance doivent être routées à partir d’une ressource particulière, et vers *où* ces données doivent être routées.

1. Dans la section qui s’affiche, attribuez un **nom** à votre paramètre, puis activez la case à cocher **Archiver dans un compte de stockage**.

    Sélectionnez le compte de stockage auquel vous souhaitez envoyer des journaux, puis appuyez sur **OK**.
1. Activez toutes les cases à cocher sous **Journal** et **Métrique**. Selon le type de ressource, il se peut qu’une seule de ces options soit disponible. Ces cases à cocher déterminent les catégories de données de journal et de métrique disponibles pour ce type de ressource, qui sont envoyées à la destination que vous avez sélectionnée, en l’occurrence, un compte de stockage.

   ![Section Paramètres de diagnostic](media/media-services-diagnostic-logs/logs02.png)
1. Positionnez le curseur **Rétention (jours)** sur 30. Ce curseur définit le nombre de jours de rétention des données de surveillance dans le compte de stockage. Azure Monitor supprime automatiquement les données antérieures au nombre de jours spécifié. Si la durée de la période de rétention est zéro, les données sont conservées pendant une durée indéfinie.
1. Cliquez sur **Enregistrer**.

Les données de surveillance de votre ressource sont maintenant transférées dans le compte de stockage.

## <a name="route-data-to-the-storage-account-using-the-azure-cli"></a>Acheminer les données vers le compte de stockage à l’aide d’Azure CLI

Pour activer le stockage des journaux de diagnostic dans un compte de stockage, exécutez la commande Azure CLI `az monitor diagnostic-settings` suivante :

```azurecli-interactive
az monitor diagnostic-settings create --name <diagnostic name> \
    --storage-account <name or ID of storage account> \
    --resource <target resource object ID> \
    --resource-group <storage account resource group> \
    --logs '[
    {
        "category": <category name>,
        "enabled": true,
        "retentionPolicy": {
            "days": <# days to retain>,
            "enabled": true
        }
    }]'
```

Par exemple :

```azurecli-interactive
az monitor diagnostic-settings create --name amsv3diagnostic \
    --storage-account storageaccountforams  \
    --resource "/subscriptions/00000000-0000-0000-0000-0000000000/resourceGroups/amsResourceGroup/providers/Microsoft.Media/mediaservices/amsaccount" \
    --resource-group "amsResourceGroup" \
    --logs '[{"category": "KeyDeliveryRequests",  "enabled": true, "retentionPolicy": {"days": 3, "enabled": true }}]'
```

## <a name="view-data-in-the-storage-account-using-the-portal"></a>Affichez les données dans le compte de stockage à l’aide du portail

Si vous avez suivi les étapes précédentes, les données ont commencé à circuler vers votre compte de stockage.

Il se peut que vous deviez attendre cinq minutes avant que l’événement apparaisse dans le compte de stockage.

1. Dans le portail, accédez à la section **Comptes de stockage** en la recherchant dans la barre de navigation de gauche.
1. Identifiez le compte de stockage que vous avez créé dans la section précédente, puis cliquez dessus.
1. Cliquez sur **Blobs**, puis sur le conteneur nommé **insights-logs-keydeliveryrequests**. Il s’agit du conteneur qui contient vos journaux. Les données de surveillance sont réparties dans les conteneurs par ID de ressource, puis par date et heure.
1. Accédez au fichier PT1H.json en cliquant dans les conteneurs pour l’ID de ressource, la date et l’heure. Cliquez sur le fichier PT1H.json, puis sur **Télécharger**.

 Vous pouvez à présent afficher l’événement JSON stocké dans le compte de stockage.

### <a name="examples-of-pt1hjson"></a>Exemples de PT1H.json

#### <a name="clear-key-delivery-log"></a>Journal de remise de clé

```json
{
  "time": "2019-05-21T00:07:33.2820450Z",
  "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/amsResourceGroup/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/AMSACCOUNT",
  "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
  "operationVersion": "1.0",
  "category": "KeyDeliveryRequests",
  "resultType": "Succeeded",
  "resultSignature": "OK",
  "durationMs": 253,
  "identity": {
    "authorization": {
      "issuer": "myIssuer",
      "audience": "myAudience"
    },
    "claims": {
      "urn:microsoft:azure:mediaservices:contentkeyidentifier": "e4276e1d-c012-40b1-80d0-ac15808b9277",
      "nbf": "1558396914",
      "exp": "1558400814",
      "iss": "myIssuer",
      "aud": "myAudience"
    }
  },
  "level": "Informational",
  "location": "westus2",
  "properties": {
    "requestId": "fb5c2b3a-bffa-4434-9c6f-73d689649add",
    "keyType": "Clear",
    "keyId": "e4276e1d-c012-40b1-80d0-ac15808b9277",
    "policyName": "SharedContentKeyPolicyUsedByAllAssets",
    "tokenType": "JWT",
    "statusMessage": "OK"
  }
}
```

#### <a name="widevine-encrypted-key-delivery-log"></a>Journal de remise de clé chiffrée de Widevine

```json
{
  "time": "2019-05-20T23:15:22.7088747Z",
  "resourceId": "/SUBSCRIPTIONS/00000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/amsResourceGroup/PROVIDERS/MICROSOFT.MEDIA/MEDIASERVICES/AMSACCOUNT",
  "operationName": "MICROSOFT.MEDIA/MEDIASERVICES/CONTENTKEYS/READ",
  "operationVersion": "1.0",
  "category": "KeyDeliveryRequests",
  "resultType": "Succeeded",
  "resultSignature": "OK",
  "durationMs": 69,
  "identity": {
    "authorization": {
      "issuer": "myIssuer",
      "audience": "myAudience"
    },
    "claims": {
      "urn:microsoft:azure:mediaservices:contentkeyidentifier": "0092d23a-0a42-4c5f-838e-6d1bbc6346f8",
      "nbf": "1558392430",
      "exp": "1558396330",
      "iss": "myIssuer",
      "aud": "myAudience"
    }
  },
  "level": "Informational",
  "location": "westus2",
  "properties": {
    "requestId": "49613dd2-16aa-4595-a6e0-4e68beae6d37",
    "keyType": "Widevine",
    "keyId": "0092d23a-0a42-4c5f-838e-6d1bbc6346f8",
    "policyName": "DRMContentKeyPolicy",
    "tokenType": "JWT",
    "statusMessage": "OK"
  }
}
```

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="see-also"></a>Voir aussi

* [Mesures Azure Monitor](../../azure-monitor/platform/data-platform.md)
* [Journaux de diagnostics Azure Monitor](../../azure-monitor/platform/platform-logs-overview.md)
* [Comment collecter et consommer les données des journaux de vos ressources Azure](../../azure-monitor/platform/platform-logs-overview.md)

## <a name="next-steps"></a>Étapes suivantes

[Mesures de surveillance](media-services-metrics-howto.md)
