---
title: Inscription combinée pour la réinitialisation de mot de passe en libre-service et Azure AD Multi-Factor Authentication - Azure Active Directory
description: Familiarisez-vous avec l'expérience d'inscription combinée d'Azure Active Directory qui permet aux utilisateurs de s'inscrire à la fois à l'authentification Azure AD MFA et à la réinitialisation de mot de passe en libre-service (SSPR)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 12/04/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: c8dce284c0fffe10fe077fcb6c6713ba65c45751
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743902"
---
# <a name="combined-security-information-registration-for-azure-active-directory-overview"></a>Vue d’ensemble de l’inscription combinée d’informations de sécurité pour Azure Active Directory

Avant l'inscription combinée, les utilisateurs s'inscrivaient séparément aux méthodes d'authentification Azure AD Multi-Factor Authentication (Azure AD MFA) et Réinitialisation de mot de passe en libre-service (SSPR). Certains étaient troublés par le fait que des méthodes similaires servaient pour l’authentification multifacteur et la réinitialisation SSPR, mais qu’ils devaient s’inscrire aux deux fonctionnalités. Désormais, avec l’inscription combinée, les utilisateurs peuvent s’inscrire une seule fois et bénéficier des avantages de l’authentification multifacteur et de la réinitialisation SSPR.

> [!NOTE]
> À partir du 15 août 2020, l’inscription combinée sera automatiquement activée pour tous les nouveaux locataires Azure AD. À partir du 14 décembre 2020, l’inscription combinée sera automatiquement activée pour tous les locataires Azure AD ayant un utilisateur.

Cet article explique ce qu’est l’inscription de sécurité combinée. Pour prendre en main l’inscription de sécurité combinée, consultez l’article suivant :

> [!div class="nextstepaction"]
> [Activer l’inscription de sécurité combinée](howto-registration-mfa-sspr-combined.md)

![Page Mon profil affichant les informations de sécurité d’un utilisateur inscrit](media/concept-registration-mfa-sspr-combined/combined-security-info-defualts-registered.png)

Avant d’activer la nouvelle expérience, passez en revue cette documentation axée sur l’administrateur et la documentation orientée utilisateur pour vous assurer de bien comprendre les fonctions et l’effet de cette fonctionnalité. Basez votre formation sur la [documentation pour les utilisateurs](../user-help/security-info-setup-signin.md) afin de préparer vos utilisateurs à cette nouvelle expérience et de favoriser un déploiement réussi.

L’inscription combinée d’informations de sécurité Azure AD n’est pas disponible actuellement sur les clouds nationaux tels qu’Azure Allemagne et Azure China 21Vianet. Elle est disponible pour Azure - Gouvernement des États-Unis.

> [!IMPORTANT]
> Les utilisateurs activés pour la préversion d’origine et l’expérience d’inscription combinée améliorée voient le nouveau comportement. Les utilisateurs activés pour les deux expériences ne voient que la nouvelle expérience de la page Mon profil. La nouvelle page *Mon profil* s’aligne sur l’apparence de l’inscription combinée et offre une expérience transparente pour les utilisateurs. Les utilisateurs peuvent voir la page Mon profil en accédant à [https://myprofile.microsoft.com](https://myprofile.microsoft.com).
>
> Un message d’erreur peut s’afficher lors de la tentative d’accès à l’option Informations de sécurité, tel que : « Désolés, nous ne pouvons pas vous connecter ». Vérifiez que vous n’avez pas d’objet de configuration ou de stratégie de groupe qui bloque les cookies tiers sur le navigateur web.

Les pages *Mon profil* sont localisées sur la base des paramètres linguistiques de l’ordinateur qui y accède. Microsoft stocke la dernière langue utilisée dans le cache du navigateur, afin que les tentatives suivantes d’accès aux pages continuent de s’afficher dans la dernière langue utilisée. Si vous effacez le cache, les pages sont réaffichées.

Pour imposer l’utilisation d’une langue spécifique, vous pouvez ajouter `?lng=<language>` à la fin de l’URL, où `<language>` est le code de la langue que vous souhaitez utiliser.

![Configurer la réinitialisation SSPR ou d’autres méthodes de vérification de sécurité](media/howto-registration-mfa-sspr-combined/combined-security-info-my-profile.png)

## <a name="methods-available-in-combined-registration"></a>Méthodes disponibles dans l’inscription combinée

L’inscription combinée prend en charge les méthodes et les actions d’authentification suivantes :

| Méthode | Inscrire | Modifier | DELETE |
| --- | --- | --- | --- |
| Microsoft Authenticator | Oui (jusqu'à 5) | Non | Oui |
| Autre application d’authentification | Oui (jusqu'à 5) | Non | Oui |
| Jeton matériel | Non | Non | Oui |
| Téléphone | Oui | Oui | Oui |
| Autre téléphone | Oui | Oui | Oui |
| Téléphone de bureau | Oui | Oui | Oui |
| E-mail | Oui | Oui | Oui |
| Questions de sécurité | Oui | Non | Oui |
| Mots de passe d'application | Oui | Non | Oui |
| Clés de sécurité FIDO2<br />*Mode géré uniquement à partir de la page [Informations de sécurité](https://mysignins.microsoft.com/security-info)*| Oui | Oui | Oui |

> [!NOTE]
> Les mots de passe d’application sont disponibles uniquement pour les utilisateurs qui ont été paramétrés pour l’authentification multifacteur. Les mots de passe d’application ne sont pas disponibles pour les utilisateurs activés pour l’authentification multifacteur via une stratégie d’accès conditionnel.

Les utilisateurs peuvent définir l’une des options suivantes comme méthode d’authentification multifacteur par défaut :

- Microsoft Authenticator – notification.
- Application d’authentification ou jeton matériel – code.
- Appel téléphonique.
- SMS.

Au fur et à mesure que nous ajouterons de nouvelles méthodes d’authentification à Azure AD, ces méthodes seront disponibles dans le cadre d’une inscription combinée.

## <a name="combined-registration-modes"></a>Modes d’inscription combinée

Il existe deux modes d’inscription combinée : le mode d’interruption et le mode de gestion.

- Le **mode d’interruption** est une expérience de type Assistant, présentée aux utilisateurs lorsqu’ils s’inscrivent ou actualisent les informations de sécurité à la connexion.
- Le **mode de gestion** fait partie du profil utilisateur et permet aux utilisateurs de gérer leurs informations de sécurité.

Pour ces deux modes, les utilisateurs qui se sont inscrits précédemment à une méthode pouvant être utilisée pour l’authentification multifacteur doivent effectuer une authentification multifacteur avant de pouvoir accéder à leurs informations de sécurité. Les utilisateurs doivent confirmer leurs informations avant de continuer à utiliser leurs méthodes précédemment inscrites. 

### <a name="interrupt-mode"></a>Mode d’interruption

L’inscription combinée respecte les stratégies d’authentification multifacteur et de réinitialisation SSPR, si les deux sont activées pour votre locataire. Ces stratégies contrôlent le fait que l’inscription d’un utilisateur est interrompue ou non pendant la connexion, ainsi que les méthodes d’inscription disponibles.

Voici quelques exemples de scénarios dans lesquels les utilisateurs peuvent être invités à s’inscrire ou à actualiser leurs informations de sécurité :

- *Inscription à l’authentification multifacteur appliquée via Identity Protection :* les utilisateurs sont invités à s’inscrire lors de la connexion. Ils s’inscrivent à des méthodes d’authentification multifacteur et de réinitialisation SSPR (si l’utilisateur est activé pour SSPR).
- *Inscription à l’authentification multifacteur appliquée via l’authentification multifacteur par utilisateur :* les utilisateurs sont invités à s’inscrire lors de la connexion. Ils s’inscrivent à des méthodes d’authentification multifacteur et de réinitialisation SSPR (si l’utilisateur est activé pour SSPR).
- *Inscription à l’authentification multifacteur appliquée via l’accès conditionnel ou d’autres stratégies :* les utilisateurs sont invités à s’inscrire lorsqu’ils utilisent une ressource qui nécessite l’authentification multifacteur. Ils s’inscrivent à des méthodes d’authentification multifacteur et de réinitialisation SSPR (si l’utilisateur est activé pour SSPR).
- *Inscription à la réinitialisation de mot de passe en libre-service appliquée :* les utilisateurs sont invités à s’inscrire lors de la connexion. Ils s’inscrivent uniquement aux méthodes SSPR.
- *Actualisation de la réinitialisation de mot de passe en libre-service appliquée :* les utilisateurs doivent vérifier leurs informations de sécurité à la fréquence définie par l’administrateur. Les informations sont montrées aux utilisateurs et ces derniers peuvent confirmer les informations actuelles ou apporter des modifications si nécessaire.

Lorsque l’inscription est appliquée, les utilisateurs peuvent voir le nombre minimal de méthodes nécessaires pour être conformes aux stratégies d’authentification multifacteur et SSPR, de la plus à la moins sécurisée.

Examinez l’exemple de scénario suivant :

- Un utilisateur est activé pour la réinitialisation SSPR. La stratégie SSPR nécessitait deux méthodes pour réinitialiser et activait le code de l’application mobile, le courrier et le téléphone.
- Cet utilisateur est tenu de s’inscrire à deux méthodes.
   - L’utilisateur voit l’application d’authentification et le téléphone par défaut.
   - L’utilisateur peut choisir de s’inscrire par e-mail à la place de l’application d’authentification ou du téléphone.

L’organigramme suivant décrit les méthodes qui sont montrées à un utilisateur quand son inscription est interrompue pendant la connexion :

![Organigramme des informations de sécurité combinée](media/concept-registration-mfa-sspr-combined/combined-security-info-flow-chart.png)

Si vous avez activé l’authentification multifacteur et la réinitialisation SSPR, nous vous recommandons d’appliquer l’inscription à l’authentification multifacteur.

Si la stratégie SSPR nécessite que les utilisateurs vérifient leurs informations de sécurité à intervalles réguliers, les utilisateurs sont interrompus pendant la connexion et toutes leurs méthodes inscrites leur sont montrées. Ils peuvent confirmer les informations actuelles si elle sont à jour, ou apporter des modifications si nécessaire. Les utilisateurs doivent effectuer une authentification multifacteur lors de l’accès à cette page.

### <a name="manage-mode"></a>Mode de gestion

Les utilisateurs peuvent accéder au mode de gestion en accédant à [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo) ou en sélectionnant **Informations de sécurité** dans Mon profil. Là, les utilisateurs peuvent ajouter des méthodes, supprimer ou modifier les méthodes existantes, modifier la méthode par défaut et bien plus encore.

## <a name="key-usage-scenarios"></a>Principaux scénarios d’utilisation

### <a name="set-up-security-info-during-sign-in"></a>Configurer les informations de sécurité pendant la connexion

Un administrateur a appliqué l’inscription.

Un utilisateur n’a pas configuré toutes les informations de sécurité requises et accède au portail Azure. Après avoir entré le nom d’utilisateur et le mot de passe, l’utilisateur est invité à configurer les informations de sécurité. Ensuite, l’utilisateur suit les étapes indiquées dans l’Assistant pour configurer les informations de sécurité requises. Si vos paramètres le permettent, l’utilisateur peut choisir de configurer d’autres méthodes que celles affichées par défaut. À l’issue de l’Assistant, les utilisateurs passent en revue les méthodes qu'ils configurent et leur méthode par défaut pour l’authentification multifacteur. Pour terminer le processus de configuration, l’utilisateur confirme les informations et continue vers le portail Azure.

### <a name="set-up-security-info-from-my-profile"></a>Configurer les informations de sécurité à partir de Mon profil

Un administrateur n’a pas appliqué l’inscription.

Un utilisateur qui n’a pas encore configuré toutes les informations de sécurité requises accède à [https://myprofile.microsoft.com](https://myprofile.microsoft.com). L’utilisateur sélectionne **Informations de sécurité** dans le volet gauche. Là, l’utilisateur choisit d’ajouter une méthode, sélectionne une des méthodes disponibles et suit la procédure requise pour configurer cette méthode. Quand il a terminé, l’utilisateur voit la méthode configurée sur la page Informations de sécurité.

### <a name="delete-security-info-from-my-profile"></a>Supprimer les informations de sécurité à partir de Mon profil

Un utilisateur qui a configuré précédemment au moins une méthode accède à [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo). L’utilisateur choisit de supprimer une des méthodes précédemment inscrites. Une fois terminé, l’utilisateur ne voit plus cette méthode sur la page Informations de sécurité.

### <a name="change-the-default-method-from-my-profile"></a>Modifier la méthode par défaut à partir de Mon profil

Un utilisateur qui a configuré précédemment au moins une méthode pouvant être utilisée pour l’authentification multifacteur accède à [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo). L’utilisateur remplace la méthode par défaut actuelle par une autre méthode par défaut. Une fois terminé, l’utilisateur voit la nouvelle méthode par défaut sur la page Informations de sécurité.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer, consultez les tutoriels consacrés à l'[activation de la réinitialisation de mot de passe en libre-service](tutorial-enable-sspr.md) et à l'[activation d'Azure AD Multi-Factor Authentication](tutorial-enable-azure-mfa.md).

Découvrez comment [activer l’inscription combinée dans votre locataire](howto-registration-mfa-sspr-combined.md) ou [obliger les utilisateurs à se réinscrire à certaines méthodes d’authentification](howto-mfa-userdevicesettings.md#manage-user-authentication-options).

Vous pouvez également consulter les [méthodes disponibles pour Azure AD Multi-Factor Authentication et SSPR](concept-authentication-methods.md).
