---
title: Présentation d’Azure Active Directory Identity Protection
description: Détectez, corrigez, examinez et analysez les risques avec Azure AD Identity Protection
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: overview
ms.date: 08/24/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.custom: contperf-fy21q1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 242e0e4614994c30d0a14b8fe3d7a5c2b217bb5a
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033338"
---
# <a name="what-is-identity-protection"></a>Qu’est-ce qu’Identity Protection ?

Identity Protection est un outil qui permet aux organisations d'accomplir trois tâches principales :

- Automatiser la détection et la correction des risques liés à l'identité
- Examiner les risques à l'aide des données disponibles sur le portail
- Exporter les données de détection des risques vers des utilitaires tiers pour une analyse plus approfondie

Pour protéger vos utilisateurs, Identity Protection s'appuie sur les connaissances acquises par Microsoft au niveau des organisations avec Azure AD, de l'espace grand public avec les comptes Microsoft et des jeux avec Xbox. Microsoft analyse 6 500 milliards de signaux par jour pour identifier les menaces et protéger les clients.

Les signaux générés par Identity Protection et transmis à celui-ci peuvent également être transmis à des outils tels que l'Accès conditionnel pour prendre des décisions en matière d'accès, ou renvoyés à un outil SIEM (Security Information and Event Management) pour un examen plus approfondi en fonction des stratégies appliquées par votre organisation.

## <a name="why-is-automation-important"></a>Pourquoi l'automatisation est-elle importante ?

Dans son [billet de blog d'octobre 2018](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Eight-essentials-for-hybrid-identity-3-Securing-your-identity/ba-p/275843), Alex Weinert, qui dirige l'équipe Identity Security and Protection de Microsoft, explique pourquoi l'automatisation est si importante pour gérer le volume des événements :

> Chaque jour, nos systèmes d'apprentissage automatique et heuristiques fournissent des scores de risque pour 18 milliards de tentatives de connexion correspondant à plus de 800 millions de comptes distincts, dont 300 millions relèvent de manière perceptible d'adversaires (entités telles que des auteurs malveillants et des pirates informatiques).
>
> L'an dernier, lors de la conférence Ignite, j'ai évoqué les trois principales attaques auxquelles nos systèmes d'identité sont confrontés. Voici le volume récent de ces attaques
>   
>   - **Attaque par rejeu** : 4,6 milliards d'attaques détectées en mai 2018
>   - **Pulvérisation de mots de passe** : 350 000 en avril 2018
>   - **Hameçonnage** : ces attaques sont difficiles à quantifier, mais nous avons observé 23 millions d'événements à risque en mars 2018, dont beaucoup étaient liés au hameçonnage

## <a name="risk-detection-and-remediation"></a>Détection d'événements à risque et solutions pour y remédier

Identity Protection identifie les risques selon les classifications suivantes :

| Type de détection des risques | Description |
| --- | --- |
| Voyage inhabituel | Connexion à partir d’un emplacement inhabituel par rapport aux dernières connexions de l’utilisateur. |
| Adresse IP anonyme | Connexion à partir d'une adresse IP anonyme (par exemple : navigateur Tor, VPN anonymes). |
| Propriétés de connexion inhabituelles | Connexion avec des propriétés inhabituelles pour l’utilisateur concerné. |
| Adresse IP liée à un programme malveillant | Connexion à partir d’une adresse IP liée à un programme malveillant. |
| Informations d'identification divulguées | Indique que les informations d’identification valides de l’utilisateur ont été divulguées. |
| Pulvérisation de mots de passe | Indique que plusieurs noms d’utilisateurs font l’objet d’une attaque par force brute unifiée avec des mots de passe courants. |
| Azure AD Threat Intelligence | Les sources de renseignements sur les menaces internes et externes de Microsoft ont identifié un modèle d’attaque connu. |

Pour plus de détails sur ces risques et sur la façon dont ils sont calculés, consultez l'article [Qu'est-ce qu'un risque ?](concept-identity-protection-risks.md).

Les signaux de risque peuvent déclencher des mesures correctives (par exemple en obligeant les utilisateurs à utiliser Azure AD Multi-Factor Authentication ou à réinitialiser leur mot de passe à l’aide de la réinitialisation de mot de passe en libre-service) ou bloquer les utilisateurs jusqu’à ce qu’un administrateur intervienne.

## <a name="risk-investigation"></a>Examen des risques

Les administrateurs peuvent examiner les détections et prendre des mesures manuelles si nécessaire. Les administrateurs utilisent trois rapports clés pour les examens dans Identity Protection :

- Utilisateurs à risque
- Connexions risquées
- Détections de risques

Vous trouverez des informations supplémentaires dans l'article [Procédure : Examiner les risques](howto-identity-protection-investigate-risk.md).

### <a name="risk-levels"></a>Niveaux de risque

La protection d’identité catégorise les risques en trois niveaux : faible, moyen, sévère. 

Bien que Microsoft ne communique pas en détail sur l'évaluation du risque, nous pouvons affirmer que chaque niveau souligne avec un peu plus de certitude que l'utilisateur ou la connexion est compromis(e). Par exemple, une instance de propriétés de connexion non connues pour un utilisateur pourrait être moins dangereuse que la divulgation d’informations d’identification pour un autre utilisateur.

## <a name="exporting-risk-data"></a>Exportation des données liées aux risques

Les données d’Identity Protection peuvent être exportées vers d’autres outils à des fins d’archivage, d’examen et de mise en corrélation. Les API basées sur Microsoft Graph permettent aux organisations de collecter ces données pour un traitement ultérieur dans un outil tel que SIEM. Pour plus d'informations sur l'accès à l'API Identity Protection, consultez l'article [Prise en main d'Azure Active Directory Identity Protection et de Microsoft Graph](howto-identity-protection-graph-api.md).

Pour en savoir plus sur l'intégration des informations d'Identity Protection dans Azure Sentinel, consultez l'article [Connecter des données depuis Azure AD Identity Protection](../../sentinel/connect-azure-ad-identity-protection.md).

## <a name="permissions"></a>Autorisations

Identity Protection exige que les utilisateurs disposent d'une des autorisations d'accès suivantes : Lecteur de sécurité, Opérateur de sécurité, Administrateur de la sécurité, Lecteur général ou Administrateur général.

| Role | Peut | Ne peut pas |
| --- | --- | --- |
| Administrateur général | Accès complet à Identity Protection |   |
| Administrateur de sécurité | Accès complet à Identity Protection | Réinitialiser un mot de passe pour un utilisateur |
| Opérateur de sécurité | Afficher tous les rapports Identity Protection et le panneau Vue d’ensemble <br><br> Ignorer le risque lié à l’utilisateur, confirmer que la connexion est sécurisée, confirmer la compromission | Configurer ou modifier des stratégies <br><br> Réinitialiser un mot de passe pour un utilisateur <br><br> Configurer des alertes |
| Lecteur de sécurité | Afficher tous les rapports Identity Protection et le panneau Vue d’ensemble | Configurer ou modifier des stratégies <br><br> Réinitialiser un mot de passe pour un utilisateur <br><br> Configurer des alertes <br><br> Envoyer des commentaires sur les détections |

Actuellement, le rôle d’opérateur de sécurité ne permet pas d’accéder au rapport sur les connexions risquées.

Les administrateurs d’accès conditionnel peuvent également créer des stratégies qui prennent en compte le risque lié à la connexion en tant que condition. Pour plus d’informations, consultez l’article [Accès conditionnel : Conditions](../conditional-access/concept-conditional-access-conditions.md#sign-in-risk).

## <a name="license-requirements"></a>Conditions de licence :

[!INCLUDE [Active Directory P2 license](../../../includes/active-directory-p2-license.md)]

| Fonctionnalité | Détails  | Applications Azure AD Free / Microsoft 365 | Azure AD Premium P1|Azure AD Premium P2 |
| --- | --- | --- | --- | --- |
| Stratégies de risque | Stratégie de risque utilisateur (via Identity Protection)  | Non | Non |Oui | 
| Stratégies de risque | Stratégie de risque de connexion (via Identity Protection ou l’accès conditionnel)  | Non |  Non |Oui |
| Rapports de sécurité | Vue d’ensemble |  Non | Non |Oui |
| Rapports de sécurité | Utilisateurs à risque  | Informations limitées. Seuls les utilisateurs présentant un risque moyen ou élevé sont affichés. Aucun tiroir de détails ou historique des risques. | Informations limitées. Seuls les utilisateurs présentant un risque moyen ou élevé sont affichés. Aucun tiroir de détails ou historique des risques. | Accès total|
| Rapports de sécurité | Connexions risquées  | Informations limitées. Aucun détail sur les risques ou niveau de risque n’est affiché. | Informations limitées. Aucun détail sur les risques ou niveau de risque n’est affiché. | Accès total|
| Rapports de sécurité | Détections de risques   | Non | Informations limitées. Aucun tiroir de détails.| Accès total|
| Notifications | Alertes Utilisateurs à risque détectés  | Non | Non |Oui |
| Notifications | Synthèse hebdomadaire| Non | Non | Oui | 
| | Stratégie d'inscription MFA | Non | Non | Oui |

Vous trouverez des informations supplémentaires sur ces rapports détaillés dans l’article [Procédure : Examiner les risques](howto-identity-protection-investigate-risk.md#navigating-the-reports).

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble de la sécurité](concept-identity-protection-security-overview.md)

- [Qu'est-ce qu'un risque ?](concept-identity-protection-risks.md)

- [Stratégies disponibles pour atténuer les risques](concept-identity-protection-policies.md)
