---
title: Créer une ressource Cognitive Services avec Azure CLI
titleSuffix: Azure Cognitive Services
description: Commencez à utiliser Azure Cognitive Services en créant et en vous abonnant à une ressource avec Azure CLI.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
keywords: services cognitifs, intelligence cognitive, solutions cognitives, services ia
ms.topic: conceptual
ms.date: 09/14/2020
ms.author: aahi
ms.openlocfilehash: e276d96e8a81b435ec4d0c270cf818555d512ae0
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94368897"
---
# <a name="quickstart-create-a-cognitive-services-resource-using-the-azure-command-line-interfacecli"></a>Démarrage rapide : Créer une ressource Cognitive Services avec Azure CLI

Utilisez ce guide de démarrage rapide pour commencer à utiliser Azure Cognitive Services avec [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest).

La solution Azure Cognitive Services correspond à des services cloud avec des API REST et des kits SDK de bibliothèque de client destinés à aider les développeurs à intégrer une intelligence cognitive dans des applications sans connaissances ni compétences directes en intelligence artificielle (IA) ou en science des données. Azure Cognitive Services permet aux développeurs d’ajouter facilement des fonctionnalités cognitives dans leurs applications à l’aide de solutions cognitives capables de voir, entendre, parler, comprendre et même commencer à raisonner.

Les services Cognitive Services sont représentés par des [ressources](../azure-resource-manager/management/manage-resources-portal.md) Azure que vous créez dans votre abonnement Azure. Après avoir créé la ressource, utilisez les clés et le point de terminaison générés pour vous pour authentifier vos applications.

Dans ce guide de démarrage rapide, vous allez apprendre à vous inscrire à Azure Cognitive Services et à créer un compte disposant d’un abonnement monoservice ou multiservice, à l’aide d’[Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest). Ces services sont représentés par des [ressources](../azure-resource-manager/management/manage-resources-portal.md) Azure qui vous permettent de vous connecter à une ou plusieurs des API Cognitive Services.

[!INCLUDE [cognitive-services-subscription-types](../../includes/cognitive-services-subscription-types.md)]

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure valide : [créez-en un](https://azure.microsoft.com/free/cognitive-services) gratuitement.
* [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest)

## <a name="install-the-azure-cli-and-sign-in"></a>Installer Azure CLI et se connecter

Installez [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest). Pour vous connecter à votre installation locale de l’interface CLI, exécutez la commande [az login](/cli/azure/reference-index#az-login) :

```azurecli-interactive
az login
```

Vous pouvez également utiliser le bouton vert **Essayer** pour exécuter ces commandes dans votre navigateur.

## <a name="create-a-new-azure-cognitive-services-resource-group"></a>Créer un groupe de ressources Azure Cognitive Services

Avant de créer une ressource Cognitive Services, vous devez disposer d’un groupe de ressources Azure pour la contenir. Quand vous créez une ressource, vous avez le choix entre créer un groupe de ressources ou utiliser un groupe existant. Cet article montre comment créer un groupe de ressources.

### <a name="choose-your-resource-group-location"></a>Choisir l’emplacement de votre groupe de ressources

Pour créer une ressource, il vous faut un emplacement Azure disponible pour votre abonnement. Vous pouvez récupérer une liste des emplacements disponibles avec la commande [az account list-locations](/cli/azure/account#az-account-list-locations). La plupart des Services Cognitive est accessible depuis plusieurs emplacements. Choisissez l’emplacement le plus proche de vous, ou consultez les emplacements disponibles pour le service.

> [!IMPORTANT]
> * Retenez votre emplacement Azure, car vous en aurez besoin lors de l’appel à Azure Cognitive Services.
> * La disponibilité de certains services Cognitive Services peut varier selon la région. Pour plus d’informations, consultez les [Produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services).

```azurecli-interactive
az account list-locations \
    --query "[].{Region:name}" \
    --out table
```

Après avoir choisi un emplacement Azure, créez un groupe de ressources dans Azure CLI par le biais de la commande [az group create](/cli/azure/group#az-group-create).

Dans l’exemple ci-dessous, remplacez l’emplacement Azure `westus2` par l’un des emplacements disponibles pour votre abonnement.

```azurecli-interactive
az group create \
    --name cognitive-services-resource-group \
    --location westus2
```

## <a name="create-a-cognitive-services-resource"></a>Créer une ressource Cognitive Services

### <a name="choose-a-cognitive-service-and-pricing-tier"></a>Choisissez un service cognitif et son niveau tarifaire

Lorsque vous créez une nouvelle ressource, vous devez connaître le « type » du service que vous souhaitez utiliser, ainsi que le [niveau tarifaire](https://azure.microsoft.com/pricing/details/cognitive-services/) (ou référence SKU) souhaité. Ces informations et d’autres vous serviront de paramètres lors de la création de la ressource.

### <a name="multi-service"></a>Multiservice

| Service                    | Type                      |
|----------------------------|---------------------------|
| Plusieurs services. Pour plus d’informations, consultez la page sur la [tarification](https://azure.microsoft.com/pricing/details/cognitive-services/).            | `CognitiveServices`     |


> [!NOTE]
> De nombreux Cognitive Services ci-dessous ont un niveau gratuit que vous pouvez utiliser pour tester le service. Pour utiliser le niveau gratuit, utilisez `F0` en tant que référence SKU de votre ressource.

### <a name="vision"></a>Vision

| Service                    | Type                      |
|----------------------------|---------------------------|
| Vision par ordinateur            | `ComputerVision`          |
| Custom Vision - Prédiction | `CustomVision.Prediction` |
| Custom Vision - Formation   | `CustomVision.Training`   |
| Face                       | `Face`                    |
| Form Recognizer            | `FormRecognizer`          |
| Ink Recognizer             | `InkRecognizer`           |

### <a name="search"></a>Recherche

| Service            | Type                  |
|--------------------|-----------------------|
| Suggestion automatique Bing   | `Bing.Autosuggest.v7` |
| Recherche personnalisée Bing | `Bing.CustomSearch`   |
| Recherche d’entité Bing | `Bing.EntitySearch`   |
| Bing Search        | `Bing.Search.v7`      |
| Vérification orthographique Bing   | `Bing.SpellCheck.v7`  |

### <a name="speech"></a>Speech

| Service            | Type                 |
|--------------------|----------------------|
| Services Speech    | `SpeechServices`     |
| Reconnaissance vocale | `SpeakerRecognition` |

### <a name="language"></a>Langage

| Service            | Type                |
|--------------------|---------------------|
| Compréhension de formulaire | `FormUnderstanding` |
| LUIS               | `LUIS`              |
| QnA Maker          | `QnAMaker`          |
| Analyse de texte     | `TextAnalytics`     |
| Traduction de texte   | `TextTranslation`   |

### <a name="decision"></a>Décision

| Service           | Type               |
|-------------------|--------------------|
| Le détecteur d’anomalies  | `AnomalyDetector`  |
| Content Moderator | `ContentModerator` |
| Personalizer      | `Personalizer`     |

Vous trouverez une liste des « types » de services cognitifs disponibles avec la commande [az cognitiveservices account list-kinds](/cli/azure/cognitiveservices/account?view=azure-cli-latest#az-cognitiveservices-account-list-kinds) :

```azurecli-interactive
az cognitiveservices account list-kinds
```

### <a name="add-a-new-resource-to-your-resource-group"></a>Ajouter une ressource à votre groupe de ressources

Pour créer une ressource Cognitive Services et s’y abonner, utilisez la commande [az cognitiveservices account create](/cli/azure/cognitiveservices/account?view=azure-cli-latest#az-cognitiveservices-account-create). Cette commande ajoute une nouvelle ressource facturable au groupe de ressources créé précédemment. Lorsque vous créez votre nouvelle ressource, vous devez connaître le « type » du service que vous souhaitez utiliser, ainsi que son niveau tarifaire (ou référence SKU) et un emplacement Azure :

Vous pouvez créer une ressource F0 (gratuite) pour détecteur d’anomalies, nommé `anomaly-detector-resource` avec la commande ci-dessous.

```azurecli-interactive
az cognitiveservices account create \
    --name anomaly-detector-resource \
    --resource-group cognitive-services-resource-group \
    --kind AnomalyDetector \
    --sku F0 \
    --location westus2 \
    --yes
```

[!INCLUDE [Register Azure resource for subscription](./includes/register-resource-subscription.md)]

## <a name="get-the-keys-for-your-resource"></a>Obtenir les clés pour votre ressource

Pour vous connecter à votre installation locale de l’interface CLI, utilisez la commande [az login](/cli/azure/reference-index?view=azure-cli-latest#az-login).

```azurecli-interactive
az login
```

Utilisez la commande [az cognitiveservices account keys list](/cli/azure/cognitiveservices/account/keys?view=azure-cli-latest#az-cognitiveservices-account-keys-list) pour obtenir les clés pour votre ressource Cognitive Service.

```azurecli-interactive
    az cognitiveservices account keys list \
    --name anomaly-detector-resource \
    --resource-group cognitive-services-resource-group
```

[!INCLUDE [cognitive-services-environment-variables](../../includes/cognitive-services-environment-variables.md)]

## <a name="pricing-tiers-and-billing"></a>Niveaux tarifaires et facturation

Les niveaux tarifaires (et le montant facturé) sont basés sur le nombre de transactions que vous envoyez à l’aide de vos informations d’authentification. Chaque niveau tarifaire spécifie :
* le nombre maximal de transactions par seconde (TPS) autorisées ;
* les fonctionnalités de service activées dans le niveau tarifaire ;
* le coût d’un nombre prédéfini de transactions. Si vous dépassez ce nombre, des frais supplémentaires vous seront facturés, comme indiqué dans les [détails de la tarification](https://azure.microsoft.com/pricing/details/cognitive-services/custom-vision-service/) de votre service.

## <a name="get-current-quota-usage-for-your-resource"></a>Obtenir le rapport d’utilisation des quotas actuel pour votre ressource

Utilisez la commande [az cognitiveservices account list-usage](/cli/azure/cognitiveservices/account?view=azure-cli-latest#az-cognitiveservices-account-list-usage) afin d’obtenir le rapport d’utilisation pour votre ressource Cognitive Service.

```azurecli-interactive
az cognitiveservices account list-usage \
    --name anomaly-detector-resource \
    --resource-group cognitive-services-resource-group \
    --subscription subscription-name
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez nettoyer et supprimer une ressource Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources supprime également toutes les autres ressources se trouvant dans le groupe.

Pour supprimer le groupe de ressources et les ressources associées, utilisez la commande « az group delete ».

```azurecli-interactive
az group delete --name cognitive-services-resource-group
```

## <a name="see-also"></a>Voir aussi

* [Authentifier des requêtes auprès d’Azure Cognitive Services](authentication.md)
* [Qu’est-ce qu’Azure Cognitive Services ?](./what-are-cognitive-services.md)
* [Prise en charge en langage naturel](language-support.md)
* [Prise en charge des conteneurs Docker](cognitive-services-container-support.md)