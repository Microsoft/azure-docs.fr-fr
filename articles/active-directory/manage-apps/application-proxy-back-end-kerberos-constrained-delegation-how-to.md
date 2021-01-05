---
title: Détecter un problème de délégation Kerberos contrainte - Proxy d’application
description: Résolution des problèmes de configuration de la délégation Kerberos contrainte pour le proxy d’application
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 04/23/2019
ms.author: kenwith
ms.reviewer: asteen, japere
ms.openlocfilehash: c28e79c9a6f8c489a97d360c4fe142d431b5ab5d
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94656545"
---
# <a name="troubleshoot-kerberos-constrained-delegation-configurations-for-application-proxy"></a>Résolution des problèmes de configuration de la délégation Kerberos contrainte pour le proxy d’application

Les méthodes permettant d’utiliser l’authentification unique avec des applications publiées peuvent varier d’une application à une autre. L’une des options offertes par défaut par le proxy d’application Azure Active Directory (Azure AD) est la délégation Kerberos contrainte (KCD). Pour vos utilisateurs, vous pouvez configurer un connecteur permettant d'utiliser l'authentification Kerberos contrainte avec les applications principales.

La procédure d’activation de la délégation Kerberos contrainte est simple. Elle ne requiert qu'une compréhension générale des différents composants et flux d’authentification qui prennent en charge l’authentification unique. Mais parfois, l’authentification unique KCD ne fonctionne pas comme prévu. Vous devez alors disposer des bonnes sources d’informations pour résoudre ces scénarios.

Cet article fournit un point de référence unique qui peut vous aider à résoudre vous-même certains problèmes courants. Il explique également comment diagnostiquer des problèmes d'implémentation plus complexes.

Cet article repose sur les hypothèses suivantes :

- Le déploiement du proxy d’application Azure AD est conforme aux recommandations du document [Prise en main du proxy d'application](application-proxy-add-on-premises-application.md), et l’accès général aux applications non-KCD fonctionne comme prévu.
- L’application cible publiée est basée sur IIS (Internet Information Services) et sur l’implémentation de Kerberos par Microsoft.
- Les hôtes de serveur et d’application se trouvent dans un même domaine Azure Active Directory. Pour obtenir des informations détaillées sur les scénarios inter-domaines et inter-forêts, reportez-vous au [livre blanc sur la délégation Kerberos contrainte](https://aka.ms/KCDPaper).
- L’application est publiée dans un locataire Azure où la pré-authentification est activée. Les utilisateurs doivent s’authentifier auprès d'Azure via des formulaires d'authentification. Les scénarios d’authentification client enrichis ne sont pas couverts par cet article. Ils y seront peut-être ajoutés ultérieurement.

## <a name="prerequisites"></a>Prérequis

Le proxy d'application Azure AD peut être déployé dans de nombreux types d’infrastructures ou d'environnements. L'architecture varie d’une organisation à l’autre. Les causes les plus courantes des problèmes de délégation Kerberos contrainte (KCD) ne sont pas liées aux environnements. La plupart des problèmes sont dus à de simples erreurs de configuration ou à des erreurs générales.

Vous devez donc veiller à ce que tous les prérequis énumérés sous [Utilisation de l’authentification unique KCD avec le proxy d’application](application-proxy-configure-single-sign-on-with-kcd.md) soient respectés avant d'entamer la résolution de votre problème. Reportez-vous à la section relative à la configuration de la délégation Kerberos contrainte sous 2012 R2. Ce processus utilise une approche différente pour configurer la délégation Kerberos contrainte dans les versions antérieures de Windows. Soyez également attentif aux considérations suivantes :

- Il n’est pas rare qu’un serveur membre d'un domaine ouvre une boîte de dialogue sur un canal sécurisé avec un contrôleur de domaine spécifique. Le serveur peut se déplacer vers une autre boîte de dialogue à tout moment. Par conséquent, les hôtes de connecteur ne communiquent pas uniquement avec des contrôleurs de domaine de sites locaux spécifiques.
- Les scénarios inter-domaines s’appuient sur des références qui dirigent un hôte de connecteur vers des contrôleurs de domaine susceptibles de résider en dehors du périmètre du réseau local. Dans ce cas, il convient également de diriger le trafic vers les contrôleurs de domaine qui représentent les autres domaines respectifs, faute de quoi la délégation échouera.
- Si possible, évitez de placer des systèmes IPS ou IDS actifs entre les hôtes de connecteur et les contrôleurs de domaine. Ces appareils sont parfois trop intrusifs et interfèrent avec le trafic RPC principal.

Testez la délégation dans des scénarios simples. Plus vous introduisez de variables, plus vous augmentez le nombre d’éléments à prendre en compte. Pour gagner du temps, limitez votre test à un seul connecteur. Ajoutez des connecteurs supplémentaires une fois le problème résolu.

Certains facteurs environnementaux peuvent également contribuer à un problème. Pour écarter ces facteurs, réduisez l'architecture au minimum pendant le test. Par exemple, il est fréquent que les listes de contrôle d'accès (ACL) soient mal configurées sur les pare-feu internes. Si possible, envoyez directement le trafic d’un connecteur vers les contrôleurs de domaine et vers l’application principale.

Les connecteurs doivent être positionnés le plus près possible de leurs cibles. La présence d'un pare-feu entre eux lors du test accroît inutilement la complexité et peut prolonger vos investigations.

Quels sont les signes d'un problème de KCD ? Différents signes peuvent révéler un échec de l'authentification unique KCD. Les premiers signes d'un problème apparaissent dans le navigateur.

![Capture d’écran montrant un exemple d’erreur de configuration incorrecte de KCD, avec l’erreur « Incorrect Kerberos constrained delegation... » mise en évidence.](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic1.png)

![Exemple : Échec dû à des autorisations manquantes](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic2.png)

Ces deux illustrations présentent le même symptôme : un échec de l’authentification unique. L'accès à l’application est refusé à l'utilisateur.

## <a name="troubleshooting"></a>Dépannage

Le mode de résolution varie selon le problème et les symptômes observés. Avant de poursuivre, lisez les articles suivants. Ils fournissent des informations utiles pour résoudre les problèmes :

- [Résoudre les problèmes de proxy d’application et les messages d’erreur](application-proxy-troubleshoot.md)
- [Erreurs Kerberos](application-proxy-troubleshoot.md#kerberos-errors)
- [Utilisation de l’authentification unique lorsque les identités locales et cloud ne sont pas identiques](application-proxy-configure-single-sign-on-with-kcd.md#working-with-different-on-premises-and-cloud-identities)

Si vous êtes parvenu à ce stade, cela signifie que le principal problème subsiste. Pour commencer, séparez le flux en trois étapes distinctes afin de les résoudre l'une après l'autre.

### <a name="client-pre-authentication"></a>Pré-authentification du client

L’utilisateur externe s’authentifie auprès d'Azure via un navigateur. L’authentification unique KDC ne peut fonctionner que si la pré-authentification auprès d'Azure est possible. Vérifiez ce point et remédiez aux problèmes éventuels. L’étape de pré-authentification n’a aucun lien avec la délégation Kerberos contrainte ou avec l’application publiée. Les éventuelles incohérences peuvent être facilement corrigées en effectuant un contrôle de validité pour vérifier que le compte concerné existe dans Azure. Vérifiez également qu’il n’est pas désactivé ou bloqué. Le message d’erreur du navigateur est suffisamment descriptif pour permettre l’identification de la cause. En cas de doute, reportez-vous à d'autres articles Microsoft consacrés à la résolution de problèmes.

### <a name="delegation-service"></a>Service de délégation

Connecteur Azure Proxy qui se procure un ticket de service Kerberos pour les utilisateurs auprès d’un centre de distribution de clés Kerberos (KCD).

Les communications externes entre le client et le serveur frontal Azure n'ont aucune incidence sur la délégation Kerberos contrainte. Elles veillent uniquement au bon fonctionnement de la délégation Kerberos contrainte. Un identifiant utilisateur valide, qui sera utilisé pour l’obtention d’un ticket Kerberos, est fourni au service Azure Proxy. Sans cet identifiant, la délégation Kerberos contrainte est impossible et échoue.

Comme mentionné précédemment, les messages d’erreur du navigateur fournissent de bonnes indications sur les raisons d’un échec. Veillez à noter l’identifiant d’activité et le timestamp dans la réponse. Ces informations vous permettront de mettre le comportement en corrélation avec des événements réels dans le journal des événements Azure Proxy.

![Exemple : Erreur de configuration incorrecte de KCD](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic3.png)

Les entrées correspondantes du journal des événements apparaissent en tant qu'événement 13019 ou 12027. Vous trouverez les journaux des événements des connecteurs sous **Journaux des applications et des services** &gt; **Microsoft** &gt; **AadApplicationProxy** &gt; **Connecteur** &gt; **Admin**.

![Événement 13019 du journal des événements du proxy d’application](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic4.png)

![Événement 12027 du journal des événements du proxy d’application](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic5.png)

1. Utilisez un enregistrement **A** dans votre DNS interne pour l’adresse de l’application, et non un enregistrement **CName**.
1. Vérifiez de nouveau que l’hôte de connecteur dispose des droits nécessaires à la délégation au SPN du compte cible désigné. Vérifiez de nouveau que l'option **Utiliser tout protocole d’authentification** est sélectionnée. Pour plus d’informations, consultez [l’article consacré à la configuration de l’authentification unique](application-proxy-configure-single-sign-on-with-kcd.md).
1. Vérifiez qu’il n'existe qu'une seule instance du SPN dans Azure AD. Entrez `setspn -x` à partir d’une invite de commandes sur un hôte membre de domaine.
1. Vérifiez qu’une stratégie de domaine limitant la [taille maximale des jetons Kerberos émis](/archive/blogs/askds/maxtokensize-and-windows-8-and-windows-server-2012) est appliquée. En cas de taille excessive, cette stratégie empêche le connecteur d'obtenir un jeton.

Pour obtenir d’autres informations de base sur les problèmes rencontrés, l’étape suivante consiste à assurer un suivi réseau capturant les échanges entre l’hôte de connecteur et une délégation KDC du domaine. Pour plus d’informations, consultez le [document consacré à la résolution approfondie des problèmes](https://aka.ms/proxytshootpaper).

Si l’émission de tickets semble correcte, les journaux d’activité contiennent un événement mentionnant que l’authentification a échoué en raison du renvoi d’une erreur 401 par l’application. Cet événement indique que l’application cible a rejeté votre ticket. Passez à l’étape suivante.

### <a name="target-application"></a>Application cible

Consommateur du ticket Kerberos fourni par le connecteur. À ce stade, attendez-vous à ce que le connecteur ait envoyé un ticket de service Kerberos au serveur principal. Ce ticket correspond à un en-tête figurant dans la première demande d’application.

1. À l’aide de l’URL interne de l’application définie sur le portail, vérifiez que l’application est directement accessible à partir du navigateur sur l’hôte de connecteur. Vous pouvez alors vous connecter. Pour plus d’informations, reportez-vous à la page consacrée à la **résolution des problèmes** de connecteur.
1. Toujours sur l’hôte de connecteur, vérifiez que l’authentification entre le navigateur et l’application se fait à l’aide de Kerberos. Effectuez l'une des opérations suivantes :
1. Exécutez les outils de développement (**F12**) d'Internet Explorer, ou utilisez [Fiddler](https://blogs.msdn.microsoft.com/crminthefield/2012/10/10/using-fiddler-to-check-for-kerberos-auth/) à partir de l’hôte de connecteur. Accédez à l’application à l’aide de l’URL interne. Examinez les en-têtes d’autorisation WWW retournés dans la réponse de l’application pour vous assurer que Negotiate ou Kerberos est mentionné.

   - L’objet blob Kerberos suivant retourné dans la réponse du navigateur à l’application commence par **YII**. Ces lettres indiquent que Kerberos est en cours d'exécution. En revanche, NTLM (Microsoft NT LAN Manager) commence toujours par **TlRMTVNTUAAB**, soit NTLMSSP (NTLM Security Support Provider) lors d’un décodage en Base64. Si **TlRMTVNTUAAB** figure au début de l'objet blob, Kerberos n’est pas disponible. En revanche, si **TlRMTVNTUAAB** y figure, Kerberos est vraisemblablement disponible.

      > [!NOTE]
      > Si vous utilisez Fiddler, cette méthode nécessite de désactiver temporairement la protection étendue dans la configuration de l’application dans IIS.

      ![Fenêtre de contrôle de réseau du navigateur](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic6.png)

   - L’objet blob présenté dans cette illustration ne commence pas par **TIRMTVNTUAAB**. Par conséquent, dans cet exemple, Kerberos est disponible, et l’objet blob Kerberos ne commence pas par **YII**.

1. Supprimez temporairement NTLM de la liste des fournisseurs sur le site IIS. Accédez à l’application directement à partir d’Internet Explorer sur l’hôte de connecteur. NTLM ne figure plus dans la liste des fournisseurs. Vous ne pouvez accéder à l’application qu'à l’aide de Kerberos. Un échec de l'accès peut indiquer qu'il existe un problème de configuration de l’application. L’authentification Kerberos ne fonctionne pas.

   - Si Kerberos n’est pas disponible, vérifiez les paramètres d’authentification de l’application dans IIS. Assurez-vous que **Negotiate** figure en haut, avec NTLM juste en-dessous. En présence de **Not Negotiate**, **Kerberos or Negotiate** ou **PKU2U**, ne continuez que si Kerberos fonctionne.

     ![Fournisseurs d’authentification Windows](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic7.png)

   - Une fois Kerberos et NTLM en place, désactivez temporairement la pré-authentification de l’application sur le portail. Essayez d’y accéder sur Internet à l’aide de l’URL externe. Vous êtes invité à vous authentifier. Pour ce faire, vous pouvez utiliser le même compte qu'à l’étape précédente. Si l'authentification échoue, le problème vient de l'application principale, pas de la délégation Kerberos contrainte.
   - Réactivez la pré-authentification sur le portail. Authentifiez-vous auprès d'Azure en essayant de vous connecter à l’application à l’aide de son URL externe. Si l’authentification unique échoue, un message d’interdiction s'affiche dans le navigateur et l’événement 13022 apparaît dans le journal :

     *Le connecteur de proxy d’application Microsoft AAD ne peut pas authentifier l’utilisateur, car le serveur principal répond aux tentatives d’authentification Kerberos avec une erreur HTTP 401.*

      ![Affiche le message d’interdiction HTTP 401](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic8.png)

   - Vérifiez l’application IIS. Assurez-vous que le pool d’applications configuré et le nom principal du service (SPN) sont configurés pour utiliser le même compte dans Azure AD. Naviguez dans IIS comme indiqué dans l’illustration suivante :

      ![Fenêtre de configuration d’application IIS](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic9.png)

      Une fois l’identité connue, vérifiez que ce compte est configuré avec le SPN en question. par exemple `setspn –q http/spn.wacketywack.com`. Entrez le texte suivant dans une invite de commandes :

      ![Affiche la fenêtre de commandes SetSPN](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic10.png)

   - Comparez le SPN défini et les paramètres de l’application sur le portail. Assurez-vous que le SPN configuré sur le compte Azure AD cible est identique à celui utilisé par le pool d’applications.

      ![Configuration du SPN sur le portail Azure](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic11.png)

   - Accédez à IIS et sélectionnez l'option **Éditeur de configuration** de l’application. Accédez à **system.webServer/security/authentication/windowsAuthentication**. Assurez-vous que la valeur **UseAppPoolCredentials** est définie sur **True**.

      ![Option de configuration IIS - informations d’identification des pools d’applications](./media/application-proxy-back-end-kerberos-constrained-delegation-how-to/graphic12.png)

      Remplacez cette valeur par **True**. Supprimez tous les tickets Kerberos mis en cache sur le serveur principal en exécutant la commande suivante :

      ```powershell
      Get-WmiObject Win32_LogonSession | Where-Object {$_.AuthenticationPackage -ne 'NTLM'} | ForEach-Object {klist.exe purge -li ([Convert]::ToString($_.LogonId, 16))}
      ```

Pour plus d’informations, consultez l’article [Purge the Kerberos client ticket cache for all sessions (Purger le cache des tickets de client Kerberos pour toutes les sessions)](https://gallery.technet.microsoft.com/scriptcenter/Purge-the-Kerberos-client-b56987bf).

Si vous laissez le mode noyau activé, les opérations Kerberos seront plus performantes. Toutefois, le ticket du service demandé sera déchiffré à l’aide du compte d’ordinateur. Ce compte est également appelé système local. Définissez cette valeur sur **True** pour rompre la délégation Kerberos contrainte lorsque l’application est hébergée sur plusieurs serveurs au sein d'une batterie.

- Pour approfondir la vérification, désactivez également la protection **étendue**. Dans certains scénarios, la protection **étendue** rompt la délégation Kerberos contrainte lorsqu’elle est activée avec des configurations spécifiques. Dans ce cas, une application est publiée en tant que sous-dossier du site web par défaut. Cette application est configurée pour l’authentification anonyme uniquement. Toutes les boîtes de dialogue sont grisées, ce qui indique que les objets enfants n’héritent pas des paramètres actifs. Si possible, nous vous recommandons d'effectuer un test, mais n’oubliez pas de **réactiver** cette valeur.

  Cette vérification supplémentaire vous permet d'utiliser votre application publiée. Vous pouvez lancer des connecteurs supplémentaires également configurés pour la délégation. Pour plus d’informations, lisez la procédure technique approfondie [Résolution des problèmes de proxy d’application Azure AD](https://aka.ms/proxytshootpaper).

Si le problème persiste, contactez le support technique Microsoft. Émettez un ticket de support à partir du portail. Un ingénieur vous contactera.

## <a name="other-scenarios"></a>Autres scénarios

- Le proxy d’application Azure demande un ticket Kerberos avant d’envoyer sa demande à une application. Certaines applications tierces n’aiment pas cette méthode d’authentification. Ces applications préfèrent les négociations plus conventionnelles. La première demande est anonyme, ce qui permet à l’application de répondre avec les types d’authentification qu’elle prend en charge via une erreur 401. Ce type de négociation Kerberos peut être activé à l’aide des étapes décrites dans ce document : [Délégation contrainte Kerberos pour l’authentification unique](application-proxy-configure-single-sign-on-with-kcd.md).
- L'authentification à tronçons multiples est couramment utilisée dans les scénarios où une application est hiérarchisée, avec un serveur principal et un serveur frontal requérant tous les deux une authentification, comme SQL Server Reporting Services. Pour configurer le scénario à tronçons multiples, consultez l’article de support [La délégation Kerberos contrainte peut nécessiter une transition de protocole dans les scénarios à tronçons multiples](https://support.microsoft.com/help/2005838/kerberos-constrained-delegation-may-require-protocol-transition-in-mul).

## <a name="next-steps"></a>Étapes suivantes

[Configurer la délégation Kerberos contrainte sur un domaine géré](../../active-directory-domain-services/deploy-kcd.md).