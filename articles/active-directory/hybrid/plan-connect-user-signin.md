---
title: 'Azure AD Connect : Connexion utilisateur | Microsoft Docs'
description: Connexion utilisateur Azure AD Connect pour une configuration personnalisée.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: 547b118e-7282-4c7f-be87-c035561001df
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/31/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3365a58a0c667ca55b74a5120cdd7a78ad0abc79
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997781"
---
# <a name="azure-ad-connect-user-sign-in-options"></a>Options de connexion de l’utilisateur via Azure AD Connect
Azure Active Directory (Azure AD) Connect permet à vos utilisateurs de se connecter aux ressources cloud et locales à l’aide des mêmes mots de passe. Cet article décrit les concepts clés pour chaque modèle d’identité afin de vous aider à choisir l’identité que vous souhaitez utiliser pour vous connecter à Azure AD.

Si vous connaissez déjà le modèle d’identité Azure AD et que vous souhaitez en savoir plus sur une méthode spécifique, cliquez sur le lien adéquat :

* [Synchronisation de hachage de mot de passe](#password-hash-synchronization) avec [authentification unique transparente (SSO)](how-to-connect-sso.md)
* [Synchronisation directe](how-to-connect-pta.md) avec [authentification unique transparente (SSO)](how-to-connect-sso.md)
* [Authentification unique fédérée (avec Active Directory Federation Services (AD FS))](#federation-that-uses-a-new-or-existing-farm-with-ad-fs-in-windows-server-2012-r2)
* [Fédération avec PingFederate](#federation-with-pingfederate)

> [!NOTE] 
> Il est important de vous rappeler qu’en configurant la fédération pour Azure AD, vous établissez l’approbation entre votre client Azure AD et vos domaines fédérés. Avec ce domaine d’approbation fédéré, les utilisateurs auront accès aux ressources cloud d’Azure AD au sein du client.  
>

## <a name="choosing-the-user-sign-in-method-for-your-organization"></a>Choix de la méthode de connexion utilisateur pour votre organisation
La première décision concernant l’implémentation d’Azure AD Connect est le choix de la méthode d’authentification qui sera utilisée par vos utilisateurs pour se connecter. Il est important de choisir la méthode appropriée répondant aux exigences de sécurité et avancées de votre organisation. L’authentification est essentielle, car elle validera les identités de l’utilisateur pour accéder aux applications et aux données dans le cloud. Pour choisir la méthode d’authentification correcte, vous devez prendre en compte l’infrastructure existante, le temps nécessaire à l’implémentation, sa complexité et les coûts associés. Ces facteurs sont différents pour chaque organisation et peuvent varier au fil du temps.

Azure AD prend en charge les méthodes d’authentification suivantes : 

* **Authentification cloud** : quand vous choisissez cette méthode d’authentification, Azure AD gère le processus d’authentification pour la connexion des utilisateurs. L’authentification cloud propose deux options : 
   * **Synchronisation de hachage du mot de passe**  : la synchronisation de hachage du mot de passe permet aux utilisateurs d’utiliser les mêmes nom d’utilisateur et mot de passe qu’ils utilisent localement sans avoir à déployer une infrastructure supplémentaire en plus d’Azure AD Connect. 
   * **Authentification directe (PTA)** : cette option est similaire à la synchronisation de hachage de mot de passe, mais elle permet une validation simple du mot de passe à l’aide d’agents de logiciel sur site pour les organisations disposant de stratégies de conformité et de sécurité renforcées.
* **Authentification fédérée** : quand vous choisissez cette méthode d’authentification, Azure AD délègue le processus d’authentification à un système d’authentification approuvée distinct, par exemple AD FS ou un système de fédération tiers, pour valider la connexion de l’utilisateur. 

Pour la plupart des organisations qui souhaitent simplement permettre aux utilisateurs de se connecter à Microsoft 365, aux applications SaaS et à d'autres ressources basées sur Azure AD, nous recommandons l'option de synchronisation de hachage de mot de passe par défaut.
 
Pour plus d’informations sur le choix d’une méthode d’authentification, consultez [Choisir la méthode d’authentification adaptée à votre solution d’identité hybride Azure Active Directory](./choose-ad-authn.md).

### <a name="password-hash-synchronization"></a>Synchronisation de hachage de mot de passe
Avec la synchronisation de hachage de mot de passe, les hachages des mots de passe utilisateur sont synchronisés de votre annuaire Active Directory local vers Azure AD. Lorsque les mots de passe sont modifiés ou réinitialisés localement, les hachages des nouveaux mots de passe sont immédiatement synchronisés avec Azure AD afin que vos utilisateurs puissent toujours utiliser le même mot de passe pour les ressources cloud comme pour les ressources locales. Les mots de passe ne sont jamais envoyés à Azure AD ni stockés dans Azure AD sous forme de texte clair. Vous pouvez utiliser la synchronisation de hachage de mot de passe avec la réécriture de mot de passe pour permettre la réinitialisation de mot de passe libre-service dans Azure AD.

De plus, vous pouvez activer [l’authentification unique transparente (SSO)](how-to-connect-sso.md) pour des utilisateurs sur des machines jointes à un domaine qui se trouvent sur le réseau d’entreprise. Avec l’authentification unique, les utilisateurs activés doivent seulement entrer un nom d’utilisateur pour accéder en toute sécurité aux ressources cloud.

![Synchronisation de hachage de mot de passe](./media/plan-connect-user-signin/passwordhash.png)

Pour plus d’informations, consultez l’article sur la [synchronisation de hachage de mot de passe](how-to-connect-password-hash-synchronization.md).

### <a name="pass-through-authentication"></a>Authentification directe
Avec l’authentification directe, le mot de passe de l’utilisateur est validé par rapport au contrôleur Active Directory local. Le mot de passe n’a pas besoin d’être présent dans Azure AD sous quelque forme que ce soit. Ceci permet d’évaluer les stratégies locales au cours de l’authentification auprès des services cloud, comme pour les restrictions sur les heures d’ouverture de session.

L’authentification directe utilise un agent simple sur un ordinateur Windows Server 2012 R2 joint à un domaine dans l’environnement local. Cet agent écoute les requêtes de validation de mot de passe. Il ne nécessite pas que des ports entrants soient ouverts sur Internet.

En outre, vous pouvez activer également l’authentification unique pour des utilisateurs sur des ordinateurs joints à un domaine qui se trouvent sur le réseau d’entreprise. Avec l’authentification unique, les utilisateurs activés doivent seulement entrer un nom d’utilisateur pour accéder en toute sécurité aux ressources cloud.
![Authentification directe](./media/plan-connect-user-signin/pta.png)

Pour plus d'informations, consultez les pages suivantes :
- [Authentification directe](how-to-connect-pta.md)
- [Authentification unique](how-to-connect-sso.md)

### <a name="federation-that-uses-a-new-or-existing-farm-with-ad-fs-in-windows-server-2012-r2"></a>Fédération utilisant des services AD FS nouveaux ou existants dans la batterie de serveurs Windows Server 2012 R2
Avec l’authentification fédérée, vos utilisateurs peuvent se connecter aux services basés sur Azure AD avec leurs mots de passe locaux. Lorsqu’ils sont sur le réseau d’entreprise, ils n’ont même pas besoin d’entrer leurs mots de passe. En utilisant l’option de fédération avec les services de fédération d’Azure Directory (AD FS), vous pouvez déployer un nouveau service AD FS dans la batterie de serveurs Windows Server 2012 R2 ou en spécifier un existant. Si vous choisissez de spécifier une batterie existante, Azure AD Connect configure l’approbation entre votre batterie et Azure AD de manière à ce que vos utilisateurs puissent s’authentifier.

<center>

![Fédération avec AD FS dans Windows Server 2012 R2](./media/plan-connect-user-signin/federatedsignin.png)</center>

#### <a name="deploy-federation-with-ad-fs-in-windows-server-2012-r2"></a>Déploiement d’une fédération avec AD FS dans Windows Server 2012 R2

Si vous déployez une nouvelle batterie, il vous faut :

* Un serveur Windows Server 2012 R2 pour le serveur de fédération.
* Un serveur Windows Server 2012 R2 pour le proxy d’application web.
* Un fichier .pfx avec un certificat TLS/SSL unique pour le nom de votre service de fédération. Par exemple : fs.contoso.com.

Si vous déployez une nouvelle batterie ou si vous utilisez une batterie existante, il vous faut :

* Les informations d’identification de l’administrateur local de vos serveurs de fédération.
* Les informations d’identification de l’administrateur local des serveurs des groupes de travail (ne faisant pas partie d’un domaine) sur lesquels vous avez l’intention de déployer le rôle de proxy d’application web.
* La machine sur laquelle vous exécutez l’Assistant doit être en mesure de se connecter à n’importe quel autre appareil sur lequel vous souhaitez installer AD FS ou un proxy d’application web en utilisant Windows Remote Management.

Pour plus d’informations, consultez [Configuration de l’authentification unique avec ADFS](how-to-connect-install-custom.md#configuring-federation-with-ad-fs).

### <a name="federation-with-pingfederate"></a>Fédération avec PingFederate
Avec l’authentification fédérée, vos utilisateurs peuvent se connecter aux services basés sur Azure AD avec leurs mots de passe locaux. Lorsqu’ils sont sur le réseau d’entreprise, ils n’ont même pas besoin d’entrer leurs mots de passe.

Pour plus d’informations sur la configuration de PingFederate pour une utilisation avec Azure Active Directory, consultez [Intégration de PingFederate à Azure Active Directory et Office 365](https://www.pingidentity.com/AzureADConnect)

Pour plus d’informations sur la configuration d’Azure AD Connect à l’aide de PingFederate, consultez [Installation personnalisée d’Azure AD Connect](how-to-connect-install-custom.md#configuring-federation-with-pingfederate)

#### <a name="sign-in-by-using-an-earlier-version-of-ad-fs-or-a-third-party-solution"></a>Authentification à l’aide d’une version antérieure d’AD FS ou d’une solution tierce
Si vous avez déjà configuré l’authentification cloud à l’aide d’une version antérieure d’AD FS (par exemple, AD FS 2.0) ou à l’aide d’un fournisseur de fédération tiers, vous pouvez choisir d’ignorer la configuration de la connexion utilisateur via Azure AD Connect. Cela vous permet d’obtenir la dernière synchronisation et d’autres fonctionnalités d’Azure AD Connect tout en utilisant toujours votre solution d’authentification existante.

Pour plus d’informations, consultez la [Liste de compatibilité des fournisseurs de fédération tiers Azure AD](how-to-connect-fed-compatibility.md).


## <a name="user-sign-in-and-user-principal-name"></a>Connexion de l’utilisateur et nom d’utilisateur principal
### <a name="understanding-user-principal-name"></a>Présentation du nom d’utilisateur principal
Dans Active Directory, le suffixe du nom de principal de l’utilisateur (UPN) par défaut est le nom DNS du domaine dans lequel le compte d’utilisateur a été créé. Dans la plupart des cas, il s’agit du nom de domaine enregistré en tant que domaine d’entreprise sur Internet. Toutefois, vous pouvez ajouter d’autres suffixes UPN avec les domaines et approbations Active Directory.

L’UPN de l’utilisateur est au format username@domain. Par exemple, pour un domaine Active Directory nommé « contoso.com », un utilisateur nommé John peut avoir l’UPN suivant : « john@contoso.com ». L’UPN de l’utilisateur se base sur RFC 822. Bien que l’UPN et la messagerie partagent le même format, la valeur du nom UPN d’un utilisateur n’est pas forcément identique à l’adresse de messagerie de l’utilisateur.

### <a name="user-principal-name-in-azure-ad"></a>Nom d’utilisateur principal dans Azure AD
L’Assistant Azure AD Connect utilise l’attribut userPrincipalName ou vous laisse spécifier l’attribut (dans une installation personnalisée) à utiliser en local en tant que nom d’utilisateur principal dans Azure AD. Il s’agit de la valeur qui est utilisée pour la connexion à Azure AD. Si la valeur de l’attribut du nom principal d’utilisateur ne correspond pas à un domaine vérifié dans Azure AD, Azure AD la remplacera par une valeur .onmicrosoft.com par défaut.

Dans Azure Active Directory, chaque annuaire est fourni avec un nom de domaine intégré se présentant sous la forme contoso.onmicrosoft.com qui vous permet de commencer à utiliser Azure ou d’autres services Microsoft. Vous pouvez améliorer et simplifier l’expérience de connexion avec les domaines personnalisés. Pour plus d’informations sur les noms de domaine personnalisés dans Azure AD et sur la vérification des domaines, consultez [Ajouter votre nom de domaine personnalisé à Azure Active Directory](../fundamentals/add-custom-domain.md).

## <a name="azure-ad-sign-in-configuration"></a>Configuration de connexion AD Azure
### <a name="azure-ad-sign-in-configuration-with-azure-ad-connect"></a>Configuration de connexion AD Azure avec Azure AD Connect
L’expérience de connexion Azure AD dépend de la capacité d’Azure AD à faire correspondre le suffixe de nom principal d’un utilisateur en cours de synchronisation à l’un des domaines personnalisés vérifiés dans l’annuaire Azure AD. Azure AD Connect fournit une aide lors de la configuration des paramètres de connexion Azure AD, afin que l’expérience de connexion utilisateur sur le cloud soit similaire à l’expérience locale.

Azure AD Connect répertorie les suffixes UPN qui sont définis pour les domaines et essaie de les mettre en correspondance avec un domaine personnalisé dans Azure AD. Il vous aide ensuite avec l’action appropriée à entreprendre.
La page de connexion AD Azure répertorie les suffixes UPN définis pour Active Directory local et affiche l’état correspondant à chaque suffixe. L’état peut avoir une des valeurs suivantes :

| State | Description | Action requise |
|:--- |:--- |:--- |
| Verified |Azure AD Connect a trouvé un domaine vérifié correspondant dans Azure AD. Tous les utilisateurs de ce domaine peuvent se connecter en utilisant leurs informations d’identification locales. |Aucune action n'est nécessaire. |
| Non vérifié |Azure AD Connect a trouvé un domaine personnalisé correspondant dans Azure AD mais il n’est pas vérifié. Si le domaine n’est pas vérifié, le suffixe UPN des utilisateurs de ce domaine sera remplacé par le suffixe .onmicrosoft.com par défaut après la synchronisation. | [Vérifiez le domaine personnalisé dans Azure AD.](../fundamentals/add-custom-domain.md#verify-your-custom-domain-name) |
| Non ajouté |Azure AD Connect n’a pas trouvé de domaine personnalisé correspondant au suffixe UPN. Si le domaine n’est pas ajouté et vérifié dans Azure, le suffixe UPN des utilisateurs de ce domaine sera remplacé par le suffixe .onmicrosoft.com. | [Ajouter et vérifier un domaine personnalisé correspondant au suffixe UPN.](../fundamentals/add-custom-domain.md) |

La page de connexion AD Azure répertorie le(s) suffixe(s) UPN défini(s) pour Active Directory local et le domaine personnalisé correspondant dans Azure AD avec l’état actuel de la vérification. Dans une installation personnalisée, vous pouvez maintenant sélectionner l’attribut du nom d’utilisateur principal sur la page **Connexion à Azure AD** .

![Page de connexion d’Azure AD](./media/plan-connect-user-signin/custom_azure_sign_in.png)

Vous pouvez cliquer sur le bouton Actualiser pour extraire à nouveau le dernier état des domaines personnalisés à partir d’Azure AD.

### <a name="selecting-the-attribute-for-the-user-principal-name-in-azure-ad"></a>Sélection d’un attribut pour le nom d’utilisateur principal dans Azure AD
L'attribut userPrincipalName est utilisé par les utilisateurs lorsqu'ils se connectent à Azure AD et Microsoft 365. Vous devez vérifier les domaines (également nommés « Suffixe UPN ») utilisés dans Azure AD avant la synchronisation des utilisateurs.

Nous vous recommandons fortement de conserver l’userPrincipalName de l’attribut par défaut. Si cet attribut ne peut pas être acheminé ni vérifié, vous pouvez sélectionner un autre attribut (par exemple une adresse de messagerie électronique) comme attribut contenant l’ID de connexion. Il s’agit de l’ID secondaire. La valeur de l’attribut ID secondaire doit suivre la norme RFC 822. Vous pouvez utiliser un ID secondaire avec l’authentification unique par mot de passe et avec l’authentification unique de fédération comme solution de connexion.

> [!NOTE]
> L'utilisation d'un ID secondaire n'est pas compatible avec certaines charges de travail Microsoft 365. Pour plus d’informations, consultez [Configuration d’un ID secondaire de connexion](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id).
>
>

#### <a name="different-custom-domain-states-and-their-effect-on-the-azure-sign-in-experience"></a>Différents états de domaine personnalisé et leur impact sur l’expérience de connexion Azure
Il est très important de comprendre la relation entre les états de domaine personnalisé dans votre annuaire Azure AD et les suffixes UPN définis en local. Passons en revue les différentes expériences de connexion Azure possibles lorsque vous configurez la synchronisation avec Azure AD Connect.

Pour les informations suivantes, supposons que nous nous intéressons au suffixe UPN contoso.com utilisé dans l’annuaire local dans le nom UPN, par exemple user@contoso.com.

###### <a name="express-settingspassword-hash-synchronization"></a>Configuration rapide / Synchronisation de hachage de mot de passe

| State | Effet sur l’expérience de connexion utilisateur Azure |
|:---:|:--- |
| Non ajouté |Dans ce cas, aucun domaine personnalisé pour contoso.com n’a été ajouté à l’annuaire Azure AD. Les utilisateurs possédant un UPN local avec le suffixe @contoso.com ne pourront pas utiliser leur UPN local pour se connecter à Azure. Ils devront utiliser un nouvel UPN fourni par Azure AD en ajoutant le suffixe de l’annuaire Azure AD par défaut. Par exemple, si vous synchronisez des utilisateurs sur l’annuaire Azure AD azurecontoso.onmicrosoft.com, l’utilisateur local user@contoso.com aura un nom UPN user@azurecontoso.onmicrosoft.com. |
| Non vérifié |Dans ce cas, nous avons un domaine personnalisé contoso.com ajouté à l’annuaire Azure AD. Toutefois, la vérification n’est pas encore effectuée. Si vous poursuivez la synchronisation des utilisateurs sans vérifier le domaine, les utilisateurs se verront attribuer un nouvel UPN d’Azure AD, comme dans le scénario « Non ajouté ». |
| Verified |Dans ce cas, nous avons un domaine personnalisé contoso.com déjà ajouté et vérifié dans Azure AD pour le suffixe UPN. Les utilisateurs pourront utiliser leur nom d’utilisateur principal local, par exemple, user@contoso.com, pour se connecter à Azure une fois qu’ils seront synchronisés sur Azure AD. |

###### <a name="ad-fs-federation"></a>Fédération AD FS
Vous ne pouvez pas créer de fédération avec le domaine .onmicrosoft.com par défaut dans Azure AD ou un domaine personnalisé non vérifié dans Azure AD. Lorsque vous exécutez l’Assistant Azure AD Connect, si vous sélectionnez un domaine non vérifié pour créer une fédération, Azure AD Connect vous invitera à créer les enregistrements nécessaires à l’endroit où est hébergé votre DNS pour le domaine. Pour plus d’informations, consultez [Vérification du domaine Azure AD sélectionné pour la fédération](how-to-connect-install-custom.md#verify-the-azure-ad-domain-selected-for-federation).

Si vous avez sélectionné l’option de connexion utilisateur **Fédération avec AD FS**, vous devez disposer d’un domaine personnalisé pour poursuivre la création d’une fédération dans Azure AD. Dans notre cas de figure, cela signifie que nous devons disposer d’un domaine personnalisé contoso.com ajouté dans l’annuaire Azure AD.

| State | Effet sur l’expérience de connexion utilisateur Azure |
|:---:|:--- |
| Non ajouté |Dans ce cas, Azure AD Connect n’a pas trouvé de domaine personnalisé correspondant au suffixe UPN contoso.com dans l’annuaire Azure AD. Vous devez ajouter un domaine personnalisé contoso.com si vous avez besoin que les utilisateurs se connectent à l’aide d’AD FS avec leur UPN local (par exemple user@contoso.com). |
| Non vérifié |Dans ce cas, Azure AD Connect vous fournira les informations appropriées sur les possibilités de vérification de votre domaine à un stade ultérieur. |
| Verified |Dans ce cas, vous pouvez poursuivre la configuration sans autre action. |

## <a name="changing-the-user-sign-in-method"></a>Modification de la méthode de connexion utilisateur
Après la configuration initiale d’Azure AD Connect dans l’Assistant, vous pouvez changer la méthode de connexion utilisateur de Fédération, Synchronisation de mot de passe ou Authentification directe à l’aide des tâches disponibles dans Azure AD Connect. Réexécutez l’Assistant Azure AD Connect. La liste des tâches que vous pouvez effectuer s’affiche. Sélectionnez **Modifier la connexion utilisateur** dans la liste des tâches.

![Modifier la connexion de l’utilisateur](./media/plan-connect-user-signin/changeusersignin.png)

Sur la page suivante, vous devrez fournir les informations d’identification pour Azure AD.

![Capture d’écran montrant où vous devez saisir les informations d’identification pour Azure AD.](./media/plan-connect-user-signin/changeusersignin2.png)

Dans la page **Connexion de l’utilisateur**, sélectionnez la connexion d’utilisateur souhaitée.

![Se connecter à Azure AD](./media/plan-connect-user-signin/changeusersignin2a.png)

> [!NOTE]
> Si vous passez à la synchronisation de hachage de mot de passe de façon temporaire, cochez la case **Ne pas convertir les comptes utilisateurs**. Si vous ne cochez pas l’option, chaque utilisateur devient fédéré, et la conversion peut prendre plusieurs heures.
>
>

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur [l’intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
- En savoir plus sur [Azure AD Connect : principes de conception](plan-connect-design-concepts.md).