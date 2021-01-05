---
title: Afficher les détails de l’authentification d’Azure Maps
description: Afficher le Portail Azure pour afficher les détails de l’authentification d’Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/17/2020
ms.topic: include
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: a6ffcbf5a8c36958dd3ea74de4d826fe25a1139c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87126451"
---
Vous pouvez afficher les détails de l’authentification du compte Azure Maps dans le Portail Azure. Dans votre compte, dans le menu **Paramètres**, sélectionnez **Authentification**.

![Informations sur l’authentification](../media/how-to-manage-authentication/how-to-view-auth.png)

Une fois qu’un compte Azure Maps a été créé, la valeur `x-ms-client-id` Azure Maps est présente dans la page des détails de l’authentification du Portail Azure. Cette valeur représente le compte qui sera utilisé pour les requêtes d’API REST. Elle doit être stockée dans la configuration de l’application et récupérée avant d’effectuer des requêtes HTTP lors de l’utilisation de l’authentification Azure AD avec Azure Maps.
