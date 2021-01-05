---
title: Fichier Include
description: Fichier Include
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 07/15/2019
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 39b61c78cbd0333183c80234256d6041a109d846
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68249915"
---
L’ID de ressource Azure AD indique l’audience pour laquelle un jeton émis peut être utilisé pour fournir l’accès à une ressource Azure. Dans le cas du stockage Azure, l’ID de ressource peut être spécifique à un seul compte de stockage ou s’appliquer à n’importe quel compte de stockage. Le tableau suivant décrit les valeurs que vous pouvez fournir pour l’ID de la ressource :

|ID de ressource  |Description  |
|---------|---------|
|`https://<account>.blob.core.windows.net` <br /><br /> `https://<account>.queue.core.windows.net`    | Point de terminaison de service pour un compte de stockage donné. Utilisez cette valeur pour acquérir un jeton qui autorise uniquement les demandes adressées à ce compte de stockage Azure et à ce service. Remplacez la valeur entre crochets par le nom de votre compte de stockage.      |
|`https://storage.azure.com/`     | À utiliser pour obtenir un jeton pour l’autorisation des requêtes vers n’importe quel compte de stockage Azure.        |
