---
title: Débogueur de capture instantanée Azure Application Insights pour les applications .NET
description: Des captures instantanées de débogage sont collectées automatiquement lorsque des exceptions sont levées dans des applications .NET de production
ms.topic: conceptual
ms.custom: devx-track-dotnet
ms.date: 10/23/2019
author: cweining
ms.author: cweining
ms.reviewer: cweining
ms.openlocfilehash: 5913f5fa8d45e6bf92d6132468e0e3bf7a121c65
ms.sourcegitcommit: d79513b2589a62c52bddd9c7bd0b4d6498805dbe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/18/2020
ms.locfileid: "97673516"
---
# <a name="debug-snapshots-on-exceptions-in-net-apps"></a>Captures instantanées de débogage sur exceptions levées dans des applications .NET
Quand une exception se produit, vous pouvez collecter automatiquement une capture instantanée de débogage à partir de votre application web dynamique. La capture instantanée indique l’état du code source et des variables au moment où l’exception a été levée. Le Débogueur de capture instantanée d’[Azure Application Insights](./app-insights-overview.md) analyse la télémétrie des exceptions à partir de votre application web. Il collecte des captures instantanées sur les principales exceptions levées afin que vous disposiez des informations dont vous avez besoin pour diagnostiquer des problèmes de production. Incluez le [package NuGet de collecte des captures instantanées](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) dans votre application, et configurez éventuellement les paramètres de collecte dans [ApplicationInsights.config](./configuration-with-applicationinsights-config.md). Les captures instantanées apparaissent sur les [exceptions](./asp-net-exceptions.md) dans le portail Application Insights.

Vous pouvez afficher les captures instantanées de débogage dans le portail pour consulter la pile des appels et inspecter les variables à chaque frame de pile des appels. Pour améliorer la puissance de débogage du code source, ouvrez les instantanés avec Visual Studio 2019 Entreprise. Dans Visual Studio, vous pouvez également [définir des points de capture instantanée pour prendre des captures instantanées de manière interactive](/visualstudio/debugger/debug-live-azure-applications) sans attendre la levée d’une exception.

Les captures instantanées de débogage sont stockées pendant 15 jours. Cette stratégie de rétention est définie application par application. Si vous devez augmenter cette valeur, faites-en la demande en ouvrant une demande de support dans le portail Azure.

## <a name="enable-application-insights-snapshot-debugger-for-your-application"></a>Activer le Débogueur de capture instantanée Application Insights pour votre application
La collecte de captures instantanées est disponible pour :
* les applications .NET framework et ASP.NET exécutant .NET Framework 4.5 ou version ultérieure ;
* les applications .NET core 2.0 et ASP.NET Core 2.0 s’exécutant sous Windows.

Les environnements suivants sont pris en charge :

* [Azure App Service](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Cloud Services](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) exécutant la famille de systèmes d’exploitation 4 ou ultérieur
* [Services Azure Service Fabric](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) exécutant Windows Server 2012 R2 ou ultérieur
* [Machines virtuelles Azure et groupes de machines virtuelles identiques](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) exécutant Windows Server 2012 R2 ou ultérieur
* [Machines locales physiques ou virtuelles](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json) exécutant Windows Server 2012 R2 ou version ultérieure, ou Windows 8.1 ou version ultérieure

> [!NOTE]
> Les applications clientes (par exemple, WPF, Windows Forms ou UWP) ne sont pas prises en charge.

Si vous avez activé le Débogueur de capture instantanée et que vous ne voyez pas les instantanés, consultez notre [Guide de résolution des problèmes](snapshot-debugger-troubleshoot.md?toc=/azure/azure-monitor/toc.json).

## <a name="grant-permissions"></a>Accorder des autorisations

L’accès aux captures instantanées est protégé par le contrôle d’accès en fonction du rôle Azure (Azure RBAC). Pour que vous puissiez examiner une capture instantanée, un propriétaire d’abonnement doit d’abord vous ajouter au rôle nécessaire.

> [!NOTE]
> Les propriétaires et contributeurs ne disposent pas automatiquement de ce rôle. S’ils souhaitent afficher des captures instantanées, ils doivent s’ajouter eux-mêmes au rôle.

Les propriétaires d’abonnement doivent attribuer le rôle `Application Insights Snapshot Debugger` aux utilisateurs chargés d’examiner les captures instantanées. Ce rôle peut être attribué à des utilisateurs ou à des groupes par les propriétaires d’abonnements pour la ressource Application Insights cible, ou pour son groupe de ressources ou son abonnement.

1. Accédez à la ressource Application Insights dans le portail Azure.
1. Cliquez sur **Contrôle d’accès (IAM)** .
1. Cliquez sur le bouton **+Ajouter une attribution de rôle**.
1. Sélectionnez **Débogueur de capture instantanée d’Application Insights** dans la liste déroulante **Rôles**.
1. Entrez le nom de l’utilisateur à ajouter.
1. Cliquez sur le bouton **Enregistrer** pour ajouter l’utilisateur au rôle.


> [!IMPORTANT]
> Les captures instantanées des valeurs de variables et de paramètres peuvent contenir des informations personnelles ou sensibles.

## <a name="view-snapshots-in-the-portal"></a>Afficher les instantanés dans le portail

Lorsqu’une exception s’est produite dans votre application et qu’un instantané a été créé, des captures instantanées devraient s’afficher. Il peut s’écouler 5 à 10 minutes entre le moment où une exception se produit et celui où un instantané est prêt et peut être consulté à partir du portail. Pour afficher les captures instantanées, dans le volet **Échec**, sélectionnez le bouton **Opérations** lors de l’affichage de l’onglet **Opérations**, ou sélectionnez le bouton **Exceptions** lorsque vous affichez l’onglet **Exceptions** :

![Page Échecs](./media/snapshot-debugger/failures-page.png)

Sélectionnez une opération ou une exception dans le volet droit pour ouvrir le volet **Détails de la transaction de bout en bout**, puis sélectionnez l’événement d’exception. Si une capture instantanée est disponible pour l’exception donnée, le bouton **Ouvrir la capture instantanée de débogage** s’affiche dans le volet de droite avec les détails de [l’exception](./asp-net-exceptions.md).

![Bouton Ouvrir la capture instantanée de débogage sur l’exception](./media/snapshot-debugger/e2e-transaction-page.png)

Une pile d’appel et un volet de variables s’affichent dans la vue Capture instantanée de débogage. Quand vous sélectionnez des images de la pile d’appel dans le volet correspondant, vous voyez les variables et les paramètres locaux de cet appel de fonction dans le volet des variables.

![Afficher la capture instantanée de débogage dans le portail](./media/snapshot-debugger/open-snapshot-portal.png)

Les captures instantanées peuvent contenir des informations sensibles. Par défaut, elles ne sont pas visibles. Pour afficher les captures instantanées, le rôle `Application Insights Snapshot Debugger` doit vous être attribué.

## <a name="view-snapshots-in-visual-studio-2017-enterprise-or-above"></a>Afficher les instantanés dans Visual Studio 2017 Enterprise ou une version ultérieure
1. Cliquez sur le bouton **Télécharger la capture instantanée** pour télécharger un fichier `.diagsession`, qui peut être ouvert par Visual Studio Enterprise.

2. Pour ouvrir le fichier `.diagsession`, vous devez avoir installé le composant Débogueur de capture instantanée Visual Studio. Le composant Débogueur de capture instantanée est un composant obligatoire de la charge de travail ASP.NET dans Visual Studio qui peut être sélectionné dans la liste des composants individuels de Visual Studio Installer. Si vous utilisez une version de Visual Studio antérieure à Visual Studio 2017 version 15.5, vous devez installer l’extension à partir de [Visual Studio Marketplace](https://aka.ms/snapshotdebugger).

3. Une fois le fichier de capture instantanée ouvert, la page de débogage de minidump s’affiche dans Visual Studio. Cliquez sur **Debug Managed Code** (Déboguer le code managé) pour démarrer le débogage de la capture instantanée. La capture instantanée s’ouvre sur la ligne de code où l’exception a été levée et vous permet de déboguer l’état actuel du processus.

    ![Afficher la capture instantanée de débogage dans Visual Studio](./media/snapshot-debugger/open-snapshot-visualstudio.png)

La capture instantanée téléchargée contient tous les fichiers de symboles détectés sur votre serveur d’applications web. Ces fichiers de symboles sont nécessaires pour associer les données de capture instantanée au code source. Pour les applications App Service, veillez à activer le déploiement des symboles lorsque vous publiez vos applications web.

## <a name="how-snapshots-work"></a>Fonctionnement des captures instantanées

Le collecteur de captures instantanées est implémenté en tant que [Processeur d’Application Insights Telemetry](./configuration-with-applicationinsights-config.md#telemetry-processors-aspnet). Lorsque votre application s’exécute, le processeur de télémétrie du collecteur de captures instantanées est ajouté au pipeline de télémétrie de votre application.
Chaque fois que votre application appelle [TrackException](./asp-net-exceptions.md#exceptions), le collecteur de captures instantanées calcule un ID de problème à partir du type de l’exception levée et de la méthode de levée de l’exception.
Chaque fois que votre application appelle TrackException, un compteur est incrémenté pour l’ID de problème approprié. Lorsque le compteur atteint la valeur `ThresholdForSnapshotting`, l’ID de problème est ajouté à un Plan de collecte.

Le collecteur de captures instantanées analyse également les exceptions à mesure qu’elles sont levées en s’abonnant à l’événement [AppDomain.CurrentDomain.FirstChanceException](/dotnet/api/system.appdomain.firstchanceexception). Lorsque cet événement se déclenche, l’ID de problème de l’exception est calculé et comparé aux ID de problème figurant dans le Plan de collecte.
S’il existe une correspondance, un instantané du processus en cours d’exécution est créé. Un identificateur unique est attribué à l’instantané et l’exception est marquée avec cet identificateur. Après fois que le gestionnaire FirstChanceException a retourné l’exception levée, celle-ci est traitée normalement. Finalement, l’exception atteint à nouveau la méthode TrackException où elle est signalée à Application Insights avec l’identificateur de capture instantanée.

Le processus principal continue à s’exécuter et à assurer le trafic pour les utilisateurs avec une courte interruption. Pendant ce temps, l’instantané est remis au processus du chargeur de capture instantanée. Le chargeur de capture instantanée crée un minidump et le charge sur Application Insights, ainsi que tous les fichiers de symboles (.pdb) pertinents.

> [!TIP]
> - Une capture instantanée de processus est un clone suspendu du processus en cours d’exécution.
> - La création de l’instantané prend environ de 10 à 20 millisecondes.
> - La valeur par défaut de `ThresholdForSnapshotting` est 1. Il s’agit également la valeur minimale. Par conséquent, votre application doit déclencher la même exception **à deux reprises** avant qu’un instantané soit créé.
> - Définissez `IsEnabledInDeveloperMode` sur true si vous voulez générer des captures instantanées lors d’un débogage dans Visual Studio.
> - Le taux de création de capture instantanée est limité par le paramètre `SnapshotsPerTenMinutesLimit`. Par défaut, la limite est d’une capture instantanée toutes les dix minutes.
> - Au maximum 50 captures instantanées peuvent être chargées par jour.

## <a name="limitations"></a>Limites

Par défaut, la période de conservation des données est de 15 jours. Pour chaque instance Application Insights, un nombre maximal de 50 instantanés est autorisé par jour.

### <a name="publish-symbols"></a>Publier des symboles
Le débogueur de captures instantanées nécessite que des fichiers de symboles se trouvent sur le serveur de production pour le décodage des variables et pour fournir une fonctionnalité de débogage dans Visual Studio.
La version 15.2 de Visual Studio 2017 (ou ultérieure) publie les symboles des builds de mise en production par défaut lors de sa publication dans App Service. Dans les versions antérieures, vous devez ajouter la ligne suivante à votre fichier de profil de publication `.pubxml` afin que les symboles soient publiés en mode Mise en production :

```xml
    <ExcludeGeneratedDebugSymbol>False</ExcludeGeneratedDebugSymbol>
```

Pour Calcul Azure et d’autres types, vérifiez que les fichiers de symboles se trouvent dans le même dossier que le fichier .dll de l’application principale (généralement `wwwroot/bin`), ou sont disponibles dans le chemin d’accès actuel.

> [!NOTE]
> Pour plus d’informations sur les différentes options de symbole disponibles, voir la [documentation de Visual Studio](/visualstudio/ide/reference/advanced-build-settings-dialog-box-csharp?view=vs-2019#output
). Pour optimiser les résultats, nous vous recommandons d’utiliser les options « Full », « Portable » ou « Embedded ».

### <a name="optimized-builds"></a>Optimisation des versions
Dans certains cas, des variables locales sont invisibles dans les builds de mise en production en raison d’optimisations appliquées par le compilateur JIT.
Toutefois, dans Azure App Services, le collecteur de captures instantanées peut « désoptimiser » des méthodes de levée d’exception faisant partie de son Plan de collecte.

> [!TIP]
> Installez l’extension de site Application Insights dans votre App Service pour obtenir un support de « désoptimisation ».

## <a name="next-steps"></a>Étapes suivantes
Activez le Débogueur de capture instantanée Application Insights pour votre application :

* [Azure App Service](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json)
* [Azure Cloud Services](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Services Azure Service Fabric](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Machines virtuelles Azure et Virtual Machine Scale Sets](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)
* [Machines virtuelles ou physiques locales](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json)

Au-delà du Débogueur de capture instantanée d’Application Insights :
 
* [Définir des points de capture instantanée dans votre code](/visualstudio/debugger/debug-live-azure-applications) afin obtenir des captures instantanées sans attendre la levée d’une exception.
* [Diagnostiquer des exceptions dans vos applications web](./asp-net-exceptions.md) explique comment rendre visible à Application Insights un plus grand nombre d’exceptions.
* [Détection intelligente](./proactive-diagnostics.md) permet de détecter automatiquement les anomalies relatives aux performances.