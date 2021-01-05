---
title: Présentation de la configuration d’Azure Application Gateway
description: Cet article explique comment configurer les composants d’Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 09/09/2020
ms.author: surmb
ms.openlocfilehash: 0012ac99600c77dce5940387e1da54f29c3f6ab7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89652010"
---
# <a name="application-gateway-configuration-overview"></a>Présentation de la configuration d’Application Gateway

Azure Application Gateway comprend plusieurs composants que vous pouvez configurer de différentes manières pour différents scénarios. Cet article vous montre comment configurer chacun d’eux.

![Organigramme des composants d’Application Gateway](./media/configuration-overview/configuration-overview1.png)

Cette image illustre une application dotée de trois écouteurs. Les deux premiers écouteurs sont multisites pour `http://acme.com/*` et `http://fabrikam.com/*`, respectivement. Ils écoutent tous les deux le port 80. Le troisième est un écouteur de base qui dispose d’une terminaison de sécurité TLS (Transport Layer Security) de bout en bout, précédemment connue sous le nom de terminaison de protocole SSL.

## <a name="infrastructure"></a>Infrastructure

L’infrastructure Application Gateway comprend le réseau virtuel, les sous-réseaux, les groupes de sécurité réseau et les itinéraires définis par l’utilisateur.

Pour plus d’informations, consultez [Configuration de l’infrastructure Application Gateway](configuration-infrastructure.md).



## <a name="front-end-ip-address"></a>Adresse IP front-end

Vous pouvez configurer la passerelle d’application pour qu’elle ait une adresse IP publique, une adresse IP privée ou les deux. Une adresse IP publique est nécessaire quand vous hébergez un back-end auquel les clients doivent accéder par Internet par le biais d’une adresse IP virtuelle Internet.

Pour plus d’informations, consultez [Configuration d’adresse IP frontale d’Azure Application Gateway](configuration-front-end-ip.md).

## <a name="listeners"></a>Écouteurs

Un écouteur est une entité logique qui vérifie les demandes de connexion entrante en utilisant le port, le protocole, l’hôte et l’adresse IP. Quand vous configurez l’écouteur, vous devez entrer des valeurs qui correspondent aux valeurs indiquées dans la demande entrante sur la passerelle.

Pour plus d’informations, consultez [Configuration de l’écouteur Application Gateway](configuration-listeners.md).

## <a name="request-routing-rules"></a>Règles de routage des requêtes

Quand vous créez une passerelle d’application à l’aide du portail Azure, vous créez une règle par défaut (*rule1*). Cette règle lie l’écouteur par défaut (*appGatewayHttpListener*) au pool de back-ends par défaut (*appGatewayBackendPool*) et aux paramètres HTTP du back-end par défaut ( *appGatewayBackendHttpSettings*). Après avoir créé la passerelle, vous pouvez modifier les paramètres de la règle par défaut ou créer des règles.

Pour plus d’informations, consultez [Règles d’acheminement des requêtes Application Gateway](configuration-request-routing-rules.md).

## <a name="http-settings"></a>Paramètres HTTP

La passerelle d’application route le trafic vers les serveurs back-end en utilisant la configuration que vous spécifiez ici. Après avoir créé un paramètre HTTP, vous devez l’associer à une ou plusieurs règles de routage des demandes.

Pour plus d’informations, consultez [Configuration des paramètres HTTP d’Application Gateway](configuration-http-settings.md).

## <a name="back-end-pool"></a>Pool de back-ends

Vous pouvez pointer un pool de back-ends vers quatre types de membres de back-end : une machine virtuelle spécifique, un groupe de machines virtuelles identiques, une adresse IP/un nom de domaine complet ou un service d’application. 

Après avoir créé un pool de back-ends, vous devez l’associer à une ou plusieurs règles de routage des demandes. Vous devez également configurer des sondes d’intégrité pour chaque pool de back-ends sur votre passerelle d’application. Quand une condition de règle de routage des demandes est remplie, la passerelle d’application transfère le trafic aux serveurs sains (comme déterminé par les sondes d’intégrité) dans le pool de back-ends correspondant.

## <a name="health-probes"></a>Sondes d’intégrité

Par défaut, une passerelle d’application supervise l’intégrité de toutes les ressources dans son back-end. Mais nous vous recommandons vivement de créer une sonde personnalisée pour chaque paramètre HTTP de back-end afin de mieux contrôler la supervision de l’intégrité. Pour savoir comment configurer une sonde personnalisée, consultez [Paramètres de sonde d’intégrité personnalisée](application-gateway-probe-overview.md#custom-health-probe-settings).

> [!NOTE]
> Une fois que vous avez créé une sonde d’intégrité personnalisée, vous avez besoin de l’associer à un paramètre HTTP de back-end. Une sonde personnalisée ne supervise pas l’intégrité du pool de back-ends, sauf si le paramètre HTTP correspondant est explicitement associé à un écouteur à l’aide d’une règle.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous connaissez les composants d’Application Gateway, vous pouvez effectuer les opérations suivantes :

- [Créer une passerelle d’application dans le portail Azure](quick-create-portal.md)
- [Créer une passerelle d’application à l’aide de PowerShell](quick-create-powershell.md)
- [Créer une passerelle d’application à l’aide d’Azure CLI](quick-create-cli.md)
