---
title: Apprentissage automatique responsable (préversion)
titleSuffix: Azure Machine Learning
description: Apprenez ce qu’est l’apprentissage automatique responsable et comment l’utiliser avec Azure Machine Learning pour comprendre les modèles, protéger les données et contrôler le cycle de vie des modèles.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: luquinta
author: luisquintanilla
ms.date: 11/16/2020
ms.openlocfilehash: ff85ed669fb7c3ac43e322eb747c537a30801920
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94701430"
---
# <a name="what-is-responsible-machine-learning-preview"></a>Qu’est-ce que l’apprentissage automatique responsable ? (préversion)

Cet article explique ce qu’est l’apprentissage automatique (ML) responsable et comment le mettre en pratique avec Azure Machine Learning.

## <a name="responsible-machine-learning-principles"></a>Principes de l’apprentissage automatique responsable

La confiance doit toujours rester au cœur du développement et de l’utilisation des systèmes d’intelligence artificielle. Confiance dans la plateforme, le processus et les modèles. Chez Microsoft, l’apprentissage automatique responsable englobe les valeurs et principes suivants :

- Comprendre les modèles d’apprentissage automatique
  - Interpréter et expliquer le comportement d’un modèle
  - Évaluer et atténuer la partialité d’un modèle
- Protéger les personnes et leurs données
  - Empêcher l’exposition des données avec la confidentialité différentielle
  - Utiliser des données chiffrées à l’aide du chiffrement homomorphe
- Contrôler le processus d’apprentissage automatique de bout en bout
  - Documenter le cycle de vie de l’apprentissage automatique avec des feuilles de données

:::image type="content" source="media/concept-responsible-ml/responsible-ml-pillars.png" alt-text="Les piliers du ML responsable (interprétabilité, confidentialité différentielle, chiffrement homomorphe, piste d’audit) dans Azure Machine Learning":::

À l’heure où l’intelligence artificielle et les systèmes autonomes s’intègrent plus profondément dans la structure de la société, il est important de faire l’effort d’anticiper et d’atténuer les conséquences inattendues de ces technologies.

## <a name="interpret-and-explain-model-behavior"></a>Interpréter et expliquer le comportement d’un modèle

Les systèmes de boîte opaque peuvent être problématiques, car il est difficile pour les parties prenantes, telles que les développeurs système, les régulateurs, les utilisateurs et les décideurs de comprendre pourquoi les systèmes prennent certaines décisions. Certains systèmes d’intelligence artificielle sont plus faciles à expliquer que d’autres, et ils résultent parfois d’un compromis entre l’exactitude et la transparence.

Pour créer des systèmes d’intelligence artificielle interprétables, utilisez [InterpretML](https://github.com/interpretml/interpret), un package open source créé par Microsoft. Vous pouvez [utiliser InterpretML à l’intérieur d’Azure Machine Learning](how-to-machine-learning-interpretability.md) pour [interpréter et expliquer vos modèles d’apprentissage automatique](how-to-machine-learning-interpretability-aml.md), dont des [modèles d’apprentissage automatique automatisé](how-to-machine-learning-interpretability-automl.md).

## <a name="mitigate-fairness-in-machine-learning-models"></a>Atténuer l’impartialité dans les modèles Machine Learning

Les systèmes d’intelligence artificielle étant de plus en plus mis à contribution pour les prises de décision quotidiennes, il est extrêmement important qu’ils fonctionnent bien pour fournir des résultats impartiaux ne lésant personne.

Toute partialité des systèmes d’intelligence artificielle peut entraîner les conséquences inattendues suivantes :

- Rétention d’opportunités, de ressources ou d’informations au détriment de personnes.
- Renforcement de biais et stéréotypes.

De nombreux aspects de l’impartialité ne peuvent pas être capturés ou représentés par des métriques. Il existe des outils et pratiques qui peuvent renforcer l’impartialité en lien avec la conception et le développement de systèmes d’intelligence artificielle.

L’évaluation et l’atténuation sont deux étapes clés de l’atténuation de la partialité des systèmes d’intelligence artificielle. Nous vous recommandons d’utiliser [FairLearn](https://github.com/fairlearn/fairlearn), un package open source capable d’évaluer et d’atténuer la partialité potentielle des systèmes d’intelligence artificielle. Pour en savoir plus sur l’impartialité et le package FairLearn, consultez l’[article sur l’impartialité de l’apprentissage automatique](./concept-fairness-ml.md).

## <a name="prevent-data-exposure-with-differential-privacy"></a>Empêcher l’exposition des données avec la confidentialité différentielle

Lorsque des données sont utilisées à des fins d’analyse, il est important qu’elles restent privées et confidentielles tout au long de leur utilisation. La confidentialité différentielle est un ensemble de systèmes et pratiques qui permettent de préserver la sécurité et la confidentialité des données des personnes.

Dans les scénarios traditionnels, les données brutes sont stockées dans des fichiers et des bases de données. Lorsque des utilisateurs analysent des données, ils utilisent généralement des données brutes. C’est problématique, car cela peut entraîner une violation de données à caractère personnel. La confidentialité différentielle tente de résoudre ce problème en ajoutant du « bruit » ou une composante aléatoire aux données afin que les utilisateurs ne puissent pas identifier des points de données individuels.

L’implémentation de systèmes de confidentialité différentielle est difficile. [WhiteNoise](https://github.com/opendifferentialprivacy/whitenoise-core) est un projet open source contenant différents composants pour la création de systèmes globaux assortis d’une confidentialité différentielle. Pour en savoir plus sur la confidentialité différentielle et le projet WhiteNoise, consultez l’article [Préserver la confidentialité des données à l’aide de la confidentialité différentielle et de WhiteNoise](./concept-differential-privacy.md).

> [!NOTE]
> Veuillez noter que nous allons renommer le kit de ressources et présenterons le nouveau nom dans les semaines à venir. 

## <a name="work-on-encrypted-data-with-homomorphic-encryption"></a>Travailler sur des données chiffrées à l’aide du chiffrement homomorphe

Dans les solutions traditionnelles de calcul et de stockage cloud, le cloud doit disposer d’un accès non chiffré aux données client pour le calcul. Cet accès expose les données aux opérateurs de cloud. La confidentialité des données repose sur des stratégies de contrôle d’accès implémentées par le cloud et auxquelles le client fait confiance.

Le chiffrement homomorphe permet d’effectuer des calculs sur des données chiffrées sans avoir à accéder à une clé secrète (déchiffrement). Les résultats des calculs sont chiffrés et peuvent être révélés uniquement par le propriétaire de la clé secrète. En utilisant le chiffrement homomorphe, les opérateurs de cloud n’auront jamais un accès non chiffré aux données qu’ils stockent et calculent. Les calculs sont effectués directement sur les données chiffrées. La confidentialité des données repose sur un chiffrement de pointe, et le propriétaire des données contrôle toutes les informations diffusées. Pour plus d’informations sur le chiffrement homomorphe chez Microsoft, consultez [Microsoft Research](https://www.microsoft.com/research/project/homomorphic-encryption/).

Pour commencer à utiliser le chiffrement homomorphe dans Azure Machine Learning, utilisez les liaisons Python [à inférence chiffrée](https://pypi.org/project/encrypted-inference/) pour [Microsoft SEAL](https://github.com/microsoft/SEAL). Microsoft SEAL est une bibliothèque de chiffrement homomorphe open source qui permet d’effectuer des ajouts et des multiplications sur des entiers chiffrés ou des nombres réels. Pour en savoir plus sur Microsoft SEAL, accédez à [Centre des architectures Azure](/azure/architecture/solution-ideas/articles/homomorphic-encryption-seal) ou consultez la [page de projet Microsoft Research](https://www.microsoft.com/research/project/microsoft-seal/).

Consultez l’exemple suivant pour apprendre à [déployer un service web d’inférence chiffré dans Azure Machine Learning](how-to-homomorphic-encryption-seal.md).

## <a name="document-the-machine-learning-lifecycle-with-datasheets"></a>Documenter le cycle de vie de l’apprentissage automatique avec des feuilles de données

La documentation des informations adéquates dans le processus d’apprentissage automatique est la clé de la prise de décisions responsables à chaque étape. Les feuilles de données sont un moyen de documenter les ressources d’apprentissage automatique utilisées et créées dans le cadre du cycle de vie de l’apprentissage automatique.

Il existe une tendance consistant à considérer les modèles comme des « boîtes opaques », qui a pour effet qu’il y a souvent peu d’informations à leur sujet. Les systèmes d’apprentissage automatique intervenant de plus en plus dans les prises de décisions, l’utilisation de feuilles de données est une étape vers le développement de systèmes d’apprentissage automatique plus responsables.

Voici des informations sur le modèle qu’il peut être utile de documenter dans une feuille de données :

- Usage prévu
- Architecture du modèle
- Données d’apprentissage utilisées
- Données d’évaluation utilisées
- Métriques de performances du modèle d’apprentissage
- Informations sur l’impartialité

Pour apprendre à utiliser le Kit de développement logiciel (SDK) Azure Machine Learning pour implémenter des [feuilles de données pour les modèles](https://github.com/microsoft/MLOps/blob/master/pytorch_with_datasheet/model_with_datasheet.ipynb), consultez l’exemple suivant.

## <a name="additional-resources"></a>Ressources supplémentaires

- Pour plus d’informations et en savoir plus sur les meilleures pratiques, consultez la [boîte à outils de l’innovation responsable](/azure/architecture/guide/responsible-innovation/).
- En savoir plus sur l’ensemble de recommandations [À PROPOS DE L’APPRENTISSAGE AUTOMATIQUE](https://www.partnershiponai.org/about-ml/) pour la documentation de système d’apprentissage automatique.