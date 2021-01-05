---
title: Gérer les membres d’un consortium Azure Blockchain Service – PowerShell
description: Découvrez comment gérer les membres d’un consortium Azure Blockchain Service à l’aide d’Azure PowerShell.
ms.date: 10/14/2019
ms.topic: how-to
ms.reviewer: zeyadr
ms.openlocfilehash: d40e55f177bda9edb40383b6e2c61c32633cd005
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85211338"
---
# <a name="manage-consortium-members-in-azure-blockchain-service-using-powershell"></a>Gérer les membres d’un consortium dans Azure Blockchain Service à l’aide de PowerShell

Vous pouvez utiliser PowerShell pour gérer les membres d’un consortium blockchain pour votre service Azure Blockchain Service. Les membres qui disposent des privilèges d’administrateur peuvent inviter, ajouter, supprimer et modifier des rôles pour tous les participants dans le consortium blockchain. Les membres qui disposent des privilèges d’utilisateur peuvent visualiser tous les participants du consortium blockchain et modifier leur nom complet de membre.

## <a name="prerequisites"></a>Prérequis

* Créez un membre de blockchain à l’aide du [portail Azure](create-member.md).
* Pour plus d’informations sur les nœuds, les membres et les consortiums, consultez [Consortium Azure Blockchain Service](consortium.md).

## <a name="open-azure-cloud-shell"></a>Ouvrir Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte.

Vous pouvez également ouvrir Cloud Shell dans un onglet distinct du navigateur en accédant à [shell.azure.com/powershell](https://shell.azure.com/powershell). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell et appuyez sur **Entrée** pour les exécuter.

## <a name="install-the-powershell-module"></a>Installer le module PowerShell

Installez le package Microsoft.AzureBlockchainService.ConsortiumManagement.PS à partir de PowerShell Gallery.

```powershell-interactive
Install-Module -Name Microsoft.AzureBlockchainService.ConsortiumManagement.PS -Scope CurrentUser
Import-Module Microsoft.AzureBlockchainService.ConsortiumManagement.PS
```

## <a name="set-the-information-preference"></a>Définir la préférence relative aux informations

Vous pouvez obtenir plus d’informations lors de l’exécution d’applets de commande en définissant la variable de préférence relative aux informations. Par défaut, *$InformationPreference* a la valeur *SilentlyContinue*.

Pour obtenir des informations plus détaillées des applets de commande, définissez la préférence dans PowerShell comme suit :

```powershell-interactive
$InformationPreference = 'Continue'
```

## <a name="establish-a-web3-connection"></a>Établir une connexion Web3

Pour gérer les membres du consortium, établissez une connexion Web3 au point de terminaison de votre membre Blockchain Service. Vous pouvez utiliser ce script pour définir des variables globales pour appeler les applets de commande de gestion du consortium.

```powershell-interactive
$Connection = New-Web3Connection -RemoteRPCEndpoint '<Endpoint address>'
$MemberAccount = Import-Web3Account -ManagedAccountAddress '<Member account address>' -ManagedAccountPassword '<Member account password>'
$ContractConnection = Import-ConsortiumManagementContracts -RootContractAddress '<RootContract address>' -Web3Client $Connection
```

Remplacez *\<Member account password\>* par le mot de passe du compte de membre que vous avez utilisé lorsque vous avez créé le membre.

Recherchez les autres valeurs dans le portail Azure :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez à la page **Vue d’ensemble** par défaut de votre membre Blockchain Service.

    ![Vue d’ensemble du membre](./media/manage-consortium-powershell/member-overview.png)

    Remplacez *\<Member account\>* et *\<RootContract address\>* par les valeurs du portail.

1. Pour l’adresse du point de terminaison, sélectionnez **Nœuds de transaction**, puis sélectionnez le **nœud de transaction par défaut**. Le nœud par défaut a le même nom que le membre blockchain.
1. Sélectionnez **Chaînes de connexion**.

    ![Chaînes de connexion](./media/manage-consortium-powershell/connection-strings.png)

    Remplacez *\<Endpoint address\>* par la valeur de **HTTPS (clé d’accès 1)** ou de **HTTPS (clé d’accès 2)** .

## <a name="manage-the-network-and-smart-contracts"></a>Gérer le réseau et les contrats intelligents

Utilisez les applets de commande de réseau et de contrats intelligents pour établir une connexion aux contrats intelligents du point de terminaison blockchain responsables de la gestion du consortium.

### <a name="import-consortiummanagementcontracts"></a>Import-ConsortiumManagementContracts

Utilisez cette applet de commande pour vous connecter aux contrats intelligents de la gestion du consortium. Ces contrats sont utilisés pour gérer et appliquer des membres au sein du consortium.

`Import-ConsortiumManagementContracts -RootContractAddress <String> -Web3Client <IClient>`

| Paramètre | Description | Obligatoire |
|-----------|-------------|:--------:|
| RootContractAddress | Adresse du contrat racine des contrats intelligents de gestion du consortium | Oui |
| Web3Client | Objet Web3Client obtenu à partir de New-Web3Connection | Oui |

#### <a name="example"></a>Exemple

```powershell-interactive
Import-ConsortiumManagementContracts -RootContractAddress '<RootContract address>'  -Web3Client $Connection
```

### <a name="import-web3account"></a>Import-Web3Account

Utilisez cette applet de commande pour créer un objet destiné à contenir les informations du compte de gestion d’un nœud distant.

`Import-Web3Account -ManagedAccountAddress <String> -ManagedAccountPassword <String>`

| Paramètre | Description | Obligatoire |
|-----------|-------------|:--------:|
| ManagedAccountAddress | Adresse de compte de membre blockchain | Oui |
| ManagedAccountPassword | Mot de passe de l’adresse du compte | Oui |

#### <a name="example"></a>Exemple

```powershell-interactive
Import-Web3Account -ManagedAccountAddress '<Member account address>'  -ManagedAccountPassword '<Member account password>'
```

### <a name="new-web3connection"></a>New-Web3Connection

Utilisez cette applet de commande pour établir une connexion au point de terminaison RPC d’un nœud de transaction.

`New-Web3Connection [-RemoteRPCEndpoint <String>]`

| Paramètre | Description | Obligatoire |
|-----------|-------------|:--------:|
| RemoteRPCEndpoint | Adresse du point de terminaison du membre blockchain | Oui |

#### <a name="example"></a>Exemple

```powershell-interactive
New-Web3Connection -RemoteRPCEndpoint '<Endpoint address>'
```

## <a name="manage-the-consortium-members"></a>Gérer les membres du consortium

Utilisez les applets de commande de gestion des membres de consortium pour gérer les membres au sein du consortium. Les actions disponibles varient selon le rôle de votre consortium.

### <a name="get-blockchainmember"></a>Get-BlockchainMember

Utilisez cette applet de commande pour obtenir des détails sur les membres ou pour lister les membres du consortium.

`Get-BlockchainMember [[-Name] <String>] -Members <IContract> -Web3Client <IClient>`

| Paramètre | Description | Obligatoire |
|-----------|-------------|:--------:|
| Nom | Nom du membre Blockchain Service sur lequel vous souhaitez récupérer des détails. Lorsqu’un nom est entré, il retourne les détails du membre. Lorsqu’un nom est omis, il retourne la liste de tous les membres du consortium. | Non |
| Membres | Objet de membres obtenu à partir de Import-ConsortiumManagementContracts | Oui |
| Web3Client | Objet Web3Client obtenu à partir de New-Web3Connection | Oui |

#### <a name="example"></a>Exemple

[Établissez une connexion Web3](#establish-a-web3-connection) pour définir la variable $ContractConnection.

```powershell-interactive
$ContractConnection | Get-BlockchainMember -Name <Member Name>
```

#### <a name="example-output"></a>Exemple de sortie

```
Name           : myblockchainmember
CorrelationId  : 0
DisplayName    : myCompany
SubscriptionId : <Azure subscription ID>
AccountAddress : 0x85b911c9e103d6405573151258d668479e9ebeef
Role           : ADMIN
```

### <a name="remove-blockchainmember"></a>Remove-BlockchainMember

Utilisez cette applet de commande pour supprimer un membre blockchain.

`Remove-BlockchainMember -Name <String> -Members <IContract> -Web3Account <IAccount> -Web3Client <IClient>`

| Paramètre | Description | Obligatoire |
|-----------|-------------|:--------:|
| Nom | Nom de membre à supprimer | Oui |
| Membres | Objet de membres obtenu à partir de Import-ConsortiumManagementContracts | Oui |
| Web3Account | Objet Web3Account obtenu à partir de Import-Web3Account | Oui |
| Web3Client | Objet Web3Client obtenu à partir de New-Web3Connection | Oui |

#### <a name="example"></a>Exemple

[Établissez une connexion Web3](#establish-a-web3-connection) pour définir les variables $ContractConnection et $MemberAccount.

```powershell-interactive
$ContractConnection | Remove-BlockchainMember -Name <Member Name> -Web3Account $MemberAccount
```

### <a name="set-blockchainmember"></a>Set-BlockchainMember

Utilisez cette applet de commande pour définir des attributs de membre blockchain, y compris le nom complet et le rôle du consortium.

Les administrateurs du consortium peuvent définir **DisplayName** et **Role** pour tous les membres. Un membre du consortium avec le rôle d’utilisateur peut modifier uniquement le nom complet de son propre membre.

```
Set-BlockchainMember -Name <String> [-DisplayName <String>] [-AccountAddress <String>] [-Role <String>]
 -Members <IContract> -Web3Account <IAccount> -Web3Client <IClient>
```

| Paramètre | Description | Obligatoire |
|-----------|-------------|:--------:|
| Nom | Nom du membre blockchain | Oui |
| DisplayName | Nouveau nom complet | Non |
| AccountAddress | Adresse du compte | Non |
| Membres | Objet de membres obtenu à partir de Import-ConsortiumManagementContracts | Oui |
| Web3Account | Objet Web3Account obtenu à partir de Import-Web3Account | Oui |
| Web3Client |  Objet Web3Client obtenu à partir de New-Web3Connection| Oui |

#### <a name="example"></a>Exemple

[Établissez une connexion Web3](#establish-a-web3-connection) pour définir les variables $ContractConnection et $MemberAccount.

```powershell-interactive
$ContractConnection | Set-BlockchainMember -Name <Member Name> -DisplayName <Display name> -Web3Account $MemberAccount
```

## <a name="manage-the-consortium-members-invitations"></a>Gérer les invitations des membres du consortium

Utilisez les applets de commande de gestion des invitations des membres du consortium pour gérer les invitations des membres du consortium. Les actions disponibles varient selon le rôle de votre consortium.

### <a name="new-blockchainmemberinvitation"></a>New-BlockchainMemberInvitation

Utilisez cette applet de commande pour inviter de nouveaux membres dans le consortium.

```
New-BlockchainMemberInvitation -SubscriptionId <String> -Role <String> -Members <IContract>
 -Web3Account <IAccount> -Web3Client <IClient>
```

| Paramètre | Description | Obligatoire |
|-----------|-------------|:--------:|
| SubscriptionId | ID d’abonnement Azure du membre à inviter | Oui |
| Role | Rôle du consortium. Les valeurs peuvent être ADMIN ou USER. ADMIN correspond au rôle d’administrateur du consortium. USER correspond au rôle de membre du consortium. | Oui |
| Membres | Objet de membres obtenu à partir de Import-ConsortiumManagementContracts | Oui |
| Web3Account | Objet Web3Account obtenu à partir de Import-Web3Account | Oui |
| Web3Client | Objet Web3Client obtenu à partir de New-Web3Connection | Oui |

#### <a name="example"></a>Exemple

[Établissez une connexion Web3](#establish-a-web3-connection) pour définir les variables $ContractConnection et $MemberAccount.

```powershell-interactive
$ContractConnection | New-BlockchainMemberInvitation -SubscriptionId <Azure Subscription ID> -Role USER -Web3Account $MemberAccount
```

### <a name="get-blockchainmemberinvitation"></a>Get-BlockchainMemberInvitation

Utilisez cette applet de commande pour récupérer ou lister l’état de l’invitation d’un membre du consortium.

`Get-BlockchainMemberInvitation [[-SubscriptionId] <String>] -Members <IContract> -Web3Client <IClient>`

| Paramètre | Description | Obligatoire |
|-----------|-------------|:--------:|
| SubscriptionId | ID d’abonnement Azure du membre à inviter. Si l’ID d’abonnement est fourni, il retourne les détails d’invitation de l’ID d’abonnement. Si l’ID d’abonnement est omis, il retourne la liste de toutes les invitations de membre. | Non |
| Membres | Objet de membres obtenu à partir de Import-ConsortiumManagementContracts | Oui |
| Web3Client | Objet Web3Client obtenu à partir de New-Web3Connection | Oui |

#### <a name="example"></a>Exemple

[Établissez une connexion Web3](#establish-a-web3-connection) pour définir la variable $ContractConnection.

```powershell-interactive
$ContractConnection | Get-BlockchainMemberInvitation –SubscriptionId <Azure subscription ID>
```

#### <a name="example-output"></a>Exemple de sortie

```
SubscriptionId                       Role CorrelationId
--------------                       ---- -------------
<Azure subscription ID>              USER             2
```

### <a name="remove-blockchainmemberinvitation"></a>Remove-BlockchainMemberInvitation

Utilisez cette applet de commande pour révoquer l’invitation d’un membre du consortium.

```
Remove-BlockchainMemberInvitation -SubscriptionId <String> -Members <IContract> -Web3Account <IAccount>
 -Web3Client <IClient>
```

| Paramètre | Description | Obligatoire |
|-----------|-------------|:--------:|
| SubscriptionId | ID d’abonnement Azure du membre à révoquer | Oui |
| Membres | Objet de membres obtenu à partir de Import-ConsortiumManagementContracts | Oui |
| Web3Account | Objet Web3Account obtenu à partir de Import-Web3Account | Oui |
| Web3Client | Objet Web3Client obtenu à partir de New-Web3Connection | Oui |

#### <a name="example"></a>Exemple

[Établissez une connexion Web3](#establish-a-web3-connection) pour définir les variables $ContractConnection et $MemberAccount.

```powershell-interactive
$ContractConnection | Remove-BlockchainMemberInvitation -SubscriptionId <Subscription ID> -Web3Account $MemberAccount
```

### <a name="set-blockchainmemberinvitation"></a>Set-BlockchainMemberInvitation

Utilisez cette applet de commande pour définir le **rôle** pour une invitation existante. Seuls les administrateurs du consortium peuvent modifier les invitations.

```
Set-BlockchainMemberInvitation -SubscriptionId <String> -Role <String> -Members <IContract>
 -Web3Account <IAccount> -Web3Client <IClient>
```

| Paramètre | Description | Obligatoire |
|-----------|-------------|:--------:|
| SubscriptionId | ID d’abonnement Azure du membre à inviter | Oui |
| Role | Nouveau rôle de consortium pour l’invitation. Les valeurs peuvent être **USER** ou **ADMIN**. | Oui |
| Membres |  Objet de membres obtenu à partir de Import-ConsortiumManagementContracts | Oui |
| Web3Account | Objet Web3Account obtenu à partir de Import-Web3Account | Oui |
| Web3Client | Objet Web3Client obtenu à partir de New-Web3Connection | Oui |

#### <a name="example"></a>Exemple

[Établissez une connexion Web3](#establish-a-web3-connection) pour définir les variables $ContractConnection et $MemberAccount.

```powershell-interactive
$ContractConnection | Set-BlockchainMemberInvitation -SubscriptionId <Azure subscription ID> -Role USER -Web3Account $MemberAccount
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les nœuds, les membres et les consortiums, consultez [Consortium Azure Blockchain Service](consortium.md)
