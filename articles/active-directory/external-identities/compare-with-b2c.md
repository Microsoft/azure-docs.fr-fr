---
title: Comparer des identités externes – Azure Active Directory | Microsoft Docs
description: Les identités externes Azure AD permettent à des personnes extérieures à votre organisation d’accéder à vos applications et à vos ressources en utilisant leur propre identité. Comparez les solutions pour les identités externes, à savoir Azure AD B2B Collaboration et Azure AD B2C.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: overview
ms.date: 10/23/2020
ms.custom: project-no-code
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6be0720b7eafb251b262d5cd182bdf06e201402d
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96173071"
---
# <a name="what-are-external-identities-in-azure-active-directory"></a>Présentation des identités externes dans Azure Active Directory

Avec les identités externes dans Azure AD, vous pouvez autoriser des personnes extérieures à votre organisation à accéder à vos applications et ressources tout en leur permettant de se connecter à l'aide de l'identité de leur choix. Vos partenaires, distributeurs, fournisseurs, distributeurs et autres utilisateurs invités peuvent « apporter leurs propres identités ». Qu’ils aient une identité numérique émise par le gouvernement ou une entreprise, ou une identité sociale non managée telle que Google ou Facebook, ils peuvent utiliser leurs propres informations d’identification pour se connecter. Le fournisseur d’identité de l’utilisateur externe gère leur identité, et vous gérez l’accès à vos applications avec Azure AD pour protéger vos ressources.

## <a name="external-identities-scenarios"></a>Scénarios d’identités externes

Les identités externes Azure AD se concentrent moins sur la relation d’un utilisateur avec votre organisation, et plus sur la façon dont l’utilisateur souhaite se connecter à vos applications et ressources. Dans ce cadre, Azure AD prend en charge un vaste éventail de scénarios, de la collaboration interentreprises (B2B) à la gestion de l’accès pour les applications destinées aux consommateurs, clients ou citoyens (entreprise-client ou B2C).

- **Partagez vos applications et ressources avec des utilisateurs externes (collaboration B2B)** . Invitez des utilisateurs externes dans votre propre locataire en tant qu’utilisateurs « invités » auxquels vous pouvez affecter des autorisations tout en les laissant utiliser leurs informations d’identification existantes (pour l’authentification). Les utilisateurs se connectent aux ressources partagées à l’aide d’un processus simple d’invitation et d’échange d’invitation, en utilisant leur compte professionnel ou scolaire, ou n’importe quel autre compte de messagerie. Vous pouvez utiliser la [gestion des droits d’utilisation Azure AD](../governance/entitlement-management-overview.md) pour configurer des stratégies qui vous aident à [gérer l’accès pour les utilisateurs externes](../governance/entitlement-management-external-users.md#how-access-works-for-external-users). Désormais, grâce à la disponibilité des [flux d’utilisateurs d’inscription en libre-service (préversion)](self-service-sign-up-overview.md), vous pouvez autoriser les utilisateurs externes à s’inscrire eux-mêmes à des applications. Cette expérience peut être personnalisée pour permettre l’inscription à l’aide d’une identité professionnelle, scolaire ou sociale (comme Google ou Facebook). Vous pouvez également collecter des informations sur l’utilisateur lors du processus d’inscription. Pour plus d’informations, consultez la [documentation Azure AD B2B](index.yml).

- **Créez des parcours utilisateur à l’aide d’une solution de gestion d’identité de marque blanche pour les applications destinées aux consommateurs ou clients (Azure AD B2C)** . Si vous êtes chef d’entreprise ou développeur et que vous créez des applications orientées clients, vous pouvez les mettre à l’échelle pour des millions de consommateurs, clients ou citoyens en utilisant Azure AD B2C. Les développeurs peuvent utiliser Azure AD comme système complet de gestion des identités et des accès client (CIAM) pour leurs applications. Les clients peuvent se connecter avec une identité qu’ils ont déjà établie (comme Facebook ou Gmail). Avec Azure AD B2C, vous pouvez complètement personnaliser et contrôler la façon dont les clients s’inscrivent, se connectent et gèrent leurs profils lorsqu’ils utilisent vos applications. Pour plus d’informations, consultez la [documentation Azure AD B2C](../../active-directory-b2c/index.yml).

## <a name="compare-external-identities-solutions"></a>Comparer les solutions des identités externes

Le tableau ci-dessous fournit une comparaison détaillée des scénarios que vous pouvez activer avec des identités externes Azure AD.

|   | Collaboration d’utilisateur externe (B2B) | Accès aux applications destinées aux consommateurs/clients (B2C)  |
| ---- | --- | --- |
| **Scénario principal** | Collaboration à l’aide des applications Microsoft (Microsoft 365, Teams, etc.) ou de vos propres applications (applications SaaS, applications personnalisées, etc.).  | Gestion des identités et des accès pour les applications personnalisées ou SaaS modernes (pas les applications Microsoft internes).   |
| **Usage prévu**    | Collaboration avec des partenaires commerciaux d’organisations externes tels que des fournisseurs, des partenaires et des distributeurs. Les utilisateurs apparaissent en tant qu’utilisateurs invités dans votre annuaire. Ces utilisateurs peuvent disposer ou non de services IT managés.  | Clients de votre produit. Ces utilisateurs sont gérés dans un annuaire Azure AD distinct.  |
| **Fournisseurs d’identité pris en charge** | Les utilisateurs externes peuvent collaborer à l’aide de comptes professionnels, de comptes scolaires, de n’importe quelle adresse e-mail, de fournisseurs d’identité basés sur SAML et WS-Fed, de Gmail ou de Facebook.  | Utilisateurs consommateurs disposant de comptes d’application locaux (n’importe quel nom d’utilisateur ou adresse de messagerie), diverses identités sociales prises en charge et utilisateurs dotés d’une identité émise par le gouvernement ou une entreprise via la fédération directe.       |
| **Gestion des utilisateurs externes**   | Les utilisateurs externes sont gérés dans le même annuaire que les employés, mais sont généralement annotés en tant qu’utilisateurs invités. Les utilisateurs invités peuvent être gérés de la même façon que les employés, ajoutés aux mêmes groupes, etc.    | Les utilisateurs externes sont gérés dans l’annuaire Azure AD B2C. Ils sont gérés séparément du répertoire des partenaires et des employés de l’organisation (s’il existe).  |
| **Authentification unique (SSO)**      | l’authentification unique (SSO) auprès de toutes les applications connectées à Azure AD est prise en charge. Par exemple, vous pouvez donner accès à des applications Microsoft 365 ou des applications locales, et à d’autres applications SaaS telles que Salesforce ou Workday.    | L’authentification unique auprès d’applications détenues par le client dans les clients Azure AD B2C est prise en charge. L’authentification unique auprès de Microsoft 365 ou d’autres applications SaaS Microsoft n’est pas prise en charge.    |
| **Stratégie de sécurité et conformité**        | gérée par l’organisation hôte ou qui invite (par exemple, avec des [règles d’accès conditionnelles](conditional-access.md)). | Gérée par l’organisation via l’accès conditionnel et la protection d’identité.        |
| **Personnalisation**  | la marque de l’organisation hôte (qui invite) est utilisée.    | Personnalisation complète de marque pour chaque application ou organisation.   |
| **Modèle de facturation** | [Tarifs des identités externes](https://azure.microsoft.com/en-us/pricing/details/active-directory/external-identities/) basée sur les utilisateurs actifs mensuels (MAU). <br>(Voir aussi : [Détails de configuration B2B](external-identities-pricing.md)) | [Tarifs des identités externes](https://azure.microsoft.com/en-us/pricing/details/active-directory/external-identities/) basée sur les utilisateurs actifs mensuels (MAU). <br>(Voir aussi : [Détails de configuration B2C](../../active-directory-b2c/billing.md)) |
| **Plus d’informations** | [Billet de blog](https://blogs.technet.microsoft.com/enterprisemobility/2017/02/01/azure-ad-b2b-new-updates-make-cross-business-collab-easy/), [Documentation](what-is-b2b.md)                   | [Page produit](https://azure.microsoft.com/services/active-directory-b2c/), [Documentation](../../active-directory-b2c/index.yml)       |

Sécurisez et gérez des clients et partenaires au-delà des limites de votre organisation avec des identités externes Azure AD.

## <a name="about-multitenant-applications"></a>À propos des applications multilocataires

Si vous fournissez une application en tant que service et que vous ne souhaitez pas gérer les comptes d’utilisateur de vos clients, une application multilocataire est probablement l’option qu’il vous faut. Lorsque vous développez des applications destinées à d’autres locataires Azure AD, vous pouvez cibler des utilisateurs d’une seule organisation (monolocataire) ou des utilisateurs de toute organisation ayant déjà un locataire Azure AD (applications multilocataires). Les inscriptions d’applications dans Azure AD sont de type monolocataire par défaut, mais vous pouvez rendre votre inscription multilocataire. Cette application multilocataire est inscrite une fois par vous-même dans votre propre annuaire Azure AD. Mais ensuite, tout utilisateur Azure AD de toute organisation peut utiliser l’application sans effort supplémentaire de votre part. Pour plus d’informations, consultez [Gérer l’identité dans les applications multilocataires](/azure/architecture/multitenant-identity/), [Guide pratique](../develop/howto-convert-app-to-be-multi-tenant.md).

## <a name="next-steps"></a>Étapes suivantes

- [Qu'est-ce que la collaboration B2B d'Azure AD ?](what-is-b2b.md)
- [À propos d’Azure AD B2C](../../active-directory-b2c/overview.md)