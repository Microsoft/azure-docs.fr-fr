---
title: Paramètres de sécurité par défaut Azure Active Directory
description: Stratégies de sécurité par défaut permettant de protéger les entreprises contre les attaques courantes dans Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 05/13/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: rogoya
ms.collection: M365-identity-device-management
ms.custom: contperf-fy20q4
ms.openlocfilehash: 9c912b7122a40ae39bcbb703e1387af25fff2bc8
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029904"
---
# <a name="what-are-security-defaults"></a>Présentation des paramètres de sécurité par défaut

La gestion de la sécurité peut s’avérer ardue lorsque les attaques courantes liées aux identités, telles que la pulvérisation de mot de passe, la relecture et le hameçonnage, deviennent de plus en plus monnaie courante. Les paramètres de sécurité par défaut facilitent la protection de votre organisation contre ces attaques avec des paramètres de sécurité préconfigurés :

- En exigeant que tous les utilisateurs s’inscrivent à Azure AD Multi-Factor Authentication.
- En exigeant des administrateurs qu’ils effectuent l’authentification multifacteur.
- En restreignant les protocoles d’authentification hérités.
- En exigeant des utilisateurs qu’ils effectuent l’authentification multifacteur, lorsque cela est nécessaire.
- En protégeant des activités privilégiées, telles que l’accès au Portail Azure.

![Capture d’écran du Portail Azure avec le bouton bascule pour activer les paramètres de sécurité par défaut](./media/concept-fundamentals-security-defaults/security-defaults-azure-ad-portal.png)
 
Vous trouverez plus d’informations sur les raisons pour lesquelles les paramètres de sécurité par défaut sont rendus disponibles dans le billet de blog d’Alex Weinert, [Introducing security defaults](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/introducing-security-defaults/ba-p/1061414).

## <a name="availability"></a>Disponibilité

Microsoft met les paramètres de sécurité par défaut à la disposition de tous. Le but est de s’assurer que toutes les organisations bénéficient d’un niveau de sécurité de base activé, sans coût supplémentaire. Vous activez les paramètres de sécurité par défaut dans le Portail Azure. Si votre locataire a été créé le 22 octobre 2019 ou à une date ultérieure, il est possible que les paramètres de sécurité par défaut soient déjà activés dans votre locataire. Dans le but de protéger tous nos utilisateurs, les paramètres de sécurité par défaut sont déployés sur tous les nouveaux locataires créés.

### <a name="whos-it-for"></a>À qui cela s’adresse-t-il ?

- Si vous êtes une organisation souhaitant augmenter son niveau de sécurité, mais que vous ne savez pas comment ou où commencer, les paramètres de sécurité par défaut sont pour vous.
- Si vous êtes une organisation utilisant le niveau gratuit de licences Azure Active Directory, les paramètres de sécurité par défaut sont également appropriés pour vous.

### <a name="who-should-use-conditional-access"></a>Qui devrait utiliser l’accès conditionnel ?

- Si vous êtes une organisation utilisant actuellement des stratégies d’accès conditionnel pour mettre en place des signaux, prendre des décisions et appliquer des stratégies organisationnelles, les paramètres de sécurité par défaut ne sont probablement pas appropriés pour vous. 
- Si vous êtes une organisation utilisant des licences Azure Active Directory Premium, les paramètres de sécurité par défaut ne sont probablement pas appropriés pour vous.
- Si votre organisation a des exigences de sécurité complexes, vous devriez envisager l’accès conditionnel.

## <a name="policies-enforced"></a>Stratégies appliquées

### <a name="unified-multi-factor-authentication-registration"></a>Inscription Multi-Factor Authentication unifiée

Tous les utilisateurs de votre locataire doivent s’inscrire à l’authentification multifacteur (MFA) via le formulaire d’Azure AD Multi-Factor Authentication. Les utilisateurs disposent de 14 jours pour s’inscrire à Azure AD Multi-Factor Authentication à l’aide de l’application Microsoft Authenticator. Au bout de ces 14 jours, l’utilisateur ne sera plus en mesure de se connecter, jusqu’à ce que l’inscription soit effectuée. Ainsi, la période de 14 jours d’un utilisateur commence après la première connexion interactive réussie, une fois les paramètres de sécurité par défaut activés.

### <a name="protecting-administrators"></a>Protection des administrateurs

Les utilisateurs disposant d’un accès privilégié ont un accès plus large à votre environnement. En raison de l’importance de ces comptes, vous devez leur accorder une attention particulière. Une méthode courante pour améliorer la protection de comptes privilégiés consiste à demander une forme de vérification de compte plus stricte pour se connecter. Dans Azure AD, vous pouvez obtenir une vérification plus sévère des comptes en exigeant l’authentification multifacteur.

Une fois l’inscription à Azure AD Multi-Factor Authentication terminée, les neuf rôles Administrateur Azure AD suivants sont nécessaires pour effectuer une authentification supplémentaire chaque fois qu’ils se connectent :

- Administrateur général
- Administrateur SharePoint
- Administrateur Exchange
- Administrateur de l’accès conditionnel
- Administrateur de sécurité
- Administrateur du support technique
- Administrateur de facturation
- Administrateur d’utilisateurs
- Administrateur d’authentification

### <a name="protecting-all-users"></a>Protection de tous les utilisateurs

Nous avons tendance à considérer que les comptes administrateur sont les seuls qui nécessitent des couches d’authentification supplémentaires. Les administrateurs ont largement accès à des informations sensibles et peuvent modifier des paramètres à l’échelle d’un abonnement. Pourtant, les attaquants ciblent souvent les utilisateurs finaux. 

Une fois que ces personnes malveillantes ont accès, elles peuvent demander l’accès aux informations privilégiées pour le compte du détenteur du compte d’origine. Elles peuvent même télécharger l’annuaire entier pour effectuer une attaque par hameçonnage sur l’ensemble de votre organisation. 

Une méthode courante pour améliorer la protection de tous les utilisateurs consiste à demander une forme de vérification de compte plus stricte, telle Multi-Factor Authentication, pour tous. Lorsque les utilisateurs ont terminé l’inscription Multi-Factor Authentication, ils sont invités à fournir une authentification supplémentaire chaque fois que nécessaire. Cette fonctionnalité protège toutes les applications inscrites avec Azure AD, y compris les applications SaaS.

### <a name="blocking-legacy-authentication"></a>Blocage de l’authentification héritée

Pour permettre à vos utilisateurs d’accéder facilement à vos applications cloud, Azure AD prend en charge un éventail de protocoles d’authentification, notamment l’authentification héritée. *L’authentification héritée* est un terme qui fait référence à une requête d’authentification effectuée par :

- Les clients Office qui n'utilisent pas l'authentification moderne (par exemple, un client Office 2010).
- Tout client qui utilise d’anciens protocoles de messagerie tels que IMAP, SMTP ou POP3.

Aujourd’hui, la majorité des tentatives de connexion compromettantes ont pour origine l’authentification héritée. L’authentification héritée ne prend pas en charge Multi-Factor Authentication. Même si une stratégie Multi-Factor Authentication est activée sur votre annuaire, un attaquant peut s’authentifier à l’aide d’un protocole plus ancien et contourner Multi-Factor Authentication. 

Lorsque les paramètres de sécurité par défaut sont activés dans votre locataire, toutes les demandes d’authentification effectuées par un protocole hérité sont bloquées. Les paramètres par défaut de sécurité bloquent l’authentification de base Exchange Active Sync.

> [!WARNING]
> Avant d’activer les paramètres de sécurité par défaut, assurez-vous que vos administrateurs n’utilisent aucun protocole d’authentification plus anciens. Pour plus d’informations, voir [Comment se passer de l’authentification héritée](concept-fundamentals-block-legacy-authentication.md).

- [Comment configurer une application ou un appareil multifonction pour envoyer un e-mail à l’aide de Microsoft 365](/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-microsoft-365-or-office-365)

### <a name="protecting-privileged-actions"></a>Protection des actions privilégiées

Les organisations utilisent divers services Azure managés par le biais de l’API Azure Resource Manager, entre autres :

- Portail Azure 
- Azure PowerShell 
- Azure CLI

Utiliser Azure Resource Manager pour gérer vos services est une action très privilégiée. Azure Resource Manager peut modifier des configurations à l’échelle du locataire, telles que les paramètres de service et la facturation de l’abonnement. L’authentification à facteur unique est vulnérable à diverses attaques comme le hameçonnage et la pulvérisation de mots de passe. 

Il est important de vérifier l’identité des utilisateurs qui souhaitent accéder aux configurations Azure Resource Manager et de mise à jour. Vous vérifiez leur identité en exigeant une authentification supplémentaire avant d’autoriser l’accès.

Une fois que vous avez activé les paramètres de sécurité par défaut dans votre locataire, tout utilisateurs accédant au Portail Azure, à Azure PowerShell ou à Azure CLI doit effectuer une authentification supplémentaire. Cette stratégie s’applique à tous les utilisateurs accédant à Azure Resource Manager, qu’ils soient administrateurs ou utilisateurs. 

> [!NOTE]
> L’authentification moderne est désactivée par défaut pour les locataires Exchange Online antérieurs à 2017. Pour éviter la possibilité d’une boucle de connexion lors de l’authentification par le biais de ces locataires, vous devez [activer l’authentification moderne](/exchange/clients-and-mobile-in-exchange-online/enable-or-disable-modern-authentication-in-exchange-online).

> [!NOTE]
> Le compte de synchronisation Azure AD Connect est exclu des paramètres de sécurité par défaut et ne sera pas invité à s’inscrire ou à effectuer une authentification multifacteur. Les organisations ne doivent pas utiliser ce compte à d’autres fins.

## <a name="deployment-considerations"></a>Points à prendre en considération pour le déploiement

Les considérations supplémentaires suivantes concernent le déploiement des paramètres de sécurité par défaut.

### <a name="authentication-methods"></a>Méthodes d’authentification

Ces paramètres de sécurité par défaut gratuits permettent l’inscription à Azure AD Multi-Factor Authentication et son utilisation **en utilisant uniquement l’application Microsoft Authenticator à l’aide de notifications**. L’accès conditionnel permet l'utilisation de n’importe quelle méthode d’authentification que l’administrateur choisit d’activer.

| Méthode | Paramètres de sécurité par défaut | Accès conditionnel |
| --- | --- | --- |
| Notification via une application mobile | X | X |
| Code de vérification provenant d’une application mobile ou d’un jeton matériel | X** | X |
| Message texte vers le téléphone |   | X |
| Appel vers le téléphone |   | X |
| Mots de passe d'application |   | X**_ |

- _* Les utilisateurs peuvent utiliser des codes de vérification de l’application Microsoft Authenticator, mais ils peuvent uniquement s’inscrire à l’aide de l’option de notification.
- **_ Les mots de passe d’application sont uniquement disponibles dans l’authentification multifacteur par utilisateur avec des scénarios d’authentification hérités uniquement s’ils sont activés par des administrateurs.

### <a name="disabled-mfa-status"></a>État MFA désactivé

Si votre organisation utilisait précédemment Azure AD Multi-Factor Authentication par utilisateur, ne vous inquiétez pas de ne pas voir les utilisateurs à l’état _ *Activé** ou **Appliqué** lorsque vous examinez la page d’état de MFA. L’état **Désactivé** est approprié pour les utilisateurs utilisant les paramètres de sécurité par défaut ou Azure AD Multi-Factor Authentication basé sur l’accès conditionnel.

### <a name="conditional-access"></a>Accès conditionnel

Vous pouvez utiliser l’accès conditionnel pour configurer des stratégies similaires aux paramètres de sécurité par défaut, mais avec une plus grande granularité, notamment des exclusions d’utilisateurs, non disponibles dans les paramètres de sécurité par défaut. Si vous utilisez l’accès conditionnel et que des stratégies d’accès conditionnel sont activées dans votre environnement, les paramètres de sécurité par défaut ne seront pas disponibles pour vous. Si vous disposez d’une licence qui fournit un accès conditionnel, mais que vous n’avez activé aucune stratégie d’accès conditionnel dans votre environnement, vous êtes invité à utiliser les paramètres de sécurité par défaut, tant que vous activez pas des stratégies d’accès conditionnel. Pour plus d’informations sur les licences Azure AD, consultez la [page des tarifs Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).

![Message d’avertissement indiquant que vous pouvez avoir des paramètres de sécurité par défaut ou un accès conditionnel, mais pas les deux](./media/concept-fundamentals-security-defaults/security-defaults-conditional-access.png)

Voici des guides pas à pas sur la façon dont vous pouvez utiliser l’accès conditionnel pour configurer des stratégies équivalentes à celles rendues possibles par les paramètres de sécurité par défaut :

- [Exiger MFA pour les administrateurs](../conditional-access/howto-conditional-access-policy-admin-mfa.md)
- [Exiger MFA pour la gestion Azure](../conditional-access/howto-conditional-access-policy-azure-management.md)
- [Bloquer l’authentification héritée](../conditional-access/howto-conditional-access-policy-block-legacy.md)
- [Exiger MFA pour tous les utilisateurs](../conditional-access/howto-conditional-access-policy-all-users-mfa.md)
- [Exiger l’inscription Azure AD MFA](../identity-protection/howto-identity-protection-configure-mfa-policy.md) : nécessite la fonctionnalité Azure AD Identity Protection faisant partie de l’offre Azure AD Premium P2.

## <a name="enabling-security-defaults"></a>Activation des paramètres de sécurité par défaut

Pour activer les paramètres de sécurité par défaut dans votre annuaire :

1. Connectez-vous au  [portail Azure](https://portal.azure.com)  en tant qu’administrateur de la sécurité, administrateur de l’accès conditionnel ou administrateur général.
1. Accédez à  **Azure Active Directory** > **Propriétés**.
1. Sélectionnez **Gérer les paramètres de sécurité par défaut**.
1. Définissez **Activer les paramètres de sécurité par défaut** sur **Oui** à l’aide du bouton bascule.
1. Sélectionnez **Enregistrer**.

## <a name="disabling-security-defaults"></a>Désactivation des paramètres de sécurité par défaut

Les organisations choisissant d’implémenter des stratégies d’accès conditionnel qui remplacent les paramètres de sécurité par défaut doivent désactiver les paramètres de sécurité par défaut. 

![Message d’avertissement relatif à la désactivation des paramètres de sécurité par défaut pour activer l’accès conditionnel](./media/concept-fundamentals-security-defaults/security-defaults-disable-before-conditional-access.png)

Pour désactiver les paramètres de sécurité par défaut dans votre annuaire :

1. Connectez-vous au  [portail Azure](https://portal.azure.com)  en tant qu’administrateur de la sécurité, administrateur de l’accès conditionnel ou administrateur général.
1. Accédez à  **Azure Active Directory** > **Propriétés**.
1. Sélectionnez **Gérer les paramètres de sécurité par défaut**.
1. Définissez **Activer les paramètres de sécurité par défaut** sur **Non** à l’aide du bouton bascule.
1. Sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

[Stratégies d’accès conditionnel courantes](../conditional-access/concept-conditional-access-policy-common.md)