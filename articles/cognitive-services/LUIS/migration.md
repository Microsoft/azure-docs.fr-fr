---
title: Vue d’ensemble de la migration - LUIS
description: Comprendre ce qui se trouve sur un chemin de migration
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/22/2020
ms.openlocfilehash: d6ecacf9aa1a7e650de74a412ed4f161ed0e0790
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91253800"
---
# <a name="migration-in-luis"></a>Migration dans LUIS

Plusieurs éléments se trouvent sur un chemin de migration. Utilisez le tableau suivant pour identifier les éléments affectés et quand vous avez besoin d’effectuer une migration complète.

|Domaine|Description|Date de fin de la migration|
|--|--|--|
|[API de prédiction](luis-migration-api-v3.md)|Migrer vers l’API V3.|TBD|
|[Créer des API](luis-migration-authoring-entities.md)|Migrer vers l’API V3.|TBD|
|[Ressource de création](luis-migration-authoring.md)|Migrez à partir d’aucune ressource de création à l’aide d’une ressource de création LUIS dans le portail LUIS.|2 novembre 2020 |
|[Caractéristique requise](luis-migration-authoring-entities.md#api-change-constraint-replaced-with-required-feature)|Cette modification a été effectuée en mai 2020 lors de la conférence //Build et s’applique uniquement aux API de création v3 dans lesquelles une application utilise une fonctionnalité contrainte.|19 juin 2020|
|[Entité composite](migrate-from-composite-entity.md)|Mettez à niveau une entité composite vers une entité de machine-learning pour générer une entité qui reçoit des prédictions plus complètes avec une meilleure décomposition pour déboguer l’entité.|TBD|
