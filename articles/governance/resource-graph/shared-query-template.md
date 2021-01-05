---
title: 'Démarrage rapide : Créer une requête partagée avec des modèles'
description: Dans ce guide de démarrage rapide, vous utilisez un modèle Azure Resource Manager (modèle ARM) pour créer une requête partagée Resource Graph qui compte les machines virtuelles par système d’exploitation.
ms.date: 10/14/2020
ms.topic: quickstart
ms.custom: subject-armqs
ms.openlocfilehash: a629dd5325fc20d6f173d9f4e0524885af8fdf49
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92057006"
---
# <a name="quickstart-create-a-shared-query-by-using-an-arm-template"></a>Démarrage rapide : Créer une requête partagée à l’aide d’un modèle ARM

Les requêtes Resource Graph peuvent être enregistrées comme _requête privée_ ou _requête partagée_. Une requête privée est enregistrée dans le profil du portail d’une personne et n’est visible par personne d’autre. Une requête partagée est un objet Resource Manager qui peut être partagé avec d’autres utilisateurs via des autorisations et un accès en fonction du rôle. Une requête partagée fournit une exécution courante et cohérente de la découverte des ressources. Ce guide de démarrage rapide utilise un modèle Azure Resource Manager (modèle ARM) pour créer une requête partagée.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

:::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Déployer le modèle Resource Manager pour créer une requête partagée dans Azure" border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fresourcegraph-sharedquery-countos%2Fazuredeploy.json":::

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Dans ce guide de démarrage rapide, vous créez une requête partagée appelée _Compter les VM par OS_. Pour tester cette requête dans le kit SDK ou dans le portail avec l’Explorateur Resource Graph, consultez [Exemples - Compter les VM par OS](./samples/starter.md#count-os).

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/resourcegraph-sharedquery-countos/).

:::code language="json" source="~/quickstart-templates/resourcegraph-sharedquery-countos/azuredeploy.json":::

La ressource définie dans le modèle est :

- [Microsoft.ResourceGraph/queries](/azure/templates/microsoft.resourcegraph/queries)

## <a name="deploy-the-template"></a>Déployer le modèle

> [!NOTE]
> Le service Azure Resource Graph est gratuit. Pour plus d’informations, consultez [Vue d’ensemble d’Azure Resource Graph](./overview.md).

1. Sélectionnez l’image suivante pour vous connecter au portail Azure et ouvrir le modèle :

   :::image type="content" source="../../media/template-deployments/deploy-to-azure.svg" alt-text="Déployer le modèle Resource Manager pour créer une requête partagée dans Azure" border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fresourcegraph-sharedquery-countos%2Fazuredeploy.json":::

1. Sélectionnez ou entrez les valeurs suivantes :

   | Nom | Valeur |
   |------|-------|
   | Abonnement | Sélectionnez votre abonnement Azure. |
   | Resource group | Sélectionnez **Créer**, spécifiez un nom, puis sélectionnez **OK**. |
   | Emplacement | Sélectionnez une région. Par exemple, **USA Centre**. |
   | Nom de la requête | Laissez la valeur par défaut : **Count VMs by OS**. |
   | Code de la requête | Laissez la valeur par défaut : `Resources | where type =~ 'Microsoft.Compute/virtualMachines' | summarize count() by tostring(properties.storageProfile.osDisk.osType)` |
   | Description de la requête | Laissez la valeur par défaut : **This shared query counts all virtual machine resources and summarizes by the OS type.** |
   | J’accepte les termes et conditions mentionnés ci-dessus | (Sélectionner) |

1. Sélectionnez **Achat**.

Ressources supplémentaires :

- Pour trouver des exemples de modèles supplémentaires, consultez [Modèle de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Authorization&pageNumber=1&sort=Popular).
- Pour obtenir des informations de référence sur les modèles, accédez au [document de référence des modèles Azure](/azure/templates/microsoft.resourcegraph/allversions).
- Pour savoir comment développer des modèles Resource Manager, consultez la [documentation Azure Resource Manager](../../azure-resource-manager/management/overview.md).
- Pour découvrir le déploiement au niveau de l’abonnement, consultez [Créer des groupes de ressources et des ressources au niveau de l’abonnement](../../azure-resource-manager/templates/deploy-to-subscription.md).

## <a name="validate-the-deployment"></a>Valider le déploiement

Pour exécuter la nouvelle requête partagée, suivez ces étapes :

1. Dans la barre de recherche du portail, recherchez et sélectionnez **Requêtes Resource Graph**.

1. Sélectionnez la requête partagée appelée **Compter les VM par OS**, puis sélectionnez l’onglet **Résultats** dans la page **Vue d’ensemble**.

Vous pouvez aussi ouvrir la requête partagée depuis l’Explorateur Resource Graph :

1. Dans la barre de recherche du portail, recherchez et sélectionnez **Explorateur Resource Graph**.

1. Sélectionnez le bouton **Ouvrir une requête**.

1. Remplacez le **Type** par _Requêtes partagées_. Si vous ne voyez pas **Compter les VM par OS** dans la liste, utilisez la zone de filtre pour limiter les résultats. Une fois que la requête partagée **Compter les VM par OS** est visible, sélectionnez son nom.

1. Une fois que la requête est chargée, sélectionnez le bouton **Exécuter la requête**. Les résultats s’affichent sous l’onglet **Résultats**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer la requête partagée créée, suivez ces étapes :

1. Dans la barre de recherche du portail, recherchez et sélectionnez **Requêtes Resource Graph**.

1. Cochez la case à côté de la requête partagée appelée **Compter les VM par OS**.

1. Sélectionnez le bouton **Supprimer** en haut de la page.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé une requête partagée Resource Graph.

Pour en savoir plus sur les requêtes partagées, continuez avec le tutoriel pour :

> [!div class="nextstepaction"]
> [Gérer les requêtes dans le portail Azure](./tutorials/create-share-query.md)