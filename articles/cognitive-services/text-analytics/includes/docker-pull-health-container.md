---
title: Commande docker pull pour le conteneur de santé
titleSuffix: Azure Cognitive Services
description: Commande docker pull pour le conteneur d’Analyse de texte pour la santé
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 07/07/2020
ms.author: aahi
ms.openlocfilehash: a0b2c9548f9c1289ae0abd61a72d7146a3bbca29
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965163"
---
Renseignez et envoyez le [formulaire de demande Cognitive Services](https://aka.ms/csgate) pour demander accès à la préversion publique d’Analyse de texte pour la santé.  Cette application s’utilise à la fois avec le conteneur et l’API web hébergée en préversion publique.
Le formulaire demande des informations sur vous, votre entreprise et le scénario d’utilisateur pour lequel vous allez utiliser le conteneur. Une fois que vous avez envoyé le formulaire, l’équipe Azure Cognitive Services vérifie que vous remplissez bien les critères d’accès au registre de conteneurs privé.

> [!IMPORTANT]
> * Dans le formulaire, vous devez utiliser une adresse e-mail associée à un ID d’abonnement Azure.
> * La ressource Azure que vous utilisez pour exécuter le conteneur doit avoir été créée avec l’ID d’abonnement Azure approuvé. 
> * Vérifiez votre adresse e-mail (boîtes de réception et dossiers de courrier indésirable) pour obtenir des mises à jour sur l’état de votre application auprès de Microsoft.

Après l’approbation, vous allez recevoir un e-mail contenant les informations d’identification permettant d’accéder au registre de conteneurs privé.  Utilisez la commande docker login avec les informations d’identification fournies dans votre e-mail d’accueil afin de vous connecter à notre registre de conteneurs privé pour les conteneurs Cognitive Services.


```Docker
docker login containerpreview.azurecr.io -u <username> -p <password>
```

Utilisez la commande [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger cette image conteneur à partir de notre registre de conteneurs privé.

```
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-healthcare:latest
```
