---
title: Configurer les pare-feux et réseaux virtuels d’Azure Key Vault – Azure Key Vault
description: Instructions détaillées pour configurer les pare-feux et réseaux virtuels d’Azure Key Vault
services: key-vault
author: ShaneBala-keyvault
manager: ravijan
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 10/01/2020
ms.author: sudbalas
ms.custom: devx-track-azurecli
ms.openlocfilehash: d1b1c27fe0136220d5a1851af4a5c24102a37da1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015551"
---
# <a name="configure-azure-key-vault-firewalls-and-virtual-networks"></a>Configurer les pare-feux et réseaux virtuels d’Azure Key Vault

Cet article vous donne des conseils sur la configuration du pare-feu Azure Key Vault. Ce document décrit en détail les différentes configurations possibles du pare-feu Key Vault. Il fournit également des instructions pas à pas sur la façon de configurer Azure Key Vault pour l’utiliser avec d’autres applications et services Azure.

## <a name="firewall-settings"></a>Paramètres du pare-feu

Cette section présente les différentes configurations possibles du pare-feu Azure Key Vault.

### <a name="key-vault-firewall-disabled-default"></a>Pare-feu Key Vault désactivé (configuration par défaut)

Par défaut, quand vous créez un coffre de clés, le pare-feu Azure Key Vault est désactivé. Toutes les applications et tous les services Azure peuvent accéder au coffre de clés et lui envoyer des requêtes. Notez que cette configuration n’implique pas que tous les utilisateurs pourront effectuer des opérations sur le coffre de clés. Le coffre de clés restreint toujours l’accès aux secrets, aux clés et aux certificats qu’il stocke en exigeant des autorisations de stratégie d’accès et une authentification Azure Active Directory. Pour mieux comprendre l’authentification du coffre de clés, consultez [ici](./authentication-fundamentals.md) l’article sur les concepts de base de l’authentification du coffre de clés.

### <a name="key-vault-firewall-enabled-trusted-services-only"></a>Pare-feu Key Vault activé (services approuvés uniquement)

Quand vous activez le pare-feu Key Vault, vous pouvez choisir d’autoriser les services Microsoft approuvés à contourner ce pare-feu. La liste des services approuvés n’inclut pas tous les services Azure. Par exemple, Azure DevOps n’y figure pas. **Cela ne signifie pas que les services qui ne sont pas dans la liste des services approuvés ne sont pas approuvés ou sécurisés.** La liste des services approuvés englobe les services dans lesquels tout le code exécuté est contrôlé par Microsoft. Du fait que les utilisateurs peuvent écrire du code personnalisé dans des services Azure comme Azure DevOps, Microsoft n’offre pas l’option de créer une approbation permanente pour le service. En outre, la présence d’un service dans la liste des services approuvés ne signifie pas pour autant que le service est autorisé dans tous les scénarios.

Pour déterminer si un service que vous voulez utiliser fait partie de la liste des services approuvés, consultez [ce document](./overview-vnet-service-endpoints.md#trusted-services).

### <a name="key-vault-firewall-enabled-ipv4-addresses-and-ranges---static-ips"></a>Pare-feu Key Vault activé (adresses et plages IPv4 - adresses IP statiques)

Si vous souhaitez autoriser un service particulier à accéder au coffre de clés par le biais du pare-feu Key Vault, vous pouvez ajouter son adresse IP à la liste autorisée du pare-feu Key Vault. Cette configuration est idéale pour les services qui utilisent des adresses IP statiques ou des plages connues.

Pour autoriser une adresse ou plage IP d’une ressource Azure telle qu’une application web ou une application logique, effectuez les étapes suivantes.

1. Se connecter au portail Azure.
1. Sélectionnez la ressource (instance spécifique du service).
1. Cliquez sur le panneau « Propriétés » sous « Paramètres ».
1. Recherchez le champ « Adresse IP ».
1. Copiez cette valeur ou plage et entrez-la dans la liste d’autorisation du pare-feu Key Vault.

Pour autoriser un service Azure entier, par le biais du pare-feu Key Vault, utilisez la liste des adresses IP du centre de données documentées publiquement pour Azure, accessible [ici](https://www.microsoft.com/download/details.aspx?id=41653). Recherchez les adresses IP associées au service dont vous avez besoin dans la région de votre choix et ajoutez ces adresses IP au pare-feu Key Vault à l’aide des étapes ci-dessus.

### <a name="key-vault-firewall-enabled-virtual-networks---dynamic-ips"></a>Pare-feu Key Vault activé (réseaux virtuels - adresses IP dynamiques)

Si vous essayez d’autoriser une ressource Azure telle qu’une machine virtuelle par le biais du coffre de clés, il est possible que vous ne puissiez pas utiliser des adresses IP statiques et que vous ne souhaitiez pas autoriser toutes les adresses IP de machines virtuelles Azure à accéder à votre coffre de clés.

Dans ce cas, vous devez créer la ressource dans un réseau virtuel, puis autoriser le trafic provenant du réseau virtuel et du sous-réseau spécifiques à accéder à votre coffre de clés. Pour ce faire, procédez comme suit.

1. Se connecter au portail Azure.
1. Sélectionnez le coffre de clés à configurer.
1. Sélectionnez le panneau « Réseau ».
1. Sélectionnez « + Ajouter un réseau virtuel existant ».
1. Sélectionnez le réseau virtuel et le sous-réseau que vous souhaitez autoriser via le pare-feu Key Vault.

### <a name="key-vault-firewall-enabled-private-link"></a>Pare-feu Key Vault activé (liaison privée)

Pour savoir comment configurer une connexion de liaison privée sur votre coffre de clés, consultez [ce document](./private-link-service.md).

> [!IMPORTANT]
> Une fois que les règles de pare-feu sont effectives, les utilisateurs peuvent effectuer des opérations du [plan de données](secure-your-key-vault.md#data-plane-access-control) de Key Vault seulement quand leurs demandes proviennent de réseaux virtuels ou de plages d’adresses IPv4 autorisés. Ceci s’applique également à l’accès au coffre de clés à partir du portail Azure. Si des utilisateurs peuvent accéder à un coffre de clés à partir du portail Azure, il ne peuvent pas lister les clés/secrets/certificats si leur ordinateur client ne figure pas dans la liste autorisée. Ceci affecte également l’accès au sélecteur de compte Key Vault par d’autres services Azure. Des utilisateurs qui peuvent voir la liste des coffres de clés ne peuvent pas lister les clés si des règles de pare-feu bloquent leur ordinateur client.

> [!NOTE]
> Notez les limitations de configuration suivantes :
> * Un maximum de 127 règles de réseau virtuel et 127 règles IPv4 sont autorisées. 
> * Les règles de réseau IP sont autorisées uniquement pour les adresses IP publiques. Les plages d’adresses IP réservées aux réseaux privés (comme défini dans RFC 1918) ne sont pas autorisées dans les règles IP. Les réseaux privés incluent des adresses qui commencent par **10.** , **172.16-31.** et **192.168.** . 
> * Seules les adresses IPv4 sont prises en charge pour le moment.

## <a name="use-the-azure-portal"></a>Utilisation du portail Azure

Voici comment configurer les pare-feux et les réseaux virtuels Key Vault avec le portail Azure :

1. Accédez au coffre de clés que vous voulez sécuriser.
2. Sélectionnez **Mise en réseau**, puis l’onglet **Pare-feu et réseaux virtuels**.
3. Sous **Autoriser l’accès depuis**, cliquez sur **Réseaux sélectionnés**.
4. Pour ajouter des réseaux virtuels existants à des règles de pare-feux et de réseau virtuel, sélectionnez **+ Ajouter des réseaux virtuels existants**.
5. Dans le nouveau panneau qui s’ouvre, sélectionnez l’abonnement, les réseaux virtuels et les sous-réseaux que vous voulez autoriser à accéder à ce coffre de clés. Si les réseaux virtuels et les sous-réseaux que vous sélectionnez n’ont pas de points de terminaison de service est activés, confirmez que vous voulez activer les points de terminaison de service, puis sélectionnez **Activer**. Jusqu’à 15 minutes peuvent être nécessaires pour que l’activation soit effective.
6. Sous **Réseaux IP**, ajoutez des plages d’adresses IPv4 en utilisant [la notation CIDR (Classless Inter-domain Routing)](https://tools.ietf.org/html/rfc4632) ou en entrant des adresses IP individuelles.
7. Si vous souhaitez autoriser les services Microsoft approuvés à contourner le pare-feu Key Vault, sélectionnez « Oui ». Pour obtenir la liste complète des services approuvés par Key Vault actuels, consultez le lien suivant. [Services approuvés par Azure Key Vault](./overview-vnet-service-endpoints.md#trusted-services)
7. Sélectionnez **Enregistrer**.

Vous pouvez également ajouter de nouveaux réseaux virtuels et des sous-réseaux, puis activer des points de terminaison de service pour ceux-ci en sélectionnant **+ Ajouter un nouveau réseau virtuel**. Puis suivez les invites.

## <a name="use-the-azure-cli"></a>Utilisation de l’interface de ligne de commande Microsoft Azure 

Voici comment configurer les pare-feux et les réseaux virtuels de Key Vault avec Azure CLI

1. [Installez Azure CLI](/cli/azure/install-azure-cli) et [connectez-vous](/cli/azure/authenticate-azure-cli).

2. Listez les règles de réseau virtuel disponibles. Si vous n’avez pas défini de règles pour ce coffre de clés, la liste est vide.
   ```azurecli
   az keyvault network-rule list --resource-group myresourcegroup --name mykeyvault
   ```

3. Activez un point de terminaison de service pour Key Vault sur un réseau virtuel et un sous-réseau existants.
   ```azurecli
   az network vnet subnet update --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --service-endpoints "Microsoft.KeyVault"
   ```

4. Ajoutez une règle de réseau pour un réseau virtuel et un sous-réseau.
   ```azurecli
   subnetid=$(az network vnet subnet show --resource-group "myresourcegroup" --vnet-name "myvnet" --name "mysubnet" --query id --output tsv)
   az keyvault network-rule add --resource-group "demo9311" --name "demo9311premium" --subnet $subnetid
   ```

5. Ajoutez une plage d’adresses IP à partir de laquelle autoriser le trafic.
   ```azurecli
   az keyvault network-rule add --resource-group "myresourcegroup" --name "mykeyvault" --ip-address "191.10.18.0/24"
   ```

6. Si ce coffre de clés doit être accessible à tous les services approuvés, définissez `bypass` sur `AzureServices`.
   ```azurecli
   az keyvault update --resource-group "myresourcegroup" --name "mykeyvault" --bypass AzureServices
   ```

7. Activez les règles de réseau en définissant l’action par défaut sur `Deny`.
   ```azurecli
   az keyvault update --resource-group "myresourcegroup" --name "mekeyvault" --default-action Deny
   ```

## <a name="use-azure-powershell"></a>Utilisation d'Azure PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Voici comment configurer les pare-feux et les réseaux virtuels Key Vault avec PowerShell :

1. Installez la dernière version d’[Azure PowerShell](/powershell/azure/install-az-ps) et [connectez-vous](/powershell/azure/authenticate-azureps).

2. Listez les règles de réseau virtuel disponibles. Si vous n’avez pas défini de règles pour ce coffre de clés, la liste est vide.
   ```powershell
   (Get-AzKeyVault -VaultName "mykeyvault").NetworkAcls
   ```

3. Activez un point de terminaison de service pour Azure Key Vault sur un réseau virtuel et un sous-réseau existants.
   ```powershell
   Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Set-AzVirtualNetworkSubnetConfig -Name "mysubnet" -AddressPrefix "10.1.1.0/24" -ServiceEndpoint "Microsoft.KeyVault" | Set-AzVirtualNetwork
   ```

4. Ajoutez une règle de réseau pour un réseau virtuel et un sous-réseau.
   ```powershell
   $subnet = Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" -Name "myvnet" | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"
   Add-AzKeyVaultNetworkRule -VaultName "mykeyvault" -VirtualNetworkResourceId $subnet.Id
   ```

5. Ajoutez une plage d’adresses IP à partir de laquelle autoriser le trafic.
   ```powershell
   Add-AzKeyVaultNetworkRule -VaultName "mykeyvault" -IpAddressRange "16.17.18.0/24"
   ```

6. Si ce coffre de clés doit être accessible à tous les services approuvés, définissez `bypass` sur `AzureServices`.
   ```powershell
   Update-AzKeyVaultNetworkRuleSet -VaultName "mykeyvault" -Bypass AzureServices
   ```

7. Activez les règles de réseau en définissant l’action par défaut sur `Deny`.
   ```powershell
   Update-AzKeyVaultNetworkRuleSet -VaultName "mykeyvault" -DefaultAction Deny
   ```

## <a name="references"></a>References
* Informations de référence sur les modèles ARM : [Informations de référence sur les modèles ARM pour Azure Key Vault](/azure/templates/Microsoft.KeyVault/vaults)
* Commandes Azure CLI : [az keyvault network-rule](/cli/azure/keyvault/network-rule?view=azure-cli-latest)
* Applets de commande Azure PowerShell : [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault), [Add-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Add-azKeyVaultNetworkRule), [Remove-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Remove-azKeyVaultNetworkRule), [Update-AzKeyVaultNetworkRuleSet](/powershell/module/az.KeyVault/Update-azKeyVaultNetworkRuleSet)

## <a name="next-steps"></a>Étapes suivantes

* [Points de terminaison de service de réseau virtuel pour Azure Key Vault](overview-vnet-service-endpoints.md)
* [Sécurisez votre coffre de clés](secure-your-key-vault.md)