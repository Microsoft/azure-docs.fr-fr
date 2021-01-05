---
title: Gérer l’expiration du contenu web dans Azure CDN | Microsoft Docs
description: Découvrez comment gérer l’expiration des contenus d’Azure Web Apps/Services cloud, d’ASP.NET ou d’IIS dans le réseau de distribution de contenu (CDN) Azure.
services: cdn
documentationcenter: .NET
author: asudbring
manager: danielgi
editor: ''
ms.assetid: bef53fcc-bb13-4002-9324-9edee9da8288
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: how-to
ms.date: 02/15/2018
ms.author: allensu
ms.openlocfilehash: d8eb450d2010bf2a525a26f1c5ff48f59732ce43
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93240968"
---
# <a name="manage-expiration-of-web-content-in-azure-cdn"></a>Gérer l’expiration du contenu web dans Azure CDN
> [!div class="op_single_selector"]
> * [Contenu web Azure](cdn-manage-expiration-of-cloud-service-content.md)
> * [stockage d’objets blob Azure](cdn-manage-expiration-of-blob-content.md)
> 

Les fichiers d’un serveur web d’origine accessible à tous peuvent être mis en cache dans Azure Content Delivery Network (CDN) jusqu’à l’expiration de leur durée de vie (TTL). La durée de vie est déterminée par l’en-tête `Cache-Control`dans la réponse HTTP du serveur d’origine. Cet article explique comment définir les en-têtes `Cache-Control` pour la fonctionnalité Web Apps de Microsoft Azure App Service, Services cloud Azure, les applications ASP.NET et les sites IIS (Internet Information Services), qui sont tous configurés de façon similaire. Vous pouvez définir l’en-tête `Cache-Control` à l’aide de fichiers de configuration ou par programme. 

Vous pouvez également contrôler les paramètres de cache à partir du portail Azure, en définissant les [règles de mise en cache CDN](cdn-caching-rules.md). Si vous avez créé une ou plusieurs règles de mise en cache, puis défini leur comportement sur **Remplacer** ou **Ignorer le cache** , les paramètres de mise en cache fournis à l’origine décrits dans cet article sont ignorés. Pour plus d’informations sur les concepts généraux de mise en cache, consultez la section [Fonctionnement de la mise en cache](cdn-how-caching-works.md).

> [!TIP]
> Vous pouvez choisir de ne pas définir de durée de vie sur un fichier. Dans ce cas, Azure CDN applique automatiquement une durée de vie par défaut de sept jours, sauf si vous avez configuré des règles de mise en cache dans le portail Azure. Cette valeur TTL par défaut s’applique uniquement aux optimisations de remise web générales. Pour optimiser les fichiers volumineux, la durée de vie par défaut est de 1 jour, et pour les optimisations du streaming multimédia, la durée de vie par défaut est de 1 an.
> 
> Pour plus d’informations sur la façon dont Azure CDN accélère l’accès aux fichiers et à d’autres ressources, consultez [Vue d’ensemble d’Azure Content Delivery Network](cdn-overview.md).
> 

## <a name="setting-cache-control-headers-by-using-cdn-caching-rules"></a>Définition d’en-têtes Cache-Control à l’aide de règles de mise en cache CDN
La méthode recommandée pour définir l’en-tête `Cache-Control` d’un serveur web consiste à utiliser des règles de mise en cache dans le portail Azure. Pour plus d’informations sur les règles de mise en cache CDN, consultez [Contrôler le comportement de mise en cache d’Azure Content Delivery Network au moyen de règles de mise en cache](cdn-caching-rules.md).

> [!NOTE] 
> Les règles de mise en cache sont disponibles uniquement pour les profils **CDN Azure Standard fourni par Verizon** et **CDN Azure Standard fourni par Akamai**. Pour les profils **CDN Azure Premium fourni par Verizon** , vous devez utiliser le [moteur de règles du CDN Azure](./cdn-verizon-premium-rules-engine.md) dans le portail **Gérer** pour une fonctionnalité similaire.

**Pour accéder à la page des règles de mise en cache CDN** :

1. Dans le portail Azure, sélectionnez un profil CDN, puis sélectionnez un point de terminaison pour le serveur web.

1. Dans le volet gauche, sous Paramètres, sélectionnez **Règles de mise en cache**.

   ![Bouton Règles de mise en cache CDN](./media/cdn-manage-expiration-of-cloud-service-content/cdn-caching-rules-btn.png)

   La page **Règles de mise en cache** s’affiche.

   ![Page de mise en cache du CDN](./media/cdn-manage-expiration-of-cloud-service-content/cdn-caching-page.png)


**Pour définir les en-têtes Cache-Control du serveur web à l’aide de règles de mise en cache générales :**

1. Sous **Règles de mise en cache générales** , définissez **Comportement de mise en cache des chaînes de requête** sur **Ignorer les chaînes de requête** , puis définissez **Comportement de mise en cache** sur **Remplacer**.
      
1. Pour **Durée d’expiration du cache** , entrez 3 600 dans la zone **Secondes** ou 1 dans la zone **Heures**. 

   ![Exemple de règles de mise en cache générales CDN](./media/cdn-manage-expiration-of-cloud-service-content/cdn-global-caching-rules-example.png)

   Cette règle de mise en cache générale définit une durée de mise en cache d’une heure et affecte toutes les requêtes au point de terminaison. Elle se substitue à tout en-tête HTTP `Cache-Control` ou `Expires` qui sont envoyés par le serveur d’origine spécifié par le point de terminaison.   

1. Sélectionnez **Enregistrer**.

**Pour définir les en-têtes Cache-Control d’un fichier de serveur web à l’aide de règles de mise en cache personnalisées :**

1. Sous **Règles de mise en cache personnalisées** , créez deux conditions de correspondance :

     a. Pour la première condition de correspondance, définissez **Condition de correspondance** sur **Chemin** , puis entrez `/webfolder1/*` pour **Valeur(s) de correspondance**. Définissez **Comportement de mise en cache** sur **Remplacer** et entrez 4 dans la zone **Jours**.

     b. Pour la deuxième condition de correspondance, définissez **Condition de correspondance** sur **Chemin** , puis entrez `/webfolder1/file1.txt` pour **Valeur(s) de correspondance**. Définissez **Comportement de mise en cache** sur **Remplacer** et entrez 2 dans la zone **Jours**.

    ![Exemple de règles de mise en cache personnalisées CDN](./media/cdn-manage-expiration-of-cloud-service-content/cdn-custom-caching-rules-example.png)

    La première règle de mise en cache personnalisée définit une durée de mise en cache de quatre heures pour tous les fichiers du dossier `/webfolder1` présent sur le serveur d’origine spécifié par votre point de terminaison. La deuxième règle remplace la première règle pour le fichier `file1.txt` uniquement, et définit une durée de mise en cache de deux heures pour celui-ci.

1. Sélectionnez **Enregistrer**.


## <a name="setting-cache-control-headers-by-using-configuration-files"></a>Définition d’en-têtes Cache-Control à l’aide de fichiers de configuration
Pour le contenu statique, tel que les images et les feuilles de style, vous pouvez contrôler la fréquence de mise à jour en modifiant les fichiers de configuration **applicationHost.config** ou **web.config** de votre application web. Pour définir l’en-tête `Cache-Control` pour votre contenu, utilisez l’élément `<system.webServer>/<staticContent>/<clientCache>` dans l’un des fichiers.

### <a name="using-applicationhostconfig-files"></a>Utilisation des fichiers ApplicationHost.config
Le fichier **ApplicationHost.config** est le fichier racine du système de configuration IIS. Les paramètres de configuration du fichier **applicationHost.config** affectent toutes les applications sur le site, mais sont remplacés par les paramètres des fichiers **web.config** existants pour une application web.

### <a name="using-webconfig-files"></a>Utilisation de fichiers Web.config
Avec un fichier **Web.config** , vous pouvez personnaliser le comportement de l’ensemble de votre application web ou un répertoire spécifique de celle-ci. En général, vous avez au moins un fichier **Web.config** dans le dossier racine de votre application web. Pour chaque fichier **web.config** d’un dossier spécifique, les paramètres de configuration affectent tous les éléments du dossier et de ses sous-dossiers, sauf s’ils sont remplacés au niveau sous-dossier par un autre fichier **Web.config**. 

Par exemple, vous pouvez définir un élément `<clientCache>` dans un fichier **Web.config** dans le dossier racine de votre application web pour mettre en cache tout le contenu statique de votre application web pendant trois jours. Vous pouvez également ajouter un fichier **Web.config** dans un sous-dossier avec un contenu plus variable (par exemple, `\frequent`) et définir son élément `<clientCache>` pour mettre en cache le contenu du sous-dossier pendant 6 heures. Il en résulte que le contenu de l’ensemble du site web est mis en cache pendant trois jours, à l’exception du contenu du répertoire `\frequent` qui est mis en cache pendant six heures seulement.  

L’exemple de fichier de configuration XML suivant montre comment configurer l’élément `<clientCache>` pour spécifier un âge maximal de trois jours :  

```xml
<configuration>
    <system.webServer>
        <staticContent>
            <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="3.00:00:00" />
        </staticContent>
    </system.webServer>
</configuration>
```

Pour utiliser l’attribut **cacheControlMaxAge** , vous devez définir la valeur de l’attribut **cacheControlMode** sur `UseMaxAge`. Ce paramètre a provoqué l’ajout de l’en-tête HTTP et de la directive `Cache-Control: max-age=<nnn>` à la réponse. Le format de la valeur de période pour l’attribut **cacheControlMaxAge** est `<days>.<hours>:<min>:<sec>`. Sa valeur est convertie en secondes et est utilisée comme valeur de la directive `Cache-Control` `max-age`. Pour plus d’informations sur l’élément `<clientCache>`, consultez [Cache client \<clientCache>](https://www.iis.net/ConfigReference/system.webServer/staticContent/clientCache).  

## <a name="setting-cache-control-headers-programmatically"></a>Définition d’en-têtes Cache-Control par programme
Pour les applications ASP.NET, contrôlez par programme le comportement de mise en cache dans CDN en définissant la propriété **HttpResponse.Cache** de l’API .NET. Pour plus d’informations sur la propriété **HttpResponse.Cache** , consultez les pages [HttpResponse.Cache, propriété](/dotnet/api/system.web.httpresponse.cache#System_Web_HttpResponse_Cache) et [HttpCachePolicy, classe](/dotnet/api/system.web.httpcachepolicy).  

Pour mettre en cache par programmation le contenu d’application dans ASP.NET, suivez ces étapes :
   1. Vérifiez que le contenu est marqué comme pouvant être mis en cache en définissant `HttpCacheability` sur `Public`. 
   1. Définissez un validateur de cache en appelant une des méthodes `HttpCachePolicy` suivantes :
      - Appelez `SetLastModified` pour définir une valeur d’horodatage pour l’en-tête `Last-Modified`.
      - Appelez `SetETag` pour définir une valeur pour l’en-tête `ETag`.
   1. Si vous le souhaitez, spécifiez un délai d’expiration du cache en appelant `SetExpires` pour définir une valeur pour l’en-tête `Expires`. Sinon, l’approche de cache par défaut décrite précédemment dans ce document s’applique.

Par exemple, pour mettre en cache du contenu pendant une heure, ajoutez le code C# suivant :  

```csharp
// Set the caching parameters.
Response.Cache.SetExpires(DateTime.Now.AddHours(1));
Response.Cache.SetCacheability(HttpCacheability.Public);
Response.Cache.SetLastModified(DateTime.Now);
```

## <a name="testing-the-cache-control-header"></a>Test de l’en-tête Cache-Control
Vous pouvez facilement vérifier les paramètres de durée de vie de votre contenu web. Avec les [outils de développement](https://developer.microsoft.com/microsoft-edge/platform/documentation/f12-devtools-guide/) de votre navigateur, vérifiez que votre contenu web comprend l’en-tête de réponse `Cache-Control`. Vous pouvez également utiliser un outil tel que **wget** , [Postman](https://www.getpostman.com/) ou [Fiddler](https://www.telerik.com/fiddler) pour examiner les en-têtes de réponse.

## <a name="next-steps"></a>Étapes suivantes
* [Découvrir les détails de l’élément **clientCache**](https://www.iis.net/ConfigReference/system.webServer/staticContent/clientCache)
* [Consulter la documentation de la propriété **HttpResponse.Cache**](/dotnet/api/system.web.httpresponse.cache#System_Web_HttpResponse_Cache) 
* [Lire la documentation concernant la **classe HttpCachePolicy**](/dotnet/api/system.web.httpcachepolicy)  
* [En savoir plus sur les concepts de mise en cache](cdn-how-caching-works.md)
