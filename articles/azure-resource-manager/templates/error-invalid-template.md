---
title: Erreurs de modèle non valide
description: Décrit comment résoudre les erreurs de modèle non valide quand vous déployez des modèles Azure Resource Manager.
ms.topic: troubleshooting
ms.date: 05/22/2020
ms.openlocfilehash: ba19d3c4e72a765e2aaff7393915b77a80daf2ba
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185791"
---
# <a name="resolve-errors-for-invalid-template"></a>Résoudre les erreurs de modèle non valide

Cet article décrit comment résoudre les erreurs de modèle non valide.

## <a name="symptom"></a>Symptôme

Au cours du déploiement d’un modèle, vous rencontrez l’erreur suivante :

```
Code=InvalidTemplate
Message=<varies>
```

Le message d’erreur varie selon le type d’erreur.

## <a name="cause"></a>Cause

Cette erreur peut résulter de différents types d’erreurs. Elles impliquent généralement une erreur de syntaxe ou de structure dans le modèle.

<a id="syntax-error"></a>

## <a name="solution-1---syntax-error"></a>Solution 1 - erreur de syntaxe

Si vous recevez un message d’erreur qui indique que la validation du modèle a échoué, vous avez peut-être un problème de syntaxe dans votre modèle.

```
Code=InvalidTemplate
Message=Deployment template validation failed
```

Cette erreur est facile à commettre car les expressions de modèle peuvent être complexes. Par exemple, l’affectation de nom suivante pour un compte de stockage contient un jeu de crochets, trois fonctions, trois jeux de parenthèses, un jeu de guillemets simples et une propriété :

```json
"name": "[concat('storage', uniqueString(resourceGroup().id))]",
```

Si vous ne fournissez pas la syntaxe correspondante, le modèle produit une valeur très différente de celle souhaitée.

Lorsque vous recevez ce type d’erreur, examinez attentivement la syntaxe d’expression. Vous pouvez utiliser un éditeur JSON comme [Visual Studio](create-visual-studio-deployment-project.md) ou [Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md), qui vous signale des erreurs de syntaxe.

<a id="incorrect-segment-lengths"></a>

## <a name="solution-2---incorrect-segment-lengths"></a>Solution 2 - longueurs de segments incorrectes

Une autre erreur de modèle non valide se produit lorsque le nom de la ressource n’est pas au format approprié.

```
Code=InvalidTemplate
Message=Deployment template validation failed: 'The template resource {resource-name}'
for type {resource-type} has incorrect segment lengths.
```

Une ressource au niveau racine doit avoir un segment de moins dans le nom que dans le type de ressource. Chaque segment se différencie par une barre oblique. Dans l’exemple suivant, le type comporte deux segments et le nom comporte un segment. Il s’agit donc d’un **nom valide**.

```json
{
  "type": "Microsoft.Web/serverfarms",
  "name": "myHostingPlanName",
  ...
}
```

Mais l’exemple suivant n’est **pas un nom valide** car il possède le même nombre de segments que le type.

```json
{
  "type": "Microsoft.Web/serverfarms",
  "name": "appPlan/myHostingPlanName",
  ...
}
```

Pour les ressources enfants, le type et le nom ont le même nombre de segments. Ce nombre de segments est logique, car le nom complet et le type de l’enfant incluent le nom du parent et le type. Par conséquent, le nom complet a toujours un segment de moins que le type complet.

```json
"resources": [
  {
    "type": "Microsoft.KeyVault/vaults",
    "name": "contosokeyvault",
    ...
    "resources": [
      {
        "type": "secrets",
        "name": "appPassword",
        ...
      }
    ]
  }
]
```

Obtenir des segments valides peut être difficile si des types Resource Manager sont appliqués à plusieurs fournisseurs de ressources. Par exemple, l’installation d’un verrou de ressource sur un site web nécessite un type avec quatre segments. Par conséquent, le nom comporte 3 segments :

```json
{
  "type": "Microsoft.Web/sites/providers/locks",
  "name": "[concat(variables('siteName'),'/Microsoft.Authorization/MySiteLock')]",
  ...
}
```

<a id="parameter-not-valid"></a>

## <a name="solution-3---parameter-is-not-valid"></a>Solution 3 - le paramètre n’est pas valide

Si vous fournissez une valeur de paramètre qui ne fait pas partie des valeurs autorisées, vous recevrez un message similaire à celui de l’erreur suivante :

```
Code=InvalidTemplate;
Message=Deployment template validation failed: 'The provided value {parameter value}
for the template parameter {parameter name} is not valid. The parameter value is not
part of the allowed values
```

Vérifiez les valeurs autorisées dans le modèle et fournissez-en une pendant le déploiement. Pour plus d’informations sur les valeurs de paramètres autorisées, consultez [Section Parameters des modèles Azure Resource Manager](template-syntax.md#parameters).

<a id="too-many-resource-groups"></a>

## <a name="solution-4---too-many-target-resource-groups"></a>Solution 4 : trop de groupes de ressources cibles

Vous pouvez voir cette erreur dans les déploiements antérieurs, car vous étiez limité à cinq groupes de ressources cibles dans un déploiement unique. En mai 2020, cette limite a été augmentée à 800 groupes de ressources. Pour plus d’informations, voir [Déployer des ressources Azure sur plusieurs groupes de ressources et des abonnements](./deploy-to-resource-group.md).

<a id="circular-dependency"></a>

## <a name="solution-5---circular-dependency-detected"></a>Solution 5 : dépendance circulaire détectée

Vous recevez cette erreur lorsque des ressources dépendant les unes des autres empêchent le déploiement de démarrer. À cause de l’effet combiné d’interdépendances, plusieurs ressources attendent d’autres ressources qui sont elles aussi en attente. Par exemple, la ressource 1 dépend de la ressource 3, la ressource 2 de la ressource 1 et la ressource 3 de la ressource 2. Vous pouvez généralement résoudre ce problème en supprimant les dépendances inutiles.

Pour résoudre une dépendance circulaire :

1. Dans votre modèle, recherchez la ressource identifiée dans la dépendance circulaire.
2. Pour cette ressource, examinez la propriété **dependsOn** et les utilisations de la fonction **référence** pour voir de quelles ressources elle dépend.
3. Examinez ces ressources pour voir de quelles ressources elles dépendent. Suivez les dépendances jusqu’à ce que vous trouviez une ressource qui dépend de la ressource d’origine.
5. Pour les ressources impliquées dans la dépendance circulaire, examinez attentivement toutes les utilisations de la propriété **dependsOn** pour identifier les dépendances qui ne sont pas nécessaires. Supprimer ces dépendances. Si vous n’êtes pas certain qu’une dépendance soit nécessaire, essayez de la supprimer.
6. Redéployez le modèle.

La suppression de valeurs de la propriété **dependsOn** peut provoquer des erreurs lors du déploiement du modèle. Si vous obtenez une erreur, rajoutez la dépendance dans le modèle.

Si cette approche ne résout pas le problème de dépendance circulaire, essayez de déplacer une partie de votre logique de déploiement dans les ressources enfants (par exemple, les extensions ou les paramètres de configuration). Configurez ces ressources enfants pour qu’elles se déploient après les ressources impliquées dans la dépendance circulaire. Par exemple, supposons que vous déployiez deux machines virtuelles, mais que vous deviez définir sur chacune d’elles des propriétés faisant référence les unes aux autres. Vous pouvez les déployer dans l’ordre suivant :

1. Machine virtuelle 1
2. Machine virtuelle 2
3. L’extension sur la machine virtuelle 1 dépend des machines virtuelles 1 et 2. L’extension définit sur la machine virtuelle 1 des valeurs qu’elle obtient de la machine virtuelle 2.
4. L’extension sur la machine virtuelle 2 dépend des machines virtuelles 1 et 2. L’extension définit sur la machine virtuelle 2 des valeurs qu’elle obtient de la machine virtuelle 1.

Cette approche fonctionne aussi pour les applications App Service. Essayez de déplacer des valeurs de configuration dans une ressource enfant de la ressource d’application. Vous pouvez déployer deux applications web dans l’ordre suivant :

1. Application web 1
2. Application web 2
3. La configuration de l’application web 1 dépend des applications web 1 et 2. Elle contient des paramètres d’application avec les valeurs de l’application web 2.
4. La configuration de l’application web 2 dépend des applications web 1 et 2. Elle contient des paramètres d’application avec les valeurs de l’application web 1.