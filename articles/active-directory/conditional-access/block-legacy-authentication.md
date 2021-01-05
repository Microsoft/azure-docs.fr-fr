---
title: Bloquer l’authentification héritée - Azure Active Directory
description: Apprenez à améliorer votre approche de sécurité en bloquant l’authentification héritée à l’aide de l’accès conditionnel Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 11/05/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, dawoo
ms.collection: M365-identity-device-management
ms.openlocfilehash: d8bc1a96109eda9cc2ee1f86cec892cc5ce9308d
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860318"
---
# <a name="how-to-block-legacy-authentication-to-azure-ad-with-conditional-access"></a>Procédure : Bloquer l’authentification héritée à Microsoft Azure AD avec l’accès conditionnel   

Pour permettre à vos utilisateurs d’accéder facilement à vos applications cloud, Azure Active Directory (Azure AD) prend en charge un large éventail de protocoles d’authentification, notamment l’authentification héritée. Toutefois, les protocoles hérités ne prennent pas en charge l’authentification multifacteur (MFA). L’authentification multifacteur est couramment requise dans de nombreux environnements pour lutter contre l’usurpation d’identité. 

Alex Weinert, directeur de la sécurité des identités chez Microsoft, dans son billet de blog du 12 mars 2020, [Nouveaux outils pour bloquer l’authentification héritée dans votre organisation](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/new-tools-to-block-legacy-authentication-in-your-organization/ba-p/1225302#), met l’accent sur la raison pour laquelle les organisations doivent bloquer l’authentification héritée et sur les outils supplémentaires que Microsoft fournit pour accomplir cette tâche :

> Pour que l’authentification multifacteur soit efficace, vous devez également bloquer l’authentification héritée. Cela est dû au fait que les protocoles d’authentification hérités, tels que POP, SMTP, IMAP et MAPI, ne peuvent pas appliquer une authentification multifacteur, ce qui en fait les points d’entrée de prédilection des attaques dirigées contre votre organisation...
> 
>...Les chiffres relatifs à l’authentification héritée que révèle une analyse du trafic Azure Active Directory (Azure AD) sont sans appel :
> 
> - Plus de 99 % des attaques par pulvérisation de mots de passe utilisent des protocoles d’authentification hérités
> - Plus de 97 % des attaques par bourrage d’informations d’identification utilisent une authentification héritée
> - Les comptes Azure AD dans les organisations qui ont désactivé l’authentification héritée sont 67 % moins compromis que ceux pour lesquels l’authentification héritée est activée
>

Si votre environnement est prêt à bloquer l’authentification héritée pour améliorer la protection de votre locataire, vous pouvez atteindre cet objectif grâce à l’accès conditionnel. Cet article explique comment configurer des stratégies d’accès conditionnel qui bloquent l’authentification héritée pour votre locataire.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous connaissez bien les [concepts de base](overview.md) de l’accès conditionnel Azure AD.

## <a name="scenario-description"></a>Description du scénario

Azure AD prend en charge plusieurs protocoles d’authentification et d’autorisation parmi ceux les plus couramment utilisés, notamment l’authentification héritée. L’authentification héritée concerne les protocoles qui utilisent l’authentification de base. En règle générale, ces protocoles ne peuvent pas appliquer n’importe quel type d’authentification à deux facteurs. Pour les applications qui reposent sur l’authentification héritée, citons :

- Applications Microsoft Office plus anciennes
- Les applications utilisant des protocoles de messagerie tels que POP, IMAP et SMTP

l’authentification à un seul facteur (par exemple le nom d’utilisateur et le mot de passe) ne suffit pas de nos jours. Les mots de passe sont insuffisants, car ils sont faciles à deviner et nous (les humains) ne savons pas choisir de bons mots de passe. Les mots de passe sont également vulnérables à diverses attaques comme le hameçonnage et la pulvérisation de mots de passe. L’un des moyens les plus simples pour vous protéger contre les menaces de vol de mot de passe consiste à implémenter l’authentification multifacteur (MFA). Avec l’authentification multifacteur, même si une personne malveillante obtient le mot de passe d’un utilisateur, ce seul mot de passe n’est pas suffisant pour s’authentifier et accéder aux données.

Comment pouvez-vous empêcher les applications utilisant l’authentification héritée d’accéder aux ressources de votre locataire ? Il est recommandé de les bloquer à l’aide d’une stratégie d’accès conditionnel, tout simplement. Si nécessaire, vous pouvez autoriser uniquement certains utilisateurs et des emplacements réseau spécifiques à utiliser les applications s’appuyant sur l’authentification héritée.

Des stratégies d’accès conditionnel sont appliquées au terme de l’authentification premier facteur. L’accès conditionnel ne constitue donc pas une première ligne de défense dans des scénarios comme les attaques par déni de service, mais il peut utiliser des signaux émanant de ces événements (par exemple, le niveau de risque de connexion, l’emplacement de la requête, etc.) pour déterminer l’accès.

## <a name="implementation"></a>Implémentation

Cette section explique comment configurer une stratégie d’accès conditionnel afin de bloquer l’authentification héritée. 

### <a name="legacy-authentication-protocols"></a>Protocoles d’authentification hérités

Les options suivantes sont considérées comme des protocoles d’authentification hérités.

- SMTP authentifié : utilisé par les clients POP et IMAP pour envoyer des e-mails.
- Découverte automatique : utilisé par les clients Outlook et EAS pour rechercher des boîtes aux lettres dans Exchange Online et s’y connecter.
- Exchange ActiveSync (EAS) : utilisé pour la connexion aux boîtes aux lettres dans Exchange Online.
- Exchange Online PowerShell : utilisé pour se connecter à Exchange Online à l’aide de PowerShell à distance. Si vous bloquez l’authentification de base pour Exchange Online PowerShell, vous devez utiliser le module Exchange Online PowerShell pour vous connecter. Pour obtenir des instructions, consultez [Se connecter à Exchange Online PowerShell à l’aide de l’authentification multifacteur](/powershell/exchange/exchange-online/connect-to-exchange-online-powershell/mfa-connect-to-exchange-online-powershell).
- Exchange Web Services (EWS) : interface de programmation utilisée par Outlook, Outlook pour Mac et des applications tierces.
- IMAP4 : utilisé par les clients de messagerie IMAP.
- MAPI sur HTTP (MAPI/HTTP) : utilisé par Outlook 2010 et versions ultérieures.
- Carnet d’adresses hors connexion (OAB) : copie des collections de listes d’adresses qui sont téléchargées et utilisées par Outlook.
- Outlook Anywhere (RPC sur HTTP) : utilisé par Outlook 2016 et antérieur.
- Service Outlook : utilisé par l’application Courrier et calendrier pour Windows 10.
- POP3 : utilisé par les clients de messagerie POP.
- Reporting Web Services : utilisé pour récupérer des données de rapports dans Exchange Online.
- Autres clients : autres protocoles identifiés comme utilisant l’authentification héritée.

Pour plus d’informations sur ces protocoles et services d’authentification, consultez [Rapports d’activité de connexion dans le portail Azure Active Directory](../reports-monitoring/concept-sign-ins.md#filter-sign-in-activities).

### <a name="identify-legacy-authentication-use"></a>Identifier l’utilisation de l’authentification héritée

Avant de pouvoir bloquer l’authentification héritée dans votre annuaire, vous devez savoir si vos utilisateurs disposent d’applications qui utilisent l’authentification héritée, puis déterminer quel impact cela a sur l’ensemble de votre annuaire. Les journaux de connexion Azure AD peuvent servir à déterminer si vous utilisez une authentification héritée.

1. Accédez au **portail Azure** > **Azure Active Directory** > **Connexions**.
1. Si elle n’est pas affichée, ajoutez la colonne Application cliente en cliquant sur **Colonnes** > **Application cliente**.
1. **Ajouter des filtres** > **Application cliente** > sélectionnez tous les protocoles d’authentification hérités. Sélectionnez en dehors de la boîte de dialogue de filtrage pour appliquer vos sélections, puis fermez la boîte de dialogue.

Grâce au filtrage, vous afficherez uniquement les tentatives de connexion effectuées via des protocoles d’authentification héritée. Cliquez sur chaque tentative de connexion pour afficher des détails supplémentaires. Le champ **Application cliente** affiché sous l’onglet **Informations de base** indique quel protocole d’authentification héritée a été utilisé.

Ces journaux identifient les utilisateurs qui continuent de tirer parti de la fonction d’authentification héritée, ainsi que les applications qui utilisent les protocoles hérités pour effectuer des requêtes d’authentification. Implémentez une stratégie d’accès conditionnel réservée aux utilisateurs non affichés dans ces journaux et dont vous êtes certain qu’ils n’utilisent pas l’authentification héritée.

## <a name="block-legacy-authentication"></a>Bloquer l’authentification héritée 

Il existe deux façon d’utiliser des stratégies d’accès conditionnel pour bloquer l’authentification héritée.

- [Blocage direct de l’authentification héritée](#directly-blocking-legacy-authentication)
- [Blocage indirect de l’authentification héritée](#indirectly-blocking-legacy-authentication)
 
### <a name="directly-blocking-legacy-authentication"></a>Blocage direct de l’authentification héritée

Le moyen le plus simple de bloquer l’authentification héritée dans toute votre organisation consiste à configurer une stratégie d’accès conditionnel qui s’applique spécifiquement aux clients d’authentification héritée et bloque l’accès. Lorsque vous affectez des utilisateurs et des applications à la stratégie, veillez à exclure les utilisateurs et les comptes de service qui doivent encore se connecter à l’aide de l’authentification héritée. Configurez la condition des applications clientes en sélectionnant **Clients Exchange ActiveSync** et **Autres clients**. Pour bloquer l’accès à ces applications clientes, configurez les contrôles d’accès pour bloquer l’accès.

![Condition des applications clientes configurée pour bloquer l’authentification héritée](./media/block-legacy-authentication/client-apps-condition-configured-yes.png)

### <a name="indirectly-blocking-legacy-authentication"></a>Blocage indirect de l’authentification héritée

Même si votre organisation n’est pas prête à bloquer l’authentification héritée au sein de l’ensemble de l’organisation, vous devez vous assurer que les connexions utilisant l’authentification héritée ne contournent pas les stratégies qui requièrent des contrôles d’octroi, tels que l’authentification multifacteur ou les appareils conformes/de jointure hybride Azure AD. Pendant l’authentification, les clients d’authentification héritée ne prennent pas en charge l’envoi d’informations d’état d’authentification multifacteur, de conformité des appareils ou de jointure à Azure AD. Par conséquent, appliquez des stratégies avec des contrôles d’octroi à toutes les applications clientes afin que les connexions basées sur l’authentification héritée qui ne répondent pas aux contrôles d’octroi soient bloquées. Avec la disponibilité générale de la condition des applications clientes en août 2020, les stratégies d’accès conditionnel nouvellement créées s’appliquent à toutes les applications clientes par défaut.

![Configuration par défaut des conditions des applications clientes](./media/block-legacy-authentication/client-apps-condition-configured-no.png)

## <a name="what-you-should-know"></a>Ce que vous devez savoir

En bloquant l’accès avec **Autres clients**, vous empêchez également Exchange Online PowerShell et Dynamics 365 d’utiliser l’authentification de base.

La configuration d’une stratégie pour **d’autres clients** bloque l’organisation entière à partir de certains clients tels que SPConnect. Ce blocage se produit, car les clients plus anciens s’authentifient de manière inattendue. Ce problème ne concerne pas aux principales applications Office, telles que les anciens clients Office.

L’entrée en vigueur de la stratégie peut prendre jusqu’à 24 heures.

Vous pouvez sélectionner tous les contrôles d’octroi disponibles pour la condition **Autre clients**. Toutefois, l’expérience de l’utilisateur final est toujours la même : un accès bloqué.

### <a name="sharepoint-online-and-b2b-guest-users"></a>Utilisateurs invités SharePoint Online et B2B

Pour bloquer l’accès des utilisateurs B2B à SharePoint Online via l’authentification héritée, les organisations doivent désactiver l’authentification héritée sur SharePoint à l’aide de la commande `Set-SPOTenant` PowerShell et en définissant le paramètre `-LegacyAuthProtocolsEnabled` sur `$false`. Vous trouverez plus d’informations sur la définition de ce paramètre dans le document de référence SharePoint PowerShell concernant [Set-SPOTenant](/powershell/module/sharepoint-online/set-spotenant)

## <a name="next-steps"></a>Étapes suivantes

- [Déterminer l'impact à l'aide du mode Rapport seul de l'Accès conditionnel](howto-conditional-access-insights-reporting.md)
- Si vous ne savez pas encore configurer des stratégies d’accès conditionnel, consultez la section [Exiger l’authentification multifacteur (MFA) pour certaines applications disposant d’un accès conditionnel Azure Active Directory](../authentication/tutorial-enable-azure-mfa.md) pour en avoir un exemple.
- Pour en savoir plus sur la prise en charge de l’authentification moderne, voir [Fonctionnement de l’authentification moderne pour les applications clientes Office 2013 et Office 2016](/office365/enterprise/modern-auth-for-office-2013-and-2016). 
- [Comment configurer une application ou un appareil multifonction pour envoyer un e-mail à l’aide de Microsoft 365](/exchange/mail-flow-best-practices/how-to-set-up-a-multifunction-device-or-application-to-send-email-using-office-3)
