---
title: Schéma JSON de jumeau de module - Azure
description: Cette rubrique décrit le schéma JSON de jumeau de module de Live Video Analytics sur IoT Edge.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 8bd86bdc2c8de9ee586e785db2074fa772100420
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87053061"
---
# <a name="module-twin-json-schema"></a>Schéma JSON de jumeau de module

Les jumeaux d’appareil sont des documents JSON qui stockent des informations sur l’état des appareils (métadonnées, configurations et conditions). Azure IoT Hub conserve un jumeau d’appareil pour chaque appareil que vous y connectez. Pour une explication détaillée, consultez [Comprendre et utiliser les jumeaux de Module IoT Hub](../../iot-hub/iot-hub-devguide-module-twins.md).

Cette rubrique décrit le schéma JSON de jumeau de module de Live Video Analytics sur IoT Edge.

> [!NOTE]
> Pour être autorisé à accéder aux ressources Media Services et à l’API Media Services, vous devez tout d’abord être authentifié. Pour plus d’informations, consultez [Accéder à l'API Azure Media Services](../latest/media-services-apis-overview.md#accessing-the-azure-media-services-api).

## <a name="module-twin-properties"></a>Propriétés de jumeau de module

Live Video Analytics sur IoT Edge expose les propriétés de jumeau de module suivantes. 

|Propriété |Obligatoire |Dynamique |Description |
|---|---|---|---|
|applicationDataDirectory |Oui |Non |Chemin d’accès à un volume monté pour une configuration persistante. |
|azureMediaServicesArmId |Oui |Non |Identificateur Azure Resource Manager unique du compte Media Services.|
|aadTenantId |Oui |Non |ID de locataire Azure AD client|
|aadServicePrincipalAppId |Oui |Oui |Le client a créé Azure AD AppId.|
|aadServicePrincipalCertificate |Oui<sup>*</sup>  |Oui |Le client a créé un certificat Azure AD AppId.|
|aadServicePrincipalPassword |Oui<sup>*</sup>  |Oui |Le client a créé un mot de passe Azure AD AppId.|
|aadEndpoint |Non |Non |Point de terminaison Azure AD spécifique au cloud. <br/>Valeur par défaut : `https://login.microsoftonline.com` |
|aadResourceId |Non |Non |Audience/ID de ressource Azure AD spécifique au cloud <br/>Valeur par défaut : `https://management.core.windows.net/` |
|armEndpoint |Non |Non |Point de terminaison Azure Resource Manager spécifique au cloud. <br/>Valeur par défaut : `https://management.azure.com/` |
|diagnosticsLevel |Non |Oui |Verbosité des événements : <br/>Information &#x02758; Warning &#x02758; Error &#x02758; Critical &#x02758; None |
|diagnosticsEventsOutputName |Non |Oui |Sortie hub pour les événements de diagnostic. <br/>(Vide signifie que les diagnostics ne sont pas publiés)|
|operationalEventsOutputName|Non|Oui|Sortie hub pour les événements opérationnels.<br/>(Vide signifie que les événements opérationnels ne sont pas publiés)
|logLevel|Non|Oui|Celui-ci peut avoir l'une des valeurs suivantes : <br/>&#x000B7; Verbose<br/>&#x000B7; Information (Default)<br/>&#x000B7; Warning<br/>&#x000B7; Error<br/>&#x000B7; None|
|logCategories|Non|Oui|Liste séparée par des virgules avec ce qui suit : Application, MediaPipeline, Events <br/>Valeur par défaut : Application, Events|
|debugLogsDirectory|Non|Oui|Répertoire pour les journaux de débogage. S’il est présent, les journaux sont générés, s’il n’est pas présent, les journaux de débogage sont désactivés.

<sup>*</sup>Vous DEVEZ fournir un certificat ou un mot de passe de principal de service. 

Les propriétés dynamiques peuvent être mises à jour sans redémarrage du module. Vous pouvez obtenir les valeurs de plusieurs de ces propriétés en suivant les instructions de l’article [Obtention d’accès à l’API Media Services](../latest/access-api-howto.md). 

Pour plus d’informations sur le rôle des paramètres de diagnostic facultatifs, consultez l’article [Surveillance et journalisation](monitoring-logging.md).

```
{ 
    "properties.desired": { 
        // Required 
        "applicationDataDirectory": "/var/lib/azuremediaservices", 
        "azureMediaServicesArmId": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/<rgname>/providers/microsoft.media/mediaservices/<ams_account>", 
        "aadTenantId": "00000000-0000-0000-0000-000000000000", 
        "aadServicePrincipalAppId": "00000000-0000-0000-0000-000000000000", 
        "aadServicePrincipalPassword": "{Service principal password}", 

        // Optional API Access 
        "aadEndpoint": "https://<aad-endpoint>", 
        "aadResourceId": "https://management.core.windows.net/", 
        "armEndpoint": "https://management.azure.com/", 
        
        // Optional Diagnostics 
        "diagnosticsEventsOutputName": "lvaEdgeDiagnostics",
        "operationalEventsOutputName": "lvaEdgeOperational",
        "logLevel": "Information",
        "logCategories": "Application,Events"
    } 
} 
```

## <a name="next-steps"></a>Étapes suivantes

[Méthodes directes](direct-methods.md)
