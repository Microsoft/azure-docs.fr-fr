---
title: Fichier include
description: Fichier include
services: data-factory
author: memildin
ms.service: data-factory
ms.topic: include
ms.date: 03/10/2020
ms.author: memildin
ms.custom: include file
ms.openlocfilehash: 55390a3eb2a074729b4a0868416a95e208325b76
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91400935"
---
```json
{
  "VendorName": "Microsoft",
  "AlertType": "SUSPECT_SVCHOST",
  "StartTimeUtc": "2016-12-20T13:38:00.000Z",
  "EndTimeUtc": "2019-12-20T13:40:01.733Z",
  "ProcessingEndTime": "2019-09-16T12:10:19.5673533Z",
  "TimeGenerated": "2016-12-20T13:38:03.000Z",
  "IsIncident": false,
  "Severity": "High",
  "Status": "New",
  "ProductName": "Azure Security Center",
  "SystemAlertId": "2342409243234234_F2BFED55-5997-4FEA-95BD-BB7C6DDCD061",
  "CompromisedEntity": "WebSrv1",
  "Intent": "Execution",
  "AlertDisplayName": "Suspicious process detected",
  "Description": "Suspicious process named 'SVCHOST.EXE' was running from path: %{Process Path}",
  "RemediationSteps": ["contact your security information team"],
  "ExtendedProperties": {
    "Process Path": "c:\\temp\\svchost.exe",
    "Account": "Contoso\\administrator",
    "PID": 944,
    "ActionTaken": "Detected"
  },
  "Entities": [],
  "ResourceIdentifiers": [
        {
            Type: "AzureResource",
            AzureResourceId: "/subscriptions/86057C9F-3CDD-484E-83B1-7BF1C17A9FF8/resourceGroups/backend-srv/providers/Microsoft.Compute/WebSrv1"
        },
        {
            Type: "LogAnalytics",
            WorkspaceId: "077BA6B7-8759-4F41-9F97-017EB7D3E0A8",
            WorkspaceSubscriptionId: "86057C9F-3CDD-484E-83B1-7BF1C17A9FF8",
            WorkspaceResourceGroup: "omsrg",
            AgentId: "5A651129-98E6-4E6C-B2CE-AB89BD815616",
        }
  ]
}
```

### <a name="the-data-model-of-the-schema"></a>Modèle de données du schéma

|Champ|Type de données|Description|
|----|----|----|
|**AlertDisplayName**|String|Nom complet de l’alerte.|
|**AlertType**|String|Type d’alerte. Les alertes du même type doivent avoir la même valeur. Ce champ est une chaîne indexée représentant le type d’alerte, et non une instance d’alerte. Toutes les instances d’alerte de la même logique de détection ou analytique doivent avoir la même valeur pour le type d’alerte.|
|**CompromisedEntity**|String|Nom complet de la ressource la plus pertinente par rapport à cette alerte.|
|**Description**|String|Description de l’alerte.|
|**EndTimeUtc**|DateTime|Heure du dernier événement ou de la dernière activité inclus dans l’alerte.  Le champ doit être une chaîne conforme au format ISO8601, y compris les informations de fuseau horaire UTC.|
|**Entités**|IEnumerable (IEntity)|Liste des entités liées à l’alerte. Cette liste peut contenir un mélange d’entités de types divers. Le type des entités peut être l’un des types définis dans la section Entities. Les entités qui ne figurent pas dans la liste ci-dessous peuvent également être envoyées. Toutefois, il n’est pas garanti qu’elles seront traitées (l’alerte n’échouera pas lors de la validation avec de nouveaux types d’entités).|
|**ExtendedProperties**|Dictionary (String,String)|Les fournisseurs peuvent éventuellement inclure des champs personnalisés ici.|
|**Intention**|Enum|Intention de la chaîne de destruction derrière l’alerte. Pour obtenir la liste des valeurs prises en charge, ainsi que des explications sur les intentions de chaîne de destruction prises en charge par Azure Security Center, consultez [Intentions](../articles/security-center/alerts-reference.md#intentions).<br/>Ce champ peut avoir plusieurs valeurs (séparées par des virgules).|
|**IsIncident**|Bool|Ce champ détermine si l’alerte est un incident (regroupement composé de plusieurs alertes) ou une alerte unique. La valeur par défaut du champ est « false » (ce qui signifie qu’il s’agit d’une alerte unique).|
|**ProcessingEndTime**|DateTime|Heure à laquelle l’alerte a été accessible à l’utilisateur final dans le produit d’origine contenant l’alerte.|
|**ProductName**|String|Nom du produit qui a publié cette alerte (Azure Security Center, Azure ATP, Microsoft Defender ATP, MCAS, etc.).|
|**RemediationSteps**|Liste <String>|Éléments d’action manuelle à mettre à jour pour corriger l’alerte.|
|**ResourceIdentifiers**|Liste (identificateurs de ressource)|Identificateurs de ressource pour cette alerte qui peuvent être utilisés pour diriger l’alerte vers le groupe approprié d’exposition au produit (locataire, espace de travail, abonnement, etc.). Il peut y avoir plusieurs identificateurs de type différent par alerte.|
|**Niveau de gravité**|Enum|Gravité de l’alerte telle qu’elle est signalée par le fournisseur. Valeurs possibles : informatif, faible, moyen et élevé.|
|**StartTimeUtc**|DateTime|Heure du premier événement ou de la première activité inclus dans l’alerte. Le champ doit être une chaîne conforme au format ISO8601, y compris les informations de fuseau horaire UTC.|
|**État**|Enum|État du cycle de vie de l’alerte.<br/>Les états pris en charge sont : Nouveau, Résolu, Ignoré, Inconnu.<br/>L’état « Inconnu » est attribué à une alerte qui spécifie une valeur autre que les options prises en charge.<br/>L’état « Nouveau » est attribué à une alerte qui ne spécifie pas de valeur.|
|**SystemAlertId**|String|Identificateur d’alerte.|
|**TimeGenerated**|DateTime|Heure à laquelle l’alerte a été générée par le fournisseur d’alertes. Si l’alerte n’a pas été signalée par des fournisseurs d’alertes internes, un produit peut choisir d’attribuer l’heure à laquelle elle a été reçue pour traitement par le produit.  Le champ doit être une chaîne conforme au format ISO8601, y compris les informations de fuseau horaire UTC.|
|**VendorName**|String|Nom du fournisseur qui a déclenché l’alerte.|
|||
