---
title: Connexion privée à une application web Azure avec un point de terminaison privé
description: Connexion privée à une application web avec Azure Private Endpoint
author: ericgre
ms.assetid: 2dceac28-1ba6-4904-a15d-9e91d5ee162c
ms.topic: article
ms.date: 10/09/2020
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.custom: fasttrack-edit, references_regions
ms.openlocfilehash: bebc7dcbc18a25b0d6d0761a8ca3ac476e83e581
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183037"
---
# <a name="using-private-endpoints-for-azure-web-app"></a>Utilisation de points de terminaison privés pour une application web Azure

> [!IMPORTANT]
> Le point de terminaison privé est disponible pour les applications web Windows et Linux, conteneurisées ou non, hébergées sur les plans App Service suivants : **Isolé**, **PremiumV2**, **PremiumV3**, **Functions Premium** (parfois appelé plan Premium élastique). 

Vous pouvez utiliser un point de terminaison privé pour votre application web Azure afin de permettre aux clients situés dans votre réseau privé d’accéder de façon sécurisée à l’application via une liaison privée. Le point de terminaison privé utilise une adresse IP de l’espace d’adressage de votre réseau virtuel Azure. Le trafic entre un client de votre réseau privé et l’application web traverse le réseau virtuel et une liaison privée sur le réseau principal de Microsoft, ce qui élimine son exposition à l’Internet public.

L’utilisation d’un point de terminaison privé pour votre application web vous permet d’effectuer les opérations suivantes :

- Sécuriser votre application web en configurant le point de terminaison privé, ce qui évite l’exposition publique.
- Vous connecter de manière sécurisée à l’application web à partir de réseaux locaux qui se connectent au réseau virtuel à l’aide d’un VPN ou d’un peering privé ExpressRoute
- Évitez tout exfiltration de données à partir de votre réseau virtuel. 

Si vous avez simplement besoin d’une connexion sécurisée entre votre réseau virtuel et votre application web, un point de terminaison de service est la solution la plus simple. Si vous devez également accéder à l’application web à partir d’un emplacement local via une passerelle Azure, un réseau virtuel appairé au niveau régional ou un réseau virtuel appairé à l’échelle mondiale, le point de terminaison privé est la solution qu’il vous faut.  

Pour plus d’informations, consultez [Points de terminaison de service][serviceendpoint].

## <a name="conceptual-overview"></a>Vue d'ensemble conceptuelle

Un point de terminaison privé est l’interface réseau (NIC) spéciale de votre application web Azure qui se trouve dans un sous-réseau de votre réseau virtuel.
Lorsque vous créez un point de terminaison privé pour votre application web, celui-ci fournit une connectivité sécurisée entre les clients de votre réseau privé et votre application web. Une adresse IP est attribuée au point de terminaison privé à partir de la plage d’adresses IP de votre réseau virtuel.
La connexion entre le point de terminaison privé et l’application Web utilise une [liaison privée][privatelink] sécurisée. Le point de terminaison privé est utilisé uniquement pour les flux entrants de votre application web. Les flux sortants n’utilisent pas ce point de terminaison privé, mais vous pouvez injecter des flux sortants de votre réseau dans un sous-réseau différent via la [fonctionnalité d’intégration au réseau virtuel][vnetintegrationfeature].

Le sous-réseau dans lequel vous connectez le point de terminaison privé peut comporter d’autres ressources. Vous n’avez pas besoin d’un sous-réseau vide dédié.
Vous pouvez également déployer un point de terminaison privé dans une région différente de celle de l’application web. 

> [!Note]
>La fonctionnalité d’intégration au réseau virtuel ne peut pas utiliser le même sous-réseau que le point de terminaison privé. Il s’agit d’une limitation de la fonctionnalité d’intégration au réseau virtuel.

Du point de vue de la sécurité :

- Lorsque vous activez des points de terminaison privés sur votre application web, vous empêchez tout accès public.
- Vous pouvez activer plusieurs points de terminaison privés dans d’autres réseaux virtuels et sous-réseaux, y compris dans des réseaux virtuels d’autres régions.
- L’adresse IP de l’interface réseau du point de terminaison privé doit être dynamique. Toutefois, elle ne changera que si vous supprimez le point de terminaison privé.
- L’interface réseau du point de terminaison privé ne peut pas être associée à un groupe de sécurité réseau.
- Le sous-réseau qui héberge le point de terminaison privé peut être associé à un groupe de sécurité réseau. Toutefois, vous devez désactiver l’application des stratégies réseau pour le point de terminaison privé, comme décrit dans [Désactiver les stratégies réseau pour les points de terminaison privés][disablesecuritype]. Par conséquent, vous ne pouvez pas filtrer par groupe de sécurité réseau l’accès à votre point de terminaison privé.
- Lorsque vous activez un point de terminaison privé sur votre application web, la configuration des [restrictions d’accès][accessrestrictions] de l’application web n’est pas évaluée.
- Vous pouvez éliminer le risque d’exfiltration de données à partir du réseau virtuel en supprimant toutes les règles de groupe de sécurité réseau dans lesquelles la destination est étiquetée Internet ou Services Azure. Lorsque vous déployez un point de terminaison privé pour une application web, vous pouvez uniquement atteindre cette application web spécifique via le point de terminaison privé. Si vous avez une autre application web, vous devez déployer un autre point de terminaison privé dédié pour celle-ci.

Les journaux HTTP web de votre application web contiennent l’adresse IP source du client. Cette fonctionnalité est implémentée à l’aide du protocole de proxy TCP, en transférant la propriété IP du client à l’application web. Pour plus d’informations, consultez [Obtention d’informations de connexion à l’aide du proxy TCP v2][tcpproxy].


  > [!div class="mx-imgBorder"]
  > ![Vue d’ensemble des points de terminaison privés de l’application web](media/private-endpoint/global-schema-web-app.png)

## <a name="dns"></a>DNS

Si vous utilisez un point de terminaison privé pour votre application web, l’URL demandée doit correspondre au nom de cette application (par défaut, mywebappname.azurewebsites.net).

Par défaut, si vous n’utilisez pas de point de terminaison privé, le nom public de votre application web est un nom canonique référençant le cluster.
Voici des exemples de résolution des noms :

|Nom |Type |Valeur |
|-----|-----|------|
|mywebapp.azurewebsites.net|CNAME|clustername.azurewebsites.windows.net|
|clustername.azurewebsites.windows.net|CNAME|cloudservicename.cloudapp.net|
|cloudservicename.cloudapp.net|Un|40.122.110.154| 


Lorsque vous déployez un point de terminaison privé, nous mettons à jour l’entrée DNS pour qu’elle pointe vers le nom canonique mywebapp.privatelink.azurewebsites.net.
Voici des exemples de résolution des noms :

|Nom |Type |Valeur |Remarque |
|-----|-----|------|-------|
|mywebapp.azurewebsites.net|CNAME|mywebapp.privatelink.azurewebsites.net|
|mywebapp.privatelink.azurewebsites.net|CNAME|clustername.azurewebsites.windows.net|
|clustername.azurewebsites.windows.net|CNAME|cloudservicename.cloudapp.net|
|cloudservicename.cloudapp.net|Un|40.122.110.154|<--Cette adresse IP publique n’est pas votre point de terminaison privé ; vous recevrez une erreur 403|

Vous devez configurer un serveur DNS privé ou une zone privée Azure DNS. Pour les tests, vous pouvez modifier l’entrée de l’hôte de votre machine de test.
La zone DNS à créer est celle-ci : **privatelink.azurewebsites.net**. Enregistrez votre application web avec un enregistrement A et l’adresse IP du point de terminaison privé.
Voici des exemples de résolution des noms :

|Nom |Type |Valeur |Remarque |
|-----|-----|------|-------|
|mywebapp.azurewebsites.net|CNAME|mywebapp.privatelink.azurewebsites.net|<--Azure crée cette entrée dans le DNS public Azure pour faire pointer le service d’application vers le lien privé, que nous gérons|
|mywebapp.privatelink.azurewebsites.net|Un|10.10.10.8|<--Vous définissez cette entrée dans votre système DNS pour qu’elle pointe vers l’adresse IP de votre point de terminaison privé|

Après avoir terminé cette configuration DNS, vous pouvez accéder à votre application web en privé avec le nom par défaut mywebappname.azurewebsites.net. Vous devez utiliser ce nom, car le certificat par défaut est émis pour *.azurewebsites.net.


Si vous devez utiliser un nom DNS personnalisé, vous devez ajouter celui-ci dans votre application web. Le nom personnalisé doit être validé comme n’importe quel nom personnalisé, à l’aide de la résolution DNS publique. Pour plus d’informations, consultez la rubrique relative à la [validation DNS personnalisée][dnsvalidation].

Pour la console Kudu ou l’API REST Kudu (déploiement avec les agents autohébergés Azure DevOps, par exemple), vous devez créer deux enregistrements dans votre zone privée Azure DNS ou votre serveur DNS personnalisé. 

| Nom | Type | Valeur |
|-----|-----|-----|
| mywebapp.privatelink.azurewebsites.net | Un | PrivateEndpointIP | 
| mywebapp.scm.privatelink.azurewebsites.net | Un | PrivateEndpointIP | 



## <a name="pricing"></a>Tarifs

Pour plus d’informations sur les tarifs, consultez [Tarification Liaison privée Azure][pricing].

## <a name="limitations"></a>Limites

Lorsque vous utilisez Azure Functions dans le cadre d’un plan Elastic Premium avec un point de terminaison privé, pour exploiter ou exécuter la fonction dans le portail web d’Azure, vous devez disposer d’un accès direct au réseau. Sinon, vous recevrez une erreur HTTP 403. En d’autres termes, votre navigateur doit être en mesure d’atteindre le point de terminaison privé pour exécuter la fonction à partir du portail web d’Azure. 

Vous pouvez connecter jusqu’à 100 points de terminaison privés à une application web particulière.

Les emplacements ne peuvent pas utiliser de point de terminaison privé.

La fonctionnalité de débogage à distance n’est pas disponible lorsque le point de terminaison privé est activé pour l’application web. Nous vous recommandons de déployer le code sur un emplacement et de le déboguer à distance à cet endroit.

Nous améliorons régulièrement la fonctionnalité de liaison privée et le point de terminaison privé. Pour obtenir des informations à jour sur les limitations, voir [cet article][pllimitations].

## <a name="next-steps"></a>Étapes suivantes

- Pour déployer un point de terminaison privé pour votre application web via le portail, consultez [Se connecter en privé à une application web avec le portail][howtoguide1]
- Pour déployer un point de terminaison privé pour votre application web Azure CLI, consultez [Se connecter en privé à une application web avec Azure CLI][howtoguide2]
- Pour déployer un point de terminaison privé pour votre application web PowerShell, consultez [Se connecter en privé à une application web avec PowerShell][howtoguide3]
- Pour déployer un point de terminaison privé pour votre application web à l’aide d’un modèle Azure, consultez [Se connecter en privé à une application web avec un modèle Azure][howtoguide4]
- Pour obtenir un exemple de bout en bout montrant comment connecter une application Web frontale à une application Web de backend sécurisée avec une injection de réseau virtuel et un point de terminaison privé avec un modèle ARM, consultez ce [démarrage rapide][howtoguide5]
- Pour obtenir un exemple de bout en bout montrant comment connecter une application Web frontale à une application Web de backend sécurisée avec une injection de réseau virtuel et un point de terminaison privé avec terraform, consultez cet [exemple][howtoguide6]


<!--Links-->
[serviceendpoint]: ../../virtual-network/virtual-network-service-endpoints-overview.md
[privatelink]: ../../private-link/private-link-overview.md
[vnetintegrationfeature]: ../web-sites-integrate-with-vnet.md
[disablesecuritype]: ../../private-link/disable-private-endpoint-network-policy.md
[accessrestrictions]: ../app-service-ip-restrictions.md
[tcpproxy]: ../../private-link/private-link-service-overview.md#getting-connection-information-using-tcp-proxy-v2
[dnsvalidation]: ../app-service-web-tutorial-custom-domain.md
[pllimitations]: ../../private-link/private-endpoint-overview.md#limitations
[pricing]: https://azure.microsoft.com/pricing/details/private-link/
[howtoguide1]: ../../private-link/tutorial-private-endpoint-webapp-portal.md
[howtoguide2]: ../scripts/cli-deploy-privateendpoint.md
[howtoguide3]: ../scripts/powershell-deploy-private-endpoint.md
[howtoguide4]: ../scripts/template-deploy-private-endpoint.md
[howtoguide5]: https://github.com/Azure/azure-quickstart-templates/tree/master/101-webapp-privateendpoint-vnet-injection
[howtoguide6]: ../scripts/terraform-secure-backend-frontend.md