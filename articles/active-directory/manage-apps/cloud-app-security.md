---
title: Visibilité et contrôle des applications avec Microsoft Cloud App Security
description: Découvrez des façons d’identifier les niveaux de risque des applications, stopper les violations et les fuites en temps réel et utiliser des connecteurs d’application pour tirer parti d’API de fournisseurs favorisant la visibilité et la gouvernance.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 02/03/2020
ms.author: kenwith
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8e3e0c10a7ec6b42420db30955ae4911bca27cc0
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94651224"
---
# <a name="cloud-app-visibility-and-control"></a>Visibilité et contrôle des applications cloud

Pour tirer pleinement parti des applications et des services cloud, une équipe informatique doit trouver le bon équilibre entre faciliter l’accès et garder le contrôle de façon à protéger les données critiques. Microsoft Cloud App Security offre une visibilité complète, un contrôle sur le déplacement des données et des capacités analytiques sophistiquées pour identifier et combattre les cybermenaces dans l’ensemble de vos services Microsoft et tiers.

## <a name="discover-and-manage-shadow-it-in-your-network"></a>Découvrir et gérer le Shadow IT dans votre réseau

Quand on demande aux administrateurs informatiques quel est, selon eux, le nombre d’applications utilisées par les employés, ils répondent généralement 30 ou 40. Or, dans la réalité, les employés de votre organisation utilisent en moyenne plus de 1 000 applications distinctes. Le Shadow IT vous aide à déterminer et identifier les applications qui sont utilisées ainsi que votre niveau de risque. Quatre-vingt pour cent des employés utilisent des applications non approuvées que personne n’a examinées et qui peuvent ne pas être conformes à vos stratégies de sécurité et de conformité. Et comme vos employés peuvent accéder à vos ressources et applications à l’extérieur de votre réseau d’entreprise, définir des règles et des stratégies sur vos pare-feu ne suffit plus.

Microsoft Cloud App Discovery (fonctionnalité Azure Active Directory Premium P1) permet de découvrir les applications qui sont utilisées, d’explorer les risques associés à ces applications, de configurer des stratégies pour identifier de nouvelles applications à risque et de désapprouver ces applications de façon à les bloquer en mode natif à l’aide de votre appliance de pare-feu ou proxy.

- Découvrir et identifier le Shadow IT
- Évaluer et analyser
- Gérer vos applications
- Génération de rapports de découverte Shadow IT avancée
- Contrôler les applications approuvées
 
### <a name="learn-more"></a>En savoir plus

- [Découvrir et gérer le Shadow IT dans votre réseau](/cloud-app-security/tutorial-shadow-it)
- [Applications découvertes avec Cloud App Security](/cloud-app-security/discovered-apps)
 
## <a name="user-session-visibility-and-control"></a>Visibilité et contrôle de session utilisateur 

Dans l’espace de travail d’aujourd’hui, il ne suffit pas de savoir ce qu’il se passe dans votre environnement cloud après que les faits se sont produits. Les violations et les fuites doivent être arrêtées en temps réel avant que les employés exposent de façon intentionnelle ou involontaire les données et l’organisation à des risques. De concert avec Azure Active Directory (Azure AD), Microsoft Cloud App Security permet cela à travers une expérience globale et intégrée au moyen du contrôle d’application par accès conditionnel. 

Le contrôle de session utilise une architecture de proxy inverse et est intégré de manière unique à l’accès conditionnel Azure AD. L’accès conditionnel Azure AD vous permet d’appliquer des contrôles d’accès aux applications de votre organisation en fonction de certaines conditions. Les conditions déterminent à qui (utilisateur ou groupe d’utilisateurs), à quoi (quelles applications cloud) et où (quels emplacements et réseaux) s’applique une stratégie d’accès conditionnel. Une fois que vous avez défini les conditions, vous pouvez router les utilisateurs vers Cloud App Security où vous pouvez protéger les données en temps réel.  

Ce contrôle vous offre les possibilités suivantes :  
- Contrôler les téléchargements de fichiers
- Superviser les scénarios B2B  
- Contrôler l’accès aux fichiers  
- Protéger les documents pour le téléchargement  
 
### <a name="learn-more"></a>En savoir plus

- [Protéger les applications avec le contrôle de session dans Cloud App Security ](/cloud-app-security/proxy-intro-aad)
 
## <a name="advanced-app-visibility-and-controls"></a>Visibilité et contrôle avancés des applications 

Les connecteurs d’applications utilisent les API de fournisseurs d’applications pour que Microsoft Cloud App Security bénéficie d’une plus grande visibilité et d’un plus grand contrôle sur les applications auxquelles vous vous connectez. Cloud App Security exploite les API fournies par le fournisseur de cloud. L’infrastructure et les API propres à chaque service ont des limitations, notamment en matière de bande passante, de limites d’API, de fenêtres d’API de décalage temporel dynamique, etc. L’équipe du produit Cloud App Security a collaboré avec les fournisseurs de ces services pour optimiser l’utilisation des API et assurer le meilleur niveau de performance. En prenant en compte les différentes limitations que les services imposent à leurs API, les moteurs Cloud App Security utilisent leur capacité maximale autorisée. Étant donné que certaines opérations, comme l’analyse de l’ensemble des fichiers d’un locataire, exigent de nombreux appels d’API, elles sont étendues sur une plus longue période. Plusieurs heures ou jours peuvent être nécessaires à l’exécution de certaines stratégies. 
 
### <a name="learn-more"></a>En savoir plus  

- [Connecter les applications dans Cloud App Security](/cloud-app-security/enable-instant-visibility-protection-and-governance-actions-for-your-apps)

## <a name="next-steps"></a>Étapes suivantes

- [Découvrir et gérer le Shadow IT dans votre réseau](/cloud-app-security/tutorial-shadow-it)
- [Applications découvertes avec Cloud App Security](/cloud-app-security/discovered-apps)
- [Protéger les applications avec le contrôle de session dans Cloud App Security ](/cloud-app-security/proxy-intro-aad)
- [Connecter les applications dans Cloud App Security](/cloud-app-security/enable-instant-visibility-protection-and-governance-actions-for-your-apps)