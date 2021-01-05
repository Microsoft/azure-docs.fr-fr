---
title: Contrôle de code source et branches de développement - LUIS
description: Comment gérer votre application LUIS (Language Understanding) sous contrôle de code source. Comment appliquer des mises à jour à une application LUIS tout en travaillant dans une branche de développement.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 11/18/2020
ms.openlocfilehash: 1f5c0c7a877964eeb480fa958c7e76eb5706122f
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561271"
---
# <a name="devops-practices-for-luis"></a>Pratiques DevOps pour LUIS

Les ingénieurs logiciels qui développent une application LUIS (Language Understanding) peuvent appliquer les pratiques DevOps sur le [contrôle de code source](luis-concept-devops-sourcecontrol.md), les [builds automatisées](luis-concept-devops-automation.md), les [tests](luis-concept-devops-testing.md) et la [gestion des mises en production](luis-concept-devops-automation.md#release-management) en suivant ces consignes.

## <a name="source-control-and-branch-strategies-for-luis"></a>Stratégies de contrôle de code source et de branche pour LUIS

L’un des principaux facteurs dont dépend le succès de DevOps est le [contrôle de code source](/azure/devops/user-guide/source-control?view=azure-devops). Un système de contrôle de code source permet aux développeurs de collaborer sur le code et de suivre les modifications. L’utilisation de branches permet aux développeurs de basculer entre les différentes versions de la base de code et de travailler indépendamment des autres membres de l’équipe. Quand les développeurs soumettent une [demande de tirage (pull request)](https://help.github.com/github/collaborating-with-issues-and-pull-requests/about-pull-requests) pour proposer des mises à jour depuis une branche vers une autre ou que des modifications sont fusionnées, cela peut être le déclencheur de [builds automatisées](luis-concept-devops-automation.md) pour générer et tester en continu le code.

En utilisant les concepts et les conseils décrits dans ce document, vous pouvez développer une application LUIS tout en effectuant le suivi des modifications dans un système de contrôle de code source et suivre ces bonnes pratiques en matière de génie logiciel :

- **contrôle de code source ;**
  - Le code source de votre application LUIS est dans un format lisible par l’homme.
  - Le modèle peut être généré à partir de la source de manière renouvelable.
  - Le code source peut être géré par un dépôt de code source.
  - Les informations d’identification et les secrets tels que les clés de création et d’abonnement ne sont jamais stockés dans le code source.

- **Création de branche et fusion**
  - Les développeurs peuvent travailler à partir de branches indépendantes.
  - Les développeurs peuvent travailler simultanément dans plusieurs branches.
  - Il est possible d’intégrer les modifications apportées à une application LUIS depuis une branche à une autre via une opération de rebasage ou de fusion.
  - Les développeurs peuvent fusionner une demande de tirage avec la branche parente.

- **Contrôle de version**
  - La version de chaque composant d’une grande application doit être gérée de manière indépendante, ce qui permet aux développeurs de détecter les changements cassants ou les mises à jour en examinant simplement le numéro de version.

- **Revues de code**
  - Les modifications apportées à la demande de tirage sont présentées sous la forme de code source lisible par l’homme qui peut être revu avant que ne soit acceptée la demande de tirage.

## <a name="source-control"></a>Contrôle de code source

Pour conserver la [définition de schéma d’application](./app-schema-definition.md) d’une application LUIS dans un système de gestion du code source, utilisez la représentation au [format LUDown (`.lu`)](/azure/bot-service/file-format/bot-builder-lu-file-format?view=azure-bot-service-4.0) de l’application. Le format `.lu` est préférable au format `.json`, car il est lisible par l’homme, ce qui facilite la création et la revue des modifications dans les demande de tirage.

### <a name="save-a-luis-app-using-the-ludown-format"></a>Enregistrer une application LUIS en utilisant le format LUDown

Pour enregistrer une application LUIS au format `.lu` et la placer sous le contrôle de code source :

- SOIT : [Exportez la version de l’application](./luis-how-to-manage-versions.md#other-actions) au format `.lu` à partir du [portail LUIS](https://www.luis.ai/) et ajoutez-la à votre dépôt de contrôle de code source

- SOIT : Utilisez un éditeur de texte afin de créer un fichier `.lu` pour une application LUIS et ajoutez-le à votre dépôt de contrôle de code source

> [!TIP]
> Si vous utilisez l’exportation JSON d’une application LUIS, vous pouvez [la convertir en LUDown](https://github.com/microsoft/botframework-cli/tree/master/packages/luis#bf-luisconvert).  Utilisez l’option `--sort` pour que les intentions et les énoncés soient triés par ordre alphabétique.  
> Notez que la fonctionnalité d’exportation **.LU** intégrée au portail LUIS trie déjà la sortie.

### <a name="build-the-luis-app-from-source"></a>Générer l’application LUIS à partir de la source

Dans le cas d’une application LUIS, la *génération à partir de la source* signifie [créer une version de l’application LUIS en important la source `.lu`](./luis-how-to-manage-versions.md#import-version), [entraîner la version](./luis-how-to-train.md) et [la publier](./luis-how-to-publish-app.md). Vous pouvez effectuer cette opération dans le portail LUIS ou depuis la ligne de commande :

- Utilisez le portail LUIS pour [importer la version `.lu`](./luis-how-to-manage-versions.md#import-version) de l’application à partir du contrôle de code source et [entraîner](./luis-how-to-train.md) et [publier](./luis-how-to-publish-app.md) l’application.

- Utilisez l’[interface de ligne de commande Bot Framework pour LUIS](https://github.com/microsoft/botbuilder-tools/tree/master/packages/LUIS) depuis la ligne de commande ou dans un workflow CI/CD pour [importer](https://github.com/microsoft/botframework-cli/blob/master/packages/luis/README.md#bf-luisversionimport) la version `.lu` de l’application à partir du contrôle de code source dans une application LUIS, puis [entraîner](https://github.com/microsoft/botframework-cli/blob/master/packages/luis/README.md#bf-luistrainrun) et [publier](https://github.com/microsoft/botframework-cli/blob/master/packages/luis/README.md#bf-luisapplicationpublish) l’application.

### <a name="files-to-maintain-under-source-control"></a>Fichiers à conserver sous contrôle de code source

Les types de fichiers suivants pour votre application LUIS doivent être conservés sous contrôle de code source :

- Fichier `.lu` pour l’application LUIS

- [Fichiers de définition de tests unitaires](luis-concept-devops-testing.md#writing-tests) (énoncés et résultats attendus)

- [Fichiers de test par lots](./luis-concept-batch-test.md#batch-file-format) (énoncés et résultats attendus) utilisés pour le test des performances

### <a name="credentials-and-keys-are-not-checked-in"></a>Les informations d’identification et les clés ne sont pas archivées

N’incluez pas de clés d’abonnement ou de valeurs confidentielles similaires dans les fichiers que vous archivez dans votre dépôt afin d’éviter de les exposer à du personnel non autorisé. Les clés et les autres valeurs dont vous devez empêcher l’archivage sont les suivantes :

- Clés de création et de prédiction LUIS
- Points de terminaison de création et de prédiction LUIS
- Clés d’abonnement Azure
- Jetons d’accès, tels que le jeton pour un [principal de service](/cli/azure/ad/sp?view=azure-cli-latest) Azure utilisé à des fins d’authentification de l’automatisation

#### <a name="strategies-for-securely-managing-secrets"></a>Stratégies pour la gestion sécurisée des secrets

Les stratégies pour la gestion sécurisée des secrets sont les suivantes :

- Si vous utilisez la gestion de version Git, vous pouvez stocker les secrets du runtime dans un fichier local et empêcher l’archivage du fichier en ajoutant un modèle qui fait correspondre le nom du fichier à un fichier [.gitignore](https://git-scm.com/docs/gitignore)
- Dans un workflow d’automatisation, vous pouvez stocker les secrets de manière sécurisée dans la configuration de paramètres offerte par cette technologie d’automatisation. Par exemple, si vous utilisez [GitHub Actions](https://github.com/features/actions), vous pouvez stocker les secrets de manière sécurisée dans des [secrets GitHub](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets).

## <a name="branching-and-merging"></a>Création de branche et fusion

Les systèmes de gestion de version distribués tels que Git offrent une certaine flexibilité dans la façon dont les membres de l’équipe publient, partagent, révisent et itèrent les modifications du code au moyen de branches de développement partagées avec d’autres utilisateurs. Adoptez une [stratégie de branchement Git](/azure/devops/repos/git/git-branching-guidance) appropriée pour votre équipe.

Quelle que soit la stratégie de branchement que vous adoptez, les membres de l’équipe peuvent travailler sur la solution au sein d’une *branche de fonctionnalité* indépendamment du travail en cours dans d’autres branches.

Pour qu’il soit possible de travailler de manière indépendante dans des branches avec un projet LUIS :

- **La branche primaire a sa propre application LUIS.** Cette application représente l’état actuel de votre solution pour votre projet, et sa version active actuelle doit toujours être mappée à la source `.lu` qui se trouve dans la branche primaire. Toutes les mises à jour de la source `.lu` pour cette application doivent être révisées et testées afin que cette application puisse être déployée pour créer des environnements tels que la production à tout moment. Quand des mises à jour du fichier `.lu` sont fusionnées dans la branche primaire depuis la branche de fonctionnalité, vous devez créer une version dans l’application LUIS et [incrémenter le numéro de version](#versioning).

- **Chaque branche de fonctionnalité doit utiliser sa propre instance d’une application LUIS**. Les développeurs travaillent avec cette application dans une branche de fonctionnalité sans risquer d’affecter les développeurs qui travaillent dans d’autres branches. Cette application « branche dev » est une copie de travail qui doit être supprimée lors de la suppression de la branche de fonctionnalité.

![Branche de fonctionnalité Git](./media/luis-concept-devops-sourcecontrol/feature-branch.png)

### <a name="developers-can-work-from-independent-branches"></a>Les développeurs peuvent travailler à partir de branches indépendantes

Les développeurs peuvent travailler sur des mises à jour d’une application LUIS indépendamment des autres branches en effectuant les opérations suivantes :

1. Créer une branche de fonctionnalité à partir de la branche principale (en fonction de la stratégie de branche, généralement primaire ou développement).

1. [Créer une application LUIS dans le portail LUIS](./luis-how-to-start-new-app.md) (« *application de branche dev »* ) uniquement pour prendre en charge le travail dans la branche de fonctionnalité.

   * Si la source `.lu` de votre solution existe déjà dans votre branche, du fait de son enregistrement après le travail effectué dans une autre branche auparavant dans le projet, créez votre application LUIS de branche dev en important le fichier `.lu`.

   * Si vous commencez à travailler sur un nouveau projet, vous ne disposez pas encore de la source `.lu` de votre application LUIS primaire dans le dépôt. Vous créez le fichier `.lu` en exportant votre application de branche dev à partir du portail quand vous avez terminé votre travail dans la branche de fonctionnalité et que vous le soumettez dans le cadre de votre demande de tirage.

1. Travailler sur la version active de votre application de branche dev pour implémenter les modifications requises. Nous vous recommandons de travailler dans une seule version de votre application de branche dev pour tout le travail dans la branche de fonctionnalité. Si vous créez plusieurs versions dans votre application de branche dev, veillez à effectuer le suivi de la version qui contient les modifications que vous souhaitez archiver quand vous soumettez votre demande de tirage.

1. Tester les mises à jour. Consultez [Test de LUIS DevOps](luis-concept-devops-testing.md) pour plus d’informations sur le test de votre application de branche dev.

1. Exportez la version active de votre application de branche dev au format `.lu` à partir de la [liste des versions](./luis-how-to-manage-versions.md).

1. Archiver les mises à jour et inviter les pairs à les évaluer. Si vous utilisez GitHub, vous soumettez une [demande de tirage](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests).

1. Quand les modifications sont approuvées, fusionnez les mises à jour dans la branche primaire. À ce stade, vous créez une [version](./luis-how-to-manage-versions.md) de l’application LUIS *primaire* à l’aide du fichier `.lu` mis à jour dans la branche primaire. Pour plus d’informations sur la définition du nom de version, consultez [Gestion de versions](#versioning).

1. Quand la branche de fonctionnalité est supprimée, il est judicieux de supprimer l’application LUIS de branche dev que vous avez créée pour le travail dans la branche de fonctionnalité.

### <a name="developers-can-work-in-multiple-branches-concurrently"></a>Les développeurs peuvent travailler simultanément dans plusieurs branches

Si vous suivez le modèle décrit plus haut dans [Les développeurs peuvent travailler à partir de branches indépendantes](#developers-can-work-from-independent-branches), vous utilisez une application LUIS unique dans chaque branche de fonctionnalité. Un même développeur peut travailler simultanément sur plusieurs branches, à condition qu’il bascule vers l’application LUIS de branche dev appropriée pour la branche sur laquelle il travaille actuellement.

Nous vous recommandons d’utiliser le même nom pour la branche de fonctionnalité et pour l’application LUIS de branche dev que vous créez pour le travail dans la branche de fonctionnalité, afin de réduire le risque de travailler sur la mauvaise application par inadvertance.

Comme indiqué plus haut, nous vous recommandons, pour des raisons de simplicité, de travailler dans une seule version au sein de chaque application de branche dev. Si vous utilisez plusieurs versions, veillez à activer la version appropriée quand vous basculez entre les applications de branche dev.

### <a name="multiple-developers-can-work-on-the-same-branch-concurrently"></a>Plusieurs développeurs peuvent travailler simultanément sur la même branche

Vous pouvez prendre en charge plusieurs développeurs qui travaillent sur la même branche de fonctionnalité en même temps :

- Les développeurs extraient la même branche de fonctionnalité et envoient (push) et tirent (pull) les modifications soumises par eux-mêmes et d’autres développeurs pendant le travail, comme d’habitude.

- Si vous suivez le modèle décrit plus haut dans [Les développeurs peuvent travailler à partir de branches indépendantes](#developers-can-work-from-independent-branches), cette branche utilise une application LUIS unique pour prendre en charge le développement. Cette application LUIS « branche dev » est créée par le premier membre de l’équipe de développement qui commence à travailler dans la branche de fonctionnalité.

- [Ajoutez des membres d’équipe en tant que contributeurs](./luis-how-to-collaborate.md) à l’application LUIS de branche dev.

- Une fois terminé le travail dans la branche de fonctionnalité, exportez la version active de l’application LUIS de branche dev en tant que `.lu` à partir de la [liste des versions](./luis-how-to-manage-versions.md), enregistrez le fichier `.lu` mis à jour dans le dépôt, puis archivez les modifications et soumettez-les dans une demande de tirage.

### <a name="incorporating-changes-from-one-branch-to-another-with-rebase-or-merge"></a>Incorporation de modifications d’une branche à une autre à l’aide d’une opération de rebasage ou de fusion

D’autres développeurs de votre équipe travaillant dans une autre branche peuvent avoir effectué des mises à jour de la source `.lu` et les avoir fusionnées dans la branche primaire après que vous avez créé votre branche de fonctionnalité. Vous pouvez intégrer leurs modifications dans votre version de travail avant de continuer à apporter vos propres modifications dans votre branche de fonctionnalité. Pour ce faire, vous pouvez [effectuer un rebasage ou une fusion dans la branche primaire](https://git-scm.com/book/en/v2/Git-Branching-Rebasing) de la même façon que pour n’importe quelle autre ressource de code. L’application LUIS au format LUDown étant lisible par l’homme, elle prend en charge la fusion à l’aide des outils de fusion standard.

Suivez ces conseils si vous rebasez votre application LUIS dans une branche de fonctionnalité :

- Avant d’effectuer le rebasage ou la fusion, assurez-vous que votre copie locale de la source `.lu` de votre application dispose de toutes les dernières modifications que vous avez apportées à l’aide du portail Luis, en réexportant d’abord votre application à partir du portail. Ainsi, toute modification que vous avez apportée dans le portail et qui n’a pas encore été exportée n’est perdue.

- Pendant la fusion, utilisez des outils standard pour résoudre les conflits de fusion éventuels.

- Une fois le rebasage ou la fusion terminé, n’oubliez pas de réimporter l’application dans le portail, afin d’utiliser l’application mise à jour quand vous continuez à appliquer vos propres modifications.

### <a name="merge-prs"></a>Fusionner les demandes de tirage

Une fois votre demande de tirage approuvée, vous pouvez fusionner vos modifications dans votre branche primaire. Aucune considération particulière ne s’applique à la source LUDown pour une application LUIS : étant lisible par l’homme, elle prend en charge la fusion à l’aide des outils de fusion standard. Les conflits de fusion peuvent être résolus de la même façon que pour les autres fichiers sources.

Une fois votre demande de tirage fusionnée, il est recommandé d’effectuer un nettoyage :

- Supprimer la branche dans votre dépôt

- Supprimez l’application LUIS « branche dev » que vous avez créée pour le travail dans la branche de fonctionnalité.

À l’image des ressources de code d’application, vous devez écrire des tests unitaires pour accompagner les mises à jour d’application LUIS. Vous devez utiliser des workflows d’intégration continue pour tester :

- Les mises à jour dans une demande de tirage avant la fusion de celle-ci
- L’application LUIS de branche primaire après l’approbation d’une demande de tirage et la fusion des modifications dans la branche primaire.

Pour plus d’informations sur les tests pour LUIS dans DevOps, consultez [Test de LUIS DevOps](luis-concept-devops-testing.md). Pour plus d’informations sur l’implémentation des workflows, consultez [Workflows d’automatisation pour LUIS DevOps](luis-concept-devops-automation.md).

## <a name="code-reviews"></a>Revues de code

Une application LUIS au format LUDown étant lisible par l’homme, la communication de modifications dans une demande de tirage adaptée à la révision est prise en charge. Les fichiers de tests unitaires, également écrits au format LUDown, sont aussi facilement révisables dans une demande de tirage.

## <a name="versioning"></a>Contrôle de version

Une application est constituée de plusieurs composants qui peuvent inclure des éléments tels qu’un bot s’exécutant dans [Azure Bot Service](/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0), [QnA Maker](https://www.qnamaker.ai/), le [service Azure Speech](../speech-service/overview.md) et bien plus encore. Pour que les applications soient faiblement couplées, utilisez la [gestion de version](/azure/devops/learn/git/what-is-version-control) afin que la version de chaque composant d’une application soit gérée de manière indépendante, ce qui permet aux développeurs de détecter les changements cassants ou les mises à jour en examinant simplement le numéro de version. Il est plus facile de gérer la version de votre application LUIS indépendamment des autres composants si vous conservez celle-ci dans son propre dépôt.

Un schéma de gestion de versions doit être appliqué à l’application LUIS pour la branche primaire. Quand vous fusionnez des mises à jour du fichier `.lu` pour une application LUIS dans la branche primaire, vous importez cette source mise à jour dans une nouvelle version de l’application LUIS pour la branche primaire.

Nous vous recommandons d’utiliser un schéma de gestion de versions numérique pour la version primaire de l’application LUIS, par exemple :

`major.minor[.build[.revision]]`

À chaque mise à jour, le numéro de version est incrémenté au niveau du dernier chiffre.

La version principale/mineure peut être utilisée pour indiquer l’étendue des modifications apportées aux fonctionnalités de l’application LUIS :

* Version principale : une modification importante, telle que la prise en charge d’une nouvelle [intention](./luis-concept-intent.md) ou [entité](./luis-concept-entity-types.md)
* Version mineure : changement mineur à compatibilité descendante, par exemple après un nouvel entraînement significatif
* Build : aucune modification de fonctionnalité, juste une autre build.

Une fois que vous avez déterminé le numéro de version de la dernière révision de votre application LUIS primaire, vous devez générer et tester la nouvelle version de l’application, puis la publier sur un point de terminaison où elle peut être utilisée dans différents environnements de génération, tels que l’assurance qualité ou la production. Nous vous recommandons fortement d’automatiser toutes ces étapes dans un workflow d’intégration continue (CI).

Consultez l'article :
- [Workflows d’automatisation](luis-concept-devops-automation.md) pour plus d’informations sur la façon d’implémenter un workflow CI afin de tester et de publier une application LUIS.
- [Gestion des mises en production](luis-concept-devops-automation.md#release-management) pour plus d’informations sur le déploiement de votre application LUIS.

### <a name="versioning-the-feature-branch-luis-app"></a>Gestion des versions de l’application LUIS « branche de fonctionnalité »

Quand vous travaillez avec une application LUIS « branche dev » que vous avez créée pour prendre en charge le travail dans une branche de fonctionnalité, vous exportez votre application quand votre travail est terminé et vous incluez le fichier `'lu` mis à jour dans votre demande de tirage. La branche dans votre dépôt et l’application LUIS « branche dev », doivent être supprimées une fois la demande de tirage fusionnée dans la branche primaire. Étant donné que cette application existe uniquement pour prendre en charge le travail dans la branche de fonctionnalité, vous n’avez pas besoin d’appliquer de schéma de gestion de versions particulier dans cette application.

La gestion de version doit être appliquée quand vos modifications dans votre demande de tirage sont fusionnées dans la branche primaire, afin que les versions de toutes les mises à jour dans la branche primaire soient gérées de manière indépendante.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez les [tests pour LUIS dans DevOps](luis-concept-devops-testing.md).
* Découvrez comment [implémenter DevOps pour LUIS avec GitHub](luis-how-to-devops-with-github.md).