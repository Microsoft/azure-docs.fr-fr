---
title: 'Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à une application mobile Workday | Microsoft Docs'
description: Découvrez comment configurer l’authentification unique entre Azure Active Directory et une application mobile Workday.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 08/31/2020
ms.author: jeedes
ms.openlocfilehash: ef1ca41f54a15554a04fa3edf608bb13f5fb3398
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182017"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-workday-mobile-application"></a>Tutoriel : Intégration de l’authentification unique (SSO) Azure Active Directory à une application mobile Workday

Dans ce tutoriel, vous allez apprendre à intégrer Azure Active Directory (Azure AD), l’accès conditionnel et Intune à une application mobile Workday. Quand vous intégrez une application mobile Workday à Microsoft, vous pouvez :

* Garantir que les appareils sont conformes à vos stratégies avant la connexion.
* Ajouter des contrôles à une application mobile Workday pour garantir que les utilisateurs accèdent de manière sécurisée aux données d’entreprise. 
* Contrôler dans Azure AD qui a accès à Workday.
* Permettre à vos utilisateurs de se connecter automatiquement à Workday avec leur compte Azure AD.
* gérer vos comptes à un emplacement central : le portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour commencer :

* Intégrer Workday à Azure AD.
* Lisez [Intégration de l’authentification unique (SSO) Azure Active Directory à Workday](./workday-tutorial.md).

## <a name="scenario-description"></a>Description du scénario

Dans ce tutoriel, vous allez configurer et tester des stratégies d’accès conditionnel Azure AD et Intune avec une application mobile Workday.

Pour activer l’authentification unique (SSO), vous pouvez configurer l’application fédérée Workday avec Azure AD. Pour plus d’informations, consultez [Intégration de l’authentification unique (SSO) Azure Active Directory à Workday](./workday-tutorial.md).

> [!NOTE] 
> Workday ne prend pas en charge les stratégies de protection de l’application d’Intune. Vous devez utiliser la gestion des appareils mobiles pour avoir recours à l’accès conditionnel.


## <a name="ensure-users-have-access-to-workday-mobile-application"></a>Vérifier que les utilisateurs ont accès à une application mobile Workday

Configurez Workday pour autoriser l’accès à ses applications mobiles. Vous devez configurer les stratégies suivantes pour l’application mobile Workday :

1. Accédez au rapport Stratégies de sécurité de domaine pour la zone fonctionnelle.
1. Sélectionnez la stratégie de sécurité appropriée :
    * Utilisation mobile - Android
    * Utilisation mobile - iPad
    * Utilisation mobile - iPhone
1. Sélectionnez **Modifier les autorisations**.
1. Cochez la case **View ou Modify** (Afficher ou modifier) pour accorder aux groupes de sécurité l’accès aux éléments sécurisables de rapport ou de tâche.
1. Cochez la case **Get ou Put** (Obtenir ou Placer) pour accorder aux groupes de sécurité l’accès aux actions sécurisables d’intégration et de rapport ou de tâche.

Activez les changements de stratégie de sécurité en attente en exécutant **Activer les changements de stratégie de sécurité en attente**.

## <a name="open-workday-sign-in-page-in-workday-mobile-browser"></a>Ouvrir la page de connexion Workday dans le navigateur mobile Workday

Pour appliquer l’accès conditionnel à une application mobile Workday, vous devez ouvrir cette application dans un navigateur externe. Dans **Edit Tenant Setup - Security** (Modifier la configuration du locataire - Sécurité), sélectionnez **Enable Mobile Browser SSO for Native Apps** (Activer l’authentification unique du navigateur mobile pour les applications natives). Cette opération nécessite qu’un navigateur approuvé par Intune soit installé sur l’appareil pour iOS et dans le profil professionnel pour Android.

![Capture d’écran de la connexion du navigateur mobile Workday.](./media/workday-tutorial/mobile-browser.png)

## <a name="set-up-conditional-access-policy"></a>Configurer la stratégie d’accès conditionnel

Cette stratégie n’affecte que la connexion sur un appareil iOS ou Android. Si vous voulez l’étendre à toutes les plateformes, sélectionnez **Any Device** (Tout appareil). Cette stratégie nécessite que l’appareil soit conforme à la stratégie et le vérifie par le biais d’Intune. Étant donné qu’Android possède des profils professionnels, cela empêche les utilisateurs de se connecter dans Workday, sauf s’ils se connectent par le biais de leur profil professionnel et qu’ils ont installé l’application avec le portail d’entreprise Intune. Une étape supplémentaire est nécessaire pour iOS afin de garantir que la même situation s’applique.

Workday prend en charge les contrôles d’accès suivants :
* Exiger l’authentification multifacteur
* Exiger que l’appareil soit marqué comme conforme

L’application Workday ne prend pas en charge les opérations suivantes :
* Demander une application cliente approuvée
* Exiger une stratégie de protection des applications (préversion)

Pour configurer Workday en tant qu’appareil géré, effectuez les étapes suivantes :

![Capture d’écran des options Appareils gérés uniquement et Applications ou actions cloud.](./media/workday-tutorial/managed-devices-only.png)

1. Sélectionnez **Accueil** > **Microsoft Intune** > **Stratégies d’accès conditionnel**. Sélectionnez ensuite **Appareils gérés uniquement**. 

1. Dans **Appareils gérés uniquement**, sous **Nom**, sélectionnez **Appareils gérés uniquement**, puis **Applications ou actions cloud**.

1. Dans **Applications ou actions cloud** :

    a. Définissez **Sélectionner ce à quoi cette stratégie s’applique** sur **Applications cloud**.

    b. Dans **Inclure**, choisissez **Sélectionner les applications**.

    c. Dans la liste **Sélectionner**, choisissez **Workday**.

    d. Sélectionnez **Terminé**.

1. Définissez **Appliquer la stratégie** sur **Activé**.

1. Sélectionnez **Enregistrer**.

Pour l’accès **Octroyer**, effectuez les étapes suivantes :

![Capture d’écran des options Appareils gérés uniquement et Accorder.](./media/workday-tutorial/managed-devices-only-2.png)

1. Sélectionnez **Accueil** > **Microsoft Intune** > **Stratégies d’accès conditionnel**. Sélectionnez ensuite **Appareils gérés uniquement**. 

1. Dans **Appareils gérés uniquement**, sous **Nom**, sélectionnez **Appareils gérés uniquement**. Sous **Contrôles d’accès**, sélectionnez **Accorder**.

1. Dans **Accorder** :

    a. Pour la sélection des contrôles à appliquer, choisissez **Accorder l’accès**.

    b. Sélectionnez **Exiger que l'appareil soit marqué comme conforme**.

    c. Sélectionnez **Demander un des contrôles sélectionnés**.

    d. Choisissez **Sélectionner**.

1. Définissez **Appliquer la stratégie** sur **Activé**.

1. Sélectionnez **Enregistrer**.

## <a name="set-up-device-compliance-policy"></a>Configurer la stratégie de conformité de l’appareil

Pour garantir que les appareils iOS ne peuvent se connecter que par le biais d’une application Workday managée par la gestion des appareils mobiles, vous devez bloquer l’application App Store en ajoutant **com.workday.workdayapp** à la liste des applications restreintes. Ainsi, seuls les appareils sur lesquels Workday est installé par le biais du portail d’entreprise peuvent accéder à Workday. Dans le cas d’un navigateur, les appareils peuvent uniquement accéder à Workday s’ils sont gérés par Intune et qu’ils utilisent un navigateur géré.

![Capture d’écran de la stratégie de conformité des appareils iOS.](./media/workday-tutorial/ios-policy.png)

## <a name="set-up-intune-app-configuration-policies"></a>Configurer des stratégies de configuration d’application Intune

| Scénario | Paires clé-valeur |
|----------------------------------------------------------------------------------------   |-----------|
| Renseigner automatiquement les champs Locataire et Adresse web pour :<br>● Workday sur Android quand vous activez les profils Android for Work.<br>● Workday sur iPad et iPhone.     | Utilisez ces valeurs pour configurer votre locataire : <br>● Clé de configuration = `UserGroupCode`<br>● Type de valeur = Chaîne <br>● Valeur de configuration = nom de votre locataire. Exemple : `gms`<br>Utilisez ces valeurs pour configurer votre adresse web :<br>●  Clé de configuration = `AppServiceHost`<br>●   Type de valeur = Chaîne<br>●    Valeur de configuration = URL de base de votre locataire. Exemple : `https://www.myworkday.com`                                |   |
| Désactiver ces actions pour Workday sur iPad et iPhone :<br>●    Couper, copier et coller<br>●   Imprimer                       | Définissez la valeur (booléenne) sur `False` sur ces clés pour désactiver la fonctionnalité :<br>●   `AllowCutCopyPaste`<br>●    `AllowPrint`    |
| Désactiver les captures d’écran pour Workday sur Android. |Définissez la valeur (booléenne) sur `False` sur la clé `AllowScreenshots` pour désactiver la fonctionnalité.|
| Désactiver les mises à jour suggérées pour vos utilisateurs.|Définissez la valeur (booléenne) sur `False` sur la clé `AllowSuggestedUpdates` pour désactiver la fonctionnalité.|
|Personnaliser l’URL de l’App Store pour diriger les utilisateurs mobiles vers l’App Store de votre choix.|Utilisez ces valeurs pour changer l’URL de l’App Store :<br>● Clé de configuration = `AppUpdateURL`<br>● Type de valeur = Chaîne<br> ●   Valeur de configuration = URL de l’App Store|
|       |


## <a name="ios-configuration-policies"></a>Stratégies de configuration iOS

1. Accédez au [portail Azure](https://portal.azure.com/) et connectez-vous.
1. Recherchez **Intune** ou sélectionnez le widget dans la liste.
1. Accédez à **Applications clientes** > **Applications** > **Stratégies de configuration des applications**. Sélectionnez ensuite **+ Ajouter** > **Appareils gérés**.
1. Entrez un nom.
1. Sous **Plateforme**, choisissez **iOS/iPadOS**.
1. Sous **Application associée**, choisissez l’application Workday pour iOS que vous avez ajoutée.
1. Sélectionnez **Paramètres de configuration**. Sous **Format des paramètres de configuration**, sélectionnez **Entrer des données XML**.
1. Voici un exemple de fichier XML. Ajoutez les configurations que vous voulez appliquer. Remplacez `STRING_VALUE` par la chaîne que vous voulez utiliser. Remplacez `<true /> or <false />` par `<true />` ou `<false />`. Si vous n’ajoutez pas de configuration, cet exemple fonctionne comme s’il avait la valeur `True`.

    ```
    <dict>
    <key>UserGroupCode</key>
    <string>STRING_VALUE</string>
    <key>AppServiceHost</key>
    <string>STRING_VALUE</string>
    <key>AllowCutCopyPaste</key>
    <true /> or <false />
    <key>AllowPrint</key>
    <true /> or <false />
    <key>AllowSuggestedUpdates</key>
    <true /> or <false />
    <key>AppUpdateURL</key>
    <string>STRING_VALUE</string>
    </dict>

    ```
1. Sélectionnez **Ajouter**.
1. Actualisez la page, puis sélectionnez la stratégie nouvellement créée.
1. Sélectionnez **Affectations**, puis choisissez à qui vous voulez que l’application s’applique.
1. Sélectionnez **Enregistrer**.

## <a name="android-configuration-policies"></a>Stratégies de configuration Android

1. Accédez au [portail Azure](https://portal.azure.com/) et connectez-vous.
2. Recherchez **Intune** ou sélectionnez le widget dans la liste.
3. Accédez à **Applications clientes** > **Applications** > **Stratégies de configuration des applications**. Sélectionnez ensuite **+ Ajouter** > **Appareils gérés**.
5. Entrez un nom. 
6. Sous **Plateforme**, choisissez **Android**.
7. Sous **Application associée**, choisissez l’application Workday pour Android que vous avez ajoutée.
8. Sélectionnez **Paramètres de configuration**. Sous **Format des paramètres de configuration**, sélectionnez **Entrer des données JSON**.