---
title: Docker pull du conteneur Extraction de phrases clés
titleSuffix: Azure Cognitive Services
description: Commande Docker pull pour le conteneur Extraction de phrases clés
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 02dde8a27b9687e58bf1a09c1a951f306937f0d6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90906096"
---
#### <a name="docker-pull-for-the-key-phrase-extraction-container"></a>Docker pull du conteneur Extraction de phrases clés

Utilisez la commande [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger une image conteneur à partir de Microsoft Container Registry.

Pour obtenir une description complète des balises disponibles pour les conteneurs Analyse de texte, consultez le conteneur [Extraction de phrases clés](https://go.microsoft.com/fwlink/?linkid=2018757) dans le Docker Hub.

```
docker pull mcr.microsoft.com/azure-cognitive-services/textanalytics/keyphrase:latest
```
