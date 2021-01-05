---
title: Configuration des règles de routage des requêtes d’Azure Application Gateway
description: Cet article explique comment configurer les règles d’acheminement des requêtes d’Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 09/09/2020
ms.author: surmb
ms.openlocfilehash: 047ce9b33836e2c23a37b1383942323d7c382485
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397533"
---
# <a name="application-gateway-request-routing-rules"></a>Règles de routage des requêtes Application Gateway

Quand vous créez une passerelle d’application à l’aide du portail Azure, vous créez une règle par défaut ( *rule1* ). Cette règle lie l’écouteur par défaut ( *appGatewayHttpListener* ) au pool de back-ends par défaut ( *appGatewayBackendPool* ) et aux paramètres HTTP du back-end par défaut ( *appGatewayBackendHttpSettings* ). Après avoir créé la passerelle, vous pouvez modifier les paramètres de la règle par défaut ou créer des règles.

## <a name="rule-type"></a>Type de règle

Quand vous créez une règle, vous choisissez entre le type [*de base* et le type *basé sur un chemin*](./application-gateway-components.md#request-routing-rules).

- Choisissez le type de base pour transférer toutes les demandes sur l’écouteur associé (par exemple, *blog <i></i>.contoso.com/\*)* à un seul pool de back-ends.
- Choisissez le type basé sur un chemin pour router les demandes provenant de chemins d’URL spécifiques vers des pools de back-ends spécifiques. Le modèle de chemin s’applique uniquement au chemin de l’URL, pas à ses paramètres de demande.

### <a name="order-of-processing-rules"></a>Ordre de traitement des règles

Pour la référence SKU v1 et v2, les critères spéciaux des demandes entrantes sont traités dans l’ordre dans lequel les chemins sont listés dans le mappage du chemin d’URL de la règle basée sur un chemin. Si une demande correspond au modèle dans deux chemins ou plus du mappage du chemin, le chemin listé en premier est mappé. Et la demande est transférée au back-end associé à ce chemin.

## <a name="associated-listener"></a>Écouteur associé

Associez un écouteur à la règle de sorte que la *règle de routage des demandes* associée à l’écouteur soit évaluée pour déterminer le pool de back-ends vers lequel router la demande.

## <a name="associated-back-end-pool"></a>Pool de back-ends associé

Associez à la règle le pool de back-ends qui contient les cibles back-end qui servent les demandes que l’écouteur reçoit.

 - Pour une règle de base, un seul pool de back-ends est autorisé. Toutes les demandes sur l’écouteur associé sont transférées à ce pool de back-ends.

 - Pour une règle basée sur un chemin, ajoutez plusieurs pools de back-ends qui correspondent à chaque chemin d’URL. Les demandes correspondant à ce chemin d’URL entré sont transférées au pool de back-ends correspondant. De plus, ajoutez un pool de back-ends par défaut. Les demandes qui ne correspondent à aucun chemin d’URL dans la règle sont transférées à ce pool.

## <a name="associated-back-end-http-setting"></a>Paramètre HTTP du back-end associé

Ajoutez un paramètre HTTP de back-end pour chaque règle. Les demandes sont routées depuis la passerelle d’application vers les cibles back-end à l’aide du numéro de port, du protocole et d’autres informations spécifiées dans ce paramètre.

Pour une règle de base, un seul paramètre HTTP du back-end est autorisé. Toutes les demandes sur l’écouteur associé sont transférées aux cibles back-end correspondantes en utilisant ce paramètre HTTP.

Pour une règle basée sur un chemin, ajoutez plusieurs paramètres HTTP du back-end qui correspondent à chaque chemin d’URL. Les demandes correspondant à ce chemin d’URL dans ce paramètre sont transférées aux cibles back-end correspondantes en utilisant les paramètres HTTP qui correspondent à chaque chemin d’URL. En outre, ajoutez un paramètre HTTP par défaut. Les demandes qui ne correspondent à aucun chemin d’URL dans cette règle sont transférées au pool back-end par défaut en utilisant le paramètre HTTP par défaut.

## <a name="redirection-setting"></a>Paramètre de redirection

Si la redirection est configurée pour une règle de base, toutes les demandes sur l’écouteur associé sont redirigées vers la cible. Il s’agit d’une redirection *globale*. Si la redirection est configurée pour une règle basée sur un chemin, seules les demandes dans une zone de site spécifique sont redirigées. Prenons l’exemple d’une zone de panier d’achat indiquée par */cart/\** . Il s’agit d’une redirection *basée sur un chemin*.

Pour plus d’informations sur les redirections, consultez [Vue d’ensemble de la redirection Application Gateway](redirect-overview.md).

### <a name="redirection-type"></a>Type de redirection

Choisissez le type de redirection nécessaire : *Permanent (301)* , *Temporaire (307)* , *Trouvé (302)* ou *Voir autre (303)* .

### <a name="redirection-target"></a>Cible de redirection

Choisissez un autre écouteur ou un site externe comme cible de redirection.

#### <a name="listener"></a>Écouteur

Choisissez l’écouteur comme cible de redirection pour rediriger le trafic depuis un écouteur vers un autre sur la passerelle. Ce paramètre est obligatoire quand vous voulez activer la redirection HTTP vers HTTPS. Il redirige le trafic depuis l’écouteur source qui vérifie les demandes HTTP entrantes vers l’écouteur de destination qui vérifie les demandes HTTPS entrantes. Vous pouvez également choisir d’inclure la chaîne et le chemin de requête de la demande d’origine dans la demande transférée à la cible de redirection.

![Boîte de dialogue Composants d’Application Gateway](./media/configuration-overview/configure-redirection.png)

Pour plus d’informations sur la redirection HTTP vers HTTPS, consultez :
- [Redirection HTTP vers HTTPS à l’aide du portail Azure](redirect-http-to-https-portal.md)
- [Redirection HTTP vers HTTPS à l’aide de PowerShell](redirect-http-to-https-powershell.md)
- [Redirection HTTP vers HTTPS à l’aide d’Azure CLI](redirect-http-to-https-cli.md)

#### <a name="external-site"></a>Site externe

Choisissez un site externe quand vous voulez rediriger le trafic sur l’écouteur associé à cette règle vers un site externe. Vous pouvez choisir d’inclure la chaîne de requête de la demande d’origine dans la demande transférée à la cible de redirection. Vous ne pouvez pas transférer le chemin au site externe qui était dans la demande d’origine.

Pour plus d’informations sur la redirection, consultez :
- [Rediriger le trafic vers un site externe à l’aide de PowerShell](redirect-external-site-powershell.md)
- [Rediriger le trafic vers un site externe à l’aide de l’interface de ligne de commande](redirect-external-site-cli.md)

## <a name="rewrite-http-headers-and-url"></a>Réécrire les en-têtes et les URL HTTP

En utilisant des règles de réécriture, vous pouvez ajouter, supprimer ou mettre à jour les en-têtes de requête et de réponse HTTP(S) ainsi que les paramètres du chemin URL et de la chaîne de requête lorsque les paquets de requête et de réponse se déplacent entre le client et des pools principaux via la passerelle d’application.

Vous pouvez régler les en-têtes et les paramètres d’URL sur des valeurs statiques ou d’autres en-têtes et variables de serveur. Cela facilite les cas d’usage importants, tels que l’extraction d’adresses IP des clients, la suppression d’informations sensibles relatives au back-end, le renforcement de la sécurité, etc.
Pour plus d'informations, consultez les pages suivantes :

 - [Vue d’ensemble de la réécriture des en-têtes HTTP et des URL](rewrite-http-headers-url.md)
 - [Configurer la réécriture des en-têtes HTTP](rewrite-http-headers-portal.md)
 - [Configurer la réécriture d’URL](rewrite-url-portal.md)

## <a name="next-steps"></a>Étapes suivantes

- [En savoir plus sur les paramètres HTTP](configuration-http-settings.md)