---
title: Quel est le risque ? Azure AD Identity Protection
description: Explication du risque dans Azure Active Directory Identity Protection
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: conceptual
ms.date: 11/09/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: f10d8a94be53780f732112c012600a7fb840642b
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96180640"
---
# <a name="what-is-risk"></a>Quel est le risque ?

Les détections de risques dans Azure AD Identity Protection incluent toutes les actions suspectes identifiées liées aux comptes d’utilisateur dans l’annuaire.

Identity Protection offre aux organisations un accès à des ressources puissantes pour voir et traiter rapidement ces actions suspectes. 

![Vue d’ensemble de la sécurité montrant les utilisateurs et les connexions à risque](./media/concept-identity-protection-risks/identity-protection-security-overview.png)

> [!NOTE]
> Identity Protection génère des détections de risques uniquement quand les informations d’identification correctes sont utilisées. Si des informations d’identification incorrectes sont utilisées sur une connexion, elles ne représentent pas un risque de compromission des informations d’identification.

## <a name="risk-types-and-detection"></a>Types de risques et détection

Il existe deux types d’**utilisateurs** et de **connexions** à risque, ainsi que deux types de détections ou de calculs **en temps réel** et **hors connexion**.

Les détections en temps réel peuvent ne pas apparaître dans les rapports pendant cinq à dix minutes. Les détections hors connexion peuvent ne pas apparaître dans les rapports pendant 2 à 24 heures.

### <a name="user-risk"></a>Risque de l’utilisateur

Un risque utilisateur reflète la probabilité qu’une identité ou un compte donné soit compromis. 

Ces risques sont calculés hors connexion à l’aide de sources d’informations sur les menaces internes et externes de Microsoft, dont des chercheurs en sécurité, des professionnels de la justice, des équipes de sécurité de Microsoft et d’autres sources approuvées.

| Détection d’événements à risque | Description |
| --- | --- |
| Informations d’identification divulguées | Ce type de détection d’événement à risque indique que les informations d’identification valides de l’utilisateur ont fuité. Souvent, lorsque les cybercriminels compromettent les mots de passe valides d’utilisateurs légitimes, ils le font dans le but de les rendre publics. Ce partage se fait généralement en publiant publiquement sur le « dark web », via des sites de pastebin, ou en échangeant et vendant des informations d’identification sur le marché noir. Lorsque le service d’informations de connexion fuitées de Microsoft acquiert des informations d’identification utilisateur sur le dark web, des ou d’autres sources, ces informations sont comparées aux informations d’identification valides actuelles des utilisateurs d’Azure AD pour rechercher des correspondances valides. Pour plus d’informations sur informations de connexion divulguées, consultez [Questions courantes](#common-questions). |
| Azure AD Threat Intelligence | Ce type de détection d’événement à risque indique une activité utilisateur inhabituelle pour l’utilisateur donné ou qui est cohérente avec des modèles d’attaque connus selon les sources internes et externes de Microsoft Threat Intelligence. |

### <a name="sign-in-risk"></a>Risque à la connexion

Un risque de connexion reflète la probabilité qu’une requête d’authentification donnée soit rejetée par le propriétaire de l'identité. 

Ces risques peuvent être calculés en temps réel ou hors connexion à l’aide de sources d’informations sur les menaces internes et externes de Microsoft, dont des chercheurs en sécurité, des professionnels de la justice, des équipes de sécurité de Microsoft et d’autres sources approuvées.

| Détection d’événements à risque | Type de détection | Description |
| --- | --- | --- |
| Adresse IP anonyme | Temps réel | Ce type de détection des risque détecte des connexions à partir d’adresses IP anonymes (par exemple, navigateur Tor VPN anonyme). Ces adresses IP sont généralement utilisées par des acteurs souhaitant masquer leur télémétrie de connexion (adresse IP, emplacement, appareil, etc.) dans un but potentiellement malveillant. |
| Voyage inhabituel | Hors connexion | Ce type de détection d’événement à risque identifie deux connexions depuis des emplacements géographiquement distants, dont au moins un est inhabituel pour l’utilisateur compte tenu de son comportement passé. Entre autres facteurs, cet algorithme Machine Learning prend en compte le délai écoulé entre les deux connexions et le temps nécessaire pour se déplacer du premier emplacement au second, ce qui indique qu’un autre utilisateur utilise les mêmes informations d’identification. <br><br> L’algorithme ignore les « faux positifs » évidents contribuant aux conditions de voyage impossible, tels que les VPN et les emplacements régulièrement utilisés par d’autres membres de l’organisation. Le système présente une période d’apprentissage initiale la plus proche de 14 jours ou de 10 connexions lui servant à assimiler le comportement de connexion des nouveaux utilisateurs. |
| Adresse IP liée à un programme malveillant | Hors connexion | Ce type de détection d’événement à risque indique les connexions depuis des adresses IP infectées par des logiciels malveillants, qui sont connus pour communiquer activement avec un serveur bot, Cette détection est effectuée en mettant en corrélation des adresses IP d’appareils d’utilisateurs avec des adresses ayant été en contact avec un serveur robot actif. |
| Propriétés de connexion inhabituelles | Temps réel | Ce type de détection d’événement à risque prend en compte l’historique des connexions antérieures (IP, latitude / longitude et NSA) pour rechercher des connexions anormales. Le système stocke les informations sur les emplacements précédents d’un utilisateur et considère ces emplacements comme « connus ». La détection d’événement à risque est déclenchée quand la connexion a lieu depuis un emplacement qui ne figure pas déjà dans la liste des emplacements connus. Les utilisateurs nouvellement créés seront en « mode d’apprentissage » pendant une période de temps durant laquelle les détections d’événements à risque des propriétés de connexion inconnues seront désactivées alors que nos algorithmes apprennent le comportement de l’utilisateur. La durée du mode d’apprentissage est dynamique et varie selon le temps qu’il faut à l’algorithme pour collecter suffisamment d’informations sur les modèles de connexion de l’utilisateur. La durée minimale est de 5 jours. Un utilisateur peut revenir en mode d’apprentissage après une longue période d’inactivité. Le système ignore également les connexions depuis les appareils connus et les emplacements géographiquement proches d’un emplacement connu. <br><br> Nous exécutons également cette détection pour l’authentification de base (ou les protocoles existants). Étant donné que ces protocoles ne proposent pas les propriétés modernes, telles que l’ID client, les données de télémétrie pour réduire le nombre de faux positifs sont limitées. Nous recommandons à nos clients de passer à l’authentification moderne. |
| L’administrateur a confirmé que cet utilisateur est compromis | Hors connexion | Cette détection indique qu’un administrateur a sélectionné « Confirmer que l’utilisateur est compromis » dans l’interface utilisateur Utilisateurs à risque ou à l’aide de l’API riskyUsers. Pour voir quel administrateur a confirmé que cet utilisateur est compromis, consultez l’historique des risques de l’utilisateur (par le biais de l’interface utilisateur ou de l’API). |
| Adresse IP malveillante | Hors connexion | Cette détection indique une connexion à partir d’une adresse IP malveillante. Une adresse IP est considérée comme malveillante quand elle présente un taux d’échecs élevé en raison d’informations d’identification non valides qu’elle envoie ou d’autres sources relatives à la réputation des adresses IP. |
| Règles suspectes de manipulation de boîte de réception | Hors connexion | Cette détection est découverte par [Microsoft Cloud App Security (MCAS)](/cloud-app-security/anomaly-detection-policy#suspicious-inbox-manipulation-rules). Cette détection dresse le profil de votre environnement et déclenche des alertes lorsque des règles suspectes qui suppriment ou déplacent des messages ou des dossiers sont définies dans la boîte de réception d'un utilisateur. Cela peut indiquer que le compte de l’utilisateur est compromis, que les messages sont intentionnellement masqués et que la boîte aux lettres est utilisée pour distribuer le courrier indésirable ou les logiciels malveillants dans votre organisation. |
| Pulvérisation de mots de passe | Hors connexion | Une attaque par pulvérisation de mots de passe est l’endroit où plusieurs noms d’utilisateur sont attaqués à l’aide de mots de passe communs dans une méthode de force brute unifiée pour obtenir un accès non autorisé. Cette détection des risques est déclenchée lorsqu’une attaque par pulvérisation de mots de passe a été effectuée. |
| Voyage impossible | Hors connexion | Cette détection est découverte par [Microsoft Cloud App Security (MCAS)](/cloud-app-security/anomaly-detection-policy#impossible-travel). Cette détection identifie deux activités de l’utilisateur (dans une seule ou plusieurs sessions) provenant d’emplacements éloignés sur le plan géographique au cours d’une période plus courte que la durée nécessaire à l’utilisateur pour aller du premier emplacement au second, indiquant qu’un autre utilisateur utilise les mêmes informations d’identification. |

### <a name="other-risk-detections"></a>Autres détections de risques

| Détection d’événements à risque | Type de détection | Description |
| --- | --- | --- |
| Risque supplémentaire détecté | Temps réel ou hors connexion | Cette détection indique que l’une des détections Premium ci-dessus a eu lieu. Étant donné que les détections Premium ne sont visibles que pour les clients Azure AD Premium P2, on les appelle « Risque supplémentaire détecté » pour les clients dépourvus de licences Azure AD Premium P2. |

## <a name="common-questions"></a>Questions courantes

### <a name="risk-levels"></a>Niveaux de risque

La protection d’identité catégorise les risques en trois niveaux : faible, moyen, sévère. Quand vous configurez des [stratégies Identity Protection personnalisées](./concept-identity-protection-policies.md#custom-conditional-access-policy), vous pouvez également les configurer pour qu’elles se déclenchent au niveau **Pas de risque**. Le niveau Pas de risque signifie qu’il n’existe aucune indication active que l’identité de l’utilisateur a été compromise.

Bien que Microsoft ne communique pas en détail sur l'évaluation du risque, nous pouvons affirmer que chaque niveau souligne avec un peu plus de certitude que l'utilisateur ou la connexion est compromis(e). Par exemple, une instance de propriétés de connexion non connues pour un utilisateur pourrait être moins dangereuse que la divulgation d’informations d’identification pour un autre utilisateur.

### <a name="password-hash-synchronization"></a>Synchronisation de hachage du mot de passe

Les détections de risque comme les informations d’identification divulguées et la vaporisation de mots de passe nécessitent la présence de hachages de mot de passe pour la détection. Pour plus d’informations sur la synchronisation de hachage du mot de passe, consultez l’article [Implémenter la synchronisation de hachage de mot de passe avec la synchronisation Azure AD Connect](../hybrid/how-to-connect-password-hash-synchronization.md).

### <a name="leaked-credentials"></a>Informations d’identification divulguées

#### <a name="where-does-microsoft-find-leaked-credentials"></a>Où Microsoft trouve-t-il les informations d’identification divulguées ?

Microsoft découvre des fuites d'informations d'identification à divers endroits, notamment :

- Les sites de téléchargement publics tels que pastebin.com et paste.ca où les malfaiteurs publient généralement ce genre de contenu. Ce genre de site est la première étape pour les malfaiteurs en quête d'informations d’identification volées.
- Forces de l'ordre.
- D'autres groupes chez Microsoft qui s’occupent des recherches sur le dark web.

#### <a name="why-arent-i-seeing-any-leaked-credentials"></a>Pourquoi ne vois-je aucune information d'identification divulguée ?

Les informations d'identification divulguées sont traitées chaque fois que Microsoft trouve une nouvelle occurrence de données accessibles au public. En raison de leur nature sensible, les informations d'identification divulguées sont supprimés peu après le traitement. Seules les nouvelles informations d’identification divulguées détectées après l’activation de la synchronisation de hachage de mot de passe (PHS) seront traitées pour votre locataire. La vérification par rapport aux paires d’informations d’identification précédemment détectées n’est pas effectuée. 

#### <a name="i-havent-seen-any-leaked-credential-risk-events-for-quite-some-time"></a>Je n’ai pas vu d’événements à risque concernant les informations d’identification divulguées depuis un moment.

Si vous n’avez pas vu d’événements à risque concernant les informations d’identification divulguées, cela peut être dû à plusieurs raisons :

- Vous n'avez aucun PHS activé pour votre locataire.
- Microsoft n'a trouvé aucune paire d’informations d’identification divulguées et correspondant à vos utilisateurs.

#### <a name="how-often-does-microsoft-process-new-credentials"></a>À quelle fréquence Microsoft traite-t-il les nouvelles informations d’identification ?

Les informations d’identification sont traitées immédiatement après avoir été détectées, normalement en plusieurs lots par jour.

## <a name="next-steps"></a>Étapes suivantes

- [Stratégies disponibles pour atténuer les risques](concept-identity-protection-policies.md)
- [Vue d’ensemble de la sécurité](concept-identity-protection-security-overview.md)