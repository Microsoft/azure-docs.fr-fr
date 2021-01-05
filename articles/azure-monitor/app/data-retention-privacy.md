---
title: Conservation et stockage des données dans Azure Application Insights | Microsoft Docs
description: Retention and privacy policy statement
ms.topic: conceptual
ms.date: 06/30/2020
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: 2205ab1115a66092ae6dd6d75ee7004ab281eec7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91263910"
---
# <a name="data-collection-retention-and-storage-in-application-insights"></a>Collecte, rétention et stockage des données dans Application Insights

Quand vous installez le Kit de développement logiciel (SDK) [Azure Application Insights][start] dans votre application, il envoie des données de télémétrie sur votre application au cloud. Bien-sûr, les développeurs responsables veulent savoir exactement quelles données sont envoyées, ce qu’elles deviennent et comment ils peuvent conserver le contrôle. Ils souhaitent savoir, plus particulièrement, si les données sensibles peuvent être envoyées, où elles sont stockées et à quel point elles sont sécurisées ? 

Tout d’abord, la réponse courte :

* Les modules de télémétrie standard qui s’exécutent « dès le départ » sont peu susceptibles d’envoyer des données sensibles au service. La télémétrie s’attache aux métriques de charge, de performance et d’utilisation, aux rapports d’exception et autres données de diagnostic. Les données d’utilisateur principal visibles dans les rapports de diagnostics sont des URL ; mais votre application ne doit en aucun cas mettre des données sensibles en texte brut dans une URL.
* Vous pouvez écrire du code qui envoie les données de télémétrie personnalisées supplémentaires, afin de vous aider avec les diagnostics et à surveiller l’utilisation. (Cette extensibilité est une fonctionnalité intéressante de Application Insights.) Il serait possible, par erreur, d’écrire ce code de manière à ce qu’il inclue les données personnelles et d’autres données sensibles. Si votre application fonctionne avec ces données, vous devez vérifier de manière approfondie l’ensemble du code que vous écrivez.
* Pendant le développement et le test de votre application, il est facile d’examiner ce qui est envoyé par le Kit de développement logiciel (SDK). Les données apparaissent dans les fenêtres de sortie de débogage de l’IDE et du navigateur.
* Vous pouvez sélectionner l’emplacement quand vous créez une ressource Application Insights. Vous pouvez en savoir plus sur la disponibilité d’Application Insights par région [ici](https://azure.microsoft.com/global-infrastructure/services/?products=all).
*   Passez en revue les données collectées, car celles-ci peuvent inclure des données qui sont autorisées dans certains cas, mais pas dans d’autres.  À cet égard, un nom d’appareil constitue un parfait exemple. Le nom d’appareil utilisé à partir d’un serveur n’a aucun impact sur la confidentialité et est utile, mais un nom d’appareil utilisés à partir d’un téléphone ou d’un ordinateur portable peut avoir un impact sur la confidentialité et s’avérer moins utile. Un kit de développement logiciel (SDK) développé principalement pour les serveurs cibles, collecte le nom de l’appareil par défaut, et il peut être nécessaire de le remplacer à la fois dans les événements normaux et les exceptions.

Le reste de cet article aborde plus en détail ces réponses. Il est conçu pour être autonome, afin que vous puissiez le montrer à des collègues qui ne font pas partie de votre équipe.

## <a name="what-is-application-insights"></a>Présentation d’Application Insights
[Azure Application Insights][start] est un service de Microsoft qui vous aide à améliorer les performances et la convivialité de votre application dynamique. Il analyse votre application tout au long de son exécution, pendant les tests et une fois que vous avez l’avez publiée ou déployée. Application Insights crée des graphiques et tableaux présentant, par exemple, les heures de la journée à laquelle vous avez le plus d’utilisateurs, la réactivité de l’application et comment elle est prise en charge par les services externes dont elle dépend. En cas d’incidents, d’échecs ou de problèmes de performances, vous pouvez effectuer une recherche dans les données de télémétrie pour diagnostiquer la cause en détail. Et le service vous enverra des messages électroniques si des modifications sont apportées à la disponibilité et à la performance de votre application.

Pour obtenir cette fonctionnalité, vous devez installer un Kit SDK Application Insights dans votre application, il devient partie intégrante de son code. Lorsque votre application s’exécute, le Kit SDK surveille ses opérations et envoie les données de télémétrie au service Application Insights. Il s’agit d’un service cloud hébergé par [Microsoft Azure](https://azure.com). (Mais Application Insights fonctionne pour toutes les applications, pas uniquement celles qui sont hébergées dans Azure.)

Le service Application Insights stocke et analyse les données de télémétrie. Pour afficher l’analyse ou effectuer une recherche dans la télémétrie stockée, connectez-vous à votre compte Azure et ouvrez la ressource Application Insights pour votre application. Vous pouvez également partager l’accès aux données avec d’autres membres de votre équipe, ou avec des abonnés Azure spécifiés.

Vous pouvez exporter des données à partir du service Application Insights vers une base de données ou des outils externes, par exemple. Vous devez attribuer une clé spéciale, que vous obtenez de la part du service, à chaque outil. La clé peut être révoquée si nécessaire. 

Les Kits SDK d’Application Insights sont disponibles pour une variété d’applications : les services web hébergés dans vos propres serveurs Java EE ou ASP.NET ou dans Azure ; les clients web, autrement dit, le code s’exécutant dans une page web ; les applications et les services de bureau ; les applications pour appareils comme Windows Phone, iOS et Android. Toutes envoient les données de télémétrie vers le même service.

## <a name="what-data-does-it-collect"></a>Quelles données collecte-t-il ?
Il existe trois sources de données :

* Le SDK, que vous intégrez à votre application soit [pendant le développement](./asp-net.md), soit [au moment de l’exécution](./monitor-performance-live-website-now.md). Il existe différents Kits SDK pour différents types d’applications. Il existe également un [Kit SDK pour les pages web](./javascript.md), qui se charge dans le navigateur de l’utilisateur final en même temps que la page.
  
  * Chaque Kit SDK comporte un certain nombre de [modules](./configuration-with-applicationinsights-config.md), utilisant diverses techniques pour collecter différents types de données de télémétrie.
  * Si vous installez le Kit SDK pendant le développement, vous pouvez utiliser ses API pour envoyer votre propre télémétrie, ainsi que les modules standards. Ces données de télémétrie personnalisées peuvent inclure toutes les données que vous souhaitez envoyer.
* Sur certains serveurs web, il existe également des agents qui s’exécutent en même temps que l’application et qui envoient des données de télémétrie concernant l’UC, la mémoire et l’occupation du réseau. Par exemple, les machines virtuelles Azure, les hôtes Docker et [les serveurs Java EE](./java-agent.md) peuvent disposer de ces agents.
* [Les tests de disponibilité](./monitor-web-app-availability.md) sont des processus exécutés par Microsoft qui envoient des requêtes à votre application web à intervalles réguliers. Les résultats sont envoyés au service Application Insights.

### <a name="what-kinds-of-data-are-collected"></a>Quel genre de données est collecté ?
Les principales catégories sont :

* [La télémétrie du serveur web](./asp-net.md) : requêtes HTTP.  L’URI, le temps nécessaire pour traiter la demande, le code de réponse, l’adresse IP du client. `Session id`.
* [Les pages web](./javascript.md) : page, utilisateur et décomptes de sessions. Le temps de chargement de page. Les exceptions. Appels Ajax.
* Les performances des compteurs : mémoire, UC, E/S, occupation du réseau.
* Le contexte du client et du serveur : système d’exploitation, paramètres régionaux, type d’appareil, navigateur, résolution d’écran.
* [Les exceptions](./asp-net-exceptions.md) et les incidents : **vidages de pile**, `build id`, type d’UC. 
* [Les dépendances](./asp-net-dependencies.md) : les appels aux services externes tels que REST, SQL, AJAX. L’URI ou la chaîne de connexion, la durée, la réussite, la commande.
* [Les tests de disponibilité](./monitor-web-app-availability.md) : durée et étapes du test, réponses.
* [Les journaux d’activité de suivi](./asp-net-trace-logs.md) et [la télémétrie personnalisée](./api-custom-events-metrics.md) - **tout ce que vous codez dans vos journaux d’activité ou télémétrie**.

[Plus de détails](#data-sent-by-application-insights).

## <a name="how-can-i-verify-whats-being-collected"></a>Comment puis-je vérifier ce qui a été collecté ?
Si vous développez l’application à l’aide de Visual Studio, exécutez l’application en mode débogage (F5). Les données de télémétrie s’affichent dans la fenêtre Sortie. À partir de là, vous pouvez les copier et les mettre au format JSON pour une inspection facile. 

![Capture d’écran montrant l’exécution de l’application en mode débogage dans Visual Studio.](./media/data-retention-privacy/06-vs.png)

Il existe également une vue plus lisible dans la fenêtre Diagnostics.

Pour les pages web, ouvrez la fenêtre de débogage de votre navigateur.

![Appuyez sur F12 et ouvrez l’onglet Réseau.](./media/data-retention-privacy/08-browser.png)

### <a name="can-i-write-code-to-filter-the-telemetry-before-it-is-sent"></a>Puis-je écrire le code pour filtrer les données de télémétrie avant qu’elles soient envoyées ?
Cela est possible en écrivant un [plug-in de processeur de télémétrie](./api-filtering-sampling.md).

## <a name="how-long-is-the-data-kept"></a>Combien de temps sont conservées les données ?
Les points de données brutes (autrement dit, les éléments que vous pouvez interroger dans Analytics et inspecter dans Recherche) sont conservés pendant 730 jours maximum. Vous pouvez [sélectionner une durée de rétention](./pricing.md#change-the-data-retention-period) de 30, 60, 90, 120, 180, 270, 365, 550 ou 730 jours. Si vous voulez conserver les données plus longtemps, vous pouvez utiliser [l’exportation continue](./export-telemetry.md) pour les copier dans un compte de stockage durant l’ingestion des données. 

Des frais supplémentaires sont facturés pour les données conservées pendant plus de 90 jours. Plus d’informations sur la tarification d’Application Insights sur la [page de tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

Les données agrégées (autrement dit, les nombres, moyennes et autres données statistiques que vous voyez dans Metrics Explorer) sont conservées avec une granularité de 1 minute pendant 90 jours.

[Les captures instantanées de débogage](./snapshot-debugger.md) sont stockées pendant 15 jours. Cette stratégie de rétention est définie application par application. Si vous devez augmenter cette valeur, faites-en la demande en ouvrant une demande de support dans le portail Azure.

## <a name="who-can-access-the-data"></a>Qui peut accéder aux données ?
Les données sont visibles par vous et, si vous disposez un compte d’organisation, par les membres de votre équipe. 

Vous ou les membres de votre équipe pouvez les exporter et les copier à d’autres endroits, ou les transférer à d’autres personnes.

#### <a name="what-does-microsoft-do-with-the-information-my-app-sends-to-application-insights"></a>Que fait Microsoft des informations que mon application envoie à Application Insights ?
Microsoft n’utilise les données que pour vous fournir le service.

## <a name="where-is-the-data-held"></a>Où sont conservées les données ?
* Vous pouvez sélectionner l’emplacement quand vous créez une ressource Application Insights. Vous pouvez en savoir plus sur la disponibilité d’Application Insights par région [ici](https://azure.microsoft.com/global-infrastructure/services/?products=all).

## <a name="how-secure-is-my-data"></a>Mes données sont-elles sécurisées ?
Application Insights est un service Azure. Les stratégies de sécurité sont décrites dans le [livre blanc sur la sécurité, la confidentialité et la conformité Azure](https://go.microsoft.com/fwlink/?linkid=392408).

Les données sont stockées sur des serveurs Microsoft Azure. Pour les comptes du Portail Azure, les restrictions sont décrites dans le [document relatif à la sécurité, la confidentialité et la conformité Azure](https://go.microsoft.com/fwlink/?linkid=392408).

L'accès à vos données par le personnel Microsoft est limité. Nous n'accédons à vos données qu'avec votre autorisation, et seulement pour vous aider à utiliser Application Insights. 

Les données agrégées sur toutes les applications de nos clients (comme le débit des données et la taille moyenne des suivis) sont utilisées pour améliorer Application Insights.

#### <a name="could-someone-elses-telemetry-interfere-with-my-application-insights-data"></a>Les données de télémétrie d’une autre personne peuvent-elles interférer avec mes données d’Application Insights ?
Elles peuvent envoyer des données de télémétrie supplémentaires à votre compte à l’aide de la clé d’instrumentation, qui se trouve dans le code de vos pages web. Avec suffisamment de données supplémentaires, vos mesures ne représentent pas correctement les performances et l’utilisation de votre application.

Si vous partagez le code avec d’autres projets, pensez à supprimer votre clé d’instrumentation.

## <a name="is-the-data-encrypted"></a>Les données sont-elles chiffrées ?
Toutes les données sont chiffrées, aussi bien au repos que lors de leur déplacement entre les centres de données.

#### <a name="is-the-data-encrypted-in-transit-from-my-application-to-application-insights-servers"></a>Les données sont-elles chiffrées lors de leur passage depuis mon application vers les serveurs Application Insights ?
Oui, nous utilisons le protocole HTTPS pour envoyer les données au portail à partir de presque tous les Kits de développement logiciel (SDK), y compris les serveurs web, les appareils et les pages web HTTPS. 

## <a name="does-the-sdk-create-temporary-local-storage"></a>Le kit de développement logiciel (SDK) crée-t-il un stockage local temporaire ?

Oui, certains canaux de télémétrie conservent les données localement si un point de terminaison est inaccessible. Consultez ce qui suit pour voir les infrastructures et les canaux de télémétrie concernés.

Les canaux de télémétrie qui utilisent le stockage local créent des fichiers temporaires dans les répertoires TEMP ou APPDATA, qui sont limités au compte exécutant votre application. Cela se produit lorsqu’un point de terminaison a été temporairement indisponible ou lorsque vous avez atteint la limite de bande passante. Une fois ce problème résolu, le canal de télémétrie reprend l’envoi de toutes les données nouvelles et conservées.

Ces données persistantes ne sont pas chiffrées en local. Si cela pose problème, passez en revue les données et limitez la collection de données privées. (Pour plus d’informations, consultez [Comment exporter et supprimer des données privées](../platform/personal-data-mgmt.md#how-to-export-and-delete-private-data).)

Si un client a besoin de configurer ce répertoire avec des exigences de sécurité spécifiques, il peut être configuré par infrastructure. Vérifiez que le processus exécutant votre application a accès en écriture à ce répertoire, mais veillez également à que ce répertoire soit protégé pour éviter la lecture des données de télémétrie par des utilisateurs non autorisés.

### <a name="java"></a>Java

`C:\Users\username\AppData\Local\Temp` est utilisé pour les données persistantes. Cet emplacement n’est pas configurable à partir du répertoire de configuration et les autorisations pour accéder à ce dossier sont limitées à l’utilisateur qui possède les informations d’identification requises. (Pour plus d’informations, consultez [Implémentation](https://github.com/Microsoft/ApplicationInsights-Java/blob/40809cb6857231e572309a5901e1227305c27c1a/core/src/main/java/com/microsoft/applicationinsights/internal/util/LocalFileSystemUtils.java#L48-L72).)

###  <a name="net"></a>.NET

Par défaut, `ServerTelemetryChannel` utilise le dossier de données d’application local `%localAppData%\Microsoft\ApplicationInsights` ou le dossier temp `%TMP%` de l’utilisateur actuel. (Consultez l’[implémentation](https://github.com/Microsoft/ApplicationInsights-dotnet/blob/91e9c91fcea979b1eec4e31ba8e0fc683bf86802/src/ServerTelemetryChannel/Implementation/ApplicationFolderProvider.cs#L54-L84) ici.)


Par le biais du fichier de configuration :
```xml
<TelemetryChannel Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.ServerTelemetryChannel,   Microsoft.AI.ServerTelemetryChannel">
    <StorageFolder>D:\NewTestFolder</StorageFolder>
</TelemetryChannel>
```

Par le biais du code :

- Supprimer ServerTelemetryChannel du fichier de configuration
- Ajoutez cet extrait de code à votre configuration :
  ```csharp
  ServerTelemetryChannel channel = new ServerTelemetryChannel();
  channel.StorageFolder = @"D:\NewTestFolder";
  channel.Initialize(TelemetryConfiguration.Active);
  TelemetryConfiguration.Active.TelemetryChannel = channel;
  ```

### <a name="netcore"></a>NetCore

Par défaut, `ServerTelemetryChannel` utilise le dossier de données d’application local `%localAppData%\Microsoft\ApplicationInsights` ou le dossier temp `%TMP%` de l’utilisateur actuel. (Consultez l’[implémentation](https://github.com/Microsoft/ApplicationInsights-dotnet/blob/91e9c91fcea979b1eec4e31ba8e0fc683bf86802/src/ServerTelemetryChannel/Implementation/ApplicationFolderProvider.cs#L54-L84) ici.) 

Dans un environnement Linux, le stockage local sera désactivé sauf si un dossier de stockage est spécifié.

> [!NOTE]
> Grâce à la version 2.15.0-beta3 et celles ultérieures, un stockage local est désormais automatiquement créé pour Linux, Mac et Windows. Pour les systèmes non Windows, le Kit de développement logiciel (SDK) crée automatiquement un dossier de stockage local selon la logique suivante :
> - `${TMPDIR}` : Si la variable d’environnement `${TMPDIR}` est définie, cet emplacement est utilisé.
> - `/var/tmp` : Si l’emplacement précédent n’existe pas, nous essayons `/var/tmp`.
> - `/tmp` : Si les deux emplacements précédents n’existent pas, nous essayons `tmp`. 
> - Si aucun de ces emplacements n’existe, le stockage local n’est pas créé et la configuration manuelle est toujours requise. [En savoir plus sur l’implémentation](https://github.com/microsoft/ApplicationInsights-dotnet/pull/1860).

L’extrait de code suivant montre comment définir `ServerTelemetryChannel.StorageFolder` dans la méthode `ConfigureServices()` de votre classe `Startup.cs` :

```csharp
services.AddSingleton(typeof(ITelemetryChannel), new ServerTelemetryChannel () {StorageFolder = "/tmp/myfolder"});
```

(Pour plus d’informations, consultez [Configuration personnalisée d’AspNetCore](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Custom-Configuration).)

### <a name="nodejs"></a>Node.js

Par défaut, `%TEMP%/appInsights-node{INSTRUMENTATION KEY}` est utilisé pour les données persistantes. Les autorisations d’accès à ce dossier sont limitées à l’utilisateur actuel et aux administrateurs. (Consultez l’[implémentation](https://github.com/Microsoft/ApplicationInsights-node.js/blob/develop/Library/Sender.ts) ici.)

Le préfixe du dossier `appInsights-node` peut être substitué en modifiant la valeur d’exécution de la variable statique `Sender.TEMPDIR_PREFIX` trouvée dans [Sender.ts](https://github.com/Microsoft/ApplicationInsights-node.js/blob/7a1ecb91da5ea0febf5ceab13d6a4bf01a63933d/Library/Sender.ts#L384).

### <a name="javascript-browser"></a>Javascript (navigateur)

Le [stockage de session HTML5](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage) est utilisé pour faire persister les données. Deux mémoires tampons distinctes sont utilisées : `AI_buffer` et `AI_sent_buffer`. Les données de télémétrie traitées par lots et en attente d’envoi sont stockées dans `AI_buffer`. Les données de télémétrie qui viennent d’être envoyées sont placées dans `AI_sent_buffer` jusqu’à ce que le serveur d’ingestion réponde qu’elles ont bien été reçues. Une fois que les données de télémétrie ont bien été reçues, elles sont supprimées de toutes les mémoires tampons. En cas de défaillances temporaires (par exemple, perte de connectivité pour un utilisateur), les données de télémétrie restent dans `AI_buffer` jusqu’à ce qu’elles soient bien reçues ou que le serveur d’ingestion réponde que les données de télémétrie ne sont pas valides (schéma incorrect ou trop ancien, par exemple).

Les mémoires tampons de télémétrie peuvent être désactivées en définissant [`enableSessionStorageBuffer`](https://github.com/microsoft/ApplicationInsights-JS/blob/17ef50442f73fd02a758fbd74134933d92607ecf/legacy/JavaScript/JavaScriptSDK.Interfaces/IConfig.ts#L31) sur `false`. Quand le stockage de session est désactivé, une baie locale est utilisée comme stockage persistant. Comme le kit SDK JavaScript s’exécute sur un appareil client, l’utilisateur a accès à cet emplacement de stockage via les outils de développement de son navigateur.

### <a name="opencensus-python"></a>OpenCensus Python

Par défaut, le SDK OpenCensus Python utilise le dossier de l’utilisateur actuel `%username%/.opencensus/.azure/`. Les autorisations d’accès à ce dossier sont limitées à l’utilisateur actuel et aux administrateurs. (Consultez l’[implémentation](https://github.com/census-instrumentation/opencensus-python/blob/master/contrib/opencensus-ext-azure/opencensus/ext/azure/common/storage.py) ici.) Le dossier contenant vos données persistantes va porter le nom du fichier Python qui a généré les données de télémétrie.

Vous pouvez changer l’emplacement de votre fichier de stockage en passant le paramètre `storage_path` dans le constructeur de l’exportateur que vous utilisez.

```python
AzureLogHandler(
  connection_string='InstrumentationKey=00000000-0000-0000-0000-000000000000',
  storage_path='<your-path-here>',
)
```

## <a name="how-do-i-send-data-to-application-insights-using-tls-12"></a>Comment envoyer des données à Application Insights à l’aide de TLS 1.2 ?

Pour garantir la sécurité des données en transit vers les points de terminaison Application Insights, nous encourageons vivement les clients à configurer leur application de façon à utiliser TLS version 1.2 minimum. Les versions antérieures de TLS/SSL (Secure Sockets Layer) se sont avérées vulnérables et bien qu’elles fonctionnent encore pour assurer la compatibilité descendante, elles sont **déconseillées** et le secteur évolue rapidement vers un arrêt de la prise en charge de ces anciens protocoles. 

Le [PCI Security Standards Council](https://www.pcisecuritystandards.org/) a défini une [échéance au 30 juin 30, 2018](https://www.pcisecuritystandards.org/pdfs/PCI_SSC_Migrating_from_SSL_and_Early_TLS_Resource_Guide.pdf) pour désactiver les versions antérieures de TLS/SSL et effectuer une mise à niveau vers des protocoles plus sécurisés. Une fois qu’Azure arrêtera la prise en charge des versions héritées, si votre application/vos clients ne peuvent pas communiquer via le protocole TLS version 1.2 minimum, vous ne serez plus en mesure d’envoyer des données à Application Insights. L’approche que vous adoptez pour tester et valider la prise en charge TLS de votre application varie selon le système d’exploitation/la plateforme, ainsi que le langage/l’infrastructure que votre application utilise.

Nous ne recommandons pas de configurer explicitement votre application de façon à ce qu’elle utilise uniquement TLS 1.2, sauf si cela est nécessaire, car cela peut annuler les fonctionnalités de sécurité au niveau de la plateforme qui vous permettent de détecter automatiquement et de tirer parti des protocoles plus sécurisés et plus récents dès qu’ils sont disponibles, par exemple TLS 1.3. Nous vous recommandons d’effectuer un audit complet du code de votre application pour vérifier le codage en dur des versions spécifiques de TLS/SSL.

### <a name="platformlanguage-specific-guidance"></a>Conseils spécifiques à la plateforme/au langage

|Plateforme/Langage | Support | Informations complémentaires |
| --- | --- | --- |
| Azure App Services  | Pris en charge, la configuration peut être nécessaire. | La prise en charge a été annoncée en avril 2018. Lisez l’annonce pour connaître les [détails de configuration](https://azure.github.io/AppService/2018/04/17/App-Service-and-Functions-hosted-apps-can-now-update-TLS-versions!).  |
| Applications de fonction Azure | Pris en charge, la configuration peut être nécessaire. | La prise en charge a été annoncée en avril 2018. Lisez l’annonce pour connaître les [détails de configuration](https://azure.github.io/AppService/2018/04/17/App-Service-and-Functions-hosted-apps-can-now-update-TLS-versions!). |
|.NET | Pris en charge, la configuration diffère selon la version. | Pour des informations de configuration détaillées pour .NET 4.7 et versions antérieures, reportez-vous à [ces instructions](/dotnet/framework/network-programming/tls#support-for-tls-12).  |
|Status Monitor | Pris en charge, configuration requise | Status Monitor s’appuie sur [la configuration du système d’exploitation](/windows-server/security/tls/tls-registry-settings) + [la configuration .NET](/dotnet/framework/network-programming/tls#support-for-tls-12) pour prendre en charge TLS 1.2.
|Node.js |  Pris en charge, dans v10.5.0, la configuration peut être nécessaire. | Utilisez la [documentation officielle de Node.js TLS/SSL](https://nodejs.org/api/tls.html) pour toute configuration spécifique à une application. |
|Java | Pris en charge, prise en charge JDK pour TLS 1.2 ajoutée dans [JDK 6 mise à jour 121](https://www.oracle.com/technetwork/java/javase/overview-156328.html#R160_121) et [JDK 7](https://www.oracle.com/technetwork/java/javase/7u131-relnotes-3338543.html). | JDK 8 utilise [TLS 1.2 par défaut](https://blogs.oracle.com/java-platform-group/jdk-8-will-use-tls-12-as-default).  |
|Linux | Les distributions de Linux s’appuient généralement sur [OpenSSL](https://www.openssl.org) pour la prise en charge de TLS 1.2.  | Vérifiez [OpenSSL Changelog](https://www.openssl.org/news/changelog.html) pour vous assurer que votre version d’OpenSSL est prise en charge.|
| Windows 8.0 - 10 | Pris en charge, activé par défaut. | Pour confirmer que vous utilisez toujours les [paramètres par défaut](/windows-server/security/tls/tls-registry-settings).  |
| Windows Server 2012 - 2016 | Pris en charge, activé par défaut. | Pour confirmer que vous utilisez toujours les [paramètres par défaut](/windows-server/security/tls/tls-registry-settings) |
| Windows 7 SP1 et Windows Server 2008 R2 SP1 | Pris en charge, mais non activé par défaut. | Consultez la page [Paramètres de Registre de TLS](/windows-server/security/tls/tls-registry-settings) pour plus d’informations sur l’activation.  |
| Windows Server 2008 SP2 | La prise en charge de TLS 1.2 nécessite une mise à jour. | Consultez [Mise à jour pour ajouter la prise en charge de TLS 1.2](https://support.microsoft.com/help/4019276/update-to-add-support-for-tls-1-1-and-tls-1-2-in-windows-server-2008-s) dans Windows Server 2008 SP2. |
|Windows Vista | Non pris en charge. | N/A

### <a name="check-what-version-of-openssl-your-linux-distribution-is-running"></a>Vérifier la version d’OpenSSL que votre distribution Linux exécute

Pour vérifier la version d’OpenSSL installée, ouvrez le terminal et exécutez :

```terminal
openssl version -a
```

### <a name="run-a-test-tls-12-transaction-on-linux"></a>Exécuter un test de transaction de TLS 1.2 sur Linux

Pour exécuter un test préliminaire pour voir si votre système Linux peut communiquer via TLS 1.2, ouvrez le terminal et exécutez :

```terminal
openssl s_client -connect bing.com:443 -tls1_2
```

## <a name="personal-data-stored-in-application-insights"></a>Données personnelles stockées dans Application Insights

Notre [article Données personnelles Application Insights](../platform/personal-data-mgmt.md) traite de ce problème en profondeur.

#### <a name="can-my-users-turn-off-application-insights"></a>Mes utilisateurs peuvent-ils désactiver Application Insights ?
Pas directement. Il n’existe pas de commutateur que vos utilisateurs peuvent utiliser pour désactiver Application Insights.

Toutefois, vous pouvez implémenter cette fonctionnalité dans votre application. Tous les Kits de développement logiciel (SDK) comprennent un paramètre API qui désactive la collecte de télémétrie. 

## <a name="data-sent-by-application-insights"></a>Données envoyées par Application Insights
Les kits de développement logiciel (SDK) varient en fonction des plateformes et vous pouvez installer plusieurs composants. (Consultez [Application Insights - Vue d’ensemble][start].) Chaque composant envoie des données différentes.

#### <a name="classes-of-data-sent-in-different-scenarios"></a>Classes de données envoyées dans différents scénarios

| Votre action | Classes de données collectées (voir tableau suivant) |
| --- | --- |
| [Ajouter le kit de développement logiciel (SDK) Application Insights à un projet web .NET][greenbrown] |ServerContext<br/>Inferred<br/>Perf counters<br/>Demandes<br/>**Exceptions**<br/>session<br/>users |
| [Installer Status Monitor sur IIS][redfield] |Les dépendances<br/>ServerContext<br/>Inferred<br/>Perf counters |
| [Ajouter le kit de développement logiciel (SDK) Application Insights à une application web Java][java] |ServerContext<br/>Inferred<br/>Requête<br/>session<br/>users |
| [Ajouter le kit de développement logiciel (SDK) JavaScript à une page web][client] |ClientContext <br/>Inferred<br/>Page<br/>ClientPerf<br/>Ajax |
| [Définir les propriétés par défaut][apiproperties] |**Propriétés** sur tous les événements standard et personnalisés |
| [Appeler TrackMetric][api] |Valeurs numériques<br/>**Propriétés** |
| [Appeler Track*][api] |Nom d'événement<br/>**Propriétés** |
| [Appeler TrackException][api] |**Exceptions**<br/>Vidage de pile<br/>**Propriétés** |
| Le Kit SDK ne peut pas collecter les données. Par exemple : <br/> - impossible d’accéder aux compteurs de performances<br/> - exception dans l’initialiseur de télémétrie |SDK diagnostics |

Pour les [Kits de développement logiciel (SDK) des autres plateformes][platforms], consultez les documents correspondants.

#### <a name="the-classes-of-collected-data"></a>Les classes de données collectées

| Classe des données collectées | Comprend (liste non exhaustive) |
| --- | --- |
| **Propriétés** |**Toutes les données - en fonction de votre code** |
| DeviceContext |`Id` adresse IP, paramètres régionaux, modèle d’appareil, réseau, type de réseau, nom OEM, résolution d’écran, instance de rôle, nom du rôle, type d’appareil |
| ClientContext |Système d’exploitation, paramètres régionaux, langue, réseau, résolution de la fenêtre |
| session |`session id` |
| ServerContext |Nom de l’ordinateur, paramètres régionaux, système d’exploitation, appareil, session utilisateur, contexte utilisateur, opération |
| Inferred |Emplacement géographique à partir de l’adresse IP, horodatage, système d’exploitation, navigateur |
| Mesures |Valeur et nom de la mesure |
| Événements |Valeur et nom de l’événement |
| PageViews |URL et nom de la page ou de l’écran |
| Client perf |URL/nom de la page, temps de chargement du navigateur |
| Ajax |Appels HTTP de la page web au serveur |
| Demandes |URL, durée, code de réponse |
| Les dépendances |Type (SQL, HTTP, ...), chaîne de connexion ou URI, synchronisation/désynchronisation, durée, réussite, instruction SQL (avec Status Monitor) |
| **Exceptions** |Type, **message**, piles d’appels, fichier source, numéro ligne, `thread id` |
| Crashes |`Process id`, `parent process id`, `crash thread id` ; correctif de l’application, `id`, version ; type d’exception, adresse, motif ; symboles et enregistrements masqués, adresses binaires de début et de fin, nom et chemin du fichier binaire, type de processeur |
| Trace |**Message** et niveau de gravité |
| Perf counters |Temps processeur, mémoire disponible, taux de demande, taux d’exception, octets privés du processus, taux d’E/S, durée de la demande, longueur de file d’attente de la demande |
| Disponibilité |Code de réponse de test web, durée de chaque étape de test, nom de test, horodatage, réussite, temps de réponse, emplacement de test |
| SDK diagnostics |Message de suivi ou exception |

Vous pouvez [désactiver certaines données en modifiant ApplicationInsights.config][config]

> [!NOTE]
> L’IP du client est utilisée pour déduire son emplacement géographique, mais par défaut, les données d’IP ne sont plus stockées et tous les zéros sont écrits dans le champ associé. Pour en savoir plus sur la gestion des données personnelles, nous vous invitons à lire [cet article](../platform/personal-data-mgmt.md#application-data). Si vous avez besoin de stocker des données d’adresse IP, notre [article sur la collecte des adresses IP](./ip-collection.md) vous présentera vos options.

## <a name="credits"></a>Crédits
Ce produit contient des données GeoLite2 créées par MaxMind, disponibles sur [https://www.maxmind.com](https://www.maxmind.com).



<!--Link references-->

[api]: ./api-custom-events-metrics.md
[apiproperties]: ./api-custom-events-metrics.md#properties
[client]: ./javascript.md
[config]: ./configuration-with-applicationinsights-config.md
[greenbrown]: ./asp-net.md
[java]: ./java-get-started.md
[platforms]: ./platforms.md
[pricing]: https://azure.microsoft.com/pricing/details/application-insights/
[redfield]: ./monitor-performance-live-website-now.md
[start]: ./app-insights-overview.md

