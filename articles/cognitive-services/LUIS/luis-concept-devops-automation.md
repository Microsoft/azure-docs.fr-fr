---
title: Workflows continus pour les applications LUIS
description: Implémenter des flux de travail CI/CD pour DevOps pour Language Understanding (LUIS).
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 06/5/2020
ms.openlocfilehash: 1ce78e02c652777b524964559b579530f3e022fa
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561305"
---
# <a name="continuous-integration-and-continuous-delivery-workflows-for-luis-devops"></a>Flux de travail d’intégration continue et de livraison continue pour LUIS DevOps

Les ingénieurs logiciels qui développent une application Language Understanding (LUIS) peuvent appliquer des pratiques DevOps sur le [contrôle de code source](luis-concept-devops-sourcecontrol.md), [les builds automatisés](luis-concept-devops-automation.md), les [tests](luis-concept-devops-testing.md) et la [gestion des mises en production](luis-concept-devops-automation.md#release-management). Cet article décrit les concepts d’implémentation de builds automatisés pour LUIS.

## <a name="build-automation-workflows-for-luis"></a>Générer des flux de travail d’automatisation pour LUIS

![Flux de travail en CI](./media/luis-concept-devops-automation/luis-automation.png)

Dans votre gestionnaire de contrôle des services (GCL), configurez les pipelines de build automatisés pour qu’ils s’exécutent aux événements suivants :

1. Le **flux de travail PR** est déclenché lorsqu’une [demande de tirage](https://help.github.com/github/collaborating-with-issues-and-pull-requests/about-pull-requests) (PR) est exécutée. Ce flux de travail valide le contenu du *PR avant que* les mises à jour soient fusionnées dans la branche principale.
1. Le **flux de travail CI/CD** est déclenché lorsque des mises à jour sont envoyées à la branche principale, par exemple lors de la fusion des modifications à partir d’une demande de tirage. Ce flux de travail garantit la qualité de toutes les mises à jour de la branche principale.

Le **flux de travail CI/CD** combine deux processus de développement complémentaires :

* [L’intégration continue (CI)](/azure/devops/learn/what-is-continuous-integration) est la pratique d’ingénierie consistant à valider fréquemment le code dans un référentiel partagé et à effectuer une génération automatisée sur celle-ci. Associé à une approche de [test](luis-concept-devops-testing.md) automatisé, l’intégration continue nous permet de vérifier que, pour chaque mise à jour, la source LUDown est toujours valide et peut être importée dans une application LUIS, mais également qu’elle réussit les tests qui vérifient que l’application entraînée peut reconnaître les intentions et les entités requises pour votre solution.

* La [livraison continue](/azure/devops/learn/what-is-continuous-delivery) (CD) permet au concept d’intégration continue de déployer automatiquement l’application dans un environnement de tests plus approfondis. Grâce à la livraison continue, nous pouvons découvrir rapidement tous les problèmes imprévus qui résultent des modifications apportées, mais aussi en savoir plus sur les potentielles failles de nos tests.

L’objectif de l’intégration continue et de la livraison continue est de s’assurer que la branche principale est toujours accessible. Pour une application LUIS, cela signifie que nous pourrions, si nécessaire, prendre n’importe quelle version de l’application de la branche principale LUIS et la livrer en production.

### <a name="tools-for-building-automation-workflows-for-luis"></a>Outils de génération de flux de travail automatisés pour LUIS

Il existe différentes technologies de génération d’automatisation pour créer ce type de flux de travail. Il est nécessaire de pouvoir écrire des étapes à l’aide d’une interface de ligne de commande (CLI) ou d’appels REST pour que tous ces outils puissent s’exécuter sur un serveur de génération de flux.

Utiliser les outils de génération de flux de travail automatisés suivants pour LUIS :

* [Interface CLI LUIS des outils de l’infrastructure du chatbot](https://github.com/microsoft/botbuilder-tools/tree/master/packages/LUIS) pour fonctionner avec les applications et versions LUIS (apprentissage, test) et les publier dans le service LUIS.

* [Interface de ligne de commande Azure](/cli/azure/?view=azure-cli-latest) pour interroger des abonnements Azure, récupérer des clés de prédiction et de création de LUIS, et pour créer un [principal de service Azure](/cli/azure/ad/sp?view=azure-cli-latest) utilisé pour l’authentification du flux de travail d’automatisation.

* Outil [NLU.DevOps](https://github.com/microsoft/NLU.DevOps) pour [tester une application LUIS](luis-concept-devops-testing.md) et analyser les résultats des tests.

### <a name="the-pr-workflow"></a>Flux de travail d’une demande de tirage (PR)

Comme nous l’avons vu précédemment, vous pouvez configurer ce flux de travail pour qu’il s’exécute lorsqu’un développeur déclenche une demande de tirage (PR) afin de fusionner les modifications apportées d’une branche secondaire vers la branche principale. Le but étant de vérifier la qualité des modifications de la demande de tirage avant de les fusionner dans la branche principale.

Ce flux de travail peut :

* Créer une application LUIS temporaire en important la `.lu`source dans la demande de tirage.
* Effectuer l’apprentissage de l’application LUIS puis la publier.
* Exécuter tous les [tests unitaires](luis-concept-devops-testing.md).
* Valider le flux de travail s’il réussit tous les tests ou le bloquer si ce n’est pas le cas.
* Nettoyer et supprimer l’application temporaire.

Si votre gestionnaire de contrôle des services prend cette fonctionnalité en charge, configurez les règles de protection de la branche pour vous assurer que le flux de travail soit complètement validé avant que la demande de tirage ne soit terminée.

### <a name="the-main-branch-cicd-workflow"></a>Flux de travail CI/CD de la branche principale

Configurez ce flux de travail pour qu’il s’exécute après la fusion des mises à jour de la PR dans la branche principale. L’objectif est de conserver une qualité optimale de votre branche principale en testant chaque mise à jour. Si les mises à jour sont conformes à la qualité attendue, ce flux de travail déploie la nouvelle version de l’application LUIS dans un environnement où vous pouvez effectuer des tests plus approfondis.

Ce flux de travail peut :

* Générer une nouvelle version dans votre application LUIS principale (l’application que vous gérez pour la branche principale) à l’aide du code source mis à jour.

* Effectuer l’apprentissage de l’application LUIS puis la publier.

  > [!NOTE]
  > Comme expliqué dans la section [Exécution de tests dans un flux de travail de génération automatisé](luis-concept-devops-testing.md#running-tests-in-an-automated-build-workflow) vous devez publier la version de l’application LUIS en cours de test afin d’utiliser des outils tels que NLU.DevOps. LUIS prend uniquement en charge deux emplacements de publication nommés : *mise en lot* et *production* pour une application LUIS. Vous pouvez également [publier une version directement](https://github.com/microsoft/botframework-cli/blob/master/packages/luis/README.md#bf-luisapplicationpublish) et [faire une requête par version](./luis-migration-api-v3.md#changes-by-slot-name-and-version-name). Utilisez la publication de version directe dans vos flux de travail d’automatisation pour éviter d’être limité à l’utilisation des emplacements de publication nommés.

* Exécutez tous les [tests unitaires](luis-concept-devops-testing.md).

* Si vous le souhaitez, vous pouvez exécuter des [tests par lot](luis-concept-devops-testing.md#how-to-do-unit-testing-and-batch-testing) pour mesurer la qualité et la précision de la version de l’application LUIS et la comparer à une ligne de base.

* Si les tests sont validés :
  * Balisez la source dans le référentiel.
  * Exécutez la tâche de livraison continue (CD) pour déployer la version de l’application LUIS dans les environnements pour effectuer d’autres tests.

### <a name="continuous-delivery-cd"></a>Déploiement continu (CD)

La tâche CD dans un flux de travail CI/CD s’exécute de manière conditionnelle en cas de réussite de la génération et des tests unitaires automatisés. Son travail consiste à déployer automatiquement l’application LUIS dans un environnement où vous pouvez effectuer d’autres tests.

Il n’existe pas de solution recommandée pour déployer votre application LUIS, vous devez implémenter le processus adapté à votre projet. Le référentiel de [modèle LUIS DevOps](https://github.com/Azure-Samples/LUIS-DevOps-Template) implémente une solution simple pour cela, qui consiste à [publier la nouvelle version de l’application LUIS](./luis-how-to-publish-app.md) dans l’emplacement de publication de *production*. Cela convient parfaitement pour une configuration simple. Toutefois, si vous avez besoin de prendre en charge un certain nombre d’environnements de production différents en même temps, par exemple de *développement*, de *mise en lots* et d’*UAT*, la limite de deux emplacements de publication nommés par application s’avère insuffisante.

Il existe d'autres options pour déployer une version de l'application :

* Laisser la version publiée de l'application au point de terminaison de la version directe et mettre en œuvre un processus pour configurer les environnements de production en aval avec le point de terminaison de la version directe selon les besoins.
* Maintenir différentes applications LUIS pour chaque environnement de production et écrire des étapes d'automatisation pour importer `.lu` dans une nouvelle version de l’application LUIS pour l'environnement de production cible, l’alimenter et la publier.
* Exporter la version testée de l'application LUIS dans un [conteneur docker LUIS](./luis-container-howto.md?tabs=v3) et déployer le conteneur LUIS dans les [instances de conteneur](../../container-instances/index.yml) d’Azure.

## <a name="release-management"></a>Gestion des mises en production

En règle générale, nous vous recommandons d'effectuer des livraisons continues uniquement vers vos environnements de non-production, par exemple pour le développement et la mise en lots. La plupart des équipes requièrent un processus de révision et d’approbation manuel pour le déploiement dans un environnement de production. Pour un déploiement de production, vous souhaiterez peut-être vous assurer qu’il se produit lorsque des personnes clés de l’équipe de développement sont disponibles pour assurer un support, ou pendant des périodes de faible trafic.

## <a name="next-steps"></a>Étapes suivantes

* Comment [implémenter DevOps pour LUIS avec GitHub](luis-how-to-devops-with-github.md)
* Comment écrire un [flux de travail GitHub Actions avec NLU.DevOps](https://github.com/Azure-Samples/LUIS-DevOps-Template/blob/master/docs/4-pipeline.md)