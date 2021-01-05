---
title: Haute disponibilité et équilibrage de charge – Proxy d’application Azure AD
description: Fonctionnement de la distribution du trafic avec votre déploiement de proxy d’application. Contient des conseils pour optimiser les performances du connecteur et utiliser l’équilibrage de charge pour les serveurs principaux.
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
ms.date: 10/08/2019
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 403fa4cab94ad6149e388b10acccd9d5e7a2b7a8
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658160"
---
# <a name="high-availability-and-load-balancing-of-your-application-proxy-connectors-and-applications"></a>Haute disponibilité et équilibrage de charge de vos applications et connecteurs de proxy d’application

Cet article explique le fonctionnement de la distribution du trafic avec votre déploiement de proxy d’application. Nous aborderons les points suivants :

- La façon dont le trafic est distribué entre les utilisateurs et les connecteurs, avec des conseils pour optimiser les performances du connecteur

- La façon dont le trafic circule entre les connecteurs et les serveurs d’applications principaux, avec des recommandations pour l’équilibrage de charge entre plusieurs serveurs principaux

## <a name="traffic-distribution-across-connectors"></a>Distribution du trafic entre les connecteurs

Les connecteurs établissent leurs connexions en fonction des principes de haute disponibilité. Il n’y a aucune garantie que le trafic sera toujours réparti uniformément entre les connecteurs et qu’il n’y a pas d’affinité de session. Toutefois, l’utilisation varie et les requêtes sont envoyées de manière aléatoire aux instances du service de proxy d’application. Par conséquent, le trafic est généralement distribué presque uniformément entre les connecteurs. Le diagramme et les étapes ci-dessous illustrent la façon dont les connexions sont établies entre les utilisateurs et les connecteurs.

![Diagramme montrant les connexions entre les utilisateurs et les connecteurs](media/application-proxy-high-availability-load-balancing/application-proxy-connections.png)

1. Un utilisateur sur un appareil client tente d’accéder à une application locale publiée par le biais du proxy d’application.
2. La requête passe par un Azure Load Balancer pour déterminer quelle instance de service de proxy d’application doit la prendre en charge. Par région, des dizaines d’instances sont disponibles pour accepter la requête. Cette méthode permet de répartir uniformément le trafic entre les instances de service.
3. La requête est envoyée à [Service Bus](../../service-bus-messaging/index.yml).
4. Service Bus signale à un connecteur disponible. Le connecteur récupère ensuite la requête à partir de Service Bus.
   - À l’étape 2, les requêtes sont dirigées vers différentes instances du service de proxy d’application, de sorte que les connexions sont plus susceptibles d’être effectuées avec différents connecteurs. Par conséquent, les connecteurs sont presque uniformément utilisés au sein du groupe.
5. Le connecteur transmet la requête au serveur principal de l’application. L’application renvoie ensuite la réponse au connecteur.
6. Le connecteur termine la réponse en ouvrant une connexion sortante vers l’instance de service à partir de laquelle la requête a été effectuée. Cette connexion est alors immédiatement fermée. Par défaut, chaque connecteur est limité à 200 connexions sortantes simultanées.
7. La réponse est ensuite retransmise au client à partir de l’instance de service.
8. Les requêtes suivantes émanant de la même connexion répètent les étapes ci-dessus.

Une application a souvent de nombreuses ressources et ouvre plusieurs connexions lorsqu’elle est chargée. Chaque connexion passe par les étapes ci-dessus pour être allouée à une instance de service. Sélectionnez un nouveau connecteur disponible si la connexion n’a pas encore été associée à un connecteur.


## <a name="best-practices-for-high-availability-of-connectors"></a>Meilleures pratiques pour la haute disponibilité des connecteurs

- En raison de la façon dont le trafic est distribué entre les connecteurs pour la haute disponibilité, il est essentiel de toujours avoir au moins deux connecteurs dans un groupe de connecteurs. Trois connecteurs sont préférables pour fournir une mémoire tampon supplémentaire parmi les connecteurs. Pour déterminer le nombre correct de connecteurs dont vous avez besoin, suivez la documentation sur la planification de la capacité.

- Placez les connecteurs sur des connexions sortantes différentes pour éviter un point de défaillance unique. Si les connecteurs utilisent la même connexion sortante, un problème réseau avec la connexion peut avoir un impact sur tous les connecteurs qui l’utilisent.

- Évitez de forcer le redémarrage des connecteurs quand ils sont connectés à des applications de production. Cela peut avoir un impact négatif sur la distribution du trafic entre les connecteurs. Le redémarrage des connecteurs entraîne l’indisponibilité de connecteurs supplémentaires et force les connexions au connecteur restant disponible. Le résultat est une utilisation inégale des connecteurs initialement.

- Évitez toute forme d’inspection inline sur les communications TLS sortantes établies entre les connecteurs et Azure. Ce type d’inspection inline entraîne une dégradation du flux de communication.

- Veillez à ce que les mises à jour automatiques soient en cours d’exécution pour vos connecteurs. Si le service Connector Updater du proxy d’application est en cours d’exécution, vos connecteurs se mettent à jour automatiquement et reçoivent la dernière mise à niveau. Si vous ne voyez pas le service Connector Updater sur votre serveur, vous devez réinstaller votre connecteur afin d’obtenir les mises à jour.

## <a name="traffic-flow-between-connectors-and-back-end-application-servers"></a>Flux de trafic entre les connecteurs et les serveurs d’applications principaux

Une autre zone clé dans laquelle la haute disponibilité est un facteur est la connexion entre les connecteurs et les serveurs principaux. Lorsqu’une application est publiée via le Proxy d'application Azure Active Directory, le trafic des utilisateurs vers les applications circule via trois tronçons :

1. L’utilisateur se connecte au point de terminaison public du service de Proxy d'application Azure Active Directory sur Azure. La connexion est établie entre l’adresse IP de client d’origine (public) du client et l’adresse IP du point de terminaison du proxy d’application.
2. Le connecteur du proxy d’application extrait la requête HTTP du client à partir du service de proxy d’application.
3. Le connecteur du proxy d’application se connecte à l’application cible. Le connecteur utilise sa propre adresse IP pour établir la connexion.

![Diagramme de l’utilisateur qui se connecte à une application via le proxy d’application](media/application-proxy-high-availability-load-balancing/application-proxy-three-hops.png)

### <a name="x-forwarded-for-header-field-considerations"></a>Considérations liées au champ d’en-tête X-Forwarded-For
Dans certaines situations (telles que l’audit, l’équilibrage de charge, etc.), il est obligatoire de partager l’adresse IP d’origine du client externe avec l’environnement local. Pour répondre à cette exigence, le connecteur de Proxy d'application Azure Active Directory ajoute le champ d’en-tête X-Forwarded-For avec l’adresse IP du client d’origine (public) à la requête HTTP. Le périphérique réseau approprié (équilibreur de charge, pare-feu) ou le serveur Web ou l’application principale peut ensuite lire et utiliser les informations.

## <a name="best-practices-for-load-balancing-among-multiple-app-servers"></a>Meilleures pratiques pour l’équilibrage de charge entre plusieurs serveurs d’applications
Lorsque le groupe de connecteurs affecté à l’application de proxy d’application possède au moins deux connecteurs, et que vous exécutez l’application Web principale sur plusieurs serveurs (batterie de serveurs), une bonne stratégie d’équilibrage de charge est nécessaire. Une bonne stratégie garantit que les serveurs récupèrent les requêtes des clients uniformément et empêche l’utilisation excessive ou insuffisante des serveurs dans la batterie de serveurs.
### <a name="scenario-1-back-end-application-does-not-require-session-persistence"></a>Scénario 1 : L’application principale ne nécessite pas de persistance de session
Le scénario le plus simple est l’endroit où l’application Web principale ne nécessite pas d’adhérence de session (persistance de session). Toute requête de l’utilisateur peut être gérée par n’importe quelle instance d’application principale dans la batterie de serveurs. Vous pouvez utiliser un équilibreur de charge de couche 4 et le configurer sans affinité. Certaines options incluent l’équilibrage de charge réseau Microsoft et Azure Load Balancer ou un équilibrage de charge d’un autre fournisseur. Vous pouvez également configurer DNS avec tourniquet (round robin).
### <a name="scenario-2-back-end-application-requires-session-persistence"></a>Scénario 2 : L’application principale requiert une persistance de session
Dans ce scénario, l’application web principale requiert l’adhérence de session (persistance de session) au cours de la session authentifiée. Toutes les requêtes de l’utilisateur doivent être gérées par l’instance de l’application principale qui s’exécute sur le même serveur dans la batterie de serveurs.
Ce scénario peut être plus compliqué, car le client établit généralement plusieurs connexions au service de proxy d’application. Les requêtes sur différentes connexions peuvent arriver sur des connecteurs et des serveurs différents dans la batterie. Étant donné que chaque connecteur utilise sa propre adresse IP pour cette communication, l’équilibreur de charge ne peut pas garantir l’adhérence de session en fonction de l’adresse IP des connecteurs. L’affinité IP source ne peut pas être utilisée non plus.
Voici quelques options pour le scénario 2 :

- Option 1 : Basez la persistance de session sur un cookie de session défini par l’équilibreur de charge. Cette option est recommandée car elle permet de répartir la charge de manière plus uniforme entre les serveurs principaux. Elle nécessite un équilibreur de charge de couche 7 avec cette fonctionnalité, capable de gérer le trafic HTTP et de mettre fin à la connexion TLS. Vous pouvez utiliser Azure Application Gateway (affinité de session) ou un équilibreur de charge d’un autre fournisseur.

- Option n°2 : Basez la persistance de session sur le champ d’en-tête X-Forwarded-For. Elle nécessite un équilibreur de charge de couche 7 avec cette fonctionnalité, capable de gérer le trafic HTTP et de mettre fin à la connexion TLS.  

- Option 3 : Configurez l’application principale pour qu’elle ne nécessite pas de persistance de session.

Reportez-vous à la documentation de votre fournisseur de logiciels pour comprendre les exigences en matière d’équilibrage de charge de l’application principale.

## <a name="next-steps"></a>Étapes suivantes

- [Activer le proxy d’application](application-proxy-add-on-premises-application.md)
- [Activer l’authentification unique](application-proxy-configure-single-sign-on-with-kcd.md)
- [Activer l’accès conditionnel](application-proxy-integrate-with-sharepoint-server.md)
- [Résoudre les problèmes rencontrés avec le proxy d’application](application-proxy-troubleshoot.md)
- [Découvrir comment l’architecture Azure AD prend en charge la haute disponibilité](../fundamentals/active-directory-architecture.md)