---
title: Mise à niveau vers Azure MFA Server - Azure Active Directory
description: Étapes et aide pour la mise à niveau du serveur Azure Multi-Factor Authentication vers une version plus récente.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/12/2018
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1755404a06d8586968801aa22f2af532da278802
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742321"
---
# <a name="upgrade-to-the-latest-azure-multi-factor-authentication-server"></a>Mise à niveau du serveur Azure Multi-Factor Authentication vers la dernière version

Cet article vous guide dans le processus de mise à niveau du serveur Azure Multi-Factor Authentication (MFA) v6.0 ou version ultérieure. Si vous avez besoin de mettre à niveau une ancienne version de l’Agent PhoneFactor, consultez la page [Mettre à niveau l’Agent PhoneFactor vers le serveur Azure Multi-Factor Authentication](howto-mfaserver-deploy-upgrade-pf.md).

Si vous passez de la v6.x ou d’une version antérieure à la version 7.x ou une version plus récente, tous les composants passent de .NET 2.0 à .NET 4.5. Tous les composants requièrent également Redistribuable Microsoft Visual C++ 2015 Update 1 ou une version ultérieure. Le programme d’installation du serveur MFA installe à la fois la version x86 et la version x64 de ces composants s’ils ne sont pas encore installés. Si le portail utilisateur et le service web de l’application mobile s’exécutent sur des serveurs distincts, vous devez installer ces packages avant de mettre à niveau ces composants. Vous pouvez rechercher la dernière mise à jour de Redistribuable Microsoft Visual C++ 2015 sur le [Centre de téléchargement Microsoft](https://www.microsoft.com/download/). 

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur lors des événements de connexion doivent utiliser la fonctionnalité d’authentification multifacteur Azure AD basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisés avec l’authentification multifacteur Azure AD](tutorial-enable-azure-mfa.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude.

Aperçu des étapes de mise à niveau :

* Mettre à niveau les serveurs Azure MFA (subordonnés puis Master)
* Mettre à niveau les instances du portail utilisateur
* Mettre à niveau les instances de l’adaptateur AD FS

## <a name="upgrade-azure-mfa-server"></a>Mettre à niveau le serveur Azure MFA

1. Suivez les instructions de la page [Télécharger le serveur Azure Multi-Factor Authentication](howto-mfaserver-deploy.md#download-the-mfa-server) pour obtenir la dernière version du programme d’installation du serveur Azure MFA.
2. Effectuez une sauvegarde du fichier de données du serveur MFA situé dans C:\Program Files\Multi-Factor Authentication Server\Data\PhoneFactor.pfdata (emplacement d’installation par défaut) sur votre serveur MFA maître.
3. Si vous exécutez plusieurs serveurs à des fins de haute disponibilité, modifiez les systèmes clients qui s’authentifient auprès du serveur MFA afin qu’ils arrêtent d’envoyer du trafic aux serveurs en cours de mise à niveau. Si vous utilisez un équilibreur de charge, supprimez un serveur MFA subordonné de l’équilibreur de charge, effectuez la mise à niveau, puis rajoutez le serveur à la batterie de serveurs.
4. Exécutez le nouveau programme d’installation sur chaque serveur MFA. Mettez à niveau les serveurs subordonnés en premier, car ils peuvent lire l’ancien fichier de données que le maître est en train de répliquer.

   > [!NOTE]
   > Lorsqu’un serveur est mis à niveau, il doit être supprimé de tous les équilibrages de charge ou des partages de trafic avec d’autres serveurs MFA.
   >
   > Il est inutile de désinstaller votre serveur MFA actuel avant d’exécuter le programme d’installation. Le programme d’installation effectue une mise à niveau sur place. Le chemin d’installation est récupéré à partir du Registre de l’installation précédente. L’installation se fait donc au même emplacement (par exemple, C:\Program Files\Multi-Factor Authentication Server).
  
5. Si vous êtes invité à installer un package de mise à jour Redistribuable Microsoft Visual C++ 2015, acceptez l’invite. La version x86 et la version x64 du package sont toutes deux installées.
6. Si vous utilisez le Kit de développement logiciel (SDK) du service web, vous êtes invité à installer le nouveau Kit de développement logiciel (SDK) du service web. Lorsque vous installez le nouveau Kit SDK du service web, assurez-vous que le nom du répertoire virtuel correspond à celui du répertoire virtuel déjà installé (par exemple, MultiFactorAuthWebServiceSdk).
7. Répétez les étapes sur tous les serveurs subordonnés. Promouvez l’un des subordonnés comme nouveau maître, puis mettez à niveau l’ancien serveur maître.

## <a name="upgrade-the-user-portal"></a>Mettre à niveau le portail utilisateur

Effectuez la mise à niveau de vos serveurs MFA avant de passer à cette section.

1. Effectuez une sauvegarde du fichier web.config qui se trouve dans le répertoire virtuel de l’emplacement d’installation du portail utilisateur (par exemple, C:\inetpub\wwwroot\MultiFactorAuth). Si des modifications ont été apportées au thème par défaut, effectuez également une sauvegarde du dossier App_Themes\Default. Il est préférable d’effectuer une copie du dossier Default et de créer un nouveau thème plutôt que de modifier le thème Default.
2. Si le portail utilisateur s’exécute sur le même serveur que les autres composants du serveur MFA, l’installation du serveur MFA vous invite à mettre à jour le portail utilisateur. Acceptez l’invite et installez la mise à jour du portail utilisateur. Veillez à ce que le nom du répertoire virtuel corresponde à celui du répertoire virtuel déjà installé (par exemple, MultiFactorAuth).
3. Si le portail utilisateur est sur son propre serveur, copiez le fichier MultiFactorAuthenticationUserPortalSetup64.msi dans l’emplacement d’installation de l’un des serveurs MFA et placez-le sur le serveur web du portail utilisateur. Exécutez le programme d’installation.

   Si une erreur se produit, selon laquelle Redistribuable Microsoft Visual C++ 2015 Update 1 ou version ultérieure est requis, téléchargez et installez la dernière mise à jour disponible dans le [Centre de téléchargement Microsoft](https://www.microsoft.com/download/). Installez à la fois la version x86 et la version x64.

4. Une fois le logiciel du portail utilisateur mis à jour installé, comparez la sauvegarde web.config effectuée à l’étape 1 au nouveau fichier web.config. Si le nouveau fichier web.config ne présente pas de nouvel attribut, copiez votre fichier web.config de sauvegarde dans le répertoire virtuel pour remplacer le nouveau. Une autre option consiste à copier/coller les valeurs appSettings et l’URL du Kit SDK de service web du fichier de sauvegarde dans le nouveau fichier web.config.

Si le portail utilisateur fonctionne sur plusieurs serveurs, répétez l’installation sur chacun d’eux.

## <a name="upgrade-the-mobile-app-web-service"></a>Mettre à niveau le service web de l’application mobile

> [!NOTE]
> En cas de mise à niveau du serveur Azure MFA d’une version antérieure à 8.0 vers la version 8.0+, le service web d’application mobile peut être désinstallé à l’issue de la mise à niveau.

## <a name="upgrade-the-ad-fs-adapters"></a>Mettre à niveau les adaptateurs AD FS

Effectuez la mise à niveau de vos serveurs MFA et de votre portail utilisateur avant de passer à cette section.

### <a name="if-mfa-runs-on-different-servers-than-ad-fs"></a>Si MFA s’exécute sur d’autres serveurs qu’AD FS

Ces instructions s’appliquent uniquement si vous exécutez le serveur Multi-Factor Authentication indépendamment de vos serveurs AD FS. Si les deux services s’exécutent sur les mêmes serveurs, ignorez cette section et suivez les étapes d’installation. 

1. Enregistrez une copie du fichier MultiFactorAuthenticationAdfsAdapter.config enregistrée dans AD FS, ou exportez la configuration à l’aide de la commande PowerShell suivante : `Export-AdfsAuthenticationProviderConfigurationData -Name [adapter name] -FilePath [path to config file]`. Le nom de l’adaptateur est « WindowsAzureMultiFactorAuthentication » ou « AzureMfaServerAuthentication », selon la version précédemment installée.
2. Copiez les fichiers suivants de l’emplacement d’installation du serveur MFA vers les serveurs AD FS :

   * MultiFactorAuthenticationAdfsAdapterSetup64.msi
   * Register-MultiFactorAuthenticationAdfsAdapter.ps1
   * Unregister-MultiFactorAuthenticationAdfsAdapter.ps1
   * MultiFactorAuthenticationAdfsAdapter.config

3. Modifiez le script Register-MultiFactorAuthenticationAdfsAdapter.ps1 en ajoutant `-ConfigurationFilePath [path]` à la fin de la commande `Register-AdfsAuthenticationProvider`. Remplacez *[path]* par le chemin complet vers le fichier MultiFactorAuthenticationAdfsAdapter.config ou le fichier config exporté à l’étape précédente.

   Vérifiez les attributs de la nouvelle configuration MultiFactorAuthenticationAdfsAdapter.config, pour voir s’ils correspondent à l’ancien fichier de configuration. Si des attributs ont été ajoutés ou supprimés dans la nouvelle version, copiez les valeurs des attributs de l’ancien fichier config vers le nouveau, ou bien modifiez l’ancien fichier config pour les faire correspondre.

### <a name="install-new-ad-fs-adapters"></a>Installer de nouveaux adaptateurs AD FS

> [!IMPORTANT]
> Les utilisateurs n’auront pas besoin d’effectuer la vérification en deux étapes entre les étapes 3 et 8 de cette section. Si AD FS est configuré sur plusieurs clusters, vous pouvez supprimer, mettre à niveau et restaurer chaque cluster de la batterie de serveurs indépendamment des autres clusters afin d’éviter tout temps d’arrêt.

1. Supprimez certains serveurs AD FS de la batterie de serveurs. Mettez à jour ces serveurs tandis que les autres sont en cours d’exécution.
2. Installez le nouvel adaptateur AD FS sur chaque serveur supprimé de la batterie de serveurs AD FS. Si le serveur MFA est installé sur chaque serveur AD FS, vous pouvez effectuer la mise à jour à partir de l’UX administrateur du serveur MFA. Sinon, effectuez la mise à jour en exécutant MultiFactorAuthenticationAdfsAdapterSetup64.msi.

   Si une erreur se produit, selon laquelle Redistribuable Microsoft Visual C++ 2015 Update 1 ou version ultérieure est requis, téléchargez et installez la dernière mise à jour disponible dans le [Centre de téléchargement Microsoft](https://www.microsoft.com/download/). Installez à la fois la version x86 et la version x64.

3. Accédez à **AD FS** > **Stratégies d’authentification** > **Modifier la stratégie Multi-Factor Authentication globale**. Décochez **WindowsAzureMultiFactorAuthentication** ou **AzureMFAServerAuthentication** (selon la version actuellement installée).

   Une fois cette étape terminée, la vérification en deux étapes par le serveur MFA ne sera pas disponible dans ce cluster AD FS avant la fin de l’étape 8.

4. Annulez l’enregistrement de l’ancienne version de l’adaptateur AD FS en exécutant le script PowerShell Unregister-MultiFactorAuthenticationAdfsAdapter.ps1. Assurez-vous que le paramètre *-Name* (« WindowsAzureMultiFactorAuthentication » ou « AzureMFAServerAuthentication ») correspond au nom qui était affiché à l’étape 3. Cela s’applique à tous les serveurs du même cluster AD FS, dans la mesure où il s’agit d’une configuration centrale.
5. Enregistrez le nouvel adaptateur AD FS en exécutant le script PowerShell Register-MultiFactorAuthenticationAdfsAdapter.ps1. Cela s’applique à tous les serveurs du même cluster AD FS, dans la mesure où il s’agit d’une configuration centrale.
6. Redémarrez le service AD FS sur chaque serveur supprimé de la batterie de serveurs AD FS.
7. Rajoutez les serveurs mis à jour à la batterie de serveurs AD FS et supprimez les autres serveurs de la batterie.
8. Accédez à **AD FS** > **Stratégies d’authentification** > **Modifier la stratégie Multi-Factor Authentication globale**. Cochez **AzureMfaServerAuthentication**.
9. Répétez l’étape 2 pour mettre à jour les serveurs maintenant supprimés de la batterie de serveurs AD FS et redémarrez le service AD FS sur ces serveurs.
10. Rajoutez ces serveurs à la batterie de serveurs AD FS.

## <a name="next-steps"></a>Étapes suivantes

* Consultez des exemples de [Scénarios avancés avec Azure Multi-Factor Authentication et des VPN tiers](howto-mfaserver-nps-vpn.md)

* [Synchronisez le serveur MFA avec Windows Server Active Directory](howto-mfaserver-dir-ad.md)

* [Configurez l’authentification Windows](howto-mfaserver-windows.md) pour vos applications
