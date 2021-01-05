---
title: Découvrir les propriétés des ressources
description: Décrit comment rechercher des propriétés de ressource.
ms.topic: conceptual
ms.date: 06/10/2020
ms.openlocfilehash: c8bbe0dcb1bc9dc9751a1dc0d0b98a6368473546
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91327746"
---
# <a name="discover-resource-properties"></a>Découvrir les propriétés des ressources

Avant de créer des modèles Azure Resource Manager, vous devez comprendre les types de ressources disponibles et les valeurs à utiliser dans votre modèle. Cet article présente les méthodes permettant de trouver les propriétés à inclure dans votre modèle.

## <a name="find-resource-provider-namespaces"></a>Trouver les espaces de noms des fournisseurs de ressources

Les ressources d’un modèle ARM sont définies avec un espace de noms de fournisseur de ressources et un type de ressource. Par exemple, Microsoft.Storage/storageAccounts est le nom complet du type de ressource du compte de stockage. Microsoft.Storage est l’espace de noms. Si vous ne connaissez pas déjà les espaces de noms pour les types de ressources que vous souhaitez utiliser, consultez [Fournisseurs de ressources pour les services Azure](../management/azure-services-resource-providers.md).

![Mappage de l'espace de noms du fournisseur de ressources Resource Manager](./media/view-resources/resource-provider-namespace-and-azure-service-mapping.png)

## <a name="export-templates"></a>Exporter des modèles

Le moyen le plus simple d’obtenir les propriétés de modèle pour vos ressources existantes consiste à exporter le modèle. Pour plus d’informations, consultez [Export d’une ressource unique ou de plusieurs ressources vers un modèle dans le Portail Azure](./export-template-portal.md).

## <a name="use-resource-manager-tools-extension"></a>Utilisation de l’extension des outils Resource Manager

Visual Studio Code et l’extension des outils Azure Resource Manager vous aident à voir exactement les propriétés nécessaires pour chaque type de ressource. Ils fournissent IntelliSense et des extraits de code qui simplifient la définition d’une ressource dans votre modèle. Pour plus d’informations, consultez [Démarrage rapide : Créer des modèles Azure Resource Manager avec Visual Studio Code](./quickstart-create-templates-use-visual-studio-code.md#add-an-azure-resource).

La capture d’écran suivante montre une ressource de compte de stockage ajoutée à un modèle :

![Extrait de code de l’extension d’outils Azure Resource Manager](./media/view-resources/resource-manager-tools-extension-snippets.png)

L’extension fournit également une liste d’options pour les propriétés de configuration.

![Valeurs configurables de l'extension des outils du Resource Manager](./media/view-resources/resource-manager-tools-extension-configurable-properties.png)

## <a name="use-template-reference"></a>Utiliser des informations de référence sur les modèles

La référence de modèle Azure Resource Manager est la ressource la plus complète pour le schéma de modèle. Vous pouvez trouver les versions API, le format de modèle et les informations de propriété.

1. Parcourir la [référence du modèle Azure Resource Manager](/azure/templates/).
1. Dans le volet de navigation gauche, sélectionnez **Stockage**, puis sélectionnez **Toutes les ressources**. La page Toutes les ressources résume les types de ressources et les versions.

    ![modèle référence ressource versions](./media/view-resources/resource-manager-template-reference-resource-versions.png)

    Si vous connaissez le type de ressource, vous pouvez accéder directement à cette page avec le format d’URL suivant : `https://docs.microsoft.com/azure/templates/{provider-namespace}/{resource-type}`.

1. Sélectionnez la dernière version. Il est recommandé d’utiliser la version API la plus récente.

    La section **Format du modèle** répertorie toutes les propriétés du compte de stockage. **sku** figure dans la liste :

    ![modèle référence compte de stockage format](./media/view-resources/resource-manager-template-reference-storage-account-sku.png)

    Faites défiler la liste jusqu’à **Objet SKU** dans la section **Valeurs de propriété**. L’article indique les valeurs autorisées pour le nom du SKU :

    ![modèle référence compte de stockage valeurs SKU](./media/view-resources/resource-manager-template-reference-storage-account-sku-values.png)

    À la fin de la page, la section **Modèles de démarrage rapide** répertorie certains modèles de démarrage rapide Azure qui contiennent le type de ressource :

    ![modèle référence compte de stockage modèles de démarrage rapide](./media/view-resources/resource-manager-template-reference-quickstart-templates.png)

La référence de modèle est liée à partir de chaque site de documentation du service Azure.  Par exemple, le [site de documentation Key Vault](../../key-vault/general/overview.md) :

![Informations de référence sur les modèles Resource Manager Key Vault](./media/view-resources/resource-manager-template-reference-key-vault.png)

## <a name="use-resource-explorer"></a>Utilisation de l’Explorateur de ressources

L’Explorateur de ressources est incorporé dans le Portail Azure. Avant d’utiliser cette méthode, vous avez besoin d’un compte de stockage. Si vous n’en avez pas, sélectionnez les boutons suivants pour en créer un :

[![Déployer sur Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-storage-account-create%2fazuredeploy.json)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans la boîte de recherche, entrez **explorateur de ressources**, puis sélectionnez **Explorateur de ressources**.

    ![Capture d’écran de la recherche de l’Explorateur de ressources sur le Portail Azure](./media/view-resources/azure-portal-resource-explorer.png)

1. À partir de la gauche, développez **Abonnements**, puis développez votre abonnement Azure. Vous pouvez trouver le compte de stockage sous **Fournisseurs** ou **Groupe de ressources**.

    ![Explorateur de ressources portail Azure](./media/view-resources/azure-portal-resource-explorer-home.png)

    - **Fournisseurs** : développez **Fournisseurs** -> **Microsoft.Storage** -> **storageAccounts**, puis sélectionnez votre compte de stockage.
    - **ResourceGroups** : sélectionnez le groupe de ressources qui contient le compte de stockage, sélectionnez **Ressources**, puis sélectionnez le compte de stockage.

    À droite, vous voyez la configuration SKU pour le compte de stockage existant semblable au :

    ![Référence SKU du compte de stockage Explorateur de ressources du Portail Azure](./media/view-resources/azure-portal-resource-explorer-sku.png)

## <a name="use-resourcesazurecom"></a>Utiliser Resources.azure.com

Resources.azure.com est un site web public accessible par toute personne disposant d’un abonnement Azure. Ce site est en préversion.  Envisagez d’utiliser [Explorateur de ressources](#use-resource-explorer) à la place. Cet outil fournit les fonctionnalités suivantes :

- Découvrir les API de gestion des ressources Azure.
- Obtenir la documentation et les informations de schéma API.
- Effectuer directement les appels d’API dans vos abonnements.

Pour montrer comment récupérer des informations de schéma à l’aide de cet outil, vous avez besoin d’un compte de stockage. Si vous n’en avez pas, sélectionnez les boutons suivants pour en créer un :

[![Déployer sur Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-storage-account-create%2fazuredeploy.json)

1. Parcourez jusqu’à [resources.azure.com](https://resources.azure.com/). Il faut quelques instants pour que l’outil se remplisse dans le volet gauche.
1. Sélectionner **Abonnements**.

    ![mappage de l’API resource.azure.com](./media/view-resources/resources-azure-com-api-mapping.png)

    Le nœud situé à gauche correspond à l’appel d’API situé à droite. Vous pouvez effectuer l’appel d’API en sélectionnant le bouton **GET**.
1. À partir de la gauche, développez **Abonnements**, puis développez votre abonnement Azure. Vous pouvez trouver le compte de stockage sous **Fournisseurs** ou **Groupe de ressources**.

    - **Fournisseurs** : développez **Fournisseurs** -> **Microsoft.Storage** -> **storageAccounts**, puis sélectionnez votre compte de stockage.
    - **ResourceGroups** : sélectionnez le groupe de ressources qui contient le compte de stockage puis sélectionnez **Ressources**.

    À droite, vous voyez la configuration SKU pour le compte de stockage existant semblable au :

    ![Référence SKU du compte de stockage Explorateur de ressources du Portail Azure](./media/view-resources/azure-portal-resource-explorer-sku.png)

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à rechercher des informations de schéma de modèle. Pour apprendre à créer des modèles, consultez [Comprendre la structure et la syntaxe des modèles Resource Manager](./template-syntax.md).
