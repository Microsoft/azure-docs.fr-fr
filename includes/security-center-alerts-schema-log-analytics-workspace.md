---
title: Fichier include
description: Fichier include
services: data-factory
author: memildin
ms.service: data-factory
ms.topic: include
ms.date: 03/17/2020
ms.author: memildin
ms.custom: include file
ms.openlocfilehash: 7d81799f7fbdb2b41db421daa1a85ec8cde511eb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "79541385"
---
### <a name="the-data-model-of-the-schema"></a>Modèle de données du schéma

|Champ|Description|
|----|----|
|**AlertName**|Nom d’affichage de l’alerte|
|**AlertType**|Identificateur d’alerte unique|
|**ConfidenceLevel**|(Facultatif) Niveau de confiance de cette alerte (haut/bas)|
|**ConfidenceScore**|(Facultatif) Indicateur de confiance numérique de l’alerte de sécurité|
|**Description**|Texte de description pour l’alerte|
|**DisplayName**|Nom d’affichage de l’alerte|
|**EndTime**|Heure de fin de l’impact de l’alerte (heure du dernier événement contribuant à l’alerte)|
|**Entités**|Liste des entités liées à l’alerte Cette liste peut contenir un mélange d’entités de types divers.|
|**ExtendedLinks**|(Facultatif) Conteneur pour tous les liens liés à l’alerte. Ce conteneur peut contenir un mélange de liens pour des types divers|
|**ExtendedProperties**|Conteneur de champs supplémentaires qui sont pertinents pour l’alerte|
|**IsIncident**|Détermine si l’alerte est un incident ou une alerte régulière. Un incident est une alerte de sécurité qui regroupe plusieurs alertes dans un incident de sécurité.|
|**ProcessingEndTime**|Timestamp UTC dans lequel l’alerte a été créée|
|**ProductComponentName**|(Facultatif) Nom d’un composant à l’intérieur du produit qui a généré l’alerte|
|**ProductName**|constant ('Azure Security Center')|
|**ProviderName**|inutilisé|
|**RemediationSteps**|Éléments d’action manuelle à mettre à jour pour corriger la menace de sécurité|
|**ResourceId**|Identificateur complet de la ressource affectée|
|**Niveau de gravité**|Gravité de l’alerte (haute/moyenne/faible/informative)|
|**SourceComputerId**|GUID unique pour le serveur concerné (si l’alerte est générée sur le serveur)|
|**SourceSystem**|inutilisé|
|**StartTime**|Heure de début de l’impact de l’alerte (heure du premier événement contribuant à l’alerte)|
|**SystemAlertId**|Identificateur unique de cette instance d’alerte de sécurité|
|**TenantId**|Identificateur du locataire Azure Active Directory parent de l’abonnement sous lequel la ressource analysée réside|
|**TimeGenerated**|Timestamp UTC à laquelle l’évaluation a eu lieu (heure d’analyse de Security Center) (identique à DiscoveredTimeUTC)|
|**Type**|constant ('SecurityAlert')|
|**VendorName**|Nom du fournisseur qui a déclenché l’alerte (par exemple, 'Microsoft')|
|**VendorOriginalId**|inutilisé|
|**WorkspaceResourceGroup**|Contient le nom de ce groupe de ressources d’espace de travail si l’alerte est générée sur une machine virtuelle, un serveur, un groupe de machines virtuelles identiques ou une instance App Service qui rend compte à un espace de travail.|
|**WorkspaceSubscriptionId**|Contient cet espace de travail subscriptionId si l’alerte est générée sur une machine virtuelle, un serveur, un groupe de machines virtuelles identiques ou une instance App Service qui rend compte à un espace de travail.|
|||
