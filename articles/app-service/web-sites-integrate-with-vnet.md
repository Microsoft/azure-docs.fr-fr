---
title: Intégrer une application au réseau virtuel Microsoft Azure
description: Intégrez une application à Azure App Service avec des réseaux virtuels Azure.
author: ccompy
ms.assetid: 90bc6ec6-133d-4d87-a867-fcf77da75f5a
ms.topic: article
ms.date: 08/05/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 764e0262c8a26511c55740aa1797b5ec9b59cc8e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999432"
---
# <a name="integrate-your-app-with-an-azure-virtual-network"></a>Intégrer votre application à un réseau virtuel Azure

Cet article décrit la fonctionnalité d’intégration au réseau virtuel d’Azure App Service et explique comment la configurer avec des applications dans [Azure App Service](./overview.md). Les [réseaux virtuels Azure][VNETOverview] vous permettent de placer un grand nombre de vos ressources Azure dans un réseau routable non-Internet. La fonctionnalité d’intégration au réseau virtuel permet à vos applications d’accéder à des ressources dans ou via un réseau virtuel. Elle n’autorise pas l’accès privé à vos applications.

Azure App Service propose deux variantes de la fonctionnalité d’intégration au réseau virtuel :

[!INCLUDE [app-service-web-vnet-types](../../includes/app-service-web-vnet-types.md)]

## <a name="enable-vnet-integration"></a>Activer une intégration au réseau virtuel

1. Accédez à l’interface utilisateur **Réseau** dans le portail App Service. Sous **Intégration de réseau virtuel**, sélectionnez **Cliquez ici pour configurer**.

1. Sélectionnez **Ajouter un réseau virtuel**.

   ![Sélectionner l’intégration au réseau virtuel][1]

1. La liste déroulante contient tous les réseaux virtuels Azure Resource Manager de votre abonnement dans la même région. Sous cette liste se trouve une liste des réseaux virtuels Azure Resource Manager dans toutes les autres régions. Sélectionnez le réseau virtuel avec lequel vous souhaitez effectuer l’intégration.

   ![Sélectionner le réseau virtuel][2]

   * Si le réseau virtuel se trouve dans la même région, créez un sous-réseau ou sélectionnez un sous-réseau préexistant vide.
   * Pour sélectionner un réseau virtuel dans une autre région, vous devez disposer d’une passerelle de réseau virtuel provisionnée dont l’option Point à site est activée.
   * Pour une intégration à un réseau virtuel classique, au lieu de sélectionner la liste déroulante **Réseau virtuel**, sélectionnez **Cliquez ici pour vous connecter à un réseau virtuel classique**. Sélectionnez le réseau virtuel classique que vous souhaitez utiliser. Le réseau virtuel cible doit déjà être doté d’une passerelle de réseau virtuel provisionnée dont l’option Point à site est activée.

    ![Sélectionner un réseau virtuel classique][3]

Lors de l’intégration, votre application est redémarrée. Une fois l’intégration terminée, vous verrez des informations sur le réseau virtuel auquel vous êtes intégré.

## <a name="regional-vnet-integration"></a>Intégration au réseau virtuel régional

[!INCLUDE [app-service-web-vnet-types](../../includes/app-service-web-vnet-regional.md)]

### <a name="how-regional-vnet-integration-works"></a>Fonctionnement de l’intégration au réseau virtuel régional

Les applications contenues dans App Service sont hébergées dans des rôles de travail. Les plans tarifaires de base et supérieurs sont des plans d’hébergement dédiés dans lesquels aucune autre charge de travail de client ne s’exécute sur les mêmes Workers. L’intégration au réseau virtuel régional opère en montant des interfaces virtuelles avec des adresses du sous-réseau délégué. Comme l’adresse de départ se trouve dans votre réseau virtuel, elle peut accéder à la plupart des éléments contenus dans votre réseau virtuel ou accessibles par celui-ci, comme le ferait une machine virtuelle dans votre réseau virtuel. L’implémentation de la mise en réseau est différente de l’exécution d’une machine virtuelle dans votre réseau virtuel. C’est pourquoi certaines fonctionnalités de mise en réseau ne sont pas encore disponibles.

![Fonctionnement de l’intégration au réseau virtuel régional][5]

Quand l’intégration au réseau virtuel régional est activée, votre application continue de passer des appels sortants vers Internet en empruntant les mêmes canaux que d’habitude. Les adresses sortantes figurant sur le portail des propriétés de l’application sont toujours les adresses qu’utilise votre application. Ce qui change pour votre application, c’est que les appels aux services sécurisés de point de terminaison de service ou aux adresses RFC 1918 accèdent à votre réseau virtuel. Si WEBSITE_VNET_ROUTE_ALL a la valeur 1, tout le trafic sortant peut être envoyé dans votre réseau virtuel.

> [!NOTE]
> `WEBSITE_VNET_ROUTE_ALL` n’est actuellement pas pris en charge dans les conteneurs Windows.
> 

La fonctionnalité ne prend en charge qu’une seule interface virtuelle par Worker. Une interface virtuelle par Worker signifie une intégration au réseau virtuel régional par plan App Service. Toutes les applications d’un même plan App Service peuvent utiliser la même intégration de réseau virtuel. Si vous avez besoin d’une application pour vous connecter à un autre réseau virtuel, vous devez créer un autre plan App Service. L’interface virtuelle utilisée n’est pas une ressource à laquelle les clients peuvent accéder directement.

Compte tenu du mode de fonctionnement de cette technologie, le trafic utilisé avec l’intégration au réseau virtuel n’apparaît pas dans les journaux de flux du groupe de sécurité réseau (NSG) ou d’Azure Network Watcher.

## <a name="gateway-required-vnet-integration"></a>Intégration au réseau virtuel avec passerelle obligatoire

L’intégration au réseau virtuel avec passerelle obligatoire prend en charge la connexion à un réseau virtuel d’une autre région ou à un réseau virtuel classique. Intégration au réseau virtuel avec passerelle obligatoire :

* Permet à une application de se connecter à un seul réseau virtuel à la fois.
* Permet d’intégrer jusqu’à cinq réseaux virtuels dans un plan App Service.
* Permet à un même réseau virtuel d’être utilisé par plusieurs applications dans le cadre d’un plan App Service sans impact sur le nombre total d’applications pouvant être utilisées par un plan App Service. Si vous avez six applications qui utilisent le même réseau virtuel dans le cadre d’un plan App Service, cela compte pour un réseau virtuel utilisé.
* Prend en charge un contrat SLA de 99,9 % du fait du contrat SLA de la passerelle.
* Permet à vos applications d’utiliser le DNS avec lequel le réseau virtuel est configuré.
* Nécessite une passerelle de réseau virtuel basée sur le routage et configurée avec un VPN de point à site SSTP avant de pouvoir être connecté à une application.

Vous ne pouvez pas utiliser l’intégration au réseau virtuel avec passerelle obligatoire :

* Avec un réseau virtuel connecté au moyen d’Azure ExpressRoute.
* À partir d’une application Linux.
* À partir d’un [conteneur Windows](quickstart-custom-container.md).
* Pour accéder à des ressources sécurisées de points de terminaison de service.
* Avec une passerelle de coexistence qui prend en charge à la fois les connexions ExpressRoute et les VPN de point à site ou site à site.

### <a name="set-up-a-gateway-in-your-azure-virtual-network"></a>Configurer une passerelle dans votre réseau virtuel Azure ###

Pour créer une passerelle :

1. [Créez un sous-réseau de passerelle][creategatewaysubnet] dans votre réseau virtuel.

1. [Créez la passerelle VPN][creategateway]. Sélectionnez un type de VPN basé sur les routes.

1. [Définissez les adresses de point à site][setp2saddresses]. Si la passerelle n’est pas dans la référence (SKU) de base, c’est que IKEV2 doit être désactivé dans la configuration de point à site et que SSTP doit être sélectionné. L’espace d’adressage de point à site doit se situer dans les blocs d’adresses RFC 1918, à savoir 10.0.0.0/8, 172.16.0.0/12 et 192.168.0.0/16.

Si vous créez la passerelle pour l’utiliser avec l’intégration au réseau virtuel App Service, il n’est pas nécessaire de charger un certificat. La création de la passerelle peut prendre 30 minutes. Vous ne serez pas en mesure d’intégrer votre application à votre réseau virtuel tant que la passerelle n’est pas provisionnée.

### <a name="how-gateway-required-vnet-integration-works"></a>Fonctionnement de l’intégration au réseau virtuel avec passerelle obligatoire

L’intégration au réseau virtuel avec passerelle obligatoire s’appuie sur la technologie VPN de point à site. Les VPN de point à site limitent l’accès réseau à la seule machine virtuelle qui héberge l’application. Les applications sont limitées au seul envoi du trafic vers Internet, via des connexions hybrides ou via l’intégration au réseau virtuel. Lorsque votre application est configurée avec le portail pour utiliser l’intégration au réseau virtuel avec passerelle obligatoire, une négociation complexe est managée en votre nom pour créer et attribuer des certificats sur la passerelle et du côté de l’application. Au final, les Workers utilisés pour héberger vos applications sont en mesure de se connecter directement à la passerelle de réseau virtuel, dans le réseau virtuel sélectionné.

![Fonctionnement de l’intégration au réseau virtuel avec passerelle obligatoire][6]

### <a name="access-on-premises-resources"></a>Accès aux ressources locales

Les applications peuvent accéder aux ressources locales en s’intégrant à des réseaux virtuels qui ont des connexions site à site. Si vous utilisez l’intégration au réseau virtuel avec passerelle obligatoire, mettez à jour les routes de votre passerelle VPN locale avec vos blocs d’adresses de point à site. Lors de la configuration initiale de ce VPN site à site, les scripts utilisés pour le configurer doivent définir les routes correctement. Si vous ajoutez des adresses de point à site après avoir créé votre VPN de site à site, vous devez mettre à jour les routes manuellement. La procédure détaillée dépend de la passerelle et n’est pas décrite ici. Vous ne pouvez pas configurer BGP avec une connexion VPN de site à site.

Aucune configuration supplémentaire n’est nécessaire pour permettre à la fonctionnalité d’intégration au réseau virtuel régional d’accéder à vos ressources locales via votre réseau virtuel. Vous devez simplement connecter votre réseau virtuel à vos ressources locales à l’aide d’ExpressRoute ou d’un VPN de site à site.

> [!NOTE]
> La fonctionnalité d’intégration au réseau virtuel avec passerelle obligatoire n’intègre pas une application à un réseau virtuel doté d’une passerelle ExpressRoute. Même si la passerelle ExpressRoute est configurée en [mode de coexistence][VPNERCoex], l’intégration au réseau virtuel ne fonctionne pas. Si vous avez besoin d’accéder à des ressources via une connexion ExpressRoute, utilisez la fonctionnalité d’intégration au réseau virtuel régional ou un [environnement ASE (App Service Environment)][ASE], qui s’exécute dans votre réseau virtuel.
>
>

### <a name="peering"></a>Peering

Si vous utilisez le peering avec l’intégration au réseau virtuel régional, aucune configuration supplémentaire n’est nécessaire.

Si vous utilisez l’intégration au réseau virtuel avec passerelle obligatoire ainsi que le peering, vous devez configurer quelques éléments supplémentaires. Pour configurer le peering afin qu’il fonctionne avec votre application :

1. Ajoutez une connexion de peering sur le réseau virtuel auquel votre application se connecte. Lors de l’ajout de la connexion de peering, activez **Autoriser l’accès au réseau virtuel** et sélectionnez **Autoriser le trafic transféré** et **Autoriser le transit par passerelle**.
1. Ajoutez une connexion de peering sur le réseau virtuel à appairer au réseau virtuel auquel vous êtes connecté. Lors de l’ajout de la connexion de peering sur le réseau virtuel de destination, activez **Autoriser l’accès au réseau virtuel** et sélectionnez **Autoriser le trafic transféré** et **Autoriser les passerelles distantes**.
1. Accédez à l’interface utilisateur **Plan App Service** > **Mise en réseau** > **Intégration du réseau virtuel** dans le portail. Sélectionnez le réseau virtuel auquel votre application se connecte. Dans la section Routage, ajoutez la plage d’adresses du réseau virtuel qui est appairé au réseau virtuel auquel votre application est connectée.

## <a name="manage-vnet-integration"></a>Gérer l’intégration au réseau virtuel

Les connexions et déconnexions avec un réseau virtuel se font au niveau de l’application. Les opérations qui peuvent affecter l’intégration au réseau virtuel entre plusieurs applications s’effectuent au niveau du plan App Service. Sur le portail, sous application > **Mise en réseau** > **Intégration de réseau virtuel**, vous pouvez obtenir des détails sur votre réseau virtuel. Vous pouvez consulter des informations similaires au niveau du plan App Service dans le portail **Plan App Service** > **Mise en réseau** > **Intégration du réseau virtuel**.

La seule opération que vous pouvez effectuer dans la vue d’application de votre instance d’intégration au réseau virtuel consiste à déconnecter votre application du réseau virtuel auquel elle est actuellement connectée. Pour déconnecter votre application d’un réseau virtuel, sélectionnez **Déconnecter**. Votre application est redémarrée quand vous vous déconnectez d’un réseau virtuel. La déconnexion ne change pas votre réseau virtuel. Le sous-réseau ou la passerelle ne sont pas supprimés. Donc, si vous souhaitez supprimer votre réseau virtuel, déconnectez d’abord votre application du réseau virtuel puis supprimez les ressources qu’il contient comme les passerelles.

L’interface utilisateur de l’intégration du réseau virtuel du plan App Service vous montre toutes les intégrations de réseau virtuel utilisées par les applications de votre plan App Service. Pour voir des détails supplémentaires sur chaque réseau virtuel, sélectionnez le réseau virtuel qui vous intéresse. Vous pouvez effectuer deux actions ici pour l’intégration au réseau virtuel avec passerelle obligatoire :

* **Synchroniser le réseau** : L’opération de synchronisation du réseau s’adresse uniquement à la fonctionnalité d’intégration au réseau virtuel dépendante de la passerelle. L’exécution d’une opération de synchronisation du réseau est l’assurance que vos certificats et informations réseau sont synchronisés. Si vous ajoutez ou changez le DNS de votre réseau virtuel, effectuez une opération de synchronisation du réseau. Cette opération redémarre toutes les applications qui utilisent ce réseau virtuel. Cette opération ne fonctionnera pas si l’application et le réseau virtuel utilisés appartiennent à différents abonnements.
* **Ajouter des routes** : L’ajout de routes achemine le trafic sortant vers votre réseau virtuel.

### <a name="gateway-required-vnet-integration-routing"></a>Routage dans une intégration au réseau virtuel avec passerelle obligatoire
Les routes définies dans votre réseau virtuel sont utilisées pour diriger le trafic dans votre réseau virtuel à partir de votre application. Pour envoyer du trafic sortant supplémentaire dans le réseau virtuel, ajoutez ces blocs d’adresses ici. Cette fonctionnalité n’opère qu’avec l’intégration au réseau virtuel avec passerelle obligatoire. Les tables de route n’ont pas d’incidence sur le trafic de votre application lors de l’utilisation de l’intégration au réseau virtuel avec passerelle obligatoire, comme c’est le cas pour l’intégration au réseau virtuel régional.

### <a name="gateway-required-vnet-integration-certificates"></a>Certificats dans une intégration au réseau virtuel avec passerelle obligatoire
Quand l’intégration au réseau virtuel avec passerelle obligatoire est activée, un échange de certificats est nécessaire pour garantir la sécurité de la connexion. En plus des certificats, la configuration DNS, les routes et d’autres éléments similaires décrivent le réseau.

Si des certificats ou des informations du réseau sont modifiés, sélectionnez **Synchroniser le réseau**. Quand vous sélectionnez **Synchroniser le réseau**, la connectivité entre votre application et votre réseau virtuel est brièvement interrompue. Votre application n’est pas redémarrée, et la perte de connectivité peut altérer le fonctionnement correct de votre site.

## <a name="pricing-details"></a>Détails de la tarification
L’utilisation de la fonctionnalité d’intégration au réseau virtuel régional ne s’accompagne d’aucuns frais supplémentaires au-delà des frais liés au niveau tarifaire du plan App Service.

Trois types de frais sont appliqués en cas d’utilisation de la fonctionnalité d’intégration au réseau virtuel avec passerelle obligatoire :

* **Frais liés au niveau tarifaire du plan App Service** : Vos applications doivent être dans un plan App Service Standard, Premium, PremiumV2 ou PremiumV3. Pour plus d’informations sur ces frais, consultez [Tarification d’App Service][ASPricing].
* **Coût de transfert des données** : Les sorties de données engendrent des coûts, même si le réseau virtuel est dans le même centre de données. Ces coûts sont décrits dans [Détails de tarification des transferts de données][DataPricing].
* **Coûts de la passerelle VPN** : La passerelle de réseau virtuel nécessaire pour la connexion VPN de point à site engendre un coût. Pour plus d’informations, consultez [Tarification Passerelle VPN][VNETPricing].

## <a name="troubleshooting"></a>Dépannage

> [!NOTE]
> L’intégration au réseau virtuel n’est pas prise en charge pour les scénarios Docker Compose dans App Service.
>

[!INCLUDE [app-service-web-vnet-troubleshooting](../../includes/app-service-web-vnet-troubleshooting.md)]

## <a name="automation"></a>Automatisation

Une prise en charge de l’interface CLI est disponible pour l’intégration au réseau virtuel régional. Pour accéder aux commandes suivantes, [installez l’interface de ligne de commande Azure][installCLI].

```azurecli
az webapp vnet-integration --help

Group
    az webapp vnet-integration : Methods that list, add, and remove virtual network
    integrations from a webapp.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    add    : Add a regional virtual network integration to a webapp.
    list   : List the virtual network integrations on a webapp.
    remove : Remove a regional virtual network integration from webapp.

az appservice vnet-integration --help

Group
    az appservice vnet-integration : A method that lists the virtual network
    integrations used in an appservice plan.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    list : List the virtual network integrations used in an appservice plan.
```

La prise en charge PowerShell de l’intégration au réseau virtuel régional est également disponible, mais vous devez créer une ressource générique avec un tableau de propriétés de resourceID du sous-réseau

```azurepowershell
# Parameters
$sitename = 'myWebApp'
$resourcegroupname = 'myRG'
$VNetname = 'myVNet'
$location = 'myRegion'
$integrationsubnetname = 'myIntegrationSubnet'
$subscriptionID = 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee'

#Property array with the SubnetID
$properties = @{
  subnetResourceId = "/subscriptions/$subscriptionID/resourceGroups/$resourcegroupname/providers/Microsoft.Network/virtualNetworks/$VNetname/subnets/$integrationsubnetname"
}

#Creation of the VNet integration
$vNetParams = @{
  ResourceName = "$sitename/VirtualNetwork"
  Location = $location
  ResourceGroupName = $resourcegroupname
  ResourceType = 'Microsoft.Web/sites/networkConfig'
  PropertyObject = $properties
}
New-AzResource @vNetParams
```


Pour l’intégration au réseau virtuel avec passerelle obligatoire, vous pouvez intégrer App Service à un réseau virtuel Azure à l’aide de PowerShell. Pour obtenir un script prêt à l’exécution, consultez [Connect an app in Azure App Service to an Azure Virtual Network](https://gallery.technet.microsoft.com/scriptcenter/Connect-an-app-in-Azure-ab7527e3).


<!--Image references-->
[1]: ./media/web-sites-integrate-with-vnet/vnetint-app.png
[2]: ./media/web-sites-integrate-with-vnet/vnetint-addvnet.png
[3]: ./media/web-sites-integrate-with-vnet/vnetint-classic.png
[5]: ./media/web-sites-integrate-with-vnet/vnetint-regionalworks.png
[6]: ./media/web-sites-integrate-with-vnet/vnetint-gwworks.png


<!--Links-->
[VNETOverview]: ../virtual-network/virtual-networks-overview.md
[AzurePortal]: https://portal.azure.com/
[ASPricing]: https://azure.microsoft.com/pricing/details/app-service/
[VNETPricing]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[DataPricing]: https://azure.microsoft.com/pricing/details/data-transfers/
[V2VNETP2S]: ../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md
[ILBASE]: environment/create-ilb-ase.md
[V2VNETPortal]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md
[VPNERCoex]: ../expressroute/expressroute-howto-coexist-resource-manager.md
[ASE]: environment/intro.md
[creategatewaysubnet]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#creategw
[creategateway]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#creategw
[setp2saddresses]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#addresspool
[VNETRouteTables]: ../virtual-network/manage-route-table.md
[installCLI]: /cli/azure/install-azure-cli?view=azure-cli-latest%2f
[privateendpoints]: networking/private-endpoint.md