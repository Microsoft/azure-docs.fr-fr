---
title: 'Conception d’identités hybrides : stratégie d’adoption Azure | Microsoft Docs'
description: Avec le contrôle par accès conditionnel, Azure AD vérifie les conditions spécifiques que vous choisissez lors de l’authentification de l’utilisateur et avant d’autoriser l’accès à l’application.
documentationcenter: ''
services: active-directory
author: billmath
manager: daveba
editor: ''
ms.assetid: b92fa5a9-c04c-4692-b495-ff64d023792c
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/29/2019
ms.subservice: hybrid
ms.author: billmath
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7f52e46ff9cab7d3d150af9fd7b4f1c432bec74b
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94836187"
---
# <a name="define-a-hybrid-identity-adoption-strategy"></a>Définir une stratégie d’adoption des identités hybrides
Dans le cadre de cette tâche, vous définissez la stratégie d’adoption des identités hybrides pour que votre solution d’identités hybrides réponde aux exigences de l’entreprise évoquées dans les articles suivants :

* [Déterminer les besoins métier](plan-hybrid-identity-design-considerations-business-needs.md)
* [Déterminer les exigences de synchronisation de répertoire](plan-hybrid-identity-design-considerations-directory-sync-requirements.md)
* [Déterminer les exigences d’authentification multifacteur](plan-hybrid-identity-design-considerations-multifactor-auth-requirements.md)

## <a name="define-business-needs-strategy"></a>Définir une stratégie de besoins métier
La première tâche concerne la détermination des besoins métier de l’entreprise.  Cette opération peut être très étendue et vous risquez de vous éloigner des objectifs si vous n’êtes pas prudent.  Commencez par faire simple, mais prévoyez systématiquement une conception qui prendra en charge et facilitera les modifications ultérieures.  Qu’il s’agisse d’une conception simple ou très complexe, Azure Active Directory est la plateforme d’identités Microsoft qui prend en charge Microsoft 365, Microsoft Online Services et les applications cloud.

## <a name="define-an-integration-strategy"></a>Définir une stratégie d’intégration
Microsoft possède trois scénarios principaux d’intégration : identités cloud, identités synchronisées et identités fédérées.  Vous devez prévoir d’adopter l’une de ces stratégies d’intégration.  La stratégie choisie peut varier. Il peut y avoir différents éléments à considérer : le type d’expérience utilisateur que vous souhaitez proposer, la présence ou non d’une infrastructure existante et la rentabilité notamment.  

![Scénarios d’intégration](./media/plan-hybrid-identity-design-considerations/integration-scenarios.png)

Les scénarios définis dans la figure ci-dessus sont les suivants :

* **Identités cloud**: ce sont des identités qui existent uniquement dans le cloud.  Dans le cas d’Azure AD, elles résideraient en particulier dans votre répertoire Azure AD.
* **Synchronisée**: ce sont des identités qui existent en local et dans le cloud.  À l’aide d’Azure AD Connect, ces utilisateurs sont créés ou joints avec des comptes Azure AD existants.  Le hachage du mot de passe de l’utilisateur est synchronisé à partir de l’environnement local vers le cloud dans ce que l’on appelle un hachage du mot de passe.  L’utilisation de l’option Synchronisée présente un inconvénient : si un utilisateur est désactivé dans l’environnement local, l’affichage de l’état du compte dans Azure AD peut prendre jusqu’à trois heures.  Cela est dû à l’intervalle de synchronisation.
* **Fédérée**: ces identités existent à la fois en local et dans le cloud.  À l’aide d’Azure AD Connect, ces utilisateurs sont créés ou joints avec des comptes Azure AD existants.  

> [!NOTE]
> Pour plus d’informations sur les options de synchronisation, consultez l’article [Intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
> 
> 

Le tableau ci-après vous aide à déterminer les avantages et inconvénients de chacune des stratégies suivantes :

| Stratégie | Avantages | Inconvénients |
| --- | --- | --- |
| **Identités cloud** |Plus faciles à gérer pour les petites entreprises. <br> Rien à installer en local. Pas de matériel supplémentaire requis.<br>Faciles à désactiver si l’utilisateur quitte l’entreprise |Les utilisateurs doivent se connecter lorsqu’ils accèdent aux charges de travail dans le cloud <br> Les mots de passe peuvent être les mêmes ou non pour les identités cloud et locales |
| **Synchronisée** |Le mot de passe local authentifie les répertoires locaux et cloud <br>Plus faciles à gérer pour les petites, moyennes et grandes entreprises <br>Les utilisateurs peuvent disposer de l’authentification unique (SSO) pour certaines ressources <br> Méthode Microsoft préférée pour la synchronisation <br> Plus faciles à gérer |Certains clients peuvent hésiter à synchroniser leurs annuaires avec le cloud en raison de stratégies spécifiques de l’entreprise. |
| **Adresses IP fédérées** |Les utilisateurs peuvent disposer de l’authentification unique (SSO) <br>Si un utilisateur a achevé sa mission ou quitte l’entreprise, le compte peut immédiatement être désactivé et l’accès révoqué<br> Prise en charge de scénarios avancés qui ne peuvent pas être concrétisés avec l’option Synchronisée |Étapes supplémentaires pour l’installation et la configuration <br> Maintenance plus élevée <br> Peut requérir du matériel supplémentaire pour l’infrastructure STS <br> Peut nécessiter un matériel supplémentaire pour l’installation du serveur de fédération. Requiert des logiciels supplémentaires en cas d’utilisation des services de fédération Active Directory (AD FS). <br> Requiert une installation complète pour l’authentification unique <br> Point de défaillance critique : si le serveur de fédération est arrêté, les utilisateurs ne sont pas en mesure de s’authentifier |

### <a name="client-experience"></a>Expérience client
La stratégie que vous utilisez détermine l’expérience de connexion utilisateur.  Les tableaux suivants vous fournissent des informations sur ce que les utilisateurs peuvent attendre de leur expérience de connexion.  Certains fournisseurs d’identité fédérée ne prennent pas en charge l’authentification unique dans tous les scénarios.

**Applications réseau jointes à un domaine et privées**:

| Application | Identité synchronisée | Identité fédérée |
| --- | --- | --- |
| Navigateurs web |Authentification basée sur les formulaires |Authentification unique, parfois requise pour fournir l’ID d’organisation |
| Outlook |Demander les informations d’identification |Demander les informations d’identification |
| Skype Entreprise (Lync) |Demander les informations d’identification |Authentification unique pour Lync, informations d’identification demandées pour Exchange |
| OneDrive Entreprise |Demander les informations d’identification |Authentification unique |
| Abonnement Office Professionnel Plus |Demander les informations d’identification |Authentification unique |

**Sources externes ou non fiables** :

| Application | Identité synchronisée | Identité fédérée |
| --- | --- | --- |
| Navigateurs web |Authentification basée sur les formulaires |Authentification basée sur les formulaires |
| Outlook, Skype Entreprise (Lync), OneDrive Entreprise, abonnement Office |Demander les informations d’identification |Demander les informations d’identification |
| Exchange ActiveSync |Demander les informations d’identification |Authentification unique pour Lync, informations d’identification demandées pour Exchange |
| Applications mobiles |Demander les informations d’identification |Demander les informations d’identification |

Si vous avez déterminé lors de la tâche 1 que vous recourez ou allez recourir à un fournisseur d’identité tiers pour assurer une fédération avec Azure AD, vous devez connaître les fonctionnalités prises en charge suivantes :

* Tout fournisseur SAML 2.0 compatible avec le profil SP-Lite peut prendre en charge l’authentification auprès d’Azure AD et des applications associées
* Prend en charge l’authentification passive, qui facilite l’authentification OWA, SPO, etc.
* Les clients Exchange Online peuvent être pris en charge avec le profil client amélioré (ECP) SAML 2.0

Vous devez également avoir conscience des fonctionnalités qui ne seront pas disponibles :

* Sans prise en charge de WS-Trust/WS-Federation, tous les autres clients actifs s’interrompent.
  * Cela signifie aucun client Lync, client OneDrive, abonnement Office ou Office Mobile antérieur à Office 2016.
* La transition d’Office vers l’authentification passive permet la prise en charge de fournisseurs d’identité SAML 2.0 purs, mais cette prise en charge s’effectuera toujours client par client.

> [!NOTE]
> Pour obtenir la liste la plus récente, lisez l’article [Liste de compatibilité de fédération Azure AD](how-to-connect-fed-compatibility.md).
> 
> 

## <a name="define-synchronization-strategy"></a>Définir la stratégie de synchronisation
Dans cette tâche, vous allez définir les outils qui seront utilisés pour synchroniser les données locales de l’organisation sur le cloud et la topologie à utiliser.  Étant donné que la plupart des organisations utilisent Active Directory, les informations sur l’utilisation d’Azure AD Connect concernant les questions ci-dessus sont très détaillées.  Pour les environnements qui ne disposent pas d’Active Directory, des informations sur l’utilisation de FIM 2010 R2 ou MIM 2016 sont disponibles pour aider à planifier cette stratégie.  Toutefois, les versions futures d’Azure AD Connect prendront en charge les répertoires LDAP. Par conséquent, selon votre chronologie, ces informations sont susceptibles de vous aider.

### <a name="synchronization-tools"></a>Outils de synchronisation
Au fil des années, plusieurs outils de synchronisation ont existé et ont été utilisés pour différents scénarios.  Actuellement, Azure AD Connect est l’outil de prédilection pour tous les scénarios pris en charge.  AAD Sync et DirSync sont également toujours disponibles et peuvent même être présents dans votre environnement actuel. 

> [!NOTE]
> Pour les dernières informations concernant les fonctionnalités prises en charge de chaque outil, consultez l’article [Comparaison des outils d’intégration de répertoire](plan-hybrid-identity-design-considerations-tools-comparison.md) .  
> 
> 

### <a name="supported-topologies"></a>Topologies prises en charge
Lorsque vous définissez une stratégie de synchronisation, la topologie qui est utilisée doit être déterminée. Selon les informations qui ont été déterminées à l’étape 2, vous pouvez déterminer la topologie adaptée à utiliser. La forêt unique, topologie Azure AD unique, est la plus courante et se compose d’une forêt Active Directory unique et d’une seule instance d’Azure AD.  Ces éléments seront utilisés dans la majorité des scénarios et constituent la topologie attendue lors de l’utilisation de l’installation Express Azure AD Connect comme indiqué dans la figure ci-dessous.

![Topologies prises en charge](./media/plan-hybrid-identity-design-considerations/single-forest.png) Scénario de forêt unique Les grandes entreprises, et même les petites, disposent couramment de plusieurs forêts, comme illustré à la figure 5.

> [!NOTE]
> Pour plus d’informations sur les différentes topologies locales et Azure AD avec la synchronisation Azure AD Connect, consultez l’article [Topologies pour Azure AD Connect](plan-connect-topologies.md).
> 
> 

![Topologie de forêts multiples](./media/plan-hybrid-identity-design-considerations/multi-forest.png) 

Scénario de forêts multiples

La topologie Azure AD unique à plusieurs forêts doit être envisagée si les éléments suivants sont vrais :

* Les utilisateurs ont 1 identité uniquement dans toutes les forêts ; la section relative à l’identification unique des utilisateurs ci-dessous décrit cela plus en détail.
* L’utilisateur s’authentifie à la forêt dans laquelle se trouve son identité.
* Le nom UPN et l’ancre source (ID immuable) proviendront de cette forêt.
* Toutes les forêts sont accessibles par Azure AD Connect : cela signifie qu’elles ne doivent pas obligatoirement être jointes à un domaine et qu’elles peuvent être placées dans une zone DMZ si cela facilite ce processus.
* Les utilisateurs n’ont qu’une seule boîte aux lettres.
* La forêt qui héberge la boîte aux lettres d’un utilisateur a la meilleure qualité de données pour les attributs visibles dans la liste d’adresses globale (LAG) Exchange.
* Si aucune boîte aux lettres n’est associée à l’utilisateur, n’importe quelle forêt peut être utilisée pour fournir ces valeurs.
* Si vous avez une boîte aux lettres liée, il existe également un autre compte dans une forêt différente de celle utilisée pour la connexion.

> [!NOTE]
> Les objets qui existent à la fois en local et dans le cloud sont « connectés » via un identificateur unique. Dans le contexte de synchronisation de répertoire, cet identificateur unique est appelé SourceAnchor. Dans le contexte de l’authentification unique, il est appelé ImmutableId. [Principes de conception pour Azure AD Connect](plan-connect-design-concepts.md#sourceanchor) pour plus d’informations concernant l’utilisation de SourceAnchor.
> 
> 

Si la partie ci-dessus n’est pas vraie et que vous avez plus d’un compte actif ou plusieurs boîtes aux lettres, Azure AD Connect choisira un élément et ignorera les autres.  Si vous avez lié des boîtes aux lettres, mais aucun autre compte, ces comptes ne seront pas exportés vers Azure AD et cet utilisateur ne sera membre d’aucun groupe.  La situation a évolué avec DirSync et l’intention est de mieux prendre en charge ces scénarios de forêts multiples. Un scénario de forêts multiples est illustré dans la figure ci-dessous.

![Plusieurs clients Azure AD](./media/plan-hybrid-identity-design-considerations/multiforest-multipleAzureAD.png) 

**Scénario Azure AD de plusieurs forêts multiples**

Il est recommandé de disposer seulement d’un répertoire unique dans Azure AD pour une organisation, mais une relation 1 à 1 reste possible entre un serveur de synchronisation Azure AD Connect et un répertoire Azure AD.  Pour chaque instance d’Azure AD, vous devez disposer d’une installation d’Azure AD Connect.  En outre, par conception, Azure AD est isolé et les utilisateurs dans une instance d’Azure AD ne seront pas en mesure de voir les utilisateurs dans une autre instance.

Il est possible et pris en charge de connecter une instance locale d’Active Directory à plusieurs répertoires Azure AD, comme illustré dans la figure ci-dessous :

![Une seule forêt filtrée](./media/plan-hybrid-identity-design-considerations/single-forest-flitering.png) 

**Scénario de filtrage d’une forêt unique**

Pour cela, les éléments suivants doivent se vérifier :

* Les serveurs de synchronisation Azure AD Connect doivent être configurés pour le filtrage et ils ont donc chacun un ensemble d’objets mutuellement exclusifs.  Cela est possible, par exemple, en délimitant l’étendue de chaque serveur à un domaine ou à une unité d’organisation spécifique.
* Un domaine DNS ne peut être inscrit que dans un seul répertoire Azure AD. Les UPN des utilisateurs d’Active Directory local doivent utiliser des espaces de noms distincts.
* Les utilisateurs dans une instance d’Azure AD seront uniquement en mesure de voir les utilisateurs à partir de leur instance.  Ils ne pourront pas voir les utilisateurs dans les autres instances.
* Seul un des répertoires Azure AD peut activer Exchange hybride avec AD local.
* L’exclusivité mutuelle s’applique également à l’écriture différée.  Ainsi, certaines fonctionnalités d’écriture différée ne sont pas prises en charge avec cette topologie, car elles supposent une configuration locale unique.  notamment :
  * Écriture différée des groupes avec la configuration par défaut
  * Écriture différée des appareils

Les éléments ci-après ne sont pas pris en charge et ne doivent pas être choisis en guise d’implémentation :

* La connexion de plusieurs serveurs de synchronisation Azure AD Connect au même répertoire Azure AD n’est pas prise en charge, même s’ils sont configurés pour synchroniser un ensemble d’objets mutuellement exclusifs.
* La synchronisation d’un même utilisateur vers plusieurs annuaires Azure AD n’est pas prise en charge. 
* La modification d’une configuration pour faire en sorte que les utilisateurs dans un annuaire Azure AD apparaissent comme contacts dans un autre annuaire Azure AD n’est pas prise en charge. 
* La modification d’Azure AD Connect Sync pour qu’il se connecte à plusieurs annuaires Azure AD n’est pas non plus prise en charge.
* Les annuaires Azure AD sont isolés par conception. La modification de la configuration d’Azure AD Connect Sync pour lire des données à partir d’un autre annuaire Azure AD pour générer une liste d’adresses globale commune et unifiée entre les annuaires n’est pas prise en charge. L’exportation d’utilisateurs comme contacts vers un autre annuaire Active Directory local avec Azure AD Connect Sync n’est pas prise en charge.

> [!NOTE]
> Si votre organisation limite la connexion des ordinateurs de votre réseau à Internet, cet article liste les points de terminaison (noms de domaine complets, plages d’adresses IPv4 et IPv6) que vous devez inclure dans vos listes d’autorisation sortante et dans votre zone Sites de confiance Internet Explorer d’ordinateurs clients pour garantir que vos ordinateurs peuvent utiliser Microsoft 365. Pour plus d’informations, consultez [URL et plages d’adresses IP Office 365](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2?ui=en-US&rs=en-US&ad=US).
> 
> 

## <a name="define-multi-factor-authentication-strategy"></a>Définir la stratégie d’authentification multifacteur
Dans cette tâche, vous allez définir la stratégie d’authentification multifacteur à utiliser.  Deux versions distinctes d'Azure AD Multi-Factor Authentication sont disponibles.  L’une est basée sur le cloud, et l’autre est locale et utilise le serveur MFA Azure.  En vous appuyant sur l’évaluation effectuée précédemment, vous pouvez déterminer quelle solution est correcte pour votre stratégie.  Utilisez le tableau ci-après pour déterminer l’option de conception répondant le mieux aux exigences de sécurité de votre entreprise :

Options de conception multifacteur :

| Élément multimédia à sécuriser | MFA dans le cloud | MFA en local |
| --- | --- | --- |
| Applications Microsoft |Oui |Oui |
| Applications SaaS dans la galerie d’applications |Oui |Oui |
| Applications IIS publiées via le proxy d'application Azure AD |Oui |Oui |
| Applications IIS non publiées via le proxy Azure AD App |non |Oui |
| Accès à distance en tant que VPN, passerelle Bureau à distance (RDG) |non |Oui |

Même si vous avez opté pour une solution pour votre stratégie, vous devez toujours utiliser l’évaluation ci-dessus sur l’emplacement où se trouvent vos utilisateurs.  Cela peut entraîner la modification de la solution.  Utilisez le tableau ci-dessous pour vous aider à déterminer les éléments suivants :

| Emplacement de l’utilisateur | Option de conception préférée |
| --- | --- |
| Azure Active Directory |Multi-Factor Authentication dans le cloud |
| Azure AD et AD local à l'aide de la fédération avec AD FS |Les deux |
| Azure AD et AD local utilisant Azure AD Connect : aucune synchronisation de mot de passe |Les deux |
| Azure AD et local utilisant Azure AD Connect avec synchronisation de mot de passe |Les deux |
| AD local |Serveur Multi-Factor Authentication |

> [!NOTE]
> Vous devez également vous assurer que l’option de conception de l’authentification multifacteur sélectionnée prend en charge les fonctionnalités requises pour votre conception.  Pour plus d’informations, consultez la rubrique [Choix de la solution de sécurité multifacteur la mieux adaptée](../authentication/concept-mfa-howitworks.md).
> 

## <a name="multi-factor-auth-provider"></a>Fournisseur d’authentification multi facteurs
L’authentification multifacteur est disponible par défaut pour les administrateurs généraux disposant d’un locataire Azure Active Directory. Toutefois, si vous souhaitez étendre l’authentification multifacteur à tous vos utilisateurs et/ou souhaitez que vos administrateurs généraux puissent tirer pleinement parti de certaines fonctionnalités telles que le portail de gestion, les messages de bienvenue personnalisés et les rapports, vous devez acheter et configurer un fournisseur Multi-Factor Authentication.

> [!NOTE]
> Vous devez également vous assurer que l’option de conception de l’authentification multifacteur sélectionnée prend en charge les fonctionnalités requises pour votre conception. 
> 
> 

## <a name="next-steps"></a>Étapes suivantes
[Déterminer les exigences de protection des données](plan-hybrid-identity-design-considerations-dataprotection-requirements.md)

## <a name="see-also"></a>Voir aussi
[Présentation des considérations relatives à la conception](plan-hybrid-identity-design-considerations-overview.md)

