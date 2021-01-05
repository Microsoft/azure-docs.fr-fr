---
title: 'Démarrage rapide : Déployer l’API Azure pour FHIR à l’aide de PowerShell'
description: Dans ce guide de démarrage rapide, vous allez apprendre à déployer l’API Azure pour FHIR à l’aide de PowerShell.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 10/15/2019
ms.author: cavoeg
ms.openlocfilehash: 2cb50f2ae98dab1e64e01498e0913d1932e455cd
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92339405"
---
# <a name="quickstart-deploy-azure-api-for-fhir-using-powershell"></a>Démarrage rapide : Déployer l’API Azure pour FHIR à l’aide de PowerShell

Dans ce guide de démarrage rapide, vous allez apprendre à déployer l’API Azure pour FHIR à l’aide de PowerShell.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="register-the-azure-api-for-fhir-resource-provider"></a>Inscrire le fournisseur de ressources d’API Azure pour FHIR

Si le fournisseur de ressources `Microsoft.HealthcareApis` n’est pas déjà inscrit pour votre abonnement, vous pouvez l’inscrire avec :

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.HealthcareApis
```

## <a name="create-azure-resource-group"></a>Créer un groupe de ressources Azure

```azurepowershell-interactive
New-AzResourceGroup -Name "myResourceGroupName" -Location westus2
```

## <a name="deploy-azure-api-for-fhir"></a>Déployer l’API Azure pour FHIR

```azurepowershell-interactive
New-AzHealthcareApisService -Name nameoffhirservice -ResourceGroupName myResourceGroupName -Location westus2 -Kind fhir-R4
```

> [!NOTE]
> En fonction de la version du module PowerShell `Az` que vous avez installée, le serveur FHIR provisionné peut être configuré pour utiliser l’authentification [RBAC locale](configure-local-rbac.md). De plus, il est possible qu’il comporte l’utilisateur PowerShell connecté dans la liste des ID d’objet d’identité autorisés pour le service FHIR déployé. À l’avenir, nous vous recommandons d’[utiliser Azure RBAC](configure-azure-rbac.md) pour attribuer des rôles de plan de données. Vous devrez peut-être supprimer cet ID d’objet utilisateur après le déploiement pour activer le mode Azure RBAC.


## <a name="fetch-capability-statement"></a>Récupérer la déclaration de capacité

Vous pourrez vérifier que le compte d’API Azure pour FHIR est en cours d’exécution en récupérant une déclaration de capacité FHIR :

```azurepowershell-interactive
$metadata = Invoke-WebRequest -Uri "https://nameoffhirservice.azurehealthcareapis.com/metadata"
$metadata.RawContent
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, supprimez le groupe de ressources en effectuant les étapes suivantes :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé l’API Azure pour FHIR sur votre abonnement. Pour définir des paramètres supplémentaires dans votre API Azure pour FHIR, consultez le guide pratique relatif aux paramètres supplémentaires. Si vous êtes prêt à commencer à utiliser l’API Azure pour FHIR, apprenez-en davantage sur la façon d’inscrire des applications.

>[!div class="nextstepaction"]
>[Paramètres supplémentaires dans l’API Azure pour FHIR](azure-api-for-fhir-additional-settings.md)

>[!div class="nextstepaction"]
>[Présentation de l’inscription d’applications](fhir-app-registration.md)
