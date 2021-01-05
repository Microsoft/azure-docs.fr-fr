---
title: Activer l’accès distant à Power BI à l’aide du Proxy d’application Azure AD
description: Aborde les notions de base relatives à l’intégration d’un serveur Power BI local au proxy d’application Azure AD.
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
ms.date: 07/25/2019
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro, has-adal-ref
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0c3188571e9188add7bc8f4f4d07ea5a562a79b3
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658109"
---
# <a name="enable-remote-access-to-power-bi-mobile-with-azure-ad-application-proxy"></a>Activer l’accès distant à Power BI Mobile avec le proxy d’application Azure AD

Cet article explique comment utiliser le proxy d’application Azure AD pour permettre à l’application mobile Power BI de se connecter à Power BI Report Server (PBIRS) et à SQL Server Reporting Services (SSRS) 2016 et versions ultérieures. Grâce à cette intégration, les utilisateurs qui se trouvent en dehors du réseau d’entreprise peuvent accéder à leurs rapports Power BI à partir de l’application mobile Power BI et être protégés par l’authentification Azure AD. Cette protection comprend des [avantages en matière de sécurité](application-proxy-security.md#security-benefits), comme l’accès conditionnel et l’authentification multifacteur.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous avez déjà déployé Report Services et [activé le proxy d’application](application-proxy-add-on-premises-application.md).

- L’activation du proxy d’application nécessite l’installation d’un connecteur sur un serveur Windows et l’exécution des [conditions préalables](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment) afin que le connecteur puisse communiquer avec les services Azure AD.
- Lors de la publication Power BI, nous vous recommandons d’utiliser les mêmes domaines interne et externe. Pour en savoir plus sur les domaines personnalisés, consultez [Utilisation des domaines personnalisés dans le proxy d’application](./application-proxy-configure-custom-domain.md).
- Cette intégration est disponible pour l’application **Power BI Mobile iOS et Android**.

## <a name="step-1-configure-kerberos-constrained-delegation-kcd"></a>Étape 1 : Configurer une délégation Kerberos contrainte (KCD)

Pour les applications locales qui utilisent l’authentification Windows, vous pouvez obtenir l’authentification unique (SSO) à l’aide du protocole d’authentification Kerberos et d’une fonctionnalité appelée délégation Kerberos contrainte (KCD). Lorsqu’elle est configurée, la délégation KCD permet au connecteur de proxy d’application d’obtenir un jeton Windows pour un utilisateur, même si ce dernier ne s’est pas connecté directement à Windows. Pour en savoir plus sur la délégation KCD, consultez [Présentation de la délégation Kerberos contrainte](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/jj553400(v=ws.11)) et [Délégation contrainte Kerberos pour l’authentification unique à vos applications avec le proxy d’application](application-proxy-configure-single-sign-on-with-kcd.md).

Il n’y a pas grand chose à configurer dans Reporting Services. Veillez simplement à disposer d’un nom de principal du service (SPN) pour permettre une authentification Kerberos appropriée. Assurez-vous également que le serveur Reporting Services est activé pour l’authentification par négociation.

Pour configurer la délégation KCD pour Reporting Services, effectuez les étapes suivantes.

### <a name="configure-the-service-principal-name-spn"></a>Configurer le nom de principal du service (SPN)

Le SPN est un identificateur unique pour un service qui utilise l’authentification Kerberos. Vous devez vous assurer que vous disposez d’un SPN HTTP approprié pour votre serveur de rapports. Pour plus d’informations sur la configuration du nom de principal du service (SPN) approprié pour votre serveur de rapports, consultez [Inscrire un nom de principal du service (SPN) pour un serveur de rapports](/sql/reporting-services/report-server/register-a-service-principal-name-spn-for-a-report-server).
Vous pouvez vérifier que le SPN a été ajouté en exécutant la commande Setspn avec l’option -L. Pour en savoir plus sur cette commande, consultez [Setspn](https://social.technet.microsoft.com/wiki/contents/articles/717.service-principal-names-spn-setspn-syntax.aspx).

### <a name="enable-negotiate-authentication"></a>Activer l’authentification par négociation

Pour permettre à un serveur de rapports d’utiliser l’authentification Kerberos, configurez le type d’authentification du serveur de rapports sur RSWindowsNegotiate. Configurez ce paramètre à l’aide du fichier rsreportserver.config.

```xml
<AuthenticationTypes>
    <RSWindowsNegotiate />
    <RSWindowsKerberos />
    <RSWindowsNTLM />
</AuthenticationTypes>
```

Pour plus d’informations, consultez [Modifier un fichier de configuration Reporting Services](/sql/reporting-services/report-server/modify-a-reporting-services-configuration-file-rsreportserver-config) et [Configurer l’authentification Windows sur un serveur de rapports](/sql/reporting-services/security/configure-windows-authentication-on-the-report-server).

### <a name="ensure-the-connector-is-trusted-for-delegation-to-the-spn-added-to-the-reporting-services-application-pool-account"></a>Vérifiez que le connecteur est approuvé pour la délégation pour le SPN ajouté au compte de pool d’applications Reporting Services
Configurez la délégation KCD afin que le service de proxy d’application Azure AD puisse déléguer des identités d’utilisateur au compte de pool d’applications Reporting Services. Pour configurer la délégation KCD, activez le connecteur de proxy d’application pour récupérer les tickets Kerberos des utilisateurs qui ont été authentifiés dans Azure AD. Puis ce serveur transmet le contexte à l’application cible, ou Reporting Services dans ce cas.

Pour configurer la délégation KCD, répétez les étapes suivantes pour chaque machine de connecteur :

1. Connectez-vous à un contrôleur de domaine en tant qu’administrateur de domaine, puis ouvrez **Utilisateurs et ordinateurs Active Directory**.
2. Trouvez l’ordinateur sur lequel le connecteur est en cours d’exécution.
3. Double-cliquez sur l’ordinateur, puis sélectionnez l’onglet **Délégation**.
4. Définissez les paramètres de délégation sur **N’approuver cet ordinateur que pour la délégation aux services spécifiés**. Sélectionnez ensuite **Utiliser tout protocole d’authentification**.
5. Sélectionnez **Ajouter**, puis **Utilisateurs ou ordinateurs**.
6. Entrez le compte de service que vous utilisez pour Reporting Services. Il s’agit du compte auquel vous avez ajouté le SPN dans la configuration de Reporting Services.
7. Cliquez sur **OK**. Cliquez à nouveau sur **OK** pour enregistrer les modifications.

Pour plus d’informations, consultez [Délégation contrainte Kerberos pour l’authentification unique à vos applications avec le proxy d’application](application-proxy-configure-single-sign-on-with-kcd.md).

## <a name="step-2-publish-report-services-through-azure-ad-application-proxy"></a>Étape 2 : Publier des services de rapports via le proxy d’application Azure AD

Vous êtes maintenant prêt à configurer un proxy d’application Azure AD.

1. Publiez des services de rapports via le proxy d’application avec les paramètres suivants. Pour obtenir des instructions pas à pas sur la publication d’une application via le proxy d’application, consultez [Publication d’applications à l’aide d’un proxy d’application Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad).
   - **URL interne** : Entrez l’URL du serveur de rapports auquel le connecteur peut accéder dans le réseau d’entreprise. Assurez-vous que cette URL est accessible à partir du serveur sur lequel le connecteur est installé. Il est conseillé d’utiliser un domaine de premier niveau, par exemple `https://servername/`, pour éviter les problèmes liés aux sous-chemins publiés via le proxy d’application. Par exemple, utilisez `https://servername/` et non `https://servername/reports/` ou `https://servername/reportserver/`.
     > [!NOTE]
     > Nous vous recommandons d’utiliser une connexion HTTPS sécurisée au serveur de rapports. Pour plus d’informations sur la marche à suivre, consultez [Configure SSL connections on a native mode report server](/sql/reporting-services/security/configure-ssl-connections-on-a-native-mode-report-server?view=sql-server-2017) (Configurer des connexions SSL sur un serveur de rapports en mode natif).
   - **URL externe** : Entrez l’URL publique à laquelle l’application mobile Power BI se connectera. Par exemple, elle peut prendre la forme `https://reports.contoso.com` si un domaine personnalisé est utilisé. Pour utiliser un domaine personnalisé, chargez un certificat pour le domaine et pointez un enregistrement DNS vers le domaine msappproxy.net par défaut de votre application. Pour les étapes détaillées, consultez [Utilisation des domaines personnalisés dans le proxy d’application Azure AD](application-proxy-configure-custom-domain.md).

   - **Méthode de pré-authentification** : Azure Active Directory

2. Une fois que votre application est publiée, configurez les paramètres d’authentification unique en effectuant les étapes suivantes :

   a. Dans la page de l’application dans le portail, sélectionnez **Authentification unique**.

   b. Pour le **mode d’authentification unique**, sélectionnez **Authentification Windows intégrée**.

   c. Définissez l’option **SPN d’application interne** sur la valeur que vous avez définie précédemment.

   d. Choisissez l’**Identité de connexion déléguée** pour le connecteur à utiliser pour le compte de vos utilisateurs. Pour plus d’informations, consultez [Utilisation d’identités cloud et locales différentes](application-proxy-configure-single-sign-on-with-kcd.md#working-with-different-on-premises-and-cloud-identities).

   e. Cliquez sur **Enregistrer** pour enregistrer vos modifications.

Pour terminer la configuration de votre application, accédez à la section **Utilisateurs et groupes** et affectez des utilisateurs pouvant accéder à cette application.

## <a name="step-3-modify-the-reply-uris-for-the-application"></a>Étape 3 : Modifier les URI de réponse pour l’application

Pour pouvoir connecter l’application mobile Power BI et accéder aux services de rapports, vous devez configurer l’inscription de l’application créée automatiquement à l’étape 2.

1. Dans la page de **vue d’ensemble d’Azure Active Directory**, sélectionnez **Inscriptions d’applications**.
2. Sous l’onglet **Toutes les applications**, recherchez celle que vous avez créée à l’étape 2.
3. Sélectionnez-la, puis sélectionnez **Authentification**.
4. Ajoutez les URI de redirection suivants selon la plateforme que vous utilisez.

   Quand vous configurez l’application pour Power BI Mobile **iOS**, ajoutez les URI de redirection suivants de type Client public (mobile et bureau) :
   - `msauth://code/mspbi-adal%3a%2f%2fcom.microsoft.powerbimobile`
   - `msauth://code/mspbi-adalms%3a%2f%2fcom.microsoft.powerbimobilems`
   - `mspbi-adal://com.microsoft.powerbimobile`
   - `mspbi-adalms://com.microsoft.powerbimobilems`

   Quand vous configurez l’application pour Power BI Mobile **Android**, ajoutez les URI de redirection suivants de type Client public (mobile et bureau) :
   - `urn:ietf:wg:oauth:2.0:oob`
   - `mspbi-adal://com.microsoft.powerbimobile`
   - `msauth://com.microsoft.powerbim/g79ekQEgXBL5foHfTlO2TPawrbI%3D`
   - `msauth://com.microsoft.powerbim/izba1HXNWrSmQ7ZvMXgqeZPtNEU%3D`

   > [!IMPORTANT]
   > Les URI de redirection doivent être ajoutés pour que l’application fonctionne correctement. Si vous configurez l’application pour Power BI Mobile iOS et Android, ajoutez l’URI de redirection suivant de type Client public (mobile et bureau) à la liste des URI de redirection configurés pour iOS : `urn:ietf:wg:oauth:2.0:oob`.

## <a name="step-4-connect-from-the-power-bi-mobile-app"></a>Étape 4 : Se connecter à partir de l’application Power BI Mobile

1. Dans l’application mobile Power BI, connectez-vous à votre instance Reporting Services. Pour ce faire, entrez l’**URL externe** de l’application que vous avez publiée par le biais du proxy d’application.

   ![Application mobile Power BI avec une URL externe](media/application-proxy-integrate-with-power-bi/app-proxy-power-bi-mobile-app.png)

2. Sélectionnez **Connecter**. Vous serez redirigé vers la page de connexion Azure Active Directory.

3. Entrez des informations d’identification valides pour votre utilisateur et sélectionnez **Se connecter**. Vous verrez les éléments de votre serveur Reporting Services.

## <a name="step-5-configure-intune-policy-for-managed-devices-optional"></a>Étape 5 : Configurer la stratégie Intune pour les appareils gérés (facultatif)

Vous pouvez utiliser Microsoft Intune pour gérer les applications clientes que le personnel de votre entreprise utilise. Intune vous permet d’utiliser des fonctionnalités telles que le chiffrement des données et les exigences d’accès supplémentaires. Pour en savoir plus sur la gestion des applications par le biais d’Intune, consultez Gestion des applications Intune. Pour que l’application mobile Power BI puisse fonctionner avec la stratégie Intune, procédez comme suit.

1. Allez dans **Azure Active Directory**, puis **Inscriptions d’applications**.
2. Sélectionnez l’application configurée à l’étape 3 lors de l’inscription de votre application cliente native.
3. Dans la page de l’application, sélectionnez **Autorisations de l’API**.
4. Cliquez sur **Ajouter une autorisation**.
5. Sous **API utilisées par mon organisation**, recherchez « Gestion des applications mobiles Microsoft », puis sélectionnez-la.
6. Ajoutez l’autorisation **DeviceManagementManagedApps.ReadWrite** à l’application
7. Cliquez sur **Accorder le consentement administrateur** pour accorder l’autorisation d’accès à l’application.
8. Configurez la stratégie Intune souhaitée en vous référant à [Comment créer et assigner des stratégies de protection des applications](/intune/app-protection-policies).

## <a name="troubleshooting"></a>Dépannage

Si l’application retourne une page d’erreur après avoir essayé de charger un rapport pendant plusieurs minutes, vous devrez peut-être modifier le paramètre du délai d’expiration. Par défaut, le proxy d’application prend en charge les applications qui mettent jusqu’à 85 secondes pour répondre à une requête. Pour allonger ce paramètre à 180 secondes, sélectionnez le délai d’expiration du serveur principal **Long** dans la page des paramètres du proxy d’application pour l’application. Pour obtenir des conseils sur la création de rapports rapides et fiables, consultez [Power BI Reports Best Practices](/power-bi/power-bi-reports-performance) (Meilleures pratiques de rapports Power BI).

## <a name="next-steps"></a>Étapes suivantes

- [Autoriser les applications clientes natives à interagir avec des applications de proxy](application-proxy-configure-native-client-application.md)
- [View on-premises report server reports and KPIs in the Power BI mobile apps](/power-bi/consumer/mobile/mobile-app-ssrs-kpis-mobile-on-premises-reports) (Afficher les rapports et indicateurs de performance clé de serveur de rapports local dans les applications mobiles Power BI)