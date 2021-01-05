---
title: Déployer la protection par mot de passe Azure AD localement
description: Découvrez comment planifier et déployer la protection par mot de passe Azure AD dans un environnement Active Directory Domain Services local
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 03/05/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: deb1f74902fe28d53a5180e4f341547f339a83ac
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741982"
---
# <a name="plan-and-deploy-on-premises-azure-active-directory-password-protection"></a>Planifiez et déployez localement la protection par mot de passe Azure Active Directory

Les utilisateurs créent souvent des mots de passe basés sur des mots locaux courants, par exemple une école, une équipe de sport ou une personne célèbre. Ces mots de passe sont faciles à deviner et offrent une faible protection contre les attaques par dictionnaire. Pour appliquer des mots de passe forts au sein de votre organisation, la protection par mot de passe Azure Active Directory (Azure AD) fournit une liste globale et personnalisée de mots de passe interdits. Toute demande de changement de mot de passe échoue s’il existe une correspondance dans la liste personnalisée de mots de passe interdits.

Pour protéger votre environnement Active Directory Domain Services (AD DS) local, vous pouvez installer et configurer la protection par mot de passe Azure AD pour qu’elle fonctionne avec votre contrôleur de domaine local. Cet article explique comment installer et inscrire le service proxy de protection par mot de passe Azure AD et l’agent du contrôleur de domaine de protection par mot de passe Azure AD dans votre environnement local.

Pour plus d’informations sur le fonctionnement de la protection par mot de passe Azure AD dans un environnement local, voir [Comment appliquer la protection par mot de passe Azure AD pour Windows Server Active Directory](concept-password-ban-bad-on-premises.md).

## <a name="deployment-strategy"></a>Stratégie de déploiement

Le diagramme suivant montre comment les composants de base de la protection par mot de passe Azure AD opèrent ensemble dans un environnement Active Directory local :

![Comment les composants de protection par mot de passe Azure AD fonctionnent ensemble](./media/concept-password-ban-bad-on-premises/azure-ad-password-protection.png)

Il est judicieux d’examiner comment le logiciel fonctionne avant de le déployer. Pour plus d’informations, voir [Vue d’ensemble conceptuelle de la protection par mot de passe Azure AD](concept-password-ban-bad-on-premises.md).

Nous vous recommandons de commencer les déploiements en mode *audit*. Le mode audit est la configuration initiale par défaut où il est possible de continuer à définir des mots de passe. Les mots de passe qui seraient bloqués sont enregistrés dans le journal des événements. Après avoir déployé les serveurs proxy et les agents du contrôleur de domaine en mode audit, surveillez l’impact qu’aura la stratégie de mot de passe sur les utilisateurs et lors de l’application de la stratégie.

Au cours de l’étape d’audit, de nombreuses organisations découvrent les situations suivantes :

* Elles doivent améliorer les processus opérationnels existants pour utiliser des mots de passe plus sécurisés.
* Les utilisateurs utilisent souvent des mots de passe non sécurisés.
* Elles doivent informer les utilisateurs des modifications de sécurisation à venir, de l’impact que cela peut avoir sur eux et de la façon de choisir des mots de passe plus sécurisés.

Il est également possible que la validation de mot de passe plus fort affecte l’automatisation de votre déploiement de contrôleur de domaine Active Directory existant. Nous vous recommandons d’effectuer au moins une promotion et une régression de contrôleur de domaine au cours de l’évaluation de la période d’audit afin de faciliter la détection de ces problèmes. Pour plus d’informations, consultez les articles suivants :

* [Ntdsutil.exe ne peut pas définir un mot de passe de mode de réparation des services d’annuaire faible](howto-password-ban-bad-on-premises-troubleshoot.md#ntdsutilexe-fails-to-set-a-weak-dsrm-password)
* [La promotion du réplica du contrôleur de domaine échoue en raison d’un mot de passe de mode de réparation des services d’annuaire faible](howto-password-ban-bad-on-premises-troubleshoot.md#domain-controller-replica-promotion-fails-because-of-a-weak-dsrm-password)
* [La rétrogradation du contrôleur de domaine échoue en raison d’un mot de passe administrateur local faible](howto-password-ban-bad-on-premises-troubleshoot.md#domain-controller-demotion-fails-due-to-a-weak-local-administrator-password)

Une fois que la fonctionnalité a été exécutée en mode audit pendant une période raisonnable, vous pouvez basculer la configuration de *Audit* à *Appliquer* pour exiger des mots de passe plus sécurisés. Durant cette période, il est recommandé d’exercer une surveillance supplémentaire.

Il est important de noter que la protection par mot de passe Azure AD ne peut valider des mots de passe que lors d’opérations de modification ou de définition de mot de passe. Les mots de passe acceptés et stockés dans Active Directory avant le déploiement de la protection par mot de passe Azure AD ne seront jamais validés et continueront à fonctionner en l’état. Au fil du temps, tous les utilisateurs et comptes finiront par utiliser des mots de passe validés par la protection par mot de passe Azure AD à mesure que leurs mots de passe existants viendront normalement à expiration. Les comptes configurés avec l’option « Le mot de passe n’expire jamais » ne sont pas concernés.

### <a name="multiple-forest-considerations"></a>Considérations relatives aux forêts multiples

Il n’y a aucune exigence supplémentaire pour déployer la protection par mot de passe Azure AD dans plusieurs forêts.

Chaque forêt est configurée indépendamment, comme décrit dans la section suivante pour [déployer la protection par mot de passe Azure AD localement](#download-required-software). Chaque proxy de protection par mot de passe Azure AD ne peut prendre en charge que les contrôleurs de domaine de la forêt à laquelle il est joint.

Le logiciel de protection par mot de passe Azure AD figurant dans une forêt donnée n’a pas connaissance des logiciels de protection par mot de passe déployés dans d’autres forêts, quelles que soient les configurations d’approbation Active Directory.

### <a name="read-only-domain-controller-considerations"></a>Considérations relatives au contrôleur de domaine en lecture seule

Les événements de modification ou de définition de mot de passe ne sont pas traités ou conservés sur les contrôleurs de domaine en lecture seule (RODC). Au lieu de cela, ils sont transférés vers des contrôleurs de domaine accessibles en écriture. Vous n’êtes pas obligé d’installer le logiciel de l’agent DC de protection par mot de passe Azure AD sur les RODC.

Il n’est pas possible non plus d’exécuter le service proxy de protection par mot de passe Azure AD sur un contrôleur de domaine en lecture seule.

### <a name="high-availability-considerations"></a>Considérations relatives à la haute disponibilité

La principale préoccupation pour la protection par mot de passe est la disponibilité des serveurs proxy de protection par mot de passe Azure AD lorsque les contrôleurs de domaine dans une forêt essaient de télécharger de nouvelles stratégies ou d’autres données à partir d’Azure. Chaque agent DC de protection par mot de passe Azure AD utilise un algorithme de style tourniquet (round-robin) simple lorsqu’il décide du serveur proxy à appeler. L’agent ignore les serveurs proxy qui ne répondent pas.

Pour la plupart des déploiements Active Directory totalement connectés avec réplication saine de l’état de l’annuaire et du dossier sysvol, deux serveurs proxy de protection par mot de passe AD DS suffisent pour garantir la disponibilité. Cette configuration entraîne un téléchargement en temps voulu des nouvelles stratégies et d’autres données. Si vous le souhaitez, vous pouvez déployer des serveurs proxy de protection par mot de passe Azure AD supplémentaires.

La conception du logiciel de l’agent DC de protection par mot de passe Azure AD atténue les problèmes habituels associés à la haute disponibilité. L’agent DC de protection par mot de passe Azure AD gère un cache local de la stratégie de mot de passe la plus récemment téléchargée. Même si tous les serveurs proxy inscrits deviennent indisponibles, les agents DC de protection par mot de passe Azure AD continuent d’appliquer leur stratégie de mot de passe mis en cache.

Une fréquence de mise à jour raisonnable pour les stratégies de mot de passe dans un déploiement à grande échelle se compte généralement en jours, pas en heures, ni moins. Par conséquent, des pannes brèves de serveurs proxy n’ont pas d’impact important sur la protection par mot de passe Azure AD.

## <a name="deployment-requirements"></a>Conditions requises pour le déploiement

Pour plus d’informations sur les licences, voir [Exigences en termes de licence pour la protection par mot de passe Azure AD](concept-password-ban-bad.md#license-requirements).

Les principales exigences suivantes s’appliquent :

* Le runtime C universel doit être installé sur toutes les machines, y compris les contrôleurs de domaine, sur lesquelles les composants de protection par mot de passe Azure AD sont installés.
    * Vous pouvez obtenir le runtime en vous assurant que vous disposez de toutes les mises à jour à partir de Windows Update. Ou vous pouvez l’obtenir dans un package de mise à jour spécifique au système d’exploitation. Pour plus d’informations, consultez [Mise à jour du runtime C universel sous Windows](https://support.microsoft.com/help/2999226/update-for-uniersal-c-runtime-in-windows).
* Vous avez besoin d’un compte disposant des privilèges d’administrateur de domaine Active Directory dans le domaine racine de la forêt pour inscrire la forêt Windows Server Active Directory auprès d’Azure AD.
* Le service de distribution de clés doit être activé sur tous les contrôleurs de domaine figurant dans le domaine qui exécutent Windows Server 2012. Par défaut, ce service est activé par le début du déclencheur manuel.
* Une connectivité réseau doit exister entre au moins un contrôleur de domaine dans chaque domaine et au moins un serveur hébergeant le service proxy pour la protection par mot de passe Azure AD. Cette connectivité doit autoriser le contrôleur de domaine à accéder au port 135 du mappeur de point de terminaison RPC et au port du serveur RPC sur le service proxy.
    * Par défaut, le port du serveur RPC est un port RPC dynamique, mais il peut être configuré pour [utiliser un port statique](#static).
* Tous les ordinateurs sur lesquels le service proxy de protection par mot de passe Azure AD sera installé doivent disposer d’un accès réseau aux points de terminaison suivants :

    |**Point de terminaison**|**Objectif**|
    | --- | --- |
    |`https://login.microsoftonline.com`|Demandes d’authentification|
    |`https://enterpriseregistration.windows.net`|Fonctionnalité de protection par mot de passe Azure AD|

### <a name="azure-ad-password-protection-dc-agent"></a>agent DC de protection par mot de passe Azure AD

Les conditions suivantes s’appliquent à l’agent DC de protection de mot de passe Azure AD :

* Tous les ordinateurs sur lesquels le logiciel de l’agent DC de protection par mot de passe Azure AD seront installés doivent exécuter Windows Server 2012 ou une version ultérieure.
    * Le domaine ou la forêt Active Directory n’ont pas besoin d’être au niveau fonctionnel du domaine Windows Server 2012 (DFL) ou au niveau fonctionnel de la forêt (FFL). Comme mentionné dans [Principes de conception](concept-password-ban-bad-on-premises.md#design-principles), aucun niveau fonctionnel de domaine (DFL) ou de forêt (FFL) minimal n’est requis pour le logiciel de l’agent DC ou le logiciel de proxy à exécuter.
* .NET 4.5 doit être installé sur toutes les machines qui exécutent l’agent DC de protection par mot de passe Azure AD.
* Les domaines Active Directory qui exécutent le service de l’agent DC de protection par mot de passe Azure AD doivent utiliser la réplication du système de fichiers distribué (DFSR) pour la réplication sysvol.
   * Si votre domaine n’utilise pas encore DFSR, vous devez le migrer pour avant d’installer la protection par mot de passe Azure AD. Pour plus d’informations, voir [Guide de migration de la réplication SYSVOL : Réplication FRS à DFS](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd640019(v=ws.10))

    > [!WARNING]
    > Le logiciel de l’agent DC de protection par mot de passe Azure AD s’installe actuellement sur des contrôleurs de domaine dans des domaines qui utilisent encore FRS (la technologie qui précède DFSR) pour la réplication sysvol, mais le logiciel ne fonctionne pas correctement dans cet environnement.
    >
    > Des effets secondaires supplémentaires peuvent entraîner l’échec de la réplication des fichiers individuels, et le succès apparent des procédures de restauration sysvol qui ne parviennent pas à répliquer tous les fichiers en mode silencieux.
    >
    > Migrez votre domaine pour utiliser DFSR dès que possible, à la fois pour les avantages inhérents à DFSR et pour débloquer le déploiement de la protection par mot de passe Azure AD. Les versions ultérieures du logiciel seront automatiquement désactivées lors de l’exécution dans un domaine utilisant encore FRS.

### <a name="azure-ad-password-protection-proxy-service"></a>Service proxy de protection de mot de passe Azure AD

Les conditions suivantes s’appliquent au service proxy de protection par mot de passe Azure AD :

* Tous les ordinateurs sur lesquels le service proxy de protection par mot de passe Azure AD sera installé doivent exécuter Windows Server 2012 R2 ou ultérieur.

    > [!NOTE]
    > Le déploiement du service proxy de protection par mot de passe Azure AD est une condition préalable obligatoire pour le déploiement de la protection de mot de passe Azure AD, même si le contrôleur de domaine peut avoir une connectivité sortant Internet directe.

* Toutes les machines sur lesquelles le service proxy de protection par mot de passe Azure AD sera installé doivent disposer de .NET 4.7.
    * .NET 4.7 doit déjà être installé sur les instances de Windows Server entièrement mises à jour. Si nécessaire, téléchargez et exécutez le programme d’installation disponible sur la page [Programme d’installation hors connexion de .NET Framework 4.7 pour Windows](https://support.microsoft.com/help/3186497/the-net-framework-4-7-offline-installer-for-windows).
* Toutes les machines qui hébergent le service proxy de protection par mot de passe Azure AD doivent être configurées pour autoriser les contrôleurs de domaine à ouvrir une session sur le service proxy. Cette capacité est contrôlée par le biais de l’affectation du privilège « Accéder à cet ordinateur à partir du réseau ».
* Toutes les machines hébergeant le service proxy de protection par mot de passe Azure AD doivent être configurées de manière à autoriser le trafic HTTP TLS 1.2 sortant.
* Un compte *Administrateur général* ou *Administrateur de la sécurité* pour inscrire la forêt et le service proxy de protection par mot de passe Azure AD auprès d’Azure AD.
* L’accès réseau doit être activé pour l’ensemble des ports et des URL spécifiés dans les [procédures de configuration de l’environnement proxy d’application](../manage-apps/application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment).

### <a name="microsoft-azure-ad-connect-agent-updater-prerequisites"></a>Conditions préalables à l’utilisation du programme de mise à jour de l’agent Microsoft Azure AD Connect

Le service du programme de mise à jour de l’agent Microsoft Azure AD Connect est installé parallèlement au service proxy de protection par mot de passe Azure AD. Une configuration supplémentaire est requise pour que le service du programme de mise à jour de l’agent Microsoft Azure AD Connect puisse fonctionner :

* Si votre environnement utilise un serveur proxy HTTP, suivez les instructions spécifiées dans [Travailler avec des serveurs proxy locaux existants](../manage-apps/application-proxy-configure-connectors-with-proxy-servers.md).
* Le service du programme de mise à jour de l’agent Microsoft Azure AD Connect nécessite également les étapes 1.2 spécifiées dans [Exigences relatives à TLS](../manage-apps/application-proxy-add-on-premises-application.md#tls-requirements).

> [!WARNING]
> Le proxy de protection par mot de passe et le proxy d’application Azure AD installent différentes versions du service du programme de mise à jour de l’agent Microsoft Azure AD Connect. Cela explique pourquoi les instructions font référence au contenu du proxy d’application. Ces différentes versions sont incompatibles quand elles sont installées côte à côte, car cela empêche le service de mise à jour de l’agent de contacter Azure pour les mises à jour logicielles. Vous ne devez donc jamais installer le proxy de protection par mot de passe Azure AD et le proxy d’application sur le même ordinateur.

## <a name="download-required-software"></a>Télécharger les logiciels requis

Il existe deux programmes d’installation requis pour un déploiement local de la protection par mot de passe Azure AD :

* Agent DC de protection de mot de passe Azure AD (*AzureADPasswordProtectionDCAgentSetup.msi*)
* Proxy de protection par mot de passe Azure AD (*AzureADPasswordProtectionProxySetup.exe*)

Téléchargez les deux programmes d’installation à partir du [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=57071).

## <a name="install-and-configure-the-proxy-service"></a>Installer et configurer le service proxy

Le service proxy de protection par mot de passe Azure AD se trouve généralement sur un serveur membre au sein de votre environnement AD DS local. Une fois l’installation terminée, le service proxy de protection par mot de passe Azure AD communique avec Azure AD pour conserver une copie des listes globales et client de mots de passe interdits pour votre locataire Azure AD.

Dans la section suivante, vous allez installer les agents DC de protection par mot de passe Azure AD sur des contrôleurs de domaine dans votre environnement AD DS local. Ces agents DC communiquent avec le service proxy pour obtenir les dernières listes de mots de passe interdits à utiliser lors du traitement des événements de modification de mot de passe au sein du domaine.

Choisissez un ou plusieurs serveurs pour héberger le service proxy de protection par mot de passe Azure AD. Les considérations suivantes s’appliquent pour le(s) serveur(s) :

* Chacun de ces services fournit uniquement des stratégies de mot de passe pour une forêt unique. La machine hôte doit être jointe à n’importe quel domaine dans cette forêt.
* Cela prend en charge l’installation de proxy du service dans des domaines racines ou enfants, ou dans une combinaison de ceux-ci.
* Vous avez besoin d’une connectivité réseau entre au moins un contrôleur de domaine dans chaque domaine de la forêt et un serveur proxy de protection par mot de passe.
* Vous pouvez exécuter le service proxy de protection par mot de passe Azure AD sur un contrôleur de domaine à des fins de test mais, dans ce cas, le contrôleur de domaine nécessite une connectivité Internet. Celle-ci peut constituer un problème de sécurité. Nous recommandons cette configuration à des fins de test uniquement.
* Nous vous recommandons d’utiliser au moins deux serveurs proxy de protection par mot de passe Azure AD par forêt à des fins de redondance, comme indiqué dans la section précédente [Considérations relatives à la haute disponibilité](#high-availability-considerations).
* Il n’est pas possible d’exécuter le service de proxy de protection par mot de passe Azure AD sur un contrôleur de domaine en lecture seule.

Pour installer le service proxy de protection par mot de passe Azure AD, procédez comme suit :

1. Pour installer le service proxy de protection de mot de passe Azure AD, exécutez le programme d’installation du logiciel `AzureADPasswordProtectionProxySetup.exe`.

    L’installation du logiciel ne nécessite pas de redémarrage et peut être automatisé à l’aide de procédures MSI standard, comme dans l’exemple suivant :
    
    ```console
    AzureADPasswordProtectionProxySetup.exe /quiet
    ```
    
    > [!NOTE]
    > Le service Pare-feu Windows doit être en cours d’exécution avant l’installation du package `AzureADPasswordProtectionProxySetup.exe` pour éviter toute erreur d’installation.
    >
    > Si le Pare-feu Windows est configuré pour ne pas s’exécuter, la solution de contournement consiste à activer et démarrer temporairement le service de Pare-feu Windows pendant l’installation. Le logiciel de proxy n’a aucune dépendance particulière sur le Pare-feu Windows après l’installation.
    >
    > Si vous utilisez un pare-feu tiers, il doit encore être configuré pour répondre aux exigences de déploiement. Celles-ci incluent l’autorisation de l’accès entrant au port 135 et au port du serveur RPC proxy. Pour plus d’informations, voir la section précédente, [Composants requis pour le déploiement](#deployment-requirements).

1. Le logiciel de proxy de protection par mot de passe Azure AD inclut un nouveau module PowerShell, `AzureADPasswordProtection`. Les étapes suivantes exécutent diverses applets de commande à partir de ce module PowerShell.

    Pour utiliser ce module, ouvrez une fenêtre PowerShell en tant qu’administrateur et importez le nouveau module comme suit :
    
    ```powershell
    Import-Module AzureADPasswordProtection
    ```

1. Pour vérifier que le service proxy de protection par mot de passe Azure AD est en cours d’exécution, utilisez la commande PowerShell suivante :

    ```powershell
    Get-Service AzureADPasswordProtectionProxy | fl
    ```

    Le résultat doit afficher l’**État** *En cours d’exécution*.

1. Le service proxy est en cours d’exécution sur la machine, mais ne dispose pas des informations d’identification nécessaires pour communiquer avec Azure AD. Inscrivez le serveur proxy de protection de mot de passe Azure AD auprès d’Azure AD en utilisant la cmdlet `Register-AzureADPasswordProtectionProxy`.

    Cette cmdlet exige les informations d’identification de l’*administrateur général* ou de l’*Administrateur de la sécurité* pour votre locataire Azure. Cette cmdlet doit également être exécutée à l'aide d'un compte disposant de privilèges d'administrateur local.

    Une fois que cette commande a réussi pour un service proxy de protection par mot de passe Azure AD, des appels supplémentaires de celle-ci réussissent mais ne sont pas nécessaires.

    L’applet de commande `Register-AzureADPasswordProtectionProxy` prend en charge les trois modes d’authentification suivants. Les deux premiers modes prennent en charge Azure AD Multi-Factor Authentication, mais pas le troisième.

    > [!TIP]
    > Il peut y avoir un retard notable avant la fin, la première fois que vous exécutez cette applet de commande pour un locataire Azure spécifique. À moins qu’une erreur soit signalée, ne vous inquiétez pas de ce retard.

     * Mode d’authentification interactive :

        ```powershell
        Register-AzureADPasswordProtectionProxy -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com'
        ```

        > [!NOTE]
        > Ce mode ne fonctionne pas sur les systèmes d’exploitation Server Core. À la place, utilisez l’un des modes d’authentification suivants. De plus, ce mode peut échouer si la configuration de sécurité renforcée d’Internet Explorer est activée. La solution de contournement consiste à désactiver cette configuration, à inscrire le proxy, puis à la réactiver.

     * Mode d’authentification de code d’appareil :

        ```powershell
        Register-AzureADPasswordProtectionProxy -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com' -AuthenticateUsingDeviceCode
        ```

        Quand vous y êtes invité, suivez le lien pour ouvrir un navigateur web et entrer le code d’authentification.

     * Mode d’authentification silencieuse (basée sur un mot de passe) :

        ```powershell
        $globalAdminCredentials = Get-Credential
        Register-AzureADPasswordProtectionProxy -AzureCredential $globalAdminCredentials
        ```

        > [!NOTE]
        > Ce mode échoue si Azure AD Multi-Factor Authentication est requis pour votre compte. Dans ce cas, utilisez l’un des deux modes d’authentification précédents, ou bien utilisez un autre compte qui ne nécessite pas d’authentification multifacteur.
        >
        > L’authentification multifacteur peut également être requise si l’inscription de l’appareil Azure (qui est utilisée en arrière-plan par la protection de mot de passe Azure AD) a été configurée pour exiger globalement une authentification multifacteur. Pour contourner cette exigence, vous pouvez utiliser un autre compte prenant en charge l’authentification multifacteur avec l’un des deux modes d’authentification précédents, ou vous pouvez également assouplir temporairement l’exigence d’authentification multifacteur pour l’inscription d’un appareil Azure.
        >
        > Pour apporter cette modification, recherchez et sélectionnez **Azure Active Directory** dans le Portail Azure, puis sélectionnez **Appareils > Paramètres de l’appareil**. Définissez l’option **Exiger Multi-factor Auth pour joindre des appareils** sur *Non*. Veillez à reconfigurer ce paramètre sur *Oui* une fois l’inscription terminée.
        >
        > Nous vous recommandons de contourner les exigences de l’authentification multifacteur à des fins de test uniquement.

    Vous n’êtes pas tenu actuellement de spécifier le paramètre *-ForestCredential*, qui est réservé pour de futures fonctionnalités.

    L’inscription du service proxy de protection par mot de passe Azure AD est requise une seule fois au cours de la durée de vie du service. Après cela, le service proxy de protection par mot de passe Azure AD effectuera automatiquement toutes les autres tâches de maintenance nécessaires.

1. Inscrivez à présent la forêt Active Directory locale avec les informations d’identification nécessaires pour communiquer avec Azure à l’aide de la cmdlet PowerShell `Register-AzureADPasswordProtectionForest`.

    > [!NOTE]
    > Si plusieurs serveurs proxy de protection par mot de passe Azure AD sont installés dans votre environnement, peu importe le serveur proxy que vous utilisez pour inscrire la forêt.

    La cmdlet exige les informations d’identification de l’*administrateur général* ou de l’*Administrateur de la sécurité* pour votre locataire Azure. Elle requiert également des privilèges d’administrateur d’entreprise Active Directory en local. Vous devez également exécuter cette applet de commande à l’aide d’un compte avec des privilèges d’administrateur local. Le compte Azure utilisé pour inscrire la forêt peut être différent du compte Windows Server AD.
    
    Cette étape est exécutée une seule fois par forêt.

    L’applet de commande `Register-AzureADPasswordProtectionForest` prend en charge les trois modes d’authentification suivants. Les deux premiers modes prennent en charge Azure AD Multi-Factor Authentication, mais pas le troisième.

    > [!TIP]
    > Il peut y avoir un retard notable avant la fin, la première fois que vous exécutez cette applet de commande pour un locataire Azure spécifique. À moins qu’une erreur soit signalée, ne vous inquiétez pas de ce retard.

     * Mode d’authentification interactive :

        ```powershell
        Register-AzureADPasswordProtectionForest -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com'
        ```

        > [!NOTE]
        > Ce mode ne fonctionnera pas sur les systèmes d’exploitation Server Core. À la place, utilisez l’un des deux modes d’authentification suivants. De plus, ce mode peut échouer si la configuration de sécurité renforcée d’Internet Explorer est activée. La solution de contournement consiste à désactiver cette configuration, à inscrire la forêt, puis à la réactiver.  

     * Mode d’authentification de code d’appareil :

        ```powershell
        Register-AzureADPasswordProtectionForest -AccountUpn 'yourglobaladmin@yourtenant.onmicrosoft.com' -AuthenticateUsingDeviceCode
        ```

        Quand vous y êtes invité, suivez le lien pour ouvrir un navigateur web et entrer le code d’authentification.

     * Mode d’authentification silencieuse (basée sur un mot de passe) :

        ```powershell
        $globalAdminCredentials = Get-Credential
        Register-AzureADPasswordProtectionForest -AzureCredential $globalAdminCredentials
        ```

        > [!NOTE]
        > Ce mode échoue si Azure AD Multi-Factor Authentication est requis pour votre compte. Dans ce cas, utilisez l’un des deux modes d’authentification précédents, ou bien utilisez un autre compte qui ne nécessite pas d’authentification multifacteur.
        >
        > L’authentification multifacteur peut également être requise si l’inscription de l’appareil Azure (qui est utilisée en arrière-plan par la protection de mot de passe Azure AD) a été configurée pour exiger globalement une authentification multifacteur. Pour contourner cette exigence, vous pouvez utiliser un autre compte prenant en charge l’authentification multifacteur avec l’un des deux modes d’authentification précédents, ou vous pouvez également assouplir temporairement l’exigence d’authentification multifacteur pour l’inscription d’un appareil Azure.
        >
        > Pour apporter cette modification, recherchez et sélectionnez **Azure Active Directory** dans le Portail Azure, puis sélectionnez **Appareils > Paramètres de l’appareil**. Définissez l’option **Exiger Multi-factor Auth pour joindre des appareils** sur *Non*. Veillez à reconfigurer ce paramètre sur *Oui* une fois l’inscription terminée.
        >
        > Nous vous recommandons de contourner les exigences de l’authentification multifacteur à des fins de test uniquement.

       Ces exemples fonctionnent uniquement si l’utilisateur actuellement connecté est également un administrateur de domaine Active Directory pour le domaine racine. Si ce n’est pas le cas, vous pouvez fournir d’autres informations d’identification de domaine via le paramètre *-ForestCredential*.

    L’inscription de la forêt Active Directory est requise une seule fois au cours de la durée de vie de la forêt. Après cela, les agents DC de protection par mot de passe Azure AD dans la forêt effectuent automatiquement toutes les autres tâches de maintenance nécessaires. Après que l’exécution de la cmdlet `Register-AzureADPasswordProtectionForest` pour une forêt a réussi, les appels supplémentaires de celle-ci réussissent mais sont superflus.
    
    Pour que l’exécution de `Register-AzureADPasswordProtectionForest` réussisse, il faut qu’au moins un contrôleur de domaine exécutant Windows Server 2012 ou version ultérieure soit disponible dans le domaine du serveur proxy de protection par mot de passe Azure AD. Le logiciel de l’agent DC de protection par mot de passe Azure AD ne doit pas nécessairement être installé sur des contrôleurs de domaine avant cette étape.

### <a name="configure-the-proxy-service-to-communicate-through-an-http-proxy"></a>Configurer le service proxy pour qu’il communique via un proxy HTTP

Si votre environnement nécessite l’utilisation d’un proxy HTTP spécifique pour communiquer avec Azure, procédez comme suit pour configurer le service de protection de mot de passe Azure AD.

Créez un fichier *AzureADPasswordProtectionProxy.exe.config* dans le dossier `%ProgramFiles%\Azure AD Password Protection Proxy\Service`. Incluez le contenu suivant :

   ```xml
   <configuration>
      <system.net>
         <defaultProxy enabled="true">
         <proxy bypassonlocal="true"
            proxyaddress="http://yourhttpproxy.com:8080" />
         </defaultProxy>
      </system.net>
   </configuration>
   ```

Si votre proxy HTTP exige une authentification, ajoutez la balise *useDefaultCredentials* :

   ```xml
   <configuration>
      <system.net>
         <defaultProxy enabled="true" useDefaultCredentials="true">
         <proxy bypassonlocal="true"
            proxyaddress="http://yourhttpproxy.com:8080" />
         </defaultProxy>
      </system.net>
   </configuration>
   ```

Dans les deux cas, remplacez `http://yourhttpproxy.com:8080` par l’adresse et le port de votre serveur proxy HTTP spécifique.

Si votre proxy HTTP est configuré pour utiliser une stratégie d’autorisation, vous devez accorder l’accès au compte d’ordinateur Active Directory de la machine qui héberge le service proxy de protection par mot de passe.

Nous vous recommandons d’arrêter et de redémarrer le service proxy de protection par mot de passe Azure AD, une fois que vous avez créé ou mis à jour le fichier *AzureADPasswordProtectionProxy.exe.config*.

Le service proxy ne prend pas en charge l’utilisation d’informations d’identification spécifiques pour la connexion à un proxy HTTP.

### <a name="configure-the-proxy-service-to-listen-on-a-specific-port"></a>Configurer le service proxy pour l’écoute sur un port spécifique

Le logiciel de l’agent DC de protection par mot de passe Azure AD sur utilise RPC sur TCP pour communiquer avec le service proxy. Par défaut, le service proxy de protection par mot de passe Azure AD est à l’écoute sur n’importe quel point de terminaison RPC dynamique disponible. Vous pouvez configurer le service pour qu'il écoute sur un port TCP spécifique, si cela s'avère nécessaire en raison de la topologie du réseau ou de la configuration du pare-feu de votre environnement.

<a id="static" /></a>Pour configurer le service afin qu’il s’exécute sous un port statique, utilisez la cmdlet `Set-AzureADPasswordProtectionProxyConfiguration` comme suit :

```powershell
Set-AzureADPasswordProtectionProxyConfiguration –StaticPort <portnumber>
```

> [!WARNING]
> Pour que ces modifications prennent effet, vous devez arrêter et redémarrer le service proxy de protection par mot de passe Azure AD.

Pour configurer le service afin qu’il s’exécute sous un port dynamique, utilisez la même procédure mais redéfinissez *StaticPort* sur zéro :

```powershell
Set-AzureADPasswordProtectionProxyConfiguration –StaticPort 0
```

> [!WARNING]
> Pour que ces modifications prennent effet, vous devez arrêter et redémarrer le service proxy de protection par mot de passe Azure AD.

Le service proxy de protection par mot de passe Azure AD nécessite un redémarrage manuel après toute modification de la configuration du port. Vous n’êtes pas obligé de redémarrer le service de l’agent DC de protection par mot de passe Azure AD sur les contrôleurs de domaine après avoir apporté ces modifications de configuration.

Pour exécuter une requête afin d’obtenir la configuration actuelle du service, utilisez la cmdlet `Get-AzureADPasswordProtectionProxyConfiguration` comme illustré dans l’exemple

```powershell
Get-AzureADPasswordProtectionProxyConfiguration | fl
```

L’exemple de sortie suivant montre que le service proxy de protection par mot de passe Azure AD utilise un port dynamique :

```output
ServiceName : AzureADPasswordProtectionProxy
DisplayName : Azure AD password protection Proxy
StaticPort  : 0
```

## <a name="install-the-dc-agent-service"></a>Installer le service de l’agent DC

Pour installer le service de l’agent DC de protection par mot de passe Azure AD, exécutez le package `AzureADPasswordProtectionDCAgentSetup.msi`.

Vous pouvez automatiser l’installation du logiciel à l’aide de procédures MSI standard, comme illustré dans l’exemple suivant :

```console
msiexec.exe /i AzureADPasswordProtectionDCAgentSetup.msi /quiet /qn /norestart
```

Vous pouvez omettre l’indicateur `/norestart` si vous préférez que le programme d’installation redémarre automatiquement la machine.

L’installation ou la désinstallation du logiciel nécessitent un redémarrage. Cette exigence découle du fait que les DLL de filtrage de mots de passe ne sont chargées ou déchargées que par un redémarrage.

L’installation de la protection par mot de passe Azure AD localement se termine une fois que le logiciel de l’agent DC a été installé sur un contrôleur de domaine et que cet ordinateur a été redémarré. Aucune configuration supplémentaire n'est nécessaire ou possible. Les événements de modification de mot de passe sur les contrôleurs de local utilisent les listes de mots de passe interdits configurés d’Azure AD.

Pour activer la protection par mot de passe Azure AD localement à partir du portail Azure ou configurer des mots de passe interdits personnalisés, voir [Activer la protection par mot de passe Azure AD localement](howto-password-ban-bad-on-premises-operations.md).

> [!TIP]
> Vous pouvez installer l’agent DC de protection par mot de passe Azure AD sur une machine qui n’est pas encore un contrôleur de domaine. Dans ce cas, le service démarre et s’exécute, mais reste inactif jusqu’à ce que la machine soit promue en tant que contrôleur de domaine.

## <a name="upgrading-the-proxy-service"></a>Mise à niveau du service proxy

Le service proxy de protection par mot de passe Azure AD prend en charge la mise à niveau automatique. La mise à niveau automatique utilise le service du programme de mise à jour de l’agent Microsoft Azure AD Connect, installé à côté du service proxy. La mise à niveau automatique est activée par défaut et peut être activée ou désactivée à l’aide de l’applet de commande `Set-AzureADPasswordProtectionProxyConfiguration`.

Pour connaître le paramétrage actuel, vous pouvez utiliser l’applet de commande `Get-AzureADPasswordProtectionProxyConfiguration`. Nous recommandons de laisser la mise à niveau automatique activée en permanence.

Vous pouvez utiliser la cmdlet `Get-AzureADPasswordProtectionProxy` pour interroger la version logicielle de tous les serveurs proxy de protection de mot de passe Azure AD actuellement installés dans une forêt.

### <a name="manual-upgrade-process"></a>Processus de mise à niveau manuelle

Une mise à niveau manuelle s’accomplit en exécutant la dernière version du programme d’installation du logiciel `AzureADPasswordProtectionProxySetup.exe`. La dernière version du logiciel est disponible dans le [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=57071).

Il n’est pas nécessaire de désinstaller la version actuelle du service proxy de protection par mot de passe Azure AD, car le programme d’installation effectue une mise à niveau sur place. Aucun redémarrage n’est nécessaire lors de la mise à niveau du service proxy. Vous pouvez automatiser la mise à niveau du logiciel à l’aide de procédures MSI standard, par exemple `AzureADPasswordProtectionProxySetup.exe /quiet`.

## <a name="upgrading-the-dc-agent"></a>Mise à niveau de l’agent DC

Quand une version plus récente du logiciel de l’agent DC de protection par mot de passe Azure AD est disponible, la mise à niveau s’effectue en exécutant la dernière version du package logiciel `AzureADPasswordProtectionDCAgentSetup.msi`. La dernière version du logiciel est disponible dans le [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=57071).

Il n’est pas nécessaire de désinstaller la version actuelle du logiciel de l’agent DC, car le programme d’installation effectue une mise à niveau sur place. Un redémarrage est toujours nécessaire lors de la mise à niveau du logiciel de l’agent DC. Cette exigence découle du comportement de base de Windows.

Vous pouvez automatiser la mise à niveau du logiciel à l’aide de procédures MSI standard, par exemple `msiexec.exe /i AzureADPasswordProtectionDCAgentSetup.msi /quiet /qn /norestart`.

Vous pouvez omettre l'indicateur `/norestart` si vous préférez que le programme d’installation redémarre automatiquement l’ordinateur.

Vous pouvez utiliser la cmdlet `Get-AzureADPasswordProtectionDCAgent` pour interroger la version logicielle de tous les agents DC de protection par mot de passe Azure AD actuellement installés dans une forêt.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez installé les services dont vous avez besoin pour la protection par mot de passe Azure AD sur vos serveurs locaux, [activez la protection par mot de passe Azure AD localement via le portail Azure](howto-password-ban-bad-on-premises-operations.md) pour achever votre déploiement.