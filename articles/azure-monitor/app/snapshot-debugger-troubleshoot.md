---
title: Dépannage du débogueur de capture instantanée d’Azure Application Insights
description: Cet article décrit les étapes de résolution des problèmes et donne des informations afin d’aider les développeurs à activer et à utiliser l’extension Débogueur de capture instantanée d’Application Insights.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 03/07/2019
ms.reviewer: mbullwin
ms.openlocfilehash: 3e45b5a48f9a7fb66d7539e83f385203ae371ad7
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673550"
---
# <a name="troubleshoot-problems-enabling-application-insights-snapshot-debugger-or-viewing-snapshots"></a><a id="troubleshooting"></a> Résoudre les problèmes d’activation du Débogueur de capture instantanée Application Insights ou d’affichage d’instantanés
Si vous avez activé Débogueur de capture instantanée Application Insights pour votre application, mais que vous ne voyez pas de captures instantanées pour les exceptions, vous pouvez utiliser ces instructions pour résoudre les problèmes.

Il peut y avoir de nombreuses raisons différentes pour lesquelles les captures instantanées ne sont pas générées. Vous pouvez commencer par exécuter le contrôle d’intégrité de capture instantanée pour identifier certaines des causes courantes possibles.

## <a name="use-the-snapshot-health-check"></a>Utiliser le contrôle d’intégrité de capture instantanée
Plusieurs problèmes courants empêchent l’affichage du message Ouvrir l’instantané de débogage. L’utilisation d’un collecteur de captures instantanées obsolète, par exemple lié à l’atteinte de la limite de chargement quotidienne, entraîne un temps de téléchargement important. Utilisez le contrôle d’intégrité de capture instantanée pour résoudre des problèmes courants.

Il existe un lien dans le volet d’exception de l’affichage de suivi de bout en bout, qui permet d’accéder au contrôle d’intégrité de capture instantanée.

![Entrer le contrôle d’intégrité de capture instantanée](./media/snapshot-debugger/enter-snapshot-health-check.png)

L’interface interactive, de type conversation, recherche des problèmes courants et vous guide pour les résoudre.

![Contrôle d’intégrité](./media/snapshot-debugger/healthcheck.png)

Si cela ne résout pas le problème, consultez les étapes de dépannage manuel suivantes.

## <a name="verify-the-instrumentation-key"></a>Vérifier la clé d’instrumentation

Assurez-vous que vous utilisez la clé d’instrumentation correcte dans votre application publiée. En règle générale, la clé d’instrumentation est lue à partir du fichier ApplicationInsights.config. Vérifiez que la valeur est identique à la clé d’instrumentation de la ressource Application Insights que vous voyez dans le portail.

## <a name="check-ssl-client-settings-aspnet"></a><a id="SSL"></a>Vérifier les paramètres du client SSL (ASP.NET)

Si vous avez une application ASP.NET hébergée dans Azure App Service ou IIS sur une machine virtuelle, votre application risque d’échouer à se connecter au service Débogueur de capture instantanée en raison d’un protocole de sécurité SSL manquant.
[Le point de terminaison Débogueur de capture instantanée nécessite la version 1.2 du protocole TLS](snapshot-debugger-upgrade.md?toc=/azure/azure-monitor/toc.json). L’ensemble des protocoles de sécurité SSL est l’un des quirks activés par la valeur httpRuntime targetFramework dans la section system.web de web.config. Si la valeur de httpRuntime targetFramework est 4.5.2 ou inférieure, TLS 1.2 n’est pas inclus par défaut.

> [!NOTE]
> La valeur httpRuntime targetFramework est indépendante de la version cible du framework cible utilisée lors de la génération de votre application.

Pour vérifier le paramètre, ouvrez votre fichier web.config et recherchez la section system.web. Assurez-vous que le `targetFramework` pour `httpRuntime` est défini sur 4.6 ou une version ultérieure.

   ```xml
   <system.web>
      ...
      <httpRuntime targetFramework="4.7.2" />
      ...
   </system.web>
   ```

> [!NOTE]
> La modification de la valeur httpRuntime targetFramework modifie les quirks du runtime appliquées à votre application et peut entraîner d’autres changements de comportement subtils. Veillez à tester soigneusement votre application après avoir apporté cette modification. Pour obtenir la liste complète des modifications de compatibilité, consultez https://docs.microsoft.com/dotnet/framework/migration-guide/application-compatibility#retargeting-changes

> [!NOTE]
> Si le targetFramework est 4.7 ou une version ultérieure, Windows détermine les protocoles disponibles. Dans Azure App Service, TLS 1.2 est disponible. Toutefois, si vous utilisez votre propre machine virtuelle, vous devrez peut-être activer TLS 1.2 dans le système d’exploitation.

## <a name="preview-versions-of-net-core"></a>Préversions de .NET Core
Si vous utilisez une préversion de .NET Core ou que votre application fait référence au Kit de développement logiciel (SDK) Application Insights, directement ou indirectement via un assembly dépendant, suivez les instructions pour [activer Débogueur de capture instantanée pour d’autres environnements](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json).

## <a name="check-the-diagnostic-services-site-extension-status-page"></a>Vérifier la page d’état de l’extension de site Services de diagnostic
Si Débogueur de capture instantanée a été activé via le [volet Application Insights](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json) du portail, il a été activé par l’extension de site Services de diagnostic.

Vous pouvez consulter la page d’état de cette extension en accédant à l’URL suivante : `https://{site-name}.scm.azurewebsites.net/DiagnosticServices`.

> [!NOTE]
> Le domaine du lien de la page d’état varie en fonction du cloud.
Ce domaine sera le même que le site de gestion Kudu pour App Service.

Cette page d’état indique l’état d’installation des agents Profiler et Snapshot Collector. Si une erreur inattendue se produit, elle s’affiche et montre comment la corriger.

Vous pouvez utiliser le site de gestion Kudu pour App Service pour obtenir l’URL de base de cette page d’état :
1. Ouvrez votre application App Service dans le portail Azure.
2. Sélectionnez **Outils avancés** ou recherchez **Kudu**.
3. Sélectionnez **Go**.
4. Une fois que vous êtes sur le site de gestion Kudu, dans l’URL, **ajoutez l’élément `/DiagnosticServices`, puis appuyez sur Entrée**.
 L’URL se terminera comme suit : `https://<kudu-url>/DiagnosticServices`.

Une page d’état similaire à celle ci-dessous s’affiche : ![Page d’état de Services de diagnostic](./media/diagnostic-services-site-extension/status-page.png)

## <a name="upgrade-to-the-latest-version-of-the-nuget-package"></a>Mettre à niveau vers la dernière version du package NuGet
En fonction de la façon dont Débogueur de capture instantanée a été activé, consultez les options suivantes :

* Si le Débogueur de capture instantanée a été activé par le biais du [volet Application Insights du portail](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json), votre application doit déjà exécuter le dernier package NuGet.

* Si le Débogueur de capture instantanée a été activé en incluant le package NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector), utilisez le Gestionnaire de package NuGet de Visual Studio pour vérifier que vous utilisez la dernière version de Microsoft.ApplicationInsights.SnapshotCollector.

Pour obtenir les mises à jour et correctifs de bogues les plus récents, [consultez les notes de publication](./snapshot-collector-release-notes.md).

## <a name="check-the-uploader-logs"></a>Vérifier les journaux d’activité du chargeur

Une fois une capture instantanée créée, un fichier minidump (.dmp) est créé sur le disque. Un processus distinct du chargeur crée ce fichier minidump et le charge, ainsi que tous les fichiers PDB associés, vers le stockage du Débogueur de capture instantanée d’Application Insights. Une fois le fichier minidump correctement chargé, il est supprimé du disque. Les fichiers journaux du processus de chargement sont conservés sur disque. Dans un environnement App Service, vous pouvez trouver ces fichiers journaux d’activité dans `D:\Home\LogFiles`. Le site de gestion Kudu pour App Service permet de rechercher ces fichiers journaux.

1. Ouvrez votre application App Service dans le portail Azure.
2. Sélectionnez **Outils avancés** ou recherchez **Kudu**.
3. Sélectionnez **Go**.
4. Dans le zone de liste déroulante de la **Console de débogage**, sélectionnez **CMD**.
5. Sélectionnez **LogFiles**.

Il devrait y avoir au moins un fichier dont le nom commence par `Uploader_` ou `SnapshotUploader_` et qui comporte l’extension `.log`. Sélectionnez l’icône appropriée pour télécharger tous les fichiers journaux ou les ouvrir dans un navigateur.
Le nom de fichier comporte un suffixe unique qui identifie l’instance App Service. Si votre instance App Service est hébergée sur plusieurs machines, il existe des fichiers journaux distincts pour chacune d’elles. Lorsque le chargeur détecte la présence d’un nouveau fichier minidump, celui-ci est enregistré dans le fichier journal. Voici un exemple de capture instantanée et de chargement réussis :

```
SnapshotUploader.exe Information: 0 : Received Fork request ID 139e411a23934dc0b9ea08a626db16c5 from process 6368 (Low pri)
    DateTime=2018-03-09T01:42:41.8571711Z
SnapshotUploader.exe Information: 0 : Creating minidump from Fork request ID 139e411a23934dc0b9ea08a626db16c5 from process 6368 (Low pri)
    DateTime=2018-03-09T01:42:41.8571711Z
SnapshotUploader.exe Information: 0 : Dump placeholder file created: 139e411a23934dc0b9ea08a626db16c5.dm_
    DateTime=2018-03-09T01:42:41.8728496Z
SnapshotUploader.exe Information: 0 : Dump available 139e411a23934dc0b9ea08a626db16c5.dmp
    DateTime=2018-03-09T01:42:45.7525022Z
SnapshotUploader.exe Information: 0 : Successfully wrote minidump to D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp
    DateTime=2018-03-09T01:42:45.7681360Z
SnapshotUploader.exe Information: 0 : Uploading D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp, 214.42 MB (uncompressed)
    DateTime=2018-03-09T01:42:45.7681360Z
SnapshotUploader.exe Information: 0 : Upload successful. Compressed size 86.56 MB
    DateTime=2018-03-09T01:42:59.6184651Z
SnapshotUploader.exe Information: 0 : Extracting PDB info from D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp.
    DateTime=2018-03-09T01:42:59.6184651Z
SnapshotUploader.exe Information: 0 : Matched 2 PDB(s) with local files.
    DateTime=2018-03-09T01:42:59.6809606Z
SnapshotUploader.exe Information: 0 : Stamp does not want any of our matched PDBs.
    DateTime=2018-03-09T01:42:59.8059929Z
SnapshotUploader.exe Information: 0 : Deleted D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp
    DateTime=2018-03-09T01:42:59.8530649Z
```

> [!NOTE]
> L’exemple ci-dessus concerne la version 1.2.0 du package NuGet Microsoft.ApplicationInsights.SnapshotCollector. Dans les versions antérieures, le processus de chargement est appelé `MinidumpUploader.exe` et le journal est moins détaillé.

Dans l’exemple précédent, la clé d’instrumentation est `c12a605e73c44346a984e00000000000`. Cette valeur doit correspondre à la clé d’instrumentation de votre application.
Le fichier minidump est associé à une capture instantanée dont l’ID est `139e411a23934dc0b9ea08a626db16c5`. Vous pouvez utiliser cet ID ultérieurement pour Rechercher l’enregistrement d’exception associé dans Application Insights Analytics.

Le chargeur analyse les nouveaux fichiers PDB environ toutes les 15 minutes. Voici un exemple :

```
SnapshotUploader.exe Information: 0 : PDB rescan requested.
    DateTime=2018-03-09T01:47:19.4457768Z
SnapshotUploader.exe Information: 0 : Scanning D:\home\site\wwwroot for local PDBs.
    DateTime=2018-03-09T01:47:19.4457768Z
SnapshotUploader.exe Information: 0 : Local PDB scan complete. Found 2 PDB(s).
    DateTime=2018-03-09T01:47:19.4614027Z
SnapshotUploader.exe Information: 0 : Deleted PDB scan marker : D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\6368.pdbscan
    DateTime=2018-03-09T01:47:19.4614027Z
```

Pour les applications qui ne sont _pas_ hébergées dans App Service, les fichiers journaux d’activité du chargeur figurent dans le même dossier que les fichiers minidump : `%TEMP%\Dumps\<ikey>` (où `<ikey>` est votre clé d’instrumentation).

## <a name="troubleshooting-cloud-services"></a>Dépannage de Cloud Services
Dans Cloud Services, le dossier temporaire par défaut peut être trop petit pour contenir les fichiers minidump, conduisant à des pertes d’instantanés.

L’espace nécessaire varie selon le jeu de travail total de votre application et le nombre d’instantanés simultanés.

Le jeu de travail du rôle web ASP.NET 32 bits contient en général de 200 à 500 Mo. Autorisez au moins deux instantanés simultanés.

Par exemple, si votre application utilise un jeu de travail de 1 Go au total, vous devez vous assurer qu’il y a au moins 2 Go d’espace disque pour stocker les instantanés.

Suivez ces étapes pour configurer votre rôle service cloud avec une ressource locale dédiée pour les instantanés.

1. Ajoutez une nouvelle ressource locale à votre service cloud en modifiant le fichier de définition (.csdef) du service cloud. L’exemple suivant définit une ressource appelée `SnapshotStore` avec une taille de 5 Go.
   ```xml
   <LocalResources>
     <LocalStorage name="SnapshotStore" cleanOnRoleRecycle="false" sizeInMB="5120" />
   </LocalResources>
   ```

2. Modifiez le code de démarrage de votre rôle pour ajouter une variable d’environnement qui pointe vers la ressource locale `SnapshotStore`. Dans le cas des rôles de travail, le code doit être ajouté à la méthode `OnStart` du rôle :
   ```csharp
   public override bool OnStart()
   {
       Environment.SetEnvironmentVariable("SNAPSHOTSTORE", RoleEnvironment.GetLocalResource("SnapshotStore").RootPath);
       return base.OnStart();
   }
   ```
   Dans le cas des rôles Web (ASP.NET), le code doit être ajouté à la méthode `Application_Start` de l’application web :
   ```csharp
   using Microsoft.WindowsAzure.ServiceRuntime;
   using System;

   namespace MyWebRoleApp
   {
       public class MyMvcApplication : System.Web.HttpApplication
       {
          protected void Application_Start()
          {
             Environment.SetEnvironmentVariable("SNAPSHOTSTORE", RoleEnvironment.GetLocalResource("SnapshotStore").RootPath);
             // TODO: The rest of your application startup code
          }
       }
   }
   ```

3. Mettez à jour le fichier ApplicationInsights.config de votre rôle pour remplacer l’emplacement de dossier temporaire utilisé par `SnapshotCollector`
   ```xml
   <TelemetryProcessors>
    <Add Type="Microsoft.ApplicationInsights.SnapshotCollector.SnapshotCollectorTelemetryProcessor, Microsoft.ApplicationInsights.SnapshotCollector">
      <!-- Use the SnapshotStore local resource for snapshots -->
      <TempFolder>%SNAPSHOTSTORE%</TempFolder>
      <!-- Other SnapshotCollector configuration options -->
    </Add>
   </TelemetryProcessors>
   ```

## <a name="overriding-the-shadow-copy-folder"></a>Remplacement du dossier des clichés instantanés

Lorsque le collecteur de captures instantanées démarre, il essaie de rechercher un dossier sur le disque convenant pour l’exécution du processus du chargeur des captures instantanées. Le dossier choisi est appelé dossier des clichés instantanés.

Le collecteur de captures instantanées vérifie quelques emplacements connus, en s’assurant qu’il dispose des autorisations pour copier les fichiers binaires du chargeur des captures instantanées. Les variables d'environnement suivantes sont utilisées :
- Fabric_Folder_App_Temp
- LOCALAPPDATA
- APPDATA
- TEMP

Si aucun dossier approprié n’est trouvé, Snapshot Collector signale une erreur avec le message _« Impossible de trouver un dossier approprié de captures instantanées »_ .

Si la copie échoue, le collecteur de captures instantanées indique une erreur `ShadowCopyFailed`.

Si le chargeur ne peut pas être lancé, le collecteur de captures instantanées indique une erreur `UploaderCannotStartFromShadowCopy`. Le corps du message comprend souvent `System.UnauthorizedAccessException`. Cette erreur se produit généralement lorsque l’application est exécutée avec un compte disposant d’autorisations réduites. Le compte a l’autorisation d’écrire dans le dossier des clichés instantanés, mais il n’est pas autorisé à exécuter du code.

Étant donné que ces erreurs se produisent généralement lors du démarrage, elles seront généralement suivies d’une erreur `ExceptionDuringConnect` indiquant _« Échec du démarrage du chargeur »._

Pour contourner ces erreurs, vous pouvez spécifier manuellement le dossier des clichés instantanés avec l’option de configuration `ShadowCopyFolder`. Par exemple, en utilisant ApplicationInsights.config :

   ```xml
   <TelemetryProcessors>
    <Add Type="Microsoft.ApplicationInsights.SnapshotCollector.SnapshotCollectorTelemetryProcessor, Microsoft.ApplicationInsights.SnapshotCollector">
      <!-- Override the default shadow copy folder. -->
      <ShadowCopyFolder>D:\SnapshotUploader</ShadowCopyFolder>
      <!-- Other SnapshotCollector configuration options -->
    </Add>
   </TelemetryProcessors>
   ```

Ou bien, si vous utilisez appsettings.json avec une application .NET Core :

   ```json
   {
     "ApplicationInsights": {
       "InstrumentationKey": "<your instrumentation key>"
     },
     "SnapshotCollectorConfiguration": {
       "ShadowCopyFolder": "D:\\SnapshotUploader"
     }
   }
   ```

## <a name="use-application-insights-search-to-find-exceptions-with-snapshots"></a>Utilisez une recherche Application Insights pour trouver des exceptions avec des captures instantanées

Quand un instantané est créé, l’exception levée est marquée avec un ID d’instantané. Cet ID d’instantané est inclus en tant que propriété personnalisée lorsque l’exception est signalée à Application Insights. La fonction **Rechercher** dans Application Insights vous permet de trouver tous les enregistrements ayant la propriété personnalisée `ai.snapshot.id`.

1. Parcourez votre ressource Application Insights dans le portail Azure.
2. Sélectionnez **Recherche**.
3. Tapez `ai.snapshot.id` dans la zone de texte Rechercher, puis appuyez sur Entrée.

![Rechercher une télémétrie avec un ID d’instantané dans le portail](./media/snapshot-debugger/search-snapshot-portal.png)

Si cette recherche ne retourne aucun résultat, cela signifie qu’aucun instantané n’a été signalé à Application Insights dans l’intervalle de temps sélectionné.

Pour rechercher un ID d’instantané spécifique dans les fichiers journaux d’activité du chargeur, tapez cet ID dans la zone de recherche. Si vous ne trouvez aucun enregistrement d’une capture instantanée dont vous savez qu’elle a été chargée, procédez comme suit :

1. Vérifiez soigneusement que vous examinez la ressource Application Insights appropriée en vérifiant la clé d’instrumentation.

2. À l’aide de l’horodateur du fichier journal du chargeur, ajustez le filtre Intervalle de temps de la recherche pour couvrir cet intervalle.

Si vous ne voyez toujours pas d’exception avec cet ID d’instantané, cela signifie que l’enregistrement de l’exception n’a pas été signalé à Application Insights. Cette situation peut se produire si votre application s’est arrêtée anormalement après avoir pris la capture instantanée, mais avant d’avoir signalé l’enregistrement de l’exception. Dans ce cas, vérifiez les journaux d’activité d’App Service sous `Diagnose and solve problems` pour voir si des redémarrages intempestifs ou des exceptions non prises en charge se sont produits.

## <a name="edit-network-proxy-or-firewall-rules"></a>Modifier les règles de pare-feu ou de proxy réseau

Si votre application se connecte à Internet via un proxy ou un pare-feu, il se peut que vous deviez mettre à jour les règles pour communiquer avec le service Débogueur de capture instantanée.

Les adresses IP utilisées par Débogueur de capture instantanée Application Insights sont incluses dans l’étiquette de service Azure Monitor. Pour plus d’informations, consultez [Documentation relative aux étiquettes de service](https://docs.microsoft.com/azure/virtual-network/service-tags-overview).
