---
title: Considérations sur la topologie du réseau du proxy d’application Azure AD
description: Couvre les considérations sur la topologie du réseau lors de l’utilisation du proxy d’application Azure AD.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 07/22/2019
ms.author: kenwith
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1d814a3ce62d7e9b3ce04953e05293d17ecc08d4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84764466"
---
# <a name="network-topology-considerations-when-using-azure-active-directory-application-proxy"></a>Considérations sur la topologie du réseau lors de l’utilisation du proxy d’application Azure Active Directory

Cet article explique les considérations de topologie réseau lors de l’utilisation de l’application de proxy d’application Azure Active Directory (Azure AD) pour la publication et l’accès à distance de vos applications.

## <a name="traffic-flow"></a>Flux de trafic

Lorsqu’une application est publiée via le proxy d’application Azure AD, le trafic des utilisateurs vers les applications circule via trois connexions :

1. L’utilisateur se connecte au point de terminaison public du service de proxy d’application Azure AD sur Azure
1. Le service de proxy d’application se connecte au connecteur de proxy d’application
1. Le connecteur de proxy d’application se connecte à l’application cible

![Diagramme illustrant le flux de trafic de l’utilisateur vers l’application cible](./media/application-proxy-network-topology/application-proxy-three-hops.png)

## <a name="tenant-location-and-application-proxy-service"></a>Emplacement du locataire et service de proxy d’application

Lorsque vous vous inscrivez pour un locataire Azure AD, la région de votre locataire est déterminée par le pays/la région que vous spécifiez. Lorsque vous activez le service de proxy d’application, les instances du service de proxy d’application de votre locataire sont sélectionnées ou créées dans la même région que votre locataire Azure AD, ou la région la plus proche de ce dernier.

Par exemple, si le pays ou la région de votre locataire Azure AD est le Royaume-Uni, tous vos connecteurs de proxy d’application utilisent des instances de service dans des centres de données européens. Lorsque vos utilisateurs accèdent à des applications publiées, leur trafic passe par les instances du service de proxy d’application à cet emplacement.

## <a name="considerations-for-reducing-latency"></a>Considérations relatives à la réduction de la latence

Toutes les solutions de proxy ajoutent de la latence à votre connexion réseau. Quel que soit le proxy ou la solution VPN que vous choisissez comme solution d’accès à distance, cela comprend toujours un ensemble de serveurs permettant la connexion à votre réseau d’entreprise.

Les organisations comptent généralement des points de terminaison de serveur dans leur réseau de périmètre. Avec le proxy d’application Azure AD, le trafic passe toutefois par le service de proxy dans le cloud tandis que les connecteurs se trouvent sur votre réseau d’entreprise. Aucun réseau de périmètre n’est requis.

Les sections suivantes contiennent des suggestions supplémentaires pour vous aider à réduire encore davantage la latence. 

### <a name="connector-placement"></a>Placement d’un connecteur

Le proxy d’application choisit l’emplacement des instances pour vous, en fonction de l’emplacement de votre locataire. Cependant, vous pouvez décider de l’emplacement auquel installer le connecteur, ce qui vous donne la possibilité de définir les caractéristiques de latence de votre trafic réseau.

Lorsque vous configurez le service de proxy d’application, posez-vous les questions suivantes :

- Où se trouve l’application ?
- Où se trouvent la plupart des utilisateurs qui accèdent à l’application ?
- Où se trouve l’instance de proxy d’application ?
- Disposez-vous déjà d’une connexion réseau dédiée vers les centres de données Azure comme Azure ExpressRoute ou un VPN similaire ?

Le connecteur doit communiquer avec Azure et vos applications (étapes 2 et 3 du schéma Flux de trafic). Le placement du connecteur affecte donc la latence de ces deux connexions. Lorsque vous évaluez le placement du connecteur, tenez compte des facteurs suivants :

- Si vous souhaitez utiliser la délégation Kerberos contrainte (KCD) pour l’authentification unique, le connecteur a besoin d’une ligne de vue sur un centre de données. En outre, le serveur de connecteur doit être joint à un domaine.  
- En cas de doute, rapprochez le connecteur de l’application.

### <a name="general-approach-to-minimize-latency"></a>Approche générale pour la réduction de la latence

Vous pouvez réduire la latence du trafic de bout en bout en optimisant chaque connexion réseau. Chaque connexion peut être optimisée en :

- Réduisant la distance entre les deux extrémités du tronçon.
- Choisissant le réseau approprié à parcourir. Par exemple, parcourir un réseau privé au lieu de l’Internet public peut être plus rapide en raison des liaisons dédiées.

Si vous disposez d’une liaison VPN ou ExpressRoute dédiée entre Azure et votre réseau d’entreprise, vous souhaiterez peut-être l’utiliser.

## <a name="focus-your-optimization-strategy"></a>Concentrer votre stratégie d’optimisation

Vous avez peu de latitude pour contrôler la connexion entre vos utilisateurs et le service du proxy d’application. Les utilisateurs peuvent accéder à vos applications à partir d’un réseau domestique, d’un café ou d’un autre pays/une autre région. Au lieu de cela, vous pouvez optimiser les connexions entre le service de proxy d’application et les connecteurs de proxy d’application vers les applications. Envisagez d’intégrer les modèles suivants dans votre environnement.

### <a name="pattern-1-put-the-connector-close-to-the-application"></a>Modèle 1 : Placer le connecteur à proximité de l’application

Placez le connecteur près de l’application cible dans le réseau du client. Cette configuration minimise l’étape 3 dans le schéma de topographie, car le connecteur et l’application sont proches.

Si votre connecteur a besoin d’une ligne de vue sur le contrôleur de domaine, ce modèle est avantageux. La plupart de nos clients utilisent ce modèle, car il fonctionne bien pour la majorité des scénarios. Ce modèle peut également être associé au modèle 2 pour optimiser le trafic entre le service et le connecteur.

### <a name="pattern-2-take-advantage-of-expressroute-with-microsoft-peering"></a>Modèle 2 : Tirer parti d’ExpressRoute avec peering Microsoft

Si vous avez une installation ExpressRoute avec peering Microsoft, vous pouvez utiliser la connexion ExpressRoute plus rapide pour le trafic entre le proxy d’application et le connecteur. Le connecteur se trouve toujours sur votre réseau, à proximité de l’application.

### <a name="pattern-3-take-advantage-of-expressroute-with-private-peering"></a>Modèle 3 : Tirer parti d’ExpressRoute avec peering privé

Si vous avez une installation VPN ou ExpressRoute dédiée avec le peering privé entre Azure et votre réseau d’entreprise, vous avez une autre option. Dans cette configuration, le réseau virtuel dans Azure est généralement considéré comme une extension du réseau de l’entreprise. Par conséquent, vous pouvez installer le connecteur dans le centre de données Azure tout en répondant aux exigences de faible latence de la connexion connecteur à application.

La latence n’est pas compromise, car le trafic circule sur une connexion dédiée. Vous obtenez également une meilleure latence entre le service de proxy d’application et le connecteur, car le connecteur est installé dans un centre de données Azure proche de l’emplacement de votre locataire Azure AD.

![Diagramme illustrant un connecteur installé dans un centre de données Azure](./media/application-proxy-network-topology/application-proxy-expressroute-private.png)

### <a name="other-approaches"></a>Autres approches

Bien que cet article ait pour objectif le placement du connecteur, vous pouvez également modifier le placement de l’application pour obtenir de meilleures caractéristiques de latence.

De plus en plus d’organisations déplacent leurs réseaux dans des environnements hébergés. Cela leur permet de placer leurs applications dans un environnement hébergé qui fait également partie de leur réseau d’entreprise tout en restant au sein du domaine. Dans ce cas, les modèles décrits dans les sections précédentes peuvent être appliqués au nouvel emplacement de l’application. Si vous envisagez cette option, consultez la page [Services de domaine Azure AD](../../active-directory-domain-services/overview.md).

Envisagez également d’organiser vos connecteurs à l’aide de [groupes de connecteurs](application-proxy-connector-groups.md) pour les applications cibles qui se trouvent dans des réseaux et des emplacements différents.

## <a name="common-use-cases"></a>Cas d’utilisation courants

Dans cette section, nous allons étudier quelques scénarios courants. Supposons que le locataire Azure AD (et par conséquent le point de terminaison du service de proxy) se trouve aux États-Unis. Les considérations présentées dans ces cas d’utilisation s’appliquent également à d’autres régions du monde.

Dans ces scénarios, nous appelons chaque connexion un « tronçon » et nous les numérotons dans un souci de simplification :

- **Tronçon 1** : Utilisateur vers le service de proxy d’application
- **Tronçon 2** : Service de proxy d’application vers le connecteur de proxy d’application
- **Tronçon 3** : Connecteur de proxy d’application vers l’application cible 

### <a name="use-case-1"></a>Cas d’utilisation 1

**Scénario :** L’application se trouve dans un réseau d’entreprise aux États-Unis avec des utilisateurs dans la même région. Aucun ExpressRoute ou VPN n’existe entre le centre de données Azure et le réseau d’entreprise.

**Recommandation :** Suivez le modèle 1, expliqué dans la section précédente. Pour une latence améliorée, envisagez l’utilisation d’ExpressRoute, si nécessaire.

Il s’agit d’un modèle simple. Vous optimisez le tronçon 3 en plaçant le connecteur à proximité de l’application. C’est également un choix naturel, étant donné que le connecteur est généralement installé avec la ligne de vue vers l’application et le centre de données pour effectuer des opérations de KCD.

![Diagramme montrant que les utilisateurs, le proxy, le connecteur et l’application se trouvent tous aux États-Unis](./media/application-proxy-network-topology/application-proxy-pattern1.png)

### <a name="use-case-2"></a>Cas d’utilisation 2

**Scénario :** L’application se trouve dans un réseau d’entreprise aux États-Unis avec des utilisateurs répartis dans le monde. Aucun ExpressRoute ou VPN n’existe entre le centre de données Azure et le réseau d’entreprise.

**Recommandation :** Suivez le modèle 1, expliqué dans la section précédente.

Là encore, le modèle courant consiste à optimiser le tronçon 3, où vous placez le connecteur à proximité de l’application. Le tronçon 3 n’est généralement pas coûteux, si le tout est contenu dans la même région. Toutefois, le tronçon 1 peut être plus coûteux selon l’emplacement de l’utilisateur, car les utilisateurs du monde entier doivent accéder à l’instance du proxy d’application aux États-Unis. Il est important de noter que toutes les solutions de proxy ont des caractéristiques similaires en ce qui concerne les utilisateurs répartis globalement.

![Les utilisateurs sont répartis dans le monde entier, mais tout le reste se trouve aux États-Unis](./media/application-proxy-network-topology/application-proxy-pattern2.png)

### <a name="use-case-3"></a>Cas d’utilisation 3

**Scénario :** L’application se trouve dans un réseau d’entreprise aux États-Unis. ExpressRoute avec peering Microsoft existe entre Azure et le réseau d’entreprise.

**Recommandation :** Suivez les modèles 1 et 2, comme expliqué dans la section précédente.

Tout d’abord, placez le connecteur aussi près que possible de l’application. Le système utilise automatiquement ExpressRoute pour le tronçon 2.

Si la liaison ExpressRoute utilise le peering Microsoft, le trafic entre le proxy et le connecteur suit cette liaison. Le tronçon 2 a une latence optimisée.

![Diagramme montrant ExpressRoute entre le proxy et le connecteur](./media/application-proxy-network-topology/application-proxy-pattern3.png)

### <a name="use-case-4"></a>Cas d’utilisation 4

**Scénario :** L’application se trouve dans un réseau d’entreprise aux États-Unis. ExpressRoute avec le peering privé existe entre Azure et le réseau d’entreprise.

**Recommandation :** Suivez le modèle 3, expliqué dans la section précédente.

Placez le connecteur dans le centre de données Azure qui est connecté au réseau d’entreprise par le biais du peering privé ExpressRoute.

Le connecteur peut être placé dans le centre de données Azure. Dans la mesure où le connecteur a toujours une ligne de vue à l’application et le centre de données via le réseau privé, le tronçon 3 reste optimisé. En outre, le tronçon 2 est davantage optimisé.

![Connecteur dans Azure Datacenter, ExpressRoute entre le connecteur et l’application](./media/application-proxy-network-topology/application-proxy-pattern4.png)

### <a name="use-case-5"></a>Cas d’utilisation 5

**Scénario :** L’application se trouve dans un réseau d’entreprise en Europe avec l’instance de proxy d’application et la plupart des utilisateurs aux États-Unis.

**Recommandation :** Placez le connecteur près de l’application. Étant donné que les utilisateurs des États-Unis ont accès à une instance de proxy d’application qui se trouve dans la même région, le tronçon 1 n’est pas trop onéreux. Le tronçon 3 est optimisé. Envisagez d’utiliser ExpressRoute pour optimiser le tronçon 2.

![Le diagramme montre les utilisateurs et le proxy aux États-Unis, le connecteur et l’application en Europe](./media/application-proxy-network-topology/application-proxy-pattern5b.png)

Vous pouvez également envisager d’utiliser une autre variante dans cette situation. Si la plupart des utilisateurs de l’organisation sont aux États-Unis, il est probable que votre réseau « s’étende » aussi vers les États-Unis. Placez le connecteur aux États-Unis et utilisez la ligne de réseau d’entreprise interne dédiée vers l’application en Europe. Les tronçons 2 et 3 sont ainsi optimisés.

![Le diagramme montre les utilisateurs, le proxy et le connecteur aux États-Unis, l’application en Europe](./media/application-proxy-network-topology/application-proxy-pattern5c.png)

## <a name="next-steps"></a>Étapes suivantes

- [Activer le proxy d’application](application-proxy-add-on-premises-application.md)
- [Activer l’authentification unique](application-proxy-configure-single-sign-on-with-kcd.md)
- [Activer l’accès conditionnel](application-proxy-integrate-with-sharepoint-server.md)
- [Résoudre les problèmes rencontrés avec le proxy d’application](application-proxy-troubleshoot.md)
