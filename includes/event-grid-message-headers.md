---
title: Fichier Include
description: Fichier Include
services: event-grid
author: spelluru
ms.service: event-grid
ms.topic: include
ms.date: 11/19/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: c37e2c28eb6aee9edf4bdd97066ce5f15e7447c1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005626"
---
## <a name="message-headers"></a>En-têtes de message
Les propriétés que vous recevez dans les en-têtes de message sont les suivantes :

| Nom de la propriété | Description |
| ------------- | ----------- | 
| aeg-subscription-name | Nom de l’abonnement aux événements. |
| aeg-delivery-count | Nombre de tentatives effectuées pour l’événement. |
| aeg-event-type | <p>Type de l’événement.</p><p>Ce peut être l’une des valeurs suivantes :</p><ul><li>SubscriptionValidation</li><li>Notification</li><li>SubscriptionDeletion</li></ul> | 
| aeg-metadata-version | <p>Version de métadonnées de l’événement.<p> Pour un **schéma d’événement Event Grid**, cette propriété représente la version des métadonnées et, pour un **schéma d’événements cloud**, elle représente la **version des spécifications**. </p>|
| aeg-data-version | <p>Version de données de l’événement.</p><p>Pour un **schéma d’événement Event Grid**, cette propriété représente la version des données et, pour un **schéma d’événements cloud**, elle ne s’applique pas.</p> |
| aeg-output-event-id | ID de l’événement Event Grid. |


