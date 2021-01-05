---
title: Configurer le délai d’inactivité et la réinitialisation TCP de l’équilibreur de charge
titleSuffix: Azure Load Balancer
description: Dans cet article, découvrez comment configurer le délai d’inactivité et la réinitialisation TCP d’Azure Load Balancer.
services: load-balancer
documentationcenter: na
author: asudbring
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/26/2020
ms.author: allensu
ms.openlocfilehash: 8a6be588544883b77c3ff115c9dba5e6ecd5fbd7
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92747209"
---
# <a name="configure-tcp-reset-and-idle-timeout-for-azure-load-balancer"></a>Configurer le délai d’inactivité et la réinitialisation TCP pour Azure Load Balancer

Azure Load Balancer a la plage de délai d’inactivité suivante :

* 4 à 100 minutes pour les règles de trafic sortant
* 4 à 30 minutes pour les règles d’équilibrage de charge et les règles NAT de trafic entrant

Par défaut, il est défini sur 4 minutes. Si une période d’inactivité est supérieure à la valeur du délai d’expiration, il n’est pas garanti que la session TCP ou HTTP soit maintenue entre le client et votre service. 

Les sections suivantes décrivent comment modifier les paramètres de délai d’inactivité et de réinitialisation TCP pour les ressources d’équilibreur de charge.

## <a name="set-tcp-reset-and-idle-timeout"></a>Définir la réinitialisation TCP et le délai d’inactivité
---
# <a name="portal"></a>[**Portail**](#tab/tcp-reset-idle-portal)

Pour définir le délai d’inactivité et la réinitialisation TCP pour un équilibreur de charge, modifiez la règle d’équilibrage de charge. 

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Dans le menu de gauche, sélectionnez **Groupes de ressources**.

3. Sélectionnez le groupe de ressources pour votre équilibreur de charge. Dans cet exemple, le groupe de ressources est nommé **myResourceGroup**.

4. Sélectionnez votre équilibreur de charge. Dans cet exemple, l’équilibreur de charge est nommé **myLoadBalancer**.

5. Dans **Paramètres** , sélectionnez **Règles d’équilibrage de charge**.

     :::image type="content" source="./media/load-balancer-tcp-idle-timeout/portal-lb-rules.png" alt-text="Modifier les règles de l’équilibreur de charge." border="true":::

6. Sélectionnez votre règle d’équilibrage de charge. Dans cet exemple, la règle d’équilibrage de charge est nommée **myLBrule**.

7. Dans la règle d’équilibrage de charge, déplacez le curseur dans **Délai d’inactivité (minutes)** sur la valeur du délai d’attente.  

8. Sous **Réinitialisation TCP** , sélectionnez **Activé**.

   :::image type="content" source="./media/load-balancer-tcp-idle-timeout/portal-lb-rules-tcp-reset.png" alt-text="Définir le délai d’inactivité et la réinitialisation TCP." border="true":::

9. Sélectionnez **Enregistrer**.

# <a name="powershell"></a>[**PowerShell**](#tab/tcp-reset-idle-powershell)

Pour définir le délai d’inactivité et la réinitialisation TCP, définissez les valeurs des paramètres de règle d’équilibrage de charge suivants avec [Set-AzLoadBalancer](/powershell/module/az.network/set-azloadbalancer) :

* **IdleTimeoutInMinutes**
* **EnableTcpReset**

Si vous choisissez d’installer et d’utiliser PowerShell en local, vous devez exécuter le module Azure PowerShell version 5.4.1 ou ultérieure pour les besoins de cet article. Exécutez `Get-Module -ListAvailable Az` pour rechercher la version installée. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Si vous exécutez PowerShell en local, vous devez également exécuter `Connect-AzAccount` pour créer une connexion avec Azure.

Remplacez les exemples suivants par les valeurs de vos ressources :

* **myResourceGroup**
* **myLoadBalancer**

```azurepowershell
$lb = Get-AzLoadBalancer -Name "myLoadBalancer" -ResourceGroup "myResourceGroup"
$lb.LoadBalancingRules[0].IdleTimeoutInMinutes = '15'
$lb.LoadBalancingRules[0].EnableTcpReset = 'true'
Set-AzLoadBalancer -LoadBalancer $lb
```

# <a name="azure-cli"></a>[**Azure CLI**](#tab/tcp-reset-idle-cli)

Pour définir le délai d’inactivité et la réinitialisation TCP, utilisez les paramètres suivants pour [az network lb rule update](/cli/azure/network/lb/rule?az_network_lb_rule_update) :

* **--idle-timeout**
* **--enable-tcp-reset**

Validez votre environnement avant de commencer :

* Connectez-vous au portail Azure et vérifiez que votre abonnement est actif en exécutant la commande `az login`.
* Vérifiez votre version d’Azure CLI dans un terminal ou une fenêtre de commande en exécutant `az --version`. Pour obtenir la version la plus récente, consultez les [notes de publication les plus récentes](/cli/azure/release-notes-azure-cli?tabs=azure-cli).
  * Si vous ne disposez pas de la dernière version, mettez à jour votre installation en suivant le [guide d’installation pour votre système d’exploitation ou votre plateforme](/cli/azure/install-azure-cli).

Remplacez les exemples suivants par les valeurs de vos ressources :

* **myResourceGroup**
* **myLoadBalancer**
* **myLBrule**


```azurecli
az network lb rule update \
    --resource-group myResourceGroup \
    --name myLBrule \
    --lb-name myLoadBalancer \
    --idle-timeout 15 \
    --enable-tcp-reset true
```
---
## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le délai d’inactivité et la réinitialisation TCP, consultez [Réinitialisation TCP de l’équilibreur de charge et délai d’inactivité](load-balancer-tcp-reset.md).

Pour plus d’informations sur la configuration du mode de distribution de l’équilibreur de charge, consultez [Configurer un mode de distribution d’équilibrage de charge](load-balancer-distribution-mode.md).
