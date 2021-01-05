---
title: Résolution des problèmes de l’extension Diagnostics Azure
description: Résolution des problèmes lors de l'utilisation des diagnostics Azure dans Azure Virtual Machines, Service Fabric ou Cloud Services.
ms.subservice: diagnostic-extension
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/08/2019
ms.openlocfilehash: de42a70cf2950aca3dbe151407671306c793ed10
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86515493"
---
# <a name="azure-diagnostics-troubleshooting"></a>Résolution des problèmes de diagnostics Azure
Cet article contient des informations de dépannage pour Diagnostics Azure. Pour plus d’informations sur les diagnostics Microsoft Azure, voir [Vue d’ensemble des diagnostics Azure](diagnostics-extension-overview.md).

## <a name="logical-components"></a>Composants logiques
**Lanceur du plug-in Diagnostics (DiagnosticsPluginLauncher.exe)**  : Lance l’extension Diagnostics Azure. Sert de point d’entrée de processus.

**Plug-in Diagnostics (DiagnosticsPlugin.exe)**  : configure, exécute et gère la durée de vie de l’agent de supervision. Il s’agit du principal processus exécuté par le lanceur.

**Monitoring Agent (processus MonAgent\*.exe)**  : supervise, collecte et transfère les données de diagnostic.  

## <a name="logartifact-paths"></a>Chemins d’accès des journaux/artefacts
Voici les chemins d’accès de quelques journaux d’activité et artefacts importants. Nous nous référons à ces informations dans le reste du document.

### <a name="azure-cloud-services"></a>Services cloud Azure
| Artefact | Path |
| --- | --- |
| **Fichier de configuration de Diagnostics Azure** | %SystemDrive%\Packages\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\<version>\Config.txt |
| **Fichiers journaux** | C:\Logs\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\<version>\ |
| **Magasin local pour les données de diagnostic** | C:\Resources\Directory\<CloudServiceDeploymentID>.\<RoleName>.DiagnosticStore\WAD0107\Tables |
| **Fichier de configuration de l’agent de surveillance** | C:\Resources\Directory\<CloudServiceDeploymentID>.\<RoleName>.DiagnosticStore\WAD0107\Configuration\MaConfig.xml |
| **Package d’extension Diagnostics Azure** | %SystemDrive%\Packages\Plugins\Microsoft.Azure.Diagnostics.PaaSDiagnostics\<version> |
| **Chemin d’accès à l’utilitaire de collecte des journaux** | %SystemDrive%\Packages\GuestAgent\ |
| **Fichier journal MonAgentHost** | C:\Resources\Directory\<CloudServiceDeploymentID>.\<RoleName>.DiagnosticStore\WAD0107\Configuration\MonAgentHost.<seq_num>.log |

### <a name="virtual-machines"></a>Machines virtuelles
| Artefact | Path |
| --- | --- |
| **Fichier de configuration de Diagnostics Azure** | C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<version>\RuntimeSettings |
| **Fichiers journaux** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\ |
| **Magasin local pour les données de diagnostic** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\WAD0107\Tables |
| **Fichier de configuration de l’agent de surveillance** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\WAD0107\Configuration\MaConfig.xml |
| **Fichier d’état** | C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<version>\Status |
| **Package d’extension Diagnostics Azure** | C:\Packages\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>|
| **Chemin d’accès à l’utilitaire de collecte des journaux** | C:\WindowsAzure\Logs\WaAppAgent.log |
| **Fichier journal MonAgentHost** | C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics\<DiagnosticsVersion>\WAD0107\Configuration\MonAgentHost.<seq_num>.log |

## <a name="metric-data-doesnt-appear-in-the-azure-portal"></a>Les données métriques ne s’affichent pas dans le portail Azure
Diagnostics Azure fournit des données métriques qu’il est possible d’afficher dans le portail Azure. Si vous rencontrez des problèmes liés à l’affichage de ces données dans le portail, consultez la table WADMetrics\* dans le compte de stockage de Diagnostics Azure pour voir si les enregistrements de métriques correspondants sont bien présents et vous assurer que le [fournisseur de ressources](../../azure-resource-manager/management/resource-providers-and-types.md) Microsoft.Insights est inscrit.

Ici, la valeur **PartitionKey** de la table correspond à l’ID de ressource, à la machine virtuelle ou à un groupe de machines virtuelles identiques. La valeur **RowKey** correspond au nom de la métrique (également appelé nom du compteur de performances).

Si l'ID de ressource est incorrect, vérifiez sa configuration dans **Configuration de** **diagnostic** > **Métriques** > **ID de ressource**.

S’il n’existe aucune donnée pour cette métrique en particulier, vérifiez **Configuration de diagnostic** > **PerformanceCounter** pour voir si la métrique (compteur de performances) est bien incluse. Par défaut, nous activons les compteurs suivants :
- \Processor(_Total)\% temps processeur
- \Memory\Octets disponibles
- \ASP.NET Applications(__Total__)\Requests/Sec
- \ASP.NET Applications(__Total__)\Errors Total/Sec
- \ASP.NET\Demandes en file d’attente
- \ASP.NET\Demandes rejetées
- \Processor(w3wp)\% Temps processeur
- \Processus(w3wp) \Octets privés
- \Processus(WaIISHost)\% Temps processeur
- \Processus(WaIISHost)\Octets privés
- \Processus(WaWorkerHost)\% Temps processeur
- \Processus(WaWorkerHost)\Octets privés
- \Mémoire\Défauts de page/s
- \.Mémoire NET CLR(_Global_)\% Temps en GC
- \LogicalDisk(C:)\Disk Write Bytes/sec
- \LogicalDisk(C:)\Disk Read Bytes/sec
- \LogicalDisk(D:)\Disk Write Bytes/sec
- \LogicalDisk(D:)\Disk Read Bytes/sec

Si la configuration est correctement définie, mais que vous ne voyez toujours pas les données métriques, reportez-vous aux indications suivantes. Elles vous aideront à résoudre les problèmes rencontrés.


## <a name="azure-diagnostics-is-not-starting"></a>Diagnostics Azure ne démarre pas
Pour savoir pourquoi Diagnostics Azure ne démarre pas, voir les journaux **DiagnosticsPluginLauncher.log** et **DiagnosticsPlugin.log** dont nous avons fourni le chemin d’accès précédemment.

Si ces journaux d’activité indiquent `Monitoring Agent not reporting success after launch`, cela signifie que le lancement de MonAgentHost.exe a échoué. Consultez ces journaux d’activité à l’emplacement indiqué pour `MonAgentHost log file` dans la section précédente.

La dernière ligne des fichiers journaux contient le code de sortie.  

```
DiagnosticsPluginLauncher.exe Information: 0 : [4/16/2016 6:24:15 AM] DiagnosticPlugin exited with code 0
```
Si le code de sortie est **négatif**, reportez-vous à la [table des codes de sortie](#azure-diagnostics-plugin-exit-codes) dans la [section Références](#references).

## <a name="diagnostics-data-is-not-logged-to-azure-storage"></a>Les données de diagnostic ne sont pas consignées dans le stockage Microsoft Azure
Tentez de déterminer si aucune des données ne s’affiche ou si certaines d’entre elles s’affichent.

### <a name="diagnostics-infrastructure-logs"></a>Journaux d’activité d’infrastructure de diagnostics
Azure Diagnostics consigne toutes les erreurs dans les journaux d’activité d’infrastructure de diagnostics. Assurez-vous que vous avez activé la [capture de journaux d’activité d’infrastructure de diagnostics dans votre configuration](#how-to-check-diagnostics-extension-configuration). Vous pouvez ensuite rechercher rapidement toutes les erreurs pertinentes qui apparaissent dans la table `DiagnosticInfrastructureLogsTable`, dans votre compte de stockage configuré.

### <a name="no-data-is-appearing"></a>Aucune donnée ne s’affiche
Le plus souvent, quand aucune donnée d’événement ne s’affiche, le problème provient des informations du compte de stockage qui ne sont pas définies correctement.

Solution : corrigez la configuration du plug-in Diagnostics et réinstallez-le.

Si le compte de stockage est configuré correctement, accédez à distance à la machine, puis vérifiez que les fichiers *DiagnosticsPlugin.exe* et *MonAgentCore.exe* sont bien en cours d’exécution. Dans le cas contraire, suivez les étapes décrites dans la section [Diagnostics Azure ne démarre pas](#azure-diagnostics-is-not-starting).

Si ces processus sont en cours d’exécution, reportez-vous à la section [La capture des données intervient-elle en local ?](#is-data-getting-captured-locally), puis suivez les instructions fournies.

Si cela ne résout pas le problème, essayez les manipulations suivantes :

1. Désinstaller l’agent
2. Supprimer le répertoire C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.Diagnostics.IaaSDiagnostics
3. Réinstaller l’agent


### <a name="part-of-the-data-is-missing"></a>Une partie des données est manquante
Si vous obtenez une partie des données, mais pas la totalité, cela signifie que le pipeline de collecte ou de transfert des données est défini correctement. Suivez les instructions des sous-sections ci-après pour tenter de localiser le problème.

#### <a name="is-the-collection-configured"></a>La collecte est-elle configurée ?
La configuration des diagnostics contient des instructions pour un type bien précis de données à collecter. [Passez en revue votre configuration](#how-to-check-diagnostics-extension-configuration) pour vérifier que vous recherchez uniquement les données que vous avez configurées pour la collecte.

#### <a name="is-the-host-generating-data"></a>L’hôte génère-t-il des données ?
- **Compteurs de performances** : ouvrez PerfMon et vérifiez le compteur.

- **Journaux d’activité des traces** :  accédez à distance à la machine virtuelle, puis ajoutez un élément TextWriterTraceListener dans le fichier de configuration de l’application.  Consultez https://msdn.microsoft.com/library/sk36c28t.aspx pour configurer l’écouteur de texte.  Vérifiez que l’élément `<trace>` a la valeur `<trace autoflush="true">`.<br />
Si aucun journal de suivi n’a été généré, voir En savoir plus sur les journaux de suivi manquants.

- **Traces ETW** : accédez à distance à la machine virtuelle, puis installez PerfView.  Dans PerfView, exécutez **Fichier** > **Commande utilisateur** > **Écouter etwprovder1** > **etwprovider2**, etc. Notez que la commande **Écouter** est sensible à la casse et que les espaces ne sont pas autorisés entre les listes de fournisseurs ETW séparées par des virgules. En cas d’échec de l’exécution de la commande, vous pouvez cliquer sur le bouton **Journal** dans l’angle inférieur droit de l’outil PerfView pour voir l’exécution qui était attendue et le résultat final de celle-ci.  Partons du principe que l’entrée est correcte. Une nouvelle fenêtre s’ouvre alors. En quelques secondes, vous commencer à voir les traces ETW.

- **Journaux d’événements** : accédez à distance à la machine virtuelle. Ouvrez `Event Viewer` et assurez-vous que les événements existent.

#### <a name="is-data-getting-captured-locally"></a>La capture des données intervient-elle en local ?
À présent, vérifiez que les données sont bien capturées en local.
Les données sont stockées localement dans les fichiers `*.tsf` dans le magasin local des données de diagnostic. Différents types de journaux d’activité sont collectés dans différents fichiers `.tsf`. Les noms sont semblables à ceux des tables dans le stockage Microsoft Azure.

Par exemple, les valeurs `Performance Counters` sont collectées dans `PerformanceCountersTable.tsf`. Les journaux des événements sont collectés dans `WindowsEventLogsTable.tsf`. Suivez les instructions indiquées dans la section [Extraction locale des journaux](#local-log-extraction) pour ouvrir les fichiers de la collecte locale, puis assurez-vous qu’ils sont bien collectés sur le disque.

Si vous ne voyez pas les journaux d’activité collectés en local et si vous avez déjà vérifié que l’hôte génère des données, vous rencontrez probablement un problème de configuration. Passez au crible vos paramètres de configuration.

Analysez également la configuration qui a été générée pour MonitoringAgent MaConfig.xml. Vérifiez qu’il existe bien une section décrivant la source des journaux. Vérifiez ensuite qu’elle n’a pas disparu entre le moment de la configuration des diagnostics et celui de la configuration de l’agent de surveillance.

#### <a name="is-data-getting-transferred"></a>Les données sont-elles transférées ?
Si vous avez vérifié que les données sont bien capturées en local, mais qu’elles n’apparaissent toujours pas dans votre compte de stockage, envisagez les étapes suivantes :

- Vérifiez que vous avez fourni un compte de stockage correct et que vous n’avez pas renouvelé les clés correspondantes. Concernant Azure Cloud Services, il arrive parfois que les utilisateurs ne mettent pas à jour le paramètre `useDevelopmentStorage=true`.

- Vérifiez que le compte de stockage fourni est bien correct. Assurez-vous qu’aucune restriction réseau n’empêche les composants d’atteindre les points de terminaison de stockage public. Pour ce faire, vous pouvez accéder à distance à la machine, puis essayer d’écrire quelque chose par vous-même dans le même compte de stockage.

- Enfin, vous pouvez essayer d’étudier les échecs signalés par l’agent de surveillance. Les journaux de l’agent de surveillance se trouvent dans le fichier `maeventtable.tsf`, dans le magasin local des données de diagnostic. Pour ouvrir ce fichier, suivez les instructions figurant dans la section [Extraction locale des journaux](#local-log-extraction). Essayez ensuite de déterminer si des `errors` signalent un échec de lecture au niveau des fichiers locaux sachant que ces fichiers écrivent des données dans le stockage.

### <a name="capturing-and-archiving-logs"></a>Capture et archivage des journaux d’activité
Si vous envisagez de contacter le support technique, sachez que vous devrez fournir en premier les journaux d’activité générés par votre machine. Vous pouvez gagner du temps en effectuant vous-même cette procédure. Exécutez l’utilitaire `CollectGuestLogs.exe` (chemin d’accès de l’utilitaire de collecte des journaux). Il génère un fichier .zip qui regroupe dans un même dossier tous les journaux d’activité Azure pertinents.

## <a name="diagnostics-data-tables-not-found"></a>Les tables de données de diagnostic sont introuvables
Les tables de stockage Azure qui contiennent les événements ETW utilisent le code suivant dans leur nom :

```csharp
        if (String.IsNullOrEmpty(eventDestination)) {
            if (e == "DefaultEvents")
                tableName = "WADDefault" + MD5(provider);
            else
                tableName = "WADEvent" + MD5(provider) + eventId;
        }
        else
            tableName = "WAD" + eventDestination;
```

Voici un exemple :

```XML
        <EtwEventSourceProviderConfiguration provider="prov1">
          <Event id="1" />
          <Event id="2" eventDestination="dest1" />
          <DefaultEvents />
        </EtwEventSourceProviderConfiguration>
        <EtwEventSourceProviderConfiguration provider="prov2">
          <DefaultEvents eventDestination="dest2" />
        </EtwEventSourceProviderConfiguration>
```
```JSON
"EtwEventSourceProviderConfiguration": [
    {
        "provider": "prov1",
        "Event": [
            {
                "id": 1
            },
            {
                "id": 2,
                "eventDestination": "dest1"
            }
        ],
        "DefaultEvents": {
            "eventDestination": "DefaultEventDestination",
            "sinks": ""
        }
    },
    {
        "provider": "prov2",
        "DefaultEvents": {
            "eventDestination": "dest2"
        }
    }
]
```
Ce code génère les quatre tables :

| Événement | Nom de la table |
| --- | --- |
| provider="prov1" &lt;Event id="1" /&gt; |WADEvent+MD5("prov1")+"1" |
| provider="prov1" &lt;Event id="2" eventDestination="dest1" /&gt; |WADdest1 |
| provider="prov1" &lt;DefaultEvents /&gt; |WADDefault+MD5("prov1") |
| provider="prov2" &lt;DefaultEvents eventDestination="dest2" /&gt; |WADdest2 |

## <a name="references"></a>References

### <a name="how-to-check-diagnostics-extension-configuration"></a>Comment vérifier la configuration de l’extension Diagnostics
Pour vérifier la configuration de l’extension, le plus simple est d’accéder à [Azure Resource Explorer](https://resources.azure.com), puis à la machine virtuelle ou au service cloud sur lesquels se trouve l’extension Diagnostics Azure (IaaSDiagnostics/PaaDiagnostics).

Vous pouvez également activer le Bureau à distance sur la machine, puis regarder le fichier de configuration Diagnostics Azure qui est décrit dans la section des chemins d’accès des artefacts de journaux.

Dans les deux cas, recherchez **Microsoft.Azure.Diagnostics**, puis le champ **xmlCfg** ou **WadCfg**.

Dans le cas de la machine virtuelle, si le champ **WadCfg** est présent, cela signifie que la configuration est au format JSON. Si le champ **xmlCfg** est présent, cela signifie que la configuration est au format XML et codée en Base64. Vous devez [la décoder](https://www.bing.com/search?q=base64+decoder) pour afficher le code XML chargé par l’extension Diagnostics.

Pour le rôle de service cloud, si vous sélectionnez la configuration à partir du disque, les données sont encodées en Base64. Vous devrez donc [les décoder](https://www.bing.com/search?q=base64+decoder) pour afficher le code XML qui a été chargé par Diagnostics.

### <a name="azure-diagnostics-plugin-exit-codes"></a>Codes de sortie du plug-in Diagnostics Azure
Le plug-in renvoie les codes de sortie suivants :

| Code de sortie | Description |
| --- | --- |
| 0 |Réussite. |
| -1 |Erreur générique. |
| -2 |Impossible de charger le fichier rcf.<p>Cette erreur interne ne doit se produire que si le lanceur du plug-in d’agent invité est appelé manuellement et de manière incorrecte sur la machine virtuelle. |
| -3 |Impossible de charger le fichier de configuration Diagnostics.<p><p>Solution : Cette erreur se produit lorsqu’un fichier de configuration ne passe pas l’étape de la validation du schéma. La solution consiste à fournir un fichier de configuration qui est conforme au schéma. |
| -4 |Une autre instance de l'agent de surveillance Diagnostics utilise déjà le répertoire de ressources local.<p><p>Solution : spécifiez une valeur différente pour **LocalResourceDirectory**. |
| -6 |Le lanceur du plug-in d'agent invité a tenté de démarrer Diagnostics avec une ligne de commande non valide.<p><p>Cette erreur interne ne doit se produire que si le lanceur du plug-in d’agent invité est appelé manuellement et de manière incorrecte sur la machine virtuelle. |
| -10 |Le plug-in Diagnostics s'est terminé avec une exception non prise en charge. |
| -11 |L'agent invité n'a pas pu créer le processus responsable du lancement et de la surveillance de l'agent de surveillance.<p><p>Solution : vérifiez que les ressources système disponibles sont suffisantes pour lancer de nouveaux processus.<p> |
| -101 |Arguments invalides lors de l'appel du plug-in Diagnostics.<p><p>Cette erreur interne ne doit se produire que si le lanceur du plug-in d’agent invité est appelé manuellement et de manière incorrecte sur la machine virtuelle. |
| -102 |Le processus du plug-in ne peut pas démarrer tout seul.<p><p>Solution : vérifiez que les ressources système disponibles sont suffisantes pour lancer de nouveaux processus. |
| -103 |Le processus du plug-in ne peut pas démarrer tout seul. Plus précisément, il n’est pas capable de créer l’objet enregistreur d’événements.<p><p>Solution : vérifiez que les ressources système disponibles sont suffisantes pour lancer de nouveaux processus. |
| -104 |Impossible de charger le fichier rcf fourni par l'agent invité.<p><p>Cette erreur interne ne doit se produire que si le lanceur du plug-in d’agent invité est appelé manuellement et de manière incorrecte sur la machine virtuelle. |
| -105 |Le plug-in Diagnostics ne peut pas ouvrir le fichier de configuration Diagnostics.<p><p>Cette erreur interne ne doit se produire que si le plug-in Diagnostics est appelé manuellement et de manière incorrecte sur la machine virtuelle. |
| -106 |Impossible de lire le fichier de configuration Diagnostics.<p><p>Cette erreur se produit lorsqu’un fichier de configuration ne passe pas l’étape de la validation du schéma. <br><br>Solution : fournissez un fichier de configuration conforme au schéma. Pour plus d’informations, voir [Comment vérifier la configuration de l’extension Diagnostics](#how-to-check-diagnostics-extension-configuration). |
| -107 |La transmission du répertoire de ressources à l'agent de surveillance n'est pas valide.<p><p>Cette erreur interne ne doit se produire que si l’agent de surveillance est appelé manuellement et de manière incorrecte sur la machine virtuelle.</p> |
| -108 |Impossible de convertir le fichier de configuration Diagnostics dans le fichier de configuration de l'agent de surveillance.<p><p>Cette erreur interne ne doit se produire que si le plug-in Diagnostics est appelé manuellement avec un fichier de configuration non valide. |
| -110 |Erreur de configuration générale de Diagnostics.<p><p>Cette erreur interne ne doit se produire que si le plug-in Diagnostics est appelé manuellement avec un fichier de configuration non valide. |
| -111 |Impossible de démarrer l’agent de surveillance.<p><p>Solution : vérifiez que les ressources système disponibles sont suffisantes. |
| -112 |Erreur générale |

### <a name="local-log-extraction"></a>Extraction locale des journaux
L’agent de supervision collecte les journaux d’activité et artefacts en tant que fichiers `.tsf`. Le fichier `.tsf` n’est pas lisible, mais vous pouvez le convertir en fichier `.csv`, comme suit :

```
<Azure diagnostics extension package>\Monitor\x64\table2csv.exe <relevantLogFile>.tsf
```
Un nouveau fichier appelé `<relevantLogFile>.csv` est créé et il a le même chemin d’accès que le fichier `.tsf` correspondant.

>[!NOTE]
> Vous devez exécuter cet utilitaire uniquement dans le fichier .tsf principal (par exemple, PerformanceCountersTable.tsf). Les fichiers d’accompagnement (par exemple, PerformanceCountersTables_\*\*001.tsf, PerformanceCountersTables_\*\*002. tsf, etc.) sont traités automatiquement.

### <a name="more-about-missing-trace-logs"></a>En savoir plus sur les journaux d’activité de suivi manquants

>[!NOTE]
> Les informations suivantes concernent principalement Azure Cloud Services, à moins que vous n’ayez configuré l’élément DiagnosticsMonitorTraceListener dans une application qui s’exécute sur votre machine virtuelle IaaS.

- Vérifiez que l’élément **DiagnosticMonitorTraceListener** est configuré dans le fichier web.config ou app.config.  Il est configuré par défaut dans les projets de service cloud. Toutefois, certains clients y ajoutent des commentaires, ce qui empêche la collecte des instructions de suivi par les diagnostics.

- Si aucune écriture des journaux d’activité n’intervient à partir de la méthode **OnStart** ou **Run**, assurez-vous que l’élément **DiagnosticMonitorTraceListener** figure bien dans le fichier app.config.  Par défaut, il se trouve dans le fichier web.config, mais cela s’applique uniquement au code s’exécutant dans w3wp.exe. Par conséquent, vous en avez besoin dans le fichier app.config pour capturer le suivi dans WaIISHost.exe.

- Veillez à utiliser **Diagnostics.Trace.TraceXXX** plutôt que **Diagnostics.Debug.WriteXXX**. Les instructions de débogage seront supprimées de la version de mise en production.

- Vérifiez que le code compilé contient effectivement les **lignes Diagnostics.Trace** (utilisez Reflector, ildasm ou ILSpy pour le vérifier). Les commandes **Diagnostics.Trace** sont supprimées du fichier binaire compilé, sauf si vous utilisez le symbole de compilation conditionnelle TRACE. Il s’agit d’un problème courant qui se produit lorsque vous utilisez msbuild pour générer un projet.   

## <a name="known-issues-and-mitigations"></a>Problèmes connus et atténuations des risques
Voici la liste des problèmes connus avec les atténuations des risques :

**1. Dépendance vis-à-vis de .NET 4.5**

L’extension Diagnostics Azure pour Windows a une dépendance de runtime avec le framework .NET 4.5 ou ultérieur. Au moment de la rédaction de cet article, la version 4.5 ou ultérieure de .NET est installée sur toutes les machines configurées pour Azure Cloud Services, ainsi que sur toutes les images officielles basées sur des machines virtuelles Azure.

Toutefois, rien n’empêche une situation dans laquelle vous essayez d’exécuter l’extension Diagnostics Azure pour Windows sur un ordinateur qui n’est pas équipé de la version 4.5 ou ultérieure de .NET. C’est le cas lorsque vous créez votre machine à partir d’une ancienne image ou d’un ancien instantané ou lorsque vous apportez votre propre disque personnalisé.

Cela se manifeste généralement par le code de sortie **255** lorsque vous exécutez **DiagnosticsPluginLauncher.exe**. Un échec se produit en raison de l’exception non prise en charge suivante :
```
System.IO.FileLoadException: Could not load file or assembly 'System.Threading.Tasks, Version=1.5.11.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a' or one of its dependencies
```

**Atténuation :** installez .NET 4.5 ou une version ultérieure sur votre machine.

**2. Les données des compteurs de performances sont disponibles dans le stockage, mais ne s’affichent pas dans le portail**

Par défaut, le portail des machines virtuelles affiche certains compteurs de performances. Si vous ne voyez pas ces compteurs alors que vous savez que les données correspondantes sont générées, tout simplement parce qu’elles sont disponibles dans le stockage, vous devez vérifier les points suivants :

- Les noms de compteur des données disponibles dans le stockage sont en anglais. Si les noms de compteur ne sont pas en anglais, ils n’apparaîtront pas dans le graphique des métriques du portail. **Atténuation** : modifiez la langue de la machine pour la définir sur l’anglais pour les comptes système. Pour ce faire, sélectionnez **Panneau de configuration** > **Région** > **Administrative** > **Copier les paramètres**. Ensuite, désactivez l’option **Écran d’accueil et comptes système** afin que la langue personnalisée ne soit pas appliquée au compte système.

- Si vous utilisez des caractères génériques (\*) dans les noms de compteur de performances, le portail ne peut pas établir de corrélation entre le compteur configuré et le compteur collecté lors de l’envoi des compteurs de performances vers le récepteur du Stockage Azure. **Atténuation** : pour pouvoir utiliser des caractères génériques et permettre au portail de développer (\*), dirigez vos compteurs de performances vers le récepteur Azure Monitor.
