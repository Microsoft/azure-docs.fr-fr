---
author: IEvangelist
ms.author: dapine
ms.date: 02/19/2020
ms.service: cognitive-services
ms.topic: include
ms.openlocfilehash: 2ac93f5aba722eea78267a512999a5581a887b99
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95996414"
---
Les requêtes sur le conteneur sont facturées au niveau tarifaire de la ressource Azure utilisée pour `ApiKey`.

Les conteneurs Azure Cognitive Services ne sont pas concédés sous licence dans le but de s’exécuter sans être connectés au point de terminaison de mesure/facturation. Vous devez configurer les conteneurs de manière à ce qu’ils communiquent les informations de facturation au point de terminaison de facturation à tout moment. Les conteneurs Cognitive Services n’envoient pas de données client, telles que l’image ou le texte analysé, à Microsoft.

### <a name="connect-to-azure"></a>Connexion à Azure

Le conteneur a besoin des valeurs d’arguments de facturation pour s’exécuter. Ces valeurs permettent au conteneur de se connecter au point de terminaison de facturation. Le conteneur crée des rapports sur l’utilisation toutes les 10 à 15 minutes. Si le conteneur ne se connecte pas à Azure dans la fenêtre de temps imparti, il continue de s’exécuter, mais ne traite pas les requêtes tant que le point de terminaison de facturation n’est pas restauré. Une tentative de connexion est effectuée 10 fois, toutes les 10 à 15 minutes. S’il ne parvient pas à se connecter au point de terminaison de facturation au cours de ces 10 essais, le conteneur cesse de traiter les demandes.

### <a name="billing-arguments"></a>Arguments de facturation

La commande <a href="https://docs.docker.com/engine/reference/commandline/run/" target="_blank">`docker run` <span class="docon docon-navigate-external x-hidden-focus"></span></a> démarre le conteneur quand les trois options suivantes ont des valeurs valides :

| Option | Description |
|--------|-------------|
| `ApiKey` | Clé API de la ressource Cognitive Services utilisée pour le suivi des informations de facturation.<br/>La valeur de cette option doit être définie sur une clé API pour la ressource provisionnée spécifiée dans `Billing`. |
| `Billing` | Point de terminaison de la ressource Cognitive Services utilisé pour le suivi des informations de facturation.<br/>La valeur de cette option doit être définie sur l’URI de point de terminaison d’une ressource Azure provisionnée.|
| `Eula` | Indique que vous avez accepté la licence pour le conteneur.<br/>La valeur de cette option doit être définie sur **accepter**. |
