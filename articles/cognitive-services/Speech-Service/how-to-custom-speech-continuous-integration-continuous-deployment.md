---
title: CI/CD pour Custom Speech - Service Speech
titleSuffix: Azure Cognitive Services
description: Appliquez DevOps avec des workflows Custom Speech et CI/CD. Implémentez une solution DevOps existante pour votre propre projet.
services: cognitive-services
author: KatieProchilo
manager: cmayomsft
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/09/2020
ms.author: kaprochi
ms.openlocfilehash: f82ea154d5949f4d229ac76e7a7ce2a89d15ac13
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025665"
---
# <a name="cicd-for-custom-speech"></a>CI/CD pour Custom Speech

Implémentez une formation automatisée, des tests et une gestion des mises en production pour permettre l’amélioration continue des modèles Custom Speech lorsque vous appliquez des mises à jour à des données d’apprentissage et de test. Grâce à l’implémentation efficace de workflows CI/CD, vous pouvez vous assurer que le point de terminaison pour le modèle Custom Speech le plus performant est toujours disponible.

[L’intégration continue (CI)](/azure/devops/learn/what-is-continuous-integration) est la pratique d’ingénierie consistant à valider fréquemment des mises à jour dans un référentiel partagé et à effectuer une génération automatisée sur celle-ci. Les workflows CI pour Custom Speech forment un nouveau modèle à partir de ses sources de données et effectuent des tests automatisés sur le nouveau modèle pour s’assurer qu’il fonctionne mieux que le modèle précédent.

[La livraison continue](/azure/devops/learn/what-is-continuous-delivery) (CD) sélectionne des modèles à partir du processus CI et crée un point de terminaison pour chaque modèle Custom Speech amélioré. Un workflow CD permet d’intégrer facilement des points de terminaison à des solutions.

Des solutions CI/CD personnalisées sont possibles, mais pour bénéficier d’une solution robuste et prédéfinie, utilisez le [référentiel de modèles DevOps Speech](https://github.com/Azure-Samples/Speech-Service-DevOps-Template), qui exécute les workflows CI/CD à l’aide d’actions GitHub.

## <a name="cicd-workflows-for-custom-speech"></a>Workflows CI/CD pour Custom Speech

L'objectif de ces workflows est de garantir que chaque modèle Custom Speech offre une meilleure précision de reconnaissance que la version précédente. Si les mises à jour des données de test et/ou d’apprentissage améliorent la précision, ces workflows créent un nouveau point de terminaison Custom Speech.

Les serveurs Git tels que GitHub et Azure DevOps peuvent exécuter des workflows automatisés quand des événements Git spécifiques se produisent, notamment des fusions ou des demandes de tirage. Par exemple, un workflow CI peut être déclenché lorsque des mises à jour de données de test sont envoyées à la branche *principale*. Différents serveurs Git proposeront différents outils, mais autoriseront les commandes de l’interface de ligne de commande (CLI) de script afin qu’elles puissent s’exécuter sur un serveur de builds.

En chemin, les workflows doivent nommer et stocker des données, des tests, des fichiers de test, des modèles et des points de terminaison, de sorte qu’ils puissent remonter jusqu’à la validation ou à la version dont ils proviennent. Il est également utile de nommer ces ressources pour identifier facilement celles créées après la mise à jour des données de test par rapport aux données d’apprentissage.

### <a name="ci-workflow-for-testing-data-updates"></a>Workflow CI pour les mises à jour de données de test

Le principal objectif des workflows CI/CD est de créer un nouveau modèle à l’aide des données d’apprentissage, et de tester ce modèle à l’aide des données de test pour déterminer si le [taux d’erreur de mots](how-to-custom-speech-evaluate-data.md#evaluate-custom-speech-accuracy) (WER) a été amélioré par rapport au modèle le plus performant précédent le (« modèle de référence »). Si le nouveau modèle est plus performant, il devient le nouveau modèle de référence auquel les futurs modèles seront comparés.

Le workflow CI pour les mises à jour de données de test doit tester à nouveau le modèle de référence actuel avec les données de test mises à jour afin de calculer le taux WER modifié. Cela garantit que lorsque le taux WER d’un nouveau modèle est comparé à celui du modèle de référence, les deux modèles ont été testés avec les mêmes données de test et comparés avec les mêmes critères.

Ce workflow doit se déclencher sur les mises à jour des données de test et :

- Tester le modèle de référence sur les données de test mises à jour.
- Stocker le résultat du test, qui inclut le taux WER du modèle de référence, en utilisant les données mises à jour.
- Le taux WER issu de ces tests deviendra le nouveau taux WER de référence que les futurs modèles devront dépasser.
- Le workflow CD ne s'exécute pas pour les mises à jour des données de test.

### <a name="ci-workflow-for-training-data-updates"></a>Workflow CI pour les mises à jour de données d’apprentissage

Mettre à jour les données d’apprentissage signifie mettre à jour le modèle personnalisé.

Ce workflow doit se déclencher sur les mises à jour des données d’apprentissage et :

- Former un nouveau modèle avec les données d’apprentissage mises à jour.
- Tester le nouveau modèle avec les données de test.
- Stocker le résultat du test, qui contient le taux WER.
- Comparer le taux WER du nouveau modèle au taux WER du modèle de référence.
- Si le taux WER ne s’améliore pas, arrêter le workflow.
- Si le taux WER s’améliore, exécuter le workflow CD pour créer un point de terminaison Custom Speech.

### <a name="cd-workflow"></a>Workflow CD

Si une mise à jour des données d’apprentissage améliore la reconnaissance d’un modèle, le workflow CD doit s’exécuter automatiquement pour créer un nouveau point de terminaison pour ce modèle, et rendre ce point de terminaison disponible de sorte qu’il puisse être utilisé dans une solution.

#### <a name="release-management"></a>Gestion des mises en production

La plupart des équipes requièrent un processus de révision et d’approbation manuel pour le déploiement dans un environnement de production. Pour un déploiement de production, vous souhaiterez peut-être vous assurer qu’il se produit lorsque des personnes clés de l’équipe de développement sont disponibles pour assurer un support, ou pendant des périodes de faible trafic.

### <a name="tools-for-custom-speech-workflows"></a>Outils pour workflows Custom Speech

Utilisez les outils suivants pour les workflows d’automatisation CI/CD pour Custom Speech :

- [L’interface de ligne de commande Azure](/cli/azure/?view=azure-cli-latest), pour créer une authentification du principal du service Azure, interroger les abonnements Azure, et stocker les résultats des tests dans Azure Blob.
- [L’interface de ligne de commande Azure Speech](spx-overview.md), pour interagir avec le service Speech à partir de la ligne de commande ou d’un workflow automatisé.

## <a name="devops-solution-for-custom-speech-using-github-actions"></a>Solution DevOps pour Custom Speech à l’aide d’actions GitHub

Avec une solution DevOps déjà implémentée pour Custom Speech, accédez au [modèle de référentiel Speech DevOps](https://github.com/Azure-Samples/Speech-Service-DevOps-Template). Créez une copie du modèle et commencez le développement de modèles personnalisés avec un système DevOps robuste, incluant les tests, l’apprentissage et le contrôle de version à l’aide d’actions GitHub. Le référentiel fournit des exemples de données de test et d’apprentissage pour vous aider à configurer et à expliquer le workflow. Après l’installation initiale, remplacez les exemples de données par les données de votre projet.

Le [modèle de référentiel Speech DevOps](https://github.com/Azure-Samples/Speech-Service-DevOps-Template) fournit l’infrastructure et des conseils détaillés pour :

- Copier le modèle de référentiel dans votre compte GitHub puis créer des ressources Azure et un [principal de service](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) pour les workflows CI/CD Actions GitHub.
- Passer en revue la « [boucle interne de développement](https://mitchdenny.com/the-inner-loop/) ». Mettre à jour les données d’apprentissage et de test à partir d’une branche de fonctionnalités, tester les modifications avec un modèle de développement temporaire, et déclencher une demande de tirage pour proposer et examiner les modifications.
- Lorsque les données d’apprentissage sont mises à jour dans une demande de tirage pour une branche *principale*, former les modèles avec le workflow CI Actions GitHub.
- Effectuer des tests d’exactitude automatisés pour établir le [taux d’erreur de mots](how-to-custom-speech-evaluate-data.md#evaluate-custom-speech-accuracy) (WER) d’un modèle. Stocker les résultats des tests dans Azure Blob.
- Exécuter le workflow CD pour créer un point de terminaison lorsque le taux WER s’améliore.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur DevOps avec Speech :

- Utilisez le [modèle de référentiel DevOps Speech](https://github.com/Azure-Samples/Speech-Service-DevOps-Template) afin d’implémenter DevOps pour Custom Speech avec des actions GitHub.