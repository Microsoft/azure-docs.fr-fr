---
title: Serveurs principaux multitenants
titleSuffix: Azure Application Gateway
description: Cette page présente la prise en charge des serveurs principaux multilocataires par Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 06/09/2020
ms.author: victorh
ms.openlocfilehash: fe6ea6f348d796962141bd39ff858d891a29a2f6
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397686"
---
# <a name="application-gateway-support-for-multi-tenant-back-ends-such-as-app-service"></a>Prise en charge d’Application Gateway pour les back-ends multilocataires tels qu’App Service

Dans les conceptions d’architecture mutualisées des serveurs web, plusieurs sites web sont en cours d’exécution sur la même instance de serveur web. Les noms d’hôtes sont utilisés pour faire la distinction entre les différentes applications hébergées. Par défaut, la passerelle d’application ne modifie pas l’en-tête d’hôte HTTP entrant à partir du client et envoie l’en-tête non modifié au serveur principal. Cela fonctionne bien pour les membres du pool principal, telles que les cartes d’interface réseau, les groupes de machines virtuelles identique, les adresses IP publiques, les adresses IP internes et les noms de domaine complets, dans la mesure où ils ne reposent pas sur un en-tête d’hôte spécifique ou une extension SNI pour effectuer la résolution vers le point de terminaison approprié. Toutefois, il existe de nombreux services comme Azure Web Apps et Gestion des API Azure qui sont multilocataires par nature et s’appuient sur un en-tête d’hôte spécifique ou une extension SNI pour effectuer la résolution vers le point de terminaison approprié. En règle générale, le nom DNS de l’application, qui à son tour est le nom DNS associé à la passerelle d’application, est différent du nom de domaine du service principal. Par conséquent, l’en-tête d’hôte de la requête d’origine reçu par la passerelle d’application n’est pas le même que le nom d’hôte du service principal. Pour cette raison, sauf si l’en-tête d’hôte de la requête provenant de la passerelle d’application vers le serveur principal est modifié pour porter le nom d’hôte du service principal, les back-ends multilocataires ne sont pas en mesure d’effectuer la résolution. de la requête vers le point de terminaison approprié. 

La passerelle Application Gateway fournit une fonctionnalité qui permet aux utilisateurs de remplacer l’en-tête d’hôte HTTP dans la demande en fonction du nom d’hôte du back-end. Cette fonctionnalité permet la prise en charge des back-ends multilocataires tels que les applications web Azure App Service et la gestion des API. Elle est disponible pour les références SKU standard v1 et v2, et WAF. 

![Remplacement d’hôte](./media/application-gateway-web-app-overview/host-override.png)

> [!NOTE]
> Cela n’est pas applicable à App Service Environment (ASE) dans la mesure où cet environnement est une ressource dédiée, à la différence d’Azure App service, qui est une ressource multilocataires.

## <a name="override-host-header-in-the-request"></a>Remplacer l’en-tête d’hôte dans la requête

La possibilité de spécifier un remplacement d’hôte est définie au niveau des [paramètres HTTP](./configuration-overview.md#http-settings) et peut être appliquée à n’importe quel pool de serveurs principaux lors de la création de règles. Les serveurs principaux multilocataires prennent en charge les deux méthodes de remplacement d’en-tête d’hôte et l’extension SNI suivantes :

- La possibilité de définir le nom d’hôte sur une valeur fixe entrée de manière explicite dans les paramètres HTTP. Cette fonctionnalité garantit que l’en-tête d’hôte est remplacé par cette valeur pour tout le trafic en direction du pool de serveurs principaux où les paramètres HTTP spécifiques sont appliqués. Lorsque vous utilisez le protocole TLS de bout en bout, ce nom d'hôte remplacé est utilisé dans l'extension SNI. Cette fonctionnalité permet de mettre en œuvre des scénarios dans lesquels une batterie de pools de serveurs principaux attend un en-tête d’hôte différent de l’en-tête d’hôte client entrant.

- La possibilité de dériver le nom d’hôte de l’adresse IP ou du nom de domaine complet des membres du pool de serveurs principaux. Les paramètres HTTP fournissent également une option pour choisir de manière dynamique le nom d’hôte à partir d’un nom de domaine complet de membre du pool de serveurs principaux si configuré avec l’option permettant de dériver le nom d’hôte d’un membre du pool de serveurs principaux. Lorsque vous utilisez le protocole TLS de bout en bout, ce nom d'hôte est dérivé du nom de domaine complet et utilisé dans l'extension SNI. Cette fonctionnalité permet de mettre en œuvre des scénarios dans lesquels un pool de serveurs principaux peut avoir deux ou plusieurs services PaaS multilocataires tels que des applications web Azure et l’en-tête de l’hôte de la demande vers chaque membre contient le nom d’hôte dérivé de son nom de domaine complet. Pour implémenter ce scénario, nous utilisons un commutateur dans les paramètres HTTP appelé [Choisir un nom d’hôte à partir d’une adresse backend](./configuration-http-settings.md#pick-host-name-from-back-end-address) qui remplace dynamiquement l’en-tête d’hôte de la requête d’origine par celui mentionné dans le pool principal.  Par exemple, si votre nom de domaine complet du pool principal contient « contoso11.azurewebsites.net » et « contoso22.azurewebsites.net », l’en-tête d’hôte de la requête d’origine qui est contoso.com est remplacé par contoso11.azurewebsites.net ou contoso22.azurewebsites.net lorsque la requête est envoyée au serveur principal approprié. 

  ![scénario d’application web](./media/application-gateway-web-app-overview/scenario.png)

Avec cette fonctionnalité, les clients spécifient les options dans les paramètres HTTP et les sondes personnalisées pour la configuration appropriée. Ce paramètre est ensuite lié à un écouteur et à un pool de serveurs principaux à l’aide d’une règle.

## <a name="special-considerations"></a>Considérations spéciales

### <a name="tls-termination-and-end-to-end-tls-with-multi-tenant-services"></a>Arrêt TLS et protocole TLS de bout en bout avec les services multi-locataires

L'arrêt TLS et le chiffrement TLS de bout en bout sont tous deux pris en charge avec les services multi-locataires. Pour un arrêt TLS au niveau de la passerelle d'application, l'ajout du certificat TLS à l'écouteur de la passerelle d'application est toujours requis. En revanche, pour le de protocole TLS de bout en bout, les services Azure approuvés, tels que les applications web du service Azure App, n’exigent pas l’autorisation des serveurs principaux de la passerelle d’application. Par conséquent, il n’est pas nécessaire d’ajouter de certificat d’authentification. 

![Protocole TLS de bout en bout](./media/application-gateway-web-app-overview/end-to-end-ssl.png)

Notez que dans l’illustration ci-dessus, il n’est pas nécessaire d’ajouter des certificats d’authentification lorsque App service est sélectionné en tant que serveur principal.

### <a name="health-probe"></a>Sonde d’intégrité

Le remplacement de l’en-tête d’hôte dans les **paramètres HTTP** affecte uniquement la requête et son routage. Il n’affecte pas le comportement de la sonde d’intégrité. Pour que la fonctionnalité de bout en bout fonctionne, la sonde et les paramètres HTTP doivent être modifiés pour refléter la configuration correcte. En plus d’offrir la possibilité de spécifier un en-tête d’hôte dans la configuration de sonde, les sondes personnalisées prennent également en charge la possibilité de dériver l’en-tête d’hôte à partir des paramètres HTTP configurés actuellement. Cette configuration peut être spécifiée à l’aide du paramètre `PickHostNameFromBackendHttpSettings` dans la configuration de la sonde.

### <a name="redirection-to-app-services-url-scenario"></a>Redirection vers le scénario de l’URL App Service

Il peut y avoir des scénarios où le nom d’hôte figurant dans la réponse issue du service App service peut diriger le navigateur de l’utilisateur final vers le nom d’hôte *.azurewebsites.net plutôt que vers le domaine associé à la passerelle Application Gateway. Ce problème peut se produire lorsque :

- Vous avez configuré la redirection sur votre App Service. La redirection peut être aussi simple que l’ajout d’une barre oblique à la demande.
- L’authentification Azure AD provoque la redirection.

Pour résoudre de tels cas, consultez [Résolution des problèmes de redirection vers l’URL de l’App Service](./troubleshoot-app-service-redirection-app-service-url.md).

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment configurer une passerelle d’application avec une application multi-locataires, telles qu’une application web Azure App service, en tant que membre du pool de serveurs principaux en consultant l’article suivant : [Configurer les applications web App Service avec Application Gateway](./configure-web-app-portal.md).
