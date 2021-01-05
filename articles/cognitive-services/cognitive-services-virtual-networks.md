---
title: Virtual Network
titleSuffix: Azure Cognitive Services
description: Configurez une sécurité réseau multiniveau pour vos ressources Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 12/04/2020
ms.author: aahi
ms.openlocfilehash: 3b6c2a5a50cedadd8818eae735df55b661e794ef
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97034018"
---
# <a name="configure-azure-cognitive-services-virtual-networks"></a>Configurer des réseaux virtuels Azure Cognitive Services

Azure Cognitive Services propose un modèle de sécurité multiniveau. Ce modèle vous permet de sécuriser vos comptes Cognitive Services pour un sous-ensemble spécifique de réseaux. Quand des règles de réseau sont configurées, seules les applications demandant des données sur l’ensemble de réseaux spécifié peuvent accéder au compte. Vous pouvez limiter l’accès à vos ressources avec le filtrage des demandes. Vous pouvez autoriser uniquement les demandes provenant d’adresses IP ou de plages d’adresses IP spécifiées, ou d’une liste de sous-réseaux des [réseaux virtuels Azure](../virtual-network/virtual-networks-overview.md).

Une application qui accède à une ressource Cognitive Services alors que des règles de réseau sont en vigueur a besoin d’une autorisation. L’autorisation est prise en charge avec des informations d’identification [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md) (Azure AD) ou avec une clé API valide.

> [!IMPORTANT]
> L’activation de règles de pare-feu pour votre compte Cognitive Services bloque les demandes entrantes de données par défaut. Pour autoriser l’entrée des demandes, l’une des conditions suivantes doit être remplie :

> * La demande doit provenir d’un service fonctionnant au sein d’un réseau virtuel Azure figurant dans la liste des sous-réseaux autorisés du compte Cognitive Services cible. Le point de terminaison dans les demandes provenant du réseau virtuel doit être défini en tant que [sous-domaine personnalisé](cognitive-services-custom-subdomains.md) de votre compte Cognitive Services.
> * Ou la demande doit provenir d’une liste d’adresses IP autorisées.
>
> Les demandes qui sont bloquées comprennent les demandes émanant d’autres services Azure, du portail Azure, des services de journalisation et de métriques, etc.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="scenarios"></a>Scénarios

Pour sécuriser votre ressource Cognitive Services, vous devez commencer par configurer une règle pour refuser l’accès au trafic de tous les réseaux (y compris le trafic Internet) par défaut. Ensuite, vous devez configurer des règles qui autorisent l’accès au trafic en provenance de réseaux virtuels spécifiques. Cette configuration vous permet de créer une limite de réseau sécurisée pour vos applications. Vous pouvez également configurer des règles pour accorder l’accès au trafic en provenance de plages d’adresses IP Internet publiques, en autorisant des connexions à partir de clients Internet ou locaux spécifiques.

Les règles de réseau sont appliquées sur tous les protocoles réseau vers Azure Cognitive Services, notamment REST et WebSocket. Pour accéder aux données à l’aide d’outils tels que les consoles de test Azure, vous devez configurer des règles de réseau explicites. Vous pouvez appliquer des règles de réseau à des ressources Cognitive Services existantes ou lorsque vous créez des ressources Cognitive Services. Une fois appliquées, les règles de réseau concernent toutes les demandes.

## <a name="supported-regions-and-service-offerings"></a>Régions et offres de services prises en charge

Les réseaux virtuels (VNET) sont pris en charge dans les [régions où les services Cognitive Services sont disponibles](https://azure.microsoft.com/global-infrastructure/services/). Cognitive Services prend en charge les balises de service pour la configuration des règles de réseau. Les services répertoriés ci-dessous sont inclus dans la balise de service **CognitiveServicesManagement**.

> [!div class="checklist"]
> * Détecteur d’anomalies
> * Vision par ordinateur
> * Content Moderator
> * Vision personnalisée
> * Face
> * Form Recognizer
> * Lecteur immersif
> * Language Understanding (LUIS)
> * Personalizer
> * Services Speech
> * Analyse de texte
> * QnA Maker
> * Traduction de texte Translator Text


> [!NOTE]
> Si vous utilisez LUIS, la balise **CognitiveServicesManagement** vous permet uniquement d’utiliser le service à l’aide du kit de développement logiciel (SDK) ou de l’API REST. Pour accéder au portail LUIS et l’utiliser à partir d’un réseau virtuel, vous devez utiliser les balises suivantes :  
> * **AzureActiveDirectory**
> * **AzureFrontDoor.Frontend**
> * **AzureResourceManager** 
> * **CognitiveServicesManagement**



## <a name="change-the-default-network-access-rule"></a>Changer la règle d’accès réseau par défaut

Par défaut, les ressources Cognitive Services acceptent les connexions des clients de n’importe quel réseau. Pour limiter l’accès aux réseaux sélectionnés, vous devez d’abord changer l’action par défaut.

> [!WARNING]
> Le changement des règles de réseau peut impacter la capacité de vos applications à se connecter à Azure Cognitive Services. La définition de la règle de réseau par défaut sur **Refuser** bloque tout accès aux données, sauf si des règles de réseau spécifiques **accordant** l’accès sont également appliquées. Accordez l’accès uniquement aux réseaux autorisés à l’aide des règles de réseau avant de changer la règle par défaut pour refuser l’accès. Si vous autorisez la liste des adresses IP de votre réseau local, veillez à ajouter toutes les adresses IP publiques sortantes possibles de votre réseau local.

### <a name="managing-default-network-access-rules"></a>Gestion des règles d’accès réseau par défaut

Vous pouvez gérer les règles d’accès réseau par défaut pour les ressources Cognitive Services par le biais du portail Azure, de PowerShell ou d’Azure CLI.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

1. Accédez à la ressource Cognitive Services à sécuriser.

1. Sélectionnez le menu **GESTION DES RESSOURCES** appelé **Réseau virtuel**.

   ![Option Réseau virtuel](media/vnet/virtual-network-blade.png)

1. Pour refuser l’accès par défaut, choisissez d’autoriser l’accès à partir de **Réseaux sélectionnés**. Avec le paramètre **Réseaux sélectionnés** seul, sans aucun **réseau virtuel** ni aucune **plage d’adresses** configurés, tout accès est effectivement refusé. Lorsque tous les accès sont refusés, les demandes de consommation de la ressource Cognitive Services ne sont pas autorisées. Vous pouvez toujours utiliser le portail Azure, Azure PowerShell ou Azure CLI pour configurer la ressource Cognitive Services.
1. Pour autoriser le trafic de tous les réseaux, choisissez d’autoriser l’accès à partir de **Tous les réseaux**.

   ![Refus des réseaux virtuels](media/vnet/virtual-network-deny.png)

1. Sélectionnez **Enregistrer** pour enregistrer vos modifications.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Installez [Azure PowerShell](/powershell/azure/install-az-ps) et [connectez-vous](/powershell/azure/authenticate-azureps) ou sélectionnez **Essayer**.

1. Affichez l’état de la règle par défaut pour la ressource Cognitive Services.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
    }
    (Get-AzCognitiveServicesAccountNetworkRuleSet @parameters).DefaultAction
    ```

1. Définissez la règle par défaut pour refuser l’accès réseau par défaut.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -DefaultAction Deny
    }
    Update-AzCognitiveServicesAccountNetworkRuleSet @parameters
    ```

1. Définissez la règle par défaut pour autoriser l’accès réseau par défaut.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -DefaultAction Allow
    }
    Update-AzCognitiveServicesAccountNetworkRuleSet @parameters
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Installez [Azure CLI](/cli/azure/install-azure-cli) et [connectez-vous](/cli/azure/authenticate-azure-cli) ou sélectionnez **Essayer**.

1. Affichez l’état de la règle par défaut pour la ressource Cognitive Services.

    ```azurecli-interactive
    az cognitiveservices account show \
        -g "myresourcegroup" -n "myaccount" \
        --query networkRuleSet.defaultAction
    ```

1. Définissez la règle par défaut pour refuser l’accès réseau par défaut.

    ```azurecli-interactive
    az cognitiveservices account update \
        -g "myresourcegroup" -n "myaccount" \
        --default-action Deny
    ```

1. Définissez la règle par défaut pour autoriser l’accès réseau par défaut.

    ```azurecli-interactive
    az cognitiveservices account update \
        -g "myresourcegroup" -n "myaccount" \
        --default-action Allow
    ```

**_

## <a name="grant-access-from-a-virtual-network"></a>Accorder l’accès à partir d’un réseau virtuel

Vous pouvez configurer les ressources Cognitive Services de sorte à y autoriser l’accès uniquement à partir de sous-réseaux spécifiques. Les sous-réseaux autorisés peuvent appartenir à un réseau virtuel dans le même abonnement ou dans un autre abonnement, y compris dans un abonnement appartenant à un autre locataire Azure Active Directory.

Activez un [point de terminaison de service](../virtual-network/virtual-network-service-endpoints-overview.md) pour Azure Cognitive Services au sein du réseau virtuel. Le point de terminaison de service route le trafic depuis le réseau virtuel par le biais d’un chemin optimal vers le service Azure Cognitive Services. Les identités du sous-réseau et du réseau virtuel sont également transmises avec chaque demande. Les administrateurs peuvent ensuite configurer des règles de réseau pour la ressource Cognitive Services qui autorisent la réception des demandes à partir de sous-réseaux spécifiques d’un réseau virtuel. Les clients qui obtiennent un accès par le biais de ces règles de réseau doivent continuer à respecter les exigences d’autorisation de la ressource Cognitive Services pour accéder aux données.

Chaque ressource Cognitive Services prend en charge jusqu’à 100 règles de réseau virtuel qui peuvent être combinées avec des [règles de réseau IP](#grant-access-from-an-internet-ip-range).

### <a name="required-permissions"></a>Autorisations requises

Pour appliquer une règle de réseau virtuel à une ressource Cognitive Services, l’utilisateur doit disposer des autorisations appropriées pour les sous-réseaux à ajouter. L’autorisation nécessaire correspond au rôle _Contributeur* par défaut ou au rôle *Contributeur Cognitive Services*. Les autorisations nécessaires peuvent également être ajoutées à des définitions de rôle personnalisées.

La ressource Cognitive Services et les réseaux virtuels auxquels l’accès est accordé peuvent se trouver dans des abonnements différents, y compris des abonnements appartenant à un autre locataire Azure AD.

> [!NOTE]
> La configuration de règles qui accordent l’accès à des sous-réseaux de réseaux virtuels qui font partie d’un autre locataire Azure Active Directory n’est actuellement possible que via PowerShell, l’interface de ligne de commande et des API REST. S’il est possible de consulter ces règles sur le portail Azure, il est impossible de les y configurer.

### <a name="managing-virtual-network-rules"></a>Gestion des règles de réseau virtuel

Vous pouvez gérer les règles de réseau virtuel pour les ressources Cognitive Services par le biais du portail Azure, de PowerShell ou d’Azure CLI.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

1. Accédez à la ressource Cognitive Services à sécuriser.

1. Sélectionnez le menu **GESTION DES RESSOURCES** appelé **Réseau virtuel**.

1. Vérifiez que vous avez choisi d’autoriser l’accès à partir des **Réseaux sélectionnés**.

1. Pour accorder l’accès à un réseau virtuel avec une règle de réseau existante, sélectionnez **Ajouter un réseau virtuel existant** sous **Réseaux virtuels**.

   ![Ajouter un réseau virtuel existant](media/vnet/virtual-network-add-existing.png)

1. Sélectionnez les options **Réseaux virtuels** et **Sous-réseaux**, puis sélectionnez **Activer**.

   ![Ajouter les détails du réseau virtuel existant](media/vnet/virtual-network-add-existing-details.png)

1. Pour créer un réseau virtuel et lui accorder l’accès, sélectionnez **Ajouter un nouveau réseau virtuel**.

   ![Ajouter un nouveau réseau virtuel](media/vnet/virtual-network-add-new.png)

1. Fournissez les informations nécessaires pour créer le nouveau réseau virtuel, puis sélectionnez **Créer**.

   ![Créer un réseau virtuel](media/vnet/virtual-network-create.png)

    > [!NOTE]
    > Si un point de terminaison de service pour Azure Cognitive Services n’a pas déjà été configuré pour le réseau virtuel et les sous-réseaux sélectionnés, vous pouvez le configurer dans le cadre de cette opération.
    >
    > Actuellement, seuls des réseaux virtuels appartenant à un même locataire Azure Active Directory s’affichent pour sélection lors de la création d’une règle. Pour accorder l’accès à un sous-réseau d’un réseau virtuel appartenant à un autre locataire, utilisez PowerShell, l’interface de ligne de commande ou des API REST.

1. Pour supprimer une règle de réseau ou sous-réseau virtuel, sélectionnez **...** pour ouvrir le menu contextuel du réseau ou sous-réseau virtuel, puis sélectionnez **Supprimer**.

   ![Supprimer un réseau virtuel](media/vnet/virtual-network-remove.png)

1. Sélectionnez **Enregistrer** pour enregistrer vos modifications.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Installez [Azure PowerShell](/powershell/azure/install-az-ps) et [connectez-vous](/powershell/azure/authenticate-azureps) ou sélectionnez **Essayer**.

1. Listez les règles de réseau virtuel.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
    }
    (Get-AzCognitiveServicesAccountNetworkRuleSet @parameters).VirtualNetworkRules
    ```

1. Activez le point de terminaison de service pour Azure Cognitive Services sur un réseau virtuel et un sous-réseau existants.

    ```azurepowershell-interactive
    Get-AzVirtualNetwork -ResourceGroupName "myresourcegroup" `
        -Name "myvnet" | Set-AzVirtualNetworkSubnetConfig -Name "mysubnet" `
        -AddressPrefix "10.0.0.0/24" `
        -ServiceEndpoint "Microsoft.CognitiveServices" | Set-AzVirtualNetwork
    ```

1. Ajoutez une règle de réseau pour un réseau virtuel et un sous-réseau.

    ```azurepowershell-interactive
    $subParameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myvnet"
    }
    $subnet = Get-AzVirtualNetwork @subParameters | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"

    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -VirtualNetworkResourceId $subnet.Id
    }
    Add-AzCognitiveServicesAccountNetworkRule @parameters
    ```

    > [!TIP]
    > Pour ajouter une règle de réseau pour un sous-réseau d’un réseau virtuel appartenant à un autre locataire Azure AD, utilisez un paramètre **VirtualNetworkResourceId** complet sous la forme « /subscriptions/subscription-ID/resourceGroups/resourceGroup-Name/providers/Microsoft.Network/virtualNetworks/vNet-name/subnets/subnet-name ».

1. Supprimez une règle de réseau pour un réseau virtuel et un sous-réseau.

    ```azurepowershell-interactive
    $subParameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myvnet"
    }
    $subnet = Get-AzVirtualNetwork @subParameters | Get-AzVirtualNetworkSubnetConfig -Name "mysubnet"

    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -VirtualNetworkResourceId $subnet.Id
    }
    Remove-AzCognitiveServicesAccountNetworkRule @parameters
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Installez [Azure CLI](/cli/azure/install-azure-cli) et [connectez-vous](/cli/azure/authenticate-azure-cli) ou sélectionnez **Essayer**.

1. Listez les règles de réseau virtuel.

    ```azurecli-interactive
    az cognitiveservices account network-rule list \
        -g "myresourcegroup" -n "myaccount" \
        --query virtualNetworkRules
    ```

1. Activez le point de terminaison de service pour Azure Cognitive Services sur un réseau virtuel et un sous-réseau existants.

    ```azurecli-interactive
    az network vnet subnet update -g "myresourcegroup" -n "mysubnet" \
    --vnet-name "myvnet" --service-endpoints "Microsoft.CognitiveServices"
    ```

1. Ajoutez une règle de réseau pour un réseau virtuel et un sous-réseau.

    ```azurecli-interactive
    $subnetid=(az network vnet subnet show \
        -g "myresourcegroup" -n "mysubnet" --vnet-name "myvnet" \
        --query id --output tsv)

    # Use the captured subnet identifier as an argument to the network rule addition
    az cognitiveservices account network-rule add \
        -g "myresourcegroup" -n "myaccount" \
        --subnet $subnetid
    ```

    > [!TIP]
    > Pour ajouter une règle pour un sous-réseau d’un réseau virtuel appartenant à un autre locataire Azure AD, utilisez un ID de sous-réseau complet sous la forme « /subscriptions/subscription-ID/resourceGroups/resourceGroup-Name/providers/Microsoft.Network/virtualNetworks/vNet-name/subnets/subnet-name ».
    > 
    > Vous pouvez utiliser le paramètre **subscription** pour récupérer l’ID de sous-réseau d’un réseau virtuel appartenant à un autre locataire Azure AD.

1. Supprimez une règle de réseau pour un réseau virtuel et un sous-réseau.

    ```azurecli-interactive
    $subnetid=(az network vnet subnet show \
        -g "myresourcegroup" -n "mysubnet" --vnet-name "myvnet" \
        --query id --output tsv)

    # Use the captured subnet identifier as an argument to the network rule removal
    az cognitiveservices account network-rule remove \
        -g "myresourcegroup" -n "myaccount" \
        --subnet $subnetid
    ```

**_

> [!IMPORTANT]
> Veillez à [définir la règle par défaut](#change-the-default-network-access-rule) sur _*Refuser**, sinon les règles de réseau n’ont aucun effet.

## <a name="grant-access-from-an-internet-ip-range"></a>Accorder l’accès à partir d’une plage d’adresses IP Internet

Vous pouvez configurer les ressources Cognitive Services pour autoriser l’accès à partir de plages d’adresses IP Internet publiques spécifiques. Cette configuration donne accès à des services et réseaux locaux spécifiques, permettant de bloquer le trafic Internet général.

Fournissez les plages d’adresses Internet autorisées à l’aide de la [notation CIDR](https://tools.ietf.org/html/rfc4632) sous la forme `16.17.18.0/24` ou sous la forme d’adresses IP individuelles de type `16.17.18.19`.

   > [!Tip]
   > Les petites plages d’adresses qui utilisent les tailles de préfixe « /31 » ou « /32 » ne sont pas prises en charge. Ces plages doivent être configurées à l’aide des règles d’adresses IP individuelles.

Les règles de réseau IP sont autorisées uniquement pour les adresses IP **Internet publiques**. Les plages d’adresses IP réservées aux réseaux privés (comme défini dans [RFC 1918](https://tools.ietf.org/html/rfc1918#section-3)) ne sont pas autorisées dans les règles IP. Les réseaux privés incluent des adresses qui commencent par `10.*`, `172.16.*` - `172.31.*` et `192.168.*`.

   > [!NOTE]
   > Les règles de réseau IP n’ont aucun effet sur les demandes provenant de la même région Azure que celle de la ressource Cognitive Services. Utilisez des [règles de réseau virtuel](#grant-access-from-a-virtual-network) pour autoriser les requêtes de même région.

Seules les adresses IPv4 sont prises en charge pour le moment. Chaque ressource Cognitive Services prend en charge jusqu’à 100 règles de réseau IP qui peuvent être combinées avec des [règles de réseau virtuel](#grant-access-from-a-virtual-network).

### <a name="configuring-access-from-on-premises-networks"></a>Configuration de l’accès à partir de réseaux locaux

Pour accorder l’accès à votre ressource Cognitive Services à partir de réseaux locaux avec une règle de réseau IP, vous devez identifier les adresses IP Internet utilisées par votre réseau. Contactez votre administrateur réseau pour obtenir de l’aide.

Si vous utilisez [ExpressRoute](../expressroute/expressroute-introduction.md) localement pour le peering public ou Microsoft, vous devez identifier les adresses IP NAT (traduction d’adresses réseau). Pour le peering public, chaque circuit ExpressRoute utilise par défaut deux adresses IP NAT. Chacune d’elles s’applique au trafic du service Azure quand ce dernier pénètre le réseau principal de Microsoft Azure. Pour le peering Microsoft, les adresses IP NAT utilisées sont fournies par le client ou par le fournisseur de services. Pour autoriser l’accès à vos ressources de votre service, vous devez autoriser ces adresses IP publiques dans le paramètre de pare-feu IP de ressource. Pour trouver les adresses IP de votre circuit ExpressRoute de peering public, [ouvrez un ticket de support avec ExpressRoute](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) via le portail Azure. Découvrez d’autres informations sur le [peering public et Microsoft NAT pour ExpressRoute.](../expressroute/expressroute-nat.md#nat-requirements-for-azure-public-peering)

### <a name="managing-ip-network-rules"></a>Gestion des règles de réseau IP

Vous pouvez gérer les règles de réseau IP pour les ressources Cognitive Services par le biais du portail Azure, de PowerShell ou d’Azure CLI.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

1. Accédez à la ressource Cognitive Services à sécuriser.

1. Sélectionnez le menu **GESTION DES RESSOURCES** appelé **Réseau virtuel**.

1. Vérifiez que vous avez choisi d’autoriser l’accès à partir des **Réseaux sélectionnés**.

1. Pour accorder l’accès à une plage d’adresses IP Internet, entrez l’adresse IP ou la plage d’adresses IP (au [format CIDR](https://tools.ietf.org/html/rfc4632)) sous **Pare-feu** > **Plages d’adresses**. Seules les adresses IP publiques valides (non réservées) sont acceptées.

   ![Ajouter une plage d’adresses IP](media/vnet/virtual-network-add-ip-range.png)

1. Pour supprimer une règle de réseau IP, sélectionnez l’icône Corbeille <span class="docon docon-delete x-hidden-focus"></span> en regard de la plage d’adresses.

   ![Supprimer une plage d’adresses IP](media/vnet/virtual-network-delete-ip-range.png)

1. Sélectionnez **Enregistrer** pour enregistrer vos modifications.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

1. Installez [Azure PowerShell](/powershell/azure/install-az-ps) et [connectez-vous](/powershell/azure/authenticate-azureps) ou sélectionnez **Essayer**.

1. Listez les règles de réseau IP.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
    }
    (Get-AzCognitiveServicesAccountNetworkRuleSet @parameters).IPRules
    ```

1. Ajoutez une règle de réseau pour une adresse IP individuelle.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.19"
    }
    Add-AzCognitiveServicesAccountNetworkRule @parameters
    ```

1. Ajoutez une règle de réseau pour une plage d’adresses IP.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.0/24"
    }
    Add-AzCognitiveServicesAccountNetworkRule @parameters
    ```

1. Supprimez une règle de réseau pour une adresse IP individuelle.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.19"
    }
    Remove-AzCognitiveServicesAccountNetworkRule @parameters
    ```

1. Supprimez une règle de réseau pour une plage d’adresses IP.

    ```azurepowershell-interactive
    $parameters = @{
        -ResourceGroupName "myresourcegroup"
        -Name "myaccount"
        -IPAddressOrRange "16.17.18.0/24"
    }
    Remove-AzCognitiveServicesAccountNetworkRule @parameters
    ```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Installez [Azure CLI](/cli/azure/install-azure-cli) et [connectez-vous](/cli/azure/authenticate-azure-cli) ou sélectionnez **Essayer**.

1. Listez les règles de réseau IP.

    ```azurecli-interactive
    az cognitiveservices account network-rule list \
        -g "myresourcegroup" -n "myaccount" --query ipRules
    ```

1. Ajoutez une règle de réseau pour une adresse IP individuelle.

    ```azurecli-interactive
    az cognitiveservices account network-rule add \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.19"
    ```

1. Ajoutez une règle de réseau pour une plage d’adresses IP.

    ```azurecli-interactive
    az cognitiveservices account network-rule add \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.0/24"
    ```

1. Supprimez une règle de réseau pour une adresse IP individuelle.

    ```azurecli-interactive
    az cognitiveservices account network-rule remove \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.19"
    ```

1. Supprimez une règle de réseau pour une plage d’adresses IP.

    ```azurecli-interactive
    az cognitiveservices account network-rule remove \
        -g "myresourcegroup" -n "myaccount" \
        --ip-address "16.17.18.0/24"
    ```

**_

> [!IMPORTANT]
> Veillez à [définir la règle par défaut](#change-the-default-network-access-rule) sur _*Refuser**, sinon les règles de réseau n’ont aucun effet.

## <a name="use-private-endpoints"></a>Utiliser des points de terminaison privés

Vous pouvez utiliser des [points de terminaison privés](../private-link/private-endpoint-overview.md) pour vos ressources Cognitive Services afin de permettre aux clients d’un réseau virtuel d’accéder de manière sécurisée aux données via une [liaison privée](../private-link/private-link-overview.md). Le point de terminaison privé utilise une adresse IP de l’espace d’adressage du réseau virtuel pour votre ressource Cognitive Services. Le trafic réseau entre les clients sur le réseau virtuel et la ressource traverse le réseau virtuel et une liaison privée sur le réseau principal de Microsoft, ce qui élimine l’exposition sur l’Internet public.

Les points de terminaison privés pour les ressources Cognitive Services vous permettent de faire ce qui suit :

* Sécuriser votre ressource Cognitive Services en configurant le pare-feu afin de bloquer toutes les connexions sur le point de terminaison public pour le service Cognitive Services.
* Améliorer la sécurité du réseau virtuel en vous permettant de bloquer l’exfiltration des données à partir du réseau virtuel.
* Vous connecter de manière sécurisée aux ressources Cognitive Services à partir de réseaux locaux qui se connectent au réseau virtuel à l’aide d’un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) ou de circuits [ExpressRoute](../expressroute/expressroute-locations.md) avec un peering privé.

### <a name="conceptual-overview"></a>Vue d'ensemble conceptuelle

Un point de terminaison privé est une interface réseau spéciale pour une ressource Azure dans votre [réseau virtuel](../virtual-network/virtual-networks-overview.md). La création d’un point de terminaison privé pour votre ressource Cognitive Services offre une connectivité sécurisée entre les clients de votre réseau virtuel et votre ressource. Une adresse IP est attribuée au point de terminaison privé à partir de la plage d’adresses IP de votre réseau virtuel. La connexion entre le point de terminaison privé et le service Cognitive Services utilise une liaison privée sécurisée.

Les applications du réseau virtuel peuvent se connecter au service sans interruption sur le point de terminaison privé à l’aide des mêmes chaînes de connexion et mécanismes d’autorisation qu’ils utilisent dans tous les cas. La seule exception est les Speech Services qui nécessitent un point de terminaison distinct. Consultez la section relative aux [points de terminaison privés avec les Speech Services](#private-endpoints-with-the-speech-services). Les points de terminaison privés peuvent être utilisés avec tous les protocoles pris en charge par la ressource Cognitive Services, notamment l’API REST.

Vous pouvez créer des points de terminaison privés dans des sous-réseaux qui utilisent des [points de terminaison de service](../virtual-network/virtual-network-service-endpoints-overview.md). Les clients d’un sous-réseau peuvent se connecter à une ressource Cognitive Services à l’aide d’un point de terminaison privé, tout en utilisant des points de terminaison de service pour accéder à d’autres.

Quand vous créez un point de terminaison privé pour une ressource Cognitive Services dans votre réseau virtuel, une demande de consentement est envoyée pour approbation au propriétaire de la ressource Cognitive Services. Si l’utilisateur qui demande la création du point de terminaison privé est également propriétaire de la ressource, cette demande de consentement est automatiquement approuvée.

Les propriétaires de ressource Cognitive Services peuvent gérer les demandes de consentement et les points de terminaison privés via l’onglet *Points de terminaison privés* de la ressource Cognitive Services dans le [portail Azure](https://portal.azure.com).

### <a name="private-endpoints"></a>Instances Private Endpoint

Quand vous créez le point de terminaison privé, vous devez spécifier la ressource Cognitive Services à laquelle il se connecte. Pour plus d’informations sur la création d’un point de terminaison privé, consultez :

* [Créer un point de terminaison privé à l’aide du centre Private Link dans le portail Azure](../private-link/create-private-endpoint-portal.md)
* [Créer un point de terminaison privé à l’aide d’Azure CLI](../private-link/create-private-endpoint-cli.md)
* [Créer un point de terminaison privé à l’aide d’Azure PowerShell](../private-link/create-private-endpoint-powershell.md)

### <a name="connecting-to-private-endpoints"></a>Connexion à des points de terminaison privés

Les clients d’un réseau virtuel qui utilisent le point de terminaison privé doivent utiliser la même chaîne de connexion pour la ressource Cognitive Services que les clients qui se connectent au point de terminaison public. La seule exception est les Speech Services qui nécessitent un point de terminaison distinct. Consultez la section relative aux [points de terminaison privés avec les Speech Services](#private-endpoints-with-the-speech-services). Nous nous appuyons sur la résolution DNS pour router automatiquement les connexions entre le réseau virtuel et la ressource Cognitive Services sur une liaison privée. 

Nous créons une [zone DNS privée](../dns/private-dns-overview.md) attachée au réseau virtuel avec les mises à jour nécessaires pour les points de terminaison privés, par défaut. Toutefois, si vous utilisez votre propre serveur DNS, vous devrez peut-être apporter des modifications supplémentaires à votre configuration DNS. La section sur les [modifications DNS](#dns-changes-for-private-endpoints) ci-dessous décrit les mises à jour requises pour les points de terminaison privés.

### <a name="private-endpoints-with-the-speech-services"></a>Points de terminaison privés avec les Speech Services

Consultez [Utilisation de Speech Services avec des points de terminaison privés fournis par Azure Private Link](Speech-Service/speech-services-private-link.md).

### <a name="dns-changes-for-private-endpoints"></a>Modifications DNS pour les points de terminaison privés

Quand vous créez un point de terminaison privé, l’enregistrement de la ressource CNAME DNS pour la ressource Cognitive Services est mis à jour avec un alias dans un sous-domaine avec le préfixe « *privatelink* ». Par défaut, nous créons également une [zone DNS privée](../dns/private-dns-overview.md) correspondant au sous-domaine « *privatelink* », avec les enregistrements de ressource DNS A pour les points de terminaison privés.

Quand vous résolvez l’URL du point de terminaison à l’extérieur du réseau virtuel avec le point de terminaison privé, elle correspond au point de terminaison public de la ressource Cognitive Services. En cas de résolution à partir du réseau virtuel hébergeant le point de terminaison privé, l’URL du point de terminaison correspond à l’adresse IP du point de terminaison privé.

Cette approche permet d’accéder à la ressource Cognitive Services avec la même chaîne de connexion pour les clients sur le réseau virtuel hébergeant les points de terminaison privés ainsi que les clients en dehors du réseau virtuel.

Si vous utilisez un serveur DNS personnalisé sur votre réseau, les clients doivent pouvoir résoudre le nom de domaine complet (FQDN) du point de terminaison de la ressource Cognitive Services en adresse IP du point de terminaison privé. Configurez votre serveur DNS de manière à déléguer votre sous-domaine de liaison privée à la zone DNS privée du réseau virtuel.

> [!TIP]
> Quand vous utilisez un serveur DNS personnalisé ou local, vous devez configurer votre serveur DNS pour résoudre le nom de la ressource Cognitive Services dans le sous-domaine « privatelink » en adresse IP du point de terminaison privé. Pour ce faire, vous pouvez déléguer le sous-domaine « privatelink » à la zone DNS privée du réseau virtuel ou configurer la zone DNS sur votre serveur DNS et ajouter les enregistrements A DNS.

Pour plus d’informations sur la configuration de votre propre serveur DNS pour la prise en charge des points de terminaison privés, reportez-vous aux articles suivants :

* [Résolution de noms pour des ressources dans les réseaux virtuels Azure](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server)
* [Configuration DNS pour les points de terminaison privés](../private-link/private-endpoint-overview.md#dns-configuration)

### <a name="pricing"></a>Tarifs

Pour plus d’informations sur les tarifs, consultez [Tarification Liaison privée Azure](https://azure.microsoft.com/pricing/details/private-link).

## <a name="next-steps"></a>Étapes suivantes

* Explorer les offres [Azure Cognitive Services](./what-are-cognitive-services.md)
* En savoir plus sur les [points de terminaison de service de réseau virtuel Azure](../virtual-network/virtual-network-service-endpoints-overview.md)