---
title: Concevoir des workflows Azure Policy en tant que code
description: Apprenez à concevoir des workflows pour déployer vos définitions Azure Policy grâce à du code et valider automatiquement les ressources.
ms.date: 10/20/2020
ms.topic: conceptual
ms.openlocfilehash: 74d2097e4db4442e6e65f30541864fb554f7379d
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94359678"
---
# <a name="design-azure-policy-as-code-workflows"></a>Concevoir des workflows Azure Policy en tant que code

Au fil de votre progression dans la gouvernance cloud, vous allez chercher à passer de la gestion manuelle de chacune des définitions de stratégies sur le Portail Azure ou à l’aide des différents kits de développement logiciel (SDK) à un processus plus gérable et reproductible à l’échelle de l’entreprise. Voici deux des approches prédominantes de la gestion des systèmes à grande échelle dans le cloud :

- Infrastructure as Code : pratique consistant à traiter en tant que code source tout le contenu qui définit les environnements, des modèles Azure Resource Manager (modèles ARM) aux définitions Azure Policy en passant par Azure Blueprints.
- DevOps : rassemblement des personnes, des processus et des produits qui permettent une livraison continue de valeur ajoutée aux clients finaux.

Azure Policy en tant que code est la combinaison de ces idées. Pour l’essentiel, vous conservez vos définitions de stratégies dans le contrôle de code source, puis testez et validez chaque modification effectuée. Toutefois, l’implication des stratégies avec l’Infrastructure as Code ou le DevOps ne devrait pas s’arrêter là.

L’étape de validation devrait également être un composant d’autres workflows d’intégration continue ou de déploiement continu. Citons notamment le déploiement d’un environnement d’application ou d’une infrastructure virtuelle. En faisant de la validation Azure Policy l’un des premiers composants du processus de build et de déploiement, les équipes chargées des applications et des opérations détectent si leurs modifications ne sont pas conformes bien avant qu’il ne soit trop tard et qu’il faille les déployer en production.

## <a name="definitions-and-foundational-information"></a>Définitions et informations fondamentales

Avant d’entrer dans les détails du workflow Azure Policy en tant que code, examinez les définitions et exemples suivants :

- [Définition de stratégie](./definition-structure.md)
- [Définition d’initiative](./initiative-definition-structure.md)

Les noms de fichiers s’alignent sur des parties de la définition de la stratégie ou de l’initiative :
- `policy(set).json` : la définition entière
- `policy(set).parameters.json` : la partie `properties.parameters` de la définition
- `policy.rules.json` : la partie `properties.policyRule` de la définition
- `policyset.definitions.json` : la partie `properties.policyDefinitions` de la définition

Des exemples de ces formats de fichier sont accessibles dans le [référentiel GitHub d’Azure Policy](https://github.com/Azure/azure-policy/) :

- Définition de stratégie : [Ajouter une étiquette aux ressources](https://github.com/Azure/azure-policy/tree/master/samples/Tags/add-tag)
- Définition d’initiative : [Étiquettes de facturation](https://github.com/Azure/azure-policy/tree/master/samples/PolicyInitiatives/multiple-billing-tags)

## <a name="workflow-overview"></a>Vue d’ensemble du workflow

Le workflow général recommandé d’Azure Policy en tant que code se présente comme ce diagramme :

:::image type="complex" source="../media/policy-as-code/policy-as-code-workflow.png" alt-text="Diagramme montrant les cadres du workflow Azure Policy en tant que code de la création jusqu’au déploiement, en passant par les tests." border="false":::
   Diagramme montrant les cadres du workflow Azure Policy en tant que code. « Créer » couvre la création des définitions de stratégie et d’initiative. « Tester » couvre l’attribution avec le mode d’application désactivé. Une vérification de l’état de conformité de la passerelle est suivie en accordant des autorisations MSI et en corrigeant les ressources.  « Déployer » couvre la mise à jour de l’attribution avec le mode d’application activé.
:::image-end:::

### <a name="create-and-update-policy-definitions"></a>Créer et mettre à jour des définitions de stratégies

Les définitions de stratégies sont créées avec des fichiers JSON et stockées dans le contrôle de code source. Chaque stratégie possède son propre ensemble de fichiers (paramètres, règles et paramètres d’environnement) qui doivent être stockés dans le même dossier. Nous vous recommandons la structure suivante pour conserver vos définitions de stratégies dans le contrôle de code source.

```text
.
|
|- policies/  ________________________ # Root folder for policy resources
|  |- policy1/  ______________________ # Subfolder for a policy
|     |- policy.json _________________ # Policy definition
|     |- policy.parameters.json ______ # Policy definition of parameters
|     |- policy.rules.json ___________ # Policy rule
|     |- assign.<name1>.json _________ # Assignment 1 for this policy definition
|     |- assign.<name2>.json _________ # Assignment 2 for this policy definition
|  |- policy2/  ______________________ # Subfolder for a policy
|     |- policy.json _________________ # Policy definition
|     |- policy.parameters.json ______ # Policy definition of parameters
|     |- policy.rules.json ___________ # Policy rule
|     |- assign.<name1>.json _________ # Assignment 1 for this policy definition
|     |- assign.<name2>.json _________ # Assignment 2 for this policy definition
|
```

Lorsqu’une stratégie est mise à jour ou qu’une nouvelle est ajoutée, le workflow doit automatiquement mettre à jour la définition de stratégie dans Azure. Le test de la définition de stratégie ajoutée ou mise à jour sera effectué dans une étape ultérieure.

Consultez également l’article [Exporter des ressources Azure Policy](../how-to/export-resources.md) pour obtenir vos définitions et attributions existantes dans l’environnement de gestion du code source [GitHub](https://www.github.com).

### <a name="create-and-update-initiative-definitions"></a>Créer et mettre à jour des définitions d’initiatives

De même, les initiatives ont leur propre fichier JSON et les fichiers associés qui doivent être stockés dans le même dossier. La définition de l’initiative exige que la définition de stratégie existe déjà. Vous ne pouvez donc pas la créer ni la mettre à jour tant que la source de la stratégie n’a pas été mise à jour dans le contrôle de code source, puis dans Azure. Nous vous recommandons la structure suivante pour conserver vos définitions d’initiatives dans le contrôle de code source :

```text
.
|
|- initiatives/ ______________________ # Root folder for initiatives
|  |- init1/ _________________________ # Subfolder for an initiative
|     |- policyset.json ______________ # Initiative definition
|     |- policyset.definitions.json __ # Initiative list of policies
|     |- policyset.parameters.json ___ # Initiative definition of parameters
|     |- assign.<name1>.json _________ # Assignment 1 for this policy initiative
|     |- assign.<name2>.json _________ # Assignment 2 for this policy initiative
|
|  |- init2/ _________________________ # Subfolder for an initiative
|     |- policyset.json ______________ # Initiative definition
|     |- policyset.definitions.json __ # Initiative list of policies
|     |- policyset.parameters.json ___ # Initiative definition of parameters
|     |- assign.<name1>.json _________ # Assignment 1 for this policy initiative
|     |- assign.<name2>.json _________ # Assignment 2 for this policy initiative
|
```

Comme pour les définitions de stratégies, à l’ajout ou la mise à jour d’une initiative existante, le workflow doit automatiquement mettre à jour la définition d’initiative dans Azure. Le test de la définition d’initiative ajoutée ou mise à jour sera effectué dans une étape ultérieure.

### <a name="test-and-validate-the-updated-definition"></a>Tester et valider la définition mise à jour

Maintenant que l’automatisation s’est occupée des définitions de stratégies ou d’initiatives créées ou mises à jour et a effectué la mise à jour sur l’objet dans Azure, il est temps de tester les modifications apportées. La stratégie ou la (ou les) initiative(s) à laquelle (auxquelles) elle appartient doit ensuite être affectée aux ressources de l’environnement le plus éloigné de la production, généralement _Dev_.

L’affectation doit utiliser [enforcementMode](./assignment-structure.md#enforcement-mode)_disabled_ afin que la création et la mise à jour des ressources ne soient pas bloquées, mais que la conformité des ressources existantes à la définition de stratégie mise à jour soit toujours auditée. Même avec enforcementMode, il est recommandé que l’étendue d’affectation soit un groupe de ressources ou un abonnement servant spécialement à valider des stratégies.

> [!NOTE]
> Si enforcementMode est utile, il ne remplace pas pour autant un test rigoureux d’une définition de stratégie dans différentes conditions. La définition de stratégie doit être testée avec des appels d’API REST `PUT` et `PATCH`, des ressources conformes et non conformes, ainsi que des cas limites comme une propriété manquantes dans la ressource.

Une fois l’affectation déployée, utilisez le Kit de développement logiciel (SDK) Azure Policy, l’[action GitHub d’analyse de la conformité Azure Policy](https://github.com/marketplace/actions/azure-policy-compliance-scan) ou la [tâche d’évaluation de la sécurité et de la conformité Azure Pipelines](/azure/devops/pipelines/tasks/deploy/azure-policy) pour [récupérer des données de conformité](../how-to/get-compliance-data.md) pour la nouvelle affectation. L’environnement servant à tester les stratégies et les affectations doit comporter à la fois des ressources conformes et des ressources non conformes.
À l’instar d’un bon test unitaire pour le code, il est important de vérifier que les ressources sont bien celles escomptées et qu’il n’y a pas de faux positifs ou de faux négatifs. Si vous vous contentez de tester et de valider ce que vous attendez, la stratégie risque d’avoir un impact imprévu et non identifié. Pour plus d’informations, consultez [Évaluer l’impact d’une nouvelle définition Azure Policy](./evaluate-impact.md).

### <a name="enable-remediation-tasks"></a>Activer les tâches de correction

Si la validation de l’affectation répond aux attentes, il s’agit ensuite de valider la correction.
Les stratégies qui utilisent [deployIfNotExists](./effects.md#deployifnotexists) ou [modify](./effects.md#modify) peuvent être transformées en une tâche de correction des ressources non conformes.

La première étape de la correction des ressources consiste à accorder à l’affectation de stratégie l’attribution de rôle définie dans la définition de stratégie. Cette attribution de rôle accorde à l’identité managée de l’affectation de stratégie des droits suffisants pour apporter les modifications permettant de rendre la ressource conforme.

Dès que l’affectation de stratégie dispose des autorisations nécessaires, utilisez le kit SDK Policy pour déclencher une tâche de correction sur un ensemble de ressources connues pour être non conformes. Avant de continuer, trois tests doivent être effectués sur ces tâches corrigées :

- Vérifier que la tâche de correction a réussi
- Exécuter l’évaluation de la stratégie pour voir si les résultats de conformité de la stratégie ont été mis à jour comme prévu
- Exécuter un test unitaire d’environnement directement sur les ressources pour vérifier que leurs propriétés ont changé

Le fait de tester à la fois les résultats de l’évaluation de la stratégie mise à jour et l’environnement lui-même permet de confirmer que les tâches de correction ont changé ce qui était attendu et que la définition de stratégie a constaté la modification de conformité comme prévu.

### <a name="update-to-enforced-assignments"></a>Mettre à jour pour appliquer les affectations

Une fois toutes les épreuves de validation effectuées, mettez à jour l’affectation pour utiliser **enforcementMode**_enabled_. Il est recommandé d’effectuer cette modification au départ dans le même environnement éloigné de la production. Après la vérification que cet environnement fonctionne comme prévu, la modification doit être étendue de façon à inclure l’environnement suivant, et ainsi de suite, jusqu’à ce que la stratégie soit déployée sur les ressources de production.

## <a name="process-integrated-evaluations"></a>Traiter les évaluations intégrées

Le workflow général d’Azure Policy en tant que code vise à développer et à déployer des stratégies et des initiatives dans un environnement à grande échelle. Toutefois, l’évaluation de la stratégie doit faire partie du processus de déploiement de tous les workflows qui déploient ou créent des ressources dans Azure, par exemple le déploiement d’applications ou l’exécution de modèles ARM dans le but de créer une infrastructure.

Dans ce cas, une fois le déploiement de l’application ou de l’infrastructure effectué sur un abonnement ou un groupe de ressources de test, l’évaluation de la stratégie doit être effectuée pour cette validation de toutes les stratégies et initiatives existantes. Bien qu’elles puissent être configurées comme **enforcementMode** _disabled_ dans un environnement de ce type, il est utile de savoir très vite si le déploiement d’une application ou d’une infrastructure est contraire aux définitions de stratégies. Cette évaluation de stratégie doit donc constituer une étape de ces workflows et faire échouer les déploiements qui créent des ressources non conformes.

## <a name="review"></a>Révision

Cet article traite du workflow général d’Azure Policy en tant que code et explique que l’évaluation de la stratégie doit faire partie d’autres workflows de déploiement. Ce workflow peut être utilisé dans n’importe quel environnement prenant en charge les scripts et l’automatisation par déclencheurs. Pour accéder à un tutoriel sur l’utilisation de ce workflow sur GitHub, consultez [Tutoriel : Implémenter Azure Policy en tant que code avec GitHub](../tutorials/policy-as-code-github.md).

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [structure des définitions de stratégies](./definition-structure.md)
- En savoir plus sur la [structure des attributions de stratégies](./assignment-structure.md)
- Découvrez comment [créer des stratégies par programmation](../how-to/programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](../how-to/get-compliance-data.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).
