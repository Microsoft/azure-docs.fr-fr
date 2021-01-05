---
title: Configurer une règle WAF de restriction d’adresse IP pour Azure Front Door
description: Découvrez comment configurer une règle de Pare-feu d’application web afin de limiter les adresses IP pour un point de terminaison Azure Front Door existant.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.topic: article
ms.date: 03/26/2020
ms.author: tyao
ms.openlocfilehash: f260bfc7b097931cc1a978e790c1d9dd966703ac
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94563509"
---
# <a name="configure-an-ip-restriction-rule-with-a-web-application-firewall-for-azure-front-door"></a>Configurer une règle de restriction IP avec un Pare-feu d’applications web pour Azure Front Door

Cet article vous montre comment configurer des règles de restriction IP dans un Pare-feu d’applications web (WAF) pour Azure Front Door à l’aide du portail Azure, d’Azure CLI, d’Azure PowerShell ou d’un modèle Azure Resource Manager.

Une règle de contrôle d’accès basé sur l’adresse IP est une règle WAF personnalisée qui vous permet de contrôler l’accès à vos applications web. Elle remplit cette fonction en spécifiant une liste d’adresses IP ou de plages d’adresses IP dans un format CIDR (Classless Inter-Domain Routing).

Par défaut, votre application web est accessible depuis Internet. Si vous souhaitez limiter l’accès aux clients à partir d’une liste d’adresses IP ou de plages d’adresses IP connues, vous pouvez créer une règle de correspondance IP qui contient la liste des adresses IP en tant que valeurs correspondantes et définit l’opérateur sur « Non » (en attribuant au paramètre negate la valeur true) et l’action sur **Block**. Une fois une règle de restriction IP appliquée, les demandes provenant d’adresses qui ne figurent pas dans cette liste autorisée recevront une réponse 403 Interdit.

## <a name="configure-a-waf-policy-with-the-azure-portal"></a>Configurer une stratégie WAF avec le portail Azure

### <a name="prerequisites"></a>Prérequis

Créez un profil Azure Front Door en suivant les instructions décrites dans [Démarrage rapide : Créer une porte d’entrée pour une application web globale hautement disponible](../../frontdoor/quickstart-create-front-door.md).

### <a name="create-a-waf-policy"></a>Créer une stratégie de pare-feu d’applications web (WAF)

1. Dans le portail Azure, sélectionnez **Créer une ressource**, saisissez **Pare-feu d’applications web** dans la zone de recherche, puis sélectionnez **Pare-feu d’applications web (WAF)** .
2. Sélectionnez **Create** (Créer).
3. Dans la page **Créer une stratégie WAF**, utilisez les valeurs suivantes pour renseigner l’onglet **De base** :
   
   |Paramètre  |Valeur  |
   |---------|---------|
   |Stratégie pour     |WAF global (Front Door)|
   |Abonnement     |Sélectionnez votre abonnement|
   |Resource group     |Sélectionnez le groupe de ressources où se trouve votre instance Front Door.|
   |Nom de stratégie     |Saisissez un nom pour votre stratégie|
   |État de la stratégie     |activé|

   Sélectionnez **Suivant : Paramètres de stratégie**

1. Sous l’onglet **Paramètres de stratégie**, sélectionnez **Prévention**. Pour le **Corps de réponse de bloc**, saisissez *Vous avez été bloqué !* pour que vous puissiez vérifier que votre règle personnalisée est appliquée.
2. Sélectionnez **Suivant : Règles managées**.
3. Sélectionnez **Suivant : Règles personnalisées**.
4. Sélectionnez **Ajouter une règle personnalisée**.
5. Dans la page **Ajouter une règle personnalisée**, utilisez les valeurs de test suivantes pour créer une règle personnalisée :

   |Paramètre  |Valeur  |
   |---------|---------|
   |Nom de la règle personnalisée     |FdWafCustRule|
   |Statut     |activé|
   |Type de règle     |Correspond|
   |Priority    |100|
   |Type de correspondance     |Adresse IP|
   |Variable de correspondance|RemoteAddr|
   |Opération|Ne contient pas|
   |Adresse IP ou plage d'adresses IP|10.10.10.0/24|
   |Alors|Refuser le trafic|

   :::image type="content" source="../media/waf-front-door-configure-ip-restriction/custom-rule.png" alt-text="Règle personnalisée":::

   Sélectionnez **Ajouter**.
6. Sélectionnez **Suivant : Association**.
7. Sélectionnez **Ajouter un hôte front-end**.
8. Pour **Hôte front-end**, sélectionnez votre hôte front-end, puis sélectionnez **Ajouter**.
9. Sélectionnez **Revoir + créer**.
10. Une fois la validation de votre stratégie réussie, sélectionnez **Créer**.

### <a name="test-your-waf-policy"></a>Tester votre stratégie WAF

1. Une fois le déploiement de la stratégie WAF terminé, accédez au nom de votre hôte front-end Front Door.
2. Vous devez voir votre message de bloc personnalisé.

   :::image type="content" source="../media/waf-front-door-configure-ip-restriction/waf-rule-test.png" alt-text="Test de règle WAF":::

   > [!NOTE]
   > Une adresse IP privée a été utilisée intentionnellement dans la règle personnalisée pour garantir le déclenchement de la règle. Dans un déploiement réel, créez des règles *Autoriser* et *Refuser* à l’aide d’adresses IP pour votre situation particulière.

## <a name="configure-a-waf-policy-with-the-azure-cli"></a>Configurer une stratégie WAF avec Azure CLI

### <a name="prerequisites"></a>Prérequis
Avant de commencer à configurer une stratégie de restriction d’IP, configurez votre environnement CLI et créez un profil Azure Front Door.

#### <a name="set-up-the-azure-cli-environment"></a>Configurer l'environnement Azure CLI
1. Installez [Azure CLI](/cli/azure/install-azure-cli) ou utilisez Azure Cloud Shell. Azure Cloud Shell est un interpréteur de commandes Bash gratuit, que vous pouvez exécuter directement dans le portail Azure. L’interface Azure CLI est préinstallée et configurée pour être utilisée avec votre compte. Sélectionnez le bouton **Essayez-le** dans les commandes CLI qui suivent, puis connectez-vous à votre compte Azure dans la session Cloud Shell qui s’ouvre. Une fois la session ouverte, saisissez `az extension add --name front-door` pour ajouter l’extension Azure Front Door.
 2. Si vous utilisez l’interface CLI localement dans Bash, connectez-vous à Azure à l’aide de `az login`.

#### <a name="create-an-azure-front-door-profile"></a>Créer un profil Azure Front Door
Créez un profil Azure Front Door en suivant les instructions décrites dans [Démarrage rapide : Créer une porte d’entrée pour une application web globale hautement disponible](../../frontdoor/quickstart-create-front-door.md).

### <a name="create-a-waf-policy"></a>Créer une stratégie de pare-feu d’applications web (WAF)

Créer une stratégie WAF à l’aide de la commande [az network front-door waf-policy create](/cli/azure/ext/front-door/network/front-door/waf-policy?view=azure-cli-latest#ext-front-door-az-network-front-door-waf-policy-create). Dans l’exemple suivant, remplacez le nom de la stratégie *IPAllowPolicyExampleCLI* par un nom unique.

```azurecli-interactive 
az network front-door waf-policy create \
  --resource-group <resource-group-name> \
  --subscription <subscription ID> \
  --name IPAllowPolicyExampleCLI
  ```
### <a name="add-a-custom-ip-access-control-rule"></a>Ajouter une règle de contrôle d’accès IP personnalisée

Utilisez la commande [az network front-door waf-policy custom-rule create](/cli/azure/ext/front-door/network/front-door/waf-policy/rule?view=azure-cli-latest#ext-front-door-az-network-front-door-waf-policy-rule-create) pour ajouter une règle de contrôle d’accès IP personnalisée pour une stratégie WAF que vous venez de créer.

Dans les exemples suivants :
-  Remplacez *IPAllowPolicyExampleCLI* par votre stratégie unique créée précédemment.
-  Remplacez *ip-address-range-1*, *ip-address-range-2* par votre propre plage.

Tout d’abord, créez une règle d’autorisation IP pour la stratégie créée à l’étape précédente. 
> [!NOTE]
> **--defer** est requis, car une règle doit avoir une condition de correspondance pour être ajoutée à l’étape suivante.

```azurecli
az network front-door waf-policy rule create \
  --name IPAllowListRule \
  --priority 1 \
  --rule-type MatchRule \
  --action Block \
  --resource-group <resource-group-name> \
  --policy-name IPAllowPolicyExampleCLI --defer
```
Ensuite, ajoutez une condition de correspondance à la règle :

```azurecli
az network front-door waf-policy rule match-condition add \
--match-variable RemoteAddr \
--operator IPMatch \
--values "ip-address-range-1" "ip-address-range-2" \
--negate true \
--name IPAllowListRule \
  --resource-group <resource-group-name> \
  --policy-name IPAllowPolicyExampleCLI 
  ```
                                                   
### <a name="find-the-id-of-a-waf-policy"></a>Rechercher l’ID d’une stratégie WAF 
Recherchez une stratégie WAF à l’aide de la commande [az network front-door waf-policy show](/cli/azure/ext/front-door/network/front-door/waf-policy?view=azure-cli-latest#ext-front-door-az-network-front-door-waf-policy-show). Là encore, remplacez *IPAllowPolicyExampleCLI* dans l’exemple suivant par votre stratégie unique que vous avez créée précédemment.

   ```azurecli
   az network front-door  waf-policy show \
     --resource-group <resource-group-name> \
     --name IPAllowPolicyExampleCLI
   ```

### <a name="link-a-waf-policy-to-an-azure-front-door-front-end-host"></a>Lier une stratégie WAF à un hôte front-end Azure Front Door

Définissez l’ID *WebApplicationFirewallPolicyLink* d’Azure Front Door sur l’ID de la stratégie à l’aide de la commande [az network front-door update](/cli/azure/ext/front-door/network/front-door?view=azure-cli-latest#ext-front-door-az-network-front-door-update). Remplacez *IPAllowPolicyExampleCLI* par votre stratégie unique créée précédemment.

   ```azurecli
   az network front-door update \
     --set FrontendEndpoints[0].WebApplicationFirewallPolicyLink.id=/subscriptions/<subscription ID>/resourcegroups/resource-group-name/providers/Microsoft.Network/frontdoorwebapplicationfirewallpolicies/IPAllowPolicyExampleCLI \
     --name <frontdoor-name>
     --resource-group <resource-group-name>
   ```
Dans cet exemple, la stratégieWAF est appliquée à **FrontendEndpoints [0]** . Vous pouvez lier la stratégie WAF à n’importe quel front-end en votre possession.
> [!Note]
> Vous devez définir la propriété **WebApplicationFirewallPolicyLink** une seule fois pour lier une stratégie WAF à un front-end Azure Front Door. Les mises à jour de stratégie suivantes sont automatiquement appliquées au front-end.

## <a name="configure-a-waf-policy-with-azure-powershell"></a>Configurer une stratégie WAF avec Azure PowerShell

### <a name="prerequisites"></a>Prérequis
Avant de commencer à configurer une stratégie de restriction d’IP, configurez votre environnement PowerShell et créez un profil Azure Front Door.

#### <a name="set-up-your-powershell-environment"></a>Configurer votre environnement PowerShell
Azure PowerShell fournit un ensemble de cmdlets qui utilisent le modèle [Azure Resource Manager](../../azure-resource-manager/management/overview.md) pour gérer des ressources Azure.

Vous pouvez installer [Azure PowerShell](/powershell/azure/) sur votre ordinateur local et l’utiliser sur n’importe quelle session PowerShell. Suivez les instructions sur la page pour vous connecter à PowerShell avec vos informations d’identification Azure, puis installez le module Az.

1. Connectez-vous à Azure avec la commande suivante, puis utilisez une boîte de dialogue interactive pour vous connecter.
    ```
    Connect-AzAccount
    ```
 2. Avant d’installer le module Azure Front Door, assurez-vous que vous avez installé la version actuelle du module PowerShellGet. Exécutez la commande suivante, puis réouvrez PowerShell.

    ```
    Install-Module PowerShellGet -Force -AllowClobber
    ``` 

3. Installez le module Az.FrontDoor du collecteur de données à l’aide de la commande suivante. 
    
    ```
    Install-Module -Name Az.FrontDoor
    ```
### <a name="create-an-azure-front-door-profile"></a>Créer un profil Azure Front Door
Créez un profil Azure Front Door en suivant les instructions décrites dans [Démarrage rapide : Créer une porte d’entrée pour une application web globale hautement disponible](../../frontdoor/quickstart-create-front-door.md).

### <a name="define-an-ip-match-condition"></a>Définir une condition de correspondance IP
Utilisez la commande [New-AzFrontDoorWafMatchConditionObject](/powershell/module/az.frontdoor/new-azfrontdoorwafmatchconditionobject) pour définir une condition de correspondance IP.
Dans l’exemple suivante, remplacez *ip-address-range-1*, *ip-address-range-2* par votre propre plage.    
```powershell
$IPMatchCondition = New-AzFrontDoorWafMatchConditionObject `
-MatchVariable  RemoteAddr `
-OperatorProperty IPMatch `
-MatchValue "ip-address-range-1", "ip-address-range-2"
-NegateCondition 1
```
     
### <a name="create-a-custom-ip-allow-rule"></a>Créer une règle d’autorisation IP personnalisée

Utilisez la commande [New-AzFrontDoorWafCustomRuleObject](/powershell/module/Az.FrontDoor/New-azfrontdoorwafcustomruleobject) pour définir une action et une priorité. Dans l’exemple suivant, les requêtes ne provenant pas d’adresses IP clientes qui correspondent à la liste sont autorisées.

```azurepowershell
$IPAllowRule = New-AzFrontDoorWafCustomRuleObject `
-Name "IPAllowRule" `
-RuleType MatchRule `
-MatchCondition $IPMatchCondition `
-Action Block -Priority 1
```

### <a name="configure-a-waf-policy"></a>Configurer une stratégie WAF
Recherchez le nom du groupe de ressources qui contient le profil Azure Front Doorà l’aide de `Get-AzResourceGroup`. Configurez ensuite une stratégie WAF avec la règle IP à l’aide de la commande [New-AzFrontDoorWafPolicy](/powershell/module/az.frontdoor/new-azfrontdoorwafpolicy).

```azurepowershell
  $IPAllowPolicyExamplePS = New-AzFrontDoorWafPolicy `
    -Name "IPRestrictionExamplePS" `
    -resourceGroupName <resource-group-name> `
    -Customrule $IPAllowRule`
    -Mode Prevention `
    -EnabledState Enabled
   ```

### <a name="link-a-waf-policy-to-an-azure-front-door-front-end-host"></a>Lier une stratégie WAF à un hôte front-end Azure Front Door

Liez un objet de stratégie WAF à un hôte front-end existant et mettez à jour les propriétés d’Azure Front Door. Récupérez tout d’abord l’objet Azure Front Door avec la commande [Get-AzFrontDoor](/powershell/module/Az.FrontDoor/Get-AzFrontDoor). Définissez ensuite la propriété **WebApplicationFirewallPolicyLink** sur l’ID de la ressource de *$IPAllowPolicyExamplePS*, créée à l’étape précédente, via la commande [Set-AzFrontDoor](/powershell/module/Az.FrontDoor/Set-AzFrontDoor).

```azurepowershell
  $FrontDoorObjectExample = Get-AzFrontDoor `
    -ResourceGroupName <resource-group-name> `
    -Name $frontDoorName
  $FrontDoorObjectExample[0].FrontendEndpoints[0].WebApplicationFirewallPolicyLink = $IPBlockPolicy.Id
  Set-AzFrontDoor -InputObject $FrontDoorObjectExample[0]
```

> [!NOTE]
> Dans cet exemple, la stratégieWAF est appliquée à **FrontendEndpoints [0]** . Vous pouvez lier une stratégie WAF à n’importe quel front-end en votre possession. Vous devez définir la propriété **WebApplicationFirewallPolicyLink** une seule fois pour lier une stratégie WAF à un front-end Azure Front Door. Les mises à jour de stratégie suivantes sont automatiquement appliquées au front-end.


## <a name="configure-a-waf-policy-with-a-resource-manager-template"></a>Configurer une stratégie WAF avec un modèle Resource Manager
Pour afficher le modèle qui crée une stratégie Azure Front Door et WAF avec des règles de restriction IP personnalisées, rendez-vous sur [GitHub](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-waf-clientip).


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [Créer un profil Azure Front Door](../../frontdoor/quickstart-create-front-door.md).