---
title: 'Démarrage rapide : Déployer l’API Azure pour FHIR à l’aide d’Azure CLI'
description: Dans ce guide de démarrage rapide, vous allez apprendre à déployer l’API Azure pour FHIR sur Azure à l’aide d’Azure CLI.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 10/15/2019
ms.author: cavoeg
ms.custom: devx-track-azurecli
ms.openlocfilehash: 3340cd0a39b0f5311487fec5b05d37e3376d433a
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659249"
---
# <a name="quickstart-deploy-azure-api-for-fhir-using-azure-cli"></a>Démarrage rapide : Déployer l’API Azure pour FHIR à l’aide d’Azure CLI

Dans ce guide de démarrage rapide, vous allez apprendre à déployer l’API Azure pour FHIR sur Azure à l’aide d’Azure CLI.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="add-healthcareapis-extension"></a>Ajouter l’extension HealthcareAPIs

```azurecli-interactive
az extension add --name healthcareapis
```

Obtenez la liste des commandes pour HealthcareAPIs :

```azurecli-interactive
az healthcareapis --help
```

## <a name="create-azure-resource-group"></a>Créer un groupe de ressources Azure

Choisissez le nom du groupe de ressources qui doit contenir l’API Azure pour FHIR, puis créez ce groupe :

```azurecli-interactive
az group create --name "myResourceGroup" --location westus2
```

## <a name="deploy-the-azure-api-for-fhir"></a>Déployer l’API Azure pour FHIR

```azurecli-interactive
az healthcareapis create --resource-group myResourceGroup --name nameoffhiraccount --kind fhir-r4 --location westus2 
```

## <a name="fetch-fhir-api-capability-statement"></a>Récupérer une déclaration de capacité relative à l’API FHIR

Obtenez une déclaration de capacité de l’API FHIR à l’aide de :

```azurecli-interactive
curl --url "https://nameoffhiraccount.azurehealthcareapis.com/metadata"
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, supprimez le groupe de ressources en effectuant les étapes suivantes :

```azurecli-interactive
az group delete --name "myResourceGroup"
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé l’API Azure pour FHIR sur votre abonnement. Pour définir des paramètres supplémentaires dans votre API Azure pour FHIR, consultez le guide pratique relatif aux paramètres supplémentaires. Si vous êtes prêt à commencer à utiliser l’API Azure pour FHIR, apprenez-en davantage sur la façon d’inscrire des applications.

>[!div class="nextstepaction"]
>[Paramètres supplémentaires dans l’API Azure pour FHIR](azure-api-for-fhir-additional-settings.md)

>[!div class="nextstepaction"]
>[Présentation de l’inscription d’applications](fhir-app-registration.md)
