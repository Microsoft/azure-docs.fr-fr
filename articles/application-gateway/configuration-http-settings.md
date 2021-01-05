---
title: Configuration des paramètres HTTP d'Azure Application Gateway
description: Cet article explique comment configurer les paramètres HTTP d'Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 09/09/2020
ms.author: surmb
ms.openlocfilehash: c0c939a6a8323dfdfafddb46ccdb7d7ef3dd2f2c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89652594"
---
# <a name="application-gateway-http-settings-configuration"></a>Configuration des paramètres HTTP d'Application Gateway

La passerelle d’application route le trafic vers les serveurs back-end en utilisant la configuration que vous spécifiez ici. Après avoir créé un paramètre HTTP, vous devez l’associer à une ou plusieurs règles de routage des demandes.

## <a name="cookie-based-affinity"></a>Affinité basée sur les cookies

Azure Application Gateway utilise des cookies gérés par passerelle pour gérer les sessions utilisateur. Quand un utilisateur envoie la première demande à Application Gateway, il définit un cookie d’affinité dans la réponse avec une valeur de hachage qui contient les détails de la session, afin que les demandes suivantes incluant le cookie d’affinité soient routées vers le même serveur back-end pour conserver l’adhérence. 

Cette fonctionnalité est pratique quand vous voulez conserver une session utilisateur sur le même serveur et quand l’état de session est enregistré localement sur le serveur pour une session utilisateur. Si l’application ne peut pas gérer l’affinité basée sur les cookies, vous ne pouvez pas utiliser cette fonctionnalité. Pour l’utiliser, assurez-vous que les clients prennent en charge les cookies.

La [mise à jour v80](https://chromiumdash.appspot.com/schedule) du [navigateur Chromium](https://www.chromium.org/Home) a permis que les cookies HTTP sans attribut [SameSite](https://tools.ietf.org/id/draft-ietf-httpbis-rfc6265bis-03.html#rfc.section.5.3.7) soient traités comme SameSite=Lax. Dans le cas de requêtes CORS (Cross-Origin Resource Sharing), si le cookie doit être envoyé dans un contexte tiers, il doit utiliser les attributs *SameSite=None; Secure* et être envoyé seulement via HTTPS. Dans le cas contraire, dans un scénario HTTP uniquement, le navigateur n’envoie pas les cookies dans le contexte tiers. L’objectif de cette mise à jour à partir de Chrome est d’améliorer la sécurité et d’éviter les attaques par falsification de requête intersites (CSRF). 

Pour prendre en charge ce changement, à partir du 17 février 2020, Application Gateway (tous les types de références SKU) injecte un autre cookie appelé *ApplicationGatewayAffinityCORS* en plus du cookie *ApplicationGatewayAffinity* existant. Le cookie *ApplicationGatewayAffinityCORS* a deux attributs supplémentaires qui lui sont ajoutés ( *« SameSite=None; Secure »* ), afin que les sessions persistantes soient conservées même pour les requêtes cross-origin.

Notez que le nom du cookie d’affinité par défaut est *ApplicationGatewayAffinity* et que vous pouvez changer ce nom. Si vous utilisez un nom de cookie d’affinité personnalisé, un cookie supplémentaire est ajouté avec le suffixe CORS. Par exemple, *CustomCookieNameCORS*.

> [!NOTE]
> Si l’attribut *SameSite=None* est défini, le cookie doit obligatoirement contenir aussi l’indicateur *Secure* et être envoyé via HTTPS.  Si l’affinité de session est exigée sur CORS, vous devez migrer votre charge de travail vers HTTPS. Reportez-vous à la documentation sur le déchargement TLS et le protocole TLS de bout en bout pour Application Gateway ici : [Vue d’ensemble](ssl-overview.md), [Configurer une passerelle Application Gateway avec terminaison TLS à l’aide du portail Azure](create-ssl-portal.md), [Configurer le protocole TLS de bout en bout à l’aide d’Application Gateway avec le portail](end-to-end-ssl-portal.md).

## <a name="connection-draining"></a>Vidage des connexions

Le vidage des connexions vous permet de supprimer élégamment des membres du pool de back-ends pendant les mises à jour de service planifiées. Vous pouvez appliquer ce paramètre à tous les membres d’un pool principal en activant le drainage de la connexion sur le paramètre HTTP. Elle garantit que toutes les instances de désinscription d’un pool de back-ends continuent à gérer les connexions existantes et à traiter les demandes en cours pendant un délai d’expiration configurable, et ne reçoivent aucune nouvelle demande ou connexion. La seule exception concerne les demandes liées à la désinscription des instances en raison d’une affinité de session gérée par la passerelle et qui continueront d’être transmises par proxy aux instances de désinscription. Le vidage des connexions s’applique aux instances back-end qui sont explicitement supprimées du pool de back-ends.

## <a name="protocol"></a>Protocol

Application Gateway prend en charge les protocoles HTTP et HTTPS pour router les demandes vers les serveurs back-end. Si vous choisissez HTTP, le trafic vers les serveurs back-end est non chiffré. Si des communications non chiffrées ne sont pas acceptables, sélectionnez HTTPS.

Ce paramètre combiné avec HTTPS dans l’écouteur prend en charge le chiffrement [TLS de bout en bout](ssl-overview.md). Celui-ci vous permet de transmettre en toute sécurité des données sensibles chiffrées au serveur back-end. Chaque serveur back-end du pool de back-ends pour lequel un chiffrement TLS de bout en bout est activé doit être configuré avec un certificat pour permettre une communication sécurisée.

## <a name="port"></a>Port

Ce paramètre spécifie le port où les serveurs back-end écoutent le trafic provenant de la passerelle d’application. Vous pouvez configurer des ports allant de 1 à 65535.

## <a name="request-timeout"></a>Délai d’expiration de la demande

Ce paramètre correspond au nombre de secondes pendant lesquelles la passerelle d’application attend de recevoir une réponse de la part du serveur back-end.

## <a name="override-back-end-path"></a>Remplacer le chemin back-end

Ce paramètre vous permet de configurer un chemin de transfert personnalisé facultatif à utiliser quand la demande est transférée au back-end. Toute partie du chemin entrant qui correspond au chemin personnalisé dans le champ **Remplacer le chemin back-end** est copiée dans le chemin transféré. Le tableau suivant montre comment agit cette fonctionnalité :

- Quand le paramètre HTTP est attaché à une règle de routage des demandes de base :

  | Demande d’origine  | Remplacer le chemin back-end | Demande transférée au back-end |
  | ----------------- | --------------------- | ---------------------------- |
  | /home/            | /override/            | /override/home/              |
  | /home/secondhome/ | /override/            | /override/home/secondhome/   |

- Quand le paramètre HTTP est attaché à une règle de routage des demandes basée sur un chemin :

  | Demande d’origine           | Règle de chemin       | Remplacer le chemin back-end | Demande transférée au back-end |
  | -------------------------- | --------------- | --------------------- | ---------------------------- |
  | /pathrule/home/            | /pathrule*      | /override/            | /override/home/              |
  | /pathrule/home/secondhome/ | /pathrule*      | /override/            | /override/home/secondhome/   |
  | /home/                     | /pathrule*      | /override/            | /override/home/              |
  | /home/secondhome/          | /pathrule*      | /override/            | /override/home/secondhome/   |
  | /pathrule/home/            | /pathrule/home* | /override/            | /override/                   |
  | /pathrule/home/secondhome/ | /pathrule/home* | /override/            | /override/secondhome/        |
  | /pathrule/                 | /pathrule/      | /override/            | /override/                   |

## <a name="use-for-app-service"></a>Utiliser pour App Service

Il s’agit d’un raccourci d’interface utilisateur uniquement qui sélectionne les deux paramètres nécessaires au back-end Azure App Service. Ce raccourci permet de **choisir un nom d’hôte à partir d’une adresse back-end**, puis il crée une sonde personnalisée si vous n’en avez pas déjà une. (Pour plus d'informations, consultez la section du paramètre [Choisir un nom d’hôte à partir d’une adresse back-end](#pick-host-name-from-back-end-address) dans cet article). Une nouvelle sonde est créée, puis l’en-tête de la sonde est choisie à partir de l’adresse du membre back-end.

## <a name="use-custom-probe"></a>Utiliser une sonde personnalisée

Ce paramètre associe une [sonde personnalisée](application-gateway-probe-overview.md#custom-health-probe) à un paramètre HTTP. Vous pouvez associer une seule sonde personnalisée à un paramètre HTTP. Si vous n’associez pas explicitement une sonde personnalisée, la [sonde par défaut](application-gateway-probe-overview.md#default-health-probe-settings) est utilisée pour superviser l’intégrité du back-end. Nous vous recommandons de créer une sonde personnalisée pour mieux contrôler la supervision de l’intégrité de vos back-ends.

> [!NOTE]
> La sonde personnalisée ne supervise pas l’intégrité du pool de back-ends, sauf si le paramètre HTTP correspondant est explicitement associé à un écouteur.

## <a name="pick-host-name-from-back-end-address"></a>Choisir un nom d’hôte à partir d’une adresse back-end

Cette fonctionnalité définit dynamiquement l’en-tête de l’*hôte* dans la demande sur le nom d’hôte du pool de back-ends. Elle utilise une adresse IP ou un nom de domaine complet.

Cette fonctionnalité s’avère utile quand le nom de domaine du back end est différent du nom DNS de la passerelle d’application et que le back-end s’appuie sur un en-tête d’hôte spécifique pour se résoudre en point de terminaison correct.

Exemple : des services multilocataires en tant que back-end. Un service d’application est un service multilocataire qui utilise un espace partagé avec une seule adresse IP. Ainsi, un service d’application est uniquement accessible par le biais des noms d’hôte configurés dans les paramètres du domaine personnalisé.

Par défaut, le nom de domaine personnalisé est *example.azurewebsites.net*. Pour accéder à votre service d’application à l’aide d’une passerelle d’application par le biais d’un nom d’hôte qui n’est pas explicitement inscrit dans le service d’application ou par le biais du nom de domaine complet de la passerelle d’application, vous remplacez le nom d’hôte dans la demande d’origine par celui du service d’application. Pour cela, activez le paramètre **Choisir un nom d’hôte à partir d’une adresse back-end**.

Pour un domaine personnalisé dont le nom DNS personnalisé existant est mappé au service d’application, vous n’êtes pas obligé d’activer ce paramètre.

> [!NOTE]
> Il n’est pas nécessaire pour App Service Environment, qui est un déploiement dédié.

## <a name="host-name-override"></a>Remplacement du nom d’hôte

Cette fonctionnalité remplace l’en-tête de l’*hôte* dans la demande entrante sur la passerelle d’application par le nom d’hôte que vous spécifiez.

Par exemple, si *www.contoso.com* est spécifié dans le paramètre **Nom d’hôte**, la demande d’origine *`https://appgw.eastus.cloudapp.azure.com/path1` est remplacée par *`https://www.contoso.com/path1` quand la demande est transférée au back-end.

## <a name="next-steps"></a>Étapes suivantes

- [En savoir plus sur le pool principal](configuration-overview.md#back-end-pool)