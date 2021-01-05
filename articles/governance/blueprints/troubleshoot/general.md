---
title: Résolution des erreurs courantes
description: Découvrez comment détecter un problème lié à la création, à l’attribution et à la suppression de blueprints tels que les violations de stratégie et les fonctions de paramètres blueprint.
ms.date: 10/14/2020
ms.topic: troubleshooting
ms.openlocfilehash: a1689141f95aaac9183391af79edb0cabf5343b6
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92058282"
---
# <a name="troubleshoot-errors-using-azure-blueprints"></a>Résoudre les erreurs à l’aide de blueprints Azure

Vous pouvez rencontrer des erreurs lors de la création, de l’affectation ou de la suppression des blueprints. Cet article décrit différentes erreurs qui peuvent se produire et explique comment les résoudre.

## <a name="finding-error-details"></a>Recherche des détails de l’erreur

De nombreuses erreurs sont dues à l’affectation d’un blueprint à une étendue. Quand une affectation échoue, le blueprint fournit des détails sur le déploiement ayant échoué. Ces informations précisent le problème afin que vous puissiez le résoudre et que le déploiement suivant réussisse.

1. Sélectionnez **Tous les services** dans le volet gauche. Recherchez et sélectionnez **Blueprints**.

1. Sélectionnez **Blueprints affectés** dans la page de gauche et utilisez la zone de recherche pour filtrer les affectations de blueprints afin d’identifier celle qui a échoué. Vous pouvez également trier la table des affectations d’après la colonne **État de l’approvisionnement** pour regrouper toutes les affectations ayant échoué.

1. Sélectionnez le blueprint ayant l’état _Échec_, ou cliquez avec le bouton droit et sélectionnez **Voir les détails de l’affectation**.

1. Une bannière rouge signalant que l’affectation a échoué figure en haut de la page d’affectation de blueprint. Sélectionnez n’importe où sur la bannière pour obtenir plus de détails.

Il est courant que l’erreur soit due à un artefact et non au blueprint lui-même. Si un artefact crée un coffre de clés, mais que la stratégie Azure empêche la création du coffre de clés, l’affectation entière échoue.

## <a name="general-errors"></a>Erreurs générales.

### <a name="scenario-policy-violation"></a><a name="policy-violation"></a>Scénario : Violation de stratégie

#### <a name="issue"></a>Problème

Le déploiement du modèle a échoué à cause de la violation de stratégie.

#### <a name="cause"></a>Cause

Une stratégie peut entrer en conflit avec le déploiement pour plusieurs raisons :

- La ressource en cours de création est limitée par la stratégie (généralement des restrictions de référence SKU ou d’emplacement)
- Le déploiement définit des champs qui sont configurés par la stratégie (courant avec les étiquettes)

#### <a name="resolution"></a>Résolution

Modifiez le blueprint pour qu’il ne soit pas en conflit avec les stratégies indiquées dans les détails de l’erreur. Si cette modification n’est pas possible, une autre option consiste à changer l’étendue de l’affectation de stratégie de sorte que le blueprint ne soit plus en conflit avec la stratégie.

### <a name="scenario-blueprint-parameter-is-a-function"></a><a name="escape-function-parameter"></a>Scénario : le paramètre de blueprint est une fonction

#### <a name="issue"></a>Problème

Les paramètres de blueprint qui sont des fonctions sont traitées avant d’être transmises aux artefacts.

#### <a name="cause"></a>Cause

En passant un paramètre de blueprint qui utilise une fonction, comme `[resourceGroup().tags.myTag]`, à un artefact fait que le résultat traité de la fonction est défini sur l’artefact et non sur la fonction dynamique.

#### <a name="resolution"></a>Résolution

Pour passer une fonction en tant que paramètre, ajoutez un caractère d’échappement devant toute la chaîne avec `[` pour que le paramètre de blueprint se présente sous la forme `[[resourceGroup().tags.myTag]`. Le caractère d’échappement fait que les blueprints traitent la valeur comme une chaîne lors du traitement du blueprint. Le service Blueprints place ensuite la fonction sur l’artefact, ce qui lui permet d’être dynamique comme prévu. Pour plus d’informations, voir [Syntaxe et expressions dans les modèles Azure Resource Manager](../../../azure-resource-manager/templates/template-expressions.md).

## <a name="delete-errors"></a>Supprimer les erreurs

### <a name="scenario-assignment-deletion-timeout"></a><a name="assign-delete-timeout"></a>Scénario : Expiration du délai de suppression de l’attribution

#### <a name="issue"></a>Problème

La suppression d’une attribution de blueprint ne se termine pas.

#### <a name="cause"></a>Cause

Une attribution de blueprint peut être bloquée dans un état non terminal lors de sa suppression. Cet état est dû au fait que les ressources créées par l’attribution de blueprint sont toujours en attente de suppression ou ne retournent pas de code d’état à Azure Blueprints.

#### <a name="resolution"></a>Résolution

Les attributions de blueprint dans un état non terminal sont automatiquement marquées **Échec** après un délai d’expiration de _six heures_. Une fois que le délai d’expiration a ajusté l’état de l’attribution de blueprint, la suppression peut être retentée.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

- Obtenez des réponses de la part d’experts Azure via les [Forums Azure](https://azure.microsoft.com/support/forums/).
- Connectez-vous avec [@AzureSupport](https://twitter.com/azuresupport), qui est le compte Microsoft Azure officiel pour améliorer l’expérience client en connectant la communauté Azure aux ressources appropriées : réponses, support technique et experts.
- Si vous avez besoin de plus d’aide, vous pouvez signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur **Obtenir un support**.