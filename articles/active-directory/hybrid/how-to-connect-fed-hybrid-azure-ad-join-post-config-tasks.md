---
title: 'Azure AD Connect : Tâches post-configuration de jonction Azure AD Hybride| Microsoft Docs'
description: Ce document décrit en détail les tâches post-configuration requises pour effectuer la jonction Azure AD Hybride
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: billmath
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/10/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: da5cefbacbd3851d2609a687c1948d9bcba5ffae
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88612467"
---
# <a name="post-configuration-tasks-for-hybrid-azure-ad-join"></a>Tâches post-configuration concernant la jonction Azure AD Hybride

Après avoir exécuté Azure AD Connect afin de configurer votre organisation pour la jonction Azure AD Hybride, vous devez suivre quelques étapes supplémentaires pour finaliser cette configuration.  Exécutez uniquement les étapes qui s’appliquent à vos appareils.

## <a name="1-configure-controlled-rollout-optional"></a>1. Configurer un lancement contrôlé (facultatif)
Tous les appareils joints à un domaine qui exécutent Windows 10 et Windows Server 2016 s’inscrivent automatiquement auprès d’Azure AD à l’issue de la procédure de configuration. Si vous souhaitez opter pour un lancement contrôlé plutôt que pour cette inscription automatique, vous pouvez utiliser une stratégie de groupe pour activer ou désactiver de manière sélective le lancement automatique.  Vous devez définir cette stratégie de groupe avant d’exécuter l’autre procédure de configuration :
* Créez un objet de stratégie de groupe dans Active Directory.
* Attribuez-lui un nom (par exemple, Jonction Azure AD Hybride).
* Modifier et accéder à :  Configuration ordinateur > Stratégies > Modèles d’administration > Composants Windows > Inscription de l’appareil.

>[!NOTE]
>Dans le cas de la version 2012R2, les paramètres de stratégie sont accessibles à l’emplacement **Configuration ordinateur > Stratégies > Modèles d’administration > Composants Windows > Rattacher à l’espace de travail > Joindre automatiquement les ordinateurs clients à l’espace de travail**.

* Activez ce paramètre :  Enregistrer les ordinateurs appartenant au domaine en tant qu’appareils.
* Appliquez vos modifications, puis cliquez sur OK.
* Liez l’objet de stratégie de groupe à l’emplacement de votre choix (unité d’organisation, groupe de sécurité ou domaine pour tous les appareils).

## <a name="2-configure-network-with-device-registration-endpoints"></a>2. Configurer le réseau avec des points de terminaison d’inscription d’appareil
Assurez-vous que les URL ci-après sont accessibles à partir d’ordinateurs au sein du réseau de votre organisation pour l’inscription auprès d’Azure AD :

* `https://enterpriseregistration.windows.net`
* `https://login.microsoftonline.com`
* `https://device.login.microsoftonline.com` 

## <a name="3-implement-wpad-for-windows-10-devices"></a>3. Implémenter WPAD pour les appareils Windows 10
Si votre organisation accède à Internet par le biais d’un proxy sortant, implémentez le service de détection automatique de proxy web (WPAD) pour permettre aux ordinateurs Windows 10 de s’inscrire auprès d’Azure AD.

## <a name="4-configure-the-scp-in-any-forests-that-were-not-configured-by-azure-ad-connect"></a>4. Configurer le SCP dans toutes les forêts non configurées par Azure AD Connect 

Le point de connexion de service (SCP) contient vos informations de locataire Azure AD qui seront utilisées par vos appareils pour l’inscription automatique.  Exécutez le script PowerShell, ConfigureSCP.ps1, que vous avez téléchargé à partir d’Azure AD Connect.

## <a name="5-configure-any-federation-service-that-was-not-configured-by-azure-ad-connect"></a>5. Configurer tout service de fédération non configuré par Azure AD Connect

Si votre organisation utilise un service de fédération pour se connecter à Azure AD, les règles de revendication dans votre approbation de partie de confiance Azure AD doivent autoriser l’authentification des appareils. Si vous utilisez la fédération avec AD FS, accédez à [l’aide d’AD FS](https://aka.ms/aadrptclaimrules) pour générer les règles de revendication. Si vous utilisez une solution de fédération non-Microsoft, demandez conseil au fournisseur de cette solution.  

>[!NOTE]
>Si vous disposez d’appareils Windows de bas niveau, le service doit prendre en charge l’émission des revendications authenticationmethod et wiaormultiauthn lors de la réception des demandes de confiance Azure AD. Dans AD FS, vous devez ajouter une règle de transformation d’émission qui est transmise par le biais de la méthode d’authentification.

## <a name="6-enable-azure-ad-seamless-sso-for-windows-down-level-devices"></a>6. Activer l’authentification unique (SSO) transparente Azure AD pour les appareils Windows de bas niveau

Si votre organisation utilise la synchronisation de hachage du mot de passe ou l’authentification directe pour se connecter à Azure AD, activez l’authentification unique transparente Azure AD avec cette méthode de connexion afin d’authentifier les appareils Windows de bas niveau : https://docs.microsoft.com/azure/active-directory/connect/active-directory-aadconnect-sso. 

## <a name="7-set-azure-ad-policy-for-windows-down-level-devices"></a>7. Définir la stratégie Azure AD pour les appareils Windows de bas niveau

Pour inscrire des appareils Windows de bas niveau, vous devez vous assurer que la stratégie Azure AD autorise les utilisateurs à inscrire des appareils. 

* Connectez-vous à votre compte dans le Portail Microsoft Azure.
* Accédez à :  Azure Active Directory > Appareils > Paramètres de l’appareil
* Définissez le paramètre « Les utilisateurs peuvent inscrire leurs appareils sur Azure AD » sur TOUS.
* Cliquez sur Enregistrer.

## <a name="8-add-azure-ad-endpoint-to-windows-down-level-devices"></a>8. Ajouter un point de terminaison Azure AD aux appareils Windows de bas niveau

Ajoutez le point de terminaison d’authentification d’appareil Azure AD aux zones Intranet local sur vos appareils Windows de bas niveau afin d’éviter les invites de certificat lors de l’authentification des appareils : `https://device.login.microsoftonline.com` 

Si vous utilisez [l’authentification unique transparente](how-to-connect-sso.md), activez également le paramètre « Autoriser les mises à jour à la barre d’état via le script » sur cette zone et ajoutez le point de terminaison suivant : `https://autologon.microsoftazuread-sso.com` 

## <a name="9-install-microsoft-workplace-join-on-windows-down-level-devices"></a>9. Installer Microsoft Workplace Join sur les appareils Windows de bas niveau

Ce programme d’installation crée une tâche planifiée sur le système de l’appareil qui s’exécute dans le contexte de l’utilisateur. La tâche est déclenchée lorsque l’utilisateur se connecte à Windows. La tâche assure la jonction de l’appareil en mode silencieux à Azure AD avec les informations d’identification de l’utilisateur après l’avoir authentifié à l’aide de l’authentification Windows intégrée. Le centre de téléchargement est accessible à l’adresse https://www.microsoft.com/download/details.aspx?id=53554. 

## <a name="10-configure-group-policy-to-allow-device-registration"></a>10. Configurer la stratégie de groupe pour autoriser l’inscription d’appareil

Pour savoir comment autoriser la jonction Azure AD Hybride pour certains appareils, consultez [Validation contrôlée de la jonction Azure AD Hybride](../devices/hybrid-azuread-join-control.md).

## <a name="next-steps"></a>Étapes suivantes
[Configurer la réécriture d’appareil](how-to-connect-device-writeback.md)
