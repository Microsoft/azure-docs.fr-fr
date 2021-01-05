---
title: Connexion par clé de sécurité sans mot de passe pour Windows – Azure Active Directory
description: Découvrez connexion activer la connexion par clé de sécurité sans mot de passe à Azure Active Directory à l’aide de clés de sécurité FIDO2 (préversion)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/24/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: librown, aakapo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 04a46a691b2f629b64cfe09c22813b05c593af1c
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743460"
---
# <a name="enable-passwordless-security-key-sign-in-to-windows-10-devices-with-azure-active-directory-preview"></a>Activer la connexion par clé de sécurité sans mot de passe à des appareils Windows 10 à l’aide d’Azure Active Directory (préversion)

Ce document met l’accent sur l’activation de l’authentification sans mot de passe, basée sur une clé de sécurité FIDO2, avec des appareils Windows 10. À la fin de cet article, vous serez en mesure de vous connecter à vos appareils Windows 10 joints à Azure AD et à ceux joints à Azure AD Hybride avec votre compte Azure AD, à l’aide d’une clé de sécurité FIDO2.

> [!NOTE]
> Les clés de sécurité FIDO2 sont une fonctionnalité d’évaluation publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="requirements"></a>Spécifications

| Type d’appareil | Appareil joints Azure AD | Appareils joints Azure AD hybrides |
| --- | --- | --- |
| [Azure AD Multi-Factor Authentication](howto-mfa-getstarted.md) | X | X |
| [Inscription d’informations de sécurité combinée – Préversion](concept-registration-mfa-sspr-combined.md) | X | X |
| [Clés de sécurité FIDO2](concept-authentication-passwordless.md#fido2-security-keys) compatibles | X | X |
| WebAuthN nécessite Windows 10 version 1903 ou plus | X | X |
| [Les appareils joints à Azure AD](../devices/concept-azure-ad-join.md) nécessitent Windows 10 version 1909 ou ultérieure | X |   |
| Les [appareils de jointure Azure AD Hybride](../devices/concept-azure-ad-join-hybrid.md) nécessitent Windows 10 version 2004 ou ultérieure |   | X |
| Contrôleurs de domaine Windows Server 2016/2019 entièrement corrigés |   | X |
| [Azure AD Connect](../hybrid/how-to-connect-install-roadmap.md#install-azure-ad-connect) version 1.4.32.0 ou ultérieure |   | X |
| [Microsoft Intune](/intune/fundamentals/what-is-intune) (facultatif) | X | X |
| Package d’approvisionnement (facultatif) | X | X |
| Stratégie de groupe (facultatif) |   | X |

### <a name="unsupported-scenarios"></a>Scénarios non pris en charge

Les scénarios suivants ne sont pas pris en charge :

- Déploiement de Windows Server joint à un domaine Active Directory Domain Services (AD DS) (appareils locaux uniquement).
- Scénarios RDP, VDI et Citrix utilisant une clé de sécurité.
- S/MIME utilisant une clé de sécurité.
- Identification utilisant une clé de sécurité.
- Connexion à un serveur à l’aide d’une clé de sécurité.
- Si vous n’avez pas utilisé votre clé de sécurité pour vous connecter à votre appareil lorsqu’il est en ligne, vous ne pouvez pas l’utiliser pour vous connecter ou le déverrouiller hors connexion.
- Connexion ou déverrouillage d’un appareil Windows 10 avec une clé de sécurité contenant plusieurs comptes Azure AD. Ce scénario utilise le dernier compte ajouté à la clé de sécurité. WebAuthN permet aux utilisateurs de choisir le compte qu’ils souhaitent utiliser.
- Déverrouillez un appareil exécutant Windows 10 version 1809. Vous bénéficierez d’une expérience optimale sur Windows 10 version 1903 ou ultérieure.

## <a name="prepare-devices-for-preview"></a>Préparer les appareils pour la préversion

Les appareils joints à Azure AD avec lesquels vous exécutez des pilotes pendant l’évaluation des fonctionnalités doivent exécuter Windows 10 version 1909 ou ultérieure.

Les appareils de jointure Azure AD Hybride doivent exécuter Windows 10 version 2004 ou ultérieure.

## <a name="enable-security-keys-for-windows-sign-in"></a>Activer les clés de sécurité pour la connexion Windows

Les organisations peuvent choisir d’utiliser une ou plusieurs des méthodes suivantes pour activer l’utilisation de clés de sécurité pour la connexion Windows sur la base des exigences de l’organisation :

- [Activer avec Intune](#enable-with-intune)
- [Déploiement Intune ciblé](#targeted-intune-deployment)
- [Activer avec un package d’approvisionnement](#enable-with-a-provisioning-package)
- [Activer avec une stratégie de groupe (appareils joints à Azure AD Hybride uniquement)](#enable-with-group-policy)

> [!IMPORTANT]
> Les organisations disposant d’**appareils joints à Azure AD Hybride** doivent **également** suivre les étapes décrites dans l’article [Activer l’authentification FIDO2 pour les ressources locales](howto-authentication-passwordless-security-key-on-premises.md) avant que l’authentification par clé de sécurité FIDO2 ne fonctionne sous Windows 10.
>
> Les organisations disposant d’**appareils joints à Azure AD** doivent effectuer cette opération avant que leurs appareils puissent s’authentifier auprès des ressources locales avec des clés de sécurité FIDO2.

### <a name="enable-with-intune"></a>Activer avec Intune

Pour activer l’utilisation des clés de sécurité à l’aide d’Intune, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez à **Microsoft Intune** > **Inscription d’appareils** > **Inscription Windows** > **Windows Hello Entreprise** > **Propriétés**.
1. Sous **Paramètres**, définissez **Utiliser des clés de sécurité pour la connexion** sur **Activé**.

La configuration de clés de sécurité pour la connexion ne dépend pas de la configuration de Windows Hello Entreprise.

### <a name="targeted-intune-deployment"></a>Déploiement Intune ciblé

Pour cibler des groupes d’appareils spécifiques pour activer le fournisseur d’informations d’identification, utilisez les paramètres personnalisés suivants via Intune :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez à **Microsoft Intune** > **Configuration de l’appareil** > **Profils** > **Créer un profil**.
1. Configurez le nouveau profil avec les paramètres suivants :
   - Nom : Clés de sécurité pour la connexion Windows
   - Description : Permet d’utiliser des clés de sécurité FIDO lors de la connexion à Windows
   - Platform : Windows 10 et versions ultérieures
   - Type de profil : Custom
   - Paramètres OMA-URI personnalisés :
      - Nom : Activer les clés de sécurité FIDO pour la connexion à Windows
      - OMA-URI : ./Device/Vendor/MSFT/PassportForWork/SecurityKey/UseSecurityKeyForSignin
      - Type de données : Integer
      - Valeur : 1
1. Cette stratégie peut être attribuée à des utilisateurs, des appareils ou des groupes spécifiques. Pour plus d’informations, consultez [Attribuer des profils d’utilisateur et d’appareil dans Microsoft Intune](/intune/device-profile-assign).

![Création de stratégies de configuration d’appareil Intune personnalisées](./media/howto-authentication-passwordless-security-key/intune-custom-profile.png)

### <a name="enable-with-a-provisioning-package"></a>Activer avec un package d’approvisionnement

Pour les appareils non gérés par Intune, un package d’approvisionnement peut être installé pour activer la fonctionnalité. L’application Windows Configuration Designer peut être installée à partir du [Microsoft Store](https://www.microsoft.com/p/windows-configuration-designer/9nblggh4tx22). Pour créer un package d’approvisionnement, procédez comme suit :

1. Lancez Windows Configuration Designer.
1. Sélectionnez **Fichier** > **Nouveau projet**.
1. Donnez un nom à votre projet et notez le chemin d’accès à l’emplacement où votre projet a été créé, puis sélectionnez **Suivant**.
1. Laissez *Package d’approvisionnement* sélectionné comme **Flux de travail de projet sélectionné** et sélectionnez **Suivant**.
1. Sélectionnez *Toutes les éditions de bureau de Windows* sous **Choisir les paramètres à afficher et configurer**, puis sélectionnez **Suivant**.
1. Sélectionnez **Terminer**.
1. Dans votre projet nouvellement créé, accédez à **Paramètres d’exécution** > **WindowsHelloForBusiness** > **SecurityKeys** > **UseSecurityKeyForSignIn**.
1. Définissez **UseSecurityKeyForSignIn** sur *Activé*.
1. Sélectionnez **Exporter** > **Package d’approvisionnement**
1. Laissez les valeurs par défaut dans la fenêtre **Générer** sous **Décrire le package d’approvisionnement**, puis sélectionnez **Suivant**.
1. Laissez les valeurs par défaut dans la fenêtre **Générer** sous **Sélectionner les détails de sécurité pour le package d’approvisionnement**, et sélectionnez **Suivant**.
1. Prenez note du chemin d’accès (ou modifiez-le) dans la fenêtre **Générer** sous **Sélectionnez l’emplacement d’enregistrement du package d’approvisionnement**, et sélectionnez **Suivant**.
1. Sélectionnez **Générer** sur la page **Générer le package d’approvisionnement**.
1. Enregistrez les deux fichiers créés (*ppkg* et *cat*) dans un emplacement où vous pourrez les appliquer aux machines plus tard.
1. Pour appliquer le package d’approvisionnement que vous avez créé, consultez [Appliquer un package d’approvisionnement](/windows/configuration/provisioning-packages/provisioning-apply-package).

> [!NOTE]
> Les appareils exécutant Windows 10 version 1903 doivent également activer le mode PC partagé (*EnableSharedPCMode*). Pour en savoir plus sur l’activation de cette fonctionnalité, consultez [Configurer un PC partagé ou invité avec Windows 10](/windows/configuration/set-up-shared-or-guest-pc).

### <a name="enable-with-group-policy"></a>Activer avec la stratégie de groupe

Pour les **appareils joints à Azure AD Hybride**, les organisations peuvent configurer le paramètre de stratégie de groupe suivant pour activer la connexion par clé de sécurité FIDO. Le paramètre est accessible sous **Configuration ordinateur** > **Modèles d’administration** > **Système** > **Ouverture de session** > **Activer la connexion par clé de sécurité** :

- La définition de cette stratégie sur **Activée** permet aux utilisateurs de se connecter avec des clés de sécurité.
- La définition de cette stratégie sur **Désactivée** ou **Non configurée** empêche les utilisateurs de se connecter avec des clés de sécurité.

Ce paramètre de stratégie de groupe requiert une version mise à jour du modèle de stratégie de groupe `CredentialProviders.admx`. Ce nouveau modèle est disponible avec la prochaine version de Windows Server et avec Windows 10 20H1. Ce paramètre peut être géré à l’aide d’un appareil exécutant une de ces versions plus récentes de Windows ou de manière centralisée en suivant les instructions de la rubrique de support [Comment créer et gérer le magasin central pour les modèles d’administration de stratégie de groupe dans Windows](https://support.microsoft.com/help/3087759/how-to-create-and-manage-the-central-store-for-group-policy-administra).

## <a name="sign-in-with-fido2-security-key"></a>Se connecter avec une clé de sécurité FIDO2

Dans l’exemple ci-dessous, un utilisateur appelé Bala Sandhu a déjà approvisionné sa clé de sécurité FIDO2 en suivant les étapes décrites dans l’article précédent, portant sur [l’activation de la connexion par clé de sécurité sans mot de passe](howto-authentication-passwordless-security-key.md#user-registration-and-management-of-fido2-security-keys). Pour les appareils joints à Azure AD Hybride, assurez-vous que vous avez également [activé la connexion par clé de sécurité sans mot de passe pour les ressources locales](howto-authentication-passwordless-security-key-on-premises.md). Bala peut choisir le fournisseur d’informations d’identification de clé de sécurité à partir de l’écran de verrouillage Windows 10, et insérer la clé de sécurité pour se connecter à Windows.

![Connexion par clé de sécurité sur l’écran de verrouillage Windows 10](./media/howto-authentication-passwordless-security-key/fido2-windows-10-1903-sign-in-lock-screen.png)

### <a name="manage-security-key-biometric-pin-or-reset-security-key"></a>Gérer la clé de sécurité biométrique, le code confidentiel, ou réinitialiser la clé de sécurité

* Windows 10 version 1903 ou ultérieure
   * Les utilisateurs peuvent ouvrir les **Paramètres Windows** sur leur appareil > **Comptes** > **Clé de sécurité**
   * Les utilisateurs peuvent modifier leur code confidentiel, mettre à jour les informations biométriques ou réinitialiser leur clé de sécurité

## <a name="troubleshooting-and-feedback"></a>Résolution des problèmes de commentaires

Si vous souhaitez partager des commentaires ou si vous rencontrez des problèmes lors de l’évaluation de cette fonctionnalité, partagez vos remarques via l’application Hub de commentaires Windows en procédant comme suit :

1. Lancez le **concentrateur de commentaires** et assurez-vous que vous êtes connecté.
1. Classez vos commentaires dans les catégories suivantes avant de les envoyer :
   - Catégorie : Sécurité et confidentialité
   - Sous-catégorie : FIDO
1. Pour capturer des journaux, utilisez l’option **Recréer mon problème**

## <a name="next-steps"></a>Étapes suivantes

[Activer l’accès aux ressources locales pour les appareils joints à Azure AD et à Azure AD Hybride](howto-authentication-passwordless-security-key-on-premises.md)

[En savoir plus sur l’inscription des appareils](../devices/overview.md)

[En savoir plus sur Azure AD Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md)