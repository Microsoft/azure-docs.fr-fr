---
title: Configurer Identity Protection et l’accès conditionnel dans Azure AD B2C
description: Découvrez comment configurer Identity Protection et l’accès conditionnel pour votre locataire Azure AD B2C afin d’afficher les événements relatifs aux connexions risquées et autres événements à risque, et de créer des stratégies basées sur les détections des risques.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: overview
ms.date: 09/01/2020
ms.author: mimart
author: msmimart
manager: celested
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2472183673e5f06f5664a306a69d14c2eaf5f82d
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94949766"
---
# <a name="set-up-identity-protection-and-conditional-access-in-azure-ad-b2c"></a>Configurer Identity Protection et l’accès conditionnel dans Azure AD B2C

[!INCLUDE [b2c-public-preview-feature](../../includes/active-directory-b2c-public-preview.md)]

Identity Protection assure la détection des risques en continu pour votre locataire Azure AD B2C. Si le niveau tarifaire de votre locataire Azure AD B2C est Premium P2, vous pouvez afficher les événements à risque Identity Protection détaillés dans le portail Azure. Vous pouvez également utiliser des stratégies d’[accès conditionnel](../active-directory/conditional-access/overview.md) basées sur ces détections des risques afin de déterminer les actions à effectuer et d’appliquer des stratégies organisationnelles.

## <a name="prerequisites"></a>Prérequis

- Votre locataire Azure AD B2C doit être [lié à un abonnement Azure AD](billing.md#link-an-azure-ad-b2c-tenant-to-a-subscription).
- Azure AD B2C Premium P2 est requis pour l’utilisation de l’accès conditionnel basé sur les risques des utilisateurs et des connexions. Si nécessaire, [passez au niveau tarifaire Azure AD B2C Premium P2](./billing.md). 
- Pour gérer Identity Protection et l’accès conditionnel dans votre locataire B2C, vous avez besoin d’un compte disposant du rôle Administrateur général ou Administrateur de la sécurité.
- Pour utiliser ces fonctionnalités dans votre locataire, vous devez d’abord basculer vers le niveau tarifaire Azure AD B2C Premium P2.

## <a name="set-up-identity-protection"></a>Configurer Identity Protection

Identity Protection est activé par défaut. Pour pouvoir afficher les événements à risque Identity Protection dans votre locataire Azure AD B2C, il vous suffit de lier votre locataire Azure AD B2C à un abonnement Azure AD et de sélectionner le niveau tarifaire Azure AD B2C Premium P2. Vous pouvez afficher des rapports détaillés sur les événements à risque dans le portail Azure.

### <a name="supported-identity-protection-risk-detections"></a>Détections des risques Identity Protection prises en charge

Les détections des risques suivantes sont actuellement prises en charge pour Azure AD B2C :  

|Type de détection des risques  |Description  |
|---------|---------|
| Voyage inhabituel     | Connexion à partir d’un emplacement inhabituel par rapport aux dernières connexions de l’utilisateur.        |
|Adresse IP anonyme     | Connexion à partir d'une adresse IP anonyme (par exemple : navigateur Tor, VPN anonymes).        |
|Adresse IP liée à un programme malveillant     | Connexion à partir d’une adresse IP liée à un programme malveillant.         |
|Propriétés de connexion inhabituelles     | Connexion avec des propriétés inhabituelles pour l’utilisateur concerné.        |
|L’administrateur a confirmé que cet utilisateur est compromis    | Un administrateur a indiqué qu’un utilisateur a été compromis.             |
|Pulvérisation de mots de passe     | Connectez-vous par le biais d’une attaque par pulvérisation de mots de passe.      |
|Azure AD Threat Intelligence     | Les sources de renseignements sur les menaces internes et externes de Microsoft ont identifié un modèle d’attaque connu.        |

## <a name="view-risk-events-for-your-azure-ad-b2c-tenant"></a>Afficher les événements à risque pour votre locataire Azure AD B2C

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.

1. Dans la Portail Azure, recherchez et sélectionnez **Azure AD B2C**.

1. Sous **Sécurité**, sélectionnez **Utilisateurs à risque (préversion)** .

   ![Utilisateurs à risque](media/conditional-access-identity-protection-setup/risky-users.png)

1. Sous **Sécurité**, sélectionnez **Détections de risque (préversion)** .

   ![Détections de risques](media/conditional-access-identity-protection-setup/risk-detections.png)

## <a name="add-a-conditional-access-policy"></a>Ajouter une stratégie d’accès conditionnel 

Pour ajouter une stratégie d’accès conditionnel basée sur les détections des risques Identity Protection, vérifiez que les paramètres de sécurité par défaut sont désactivés pour votre locataire Azure AD B2C, puis créez des stratégies d’accès conditionnel.

### <a name="to-disable-security-defaults"></a>Pour désactiver les paramètres de sécurité par défaut

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.

3. Dans le Portail Azure, recherchez et sélectionnez **Azure Active Directory**.

4. Sélectionnez **Propriétés**, puis **Gérer les paramètres de sécurité par défaut**.

   ![Désactiver les paramètres de sécurité par défaut](media/conditional-access-identity-protection-setup/disable-security-defaults.png)

5. Sous Activer les paramètres de sécurité par défaut, sélectionnez Non. 

   ![Définissez le bouton bascule Activer les paramètres de sécurité par défaut sur la valeur Non](media/conditional-access-identity-protection-setup/enable-security-defaults-toggle.png)

### <a name="to-create-a-conditional-access-policy"></a>Pour créer une stratégie d’accès conditionnel

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.

1. Dans la Portail Azure, recherchez et sélectionnez **Azure AD B2C**.

1. Sous **Sécurité**, sélectionnez **Accès conditionnel (préversion)** . La page **Stratégies d’accès conditionnel** s’ouvre. 

1. Sélectionnez **Nouvelle stratégie** et suivez la documentation sur l’accès conditionnel Azure AD pour créer une stratégie. Par exemple :

   - [Accès conditionnel basé sur les risques de connexion : Activer avec la stratégie d’accès conditionnel](../active-directory/conditional-access/howto-conditional-access-policy-risk.md#enable-with-conditional-access-policy)

   > [!IMPORTANT]
   > Lorsque vous sélectionnez les utilisateurs auxquels vous souhaitez appliquer la stratégie, ne sélectionnez pas uniquement **Tous les utilisateurs**, car vous risqueriez de ne pas pouvoir vous connecter.

## <a name="test-the-conditional-access-policy"></a>Tester la stratégie d’accès conditionnel

1. Créez une stratégie d’accès conditionnel comme indiqué ci-dessus, avec les paramètres suivants :
   
   - Pour **Utilisateurs et groupes**, sélectionnez l’utilisateur de test. Ne sélectionnez pas **Tous les utilisateurs**, car vous ne pourriez plus vous connecter.
   - Pour **Applications ou actions cloud**, choisissez **Sélectionner les applications**, puis choisissez votre application par partie de confiance.
   - Pour Conditions, sélectionnez **Risque de connexion** et les niveaux de risque **Élevé**, **Moyen** et **Faible**.
   - Pour **Accorder**, choisissez **Bloquer l’accès**.

      ![Choisissez Bloquer l’accès](media/conditional-access-identity-protection-setup/test-conditional-access-policy.png)

1. Activez votre stratégie d’accès conditionnel de test en sélectionnant **Créer**.

1. Simulez une connexion risquée à l’aide du [navigateur Tor](https://www.torproject.org/download/). 

1. Dans le jeton décodé jwt.ms de la tentative de connexion, vous devez voir que la connexion a été bloquée :

   ![Tester une connexion bloquée](media/conditional-access-identity-protection-setup/test-blocked-sign-in.png)

## <a name="review-conditional-access-outcomes-in-the-audit-report"></a>Examiner les résultats de l’accès conditionnel dans le rapport d’audit

Pour examiner le résultat d’un événement d’accès conditionnel

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.

3. Dans la Portail Azure, recherchez et sélectionnez **Azure AD B2C**.

4. Sous **Activités**, sélectionnez **Journaux d’audit**.

5. Filtrez le journal d’audit en choisissant **B2C** comme **Catégorie** et **IdentityProtection** comme **Type de ressource d’activité**. Ensuite, sélectionnez **Appliquer**.

6. Passez en revue l’activité d’audit des sept derniers jours. Les types d’activité suivants sont inclus :

   - **Évaluer les stratégies d’accès conditionnel** : cette entrée du journal d’audit indique qu’une évaluation de l’accès conditionnel a été effectuée pendant une authentification.
   - **Atténuer l’utilisateur** : cette entrée indique que l’accord ou les exigences d’une stratégie d’accès conditionnel ont été satisfaites par l’utilisateur final, et que cette activité a été signalée au moteur de risque afin de réduire (atténuer) le risque utilisateur.

7. Sélectionnez une entrée de journal **Évaluer la stratégie d’accès conditionnel** dans la liste pour ouvrir la page **Détails de l’activité : Journal d’audit**, qui affiche les identificateurs du journal d’audit, ainsi que ces informations dans la section **Détails supplémentaires** :

   - ConditionalAccessResult : accord requis par l’évaluation de la stratégie conditionnelle.
   - AppliedPolicies : liste de toutes les stratégies d’accès conditionnel où les conditions ont été satisfaites et les stratégies sont activées.
   - ReportingPolicies : liste des stratégies d’accès conditionnel qui ont été définies en mode rapport seul et où les conditions ont été satisfaites.

## <a name="next-steps"></a>Étapes suivantes

[Ajouter l’accès conditionnel à un flux d’utilisateur](conditional-access-user-flow.md)