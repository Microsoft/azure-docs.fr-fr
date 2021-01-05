---
title: Résoudre les problèmes relatifs à la redirection vers le scénario de l’URL App Service
titleSuffix: Azure Application Gateway
description: Cet article fournit des informations sur la façon de résoudre les problèmes de redirection lorsque vous utilisez Azure Application Gateway avec Azure App Service
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: troubleshooting
ms.date: 11/14/2019
ms.author: absha
ms.openlocfilehash: 1cc7df755198461643703cac988c8c31f2ac25db
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182884"
---
# <a name="troubleshoot-app-service-issues-in-application-gateway"></a>Résoudre les problèmes d’App Service dans Application Gateway

Découvrez comment diagnostiquer et résoudre les problèmes que vous pouvez rencontrer quand Azure App Service est utilisé comme cible de back-end avec Azure Application Gateway.

## <a name="overview"></a>Vue d’ensemble

Cet article explique comment résoudre les problèmes suivants :

* L’URL d’App Service est exposée dans le navigateur en cas de redirection.
* Le domaine du cookie ARRAffinity d’App Service est défini sur nom d’hôte d’App Service (example.azurewebsites.net) au lieu de l’hôte d’origine.

Quand une application back-end envoie une réponse de redirection, vous pouvez souhaiter rediriger le client vers une URL différente de celle spécifiée par l’application back-end. Cela peut être le cas quand un service d’application est hébergé derrière une passerelle d’application et demande au client d’effectuer une redirection vers son chemin relatif. C’est, par exemple, le cas d’une redirection de contoso.azurewebsites.net/path1 vers contoso.azurewebsites.net/path2. 

Quand le service d’application envoie une réponse de redirection, il utilise le nom d’hôte qui figure dans l’en-tête d’emplacement de sa réponse, qui est identique à celui situé dans la requête qu’il reçoit de la passerelle d’application. Par exemple, le client adresse directement la requête à contoso.azurewebsites.net/path2 au lieu de passer par la passerelle d’application contoso.com/path2. Vous ne souhaitez pas ignorer la passerelle d’application.

Ce problème peut se produire pour les raisons suivantes :

- Vous avez configuré la redirection sur votre App Service. La redirection peut être aussi simple que l’ajout d’une barre oblique à la demande.
- L’authentification Azure Active Directory provoque la redirection.

En outre, lorsque vous utilisez des App Services derrière une passerelle d’application, le nom de domaine associé à celle-ci (example.com) diffère du nom de domaine de l’App Service (par exemple, example.azurewebsites.net). La valeur de domaine pour le cookie ARRAffinity défini par l’App Service porte le nom de domaine example.azurewebsites.net, ce qui n’est pas souhaitable. Le nom d’hôte d’origine, example.com, doit être la valeur du nom de domaine dans le cookie.

## <a name="sample-configuration"></a>Exemple de configuration

- Écouteur HTTP : De base ou de multi-sites
- Pool d’adresses IP back-end : App Service
- Paramètres HTTP : **Choisir le nom d’hôte à partir de l’adresse du serveur principal** activé
- Sonde : **Choisir le nom d’hôte à partir des paramètres HTTP** activé

## <a name="cause"></a>Cause

App Service étant un service multilocataire, il utilise l’en-tête d’hôte de la requête pour router celle-ci vers le point de terminaison approprié. Le nom de domaine par défaut des App Services, *.azurewebsites.net (par exemple, contoso.azurewebsites.net), est différent du nom de domaine de la passerelle d’application (par exemple, contoso.com). 

La demande d’origine du client a le nom de domaine de la passerelle d’application, contoso.com, en tant que nom d’hôte. Vous devez configurer la passerelle d’application pour remplacer le nom d’hôte dans la demande d’origine par le nom d’hôte de l’App Service lors de du routage de la demande vers le serveur principal du service. Utilisez le commutateur **Choisir le nom d’hôte à partir de l’adresse du serveur principal** dans la configuration du paramètre HTTP de la passerelle d’application. Utilisez le commutateur **Choisir le nom d’hôte à partir des paramètres HTTP du serveur principal** dans la configuration de la sonde d’intégrité.



![La passerelle d’application change le nom d’hôte](./media/troubleshoot-app-service-redirection-app-service-url/appservice-1.png)

Quand l’App Service effectue une redirection, il utilise le nom d’hôte remplacé « contoso.azurewebsites.net » dans l’en-tête d’emplacement, au lieu du nom d’hôte d’origine « contoso.com », sauf en cas de configuration différente. Examinez les exemples suivants d’en-tête de requête et de réponse.
```
## Request headers to Application Gateway:

Request URL: http://www.contoso.com/path

Request Method: GET

Host: www.contoso.com

## Response headers:

Status Code: 301 Moved Permanently

Location: http://contoso.azurewebsites.net/path/

Server: Microsoft-IIS/10.0

Set-Cookie: ARRAffinity=b5b1b14066f35b3e4533a1974cacfbbd969bf1960b6518aa2c2e2619700e4010;Path=/;HttpOnly;Domain=contoso.azurewebsites.net

X-Powered-By: ASP.NET
```
Dans l’exemple précédent, notez que l’en-tête de réponse a un code d’état 301 pour la redirection. L’en-tête d’emplacement contient le nom d’hôte App Service au lieu du nom d’hôte d’origine `www.contoso.com`.

## <a name="solution-rewrite-the-location-header"></a>Solution : Réécrire l’en-tête d’emplacement

Définissez le nom d’hôte de l’en-tête d’emplacement sur le nom de domaine de la passerelle d’application. Pour ce faire, créez une [règle de réécriture](./rewrite-http-headers.md) avec une condition qui détermine si l’en-tête d’emplacement de la réponse contient azurewebsites.net. Elle doit aussi exécuter une action de façon à réécrire l’en-tête d’emplacement et lui attribuer le nom d’hôte de la passerelle d’application. Pour plus d’informations, voir les instructions de [réécriture de l’en-tête d’emplacement](./rewrite-http-headers.md#modify-a-redirection-url).

> [!NOTE]
> La prise en charge de la réécriture d’en-tête HTTP n’est disponible que pour les [références (SKU) Standard_v2 et WAF_v2](./application-gateway-autoscaling-zone-redundant.md) d’Application Gateway. Si vous utilisez la référence (SKU) v1, nous vous recommandons de [migrer de v1 vers v2](./migrate-v1-v2.md). Vous pouvez utiliser la réécriture et d’autres [fonctionnalités avancées](./application-gateway-autoscaling-zone-redundant.md#feature-comparison-between-v1-sku-and-v2-sku) disponibles avec la référence (SKU) v2.

## <a name="alternate-solution-use-a-custom-domain-name"></a>Autre solution : Ajouter un nom de domaine personnalisé

Si vous utilisez la référence (SKU) v1, vous ne pouvez pas réécrire l’en-tête d’emplacement. Cette fonctionnalité est disponible uniquement pour la référence (SKU) v2. Pour résoudre le problème de redirection, vous devrez transmettre le nom d’hôte reçu par la passerelle d’application à l’App Service au lieu de modifier l’hôte.

L’App Service effectue ensuite la redirection (le cas échéant) sur le même en-tête hôte d’origine qui pointe vers la passerelle d’application et pas le sien.

Vous devez posséder un domaine personnalisé et procéder comme suit :

- Enregistrez le domaine dans la liste de domaines personnalisés de l’App Service. Vous devez disposer d’un enregistrement CNAME dans votre domaine personnalisé qui pointe vers le nom de domaine complet de l’App Service. Pour plus d’informations, consultez [Mapper un nom DNS personnalisé existant à Azure App Service](../app-service/app-service-web-tutorial-custom-domain.md).

    ![Liste de domaines personnalisés d’App Service](./media/troubleshoot-app-service-redirection-app-service-url/appservice-2.png)

- Votre App Service est prêt à accepter le nom d’hôte `www.contoso.com`. Modifiez votre entrée CNAME dans le DNS afin qu’elle pointe vers le nom de domaine complet de la passerelle d’application (par exemple, `appgw.eastus.cloudapp.azure.com`).

- Assurez-vous que votre domaine `www.contoso.com` est résolu avec le nom de domaine complet de la passerelle d’application lorsque vous effectuez une requête DNS.

- Configurez votre sonde personnalisée pour désactiver **Choisir le nom d’hôte à partir des paramètres HTTP du serveur principal**. Dans le portail Azure, désactivez la case à cocher dans les paramètres de sonde. Dans PowerShell, n’utilisez pas le commutateur **-PickHostNameFromBackendHttpSettings** dans la commande **Set-AzApplicationGatewayProbeConfig**. Dans le champ du nom d’hôte de la sonde, entrez le nom de domaine complet de App Service, example.azurewebsites.net. Les demandes de sondage envoyées par la passerelle d’application transportent ce nom de domaine complet dans l’en-tête de l’hôte.

  > [!NOTE]
  > Pour l’étape suivante, assurez-vous que votre sonde personnalisée n’est pas associée aux paramètres HTTP du serveur principal. À ce stade, le commutateur **Choisir le nom d’hôte à partir de l’adresse du serveur principal** est toujours activé dans vos paramètres HTTP.

- Configurez les paramètres HTTP d’Application Gateway pour désactiver **Choisir le nom d’hôte à partir de l’adresse du serveur principal**. Dans le portail Azure, désactivez la case à cocher. Dans PowerShell, n’utilisez pas le commutateur **-PickHostNameFromBackendAddress** dans la commande **Set-AzApplicationGatewayBackendHttpSettings**.

- Associez à nouveau la sonde personnalisée aux paramètres HTTP du serveur principal, et vérifiez l’intégrité de celui-ci.

- La passerelle d’application doit maintenant transférer le même nom d’hôte (`www.contoso.com`) vers le service App Service. La redirection a lieu sur le même nom d’hôte. Examinez les exemples suivants d’en-tête de requête et de réponse.

Pour implémenter les étapes précédentes à l’aide de PowerShell pour une configuration existante, suivez l’exemple de script PowerShell ci-dessous. Notez que nous n’avons pas utilisé les commutateurs **-PickHostname** dans la configuration de la sonde et des paramètres HTTP.

```azurepowershell-interactive
$gw=Get-AzApplicationGateway -Name AppGw1 -ResourceGroupName AppGwRG
Set-AzApplicationGatewayProbeConfig -ApplicationGateway $gw -Name AppServiceProbe -Protocol Http -HostName "example.azurewebsites.net" -Path "/" -Interval 30 -Timeout 30 -UnhealthyThreshold 3
$probe=Get-AzApplicationGatewayProbeConfig -Name AppServiceProbe -ApplicationGateway $gw
Set-AzApplicationGatewayBackendHttpSettings -Name appgwhttpsettings -ApplicationGateway $gw -Port 80 -Protocol Http -CookieBasedAffinity Disabled -Probe $probe -RequestTimeout 30
Set-AzApplicationGateway -ApplicationGateway $gw
```
  ```
  ## Request headers to Application Gateway:

  Request URL: http://www.contoso.com/path

  Request Method: GET

  Host: www.contoso.com

  ## Response headers:

  Status Code: 301 Moved Permanently

  Location: http://www.contoso.com/path/

  Server: Microsoft-IIS/10.0

  Set-Cookie: ARRAffinity=b5b1b14066f35b3e4533a1974cacfbbd969bf1960b6518aa2c2e2619700e4010;Path=/;HttpOnly;Domain=www.contoso.com

  X-Powered-By: ASP.NET
  ```
  ## <a name="next-steps"></a>Étapes suivantes

Si les étapes précédentes ne vous permettent pas de résoudre le problème, ouvrez un [ticket de support](https://azure.microsoft.com/support/options/).