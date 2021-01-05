---
title: Gérer les ressources avec le portail Azure
description: Utilisez le portail Azure et Azure Resource Manager pour déployer vos ressources dans un groupe de ressources de votre abonnement.
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: b87ac48bbaec7f94d5e75939cf5ec17df0ff2d2f
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92668771"
---
# <a name="deploy-resources-with-arm-templates-and-azure-portal"></a>Déployer des ressources à l’aide de modèles ARM et le portail Azure

Découvrez comment utiliser le [portail Azure](https://portal.azure.com) avec des [modèles Resource Manager](overview.md) pour déployer vos ressources Azure. Pour en savoir plus sur la gestion de vos ressources, consultez [Gérer des ressources avec le portail Azure](../management/manage-resources-portal.md).

Le déploiement de ressources Azure à l’aide du portail Azure implique généralement deux étapes :

- Créez un groupe de ressources.
- Déployez des ressources sur un groupe de ressources.

En outre, vous pouvez créer un modèle Resource Manager personnalisé pour déployer des ressources Azure.

Cet article illustre les deux méthodes.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

1. Pour créer un groupe de ressources, sélectionnez **Groupes de ressources** depuis le [portail Azure](https://portal.azure.com).

   ![Sélectionner des groupes de ressources](./media/deploy-portal/select-resource-groups.png)

1. Dans Groupes de ressources, sélectionnez **Ajouter**.

   ![Ajouter un groupe de ressource](./media/deploy-portal/add-resource-group.png)

1. Sélectionnez ou entrez les valeurs de propriété suivantes :

    - **Abonnement** : Sélectionnez un abonnement Azure.
    - **Groupe de ressources**  : Nommez le groupe de ressources.
    - **Région**  : Spécifiez un emplacement Azure. Il s’agit de l’emplacement où le groupe de ressources stocke les métadonnées relatives aux ressources. Pour des raisons de conformité, vous souhaiterez peut-être indiquer où sont stockées métadonnées. En règle générale, nous vous recommandons de spécifier l’emplacement où se trouvera la plupart de vos ressources. L’utilisation du même emplacement permet de simplifier votre modèle.

   ![Définir les valeurs d’un groupe](./media/deploy-portal/set-group-properties.png)

1. Sélectionnez **Revoir + créer**.
1. Passez en revue les valeurs, puis sélectionnez **Créer**.
1. Sélectionnez **Actualiser** pour voir le nouveau groupe de ressources dans la liste.

## <a name="deploy-resources-to-a-resource-group"></a>Déployer des ressources sur un groupe de ressources

Une fois votre groupe de ressources créé, vous pouvez y déployer le groupe à partir de Marketplace. Marketplace fournit des solutions prédéfinies pour les scénarios courants.

1. Pour démarrer un déploiement, sélectionnez **Créer une ressource** depuis le [portail Azure](https://portal.azure.com).

   ![Nouvelle ressource](./media/deploy-portal/new-resources.png)

1. Recherchez le type de ressource que vous souhaitez déployer. Les ressources sont organisées en catégories. Si vous ne voyez pas la solution que vous souhaitez déployer, vous pouvez lancer une recherche sur la Place de marché. La capture d’écran qui suit montre que le serveur Ubuntu est sélectionné.

   ![sélectionner le type de ressource](./media/deploy-portal/select-resource-type.png)

1. Selon le type de ressource que vous avez sélectionné, vous devrez définir une collection de propriétés pertinentes avant le déploiement. Pour tous les types, vous devez sélectionner un groupe de ressources de destination. L’illustration suivante montre comment créer une machine virtuelle Linux et la déployer dans le groupe de ressources que vous avez créé.

   ![Créer un groupe de ressources](./media/deploy-portal/select-existing-group.png)

   Vous pouvez décider de créer un groupe de ressources lors du déploiement de vos ressources. Sélectionnez **Créer** et donnez un nom au groupe de ressources.

1. Votre déploiement se met en route. Ce déploiement peut prendre plusieurs minutes. Certaines ressources prennent plus de temps que d’autres. Vous recevez une notification une fois le déploiement terminé. Sélectionnez **Accéder aux ressources** pour ouvrir

   ![Afficher une notification](./media/deploy-portal/view-notification.png)

1. Après avoir déployé vos ressources, vous pouvez ajouter d’autres ressources au groupe de ressources en cliquant sur **Ajouter**.

   ![Ajouter une ressource](./media/deploy-portal/add-resource.png)

Bien que vous ne l’ayez pas vu, le portail utilisait un modèle Resource Manager pour déployer les ressources que vous avez sélectionnées. Vous pouvez trouver le modèle à partir de l’historique de déploiement. Pour plus d’informations, consultez [Exporter un modèle après le déploiement](export-template-portal.md#export-template-after-deployment).

## <a name="deploy-resources-from-custom-template"></a>Déployer des ressources à partir d’un modèle personnalisé

Si vous souhaitez effectuer un déploiement sans utiliser l’un des modèles de Marketplace, vous pouvez créer un modèle personnalisé qui définit l’infrastructure de votre solution. Pour apprendre à créer des modèles, consultez [Comprendre la structure et la syntaxe des modèles ARM](template-syntax.md).

> [!NOTE]
> L’interface du portail ne prend pas en charge le référencement à un [secret d’un coffre de clés](key-vault-parameter.md). Utilisez plutôt [PowerShell](deploy-powershell.md) ou [Azure CLI](deploy-cli.md) pour déployer votre modèle localement ou à partir d’un URI externe.

1. Pour déployer un modèle personnalisé grâce au portail, sélectionnez **Créer une ressource** , puis recherchez le **modèle**. et sélectionnez **Déploiement de modèle**.

   ![Rechercher un déploiement de modèle](./media/deploy-portal/search-template.png)

1. Sélectionnez **Create** (Créer).
1. Vous voyez plusieurs options pour créer un modèle :

    - **Créer votre propre modèle dans l’éditeur**  : Créez votre propre modèle dans l’éditeur de modèle du portail.
    - **Modèles communs** : Sélectionnez parmi les solutions courantes.
    - **Charger un modèle de démarrage rapide GitHub**  : Sélectionnez parmi les [modèles de démarrage rapide](https://azure.microsoft.com/resources/templates/).

   ![Options d’affichage](./media/deploy-portal/see-options.png)

    Ce tutoriel fournit les instructions nécessaires au chargement d’un modèle de démarrage rapide.

1. Sous **Charger un modèle de démarrage rapide GitHub** , tapez ou sélectionnez **101-storage-account-create**.

    Deux options s'offrent à vous :

    - **Sélectionner un modèle** : déployez le modèle.
    - **Modifier le modèle** : modifiez le modèle de démarrage rapide, avant de le déployer.

1. Sélectionnez **Modifier le modèle** pour explorer l’éditeur de modèle du portail. Le modèle est chargé dans l’éditeur. Il y a deux paramètres : **storageAccountType** et **location**.

   ![Créer un modèle](./media/deploy-portal/show-json.png)

1. Effectuez une modification relativement mineure au modèle. Par exemple, mettez à jour la variable **storageAccountName** :

    ```json
    "storageAccountName": "[concat('azstore', uniquestring(resourceGroup().id))]"
    ```

1. Sélectionnez **Enregistrer**. Vous voyez maintenant l’interface de déploiement de modèle du portail. Vous voyez les deux paramètres que vous avez définis dans le modèle.
1. Entrez ou sélectionnez les valeurs de propriété :

    - **Abonnement** : Sélectionnez un abonnement Azure.
    - **Groupe de ressources**  : Sélectionnez **Créer** et donnez un nom.
    - **Emplacement** : Sélectionnez un emplacement Azure.
    - **Type de compte de stockage** : Utilisez la valeur par défaut.
    - **Emplacement** : Utilisez la valeur par défaut.
    - **J’accepte les termes et conditions mentionnés ci-dessus** : (cochez la case)

1. Sélectionnez **Achat**.

## <a name="next-steps"></a>Étapes suivantes

- Pour visualiser les journaux d’audit, voir [Opérations d’audit avec Resource Manager](../management/view-activity-logs.md).
- Pour résoudre les erreurs de déploiement, consultez [Voir les opérations de déploiement](deployment-history.md).
- Pour exporter un modèle à partir d’un déploiement ou d’un groupe de ressources, consultez [Exporter des modèles ARM](export-template-portal.md).
- Pour déployer votre service sur plusieurs régions de façon sécurisée, consultez [Azure Deployment Manager](deployment-manager-overview.md).
