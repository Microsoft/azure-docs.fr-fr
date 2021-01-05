---
title: Créer un membre Azure Blockchain Service - Azure PowerShell
description: Créez un membre Azure Blockchain Service pour un consortium de blockchain à l’aide d’Azure PowerShell.
ms.date: 9/22/2020
ms.topic: quickstart
ms.reviewer: ravastra
ms.custom: references_regions, devx-track-azurepowershell
ms.openlocfilehash: b57c44e79d599ab41b2c3356ee337811acdf639d
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91948338"
---
# <a name="quickstart-create-an-azure-blockchain-service-blockchain-member-using-azure-powershell"></a>Démarrage rapide : Créer un membre blockchain dans Azure Blockchain Service à l’aide d’Azure PowerShell

Dans ce guide de démarrage rapide, vous allez déployer un nouveau membre blockchain et un nouveau consortium dans Azure Blockchain Service à l’aide d’Azure PowerShell.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

Si vous choisissez d’utiliser PowerShell localement, cet article vous demande d’installer le module Az PowerShell et de vous connecter à votre compte Azure avec l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Pour en savoir plus sur l’installation du module Az PowerShell, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps).

> [!IMPORTANT]
> Tant que le module PowerShell **Az.Blockchain** est en préversion, vous devez l’installer séparément du module PowerShell Az à l’aide de la commande suivante : `Install-Module`. Quand ce module PowerShell sera en disponibilité générale, il fera partie intégrante des versions futures du module PowerShell Az et sera disponible en mode natif dans Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.Blockchain
```

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

## <a name="register-resource-provider"></a>S’inscrire auprès du fournisseur de ressources

Si c’est la première fois que vous utilisez le service Azure Blockchain, vous devez inscrire le fournisseur de ressources **Microsoft.Blockchain**.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.Blockchain
```

## <a name="choose-a-specific-azure-subscription"></a>Choisir un abonnement Azure spécifique

Si vous avez plusieurs abonnements Azure, sélectionnez l’abonnement approprié dans lequel les ressources doivent être facturées. Sélectionnez un abonnement spécifique avec l’applet de commande [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

```azurepowershell-interactive
Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
```

## <a name="define-variables"></a>Définir des variables

Vous utiliserez plusieurs types d’informations de façon répétée. Créez des variables pour stocker les informations.

```azurepowershell-interactive
# Name of resource group used throughout this article
$resourceGroupName = 'myResourceGroup'

# Azure region
$location = 'eastus'
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un [groupe de ressources Azure](../../azure-resource-manager/management/overview.md) avec l’applet de commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées en tant que groupe.

L’exemple suivant crée un groupe de ressources en fonction du nom de la variable `$resourceGroupName` dans la région spécifiée de la variable `$location`.

```azurepowershell-interactive
New-AzResourceGroup -Name $resourceGroupName -Location $location
```

## <a name="create-a-blockchain-member"></a>Créer un membre blockchain

Un membre Azure Blockchain Service est un nœud blockchain dans un réseau blockchain de consortium privé.
Lors du provisionnement d’un membre, vous pouvez créer ou rejoindre un réseau de consortium. Vous avez besoin d’au moins un membre pour former un réseau de consortium. Le nombre de membres blockchain nécessaires par participant dépend de votre scénario. Les participants au consortium peuvent avoir un ou plusieurs membres blockchain ou ils peuvent partager des membres avec d’autres participants. Pour plus d’informations sur les consortiums, consultez [Consortium Azure Blockchain Service](consortium.md).

Vous devez passer plusieurs paramètres et propriétés. Remplacez les exemples de paramètres par vos propres valeurs.

```azurepowershell-interactive
$passwd = Read-Host -Prompt 'Enter the members default transaction node password' -AsSecureString
$csPasswd = Read-Host -Prompt 'Enter  the consortium account password' -AsSecureString

$memberParams = @{
  Name = 'myblockchainmember'
  ResourceGroupName = $resourceGroupName
  Consortium = 'myconsortium'
  ConsortiumManagementAccountPassword = $csPasswd
  Location = $location
  Password = $passwd
  Protocol = 'Quorum'
  Sku = 'S0'
}
New-AzBlockchainMember @memberParams
```

| Paramètre | Description |
|---------|-------------|
| **ResourceGroupName** | Nom du groupe de ressources dans lequel les ressources Azure Blockchain Service sont créées. Utilisez le groupe de ressources que vous avez créé dans la section précédente.
| **Nom** | Nom unique qui identifie le membre blockchain Azure Blockchain Service. Le nom est utilisé pour l’adresse de point de terminaison public. Par exemple : `myblockchainmember.blockchain.azure.com`.
| **Lieu** | Région Azure dans laquelle le membre blockchain est créé. Par exemple : `westus2`. Choisissez l’emplacement le plus proche de vos utilisateurs ou de vos autres applications Azure. Les fonctionnalités peuvent ne pas être disponibles dans certaines régions. Azure Blockchain Data Manager est disponible dans les régions Azure suivantes : USA Est et Europe Ouest.
| **Mot de passe** | Le mot de passe du nœud de transaction par défaut du membre. Utilisez le mot de passe pour l’authentification de base lorsque vous vous connectez au point de terminaison public du nœud de transaction par défaut du membre blockchain.
| **Protocole** | Protocole Blockchain. Actuellement, le protocole _Quorum_ est pris en charge.
| **Consortium** | Nom du consortium à rejoindre ou créer. Pour plus d’informations sur les consortiums, consultez [Consortium Azure Blockchain Service](consortium.md).
| **ConsortiumManagementAccountPassword** | Le mot de passe du compte consortium est également appelé mot de passe du compte du membre. Le mot de passe du compte du membre sert à chiffrer la clé privée du compte Ethereum créé pour votre membre. Vous utilisez le compte du membre et le mot de passe du compte du membre pour la gestion du consortium.
| **Sku** | Niveau de service. **S0** pour Standard ou **B0** pour De base. Utilisez le niveau _De base_ pour le développement, les tests et les preuves de concept. Utilisez le niveau _Standard_ pour les déploiements en production. Vous devez également utiliser le niveau _Standard_ si vous utilisez Blockchain Data Manager, ou si vous envoyez un volume élevé de transactions privées. Après la création du membre, le changement du niveau tarifaire entre le niveau de base et le niveau standard n’est pas prise en charge.

La création du membre blockchain et des ressources de prise en charge prend environ 10 minutes.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez utiliser le membre blockchain que vous avez créé pour le guide de démarrage rapide ou tutoriel suivant. Lorsque vous n’en aurez plus besoin, vous pourrez supprimer les ressources en supprimant le groupe de ressources `myResourceGroup` que vous avez créé dans le guide de démarrage rapide.

> [!CAUTION]
> L’exemple suivant supprime le groupe de ressources spécifié et toutes les ressources qu’il contient.
> Si des ressources en dehors du cadre de cet article existent dans le groupe de ressources spécifié, elles seront également supprimées.

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé un membre Azure Blockchain Service et un nouveau consortium. Dans le guide de démarrage rapide qui suit, vous allez utiliser Azure Blockchain Development Kit pour Ethereum afin d’effectuer une jonction à un membre Azure Blockchain Service.

> [!div class="nextstepaction"]
> [Utiliser Visual Studio Code pour se connecter à Azure Blockchain Service](connect-vscode.md)