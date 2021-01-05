---
title: Application de DevOps avec GitHub - LUIS
titleSuffix: Azure Cognitive Services
description: Appliquez DevOps avec LUIS et GitHub.
services: cognitive-services
author: andycw
manager: cmayo
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 06/5/2020
ms.author: anwigley
ms.openlocfilehash: b733e90b69be4e2bd458be5486564747ed4cca78
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018869"
---
# <a name="apply-devops-to-luis-app-development-using-github-actions"></a>Appliquer DevOps au développement d’applications LUIS à l’aide de GitHub Actions

Accédez au [référentiel de modèles DevOps LUIS](https://github.com/Azure-Samples/LUIS-DevOps-Template) pour obtenir une solution complète qui implémente les pratiques recommandées de DevOps et d’ingénierie logicielle pour LUIS. Vous pouvez utiliser ce référentiel de modèles pour créer votre propre référentiel avec la prise en charge intégrée des flux de travail CI/CD et des pratiques qui permettent d’activer le [contrôle de code source](luis-concept-devops-sourcecontrol.md), des [builds automatisés](luis-concept-devops-automation.md), des [tests](luis-concept-devops-testing.md) et la [gestion des mises en production](luis-concept-devops-automation.md#release-management) avec LUIS pour votre propre projet.

## <a name="the-luis-devops-template-repo"></a>Référentiel de modèles DevOps LUIS

Le [référentiel de modèles DevOps LUIS](https://github.com/Azure-Samples/LUIS-DevOps-Template) vous indique comment effectuer les tâches suivantes :

* **Cloner le référentiel de modèles** - Copiez le modèle dans votre propre référentiel GitHub.
* **Configurer les ressources LUIS** - Créez les [ressources LUIS de création et de prédiction dans Azure](./luis-how-to-azure-subscription.md) qui seront utilisées par les flux de travail d’intégration en continu.
* **Configurer les flux de travail CI/CD** - Configurez les paramètres des flux de travail CI/CD et stockez-les dans [GitHub Secrets](https://help.github.com/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets).
* **Décrit la [« boucle interne de développement »](https://mitchdenny.com/the-inner-loop/)** - Le développeur apporte des mises à jour à un exemple d’application LUIS tout en travaillant dans une branche de développement, teste les mises à jour, puis lève une demande de tirage (pull request) pour proposer des modification et obtenir une approbation après révision.
* **Exécuter des flux de travail CI/CD** - Exécutez des [flux de travail d’intégration en continu pour générer et tester une application LUIS](luis-concept-devops-automation.md) à l’aide de GitHub Actions.
* **Exécuter des tests automatisés** - Exécutez des [tests automatisés par lot pour une application LUI](luis-concept-devops-testing.md) afin d’évaluer la qualité de l’application.
* **Déployer l’application LUIS** - Exécutez un [travail de livraison continue (CD)](luis-concept-devops-automation.md#continuous-delivery-cd) pour publier l’application LUIS.
* **Utiliser le référentiel avec votre propre projet** - Explique comment utiliser référentiel avec votre propre application LUIS.

## <a name="next-steps"></a>Étapes suivantes

* Utilisez le [référentiel de modèles DevOps LUIS](https://github.com/Azure-Samples/LUIS-DevOps-Template) pour appliquer DevOps avec votre propre projet.
* [Stratégies de contrôle de code source et de branche pour LUIS](luis-concept-devops-sourcecontrol.md)
* [Test de LUIS DevOps](luis-concept-devops-testing.md)
* [Flux de travail d’automatisation pour DevOps LUIS](luis-concept-devops-automation.md)
