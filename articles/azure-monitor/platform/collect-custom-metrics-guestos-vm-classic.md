---
title: Envoyer des mesures de machine virtuelle Windows classiques à une base de données de mesures Azure Monitor
description: Envoyer les métriques du système d’exploitation invité d’une machine virtuelle Windows (classique) au magasin de données Azure Monitor
author: anirudhcavale
services: azure-monitor
ms.topic: conceptual
ms.date: 09/09/2019
ms.author: ancav
ms.subservice: ''
ms.openlocfilehash: 87277c0c61f6d63e453386724dd472d2663e3148
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87045206"
---
# <a name="send-guest-os-metrics-to-the-azure-monitor-metrics-database-for-a-windows-virtual-machine-classic"></a>Envoyer les mesures du système d’exploitation invité à une base de données de mesures Azure Monitor d’une machine virtuelle Windows (classique)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[L’extension Diagnostics](./diagnostics-extension-overview.md) (également appelée « WAD » ou « Diagnostics ») d’Azure Monitor vous permet de collecter des métriques et des journaux d’activité à partir du système d’exploitation invité qui est exécuté dans le cadre d’une machine virtuelle, d’un service cloud ou d’un cluster Service Fabric. L’extension peut envoyer des données de télémétrie à de [nombreux emplacements différents](./data-platform.md?toc=/azure/azure-monitor/toc.json).

Cet article décrit le processus permettant d’envoyer les mesures de performances du système d’exploitation invité d’une machine virtuelle Windows (classique) à la base de données des mesures Azure Monitor. À partir de la version 1.11 de l’extension Diagnostics, vous pouvez écrire des métriques directement dans le magasin de métriques Azure Monitor, où les métriques standard de la plateforme sont déjà collectées. 

En les stockant dans cet emplacement, vous avez accès aux mêmes actions que celles disponibles pour les métriques de la plateforme. Ces actions incluent notamment la génération d’alertes en temps quasi réel, la création de graphiques, le routage, l’accès à partir d’une API REST, etc. Avant, l’extension Diagnostics écrivait les données dans le stockage Azure, et non dans le magasin de données d’Azure Monitor. 

La procédure décrite dans cet article fonctionne uniquement avec les machines virtuelles classiques qui exécutent un système d’exploitation Windows.

## <a name="prerequisites"></a>Prérequis

- Vous devez être [administrateur ou coadministrateur de services fédérés](../../cost-management-billing/manage/add-change-subscription-administrator.md) dans votre abonnement Azure. 

- Votre abonnement doit être inscrit auprès de [Microsoft.Insights](../../azure-resource-manager/management/resource-providers-and-types.md). 

- Vous devez avoir installé [Azure PowerShell](/powershell/azure) ou [Azure Cloud Shell](../../cloud-shell/overview.md).

- Votre ressource de machine virtuelle doit se trouver dans une région [ prenant en charge les métriques personnalisées](metrics-custom-overview.md#supported-regions).

## <a name="create-a-classic-virtual-machine-and-storage-account"></a>Créer une machine virtuelle classique et un compte de stockage

1. Créez une machine virtuelle classique à l’aide du portail Azure.
   ![Créer une machine virtuelle classique](./media/collect-custom-metrics-guestos-vm-classic/create-classic-vm.png)

1. Lorsque vous créez cette machine virtuelle, choisissez l’option permettant de créer un compte de stockage classique. Nous utiliserons ce compte de stockage dans les étapes ultérieures.

1. Dans le portail Azure, accédez au panneau de ressource **Comptes de stockage**. Sélectionnez **Clés**, puis notez le nom du compte de stockage et la clé du compte de stockage. Vous aurez besoin de ces informations pour les étapes ultérieures.
   ![Clés d’accès de stockage](./media/collect-custom-metrics-guestos-vm-classic/storage-access-keys.png)

## <a name="create-a-service-principal"></a>Créer un principal du service

Créez un principal du service dans votre locataire Azure Active Directory en suivant les instructions indiquées dans [Créer un principal du service](../../active-directory/develop/howto-create-service-principal-portal.md). Notez ce qui suit au cours de ce processus : 
- Créez un secret client pour cette application.
- Enregistrez la clé et l’ID client pour les utiliser ultérieurement.

Donnez à l’application créée dans « Surveillance de l’éditeur de métriques » les autorisations pour la ressource dont vous souhaitez obtenir des métriques. Vous pouvez utiliser un groupe de ressources ou tout un abonnement.  

> [!NOTE]
> L’extension Diagnostics utilise le principal de service pour s’authentifier auprès d’Azure Monitor et obtenir des métriques pour votre machine virtuelle classique.

## <a name="author-diagnostics-extension-configuration"></a>Créer la configuration de l’extension Diagnostics

1. Préparez le fichier config de l’extension Diagnostics. Ce fichier détermine quels journaux d’activité et compteurs de performances doivent être collectés par l’extension Diagnostics pour votre machine virtuelle classique. Vous trouverez ci-dessous un exemple :

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <DiagnosticsConfiguration xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
        <WadCfg>
        <DiagnosticMonitorConfiguration overallQuotaInMB="4096" sinks="applicationInsights.errors">
            <DiagnosticInfrastructureLogs scheduledTransferLogLevelFilter="Error" />
            <Directories scheduledTransferPeriod="PT1M">
                <IISLogs containerName="wad-iis-logfiles" />
                <FailedRequestLogs containerName="wad-failedrequestlogs" />
            </Directories>
            <PerformanceCounters scheduledTransferPeriod="PT1M">
                <PerformanceCounterConfiguration counterSpecifier="\Processor(*)\% Processor Time" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\Available Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\Memory\% Committed Bytes" sampleRate="PT15S" />
                <PerformanceCounterConfiguration counterSpecifier="\LogicalDisk(*)\Disk Read Bytes/sec" sampleRate="PT15S" />
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
            <Metrics resourceId="/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/MyResourceGroup/providers/Microsoft.ClassicCompute/virtualMachines/MyClassicVM">
                <MetricAggregation scheduledTransferPeriod="PT1M" />
                <MetricAggregation scheduledTransferPeriod="PT1H" />
            </Metrics>
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
1. Dans la section « SinksConfig » de votre fichier de diagnostics, définissez un nouveau récepteur Azure Monitor de la façon suivante :

    ```xml
    <SinksConfig>
        <Sink name="AzMonSink">
            <AzureMonitor>
                <ResourceId>Provide the resource ID of your classic VM </ResourceId>
                <Region>The region your VM is deployed in</Region>
            </AzureMonitor>
        </Sink>
    </SinksConfig>
    ```

1. Dans la section de votre fichier de configuration où figure la liste des compteurs de performances à collecter, dirigez les compteurs de performances vers le récepteur Azure Monitor « AzMonSink ».

    ```xml
    <PerformanceCounters scheduledTransferPeriod="PT1M" sinks="AzMonSink">
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT15S" />
    ...
    </PerformanceCounters>
    ```

1. Dans la configuration privée, définissez le compte Azure Monitor. Ensuite, ajoutez les informations du principal de service à utiliser pour obtenir des métriques.

    ```xml
    <PrivateConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
    <StorageAccount name="" endpoint="" />
        <AzureMonitorAccount>
            <ServicePrincipalMeta>
                <PrincipalId>clientId for your service principal</PrincipalId>
                <Secret>client secret of your service principal</Secret>
            </ServicePrincipalMeta>
        </AzureMonitorAccount>
    </PrivateConfig>
    ```

1. Enregistrez ce fichier localement.

## <a name="deploy-the-diagnostics-extension-to-your-cloud-service"></a>Déployer l’extension Diagnostics dans votre service cloud

1. Lancez PowerShell et connectez-vous.

    ```powershell
    Login-AzAccount
    ```

1. Commencez par définir le contexte pour votre machine virtuelle classique.

    ```powershell
    $VM = Get-AzureVM -ServiceName <VM’s Service_Name> -Name <VM Name>
    ```

1. Définissez le contexte du compte de stockage classique créé avec la machine virtuelle.

    ```powershell
    $StorageContext = New-AzStorageContext -StorageAccountName <name of your storage account from earlier steps> -storageaccountkey "<storage account key from earlier steps>"
    ```

1.  Définissez le chemin du fichier de diagnostics sur une variable en utilisant la commande suivante :

    ```powershell
    $diagconfig = “<path of the diagnostics configuration file with the Azure Monitor sink configured>”
    ```

1.  Préparez la mise à jour de votre machine virtuelle classique avec le fichier de diagnostics en utilisant le récepteur Azure Monitor configuré.

    ```powershell
    $VM_Update = Set-AzureVMDiagnosticsExtension -DiagnosticsConfigurationPath $diagconfig -VM $VM -StorageContext $Storage_Context
    ```

1.  Déployez la mise à jour sur votre machine virtuelle en exécutant la commande suivante :

    ```powershell
    Update-AzureVM -ServiceName "ClassicVMWAD7216" -Name "ClassicVMWAD" -VM $VM_Update.VM
    ```

> [!NOTE]
> Il est obligatoire de fournir un compte de stockage dans le cadre de l’installation de l’extension Diagnostics. Les journaux d’activité et les compteurs de performances spécifiés dans le fichier config de diagnostics sont écrits dans le compte de stockage spécifié.

## <a name="plot-the-metrics-in-the-azure-portal"></a>Traçage des métriques dans le portail Azure

1.  Accédez au portail Azure. 

1.  Dans le menu de gauche, sélectionnez **Surveiller**.

1.  Dans le panneau **Surveiller**, sélectionnez **Métriques**.

    ![Accéder aux métriques](./media/collect-custom-metrics-guestos-vm-classic/navigate-metrics.png)

1. Dans la liste déroulante des ressources, sélectionnez votre machine virtuelle classique.

1. Dans le menu déroulant des espaces de noms, sélectionnez **azure.vm.windows.guest**.

1. Dans la liste déroulante des métriques, sélectionnez **Mémoire\Octets validés en cours d’utilisation**.
   ![Tracer des métriques](./media/collect-custom-metrics-guestos-vm-classic/plot-metrics.png)


## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [métriques personnalisées](metrics-custom-overview.md).
