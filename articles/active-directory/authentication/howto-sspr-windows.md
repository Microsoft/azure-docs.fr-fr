---
title: Réinitialisation de mot de passe en libre-service pour les appareils Windows - Azure Active Directory
description: Découvrez comment activer la réinitialisation de mot de passe en libre-service Azure Active Directory depuis l’écran de connexion Windows.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/17/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 478ae6146caeb8a27cdaf13b7f33e421b8121afc
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741488"
---
# <a name="enable-azure-active-directory-self-service-password-reset-at-the-windows-sign-in-screen"></a>Activer la réinitialisation de mot de passe en libre-service Azure Active Directory depuis l’écran de connexion Windows

La réinitialisation de mot de passe en libre-service (SSPR) permet aux utilisateurs d’Azure Active Directory (Azure AD) de modifier ou de réinitialiser leur mot de passe sans intervention d’un administrateur ou du support technique. En règle générale, les utilisateurs ouvrent un navigateur web sur un autre appareil pour accéder au [portail SSPR](https://aka.ms/sspr). Pour améliorer l’expérience sur les ordinateurs exécutant Windows 7, 8, 8.1 et 10, vous pouvez permettre aux utilisateurs de réinitialiser leur mot de passe depuis l’écran de connexion Windows.

![Exemples d’écrans de connexion Windows 7 et 10 avec le lien vers la réinitialisation de mot de passe en libre-service affiché](./media/howto-sspr-windows/windows-reset-password.png)

> [!IMPORTANT]
> Ce tutoriel montre comment un administrateur peut activer SSPR pour des appareils Windows dans une entreprise.
>
> Si votre service informatique n’a pas activé l’utilisation de SSPR à partir de votre appareil Windows, ou si vous rencontrez des problèmes durant la connexion, contactez votre support technique pour obtenir une assistance supplémentaire.

## <a name="general-limitations"></a>Limitations générales

Les limitations suivantes s’appliquent à l’utilisation de SSPR depuis l’écran de connexion Windows :

- La réinitialisation de mot de passe n’est pas prise en charge à partir d’une session Bureau à distance ou des sessions améliorées Hyper-V.
- Certains fournisseurs tiers d’informations d’identification sont connus pour poser des problèmes avec cette fonctionnalité.
- La désactivation du Contrôle de compte d’utilisateur via la modification de la [clé de Registre EnableLUA](/openspecs/windows_protocols/ms-gpsb/958053ae-5397-4f96-977f-b7700ee461ec) est connue pour provoquer des problèmes.
- Cette fonctionnalité est inopérante pour les réseaux où l’authentification réseau 802.1x est déployée et où l’option « Immédiatement avant l’ouverture de session de l’utilisateur » est activée. Pour les réseaux où l’authentification réseau 802.1x est déployée, il est recommandé d’utiliser l’authentification de la machine afin d’activer cette fonctionnalité.
- Les machines jointes Azure AD Hybride doivent disposer d’une connectivité réseau à un contrôleur de domaine pour utiliser le nouveau mot de passe et mettre à jour les informations d’identification mises en cache. Cela signifie que les appareils doivent se trouver sur le réseau interne de l’organisation ou sur un VPN disposant d’un accès réseau à un contrôleur de domaine local.
- Si vous utilisez une image, avant d’exécuter sysprep, vérifiez que le cache web est effacé pour le compte Administrateur intégré avant d’effectuer l’étape CopyProfile. Vous trouverez plus d’informations au sujet de cette étape dans l’article de support [Performances médiocres lors de l’utilisation du profil d’utilisateur par défaut personnalisé](https://support.microsoft.com/help/4056823/performance-issue-with-custom-default-user-profile).
- Les paramètres suivants sont connus pour interférer avec la fonctionnalité de réinitialisation des mots de passe sur les appareils Windows 10 :
    - Si Ctrl+Alt+Suppr est imposé par une stratégie dans les versions de Windows 10 antérieure à la version 1909, la commande **Réinitialiser le mot de passe** ne fonctionnera pas.
    - Si les notifications de l’écran de verrouillage sont désactivées, la commande **Réinitialiser le mot de passe** ne fonctionnera pas.
    - *HideFastUserSwitching* a la valeur enabled ou 1
    - *DontDisplayLastUserName* a la valeur enabled ou 1
    - *NoLockScreen* a la valeur enabled ou 1
    - *EnableLostMode* est défini sur l’appareil
    - Explorer.exe est remplacé par un interpréteur de commandes personnalisé
- La combinaison des trois paramètres spécifiques suivants peut empêcher le fonctionnement de cette fonctionnalité.
    - Ouverture de session interactive : N’exige pas la combinaison de touches CTRL + ALT + SUPPR = Désactivé
    - *DisableLockScreenAppNotifications* = 1 ou Enabled
    - La référence SKU Windows ne correspond pas à l’édition Famille ou Professionnel

## <a name="windows-10-password-reset"></a>Réinitialisation de mot de passe de Windows 10

Si vous souhaitez configurer un appareil Windows 10 pour SSPR à l’écran de connexion, passez en revue les prérequis et les étapes de configuration suivantes.

### <a name="windows-10-prerequisites"></a>Prérequis pour Windows 10

- Un administrateur [doit activer la réinitialisation de mot de passe en libre-service Azure AD à partir du portail Azure](tutorial-enable-sspr.md).
- Les utilisateurs doivent s’inscrire à SSPR pour pouvoir utiliser cette fonctionnalité sur [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup)
    - En plus de l’utilisation de SSPR à partir de l’écran de connexion Windows, tous les utilisateurs doivent fournir les informations de contact relatives à l’authentification pour pouvoir réinitialiser leur mot de passe.
- Exigences relatives au proxy réseau :
    - Port 443 vers `passwordreset.microsoftonline.com` et `ajax.aspnetcdn.com`
    - Les appareils Windows 10 prennent uniquement en charge la configuration du proxy au niveau de la machine.
- Exécutez au moins Windows 10, avec la mise à jour d’avril 2018 (v1803), et les appareils doivent être :
    - Appareil joints Azure AD
    - Appareils joints Azure AD hybrides

### <a name="enable-for-windows-10-using-intune"></a>Activez pour Windows 10 à l’aide de Intune

Le déploiement du changement de configuration pour activer SSPR à partir de l’écran de connexion à l’aide d’Intune est la méthode la plus flexible. Intune vous permet de déployer les modifications de configuration vers un groupe spécifique de machines que vous définissez. Cette méthode nécessite un appareil inscrit à Intune.

#### <a name="create-a-device-configuration-policy-in-intune"></a>Créer une stratégie de configuration d’appareil dans Intune

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez **Intune**.
1. Créez un profil de configuration d’appareil en accédant à **Configuration de l’appareil** > **Profils**, puis sélectionnez **+ Créer un profil**
   - Pour **Plateforme**, choisissez *Windows 10 et ultérieur*
   - Pour **Type de profil**, choisissez *Personnalisé*
1. Sélectionnez **Créer**, puis indiquez un nom de profil explicite, par exemple *Écran de connexion Windows 10 pour SSPR*

    Indiquez éventuellement une description explicite du profil, puis sélectionnez **Suivant**.
1. Sous *Paramètres de configuration*, sélectionnez **Ajouter**, puis indiquez le paramètre OMA-URI suivant pour activer le lien de réinitialisation de mot de passe :
      - Indiquez un nom explicite pour décrire l’action du paramètre, par exemple *Ajouter un lien SSPR*.
      - Indiquez éventuellement une description explicite du paramètre.
      - Paramètre **OMA-URI** défini sur `./Vendor/MSFT/Policy/Config/Authentication/AllowAadPasswordReset`
      - Paramètre **Type de données** défini sur la valeur **Entier**
      - Paramètre **Valeur** défini sur **1**

    Sélectionnez **Ajouter**, puis **Suivant**.
1. La stratégie peut être affectée à des utilisateurs, des appareils ou des groupes spécifiques. Affectez d’abord le profil souhaité à votre environnement, idéalement à un groupe de test d’appareils, puis sélectionnez **Suivant**.

    Pour plus d’informations, consultez [Attribuer des profils d’utilisateur et d’appareil dans Microsoft Intune](/mem/intune/configuration/device-profile-assign).

1. Configurez les règles d’applicabilité souhaitées pour votre environnement, par exemple *Affecter un profil si l’édition du système d’exploitation est Windows 10 Entreprise*, puis sélectionnez **Suivant**.
1. Passez en revue votre profil, puis sélectionnez **Créer**.

### <a name="enable-for-windows-10-using-the-registry"></a>Activez pour Windows 10 à l’aide du Registre

Pour activer SSPR à l’écran de connexion à l’aide d’une clé de Registre, effectuez les étapes suivantes :

1. Connectez-vous au PC Windows à l’aide d’informations d’identification d’administration.
1. Appuyez sur **Windows** + **R** pour ouvrir la boîte de dialogue *Exécuter*, puis exécutez **regedit** en tant qu’administrateur
1. Définissez la clé de Registre suivante :

    ```cmd
    HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\AzureADAccount
       "AllowPasswordReset"=dword:00000001
    ```

#### <a name="troubleshooting-windows-10-password-reset"></a>Résolution des problèmes de réinitialisation de mot de passe de Windows 10

Si vous rencontrez des problèmes d’utilisation de SSPR à partir de l’écran de connexion Windows, le journal d’audit Azure AD contient des informations sur l’adresse IP et le *ClientType* où la réinitialisation de mot de passe s’est produite, comme illustré dans l’exemple de sortie suivant :

![Exemple de réinitialisation de mot de passe sur Windows 7 dans le journal d’audit Azure AD](media/howto-sspr-windows/windows-7-sspr-azure-ad-audit-log.png)

Quand les utilisateurs réinitialisent leur mot de passe à partir de l’écran de connexion d’un appareil Windows 10, un compte temporaire doté de faibles privilèges appelé `defaultuser1` est créé. Ce compte sert à sécuriser le processus de réinitialisation du mot de passe.

Le compte lui-même comporte un mot de passe généré de manière aléatoire, ne s’affiche pas pour la connexion à l’appareil et est supprimé automatiquement une fois que l’utilisateur a réinitialisé son mot de passe. Il peut exister plusieurs profils `defaultuser`, mais vous pouvez les ignorer sans risque.

## <a name="windows-7-8-and-81-password-reset"></a>Réinitialisation de mot de passe de Windows 7, 8 et 8.1

Si vous souhaitez configurer un appareil Windows 7, 8 ou 8.1 pour SSPR à l’écran de connexion, passez en revue les prérequis et les étapes de configuration suivantes.

### <a name="windows-7-8-and-81-prerequisites"></a>Prérequis pour Windows 7, 8 et 8.1

- Un administrateur [doit activer la réinitialisation de mot de passe en libre-service Azure AD à partir du portail Azure](tutorial-enable-sspr.md).
- Les utilisateurs doivent s’inscrire à SSPR pour pouvoir utiliser cette fonctionnalité sur [https://aka.ms/ssprsetup](https://aka.ms/ssprsetup)
    - En plus de l’utilisation de SSPR à partir de l’écran de connexion Windows, tous les utilisateurs doivent fournir les informations de contact relatives à l’authentification pour pouvoir réinitialiser leur mot de passe.
- Exigences relatives au proxy réseau :
    - Port 443 vers `passwordreset.microsoftonline.com`
- Système d’exploitation Windows 7 ou Windows 8.1 corrigé.
- TLS 1.2 activé à l’aide des instructions fournies dans l’article [Paramètres de Registre TLS](/windows-server/security/tls/tls-registry-settings#tls-12).
- Si plusieurs fournisseurs tiers d’informations d’identification sont activés sur votre machine, les utilisateurs voient plusieurs profils utilisateur sur l’écran de connexion.

> [!WARNING]
> TLS 1.2 doit être activé, et non simplement défini sur négociation automatique.

### <a name="install"></a>Installer

Pour Windows 7, 8 et 8.1, un petit composant doit être installé sur la machine pour permettre l’activation de SSPR à l’écran de connexion. Pour installer ce composant SSPR, effectuez les étapes suivantes :

1. Téléchargez le programme d’installation correspondant à la version de Windows que vous souhaitez activer.

    Le programme d’installation du logiciel est disponible auprès du Centre de téléchargement Microsoft à l’adresse [https://aka.ms/sspraddin](https://aka.ms/sspraddin)
1. Connectez-vous à la machine sur laquelle vous souhaitez installer et exécuter le programme d’installation.
1. Après l’installation, un redémarrage est vivement recommandé.
1. Une fois le redémarrage effectué, à l’écran de connexion, choisissez un utilisateur, puis sélectionnez « Mot de passe oublié ? » pour lancer le flux de travail de réinitialisation de mot de passe.
1. Exécutez le flux de travail en suivant les étapes à l’écran pour réinitialiser votre mot de passe.

![Exemple Windows 7 de flux de réinitialisation de mot de passe en libre-service Flux de réinitialisation de mot de passe en libre-service](media/howto-sspr-windows/windows-7-sspr.png)

#### <a name="silent-installation"></a>Installation sans assistance

Vous pouvez installer ou désinstaller le composant SSPR sans invites à l’aide des commandes suivantes :

- Pour une installation sans assistance, utilisez la commande « msiexec /i SsprWindowsLogon.PROD.msi /qn ».
- Pour une désinstallation sans assistance, utilisez la commande « msiexec /x SsprWindowsLogon.PROD.msi /qn ».

#### <a name="troubleshooting-windows-7-8-and-81-password-reset"></a>Résolution des problèmes de réinitialisation de mot de passe de Windows 7, 8 et 8.1

Si vous rencontrez des problèmes durant l’utilisation de SSPR à partir de l’écran de connexion Windows, les événements sont journalisés à la fois sur la machine et dans Azure AD. Les événements Azure AD incluent des informations sur l’adresse IP et le ClientType où la réinitialisation de mot de passe s’est produite, comme illustré dans l’exemple de sortie suivant :

![Exemple de réinitialisation de mot de passe sur Windows 7 dans le journal d’audit Azure AD](media/howto-sspr-windows/windows-7-sspr-azure-ad-audit-log.png)

Si une journalisation supplémentaire est requise, vous pouvez modifier une clé de Registre sur la machine pour activer la journalisation détaillée. Activez la journalisation détaillée uniquement pour la résolution des problèmes à l’aide de la valeur de clé de Registre suivante :

```cmd
HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Providers\{86D2F0AC-2171-46CF-9998-4E33B3D7FD4F}
```

- Pour activer la journalisation détaillée, créez un `REG_DWORD: "EnableLogging"`, et affectez-lui la valeur 1.
- Pour désactiver la journalisation détaillée, faites basculer la valeur de `REG_DWORD: "EnableLogging"` à 0.

## <a name="what-do-users-see"></a>Ce que voient les utilisateurs

Une fois SSPR configuré pour vos appareils Windows, quels sont les changements pour l’utilisateur ? Comment peut-il savoir qu’il est possible de réinitialiser un mot de passe sur l’écran de connexion ? Les exemples de captures d’écran suivants montrent les options supplémentaires qui permettent à un utilisateur de réinitialiser son mot de passe à l’aide de SSPR :

![Exemples d’écrans de connexion Windows 7 et 10 avec le lien vers la réinitialisation de mot de passe en libre-service affiché](./media/howto-sspr-windows/windows-reset-password.png)

Quand les utilisateurs tentent de se connecter, ils voient un lien **Réinitialiser le mot de passe** ou **Mot de passe oublié** qui permet d’ouvrir l’expérience de réinitialisation de mot de passe en libre-service à l’écran de connexion. Cette fonctionnalité permet aux utilisateurs de réinitialiser leur mot de passe sans avoir à utiliser un autre appareil pour accéder à un navigateur web.

Pour plus d’informations sur l’utilisation de cette fonctionnalité, consultez [Réinitialiser votre mot de passe professionnel ou scolaire](../user-help/active-directory-passwords-update-your-own-password.md)

## <a name="next-steps"></a>Étapes suivantes

Pour simplifier l’expérience d’inscription des utilisateurs, vous pouvez [préremplir les informations de contact relatives à l’authentification utilisateur pour SSPR](howto-sspr-authenticationdata.md).