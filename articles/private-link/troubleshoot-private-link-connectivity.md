---
title: Résoudre les problèmes de connectivité d’Azure Private Link
description: Guide pas à pas pour diagnostiquer la connectivité de liaison privée
services: private-link
documentationcenter: na
author: rdhillon
manager: narayan
editor: ''
ms.service: private-link
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/31/2020
ms.author: rdhillon
ms.openlocfilehash: 45a7a146dd929408b50a0045fe2598726ee05505
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95544307"
---
# <a name="troubleshoot-azure-private-link-connectivity-problems"></a>Résoudre les problèmes de connectivité d’Azure Private Link

Cet article donne les instructions pas à pas à suivre pour valider et diagnostiquer la connectivité d’une installation Azure Private Link.

Azure Private Link permet d’accéder aux services PaaS (platform as a service) Azure, par exemple Stockage Azure, Azure Cosmos DB et Azure SQL Database, ainsi qu’aux services de partenaires ou de clients hébergés par Azure sur un point de terminaison privé du réseau virtuel. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, ce qui élimine toute exposition sur l’Internet public. Vous pouvez également créer votre propre service Private Link dans votre réseau virtuel et le distribuer en privé à vos clients.

Vous pouvez activer votre service qui s’exécute en arrière-plan du niveau Standard d’Azure Load Balancer pour l’accès à Private Link. Les consommateurs de votre service peuvent créer un point de terminaison privé à l’intérieur de leur réseau virtuel, et le mapper à ce service pour y accéder en privé.

Voici les scénarios de connectivité disponibles avec Private Link :

- Réseau virtuel de la même région
- réseaux virtuels avec peering régional ;
- réseaux virtuels avec peering mondial ;
- Circuits Azure ExpressRoute ou client local via VPN

## <a name="deployment-troubleshooting"></a>Résolution des problèmes liés au déploiement

Lisez les informations de la section [Désactivation des stratégies réseau sur le service Private Link](./disable-private-link-service-network-policy.md) pour résoudre les problèmes vous empêchant de sélectionner l’adresse IP source à partir du sous-réseau de votre choix pour votre service Private Link.

Vérifiez que le paramètre **privateLinkServiceNetworkPolicies** est désactivé pour le sous-réseau à partir duquel vous sélectionnez l’adresse IP source.

## <a name="diagnose-connectivity-problems"></a>Diagnostiquer les problèmes de connectivité

Si vous rencontrez des problèmes de connectivité liés à votre installation Private Link, passez en revue ces étapes pour vérifier que toutes les configurations habituelles sont bien correctes.

1. Vérifiez la configuration Private Link en parcourant la ressource.

    a. Accédez au **Centre Private Link**.

      ![Centre de liaisons privées](./media/private-link-tsg/private-link-center.png)

    b. Dans le volet gauche, sélectionnez **Services Private Link**.

      ![Services Private Link](./media/private-link-tsg/private-link-service.png)

    c. Filtrez et sélectionnez le service Private Link que vous souhaitez diagnostiquer.

    d. Vérifiez les connexions de point de terminaison privé.
     - Vérifiez que l’état de la connexion du point de terminaison privé à partir duquel vous voulez établir une connectivité est **Approuvé**.
     - S’il est **En attente**, sélectionnez-le et approuvez-le.

       ![Connexions des points de terminaison privés](./media/private-link-tsg/pls-private-endpoint-connections.png)

     - Accédez au point de terminaison privé à partir duquel vous vous connectez en sélectionnant son nom. Vérifiez que l’état de la connexion est **Approuvé**.

       ![Vue d’ensemble de connexion du point de terminaison privé](./media/private-link-tsg/pls-private-endpoint-overview.png)

     - Une fois les deux côtés approuvés, réessayez de vous connecter.

    e. Vérifiez **Alias** dans l’onglet **Vue d’ensemble** et **ID de la ressource** dans l’onglet **Propriétés**.
     - Vérifiez que les informations **Alias** et **ID de la ressource** correspondent aux informations **Alias** et **ID de la ressource** que vous utilisez pour créer un point de terminaison privé vers ce service.

       ![Vérifier l’information Alias](./media/private-link-tsg/pls-overview-pane-alias.png)

       ![Vérifier l’information ID de la ressource](./media/private-link-tsg/pls-properties-pane-resourceid.png)

    f. Vérifiez l’information **Visibilité** dans l’onglet **Vue d’ensemble**.
     - Vérifiez que votre abonnement se trouve dans l’étendue **Visibilité**.

       ![Vérifier l’information Visibilité](./media/private-link-tsg/pls-overview-pane-visibility.png)

    g. Vérifiez l’information **Équilibreur de charge** dans l’onglet **Vue d’ensemble**.
     - Pour accéder à l’équilibreur de charge, sélectionnez le lien correspondant.

       ![Vérifier l’information Équilibreur de charge](./media/private-link-tsg/pls-overview-pane-ilb.png)

     - Vérifiez que les paramètres de l’équilibreur de charge sont configurés conformément à vos attentes.
       - Vérifiez **Configuration IP front-end**.
       - Vérifiez **Pools back-end**.
       - Vérifiez **Règles d’équilibrage de charge**.

       ![Vérifier les propriétés de l’équilibreur de charge](./media/private-link-tsg/pls-ilb-properties.png)

     - Vérifiez que l’équilibreur de charge fonctionne suivant les paramètres précédents.
       - Sélectionnez une machine virtuelle dans un sous-réseau autre que celui où le pool back-end de l’équilibreur de charge est disponible.
       - Essayez d’accéder au front-end de l’équilibreur de charge à partir de la machine virtuelle précédente.
       - Si la connexion est établie avec le pool back-end suivant les règles d’équilibrage de charge, votre équilibreur de charge est opérationnel.
       - Vous pouvez également consulter la métrique de l’équilibreur de charge avec Azure Monitor pour voir si les données transitent par l’équilibreur de charge.

1. Utilisez [Azure Monitor](../azure-monitor/overview.md) pour vérifier que les données sont transmises.

    a. Sur une ressource du service Private Link, sélectionnez **Métriques**.
     - Sélectionnez **Octets entrants** ou **Octets sortants**.
     - Vérifiez que les données sont transmises lorsque vous tentez de vous connecter au service Private Link. Prévoyez un retard d’environ 10 minutes.

       ![Vérifier les métriques du service Private Link](./media/private-link-tsg/pls-metrics.png)

1. Si votre problème n’est toujours pas résolu et qu’un problème de connectivité persiste, contactez l’[équipe de support Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

## <a name="next-steps"></a>Étapes suivantes

 * [Créer un service Private Link (CLI)](./create-private-link-service-cli.md)
 * [Guide de résolution des problèmes des points de terminaison privés Azure](troubleshoot-private-endpoint-connectivity.md)