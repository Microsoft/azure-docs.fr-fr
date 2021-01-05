---
title: Envoyer des métriques Cloud Services classiques à une base de données de métriques Azure Monitor
description: Décrit le processus permettant d’envoyer au magasin de métriques Azure Monitor les métriques de performances du système d’exploitation invité concernant les services cloud classiques Azure.
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 09/09/2019
ms.author: ancav
ms.subservice: metrics
ms.openlocfilehash: f539786de589dbab3a191a5343ba315349533447
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91360988"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metric-store-classic-cloud-services"></a>Envoyer des métriques de système d’exploitation invité au magasin de métriques Azure Monitor pour les services cloud classiques 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Avec [l’extension Diagnostics](diagnostics-extension-overview.md) d’Azure Monitor, vous pouvez collecter des métriques et des journaux d’activité à partir du système d’exploitation invité qui est exécuté dans le cadre d’une machine virtuelle, d’un service cloud ou d’un cluster Service Fabric. L’extension peut envoyer des données de télémétrie à de [nombreux emplacements différents](./data-platform.md?toc=/azure/azure-monitor/toc.json).

Cet article décrit le processus permettant d’envoyer au magasin de métriques Azure Monitor les métriques de performances du système d’exploitation invité concernant les services cloud classiques Azure. À partir de la version 1.11 de l’extension Diagnostics, vous pouvez écrire des métriques directement dans le magasin de métriques Azure Monitor, où les métriques standard de la plateforme sont déjà collectées. 

En les stockant dans cet emplacement, vous avez accès aux mêmes actions que celles disponibles pour les métriques de la plateforme. Ces actions incluent notamment la génération d’alertes en temps quasi réel, la création de graphiques, le routage, l’accès à partir d’une API REST, etc.  Avant, l’extension Diagnostics écrivait les données dans le stockage Azure, et non dans le magasin de données d’Azure Monitor.  

Le processus décrit dans cet article fonctionne uniquement pour les compteurs de performances des services cloud Azure. Il ne fonctionne pas pour les autres métriques personnalisées. 

## <a name="prerequisites"></a>Prérequis

- Vous devez être [administrateur ou coadministrateur de services fédérés](../../cost-management-billing/manage/add-change-subscription-administrator.md) dans votre abonnement Azure. 

- Votre abonnement doit être inscrit auprès de [Microsoft.Insights](../../azure-resource-manager/management/resource-providers-and-types.md). 

- Vous devez avoir installé [Azure PowerShell](/powershell/azure) ou [Azure Cloud Shell](../../cloud-shell/overview.md).

- Votre service cloud doit se trouver dans une région [ prenant en charge les métriques personnalisées](metrics-custom-overview.md#supported-regions).

## <a name="provision-a-cloud-service-and-storage-account"></a>Provisionner un service cloud et un compte de stockage 

1. Créez et déployez un service cloud classique. Pour voir un exemple d’application de services cloud classiques avec son déploiement, consultez [Prise en main des services cloud Azure et d’ASP.NET](../../cloud-services/cloud-services-dotnet-get-started.md). 

2. Vous pouvez utiliser un compte de stockage existant ou en déployer un nouveau. Il est préférable que le compte de stockage se trouve dans la même région que le service cloud classique que vous avez créé. Dans le portail Azure, accédez au panneau de ressource **Comptes de stockage**, puis sélectionnez **Clés**. Notez le nom du compte de stockage et la clé du compte de stockage. Vous aurez besoin de ces informations pour les étapes ultérieures.

   ![Clés de compte de stockage](./media/collect-custom-metrics-guestos-vm-cloud-service-classic/storage-keys.png)

## <a name="create-a-service-principal"></a>Créer un principal du service 

Créez un principal du service dans votre locataire Azure Active Directory à l’aide des instructions fournies dans [Utiliser le portail pour créer une application et un principal du service Azure Active Directory pouvant accéder aux ressources](../../active-directory/develop/howto-create-service-principal-portal.md). Notez ce qui suit au cours de ce processus : 

- Vous pouvez indiquer n’importe quelle URL pour l’URL de connexion.  
- Créez un secret client pour cette application.  
- Enregistrez la clé et l’ID client pour les utiliser ultérieurement.  

Attribuez à l’application créée à l’étape précédente des autorisations *Surveillance de l’éditeur de métriques* afin qu’elle puisse accéder à la ressource à partir de laquelle vous souhaitez générer des métriques. Si vous envisagez d’utiliser l’application pour émettre des métriques personnalisées concernant de nombreuses ressources, vous pouvez accorder ces autorisations au niveau du groupe de ressources ou de l’abonnement.  

> [!NOTE]
> L’extension Diagnostics utilise le principal de service pour s’authentifier auprès d’Azure Monitor et pour générer des métriques concernant votre service cloud.

## <a name="author-diagnostics-extension-configuration"></a>Créer la configuration de l’extension Diagnostics 

Préparez le fichier config de l’extension Diagnostics. Ce fichier détermine quels journaux d’activité et compteurs de performances doivent être collectés par l’extension Diagnostics pour votre service cloud. Vous trouverez ci-dessous un exemple de fichier config Diagnostics :  

```XML
<?xml version="1.0" encoding="utf-8"?> 
<DiagnosticsConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
  <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
    <WadCfg> 
      <DiagnosticMonitorConfiguration overallQuotaInMB="4096"> 
        <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error" /> 
        <Directories scheduledTransferPeriod="PT1M"> 
          <IISLogs containerName="wad-iis-logfiles" /> 
          <FailedRequestLogs containerName="wad-failedrequestlogs" /> 
        </Directories> 
        <PerformanceCounters scheduledTransferPeriod="PT1M"> 
          <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" /> 
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Available MBytes" sampleRate="PT15S" /> 
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT15S" /> 
          <PerformanceCounterConfiguration counterSpecifier="\Memory\Page Faults/sec" sampleRate="PT15S" /> 
        </PerformanceCounters> 
        <WindowsEventLog scheduledTransferPeriod="PT1M"> 
          <DataSource name="Application!*[System[(Level=1 or Level=2 or Level=3)]]" /> 
          <DataSource name="Windows Azure!*[System[(Level=1 or Level=2 or Level=3 or Level=4)]]" /> 
        </WindowsEventLog> 
        <CrashDumps> 
          <CrashDumpConfiguration processName="WaIISHost.exe" /> 
          <CrashDumpConfiguration processName="WaWorkerHost.exe" /> 
          <CrashDumpConfiguration processName="w3wp.exe" /> 
        </CrashDumps> 
        <Logs scheduledTransferPeriod="PT1M" scheduledTransferLogLevelFilter="Error" /> 
      </DiagnosticMonitorConfiguration> 
      <SinksConfig> 
      </SinksConfig> 
    </WadCfg> 
    <StorageAccount /> 
  </PublicConfig> 
  <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
    <StorageAccount name="" endpoint="" /> 
</PrivateConfig> 
  <IsEnabled>true</IsEnabled> 
</DiagnosticsConfiguration> 
```

Dans la section « SinksConfig » de votre fichier de diagnostics, définissez un nouveau récepteur Azure Monitor : 

```XML
  <SinksConfig> 
    <Sink name="AzMonSink"> 
    <AzureMonitor> 
      <ResourceId>-Provide ClassicCloudService’s Resource ID-</ResourceId> 
      <Region>-Azure Region your Cloud Service is deployed in-</Region> 
    </AzureMonitor> 
    </Sink> 
  </SinksConfig> 
```

Dans la section de votre fichier config où vous répertoriez les compteurs de performances à collecter, ajoutez le récepteur Azure Monitor. Cette entrée garantit que tous les compteurs de performances spécifiés sont acheminés vers Azure Monitor en tant que métriques. Vous pouvez ajouter ou supprimer des compteurs de performances selon vos besoins. 

```xml
    <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="AzMonSink">
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" />
    ...
    </PerformanceCounters>
```

Enfin, dans la configuration privée, ajoutez une section *Azure Monitor Account* (Compte Azure Monitor). Entrez l’ID de client du principal de service et le secret que vous avez créés précédemment. 

```XML
<PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration"> 
  <StorageAccount name="" endpoint="" /> 
    <AzureMonitorAccount> 
      <ServicePrincipalMeta> 
        <PrincipalId>clientId from step 3</PrincipalId> 
        <Secret>client secret from step 3</Secret> 
      </ServicePrincipalMeta> 
    </AzureMonitorAccount> 
</PrivateConfig> 
```

Enregistrez ce fichier de diagnostics en local.  

## <a name="deploy-the-diagnostics-extension-to-your-cloud-service"></a>Déployer l’extension Diagnostics dans votre service cloud 

Lancez PowerShell et connectez-vous à Azure. 

```powershell
Login-AzAccount 
```

Utilisez les commandes suivantes pour stocker les informations du compte de stockage que vous avez créé précédemment. 

```powershell
$storage_account = <name of your storage account from step 3> 
$storage_keys = <storage account key from step 3> 
```

De la même façon, définissez le chemin du fichier de diagnostics sur une variable en utilisant la commande suivante :

```powershell
$diagconfig = “<path of the Diagnostics configuration file with the Azure Monitor sink configured>” 
```

Déployez l’extension Diagnostics dans votre service cloud à l’aide du fichier de diagnostics, avec le récepteur Azure Monitor configuré à l’aide de la commande suivante :  

```powershell
Set-AzureServiceDiagnosticsExtension -ServiceName <classicCloudServiceName> -StorageAccountName $storage_account -StorageAccountKey $storage_keys -DiagnosticsConfigurationPath $diagconfig 
```

> [!NOTE] 
> Vous devez toujours fournir un compte de stockage lors de l’installation de l’extension Diagnostics. Les journaux d’activité et les compteurs de performances spécifiés dans le fichier config de diagnostics sont écrits dans le compte de stockage spécifié.  

## <a name="plot-metrics-in-the-azure-portal"></a>Tracer des métriques dans le Portail Azure 

1. Accédez au portail Azure. 

   ![Capture d’écran montrant le portail Azure avec les options Moniteur et Métriques sélectionnées.](./media/collect-custom-metrics-guestos-vm-cloud-service-classic/navigate-metrics.png)

2. Dans le menu de gauche, sélectionnez **Surveiller**.

3. Dans le panneau **Surveiller**, sélectionnez l’onglet **Métriques (préversion)** .

4. Dans la liste déroulante des ressources, sélectionnez votre service cloud classique.

5. Dans le menu déroulant des espaces de noms, sélectionnez **azure.vm.windows.guest**. 

6. Dans la liste déroulante des métriques, sélectionnez **Mémoire\Octets validés en cours d’utilisation**. 

Vous pouvez utiliser les fonctionnalités de filtrage et de fractionnement des dimensions pour afficher la mémoire totale utilisée par un rôle ou une instance de rôle. 

 ![Capture d’écran montrant des données métriques.](./media/collect-custom-metrics-guestos-vm-cloud-service-classic/metrics-graph.png)

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [métriques personnalisées](metrics-custom-overview.md).
