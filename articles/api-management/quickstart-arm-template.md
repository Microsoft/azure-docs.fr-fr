---
title: Démarrage rapide - Créer une instance Azure API Management avec un modèle ARM
description: Découvrez comment créer une instance Azure API Management dans le niveau Développeur en utilisant un modèle Azure Resource Manager (modèle ARM).
services: azure-resource-manager
author: dlepow
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: danlep
ms.date: 10/09/2020
ms.openlocfilehash: 1610c51b613712c06410247e2eb673a50a9988b3
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792240"
---
# <a name="quickstart-create-a-new-azure-api-management-service-instance-using-an-arm-template"></a>Démarrage rapide : Créer une instance de service Azure API Management avec un modèle ARM

Ce guide de démarrage rapide explique comment utiliser un modèle Azure Resource Manager (modèle ARM) pour créer une instance du service Azure API Management (APIM). APIM permet aux organisations de publier des API à l’intention des développeurs internes, partenaires et externes pour libérer le potentiel de leurs données et services. Il offre les compétences essentielles qui garantissent un programme d’API réussi au travers de l’engagement des développeurs, des perspectives commerciales, de l’analytique, de la sécurité et de la protection. APIM vous permet de créer et de gérer des passerelles d’API modernes pour des services back-end existants, où qu’ils soient hébergés. Pour plus d'informations, consultez la [Vue d’ensemble](api-management-key-concepts.md).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure** . Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azure-api-management-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-azure-api-management-create/).

:::code language="json" source="~/quickstart-templates/101-azure-api-management-create/azuredeploy.json":::

La ressource suivante est définie dans le modèle :

- **[Microsoft.ApiManagement/service](/azure/templates/microsoft.apimanagement/service)**

Vous trouverez d’autres exemples de modèles Azure API Management dans [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Apimanagement&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Déployer le modèle

1. Cliquez sur l’image ci-après pour vous connecter à Azure et ouvrir un modèle. Le modèle crée une instance du service API Management avec un nom généré automatiquement.

    [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azure-api-management-create%2Fazuredeploy.json)

    Dans cet exemple, l’instance est configurée dans le niveau Développeur, ce qui constitue une option économique pour évaluer Azure API Management. Ce niveau n’est pas destiné à la production. Pour en savoir plus sur la mise à l’échelle des niveaux du service Gestion des API, consultez [Mettre à niveau et mettre à l’échelle](upgrade-and-scale.md).

1. Sélectionnez ou entrez les valeurs suivantes.
    - **Abonnement** : sélectionnez un abonnement Azure.
    - **Groupe de ressources**  : sélectionnez **Créer** , entrez un nom unique pour le groupe de ressources, puis sélectionnez **OK** .
    - **Région**  : sélectionnez un emplacement pour le groupe de ressources. Exemple : **USA Centre** .
    - **Adresse e-mail de l’éditeur**  : entrez l’adresse e-mail où seront reçues les notifications.
    - **Nom de l’éditeur**  : entrez le nom que vous voulez attribuer à l’éditeur d’API.
    - **Référence SKU**  : acceptez la valeur par défaut **Développeur** .
    - **Nombre de références SKU**  : acceptez la valeur par défaut.
    - **Emplacement**  : acceptez l’emplacement généré pour le service API Management.

    :::image type="content" source="media/quickstart-arm-template/create-instance-template.png" alt-text="Propriétés du modèle API Management":::

1. Sélectionnez **Vérifier + créer** , puis passez en revue les conditions générales. Si vous les acceptez, sélectionnez **Créer** .

    > [!TIP]
    >  La création et l’activation d’un service API Management au niveau Développeur peuvent prendre entre 30 et 40 minutes.

1. Une fois l’instance créée avec succès, vous recevez une notification :

    :::image type="content" source="media/quickstart-arm-template/deployment-notification.png" alt-text="Propriétés du modèle API Management":::

 Le portail Azure est utilisé pour déployer le modèle. Outre le portail Azure, vous pouvez également utiliser Azure PowerShell, l’interface Azure CLI et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-cli.md).

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Utilisez le portail Azure pour vérifier les ressources déployées ou servez-vous d’outils comme Azure CLI ou Azure PowerShell pour lister les ressources déployées.

1. Sur le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Services Gestion des API** , puis sélectionnez l’instance de service que vous avez créée.
1. Passez en revue les propriétés de votre service dans la page **Vue d’ensemble** .

:::image type="content" source="media/quickstart-arm-template/service-instance-created.png" alt-text="Propriétés du modèle API Management":::

Une fois votre instance Gestion des API en ligne, vous êtes prêt à l’utiliser. Commencez par le tutoriel [Importer et publier votre première API](import-and-publish.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous prévoyez d’utiliser d’autres tutoriels, vous pouvez conserver cette instance API Management. Quand vous n’en avez plus besoin, supprimez le groupe de ressources, ce qui supprime également les ressources qu’il contient.

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Groupes de ressources** . Vous pouvez également sélectionner **Groupes de ressources** depuis la page **d’accueil** .
1. Sur la page **Groupes de ressources** , sélectionnez votre groupe de ressources.
1. Sur la page Groupe de ressources, sélectionnez **Supprimer un groupe de ressources** .

    Supprimer le groupe de ressources
1. Saisissez le nom de votre groupe de ressources, puis sélectionnez **Supprimer** .

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Importer et publier votre première API](import-and-publish.md)
