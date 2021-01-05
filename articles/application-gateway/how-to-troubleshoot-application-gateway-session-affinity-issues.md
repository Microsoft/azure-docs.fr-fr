---
title: Résoudre les problèmes d’affinité de session
titleSuffix: Azure Application Gateway
description: Cet article fournit des informations sur la façon de résoudre des problèmes d’affinité de session dans Azure Application Gateway
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: troubleshooting
ms.date: 11/14/2019
ms.author: absha
ms.openlocfilehash: 548bda36ed2b167c159d32a575b63ecbf10b16dd
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397567"
---
# <a name="troubleshoot-azure-application-gateway-session-affinity-issues"></a>Résoudre des problèmes d’affinité de session dans Azure Application Gateway

Découvrez comment diagnostiquer et résoudre des problèmes d’affinité de session avec Azure Application Gateway.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="overview"></a>Vue d’ensemble

La fonctionnalité d’affinité de session basée sur les cookies est utile lorsque vous souhaitez conserver une session utilisateur sur le même serveur. En utilisant des cookies gérés de passerelle, la passerelle Application Gateway peut diriger le trafic pour traitement à partir d’une session utilisateur vers le même serveur. Ceci est important lorsque l’état de la session est enregistré localement sur le serveur pour une session utilisateur.

## <a name="possible-problem-causes"></a>Causes de problème possibles

Le problème du maintien de l’affinité de session basée sur des cookies peut se produire principalement pour les raisons suivantes :

- Le paramètre « Affinité basée sur les cookies » n’est pas activé.
- Votre application ne peut pas gérer l’affinité basée sur les cookies.
- L’application utilise une affinité basée sur les cookies, mais les demandes continuent de rebondir entre les serveurs principaux.

### <a name="check-whether-the-cookie-based-affinity-setting-is-enabled"></a>Vérifier si le paramètre « Affinité basée sur les cookies » est activé

Des problèmes d’affinité de session peuvent parfois se produire si vous oubliez d’activer le paramètre « Affinité basée sur les cookies ». Pour déterminer si vous avez l’activé sous l’onglet Paramètres HTTP dans le portail Azure, procédez comme suit :

1. Connectez-vous au [Portail Azure](https://portal.azure.com/).

2. Dans le **volet de navigation gauche** , cliquez sur **Toutes les ressources**. Cliquez sur le nom de la passerelle d’application dans le panneau Toutes les ressources. Si l’abonnement sélectionné comprend déjà plusieurs ressources, vous pouvez entrer le nom de la passerelle d’application dans la zone **Filtrer par nom...** pour accéder facilement à la passerelle d’application.

3. Sous **PARAMÈTRES** , sélectionnez **Paramètres HTTP**.

   ![Capture d’écran qui montre PARAMÈTRES avec les paramètres HTTP sélectionnés.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-1.png)

4. Cliquez sur **appGatewayBackendHttpSettings** sur le côté droit pour vérifier si vous avez sélectionné **Activé** pour l’option Affinité basée sur les cookies.

   ![Capture d’écran qui montre les paramètres d’une passerelle applicative, y compris si l’affinité basée sur les cookies est sélectionnée.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-2.jpg)



Vous pouvez également vérifier que la valeur de **CookieBasedAffinity** est définie sur *Activé* sous **backendHttpSettingsCollection** en utilisant l’une des méthodes suivantes :

- Exécutez [Get-AzApplicationGatewayBackendHttpSetting](/powershell/module/az.network/get-azapplicationgatewaybackendhttpsetting) dans PowerShell.
- Examinez le fichier JSON en utilisant le modèle Azure Resource Manager.

```
"cookieBasedAffinity": "Enabled", 
```

### <a name="the-application-cannot-handle-cookie-based-affinity"></a>L’application ne peut pas gérer l’affinité basée sur les cookies

#### <a name="cause"></a>Cause

Le passerelle d’application ne peut gérer l’affinité de session qu’en utilisant un cookie.

#### <a name="workaround"></a>Solution de contournement

Si l’application ne peut pas gérer l’affinité basée sur les cookies, vous devez utiliser un équilibreur de charge Azure interne ou externe ou une solution tierce.

### <a name="application-is-using-cookie-based-affinity-but-requests-still-bouncing-between-back-end-servers"></a>L’application utilise une affinité basée sur les cookies, mais les demandes continuent de rebondir entre les serveurs principaux.

#### <a name="symptom"></a>Symptôme

Si vous avez activé le paramètre Affinité basée sur les cookies, lorsque vous accédez à Application Gateway via une URL au nom court telle que `http://website` dans Internet Explorer, la requête continue de rebondir entre les serveurs principaux.

Pour identifier ce problème, suivez les instructions :

1. Prenez une trace de débogueur web sur le « Client » qui se connecte à l’application derrière Application Gateway (nous utilisons Fiddler dans cet exemple).
    **Conseil** Si vous ignorez comment utiliser Fiddler, activez l’option **I want to collect network traffic and analyze it using web debugger** (Je veux collecter le trafic réseau et l’analyser à l’aide du débogueur web) en bas de la page.

2. Vérifiez et analysez les journaux de session afin de déterminer si les cookies fournis par le client contiennent les détails ARRAffinity. Si vous ne trouvez pas les détails ARRAffinity, notamment **ARRAffinity=** *ARRAffinityValue* dans le jeu de cookies, cela signifie que le client ne répond pas avec le cookie ARRA fourni par Application Gateway.
    Par exemple :

    ![Capture d’écran qui montre un journal de session avec une entrée mise en évidence.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-3.png)

    ![Capture d’écran qui montre les en-têtes de requête pour HTTP, y compris les informations de cookie.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-4.png)

L’application continue d’essayer de définir le cookie sur chaque demande jusqu’à ce qu’elle reçoive une réponse.

#### <a name="cause"></a>Cause

Ce problème se produit parce que Internet Explorer et d’autres navigateurs ne peuvent pas stocker ou utiliser le cookie avec une URL au short name.

#### <a name="resolution"></a>Résolution

Pour résoudre ce problème, vous devez accéder à la passerelle d’application en utilisant un nom de domaine complet. Par exemple, utilisez [http://website.com](https://website.com/) ou [http://appgw.website.com](http://website.com/).

## <a name="additional-logs-to-troubleshoot"></a>Journaux supplémentaires pour résoudre des problèmes

Vous pouvez collecter des journaux supplémentaires et les analyser pour résoudre des problèmes liés à l’affinité de session basée sur les cookies.

### <a name="analyze-application-gateway-logs"></a>Analyser les journaux Application Gateway

Pour collecter les journaux Application Gateway, procédez comme suit :

Activation de la journalisation avec le portail Azure

1. Dans le [portail Azure](https://portal.azure.com/), recherchez votre ressource, puis cliquez sur **Journaux de diagnostic**.

   Pour Application Gateway, trois journaux d’activité d’audit sont disponibles : Journal d’accès, Journal des performances, Journal du pare-feu

2. Pour démarrer la collecte de données, cliquez sur **Activer les diagnostics**.

   ![Capture d’écran qui montre une passerelle applicative avec les journaux de diagnostic sélectionnés.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-5.png)

3. Le panneau **Paramètres de diagnostic** contient les paramètres des journaux de diagnostic. Dans cet exemple, Log Analytics stocke les journaux d’activité. Cliquez sur **Configurer** sous **Log Analytics** pour définir votre espace de travail. Vous pouvez également utiliser des concentrateurs d’événements et un compte de stockage pour enregistrer les journaux de diagnostic.

   ![Capture d’écran qui montre le volet Paramètres de diagnostic avec Log Analytics Configurer sélectionné.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-6.png)

4. Vérifiez les paramètres, puis cliquez sur **Enregistrer**.

   ![Capture d’écran qui montre le volet Paramètres de diagnostic avec l’option Enregistrer sélectionnée.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-7.png)

#### <a name="view-and-analyze-the-application-gateway-access-logs"></a>Afficher et analyser les journaux d’accès Application Gateway

1. Dans le portail Azure, sous l’affichage des ressources Application Gateway, dans la section **SURVEILLANCE** , sélectionnez **Journaux de diagnostic**.

   ![Capture d’écran qui montre SUPERVISION avec Journaux de diagnostic sélectionné.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-8.png)

2. Sur le côté droit, dans la liste déroulante sous **Catégories de journal** , sélectionnez **ApplicationGatewayAccessLog**.  

   ![Capture d’écran qui montre la liste déroulante des catégories de journaux avec ApplicationGatewayAccessLog sélectionné.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-9.png)

3. Dans la liste Journal d’accès d’Application Gateway, cliquez sur le journal que vous souhaitez analyser et exporter, puis exportez le fichier JSON.

4. Convertissez le fichier JSON que vous avez exporté à l’étape 3 en fichier CSV, puis affichez-le dans Excel, Power BI ou tout autre outil de visualisation de données.

5. Vérifiez les données suivantes :

- **ClientIP**  : adresse IP du client qui se connecte.
- **ClientPort**  : port source du client qui se connecte pour la demande.
- **RequestQuery**  : serveur de destination où la demande est reçue.
- **Acheminée par le serveur**  : instance de pool principal où la demande est reçue.
- **X-AzureApplicationGateway-LOG-ID**  : ID de corrélation utilisé pour la requête. Peut être utilisée pour résoudre les problèmes de trafic sur les serveurs principaux. Par exemple : X-AzureApplicationGateway-CACHE-HIT=0&SERVER-ROUTED=10.0.2.4.

  - **ÉTAT DU SERVEUR**  : code de réponse HTTP reçu par Application Gateway à partir du back-end.

  ![Capture d’écran qui montre l’état du serveur en texte brut, principalement masqué, avec clientPort et SERVER-ROUTED mis en évidence.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-11.png)

Si vous constatez que deux éléments proviennent des mêmes Adresse IP cliente et Port client et qu’ils sont envoyés au même serveur principal, cela signifie qu’Application Gateway a été configuré correctement.

Si vous constatez que deux éléments proviennent des mêmes Adresse IP cliente et Port client et qu’ils sont envoyés à des serveurs principaux différents, ce qui signifie que la demande rebondit entre les serveurs principaux, consultez la section **L’application utilise une affinité basée sur les cookies, mais les demandes continuent de rebondir entre les serveurs principaux**  plus bas pour résoudre le problème.

### <a name="use-web-debugger-to-capture-and-analyze-the-http-or-https-traffics"></a>Utiliser le débogueur web pour capturer et analyser les trafics HTTP ou HTTPS

Des outils de débogage web tels que Fiddler peuvent vous aider à déboguer des applications web en capturant le trafic réseau entre Internet et des ordinateurs de test. Ces outils vous permettent d’inspecter les données entrantes et sortantes à mesure que le navigateur les reçoit et les envoie. Dans cet exemple, Fiddler offre l’option de relecture HTTP qui peut vous aider à résoudre des problèmes d’applications web côté client, en particulier en lien avec l’authentification.

Utilisez le débogueur web de votre choix. Dans cet exemple, nous allons utiliser Fiddler pour capturer et analyser des trafics HTTP ou HTTPS. Procédez comme suit :

1. Téléchargez l’outil Fiddler à partir de la page <https://www.telerik.com/download/fiddler>.

    > [!NOTE]
    > Choisissez Fiddler4 si .NET 4 est installé sur l’ordinateur de capture. Sinon, choisissez Fiddler2.

2. Cliquez avec le bouton droit sur l’exécutable d’installation, puis exécutez-le en tant qu’administrateur pour effectuer l’installation.

    ![Capture d’écran qui montre le programme d’installation de l’outil Fiddler avec un menu contextuel avec l’option Exécuter en tant qu’administrateur sélectionnée.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-12.png)

3. Lorsque vous ouvrez Fiddler, l’application devrait démarrer automatiquement la capture du trafic (notez que vous pouvez voir l’opération en cours dans l’angle inférieur gauche). Appuyez sur F12 pour démarrer ou arrêter la capture du trafic.

    ![Capture d’écran qui montre le débogueur web Fiddler avec l’indicateur de capture mis en évidence.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-13.png)

4. Comme vous vous intéressez très probablement au trafic HTTPS déchiffré, vous pouvez activer le déchiffrement HTTPS en sélectionnant **Outils** > **Options Fiddler** , puis en activant la case **Déchiffrer le trafic HTTPS**.

    ![Capture d’écran qui montre les options de Fiddler avec HTTP sélectionné et Decrypt HTTPS traffic sélectionné.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-14.png)

5. Vous pouvez supprimer des sessions antérieures non liées avant de reproduire le problème en cliquant sur **X** (icône) > **Supprimer tout** comme dans la capture d’écran suivante : 

    ![Capture d’écran qui montre l’icône X sélectionnée, qui affiche l’option Remove all.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-15.png)

6. Une fois que vous avez reproduit le problème, enregistrez le fichier pour révision en sélectionnant **Fichier** > **Enregistrer** > **Toutes les Sessions...** . 

    ![Capture d’écran qui montre l’option File Save All Sessions sélectionnée.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-16.png)

7. Vérifiez et analysez les journaux de session pour déterminer la cause du problème.

    Exemples :

- **Exemple A :** Vous trouvez un journal de session indiquant que la demande est envoyée à partir du client à l’adresse IP publique d’Application Gateway. Cliquez sur ce journal pour afficher les détails.  Sur le côté droit, les données figurant dans la zone inférieure sont celles qu’Application Gateway renvoie au client. Sélectionnez l’onglet « RAW » pour déterminer si le client reçoit le cookie **Set-Cookie: ARRAffinity=** *ARRAffinityValue*. S’il n’y a aucun cookie, cela signifie que l’affinité de session n’est pas activée ou qu’Application Gateway de réapplique pas le cookie au client.

   > [!NOTE]
   > Cette valeur ARRAffinity est l’ID de cookie qu’Application Gateway définit pour le client, à envoyer à un serveur principal particulier.

   ![Capture d’écran qui montre un exemple de détails d’une entrée de journal avec la valeur Set-Cookie mise en évidence.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-17.png)

- **Exemple B :** Le prochain journal de session suivi par le précédent est le client qui répond à l’Application Gateway qui a défini la valeur ARRAAffinity. Si l’ID de cookie ARRAffinity correspond, le paquet doit être envoyé au serveur principal utilisé précédemment. Consultez les quelques lignes de communication HTTP suivantes pour voir si le cookie ARRAffinity du client est en train de changer.

   ![Capture d’écran qui montre un exemple de détails d’une entrée de journal avec un cookie mis en évidence.](./media/how-to-troubleshoot-application-gateway-session-affinity-issues/troubleshoot-session-affinity-issues-18.png)

> [!NOTE]
> Pour la même session de communication, le cookie ne devrait pas changer. Vérifiez la zone supérieure sur le côté droit, sélectionnez l’onglet « Cookies » pour voir si le client utilise le cookie et le renvoie à Application Gateway. Si ce n’est pas le cas, cela signifie que le navigateur client ne conserve et n’utilise pas le cookie pour les conversations. Parfois, le client peut mentir.

 

## <a name="next-steps"></a>Étapes suivantes

Si les étapes précédentes ne vous permettent pas de résoudre le problème, ouvrez un [ticket d’incident](https://azure.microsoft.com/support/options/).