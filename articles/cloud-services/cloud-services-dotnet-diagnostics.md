---
title: Utilisation des diagnostics (.NET) Azure avec Cloud Services | Microsoft Docs
description: Utilisation des diagnostics Azure pour rassembler des données à partir d’Azure Cloud Services pour le débogage, la mesure des performances, la surveillance, l’analyse du trafic, etc.
services: cloud-services
documentationcenter: .net
author: tgore03
manager: carmonm
ms.service: cloud-services
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.date: 05/22/2017
ms.author: tagore
ms.openlocfilehash: 6a015a8d56cf3991d04b212db73d5b752c13a793
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92077539"
---
# <a name="enabling-azure-diagnostics-in-azure-cloud-services"></a>Activation des diagnostics Azure dans Azure Cloud Services
Consultez la page [Présentation des diagnostics Azure](../azure-monitor/platform/diagnostics-extension-overview.md) pour obtenir des informations sur les diagnostics Azure.

## <a name="how-to-enable-diagnostics-in-a-worker-role"></a>Activation de Diagnostics dans un rôle de travail
Cette procédure pas à pas décrit comment mettre en oeuvre un rôle de travail Azure qui émet des données télémétriques à l'aide de la classe EventSource .NET. Diagnostics Azure est utilisé pour collecter des données télémétriques et les stocker dans un compte de stockage Azure. Lors de la création d'un rôle de travail, Visual Studio active automatiquement Diagnostics 1.0 dans le cadre de la solution dans les Kits de développement logiciel (SDK) pour .NET 2.4 et versions antérieures. Les instructions suivantes décrivent le processus de création d'un rôle de travail, de désactivation de Diagnostics 1.0 de la solution, et de déploiement de Diagnostics 1.2 ou 1.3 sur votre rôle de travail.

### <a name="prerequisites"></a>Prérequis
Cet article part du principe que vous disposez d’un abonnement Azure et que vous utilisez Visual Studio avec le Kit de développement logiciel (SDK) Azure. Si vous n'avez pas d'abonnement Azure, vous pouvez vous inscrire pour bénéficier d'une [évaluation gratuite][Free Trial]. Assurez-vous d'avoir [installé et configuré Azure PowerShell version 0.8.7 ou ultérieure][Install and configure Azure PowerShell version 0.8.7 or later].

### <a name="step-1-create-a-worker-role"></a>Étape 1 : création du rôle de travail
1. Lancez **Visual Studio**.
2. Créez un projet **Azure Cloud Services** à partir du modèle **Cloud** qui cible .NET Framework 4.5.  Nommez le projet « WadExample » et cliquez sur OK.
3. Sélectionnez **Rôle de travail** et cliquez sur Ok. Le projet est créé.
4. Dans **Explorateur de solutions**, double-cliquez sur le fichier de propriétés **WorkerRole1**.
5. Dans l’onglet **Configuration**, décochez **Activer Diagnostics** pour désactiver Diagnostics 1.0 (Kit de développement Azure 2.4 et versions antérieures).
6. Générez votre solution pour vérifier que vous n'avez pas d'erreurs.

### <a name="step-2-instrument-your-code"></a>Étape 2 : instrumentalisation de votre code
Remplacez le contenu de WorkerRole.cs par le code suivant : La classe SampleEventSourceWriter, héritée de la [classe EventSource][EventSource Class], implémente quatre méthodes de journalisation : **SendEnums**, **MessageMethod**, **SetOther** et **HighFreq**. Le premier paramètre de la méthode **WriteEvent** définit l’ID de l’événement respectif. La méthode Run implémente une boucle infinie qui appelle chacune des méthodes de journalisation implémentées dans la classe **SampleEventSourceWriter** toutes les 10 secondes.

```csharp
using Microsoft.WindowsAzure.ServiceRuntime;
using System;
using System.Diagnostics;
using System.Diagnostics.Tracing;
using System.Net;
using System.Threading;

namespace WorkerRole1
{
    sealed class SampleEventSourceWriter : EventSource
    {
        public static SampleEventSourceWriter Log = new SampleEventSourceWriter();
        public void SendEnums(MyColor color, MyFlags flags) { if (IsEnabled())  WriteEvent(1, (int)color, (int)flags); }// Cast enums to int for efficient logging.
        public void MessageMethod(string Message) { if (IsEnabled())  WriteEvent(2, Message); }
        public void SetOther(bool flag, int myInt) { if (IsEnabled())  WriteEvent(3, flag, myInt); }
        public void HighFreq(int value) { if (IsEnabled()) WriteEvent(4, value); }

    }

    enum MyColor
    {
        Red,
        Blue,
        Green
    }

    [Flags]
    enum MyFlags
    {
        Flag1 = 1,
        Flag2 = 2,
        Flag3 = 4
    }

    public class WorkerRole : RoleEntryPoint
    {
        public override void Run()
        {
            // This is a sample worker implementation. Replace with your logic.
            Trace.TraceInformation("WorkerRole1 entry point called");

            int value = 0;

            while (true)
            {
                Thread.Sleep(10000);
                Trace.TraceInformation("Working");

                // Emit several events every time we go through the loop
                for (int i = 0; i < 6; i++)
                {
                    SampleEventSourceWriter.Log.SendEnums(MyColor.Blue, MyFlags.Flag2 | MyFlags.Flag3);
                }

                for (int i = 0; i < 3; i++)
                {
                    SampleEventSourceWriter.Log.MessageMethod("This is a message.");
                    SampleEventSourceWriter.Log.SetOther(true, 123456789);
                }

                if (value == int.MaxValue) value = 0;
                SampleEventSourceWriter.Log.HighFreq(value++);
            }
        }

        public override bool OnStart()
        {
            // Set the maximum number of concurrent connections
            ServicePointManager.DefaultConnectionLimit = 12;

            // For information on handling configuration changes
            // see the MSDN topic at https://go.microsoft.com/fwlink/?LinkId=166357.

            return base.OnStart();
        }
    }
}
```


### <a name="step-3-deploy-your-worker-role"></a>Étape 3 : déploiement de votre rôle de travail

[!INCLUDE [cloud-services-wad-warning](../../includes/cloud-services-wad-warning.md)]

1. Déployez votre rôle de travail vers Azure à partir de Visual Studio en sélectionnant le projet **WadExample** dans l’Explorateur de solutions, puis **Publier** à partir du menu **Générer**.
2. Choisissez votre abonnement.
3. Dans la boîte de dialogue **Paramètres de publication Microsoft Azure**, sélectionnez **Créer**.
4. Dans la boîte de dialogue **Créer un service cloud et un compte de stockage**, saisissez un **Nom** (par exemple « WadExample »), puis sélectionnez une région ou un groupe d’affinités.
5. Définissez l’**Environnement** sur **Intermédiaire**.
6. Modifiez d’autres **Paramètres** le cas échéant, puis cliquez sur **Publier**.
7. Une fois le déploiement terminé, vérifiez dans le portail Azure Classic que votre service cloud est en cours d’ **Exécution** .

### <a name="step-4-create-your-diagnostics-configuration-file-and-install-the-extension"></a>Étape 4 : création de votre fichier de configuration Diagnostics et installation de l’extension
1. Téléchargez la définition de schéma de fichier de configuration publique en exécutant la commande PowerShell suivante :

    ```powershell
    (Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PublicConfigurationSchema | Out-File -Encoding utf8 -FilePath 'WadConfig.xsd'
    ```
2. Ajoutez un fichier XML à votre projet **WorkerRole1** en cliquant avec le bouton droit sur le projet **WorkerRole1**, puis sélectionnez **Ajouter** -> **Nouvel élément**. -> **Éléments Visual C#**  -> **Données** -> **Fichier XML**. Nommez le fichier « WadExample.xml ».

   ![CloudServices_diag_add_xml](./media/cloud-services-dotnet-diagnostics/AddXmlFile.png)
3. Associez le fichier WadConfig.xsd avec le fichier de configuration. Assurez-vous que la fenêtre de l'éditeur WadExample.xml est la fenêtre active. Appuyez sur **F4** pour ouvrir la fenêtre **Propriétés**. Cliquez sur la propriété **Schémas** dans la fenêtre **Propriétés**. Cliquez sur **…** in the **Schémas** . Cliquez sur **Ajouter…** et naviguez jusqu’à l’emplacement où vous avez enregistré le fichier XSD, puis sélectionnez le fichier WadConfig.xsd. Cliquez sur **OK**.

4. Remplacez le contenu du fichier de configuration WadExample.xml par le XML suivant, puis enregistrez le fichier. Ce fichier de configuration définit deux compteurs de performances à collecter : un pour l'utilisation du processeur et l'autre pour l'utilisation de la mémoire. Ensuite, la configuration définit les quatre événements correspondant aux méthodes de la classe SampleEventSourceWriter.

```xml
<?xml version="1.0" encoding="utf-8"?>
<PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <WadCfg>
    <DiagnosticMonitorConfiguration overallQuotaInMB="25000">
      <PerformanceCounters scheduledTransferPeriod="PT1M">
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT1M" unit="percent" />
        <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT1M" unit="bytes"/>
      </PerformanceCounters>
      <EtwProviders>
        <EtwEventSourceProviderConfiguration provider="SampleEventSourceWriter" scheduledTransferPeriod="PT5M">
          <Event id="1" eventDestination="EnumsTable"/>
          <Event id="2" eventDestination="MessageTable"/>
          <Event id="3" eventDestination="SetOtherTable"/>
          <Event id="4" eventDestination="HighFreqTable"/>
          <DefaultEvents eventDestination="DefaultTable" />
        </EtwEventSourceProviderConfiguration>
      </EtwProviders>
    </DiagnosticMonitorConfiguration>
  </WadCfg>
</PublicConfig>
```

### <a name="step-5-install-diagnostics-on-your-worker-role"></a>Étape 5 : installation de Diagnostics sur votre rôle de travail
Les applets de commande PowerShell pour la gestion de Diagnostics sur un rôle Web ou de travail sont : Set-AzureServiceDiagnosticsExtension, Get-AzureServiceDiagnosticsExtension et Remove-AzureServiceDiagnosticsExtension.

1. Ouvrez Azure PowerShell.
2. Exécutez le script pour installer Diagnostics sur votre rôle de travail (remplacez *StorageAccountKey* par la clé du compte de stockage de votre compte de stockage wadexample) et *config_path* par le chemin du fichier *WadExample.xml*) :

```powershell
$storage_name = "wadexample"
$key = "<StorageAccountKey>"
$config_path="c:\users\<user>\documents\visual studio 2013\Projects\WadExample\WorkerRole1\WadExample.xml"
$service_name="wadexample"
$storageContext = New-AzureStorageContext -StorageAccountName $storage_name -StorageAccountKey $key
Set-AzureServiceDiagnosticsExtension -StorageContext $storageContext -DiagnosticsConfigurationPath $config_path -ServiceName $service_name -Slot Staging -Role WorkerRole1
```

### <a name="step-6-look-at-your-telemetry-data"></a>Étape 6 : examen de vos données télémétriques
Dans l' **Explorateur de serveurs** de Visual Studio, naviguez jusqu'au compte de stockage wadexample. Une fois que le service cloud a été exécuté pendant environ cinq (5) minutes, vous devriez voir les tables **WADEnumsTable**, **WADHighFreqTable**, **WADMessageTable**, **WADPerformanceCountersTable** et **WADSetOtherTable**. Double-cliquez sur l'une des tables pour afficher les données télémétriques qui ont été collectées.

![CloudServices_diag_tables](./media/cloud-services-dotnet-diagnostics/WadExampleTables.png)

## <a name="configuration-file-schema"></a>Schéma du fichier de configuration
Le fichier de configuration des diagnostics définit les valeurs qui sont utilisées pour initialiser les paramètres de configuration de diagnostic lorsque l’agent de diagnostics démarre. Consultez la [dernière référence de schéma](../azure-monitor/platform/diagnostics-extension-versions.md) pour obtenir les valeurs valides et des exemples.

## <a name="troubleshooting"></a>Dépannage
Si vous rencontrez des problèmes, consultez la page [Résolution de problèmes des diagnostics Azure](../azure-monitor/platform/diagnostics-extension-troubleshooting.md) pour obtenir de l’aide.

## <a name="next-steps"></a>Étapes suivantes
[Consultez la liste des articles connexes sur les diagnostics relatifs aux machines virtuelles Azure ](../azure-monitor/platform/diagnostics-extension-overview.md) pour modifier les données que vous collectez, résoudre des problèmes ou pour en savoir plus sur les diagnostics en général.

[EventSource Class]: /dotnet/api/system.diagnostics.tracing.eventsource

[Debugging an Azure Application]: https://msdn.microsoft.com/library/windowsazure/ee405479.aspx   
[Collect Logging Data by Using Azure Diagnostics]: /previous-versions/azure/gg433048(v=azure.100)
[Free Trial]: https://azure.microsoft.com/pricing/free-trial/
[Install and configure Azure PowerShell version 0.8.7 or later]: /powershell/azure/