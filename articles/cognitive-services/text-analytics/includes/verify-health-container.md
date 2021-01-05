---
title: Vérification de l’instance de conteneur de santé
titleSuffix: Azure Cognitive Services
description: Découvrez comment vérifier l’instance de conteneur de santé.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 07/07/2020
ms.author: aahi
ms.openlocfilehash: 83a9eeb7644d107a808494ad06a8bef91d471fe1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96009881"
---
### <a name="verify-that-a-container-is-running"></a>Vérification de l’exécution du conteneur

1. Sélectionnez l’onglet **Vue d’ensemble** et copiez l’adresse IP.
1. Ouvrez un nouvel onglet de navigateur et entrez l’adresse IP. Par exemple, entrez `http://<IP-address>:5000 (http://55.55.55.55:5000`. La page d’accueil du conteneur s’affiche, vous informant que le conteneur est en cours d’exécution.

    ![Affichez la page d’accueil du conteneur pour vérifier qu’elle est en cours d’exécution](../media/how-tos/container-instance/swagger-docs-on-container.png)

1. Veuillez sélectionner le lien **Description de l’API de service** pour accéder à la page Swagger des conteneurs.

1. Choisissez une des API **POST** et sélectionnez **Essayer**. Les paramètres sont affichés, y compris l’exemple d’entrée.

Il existe également plusieurs URL permettant de vérifier que le conteneur est en cours d’exécution.

|Requête|Objectif|
|--|--|
|`http://localhost:5000/`|Le conteneur fournit une page d’accueil.|
|`http://localhost:5000/ready`|Cette demande, qui utilise GET, permet de vérifier que le conteneur est prêt à accepter une requête sur le modèle. Cette requête peut être utilisée pour les [probes liveness et readiness](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) de Kubernetes.|
|`http://localhost:5000/status`|Comme le point de terminaison/ready, cette demande, qui utilise GET, permet de confirmer que le conteneur s’exécute sans exécuter de requête sur le modèle. Cette requête peut être utilisée pour les [probes liveness et readiness](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) de Kubernetes.|
|`http://localhost:5000/swagger`|Par le biais de cette URL, le conteneur fournit un ensemble complet de documentation pour les points de terminaison et une fonctionnalité `Try it now`. Avec cette fonctionnalité, vous pouvez entrer vos paramètres dans un formulaire HTML basé sur le web, et constituer la requête sans avoir à écrire du code. Une fois la requête retournée, un exemple de commande CURL est fourni pour illustrer les en-têtes HTTP, et le format du corps qui est nécessaire. |
|`http://localhost:5000/demo`| Par le biais d’un navigateur, cette fonctionnalité fournit une visualisation interactive des résultats des requêtes pour les exemples de texte d’entrée ou le texte de votre choix.  |

Utilisez cette URL de demande (`http://localhost:5000/text/analytics/v3.2-preview.1/entities/health`) pour envoyer une requête au conteneur.
