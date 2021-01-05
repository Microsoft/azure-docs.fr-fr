---
title: Déployer des ressources avec PowerShell et un modèle
description: Utilisez Azure Resource Manager et Azure PowerShell pour déployer des ressources sur Azure. Les ressources sont définies dans un modèle Resource Manager.
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: 5266aa51422dce6dfa4b82238e905f4f630ccf48
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92668557"
---
# <a name="deploy-resources-with-arm-templates-and-azure-powershell"></a>Déployer des ressources à l’aide de modèles Resource Manager et d’Azure PowerShell

Cet article explique comment utiliser Azure PowerShell avec les modèles Azure Resource Manager (ARM) pour déployer vos ressources dans Azure. Si vous n’avez pas une bonne connaissance des concepts de déploiement et de gestion des solutions Azure, consultez [Vue d’ensemble du déploiement de modèles](overview.md).

## <a name="prerequisites"></a>Prérequis

Il vous faut un modèle à déployer. Si vous n’en avez pas déjà un, téléchargez et enregistrez un [exemple de modèle](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json) à partir du référentiel de modèles de démarrage rapide Azure. Le nom du fichier local utilisé dans cet article est **c:\MyTemplates\azuredeploy.json** .

Vous devez installer Azure PowerShell et vous connecter à Azure :

- **Installez les cmdlets Azure PowerShell sur votre ordinateur local.** Pour plus d’informations, consultez [Bien démarrer avec Azure PowerShell](/powershell/azure/get-started-azureps).
- **Connectez-vous à Azure à l'aide de [Connect-AZAccount](/powershell/module/az.accounts/connect-azaccount)** . Si vous disposez de plusieurs abonnements Azure, vous devrez peut-être également exécuter [Set-AzContext](/powershell/module/Az.Accounts/Set-AzContext). Pour plus d'informations, consultez [Utiliser plusieurs abonnements Azure](/powershell/azure/manage-subscriptions-azureps).

Si vous n’avez pas installé PowerShell, vous pouvez utiliser le Cloud Shell. Pour plus d’informations, consultez [Déployer des modèles Azure Resource Manager à partir de Cloud Shell](deploy-cloud-shell.md).

## <a name="deployment-scope"></a>Étendue du déploiement

Vous pouvez cibler votre déploiement au niveau d’un groupe de ressources, d’un abonnement, d’un groupe d’administration ou d’un locataire. Les commandes à utiliser diffèrent en fonction de l’étendue du déploiement.

* Pour un déploiement dans un **groupe de ressources** , utilisez la commande [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) :

  ```azurepowershell
  New-AzResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateFile <path-to-template>
  ```

* Pour opérer un déploiement vers un **abonnement** , utilisez New-AzSubscriptionDeployment :

  ```azurepowershell
  New-AzSubscriptionDeployment -Location <location> -TemplateFile <path-to-template>
  ```

  Pour plus d’informations sur les déploiements au niveau de l’abonnement, consultez [Créer des groupes de ressources et des ressources au niveau de l’abonnement](deploy-to-subscription.md).

* Pour opérer un déploiement vers un **groupe d’administration** , utilisez [New-AzManagementGroupDeployment](/powershell/module/az.resources/New-AzManagementGroupDeployment).

  ```azurepowershell
  New-AzManagementGroupDeployment -Location <location> -TemplateFile <path-to-template>
  ```

  Pour plus d’informations sur les déploiements au niveau du groupe d’administration, consultez [Créer des ressources au niveau du groupe d’administration](deploy-to-management-group.md).

* Pour opérer un déploiement vers un **locataire** , utilisez [New-AzTenantDeployment](/powershell/module/az.resources/new-aztenantdeployment).

  ```azurepowershell
  New-AzTenantDeployment -Location <location> -TemplateFile <path-to-template>
  ```

  Pour plus d’informations sur les déploiements au niveau d’un locataire, consultez [Créer des ressources au niveau du locataire](deploy-to-tenant.md).

Pour chaque étendue, l’utilisateur qui déploie le modèle doit disposer des autorisations nécessaires pour créer des ressources.

## <a name="deploy-local-template"></a>Déployer un modèle local

Vous pouvez déployer un modèle à partir de votre ordinateur local ou d’un modèle stocké en externe. Cette section décrit le déploiement d’un modèle local.

Si vous effectuez un déploiement vers un groupe de ressources qui n’existe pas, vous devez commencer par créer ce dernier. Le nom du groupe de ressources ne peut contenir que des caractères alphanumériques, des points, des traits de soulignement, des traits d'union et des parenthèses. Il peut comprendre jusqu’à 90 caractères. Le nom ne peut pas se terminer par un point.

```azurepowershell
New-AzResourceGroup -Name ExampleGroup -Location "Central US"
```

Pour déployer un modèle local, utilisez le paramètre `-TemplateFile` dans la commande de déploiement. L’exemple suivant montre également comment définir une valeur de paramètre provenant du modèle.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name ExampleDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateFile c:\MyTemplates\azuredeploy.json
```

Le déploiement peut prendre plusieurs minutes.

## <a name="deploy-remote-template"></a>Déployer un modèle distant

Au lieu de stocker les modèles Resource Manager sur votre ordinateur local, vous pouvez les stocker dans un emplacement externe. Vous pouvez stocker des modèles dans un dépôt de contrôle de code source (par exemple, GitHub). Vous pouvez aussi les stocker dans un compte de stockage Azure pour mettre en place un accès partagé dans votre organisation.

Si vous effectuez un déploiement vers un groupe de ressources qui n’existe pas, vous devez commencer par créer ce dernier. Le nom du groupe de ressources ne peut contenir que des caractères alphanumériques, des points, des traits de soulignement, des traits d'union et des parenthèses. Il peut comprendre jusqu’à 90 caractères. Le nom ne peut pas se terminer par un point.

```azurepowershell
New-AzResourceGroup -Name ExampleGroup -Location "Central US"
```

Pour déployer un modèle externe, utilisez le paramètre `-TemplateUri`.

```azurepowershell
New-AzResourceGroupDeployment `
  -Name ExampleDeployment `
  -ResourceGroupName ExampleGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
```

L’exemple précédent nécessite un URI accessible publiquement pour le modèle, ce qui convient pour la plupart des scénarios, sachant que votre modèle ne doit pas inclure de données sensibles. Si vous avez besoin de spécifier des données sensibles (par exemple, un mot de passe d’administrateur), passez cette valeur en tant que paramètre sécurisé. Toutefois, si vous souhaitez gérer l’accès au modèle, envisagez d’utiliser des [spécifications de modèle](#deploy-template-spec).

## <a name="deployment-name"></a>Nom du déploiement

Lors du déploiement d’un modèle ARM, vous pouvez attribuer un nom au déploiement. Ce nom peut vous aider à récupérer le déploiement à partir de l’historique de déploiement. Si vous n’attribuez pas de nom au déploiement, le nom du fichier de modèle est utilisé. Par exemple, si vous déployez un modèle nommé `azuredeploy.json` et que vous ne spécifiez pas de nom de déploiement, le déploiement est nommé `azuredeploy`.

Chaque fois que vous exécutez un déploiement, une entrée est ajoutée à l’historique de déploiement du groupe de ressources avec le nom du déploiement. Si vous exécutez un autre déploiement et que vous lui attribuez le même nom, l’entrée précédente est remplacée par le déploiement actuel. Si vous souhaitez conserver des entrées uniques dans l’historique de déploiement, attribuez un nom unique à chaque déploiement.

Pour créer un nom unique, vous pouvez assigner un numéro aléatoire.

```azurepowershell-interactive
$suffix = Get-Random -Maximum 1000
$deploymentName = "ExampleDeployment" + $suffix
```

Vous pouvez aussi ajouter une valeur de date.

```azurepowershell-interactive
$today=Get-Date -Format "MM-dd-yyyy"
$deploymentName="ExampleDeployment"+"$today"
```

Si vous exécutez des déploiements simultanés dans le même groupe de ressources avec le même nom de déploiement, seul le dernier déploiement aboutit. Les déploiements de même nom qui n’arrivent pas à terme sont remplacés par le dernier déploiement. Par exemple, si vous exécutez un déploiement nommé `newStorage` qui déploie un compte de stockage nommé `storage1` et que, dans le même temps, vous exécutez un autre déploiement nommé `newStorage` qui déploie un compte de stockage nommé `storage2`, vous ne déployez qu’un seul compte de stockage. Le compte de stockage qui en résulte est nommé `storage2`.

En revanche, si vous exécutez un déploiement nommé `newStorage` qui déploie un compte de stockage nommé `storage1` et que, aussitôt terminé, vous exécutez un autre déploiement nommé `newStorage` qui déploie un compte de stockage nommé `storage2`, vous disposez de deux comptes de stockage : un nommé `storage1` et l’autre nommé `storage2`. Cependant, l’historique de déploiement ne présente qu’une seule entrée.

Quand vous spécifiez un nom unique pour chaque déploiement, vous pouvez les exécuter simultanément sans conflit. Si vous exécutez un déploiement nommé `newStorage1` qui déploie un compte de stockage nommé `storage1` et que, dans le même temps, vous exécutez un autre déploiement nommé `newStorage2` qui déploie un compte de stockage nommé `storage2`, vous disposez de deux comptes de stockage et l’historique de déploiement présente deux entrées.

Pour éviter les conflits lors de déploiements simultanés et faire en sorte que l’historique de déploiement présente des entrées uniques, attribuez un nom unique à chaque déploiement.

## <a name="deploy-template-spec"></a>Déployer une spec de modèle

Au lieu de déployer un modèle local ou distant, vous pouvez créer une [spécification de modèle](template-specs.md). La spécification de modèle est une ressource de votre abonnement Azure qui contient un modèle ARM. Elle facilite le partage sécurisé du modèle avec les utilisateurs de votre organisation. Vous utilisez le contrôle d’accès Azure en fonction du rôle (Azure RBAC) pour accorder l’accès à la spécification de modèle. Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire.

Les exemples suivants montrent comment créer et déployer une spécification de modèle. Ces commandes sont disponibles seulement si vous vous êtes [inscrit pour la préversion](https://aka.ms/templateSpecOnboarding).

Tout d’abord, créez la spécification de modèle en fournissant le modèle ARM.

```azurepowershell
New-AzTemplateSpec `
  -Name storageSpec `
  -Version 1.0 `
  -ResourceGroupName templateSpecsRg `
  -Location westus2 `
  -TemplateJsonFile ./mainTemplate.json
```

Ensuite, recevez l’ID de la spécification de modèle et déployez-le.

```azurepowershell
$id = (Get-AzTemplateSpec -Name storageSpec -ResourceGroupName templateSpecsRg -Version 1.0).Version.Id

New-AzResourceGroupDeployment `
  -ResourceGroupName demoRG `
  -TemplateSpecId $id
```

Pour plus d’informations, consultez [Spécification de modèle Azure Resource Manager (préversion)](template-specs.md).

## <a name="preview-changes"></a>Prévisualiser les modifications

Avant de déployer votre modèle, vous pouvez afficher un aperçu des modifications que le modèle apportera à votre environnement. Utilisez l’[opération de simulation](template-deploy-what-if.md) pour vérifier que le modèle apporte les changements prévus. Cette opération vérifie aussi que le modèle est exempt d’erreurs.

## <a name="pass-parameter-values"></a>Passer les valeurs de paramètre

Pour passer les valeurs de paramètre, vous pouvez utiliser des paramètres inline ou un fichier de paramètres.

### <a name="inline-parameters"></a>Paramètres inline

Pour passer les paramètres inline, indiquez les noms des paramètres au moyen de la commande `New-AzResourceGroupDeployment`. Par exemple, pour passer une chaîne et un tableau à un modèle, utilisez :

```powershell
$arrayParam = "value1", "value2"
New-AzResourceGroupDeployment -ResourceGroupName testgroup `
  -TemplateFile c:\MyTemplates\demotemplate.json `
  -exampleString "inline string" `
  -exampleArray $arrayParam
```

Vous pouvez également récupérer le contenu d’un fichier et fournir ce contenu en tant que paramètre inline.

```powershell
$arrayParam = "value1", "value2"
New-AzResourceGroupDeployment -ResourceGroupName testgroup `
  -TemplateFile c:\MyTemplates\demotemplate.json `
  -exampleString $(Get-Content -Path c:\MyTemplates\stringcontent.txt -Raw) `
  -exampleArray $arrayParam
```

Obtenir une valeur de paramètre à partir d’un fichier est utile lorsque vous devez fournir des valeurs de configuration. Par exemple, vous pouvez fournir des [valeurs cloud-init pour une machine virtuelle Linux](../../virtual-machines/linux/using-cloud-init.md).

Si vous avez besoin de transmettre un tableau d’objets, créez des tables de hachage dans PowerShell et ajoutez-les à un tableau. Transmettez ce tableau en tant que paramètre lors du déploiement.

```powershell
$hash1 = @{ Name = "firstSubnet"; AddressPrefix = "10.0.0.0/24"}
$hash2 = @{ Name = "secondSubnet"; AddressPrefix = "10.0.1.0/24"}
$subnetArray = $hash1, $hash2
New-AzResourceGroupDeployment -ResourceGroupName testgroup `
  -TemplateFile c:\MyTemplates\demotemplate.json `
  -exampleArray $subnetArray
```

### <a name="parameter-files"></a>Fichiers de paramètres

Au lieu de passer des paramètres en tant que valeurs inline dans votre script, il peut s’avérer plus facile d’utiliser un fichier JSON qui contient les valeurs des paramètres. Le fichier de paramètres peut être un fichier local ou un fichier externe avec un URI accessible.

Pour plus d’informations sur le fichier de paramètres, consultez [Créer un fichier de paramètres Resource Manager](parameter-files.md).

Pour transmettre un fichier de paramètres local, utilisez le paramètre **TemplateParameterFile**  :

```powershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateFile c:\MyTemplates\azuredeploy.json `
  -TemplateParameterFile c:\MyTemplates\storage.parameters.json
```

Pour transmettre un fichier de paramètres externe, utilisez le paramètre **TemplateParameterUri**  :

```powershell
New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleResourceGroup `
  -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json `
  -TemplateParameterUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.parameters.json
```

## <a name="next-steps"></a>Étapes suivantes

- Pour restaurer un déploiement réussi lorsque vous obtenez une erreur, consultez [Restaurer en cas d’erreur vers un déploiement réussi](rollback-on-error.md).
- Pour spécifier comment gérer les ressources présentes dans le groupe de ressources, mais non définies dans le modèle, consultez [Modes de déploiement Azure Resource Manager](deployment-modes.md).
- Pour comprendre comment définir des paramètres dans votre modèle, consultez [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](template-syntax.md).
- Pour plus d’informations sur le déploiement d’un modèle qui nécessite un jeton SAP, consultez [Déploiement d’un modèle privé avec un jeton SAP](secure-template-with-sas-token.md).
