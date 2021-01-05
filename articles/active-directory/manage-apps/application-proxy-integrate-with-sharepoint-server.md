---
title: Activer l’accès distant à SharePoint – Proxy d’application Azure AD
description: Aborde les notions de base relatives à l’intégration d’un serveur SharePoint local au proxy d’application Azure AD.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 10/02/2019
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: c318c539b1c09761ed81e7602808e415fdaf8b80
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658177"
---
# <a name="enable-remote-access-to-sharepoint-with-azure-ad-application-proxy"></a>Activer l’accès distant à SharePoint avec le proxy d’application Azure AD

Ce guide détaillé explique comment intégrer une batterie de serveurs SharePoint locale au proxy d’application Azure Active Directory (Azure AD).

## <a name="prerequisites"></a>Prérequis

Pour effectuer la configuration, vous avez besoin des ressources suivantes :
- Une batterie de serveurs SharePoint 2013 ou version ultérieure.
- Un locataire Azure AD avec un plan qui inclut le proxy d’application. En savoir plus sur les [formules et la tarification Azure AD](https://azure.microsoft.com/pricing/details/active-directory/).
- Un [domaine vérifié personnalisé](../fundamentals/add-custom-domain.md) dans le locataire Azure AD.
- Le service Active Directory local a été synchronisé avec Azure AD Connect, ce qui permet aux utilisateurs de [se connecter à Azure](../hybrid/plan-connect-user-signin.md).
- Un connecteur de proxy d’application installé et en cours d’exécution sur un ordinateur au sein du domaine d’entreprise.

La configuration de SharePoint avec le proxy d’application requiert deux URL :
- une URL externe, visible par les utilisateurs finaux et déterminée dans Azure AD. Cette URL peut utiliser un domaine personnalisé. En savoir plus sur l’[utilisation des domaines personnalisés dans le proxy d’application Azure AD](application-proxy-configure-custom-domain.md).
- Une URL interne, connue uniquement dans le domaine d’entreprise et jamais utilisée directement.

> [!IMPORTANT]
> Pour vous assurer que les liens sont correctement mappés, suivez ces recommandations pour l’URL interne :
> - utiliser le protocole HTTPS ;
> - n’utilisez pas de ports personnalisés.
> - Dans le DNS (Domain Name System) d’entreprise, créez un hôte (A) pointant vers le serveur WFE (ou équilibreur de charge) SharePoint, et pas un Alias (CName).

Cet article utilise les valeurs suivantes :
- URL interne : `https://sharepoint`
- URL externe : `https://spsites-demo1984.msappproxy.net/`
- Compte du pool d’applications pour l’application web SharePoint : `Contoso\spapppool`

## <a name="step-1-configure-an-application-in-azure-ad-that-uses-application-proxy"></a>Étape 1 : Configurer une application dans Azure AD qui utilise le proxy d’application

Dans cette étape, vous allez créer une application dans votre locataire Azure Active Directory qui utilise le proxy d’application. Vous définissez l’URL externe et spécifiez l’URL interne, les deux étant utilisées ultérieurement dans SharePoint.

1. Créez l’application comme décrit avec les paramètres suivants. Pour obtenir des instructions pas à pas, consultez [Publication d’applications à l’aide du proxy d’application Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad).
   * **URL interne** : URL interne SharePoint qui sera définie ultérieurement dans SharePoint, par exemple `https://sharepoint`.
   * **Pré-authentification** : Azure Active Directory
   * **Traduire les URL dans les en-têtes** : Non
   * **Traduisez les URL dans le corps de l’application** : Non

   ![Publier SharePoint en tant qu’application](./media/application-proxy-integrate-with-sharepoint-server/publish-app.png)

1. Une fois que votre application est publiée, suivez les étapes suivantes pour configurer les paramètres d’authentification unique :

   1. Dans la page de l’application dans le portail, sélectionnez **Authentification unique**.
   1. Pour le **mode d’authentification unique**, sélectionnez **Authentification Windows intégrée**.
   1. Définissez **SPN d’application interne** sur la valeur que vous avez définie précédemment. Pour cet exemple, la valeur est `HTTP/sharepoint`.
   1. Sous **Identité de connexion déléguée**, sélectionnez l’option la plus appropriée pour la configuration de votre forêt Active Directory. Par exemple, si vous avez un seul domaine Active Directory dans votre forêt, sélectionnez **nom de compte SAM local** (comme indiqué dans la capture d’écran suivante). Toutefois, si vos utilisateurs ne se trouvent pas dans le même domaine que SharePoint et les serveurs Application Proxy Connector, sélectionnez **Nom d’utilisateur principal local** (non affiché dans la capture d’écran).

   ![Configurer l’authentification Windows intégrée pour l’authentification unique](./media/application-proxy-integrate-with-sharepoint-server/configure-iwa.png)

1. Pour terminer la configuration de votre application, accédez à la section **Utilisateurs et groupes** et affectez des utilisateurs pouvant accéder à cette application. 

## <a name="step-2-configure-the-sharepoint-web-application"></a>Étape 2 : Configurer l’application web SharePoint

L’application web SharePoint doit être configurée avec Kerberos et les mappages d’accès de substitution appropriés pour fonctionner correctement avec le proxy d’application Azure AD. Il existe deux options possibles :

- Créez une application web et utilisez uniquement la zone par défaut. C’est l’option recommandée, car elle offre une expérience optimale avec SharePoint (par exemple, les liens des alertes par e-mail générées par SharePoint pointent toujours vers la zone par défaut).
- Étendez une application web existante pour configurer Kerberos dans une zone autre que celle par défaut.

> [!IMPORTANT]
> Quelle que soit la zone utilisée, le compte du pool d’applications de l’application web SharePoint doit être un compte de domaine pour que Kerberos fonctionne correctement.

### <a name="provision-the-sharepoint-web-application"></a>Provisionner l’application web SharePoint

- Si vous créez une application web et utilisez uniquement la zone par défaut (option recommandée) :

    1. Démarrez le **shell d’administration SharePoint** et exécutez le script suivant :

       ```powershell
       # This script creates a web application and configures the Default zone with the internal/external URL needed to work with Azure AD Application Proxy
       # Edit variables below to fit your environment. Note that the managed account must exist and it must be a domain account
       $internalUrl = "https://sharepoint"
       $externalUrl = "https://spsites-demo1984.msappproxy.net/"
       $applicationPoolManagedAccount = "Contoso\spapppool"
            
       $winAp = New-SPAuthenticationProvider -UseWindowsIntegratedAuthentication -DisableKerberos:$false
       $wa = New-SPWebApplication -Name "SharePoint - AAD Proxy" -Port 443 -SecureSocketsLayer -URL $externalUrl -ApplicationPool "SharePoint - AAD Proxy" -ApplicationPoolAccount (Get-SPManagedAccount $applicationPoolManagedAccount) -AuthenticationProvider $winAp
       New-SPAlternateURL -Url $internalUrl -WebApplication $wa -Zone Default -Internal
       ```

    2. Ouvrez le site **Administration centrale de SharePoint**.
    1. Sous **Paramètres système**, sélectionnez **Configurer des mappages d’accès alternatifs**. La zone **Collection de mappages d’accès alternatif** s’ouvre.
    1. Filtrez l’affichage avec la nouvelle application web et vérifiez que vous voyez ce qui suit :

       ![Mappages d’accès de substitution de l’application web](./media/application-proxy-integrate-with-sharepoint-server/new-webapp-aam.png)

- Si vous étendez une application web existante à une nouvelle zone (si vous ne pouvez pas utiliser la zone par défaut) :

    1. Démarrez SharePoint Management Shell et exécutez le script suivant :

       ```powershell
       # This script extends an existing web application to Internet zone with the internal/external URL needed to work with Azure AD Application Proxy
       # Edit variables below to fit your environment
       $webAppUrl = "http://spsites/"
       $internalUrl = "https://sharepoint"
       $externalUrl = "https://spsites-demo1984.msappproxy.net/"
       
       $winAp = New-SPAuthenticationProvider -UseWindowsIntegratedAuthentication -DisableKerberos:$false
       $wa = Get-SPWebApplication $webAppUrl
       New-SPWebApplicationExtension -Name "SharePoint - AAD Proxy" -Identity $wa -SecureSocketsLayer -Zone Extranet -Url $externalUrl -AuthenticationProvider $winAp
       New-SPAlternateURL -Url $internalUrl -WebApplication $wa -Zone Extranet -Internal
       ```

    2. Ouvrez le site **Administration centrale de SharePoint**.
    1. Sous **Paramètres système**, sélectionnez **Configurer des mappages d’accès alternatifs**. La zone **Collection de mappages d’accès alternatif** s’ouvre.
    1. Filtrez l’affichage avec l’application web qui a été étendue et vérifiez que vous voyez ce qui suit :

        ![Mappages d’accès de substitution de l’application étendue](./media/application-proxy-integrate-with-sharepoint-server/extend-webapp-aam.png)

### <a name="make-sure-the-sharepoint-web-application-is-running-under-a-domain-account"></a>Vérifiez que l’application web SharePoint s’exécute sous un compte de domaine

Pour identifier le compte qui exécute le pool d’applications de l’application web SharePoint et vous assurer qu’il s’agit d’un compte de domaine, procédez comme suit :

1. Ouvrez le site **Administration centrale de SharePoint**.
1. Accédez à **Sécurité**, puis sélectionnez **Configurer les comptes de service**.
1. Sélectionnez **Pool d’applications web - YourWebApplicationName**.

   ![Choix disponibles pour configurer un compte de service](./media/application-proxy-integrate-with-sharepoint-server/service-web-application.png)

1. Confirmez que **Sélectionner un compte pour ce composant** retourne un compte de domaine et mémorisez-le, car vous en aurez besoin à l’étape suivante.

### <a name="make-sure-that-an-https-certificate-is-configured-for-the-iis-site-of-the-extranet-zone"></a>Vérifiez qu’un certificat HTTPS est configuré pour le site IIS de la zone Extranet

Étant donné que l’URL interne utilise le protocole HTTPS (`https://SharePoint/`), un certificat doit être défini sur le site Internet Information Services (IIS).

1. Ouvrez la console Windows PowerShell.
1. Exécutez le script suivant pour générer un certificat auto-signé et l’ajouter à MON magasin sur l’ordinateur :

   ```powershell
   # Replace "SharePoint" with the actual hostname of the Internal URL of your Azure AD proxy application
   New-SelfSignedCertificate -DnsName "SharePoint" -CertStoreLocation "cert:\LocalMachine\My"
   ```

   > [!IMPORTANT]
   > Les certificats auto-signés sont uniquement destinés à des fins de test. Pour les environnements de production, nous vous recommandons vivement d’utiliser des certificats émis par une autorité de certification à la place.

1. Ouvrez la console du gestionnaire des services Internet Information Services.
1. Développez le serveur dans l’arborescence, développez **Sites**, sélectionnez le site **SharePoint – Proxy AAD**, puis **Liaisons**.
1. Sélectionnez la **liaison https**, puis **Modifier**.
1. Dans le champ de certificat TLS/SSL, choisissez le certificat **SharePoint**, puis sélectionnez **OK**.

Vous pouvez maintenant accéder au site SharePoint en externe par l’intermédiaire du proxy d’application Azure AD.

## <a name="step-3-configure-kerberos-constrained-delegation"></a>Étape 3 : Configurer une délégation Kerberos contrainte

Les utilisateurs s’authentifient initialement dans Azure AD, puis dans SharePoint à l’aide de Kerberos via le connecteur de proxy Azure AD. Pour permettre au connecteur d’obtenir un jeton Kerberos pour le compte de l’utilisateur Azure AD, vous devez configurer la délégation Kerberos contrainte (KCD) avec la transition de protocole. Pour en savoir plus sur la KCD, consultez [Présentation de la délégation Kerberos contrainte](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj553400(v=ws.11)).

### <a name="set-the-spn-for-the-sharepoint-service-account"></a>Définir le SPN pour le compte de service SharePoint

Dans cet article, l’URL interne est `https://sharepoint`, et le nom de principal du service (SPN) est donc `HTTP/sharepoint`. Vous devez remplacer ces valeurs par les valeurs qui correspondent à votre environnement.
Pour inscrire le SPN `HTTP/sharepoint` pour le compte du pool d’applications SharePoint `Contoso\spapppool`, exécutez la commande suivante à partir d’une invite de commandes, en tant qu’administrateur du domaine :

`setspn -S HTTP/sharepoint Contoso\spapppool`

La commande `Setspn` recherche le SPN avant de l’ajouter. Si le SPN existe déjà, une erreur **Valeur SPN en double** apparaît. Dans ce cas, envisagez de supprimer le SPN existant s’il n’est pas défini sous le compte de pool d’applications correct. Vous pouvez vérifier que le SPN a bien été ajouté en exécutant la commande `Setspn` avec l’option -L. Pour en savoir plus sur cette commande, consultez [Setspn](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/cc731241(v=ws.11)).

### <a name="make-sure-the-connector-is-trusted-for-delegation-to-the-spn-that-was-added-to-the-sharepoint-application-pool-account"></a>Vérifiez que le connecteur est approuvé pour la délégation au SPN qui a été ajouté au compte de pool d’applications SharePoint

Configurez la délégation KCD afin que le service de proxy d’application Azure AD puisse déléguer des identités d’utilisateur au compte de pool d’applications SharePoint. Pour configurer la délégation KCD, activez le connecteur de proxy d’application pour récupérer les tickets Kerberos des utilisateurs qui ont été authentifiés dans Azure AD. Puis ce serveur transmet le contexte à l’application cible (ici SharePoint).

Pour configurer la délégation KCD, suivez les étapes suivantes pour chaque ordinateur connecteur :

1. Connectez-vous à un contrôleur de domaine en tant qu’administrateur de domaine, puis ouvrez Utilisateurs et ordinateurs Active Directory.
1. Recherchez l’ordinateur exécutant le connecteur de proxy Azure AD. Dans cet exemple, il s’agit du serveur SharePoint.
1. Double-cliquez sur l’ordinateur, puis sélectionnez l’onglet **Délégation**.
1. Vérifiez que les options de délégation sont définies sur **N’approuver cet ordinateur que pour la délégation aux services spécifiés**. Sélectionnez ensuite **Utiliser tout protocole d’authentification**.
1. Cliquez sur le bouton **Ajouter**, sélectionnez **Utilisateurs ou ordinateurs**, puis recherchez le compte de pool d’applications SharePoint. Par exemple : `Contoso\spapppool`.
1. Dans la liste des noms de principal de service, sélectionnez celui que vous avez créé précédemment pour le compte de service.
1. Sélectionnez **OK**, puis de nouveau **OK** pour enregistrer vos modifications.
  
   ![Paramètres de délégation](./media/application-proxy-integrate-with-sharepoint-server/delegation-box2.png)

Vous êtes maintenant prêt à vous connecter à SharePoint à l’aide de l’URL externe et à vous authentifier auprès d’Azure.

## <a name="troubleshoot-sign-in-errors"></a>Résoudre les erreurs de connexion

Si la connexion au site ne fonctionne pas, vous pouvez obtenir plus d’informations sur le problème dans les journaux du connecteur : À partir de l’ordinateur exécutant le connecteur, ouvrez l’observateur d’événements, accédez à **Journaux des applications et des services** > **Microsoft** > **AadApplicationProxy** > **Connecteur** et inspectez le journal **Admin**.

## <a name="next-steps"></a>Étapes suivantes

* [Utilisation des domaines personnalisés dans le proxy d’application Azure AD](application-proxy-configure-custom-domain.md)
* [Présentation des connecteurs de proxy d’application Azure AD](application-proxy-connectors.md)