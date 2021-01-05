---
title: Forum aux questions sur le Modèle Resource Manager
description: Questions fréquentes (FAQ) sur les modèles Azure Resource Manager (modèles ARM).
ms.topic: conceptual
ms.date: 09/17/2020
ms.author: tomfitz
author: tfitzmac
ms.openlocfilehash: 757dfa119649e21c6e1787671afa61ee18184300
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561900"
---
# <a name="frequently-asked-questions-about-arm-templates"></a>Forum aux questions sur les modèles Resource Manager

Cet article répond aux questions fréquentes sur les modèles Azure Resource Manager (modèles ARM).

## <a name="getting-started"></a>Prise en main

* **Que sont les modèles Resource Manager et pourquoi dois-je les utiliser ?**

  Les modèles Resource Manager sont des fichiers JSON dans lesquels vous définissez ce que vous souhaitez déployer sur Azure. Les modèles vous aident à implémenter une solution d’infrastructure en tant que code pour Azure. Votre organisation peut déployer de manière répétée et fiable l’infrastructure requise dans différents environnements.

  Pour en savoir plus sur la façon dont les modèles Resource Manager vous aident à gérer votre infrastructure Azure, consultez [Qu’est-ce qu’un modèle Resource Manager ?](overview.md)

* **Comment faire pour commencer à utiliser les modèles ?**

  Pour simplifier la création de modèles Resource Manager, vous avez besoin des bons outils. Nous vous recommandons d’installer [Visual Studio Code](https://code.visualstudio.com/) et [l’extension Azure Resource Manager Tools](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools). Pour une présentation rapide de ces outils, consultez [Démarrage rapide : Créer des modèles ARM avec Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

  Quand vous êtes prêt à en savoir plus sur la création de modèles Resource Manager, commencez la [série de tutoriels pour débutants sur les modèles ARM](template-tutorial-create-first-template.md). Ces tutoriels vous guident pas à pas dans le processus de construction d’un modèle Resource Manager. Découvrez les différentes sections du modèle et comment elles fonctionnent ensemble. Ce contenu est également disponible sous la forme d’un [module Microsoft Learn](/learn/modules/authoring-arm-templates/).

* **Dois-je utiliser des modèles Resource Manager ARM ou Terraform pour effectuer un déploiement sur Azure ?**

  Utilisez l’option qui vous convient le mieux. Les deux services vous aident à automatiser les déploiements sur Azure.

  Nous pensons qu’il y a des avantages à utiliser des modèles Resource Manager sur d’autres services d’infrastructure en tant que code. Pour en savoir plus sur ces avantages, consultez [Pourquoi choisir des modèles Resource Manager ?](overview.md#why-choose-arm-templates)

## <a name="build-2020"></a>Build 2020

* **J’ai manqué votre présentation de Microsoft Build 2020. La présentation est-elle disponible en replay ?**

  Oui, nous vous invitons à [la regarder à tout moment](https://mybuild.microsoft.com/sessions/82984db4-37a4-4ed3-bf8b-13298841ed18?source=sessions).

* **Où puis-je obtenir plus d’informations sur les nouvelles fonctionnalités que vous avez annoncées à Build ?**

  Pour obtenir des informations générales sur les fonctionnalités que nous développons, rejoignez notre groupe [Azure Advisors Deployments Yammer](https://aka.ms/ARMMeet).

  Pour en savoir plus sur le nouveau langage de modèle, [inscrivez-vous aux notifications](https://aka.ms/armLangUpdates).

  Pour en savoir plus sur les specs de modèles, consultez [Specs de modèles Azure Resource Manager (préversion)](template-specs.md).

## <a name="creating-and-testing-templates"></a>Créer et tester des modèles

* **Où puis-je en savoir plus sur les meilleures pratiques en matière de modèles Resource Manager ?**

  Pour obtenir des suggestions sur l’implémentation de modèles, consultez [Bonnes pratiques relatives aux modèles Resource Manager](template-best-practices.md). Après avoir créé un modèle, exécutez le [kit à outils de test Resource Manager](https://github.com/azure/arm-ttk). Il vérifie si votre modèle correspond aux pratiques recommandées.

* **J’ai configuré mon environnement par le biais du portail. Existe-t-il un moyen de récupérer le modèle à partir d’un groupe de ressources existant ?**

  Oui, vous pouvez [exporter le modèle](export-template-portal.md) à partir d’un groupe de ressources. Le modèle exporté est un bon point de départ pour en savoir plus sur les modèles, mais vous voudrez probablement le modifier avant de l’utiliser dans un environnement de production.

  Lorsque vous exportez le modèle, vous pouvez sélectionner les ressources que vous souhaitez inclure dans le modèle.

* **Puis-je créer un groupe de ressources dans un modèle Resource Manager et y déployer des ressources ?**

  Oui, vous pouvez créer un groupe de ressources dans un modèle lorsque vous déployez le modèle au niveau de votre abonnement Azure. Pour obtenir un exemple de création d’un groupe de ressources et de déploiement de ressources, consultez [Groupe de ressources et ressources](deploy-to-subscription.md#resource-groups).

* **Puis-je créer un abonnement dans un modèle Resource Manager ?**

  Oui. Pour plus d’informations, consultez [Créer des abonnements Azure programmatiquement avec les dernières API](../../cost-management-billing/manage/programmatically-create-subscription-enterprise-agreement.md).

* **Comment puis-je tester mon modèle avant de le déployer ?**

  Nous vous recommandons d’exécuter le [kit à outils de test Resource Manager](https://github.com/azure/arm-ttk) et l’opération [de simulation](template-deploy-what-if.md) sur vos modèles avant de les déployer. Le kit à outils de test vérifie si votre modèle utilise les meilleures pratiques. Il fournit des avertissements lorsqu’il identifie des modifications qui peuvent améliorer la façon dont vous avez implémenté votre modèle.

  L’opération de simulation affiche les modifications apportées par votre modèle à votre environnement. Vous pouvez voir les modifications inattendues avant qu’elles ne soient déployées. What-if permet également de renvoyer les erreurs détectées lors de la validation avant le lancement. Par exemple, si votre modèle contient une erreur syntaxique, il retournera cette erreur. Il retournera également toutes les erreurs qu’il peut déterminer à propos de l’état final des ressources déployées. Par exemple, si votre modèle déploie un compte de stockage avec un nom qui est déjà utilisé, il retournera cette erreur.

* **Où puis-je trouver des informations sur les propriétés qui sont disponibles pour chaque type de ressource ?**

  VS Code fournit IntelliSense pour l’utilisation des propriétés de ressource. Vous pouvez également afficher les [références du modèle](/azure/templates/) pour les propriétés et les descriptions.

* **J’ai besoin de créer plusieurs instances de types de ressource. Comment créer un itérateur dans mon modèle ?**

  Utilisez l’élément copy pour spécifier plusieurs instances. Vous pouvez également utiliser l’élément copy avec les éléments [ressources](copy-resources.md), [propriétés](copy-properties.md), [variables](copy-variables.md) et [sorties](copy-outputs.md).

## <a name="template-language"></a>Langage de modèle

* **J’ai entendu dire que vous travaillez sur un nouveau langage de modèle. Où puis-je en savoir plus ?**

  Pour afficher un aperçu du nouveau langage, consultez le [dépôt du projet Bicep](https://github.com/Azure/bicep). Pour rester informé au sujet du nouveau langage, [inscrivez-vous aux notifications](https://aka.ms/armLangUpdates).

* **Existe-t-il un plan de prise en charge de la création de modèles dans YAML ?**

  Actuellement, il n’existe aucun plan pour prendre en charge YAML. Nous pensons que le nouveau langage de modèle offrira une solution plus facile à utiliser que YAML ou JSON.

* **Puis-je toujours écrire des modèles dans JSON une fois que le nouveau langage de modèle a été publié ?**

  Oui, vous pouvez continuer à utiliser des modèles JSON.

* **Proposerez-vous un outil pour convertir mes modèles JSON dans le nouveau langage de modèle ?**

  Oui.

## <a name="template-specs"></a>Specs de modèle

* **Comment puis-je démarrer avec la préversion des specs de modèle ?**

  Installez la version la plus récente de PowerShell ou d’Azure CLI. Pour Azure PowerShell, utilisez la [version 5.0.0 ou ultérieure](/powershell/azure/install-az-ps). Pour Azure CLI, utilisez la [version 2.14.2 ou ultérieure](/cli/azure/install-azure-cli).

* **Comment les spécifications de modèle et Azure Blueprints sont-ils liés ?**

  Azure Blueprints utilisera les spécifications de modèle dans son implémentation en remplaçant la ressource `blueprint definition` par une ressource `template spec`. Nous allons fournir un chemin de migration pour convertir la définition de blueprint en spec de modèle, mais les API de définition de blueprint seront toujours prises en charge. Il n'y a aucune modification de la ressource `blueprint assignment`. Les blueprints resteront une expérience utilisateur pour composer un environnement régi dans Azure.

* **Les spécifications de modèle remplacent-elles les modèles liés ?**

  Non, mais les spécifications de modèle sont conçues pour fonctionner correctement avec les modèles liés. Vous n’avez pas besoin de déplacer le modèle lié vers un point de terminaison accessible publiquement avant de déployer le modèle parent. Au lieu de cela, vous regroupez le modèle parent et ses artefacts lors de la création de la spécification du modèle.

* **Les spécifications de modèle peuvent-elles être partagées entre les abonnements ?**

  Oui, elles peuvent être utilisés dans les abonnements tant que l’utilisateur dispose d’un accès en lecture aux spécifications du modèle. Les spécifications de modèle ne peuvent pas être utilisées dans les locataires.

## <a name="scripts-in-templates"></a>Scripts dans les modèles

* **Puis-je inclure un script dans mon modèle pour effectuer des tâches qui ne sont pas possibles dans un modèle ?**

  Oui, utilisez pour cela des [scripts de déploiement](deployment-script-template.md). Vous pouvez inclure des scripts Azure PowerShell ou Azure CLI dans vos modèles. Cette fonctionnalité est en version préliminaire.

* **Puis-je continuer à utiliser les extensions de script personnalisées et DSC (Desired State Configuration) ?**

  Ces options sont toujours disponibles et n’ont pas changé. Les scripts de déploiement sont conçus pour effectuer des actions qui ne sont pas liées à l’invité de la machine virtuelle. Si vous devez exécuter un script sur un système d’exploitation hôte dans une machine virtuelle, l’extension de script personnalisé et/ou DSC est un meilleur choix. Toutefois, les scripts de déploiement présentent des avantages, tels que la définition de la durée du délai d’attente.

* **Les scripts de déploiement sont-ils pris en charge dans Azure Government ?**

  Oui, vous pouvez utiliser des scripts de déploiement dans US Gov Arizona et US Gov Virginie.

## <a name="preview-changes-before-deployment"></a>Aperçu des modifications avant le déploiement

* **Puis-je obtenir un aperçu des modifications qui se produiront avant de déployer un modèle ?**

  Oui, utilisez [la fonctionnalité de simulation ](template-deploy-what-if.md). Elle évalue l’état actuel de votre environnement et le compare à l’état qui existera après le déploiement. Vous pouvez examiner les modifications récapitulatives pour vous assurer que le modèle n’a pas de résultats inattendus.

* **Puis-je utiliser What-if avec les modes incrémentiel et complet ?**

  Oui, les [deux modes de déploiement ](deployment-modes.md) sont pris en charge. Pour obtenir un exemple d’utilisation du mode incrémentiel, consultez [Exécuter une opération de simulation](template-deploy-what-if.md#run-what-if-operation). Pour obtenir un exemple d’utilisation du mode complet, consultez [Confirmer la suppression](template-deploy-what-if.md#confirm-deletion).

* **What-if fonctionne-t-il avec des modèles liés ?**

  Oui, What-if évalue l’état du modèle parent et de ses modèles liés.

* **Puis-je utiliser What-if dans un pipeline Azure ?**

  Oui, vous pouvez utiliser What-if qui permet de vérifier que le pipeline doit continuer.

* **Quand j’utilise What-if, je vois des modifications dans les propriétés qui ne sont pas dans mon modèle. Ce « parasite » est-:il prévu ?**

  Nous travaillons actuellement à la réduction des parasites. Aidez-nous à nous améliorer en nous signalant les problèmes dans notre référentiel GitHub ici : https://aka.ms/WhatIfIssues

## <a name="template-visualizer"></a>Visualiseur de modèle

* **Est-il possible de visualiser mon modèle Resource Manager et ses ressources ?**

  Nous avons une [extension de VS Code alimentée par la communauté](https://aka.ms/ARMVisualizer) qui est un excellent outil de visualisation de votre modèle Resource Manager. Elle montre les ressources que vous déployez et les relations entre elles.

* **Puis-je utiliser le visualiseur de modèle en dehors de VS Code ?**

  Le visualiseur de modèle est en cours d’aperçu dans le portail. Pour plus d’informations, regardez cette [brève session brève du Build](https://mybuild.microsoft.com/sessions/0525094b-1fd2-4f69-bfd6-6d2fff6ffe5f?source=sessions).

## <a name="deployment-limits"></a>Limites du déploiement

* **Combien de groupes de ressources puis-je déployer en une seule opération de déploiement ?**

  Dans le passé, cette limite était de cinq groupes de ressources. Elle est récemment passée à 800 groupes de ressources. Pour plus d’informations, consultez [Créer des groupes de ressources et des ressources au niveau de l’abonnement](deploy-to-subscription.md).

* **Une erreur est survenue dans l'historique des déploiements, qui était limité à 800. Que dois-je faire ?**

  Nous changeons la façon dont l'historique du déploiement d'un groupe de ressources est conservé. Dans le passé, vous deviez supprimer manuellement les déploiements de cet historique pour éviter cette erreur. À compter de juin 2020, les déploiements sont automatiquement supprimés de l’historique quand vous vous approchez de la limite. Pour plus d’informations, consultez [Suppressions automatiques de l’historique de déploiement](deployment-history-deletions.md).

  La suppression d’un déploiement à partir de l’historique des déploiements n’affecte pas les ressources déployées.

## <a name="templates-and-devops"></a>Modèles et DevOps

* **Puis-je intégrer des modèles ARM dans Azure Pipelines ?**

  Oui. Pour obtenir une explication de l’utilisation du modèle et des pipelines, consultez le [tutoriel : Intégration continue de modèles ARM à Azure Pipelines](deployment-tutorial-pipeline.md) et [Intégrer des modèles ARM à Azure Pipelines](add-template-to-azure-pipelines.md).

* **Puis-je utiliser GitHub Actions pour déployer un modèle ?**

  Oui. Pour cela, consultez [Déployer des modèles ARM à l’aide de GitHub Actions](deploy-github-actions.md).

## <a name="next-steps"></a>Étapes suivantes

Pour une présentation des modèles Resource Manager, consultez [Qu’est-ce qu’un modèle Resource Manager ?](overview.md).
