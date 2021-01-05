---
title: Présentation d’Azure AD Connect et Connect Health | Microsoft Docs
description: Découvrez les outils utilisés pour synchroniser et superviser votre environnement local avec Azure AD.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: overview
ms.date: 01/08/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 74a6c816979732e21813a0f2024167c68ca1c5dc
ms.sourcegitcommit: 642988f1ac17cfd7a72ad38ce38ed7a5c2926b6c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94874098"
---
# <a name="what-is-azure-ad-connect"></a>Qu’est-ce qu’Azure AD Connect ?

L’outil Microsoft Azure AD Connect a été conçu pour vous permettre d’atteindre et de remplir vos objectifs en matière d’identité hybride.  Elle fournit les fonctionnalités suivantes :
     
- [Synchronisation de hachage de mot de passe](whatis-phs.md) : méthode d’authentification qui synchronise un hachage du mot de passe AD local d’un utilisateur avec Azure AD.
- [Authentification directe](how-to-connect-pta.md) : méthode d’authentification qui permet aux utilisateurs d’utiliser le même mot de passe localement et dans le cloud, mais sans nécessiter l’infrastructure supplémentaire d’un environnement fédéré.
- [Intégration de fédération](how-to-connect-fed-whatis.md) : la fédération est une partie facultative d’Azure AD Connect qui peut servir à configurer un environnement hybride à l’aide d’une infrastructure AD FS locale. Elle offre également des fonctionnalités de gestion AD FS telles que le renouvellement de certificat et les déploiements de serveurs AD FS supplémentaires.
- [Synchronisation](how-to-connect-sync-whatis.md) : ce composant est chargé de créer des utilisateurs, des groupes et d’autres objets,  et également de s’assurer que les informations d’identité relatives aux utilisateurs et aux groupes dans votre environnement local correspondent à celles qui se trouvent dans le cloud.  Cette synchronisation inclut également des hachages de mot de passe.
- [Analyse du fonctionnement]() : Azure AD Connect Health peut assurer une supervision robuste et offrir un emplacement central dans le Portail Azure pour la visualisation de cette activité. 


![Qu’est-ce qu’Azure AD Connect ?](./media/whatis-hybrid-identity/arch.png)



## <a name="what-is-azure-ad-connect-health"></a>Qu’est-ce qu’Azure AD Connect Health ?

Azure Active Directory (Azure AD) Connect Health fournit une supervision robuste de votre infrastructure d’identité locale. Il vous permet de conserver une connexion fiable à Microsoft 365 et aux services en ligne Microsoft.  Cette fiabilité est obtenue en fournissant des fonctionnalités de supervision pour vos composants d’identités clés. De plus, il rend les points de données clés relatifs à ces composants facilement accessibles.

Ces informations sont toutes présentées dans le [portail Azure AD Connect Health](https://aka.ms/aadconnecthealth). Utilisez le portail Azure AD Connect Health pour voir les alertes, la supervision des performances, l’analytique des utilisations et d’autres informations. Azure AD Connect Health prend en compte l’intégrité des composants d’identité clé, le tout à un seul endroit.

![Présentation d’Azure AD Connect Health](./media/whatis-hybrid-identity-health/aadconnecthealth2.png)

## <a name="why-use-azure-ad-connect"></a>Pourquoi utiliser Azure AD Connect ?
L’intégration de vos annuaires locaux avec Azure AD améliore la productivité de vos utilisateurs en leur fournissant une identité commune pour accéder aux ressources cloud et locales. Les utilisateurs et les organisations bénéficient des avantages suivants :

* Les utilisateurs peuvent utiliser une identité unique pour accéder aux applications locales et aux services cloud comme Microsoft 365.
* Outil unique offrant une expérience de déploiement simple pour la synchronisation et la connexion.
* Fournit les fonctionnalités les plus récentes pour vos scénarios. Azure AD Connect remplace les versions antérieures des outils d’intégration d’identité tels que DirSync et Azure AD Sync. Pour plus d’informations, consultez [Identité hybride : Comparaison des outils d’intégration d’annuaire](plan-hybrid-identity-design-considerations-tools-comparison.md).

## <a name="why-use-azure-ad-connect-health"></a>Pourquoi utiliser Azure AD Connect Health ?
Avec Azure AD, vos utilisateurs gagnent en productivité car ils disposent d’une identité commune pour accéder aux ressources cloud et locales. Garantir la fiabilité de l’environnement afin que les utilisateurs puissent accéder à ces ressources devient un défi.  Azure AD Connect Health vous aide à superviser et à obtenir des insights concernant votre infrastructure d’identité locale, garantissant ainsi la fiabilité de cet environnement. Son installation est aussi simple que celle d’un agent sur chacun de vos serveurs d’identité local.

Azure AD Connect Health pour AD FS prend en charge AD FS 2.0 sur Windows Server 2008 R2, Windows Server 2012, Windows Server 2012 R2 et Windows Server 2016. Cette prise en charge inclut également la surveillance de tous les serveurs proxy AD FS ou serveurs proxy d’application web qui prennent en charge l’authentification pour l’accès extranet. Avec une installation simple et rapide du programme Health Agent, Azure AD Connect Health pour AD FS vous offre un ensemble de fonctionnalités clés.

Avantages clés et bonnes pratiques :

|Principaux avantages|Bonnes pratiques|
|-----|-----|
|Sécurité améliorée|[Tendances de verrouillage extranet](how-to-connect-health-adfs.md#usage-analytics-for-ad-fs)</br>[Rapport sur les échecs de connexions](how-to-connect-health-adfs-risky-ip.md)</br>[Conformité aux réglementations relatives à la confidentialité](reference-connect-health-user-privacy.md)|
|Obtention d’alertes pour tous les [problèmes critiques liés au système ADFS](how-to-connect-health-alert-catalog.md#alerts-for-active-directory-federation-services)|Configuration et disponibilité de serveur</br>[Performances et connectivité](how-to-connect-health-adfs.md#performance-monitoring-for-ad-fs)</br>Maintenance régulière|
|Facilité de déploiement et de gestion|[Installation d’agent rapide](how-to-connect-health-agent-install.md#install-the-agent-for-ad-fs)</br>Mise à niveau automatique d’agent vers la dernière version</br>Données disponibles dans le portail en quelques minutes|
[Métriques d’utilisation](how-to-connect-health-adfs.md#usage-analytics-for-ad-fs) enrichies|Utilisation des principales applications</br>Emplacements réseau et connexion TCP</br>Demandes de jetons par serveur|
|Meilleure expérience utilisateur|Présentation sous forme de tableaux de bord du Portail Azure</br>[Alertes par e-mail](how-to-connect-health-adfs.md#alerts-for-ad-fs)|


## <a name="license-requirements-for-using-azure-ad-connect"></a>Licences requises pour Azure AD Connect

[!INCLUDE [active-directory-free-license.md](../../../includes/active-directory-free-license.md)]

## <a name="license-requirements-for-using-azure-ad-connect-health"></a>Licences requises pour Azure AD Connect Health
[!INCLUDE [active-directory-free-license.md](../../../includes/active-directory-p1-license.md)]

## <a name="next-steps"></a>Étapes suivantes

- [Matériel et conditions préalables](how-to-connect-install-prerequisites.md) 
- [Paramètres Express](how-to-connect-install-express.md)
- [Paramètres personnalisés](how-to-connect-install-custom.md)
- [Installation des agents Azure AD Connect Health](how-to-connect-health-agent-install.md)