---
title: Nouvelle affectation de stratégie à l’aide de modèles
description: Dans ce guide de démarrage rapide, vous allez utiliser un modèle Azure Resource Manager (modèle ARM) pour créer une affectation de stratégie dans le but d’identifier les ressources non conformes.
ms.date: 08/17/2020
ms.topic: quickstart
ms.custom: subject-armqs
ms.openlocfilehash: c32f833520a811d47129196ae4d67c92bad7e21f
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/10/2020
ms.locfileid: "89651421"
---
# <a name="quickstart-create-a-policy-assignment-to-identify-non-compliant-resources-by-using-an-arm-template"></a>Créer une affectation de stratégie pour identifier des ressources non conformes à l’aide d’un modèle ARM

La première étape pour comprendre la conformité dans Azure consiste à identifier l’état de vos ressources.
Ce guide de démarrage rapide explique pas à pas comment utiliser un modèle Azure Resource Manager (modèle ARM) pour créer une attribution de stratégie afin d’identifier les machines virtuelles qui n’utilisent pas de disques managés. À la fin de ce processus, vous aurez identifié correctement les machines virtuelles qui n’utilisent pas de disques managés. Elles sont _non conformes_ à l’attribution de stratégie.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Bouton pour déployer le modèle Resource Manager pour l’affectation d’une stratégie Azure à Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurepolicy-assign-builtinpolicy-resourcegroup%2Fazuredeploy.json":::

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Dans ce guide de démarrage rapide, vous allez créer une affectation de stratégie et affecter une définition de stratégie intégrée appelée _Auditer les machines virtuelles qui n’utilisent pas de disques managés_. Pour obtenir une liste partielle des stratégies intégrées disponibles, consultez [Exemples Azure Policy](./samples/index.md).

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-azurepolicy-assign-builtinpolicy-resourcegroup/).

:::code language="json" source="~/quickstart-templates/101-azurepolicy-assign-builtinpolicy-resourcegroup/azuredeploy.json":::

La ressource définie dans le modèle est :

- [Microsoft.Authorization/policyAssignments](/azure/templates/microsoft.authorization/policyassignments)

## <a name="deploy-the-template"></a>Déployer le modèle

> [!NOTE]
> Le service Azure Policy est gratuit. Pour plus d’informations, consultez [Vue d’ensemble d’Azure Policy](./overview.md).

1. Sélectionnez l’image suivante pour vous connecter au portail Azure et ouvrir le modèle :

   :::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Bouton pour déployer le modèle Resource Manager pour l’affectation d’une stratégie Azure à Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azurepolicy-assign-builtinpolicy-resourcegroup%2Fazuredeploy.json":::

1. Sélectionnez ou entrez les valeurs suivantes :

   | Name | Valeur |
   |------|-------|
   | Abonnement | Sélectionnez votre abonnement Azure. |
   | Resource group | Sélectionnez **Créer**, spécifiez un nom, puis sélectionnez **OK**. Dans la capture d’écran, le nom du groupe de ressources est _mypolicyquickstart\<Date in MMDD\>rg_. |
   | Emplacement | Sélectionnez une région. Par exemple, **USA Centre**. |
   | Nom de l’affectation de stratégie | Spécifiez un nom d’affectation de stratégie. Vous pouvez utiliser l’écran de définition de stratégie si vous le souhaitez. Par exemple, _Auditer les machines virtuelles qui n’utilisent pas de disques managés_. |
   | Nom du groupe de ressources | Spécifiez le nom du groupe de ressources auquel vous voulez affecter la stratégie. Dans ce démarrage rapide, utilisez la valeur par défaut **[resourceGroup().name]**. **[resourceGroup()](../../azure-resource-manager/templates/template-functions-resource.md#resourcegroup)** est une fonction de modèle qui récupère le groupe de ressources. |
   | ID de définition de stratégie | Spécifiez **/providers/Microsoft.Authorization/policyDefinitions/0a914e76-4921-4c19-b460-a2d36003525a**. |
   | J’accepte les termes et conditions mentionnés ci-dessus | (Sélectionner) |

1. Sélectionnez **Achat**.

Ressources supplémentaires :

- Pour trouver des exemples de modèles supplémentaires, consultez [Modèle de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Authorization&pageNumber=1&sort=Popular).
- Pour obtenir des informations de référence sur les modèles, accédez au [document de référence des modèles Azure](/azure/templates/microsoft.authorization/allversions).
- Pour savoir comment développer des modèles Resource Manager, consultez la [documentation Azure Resource Manager](../../azure-resource-manager/management/overview.md).
- Pour découvrir le déploiement au niveau de l’abonnement, consultez [Créer des groupes de ressources et des ressources au niveau de l’abonnement](../../azure-resource-manager/templates/deploy-to-subscription.md).

## <a name="validate-the-deployment"></a>Valider le déploiement

Sélectionnez **Conformité** dans la partie gauche de la page. Recherchez ensuite l’affectation de stratégie _Auditer les machines virtuelles qui n’utilisent pas de disques managés_ que vous avez créée.

:::image type="content" source="./media/assign-policy-template/policy-compliance.png" alt-text="Bouton pour déployer le modèle Resource Manager pour l’affectation d’une stratégie Azure à Azure." border="false":::

Si des ressources existantes ne sont pas conformes à cette nouvelle affectation, elles apparaissent sous **Ressources non conformes**.

Pour plus d’informations, consultez [Fonctionnement de la conformité](./how-to/get-compliance-data.md#how-compliance-works).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer l’affectation créée, procédez comme suit :

1. Sélectionnez **Conformité** (ou **Affectations**) dans la partie gauche de la page Azure Policy et recherchez l’affectation de stratégie _Auditer les machines virtuelles qui n’utilisent pas de disques managés_ que vous avez créée.

1. Cliquez avec le bouton droit sur l’affectation de stratégie _Auditer les machines virtuelles qui n’utilisent pas de disques managés_ et sélectionnez **Supprimer l’attribution**.

   :::image type="content" source="./media/assign-policy-template/delete-assignment.png" alt-text="Bouton pour déployer le modèle Resource Manager pour l’affectation d’une stratégie Azure à Azure." border="false":::

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez affecté une définition de stratégie intégrée à une étendue et vous avez évalué son rapport de conformité. La définition de stratégie permet de vérifier que toutes les ressources dans l’étendue sont conformes, ainsi que d’identifier celles qui ne le sont pas.

Pour en savoir plus sur l’affectation de stratégies visant à vérifier que les nouvelles ressources sont conformes, suivez le tutoriel :

> [!div class="nextstepaction"]
> [Création et gestion des stratégies](./tutorials/create-and-manage.md)