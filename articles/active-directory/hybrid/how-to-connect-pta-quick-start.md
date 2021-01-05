---
title: Authentification directe Azure AD - Démarrage rapide | Microsoft Docs
description: Cet article explique comment commencer à utiliser l’authentification directe d’Azure Active Directory (Azure AD).
services: active-directory
keywords: Authentification directe Azure AD Connect, installation d’Active Directory, composants requis pour Azure AD, SSO, Authentification unique
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 04/13/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 31bdfc7e3eca2a2678b75b80973a1680448737d1
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507674"
---
# <a name="azure-active-directory-pass-through-authentication-quickstart"></a>Authentification directe Azure Active Directory : Démarrage rapide

## <a name="deploy-azure-ad-pass-through-authentication"></a>Déployer l’authentification directe Azure AD

L’authentification directe Azure Active Directory (Azure AD) permet à vos utilisateurs de se connecter aux applications sur site et aux applications cloud à l’aide des mêmes mots de passe. L'authentification directe connecte les utilisateurs en validant leurs mots de passe directement par rapport à l'annuaire Active Directory sur site.

>[!IMPORTANT]
>Si vous procédez à une migration depuis AD FS (ou d’autres technologies de fédération) vers l’authentification directe, nous vous recommandons vivement de vous référer à notre guide de déploiement détaillé, publié [ici](https://aka.ms/adfstoPTADPDownload).

>[!NOTE]
>Si vous déployez l’authentification directe avec le cloud Azure Government, consultez [Considérations sur les identités hybrides pour Azure Government](./reference-connect-government-cloud.md).

Suivez ces instructions pour déployer l’authentification directe sur votre locataire :

## <a name="step-1-check-the-prerequisites"></a>Étape 1 : Vérifier les prérequis

Vérifiez que les prérequis suivants sont remplis.

>[!IMPORTANT]
>Du point de vue de la sécurité, les administrateurs doivent traiter le serveur exécutant l’agent PTA comme s’il s’agissait d’un contrôleur de domaine.  Les serveurs d’agent PTA doivent être renforcés de la même manière que décrit dans [Sécurisation des contrôleurs de domaine contre les attaques](/windows-server/identity/ad-ds/plan/security-best-practices/securing-domain-controllers-against-attack)

### <a name="in-the-azure-active-directory-admin-center"></a>Dans le Centre d’administration Azure Active Directory

1. Créez un compte d’administrateur général « cloud uniquement » dans votre locataire Azure AD. De cette façon, vous pouvez gérer la configuration de votre locataire si vos services locaux venaient à échouer ou ne plus être disponibles. Découvrez comment [ajouter un compte d’administrateur général de type cloud uniquement](../fundamentals/add-users-azure-active-directory.md). Cette étape est essentielle si vous voulez éviter de vous retrouver en dehors de votre locataire.
2. Ajoutez un ou plusieurs [noms de domaine personnalisés](../fundamentals/add-custom-domain.md) à votre locataire Azure AD. Vos utilisateurs peuvent se connecter à l’aide de l’un de ces noms de domaine.

### <a name="in-your-on-premises-environment"></a>Dans votre environnement local

1. Identifiez un serveur Windows Server 2012 R2 ou ultérieur sur lequel exécuter Azure AD Connect. Le cas échéant, [activez TLS 1.2 sur le serveur](./how-to-connect-install-prerequisites.md#enable-tls-12-for-azure-ad-connect). Ajoutez ce serveur à la même forêt Active Directory que celle des utilisateurs dont vous devez valider les mots de passe.
2. Installez la [version la plus récente d’Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) sur le serveur identifié à l’étape précédente. Si vous exécutez déjà Azure AD Connect, vérifiez qu’il s’agit de la version 1.1.750.0 ou d’une version ultérieure.

    >[!NOTE]
    >Les versions 1.1.557.0, 1.1.558.0, 1.1.561.0 et 1.1.614.0 d’Azure AD Connect comportent un problème lié à la synchronisation de hachage de mot de passe. Si vous _ne prévoyez pas_ d’utiliser la synchronisation de hachage de mot de passe en même temps que l’authentification directe, lisez les [Notes de publication Azure AD Connect](./reference-connect-version-history.md).

3. Identifiez un ou plusieurs serveurs supplémentaires (exécutant Windows Server 2012 R2 ou version ultérieure, avec TLS 1.2 activé) sur lesquels vous pouvez exécuter des agents d’authentification autonomes. Ces serveurs supplémentaires sont nécessaires pour garantir une haute disponibilité des requêtes de connexion. Ajoutez ces serveurs à la même forêt Active Directory que celle des utilisateurs dont vous devez valider les mots de passe.

    >[!IMPORTANT]
    >Dans les environnements de production, nous vous recommandons l’utilisation d’au moins 3 agents d’authentification s’exécutant sur votre locataire. Il existe une limite système de 40 agents d’authentification par client. En tant que bonne pratique, traitez tous les serveurs exécutant des agents d’authentification comme des systèmes de niveau 0 (voir [référence](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material)).

4. S’il existe un pare-feu entre vos serveurs et Azure AD, configurez les éléments suivants :
   - Assurez-vous que les agents d’authentification peuvent effectuer des requêtes *sortantes* vers Azure AD sur les ports suivants :

     | Numéro de port | Utilisation |
     | --- | --- |
     | **80** | Télécharge les listes de révocation de certificats lors de la validation du certificat TLS/SSL |
     | **443** | Gère toutes les communications sortantes avec le service |
     | **8080** (facultatif) | Les agents d’authentification signalent leur état toutes les dix minutes sur le port 8080 (si le port 443 n’est pas disponible). Cet état est affiché sur le portail Azure AD. Le port 8080 _n’est pas utilisé_ pour les connexions utilisateur. |
     
     Si votre pare-feu applique les règles en fonction des utilisateurs d’origine, ouvrez ces ports au trafic provenant des services Windows exécutés en tant que service réseau.
   - Si votre pare-feu ou proxy vous permet d’ajouter des entrées DNS à une liste d’autorisation, ajoutez des connexions à **\*.msappproxy.net** et **\*.servicebus.windows.net**. Dans le cas contraire, autorisez l’accès aux [plages d’adresses IP du centre de données Azure](https://www.microsoft.com/download/details.aspx?id=41653), qui sont mises à jour chaque semaine.
   - Vos agents d’authentification doivent accéder à **login.windows.net** et à **login.microsoftonline.net** pour l’inscription initiale. Par conséquent, ouvrez également votre pare-feu pour ces URL.
    - Pour la validation des certificats, débloquez les URL suivantes : **crl3.digicert.com:80**, **crl4.digicert.com:80**, **ocsp.digicert.com:80**, **www\.d-trust.net:80**, **root-c3-ca2-2009.ocsp.d-trust.net:80**, **crl.microsoft.com:80**, **oneocsp.microsoft.com:80** et **ocsp.msocsp.com:80**. Ces URL étant utilisées pour la validation de certificat avec d’autres produits Microsoft, elles sont peut-être déjà débloquées.

### <a name="azure-government-cloud-prerequisite"></a>Cloud Azure Government - Prérequis
Avant d'activer l'authentification directe via Azure AD Connect à l'étape 2, téléchargez la dernière version de l'agent PTA à partir du portail Azure.  Vous devez vous assurer que vous disposez de la version **1.5.1742.0.** ou version ultérieure.  Pour vérifier votre agent, consultez [Mettre à niveau les agents d'authentification](how-to-connect-pta-upgrade-preview-authentication-agents.md).

Après avoir téléchargé la dernière version de l'agent, suivez les instructions ci-dessous pour configurer l'authentification directe via Azure AD Connect.

## <a name="step-2-enable-the-feature"></a>Étape 2 : Activer la fonctionnalité

Activez l’authentification directe via [Azure AD Connect](whatis-hybrid-identity.md).

>[!IMPORTANT]
>Vous pouvez activer l’authentification directe sur le serveur principal ou sur un serveur intermédiaire Azure AD Connect. Nous vous recommandons vivement de l’activer à partir du serveur principal. Si vous installez un serveur intermédiaire Azure AD Connect plus tard, vous **devez** continuer à choisir l’authentification directe comme option de connexion. Le choix d’une autre option **désactivera** l’authentification directe sur le client et remplacera le paramètre dans le serveur principal.

Si vous installez Azure AD Connect pour la première fois, choisissez le [chemin d’installation personnalisé](how-to-connect-install-custom.md). Dans la page **Connexion utilisateur**, choisissez **Authentification directe** comme **méthode d’authentification**. Si l’opération réussit, un agent d’authentification directe est installé sur le même serveur qu’Azure AD Connect. La fonctionnalité d’authentification directe est également activée sur votre locataire.

![Azure AD Connect : Connexion de l’utilisateur](./media/how-to-connect-pta-quick-start/sso3.png)

Si vous avez déjà installé Azure AD Connect à l’aide du chemin [d’installation rapide](how-to-connect-install-express.md) ou [d’installation personnalisée](how-to-connect-install-custom.md), sélectionnez la tâche **Modifier la connexion utilisateur** dans Azure AD Connect, puis cliquez sur **Suivant**. Ensuite, sélectionnez **Authentification directe** comme mode d’authentification. Si l’opération réussit, un agent d’authentification directe est installé sur le même serveur qu’Azure AD Connect et la fonctionnalité est activée sur votre locataire.

![Azure AD Connect : Modifier la connexion de l’utilisateur](./media/how-to-connect-pta-quick-start/changeusersignin.png)

>[!IMPORTANT]
>L’authentification directe est une fonctionnalité au niveau du locataire. Son activation affecte la connexion des utilisateurs dans _tous_ les domaines gérés dans votre locataire. Si vous passez d'Active Directory Federation Services (ADFS) à l'authentification directe, vous devez attendre au moins 12 heures avant de fermer votre infrastructure AD FS. Ce délai d'attente permet aux utilisateurs de continuer à se connecter à Exchange ActiveSync pendant la transition. Pour obtenir plus d’informations sur la migration d’AD FS vers l’authentification directe, consultez notre plan de déploiement détaillé publié [ici](https://aka.ms/adfstoptadpdownload).

## <a name="step-3-test-the-feature"></a>Étape 3 : Tester la fonctionnalité

Suivez ces instructions pour vérifier que vous avez activé correctement l’authentification directe :

1. Connectez-vous au [Centre d’administration Azure Active Directory](https://aad.portal.azure.com) à l’aide des informations d’identification d’administrateur général de votre locataire.
2. Sélectionnez **Active Directory** dans le volet de gauche.
3. Sélectionnez ensuite **Azure AD Connect**.
4. Vérifiez que la fonctionnalité **Authentification directe** est **activée**.
5. Sélectionnez **Authentification directe**. Le volet **Authentification directe** répertorie les serveurs sur lesquels vos agents d’authentification sont installés.

![Centre d'administration Azure Active Directory : Volet Azure AD Connect](./media/how-to-connect-pta-quick-start/pta7.png)

![Centre d'administration Azure Active Directory : Volet Authentification directe](./media/how-to-connect-pta-quick-start/pta8.png)

À ce stade, les utilisateurs de tous les domaines gérés de votre locataire peuvent se connecter à l’aide de l’authentification directe. Toutefois, les utilisateurs des domaines fédérés continuent à se connecter à l’aide d'AD FS ou d’un autre fournisseur de fédération précédemment configuré. Si vous convertissez un domaine fédéré en domaine géré, tous les utilisateurs de ce domaine se connectent alors automatiquement à l’aide de l’authentification directe. La fonctionnalité Authentification directe n’a pas d’incidence sur les utilisateurs cloud uniquement.

## <a name="step-4-ensure-high-availability"></a>Étape 4 : Garantir une haute disponibilité

Si vous envisagez de déployer l’authentification directe dans un environnement de production, vous devez installer des agents d’authentification autonomes supplémentaires. Installez ces agents d’authentification sur des serveurs _autres_ que celui qui exécute Azure AD Connect. Cette configuration fournit une haute disponibilité pour les requêtes de connexion des utilisateurs.

>[!IMPORTANT]
>Dans les environnements de production, nous vous recommandons l’utilisation d’au moins 3 agents d’authentification s’exécutant sur votre locataire. Il existe une limite système de 40 agents d’authentification par client. En tant que bonne pratique, traitez tous les serveurs exécutant des agents d’authentification comme des systèmes de niveau 0 (voir [référence](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material)).

L’installation de plusieurs agents d’authentification directe assure une haute disponibilité, mais pas d’équilibrage de charge entre les agents d’authentification. Pour savoir combien d’agents d’authentification sont nécessaires pour votre client, tenez compte des pics et de la charge moyenne des demandes de connexion que vous attendez sur votre client. À titre de référence, un seul agent d’authentification peut gérer entre 300 et 400 authentifications par seconde sur un serveur doté d’un CPU à 4 cœurs et de 16 Go de RAM.

Pour estimer le trafic réseau, suivez les instructions de dimensionnement suivantes :
- La taille de la charge utile de chaque requête est de (0,5 K + 1 K x num_of_agents) octets. Cela correspond aux données envoyées entre Azure AD et l'agent d'authentification. Ici, « num_of_agents » indique le nombre d’agents d’authentification inscrit sur votre abonné.
- La taille de la charge utile de chaque réponse est de 1 kilooctet. Cela correspond aux données envoyées entre l'agent d'authentification et Azure AD.

Pour la plupart des clients, trois agents d’authentification au total suffisent à offrir la haute disponibilité et suffisamment de capacité. Vous devriez installer des agents d’authentification près de vos contrôleurs de domaine pour améliorer la latence de connexion.

Pour commencer, suivez ces instructions pour télécharger l’agent d’authentification :

1. Pour télécharger la dernière version de l’agent d’authentification (version 1.5.193.0 ou ultérieure), connectez-vous au [Centre d’administration Azure Active Directory](https://aad.portal.azure.com) avec les droits d’administrateur général de votre locataire.
2. Sélectionnez **Active Directory** dans le volet de gauche.
3. Sélectionnez **Azure AD Connect**, **Authentification directe**, puis **Télécharger l'agent**.
4. Cliquez sur le bouton **Accepter les conditions et télécharger**.

![Centre d'administration Azure Active Directory : Bouton de téléchargement de l'Agent d'authentification](./media/how-to-connect-pta-quick-start/pta9.png)

![Centre d'administration Azure Active Directory : Volet Télécharger l'Agent](./media/how-to-connect-pta-quick-start/pta10.png)

>[!NOTE]
>Vous pouvez aussi directement [télécharger le logiciel de l’agent d’authentification](https://aka.ms/getauthagent). Passez en revue et acceptez les [conditions de service](https://aka.ms/authagenteula) de l’agent d’authentification _avant_ de l’installer.

Il existe deux méthodes pour déployer un agent d’authentification autonome :

Tout d’abord, vous pouvez le faire interactivement en exécutant le fichier exécutable de l’agent d’authentification téléchargé et en fournissant les informations d’identification d’administrateur général de votre locataire lorsque vous y êtes invité.

La deuxième solution consiste à créer et à exécuter un script de déploiement sans assistance. C’est utile lorsque vous souhaitez déployer plusieurs agents d’authentification à la fois, ou installer des agents d’authentification sur des serveurs Windows qui ne disposent pas d’une interface utilisateur, ou auxquels vous ne pouvez pas accéder avec le Bureau à distance. Voici des instructions pour utiliser cette méthode :

1. Exécutez la commande suivante pour installer un agent d’authentification : `AADConnectAuthAgentSetup.exe REGISTERCONNECTOR="false" /q`.
2. Vous pouvez inscrire l’agent d’authentification auprès de notre service à l’aide de Windows PowerShell. Créez un objet d’informations d’identification PowerShell `$cred` qui contient un nom d’utilisateur et un mot de passe d’administrateur général pour votre locataire. Exécutez la commande suivante, en remplaçant *\<username\>* et *\<password\>* :

  ```powershell
  $User = "<username>"
  $PlainPassword = '<password>'
  $SecurePassword = $PlainPassword | ConvertTo-SecureString -AsPlainText -Force
  $cred = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $User, $SecurePassword
  ```
3. Accédez à **C:\Program Files\Microsoft Azure AD Connect Authentication Agent** et exécutez le script suivant en utilisant l’objet `$cred` que vous venez de créer :

  ```powershell
  RegisterConnector.ps1 -modulePath "C:\Program Files\Microsoft Azure AD Connect Authentication Agent\Modules\" -moduleName "PassthroughAuthPSModule" -Authenticationmode Credentials -Usercredentials $cred -Feature PassthroughAuthentication
  ```

>[!IMPORTANT]
>Si un agent d’authentification est installé sur une machine virtuelle, vous ne pouvez pas cloner cette machine virtuelle pour installer un autre agent d’authentification. Cette méthode n’est **pas prise en charge**.

## <a name="step-5-configure-smart-lockout-capability"></a>Étape 5 : Configurer la fonctionnalité de verrouillage intelligent

Le verrouillage intelligent empêche les personnes malveillantes de deviner vos mots de passe ou d’utiliser des méthodes de force brute pour rentrer dans vos systèmes. En configurant les paramètres de verrouillage intelligent dans Azure AD et/ou les paramètres de verrouillage appropriés dans l’Active Directory local, les attaques peuvent être filtrées avant qu’elles n’atteignent Active Directory. Lisez [cet article](../authentication/howto-password-smart-lockout.md) pour en savoir plus sur la configuration des paramètres de verrouillage intelligent sur votre locataire pour protéger vos comptes d’utilisateur.

## <a name="next-steps"></a>Étapes suivantes
- [Migrer à partir d’AD FS vers l’authentification directe](https://aka.ms/adfstoptadp) : guide détaillé de la migration d’AD FS (ou d’autres technologies de fédération) vers l’authentification directe.
- [Verrouillage intelligent](../authentication/howto-password-smart-lockout.md) : apprenez à configurer la fonctionnalité Verrouillage intelligent sur votre locataire pour protéger les comptes d'utilisateur.
- [Limitations actuelles](how-to-connect-pta-current-limitations.md) : découvrez les scénarios pris en charge et non pris en charge avec l'authentification directe.
- [Présentation technique approfondie](how-to-connect-pta-how-it-works.md) : découvrez comment fonctionne la fonctionnalité d'authentification directe.
- [Forum Aux Questions](how-to-connect-pta-faq.md) : obtenez des réponses aux questions fréquemment posées.
- [Résoudre les problèmes](tshoot-connect-pass-through-authentication.md) : apprenez à résoudre les problèmes courants liés à la fonctionnalité d’authentification directe.
- [Présentation approfondie de sécurité](how-to-connect-pta-security-deep-dive.md) : obtenez des informations techniques sur la fonctionnalité d'authentification directe.
- [Authentification unique fluide Azure AD](how-to-connect-sso.md) : explorez en détail cette fonctionnalité complémentaire.
- [UserVoice](https://feedback.azure.com/forums/169401-azure-active-directory/category/160611-directory-synchronization-aad-connect) : utilisez le Forum Azure Active Directory pour consigner de nouvelles demandes de fonctionnalités.
