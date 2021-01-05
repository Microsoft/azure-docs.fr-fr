---
title: VPN avec Azure AD MFA à l’aide de l’extension NPS – Azure Active Directory
description: Intégrez votre infrastructure VPN à Azure AD MFA en utilisant l’extension NPS (Network Policy Server) pour Microsoft Azure.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1c4c5c4e632943ebbe68003f663aebbeaab9ebaf
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743443"
---
# <a name="integrate-your-vpn-infrastructure-with-azure-ad-mfa-by-using-the-network-policy-server-extension-for-azure"></a>Intégrer votre infrastructure VPN à Azure AD MFA en utilisant l’extension NPS (Network Policy Server) pour Azure

L’extension NPS (Network Policy Server) pour Azure permet aux organisations de protéger l’authentification du client RADIUS (Remote Authentication Dial-In User Service) à l’aide du service informatique [Azure AD Multi-Factor Authentication (MFA)](howto-mfaserver-nps-rdg.md), qui offre une vérification en deux étapes.

Cet article fournit des instructions sur l’intégration de l’infrastructure NPS avec MFA par le biais de l’extension NPS pour Azure. Ce processus permet une vérification sécurisée en deux étapes des utilisateurs qui tentent de se connecter à votre réseau via un réseau privé virtuel.

Les services de stratégie et d’accès réseau donnent aux entreprises les possibilités suivantes :

* Attribuer un emplacement central pour la gestion et le contrôle des requêtes réseau afin de spécifier :

  * Qui peut se connecter

  * Les heures de connexion autorisées pendant la journée

  * La durée des connexions

  * Le niveau de sécurité que les clients doivent utiliser pour se connecter

    Plutôt que de spécifier des stratégies sur chaque serveur VPN ou serveur de passerelle Bureau à distance, vous le faites une fois qu’ils sont dans un emplacement central. Le protocole RADIUS sert à fournir de manière centralisée l’authentification, l’autorisation et la gestion des comptes.

* Établissez et appliquez les stratégies de contrôle d’intégrité client de Protection d’accès réseau (NAP) qui déterminent si les périphériques sont accordés avec ou sans restrictions d’accès aux ressources réseau.

* Donnez un moyen d’appliquer l’authentification et l’autorisation d’accès aux points d’accès sans fil et commutateurs Ethernet compatibles avec 802.1x.
  Pour plus d’informations, consultez [Serveur NPS (Network Policy Server)](/windows-server/networking/technologies/nps/nps-top).

Afin d’améliorer la sécurité et de fournir un niveau élevé de conformité, les organisations peuvent intégrer NPS à Azure AD Multi-Factor Authentication et ainsi garantir que les utilisateurs utilisent une vérification en deux étapes pour se connecter au port virtuel sur le serveur VPN. Pour que les utilisateurs puissent obtenir l’accès, ils doivent communiquer leur combinaison de nom d’utilisateur et de mot de passe ainsi que d’autres informations sur lesquelles ils disposent d’un contrôle. Ces informations doivent être approuvées et difficilement dupliquées. Elles peuvent comporter un numéro de téléphone portable, un numéro de téléphone fixe ou être une application sur un appareil mobile.

Avant la disponibilité de l’extension de serveur NPS pour Azure, les clients qui souhaitaient mettre en œuvre la vérification en deux étapes pour les environnements MFA et de serveur NPS intégrés devaient configurer et gérer un serveur MFA distinct dans un environnement local. Ce type d’authentification est offert par la passerelle Bureau à distance et le serveur Microsoft Azure Multi-Factor Authentication à l’aide de RADIUS.

Avec l’extension de serveur NPS pour Azure, les entreprises peuvent sécuriser l’authentification client RADIUS en déployant une solution MFA locale ou une solution MFA basée sur le cloud.

## <a name="authentication-flow"></a>Flux d’authentification

Lorsque les utilisateurs se connectent à un port virtuel sur un serveur VPN, ils doivent tout d’abord s’authentifier par le biais d’un certain nombre de protocoles. Ces protocoles autorisent l’utilisation d’une combinaison de nom d’utilisateur et de mot de passe ainsi que des méthodes d’authentification basées sur les certificats.

Outre l’authentification et la vérification d’identité, les utilisateurs doivent disposer des autorisations d’accès appropriées. Dans les implémentations simples, les autorisations d’accès à distance qui permettent l’accès sont définies directement sur les objets utilisateur Active Directory.

![Onglet Appel entrant dans les propriétés d’utilisateur Utilisateurs et ordinateurs Active Directory](./media/howto-mfa-nps-extension-vpn/image1.png)

Dans les implémentations simples, chaque serveur VPN autorise ou refuse l’accès en fonction des stratégies qui sont définies sur chacun des serveurs VPN locaux.

Dans les mises en œuvre plus importantes et plus évolutives, les stratégies qui accordent ou refusent l’accès VPN sont centralisées sur les serveurs RADIUS. Dans ces cas-là, le serveur VPN agit comme un serveur d’accès (client RADIUS) qui transfère les demandes de connexion et les messages de compte à un serveur RADIUS. Pour se connecter au port virtuel sur le serveur VPN, les utilisateurs doivent être authentifiés et répondre aux conditions qui sont définies de manière centralisée sur les serveurs RADIUS.

Lorsque l’extension NPS pour Azure est intégrée au serveur NPS, un flux d’authentification réussie se traduit de la manière suivante :

1. Le serveur VPN reçoit une demande d’authentification à partir d’un utilisateur VPN qui inclut le nom d’utilisateur et le mot de passe pour se connecter à une ressource, par exemple une session Bureau à distance.
2. Agissant comme un client RADIUS, le serveur VPN convertit la requête en message RADIUS de *demande d’accès* qu’il envoie (avec un mot de passe chiffré) au serveur RADIUS où est installée l’extension de serveur NPS.
3. La combinaison nom d’utilisateur et mot de passe est vérifiée dans Active Directory. Si le nom d’utilisateur ou le mot de passe est incorrect, le serveur RADIUS envoie un message de *refus d’accès*.
4. Si toutes les conditions, telles que spécifiées dans les stratégies de réseau et dans la demande de connexion NPS, sont remplies (par exemple, restrictions d’heure ou d’appartenance à un groupe), l’extension NPS déclenche une demande d’authentification secondaire avec Azure AD Multi-Factor Authentication.
5. Azure AD Multi-Factor Authentication communique avec Azure Active Directory pour récupérer les informations de l’utilisateur et procéder à l’authentification secondaire par le biais de la méthode configurée par l’utilisateur (appel sur téléphone portable, SMS ou application mobile).
6. Lorsque la demande d’authentification MFA est réussie, Azure AD Multi-Factor Authentication communique le résultat à l’extension NPS.
7. Après l’authentification et l’autorisation de la tentative de connexion, le serveur NPS où l’extension est installée envoie un message RADIUS *d’acceptation d’accès* au serveur VPN (client RADIUS).
8. L’utilisateur a accès au port virtuel sur le serveur VPN et établit un tunnel VPN chiffré.

## <a name="prerequisites"></a>Prérequis

Cette section détaille les prérequis à respecter avant de pouvoir intégrer MFA au VPN. Avant de commencer, les éléments suivants doivent être en place :

* Infrastructure VPN
* Rôle Services de stratégie et d’accès réseau
* Licence Azure AD Multi-Factor Authentication
* Logiciel Windows Server
* Bibliothèques
* Azure Active Directory (Azure AD) synchronisé avec Active Directory local
* ID du GUID Azure Active Directory

### <a name="vpn-infrastructure"></a>Infrastructure VPN

Cet article suppose que vous disposez d’une infrastructure VPN opérationnelle qui utilise Microsoft Windows Server 2016, et que votre serveur VPN n’est pas actuellement configuré pour transférer les demandes de connexion directe à un serveur RADIUS. Dans cet article, vous configurez l’infrastructure VPN pour utiliser un serveur RADIUS central.

Si vous ne disposez pas d’une infrastructure VPN opérationnelle en place, vous pouvez rapidement en créer une en suivant les instructions des nombreux didacticiels consacrés à l’installation VPN, disponibles sur le site de Microsoft et sur des sites tiers.

### <a name="the-network-policy-and-access-services-role"></a>Le rôle Services de stratégie et d’accès réseau

Les services de stratégie et d’accès réseau fournissent les fonctionnalités de client et de serveur RADIUS. Cet article suppose que vous avez installé le rôle Services de stratégie et d’accès réseau sur un serveur membre ou un contrôleur de domaine dans votre environnement. Dans ce guide, vous configurez RADIUS pour une configuration de VPN. Installez le rôle Services de stratégie et d’accès réseau sur un serveur *autre que* votre serveur VPN.

Pour plus d’informations sur l’installation du service de rôle Services de stratégie et d’accès réseau dans Windows Server 2012 ou version ultérieure, consultez [Installer un serveur de stratégie de contrôle d’intégrité NAP](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd296890(v=ws.10)). La protection d’accès réseau (NAP) est dépréciée dans Windows Server 2016. Pour obtenir une description des bonnes pratiques pour NPS, y compris la recommandation d’installation de NPS sur un contrôleur de domaine, consultez [Bonnes pratiques pour NPS](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc771746(v=ws.10)).

### <a name="azure-ad-mfa-license"></a>Licence Azure AD MFA

Une licence est nécessaire pour Azure AD Multi-Factor Authentication : elle est disponible via une licence indépendante Azure AD Premium, Enterprise Mobility + Security ou Multi-Factor Authentication. Les licences Azure AD MFA basées sur la consommation, telles que les licences par utilisateur ou par authentification, ne sont pas compatibles avec l’extension NPS. Pour plus d’informations, consultez [Guide pratique pour obtenir Azure AD Multi-Factor Authentication](concept-mfa-licensing.md). À des fins de test, vous pouvez utiliser un abonnement d’évaluation.

### <a name="windows-server-software"></a>Logiciel Windows Server

L’extension de serveur NPS nécessite Windows Server 2008 R2 SP1 ou version ultérieure, avec le rôle Services de stratégie et d’accès réseau installé. Toutes les étapes de ce guide ont été effectuées avec Windows Server 2016.

### <a name="libraries"></a>Bibliothèques

Les bibliothèques suivantes sont installées automatiquement avec l’extension de serveur NPS :

-    [Visual C++ Redistributable Packages pour Visual Studio 2013 (X64)](https://www.microsoft.com/download/details.aspx?id=40784)
-    [Module Microsoft Azure Active Directory pour Windows PowerShell version 1.1.166.0](https://connect.microsoft.com/site1164/Downloads/DownloadDetails.aspx?DownloadID=59185)

Si le module PowerShell Microsoft Azure Active Directory n’est pas déjà présent, il est installé par le biais d’un script de configuration que vous exécutez dans le cadre du processus d’installation. Il est inutile d’installer ce module en avance s’il n’est pas déjà installé.

### <a name="azure-active-directory-synced-with-on-premises-active-directory"></a>Azure Active Directory synchronisé avec Active Directory local

Pour utiliser l’extension de serveur NPS, les utilisateurs locaux doivent être synchronisés avec Azure Active Directory, et activés pour MFA. Ce guide présuppose que les utilisateurs locaux sont synchronisés avec Azure Active Directory via AD Connect. Vous trouverez ci-dessous des instructions pour fournir aux utilisateurs l’authentification multifacteur.

Pour obtenir plus d’informations sur Azure AD Connect, consultez [Intégrer vos annuaires locaux à Azure Active Directory](../hybrid/whatis-hybrid-identity.md).

### <a name="azure-active-directory-guid-id"></a>ID du GUID Azure Active Directory

Pour installer l’extension de serveur NPS, vous devez connaître le GUID d’Azure Active Directory. Instructions pour rechercher le GUID d’Azure Active Directory sont fournies dans la section suivante.

## <a name="configure-radius-for-vpn-connections"></a>Configurer RADIUS pour les connexions VPN

Si vous avez installé le rôle NPS sur un serveur membre, vous devez le configurer pour authentifier et autoriser le client VPN qui demande des connexions VPN. 

Cette section présuppose que vous avez installé le rôle Services de stratégie et d’accès réseau, mais que vous ne l’avez pas configuré pour l’utiliser dans votre infrastructure.

> [!NOTE]
> Si vous disposez déjà d’un serveur VPN de travail qui utilise un serveur RADIUS centralisé pour l’authentification, vous pouvez ignorer cette section.
>

### <a name="register-server-in-active-directory"></a>Enregistrer le serveur dans Active Directory

Pour fonctionner correctement dans ce scénario, le serveur NPS doit être enregistré dans Active Directory.

1. Ouvrez le Gestionnaire de serveurs.

2. Dans Gestionnaire de serveur, sélectionnez **Outils**, puis **Network Policy Server**.

3. Dans la console NPS, cliquez avec le bouton droit sur **NPS (local)** , puis sélectionnez **Enregistrer un serveur dans Active Directory**. Cliquez deux fois sur **OK**.

    ![Option de menu Enregistrer le serveur dans Active Directory](./media/howto-mfa-nps-extension-vpn/image2.png)

4. Laissez la console ouverte pour la procédure suivante.

### <a name="use-wizard-to-configure-the-radius-server"></a>Utiliser l’Assistant pour configurer le serveur RADIUS

Vous pouvez utiliser un standard (basée sur l’Assistant) ou l’option de configuration avancée pour configurer le serveur RADIUS. Cette section suppose que vous utilisez l’option de configuration standard de l’Assistant.

1. Dans la console NPS, sélectionnez **NPS (local)** .

2. Sous **Configuration standard**, sélectionnez **Serveur RADIUS pour les connexions d’accès à distance ou VPN**, puis **Configurer une connexion VPN ou d’accès à distance**.

    ![Configurer Serveur RADIUS pour les connexions d’accès à distance ou VPN](./media/howto-mfa-nps-extension-vpn/image3.png)

3. Dans la fenêtre **Sélectionner le type de connexions d’accès à distance ou de réseau privé virtuel (VPN)** , sélectionnez **Connexions de réseau privé virtuel**, puis **Suivant**.

    ![Configurer Connexions de réseau privé virtuel (VPN)](./media/howto-mfa-nps-extension-vpn/image4.png)

4. Dans la fenêtre **Spécifier un serveur d’accès à distance ou VPN**, sélectionnez **Ajouter**.

5. Dans la fenêtre **Nouveau client RADIUS**, indiquez un nom convivial, entrez le nom pouvant être résolu ou l’adresse IP du serveur VPN et entrez un mot de passe secret partagé. Créez un mot de passe secret partagé long et complexe. Notez-le, car vous en avez besoin à la section suivante.

    ![Créer une fenêtre Nouveau client RADIUS](./media/howto-mfa-nps-extension-vpn/image5.png)

6. Sélectionnez **OK**, puis **Suivant**.

7. Dans la fenêtre **Configurer les méthodes d’authentification**, acceptez la sélection par défaut (**Authentification chiffrée Microsoft version 2 [MS-CHAPv2]** ) ou choisissez une autre option, puis sélectionnez **Suivant**.

    > [!NOTE]
    > Si vous configurez le protocole EAP (Extensible Authentication Protocol), vous devez utiliser le protocole d’authentification de négociation par défi Microsoft (CHAPv2) ou le protocole PEAP (Protected Extensible Authentication Protocol). Aucun autre EAP n’est pris en charge.

8. Dans la fenêtre **Spécifier des groupes d’utilisateurs**, sélectionnez **Ajouter** et sélectionnez un groupe approprié. Si aucun groupe n’existe, laissez le champ vide pour accorder l’accès à tous les utilisateurs.

    ![Fenêtre Spécifier des groupes d’utilisateurs pour autoriser ou refuser l’accès](./media/howto-mfa-nps-extension-vpn/image7.png)

9. Sélectionnez **Suivant**.

10. Dans la fenêtre **Spécifier des filtres IP**, sélectionnez **Suivant**.

11. Dans la fenêtre **Spécifier les paramètres de chiffrement**, acceptez les paramètres par défaut, puis sélectionnez **Suivant**.

    ![Fenêtre Spécifier les paramètres de chiffrement](./media/howto-mfa-nps-extension-vpn/image8.png)

12. Dans la fenêtre **Spécifier un nom de domaine**, laissez le nom de domaine vide, acceptez le paramètre par défaut et sélectionnez **Suivant**.

    ![Fenêtre Spécifier un nom de domaine](./media/howto-mfa-nps-extension-vpn/image9.png)

13. Dans la fenêtre **Fin de la configuration des nouvelles connexions d’accès à distance ou de réseau privé virtuel (VPN) et des clients RADIUS**, sélectionnez **Terminer**.

    ![Fenêtre de configuration terminée](./media/howto-mfa-nps-extension-vpn/image10.png)

### <a name="verify-the-radius-configuration"></a>Vérifier la configuration RADIUS

Cette section détaille la configuration que vous avez créée en utilisant l’Assistant.

1. Sur le serveur NPS (Network Policy Server), dans la console NPS (locale), développez **Clients RADIUS** et sélectionnez **Clients RADIUS**.

2. Dans le volet de détails, cliquez avec le bouton droit sur le client RADIUS que vous avez créé, puis sélectionnez **Propriétés**. Les propriétés de votre client RADIUS (le serveur VPN) doivent être semblables à celles présentées ici :

    ![Vérifier les propriétés VPN et la configuration](./media/howto-mfa-nps-extension-vpn/image11.png)

3. Sélectionnez **Annuler**.

4. Sur le serveur NPS, dans la console NPS (locale), développez **Stratégies**, puis sélectionnez **Stratégies de demande de connexion**. La stratégie des connexions VPN s’affiche, comme illustré dans l’image suivante :

    ![Stratégies de demande de connexion illustrant la stratégie Connexion VPN](./media/howto-mfa-nps-extension-vpn/image12.png)

5. Sous **Stratégies**, sélectionnez **Stratégies réseau**. Vous devez voir une stratégie Connexions de réseau privé virtuel (VPN) qui ressemble à celle illustrée dans l’image suivante :

    ![Stratégies réseau illustrant la stratégie Connexions de réseau privé virtuel](./media/howto-mfa-nps-extension-vpn/image13.png)

## <a name="configure-your-vpn-server-to-use-radius-authentication"></a>Configurer votre serveur VPN pour utiliser l’authentification RADIUS

Dans cette section, vous configurez votre serveur VPN pour utiliser l’authentification RADIUS. Cette section présuppose que vous disposez d’une configuration de serveur VPN qui fonctionne, mais que vous ne l’avez pas configuré pour utiliser l’authentification RADIUS. Après avoir configuré le serveur VPN, assurez-vous que votre configuration fonctionne comme prévu.

> [!NOTE]
> Si vous disposez déjà d’une configuration de serveur VPN de travail qui utilise l’authentification RADIUS, vous pouvez ignorer cette section.
>

### <a name="configure-authentication-provider"></a>Configurer le fournisseur d’authentification

1. Sur le serveur VPN, ouvrez Gestionnaire de serveurs.

2. Dans le Gestionnaire de serveur, sélectionnez **Outils**, puis **Routage et accès à distance**.

3. Dans la fenêtre **Routage et accès à distance**, cliquez-droit sur **\<server name>(local)** et sélectionnez **Propriétés**.

4. Dans la fenêtre des **\<server name> Propriétés (local)** , sélectionnez l’onglet **Sécurité**.

5. Sous l’onglet **Sécurité**, dans la zone **Fournisseur d’authentification**, sélectionnez **Authentification RADIUS**, puis **Configurer**.

    ![Configurer le fournisseur d’authentification RADIUS](./media/howto-mfa-nps-extension-vpn/image15.png)

6. Dans la fenêtre **Authentification RADIUS**, sélectionnez **Ajouter**.

7. Dans la fenêtre **Ajouter un serveur RADIUS**, effectuez les actions suivantes :

    a. Dans la zone **Nom du serveur**, indiquez le nom ou l’adresse IP du serveur RADIUS que vous avez configuré à la section précédente.

    b. Dans **Secret partagé**, sélectionnez **Modifier** et entrez le mot de passe secret partagé que vous avez créé et noté précédemment.

    c. Dans le champ **Délai (en secondes)** , entrez la valeur **30**.  
    Il est nécessaire de prévoir un délai suffisant pour exécuter le second facteur d’authentification. Il existe des VPN ou régions qui requièrent des paramètres de délai d’expiration supérieurs à 30 secondes pour empêcher les utilisateurs de recevoir plusieurs appels téléphoniques. Si des utilisateurs rencontrent ce problème, augmentez la valeur **Délai d’expiration (secondes)** par incréments de 30 secondes jusqu’à ce que le problème disparaisse.

    ![Fenêtre Ajouter un serveur RADIUS configurant le délai](./media/howto-mfa-nps-extension-vpn/image16.png) 

8. Sélectionnez **OK**.

### <a name="test-vpn-connectivity"></a>Connectivité VPN de test

Dans cette section, assurez-vous que le client VPN est authentifié et autorisé par le serveur RADIUS lorsque vous essayez de vous connecter au port virtuel VPN. Ces instructions présupposent que vous utilisez Windows 10 comme client VPN.

> [!NOTE]
> Si vous avez déjà configuré un client VPN pour vous connecter au serveur VPN et avez enregistré les paramètres, vous pouvez ignorer les étapes relatives à la configuration et à l’enregistrement d’un objet de connexion VPN.
>

1. Sur votre ordinateur client VPN, sélectionnez le bouton **Démarrer**, puis le bouton **Paramètres**.

2. Dans la fenêtre **Paramètres Windows**, sélectionnez **Réseau et Internet**.

3. Sélectionnez **VPN**.

4. Sélectionnez **Ajouter une connexion VPN**.

5. Dans la fenêtre **Ajouter une connexion VPN**, dans la zone **Fournisseur VPN**, sélectionnez **Windows (intégré)** , renseignez les champs restants, comme il convient, et sélectionnez **Enregistrer**.

    ![Fenêtre « Ajouter une connexion VPN »](./media/howto-mfa-nps-extension-vpn/image17.png)

6. Accédez au **Panneau de configuration**, puis sélectionnez **Centre Réseau et partage**.

7. Sélectionnez **Modifier les paramètres de la carte**.

    ![Centre Réseau et partage - Changer les paramètres de la carte](./media/howto-mfa-nps-extension-vpn/image18.png)

8. Cliquez avec le bouton droit sur la connexion réseau VPN, puis sélectionnez **Propriétés**.

9. Dans la fenêtre des propriétés VPN, sélectionnez l’onglet **Sécurité**.

10. Sous l’onglet **Sécurité**, assurez-vous que seul le protocole **Microsoft CHAP Version 2 (MS-CHAP v2)** est sélectionné, puis sélectionnez **OK**.

    ![Option « Autoriser ces protocoles »](./media/howto-mfa-nps-extension-vpn/image20.png)

11. Cliquez avec le bouton droit sur la connexion VPN, puis sélectionnez **Connexion**.

12. Dans la fenêtre **Paramètres**, sélectionnez **Connexion**.  
    Un message de connexion réussie s’affiche dans le journal de sécurité sur le serveur RADIUS en tant qu’ID d’événement 6272, comme illustré ici :

    ![Fenêtre Propriétés de l’événement indiquant une connexion réussie](./media/howto-mfa-nps-extension-vpn/image21.png)

## <a name="troubleshooting-radius"></a>Résolution des problèmes de RADIUS

Supposons que votre configuration VPN fonctionnait avant que vous ayez configuré le serveur VPN pour utiliser un serveur RADIUS centralisé pour l’authentification et l’autorisation. Si la configuration était active, il est probable que le problème soit provoqué par une configuration incorrecte du serveur RADIUS, ou par l’utilisation d’un nom d’utilisateur ou d’un mot de passe non valides. Par exemple, si vous utilisez l’autre suffixe UPN dans le nom d’utilisateur, la tentative de connexion peut échouer. Utilisez le même nom de compte pour de meilleurs résultats.

Pour résoudre ces problèmes, l’idéal est d’examiner les journaux des événements de sécurité sur le serveur RADIUS. Pour gagner du temps dans la recherche des événements, vous pouvez utiliser une vue personnalisée des services de stratégie et d’accès réseau basée sur les rôles dans l’Observateur d’événements, comme illustré ici. La mention « ID d’événement 6273 » indique les événements dans lesquels le serveur NPS a refusé l’accès à un utilisateur.

![Observateur d’événements affichant des événements NPAS](./media/howto-mfa-nps-extension-vpn/image22.png)

## <a name="configure-multi-factor-authentication"></a>Configurer l’authentification multifacteur

Pour obtenir de l’aide sur la configuration des utilisateurs pour Multi-Factor Authentication, consultez les articles [Planifier un déploiement informatique d’Azure AD Multi-Factor Authentication](howto-mfa-getstarted.md#create-conditional-access-policy) et [Configurer mon compte pour la vérification en deux étapes](../user-help/multi-factor-authentication-end-user-first-time.md)

## <a name="install-and-configure-the-nps-extension"></a>Installer et configurer l’extension de serveur NPS

Cette section fournit des instructions permettant de configurer l’extension VPN en vue d’utiliser MFA dans le processus d’authentification client avec le serveur VPN.

> [!NOTE]
> La clé de Registre REQUIRE_USER_MATCH respecte la casse. Toutes les valeurs doivent être affichées en MAJUSCULES.
>

Après avoir installé et configuré l’extension de serveur NPS, toutes les authentifications client RADIUS traitées par ce serveur doivent obligatoirement utiliser MFA. Si vos utilisateurs VPN ne sont pas tous inscrits à Azure AD Multi-Factor Authentication, vous pouvez effectuer l’une des actions suivantes :

* Configurer un autre serveur RADIUS pour authentifier les utilisateurs qui ne sont pas configurés pour utiliser MFA

* Créer une entrée de Registre qui permet aux utilisateurs testés de fournir un second facteur d’authentification s’ils sont inscrits à Azure AD Multi-Factor Authentication

Créez une valeur de chaîne nommée _REQUIRE_USER_MATCH dans HKLM\SOFTWARE\Microsoft\AzureMfa_ et définissez la valeur sur *TRUE* ou *FALSE*.

![Paramètre « Exiger une correspondance d’utilisateur »](./media/howto-mfa-nps-extension-vpn/image34.png)

Si la valeur est définie sur *TRUE*, ou qu’elle n’est pas définie, toutes les demandes d’authentification sont soumises à un défi MFA. Si la valeur est définie sur *FALSE*, les défis MFA sont seulement émis pour les utilisateurs qui sont inscrits à Azure AD Multi-Factor Authentication. Utilisez uniquement le paramètre *FALSE* dans des tests ou des environnements de production pendant une période d’intégration.



### <a name="obtain-the-azure-active-directory-tenant-id"></a>Obtenir l’ID locataire Azure Active Directory

Dans le cadre de la configuration de l’extension de serveur NPS, vous devez fournir les informations d’identification d’administrateur et l’ID de votre locataire Azure AD. Pour obtenir l’ID locataire, suivez ces étapes :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général du client Azure.
1. Dans le menu du portail Azure, sélectionnez **Azure Active Directory** ou recherchez et sélectionnez **Azure Active Directory** dans n’importe quelle page.
1. Dans la page **Vue d’ensemble**, les *Informations du locataire* sont affichées. À côté de l’*ID locataire*, sélectionnez l’icône **Copier**, comme indiqué dans la capture d’écran de l’exemple suivant :

   ![Obtention l’ID locataire depuis le Portail Azure](./media/howto-mfa-nps-extension-vpn/azure-active-directory-tenant-id-portal.png)

### <a name="install-the-nps-extension"></a>Installer l’extension NPS

L’extension de serveur NPS doit être installée sur un serveur doté du rôle Services de stratégie et d’accès réseau installé, et qui fonctionne comme serveur RADIUS dans votre conception. N’installez *pas* l’extension de serveur NPS sur votre serveur VPN.

1. Téléchargez l’extension de serveur NPS depuis le [Centre de téléchargement Microsoft](https://aka.ms/npsmfa).

2. Copiez le fichier exécutable du programme d’installation (*NpsExtnForAzureMfaInstaller.exe*) sur le serveur NPS.

3. Sur le serveur NPS, double-cliquez sur **NpsExtnForAzureMfaInstaller.exe** et, si vous y êtes invité, sélectionnez **Exécuter**.

4. Dans la fenêtre **Installation de l’extension NPS pour Azure AD MFA**, passez en revue les termes du contrat de licence logiciel, cochez la case **J’accepte les conditions générales du contrat de licence**, puis sélectionnez **Installer**.

    ![Fenêtre « Installation de l’extension NPS pour Azure AD MFA »](./media/howto-mfa-nps-extension-vpn/image36.png)

5. Dans la fenêtre **Installation de l’extension NPS pour Azure AD MFA**, sélectionnez **Fermer**.  

    ![Fenêtre de confirmation « Installation réussie »](./media/howto-mfa-nps-extension-vpn/image37.png)

### <a name="configure-certificates-for-use-with-the-nps-extension-by-using-a-powershell-script"></a>Configurer les certificats à utiliser avec l’extension de serveur NPS au moyen d’un script PowerShell

Pour garantir des communications et une assurance sécurisées, configurez les certificats à utiliser par l’extension de serveur NPS. Les composants de serveur NPS incluent un script Windows PowerShell qui configure un certificat auto-signé à utiliser avec le serveur NPS.

Le script effectue les actions suivantes :

* Crée un certificat auto-signé
* Associe la clé publique du certificat au principal du service sur Azure AD
* Stocke le certificat dans le magasin de la machine locale
* Accorde l’accès de la clé privée du certificat à l’utilisateur réseau.
* Redémarre le service NPS

Si vous souhaitez utiliser vos propres certificats, vous devez associer la clé publique de votre certificat au principal de service sur Azure AD, et ainsi de suite.

Pour utiliser le script, fournissez l’extension avec vos informations d’identification d’administration Azure Active Directory et l’ID du locataire Azure Active Directory que vous avez copié précédemment. Le compte doit se trouver dans le même locataire Azure AD que celui pour lequel vous souhaitez activer l’extension. Exécutez le script sur chaque serveur NPS où vous avez installé l’extension du serveur NPS.

1. Exécutez Windows PowerShell en tant qu’administrateur.

2. À l’invite de commandes PowerShell, indiquez **cd "c:\Program Files\Microsoft\AzureMfa\Config"** et sélectionnez Entrée.

3. À l’invite de commande suivante, entrez **.\AzureMfaNpsExtnConfigSetup.ps1** et sélectionnez Entrée. Le script vérifie si le module PowerShell Azure AD est installé. Si ce n’est pas le cas, le script installe le module pour vous.

    ![Exécution du script de configuration AzureMfsNpsExtnConfigSetup.ps1](./media/howto-mfa-nps-extension-vpn/image38.png)

    Si vous recevez une erreur de sécurité due au protocole TLS, activez le protocole TLS 1.2 à l’aide de la commande `[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12` à partir de votre invite PowerShell.
    
    Après la vérification par le script de l’installation du module PowerShell, la fenêtre de connexion du module PowerShell Azure Active Directory s’affiche.

4. Entrez vos informations d’identification d’administrateur Azure AD et votre mot de passe, puis sélectionnez **Connexion**.

    ![S’authentifier auprès d’Azure AD PowerShell](./media/howto-mfa-nps-extension-vpn/image39.png)

5. À l’invite de commandes, collez l’ID de locataire que vous avez copié précédemment et sélectionnez Entrée.

    ![Entrer l’ID Azure AD copié auparavant](./media/howto-mfa-nps-extension-vpn/image40.png)

    Le script crée un certificat auto-signé et effectue d’autres modifications de configuration. Le résultat ressemble à celui de l’image suivante :

    ![Fenêtre PowerShell affichant le certificat auto-signé](./media/howto-mfa-nps-extension-vpn/image41.png)

6. Redémarrez le serveur.

### <a name="verify-the-configuration"></a>Vérifier la configuration

Pour vérifier la configuration, vous devez établir une nouvelle connexion VPN avec le serveur VPN. Après avoir correctement entré vos informations d’identification pour l’authentification principale, la connexion VPN attend que l’authentification secondaire réussisse avant d’établir la connexion, comme montré ci-dessous.

![Fenêtre VPN dans les Paramètres Windows](./media/howto-mfa-nps-extension-vpn/image42.png)

Si vous vous authentifiez correctement avec la méthode de vérification secondaire que vous avez configurée précédemment dans Azure AD MFA, vous êtes connecté à la ressource. En revanche, si l’authentification secondaire ne réussit pas, l’accès à la ressource vous est refusé.

Dans l’exemple suivant, l’application Microsoft Authenticator installée sur un Windows Phone fournit l’authentification secondaire :

![Exemple d’invite MFA sur Windows Phone](./media/howto-mfa-nps-extension-vpn/image43.png)

Après avoir été correctement authentifié par le biais de la méthode secondaire, vous obtenez l’accès au port virtuel sur le serveur VPN. L’utilisation d’une méthode d’authentification secondaire par le biais d’une application mobile sur un appareil approuvé étant obligatoire, le processus de connexion est plus sûr qu’avec la seule combinaison d’un nom d’utilisateur et d’un mot de passe.

### <a name="view-event-viewer-logs-for-successful-sign-in-events"></a>Afficher les journaux d’activité de l’Observateur d’événements pour les événements de connexion réussie

Pour afficher les événements de connexion réussie dans les journaux de l’observateur d’événements Windows, interrogez le journal de sécurité Windows sur le serveur NPS en entrant la commande PowerShell suivante :

```powershell
Get-WinEvent -Logname Security | where {$_.ID -eq '6272'} | FL
```

![PowerShell pour l’Observateur d’événements de sécurité](./media/howto-mfa-nps-extension-vpn/image44.png)

Vous pouvez également afficher le journal de sécurité ou la vue personnalisée de Services de stratégie et d’accès réseau, comme illustré ici :

![Exemple de journal de serveur NPS](./media/howto-mfa-nps-extension-vpn/image45.png)

Sur le serveur où vous avez installé l’extension NPS pour Azure AD Multi-Factor Authentication, vous pouvez rechercher les journaux des applications de l’observateur d’événements qui sont propres à l’extension à l’emplacement *Journaux des applications et des services\Microsoft\AzureMfa*.

```powershell
Get-WinEvent -Logname Security | where {$_.ID -eq '6272'} | FL
```

![Exemple de volet de journaux AuthZ de l’observateur d’événements](./media/howto-mfa-nps-extension-vpn/image46.png)

## <a name="troubleshooting-guide"></a>Guide de résolution des problèmes

Si la configuration ne fonctionne pas comme prévu, la première chose à faire pour résoudre les problèmes consiste à vérifier que l’utilisateur est bien configuré pour utiliser MFA. Connectez l’utilisateur au [portail Azure](https://portal.azure.com). Si l’utilisateur est invité à utiliser une authentification secondaire et qu’il peut correctement s’authentifier, vous pouvez éliminer les problèmes de configuration incorrecte de MFA.

Si l’authentification MFA fonctionne pour l’utilisateur, consultez les journaux d’activité de l’Observateur d’événements pertinents. Citons notamment les journaux des événements de sécurité, des opérations de la passerelle et d’Azure AD Multi-Factor Authentication qui sont décrits dans la section précédente.

Voici un exemple de journal de sécurité qui affiche un événement d’échec de connexion (ID d’événement 6273) :

![Journal de sécurité montrant un événement de connexion ayant échoué](./media/howto-mfa-nps-extension-vpn/image47.png)

Un événement connexe du journal Azure AD Multi-Factor Authentication est illustré ici :

![Journaux Azure AD Multi-Factor Authentication](./media/howto-mfa-nps-extension-vpn/image48.png)

Pour résoudre les problèmes de manière plus poussée, consultez les fichiers journaux au format de base de données NPS où est installé le service NPS. Les fichiers journaux sont créés dans le dossier _%SystemRoot%\System32\Logs_ sous forme de fichiers texte délimité par des virgules. Pour obtenir une description des fichiers journaux, consultez [Interpréter des fichiers journaux au format de base de données NPS](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc771748(v=ws.10)).

Les entrées de ces fichiers journaux sont difficilement interprétables à moins de les exporter dans une feuille de calcul ou une base de données. De nombreux outils d’analyse de Service authentification Internet (IAS) sont à votre disposition en ligne pour vous aider à interpréter les fichiers journaux. Voici illustré le résultat d’une de ces [applications de logiciel à contribution volontaire](https://www.deepsoftware.com/iasviewer) téléchargeable :

![Exemple d’analyseur IAS d’application de logiciel à contribution volontaire](./media/howto-mfa-nps-extension-vpn/image49.png)

Pour résoudre des problèmes supplémentaires, vous pouvez utiliser un analyseur de protocoles, tel que Wireshark ou [Microsoft Message Analyzer](/message-analyzer/microsoft-message-analyzer-operating-guide). L’image suivante, à partir de Wireshark, affiche les messages RADIUS entre le serveur VPN et NPS.

![Microsoft Message Analyzer affichant le trafic filtré](./media/howto-mfa-nps-extension-vpn/image50.png)

Pour plus d’informations, consultez [Intégrer votre infrastructure NPS à Azure AD Multi-Factor Authentication](howto-mfa-nps-extension.md).

## <a name="next-steps"></a>Étapes suivantes

[Obtenir Azure AD Multi-Factor Authentication](concept-mfa-licensing.md)

[Passerelle des services Bureau à distance et serveur Multi-Factor Authentication avec RADIUS](howto-mfaserver-nps-rdg.md)

[Intégrer vos répertoires locaux à Azure Active Directory](../hybrid/whatis-hybrid-identity.md)