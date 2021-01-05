---
title: Partage de données LinkedIn et consentement - Azure Active Directory | Microsoft Docs
description: Explique comment l'intégration LinkedIn partage les données par le biais des applications Microsoft dans Azure Active Directory
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: beengen
ms.custom: it-pro;seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8d13ae1e5198666e115507fd91bb0fedb1c5fa64
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96545908"
---
# <a name="linkedin-account-connections-data-sharing-and-consent"></a>Consentement et partage de données connexions de compte LinkedIn

Vous pouvez permettre à des utilisateurs au sein de votre organisation Azure Active Directory (Azure AD) de consentir à la connexion à leur compte professionnel ou scolaire Microsoft avec leur compte LinkedIn. Quand un utilisateur se connecte à son compte, des informations et points saillants de LinkedIn sont disponibles dans certains services et applications Microsoft. Les utilisateurs peuvent également s’attendre à une amélioration et un enrichissement de leur expérience de réseau sur LinkedIn avec les informations issues de Microsoft.

Pour afficher des informations LinkedIn dans les applications et services Microsoft, les utilisateurs doivent donner leur consentement afin de se connecter à leurs propres comptes Microsoft et LinkedIn. Les utilisateurs sont invités à se connecter à leur compte la première fois qu’ils cliquent pour afficher les informations LinkedIn d’une personne sur une carte de visite Outlook, OneDrive ou SharePoint Online. Les connexions de compte LinkedIn ne sont pas entièrement activées pour vos utilisateurs jusqu’à ce qu’ils aient donné leur consentement pour l’expérience et la connexion à leur compte.

[!INCLUDE [active-directory-gdpr-note](../../../includes/gdpr-hybrid-note.md)]

## <a name="benefits-of-sharing-linkedin-information"></a>Avantages du partage des informations LinkedIn

L’accès aux informations LinkedIn dans les applications et services Microsoft facilite la connexion, l’implication et l’instauration de relations professionnelles de vos utilisateurs avec des collègues, clients et partenaires au sein et en dehors de votre organisation. Les nouveaux utilisateurs peuvent être opérationnels plus rapidement en se connectant avec des collègues, en ayant plus d’informations à leur sujet et en accédant facilement à plus de données. Voici un exemple d’affichage d’informations LinkedIn sur la carte de visite dans les applications Microsoft :

![Activation de l’intégration de LinkedIn dans votre organisation](./media/linkedin-user-consent/display-example.png)

## <a name="enable-and-announce-linkedin-integration"></a>Activer et annoncer l’intégration LinkedIn

Vous devez être administrateur Azure Active Directory pour gérer le paramètre de votre organisation. Vous pouvez l’activer pour tous les utilisateurs ou pour un ensemble spécifique d’utilisateurs.

1. Pour activer ou désactiver l’intégration, procédez de la manière décrite dans [Consentir à l’intégration de LinkedIn pour votre organisation Azure AD](linkedin-integration.md).
2. Lorsque vous annoncez l’intégration LinkedIn dans votre organisation, orientez vos utilisateurs vers la FAQ sur les [informations LinkedIn dans les applications et services Microsoft](https://support.office.com/article/about-linkedin-information-and-features-in-microsoft-apps-and-services-dc81cc70-4d64-4755-9f1c-b9536e34d381). L’article vous renseigne sur l’emplacement des informations LinkedIn, la [confidentialité et le partage des données](https://support.microsoft.com/office/your-data-ae9c08a7-4d06-45b5-a065-320a97bc1400), le [mode de connexion des comptes](https://support.microsoft.com/office/connect-your-linkedin-and-work-or-school-accounts-c7c245f2-fa56-4c9b-ba20-3fceb23c5772), etc.

Vous devez annoncer l’intégration de LinkedIn à vos utilisateurs et leur fournir toutes les informations concernant [la confidentialité et le partage des données avec l’intégration de LinkedIn](https://support.microsoft.com/office/your-data-ae9c08a7-4d06-45b5-a065-320a97bc1400). 

## <a name="user-consent-for-data-access-in-microsoft-and-linkedin"></a>Consentement de l’utilisateur pour accéder aux données dans Microsoft et LinkedIn

Les données accessibles à partir de LinkedIn ne sont pas stockées définitivement dans les services Microsoft. Les données accessibles à partir de Microsoft ne sont pas stockées définitivement avec LinkedIn, à l’exception des Contacts. Les Contacts Microsoft sont stockés sur LinkedIn jusqu’à ce que les utilisateurs les suppriment, comme décrit dans [deleting imported contacts from LinkedIn](https://www.linkedin.com/help/linkedin/answer/43377) (Suppression des contacts importés à partir de LinkedIn).

Lorsque les utilisateurs se connectent à leur compte, les informations et les aperçus de LinkedIn sont disponibles dans certaines applications Microsoft, comme la carte de visite. Les utilisateurs peuvent également s’attendre à une amélioration et un enrichissement de leur expérience de réseau sur LinkedIn avec les informations issues de Microsoft.
Quand des utilisateurs de votre organisation se connectent à leurs comptes LinkedIn et Microsoft scolaire ou professionnel, ils ont deux options :

* Autorisez l’accès aux données à partir de ces deux comptes. Cela signifie qu’ils donnent à leur compte Microsoft ou professionnel l’autorisation d’accéder aux données à partir de leur compte LinkedIn, et à [leur compte LinkedIn l’autorisation d’accéder aux données à partir de leur compte scolaire ou professionnel Microsoft](https://www.linkedin.com/help/linkedin/answer/84077).
* Autorisez uniquement l’accès des données LinkedIn au moyen de leur compte professionnel ou scolaire Microsoft.

Les utilisateurs peuvent se déconnecter de leur compte et supprimer les autorisations d’accès aux données à tout moment. [Ils peuvent aussi contrôler la manière dont leur profil LinkedIn s’affiche](https://www.linkedin.com/help/linkedin/answer/83), et notamment choisir si leur profil peut apparaître dans les applications Microsoft.

### <a name="linkedin-account-data"></a>Données de compte LinkedIn

Lorsque vous connectez vos comptes Microsoft et LinkedIn, vous autorisez LinkedIn à fournir les données suivantes à Microsoft :

* Données de profil : cela inclut l’identité LinkedIn, les informations de contact et les informations que vous partagez avec d’autres utilisateurs sur votre [profil LinkedIn](https://www.linkedin.com/help/linkedin/answer/15493).
* Données d’intérêt : cela inclut les centres d’intérêt sur LinkedIn, comme les personnes et les sujets vous suivez, les groupes de formation, et le contenu que vous aimez et partagez.
* Données d’abonnement : cela inclut les abonnements aux applications et services LinkedIn, ainsi que les données associées. 
* Données de connexion : cela inclut votre [réseau LinkedIn](https://www.linkedin.com/help/linkedin/answer/110), y compris les profils et les informations de contact de vos connexions au 1er degré.

Les données accessibles à partir de LinkedIn ne sont pas stockées définitivement dans les services Microsoft. Pour plus d’informations sur l’utilisation des données personnelles par Microsoft, consultez la [Déclaration de confidentialité Microsoft](https://privacy.microsoft.com/privacystatement/).

### <a name="microsoft-work-or-school-account-data"></a>Données de compte professionnel ou scolaire Microsoft

Lorsque vous connectez vos comptes Microsoft et LinkedIn, vous autorisez Microsoft à fournir les données suivantes à LinkedIn :

* Données de profil : cela inclut des informations telles que votre prénom, votre nom de famille, votre photo de profil, votre adresse e-mail, votre responsable et les personnes dont vous êtes responsable.
* Données de calendrier : cela inclut les réunions inscrites dans vos calendriers, leurs horaires, leurs lieux et les informations de contact des participants. Les informations sur la réunion, comme l’ordre du jour, le contenu ou le titre de la réunion, ne sont pas incluses dans les données de calendrier.
* Données d’intérêt : cela inclut les centres d’intérêt associés à votre compte, en fonction de votre utilisation des services Microsoft, comme Cortana et Bing for Business.
* Données d’abonnement : cela inclut les abonnements fournis par votre organisation aux applications et services Microsoft, comme Microsoft 365.
* Données de contact : cela inclut les listes de contacts dans Outlook, Skype et d’autres services de compte Microsoft, y compris les informations de contact des personnes avec lesquelles vous communiquez ou collaborez fréquemment. Les contacts seront régulièrement importés, stockés et utilisés par LinkedIn, par exemple pour suggérer des connexions, vous aider à organiser vos contacts et afficher les mises à jour sur les contacts.

Les données accessibles à partir de Microsoft ne sont pas stockées définitivement avec LinkedIn, à l’exception des Contacts. Les Contacts Microsoft sont stockés sur LinkedIn jusqu’à ce que les utilisateurs les suppriment. En savoir plus sur la [suppression des contacts importés à partir de LinkedIn](https://www.linkedin.com/help/linkedin/answer/43377).

Pour plus d’informations sur l’utilisation des données personnelles de LinkedIn, consultez la [politique de confidentialité LinkedIn](https://www.linkedin.com/legal/privacy-policy). Pour les services LinkedIn, le transfert de données et le stockage, les données peuvent circuler de l’Union européenne vers les États-Unis et inversement, et votre vie privée est protégée conformément aux [directives européennes régissant les transferts des données](https://www.linkedin.com/help/linkedin/answer/62533).

## <a name="next-steps"></a>Étapes suivantes

* [LinkedIn dans les applications Microsoft avec votre compte professionnel ou scolaire](https://www.linkedin.com/help/linkedin/answer/84077)
