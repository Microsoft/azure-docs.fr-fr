---
title: Publier des applications sur des réseaux distincts via des groupes de connecteurs - Azure AD
description: Explique comment créer et gérer des groupes de connecteurs dans le proxy d’Application Azure AD.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 11/08/2018
ms.author: kenwith
ms.reviewer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: be3e99ca57957e1975313fed8609533f4a65b102
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84764721"
---
# <a name="publish-applications-on-separate-networks-and-locations-using-connector-groups"></a>Publier des applications sur des réseaux et emplacements distincts à l’aide de groupes de connecteurs

Les clients utilisent le proxy d’application Azure AD pour obtenir toujours plus de scénarios et d’applications. C’est pourquoi nous avons rendu le proxy d’application encore plus flexible en activant davantage de topologies. Vous pouvez créer des groupes de connecteurs de proxy d’application pour attribuer des connecteurs spécifiques permettant de servir des applications spécifiques. Cette fonctionnalité vous offre davantage de contrôle et de méthodes pour optimiser votre déploiement de proxy d’application.

Chaque connecteur de proxy d’application est attribué à un groupe de connecteurs. Tous les connecteurs qui appartiennent au même groupe de connecteurs agissent comme une unité distincte pour la haute disponibilité et l’équilibrage de charge. Tous les connecteurs appartiennent à un groupe de connecteurs. Si vous ne créez pas de groupes, tous vos connecteurs se trouvent dans un groupe par défaut. Votre administrateur peut créer des groupes et leur attribuer des connecteurs dans le portail Azure.

Toutes les applications sont affectées à un groupe de connecteurs. Si vous ne créez pas de groupes, toutes vos applications sont affectées à un groupe par défaut. Mais si vous organisez vos connecteurs en groupes, vous pouvez définir chaque application de façon à ce qu’elle fonctionne avec un groupe de connecteurs spécifique. Dans ce cas, seuls les connecteurs de ce groupe servent l’application sur demande. Cette fonctionnalité s’avère utile si vos applications sont hébergées à des emplacements différents. Vous pouvez créer des groupes de connecteurs en fonction de l’emplacement, afin que les applications soient toujours servies par les connecteurs qui leur sont physiquement proches.

> [!TIP]
> Si vous avez un grand déploiement de proxy d’application, n’affectez aucune application au groupe de connecteurs par défaut. Ainsi, les nouveaux connecteurs ne reçoivent pas de trafic live tant que vous ne les affectez pas à un groupe de connecteurs actif. Cette configuration vous permet également de passer des connecteurs en mode inactif en les réintégrant au groupe par défaut, pour que vous puissiez en effectuer la maintenance sans impacter vos utilisateurs.

## <a name="prerequisites"></a>Prérequis

Pour regrouper vos connecteurs, vous devez vous assurer que vous avez [installé plusieurs connecteurs](application-proxy-add-on-premises-application.md). Lorsque vous installez un nouveau connecteur, il est automatiquement ajouté au groupe de connecteurs **par défaut**.

## <a name="create-connector-groups"></a>Créer des groupes de connecteurs

Procédez comme suit pour créer autant de groupes de connecteurs que vous le souhaitez.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez **Azure Active Directory** > **Applications d’entreprise** > **Proxy d’application**.
1. Sélectionnez **Nouveau groupe de connecteurs**. Le panneau Nouveau groupe de connecteurs s’affiche.

   ![Il comporte l’écran permettant de sélectionner un nouveau groupe de connecteurs](./media/application-proxy-connector-groups/new-group.png)

1. Donnez un nom à votre nouveau groupe de connecteurs, puis utilisez le menu déroulant pour sélectionner les connecteurs qui appartiennent à ce groupe.
1. Sélectionnez **Enregistrer**.

## <a name="assign-applications-to-your-connector-groups"></a>Affecter des applications à vos groupes de connecteurs

Procédez comme suit pour chaque application que vous avez publiée avec le proxy d’application. Vous pouvez affecter une application à un groupe de connecteurs lors de sa première publication, ou bien vous pouvez utiliser ces étapes pour modifier l’affectation à tout moment.

1. Dans le tableau de bord de gestion de votre répertoire, sélectionnez **Applications d’entreprise** > **Toutes les applications** > l’application que vous souhaitez affecter à un groupe de connecteurs > **Proxy d’application**.
1. Utilisez le menu déroulant **Groupe de connecteurs** pour sélectionner le groupe que l’application doit utiliser.
1. Sélectionnez **Enregistrer** pour appliquer la modification.

## <a name="use-cases-for-connector-groups"></a>Cas d’utilisation des groupes de connecteurs

Les groupes de connecteurs sont utiles dans divers scénarios, notamment les suivants :

### <a name="sites-with-multiple-interconnected-datacenters"></a>Sites contenant plusieurs centres de données interconnectés

De nombreuses organisations possèdent plusieurs centres de données interconnectés. Dans ce cas, vous souhaitez conserver autant de trafic que possible au sein du centre de données, car les liaisons entre les centres de données sont coûteuses et lentes. Vous pouvez déployer des connecteurs dans chaque centre de données pour traiter uniquement les applications contenues dans le centre de données. Cette approche réduit au minimum les liaisons entre centres de données et fournit à vos utilisateurs une expérience entièrement transparente.

### <a name="applications-installed-on-isolated-networks"></a>Applications installées sur des réseaux isolés

Les applications peuvent être hébergées sur des réseaux qui ne font pas partie du réseau d’entreprise principal. Vous pouvez utiliser des groupes de connecteurs pour installer des connecteurs dédiés sur des réseaux isolés afin d’isoler les applications sur le réseau. Cela se produit généralement lorsqu’un fournisseur tiers gère une application spécifique pour votre organisation.

Les groupes de connecteurs vous permettent d’installer des connecteurs dédiés aux réseaux qui publient uniquement des applications spécifiques, ce qui permet de sous-traiter la gestion des applications à des fournisseurs tiers plus facilement et en toute sécurité.

### <a name="applications-installed-on-iaas"></a>Applications installées sur IaaS

Pour les applications installées sur IaaS pour l’accès au cloud, les groupes de connecteurs fournissent un service commun pour sécuriser l’accès à toutes les applications. Les groupes de connecteurs ne créent pas de dépendances supplémentaires sur votre réseau d’entreprise, et ne fragmentent pas l’expérience de l’application. Les connecteurs peuvent être installés sur chaque centre de données du cloud et servir uniquement les applications qui se trouvent sur ce réseau. Vous pouvez installer plusieurs connecteurs pour atteindre une haute disponibilité.

Prenons par exemple une organisation qui a plusieurs machines virtuelles connectées à son propre réseau virtuel IaaS hébergé. Pour permettre aux employés d’utiliser ces applications, ces réseaux privés sont connectés au réseau d’entreprise via un réseau VPN de site à site. Les employés qui se trouvent sur site bénéficient ainsi d’une bonne expérience. Toutefois, cela peut ne pas être idéal pour les employés à distance, car cela nécessite une infrastructure locale supplémentaire pour acheminer l’accès, comme vous pouvez le voir dans le schéma ci-dessous :

![Diagramme illustrant le réseau Azure AD IaaS](./media/application-proxy-connector-groups/application-proxy-iaas-network.png)
  
Avec les groupes de connecteurs de proxy d’application Azure AD, vous pouvez activer un service commun pour sécuriser l’accès à toutes les applications sans créer de dépendance supplémentaire sur votre réseau d’entreprise :

![Fournisseurs de cloud multiples IaaS Azure AD](./media/application-proxy-connector-groups/application-proxy-multiple-cloud-vendors.png)

### <a name="multi-forest--different-connector-groups-for-each-forest"></a>Forêts multiples : groupes de connecteurs différents pour chaque forêt

La plupart des clients qui ont déployé le proxy d’application utilisent ses fonctionnalités d’authentification unique (SSO) par le biais de la délégation Kerberos contrainte (KCD). Pour ce faire, les machines du connecteur doivent être jointes à un domaine qui peut déléguer les utilisateurs vers l’application. KCD prend en charge les fonctionnalités inter-forêts. Mais pour les entreprises qui disposent d’environnements distincts à plusieurs forêts sans approbation entre eux, il est impossible d’utiliser un connecteur unique pour toutes les forêts. 

Dans ce cas, des connecteurs spécifiques peuvent être déployés par forêt et définis pour traiter les applications qui ont été publiées pour traiter uniquement les utilisateurs de cette forêt spécifique. Chaque groupe de connecteurs représente une forêt différente. Bien que le client et la majorité de l’expérience soient unifiés pour toutes les forêts, les utilisateurs peuvent être affectés à leurs applications de forêt à l’aide de groupes Azure AD.

### <a name="disaster-recovery-sites"></a>Sites de récupération d’urgence

Il existe deux approches différentes que vous pouvez utiliser avec un site de récupération d’urgence, en fonction de l’implémentation de vos sites :

* Si votre site de récupération d’urgence est créé en mode actif - actif où il est identique au site principal et dispose des mêmes paramètres de mise en réseau et AD, vous pouvez créer les connecteurs sur le site de récupération d’urgence dans le même groupe de connecteurs que le site principal. Cela permet à Azure AD de détecter les basculements.
* Si votre site de récupération d’urgence est distinct du site principal, vous pouvez créer un groupe de connecteurs différent dans le site de récupération d’urgence et soit 1) avoir des applications de sauvegarde, soit 2) diriger manuellement l’application existante vers le groupe de connecteurs de récupération d’urgence si nécessaire.

### <a name="serve-multiple-companies-from-a-single-tenant"></a>Traiter plusieurs entreprises à partir d’un seul client

Il existe différentes manières d’implémenter un modèle dans lequel un fournisseur de services unique déploie et gère les services relatifs à Azure AD pour plusieurs entreprises. Les groupes de connecteurs aident l’administrateur à séparer les connecteurs et les applications en différents groupes. Une méthode, qui est appropriée pour les petites entreprises, est d’avoir un seul client Azure AD tandis que les différentes entreprises ont leurs propres nom de domaine et réseaux. Cela vaut également pour les scénarios et les situations M&A où une division informatique unique sert plusieurs entreprises pour des raisons réglementaires ou professionnelles.

## <a name="sample-configurations"></a>Exemples de configurations

Certains exemples que vous pouvez implémenter incluent les groupes de connecteurs suivants.

### <a name="default-configuration--no-use-for-connector-groups"></a>Configuration par défaut : inutile pour les groupes de connecteurs

Si vous n’utilisez pas les groupes de connecteurs, votre configuration ressemblerait à ceci :

![Exemple : AzureAD Aucun groupe de connecteurs](./media/application-proxy-connector-groups/application-proxy-sample-config-1.png)

Cette configuration est suffisante pour les tests et les petits déploiements. Elle est également adaptée si votre organisation dispose d’une topologie de réseau à plat.

### <a name="default-configuration-and-an-isolated-network"></a>Configuration par défaut et réseau isolé

Cette configuration est une évolution de la configuration par défaut, dans laquelle une application spécifique s’exécute sur un réseau isolé, comme un réseau virtuel IaaS :

![Exemple : AzureAD Aucun groupe de connecteurs et réseau isolé](./media/application-proxy-connector-groups/application-proxy-sample-config-2.png)

### <a name="recommended-configuration--several-specific-groups-and-a-default-group-for-idle"></a>Configuration recommandée : plusieurs groupes spécifiques et un groupe par défaut inactif

La configuration recommandée pour les organisations de grande taille et complexes consiste à disposer du groupe de connecteurs par défaut en tant que groupe qui ne traite aucune application et qui est utilisé pour les connecteurs inactifs ou nouvellement installés. Toutes les applications sont traitées à l’aide de groupes de connecteurs personnalisés. Ainsi, toute la complexité des scénarios décrits ci-dessus est permise.

Dans l’exemple ci-dessous, l’entreprise a deux centres de données, A et B, avec deux connecteurs qui servent chaque site. Des applications différentes s’exécutent sur chaque site.

![Exemple d’une société avec 2 centres de données et 2 connecteurs](./media/application-proxy-connector-groups/application-proxy-sample-config-3.png)

## <a name="next-steps"></a>Étapes suivantes

* [Présentation des connecteurs de proxy d’application Azure AD](application-proxy-connectors.md)
* [Activer l’authentification unique](what-is-single-sign-on.md)
