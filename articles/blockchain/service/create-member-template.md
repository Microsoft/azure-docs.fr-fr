---
title: Créer un membre Azure Blockchain Service à l’aide d’un modèle Azure Resource Manager
description: Découvrez comment créer un membre Azure Blockchain Service à l’aide d’un modèle Azure Resource Manager.
services: azure-resource-manager
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs, references_regions
ms.date: 09/16/2020
ms.openlocfilehash: e9893336f2e6633519853aceecc945ee6bf0bf4b
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91292759"
---
# <a name="quickstart-create-an-azure-blockchain-service-member-using-an-arm-template"></a>Démarrage rapide : Créer un membre d’Azure Blockchain Service à l’aide d’un modèle ARM

Dans ce guide de démarrage rapide, vous allez déployer un nouveau membre blockchain et un nouveau consortium dans Azure Blockchain Service à l’aide d’un modèle ARM (Azure Resource Manager). Un membre Azure Blockchain Service est un nœud blockchain dans un réseau blockchain de consortium privé. Lors du provisionnement d’un membre, vous pouvez créer ou rejoindre un réseau de consortium. Vous avez besoin d’au moins un membre pour former un réseau de consortium. Le nombre de membres blockchain nécessaires par participant dépend de votre scénario. Les participants au consortium peuvent avoir un ou plusieurs membres blockchain ou ils peuvent partager des membres avec d’autres participants. Pour plus d’informations sur les consortiums, consultez [Consortium Azure Blockchain Service](consortium.md).

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-blockchain-asaservice%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/201-blockchain-asaservice/).

:::code language="json" source="~/quickstart-templates/201-blockchain-asaservice/azuredeploy.json":::

Ressources Azure définies dans le modèle :

* [**Microsoft.Blockchain/blockchainMembers**](/azure/templates/microsoft.blockchain/blockchainmembers)

## <a name="deploy-the-template"></a>Déployer le modèle

1. Sélectionnez le lien suivant pour vous connecter à Azure et ouvrir un modèle.

    [![Déployer sur Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-blockchain-asaservice%2Fazuredeploy.json)

1. Spécifiez les paramètres du membre Azure Blockchain Service.

    Paramètre | Description
    --------|------------
    Abonnement | Sélectionnez l’abonnement Azure que vous souhaitez utiliser pour votre serveur. Si vous avez plusieurs abonnements, sélectionnez l’abonnement dans lequel la ressource est facturée.
    Resource group | Créez un nom de groupe de ressources ou choisissez un nom de groupe existant dans votre abonnement.
    Région | Choisissez une région où créer le groupe de ressources. Tous les membres du consortium doivent se trouver au même emplacement. Les emplacements disponibles pour le déploiement sont *westeurope, eastus, southeastasia, westeurope, northeurope, westus2* et *japaneast*. Les fonctionnalités peuvent ne pas être disponibles dans certaines régions. Azure Blockchain Data Manager est disponible dans les régions Azure suivantes : USA Est et Europe Ouest.
    Nom du membre Bc | Choisissez un nom unique pour le membre Azure Blockchain Service. Le nom du membre blockchain doit être uniquement composé de lettres minuscules et de chiffres. Le premier caractère doit être une lettre. Le nom doit comprendre entre 2 et 20 caractères.
    Nom du consortium | Entrez un nom unique. Pour plus d’informations sur les consortiums, consultez [Consortium Azure Blockchain Service](consortium.md).
    Mot de passe du membre | Le mot de passe du nœud de transaction par défaut du membre. Utilisez le mot de passe pour l’authentification de base lorsque vous vous connectez au point de terminaison public du nœud de transaction par défaut du membre blockchain.
    Mot de passe du compte de gestion du consortium | Le mot de passe du compte du consortium sert à chiffrer la clé privée du compte Ethereum créé pour votre membre. Il est utilisé pour la gestion du consortium.
    Niveau de référence SKU | Niveau tarifaire pour votre nouveau service. Choisissez le niveau **Standard** ou **De base**. Utilisez le niveau *De base* pour le développement, les tests et les preuves de concept. Utilisez le niveau *Standard* pour les déploiements en production. Utilisez également le niveau *Standard* si vous utilisez Blockchain Data Manager ou si vous envoyez un volume élevé de transactions privées. Le passage du niveau tarifaire de base au niveau standard (ou inversement) après la création du membre n'est pas pris en charge.
    Nom de la référence SKU | Configuration des nœuds et coût de votre nouveau service. Utilisez **B0** pour De base ou **S0** pour Standard.
    Emplacement | Choisissez un emplacement où créer le membre. Par défaut, l’emplacement du groupe de ressources (`[resourceGroup().location]`) est utilisé. Tous les membres du consortium doivent se trouver au même emplacement. Les emplacements disponibles pour le déploiement sont *westeurope, eastus, southeastasia, westeurope, northeurope, westus2* et *japaneast*. Les fonctionnalités peuvent ne pas être disponibles dans certaines régions. Azure Blockchain Data Manager est disponible dans les régions Azure suivantes : USA Est et Europe Ouest.

1. Sélectionnez **Vérifier + créer** pour vérifier et déployer le modèle.

  Le portail Azure est utilisé ici pour déployer le modèle. Vous pouvez également utiliser Azure PowerShell, Azure CLI ou l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Vous pouvez utiliser le portail Azure pour afficher les détails du membre Azure Blockchain Service déployé. Dans le portail, accédez au groupe de ressources contenant votre membre Azure Blockchain Service. Sélectionnez le membre blockchain que vous avez créé.

![Détails dans le portail Azure de la vue d’ensemble du membre Azure Blockchain déployé](./media/create-member-template/deployed-member.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez utiliser le membre blockchain que vous avez créé pour le guide de démarrage rapide ou tutoriel suivant. Quand vous n’en aurez plus besoin, vous pourrez supprimer les ressources en supprimant le groupe de ressources que vous avez créé pour le guide de démarrage rapide.

Pour supprimer le groupe de ressources :

1. Dans le portail Azure, accédez à **Groupe de ressources** dans le volet de navigation de gauche et sélectionnez le groupe de ressources que vous souhaitez supprimer.
2. Sélectionnez **Supprimer le groupe de ressources**. Validez la suppression en entrant le nom du groupe de ressources, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé un membre Azure Blockchain Service et un nouveau consortium. Dans le guide de démarrage rapide qui suit, vous allez utiliser Azure Blockchain Development Kit pour Ethereum afin d’effectuer une jonction à un membre Azure Blockchain Service.

> [!div class="nextstepaction"]
> [Utiliser Visual Studio Code pour se connecter à Azure Blockchain Service](connect-vscode.md)