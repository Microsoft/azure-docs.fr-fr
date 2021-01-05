---
title: Fichier include
description: Fichier include
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 06/11/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 020e59f029b09f3c7656f67039731e4141e68d31
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177667"
---
Si Python rencontre une erreur lors du démarrage de votre application, seule une page d’erreur simple est retournée (par exemple « La page ne peut pas être affichée, car une erreur de serveur interne s’est produite. »).

Pour capturer les erreurs d’application Python :

1. Dans le portail Azure, dans votre application web, sélectionnez **Paramètres**.
2. Sous l’onglet **Paramètres**, sélectionnez **Paramètres de l’application**.
3. Sous **Paramètres de l’application**, entrez la paire clé/valeur suivante :
    * Clé : WSGI_LOG
    * Valeur : D:\home\site\wwwroot\logs.txt (entrez le nom de fichier de votre choix)

Les erreurs doivent maintenant être journalisées dans le fichier logs.txt du dossier wwwroot.
