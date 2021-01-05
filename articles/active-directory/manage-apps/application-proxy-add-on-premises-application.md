---
title: 'Tutoriel : Ajouter une application locale - Proxy d’application dans Azure AD'
description: Le service Proxy d'application d'Azure Active Directory (Azure AD) permet aux utilisateurs d'accéder aux applications locales en se connectant avec leur compte Azure AD. Ce tutoriel vous montre comment préparer votre environnement à l’utilisation du service Proxy d’application. Il ajoute ensuite une application locale à votre locataire Azure AD à partir du portail Azure.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: tutorial
ms.date: 12/10/2020
ms.author: kenwith
ms.reviewer: japere
ms.custom: contperf-fy21q2
ms.openlocfilehash: bcb484d62b7c4add7e1ab5562c19417a90cfb7e1
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97587551"
---
# <a name="tutorial-add-an-on-premises-application-for-remote-access-through-application-proxy-in-azure-active-directory"></a>Tutoriel : Ajouter une application locale pour un accès à distance via le service Proxy d'application d'Azure Active Directory

Le service Proxy d'application d'Azure Active Directory (Azure AD) permet aux utilisateurs d'accéder aux applications locales en se connectant avec leur compte Azure AD. Ce tutoriel prépare votre environnement à l'utilisation du service Proxy d'application. Lorsque votre environnement sera prêt, vous utiliserez le portail Azure pour ajouter une application locale à votre client Azure AD.

Les connecteurs sont un élément clé du proxy d’application. Pour en découvrir plus sur les connecteurs, consultez [Présentation des connecteurs de proxy d’application Azure AD](application-proxy-connectors.md).

Ce didacticiel :

> [!div class="checklist"]
> * ouvre les ports pour le trafic sortant et permet l'accès à des URL spécifiques ;
> * installe le connecteur sur votre serveur Windows et l'inscrit auprès du service Proxy d'application ;
> * vérifie que le connecteur est correctement installé et inscrit ;
> * ajoute une application locale à votre client Azure AD ;
> * vérifie qu’un utilisateur de test peut se connecter à l’application à l’aide d’un compte Azure AD.

## <a name="prerequisites"></a>Prérequis

Pour intégrer une application locale à Azure AD, vous devez :

* Avoir un [abonnement Microsoft Azure AD Premium](https://azure.microsoft.com/pricing/details/active-directory)
* Avoir un compte administrateur d’application
* Les identités utilisateur doivent être synchronisées à partir d’un annuaire local ou créées directement dans vos locataires Azure AD. La synchronisation des identités permet à Azure AD de pré-authentifier les utilisateurs avant de leur accorder l’accès aux applications publiées via le proxy d’application, mais aussi d’obtenir les identificateurs d’utilisateur nécessaires pour effectuer l’authentification unique (SSO).

### <a name="windows-server"></a>Windows Server

Pour utiliser le proxy d’application, vous devez disposer d’un serveur Windows exécutant Windows Server 2012 R2 ou ultérieur. Vous allez installer le connecteur de proxy d’application sur le serveur. Ce serveur de connecteurs doit se connecter aux services Proxy d'application d'Azure et aux applications locales que vous envisagez de publier.

Pour bénéficier d'une haute disponibilité dans votre environnement de production, nous vous recommandons d'utiliser plusieurs serveurs. Pour ce tutoriel, un seul serveur Windows suffira.

> [!IMPORTANT]
> Si vous installez le connecteur sur Windows Server 2019, vous devez désactiver la prise en charge du protocole HTTP2 dans le composant WinHttp pour que la délégation contrainte Kerberos fonctionne correctement. Ceci est désactivé par défaut dans les versions antérieures des systèmes d’exploitation pris en charge. L’ajout de la clé de Registre suivante et le redémarrage du serveur la désactivent sur Windows Server 2019. Notez qu’il s’agit d’une clé de Registre au niveau de la machine.
>
> ```
> Windows Registry Editor Version 5.00
> 
> HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\WinHttp\EnableDefaultHttp2 Value: 0
> ```
>
> La clé peut être définie par le biais de PowerShell à l’aide de la commande suivante.
> ```
> Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\WinHttp\' -Name EnableDefaultHTTP2 -Value 0
>

#### <a name="recommendations-for-the-connector-server"></a>Recommandations pour le serveur de connecteurs

1. Placez physiquement le serveur de connecteurs à proximité des serveurs d'applications afin d'optimiser les performances entre le connecteur et l'application. Pour plus d'informations, consultez [Considérations sur la topologie réseau](application-proxy-network-topology.md).
1. Le serveur de connecteurs et les serveurs d’applications web doivent appartenir au même domaine Active Directory ou chevaucher des domaines approbateurs. Les serveurs doivent impérativement se trouver dans le même domaine ou dans des domaines approbateurs pour utiliser l’authentification unique avec l’authentification Windows intégrée (IWA) et la délégation Kerberos contrainte (KCD). Si le serveur de connecteurs et les serveurs d'applications web se trouvent dans des domaines Active Directory différents, vous devez utiliser la délégation basée sur les ressources pour l'authentification unique. Pour plus d’informations, consultez [KCD pour l’authentification unique avec le proxy d’application](application-proxy-configure-single-sign-on-with-kcd.md).

> [!WARNING]
> Si vous avez déployé le proxy de protection de mot de passe Azure AD, n’installez pas le proxy d’application Azure AD et le proxy de protection de mot de passe Azure AD sur le même ordinateur. Le proxy d’application Azure AD et le proxy de protection de mot de passe Azure AD installent différentes versions du programme de mise à jour de l’agent Azure AD Connect. Ces différentes versions sont incompatibles quand elles sont installées ensemble sur le même ordinateur.

#### <a name="tls-requirements"></a>Exigences relatives à TLS

TLS 1.2 doit être activé sur le serveur de connecteurs Windows Server avant l'installation du connecteur du Proxy d'application.

Pour activer TLS 1.2 :

1. Définissez les clés de Registre suivantes :

   ```
   Windows Registry Editor Version 5.00

   [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2]
   [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client]
   "DisabledByDefault"=dword:00000000
   "Enabled"=dword:00000001
   [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server]
   "DisabledByDefault"=dword:00000000
   "Enabled"=dword:00000001
   [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319]
   "SchUseStrongCrypto"=dword:00000001
   ```

1. Redémarrez le serveur.

> [!Note]
> Microsoft met à jour les services Azure pour qu’ils utilisent des certificats TLS issus d’un autre ensemble d’autorités de certification racines. Cette modification est effectuée, car les certificats d’autorité de certification actuels ne sont pas conformes à l’une des exigences de base du CA/Browser Forum. Pour plus d’informations, consultez [Changements des certificats Azure TLS](https://docs.microsoft.com/azure/security/fundamentals/tls-certificate-changes).

## <a name="prepare-your-on-premises-environment"></a>Préparer votre environnement local

Tout d’abord, activez la communication avec les centres de données Azure afin de préparer votre environnement pour le service Proxy d’application Azure AD. S’il y a un pare-feu dans le chemin, vérifiez qu’il est ouvert. La présence d’un pare-feu ouvert permet au connecteur d’envoyer des requêtes HTTPS (TCP) au service Proxy d’application.

> [!IMPORTANT]
> Si vous installez le connecteur pour le cloud Azure Government, suivez les [prérequis](../hybrid/reference-connect-government-cloud.md#allow-access-to-urls) et les [étapes d’installation](../hybrid/reference-connect-government-cloud.md#install-the-agent-for-the-azure-government-cloud). Cela nécessite l’activation de l’accès à un autre ensemble d’URL et un paramètre supplémentaire pour exécuter l’installation.

### <a name="open-ports"></a>Ouvrir les ports

Ouvrez les ports suivants pour le trafic **sortant**.

   | Numéro de port | Utilisation |
   | --- | --- |
   | 80 | Téléchargement de la liste de révocation de certificats lors de la validation du certificat TLS/SSL |
   | 443 | Toutes les communications sortantes avec le service de proxy d’application |

Si votre pare-feu régule le trafic en fonction des utilisateurs d'origine, ouvrez également les ports 80 et 443 au trafic provenant des services Windows exécutés en tant que service réseau.

### <a name="allow-access-to-urls"></a>Autoriser l'accès à des URL

Autorisez l'accès aux URL suivantes :

| URL | Port | Utilisation |
| --- | --- | --- |
| &ast;.msappproxy.net<br>&ast;.servicebus.windows.net | 443/HTTPS | Communication entre le connecteur et le service cloud Proxy d'application |
| crl3.digicert.com<br>crl4.digicert.com<br>ocsp.digicert.com<br>crl.microsoft.com<br>oneocsp.microsoft.com<br>ocsp.msocsp.com<br> | 80/HTTP |Le connecteur utilise ces URL pour vérifier les certificats. |
| login.windows.net<br>secure.aadcdn.microsoftonline-p.com<br>&ast;.microsoftonline.com<br>&ast;.microsoftonline-p.com<br>&ast;.msauth.net<br>&ast;.msauthimages.net<br>&ast;.msecnd.net<br>&ast;.msftauth.net<br>&ast;.msftauthimages.net<br>&ast;.phonefactor.net<br>enterpriseregistration.windows.net<br>management.azure.com<br>policykeyservice.dc.ad.msft.net<br>ctldl.windowsupdate.com<br>www.microsoft.com/pkiops | 443/HTTPS |Le connecteur utilise ces URL lors du processus d'inscription. |
| ctldl.windowsupdate.com | 80/HTTP |Le connecteur utilise cette URL lors du processus d’inscription. |

Vous pouvez autoriser les connexions à &ast;.msappproxy.net, &ast;.servicebus.windows.net et les autres URL ci-dessus si votre pare-feu ou proxy vous permet de configurer des listes vertes DNS. Si ce n’est pas le cas, vous devez autoriser l’accès aux [Plages d’adresses IP et étiquettes des services Azure – Cloud public](https://www.microsoft.com/download/details.aspx?id=56519). Ces dernières sont mises à jour chaque semaine.

## <a name="install-and-register-a-connector"></a>Installer et inscrire un connecteur

Pour utiliser le service Proxy d’application, installez un connecteur sur chacun des serveurs Windows que vous utilisez avec le service Proxy d’application. Le connecteur est un agent qui gère la connexion sortante des serveurs d'applications locales vers le service Proxy d'applications d'Azure AD. Vous pouvez installer un connecteur sur des serveurs sur lesquels d'autres agents d'authentification, tels qu'Azure AD Connect, sont également installés.


Pour installer le connecteur :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu'administrateur d'application du répertoire qui utilise le service Proxy d'application. Par exemple, si le domaine du client est contoso.com, l'administrateur doit être admin@contoso.com ou tout autre alias administratif sur ce domaine.
1. Sélectionnez votre nom d’utilisateur en haut à droite. Vérifiez que vous êtes connecté à un répertoire qui utilise le service Proxy d’application. Si vous devez changer de répertoire, sélectionnez **Changer de répertoire** et choisissez un répertoire qui utilise le service Proxy d’application.
1. Dans le volet de navigation de gauche, sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Proxy d’application**.
1. Sélectionnez **Télécharger le service de connecteur**.

    ![Télécharger le service de connecteur pour voir les conditions d’utilisation du service](./media/application-proxy-add-on-premises-application/application-proxy-download-connector-service.png)

1. Lisez les conditions d'utilisation du service. Quand vous êtes prêt, sélectionnez **Accepter les conditions d’utilisation et télécharger**.
1. En bas de la fenêtre, sélectionnez **Exécuter** pour installer le connecteur. Un Assistant d'installation s'ouvre.
1. Suivez les instructions de l’Assistant pour installer le service. Lorsque vous êtes invité à inscrire le connecteur auprès du service Proxy d'application de votre client Azure AD, fournissez vos informations d'identification d'administrateur d'application.
    - Pour Internet Explorer (IE), si l'option **Configuration de sécurité renforcée d'Internet Explorer** est **Activée**, l'écran d'inscription peut ne pas s'afficher. Suivez les instructions du message d’erreur pour obtenir l’accès. Assurez-vous que l’option **Configuration de sécurité renforcée d’Internet Explorer** est **Désactivée**.

### <a name="general-remarks"></a>Remarques générales

Si vous avez déjà installé un connecteur, réinstallez-le pour obtenir la dernière version. Pour avoir des informations sur les versions précédentes et les changements qu’elles incluent, consultez [Proxy d’application : historique des versions](application-proxy-release-version-history.md).

Si vous choisissez d'utiliser plusieurs serveurs Windows pour vos applications locales, vous devez installer et inscrire le connecteur sur chaque serveur. Vous pouvez organiser les connecteurs en groupes de connecteurs. Pour plus d'informations, consultez [Groupes de connecteurs](application-proxy-connector-groups.md).

Si votre organisation utilise des serveurs proxy pour se connecter à Internet, vous devez les configurer pour le proxy d’application.  Pour plus d’informations, consultez [Utiliser des serveurs proxy locaux existants](application-proxy-configure-connectors-with-proxy-servers.md). 

Pour plus d'informations sur les connecteurs, la planification de capacité et leur mise à jour, consultez [Présentation des connecteurs du service Proxy d'application d'Azure AD](application-proxy-connectors.md).

## <a name="verify-the-connector-installed-and-registered-correctly"></a>Vérifier que le connecteur est correctement installé et inscrit

Vous pouvez utiliser le portail Azure ou votre serveur Windows pour vérifier qu'un nouveau connecteur est correctement installé.

### <a name="verify-the-installation-through-azure-portal"></a>Vérifier l’installation par le biais du portail Azure

Pour vérifier que le connecteur est correctement installé et inscrit :

1. Connectez-vous au répertoire de votre client sur le [portail Azure](https://portal.azure.com).
1. Dans le volet de navigation de gauche, sélectionnez **Azure Active Directory**, puis sélectionnez **Proxy d’application** sous la section **Gérer**. Tous les connecteurs et les groupes de connecteurs apparaissent sur cette page.
1. Affichez le connecteur pour vérifier les détails associés. En principe, les connecteurs sont développés par défaut. Si le connecteur qui vous intéresse n’est pas développé, développez-le pour afficher les détails associés. Une étiquette verte active indique que votre connecteur peut se connecter au service. Toutefois, même si l'étiquette est verte, un problème réseau peut empêcher le connecteur de recevoir les messages.

    ![Connecteurs de proxy d’application Azure AD](./media/application-proxy-add-on-premises-application/app-proxy-connectors.png)

Pour obtenir de l’aide sur l’installation d’un connecteur, consultez [Problèmes liés à l’installation du connecteur Proxy d’application](application-proxy-connector-installation-problem.md).

### <a name="verify-the-installation-through-your-windows-server"></a>Vérifier l’installation par le biais de votre serveur Windows

Pour vérifier que le connecteur est correctement installé et inscrit :

1. Ouvrez le Gestionnaire des services Windows en cliquant sur la touche **Windows** et en entrant *services.msc*.
1. Vérifiez que l'état des deux services suivants est **En cours d'exécution**.
   - Le **connecteur Proxy d’application Microsoft AAD** active la connectivité.
   - **Le programme de mise à jour du connecteur de proxy d’application Microsoft AAD** est un service de mise à jour automatique. Régulièrement, ce service recherche de nouvelles versions du connecteur et le met à jour si besoin.

     ![Services de connecteur de proxy d’application - capture d’écran](./media/application-proxy-add-on-premises-application/app_proxy_services.png)

1. Si l’état des services n’est pas **En cours d’exécution**, cliquez avec le bouton droit sur chaque service à sélectionner et choisissez **Démarrer**.

## <a name="add-an-on-premises-app-to-azure-ad"></a>Ajouter une application locale à Azure AD

Maintenant que vous avez préparé votre environnement et installé un connecteur, vous êtes prêt à ajouter des applications locales à Azure AD.  

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur.
2. Dans le volet de navigation de gauche, sélectionnez **Azure Active Directory**.
3. Sélectionnez **Applications d’entreprise**, puis **Nouvelle application**.
4. Sélectionnez le bouton **Ajouter une application locale** qui s’affiche à mi-chemin dans la page dans la section **Applications locales**. Vous pouvez également sélectionner **Créer votre propre application** en haut de la page, puis sélectionner **Configurer le proxy d’application pour un accès à distance sécurisé à une application locale**.
5. Dans la section **Ajouter votre propre application locale**, fournissez les informations suivantes relatives à votre application :

    | Champ | Description |
    | :---- | :---------- |
    | **Nom** | Nom de l’application qui apparaîtra dans Mes applications et sur le portail Azure. |
    | **URL interne** | URL permettant d'accéder à l'application depuis votre réseau privé. Vous pouvez spécifier un chemin spécifique sur le serveur principal à publier, alors que le reste du serveur n’est pas publié. De cette façon, vous pouvez publier des sites différents sur le même serveur en tant qu’applications distinctes et donner à chacun d’eux son propre nom et ses propres règles d’accès.<br><br>Si vous publiez un chemin d’accès, vérifiez qu’il inclut l’ensemble des images, des scripts et des feuilles de style nécessaires pour votre application. Par exemple, si votre application est sur https:\//yourapp/app et utilise des images situées dans https:\//yourapp/media, vous devez publier https:\//yourapp/ comme chemin d’accès. Cette URL interne n’est pas nécessairement la page d’accueil que vos utilisateurs voient. Pour plus d’informations, consultez [Définir une page d’accueil personnalisée pour les applications publiées](application-proxy-configure-custom-home-page.md). |
    | **URL externe** | Adresse permettant aux utilisateurs d'accéder à l'application de l'extérieur de votre réseau. Si vous ne souhaitez pas utiliser le domaine de proxy d’application par défaut, lisez la documentation relative aux [domaines personnalisés dans le proxy d’application Azure AD](application-proxy-configure-custom-domain.md).|
    | **Pré-authentification** | Façon dont le service Proxy d'application vérifie les utilisateurs avant de leur donner accès à votre application.<br><br>**Azure Active Directory** : le service Proxy d'application redirige les utilisateurs pour la connexion à Azure AD, qui authentifie leurs autorisations pour le répertoire et l'application. Nous vous recommandons de conserver cette option par défaut, qui vous permet d’utiliser les fonctionnalités de sécurité Azure AD, comme l’accès conditionnel et l’authentification multifacteur. **Azure Active Directory** est nécessaire pour la supervision de l’application avec Microsoft Cloud App Security.<br><br>**Direct** : les utilisateurs n’ont pas besoin de s’authentifier auprès d’Azure AD pour accéder à l’application. Vous pouvez toujours configurer les exigences d’authentification sur le serveur principal. |
    | **Groupe de connecteurs** | Les connecteurs gèrent l'accès à distance à votre application et les groupes de connecteurs vous aident à organiser des connecteurs et des applications par région, réseau ou objectif. Si aucun groupe de connecteurs n’est encore créé, votre application est assignée au groupe **Par défaut**.<br><br>Si votre application utilise des WebSockets pour se connecter, tous les connecteurs du groupe doivent être de la version 1.5.612.0 ou ultérieure.|

6. Si nécessaire, configurez des **paramètres supplémentaires**. Pour la plupart des applications, vous devez conserver ces paramètres dans leur état par défaut. 

    | Champ | Description |
    | :---- | :---------- |
    | **Expiration de l'application principale** | Définissez cette valeur sur **Long** uniquement si l'authentification et la connexion de votre application sont lentes. Par défaut, le délai d’expiration de l’application principale est de 85 secondes. Si cette valeur est trop importante, le délai d’expiration de l’application principale passe à 180 secondes. |
    | **Utiliser un cookie HTTP-only** | Définissez cette valeur sur **Oui** pour que les cookies du service Proxy d'application incluent un indicateur HTTPOnly dans l'en-tête de réponse HTTP. Si vous utilisez les services Bureau à distance, définissez cette valeur sur **Non**.|
    | **Utiliser un cookie sécurisé**| Définissez cette valeur sur **Oui** pour transmettre les cookies sur un canal sécurisé, comme une requête HTTPS chiffrée.
    | **Utiliser un cookie persistant**| Conservez cette valeur sur **Non**. Utilisez ce paramètre uniquement pour les applications qui ne peuvent pas partager de cookies entre les processus. Pour plus d’informations sur les paramètres de cookies, consultez [Paramètres de cookies pour l’accès aux applications locales dans Azure Active Directory](./application-proxy-configure-cookie-settings.md).
    | **Traduire l'URL dans les en-têtes** | Conservez la valeur **Oui**, sauf si votre application a demandé l'en-tête d'hôte d'origine dans la requête d'authentification. |
    | **Traduire les URL dans le corps de l'application** | Conservez la valeur **Non**, sauf si vous avez codé en dur des liens HTML vers d’autres applications locales et que vous n’utilisez pas de domaines personnalisés. Pour plus d’informations, consultez [Rediriger les liens codés en dur pour les applications publiées avec le Proxy d’application Azure AD](application-proxy-configure-hard-coded-link-translation.md).<br><br>Définissez cette valeur sur **Oui** si vous envisagez de superviser cette application avec Microsoft Cloud App Security (MCAS). Pour plus d’informations, consultez [Configurer la supervision de l’accès aux applications en temps réel avec Microsoft Cloud App Security et Azure Active Directory](application-proxy-integrate-with-microsoft-cloud-application-security.md). |

7. Sélectionnez **Ajouter**.

## <a name="test-the-application"></a>Test de l’application

Vous êtes prêt à tester que l’application a été correctement ajoutée. Dans les étapes suivantes, vous allez ajouter un compte d’utilisateur à l’application et essayer de vous connecter.

### <a name="add-a-user-for-testing"></a>Ajouter un utilisateur de test

Avant d'ajouter un utilisateur à l'application, vérifiez que le compte d'utilisateur dispose déjà des autorisations d'accès à l'application depuis le réseau de l'entreprise.

Pour ajouter un utilisateur de test :

1. Sélectionnez **Applications d’entreprise**, puis sélectionnez l’application à tester.
2. Sélectionnez **Mise en route**, puis **Assigner un utilisateur à des fins de test**.
3. Sous **Utilisateurs et groupes**, sélectionnez **Ajouter un utilisateur**.
4. Sous **Ajouter une attribution**, sélectionnez **Utilisateurs et groupes**. La section **Utilisateurs et groupes** apparaît.
5. Choisissez le compte que vous souhaitez ajouter.
6. Choisissez **Sélectionner**, puis sélectionnez **Attribuer**.

### <a name="test-the-sign-on"></a>Tester l'authentification

Pour tester l’authentification auprès de l’application :

1. À partir de l’application que vous souhaitez tester, sélectionnez **Proxy d’application**.
2. En haut de la page, sélectionnez **Tester l’application** pour exécuter un test sur l’application et vérifier les éventuels problèmes de configuration.
3. Veillez à lancer d’abord l’application pour tester la connexion à l’application, puis téléchargez le rapport de diagnostic pour consulter les conseils de dépannage en fonction des problèmes détectés.

Pour tout dépannage, consultez [Résoudre les problèmes de proxy d'application et les messages d'erreur](application-proxy-troubleshoot.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources créées dans ce tutoriel, supprimez-les.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez préparé votre environnement local pour utiliser le service Proxy d'application, puis vous avez installé et inscrit le connecteur du Proxy d'application. Ensuite, vous avez ajouté une application à votre locataire Azure AD. Vous avez vérifié qu’un utilisateur peut se connecter à l’application à l’aide d’un compte Azure AD.

Voici les étapes que vous avez effectuées :
> [!div class="checklist"]
> * Ouverture de ports pour le trafic sortant et autorisation d'accès à des URL spécifiques
> * Installation du connecteur sur votre serveur Windows et inscription de celui-ci auprès du service Proxy d'application
> * Vérification que le connecteur était correctement installé et inscrit
> * Ajout d'une application locale sur votre client Azure AD
> * Vérification qu’un utilisateur de test pouvait se connecter à l’application à l’aide d’un compte Azure AD

Vous êtes prêt à configurer l’application pour une authentification unique. Utilisez le lien suivant pour choisir une méthode d’authentification unique et trouver des tutoriels dédiés à l’authentification unique.

> [!div class="nextstepaction"]
> [Configurer l’authentification unique](sso-options.md#choosing-a-single-sign-on-method)
