---
title: Phase d’acceptation du client du cycle de vie du processus TDSP (Team Data Science Process)
description: Objectifs, tâches et livrables associés à la phase d’acceptation du client de vos projets de science des données
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: f2294ccb1d958b229a71e45bb502b8134d8d5c7f
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93305671"
---
# <a name="customer-acceptance-stage-of-the-team-data-science-process-lifecycle"></a>Phase d’acceptation du client du cycle de vie du processus TDSP (Team Data Science Process)

Cet article présente les objectifs, tâches et livrables associés à la phase d’acceptation du client du processus TDSP. Ce processus indique un cycle de vie recommandé que vous pouvez utiliser pour structurer vos projets de science des données. Le cycle de vie expose les principales phases que les projets exécutent généralement, souvent de manière itérative :

   1. **Présentation de l’entreprise**
   2. **Acquisition et compréhension des données**
   3. **Modélisation**
   4. **Déploiement**
   5. **Acceptation du client**

Voici une représentation visuelle du cycle de vie TDSP : 

![Cycle de vie TDSP](./media/lifecycle/tdsp-lifecycle2.png) 


## <a name="goal"></a>Objectif
**Finaliser les livrables du projet**  : vérifiez que le pipeline, le modèle et leur déploiement dans un environnement de production sont conformes aux objectifs du client.

## <a name="how-to-do-it"></a>Marche à suivre
Deux tâches principales sont traitées dans cette phase :

   * **Validation du système** : vérifiez que le modèle et le pipeline déployés répondent aux besoins du client.
   * **Remise du projet** : livrez le projet à l’entité chargée d’exécuter le système dans l’environnement de production.

Le client doit confirmer que le système répond à ses besoins professionnels et que les réponses apportées aux questions sont suffisamment précises pour permettre le déploiement du système en production en vue de son utilisation par son application cliente. Toute la documentation est finalisée et révisée. Le projet est remis à l’entité responsable des opérations. Cette entité peut être une équipe de science des données au sein du service informatique ou chez le client ou un agent du client chargé d’exécuter le système dans l’environnement de production. 

## <a name="artifacts"></a>Artefacts
L’artefact principal produit dans cette étape finale est le **Rapport de clôture du projet pour le client**. Ce rapport technique contient tous les détails du projet qui sont utiles pour comprendre le fonctionnement du système. Le processus TDSP fournit un modèle de [rapport de clôture](https://github.com/Azure/Azure-TDSP-ProjectTemplate/blob/master/Docs/Project/Exit%20Report.md). Vous pouvez utiliser le modèle tel quel ou le personnaliser pour les besoins des clients spécifiques. 


## <a name="next-steps"></a>Étapes suivantes

Voici les liens vers chaque étape du cycle de vie TDSP :

   1. [Présentation de l’entreprise](lifecycle-business-understanding.md)
   2. [Acquisition et compréhension des données](lifecycle-data.md)
   3. [Modélisation](lifecycle-modeling.md)
   4. [Déploiement](lifecycle-deployment.md)
   5. [Acceptation du client](lifecycle-acceptance.md)

Nous fournissons des procédures pas à pas complètes qui illustrent toutes les étapes du processus correspondant à des scénarios spécifiques. L’article [Example walkthroughs](walkthroughs.md) (Exemples de procédures pas à pas) contient une liste des scénarios ainsi que des liens et des descriptions de miniatures. Les procédures pas à pas montrent comment combiner les outils et services dans le cloud et sur site dans un flux de travail ou un pipeline pour créer une application intelligente. 

Pour obtenir des exemples sur l’exécution de procédures dans les processus TDSP utilisant Azure Machine Learning Studio, consultez [Utilisation du processus de science des données avec Azure Machine Learning](./index.yml).