---
title: Publier le bureau à distance avec le proxy d’application Azure Active Directory
description: Explique comment configurer le proxy d’application avec les Services Bureau à distance.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 11/30/2020
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: bfe8af8c30bbc2bc66c363fbd85f6764a48c28a1
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96488066"
---
# <a name="publish-remote-desktop-with-azure-ad-application-proxy"></a>Publier le Bureau à distance avec le proxy d’application Azure AD

Les Services Bureau à distance et le proxy d’application Azure AD fonctionnent ensemble pour améliorer la productivité des employés qui sont à l’extérieur du réseau d’entreprise. 

Le public concerné par cet article est le suivant :
- Clients actuels du proxy d’application qui souhaitent proposer plus d’applications à leurs utilisateurs finaux en publiant des applications locales via les Services Bureau à distance.
- Clients actuels des Services Bureau à distance qui souhaitent réduire la surface d’attaque de leur déploiement à l’aide du proxy d’application Azure AD. Ce scénario propose un ensemble de contrôles de vérification en deux étapes et d’accès conditionnel à RDS.

## <a name="how-application-proxy-fits-in-the-standard-rds-deployment"></a>Comment le proxy d’application s’intègre dans le déploiement RDS standard

Un déploiement RDS standard inclut divers services du rôle Bureau à distance s’exécutant sur Windows Server. [L’architecture des Services Bureau à distance](/windows-server/remote/remote-desktop-services/Desktop-hosting-logical-architecture) offre plusieurs options de déploiement. Contrairement à d’autres options de déploiement des Services Bureau à distance, le [déploiement des services Bureau à distance avec le proxy d’application Azure AD](/windows-server/remote/remote-desktop-services/Desktop-hosting-logical-architecture) (illustré dans le diagramme ci-dessous) dispose d’une connexion sortante permanente depuis le serveur exécutant le service du connecteur. Les autres déploiements comportent des connexions entrantes ouvertes via un équilibreur de charge.

![Le proxy d’application se situe entre la VM RDS et l’Internet public](./media/application-proxy-integrate-with-remote-desktop-services/rds-with-app-proxy.png)

Dans un déploiement RDS, le rôle Site Web Bureau à distance et le rôle Passerelle Bureau à distance s’exécutent sur des ordinateurs connectés à Internet. Ces points de terminaison sont exposés pour les raisons suivantes :
- Le rôle Site Web Bureau à distance fournit à l’utilisateur un point de terminaison public pour se connecter et afficher les applications locales et les postes de travail auxquels il peut se connecter. Lors de la sélection d’une ressource, une connexion RDP est créée à l’aide de l’application native sur le système d’exploitation.
- Le rôle Passerelle Bureau à distance apparaît lorsque l’utilisateur lance la connexion RDP. Ce rôle gère le trafic RDP chiffré provenant d’Internet et le convertit sur le serveur local auquel l’utilisateur est connecté. Dans ce scénario, le trafic reçu par la Passerelle Bureau à distance provient du proxy d’application Azure AD.

>[!TIP]
>Si vous n’avez pas déployé RDS au préalable, ou si vous souhaitez en savoir plus avant de commencer, découvrez comment [déployer RDS de manière transparente avec Azure Resource Manager et Azure Marketplace](/windows-server/remote/remote-desktop-services/rds-in-azure).

## <a name="requirements"></a>Spécifications

- Les points de terminaison du rôle Site Web Bureau à distance et du rôle Passerelle Bureau à distance doivent se trouver sur le même ordinateur et avoir une racine commune. Le rôle Site web Bureau à distance et le rôle Passerelle Bureau à distance sont publiés sous la forme d’une seule application avec le proxy d’application afin de vous proposer une expérience d’authentification unique pour les deux applications.

- Vous devez déjà avoir [déployé RDS](/windows-server/remote/remote-desktop-services/rds-in-azure) et [activé le proxy d’application](application-proxy-add-on-premises-application.md).

- Vos utilisateurs finaux doivent utiliser un navigateur compatible pour se connecter à Site Web Bureau à distance ou au client Site Web Bureau à distance. Pour plus d’informations, consultez [Prise en charge des configurations client](#support-for-other-client-configurations).

- Lors de la publication sur Site Web Bureau à distance, il est recommandé d’utiliser les mêmes noms de domaine complets interne et externe. Si les noms de domaine complets interne et externe sont différents, vous devez alors désactiver la traduction d’en-tête de requête pour éviter que le client reçoive des liens non valides.

- Si vous utilisez Site Web Bureau à distance sur Internet Explorer, vous devez activer le module complémentaire ActiveX RDS.

- Si vous utilisez le client Site Web Bureau à distance, vous devez utiliser le [connecteur version 1.5.1975 ou une version ultérieure](./application-proxy-release-version-history.md) de Proxy d’application.

- Pour le flux de préauthentification d’Azure AD, les utilisateurs peuvent se connecter uniquement aux ressources dont la publication est à leur disposition dans le volet **RemoteApp et Bureaux**. Les utilisateurs ne peuvent pas se connecter à un Bureau à l’aide du volet **Se connecter à un ordinateur distant**.

## <a name="deploy-the-joint-rds-and-application-proxy-scenario"></a>Déploiement du scénario associant RDS et proxy d’application

Après avoir configuré RDS et le proxy d’application Azure AD pour votre environnement, suivez les étapes permettant de combiner les deux solutions. Ces étapes expliquent comment publier les deux points de terminaison RDS orientés Web (Site Web Bureau à distance et Passerelle Bureau à distance) en tant qu’applications, puis diriger le trafic de votre RDS pour qu’il traverse le proxy d’application.

### <a name="publish-the-rd-host-endpoint"></a>Publication du point de terminaison hôte Bureau à distance

1. [Publiez une nouvelle application de proxy d’application](application-proxy-add-on-premises-application.md) avec les valeurs suivantes :
   - URL interne : `https://\<rdhost\>.com/`, où `\<rdhost\>` est la racine commune partagée par Site Web Bureau à distance et Passerelle Bureau à distance.
   - URL externe : Ce champ est automatiquement renseigné en fonction du nom de l’application, mais vous pouvez le modifier. Vos utilisateurs accéderont à cet URL en accédant à RDS.
   - Méthode de préauthentification : Azure Active Directory
   - Traduire l’URL dans les en-têtes : Non
2. Affectez des utilisateurs à l’application Bureau à distance publiée. Assurez-vous également qu’ils ont tous accès à RDS.
3. Conservez la méthode d’authentification unique de l’application **Authentification unique Azure AD désactivée**.

   >[!Note]
   >Les utilisateurs sont invités à s’authentifier une fois sur Azure AD et une fois sur Site Web Bureau à distance, mais profitent de l’authentification unique pour Passerelle Bureau à distance.

4. Sélectionnez **Azure Active Directory**, puis **Inscription des applications**. Choisissez votre application dans la liste.
5. Dans **Gérer**, sélectionnez **Personnalisation**.
6. Mettez à jour le champ **URL de la page d’accueil** pour pointer vers votre point de terminaison Site Web Bureau à distance (par ex., `https://\<rdhost\>.com/RDWeb`).

### <a name="direct-rds-traffic-to-application-proxy"></a>Trafic RDS direct vers le proxy d’application

Connectez-vous au déploiement RDS en tant qu’administrateur et modifiez le nom du serveur Passerelle Bureau à distance pour le déploiement. Cette configuration garantit que les connexions traversent le service du proxy d’application Azure AD.

1. Connectez-vous au serveur RDS exécutant le rôle Service Broker pour les connexions Bureau à distance.
2. Lancez le **Gestionnaire de serveur**.
3. Sélectionnez **Services Bureau à distance** dans le volet de gauche.
4. Sélectionnez **Vue d’ensemble**.
5. Dans la section Vue d’ensemble du déploiement, sélectionnez le menu déroulant et choisissez **Modifier les propriétés de déploiement**.
6. Dans l’onglet Passerelle Bureau à distance, modifiez le champ **Nom du serveur** en entrant l’URL externe définie pour le point de terminaison hôte Bureau à distance dans le proxy d’application.
7. Changez le champ **Méthode de connexion** en **Authentification par mot de passe**.

   ![Écran Propriétés de déploiement sur RDS](./media/application-proxy-integrate-with-remote-desktop-services/rds-deployment-properties.png)

8. Exécutez cette commande pour chaque collection. Remplacez *\<yourcollectionname\>* et *\<proxyfrontendurl\>* par vos propres informations. Cette commande active l’authentification unique entre Site Web Bureau à distance et Passerelle Bureau à distance, et optimise les performances :

   ```
   Set-RDSessionCollectionConfiguration -CollectionName "<yourcollectionname>" -CustomRdpProperty "pre-authentication server address:s:<proxyfrontendurl>`nrequire pre-authentication:i:1"
   ```

   **Par exemple :**
   ```
   Set-RDSessionCollectionConfiguration -CollectionName "QuickSessionCollection" -CustomRdpProperty "pre-authentication server address:s:https://remotedesktoptest-aadapdemo.msappproxy.net/`nrequire pre-authentication:i:1"
   ```
   >[!NOTE]
   >La commande ci-dessus utilise un accent grave dans « `nrequire ».

9. Pour vérifier la modification des propriétés RDP personnalisées et pour afficher le contenu du fichier RDP téléchargé à partir de RDWeb pour cette collection, exécutez la commande suivante :
    ```
    (get-wmiobject -Namespace root\cimv2\terminalservices -Class Win32_RDCentralPublishedRemoteDesktop).RDPFileContents
    ```

Maintenant que vous avez configuré le Bureau à distance, le proxy d’application Azure AD a repris le dessus comme composant accessible sur Internet de RDS. Vous pouvez supprimer les autres points de terminaison accessibles sur Internet de vos ordinateurs Site Web Bureau à distance et Passerelle Bureau à distance.

### <a name="enable-the-rd-web-client"></a>Activer le client Site Web Bureau à distance
Si vous souhaitez également que les utilisateurs puissent utiliser le client Site Web Bureau à distance, suivez les étapes de la section [Configurer le client Site Web Bureau à distance pour vos utilisateurs](/windows-server/remote/remote-desktop-services/clients/remote-desktop-web-client-admin) afin de l’activer.

Le client Site Web Bureau à distance permet aux utilisateurs d’accéder à l’infrastructure de Bureau à distance de votre organisation par le biais d’un navigateur web compatible avec HTML5 comme Microsoft Edge, Internet Explorer 11, Google Chrome, Safari ou Mozilla Firefox (v55.0 et versions ultérieures).

## <a name="test-the-scenario"></a>Test du scénario

Testez le scénario avec Internet Explorer sur un ordinateur Windows 7 ou 10.

1. Accédez à l’URL externe que vous configurez ou recherchez votre application dans le [panneau MyApps](https://myapps.microsoft.com).
2. Vous êtes invité à vous authentifier sur Azure Active Directory. Utilisez un compte de test que vous avez attribué à l’application.
3. Vous êtes invité à vous authentifier sur Site Web Bureau à distance.
4. Une fois authentifié sur RDS, vous pouvez sélectionner l’application ou le bureau de votre choix et commencer à travailler.

## <a name="support-for-other-client-configurations"></a>Prise en charge pour d’autres configurations client

La configuration décrite dans cet article concerne l’accès à RDS via Site Web Bureau à distance ou le client Site Web Bureau à distance. Toutefois, le cas échéant, vous pouvez prendre en charge d’autres systèmes d’exploitation et navigateurs. La différence réside dans la méthode d’authentification que vous utilisez.

| Méthode d'authentification | Configuration client prise en charge |
| --------------------- | ------------------------------ |
| Pré-authentification    | Site Web Bureau à distance : Windows 7/10 avec Internet Explorer* ou [le mode IE d’Edge Chromium](/deployedge/edge-ie-mode) + module complémentaire ActiveX RDS |
| Pré-authentification    | Client Site Web Bureau à distance : navigateur web compatible avec HTML5, tel que Microsoft Edge, Internet Explorer 11, Google Chrome, Safari ou Mozilla Firefox (v55.0 et versions ultérieures) |
| PassThrough | Tout autre système d’exploitation prenant en charge l’application Bureau à distance Microsoft |

\* Le mode IE de Edge Chromium est requis lorsque le portail Mes applications est utilisé pour accéder à l’application Bureau à distance.  

Le flux de pré-authentification offre plus d’avantages en matière de sécurité que le flux PassThrough. Avec la préauthentification, vous pouvez utiliser les fonctionnalités d’authentification Azure AD, telles que l’authentification unique, l’accès conditionnel et la vérification en deux étapes pour vos ressources locales. Vous garantissez également que seul le trafic authentifié atteint votre réseau.

Pour utiliser l’authentification PassThrough, seulement deux modifications doivent être apportées aux étapes répertoriées dans cet article :
1. Dans [Publication du point de terminaison hôte Bureau à distance](#publish-the-rd-host-endpoint), à l’étape 1, définissez la méthode de pré-authentification **PassThrough**.
2. Dans [Trafic RDS direct vers le proxy d’application](#direct-rds-traffic-to-application-proxy), ignorez l’étape 8 toute entière.

## <a name="next-steps"></a>Étapes suivantes
- [Activer l’accès distant pour SharePoint avec le proxy d’application Azure AD](application-proxy-integrate-with-sharepoint-server.md)
- [Considérations de sécurité pour l’accès aux applications à distance à l’aide du proxy d’application Azure AD](application-proxy-security.md)
- [Meilleures pratiques pour l’équilibrage de charge de plusieurs serveurs d’applications](application-proxy-high-availability-load-balancing.md#best-practices-for-load-balancing-among-multiple-app-servers)
