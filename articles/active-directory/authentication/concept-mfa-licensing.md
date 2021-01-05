---
title: Versions et plans de consommation d'Azure AD Multi-Factor Authentication
description: Informations sur le client Azure AD Multi-Factor Authentication ainsi que sur les différentes méthodes et versions disponibles.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 06/15/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 63d880146a0b068a5d097c452c14b28db4907098
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743953"
---
# <a name="features-and-licenses-for-azure-ad-multi-factor-authentication"></a>Fonctionnalités et licences d'Azure AD Multi-Factor Authentication

Pour protéger les comptes d’utilisateurs au sein de votre organisation, vous devez utiliser l’authentification multifacteur. Cette fonctionnalité est particulièrement importante pour les comptes administratifs qui disposent d’un accès privilégié aux ressources. Les fonctionnalités de l’authentification multifacteur de base sont disponibles pour les administrateurs Microsoft 365 et Azure Active Directory (Azure AD) sans frais supplémentaires. Si vous souhaitez procéder à la mise à niveau des fonctionnalités de vos administrateurs ou étendre l'authentification multifacteur au reste de vos utilisateurs, vous pouvez acheter Azure AD Multi-Factor Authentication de plusieurs façons.

> [!IMPORTANT]
> Cet article détaille les différentes façons d'obtenir la licence Azure AD Multi-Factor Authentication et de l'utiliser. Pour obtenir des informations précises sur la tarification et sur la facturation, consultez la [page consacrée à la tarification d'Azure AD Multi-Factor Authentication](https://azure.microsoft.com/pricing/details/multi-factor-authentication/).

## <a name="available-versions-of-azure-ad-multi-factor-authentication"></a>Versions disponibles d'Azure AD Multi-Factor Authentication

Vous pouvez vous procurer la licence Azure AD Multi-Factor Authentication et l'utiliser de différentes manières en fonction des besoins de votre organisation. Vous disposez peut-être déjà d'une licence Azure AD, EMS ou Microsoft 365 qui vous permet d'utiliser Azure AD Multi-Factor Authentication. Le tableau suivant répertorie les différents moyens de se procurer Azure AD Multi-Factor Authentication, et détaille les fonctionnalités et utilisations correspondantes.

| Si vous êtes un utilisateur de | Fonctionnalités et utilisations |
| --- | --- |
| Microsoft 365 Business Premium et EMS ou Microsoft 365 E3 et E5 | EMS E3, Microsoft 365 E3 et Microsoft 365 Business Premium incluent Azure AD Premium P1. EMS E5 ou Microsoft 365 E5 incluent Azure AD Premium P2. Vous pouvez utiliser les fonctionnalités d’accès conditionnel énoncées dans les sections suivantes pour fournir une authentification multifacteur aux utilisateurs. |
| Azure AD Premium P1 | Vous pouvez utiliser l’[accès conditionnel Azure AD](../conditional-access/howto-conditional-access-policy-all-users-mfa.md) pour inviter des utilisateurs à opter pour une authentification multifacteur dans le cadre de certains scénarios ou événements en fonction des besoins de votre entreprise. |
| Azure AD Premium P2 | Fournit la sécurité la plus forte et une expérience utilisateur améliorée. Ajoute l’[accès conditionnel en fonction des risques](../conditional-access/howto-conditional-access-policy-risk.md) aux fonctionnalités d’Azure AD Premium P1, qui s’adapte aux profils d’utilisation et minimise les invites d’authentification multifacteur. |
| Tous les plans Microsoft 365 | Azure AD Multi-Factor Authentication peut être [activé par utilisateur](howto-mfa-userstates.md), ou être activé/désactivé pour tous les utilisateurs à l'aide des [paramètres de sécurité par défaut](../fundamentals/concept-fundamentals-security-defaults.md). La gestion d'Azure AD Multi-Factor Authentication s'effectue via le portail Microsoft 365. Pour une expérience utilisateur améliorée, effectuez une mise à niveau vers Azure AD Premium P1 ou P2, et utilisez un accès conditionnel. Pour plus d’informations, consultez [Sécuriser les ressources Microsoft 365 à l’aide de l’authentification multifacteur](/microsoft-365/admin/security-and-compliance/set-up-multi-factor-authentication). |
| Azure AD gratuit | Vous pouvez utiliser des [paramètres de sécurité par défaut](../fundamentals/concept-fundamentals-security-defaults.md) pour activer l’authentification multifacteur pour tous les utilisateurs. Vous ne disposez pas d’un contrôle précis des utilisateurs ou des scénarios activés, mais cela offre une étape de sécurité supplémentaire.<br /> Même si des paramètres de sécurité par défaut ne sont pas utilisés pour activer l’authentification multifacteur pour tout le monde, les utilisateurs auxquels est attribué le rôle *Administrateur général Azure AD* peuvent être configurés pour utiliser l’authentification multifacteur. Cette fonctionnalité du niveau Gratuit veille à ce que les comptes d’administrateur critiques soient protégés par une authentification multifacteur. |

## <a name="feature-comparison-of-versions"></a>Comparaison des fonctionnalités suivant les versions

Le tableau suivant fournit la liste des fonctionnalités disponibles dans les différentes versions d'Azure AD Multi-Factor Authentication. Planifiez vos besoins en matière de sécurisation de l’authentification des utilisateurs, puis déterminez l’approche correspondant à ces exigences. Par exemple, bien qu'Azure AD Free fournisse des paramètres de sécurité par défaut incluant Azure AD Multi-Factor Authentication, seule l'application d'authentification mobile peut être utilisée pour l'invite d'authentification (les appels téléphoniques et les SMS ne sont pas pris en charge). Cette approche peut constituer une limitation si vous ne pouvez pas vous assurer que l’application d’authentification mobile est installée sur l’appareil personnel d’un utilisateur.

| Fonctionnalité | Azure AD Free – Paramètres de sécurité par défaut | Azure AD Free – Administrateurs généraux d’Azure AD | Applications Microsoft 365 | Azure AD Premium P1 ou P2 |
| --- |:---:|:---:|:---:|:---:|
| Protection des comptes Administrateur locataire Azure AD avec MFA | ● | ● (Comptes d’*administrateur généraux d’Azure AD* uniquement) | ● | ● |
| Application mobile comme second facteur | ● | ● | ● | ● |
| Appel téléphonique comme second facteur | | ● | ● | ● |
| SMS comme second facteur | | ● | ● | ● |
| Contrôle d’administration sur les méthodes de vérification | | ● | ● | ● |
| Alerte de fraude | | | | ● |
| Rapports MFA | | | | ● |
| Messages de bienvenue personnalisés pour les appels téléphoniques | | | | ● |
| ID d’appelant personnalisé pour les appels téléphoniques | | | | ● |
| Adresses IP approuvées | | | | ● |
| Mémoriser MFA pour les appareils fiables | | ● | ● | ● |
| MFA pour les applications locales | | | | ● |

## <a name="purchase-and-enable-azure-ad-multi-factor-authentication"></a>Acheter et activer Azure AD Multi-Factor Authentication

Pour utiliser Azure AD Multi-Factor Authentication, inscrivez-vous pour disposer d'un niveau Azure AD éligible ou achetez-le. Azure AD est proposé en quatre éditions : Gratuite, Applications Microsoft 365, Premium P1 et Premium P2.

L’édition gratuite fait partie de tout abonnement Azure. Pour plus d’informations sur l’utilisation des paramètres de sécurité par défaut ou la protection des comptes avec le rôle *Administrateur général Azure AD*, voir la section [ci-dessous](#azure-ad-free-tier).

Les éditions Azure AD Premium sont disponibles via votre représentant Microsoft, le [programme de licences en volume Open](https://www.microsoft.com/licensing/licensing-programs/open-license.aspx) et le [programme Fournisseurs de solutions cloud](https://go.microsoft.com/fwlink/?LinkId=614968&clcid=0x409). Les abonnés Azure et Microsoft 365 peuvent également acheter Azure Active Directory Premium P1 et P2 en ligne. [Connectez-vous](https://portal.office.com/Commerce/Catalog.aspx) pour acheter.

Après avoir acheté le niveau Azure AD requis, [planifiez et déployez Azure AD Multi-Factor Authentication](howto-mfa-getstarted.md).

### <a name="azure-ad-free-tier"></a>Niveau Azure AD Free

Tous les utilisateurs d'un locataire Azure AD Free peuvent utiliser Azure AD Multi-Factor Authentication par le biais des paramètres de sécurité par défaut. L'application d'authentification mobile est la seule méthode utilisable pour Azure AD Multi-Factor Authentication en cas de recours aux paramètres de sécurité par défaut d'Azure AD Free.

* [En savoir plus sur les paramètres de sécurité par défaut d’Azure AD](../fundamentals/concept-fundamentals-security-defaults.md)
* [Activer les paramètres de sécurité par défaut pour les utilisateurs d’Azure AD Free](../fundamentals/concept-fundamentals-security-defaults.md#enabling-security-defaults)

Si vous ne souhaitez pas activer Azure AD Multi-Factor Authentication pour tous les utilisateurs, vous pouvez protéger uniquement les comptes d'utilisateurs dotés du rôle *Administrateur général Azure AD*. Cette approche fournit des invites d’authentification supplémentaires pour les comptes d’administrateur critiques. Pour activer Azure AD Multi-Factor Authentication, utilisez l'une des méthodes suivantes, en fonction du type de compte :

* Si vous utilisez un compte Microsoft, [inscrivez-vous pour bénéficier de l’authentification multifacteur](https://support.microsoft.com/help/12408/microsoft-account-about-two-step-verification).
* Si vous n’utilisez pas un compte Microsoft, [activez l’authentification multifacteur pour un utilisateur ou un groupe dans Azure AD](howto-mfa-userstates.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d'informations sur les coûts, consultez [Tarification d'Azure AD Multi-Factor Authentication](https://azure.microsoft.com/pricing/details/multi-factor-authentication/).
* [Qu’est-ce que l’accès conditionnel](../conditional-access/overview.md)

