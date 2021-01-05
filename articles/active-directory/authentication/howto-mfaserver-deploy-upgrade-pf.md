---
title: Mise à niveau de l’agent PhoneFactor vers le serveur Azure Multi-Factor Authentication - Azure Active Directory
description: Ce document décrit comment prendre en main le serveur Azure Multi-Factor Authentication lors d’une mise à niveau à partir de l’ancien agent PhoneFactor.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/11/2018
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 808109e00decf5c4084be5be550b49e2e7d4628c
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742355"
---
# <a name="upgrade-the-phonefactor-agent-to-azure-multi-factor-authentication-server"></a>Mise à niveau de l’agent PhoneFactor vers le serveur Azure Multi-Factor Authentication

Pour mettre à niveau la version 5.x ou une version antérieure de l’agent PhoneFactor vers le serveur Microsoft Azure Multi-Factor Authentication, commencez par désinstaller l’agent PhoneFactor et les composants correspondants. Vous pouvez ensuite installer le serveur Azure Multi-Factor Authentication et les composants associés.

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur lors des événements de connexion doivent utiliser la fonctionnalité d’authentification multifacteur Azure AD basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisés avec l’authentification multifacteur Azure AD](tutorial-enable-azure-mfa.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude.

## <a name="uninstall-the-phonefactor-agent"></a>Désinstallation de l’agent PhoneFactor

1. Tout d'abord, sauvegardez le fichier de données PhoneFactor. L'emplacement d'installation par défaut est C:\Program Files\PhoneFactor\Data\Phonefactor.pfdata.

2. Si le portail de l'utilisateur est installé :
   1. Accédez au dossier d'installation et sauvegardez le fichier web.config. L'emplacement d'installation par défaut est C:\inetpub\wwwroot\PhoneFactor.

   2. Si vous avez ajouté des thèmes personnalisés au portail, sauvegardez votre dossier personnalisé sous le répertoire C:\inetpub\wwwroot\PhoneFactor\App_Themes.

   3. Désinstallez le portail de l'utilisateur via l’agent PhoneFactor (disponible uniquement s’il est installé sur le même serveur que l’agent PhoneFactor) ou les programmes et fonctionnalités Windows.

3. Si le service web de l’application mobile est installé :

   1. Accédez au dossier d’installation et sauvegardez le fichier web.config. L’emplacement d’installation par défaut est C:\inetpub\wwwroot\PhoneFactorPhoneAppWebService.

   2. Désinstaller le service web de l’application mobile via les programmes et fonctionnalités Windows.

4. Si le Kit de développement logiciel (SDK) Web Service est installé, désinstallez-le via l’agent PhoneFactor ou les programmes et fonctionnalités Windows.

5. Désinstallez l’agent PhoneFactor via les programmes et fonctionnalités Windows.

## <a name="install-the-multi-factor-authentication-server"></a>Installation du serveur Azure Multi-Factor Authentication

Le chemin d’installation est récupéré à partir du Registre de l’installation de l’agent PhoneFactor précédente. Le serveur Multi-Factor Authentication devrait donc s’installer au même emplacement (par exemple, C:\Program Files\PhoneFactor). Les nouvelles installations présentent un chemin d’installation par défaut différent (par exemple, C:\Program Files\Multi-Factor Authentication Server). Le fichier de données de l’agent PhoneFactor doit être mis à niveau lors de l'installation pour que vos utilisateurs et les paramètres puissent toujours être présents après l’installation du nouveau serveur Multi-Factor Authentication.

1. Si vous y êtes invité, activez le serveur Multi-Factor Authentication et assurez-vous qu'il est affecté au groupe de réplication approprié.

2. Si le Kit de développement logiciel (SDK) Web Service a été précédemment installé, installez le nouveau Kit de développement logiciel (SDK) Web Service via l'Interface utilisateur du serveur Multi-Factor Authentication.

   Le nom de répertoire virtuel par défaut est désormais **MultiFactorAuthWebServiceSdk** au lieu de **PhoneFactorWebServiceSdk**. Si vous souhaitez utiliser le nom précédent, vous devez modifier le nom du répertoire virtuel pendant l'installation. Dans le cas contraire, si vous autorisez le programme d’installation à utiliser le nouveau nom par défaut, vous devez modifier l’URL dans toutes les applications qui référencent le Kit de développement logiciel (SDK) du service web (telles que le portail de l’utilisateur et le service web de l’application mobile) pour pointer vers l’emplacement correct.

3. Si le portail de l'utilisateur a été précédemment installé sur le serveur de l'agent PhoneFactor, installez le nouveau portail utilisateur Multi-Factor Authentication via l'Interface Multi-Factor Authentication.

   Le nom de répertoire virtuel par défaut est désormais **MultiFactorAuth** au lieu de **PhoneFactor**. Si vous souhaitez utiliser le nom précédent, vous devez modifier le nom du répertoire virtuel pendant l'installation. Sinon, si vous autorisez le programme d'installation à utiliser le nouveau nom par défaut, vous cliquez sur l'icône Portail de l'utilisateur sur le serveur Multi-Factor Authentication et mettez à jour l'URL du portail utilisateur sous l'onglet Paramètres.

4. Si le Portail de l’utilisateur et/ou le service web de l’application mobile ont été installés précédemment sur un serveur différent de l’agent PhoneFactor :

   1. Accédez à l’emplacement d’installation (par exemple, C:\Program Files\PhoneFactor), puis copiez un ou plusieurs programmes d’installation sur l’autre serveur. Il existe des programmes d'installation 32 bits et 64 bits pour le portail de l'utilisateur et le service web de l'application mobile. Ils sont appelés MultiFactorAuthenticationUserPortalSetupXX.msi et MultiFactorAuthenticationMobileAppWebServiceSetupXX.msi.

   2. Pour installer le portail de l’utilisateur sur le serveur web, ouvrez une invite de commandes en tant qu’administrateur et exécutez MultiFactorAuthenticationUserPortalSetupXX.msi.

      Le nom de répertoire virtuel par défaut est désormais **MultiFactorAuth** au lieu de **PhoneFactor**. Si vous souhaitez utiliser le nom précédent, vous devez modifier le nom du répertoire virtuel pendant l'installation. Sinon, si vous autorisez le programme d'installation à utiliser le nouveau nom par défaut, vous cliquez sur l'icône Portail de l'utilisateur sur le serveur Multi-Factor Authentication et mettez à jour l'URL du portail utilisateur sous l'onglet Paramètres. Les utilisateurs existants doivent être informés de la nouvelle URL.

   3. Accédez à l’emplacement d’installation du portail de l’utilisateur (par exemple, C:\inetpub\wwwroot\MultiFactorAuth) et modifiez le fichier web.config. Copiez dans le nouveau fichier web.config les valeurs des sections appSettings et applicationSettings de votre fichier web.config d’origine qui a été sauvegardé avant la mise à niveau. Si le nouveau nom de répertoire virtuel par défaut a été conservé lors de l'installation du Kit de développement logiciel (SDK) Web Service, modifiez l'URL dans la section applicationSettings pour pointer vers le bon emplacement. Si les valeurs par défaut ont été modifiées dans le précédent fichier web.config, appliquez les mêmes modifications au nouveau fichier web.config.

> [!NOTE]
> En cas de mise à niveau du serveur Azure MFA d’une version antérieure à 8.0 vers la version 8.0+, le service web d’application mobile peut être désinstallé à l’issue de la mise à niveau.

## <a name="next-steps"></a>Étapes suivantes

- [Installez le portail des utilisateurs](howto-mfaserver-deploy-userportal.md) du serveur Azure Multi-Factor Authentication.

- [Configurez l’authentification Windows](howto-mfaserver-windows.md) pour vos applications. 
