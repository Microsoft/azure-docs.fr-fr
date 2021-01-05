---
title: Exemple de conteneur d’exécution de la commande d’exécution du docker
titleSuffix: Azure Cognitive Services
description: Commande Docker run du conteneur Analyse des sentiments
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 04/29/2020
ms.author: aahi
ms.openlocfilehash: 96b41c27b282bdc05dafa8c69398a11e8dbf3b1b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91779395"
---
Pour exécuter le conteneur*Analyse des sentiments v3*, exécutez la commande `docker run` suivante. Remplacez les espaces réservés suivants par vos valeurs :

| Espace réservé | Valeur | Format ou exemple |
|-------------|-------|---|
| **{API_KEY}** | Clé de votre ressource Analyse de texte. Cette information est disponible dans le portail Azure, sur la page **Key and endpoint** (Clé et point de terminaison) de votre ressource. |`xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`|
| **{ENDPOINT_URI}** | Point de terminaison pour accéder à l’API Analyse de texte. Cette information est disponible dans le portail Azure, sur la page **Key and endpoint** (Clé et point de terminaison) de votre ressource. | `https://<your-custom-subdomain>.cognitiveservices.azure.com` |

```bash
docker run --rm -it -p 5000:5000 --memory 8g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/textanalytics/sentiment \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cette commande :

* Exécute un conteneur *Analyse des sentiments* à partir de l’image conteneur
* Alloue un cœur de processeur et 8 gigaoctets (Go) de mémoire
* Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.