---
title: Vue d’ensemble des modèles
description: Décrit les avantages apportés par l’utilisation de modèles Azure Resource Manager (modèles ARM) pour le déploiement de ressources.
ms.topic: conceptual
ms.date: 12/01/2020
ms.openlocfilehash: da091d09f6d242d4b98903a8dcd76fe305e578b8
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96497994"
---
# <a name="what-are-arm-templates"></a>Que sont les modèles ARM ?

Avec la migration vers le cloud, de nombreuses équipes ont adopté des méthodes de développement agile. Ces équipes effectuent des itérations rapides. Elles doivent déployer à plusieurs reprises leurs solutions dans le cloud et savoir que leur infrastructure est dans un état fiable. Comme l’infrastructure fait partie intégrante du processus itératif, la séparation entre les opérations et le développement a disparu. Les équipes doivent gérer le code d’infrastructure et le code d’application à l’aide d’un processus unifié.

Pour relever ces défis, vous pouvez automatiser les déploiements et utiliser la pratique de l’infrastructure en tant que code. Dans le code, vous définissez l’infrastructure qui doit être déployée. Le code d’infrastructure devient partie intégrante de votre projet. Tout comme le code d’application, vous stockez le code d’infrastructure dans un dépôt source et vous gérez ses versions. Toute personne de votre équipe peut exécuter le code et déployer des environnements similaires.

Pour implémenter une infrastructure en tant que code pour vos solutions Azure, utilisez des modèles Azure Resource Manager (modèles ARM). Le modèle est un fichier JSON (JavaScript Object Notation) qui définit l’infrastructure et la configuration de votre projet. Le modèle utilise la syntaxe déclarative, qui vous permet d’indiquer ce que vous envisagez de déployer sans avoir à écrire la séquence de commandes de programmation pour le créer. Dans le modèle, vous spécifiez les ressources à déployer et les propriétés de ces ressources.

## <a name="why-choose-arm-templates"></a>Pourquoi choisir les modèles ARM ?

Si vous essayez de choisir entre utiliser des modèles Resource Manager et l’un des autres services d’infrastructure en tant que code, tenez compte des avantages suivants de l’utilisation de modèles :

* **Syntaxe déclarative** : Les modèles Resource Manager vous permettent de créer et de déployer une infrastructure Azure entière de façon déclarative. Par exemple, vous pouvez déployer non seulement des machines virtuelles, mais également l’infrastructure réseau, les systèmes de stockage et toute autre ressource dont vous pouvez avoir besoin.

* **Résultats reproductibles** : Déployez votre infrastructure à plusieurs reprises tout au long du cycle de vie de développement. Vous avez ainsi l’assurance que vos ressources sont déployées de façon cohérente. Les modèles sont idempotents, ce qui signifie que vous pouvez déployer le même modèle plusieurs fois et obtenir les mêmes types de ressources dans le même état. Vous pouvez développer un modèle qui représente l’état souhaité, plutôt que de développer un grand nombre de modèles distincts pour représenter des mises à jour.

* **Orchestration** : Vous n’avez pas à vous soucier de la complexité des opérations de commande. Resource Manager orchestre le déploiement de ressources interdépendantes afin qu’elles soient créées dans l’ordre approprié. Lorsque cela est possible, Resource Manager déploie des ressources en parallèle afin que vos déploiements se terminent plus rapidement que les déploiements en série. Vous déployez le modèle par le biais d’une seule commande, plutôt que par le biais de plusieurs commandes impératives.

   ![Comparaison de déploiements de modèles](./media/overview/template-processing.png)

* **Fichiers modulaires** : Vous pouvez diviser vos modèles en composants plus petits et réutilisables et les lier au moment du déploiement. Vous pouvez également imbriquer un modèle dans d’autres modèles.

* **Créez une ressource Azure** : Vous pouvez utiliser immédiatement les nouveaux services et fonctionnalités Azure dans les modèles. Dès qu’un fournisseur de ressources introduit de nouvelles ressources, vous pouvez les déployer par le biais de modèles. Vous n’avez pas besoin d’attendre la mise à jour des outils ou des modules avant d’utiliser les nouveaux services.

* **Extensibilité** : Avec les [scripts de déploiement](deployment-script-template.md), vous pouvez ajouter des scripts PowerShell ou Bash à vos modèles. Les scripts de déploiement étendent votre capacité à configurer des ressources pendant le déploiement. Un script peut être inclus dans le modèle, ou stocké dans une source externe et référencé dans le modèle. Les scripts de déploiement vous permettent d’effectuer la configuration de votre environnement de bout en bout dans un modèle ARM unique.

* **Test** : Vous pouvez vérifier que votre modèle suit les recommandations en le testant avec le kit d’outils de modèle ARM (arm-ttk). Ce kit de test est un script PowerShell que vous pouvez télécharger à partir de [GitHub](https://github.com/Azure/arm-ttk). Le kit d’outils vous permet de développer plus facilement une expertise à l’aide de la langue du modèle.

* **Aperçu des modifications** : Vous pouvez utiliser l’[opération de simulation](template-deploy-what-if.md) pour obtenir un aperçu des modifications avant de déployer le modèle. Avec la simulation, vous pouvez voir quelles ressources seront créées, mises à jour ou supprimées ainsi que les propriétés de ressource qui seront modifiées. L’opération de simulation vérifie l’état actuel de votre environnement et élimine la nécessité de gérer l’état.

* **Validation intégrée** : Votre modèle est déployé uniquement après avoir réussi la validation. Resource Manager vérifie le modèle avant de commencer le déploiement pour être sûr que celui-ci va réussir. Votre déploiement est moins susceptible de s’arrêter dans un état à moitié terminé.

* **Déploiements suivis** : Dans le portail Azure, vous pouvez consulter l’historique des déploiements et obtenir des informations sur le déploiement des modèles. Vous pouvez voir le modèle qui a été déployé, les valeurs de paramètre passées et toutes les valeurs de sortie. Les autres services d’infrastructure en tant que code ne font pas l’objet d’un suivi par le biais du portail.

   ![Historique de déploiement](./media/overview/deployment-history.png)

* **Stratégie en tant que code** : [Azure Policy](../../governance/policy/overview.md) est une stratégie en tant qu’infrastructure de code pour automatiser la gouvernance. Si vous utilisez des stratégies Azure, la correction des stratégies est effectuée sur des ressources non conformes quand elles sont déployées par le biais de modèles.

* **Blueprints de déploiement** : Vous pouvez tirer parti des [plans](../../governance/blueprints/overview.md) fournis par Microsoft pour respecter les normes réglementaires et de conformité. Ces blueprints incluent des modèles prédéfinis pour différentes architectures.

* **Intégration CI/CD** : Vous pouvez intégrer des modèles dans vos outils d’intégration et de déploiement continus (CI/CD), qui peuvent automatiser vos pipelines de mise en production en vue d’effectuer des mises à jour rapides et fiables des applications et des infrastructures. À l’aide de la tâche de modèle Resource Manager et Azure DevOps, vous pouvez utiliser Azure Pipelines pour générer et déployer des projets de modèle Azure Resource Manager en continu. Pour plus d’informations, consultez [Projet VS avec Azure Pipelines](add-template-to-azure-pipelines.md) et le [Tutoriel : Intégration continue de modèles Azure Resource Manager avec Azure Pipelines](./deployment-tutorial-pipeline.md).

* **Code exportable** : Vous pouvez obtenir un modèle pour un groupe de ressources existant en exportant l’état actuel du groupe de ressources ou en consultant le modèle utilisé pour un déploiement particulier. L’affichage du [modèle exporté](export-template-portal.md) est un moyen utile pour en découvrir plus sur sa syntaxe.

* **Outils de création** : Vous pouvez créer des modèles avec [Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md) et l’extension d’outil de modèle. Vous bénéficiez d’IntelliSense, de la mise en surbrillance de la syntaxe, de l’aide en ligne et de nombreuses autres fonctions de langage. Outre Visual Studio Code, vous pouvez également utiliser [Visual Studio](create-visual-studio-deployment-project.md).

## <a name="template-file"></a>Fichier de modèle

Dans votre modèle, vous pouvez écrire des [expressions de modèle](template-expressions.md) qui étendent les fonctionnalités de JSON. Ces expressions utilisent les [fonctions](template-functions.md) fournies par Resource Manager.

Le modèle comprend les sections suivantes :

* [Paramètres](template-parameters.md) - Pendant le déploiement, fournissez des valeurs qui permettent d’utiliser le même modèle avec différents environnements.

* [Variables](template-variables.md) - Définissez des valeurs qui sont réutilisées dans vos modèles. Elles peuvent être construites à partir de valeurs de paramètres.

* [Fonctions définies par l’utilisateur](template-user-defined-functions.md) - Créez des fonctions personnalisées qui simplifient votre modèle.

* [Ressources](template-syntax.md#resources) - Spécifiez les ressources à déployer.

* [Sorties](template-outputs.md) - Retournez des valeurs à partir des ressources déployées.

## <a name="template-deployment-process"></a>Processus de déploiement de modèle

Quand vous déployez un modèle, Resource Manager le convertit en opérations d’API REST. Par exemple, lorsque Resource Manager reçoit un modèle avec la définition de ressource suivante :

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2019-04-01",
    "name": "mystorageaccount",
    "location": "westus",
    "sku": {
      "name": "Standard_LRS"
    },
    "kind": "StorageV2",
    "properties": {}
  }
]
```

Il convertit la définition en opération API REST suivante, qui est envoyée au fournisseur de ressources Microsoft.Storage :

```HTTP
PUT
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/mystorageaccount?api-version=2019-04-01
REQUEST BODY
{
  "location": "westus",
  "sku": {
    "name": "Standard_LRS"
  },
  "kind": "StorageV2",
  "properties": {}
}
```

Notez que la **apiVersion** que vous définissez dans le modèle pour la ressource est utilisée comme version d’API pour l’opération REST. Vous pouvez déployer le modèle à plusieurs reprises et être sûr qu’il continuera à fonctionner. En utilisant la même version d’API, vous n’avez pas à vous soucier des changements cassants qui peuvent être introduits dans les versions ultérieures.

## <a name="template-design"></a>Conception de modèle

La manière dont vous définissez les modèles et les groupes de ressources dépend entièrement de vous et de la façon dont vous voulez gérer votre solution. Par exemple, vous pouvez déployer votre application à trois niveaux via un modèle unique pour un groupe de ressources unique.

![modèle à trois niveaux](./media/overview/3-tier-template.png)

Cependant, il est inutile de définir toute votre infrastructure dans un seul modèle. Il peut être judicieux de diviser les exigences de votre déploiement dans un ensemble de modèles ciblés destinés à un usage particulier. Vous pouvez facilement réutiliser ces modèles pour différentes solutions. Pour déployer une solution particulière, créez un modèle principal qui relie tous les modèles requis. L’illustration suivante montre comment déployer une solution à trois niveaux via un modèle parent qui inclut trois modèles imbriqués.

![modèle à niveaux imbriqués](./media/overview/nested-tiers-template.png)

Si vos niveaux vont avoir des cycles de vie distincts, vous pouvez déployer les trois niveaux dans des groupes de ressources séparés. Notez que les ressources peuvent toujours être liées aux ressources dans d’autres groupes.

![modèle niveau](./media/overview/tier-templates.png)

Pour plus d’informations sur les modèles imbriqués, consultez [Utilisation de modèles liés avec Azure Resource Manager](linked-templates.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir un didacticiel pas à pas vous guidant tout au long du processus de création d’un modèle, consultez :[Totoriel : Créer et déployer votre premier modèle ARM](template-tutorial-create-first-template.md).
* Pour plus d’informations sur les propriétés de fichiers de modèle, consultez [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](template-syntax.md).
* Pour en savoir plus sur l’exportation de modèles, consultez [Démarrage rapide : Créer et déployer des modèles Resource Manager à l’aide du portail Azure](quickstart-create-templates-use-the-portal.md).
* Pour obtenir des réponses aux questions les plus fréquentes, consultez [Foire aux questions sur les modèles ARM](frequently-asked-questions.md).
