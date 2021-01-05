---
title: Utiliser Azure AD Multi-Factor Authentication avec NPS - Azure Active Directory
description: Apprenez à utiliser les fonctionnalités d'Azure AD Multi-Factor Authentication avec votre infrastructure d'authentification NPS (Network Policy Server) existante.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 08/31/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: deb05083ca45c24a58cabf9e923b706575ef093b
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97562138"
---
# <a name="integrate-your-existing-network-policy-server-nps-infrastructure-with-azure-ad-multi-factor-authentication"></a>Intégrer votre infrastructure NPS (Network Policy Server) existante à Azure AD Multi-Factor Authentication

L'extension NPS (Network Policy Server) d'Azure AD Multi-Factor Authentication permet d'ajouter des fonctionnalités MFA basées sur le cloud à votre infrastructure d'authentification à l'aide de vos serveurs existants. Avec l’extension NPS, vous pouvez ajouter des vérifications basées sur des appels téléphoniques, des SMS ou des applications mobiles à votre flux d’authentification existant sans avoir à installer, configurer et gérer les nouveaux serveurs.

L'extension NPS joue un rôle d'adaptateur entre RADIUS et la fonctionnalité Azure AD Multi-Factor Authentication basée sur le cloud pour fournir un second facteur d'authentification aux utilisateurs fédérés ou synchronisés.

## <a name="how-the-nps-extension-works"></a>Fonctionnement de l’extension NPS

Lorsque vous utilisez l'extension NPS pour Azure AD Multi-Factor Authentication, le flux d'authentification inclut les composants suivants :

1. **Le serveur NAS/VPN** reçoit les demandes des clients VPN et les convertit en demandes RADIUS à des serveurs NPS.
2. Le **serveur NPS** se connecte à Active Directory Domain Services (AD DS) afin de procéder à l’authentification principale pour les requêtes RADIUS et, en cas de réussite, transmet la requête à toutes les extensions installées.  
3. L'**extension NPS** déclenche une requête destinée à Azure AD Multi-Factor Authentication pour l'authentification secondaire. Une fois que l’extension reçoit la réponse, et si la demande MFA réussit, elle termine la demande d’authentification en fournissant au serveur NPS des jetons de sécurité qui incluent une revendication MFA, émise par Azure STS.
4. **Azure AD MFA** communique avec Azure Active Directory (Azure AD) pour récupérer les informations de l'utilisateur, et procède à l'authentification secondaire à l'aide d'une méthode de vérification configurée par l'utilisateur.

Le diagramme suivant illustre ce flux de demande d’authentification de niveau supérieur :

![Diagramme du flux d'authentification pour l'utilisateur qui s'authentifie par le biais d'un serveur VPN auprès du serveur NPS et de l'extension NPS d'Azure AD Multi-Factor Authentication](./media/howto-mfa-nps-extension/auth-flow.png)

### <a name="radius-protocol-behavior-and-the-nps-extension"></a>Comportement du protocole RADIUS et extension NPS

RADIUS étant un protocole UDP, l’expéditeur suppose une perte de paquet et attend une réponse. Après un certain laps de temps, la connexion peut expirer. Dans ce cas, le paquet est renvoyé lorsque l’expéditeur part du principe que le paquet n’a pas atteint la destination. Dans le scénario d’authentification de cet article, les serveurs VPN envoient la requête et attendent une réponse. Si la connexion expire, le serveur VPN renvoie la requête.

![Diagramme de flux de paquet UDP RADIUS et des requêtes après expiration du délai d’attente suite à réponse du serveur NPS](./media/howto-mfa-nps-extension/radius-flow.png)

Le serveur NPS peut ne pas répondre à la requête d’origine du serveur VPN avant l’expiration de la connexion, car il se peut que la requête MFA soit encore en cours de traitement. L'utilisateur n'ayant peut-être pas répondu à l'invite MFA, l'extension NPS d'Azure AD Multi-Factor Authentication attend la fin de cet événement. Dans ce cas, le serveur NPS identifie les requêtes de serveur VPN supplémentaires comme des requêtes dupliquées. Le serveur NPS ignore ces requêtes de serveur VPN dupliquées.

![Diagramme du serveur NPS ignorant les requêtes dupliquées en provenance du serveur RADIUS](./media/howto-mfa-nps-extension/discard-duplicate-requests.png)

Si vous examinez les journaux du serveur NPS, vous pouvez constater que ces requêtes supplémentaires sont ignorées. Ce comportement est normal, et destiné à empêcher que l’utilisateur final reçoive plusieurs requêtes pour une même tentative d’authentification. Les requêtes ignorées dans le journal des événements du serveur NPS ne signifient pas qu'il y a un problème au niveau du serveur NPS ou de l'extension NPS d'Azure AD Multi-Factor Authentication.

Pour réduire le nombre de requêtes ignorées, nous recommandons de configurer les serveurs VPN avec un délai d’expiration d’au moins 60 secondes. Si nécessaire, ou afin de réduire le nombre de requêtes ignorées dans les journaux des événements, vous pouvez augmenter la valeur du délai d’attente du serveur VPN à 90 ou 120 secondes.

En raison de ce comportement de protocole UDP, le serveur NPS pourrait recevoir une requête dupliquée et envoyer une autre invite MFA, même après que l’utilisateur a déjà répondu à la requête initiale. Pour éviter cette condition, l'extension NPS d'Azure AD Multi-Factor Authentication continue de filtrer et d'ignorer les requêtes dupliquées pendant 10 secondes après l'envoi d'une réponse de succès au serveur VPN.

![Diagramme du serveur NPS continuant à ignorer les requêtes dupliquées en provenance du serveur VPN pendant 10 secondes après le renvoi d’une réponse de succès](./media/howto-mfa-nps-extension/delay-after-successful-authentication.png)

Là encore, vous pouvez constater la présence de requêtes rejetées dans les journaux des événements du serveur NPS, même lorsque l'invite Azure AD Multi-Factor Authentication a abouti. Ce comportement est attendu et ne signifie pas qu'il y a un problème au niveau du serveur NPS ou de l'extension NPS d'Azure AD Multi-Factor Authentication.

## <a name="plan-your-deployment"></a>Planifier votre déploiement

L’extension NPS gère automatiquement la redondance, vous n’avez pas besoin d’une configuration spéciale.

Vous pouvez créer autant de serveurs NPS compatibles avec Azure AD Multi-Factor Authentication que vous le souhaitez. Si vous installez plusieurs serveurs, vous devez utiliser un certificat client différent pour chacun d'entre eux. La création d’un certificat pour chaque serveur signifie que vous pouvez mettre à jour chaque certificat individuellement et que vous n’avez pas à craindre une interruption de service sur tous vos serveurs.

Comme les serveurs VPN routent les requêtes d'authentification, ils doivent connaître les nouveaux serveurs NPS compatibles avec Azure AD Multi-Factor Authentication.

## <a name="prerequisites"></a>Prérequis

L’extension NPS est conçue pour fonctionner avec votre infrastructure existante. Vérifiez que les conditions préalables suivantes sont remplies avant de commencer.

### <a name="licenses"></a>Licences

L'extension NPS d'Azure AD Multi-Factor Authentication est accessible aux clients qui disposent de [licences Azure AD Multi-Factor Authentication](./concept-mfa-howitworks.md). Les licences Azure AD Multi-Factor Authentication basées sur la consommation, telles que les licences par utilisateur ou par authentification, ne sont pas compatibles avec l'extension NPS.

### <a name="software"></a>Logiciel

Windows Server 2012 ou version ultérieure

### <a name="libraries"></a>Bibliothèques

Vous devez installer manuellement la bibliothèque suivante :

- [Package redistribuable Visual C++ pour Visual Studio 2015](https://www.microsoft.com/download/details.aspx?id=48145)

Les bibliothèques suivantes sont installées automatiquement avec l’extension.

- [Visual C++ Redistributable Packages pour Visual Studio 2013 (X64)](https://www.microsoft.com/download/details.aspx?id=40784)
- [Module Microsoft Azure Active Directory pour Windows PowerShell version 1.1.166.0](https://www.powershellgallery.com/packages/MSOnline/1.1.166.0)

Le module Microsoft Azure Active Directory pour Windows PowerShell, s’il n’est pas présent, est également installé par le biais d’un script de configuration que vous exécutez dans le cadre du processus de configuration. Il est inutile d’installer ce module en avance s’il n’est pas déjà installé.

### <a name="azure-active-directory"></a>Azure Active Directory

Tous les utilisateurs de l’extension NPS doivent être synchronisés avec Azure AD à l’aide d’Azure AD Connect et doivent être inscrits pour l’authentification MFA.

Lorsque vous installez l’extension, vous devez disposer de l’*ID de locataire* et des informations d’identification de l’administrateur pour votre locataire Azure AD. Pour obtenir l’ID locataire, suivez ces étapes :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général du client Azure.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Dans la page **Vue d’ensemble**, les *Informations du locataire* sont affichées. À côté de l’*ID locataire*, sélectionnez l’icône **Copier**, comme indiqué dans la capture d’écran de l’exemple suivant :

   ![Obtention l’ID locataire depuis le Portail Azure](./media/howto-mfa-nps-extension/azure-active-directory-tenant-id-portal.png)

### <a name="network-requirements"></a>Configuration requise pour le réseau

Le serveur NPS doit pouvoir communiquer avec les URL suivantes sur les ports 80 et 443 :

* *https:\//adnotifications.windowsazure.com*
* *https:\//login.microsoftonline.com*
* *https:\//credentials.azure.com*

En outre, la connectivité aux adresses URL suivantes est nécessaire pour terminer la [configuration de l’adaptateur à l’aide du script PowerShell fourni](#run-the-powershell-script) :

* *https:\//login.microsoftonline.com*
* *https:\//provisioningapi.microsoftonline.com*
* *https:\//aadcdn.msauth.net*

## <a name="prepare-your-environment"></a>Préparation de votre environnement

Avant d’installer l’extension NPS, préparez votre environnement pour qu’il gère le trafic d’authentification.

### <a name="enable-the-nps-role-on-a-domain-joined-server"></a>Activer le rôle NPS sur un serveur joint à un domaine

Le serveur NPS se connecte à Azure AD et authentifie les requêtes MFA. Choisissez un serveur pour ce rôle. Nous vous recommandons de choisir un serveur qui ne gère pas les demandes provenant d’autres services, car l’extension NPS génère des erreurs pour toutes les demandes qui ne sont pas RADIUS. Vous devez configurer le serveur NPS en tant que serveur d’authentification principal et secondaire pour votre environnement. Il ne peut pas servir de proxy pour les requêtes RADIUS vers un autre serveur.

1. Sur votre serveur, ouvrez **Gestionnaire de serveur**. Sélectionnez **Assistant Ajout de rôles et de fonctionnalités** dans le menu *Démarrage rapide*.
2. Choisissez **Installation basée sur un rôle ou une fonctionnalité** comme type d’installation.
3. Sélectionnez le rôle de serveur **Services de stratégie et d’accès réseau**. Une fenêtre peut s’afficher pour vous informer des fonctionnalités supplémentaires requises pour exécuter ce rôle.
4. Suivez les instructions de l’Assistant jusqu’à la page *Confirmation*. Quand vous êtes prêt, sélectionnez **Installer**.

L’installation du rôle serveur NPS peut prendre quelques minutes. Une fois terminé, poursuivez avec les sections suivantes pour configurer ce serveur afin qu’il gère les requêtes RADIUS entrantes en provenance de la solution VPN.

### <a name="configure-your-vpn-solution-to-communicate-with-the-nps-server"></a>Configurer votre solution VPN pour communiquer avec le serveur NPS

En fonction de la solution VPN utilisée, la procédure servant à configurer votre stratégie d’authentification RADIUS n’est pas la même. Configurez votre stratégie VPN pour qu’elle pointe vers votre serveur NPS RADIUS.

### <a name="sync-domain-users-to-the-cloud"></a>Synchronisation des utilisateurs de domaine dans le cloud

Même si cette étape peut déjà avoir été effectuée sur votre client, il est judicieux de vérifier qu’Azure AD Connect a récemment synchronisé vos bases de données.

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.
2. Sélectionnez **Azure Active Directory** > **Connexion Azure AD**
3. Vérifiez que votre état de synchronisation est **Activée** et que la dernière synchronisation a eu lieu il y a moins d’une heure.

Si vous avez besoin de lancer un nouveau cycle de synchronisation, consultez [Synchronisation d’Azure AD Connect : Planificateur de synchronisation Azure AD Connect](../hybrid/how-to-connect-sync-feature-scheduler.md#start-the-scheduler).

### <a name="determine-which-authentication-methods-your-users-can-use"></a>Déterminer les méthodes d’authentification que vos utilisateurs peuvent employer

Il existe deux facteurs qui affectent les méthodes d’authentification disponibles avec un déploiement d’extension NPS :

* L’algorithme de chiffrement de mot de passe utilisé entre le client RADIUS (VPN, Netscaler server ou autre) et les serveurs NPS.
   - **PAP** prend en charge toutes les méthodes d'authentification d'Azure AD Multi-Factor Authentication dans le cloud : appel téléphonique, message texte à sens unique, notification d'application mobile, jetons matériels OATH et code de vérification d'application mobile.
   - **CHAPv2** et **EAP** prennent en charge l’appel téléphonique et la notification d’application mobile.

    > [!NOTE]
    > Lorsque vous déployez l’extension NPS, utilisez ces facteurs pour déterminer quelles méthodes sont disponibles pour vos utilisateurs. Si votre point d’accès sans fil compatible 802.1X prend en charge PAP, mais que le client UX ne dispose pas des champs d’entrée pour un code de vérification, l’appel téléphonique et la notification d’application mobile sont les deux options prises en charge.
    >
    > Par ailleurs, quel que soit le protocole d’authentification utilisé (PAP, CHAP ou EAP), si vous optez pour une méthode MFA textuelle (SMS, code de vérification d’application mobile ou module de sécurité matériel OAuth) invitant l’utilisateur à entrer un code ou du texte dans le champ d’entrée de l’interface utilisateur du client VPN, il est possible que l’authentification réussisse. *Toutefois*, les attributs RADIUS configurés dans la stratégie d’accès réseau ne sont *pas* transférés au client RADIUS (l’appareil d’accès réseau, comme la passerelle VPN). Par conséquent, le client VPN pourrait disposer d’un niveau d’accès différent du niveau attendu (supérieur, inférieur ou inexistant).

* Les méthodes d’entrée que l’application cliente (VPN, Netscaler server ou autre) peut gérer. Par exemple, le client VPN dispose-t-il de moyens permettant d’autoriser l’utilisateur à taper un code de vérification à partir d’un texte ou d’une application mobile ?

Vous pouvez [désactiver les méthodes d’authentification non prises en charge](howto-mfa-mfasettings.md#verification-methods) dans Azure.

### <a name="register-users-for-mfa"></a>Inscrire des utilisateurs pour l’authentification MFA

Avant de déployer et d'utiliser l'extension NPS, les utilisateurs soumis à Azure AD Multi-Factor Authentication doivent être inscrits pour l'authentification MFA. Pour tester l'extension lorsque vous la déployez, il vous faut également au moins un compte de test entièrement inscrit pour Azure AD Multi-Factor Authentication.

Si vous devez créer et configurer un compte de test, effectuez les étapes suivantes :

1. Connectez-vous à [https://aka.ms/mfasetup](https://aka.ms/mfasetup) avec un compte de test.
2. Suivez les instructions pour configurer une méthode de vérification.
3. Dans le portail Azure en tant qu’utilisateur administrateur, [créez une stratégie d’accès conditionnel](howto-mfa-getstarted.md#create-conditional-access-policy) afin d’exiger l’authentification multifacteur pour le compte de test.

> [!IMPORTANT]
>
> Vérifiez que les utilisateurs sont bien inscrits à Azure AD Multi-Factor Authentication. S’ils ne disposent que d’une inscription à la réinitialisation de mot de passe en libre-service (SSPR), *StrongAuthenticationMethods* est activé pour leur compte. Azure AD Multi-Factor Authentication est appliqué lorsque *StrongAuthenticationMethods* est configuré, même avec une simple inscription à SSPR.
>
> L'inscription de sécurité combinée peut être activée pour configurer la réinitialisation SSPR et Azure AD Multi-Factor Authentication en même temps. Pour plus d’informations, consultez [Activation de l’inscription combinée des informations de sécurité dans Azure Active Directory](howto-registration-mfa-sspr-combined.md).
>
> Vous pouvez également [obliger les utilisateurs à se réinscrire à certaines méthodes d’authentification](howto-mfa-userdevicesettings.md#manage-user-authentication-options) s’ils n’avaient activé que SSPR.

## <a name="install-the-nps-extension"></a>Installer l’extension NPS

> [!IMPORTANT]
> Installez l’extension NPS sur un serveur autre que le point d’accès VPN.

### <a name="download-and-install-the-nps-extension-for-azure-ad-mfa"></a>Télécharger et installer l'extension NPS d'Azure AD MFA

Pour télécharger et installer l’extension NPS, effectuez les étapes suivantes :

1. [Téléchargez l’extension NPS](https://aka.ms/npsmfa) à partir du Centre de téléchargement Microsoft.
1. Copiez le fichier binaire sur le serveur NPS (Network Policy Server) que vous souhaitez configurer.
1. Exécutez le fichier *setup.exe* et suivez les instructions d’installation. Si vous rencontrez des erreurs, vérifiez que les [bibliothèques de la section Prérequis](#libraries) ont été installées avec succès.

#### <a name="upgrade-the-nps-extension"></a>Mettre à niveau l’extension NPS

Si vous effectuez ultérieurement la mise à niveau d’une extension NPS existante, effectuez les étapes suivantes pour éviter la réinitialisation du serveur sous-jacent :

1. Désinstallez la version existante.
1. Exécutez le nouveau programme d’installation.
1. Redémarrez le service *Network Policy Server (IAS)* .

### <a name="run-the-powershell-script"></a>Exécution du script PowerShell

Le programme d’installation crée un script PowerShell à `C:\Program Files\Microsoft\AzureMfa\Config` (où `C:\` est le lecteur d’installation). Ce script PowerShell effectue les opérations suivantes à chaque exécution :

* Crée un certificat auto-signé
* Associe la clé publique du certificat au principal du service sur Azure AD
* Stocke le certificat dans le magasin de certificats de la machine locale
* Accorde à l’utilisateur réseau un accès à la clé privée du certificat
* Redémarre le service NPS

À moins que vous ne souhaitiez utiliser vos propres certificats (au lieu des certificats auto-signés générés par le script PowerShell), exécutez le script PowerShell pour terminer l’installation de l’extension NPS. Si vous installez l’extension sur plusieurs serveurs, chacun d’eux doit avoir son propre certificat.

Pour fournir des fonctionnalités d’équilibrage de charge ou de redondance, répétez ces étapes sur d’autres serveurs NPS en fonction de vos besoins :

1. Ouvrez une invite Windows PowerShell en tant qu’administrateur.
1. Accédez au répertoire dans lequel le programme d’installation a créé le script PowerShell :

   ```powershell
   cd "C:\Program Files\Microsoft\AzureMfa\Config"
   ```

1. Exécutez le script PowerShell créé par le programme d’installation.

   Vous devrez peut-être d’abord activer le protocole TLS 1.2 pour que PowerShell puisse se connecter et télécharger les packages correctement :
   
   `[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12`

   > [!IMPORTANT]
   > Pour les clients qui utilisent les clouds Azure Government ou Azure China 21Vianet, commencez par modifier les cmdlets `Connect-MsolService` dans le script *AzureMfaNpsExtnConfigSetup.ps1* pour inclure les paramètres *AzureEnvironment* pour le cloud requis. Par exemple, spécifiez *-AzureEnvironment USGovernment* ou *-AzureEnvironment AzureChinaCloud*.
   >
   > Pour plus d’informations, consultez [Référence du paramètre Connect-MsolService](/powershell/module/msonline/connect-msolservice#parameters).

   ```powershell
   .\AzureMfaNpsExtnConfigSetup.ps1
   ```

1. À l’invite, connectez-vous à Azure AD en tant qu’administrateur.
1. PowerShell vous invite à entrer votre ID client. Utilisez l’*ID de locataire* que vous avez copié à partir du portail Azure dans la section Configuration requise.
1. Un message de réussite s’affiche lorsque le script est terminé.  

Si votre certificat d’ordinateur précédent est arrivé à expiration, et qu’un nouveau certificat a été généré, vous devez supprimer tous les certificats arrivés à expiration. En effet, des certificats arrivés à expiration peuvent provoquer des problèmes lors du démarrage de l’extension NPS.

> [!NOTE]
> Si vous utilisez vos propres certificats au lieu de générer des certificats avec le script PowerShell, vous devez veiller à ce qu’ils respectent la convention de nommage du serveur NPS. Le nom de l’objet doit être **CN=\<TenantID\>,OU=Microsoft NPS Extension**.

### <a name="microsoft-azure-government-or-azure-china-21vianet-additional-steps"></a>Étapes supplémentaires pour Microsoft Azure Government ou Azure Chine

Pour les clients qui utilisent les clouds Azure Government ou Azure Chine, les étapes de configuration supplémentaires suivantes sont requises sur chaque serveur NPS.

> [!IMPORTANT]
> Configurez ces paramètres de registre uniquement si vous êtes un client Azure Government ou Azure Chine.

1. Si vous êtes un client Azure Government ou Azure Chine, ouvrez l’**Éditeur du Registre** sur le serveur NPS.
1. Accédez à `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\AzureMfa`.
1. Pour les clients Azure Government, définissez les valeurs de clés suivantes :

    | Clé de Registre       | Valeur |
    |--------------------|-----------------------------------|
    | AZURE_MFA_HOSTNAME | adnotifications.windowsazure.us   |
    | STS_URL            | https://login.microsoftonline.us/ |

1. Pour les clients Azure Chine, définissez les valeurs de clés suivantes :

    | Clé de Registre       | Valeur |
    |--------------------|-----------------------------------|
    | AZURE_MFA_HOSTNAME | adnotifications.windowsazure.cn   |
    | STS_URL            | https://login.chinacloudapi.cn/   |

1. Répétez les deux étapes précédentes afin de définir les valeurs de clé de Registre pour chaque serveur NPS.
1. Redémarrez le service NPS pour chaque serveur NPS.

    Pour un impact minimal, retirez un par un les différents serveurs NPS de la rotation NLB (équilibrage de la charge réseau) et attendez que toutes les connexions soient drainées.

### <a name="certificate-rollover"></a>Substitution de certificat

Dans la version *1.0.1.32* de l’extension NPS, la lecture de plusieurs certificats est désormais prise en charge. Cette fonctionnalité contribue à faciliter la propagation des mises à jour de certificats avant leur arrivée à expiration. Si votre organisation exécute une version antérieure de l’extension NPS, mettez-la à niveau vers la version *1.0.1.32* ou ultérieure.

Les certificats créés par le script `AzureMfaNpsExtnConfigSetup.ps1` sont valides pendant 2 ans. Supervisez l’expiration des certificats. Les certificats de l’extension NPS sont placés dans le magasin de certificats de l’*Ordinateur local* sous *Personnel*, et sont *Envoyés à* l’ID de locataire fourni au script d’installation.

Lorsque la date d’expiration d’un certificat approche, un nouveau certificat doit être généré pour le remplacer.  Pour cela, vous devez exécuter à nouveau le script `AzureMfaNpsExtnConfigSetup.ps1`, en indiquant le même ID de locataire à l’invite. Ce processus doit être répété sur chaque serveur NPS au sein de votre environnement.

## <a name="configure-your-nps-extension"></a>Configurer votre extension NPS

Une fois votre environnement préparé et l’extension du serveur NPS installée sur les serveurs requis, vous pouvez configurer l’extension.

Cette section comprend des considérations relatives à la conception ainsi que des suggestions visant à garantir la réussite des déploiements d’extension NPS.

### <a name="configuration-limitations"></a>Limites de configuration

- L'extension NPS d'Azure AD Multi-Factor Authentication n'inclut pas les outils permettant de migrer les utilisateurs et les paramètres du serveur MFA vers le cloud. De ce fait, nous vous conseillons d’utiliser l’extension pour les nouveaux déploiements, plutôt que pour les déploiements existants. Si vous utilisez l’extension sur un déploiement existant, vos utilisateurs doivent suivre de nouveau la procédure de vérification pour remplir leurs détails MFA dans le cloud.  
- L'extension NPS utilise le nom d'utilisateur principal (UPN) de l'environnement AD DS local pour identifier l'utilisateur sur Azure AD Multi-Factor Authentication afin de procéder à l'authentification secondaire. L’extension peut être configurée pour utiliser un identificateur autre que l’UPN (un autre ID de connexion ou un champ AD DS personnalisé, par exemple). Pour plus d’informations, consultez l’article [Options de configuration avancée de l’extension de serveur NPS pour l’authentification multifacteur](howto-mfa-nps-extension-advanced.md).
- Tous les protocoles de chiffrement ne prennent pas en charge toutes les méthodes de vérification.
   - **PAP** prend en charge l’appel téléphonique, le message texte à sens unique, la notification de l’application mobile et le code de vérification de l’application mobile
   - **CHAPv2** et **EAP** prennent en charge l’appel téléphonique et la notification d’application mobile

### <a name="control-radius-clients-that-require-mfa"></a>Contrôler les clients RADIUS nécessitant l’authentification MFA

Une fois que vous activez MFA pour un client RADIUS à l’aide de l’extension NPS, toutes les authentifications de ce client doivent utiliser la méthode MFA. Si vous souhaitez activer l’authentification MFA pour certains clients RADIUS et d’autres non, vous pouvez configurer deux serveurs NPS et installer l’extension sur un seul d’entre eux.

Configurez les clients RADIUS pour lesquels vous souhaitez exiger l’authentification MFA de sorte qu’ils envoient des demandes au serveur NPS configuré avec l’extension, et les autres clients RADIUS pour qu’ils les envoient au serveur NPS non configuré avec l’extension.

### <a name="prepare-for-users-that-arent-enrolled-for-mfa"></a>Anticiper la présence d’utilisateurs qui ne sont pas inscrits pour l’authentification MFA

Si vous avez des utilisateurs qui ne sont pas inscrits pour l’authentification MFA, vous pouvez déterminer ce qui se passe lorsqu’ils tentent de s’authentifier. Pour contrôler ce comportement, utilisez le paramètre *REQUIRE_USER_MATCH* dans le chemin de Registre *HKLM\Software\Microsoft\AzureMFA*. Ce paramètre ne dispose que d’une seule option de configuration :

| Clé | Valeur | Default |
| --- | ----- | ------- |
| REQUIRE_USER_MATCH | TRUE/FALSE | Non défini (équivaut à TRUE) |

Ce paramètre détermine ce qu’il faut faire lorsqu’un utilisateur n’est pas inscrit pour MFA. Lorsque la clé n’existe pas, n’est pas définie ou est définie sur *TRUE*, et que l’utilisateur n’est pas inscrit, l’extension échoue à la requête d’authentification MFA.

Lorsque la clé est définie sur *FALSE* et que l’utilisateur n’est pas inscrit, l’authentification s’effectue sans procéder à l’authentification MFA. Si un utilisateur est inscrit dans MFA, il doit s’authentifier avec MFA même si *REQUIRE_USER_MATCH* est défini sur *FALSE*.

Vous pouvez choisir de créer cette clé et de lui affecter la valeur *FALSE* pour vos utilisateurs qui sont en cours d'intégration et ne sont peut-être pas encore inscrits pour Azure AD Multi-Factor Authentication. Toutefois, étant donné que la définition de la clé permet aux utilisateurs qui ne sont pas inscrits pour l’authentification MFA de se connecter, vous devez supprimer cette clé avant de passer en production.

## <a name="troubleshooting"></a>Dépannage

### <a name="nps-extension-health-check-script"></a>Script de vérification de l’intégrité de l’extension NPS

Le script suivant permet de vérifier l’intégrité de base lors de la résolution des problèmes relatifs à l’extension NPS.

[MFA_NPS_Troubleshooter.ps1](/samples/azure-samples/azure-mfa-nps-extension-health-check/azure-mfa-nps-extension-health-check/)

### <a name="how-do-i-verify-that-the-client-cert-is-installed-as-expected"></a>Comment vérifier que le certificat client est installé comme prévu ?

Recherchez le certificat auto-signé créé par le programme d’installation dans le magasin de certificats et vérifiez que la clé privée dispose d’autorisations accordées à l’utilisateur *NETWORK SERVICE*. Le certificat porte un nom d’objet de type **CN \<tenantid\>, OU = Microsoft NPS Extension**.

Les certificats auto-signés générés par le script `AzureMfaNpsExtnConfigSetup.ps1` ont une durée de validité de deux ans. Lorsque vous vérifiez que le certificat est installé, assurez-vous également qu’il n’est pas arrivé à expiration.

### <a name="how-can-i-verify-that-my-client-certificate-is-associated-to-my-tenant-in-azure-ad"></a>Comment vérifier que mon certificat client est associé à mon locataire dans Azure AD ?

Ouvrez une invite de commandes PowerShell et exécutez les commandes suivantes :

```powershell
import-module MSOnline
Connect-MsolService
Get-MsolServicePrincipalCredential -AppPrincipalId "981f26a1-7f43-403b-a875-f8b09b8cd720" -ReturnKeyValues 1
```

Ces commandes impriment tous les certificats qui associent votre client à votre instance de l’extension NPS dans votre session PowerShell. Recherchez votre certificat en exportant votre certificat client en tant que fichier *X.509 (.cer) encodé en base 64* sans la clé privée et comparez-le à la liste de PowerShell.

La commande suivante crée un fichier nommé *npscertificate* à la racine de votre lecteur *C:* au format *.cer*.

```powershell
import-module MSOnline
Connect-MsolService
Get-MsolServicePrincipalCredential -AppPrincipalId "981f26a1-7f43-403b-a875-f8b09b8cd720" -ReturnKeyValues 1 | select -ExpandProperty "value" | out-file c:\npscertificate.cer
```

Une fois que vous avez exécuté cette commande, accédez à la racine de votre lecteur *C:* , recherchez le fichier et double-cliquez dessus. Accédez aux détails, puis faites défiler jusqu’à « empreinte numérique ». Comparez l’empreinte numérique du certificat installé sur le serveur à celle-ci. Les empreintes des certificats doivent correspondre.

Les horodatages *Valid-From* (Valide à partir de) et *Valid-Until* (Valide jusqu’à), qui sont dans un format explicite, peuvent être utilisés pour filtrer les certificats de toute évidence non appropriés si la commande en retourne plusieurs.

### <a name="why-cannot-i-sign-in"></a>Pourquoi ne puis-je pas me connecter ?

Vérifiez que votre mot de passe n’a pas expiré. L’extension NPS ne prend pas en charge les changements de mots de passe dans le cadre du workflow de connexion. Contactez le service informatique de votre entreprise pour obtenir de l’aide.

### <a name="why-are-my-requests-failing-with-adal-token-error"></a>Pourquoi mes demandes échouent-elles avec une erreur de jeton ADAL ?

Cette erreur peut être due à plusieurs raisons. Effectuez les étapes suivantes pour résoudre le problème :

1. Redémarrez votre serveur NPS.
2. Vérifiez que le certificat client est installé comme prévu.
3. Vérifiez que le certificat est associé à votre client sur Azure AD.
4. Vérifiez que `https://login.microsoftonline.com/` est accessible depuis le serveur exécutant l’extension.

### <a name="why-does-authentication-fail-with-an-error-in-http-logs-stating-that-the-user-is-not-found"></a>Pourquoi l’authentification échoue-t-elle avec une erreur dans les journaux d’activité HTTP indiquant que l’utilisateur est introuvable ?

Vérifiez qu’AD Connect est en cours d’exécution et que l’utilisateur est présent dans l’environnement AD DS local et dans Azure AD.

### <a name="why-do-i-see-http-connect-errors-in-logs-with-all-my-authentications-failing"></a>Pourquoi existe-t-il des erreurs de connexion HTTP dans les journaux d’activité pour tous mes échecs d’authentification ?

Vérifiez que https://adnotifications.windowsazure.com est accessible depuis le serveur exécutant l’extension NPS.

### <a name="why-is-authentication-not-working-despite-a-valid-certificate-being-present"></a>Pourquoi l’authentification ne fonctionne-t-elle pas, même si le certificat est valide ?

Si votre certificat d’ordinateur précédent est arrivé à expiration et qu’un nouveau certificat a été généré, supprimez tous les certificats arrivés à expiration. Des certificats arrivés à expiration peuvent provoquer des problèmes lors du démarrage de l’extension NPS.

Pour vous assurer que vous disposez d’un certificat valide, vérifiez le *magasin de certificats du compte d’ordinateur* local à l’aide de MMC pour savoir si le certificat a atteint sa date d’expiration. Pour générer un nouveau certificat valide, réexécutez les étapes de la section [Exécuter le script d’installation PowerShell](#run-the-powershell-script).

### <a name="why-do-i-see-discarded-requests-in-the-nps-server-logs"></a>Pourquoi des requêtes ignorées sont-elles visibles dans les journaux du serveur NPS ?

Un serveur VPN peut envoyer des requêtes répétées au serveur NPS si la valeur du délai d’attente est trop faible. Le serveur NPS détecte ces requêtes dupliquées et les ignore. Ce comportement est lié à la conception et ne signifie pas qu'il y a un problème au niveau du serveur NPS ou de l'extension NPS d'Azure AD Multi-Factor Authentication.

Pour plus d’informations sur les raisons pour lesquelles des paquets ignorés figurent dans les journaux du serveur NPS, consultez [Comportement du protocole RADIUS et extension NPS](#radius-protocol-behavior-and-the-nps-extension) au début de cet article.

## <a name="managing-the-tlsssl-protocols-and-cipher-suites"></a>Gestion des protocoles TLS/SSL et des suites de chiffrement

Nous vous recommandons de désactiver ou de supprimer les suites de chiffrement plus anciennes et plus faibles, sauf si votre organisation en a besoin. Vous trouverez plus d’informations sur la manière d’effectuer cette tâche dans l’article [Gestion des protocoles SSL/TLS et des suites de chiffrement pour AD FS](/windows-server/identity/ad-fs/operations/manage-ssl-protocols-in-ad-fs).

### <a name="additional-troubleshooting"></a>Résolution de problèmes supplémentaires

Vous trouverez des conseils supplémentaires et des solutions potentielles dans l'article [Résoudre les messages d'erreur liés à l'extension NPS d'Azure AD Multi-Factor Authentication](howto-mfa-nps-extension-errors.md).

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble et configuration du serveur NPS dans Windows Server](/windows-server/networking/technologies/nps/nps-top)

- Configurez d’autres ID de connexion, ou créez une liste d’exceptions pour les adresses IP qui ne nécessitent pas de vérification en deux étapes dans [Options de configuration avancée de l’extension de serveur NPS pour l’authentification multifacteur](howto-mfa-nps-extension-advanced.md)

- Apprendre à intégrer la [passerelle des services Bureau à distance](howto-mfa-nps-extension-rdg.md) et des [serveurs VPN](howto-mfa-nps-extension-vpn.md) à l’aide de l’extension NPS

- [Résoudre les messages d'erreur liés à l'extension NPS d'Azure AD Multi-Factor Authentication](howto-mfa-nps-extension-errors.md)
