---
title: 'Tutoriel : Mettre à l’échelle et protéger une application web à l’aide d’Azure Front Door et d’Azure Web Application Firewall (WAF)'
description: Ce tutoriel vous explique comment utiliser Azure Web Application Firewall avec le service Azure Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/01/2020
ms.author: duau
ms.openlocfilehash: a5d51a77b1da0ae44c76d0187113105c4e53c9b4
ms.sourcegitcommit: b6f3ccaadf2f7eba4254a402e954adf430a90003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92279218"
---
# <a name="tutorial-quickly-scale-and-protect-a-web-application-by-using-azure-front-door-and-azure-web-application-firewall-waf"></a>Tutoriel : Mettre à l’échelle et protéger rapidement une application web à l’aide d’Azure Front Door et d’Azure Web Application Firewall (WAF)

Beaucoup d’applications web ont connu une augmentation rapide du trafic au cours des dernières semaines en raison du COVID-19. Ces applications web enregistrent également une augmentation du trafic malveillant, notamment des attaques par déni de service. Il existe un moyen efficace d’effectuer un scale-out de votre application face aux pics de trafic, et de vous protéger contre des attaques : configurer Azure Front Door avec Azure WAF comme une couche d’accélération, de mise en cache et de sécurité devant votre application web. Cet article fournit des conseils sur la façon d’obtenir cette configuration d’Azure Front Door avec Azure WAF pour toutes les applications web s’exécutant tant à l’intérieur qu’à l’extérieur d’Azure. 

Dans ce tutoriel, nous allons utiliser Azure CLI pour configurer le WAF. Vous pouvez effectuer cette même opération à l’aide du portail Azure, d’Azure PowerShell, d’Azure Resource Manager ou des API REST Azure. 

Ce didacticiel vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> - Créer une instance Front Door
> - Créer une stratégie Azure WAF
> - Configurez des ensembles de règles pour une stratégie WAF.
> - Associez une stratégie WAF à Front Door.
> - Configurez un domaine personnalisé.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

- Les instructions de ce tutoriel utilisent Azure CLI. [Consultez ce guide](/cli/azure/get-started-with-azure-cli?preserve-view=true&view=azure-cli-latest) pour bien démarrer avec Azure CLI.

  > [!TIP] 
  > Une méthode simple et rapide pour bien démarrer avec Azure CLI consiste à utiliser [Bash dans Azure Cloud Shell](../cloud-shell/quickstart.md).

- Assurez-vous que l’extension `front-door` est ajoutée à Azure CLI :

   ```azurecli-interactive 
   az extension add --name front-door
   ```

> [!NOTE] 
> Pour plus d’informations sur les commandes utilisées dans ce tutoriel, consultez [Informations de référence sur Azure CLI pour Front Door](/cli/azure/ext/front-door/?preserve-view=true&view=azure-cli-latest).

## <a name="create-an-azure-front-door-resource"></a>Créer une ressource Azure Front Door

```azurecli-interactive 
az network front-door create --backend-address <>  --accepted-protocols <> --name <> --resource-group <>
```

`--backend-address` : Nom de domaine complet (FQDN) de l’application que vous souhaitez protéger. Par exemple, `myapplication.contoso.com`.

`--accepted-protocols` : Spécifie les protocoles qu’Azure Front Door doit prendre en charge pour votre application web. Par exemple, `--accepted-protocols Http Https`.

`--name` : Nom de votre ressource Azure Front Door.

`--resource-group` : Groupe de ressources que vous souhaitez placer au sein de la ressource Azure Front Door. Pour en savoir plus sur les groupes de ressources, consultez [Gérer les groupes de ressources dans Azure](../azure-resource-manager/management/manage-resource-groups-portal.md).

Dans la réponse que vous recevez lorsque vous exécutez cette commande, recherchez la clé `hostName`. Vous aurez besoin de cette valeur ultérieurement. `hostName` correspond au nom DNS de la ressource Azure Front Door que vous avez créée.

## <a name="create-an-azure-waf-profile-to-use-with-azure-front-door-resources"></a>Créer un profil Azure WAF à utiliser avec les ressources Azure Front Door

```azurecli-interactive 
az network front-door waf-policy create --name <>  --resource-group <>  --disabled false --mode Prevention
```

`--name` : Nom de la nouvelle stratégie Azure WAF.

`--resource-group` : Groupe de ressources que vous souhaitez placer au sein de la ressource WAF. 

Le code CLI précédent crée une stratégie WAF qui est activée et en mode de prévention. 

> [!NOTE] 
> Vous souhaiterez peut-être créer la stratégie WAF en mode de détection et observer la manière dont elle détecte et journalise les requêtes malveillantes (sans les bloquer) avant de d’opter pour le mode de protection.

Dans la réponse que vous recevez lorsque vous exécutez cette commande, recherchez la clé `ID`. Vous aurez besoin de cette valeur ultérieurement. 

Le champ `ID` doit respecter le format suivant :

/subscriptions/**subscription id**/resourcegroups/**resource group name**/providers/Microsoft.Network/frontdoorwebapplicationfirewallpolicies/**WAF policy name**

## <a name="add-managed-rule-sets-to-the-waf-policy"></a>Ajouter des ensembles de règles managés à la stratégie WAF

Vous pouvez ajouter des ensembles de règles managés à la stratégie WAF. Un ensemble de règles managé est un ensemble de règles créé et managé par Microsoft, qui vous permet de vous protéger contre une classe de menaces. Dans cet exemple, nous ajoutons deux ensembles de règles :
- L’ensemble de règles par défaut, qui vous permet de vous protéger contre les menaces web courantes. 
- L’ensemble de règles de protection bot, qui vous permet de vous protéger contre les bots malveillants.

Ajouter l’ensemble de règles par défaut :

   ```azurecli-interactive 
   az network front-door waf-policy managed-rules add --policy-name <> --resource-group <> --type DefaultRuleSet --version 1.0
   ```

Ajouter l’ensemble de règles de protection bot :

   ```azurecli-interactive 
   az network front-door waf-policy managed-rules add --policy-name <> --resource-group <> --type Microsoft_BotManagerRuleSet --version 1.0
   ```

`--policy-name` : Nom que vous avez spécifié pour votre ressource WAF Azure.

`--resource-group` : Groupe de ressources dans lequel vous avez placé la ressource WAF.

## <a name="associate-the-waf-policy-with-the-azure-front-door-resource"></a>Associer la stratégie WAF à la ressource Azure Front Door

Au cours de cette étape, nous allons associer la stratégie WAF que nous avons créée avec la ressource Azure Front Door qui se trouve avant votre application web :

```azurecli-interactive 
az network front-door update --name <> --resource-group <> --set frontendEndpoints[0].webApplicationFirewallPolicyLink='{"id":"<>"}'
```

`--name` : Nom que vous avez spécifié pour votre ressource Azure Front Door.

`--resource-group` : Groupe de ressources dans lequel vous avez placé la ressource Azure Front Door.

`--set` : C’est là que vous mettez à jour l’attribut `WebApplicationFirewallPolicyLink` pour le `frontendEndpoint` associé à votre ressource Azure Front Door avec la nouvelle stratégie WAF. Vous devez disposer de l’ID de la stratégie WAF de la réponse reçue lorsque vous avez créé le profil WAF plus tôt dans ce tutoriel.

 > [!NOTE] 
> L’exemple précédent s’applique lorsque vous n’utilisez pas de domaine personnalisé. Si vous n’utilisez pas de domaines personnalisés pour accéder à vos applications web, vous pouvez ignorer la section suivante. Dans ce cas, vous donnez à vos clients la `hostName` que vous avez obtenue lors de la création de la ressource Azure Front Door. Ils utiliseront cette `hostName` pour accéder à votre application Web.

## <a name="configure-the-custom-domain-for-your-web-application"></a>Configurer le domaine personnalisé pour votre application web

Le nom de domaine personnalisé de votre application web est celui que les clients utilisent pour faire référence à votre application. Par exemple, contoso.com. Au départ, ce nom de domaine personnalisé pointe vers l’emplacement où il s’exécutait avant l’introduction d’Azure Front Door. Une fois que vous avez ajouté Azure Front Door Service et Azure WAF en première ligne de l’application, l’entrée DNS correspondant à ce domaine personnalisé doit pointer vers la ressource Azure Front Door. Vous pouvez effectuer cette modification en remappant l’entrée de votre serveur DNS vers l’instance Azure Front Door `hostName` que vous avez notée lors de la création de la ressource Azure Front Door.

La procédure spécifique de mise à jour de vos enregistrements DNS dépend de votre fournisseur de services DNS. Si vous utilisez Azure DNS pour héberger votre nom DNS, vous pouvez consulter la documentation relative à la [procédure de mise à jour d’un enregistrement DNS](../dns/dns-operations-recordsets-cli.md) et pointer vers l’instance Azure Front Door `hostName`. 

Il existe un aspect important à noter si vos clients doivent accéder à votre site web à l’aide de l’apex de la zone (par exemple, contoso.com). Dans ce cas, vous devez utiliser Azure DNS et son [type d’enregistrement d’alias](../dns/dns-alias.md) pour héberger votre nom DNS. 

Vous devez également mettre à jour votre configuration Azure Front Door afin d’y [ajouter le domaine personnalisé](./front-door-custom-domain.md) pour permettre la prise en compte de ce mappage.

Enfin, si vous utilisez un domaine personnalisé pour accéder à votre application web et souhaitez activer le protocole HTTPS, vous devez [configurer les certificats pour votre domaine personnalisé dans Azure Front Door](./front-door-custom-domain-https.md). 

## <a name="lock-down-your-web-application"></a>Verrouiller votre application web

Nous vous recommandons de vous assurer que seuls des périphéries d’Azure Front Door peuvent communiquer avec votre application web. Ainsi, personne ne peut contourner la protection Azure Front Door et accéder directement à votre application. Pour effectuer ce verrouillage, consultez [Comment restreindre l'accès à mon back-end uniquement à Azure Front Door ?](./front-door-faq.md#how-do-i-lock-down-the-access-to-my-backend-to-only-azure-front-door).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’avez plus besoin des ressources utilisées dans ce tutoriel, utilisez la commande [az group delete](/cli/azure/group?view=azure-cli-latest#az-group-delete&preserve-view=true) pour supprimer le groupe de ressources, Front Door et la stratégie WAF :

```azurecli-interactive
  az group delete \
    --name <>
```
`--name` : Nom du groupe de ressources de pour toutes les ressources utilisées dans ce tutoriel.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment résoudre les problèmes de votre instance Front Door, consultez les guides de résolution des problèmes :

> [!div class="nextstepaction"]
> [Résolution des problèmes de routage courants](front-door-troubleshoot-routing.md)