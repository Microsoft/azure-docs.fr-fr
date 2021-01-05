---
title: Fichier include
description: Fichier include
services: azure-sentinel
author: yelevin
ms.service: azure-sentinel
ms.topic: include
ms.date: 06/28/2020
ms.author: yelevin
ms.custom: include file
ms.openlocfilehash: 63cb53dc60a718892d4bf86140e7fd51303bd61c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88761715"
---
### <a name="the-data-model-of-the-schema"></a>Modèle de données du schéma

| Champ | Type de données | Description |
| ---- | ---- | ---- |
| **AdditionalData** | dynamique | Nombre d’alertes, nombre de signets, nombre de commentaires, noms et tactiques des produits d’alerte |
| **AlertIds** | dynamique | Alertes à partir desquelles l’incident a été créé |
| **BookmarkIds** | dynamique | Entités avec signet |
| **Classification** | string | Classification de fermeture de l'incident |
| **ClassificationComment** | string | Commentaire de classification de fermeture de l'incident |
| **ClassificationReason** | string | Raison de la classification de fermeture de l'incident |
| **ClosedTime** | DATETIME | Timestamp (UTC) de la dernière fermeture de l’incident |
| **Commentaires** | dynamique | Commentaires sur l’incident |
| **CreatedTime** | DATETIME | Timestamp (UTC) de la création de l’incident |
| **Description** | string | Description de l'incident |
| **FirstActivityTime** | DATETIME | Heure du premier événement |
| **FirstModifiedTime** | DATETIME | Timestamp (UTC) de la première modification de l’incident |
| **IncidentName** | string | GUID interne |
| **IncidentNumber** | int |  |
| **IncidentUrl** | string | Lien vers l’incident |
| **Étiquettes** | dynamique | Balises |
| **LastActivityTime** | DATETIME | Heure du dernier événement |
| **LastModifiedTime** | DATETIME | Timestamp (UTC) de la dernière modification de l’incident <br>(modification décrite par l’enregistrement en cours) |
| **ModifiedBy** | string | Utilisateur ou système qui a modifié l’incident |
| **Propriétaire** | dynamique |  |
| **RelatedAnalyticRuleIds** | dynamique | Règles à partir desquelles les alertes de l’incident ont été déclenchées |
| **Niveau de gravité** | string | Gravité de l’incident (haute/moyenne/faible/informative) |
| **SourceSystem** | string | Constante (« Azure ») |
| **État** | string |  |
| **TenantId** | string |  |
| **TimeGenerated** | DATETIME | Timestamp (UTC) de la création de l’enregistrement actuel <br>(lors de la modification de l’incident) |
| **Titre** | string | 
| **Type** | string | Constante (« SecurityIncident ») |
|
