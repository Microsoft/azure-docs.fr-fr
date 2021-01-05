---
title: Connecter votre application à SQL Managed Instance
titleSuffix: Azure SQL Managed Instance
description: Cet article explique comment connecter votre application à Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova, vanto
ms.date: 11/09/2018
ms.openlocfilehash: dd5c6527cd6a0beea291dce94ff0e5949ba00671
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791254"
---
# <a name="connect-your-application-to-azure-sql-managed-instance"></a>Connecter votre application à Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Aujourd’hui, plusieurs choix s’offrent à vous pour déterminer comment et où héberger votre application.

Vous pouvez choisir le cloud en utilisant Azure App Service ou certaines options intégrées de réseau virtuel Azure comme Azure App Service Environment, des machines virtuelles Azure, ainsi que des groupes de machines virtuelles identiques. Vous pouvez également adopter une approche du cloud hybride pour conserver vos applications localement.

Quel que soit le choix effectué, vous pouvez le connecter à Azure SQL Managed Instance. 

![Haute disponibilité](./media/connect-application-instance/application-deployment-topologies.png)

Cet article explique comment connecter une application à Azure SQL Managed Instance dans plusieurs scénarios d’application différents. 

## <a name="connect-inside-the-same-vnet"></a>Connexion à l’intérieur du même réseau virtuel

La connexion d’une application à l’intérieur du même réseau virtuel que SQL Managed Instance est le scénario le plus simple. Des machines virtuelles à l’intérieur du réseau virtuel peuvent se connecter entre elles directement même si elles sont dans des sous-réseaux différents. Cela signifie que pour connecter une application dans App Service Environment ou une machine virtuelle, il vous suffit de définir correctement la chaîne de connexion.  

## <a name="connect-inside-a-different-vnet"></a>Connexion à l’intérieur d’un autre réseau virtuel

La connexion d’une application lorsqu’elle réside dans un autre réseau virtuel de SQL Managed Instance est un peu plus complexe, car SQL Managed Instance dispose d’adresses IP privées dans son propre réseau virtuel. Pour établir la connexion, une application a besoin d’accéder au réseau virtuel dans lequel SQL Managed Instance est déployé. Ainsi, vous devez établir une connexion entre l’application et le réseau virtuel SQL Managed Instance. Les réseaux virtuels ne doivent pas nécessairement être dans le même abonnement pour que ce scénario fonctionne.

Il existe deux options pour connecter des réseaux virtuels :

- [Peering de réseaux virtuels Azure](../../virtual-network/virtual-network-peering-overview.md)
- Passerelle VPN de réseau virtuel à réseau virtuel ([portail Azure](../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md), [PowerShell](../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md), [Azure CLI](../../vpn-gateway/vpn-gateway-howto-vnet-vnet-cli.md))

Le peering est préférable car il utilise le réseau principal de Microsoft, donc du point de vue de la connectivité, il n’y a pas de différence notable de latence entre les machines virtuelles dans le réseau virtuel appairé et dans le même réseau virtuel. L’appairage de réseaux virtuels est pris en charge entre les réseaux d’une même région. L’appairage de réseaux virtuels mondiaux est également pris en charge avec la limitation décrite dans la remarque ci-dessous.  

> [!IMPORTANT]
> [Le 22/09/2020, nous avons annoncé l’appairage de réseaux virtuels mondiaux pour les clusters virtuels nouvellement créés](https://azure.microsoft.com/en-us/updates/global-virtual-network-peering-support-for-azure-sql-managed-instance-now-available/). Cela signifie que l’appairage de réseaux virtuels mondiaux est pris en charge pour les instances gérées SQL créées dans des sous-réseaux vides après la date d’annonce, ainsi que pour toutes les instances gérées ultérieures créées dans ces sous-réseaux. Pour toutes les autres instances gérées SQL, la prise en charge de l’appairage est limitée aux réseaux de la même région en raison des [contraintes de l’appairage de réseaux virtuels mondiaux](../../virtual-network/virtual-network-manage-peering.md#requirements-and-constraints). Consultez également la section appropriée de l’article [Forum Aux Questions sur les réseaux virtuel Azure](../../virtual-network/virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers) pour plus d’informations. 

## <a name="connect-from-on-premises"></a>Connexion en local 

Vous pouvez également connecter votre application locale à SQL Managed Instance. SQL Managed Instance est uniquement accessible par le biais d’une adresse IP privée. Afin d’y accéder localement, vous devez établir une connexion de site à site entre l’application et le réseau virtuel SQL Managed Instance.

Vous avez deux options pour la connexion locale à un réseau virtuel Azure :

- Connexion VPN de site à site ([portail Azure](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md), [PowerShell](../../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md), [Azure CLI](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-cli.md))
- La connexion [Azure ExpressRoute](../../expressroute/expressroute-introduction.md)  

Si vous avez établi une connexion locale à Azure et que vous ne parvenez pas à établir une connexion à SQL Managed Instance, vérifiez si votre pare-feu dispose d’une connexion sortante ouverte sur le port SQL 1433 et la plage de ports 11000 à 11999 à des fins de redirection.

## <a name="connect-the-developer-box"></a>Connexion de la box de développeur

Il est également possible de connecter votre box de développeur à SQL Managed Instance. SQL Managed Instance est uniquement accessible par le biais d’une adresse IP privée, donc pour y accéder à partir de votre box de développeur, vous devez d’abord établir une connexion entre cette dernière et le réseau virtuel SQL Managed Instance. Pour cela, configurez une connexion point à site à un réseau virtuel à l’aide de l’authentification par certificat Azure native. Pour plus d’informations, consultez [Configurer une connexion point à site pour se connecter à Azure SQL Managed Instance à partir d’un ordinateur local](point-to-site-p2s-configure.md).

## <a name="connect-with-vnet-peering"></a>Connexion avec un peering de réseaux virtuels

Un autre scénario utilisé par les clients est celui où une passerelle VPN est installée sur un réseau virtuel et dans un abonnement différent de ceux où est hébergé SQL Managed Instance. Les deux réseaux virtuels sont ensuite appairés. Le diagramme d’architecture suivant montre comment ce scénario peut être implémenté.

![Peering de réseau virtuel](./media/connect-application-instance/vnet-peering.png)

Une fois que vous avez configuré l’infrastructure de base, vous devez modifier certains paramètres afin que la passerelle VPN puisse voir les adresses IP dans le réseau virtuel qui héberge SQL Managed Instance. Pour ce faire, apportez les modifications très spécifiques qui suivent dans **Paramètres de peering** .

1. Dans le réseau virtuel qui héberge la passerelle VPN, accédez à **Peerings** , puis à la connexion de réseau virtuel appairée à SQL Managed Instance, et cliquez sur **Autoriser le transit par passerelle** .
2. Dans le réseau virtuel qui héberge SQL Managed Instance, accédez à **Peerings** , puis à la connexion de réseau virtuel appairée pour la passerelle VPN, puis cliquez sur **Utiliser des passerelles distantes** .

## <a name="connect-azure-app-service"></a>Connexion d’Azure App Service 

Vous pouvez également connecter une application hébergée par Azure App Service. SQL Managed Instance est uniquement accessible par le biais d’une adresse IP privée, donc pour y accéder à partir d’Azure App Service, vous devez d’abord établir une connexion entre l’application et le réseau virtuel SQL Managed Instance. Consultez [Intégrer une application à un réseau virtuel Azure](../../app-service/web-sites-integrate-with-vnet.md).  

Pour résoudre les problèmes, consultez [Résolution des problèmes liés aux réseaux virtuels et aux applications](../../app-service/web-sites-integrate-with-vnet.md#troubleshooting). Si aucune connexion ne peut être établie, essayez de [synchroniser la configuration de la mise en réseau](azure-app-sync-network-configuration.md).

L’intégration d’Azure App Service à un réseau homologué avec un réseau virtuel SQL Managed Instance constitue un cas spécial de connexion entre Azure App Service et le réseau virtuel SQL Managed Instance. Ce cas nécessite la configuration suivante :

- Le réseau virtuel SQL Managed Instance ne doit PAS avoir de passerelle  
- L’option `Use remote gateways` doit être définie pour le réseau virtuel SQL Managed Instance
- L’option `Allow gateway transit` doit être définie pour le réseau virtuel appairé

Ce scénario est illustré dans le diagramme suivant :

![peering d’applications intégrées](./media/connect-application-instance/integrated-app-peering.png)

>[!NOTE]
>La fonctionnalité d’intégration du réseau virtuel n’intègre pas une application à un réseau virtuel doté d’une passerelle ExpressRoute. Même si la passerelle ExpressRoute est configurée en mode de coexistence, l’intégration du réseau virtuel ne fonctionnera pas. S’il vous faut accéder aux ressources via une connexion ExpressRoute, vous pouvez utiliser App Service Environment s’exécutant dans votre réseau virtuel.

## <a name="troubleshooting-connectivity-issues"></a>Résolution des problèmes de connectivité

Pour résoudre les problèmes de connectivité, lisez ce qui suit :

- Si vous ne parvenez pas à vous connecter à SQL Managed Instance à partir d’une machine virtuelle Azure au sein du même réseau virtuel, mais que vous y parvenez à partir d’un autre sous-réseau, vérifiez si un groupe de sécurité réseau défini sur le sous-réseau de machine virtuelle bloque l’accès. De plus, ouvrez la connexion sortante sur le port SQL 1433 ainsi que les ports de la plage 11000-11999, car ceux-ci sont nécessaires pour la connexion via la redirection à l’intérieur de la limite Azure.
- Pour la table de routage associée au réseau virtuel, vérifiez que la propagation BGP est définie sur **Activé** .
- Si vous utilisez une connexion VPN point à site, accédez à la configuration dans le portail Azure pour voir si les sections **Entrée/Sortie** contiennent des chiffres. La présence de chiffres autres que zéro indiquent qu’Azure achemine le trafic entrant et sortant sur l’ordinateur local.

   ![Chiffres d’entrée et de sortie](./media/connect-application-instance/ingress-egress-numbers.png)

- Vérifiez que la machine cliente (qui exécute le client VPN) affiche des entrées de routage pour tous les réseaux virtuels auxquels vous avez besoin d’accéder. Les itinéraires sont stockés dans `%AppData%\ Roaming\Microsoft\Network\Connections\Cm\<GUID>\routes.txt`.

   ![route.txt](./media/connect-application-instance/route-txt.png)

   Comme le montre cette image, il y a deux entrées pour chaque réseau virtuel impliqué, et une troisième entrée pour le point de terminaison VPN qui est configuré dans le portail.

   Une autre façon de vérifier les itinéraires est d’utiliser la commande suivante. La sortie montre les itinéraires vers les différents sous-réseaux :

   ```cmd
   C:\ >route print -4
   ===========================================================================
   Interface List
   14...54 ee 75 67 6b 39 ......Intel(R) Ethernet Connection (3) I218-LM
   57...........................rndatavnet
   18...94 65 9c 7d e5 ce ......Intel(R) Dual Band Wireless-AC 7265
   1...........................Software Loopback Interface 1
   Adapter===========================================================================

   IPv4 Route Table
   ===========================================================================
   Active Routes:
   Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0       10.83.72.1     10.83.74.112     35
         10.0.0.0    255.255.255.0         On-link       172.26.34.2     43
         10.4.0.0    255.255.255.0         On-link       172.26.34.2     43
   ===========================================================================
   Persistent Routes:
   None
   ```

- Si vous utilisez le peering de réseaux virtuels, suivez les instructions concernant la configuration des options [Autoriser le transit par passerelle et Utiliser des passerelles distantes](#connect-from-on-premises).

- Si vous utilisez le peering de réseaux virtuels pour connecter une application hébergée Azure App Service et que le réseau virtuel SQL Managed Instance a une plage d’IP publiques, assurez-vous que les paramètres de votre application hébergée autorisent le routage du trafic sortant vers des réseaux IP publics. Suivez les instructions de la section [Intégration au réseau virtuel régional](../../app-service/web-sites-integrate-with-vnet.md#regional-vnet-integration).

## <a name="required-versions-of-drivers-and-tools"></a>Versions exigées de pilotes et d’outils

Les versions minimales suivantes des outils et des pilotes sont recommandées si vous voulez vous connecter à SQL Managed Instance :

| Pilote/outil | Version |
| --- | --- |
|.NET Framework | 4.6.1 (ou .NET Core) |
|Pilote ODBC| v17 |
|Pilote PHP| 5.2.0 |
|Pilote JDBC| 6.4.0 |
|Pilote Node.js| 2.1.1 |
|Pilote OLEDB| 18.0.2.0 |
|SSMS| 18.0 ou [ultérieur](/sql/ssms/download-sql-server-management-studio-ssms) |
|[SMO](/sql/relational-databases/server-management-objects-smo/sql-server-management-objects-smo-programming-guide) | [150](https://www.nuget.org/packages/Microsoft.SqlServer.SqlManagementObjects) ou ultérieur |

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir des informations sur SQL Managed Instance, consultez [Présentation de SQL Managed Instance](sql-managed-instance-paas-overview.md).
- Pour suivre un didacticiel vous expliquant comment créer une nouvelle instance gérée, consultez [Créer une instance gérée](instance-create-quickstart.md).