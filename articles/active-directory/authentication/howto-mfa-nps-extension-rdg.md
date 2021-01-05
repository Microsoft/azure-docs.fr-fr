---
title: Intégrer la passerelle des services Bureau à distance à l'aide de l'extension NPS d'Azure AD MFA - Azure Active Directory
description: Intégrez la passerelle des services Bureau à distance à Azure AD MFA à l'aide de l'extension NPS (Network Policy Server) pour Microsoft Azure
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
ms.openlocfilehash: 20b0150c18f2c007ed104d34daacd49ab03131a7
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743375"
---
# <a name="integrate-your-remote-desktop-gateway-infrastructure-using-the-network-policy-server-nps-extension-and-azure-ad"></a>Intégrez votre infrastructure de passerelle des services Bureau à distance à l’aide de l’extension du serveur NPS (Network Policy Server) et Azure AD

Cet article explique comment intégrer la passerelle des services Bureau à distance à Azure AD Multi-Factor Authentication (MFA) à l'aide de l'extension NPS (Network Policy Server) pour Microsoft Azure.

L’extension de serveur NPS (Network Policy Server) pour Azure permet aux clients de protéger le protocole d’authentification client RADIUS (Remote Authentication Dial-In User Service) à l’aide de l’authentification Azure [Multi-Factor Authentication (MFA)](./concept-mfa-howitworks.md) basée sur le cloud. Cette solution fournit une vérification en deux étapes pour ajouter une deuxième couche de sécurité aux connexions et transactions utilisateur.

Cet article fournit des instructions pas à pas pour intégrer l'infrastructure NPS à Azure AD MFA à l'aide de l'extension NPS pour Azure. Cela permet une vérification pour les utilisateurs tentant de se connecter à une passerelle des services Bureau à distance.

> [!NOTE]
> Cet article ne doit pas être utilisé pour les déploiements Serveur MFA ; il s'applique uniquement aux déploiements Azure AD MFA (basés sur le cloud).

Les services de stratégie et d’accès réseau (NPS) permettent aux entreprises d’effectuer les opérations suivantes :

* définir des emplacements centraux pour la gestion et le contrôle des demandes du réseau en spécifiant qui peut se connecter, les heures de connexion autorisées pendant la journée, la durée des connexions et le niveau de sécurité que les clients doivent utiliser pour se connecter, et autres. Plutôt que de spécifier ces stratégies sur chaque serveur VPN ou de passerelle des services Bureau à distance (RD), ces stratégies peuvent être spécifiées une seule fois dans un emplacement central. Le protocole RADIUS fournit l’authentification, l’autorisation et la gestion des comptes (AAA) centralisées.
* Établissez et appliquez les stratégies de contrôle d’intégrité client de Protection d’accès réseau (NAP) qui déterminent si les périphériques sont accordés avec ou sans restrictions d’accès aux ressources réseau.
* Fournissez un moyen d’appliquer l’authentification et l’autorisation d’accès aux points d’accès sans fil et commutateurs compatibles à 802.1x.

En règle générale, les organisations utilisent le serveur NPS (RADIUS) pour simplifier et centraliser la gestion des stratégies de serveur VPN. Toutefois, de nombreuses organisations utilisent également NPS pour simplifier et centraliser la gestion des stratégies d’autorisation des connexions aux services Bureau à distance (RD CAP).

Les organisations peuvent également intégrer NPS à Azure AD MFA pour améliorer la sécurité et fournir un niveau élevé de compatibilité. Cela permet de s’assurer que les utilisateurs établissent la vérification en deux étapes pour se connecter à la passerelle des services Bureau à distance. Pour que les utilisateurs puissent obtenir l’accès, ils doivent fournir leur combinaison de nom d’utilisateur et de mot de passe ainsi que des informations sur lesquelles ils ont le contrôle. Ces informations doivent être approuvées et pas facilement dupliquées, comme un numéro de téléphone portable, numéro de téléphone fixe, une application sur un appareil mobile et autres. La passerelle des services Bureau à distance prend actuellement en charge les appels téléphoniques et les notifications Push à partir des méthodes d’application Microsoft Authenticator pour l’authentification à 2 facteurs. Pour plus d’informations sur les méthodes d’authentification prises en charge, consultez la section [Déterminer les méthodes d’authentification que vos utilisateurs peuvent employer](howto-mfa-nps-extension.md#determine-which-authentication-methods-your-users-can-use).

Avant le lancement de l'extension NPS pour Azure, les clients qui souhaitaient implémenter la vérification en deux étapes pour les environnements intégrant NPS et Azure AD MFA devaient configurer et gérer un serveur MFA distinct dans l'environnement local, comme décrit dans [Passerelle des services Bureau à distance et serveur Microsoft Azure Multi-Factor Authentication utilisant RADIUS](howto-mfaserver-nps-rdg.md).

La disponibilité de l’extension de serveur NPS pour Azure permet désormais aux entreprises de choisir de déployer une solution d’authentification multifacteur locale ou une solution de l’authentification multifacteur basée sur le cloud pour l’authentification client RADIUS sécurisée.

## <a name="authentication-flow"></a>Flux d'authentification

Pour que les utilisateurs puissent obtenir l’accès aux ressources réseau via une passerelle des services Bureau à distance, il leur faut remplir les conditions spécifiées dans une stratégie d'autorisation des connexions aux services Bureau à distance (RD CAP) et une stratégie d'autorisation d'accès aux ressources via les services Bureau à distance (RD RAP). Les stratégies RD CAP spécifient qui est autorisé à se connecter à des passerelles des services Bureau à distance. Les stratégies RD CAP indiquent les ressources réseau, notamment les bureaux à distance ou les applications à distance, auxquelles l’utilisateur est autorisé à se connecter via la passerelle des services Bureau à distance.

Une passerelle des services Bureau à distance peut être configurée pour utiliser un Store de stratégies central pour les stratégies RD CAP. Les stratégies de bureau à distance ne peuvent pas utiliser une stratégie centrale, car elles sont traitées sur la passerelle des services Bureau à distance. Un exemple de passerelle des services Bureau à distance configuré pour utiliser un Store de stratégies central pour les stratégies RD CAP Bureau à distance est les suivant : un client RADIUS vers un autre serveur NPS qui sert de Store de stratégies centrales.

Lorsque l’extension de serveur NPS pour Azure est intégrée au serveur NPS et à la passerelle des services Bureau à distance, le flux d’authentification réussie est le suivant :

1. Le serveur de passerelle des services Bureau à distance reçoit une demande d’authentification à partir d’un utilisateur du bureau à distance pour se connecter à une ressource, par exemple une session Bureau à distance. Agissant comme un client RADIUS, le serveur de passerelle des services Bureau à distance convertit la demande en un message de demande d’accès RADIUS et envoie le message sur le serveur RADIUS (NPS) où est installée l’extension du serveur NPS.
1. La combinaison nom d’utilisateur et mot de passe est vérifiée dans Active Directory et l’utilisateur est authentifié.
1. Si toutes les conditions, comme spécifié dans la demande de connexion du serveur NPS, et toutes les stratégies réseau sont satisfaites (par exemple, restrictions liées à l'heure ou à l'appartenance à un groupe), l'extension NPS déclenche une demande d'authentification secondaire auprès d'Azure AD MFA.
1. Azure AD MFA communique avec Azure AD pour récupérer les informations de l'utilisateur et procède à l'authentification secondaire à l'aide d'une des méthodes prises en charge.
1. Une fois le défi MFA réussi, Azure AD MFA communique le résultat à l'extension NPS.
1. Le serveur NPS, où est installée l’extension, envoie un message RADIUS d’acceptation d’accès pour la stratégie d’autorisation des connexions aux services Bureau à distance sur le serveur de passerelle Bureau à distance.
1. L’utilisateur a accès à la ressource réseau demandée via la passerelle des services Bureau à distance.

## <a name="prerequisites"></a>Prérequis

Cette section détaille les conditions préalables qui doivent être réunies avant d'intégrer Azure AD MFA à la passerelle des services Bureau à distance. Avant de commencer, vous devez disposer des conditions requises en place suivantes.  

* Infrastructure des Services Bureau à distance (RDS)
* Licence Azure AD MFA
* Logiciel Windows Server
* Rôle des services de stratégie et d’accès réseau (NPS)
* Azure Active Directory synchronisé avec Active Directory local
* ID du GUID Azure Active Directory

### <a name="remote-desktop-services-rds-infrastructure"></a>Infrastructure des Services Bureau à distance (RDS)

Vous devez disposer d’une infrastructure de Services Bureau à distance (RDS) de travail en place. Dans le cas contraire, vous pouvez rapidement créer cette infrastructure dans Azure en utilisant le modèle de démarrage rapide suivant : [Créer un déploiement de collections de sessions de Bureau à distance](https://github.com/Azure/azure-quickstart-templates/tree/ad20c78b36d8e1246f96bb0e7a8741db481f957f/rds-deployment).

Si vous souhaitez créer manuellement et rapidement une infrastructure RDS locale pour des tests, suivez les étapes pour déployer une.
**En savoir plus** : [Déployer des services RDS avec le démarrage rapide Azure](/windows-server/remote/remote-desktop-services/rds-in-azure) et [Déploiement de l’infrastructure RDS de base](/windows-server/remote/remote-desktop-services/rds-deploy-infrastructure).

### <a name="azure-ad-mfa-license"></a>Licence Azure AD MFA

Une licence Azure AD MFA est requise. Vous pouvez vous la procurer via Azure AD Premium ou d'autres offres groupées qui l'incluent. Les licences Azure AD MFA basées sur la consommation, telles que les licences par utilisateur ou par authentification, ne sont pas compatibles avec l'extension NPS. Pour plus d'informations, consultez [Guide pratique pour obtenir Azure AD Multi-Factor Authentication](concept-mfa-licensing.md). À des fins de test, vous pouvez utiliser un abonnement d’évaluation.

### <a name="windows-server-software"></a>Logiciel Windows Server

L’extension de serveur NPS requiert Windows Server 2008 R2 SP1 ou version ultérieure avec le service de rôle NPS installé. Toutes les étapes de cette section ont été effectuées à l’aide de Windows Server 2016.

### <a name="network-policy-and-access-services-nps-role"></a>Rôle des services de stratégie et d’accès réseau (NPS)

Le service de rôle NPS fournit la fonctionnalité serveur et client RADIUS ainsi que le service de contrôle d’intégrité de la Stratégie d’accès réseau. Ce rôle doit être installé sur au moins deux ordinateurs de votre infrastructure : La passerelle des services Bureau à distance et un autre serveur membre ou contrôleur de domaine. Par défaut, le rôle est déjà présent sur l’ordinateur configuré en tant que passerelle des services Bureau à distance.  Vous devez également installer le rôle NPS sur au moins un autre ordinateur, tel qu’un contrôleur de domaine ou un serveur membre.

Pour plus d’informations sur l’installation du service de rôle de serveur NPS Windows Server 2012 ou versions plus ancienne, consultez [Installer un serveur de stratégie de contrôle d’intégrité NAP](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd296890(v=ws.10)). Pour obtenir une description des meilleures pratiques pour le serveur NPS, y compris la recommandation pour installer le serveur NPS sur un contrôleur de domaine, consultez [Meilleures pratiques pour le serveur NPS](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc771746(v=ws.10)).

### <a name="azure-active-directory-synched-with-on-premises-active-directory"></a>Azure Active Directory synchronisé avec Active Directory local

Pour utiliser l’extension de serveur NPS, les utilisateurs locaux doivent être synchronisés avec Azure AD et activés pour l’authentification multifacteur. Cette section part du principe que les utilisateurs locaux sont synchronisés avec Azure AD en utilisant AD Connect. Pour obtenir plus d’informations sur Azure AD Connect, consultez [Intégrer vos répertoires locaux avec Azure Active Directory](../hybrid/whatis-hybrid-identity.md).

### <a name="azure-active-directory-guid-id"></a>ID du GUID Azure Active Directory

Pour installer l’extension de serveur NPS, vous devez connaître le GUID d’Azure AD. Vous trouverez ci-dessous des instructions pour rechercher le GUID d’Azure AD.

## <a name="configure-multi-factor-authentication"></a>Configurer l’authentification multifacteur

Cette section fournit des instructions pour intégrer Azure AD MFA à la passerelle des services Bureau à distance. En tant qu'administrateur, vous devez configurer le service Azure AD MFA avant que les utilisateurs puissent enregistrer eux-mêmes leurs appareils ou applications multifacteurs.

Suivez les étapes décrites dans [Bien démarrer avec Azure AD Multi-Factor Authentication dans le cloud](howto-mfa-getstarted.md) afin d'activer MFA pour vos utilisateurs Azure AD.

### <a name="configure-accounts-for-two-step-verification"></a>Configurer des comptes pour la vérification en deux étapes

Lorsqu’un compte a été activé pour MFA, vous ne pouvez pas vous connecter aux ressources régies par la stratégie MFA tant que vous n’avez pas correctement configuré un appareil approuvé afin qu’il serve pour le second facteur d’authentification, et qu’il soit authentifié à l’aide de la vérification en deux étapes.

Suivez les étapes décrites dans [Présentation concrète d'Azure AD Multi-Factor Authentication](../user-help/multi-factor-authentication-end-user-first-time.md) pour en savoir plus sur la configuration de vos appareils pour l'authentification multifacteur avec votre compte d'utilisateur.

> [!IMPORTANT]
> Le comportement de connexion de la passerelle des services Bureau à distance ne permet pas d'entrer un code de vérification avec Azure AD Multi-Factor Authentication. Un compte d’utilisateur doit être configuré pour la vérification par téléphone ou l’application Microsoft Authenticator avec des notifications Push.
>
> Si l'une de ces deux méthodes d'authentification n'est pas configurée pour un utilisateur, celui-ci ne pourra pas relever le défi de l'authentification Azure AD MFA et se connecter à la passerelle des services Bureau à distance.

## <a name="install-and-configure-nps-extension"></a>Installer et configurer l’extension NPS

Cette section fournit des instructions pour configurer l'infrastructure des services Bureau à distance afin d'utiliser Azure AD MFA pour l'authentification des clients avec la passerelle des services Bureau à distance.

### <a name="acquire-azure-active-directory-tenant-id"></a>Acquérir l’ID locataire Azure Active Directory

Dans le cadre de la configuration de l’extension de serveur NPS, vous devez fournir les informations d’identification d’administrateur et l’ID Azure AD pour votre client Azure AD. Pour obtenir l’ID locataire, suivez ces étapes :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général du client Azure.
1. Dans le menu du portail Azure, sélectionnez **Azure Active Directory** ou recherchez et sélectionnez **Azure Active Directory** dans n’importe quelle page.
1. Dans la page **Vue d’ensemble**, les *Informations du locataire* sont affichées. À côté de l’*ID locataire*, sélectionnez l’icône **Copier**, comme indiqué dans la capture d’écran de l’exemple suivant :

   ![Obtention l’ID locataire depuis le Portail Azure](./media/howto-mfa-nps-extension-rdg/azure-active-directory-tenant-id-portal.png)

### <a name="install-the-nps-extension"></a>Installer l’extension NPS

Installez l’extension de serveur NPS sur un serveur ayant le rôle des services de stratégie réseau et d’accès réseau (NPS) est installé. Cela fonctionne comme serveur RADIUS pour votre conception.

> [!IMPORTANT]
> N’installez pas l’extension de serveur NPS sur votre serveur de passerelle bureau distant (RDG). Le serveur RDG n’utilise pas le protocole RADIUS avec son client, de sorte que l’extension ne peut pas interpréter et effectuer l’authentification MFA.
>
> Lorsque le serveur RDG et le serveur NPS avec l’extension NPS sont des serveurs différents, RDG utilise NPS en interne pour communiquer avec d’autres serveurs NPS et utilise RADIUS comme protocole pour communiquer correctement.

1. Téléchargez [l’extension de serveur NPS](https://aka.ms/npsmfa).
1. Copiez le fichier exécutable du programme d’installation (NpsExtnForAzureMfaInstaller.exe) sur le serveur NPS.
1. Sur le serveur NPS, double-cliquez sur **NpsExtnForAzureMfaInstaller.exe**. À l’invite, cliquez sur **Exécuter**.
1. Dans la boîte de dialogue Installation de l'extension NPS d'Azure AD MFA, passez en revue les termes du contrat de licence logiciel, cochez la case **J'accepte les termes et les conditions du contrat de licence** et cliquez sur **Installer**.
1. Dans la boîte de dialogue Installation de l'extension NPS d'Azure AD MFA, cliquez sur **Fermer**.

### <a name="configure-certificates-for-use-with-the-nps-extension-using-a-powershell-script"></a>Configurer des certificats pour une utilisation avec l’extension de serveur NPS à l’aide d’un script PowerShell

Ensuite, vous devez configurer des certificats pour une utilisation par l’extension de serveur NPS pour garantir des communications et une assurance sécurisées. Les composants de serveur NPS incluent un script Windows PowerShell qui configure un certificat auto-signé à utiliser avec le serveur NPS.

Le script effectue les actions suivantes :

* crée un certificat auto-signé
* associe la clé publique du certificat au principal du service sur Azure AD
* stocke le certificat dans le Store de l’ordinateur local
* accorde l’accès à la clé privée du certificat à l’utilisateur réseau
* redémarre le service de serveur de stratégie réseau

Si vous souhaitez utiliser vos propres certificats, vous devez associer la clé publique de votre certificat au principal de service sur Azure AD, et ainsi de suite.

Pour utiliser le script, spécifiez l’extension avec vos informations d’identification administrateur Azure AD l’ID du client Azure AD que vous avez copiée précédemment. Exécutez le script sur chaque serveur NPS où vous avez installé l’extension du serveur NPS. Faites ensuite ce qui suit :

1. Ouvrez une invite administrative Windows PowerShell.
1. À l’invite PowerShell, tapez `cd 'c:\Program Files\Microsoft\AzureMfa\Config'`, puis appuyez sur **Entrée**.
1. Tapez `.\AzureMfaNpsExtnConfigSetup.ps1`, puis appuyez sur **Entrée**. Le script vérifie si le module Azure Active Directory PowerShell est installé. Si ce n’est pas le cas, le script installe le module pour vous.

   ![Exécution de AzureMfaNpsExtnConfigSetup.ps1 dans Azure AD PowerShell](./media/howto-mfa-nps-extension-rdg/image4.png)
  
1. Une fois la vérification de l’installation du module PowerShell par le script, il affiche la boîte de dialogue du module Azure Active Directory PowerShell. Dans la boîte de dialogue, entrez vos informations d’identification d’administrateur Azure AD et un mot de passe, puis cliquez sur **Connexion**.

   ![Authentification auprès d’Azure AD dans PowerShell](./media/howto-mfa-nps-extension-rdg/image5.png)

1. Lorsque vous y êtes invité, collez l’*ID locataire* que vous avez copié précédemment dans le presse-papiers, puis appuyez sur **ENTRÉE**.

   ![Saisie de l’ID locataire dans PowerShell](./media/howto-mfa-nps-extension-rdg/image6.png)

1. Le script crée un certificat auto-signé et effectue d’autres modifications de configuration. La sortie doit être comme l’image ci-dessous.

   ![Fenêtre PowerShell affichant le certificat auto-signé](./media/howto-mfa-nps-extension-rdg/image7.png)

## <a name="configure-nps-components-on-remote-desktop-gateway"></a>Configurer les composants de serveur NPS sur la passerelle des services Bureau à distance

Dans cette section, configurez des stratégies d’autorisation des connexions de passerelle des services Bureau à distance et d’autres paramètres de RADIUS.

Le flux d’authentification nécessite que les messages RADIUS soient échangés entre la passerelle des services Bureau à distance et le serveur NPS où est installée l’extension NPS. Cela signifie que vous devez configurer les paramètres des clients RADIUS sur la passerelle des services Bureau à distance et le serveur NPS où est installée l’extension du serveur NPS.

### <a name="configure-remote-desktop-gateway-connection-authorization-policies-to-use-central-store"></a>Configurer des stratégies de l’autorisation des connexions de passerelle des services Bureau à distance pour utiliser le Store central

Les stratégies d’autorisation des connexions aux services Bureau à distance (RD CAP) spécifient les exigences techniques de connexion à un serveur de passerelle des services Bureau à distance. Les stratégies RD CAP peuvent être stockées localement (par défaut) ou être stockées dans un Store de stratégies RD CAP central qui exécute un serveur NPS. Pour configurer l'intégration d'Azure AD MFA aux services Bureau à distance, vous devez spécifier l'utilisation d'un magasin central.

1. Sur le serveur de passerelle des services Bureau à distance, ouvrez **Gestionnaire de serveurs**.
1. Dans le menu, cliquez sur **Outils**, pointez vers **Services bureau à distance** puis cliquez sur **Gestionnaire de passerelle de Bureau à distance**.
1. Dans le Gestionnaire de passerelle Bureau à distance, cliquez avec le bouton droit sur **\[Nom de serveur\] (Local)** et cliquez sur **Propriétés**.
1. Dans la boîte de dialogue Propriétés, sélectionnez l’onglet **Magasin de stratégies d’autorisation des connexions aux services Bureau à distance**.
1. Sous l’onglet Store RD CAP, sélectionnez **Serveur central exécutant NPS**. 
1. Dans le champ **Entrer une adresse IP ou un nom du serveur exécutant NPS**, tapez le nom de serveur ou l’adresse IP du serveur où vous avez installé l’extension du serveur NPS.

   ![Entrer le nom ou l’adresse IP de votre serveur NPS](./media/howto-mfa-nps-extension-rdg/image10.png)
  
1. Cliquez sur **Add**.
1. Dans la boîte de dialogue **Secret partagé**, entrez un secret partagé, puis cliquez sur **OK**. Veillez à enregistrer ce secret partagé et à stocker l’enregistrement en toute sécurité.

   >[!NOTE]
   >Le secret partagé est utilisé pour établir l’approbation entre les serveurs RADIUS et les clients. Créez une clé secrète longue et complexe.
   >

   ![Création d’un secret partagé pour établir une approbation](./media/howto-mfa-nps-extension-rdg/image11.png)

1. Cliquez sur **OK** pour fermer la boîte de dialogue.

### <a name="configure-radius-timeout-value-on-remote-desktop-gateway-nps"></a>Configurer la valeur de délai d’expiration RADIUS sur le serveur NPS de passerelle Bureau à distance

Pour vérifier qu’il y a suffisamment de temps pour valider les informations d’identification des utilisateurs, effectuez une vérification en deux étapes, recevez des réponses et répondez aux messages RADIUS, il est nécessaire d’ajuster la valeur de délai d’expiration RADIUS.

1. Sur le serveur de passerelle des services Bureau à distance, ouvrez le Gestionnaire de serveur. Dans le menu, cliquez sur **Outils**, puis sur **Serveur NPS (Network Policy Server)** .
1. Dans la console **NPS (Local)** , développez **Clients et serveurs RADIUS**, puis sélectionnez **Serveur RADIUS à distance**.

   ![Console de gestion du serveur NPS (Network Policy Server) avec le serveur RADIUS distant](./media/howto-mfa-nps-extension-rdg/image12.png)

1. Dans le volet d’informations, double-cliquez sur **GROUPE DE SERVEUR DE PASSERELLE TS**.

   >[!NOTE]
   >Ce groupe de serveurs RADIUS a été créé lorsque vous avez configuré le serveur central pour les stratégies du serveur NPS. La passerelle des services Bureau à distance transfère les messages RADIUS vers ce serveur ou un groupe de serveurs, s’il en existe plusieurs dans le groupe.
   >

1. Dans la boîte de dialogue **Propriétés du GROUPE DE SERVEURS DE PASSERELLE TS**, sélectionnez l’adresse IP ou le nom du serveur NPS configuré pour stocker les stratégies RD CAP, puis cliquez sur **Modifier**.

   ![Sélectionnez l’adresse IP ou le nom du serveur NPS configuré précédemment](./media/howto-mfa-nps-extension-rdg/image13.png)

1. Dans la boîte de dialogue **Modifier le serveur RADIUS**, sélectionnez l’onglet **Équilibrage de charge**.
1. Dans l’onglet **Équilibrage de charge** dans le champ **Nombre de secondes sans réponse avant que la demande ne soit considérée comme supprimée**, modifiez la valeur par défaut de 3 à une valeur comprise entre 30 et 60 secondes.
1. Dans le champ **Nombre de secondes entre les demandes lorsque le serveur est identifié comme non disponible**, modifiez la valeur par défaut de 30 secondes à une valeur qui est égale ou supérieure à la valeur que vous avez spécifié à l’étape précédente.

   ![Modifier les paramètres de délai d’attente du serveur Radius sur l’onglet d’équilibrage de charge](./media/howto-mfa-nps-extension-rdg/image14.png)

1. Cliquez deux fois sur **OK** pour fermer les boîtes de dialogue.

### <a name="verify-connection-request-policies"></a>Vérifiez les stratégies de demande de connexion

Par défaut, lorsque vous configurez la passerelle des services Bureau à distance pour utiliser un Store de stratégies central pour les stratégies d’autorisation de connexion, la passerelle des services Bureau à distance est configurée pour transmettre les demandes CAP au serveur NPS. Le serveur NPS sur lequel l'extension Azure AD MFA est installée traite la demande d'accès RADIUS. Les étapes suivantes vous montrent comment vérifier la stratégie de demande de connexion par défaut.  

1. Sur la passerelle des services Bureau à distance, dans la console NPS (Local), développez **Stratégies**, puis sélectionnez **Stratégies de demande de connexion**.
1. Double-cliquez sur **STRATÉGIE D’AUTORISATION DE PASSERELLE TS**.
1. Dans la boîte de dialogue **Propriétés de la STRATÉGIE D’AUTORISATION DE PASSERELLE TS**, cliquez sur l’onglet **Paramètres**.
1. Sur l’onglet **Paramètres**, sous Transfert de la demande de connexion, cliquez sur **Authentification**. Le client RADIUS est configuré pour transférer les demandes pour l’authentification.

   ![Configurer les paramètres d’authentification en spécifiant le groupe de serveurs](./media/howto-mfa-nps-extension-rdg/image15.png)

1. Cliquez sur **Annuler**.

>[!NOTE]
> Pour plus d’informations sur la création d’une stratégie de demande de connexion, consultez l’article [Configurer des stratégies de demande de connexion](/windows-server/networking/technologies/nps/nps-crp-configure#add-a-connection-request-policy) dans la documentation. 

## <a name="configure-nps-on-the-server-where-the-nps-extension-is-installed"></a>Configurer le serveur NPS sur le serveur où est installée l’extension NPS

Le serveur NPS où est installée l’extension de serveur NPS doit être en mesure d’échanger des messages RADIUS avec le serveur NPS sur la passerelle des services Bureau à distance. Pour activer l’échange de messages, vous devez configurer les composants de serveur NPS sur le serveur où est installé le service d’extension du serveur NPS.

### <a name="register-server-in-active-directory"></a>Enregistrer le serveur dans Active Directory

Pour fonctionner correctement dans ce scénario, le serveur NPS doit être enregistré dans Active Directory.

1. Sur le serveur NPS, ouvrez le **Gestionnaire de serveur**.
1. Dans Gestionnaire de serveurs, cliquez sur **Outils** puis cliquez sur **Serveur de stratégie réseau**.
1. Dans la console NPS, cliquez avec le bouton droit sur **NPS (Local)** , puis cliquez sur **Enregistrer un serveur dans Active Directory**.
1. Cliquez deux fois sur **OK**.

   ![Enregistrer le serveur NPS dans Active Directory](./media/howto-mfa-nps-extension-rdg/image16.png)

1. Laissez la console ouverte pour la procédure suivante.

### <a name="create-and-configure-radius-client"></a>Créer et configurer un client RADIUS

La passerelle des services Bureau à distance doit être configuré comme client RADIUS sur le serveur NPS.

1. Sur le serveur NPS où l’extension de serveur NPS est installée, dans la console **NPS (Local)** de la console, cliquez avec le bouton droit sur **Clients RADIUS** et cliquez sur **Nouveau**.

   ![Créer un client RADIUS dans la console NPS](./media/howto-mfa-nps-extension-rdg/image17.png)

1. Dans la boîte de dialogue **Nouveau client RADIUS**, fournissez un nom convivial, tel que _Passerelle_, et l’adresse IP ou le nom DNS du serveur de passerelle Bureau à distance.
1. Dans les champs **Secret partagé** et **Confirmer le secret partagé**, entrez le nom secret utilisé précédemment.

   ![Configurer un nom convivial et l’adresse IP ou DNS](./media/howto-mfa-nps-extension-rdg/image18.png)

1. Cliquez sur **OK** pour fermer la boîte de dialogue du nouveau client RADIUS.

### <a name="configure-network-policy"></a>Configurer la stratégie réseau

Rappelez-vous que le serveur NPS doté de l'extension Azure AD MFA est le magasin de stratégies central désigné pour la stratégie d'autorisation des connexions. Par conséquent, vous devez mettre en œuvre une stratégie d’autorisation de connexion sur le serveur NPS pour autoriser les demandes de connexions valides.  

1. Sur le serveur NPS, dans la console NPS (Local), développez **Stratégies**, puis cliquez sur **Stratégies réseau**.
1. Cliquez avec le bouton droit sur **Connexions aux autres serveurs d’accès** et cliquez sur **Dupliquer la stratégie**.

   ![Dupliquer la connexion à une autre stratégie de serveurs d’accès](./media/howto-mfa-nps-extension-rdg/image19.png)

1. Cliquez avec le bouton droit sur **Copie des connexions aux autres serveurs d’accès** et cliquez sur **Propriétés**.
1. Dans la boîte de dialogue **Copie des connexions à d’autres serveurs d’accès**, dans **Nom de la stratégie**, entrez un nom approprié, tel que _RDG_CAP_. Cochez la case **Stratégie activée** et sélectionnez **Accorder l’accès**. Le cas échéant, dans **Type de serveur d’accès réseau**, sélectionnez **Passerelle des services Bureau à distance**, sinon laissez ce champ **Non spécifié**.

   ![Nom de la stratégie, activer et accorder l’accès](./media/howto-mfa-nps-extension-rdg/image21.png)

1. Cliquez sur l’onglet **Contraintes** et cochez la case **Autoriser les clients à se connecter sans négocier une méthode d’authentification**.

   ![Modifier les méthodes d’authentification pour autoriser les clients à se connecter](./media/howto-mfa-nps-extension-rdg/image22.png)

1. Si vous le souhaitez, cliquez sur l’onglet **Conditions** et ajoutez des conditions qui doivent être remplies pour la connexion autorisée, par exemple, l’appartenance à un groupe Windows spécifique.

   ![Spécifier si besoin des conditions de connexion](./media/howto-mfa-nps-extension-rdg/image23.png)

1. Cliquez sur **OK**. Lorsque vous êtes invité à consulter la rubrique d’aide correspondante, cliquez sur **Non**.
1. Assurez-vous que votre nouvelle stratégie s’affiche en haut de la liste, que la stratégie est activée et qu’elle accorde un accès.

   ![Faire remonter votre stratégie vers le haut de la liste](./media/howto-mfa-nps-extension-rdg/image24.png)

## <a name="verify-configuration"></a>Vérifier la configuration

Pour vérifier la configuration, vous devez vous connecter à la passerelle des services Bureau à distance avec un client RDP approprié. Veillez à utiliser un compte autorisé par les stratégies d'autorisation des connexions et sur lequel Azure AD MFA est activé.

Comme indiqué dans l’image ci-dessous, vous pouvez utiliser la page **Accès Web au Bureau à distance**.

![Test de l’accès web au Bureau à distance](./media/howto-mfa-nps-extension-rdg/image25.png)

Lors de l’entrée avec succès de vos informations d’identification pour l’authentification principale, la boîte de dialogue Connexion au bureau à distance présente l’état de l’initialisation de la connexion à distance, comme indiqué ci-dessous. 

Si vous vous authentifiez correctement avec la méthode d'authentification secondaire que vous avez configurée précédemment dans Azure AD MFA, vous êtes connecté à la ressource. En revanche, si l’authentification secondaire ne réussit pas, l’accès à la ressource vous est refusé. 

![Connexion Bureau à distance établissant une connexion à distance](./media/howto-mfa-nps-extension-rdg/image26.png)

Dans l’exemple ci-dessous, l’application d’authentification sur un téléphone Windows est utilisée pour fournir l’authentification secondaire.

![Exemple d’application d’authentification Windows Phone montrant la vérification](./media/howto-mfa-nps-extension-rdg/image27.png)

Une fois que vous vous êtes correctement authentifié à l’aide de la méthode d’authentification secondaire, vous êtes connecté à la passerelle des services Bureau à distance comme d’habitude. Toutefois, étant donné que vous devez utiliser une méthode d’authentification secondaire à l’aide d’une application mobile sur un appareil approuvé, le processus de connexion est plus sûr.

### <a name="view-event-viewer-logs-for-successful-logon-events"></a>Afficher les journaux d’activité de l’Observateur d’événements pour les événements de connexion réussie

Pour afficher les événements de connexion réussie dans les journaux d’activité de l’Observateur d’événements Windows, vous pouvez émettre la commande Windows PowerShell suivante pour interroger les journaux d’activité des Services Windows Terminal et de sécurité Windows.

Pour interroger les événements de connexion réussie dans les journaux d’activité des opérations de la passerelle _(Observateur d’événements\Journaux des applications et des services\Microsoft\Windows\TerminalServices-Gateway\Opérationnel)_ , utilisez les commandes PowerShell suivantes :

* `Get-WinEvent -Logname Microsoft-Windows-TerminalServices-Gateway/Operational | where {$_.ID -eq '300'} | FL`
* Cette commande affiche les événements Windows qui indiquent que l’utilisateur a respecté les exigences de stratégies d’autorisation de ressource (RD RAP) et que l’accès lui est accordé.

![Consultation des événements avec PowerShell](./media/howto-mfa-nps-extension-rdg/image28.png)

* `Get-WinEvent -Logname Microsoft-Windows-TerminalServices-Gateway/Operational | where {$_.ID -eq '200'} | FL`
* Cette commande affiche les événements qui indiquent quand l’utilisateur a respecté les exigences de stratégies d’autorisation de connexion.

![Consultation de la stratégie d’autorisation de connexion avec PowerShell](./media/howto-mfa-nps-extension-rdg/image29.png)

Vous pouvez également afficher ce journal et filtrer les ID d’événement, 300 et 200. Pour interroger les événements de connexion réussie dans les journaux d’activité de l’observateur d’événements de sécurité, utilisez la commande suivante :

* `Get-WinEvent -Logname Security | where {$_.ID -eq '6272'} | FL`
* Cette commande peut être exécutée sur le serveur NPS central ou sur le serveur de passerelle des services Bureau à distance.

![Exemples d’événements de connexion réussie](./media/howto-mfa-nps-extension-rdg/image30.png)

Vous pouvez également afficher le journal de sécurité ou la vue personnalisée des services de stratégie et d’accès réseau, comme indiqué ci-dessous :

![Observateur d’événements des services de stratégie et d’accès réseau (NPS)](./media/howto-mfa-nps-extension-rdg/image31.png)

Sur le serveur où vous avez installé l'extension NPS d'Azure AD MFA, vous pouvez rechercher les journaux d'applications de l'observateur d'événements propres à l'extension sur _Journaux des applications et des services\Microsoft\AzureMfa_.

![Journaux d’activité d’application AuthZ de l’observateur d’événements](./media/howto-mfa-nps-extension-rdg/image32.png)

## <a name="troubleshoot-guide"></a>Guide de résolution des problèmes

Si la configuration ne fonctionne pas comme prévu, la première chose à faire pour résoudre les problèmes consiste à vérifier que l'utilisateur est configuré pour utiliser Azure AD MFA. Connectez l’utilisateur au [portail Azure](https://portal.azure.com). Si les utilisateurs sont invités à effectuer une vérification secondaire et parviennent à s'authentifier, vous pouvez éliminer une configuration incorrecte d'Azure AD MFA.

Si Azure AD MFA fonctionne pour le ou les utilisateurs, vous devez examiner les journaux d'événements pertinents. Citons notamment les journaux d'événements de sécurité, les journaux des opérations de la passerelle et les journaux Azure AD MFA décrits dans la section précédente.

Voici un exemple de sortie du journal de sécurité montrant un échec d’événement de connexion (ID d’événement 6273).

![Exemple d’événement d’échec de la connexion](./media/howto-mfa-nps-extension-rdg/image33.png)

Voici un événement associé aux journaux d’activité AzureMFA :

![Exemple de journal Azure AD MFA dans l'observateur d'événements](./media/howto-mfa-nps-extension-rdg/image34.png)

Pour exécuter des options de résolution des problèmes avancée, consultez les fichiers journaux au format base de données de serveur NPS dans lesquels est installé le service NPS. Ces fichiers journaux sont créés dans le dossier _%SystemRoot%\System32\Logs_ comme fichiers texte délimité par des virgules.

Pour obtenir une description de ces fichiers journaux, consultez [Interpréter des fichiers journaux au format base de données de serveur NPS](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc771748(v=ws.10)). Les entrées de ces fichiers journaux peuvent être difficiles à interpréter sans les importer dans une feuille de calcul ou une base de données. Vous pouvez rechercher plusieurs analyseurs IAS en ligne pour vous aider à interpréter les fichiers journaux.

L’image ci-dessous montre la sortie d’une de ces [applications de logiciel à contribution volontaire](https://www.deepsoftware.com/iasviewer) téléchargeable.

![Exemple d’analyseur IAS d’application de logiciel à contribution volontaire](./media/howto-mfa-nps-extension-rdg/image35.png)

Enfin, pour obtenir des options de résolution des problèmes supplémentaires, vous pouvez utiliser un analyseur de protocoles, tel que [Microsoft Message Analyzer](/message-analyzer/microsoft-message-analyzer-operating-guide).

L’image ci-dessous depuis Microsoft Message Analyser indique le trafic réseau filtré sur le protocole RADIUS qui contient le nom d’utilisateur **CONTOSO\AliceC**.

![Microsoft Message Analyzer affichant le trafic filtré](./media/howto-mfa-nps-extension-rdg/image36.png)

## <a name="next-steps"></a>Étapes suivantes

[Guide pratique pour obtenir Azure AD Multi-Factor Authentication](concept-mfa-licensing.md)

[Passerelle des services Bureau à distance et serveur Multi-Factor Authentication avec RADIUS](howto-mfaserver-nps-rdg.md)

[Intégrer vos répertoires locaux à Azure Active Directory](../hybrid/whatis-hybrid-identity.md)