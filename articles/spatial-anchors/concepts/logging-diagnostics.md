---
title: Journalisation et diagnostics
description: Explication détaillée de la méthode à suivre pour générer et récupérer des diagnostics et des journaux dans Azure Spatial Anchors.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: conceptual
ms.service: azure-spatial-anchors
ms.custom: devx-track-csharp
ms.openlocfilehash: da8ffd7ff0b8473ce558943bb420b36f26c3fc32
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95494639"
---
# <a name="logging-and-diagnostics-in-azure-spatial-anchors"></a>Journalisation et diagnostics dans Azure Spatial Anchors

Azure Spatial Anchors fournit un mécanisme de journalisation standard, qui est utile pour le développement d’applications. Le mode de journalisation des diagnostics Azure Spatial Anchors fournit des informations supplémentaires pour le débogage. La fonction de journalisation des diagnostics stocke les images de l’environnement.

## <a name="standard-logging"></a>Journalisation standard
Dans l’API Spatial Anchors, vous pouvez vous abonner au mécanisme de journalisation pour obtenir des journaux utiles pour le développement d’applications et le débogage. Les API de journalisation standard ne stockent aucune image de l’environnement sur le disque de l’appareil. Le Kit de développement logiciel (SDK) fournit ces journaux en tant que rappels d’événement. Il vous incombe d’intégrer ces journaux dans le mécanisme de journalisation de l’application.

### <a name="configuration-of-log-messages"></a>Configuration des messages des journaux
Deux rappels sont particulièrement intéressants pour l’utilisateur. L’exemple suivant montre comment configurer la session.

```csharp
    cloudSpatialAnchorSession = new CloudSpatialAnchorSession();
    . . .
    // set up the log level for the runtime session
    cloudSpatialAnchorSession.LogLevel = SessionLogLevel.Information;

    // configure the callback for the debug log
    cloudSpatialAnchorSession.OnLogDebug += CloudSpatialAnchorSession_OnLogDebug;

    // configure the callback for the error log
    cloudSpatialAnchorSession.Error += CloudSpatialAnchorSession_Error;
```

### <a name="events-and-properties"></a>Événements et propriétés

Ces rappels d’événement sont fournis pour traiter les journaux et les erreurs de la session :

- [LogLevel](/dotnet/api/microsoft.azure.spatialanchors.cloudspatialanchorsession.loglevel) : spécifie le niveau de détail que le runtime doit envoyer en ce qui concerne les événements.
- [OnLogDebug](/dotnet/api/microsoft.azure.spatialanchors.cloudspatialanchorsession.onlogdebug) : fournit les événements de journalisation de débogage standard.
- [Erreur](/dotnet/api/microsoft.azure.spatialanchors.cloudspatialanchorsession.error) : fournit les événements de journalisation que le runtime considère comme des erreurs.

## <a name="diagnostics-logging"></a>Journalisation des diagnostics

Outre le mode standard de fonctionnement à des fins de journalisation, Spatial Anchors propose un mode de diagnostic. Ce mode capture les images de l’environnement et les enregistre sur le disque. Vous pouvez utiliser ce mode pour déboguer certains types de problèmes, par exemple lorsque vous ne parvenez pas à localiser une ancre de manière fiable. N’activez l’enregistrement des diagnostics que pour reproduire un problème spécifique. Ensuite, désactivez-le. Ne le démarrez pas lorsque les applications s’exécutent normalement.

Cependant, lorsque vous contactez le support de Microsoft, un représentant peut vous demander si vous voulez lui envoyer un ensemble de diagnostics à des fins d’enquête. Dans ce cas, vous pouvez activer les diagnostics, afin de reproduire le problème et d’envoyer ces informations au support.

Si vous envoyez un journal de diagnostic à Microsoft sans avoir reçu l’accord préalable d’un représentant Microsoft, votre demande restera sans réponse.

Les sections suivantes indiquent comment activer le mode de diagnostic, et aussi comment envoyer des journaux de diagnostic à Microsoft.

### <a name="enable-diagnostics-logging"></a>Activer la journalisation des diagnostics

Lorsque vous activez une session pour la journalisation des diagnostics, toutes les opérations de cette session sont associées à des journaux de diagnostics dans le système de fichiers local. Lors de la journalisation, les images de l’environnement sont enregistrées sur le disque.

```csharp
private void ConfigureSession()
{
    cloudSpatialAnchorSession = new CloudSpatialAnchorSession();
    . . .

    // set up the log level for the runtime session
    cloudSpatialAnchorSession.LogLevel = SessionLogLevel.Information;

    // configure the callbacks for logging and errors
    cloudSpatialAnchorSession.OnLogDebug += CloudSpatialAnchorSession_OnLogDebug;
    cloudSpatialAnchorSession.Error += CloudSpatialAnchorSession_Error;

    // opt in to diagnostics logging of environment images
    // if this is enabled, the diagnostics bundle includes images of the environment captured by the session
    cloudSpatialAnchorSession.Diagnostics.ImagesEnabled = true;

    // set the level of detail to be collected in the diagnostics log by the session
    cloudSpatialAnchorSession.Diagnostics.LogLevel = SessionLogLevel.All;

    // set the max bundle size to capture the bug information
    cloudSpatialAnchorSession.Diagnostics.MaxDiskSizeInMB = 200;
    . . .
}
```

### <a name="submit-the-diagnostics-bundle"></a>Envoyer un ensemble de diagnostics

L’extrait de code suivant montre comment envoyer un ensemble de diagnostics à Microsoft. Cet ensemble inclut les images de l’environnement capturées par la session une fois que vous avez activé les diagnostics.

```csharp
// method to handle the diagnostics bundle submission
private async Task CreateAndSubmitBundle()
{
    // create the diagnostics bundle manifest to collect the session information
    string path = await cloudSpatialAnchorSession
                              .Diagnostics
                              .CreateManifestAsync("Description of the issue");

    // submit the manifest and data to send feedback to Microsoft
    await cloudSpatialAnchorSession.Diagnostics.SubmitManifestAsync(path);
}
```

### <a name="parts-of-a-diagnostics-bundle"></a>Contenu d’un ensemble de diagnostics
L’ensemble de diagnostics peut contenir les informations suivantes :

- **Images clés** : images de l’environnement capturées pendant la session une fois les diagnostics capturés.
- **Journaux d’activité** : événements de journalisation enregistrés par le runtime.
- **Métadonnées de session** : métadonnées qui identifient la session.