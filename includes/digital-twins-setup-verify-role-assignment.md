---
author: baanders
description: Fichier include pour vérifier l’attribution de rôle dans la configuration d’Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 7/22/2020
ms.author: baanders
ms.openlocfilehash: b4dfd154ff3fb7af48ef43b0a1dca168c5a93481
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92489034"
---
Une façon de vérifier que vous avez correctement configuré l’attribution de rôle consiste à afficher les attributions de rôle pour l’instance Azure Digital Twins dans le [portail Azure](https://portal.azure.com). Accédez à votre instance Azure Digital Twins dans le portail Azure (vous pouvez la rechercher dans la page des [instances Azure Digital Twins](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.DigitalTwins%2FdigitalTwinsInstances) ou rechercher son nom dans la barre de recherche du portail).

Ensuite, affichez tous les rôles qui lui sont attribués sous *Contrôle d’accès (IAM) > Attributions de rôle* . L’utilisateur doit s’afficher dans la liste avec le rôle *Propriétaire des données Azure Digital Twins* . 

:::image type="content" source="../articles/digital-twins/media/how-to-set-up-instance/portal/verify-role-assignment.png" alt-text="Vue des attributions de rôles pour une instance Azure Digital Twins dans le portail Azure":::