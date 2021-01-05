---
title: Erreur de dépassement de taille de la tâche
description: Décrit comment résoudre les erreurs lorsque la taille d’une tâche ou le modèle est trop important.
ms.topic: troubleshooting
ms.date: 10/07/2020
ms.openlocfilehash: 638bdef246fc908ab997bfb99e7526febdb3792e
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91822154"
---
# <a name="resolve-errors-for-job-size-exceeded"></a>Résoudre les erreurs de dépassement de taille de la tâche

Cet article explique comment résoudre les erreurs **JobSizeExceededException** et **DeploymentSizeExceededException**.

## <a name="symptom"></a>Symptôme

Lors du déploiement d’un modèle, vous recevez une erreur indiquant que le déploiement a dépassé les limites.

## <a name="cause"></a>Cause

Vous pouvez recevoir cette erreur lorsque la taille de votre modèle dépasse 4 Mo. La limite de 4 Mo s’applique à l’état final du modèle une fois développé avec les définitions des ressources qui utilisent la [copie](copy-resources.md) pour créer de nombreuses instances. L’état final comprend également les valeurs résolues pour les variables et les paramètres.

La tâche de déploiement comprend également les métadonnées relatives à la demande. Pour les modèles volumineux, les métadonnées associées au modèle peuvent dépasser la taille autorisée pour une tâche.

Les autres limites du modèle sont les suivantes :

* 256 paramètres
* 256 variables
* 800 ressources (incluant le nombre de copies)
* 64 valeurs de sortie
* 24 576 caractères dans une expression de modèle

## <a name="solution-1---simplify-template"></a>Solution 1 - Simplifier le modèle

Votre première option consiste à simplifier le modèle. Cette option fonctionne lorsque votre modèle déploie un grand nombre de types de ressources différents. Envisagez de diviser le modèle en [modèles liés](linked-templates.md). Divisez vos types de ressources en groupes logiques et ajoutez un modèle lié pour chaque groupe. Par exemple, si vous devez déployer un grand nombre de ressources réseau, vous pouvez déplacer ces ressources vers un modèle lié.

Vous pouvez définir d’autres ressources comme dépendantes du modèle lié et [récupérer des valeurs à partir de la sortie du modèle lié](linked-templates.md#get-values-from-linked-template).

## <a name="solution-2---reduce-name-size"></a>Solution 2 - Réduire la taille du nom

Essayez de raccourcir la longueur des noms que vous utilisez pour les [paramètres](template-parameters.md), [variables](template-variables.md) et [sorties](template-outputs.md). Lorsque ces valeurs sont répétées à l’aide de boucles de copie, un grand nom est multiplié plusieurs fois.

## <a name="solution-3---use-serial-copy"></a>Solution 3 - Utiliser la copie en série

Votre seconde option consiste à modifier votre boucle de copie de [de traitement parallèle en série](copy-resources.md#serial-or-parallel). Utilisez cette option uniquement lorsque vous pensez que l’erreur provient du déploiement d’un grand nombre de ressources à l’aide de la copie. Cette modification peut augmenter considérablement votre temps de déploiement, car les ressources ne sont pas déployées en parallèle.
