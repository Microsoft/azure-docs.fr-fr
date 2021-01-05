---
author: ccompy
ms.service: app-service-web
ms.topic: include
ms.date: 10/21/2020
ms.author: ccompy
ms.openlocfilehash: 57b2955f8cec059cd20d353eba31dc39ad992d50
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97506322"
---
L’utilisation de l’intégration au réseau virtuel régional permet à votre application d’accéder aux :

* Ressources d’un réseau virtuel dans la même région que votre application.
* Ressources de réseaux virtuels appairés au réseau virtuel auquel votre application est intégrée.
* Services sécurisés des points de terminaison de service.
* Ressources sur des connexions Azure ExpressRoute.
* Ressources dans le réseau virtuel auquel vous êtes intégré.
* Ressources sur des connexions appairées, notamment des connexions Azure ExpressRoute.
* Instances Private Endpoint 

Lorsque vous utilisez l’intégration au réseau virtuel avec des réseaux virtuels d’une même région, vous pouvez utiliser les fonctionnalités de même en réseau Azure suivantes :

* **Groupes de sécurité réseau (NSG)**  : Vous pouvez bloquer le trafic sortant avec un groupe de sécurité réseau placé sur votre sous-réseau d’intégration. Les règles de trafic entrant ne s’appliquent pas, car vous ne pouvez pas utiliser l’intégration au réseau virtuel pour fournir un accès entrant à votre application.
* **Tables de routage (Routes définies par l’utilisateur)**  : Vous pouvez placer une table de routage sur le sous-réseau d’intégration pour envoyer le trafic sortant où vous voulez.

Par défaut, votre application route seulement le trafic RFC1918 vers votre réseau virtuel. Si vous voulez router tout le trafic sortant vers votre réseau virtuel, appliquez le paramètre d’application WEBSITE_VNET_ROUTE_ALL à votre application. Pour configurer le paramètre d’application :

1. Accédez à l’interface utilisateur **Configuration** dans votre portail d’application. Sélectionnez **Nouveau paramètre d’application**.
1. Entrez **WEBSITE_VNET_ROUTE_ALL** dans la zone **Nom** et **1** dans la zone **Valeur**.

   ![Fournir le paramètre d’application][4]

1. Sélectionnez **OK**.
1. Sélectionnez **Enregistrer**.

> [!NOTE]
> Si vous routez tout le trafic sortant vers votre réseau virtuel, il est soumis aux groupes de sécurité réseau et aux routes définies par l’utilisateur appliqués à votre sous-réseau d’intégration. Lorsque vous acheminez tout le trafic sortant vers votre réseau virtuel, vos adresses sortantes sont toujours les adresses sortantes listées dans les propriétés de votre application, sauf si vous fournissez les routes pour envoyer le trafic ailleurs.

Il existe certaines limitations concernant l’utilisation de l’intégration au réseau virtuel avec les réseaux virtuels d’une même région :

* Vous ne pouvez pas accéder à des ressources via des connexions d’appairage mondiales.
* Cette fonctionnalité est disponible à partir de toutes les unités d’échelle App Service dans Premium v2 et Premium v3. Elle est également disponible dans Standard, mais uniquement à partir d’unités d’échelle App Service plus récentes. Si vous utilisez une unité d’échelle plus ancienne, vous ne pouvez utiliser la fonctionnalité qu’à partir d’un plan App Service Premium v2. Si vous souhaitez être certain de pouvoir utiliser la fonctionnalité dans un plan App Service Standard, créez votre application dans un plan App Service Premium v3. Ces plans ne sont pris en charge que sur les unités d’échelle les plus récentes. Vous pouvez effectuer un scale-down par la suite si vous le souhaitez.  
* Le sous-réseau d’intégration peut être utilisé par un seul plan App Service.
* La fonctionnalité ne peut pas être utilisée par des applications de plan Isolé qui se trouvent dans un environnement App Service.
* La fonctionnalité nécessite un sous-réseau inutilisé /28 ou d’une taille supérieure dans un réseau virtuel Azure Resource Manager.
* L’application et le réseau virtuel doivent être dans la même région.
* Vous ne pouvez pas supprimer un réseau virtuel avec une application intégrée. Supprimez l’intégration avant de supprimer le réseau virtuel.
* L’intégration ne peut se faire qu’avec des réseaux virtuels figurant dans le même abonnement que l’application.
* Vous ne pouvez disposer que d’une seule intégration au réseau virtuel régional par plan App Service. Plusieurs applications d’un même plan App Service peuvent utiliser le même réseau virtuel.
* Vous ne pouvez pas changer l’abonnement d’une application ou d’un plan quand une application utilise l’intégration au réseau virtuel régional.
* Votre application ne peut pas résoudre les adresses dans Azure DNS Private Zones sans modification de la configuration.

L’intégration au réseau virtuel dépend de l’utilisation d’un sous-réseau dédié.  Lorsque vous approvisionnez un sous-réseau, le sous-réseau Azure perd cinq adresses IP dès le début. Une seule adresse du sous-réseau d’intégration est utilisée pour chaque instance de plan. Si vous définissez l’échelle de votre application sur quatre instances, quatre adresses sont utilisées. Le débit de cinq adresses de la taille du sous-réseau signifie que le nombre maximal d’adresses disponibles par bloc CIDR est le suivant :

- /28 a 11 adresses
- /27 a 27 adresses
- /26 a 59 adresses

Si vous effectuez un scale-up ou un scale-down en taille, vous devez doubler votre besoin en adresses pendant une courte période de temps. Les limites de taille signifient que le nombre réel d’instances prises en charge par taille de sous-réseau sont, si votre sous-réseau est un :

- /28, votre échelle horizontale maximale est de 5 instances
- /27, votre échelle horizontale maximale est de 13 instances
- /26, votre échelle horizontale maximale est de 29 instances

Les limites indiquées sur l’échelle horizontale maximale partent du principe que vous devez effectuer un scale-up ou un scale-down à un moment donné. 

Comme la taille du sous-réseau ne peut pas être modifiée après l’affectation, utilisez un sous-réseau suffisamment grand pour s’adapter à l’échelle que votre application est susceptible d’atteindre. Pour éviter tout problème de capacité du sous-réseau, un sous-réseau /26 avec 64 adresses est la taille recommandée.  

Si vous voulez que vos applications d’un autre plan atteignent un réseau virtuel auquel sont déjà connectées des applications d’un autre plan, sélectionnez un sous-réseau différent de celui utilisé par l’intégration au réseau virtuel préexistante.

La fonctionnalité est entièrement prise en charge pour les applications Windows et Linux, notamment les [conteneurs personnalisés](../articles/app-service/quickstart-custom-container.md). Tous les comportements sont identiques entre les applications Windows et les applications Linux.

### <a name="service-endpoints"></a>Points de terminaison de service

L’intégration au réseau virtuel régional vous permet d’utiliser des points de terminaison de service. Pour utiliser des points de terminaison de service avec votre application, servez-vous de l’intégration au réseau virtuel régional pour vous connecter à un réseau virtuel sélectionné, puis configurez des points de terminaison de service avec le service de destination sur le sous-réseau que vous avez utilisé pour l’intégration. Si vous souhaitez ensuite accéder à un service via des points de terminaison de service :

1. configurez l’intégration au réseau virtuel régional à votre application web
1. accédez au service de destination et configurez des points de terminaison de service sur le sous-réseau utilisé pour l’intégration

### <a name="network-security-groups"></a>Groupes de sécurité réseau

Vous pouvez utiliser des groupes de sécurité réseau pour bloquer le trafic entrant et sortant vers des ressources d’un réseau virtuel. Une application utilisant l’intégration au réseau virtuel régional peut utiliser un [groupe de sécurité réseau][VNETnsg] pour bloquer le trafic sortant vers des ressources dans votre réseau virtuel ou sur Internet. Pour bloquer le trafic vers des adresses publiques, le paramètre d’application WEBSITE_VNET_ROUTE_ALL doit être défini sur 1. Les règles de trafic entrant dans un groupe de sécurité réseau ne s’appliquent pas à votre application, car l’intégration au réseau virtuel n’a d’incidence que sur le trafic sortant depuis votre application.

Pour contrôler le trafic entrant vers votre application, utilisez la fonctionnalité Restrictions d’accès. Un groupe de sécurité réseau appliqué à votre sous-réseau d’intégration est actif quelles que soient les routes appliquées à votre sous-réseau d’intégration. Si WEBSITE_VNET_ROUTE_ALL est défini sur 1 et que vous n’avez aucune route affectant le trafic des adresses publiques sur votre sous-réseau d’intégration, l’ensemble du trafic sortant est toujours soumis aux groupes de sécurité réseau affectés à votre sous-réseau d’intégration. Si WEBSITE_VNET_ROUTE_ALL n’est pas défini, les groupes de sécurité réseau s’appliquent seulement au trafic RFC1918.

### <a name="routes"></a>Itinéraires

Vous pouvez utiliser des tables de routage pour router le trafic sortant depuis votre application vers où vous voulez. Par défaut, les tables de routage affectent seulement votre trafic de destination RFC1918. Si définissez WEBSITE_VNET_ROUTE_ALL sur 1, tous vos appels sortants seront affectés. Les routes définies sur votre sous-réseau d’intégration n’affectent pas les réponses aux requêtes d’application entrantes. Les destinations courantes peuvent inclure des pare-feu ou des passerelles.

Si vous souhaitez router tout le trafic sortant localement, vous pouvez utiliser une table de route pour envoyer l’intégralité du trafic sortant vers votre passerelle ExpressRoute. Si vous choisissez de router le trafic vers une passerelle, veillez à définir des routes dans le réseau externe pour renvoyer des réponses.

Les routes BGP (Border Gateway Protocol) affectent également le trafic de votre application. Si vous avez des routes BGP provenant par exemple d’une passerelle ExpressRoute, le trafic sortant de votre application sera affecté. Par défaut, les routes BGP affectent seulement votre trafic de destination RFC1918. Si WEBSITE_VNET_ROUTE_ALL est défini sur 1, tout le trafic sortant peut être affecté par vos routes BGP.

### <a name="azure-dns-private-zones"></a>Azure DNS Private Zones 

Une fois que votre application est intégrée à votre réseau virtuel, elle utilise le même serveur DNS que celui avec lequel votre réseau virtuel est configuré. Par défaut, votre application ne fonctionnera pas avec Azure DNS Private Zones. Pour qu’elle fonctionne avec Azure DNS Private Zones, vous devez ajouter les paramètres d’application suivants :


1. WEBSITE_DNS_SERVER avec la valeur 168.63.129.16
1. WEBSITE_VNET_ROUTE_ALL avec la valeur 1


Ces paramètres envoient l’ensemble de vos appels sortants de votre application vers votre réseau virtuel, en plus de permettre à votre application d’utiliser Azure DNS Private Zones.   Ces paramètres vont envoyer tous les appels sortants de votre application à votre réseau virtuel. En outre, cela permet à l’application d’utiliser Azure DNS en interrogeant la zone de DNS privé au niveau du Worker. Cette fonctionnalité doit être utilisée lorsqu’une application en cours d’exécution accède à une zone de DNS privé.

> [!NOTE]
>La tentative d’ajout d’un domaine personnalisé à une application web à l’aide de la zone de DNS privé n’est pas possible avec l’intégration au réseau virtuel. La validation du domaine personnalisé est effectuée au niveau du contrôleur, et non au niveau du Worker, ce qui empêche l’affichage des enregistrements DNS. Pour utiliser un domaine personnalisé à partir d’une zone de DNS privé, la validation doit être ignorée à l’aide d’Application Gateway ou d’un environnement App Service Environment ILB.

### <a name="private-endpoints"></a>Instances Private Endpoint

Si vous souhaitez effectuer des appels vers des [points de terminaison privés][privateendpoints], vous devez vous assurer que vos recherches DNS seront résolues sur le point de terminaison privé. Pour vous assurer que les recherches DNS à partir de votre application pointeront vers vos points de terminaison privés, vous pouvez :

* l’intégrer à Azure DNS Private Zones. Si votre réseau virtuel n’a pas de serveur DNS personnalisé, cela sera automatique.
* gérer le point de terminaison privé dans le serveur DNS utilisé par votre application. Pour ce faire, vous devez connaître l’adresse du point de terminaison privé, puis pointer le point de terminaison que vous essayez d’atteindre vers cette adresse avec un enregistrement A.
* configurer votre propre serveur DNS pour le transfert vers Azure DNS Private Zones.

<!--Image references-->
[4]: ../includes/media/web-sites-integrate-with-vnet/vnetint-appsetting.png

<!--Links-->
[VNETnsg]: /azure/virtual-network/security-overview/
[privateendpoints]: ../articles/app-service/networking/private-endpoint.md
