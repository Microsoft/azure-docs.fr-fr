---
title: Configurer « Restez connecté ? » invite pour les comptes Azure Active Directory
description: Découvrez comment maintenir la connexion (KMSI), qui affiche l’invite « Reste connecté ? », comment le configurer dans le portail Azure Active Directory et comment résoudre les problèmes de connexion.
services: active-directory
author: CelesteDG
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: fundamentals
ms.date: 06/05/2020
ms.author: celested
ms.reviewer: asteen, jlu, hirsin
ms.collection: M365-identity-device-management
ms.openlocfilehash: bed6bc43dfc15abf2bdf9f38a5de2240d348d6fb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89320254"
---
# <a name="configure-the-stay-signed-in-prompt-for-azure-ad-accounts"></a>Configurer « Restez connecté ? » invite pour les comptes Azure AD

Maintenir la connexion (KMSI) affiche l’invite **Rester connecté ?** une fois que l’utilisateur a réussi à se connecter. Si un utilisateur répond **Oui** à cette invite, le service lui donne un [jeton d’actualisation](../develop/developer-glossary.md#refresh-token) permanent. Pour les locataires fédérés, cette invite s’affiche une fois que l’utilisateur s’est authentifié auprès du service d’identité fédéré.

Le diagramme suivant montre le déroulement de la connexion de l’utilisateur pour un locataire géré et un locataire fédéré, ainsi que la nouvelle invite de maintien de la connexion. Ce flux contient une logique intelligente afin que l’option **Rester connecté?** ne s’affiche pas si le système de Machine Learning détecte une connexion à haut risque ou une connexion à partir d’un appareil partagé.

:::image type="content" source="./media/keep-me-signed-in/kmsi-workflow.png" alt-text="Diagramme montrant le flux de connexion de l’utilisateur pour un locataire géré ou fédéré":::

> [!NOTE]
> La configuration de l’option de maintien la connexion vous oblige à utiliser les éditions Premium 1, Premium 2 ou de base de Azure Active Directory (Azure AD), ou à disposer d’une licence Microsoft 365. Pour plus d’informations sur les licences et les éditions, consultez [S’inscrire à Azure AD Premium](active-directory-get-started-premium.md).<br><br>Les clients vivant en Chine peuvent accéder aux éditions De base et Premium d’Azure AD à l’aide de l’instance mondiale d’Azure AD. Actuellement, les éditions De base et Premium d’Azure AD ne sont pas prises en charge dans le service Azure géré par 21Vianet en Chine. Pour plus d’informations, contactez-nous sur le [forum Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory/).

## <a name="configure-kmsi"></a>Configurer KMSI

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un compte d’administrateur général pour le répertoire.
1. Sélectionnez **Azure Active Directory**, puis sélectionnez **Marque de société** et **Configurer**.
1. Dans la section **Paramètres avancés**, recherchez le paramètre **Afficher pour rester connecté**.

   Ce paramètre vous permet de choisir si vos utilisateurs restent connectés à Azure AD jusqu’à ce qu’ils se déconnectent explicitement.
   * Si vous choisissez **Non**, l’option **Rester connecté ?** est masquée lorsque l’utilisateur se connecte avec succès et que l’utilisateur doit se connecter chaque fois que le navigateur est fermé puis rouvert.
   * Si vous choisissez **Oui**, l’option **Rester connecté ?** est présentée à l’utilisateur.

    :::image type="content" source="./media/keep-me-signed-in/kmsi-company-branding-advanced-settings-kmsi-1.png" alt-text="Diagramme montrant le flux de connexion de l’utilisateur pour un locataire géré ou fédéré":::

## <a name="troubleshoot-sign-in-issues"></a>Résoudre les problèmes de connexion

Si un utilisateur n’agit pas sur l’invite **Rester connecté ?** , comme indiqué dans le diagramme suivant, mais abandonne la tentative de connexion, vous verrez une entrée de journal de connexion qui indique l’interruption.

:::image type="content" source="./media/keep-me-signed-in/kmsi-stay-signed-in-prompt.png" alt-text="Diagramme montrant le flux de connexion de l’utilisateur pour un locataire géré ou fédéré":::

Les détails relatifs à l’erreur de connexion sont les suivants et mis en surbrillance dans l’exemple.

* **Code d’erreur de connexion** : 50140
* **Raison de la défaillance** : Cette erreur s’est produite suite à l’interruption de la fonction « Maintenir la connexion » lors de la connexion de l’utilisateur.

:::image type="content" source="./media/keep-me-signed-in/kmsi-sign-ins-log-entry.png" alt-text="Diagramme montrant le flux de connexion de l’utilisateur pour un locataire géré ou fédéré":::

Vous pouvez empêcher les utilisateurs de voir l’interruption en définissant le paramètre **afficher sur rester connecté** sur **Non** dans les paramètres de personnalisation avancés. Cela désactive l’invite KMSI pour tous les utilisateurs de votre répertoire Azure AD.

Vous pouvez également utiliser les contrôles de session persistante du navigateur dans l’accès conditionnel pour empêcher les utilisateurs d’afficher l’invite KMSI. Cette option vous permet de désactiver l’invite KMSI pour un groupe sélectionné d’utilisateurs (tels que les administrateurs généraux) sans affecter le comportement de connexion pour les utilisateurs restants dans l’annuaire. Pour plus d’informations sur cette option, consultez [Fréquence de connexion de l’utilisateur](../conditional-access/howto-conditional-access-session-lifetime.md). 

Pour vérifier que l’invite KMSI s’affiche uniquement lorsqu’elle peut être utile à l’utilisateur, elle est intentionnellement masquée dans les scénarios suivants :

* L’utilisateur est connecté via l’authentification unique transparente et l’Authentification Windows intégrée
* L’utilisateur est connecté via les services ADFS (Active Directory Federation Services) et l’Authentification Windows intégrée
* L’utilisateur est invité dans le locataire
* Le score de risque de l’utilisateur est élevé
* La connexion se produit pendant le processus de consentement de l’utilisateur ou de l’administrateur
* Le contrôle de la session de navigateur persistante est configuré dans une stratégie d’accès conditionnel

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les autres paramètres qui affectent le délai d’expiration de la session de connexion :

* Microsoft 365 : [délai d’expiration de la session inactive](/sharepoint/sign-out-inactive-users)
* Accès conditionnel Azure AD - [Fréquence de connexion de l’utilisateur](../conditional-access/howto-conditional-access-session-lifetime.md)
* Portail Azure - [Configurer le délai d’inactivité au niveau du répertoire](../../azure-portal/set-preferences.md#change-the-directory-timeout-setting-admin)