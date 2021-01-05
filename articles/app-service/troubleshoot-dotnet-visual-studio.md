---
title: Résoudre les problèmes à l’aide de Visual Studio
description: Apprenez à dépanner une application App Service à l'aide des outils de journalisation, de suivi et de débogage à distance intégrés à Visual Studio 2013.
ms.assetid: def8e481-7803-4371-aa55-64025d116c97
ms.devlang: dotnet
ms.topic: article
ms.date: 08/29/2016
ms.custom: devx-track-csharp, seodec18
ms.openlocfilehash: 346b1f83a9c18e35b009e88ae82d6984274fd4e4
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92147754"
---
# <a name="troubleshoot-an-app-in-azure-app-service-using-visual-studio"></a>Dépanner une application dans Azure App Service à l'aide de Visual Studio
## <a name="overview"></a>Vue d’ensemble
Ce didacticiel explique comment utiliser les outils Visual Studio pour déboguer une application dans [App Service](./overview.md) en activant le [Mode débogage](/visualstudio/debugger/) à distance ou en consultant les journaux des applications et des serveurs web.

Vous apprendrez ce qui suit :

* quelles sont les fonctions de gestion d'applications proposées par Visual Studio ;
* comment utiliser l'affichage à distance de Visual Studio pour apporter des modifications rapides dans une application à distance ;
* comment exécuter le mode débogage à distance lorsqu'un projet est exécuté dans Azure, pour une application comme pour une tâche web.
* la création de journaux d’activité de suivi d’application et leur affichage pendant leur création par l’application ;
* l’affichage des journaux d’activité de serveur Web, notamment des messages d’erreur détaillés et le suivi des demandes ayant échoué ;
* l’envoi de journaux de diagnostic à un compte Azure Storage et leur affichage depuis ce compte.

Si vous disposez de Visual Studio Ultimate, vous pouvez également utiliser [IntelliTrace](/visualstudio/debugger/intellitrace) à des fins de débogage. IntelliTrace n’est pas couvert dans ce didacticiel.

## <a name="prerequisites"></a><a name="prerequisites"></a>Configuration requise
Ce didacticiel fonctionne avec l'environnement de développement, le projet web et l'application App Service que vous avez configurés dans [Créer une application ASP.NET dans Azure App Service](quickstart-dotnet-framework.md). Pour les sections WebJobs, vous aurez besoin de l’application créée dans le cadre de la [Prise en main du Kit de développement logiciel (SDK) Azure WebJobs][GetStartedWJ].

Les exemples de code inclus dans ce didacticiel sont destinés à une application Web C# MVC, mais les procédures de résolution de problèmes sont identiques pour les applications Visual Basic et Web Forms.

Ce tutoriel suppose que vous utilisez Visual Studio 2019. 

La fonctionnalité de diffusion de journaux d’activité en continu est opérationnelle uniquement pour les applications ciblant .NET Framework 4 ou une version ultérieure.

## <a name="app-configuration-and-management"></a><a name="sitemanagement"></a>Configuration et gestion de l'application
Visual Studio permet d'accéder à un sous-ensemble des fonctionnalités de gestion des applications et des paramètres de configuration disponibles sur le [portail Azure](https://go.microsoft.com/fwlink/?LinkId=529715). Cette section présente les éléments disponibles en cas d’utilisation de **l’Explorateur de serveurs**. Pour connaître les dernières fonctionnalités d’intégration d’Azure, essayez également **Cloud Explorer** . Vous pouvez ouvrir les deux fenêtres à partir du menu **Affichage** .

1. Si vous n’êtes pas encore connecté à Azure dans Visual Studio, cliquez avec le bouton droit sur **Azure** et sélectionnez Se connecter à **Abonnement Microsoft Azure** dans **l’Explorateur de serveurs**.

    Vous pouvez aussi installer un certificat de gestion permettant d'accéder à votre compte. Si vous choisissez d’installer un certificat, cliquez avec le bouton droit sur le nœud **Azure** dans **l’Explorateur de serveurs**, puis sélectionnez **Gérer et filtrer les abonnements** dans le menu contextuel. Dans la boîte de dialogue **Gérer les abonnements Microsoft Azure**, cliquez sur l’onglet **Certificats**, puis sur **Importer**. Suivez la procédure pour télécharger et importer un fichier d’abonnement (portant l’extension *.publishsettings* ) pour votre compte Azure.

   > [!NOTE]
   > Si vous téléchargez un fichier d’abonnement, enregistrez-le dans un dossier situé hors de vos répertoires de code source (par exemple, dans le dossier Téléchargements), puis supprimez-le une fois l’importation terminée. Si un utilisateur malveillant accède au fichier d’abonnement, il peut modifier, créer et supprimer vos services Azure.
   >
   >

    Pour plus d’informations sur la connexion aux ressources Azure à partir de Visual Studio, consultez la page [Gérer des comptes, des abonnements et des rôles d’administrateur](../role-based-access-control/role-assignments-portal.md).
2. Dans l’**Explorateur de serveurs**, développez **Azure**, puis **App Service**.
3. Développez le groupe de ressources incluant l'application que vous avez créée dans [Créer une application ASP.NET dans Azure App Service](quickstart-dotnet-framework.md), puis cliquez avec le bouton droit sur le nœud d'application et sélectionnez **Afficher les paramètres**.

    ![Afficher les paramètres dans l'Explorateur de serveurs](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-viewsettings.png)

    L'onglet **Application web Azure** apparaît, affichant les tâches de gestion et de configuration des applications disponibles dans Visual Studio.

    ![Fenêtre Application web Microsoft Azure](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-configtab.png)

    Dans ce didacticiel, vous utiliserez les menus déroulants de suivi et de journalisation. Vous utiliserez également le débogage à distance en l'activant d'une façon différente.

    Pour plus d'informations sur les cases à cocher Paramètres de l'application et Chaînes de connexion de cette fenêtre, consultez l'article [Azure App Service: How Application Strings and Connection Strings Work](https://azure.microsoft.com/blog/windows-azure-web-sites-how-application-strings-and-connection-strings-work/) (Fonctionnement des chaînes d’application et de connexion d’Azure Web Apps).

    Pour effectuer une tâche de gestion d'application non exécutable dans cette fenêtre, cliquez sur **Ouvrir dans le portail de gestion** pour ouvrir une fenêtre de navigateur sur le portail Azure.

## <a name="access-app-files-in-server-explorer"></a><a name="remoteview"></a>Accès aux fichiers d'applications dans l'Explorateur de serveurs
En général, vous déployez un projet Web avec l’indicateur `customErrors` défini sur la valeur `On` ou `RemoteOnly` dans le fichier Web.config, ce qui signifie que vous n’obtenez pas de message d’erreur utile en cas de problème. Généralement, ces messages se présentent sous la forme d’une page comme celles qui suivent :

**Erreur de serveur dans l’application « / » :**

:::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

**Nous avons rencontré une erreur :**

:::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror1.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

**Le site Web ne peut pas afficher la page**

:::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/genericerror2.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

Généralement, la méthode la plus simple pour rechercher la cause d’une erreur consiste à activer les messages d’erreur détaillés : la première capture d’écran de la série précédente montre comment procéder. Vous devez modifier le fichier Web.config déployé. Vous pouvez modifier le fichier *Web.config* dans le projet et redéployer ce dernier, ou vous pouvez créer une [transformation Web.config](https://www.asp.net/mvc/tutorials/deployment/visual-studio-web-deployment/web-config-transformations) et déployer une build de débogage, mais la solution la plus rapide est la suivante : dans l'**Explorateur de solutions**, vous pouvez afficher et modifier directement les fichiers dans l'application distante à l'aide de la fonctionnalité d'*affichage à distance*.

1. Dans l'**Explorateur de serveurs**, développez l'élément **Azure**, **App Service**, puis le groupe de ressources où se trouve votre application, et développez le nœud de votre application.

    Vous voyez alors les nœuds qui vous donnent accès aux fichiers de contenu et aux fichiers journaux de l'application.
2. Développez le nœud **Fichiers** , puis double-cliquez sur le fichier *Web.config* .

    ![Ouvrir Web.config](./media/web-sites-dotnet-troubleshoot-visual-studio/webconfig.png)

    Visual Studio ouvre le fichier Web.config à partir de l'application distante, puis affiche l'élément [Distant] en regard du nom du fichier sur la barre de titre.
3. Ajoutez la ligne suivante à l’élément `system.web` :

    `<customErrors mode="Off"></customErrors>`

    ![Modifier Web.config](./media/web-sites-dotnet-troubleshoot-visual-studio/webconfigedit.png)
4. Actualisez le navigateur qui affiche le message d'erreur inutile : à présent, vous obtenez un message d'erreur détaillé, comme dans l'exemple suivant :

    ![Messages d’erreur détaillés](./media/web-sites-dotnet-troubleshoot-visual-studio/detailederror.png)

    L’erreur affichée est générée par l’ajout de la ligne affichée en rouge à *Views\Home\Index.cshtml*.

La modification du fichier Web.config n'est qu'un exemple des scénarios dans lesquels la capacité de lire et de modifier des fichiers sur votre application App Service simplifie la résolution des problèmes.

## <a name="remote-debugging-apps"></a><a name="remotedebug"></a>Applications de débogage à distance
Si le message d'erreur détaillé ne fournit pas assez d'informations et que vous ne pouvez pas recréer l'erreur localement, une autre méthode pour résoudre le problème consiste à exécuter le mode de débogage à distance. Vous pouvez définir des points d'arrêt, manipuler directement la mémoire, parcourir le code en détail et même modifier le chemin d'accès du code.

Le débogage à distance ne fonctionne pas avec les éditions Express de Visual Studio.

Cette section illustre comment déboguer à distance à l'aide du projet que vous créez dans [Créer une application ASP.NET dans Azure App Service](quickstart-dotnet-framework.md).

1. Ouvrez le projet web que vous avez créé dans [Créer une application ASP.NET dans Azure App Service](quickstart-dotnet-framework.md).

1. Ouvrez *Controllers\HomeController.cs*.

1. Supprimez la méthode `About()` et insérez le code suivant à la place.

    ```csharp
    public ActionResult About()
    {
        string currentTime = DateTime.Now.ToLongTimeString();
        ViewBag.Message = "The current time is " + currentTime;
        return View();
    }
    ```

1. [Définissez un point d’arrêt](/visualstudio/debugger/) sur la ligne `ViewBag.Message`.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis cliquez sur **Publier**.

1. Dans la liste déroulante **Profil**, sélectionnez le profil utilisé dans [Créer une application ASP.NET dans Azure App Service](quickstart-dotnet-framework.md). Puis, cliquez sur Paramètres.

1. Dans la boîte de dialogue **Publier**, cliquez sur l’onglet **Paramètres** et remplacez **Configuration** par **Déboguer**, puis cliquez sur **Enregistrer**.

    ![Publier en mode débogage](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-publishdebug.png)

1. Cliquez sur **Publier**. Au terme du déploiement, lorsque votre navigateur s'ouvre en affichant l'adresse URL Azure de votre application, fermez le navigateur.

1. Dans l'**Explorateur de serveurs**, cliquez avec le bouton droit sur votre application, puis cliquez sur **Attacher le débogueur**.

    :::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/tws-attachdebugger.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

    Le navigateur ouvre automatiquement votre page d'accueil exécutée dans Azure. Vous devrez peut-être attendre environ 20 secondes pendant qu'Azure configure le serveur pour le débogage. Ce retard se produit uniquement la première fois que vous exécutez le mode débogage sur une application au cours d'une période de 48 heures. Si vous commencez à déboguer à nouveau au cours de la même période, aucun retard ne se produit.

    > [!NOTE] 
    > Si vous rencontrez des problèmes au démarrage du débogueur, essayez d’utiliser **Cloud Explorer** au lieu de **l’Explorateur de serveurs**.
    >

1. Cliquez sur **À propos de** dans le menu.

    Visual Studio s'arrête sur le point d'arrêt et le code s'exécute dans Azure, et non pas sur votre ordinateur local.

1. Passez la souris sur la variable `currentTime` pour afficher la valeur de temps.

    ![Afficher une variable en mode débogage sur Azure](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugviewinwa.png)

    Le temps affiché correspond au fuseau horaire du serveur Azure, qui peut différer de celui de votre ordinateur local.

1. Entrez une nouvelle valeur pour la variable `currentTime` , comme « En cours d’exécution dans Azure ».

1. Appuyez sur F5 pour continuer.

     La page « À propos de » exécutée dans Azure affiche la nouvelle valeur entrée dans la variable currentTime.

     ![Page « À propos de » avec une nouvelle valeur](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugchangeinwa.png)

## <a name="remote-debugging-webjobs"></a><a name="remotedebugwj"></a> Débogage à distance de WebJobs
Cette section explique comment déboguer un élément à distance en utilisant le projet et l'application que vous avez créés avec le didacticiel [Prise en main du Kit de développement logiciel (SDK) Azure WebJobs](https://github.com/Azure/azure-webjobs-sdk/wiki).

Les fonctionnalités présentées dans cette section sont disponibles uniquement dans Visual Studio 2013 avec Update 4 ou version ultérieure.

Le débogage à distance fonctionne uniquement avec les tâches Web en continu. Les tâches Web planifiées et à la demande ne prennent pas en charge le débogage.

1. Ouvrez le projet Web que vous avez créé dans [Prise en main du Kit de développement logiciel (SDK) Azure WebJobs][GetStartedWJ].

2. Dans le projet ContosoAdsWebJob, ouvrez *Functions.cs*.

3. [Définissez un point d’arrêt](/visualstudio/debugger/) sur la première instruction dans la méthode `GnerateThumbnail`.

    ![Définir le point d’arrêt](./media/web-sites-dotnet-troubleshoot-visual-studio/wjbreakpoint.png)

4. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet web (pas le projet WebJobs), puis cliquez sur **Publier**.

5. Dans le menu déroulant **Profil** , sélectionnez le même profil que celui utilisé dans [Prise en main du Kit de développement logiciel (SDK) WebJobs Azure](https://github.com/Azure/azure-webjobs-sdk/wiki).

6. Cliquez sur l’onglet **Paramètres**, remplacez **Configuration** par **Debug**, puis cliquez sur **Publier**.

    Visual Studio déploie les projets de tâche web et web, et votre navigateur ouvre l'URL Azure de votre application.

7. Dans l'**Explorateur de serveurs**, développez **Azure > App Service > votre groupe de ressources > votre application > WebJobs > Continu**, puis cliquez avec le bouton droit sur **ContosoAdsWebJob**.

8. Cliquez sur **Attacher le débogueur**.

    :::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/wjattach.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

    Le navigateur ouvre automatiquement votre page d'accueil exécutée dans Azure. Vous devrez peut-être attendre environ 20 secondes pendant qu'Azure configure le serveur pour le débogage. Ce retard se produit uniquement la première fois que vous exécutez le mode débogage sur une application au cours d'une période de 48 heures. Si vous commencez à déboguer à nouveau au cours de la même période, aucun retard ne se produit.

9. Dans le navigateur web ouvert à la page d’accueil Contoso Ads, créez une publicité.

    La création d’une publicité provoque la création d’un message de file d’attente, qui est récupéré par la tâche web et traité. Quand le Kit SDK WebJobs appelle la fonction pour traiter le message de file d’attente, le code atteint votre point d’arrêt.

10. Quand le débogueur s’arrête au point d’arrêt, vous pouvez examiner et modifier les valeurs des variables pendant l’exécution du programme dans le cloud. Dans l’illustration suivante, le débogueur affiche le contenu de l’objet blobInfo qui a été passé à la méthode `GenerateThumbnail`.

     ![Objet blobInfo dans le débogueur](./media/web-sites-dotnet-troubleshoot-visual-studio/blobinfo.png)

11. Appuyez sur F5 pour continuer.

     La méthode `GenerateThumbnail` achève la création de la miniature.

12. Dans le navigateur, actualisez la page Index ; la miniature apparaît.

13. Dans Visual Studio, appuyez sur Maj+F5 pour arrêter le débogage.

14. Dans l’**Explorateur de serveurs**, cliquez avec le bouton droit sur le nœud ContosoAdsWebJob et cliquez sur **Afficher le tableau de bord**.

15. Connectez-vous avec vos informations d’identification Azure, puis cliquez sur le nom de la tâche web pour accéder à la page de votre tâche web.

     ![Cliquer sur ContosoAdsWebJob](./media/web-sites-dotnet-troubleshoot-visual-studio/clickcaw.png)

     Le tableau de bord indique que la fonction `GenerateThumbnail` a été exécutée récemment.

     (La prochaine fois que vous cliquez sur **Afficher le tableau de bord**, vous ne serez pas obligé de vous connecter et le navigateur accédera directement à la page de votre tâche Web).

16. Cliquez sur le nom de la fonction pour afficher les détails relatifs à son exécution.

     ![Détails de la fonction](./media/web-sites-dotnet-troubleshoot-visual-studio/funcdetails.png)

Si votre fonction [a écrit des journaux d’activité](https://github.com/Azure/azure-webjobs-sdk/wiki), vous pouvez cliquer sur **ToggleOutput** pour les afficher.

## <a name="notes-about-remote-debugging"></a>Notes à propos du débogage à distance

* Nous vous déconseillons d'exécuter le mode débogage en production. Si votre application de production n'est pas montée en charge sur plusieurs instances de serveur, la fonction de débogage empêche le serveur web de répondre aux autres demandes. Si vous avez plusieurs instances de serveurs web, le fait de joindre le débogueur génère une instance aléatoire et vous empêche de vérifier que les demandes suivantes de votre navigateur parviennent à la même instance. De même, comme vous ne déployez généralement pas une version de débogage dans un environnement de production, les optimisations du compilateur pour les versions Release peuvent empêcher l'affichage des réactions ligne par ligne dans votre code source. Pour résoudre les problèmes de production, la meilleure ressource est constituée des journaux d’activité de suivi d’application et de serveur Web.
* Évitez les arrêts longs aux points d'arrêt avec le débogage à distance. Azure considère qu'un processus arrêté pendant plus de quelques minutes ne répond pas, et l'arrête définitivement.
* Pendant le débogage, le serveur envoie des données à Visual Studio, ce qui peut affecter les frais de bande passante. Pour plus d'informations sur les tarifs de bande passante, consultez les [tarifs Azure](https://azure.microsoft.com/pricing/calculator/).
* Vérifiez que l’attribut `debug` de l’élément `compilation` du fichier *Web.config* est défini sur true. Il est défini sur true par défaut lorsque vous publiez une configuration de version de débogage.

    ```xml
    <system.web>
      <compilation debug="true" targetFramework="4.5" />
      <httpRuntime targetFramework="4.5" />
    </system.web>
    ```
* Si le débogueur ne parcourt pas le code que vous voulez déboguer, vous devez modifier le paramètre « Uniquement mon code ».  Pour plus d’informations, consultez la section [Specify whether to debug only user code using Just My Code in Visual Studio](/visualstudio/debugger/just-my-code) (Déterminer s’il faut déboguer uniquement le code utilisateur via l’option Uniquement mon code dans Visual Studio).
* Lorsque vous activez la fonctionnalité de débogage à distance, un compteur démarre sur le serveur : après 48 heures, la fonctionnalité est automatiquement désactivée. Cette limite de 48 heures a été définie à des fins de sécurité et de performances. Vous pouvez facilement réactiver la fonctionnalité autant de fois que nécessaire. Nous vous recommandons de la désactiver lorsque vous n'utilisez pas le débogage.
* Vous pouvez manuellement connecter le débogueur à n'importe quel processus, et pas seulement au processus de l'application (w3wp.exe). Pour plus d'informations sur l'utilisation du mode débogage dans Visual Studio, consultez la page [Débogage dans Visual Studio](/visualstudio/debugger/debugging-in-visual-studio).

## <a name="diagnostic-logs-overview"></a><a name="logsoverview"></a>Présentation des journaux de diagnostic
Une application ASP.NET qui s’exécute dans une application App Service peut créer les types de journaux d’activité suivants :

* **Journaux d’activité de suivi d’application**<br/>
  : l’application crée ces journaux d’activité en appelant des méthodes de la classe [System.Diagnostics.Trace](/dotnet/api/system.diagnostics.trace) .
* **Journaux d’activité des serveurs web**<br/>
  : le serveur web crée une entrée de journal pour chaque requête HTTP à destination de l'application.
* **Journaux d’activité détaillés des messages d’erreur**<br/>
  : le serveur Web crée une page HTML contenant certaines informations supplémentaires pour les requêtes HTTP ayant échoué (celles dont le code d’état est 400 ou plus).
* **Journaux d’activité de suivi de demandes ayant échoué**<br/>
  : le serveur Web crée un fichier XML avec des informations de suivi détaillées pour les requêtes HTTP ayant échoué. Le serveur Web fournit également un fichier XSL pour mettre en forme le XML dans un navigateur.

La fonction de journalisation affecte les performances des applications. Pour cette raison, Azure vous offre la possibilité d'activer ou de désactiver chaque type de journal, selon vos besoins. Vous pouvez définir un niveau minimal de gravité pour l’écriture des journaux d’activité d’application. Lorsque vous créez une application, la fonction de journalisation est désactivée par défaut.

Les journaux d’activité sont consignés dans des fichiers du dossier *LogFiles*, au sein du système de fichiers de votre application, et ils sont accessibles par FTP. Il est également possible d’écrire les journaux d’activité de serveur Web et d’application dans un compte Azure Storage. Vous pouvez conserver davantage de journaux d’activité sur un compte de stockage que sur le système de fichiers. Le système de fichiers peut conserver jusqu’à 100 Mo de journaux d’activité (il ne conserve pas les journaux d’activité très longtemps : Azure libère de l'espace pour les nouveaux fichiers journaux en supprimant les anciens une fois la limite atteinte).  

## <a name="create-and-view-application-trace-logs"></a><a name="apptracelogs"></a>Création et affichage des journaux d’activité de suivi d’application
Dans cette section, vous effectuerez les tâches suivantes :

* ajout d’instructions de traçage au projet web que vous avez créé dans [Prise en main d’Azure et ASP.NET](quickstart-dotnet-framework.md).
* affichage des journaux d’activité lorsque vous exécutez le projet localement ;
* affichage des journaux d’activité durant leur génération par l’application exécutée dans Azure.

Pour plus d’informations sur la création de journaux d’activité d’application dans WebJobs, consultez [Utilisation du stockage de file d’attente Microsoft Azure avec le Kit de développement logiciel (SDK) de WebJobs - Écriture de journaux d’activité](https://github.com/Azure/azure-webjobs-sdk/wiki). Les instructions suivantes permettant d’afficher les journaux d’activité et de contrôler la façon dont ils sont stockés dans Azure s’appliquent également aux journaux des applications créés par WebJobs.

### <a name="add-tracing-statements-to-the-application"></a>Ajout d'instructions de suivi à l'application
1. Ouvrez *Controllers\HomeController.cs*, puis remplacez les méthodes `Index`, `About` et `Contact` par le code suivant afin d’ajouter les instructions `Trace` et une instruction `using` pour `System.Diagnostics` :

    ```csharp
    public ActionResult Index()
    {
        Trace.WriteLine("Entering Index method");
        ViewBag.Message = "Modify this template to jump-start your ASP.NET MVC application.";
        Trace.TraceInformation("Displaying the Index page at " + DateTime.Now.ToLongTimeString());
        Trace.WriteLine("Leaving Index method");
        return View();
    }
    
    public ActionResult About()
    {
        Trace.WriteLine("Entering About method");
        ViewBag.Message = "Your app description page.";
        Trace.TraceWarning("Transient error on the About page at " + DateTime.Now.ToShortTimeString());
        Trace.WriteLine("Leaving About method");
        return View();
    }
    
    public ActionResult Contact()
    {
        Trace.WriteLine("Entering Contact method");
        ViewBag.Message = "Your contact page.";
        Trace.TraceError("Fatal error on the Contact page at " + DateTime.Now.ToLongTimeString());
        Trace.WriteLine("Leaving Contact method");
        return View();
    }        
    ```

1. Ajoutez une instruction `using System.Diagnostics;` en haut du fichier.

### <a name="view-the-tracing-output-locally"></a>Affichage de la sortie de suivi en local
1. Appuyez sur F5 pour exécuter l'application en mode débogage.

    L'écouteur de suivi par défaut écrit toutes les sorties de suivi dans la fenêtre **Sortie** , avec d'autres sorties de débogage. L’image suivante montre la sortie des instructions de suivi ajoutées à la méthode `Index` .

    ![Suivi dans la fenêtre Débogage](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-debugtracing.png)

    La procédure suivante montre comment afficher la sortie de suivi dans une page Web, sans procéder à la compilation en mode de débogage.
1. Ouvrez le fichier Web.config de l'application (celui situé dans le dossier de projet), puis ajoutez un élément `<system.diagnostics>` à la fin du fichier, juste avant l'élément de fermeture `</configuration>` :

    ``` xml
    <system.diagnostics>
    <trace>
      <listeners>
        <add name="WebPageTraceListener"
            type="System.Web.WebPageTraceListener,
            System.Web,
            Version=4.0.0.0,
            Culture=neutral,
            PublicKeyToken=b03f5f7f11d50a3a" />
      </listeners>
    </trace>
    </system.diagnostics>
    ```

L’élément `WebPageTraceListener` vous permet d’afficher la sortie de suivi en accédant à `/trace.axd`.
1. Ajoutez un <a href="/previous-versions/dotnet/netframework-4.0/6915t83k(v=vs.100)">élément de suivi</a> sous `<system.web>` dans le fichier Web.config, comme dans l’exemple suivant :

    ``` xml
    <trace enabled="true" writeToDiagnosticsTrace="true" mostRecent="true" pageOutput="false" />
    ```

1. Appuyez sur Ctrl+F5 pour exécuter l’application.
1. Dans la barre d’adresse du navigateur, ajoutez *trace.axd* à l’URL, puis appuyez sur Entrée (l’URL ressemble à `http://localhost:53370/trace.axd`).
1. Sur la page **Suivi d’application**, cliquez sur **Afficher les détails** sur la première ligne (pas la ligne BrowserLink).

    :::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/tws-traceaxd1.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

    La page **Détails de la demande** s’affiche et la section **Informations de suivi** affiche la sortie des instructions de suivi ajoutées à la méthode `Index`.

    :::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/tws-traceaxd2.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

    Par défaut, `trace.axd` est uniquement disponible localement. Si vous souhaitez le rendre disponible à partir d'une application distante, vous pouvez ajouter `localOnly="false"` à l'élément `trace` dans le fichier *Web.config*, comme le montre l'exemple suivant :

    ```xml
    <trace enabled="true" writeToDiagnosticsTrace="true" localOnly="false" mostRecent="true" pageOutput="false" />
    ```

    Toutefois, l'activation de `trace.axd` dans une application de production n'est pas recommandée pour des raisons de sécurité. Dans les sections suivantes, vous découvrirez une façon plus simple de lire les journaux d’activité de suivi dans une application App Service.

### <a name="view-the-tracing-output-in-azure"></a>Affichage de la sortie de suivi dans Azure
1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet web, puis cliquez sur **Publier**.
2. Dans la boîte de dialogue **Publier le site Web**, cliquez sur **Publier**.

    Une fois que Visual Studio a publié votre mise à jour, il ouvre une fenêtre de navigateur vers votre page d’accueil (en partant du principe que vous n’avez pas désactivé la case à cocher **URL de destination** sous l’onglet **Connexion**).
3. Dans l’**Explorateur de serveurs**, cliquez avec le bouton droit sur votre application et sélectionnez **Afficher les journaux d’activité de diffusion en continu**.

    :::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/tws-viewlogsmenu.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

    La fenêtre **Sortie** indique que vous êtes connecté au service de diffusion de journaux en continu et ajoute une ligne de notification à chaque minute passée sans affichage de journal.

    :::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/tws-nologsyet.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

4. Dans la fenêtre du navigateur qui affiche la page d'accueil de votre application, cliquez sur **Contacter**.

    En quelques secondes, la sortie du suivi de l’erreur que vous avez ajouté à la méthode `Contact` apparaît dans la fenêtre **Sortie**.

    ![Suivi d'erreur dans la fenêtre Sortie](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-errortrace.png)

    Visual Studio affiche uniquement le suivi des erreurs, car il s'agit du paramètre par défaut lorsque vous activez le service de surveillance des journaux. Lorsque vous créez une application App Service, la journalisation est désactivée par défaut, comme vous l'avez vu lorsque vous avez ouvert la page Paramètres un peu plus tôt :

    ![Journalisation d'application désactivée](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-apploggingoff.png)

    Cependant, lorsque vous avez sélectionné **Afficher les journaux d’activité de streaming**, Visual Studio a automatiquement remplacé **Journal des applications (système de fichiers)** par **Erreur**, ce qui signifie que les journaux d’activité d’erreur sont signalés. Pour afficher tous vos journaux d’activité de suivi, vous pouvez remplacer ce paramètre par **Commentaires**. Lorsque vous sélectionnez un niveau de gravité inférieur à l’erreur, tous les journaux d’activité correspondant aux niveaux de gravité supérieurs sont également signalés. Donc, lorsque vous sélectionnez Commentaires, vous pouvez également consulter des informations, des avertissements et des journaux d’activité d’erreurs.  

5. Dans l'**Explorateur de serveurs**, cliquez avec le bouton droit sur l'application, puis cliquez sur **Afficher les paramètres**, comme vous l'avez fait précédemment.
6. Remplacez **Journal des applications (Filesystem)** par **Commentaires**, puis cliquez sur **Enregistrer**.

    ![Définir le niveau de suivi sur Commentaires](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-applogverbose.png)
7. Dans la fenêtre du navigateur affichant votre page **Contact**, cliquez sur **Accueil**, sur **À propos de**, puis sur **Contact**.

    Quelques secondes après, la fenêtre **Sortie** affiche toutes vos sorties de suivi.

    ![Sortie de suivi des commentaires](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-verbosetraces.png)

    Dans cette section, vous avez activé et désactivé la journalisation à l'aide des paramètres d'application. Vous pouvez également activer et désactiver les écouteurs de suivi en modifiant le fichier Web.config. Toutefois, le fait de modifier le fichier Web.config entraîne le recyclage du domaine d'application, ce que ne fait pas l'activation de la journalisation via la configuration de l'application. Si le problème met du temps à se reproduire ou s'il est intermittent, le recyclage du domaine d'application peut le « résoudre » temporairement et vous forcer à attendre qu'il se reproduise. L’activation des diagnostics dans Azure vous permet de commencer tout de suite à saisir des informations sur une erreur sans recyclage du domaine d’application.

### <a name="output-window-features"></a>Fonctionnalités de la fenêtre Sortie
L’onglet **Journaux d’activité Microsoft Azure** de la fenêtre **Sortie** contient plusieurs boutons et une zone de texte :

:::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/tws-icons.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

Ces éléments permettent d'effectuer les opérations suivantes :

* Suppression du contenu de la fenêtre **Sortie**
* Activation ou désactivation du retour automatique à la ligne
* Démarrage ou arrêt de la supervision des journaux d’activité
* Sélection des journaux d’activité à surveiller
* Téléchargement de journaux d’activité
* Filtrage des journaux d’activité en fonction d’une chaîne de recherche ou d’une expression régulière
* Fermeture de la fenêtre **Sortie**

Si vous entrez une chaîne de recherche ou une expression régulière, Visual Studio filtre les informations de journalisation au niveau du client. Cela signifie que vous pouvez entrer les critères après l’affichage des journaux d’activité dans la fenêtre **Sortie** et que vous pouvez modifier les critères de filtrage sans avoir à régénérer les journaux d’activité.

## <a name="view-web-server-logs"></a><a name="webserverlogs"></a>Affichage des journaux d’activité de serveur Web
Les journaux d’activité de serveur Web enregistrent toutes les activités HTTP de l’application. Afin de pouvoir afficher ces journaux dans la fenêtre **Sortie**, vous devez les activer pour l'application et indiquer à Visual Studio que vous souhaitez les surveiller.

1. Dans l’onglet **Configuration de l’application web Azure** que vous avez ouvert à partir de l’**Explorateur de serveurs**, remplacez la valeur du paramètre Journalisation du serveur web par **Activé**, puis cliquez sur **Enregistrer**.

    ![Activer la journalisation de serveur Web](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-webserverloggingon.png)
2. Dans la fenêtre **Sortie**, cliquez sur le bouton **Spécifier les journaux d’activité Microsoft Azure à surveiller**.

    ![Sélection des journaux d’activité Azure à surveiller](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-specifylogs.png)
3. Dans la boîte de dialogue **Options de journalisation Microsoft Azure**, sélectionnez **Journaux d’activité du serveur web**, puis cliquez sur **OK**.

    ![Surveiller les journaux d’activité de serveur Web](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-monitorwslogson.png)
4. Dans la fenêtre du navigateur qui affiche l'application, cliquez sur **Accueil**, **À propos de**, puis **Contact**.

    Généralement, les journaux d’activité d’application s’affichent en premier, suivis par les journaux d’activité de serveur Web. Le processus d’affichage peut prendre un certain temps.

    ![Journaux d’activité de serveur Web dans la fenêtre Sortie](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-wslogs.png)

Par défaut, Azure écrit les journaux d’activité dans le système de fichiers lorsque vous activez les journaux d’activité de serveur Web en utilisant Visual Studio. Une autre méthode consiste à utiliser le portail Azure pour indiquer que les journaux d’activité de serveur web doivent être écrits sur un conteneur d’objets blob dans un compte de stockage.

Si vous utilisez le portail pour activer la journalisation de serveur web sur un compte de stockage Azure, puis désactivez la journalisation dans Visual Studio, les paramètres de votre compte de stockage sont restaurés lorsque vous réactivez la journalisation dans Visual Studio.

## <a name="view-detailed-error-message-logs"></a><a name="detailederrorlogs"></a>Affichage des journaux d’activité de messages d’erreur détaillés
Les journaux d’activité d’erreur détaillés fournissent des informations supplémentaires sur les requêtes HTTP ayant pour résultat des codes de réponse d’erreur (400 ou au-delà). Afin de pouvoir afficher ces journaux dans la fenêtre **Sortie** , vous devez les activer pour l'application et indiquer à Visual Studio que vous souhaitez les surveiller.

1. Dans l’onglet **Configuration de l’application web Azure** que vous avez ouvert à partir de l’**Explorateur de serveurs**, remplacez la valeur du paramètre **Messages d’erreur détaillés** par **Activé**, puis cliquez sur **Enregistrer**.

    ![Activer les messages d'erreur détaillés](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailedlogson.png)

2. Dans la fenêtre **Sortie**, cliquez sur le bouton **Spécifier les journaux d’activité Microsoft Azure à surveiller**.

3. Dans la boîte de dialogue **Options de journalisation Microsoft Azure**, cliquez sur **Tous les journaux d’activité**, puis sur **OK**.

    ![Surveiller tous les journaux d’activité](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-monitorall.png)

4. Dans la barre d’adresse de la fenêtre de navigateur, ajoutez un caractère supplémentaire à l’URL pour provoquer une erreur 404 (par exemple, `http://localhost:53370/Home/Contactx`) et appuyez sur Entrée.

    Après quelques secondes, le journal des erreurs détaillé s’affiche dans la fenêtre **Sortie** de Visual Studio.

    ![Journal des erreurs détaillé - fenêtre Sortie](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailederrorlog.png)

    Maintenez la touche Ctrl appuyée tout en cliquant sur le lien pour afficher le journal de sortie mis en forme dans un navigateur :

    ![Journal des erreurs détaillé - fenêtre du navigateur](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-detailederrorloginbrowser.png)

## <a name="download-file-system-logs"></a><a name="downloadlogs"></a>Téléchargement de journaux d’activité du système de fichiers
Tous les journaux d’activité que vous pouvez surveiller dans la fenêtre **Sortie** peuvent aussi être téléchargés en tant que fichiers *.zip* .

1. Dans la fenêtre **Sortie**, cliquez sur **Télécharger les journaux d’activité de diffusion en continu**.

    :::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/tws-downloadicon.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

    L'Explorateur de fichiers ouvre votre dossier *Téléchargements* dans lequel le fichier téléchargé est sélectionné.

    :::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/tws-downloadedfile.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

2. Procédez à l'extraction du fichier *.zip* pour afficher la structure de dossiers suivante :

    :::image type="content" source="./media/web-sites-dotnet-troubleshoot-visual-studio/tws-logfilefolders.png" alt-text="Capture d’écran montrant une erreur Erreur de serveur dans l’application « / » dans un navigateur web.":::

   * Les journaux d’activité de suivi d’application sont dans des fichiers *.txt* dans le dossier *LogFiles\Application*.
   * Les journaux d’activité de serveur web sont dans des fichiers *.log* dans le dossier *LogFiles\http\RawLogs*. Vous pouvez utiliser un outil tel que [Log Parser](https://www.microsoft.com/download/details.aspx?displaylang=en&id=24659) pour afficher et manipuler ces fichiers.
   * Les journaux d’activité de messages d’erreur détaillés sont dans des fichiers *.html* dans le dossier *LogFiles\DetailedErrors*.

     Le dossier *deployments* contient les fichiers créés par la publication à partir du contrôle de code source : il n'a rien à voir avec la publication Visual Studio. Le dossier *Git* contient le suivi lié à la publication à partir du contrôle de code source et au service de diffusion de fichier journal en continu.  

<!-- ## <a name="storagelogs"></a>View storage logs
Application tracing logs can also be sent to an Azure storage account, and you can view them in Visual Studio. To do that you'll create a storage account, enable storage logs in the Azure portal, and view them in the **Logs** tab of the **Azure Web App** window.

You can send logs to any or all of three destinations:

* The file system.
* Storage account tables.
* Storage account blobs.

You can specify a different severity level for each destination.

Tables make it easy to view details of logs online, and they support streaming; you can query logs in tables and see new logs as they are being created. Blobs make it easy to download logs in files and to analyze them using HDInsight, because HDInsight knows how to work with blob storage. For more information, see **Hadoop and MapReduce** in [Data Storage Options (Building Real-World Cloud Apps with Azure)](https://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/data-storage-options).

You currently have file system logs set to verbose level; the following steps walk you through setting up information level logs to go to storage account tables. Information level means all logs created by calling `Trace.TraceInformation`, `Trace.TraceWarning`, and `Trace.TraceError` will be displayed, but not logs created by calling `Trace.WriteLine`.

Storage accounts offer more storage and longer-lasting retention for logs compared to the file system. Another advantage of sending application tracing logs to storage is that you get some additional information with each log that you don't get from file system logs.

1. Right-click **Storage** under the Azure node, and then click **Create Storage Account**.

![Create Storage Account](./media/web-sites-dotnet-troubleshoot-visual-studio/createstor.png)

1. In the **Create Storage Account** dialog, enter a name for the storage account.

    The name must be must be unique (no other Azure storage account can have the same name). If the name you enter is already in use you'll get a chance to change it.

    The URL to access your storage account will be *{name}*.core.windows.net.
2. Set the **Region or Affinity Group** drop-down list to the region closest to you.

    This setting specifies which Azure datacenter will host your storage account. For this tutorial your choice won't make a noticeable difference, but for a production web app you want your web server and your storage account to be in the same region to minimize latency and data egress charges. The web app (which you'll create later) should run in a region as close as possible to the browsers accessing your web app in order to minimize latency.
3. Set the **Replication** drop-down list to **Locally redundant**.
   
    When geo-replication is enabled for a storage account, the stored content is replicated to a secondary datacenter to enable failover to that location in case of a major disaster in the primary location. Geo-replication can incur additional costs. For test and development accounts, you generally don't want to pay for geo-replication. For more information, see [Create, manage, or delete a storage account](../storage/common/storage-account-create.md).
4. Click **Create**.

    ![New storage account](./media/web-sites-dotnet-troubleshoot-visual-studio/newstorage.png)    
5. In the Visual Studio **Azure Web App** window, click the **Logs** tab, and then click **Configure Logging in Management Portal**.

     <!-- todo:screenshot of new portal if the VS page link goes to new portal -- >
    ![Configure logging](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-configlogging.png)

    This opens the **Configure** tab in the portal for your web app.
6. In the portal's **Configure** tab, scroll down to the application diagnostics section, and then change **Application Logging (Table Storage)** to **On**.
7. Change **Logging Level** to **Information**.
8. Click **Manage Table Storage**.

    ![Click Manage TableStorage](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-stgsettingsmgmtportal.png)

    In the **Manage table storage for application diagnostics** box, you can choose your storage account if you have more than one. You can create a new table or use an existing one.

    ![Manage table storage](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-choosestorageacct.png)
9. In the **Manage table storage for application diagnostics** box, click the check mark to close the box.
10. In the portal's **Configure** tab, click **Save**.
11. In the browser window that displays the application web app, click **Home**, then click **About**, and then click **Contact**.

     The logging information produced by browsing these web pages is written to the storage account.
12. In the **Logs** tab of the **Azure Web App** window in Visual Studio, click **Refresh** under **Diagnostic Summary**.

     ![Click Refresh](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-refreshstorage.png)

     The **Diagnostic Summary** section shows logs for the last 15 minutes by default. You can change the period to see more logs.

     (If you get a "table not found" error, verify that you browsed to the pages that do the tracing after you enabled **Application Logging (Storage)** and after you clicked **Save**.)

     ![Storage logs](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-storagelogs.png)

     Notice that in this view you see **Process ID** and **Thread ID** for each log, which you don't get in the file system logs. You can see additional fields by viewing the Azure storage table directly.
13. Click **View all application logs**.

     The trace log table appears in the Azure storage table viewer.

     (If you get a "sequence contains no elements" error, open **Server Explorer**, expand the node for your storage account under the **Azure** node, and then right-click **Tables** and click **Refresh**.)

     ![Storage logs in table view](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-tracelogtableview.png)

     This view shows additional fields you don't see in any other views. This view also enables you to filter logs by using special Query Builder UI for constructing a query. For more information, see Working with Table Resources - Filtering Entities in [Browsing Storage Resources with Server Explorer](https://msdn.microsoft.com/library/ff683677.aspx).
14. To look at the details for a single row, double-click one of the rows.

     ![Trace table in Server Explorer](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-tracetablerow.png)
 -->
## <a name="view-failed-request-tracing-logs"></a><a name="failedrequestlogs"></a>Affichage des journaux d’activité de suivi de demandes ayant échoué
Les journaux d’activité de suivi de demandes ayant échoué sont utiles pour comprendre les détails de gestion IIS d’une requête HTTP, dans des scénarios tels que des problèmes de réécriture d’URL ou d’authentification.

Les applications App Service utilisent la même fonctionnalité de suivi de demandes ayant échoué que celle que proposait IIS 7.0 et versions ultérieures. Cependant, vous ne pouvez pas accéder aux paramètres IIS permettant de configurer les erreurs à journaliser. Lorsque vous activez le suivi des demandes ayant échoué, toutes les erreurs sont récupérées.

Vous pouvez activer le suivi des demandes ayant échoué en utilisant Visual Studio, mais vous ne pouvez pas les afficher dans Visual Studio. Ces journaux d’activité sont des fichiers XML. Le service de journaux de streaming surveille uniquement les fichiers jugés lisibles en mode texte brut : fichiers *.txt*, *.html* et *.log*.

Vous pouvez afficher les journaux d’activité de suivi des demandes ayant échoué directement dans un navigateur via FTP ou en local, après avoir utilisé un outil FTP pour les télécharger sur votre ordinateur local. Dans cette section, nous les afficherons directement dans un navigateur.

1. Dans l’onglet **Configuration** de la fenêtre **Application web Microsoft Azure** que vous avez ouverte à partir de l’**Explorateur de serveurs**, remplacez la valeur du paramètre **Suivi des demandes ayant échoué** par **Activé**, puis cliquez sur **Enregistrer**.

    ![Activer le suivi des demandes ayant échoué](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-failedrequeston.png)
2. Sur la barre d'adresses de la fenêtre de navigateur qui affiche l'application, ajoutez un caractère supplémentaire à l'URL et appuyez sur Entrée pour provoquer une erreur 404.

    Ceci génère un journal de suivi des demandes ayant échoué. La procédure suivante vous montre comment afficher ou télécharger ce journal.

3. Dans l’onglet **Configuration** de la fenêtre **Application web Microsoft Azure** de Visual Studio, cliquez sur **Ouvrir l’élément dans le portail de gestion**.

4. Sur le [Portail Azure](https://portal.azure.com), dans la page **Paramètres** de votre application, cliquez sur **Informations d’identification du déploiement**, puis entrez un nouveau nom d’utilisateur et un nouveau mot de passe.

    ![Nouveaux nom d'utilisateur et mot de passe FTP](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-enterftpcredentials.png)

    > [!NOTE]
    > Lorsque vous vous connectez, vous devez utiliser le nom d'utilisateur complet, avec pour préfixe le nom de l'application. Par exemple, si vous entrez « monid » comme nom d’utilisateur et si le site est « monexemple », vous vous connectez en tant que « monexemple\monid ».
    >

5. Dans une nouvelle fenêtre de navigateur, accédez à l'URL affichée sous **Nom d'hôte FTP** ou **Nom d'hôte FTPS** sur la page **Vue d'ensemble** de votre application.

6. Connectez-vous en utilisant les informations d'identification FTP que vous avez créées précédemment (incluant le nom de l'application en tant que préfixe pour le nom d'utilisateur).

    Le navigateur affiche le dossier racine de l'application.

7. Ouvrez le fichier *LogFiles* .

    ![Ouvrir le dossier LogFiles](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-logfilesfolder.png)

8. Ouvrez le dossier nommé W3SVC avec une valeur numérique.

    ![Ouvrir le dossier W3SVC](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-w3svcfolder.png)

    Ce dossier contient des fichiers XML pour plusieurs erreurs journalisées une fois le suivi des demandes ayant échoué activé, ainsi qu'un fichier XSL utilisable par un navigateur pour mettre en forme les fichiers XML.

    ![Dossier W3SVC](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-w3svcfoldercontents.png)

9. Cliquez sur le fichier XML de la demande ayant échoué dont vous voulez consulter les informations de suivi.

    L'image suivante montre une partie des informations de suivi d'un exemple d'erreur.

    ![Suivi d'une demande ayant échoué dans le navigateur](./media/web-sites-dotnet-troubleshoot-visual-studio/tws-failedrequestinbrowser.png)

## <a name="next-steps"></a><a name="nextsteps"></a>Étapes suivantes
Vous avez vu en quoi Visual Studio facilite l’affichage des journaux d’activité créés par une application App Service. Les sections suivantes fournissent des liens vers des ressources supplémentaires sur des rubriques connexes :

* Résolution des problèmes App Service
* Débogage dans Visual Studio
* Débogage distant dans Azure
* Suivi dans les applications ASP.NET
* Analyse de journaux d’activité de serveur Web
* Analyse des journaux d’activité de suivi des demandes ayant échoué
* Débogage de Cloud Services

### <a name="app-service-troubleshooting"></a>Résolution des problèmes App Service
Pour en savoir plus sur la résolution des problèmes liés aux applications dans Azure App Service, consultez les ressources suivantes :

* [Surveiller les applications](web-sites-monitor.md)
* [Étude des fuites de mémoire dans Azure App Service avec Visual Studio 2013](https://devblogs.microsoft.com/devops/investigating-memory-leaks-in-azure-web-sites-with-visual-studio-2013/). Article sur le blog ALM de Microsoft concernant les fonctionnalités de Visual Studio prévues pour l'analyse de problèmes de mémoire gérés.
* [Les outils en ligne d'Azure App Service que vous devez connaître](https://azure.microsoft.com/blog/2014/03/28/windows-azure-websites-online-tools-you-should-know-about-2/). Article de blog par Amit Apple.

Pour obtenir des réponses sur une question relative à la résolution des problèmes, ouvrez un fil de discussion dans l'un des forums suivants :

* [Forum Azure sur le site ASP.NET](https://forums.asp.net/1247.aspx/1?Azure+and+ASP+NET).
* [Forum Azure sur Microsoft Q&A](/answers/topics/azure-webapps.html).
* [StackOverflow.com](https://www.stackoverflow.com).

### <a name="debugging-in-visual-studio"></a>Débogage dans Visual Studio
Pour plus d’informations sur l’utilisation du mode débogage dans Visual Studio, consultez [Débogage dans Visual Studio](/visualstudio/debugger/debugging-in-visual-studio), puis [Conseils de débogage avec Visual Studio 2010](https://weblogs.asp.net/scottgu/archive/2010/08/18/debugging-tips-with-visual-studio-2010.aspx).

### <a name="remote-debugging-in-azure"></a>Débogage distant dans Azure
Pour en savoir plus sur le débogage à distance des applications App Service et des tâches web, consultez les ressources suivantes :

* [Présentation du débogage à distance d'Azure App Service](https://azure.microsoft.com/blog/2014/05/06/introduction-to-remote-debugging-on-azure-web-sites/).
* [Présentation du débogage à distance d'Azure App Service : 2e partie - Au cœur du débogage à distance](https://azure.microsoft.com/blog/2014/05/07/introduction-to-remote-debugging-azure-web-sites-part-2-inside-remote-debugging/)
* [Présentation du débogage à distance d'Azure App Service : 3e partie - Environnement à plusieurs instances et GIT](https://azure.microsoft.com/blog/2014/05/08/introduction-to-remote-debugging-on-azure-web-sites-part-3-multi-instance-environment-and-git/)
* [Débogage de WebJobs (vidéo)](https://www.youtube.com/watch?v=ncQm9q5ZFZs&list=UU_SjTh-ZltPmTYzAybypB-g&index=1)

Si votre application utilise une API Web Azure ou un système principal Mobile Services qu'il vous faut déboguer, consultez l'article [Débogage du serveur principal .NET dans Visual Studio](/archive/blogs/azuremobile/debugging-net-backend-in-visual-studio).

### <a name="tracing-in-aspnet-applications"></a>Suivi dans les applications ASP.NET
Il n'y a pas d'autre présentation du suivi ASP.NET plus détaillée et actualisée disponible sur Internet. Nous vous conseillons de commencer par consulter les anciens documents de présentation rédigés pour Web Forms, car MVC n'existait pas encore, et de compléter cela en consultant les billets de blog les plus récents traitant de problèmes plus précis. Les ressources suivantes constituent un bon début :

* [Surveillance et télémétrie (développement d’applications cloud concrètes avec Azure)](https://www.asp.net/aspnet/overview/developing-apps-with-windows-azure/building-real-world-cloud-apps-with-windows-azure/monitoring-and-telemetry).<br>
  Chapitre de livre électronique contenant des recommandations pour le suivi dans les applications de cloud Azure.
* [Suivi ASP.NET](/previous-versions/dotnet/articles/ms972204(v=msdn.10))<br/>
  Une ressource ancienne mais toujours efficace pour une présentation de base du sujet.
* [Écouteurs de suivi](/dotnet/framework/debug-trace-profile/trace-listeners)<br/>
  Contient des informations sur les écouteurs de suivi mais ne mentionne pas l’élément [WebPageTraceListener](/dotnet/api/system.web.webpagetracelistener).
* [Procédure pas à pas : intégration du suivi ASP.NET avec le suivi System.Diagnostics](/previous-versions/b0ectfxd(v=vs.140))<br/>
  Cet article est également ancien, mais vous y trouverez des informations complémentaires que l’article de présentation ne traite pas.
* [Suivi dans les vues d’ASP.NET MVC Razor](https://devblogs.microsoft.com/aspnet/tracing-in-asp-net-mvc-razor-views/)<br/>
  En plus du suivi dans les vues Razor, ce billet explique également comment créer un filtre d'erreur pour journaliser toutes les exceptions non gérées dans une application MVC. Pour plus d'informations sur la procédure de journalisation des exceptions non gérées dans une application Web Forms, consultez l'exemple de Global.asax dans [Exemple complet pour les gestionnaires d'erreurs](/previous-versions/bb397417(v=vs.140)) sur MSDN. Dans MVC ou Web Forms, si vous voulez journaliser certaines exceptions tout en laissant le Framework par défaut les gérer, vous pouvez utiliser l'exemple de code suivant :

    ```csharp
    try
    {
       // Your code that might cause an exception to be thrown.
    }
    catch (Exception ex)
    {
        Trace.TraceError("Exception: " + ex.ToString());
        throw;
    }
    ```

* [Journalisation de suivi de diagnostics de diffusion à partir de l’outil en ligne de commande Azure (et Glimpse)](https://www.hanselman.com/blog/StreamingDiagnosticsTraceLoggingFromTheAzureCommandLinePlusGlimpse.aspx)<br/>
  Apprenez à utiliser l’outil en ligne de commande pour effectuer ce que montre ce didacticiel dans Visual Studio. [Glimpse](https://www.hanselman.com/blog/IfYoureNotUsingGlimpseWithASPNETForDebuggingAndProfilingYoureMissingOut.aspx) est un outil pour le débogage d'applications ASP.NET.
* [Utilisation des fonctions de journalisation et de diagnostic des applications web - avec David Ebbo](https://azure.microsoft.com/documentation/videos/azure-web-site-logging-and-diagnostics/) et [Journaux d’activité de streaming dans Web Apps - avec David Ebbo](https://azure.microsoft.com/documentation/videos/log-streaming-with-azure-web-sites/)<br>
  par Scott Hanselman et David Ebbo.

Pour la journalisation d'erreurs, vous pouvez éviter d'avoir à écrire votre propre code de suivi en utilisant un Framework de journalisation Open Source comme [ELMAH](https://nuget.org/packages/elmah/). Pour plus d'informations, consultez les [billets du blog de Scott Hanselman sur ELMAH](https://www.hanselman.com/blog/NuGetPackageOfTheWeek7ELMAHErrorLoggingModulesAndHandlersWithSQLServerCompact.aspx).

En outre, vous n’avez pas besoin d’utiliser ASP.NET ou le suivi `System.Diagnostics` pour obtenir les journaux d’activité de streaming à partir d’Azure. Le service de journaux de diffusion en continu de l'application App Service diffuse tous les fichiers *.txt*, *.html* ou *.log* qu'il trouve dans le dossier *LogFiles*. Par conséquent, vous pouvez créer votre propre système de journalisation, qui écrit des données dans le système de fichiers de l'application ; votre fichier est automatiquement diffusé en continu et téléchargé. Tout ce que vous avez à faire est d’écrire un code d’application qui crée les fichiers dans le dossier *d:\home\logfiles*.

### <a name="analyzing-web-server-logs"></a>Analyse de journaux d’activité de serveur Web
Pour plus d’informations sur l’analyse des journaux d’activité de serveur Web, consultez les ressources suivantes :

* [LogParser](https://www.microsoft.com/download/details.aspx?id=24659)<br/>
  Un outil pour afficher les données des journaux d’activité de serveur Web (fichiers *.log* ).
* [Résolution des problèmes de performances IIS ou des erreurs d’application à l’aide de LogParser](https://www.iis.net/learn/troubleshoot/performance-issues/troubleshooting-iis-performance-issues-or-application-errors-using-logparser)<br/>
  Une introduction à l’outil LogParser que vous pouvez utiliser pour analyser les journaux d’activité de serveur Web.
* [Billets du blog de Robert McMurray sur l’utilisation de LogParser](/archive/blogs/robert_mcmurray/using-logparser-with-ftp-7-x-sessions)<br/>
* [Code d’état HTTP dans IIS 7.0, IIS 7.5 et IIS 8.0](https://support.microsoft.com/kb/943891)

### <a name="analyzing-failed-request-tracing-logs"></a>Analyse des journaux d’activité de suivi des demandes ayant échoué
Le site web Microsoft TechNet comporte une section [Utilisation du suivi des demandes ayant échoué](https://www.iis.net/learn/troubleshoot/using-failed-request-tracing) qui permet de comprendre l’utilisation de ces journaux d’activité. Toutefois, cette documentation se concentre principalement sur la configuration du suivi des demandes ayant échoué dans IIS, ce que vous ne pouvez pas faire dans Azure App Service.

[GetStarted]: quickstart-dotnetcore.md?pivots=platform-windows
[GetStartedWJ]: https://github.com/Azure/azure-webjobs-sdk/wiki