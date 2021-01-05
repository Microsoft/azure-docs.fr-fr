---
title: Notes de publication pour le package NuGet Microsoft.ApplicationInsights.SnapshotCollector - Application Insights
description: Notes de publication pour le package NuGet Microsoft.ApplicationInsights.SnapshotCollector utilisé par le Débogueur de capture instantanée Application Insights
ms.topic: conceptual
author: pharring
ms.author: pharring
ms.date: 11/10/2020
ms.openlocfilehash: 73fea1e1928cf4e1bd5342aa0a4c885ccb5cf137
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96548169"
---
# <a name="release-notes-for-microsoftapplicationinsightssnapshotcollector"></a>Notes de publication pour Microsoft.ApplicationInsights.SnapshotCollector

Cet article contient les notes de publication relatives au package NuGet Microsoft.ApplicationInsights.SnapshotCollector pour les applications .NET, qui est utilisé par le Débogueur de capture instantanée Application Insights.

[Découvrez-en](./snapshot-debugger.md) plus sur le Débogueur de capture instantanée Azure Application Insights pour les applications .NET.

Pour obtenir des rapports de bogues et des commentaires, ouvrez un problème sur GitHub à l’adresse https://github.com/microsoft/ApplicationInsights-dotnet.

## <a name="release-notes"></a>Notes de publication

## <a name="1374"></a>[1.3.7.4](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.3.7.4)
Version intermédiaire qui résout un problème découvert lors du test du scénario d'attachement sans code d'Azure App Service.
### <a name="changes"></a>Modifications
- La cible netcoreapp3.0 dépend désormais de Microsoft.ApplicationInsights.AspNetCore version 2.1.1 (anciennement 2.1.2).

## <a name="1373"></a>[1.3.7.3](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.3.7.3)
Une version mineure corrective qui résout quelques problèmes à fort impact.
### <a name="bug-fixes"></a>Résolution des bogues
- Correction de la découverte de fichiers PDB dans le dossier wwwroot/bin, qui a été rompue lors de la modification de l’algorithme de recherche de symboles dans la version 1.3.6.
- Correction de l’exception ExtractWasCalledMultipleTimesException parasite dans les données de télémétrie.

## <a name="137"></a>[1.3.7](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.3.7)
### <a name="changes"></a>Modifications
- La cible netcoreapp2.0 de SnapshotCollector dépend de Microsoft.ApplicationInsights.AspNetCore version 2.1.1 ou supérieure (de nouveau). Cela rétablit le comportement antérieur à la version 1.3.5. Nous avons essayé de la mettre à niveau dans la version 1.3.6, mais cela a rompu certains scénarios Azure App Service.
### <a name="new-features"></a>Nouvelles fonctionnalités
- Snapshot Collector lit et analyse ConnectionString à partir de la variable d’environnement APPLICATIONINSIGHTS_CONNECTION_STRING ou de TelemetryConfiguration. Cela sert essentiellement à définir le point de terminaison pour la connexion au service de capture instantanée. Pour plus d’informations, consultez la [documentation sur les chaînes de connexion](./sdk-connection-string.md).
### <a name="bug-fixes"></a>Résolution des bogues
- Basculement vers l’utilisation de HttpClient pour toutes les cibles, à l’exception de Net45, car WebRequest échouait dans certains environnements en raison d’une incompatibilité de SecurityProtocol (nécessite TLS 1.2).

## <a name="136"></a>[1.3.6](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.3.6)
### <a name="changes"></a>Modifications
- SnapshotCollector dépend maintenant de Microsoft.ApplicationInsights version 2.5.1 ou supérieure pour toutes les versions cibles de .Net Framework. Il peut s’agir d’un changement cassant si votre application dépend d’une version antérieure du SDK Microsoft.ApplicationInsights.
- Suppression de la prise en charge de TLS 1.0 et 1.1 dans le chargeur de capture instantanée.
- La période des analyses PDB est désormais de 24 heures par défaut, au lieu de 15 minutes. Configurable par le biais de PdbRescanInterval sur SnapshotCollectorConfiguration.
- L’analyse PDB recherche uniquement les dossiers de niveau supérieur, au lieu de procéder de manière récursive. Il peut s’agir d’un changement cassant si vos symboles se trouvent dans des sous-dossiers du dossier binaire.
### <a name="new-features"></a>Nouvelles fonctionnalités
- Rotation des journaux dans SnapshotUploader pour éviter de remplir le dossier des journaux avec des fichiers anciens.
- Prise en charge de la désoptimisation (par le biais de ReJIT on attach) pour les applications .NET Core 3.0.
- Ajout de symboles au package NuGet.
- Définition de métadonnées supplémentaires lors du chargement de minidumps.
- Ajout d’une propriété Initialized à SnapshotCollectorTelemetryProcessor. Il s’agit d’un CancellationToken, qui est annulé quand Snapshot Collector est complètement initialisé et connecté au point de terminaison de service.
- Les captures instantanées peuvent désormais être prises pour les exceptions dans les méthodes générées dynamiquement. Par exemple, les arborescences d’expressions compilées générées par des requêtes Entity Framework.
### <a name="bug-fixes"></a>Résolution des bogues
- Chargement de Snapshot Collector par AmbiguousMatchException en raison de Status Monitor.
- La méthode d’extension GetSnapshotCollector effectue désormais une recherche dans tous les TelemetrySinks.
- Non-démarrage du chargeur de capture instantanée sur les plateformes non prises en charge.
- Gestion d’InvalidOperationException lors de la désoptimisation des méthodes dynamiques (par exemple, Entity Framework)

## <a name="135"></a>[1.3.5](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.3.5)
- Ajout de la prise en charge des clouds souverains (les anciennes versions ne fonctionnent pas dans les clouds souverains)
- Simplification de l’ajout du collecteur de captures instantanées à l’aide d’AddSnapshotCollector(). Des informations supplémentaires sont disponibles [ici](./snapshot-debugger-appservice.md).
- Utilisation du paramètre MD5 FISMA pour vérifier les blocs d’objets blob. Cela permet d’éviter l’algorithme de chiffrement MD5 .NET par défaut, qui n’est pas disponible quand le système d’exploitation est défini sur le mode compatible FIPS.
- Les frames .NET Framework sont ignorés lors de la désoptimisation des appels de fonction. Ce comportement peut être contrôlé par le paramètre de configuration DeoptimizeIgnoredModules.
- Ajout du paramètre de configuration `DeoptimizeMethodCount` qui permet la désoptimisation de plusieurs appels de fonction. Plus d’informations ici

## <a name="134"></a>[1.3.4](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.3.4)
- Autorisation des clés d’instrumentation structurées.
- Augmentation de la robustesse de SnapshotUploader : poursuite du démarrage même si les anciens journaux du chargeur ne peuvent pas être déplacés.
- Réactivation du signalement de données de télémétrie supplémentaires quand SnapshotUploader.exe se termine immédiatement (était désactivé dans la version 1.3.3).
- Simplification de la télémétrie interne.
- _Fonctionnalité expérimentale_ : Plans de collecte par snappoint : ajout de « snapshotOnFirstOccurence ». Plus d’informations disponibles [ici](https://gist.github.com/alexaloni/5b4d069d17de0dabe384ea30e3f21dfe).

## <a name="133"></a>[1.3.3](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.3.3)
- Correction du bogue qui empêchait SnapshotUploader.exe de répondre et de charger les captures instantanées pour les applications .NET Core.

## <a name="132"></a>[1.3.2](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.3.2)
- _Fonctionnalité expérimentale_ : Plans de collecte par snappoint. Plus d’informations disponibles [ici](https://gist.github.com/alexaloni/5b4d069d17de0dabe384ea30e3f21dfe).
- SnapshotUploader.exe se ferme quand le runtime décharge l’AppDomain à partir duquel SnapshotCollector est chargé, au lieu d’attendre que le processus se termine. Cela améliore la fiabilité du collecteur quand il est hébergé dans IIS.
- Ajout d’une configuration qui permet à plusieurs instances SnapshotCollector utilisant la même clé d’instrumentation de partager le même processus SnapshotUploader : ShareUploaderProcess (valeur par défaut `true`).
- Signalement de données de télémétrie supplémentaires quand SnapshotUploader.exe se termine immédiatement.
- Réduction du nombre de fichiers de prise en charge dont SnapshotUploader.exe a besoin pour écrire sur le disque.

## <a name="131"></a>[1.3.1](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.3.1)
- Suppression de la prise en charge de la collecte des captures instantanées avec l’API RtlCloneUserProcess et prise en charge uniquement de l’API PssCaptureSnapshots.
- Augmentation de 1 à 3 de la limite par défaut du nombre de captures instantanées pouvant être capturés en 10 minutes.
- SnapshotUploader.exe est autorisé à négocier TLS 1.1 et 1.2.
- Signalement de données de télémétrie supplémentaires quand SnapshotUploader journalise un avertissement ou une erreur
- Arrêt de la prise de captures instantanées quand le service back-end signale que le quota quotidien a été atteint (50 captures instantanées par jour)
- Ajout d’une vérification supplémentaire dans SnapshotUploader.exe pour empêcher que deux instances s’exécutent en même temps.

## <a name="130"></a>[1.3.0](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.3.0)
### <a name="changes"></a>Modifications
- Pour les applications ciblant .NET Framework, Snapshot Collector dépend maintenant de Microsoft.ApplicationInsights version 2.3.0 ou supérieure.
Il dépendait auparavant des versions 2.2.0 ou supérieures.
Nous pensons qu’il ne s’agit pas d’un problème pour la plupart des applications, mais contactez-nous si cette modification vous empêche d’utiliser la dernière version de Snapshot Collector.
- Utilisation de délais de temporisation exponentiels dans le chargeur de capture instantanée lors de nouvelles tentatives de chargement.
- Utilisation de ServerTelemetryChannel (si disponible) pour la création de rapports plus fiables sur les données de télémétrie.
- Utilisation de « SdkInternalOperationsMonitor » lors de la connexion initiale au service Débogueur de capture instantanée afin qu’il soit ignoré par le suivi des dépendances.
- Amélioration de la télémétrie autour de la connexion initiale au service Débogueur de capture instantanée.
- Signalement de données de télémétrie supplémentaires pour les éléments suivants :
  - Version Azure App Service.
  - Instances de calcul Azure.
  - Conteneurs.
  - Application Azure Function.
### <a name="bug-fixes"></a>Résolution des bogues
- Quand l’intervalle de réinitialisation du compteur de problèmes est défini sur 24 jours, cela est interprété comme étant 24 heures.
- Correction d’un bogue lié au fait que le chargeur de capture instantanée arrêtait de traiter de nouvelles captures instantanées en cas d’exception lors de la suppression d’une capture instantanée.

## <a name="123"></a>[1.2.3](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.2.3)
- Correction de la signature avec nom fort avec les binaires du chargeur de capture instantanée.

## <a name="122"></a>[1.2.2](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.2.2)
### <a name="changes"></a>Modifications
- Les fichiers nécessaires pour SnapshotUploader(64).exe sont désormais incorporés en tant que ressources dans la DLL principale. Cela signifie que le dossier SnapshotCollectorFiles n’est plus créé, ce qui simplifie la génération et le déploiement, et réduit l’encombrement dans l’Explorateur de solutions. Lors de la mise à niveau, veillez à passer en revue les modifications apportées à votre fichier `.csproj`. Le fichier `Microsoft.ApplicationInsights.SnapshotCollector.targets` n’est plus nécessaire.
- La télémétrie est journalisée dans votre ressource Application Insights même si ProvideAnonymousTelemetry est défini sur false. Nous pouvons ainsi implémenter une fonctionnalité de contrôle d’intégrité dans le portail Azure. ProvideAnonymousTelemetry n’affecte que les données de télémétrie envoyées à Microsoft pour le support et l’amélioration du produit.
- Quand TempFolder ou ShadowCopyFolder sont redirigés vers des variables d’environnement, le collecteur est maintenu inactif jusqu’à ce que ces variables d’environnement soient définies.
- Pour les applications qui se connectent à Internet via un serveur proxy, Snapshot Collector détecte désormais automatiquement les paramètres de proxy et les transmet à SnapshotUploader.exe.
- Réduction de la priorité du processus SnapshotUplaoder (dans la mesure du possible). Cette priorité peut être remplacée à l’aide de l’option IsLowPrioirtySnapshotUploader.
- Ajout d’une méthode d’extension GetSnapshotCollector sur TelemetryConfiguration pour les scénarios où vous souhaitez configurer Snapshot Collector par programmation.
- Définition de la version du SDK Application Insights (au lieu de la version de l’application) dans la télémétrie côté client.
- Envoi du premier événement de pulsation au bout de deux minutes.
### <a name="bug-fixes"></a>Résolution des bogues
- Correction de l’exception NullReferenceException quand des exceptions ont des dictionnaires de données Null ou immuables.
- Dans le chargeur, la mise en correspondance des fichiers PDB fait l’objet de plusieurs nouvelles tentatives en cas de violation de partage.
- Correction des doublons de données de télémétrie quand plusieurs threads appellent le pipeline de télémétrie au démarrage.

## <a name="121"></a>[1.2.1](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.2.1)
### <a name="changes"></a>Modifications
- Les fichiers de commentaires de documentation XML sont désormais inclus dans le package NuGet.
- Ajout d’une méthode d’extension ExcludeFromSnapshotting sur `System.Exception` pour les scénarios où vous savez que vous avez une exception parasite et que vous souhaitez éviter de créer des captures instantanées pour celle-ci.
- Ajout de la propriété de configuration IsEnabledWhenProfiling, dont la valeur par défaut est true. Il s’agit d’un changement par rapport aux versions précédentes où la création de captures instantanées était temporairement désactivée si Application Insights Profiler exécutait une collecte détaillée. Vous pouvez récupérer l’ancien comportement en affectant à cette propriété la valeur false.
### <a name="bug-fixes"></a>Résolution des bogues
- Signature de SnapshotUploader64.exe de façon correcte.
- Protection contre la double initialisation du processeur de télémétrie.
- La double journalisation des données de télémétrie est évitée dans les applications avec plusieurs pipelines.
- Correction d’un bogue lié à l’heure d’expiration d’un plan de collecte, qui pouvait empêcher les captures instantanées au bout de 24 heures.

## <a name="120"></a>[1.2.0](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.2.0)
Le changement le plus important de cette version (qui a motivé le passage à un nouveau numéro de version mineure) est une réécriture du pipeline de création et de gestion des captures instantanées. Dans les versions précédentes, cette fonctionnalité était implémentée en code natif (ProductionBreakpoints *.dll et SnapshotHolder*.exe). La nouvelle implémentation est en totalité du code managé avec des appels P/Invoke. Pour cette première version utilisant le nouveau pipeline, nous ne nous sommes pas éloignés du comportement d’origine. La nouvelle implémentation permet une meilleure génération de rapports d’erreurs et ouvre la porte à des améliorations futures.

### <a name="other-changes-in-this-version"></a>Autres modifications apportées dans cette version
- MinidumpUploader.exe a été renommé SnapshotUploader.exe (ou SnapshotUploader64.exe).
- Ajout d’une télémétrie de chronométrage aux demandes de désoptimisation et de réoptimisation.
- Ajout de la compression gzip pour les chargements de minidump.
- Résolution d’un problème lié au verrouillage des fichiers PDB, empêchant une mise à niveau de site.
- Journalisation du nom du dossier d’origine (SnapshotCollectorFiles) lors de l’exécution de clichés instantanés.
- Ajustement des limites de mémoire pour les processus 64 bits afin d’empêcher les redémarrages de site en raison de mémoire insuffisante.
- Résolution d’un problème lié au fait que les captures instantanées étaient toujours collectées, même après la désactivation de leur création.
- Journalisation des événements de pulsation dans la ressource AI du client.
- Amélioration de la vitesse des captures instantanées grâce à la suppression de « Source » de l’identificateur du problème.

## <a name="112"></a>[1.1.2](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.1.2)
### <a name="changes"></a>Modifications
Augmentation des données de télémétrie d’utilisation
- Détection et signalement de la version et du système d’exploitation .NET
- Détection et signalement des environnements Azure supplémentaires (service cloud, Service Fabric)
- Enregistrement et signalement des métriques d’exception (nombre d’exceptions de première chance et nombre d’appels TrackException) dans les données de télémétrie de pulsation.
### <a name="bug-fixes"></a>Résolution des bogues
- Correction de la gestion de SqlException où l’exception interne (Win32Exception) n’est pas levée.
- Suppression des espaces à droite des dossiers de symboles, qui entraînaient une analyse incorrecte des arguments de ligne de commande pour MinidumpUploader.
- Les échecs de connexion au point de terminaison de l’agent du Débogueur de capture instantanée ne font plus l’objet de nouvelles tentatives infinies.

## <a name="110"></a>[1.1.0](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/1.1.0)
### <a name="changes"></a>Modifications
- Renforcement de la protection de la mémoire hôte. Cette fonctionnalité réduit l’impact sur la mémoire de l’ordinateur hôte.
- Amélioration de l’expérience d’affichage des captures instantanées dans le portail Azure.