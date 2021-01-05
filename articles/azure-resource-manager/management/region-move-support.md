---
title: Prise en charge du déplacement des ressources Azure entre les régions
description: Répertorie les types de ressources Azure qui peuvent être déplacés entre les régions Azure
author: rayne-wiselman
ms.service: azure-resource-manager
ms.topic: reference
ms.date: 08/25/2020
ms.author: raynew
ms.openlocfilehash: 83cd36683a0487f13ab5707e4b1534cc7f20a88a
ms.sourcegitcommit: a2d8acc1b0bf4fba90bfed9241b299dc35753ee6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/12/2020
ms.locfileid: "91948525"
---
# <a name="support-for-moving-azure-resources-across-regions"></a>Prise en charge du déplacement des ressources Azure entre les régions

Cet article confirme si le déplacement d’un type de ressource Azure vers une autre région Azure est pris en charge. 

Accédez à un espace de noms du fournisseur de ressources :
> [!div class="op_single_selector"]
> - [Microsoft.AAD](#microsoftaad)
> - [microsoft.aadiam](#microsoftaadiam)
> - [Microsoft.AlertsManagement](#microsoftalertsmanagement)
> - [Microsoft.AnalysisServices](#microsoftanalysisservices)
> - [Microsoft.ApiManagement](#microsoftapimanagement)
> - [Microsoft.AppConfiguration](#microsoftappconfiguration)
> - [Microsoft.AppService](#microsoftappservice)
> - [Microsoft.Authorization](#microsoftauthorization)
> - [Microsoft.Automation](#microsoftautomation)
> - [Microsoft.AzureActiveDirectory](#microsoftazureactivedirectory)
> - [Microsoft.AzureData](#microsoftazuredata)
> - [Microsoft.AzureStack](#microsoftazurestack)
> - [Microsoft.Batch](#microsoftbatch)
> - [Microsoft.BatchAI](#microsoftbatchai)
> - [Microsoft.BingMaps](#microsoftbingmaps)
> - [Microsoft.BizTalkServices](#microsoftbiztalkservices)
> - [Microsoft.Blockchain](#microsoftblockchain)
> - [Microsoft.Blueprint](#microsoftblueprint)
> - [Microsoft.BotService](#microsoftbotservice)
> - [Microsoft.Cache](#microsoftcache)
> - [Microsoft.Cdn](#microsoftcdn)
> - [Microsoft.CertificateRegistration](#microsoftcertificateregistration)
> - [Microsoft.ClassicCompute](#microsoftclassiccompute)
> - [Microsoft.ClassicNetwork](#microsoftclassicnetwork)
> - [Microsoft.ClassicStorage](#microsoftclassicstorage)
> - [Microsoft.CognitiveServices](#microsoftcognitiveservices)
> - [Microsoft.Compute](#microsoftcompute)
> - [Microsoft.Container](#microsoftcontainer)
> - [Microsoft.ContainerInstance](#microsoftcontainerinstance)
> - [Microsoft.ContainerRegistry](#microsoftcontainerregistry)
> - [Microsoft.ContainerService](#microsoftcontainerservice)
> - [Microsoft.ContentModerator](#microsoftcontentmoderator)
> - [Microsoft.CortanaAnalytics](#microsoftcortanaanalytics)
> - [Microsoft.CostManagement](#microsoftcostmanagement)
> - [Microsoft.CustomerInsights](#microsoftcustomerinsights)
> - [Microsoft.CustomProviders](#microsoftcustomproviders)
> - [Microsoft.DataBox](#microsoftdatabox)
> - [Microsoft.DataBoxEdge](#microsoftdataboxedge)
> - [Microsoft.Databricks](#microsoftdatabricks)
> - [Microsoft.DataCatalog](#microsoftdatacatalog)
> - [Microsoft.DataConnect](#microsoftdataconnect)
> - [Microsoft.DataExchange](#microsoftdataexchange)
> - [Microsoft.DataFactory](#microsoftdatafactory)
> - [Microsoft.DataLake](#microsoftdatalake)
> - [Microsoft.DataLakeAnalytics](#microsoftdatalakeanalytics)
> - [Microsoft.DataLakeStore](#microsoftdatalakestore)
> - [Microsoft.DataMigration](#microsoftdatamigration)
> - [Microsoft.DataShare](#microsoftdatashare)
> - [Microsoft.DBforMariaDB](#microsoftdbformariadb)
> - [Microsoft.DBforMySQL](#microsoftdbformysql)
> - [Microsoft.DBforPostgreSQL](#microsoftdbforpostgresql)
> - [Microsoft.DeploymentManager](#microsoftdeploymentmanager)
> - [Microsoft.Devices](#microsoftdevices)
> - [Microsoft.DevSpaces](#microsoftdevspaces)
> - [Microsoft.DevTestLab](#microsoftdevtestlab)
> - [Microsoft.DocumentDB](#microsoftdocumentdb)
> - [Microsoft.DomainRegistration](#microsoftdomainregistration)
> - [Microsoft.EnterpriseKnowledgeGraph](#microsoftenterpriseknowledgegraph)
> - [Microsoft.EventGrid](#microsofteventgrid)
> - [Microsoft.EventHub](#microsofteventhub)
> - [Microsoft.Genomics](#microsoftgenomics)
> - [Microsoft.HanaOnAzure](#microsofthanaonazure)
> - [Microsoft.HDInsight](#microsofthdinsight)
> - [Microsoft.HealthcareApis](#microsofthealthcareapis)
> - [Microsoft.HybridCompute](#microsofthybridcompute)
> - [Microsoft.HybridData](#microsofthybriddata)
> - [Microsoft.ImportExport](#microsoftimportexport)
> - [microsoft.insights](#microsoftinsights)
> - [Microsoft.IoTCentral](#microsoftiotcentral)
> - [Microsoft.IoTSpaces](#microsoftiotspaces)
> - [Microsoft.KeyVault](#microsoftkeyvault)
> - [Microsoft.Kusto](#microsoftkusto)
> - [Microsoft.LabServices](#microsoftlabservices)
> - [Microsoft.LocationBasedServices](#microsoftlocationbasedservices)
> - [Microsoft.LocationServices](#microsoftlocationservices)
> - [Microsoft.Logic](#microsoftlogic)
> - [Microsoft.MachineLearning](#microsoftmachinelearning)
> - [Microsoft.MachineLearningCompute](#microsoftmachinelearningcompute)
> - [Microsoft.MachineLearningExperimentation](#microsoftmachinelearningexperimentation)
> - [Microsoft.MachineLearningModelManagement](#microsoftmachinelearningmodelmanagement)
> - [Microsoft.MachineLearningOperationalization](#microsoftmachinelearningoperationalization)
> - [Microsoft.MachineLearningServices](#microsoftmachinelearningservices)
> - [Microsoft.ManagedIdentity](#microsoftmanagedidentity)
> - [Microsoft.Maps](#microsoftmaps)
> - [Microsoft.MarketplaceApps](#microsoftmarketplaceapps)
> - [Microsoft.Media](#microsoftmedia)
> - [Microsoft.Microservices4Spring](#microsoftmicroservices4spring)
> - [Microsoft.Migrate](#microsoftmigrate)
> - [Microsoft.NetApp](#microsoftnetapp)
> - [Microsoft.Network](#microsoftnetwork)
> - [Microsoft.NotificationHubs](#microsoftnotificationhubs)
> - [Microsoft.OperationalInsights](#microsoftoperationalinsights)
> - [Microsoft.OperationsManagement](#microsoftoperationsmanagement)
> - [Microsoft.Peering](#microsoftpeering)
> - [Microsoft.Portal](#microsoftportal)
> - [Microsoft.PortalSdk](#microsoftportalsdk)
> - [Microsoft.PowerBI](#microsoftpowerbi)
> - [Microsoft.PowerBIDedicated](#microsoftpowerbidedicated)
> - [Microsoft.ProjectOxford](#microsoftprojectoxford)
> - [Microsoft.RecoveryServices](#microsoftrecoveryservices)
> - [Microsoft.Relay](#microsoftrelay)
> - [Microsoft.ResourceGraph](#microsoftresourcegraph)
> - [Microsoft.Resources](#microsoftresources)
> - [Microsoft.SaaS](#microsoftsaas)
> - [Microsoft.Scheduler](#microsoftscheduler)
> - [Microsoft.Search](#microsoftsearch)
> - [Microsoft.Security](#microsoftsecurity)
> - [Microsoft.ServerManagement](#microsoftservermanagement)
> - [Microsoft.ServiceBus](#microsoftservicebus)
> - [Microsoft.ServiceFabric](#microsoftservicefabric)
> - [Microsoft.ServiceFabricMesh](#microsoftservicefabricmesh)
> - [Microsoft.SignalRService](#microsoftsignalrservice)
> - [Microsoft.Solutions](#microsoftsolutions)
> - [Microsoft.Sql](#microsoftsql)
> - [Microsoft.SqlVirtualMachine](#microsoftsqlvirtualmachine)
> - [Microsoft.SqlVM](#microsoftsqlvm)
> - [Microsoft.Storage](#microsoftstorage)
> - [Microsoft.StorageCache](#microsoftstoragecache)
> - [Microsoft.StorageSync](#microsoftstoragesync)
> - [Microsoft.StorageSyncDev](#microsoftstoragesyncdev)
> - [Microsoft.StorageSyncInt](#microsoftstoragesyncint)
> - [Microsoft.StorSimple](#microsoftstorsimple)
> - [Microsoft.StreamAnalytics](#microsoftstreamanalytics)
> - [Microsoft.StreamAnalyticsExplorer](#microsoftstreamanalyticsexplorer)
> - [Microsoft.TerraformOSS](#microsoftterraformoss)
> - [Microsoft.TimeSeriesInsights](#microsofttimeseriesinsights)
> - [Microsoft.Token](#microsofttoken)
> - [Microsoft.VirtualMachineImages](#microsoftvirtualmachineimages)
> - [microsoft.visualstudio](#microsoftvisualstudio)
> - [Microsoft.VMwareCloudSimple](#microsoftvmwarecloudsimple)
> - [Microsoft.Web](#microsoftweb)
> - [Microsoft.WindowsIoT](#microsoftwindowsiot)
> - [Microsoft.WindowsVirtualDesktop](#microsoftwindowsvirtualdesktop)

## <a name="microsoftaad"></a>Microsoft.AAD

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- | 
> | domainservices | Non | 
> | domainservices / replicasets | Non | 

## <a name="microsoftaadiam"></a>microsoft.aadiam

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | tenants | Non |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | actionrules | Non | 

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | servers | Non |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | service |  Oui, (utilisation d’un modèle) <br/><br/> [Déplacer la Gestion des API d'une région vers une autre](../../api-management/api-management-howto-migrate.md). | 

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | configurationstores | Non | 

## <a name="microsoftappservice"></a>Microsoft.AppService

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | apiapps | Oui, (utilisation d’un modèle)<br/><br/> [Déplacer une application App Service vers une autre région](../../app-service/manage-move-across-regions.md) | 
> | appidentities | Non | 
> | gateways | Non | 


## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | policyassignments | Non |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | automationaccounts | Oui, (utilisation d’un modèle) <br/><br/> [Utilisation de la géoréplication](../../automation/automation-managing-data.md#geo-replication-in-azure-automation) |  
> | automationaccounts / configurations | Non | 
> | automationaccounts / runbooks | Non | 



## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | b2cdirectories | Non | 

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | sqlserverregistrations | Non |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | registrations | Non | 

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | batchaccounts |  Les comptes Batch ne peuvent pas être déplacés directement d’une région à une autre, mais vous pouvez utiliser un modèle pour exporter un modèle, le modifier et déployer le modèle dans la nouvelle région. <br/><br/> Apprenez-en davantage sur le [déplacement d’un compte Batch d’une région à une autre](../../batch/best-practices.md#moving-batch-accounts-across-regions). |

## <a name="microsoftbatchai"></a>Microsoft.BatchAI

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | clusters | Non <br/><br/> Le service Azure Batch AI a été mis [hors service](/previous-versions/azure/batch-ai/overview-what-happened-batch-ai).
> | fileservers | Non | 
> | jobs | Non | 
> | workspaces | Non | 

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | mapapis | Non | 

## <a name="microsoftbiztalkservices"></a>Microsoft.BizTalkServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | biztalk | Non | 

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | blockchainmembers | Non <br/><br/> Le réseau blockchain ne peut pas avoir de nœuds dans des régions différentes. 
> | watchers | Non | 

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | blueprintassignments | Non | 

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | botservices | Non | 

## <a name="microsoftcache"></a>Microsoft.Cache

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | redis | Non | 


## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | cdnwebapplicationfirewallpolicies | Non |
> | profiles | Non | 
> | profiles / endpoints | Non | 

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | certificateorders | Non | 


## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | domainnames | Aucun travail n’est planifié pour les services classiques.
> | virtualmachines | Non | 



## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | networksecuritygroups | Aucun travail n’est planifié pour les services classiques.
> | reservedips | Non | 
> | virtualnetworks | Non | 

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | storageaccounts | Oui |  


## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non | 
> | Recherche cognitive | Pris en charge avec étapes manuelles.<br/><br/> Apprenez-en davantage sur le [déplacement de votre service Recherche cognitive Azure vers une autre région](../../search/search-howto-move-across-regions.md).

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | availabilitysets | Oui <br/><br/> Utilisez [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) pour déplacer des groupes à haute disponibilité. | 
> | diskencryptionsets | Non | 
> | disks | Oui <br/><br/> Utilisez [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) pour déplacer des machines virtuelles Azure et des disques associés. | 
> | galleries | Non | 
> | galleries / images | Non | 
> | galleries / images / versions | Non | 
> | hostgroups | Non | 
> | hostgroups / hosts | Non | 
> | images | Non | 
> | proximityplacementgroups | Non | 
> | restorepointcollections | Non | 
> | sharedvmimages | Non | 
> | sharedvmimages / versions | Non | 
> | snapshots | Non | 
> | virtualmachines | Oui <br/><br/> Utilisez [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) pour déplacer des machines virtuelles Azure. | 
> | virtualmachines / extensions | Non | 
> | virtualmachinescalesets | Non | 

## <a name="microsoftcontainer"></a>Microsoft.Container

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | containergroups | Non | 

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | containergroups | Non | 

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | registries | Non |  
> | registries / buildtasks | Non |  
> | registries / replications | Non | 
> | registries / tasks | Non |  
> | registries / webhooks | Non | 

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | containerservices | Non.<br/><br/> Le service a été mis [hors service](https://azure.microsoft.com/updates/azure-container-service-will-retire-on-january-31-2020/).
> | managedclusters | Non | 
> | openshiftmanagedclusters | Non | 

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | applications | Non | 

## <a name="microsoftcortanaanalytics"></a>Microsoft.CortanaAnalytics

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non | 

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | connecteurs | Non |  

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | hubs | Non |  

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | resourceproviders | Non | 

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | jobs | Non | 

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | databoxedgedevices | Non | 

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | workspaces | Non | 

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | catalogs | Non | 
> | datacatalogs | Non | 

## <a name="microsoftdataconnect"></a>Microsoft.DataConnect

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | connectionmanagers | Non | 

## <a name="microsoftdataexchange"></a>Microsoft.DataExchange

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | packages | Non | 
> | plans | Non | 

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | datafactories | Non | 
> | factories | Non |  

## <a name="microsoftdatalake"></a>Microsoft.DataLake

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | datalakeaccounts | Non | 

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non | 

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non | 

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | services | Non | 
> | services / projects | Non | 
> | slots | Non | 

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non | 

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | servers | Vous pouvez utiliser un réplica en lecture sur plusieurs régions pour déplacer un serveur existant. [Plus d’informations](../../postgresql/howto-move-regions-portal.md)<br/><br/> Si le service est provisionné avec un stockage de sauvegarde géoredondant, vous pouvez utiliser la géorestauration pour effectuer une restauration dans d’autres régions. [Plus d’informations](../../mariadb/concepts-business-continuity.md#recover-from-an-azure-regional-data-center-outage)

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | servers | Vous pouvez utiliser un réplica en lecture sur plusieurs régions pour déplacer un serveur existant. [Plus d’informations](../../mysql/howto-move-regions-portal.md)

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | servergroups | Non | 
> | servers | Vous pouvez utiliser un réplica en lecture sur plusieurs régions pour déplacer un serveur existant. [En savoir plus](../../postgresql/howto-move-regions-portal.md).
> | serversv2 | Non | 

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | artifactsources | Non | 
> | rollouts | Non |  
> | servicetopologies | Non | 
> | servicetopologies / services | Non |  
> | servicetopologies / services / serviceunits | Non | 
> | steps | Non | 

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | elasticpools | Non. La ressource n’est pas exposée.
> | elasticpools / iothubtenants | Non. La ressource n’est pas exposée.
> | iothubs | Oui. [En savoir plus](../../iot-hub/iot-hub-how-to-clone.md)
> | provisioningservices | Non | 

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | controllers | Non | 
> | Cluster AKS | Non<br/><br/> [Apprenez-en davantage](../../dev-spaces/faq.md#can-i-migrate-my-aks-cluster-with-azure-dev-spaces-to-another-region) sur le déplacement vers une autre région.

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | labcenters | Non | 
> | labs | Non | 
> | labs / environments | Non |  
> | labs / servicerunners | Non | 
> | labs / virtualmachines | Non |  
> | schedules | Non |  

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | databaseaccounts | Non | 

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | domaines | Non | 

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | services | Non |  

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | domaines | Non |  
> | topics | Non | 

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | clusters | Non |  
> | espaces de noms | Oui (avec modèle)<br/><br/> [Déplacer un espace de noms Event Hubs vers une autre région](../../event-hubs/move-across-regions.md) | 

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non | 

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | hanainstances | Non | 
> | sapmonitors | Non |  

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | clusters | Non | 

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | services | Non |  

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | machines | Non | 

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | datamanagers |  Non | 

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | jobs |  Non | 

## <a name="microsoftinsights"></a>microsoft.insights

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non. [Plus d’informations](../../azure-monitor/faq.md#how-do-i-move-an-application-insights-resource-to-a-new-region)
> | actiongroups |  Non | 
> | activitylogalerts | Non | 
> | alertrules |  Non | 
> | autoscalesettings |  Non | 
> | components |  Non |  
> | guestdiagnosticsettings | Non | 
> | metricalerts | Non | 
> | notificationgroups | Non | 
> | notificationrules | Non | 
> | scheduledqueryrules |  Non | 
> | webtests |  Non | 
> | workbooks |  Non |  


## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | checknameavailability |  Non.<br/><br/> IoT Central fonctionne avec des zones géographiques, et non des régions.
> | graph | Non

## <a name="microsoftiothub"></a>Microsoft.IoTHub

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> |  iothub |  Oui (cloner un hub) <br/><br/> [Cloner un hub IoT dans une autre région](../../iot-hub/iot-hub-how-to-clone.md)

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | checknameavailability |  Non |  
> | graph |  Non | 

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | hsmpools | Non | 
> | vaults |  Non | 


## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | clusters |  Non |  

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | labaccounts | Non | 

## <a name="microsoftlocationbasedservices"></a>Microsoft.LocationBasedServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non | 

## <a name="microsoftlocationservices"></a>Microsoft.LocationServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non, il s’agit d’un service mondial.

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | hostingenvironments | Non | 
> | integrationaccounts |  Non |  
> | integrationserviceenvironments | Non | 
> | isolatedenvironments | Non | 
> | workflows |  Non |  

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | commitmentplans |  Non | 
> | webservices |  Non | 
> | workspaces |  Non | 

## <a name="microsoftmachinelearningcompute"></a>Microsoft.MachineLearningCompute

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | operationalizationclusters |  Non | 

## <a name="microsoftmachinelearningexperimentation"></a>Microsoft.MachineLearningExperimentation

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non | 
> | accounts / workspaces | Non | 
> | accounts / workspaces / projects | Non | 
> | teamaccounts | Non | 
> | teamaccounts / workspaces | Non | 
> | teamaccounts / workspaces / projects | Non | 

## <a name="microsoftmachinelearningmodelmanagement"></a>Microsoft.MachineLearningModelManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non | 

## <a name="microsoftmachinelearningoperationalization"></a>Microsoft.MachineLearningOperationalization

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | hostingaccounts | Non | 

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | workspaces | Non | 

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | userassignedidentities | Non | 

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts |  Non, Azure Maps est un service géospatial. 

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | classicdevservices | Aucun travail n’est planifié pour les services classiques. 

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | mediaservices |  Non | 
> | mediaservices / liveevents |  Non | 
> | mediaservices / streamingendpoints |  Non | 

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | appclusters | Non | 

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | assessmentprojects | Non | 
> | migrateprojects | Non | 
> | projects | Non | 

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | netappaccounts | Non | 
> | netappaccounts / capacitypools | Non | 
> | netappaccounts / capacitypools / volumes | Non | 
> | netappaccounts / capacitypools / volumes / mounttargets | Non | 
> | netappaccounts / capacitypools / volumes / snapshots | Non | 

## <a name="microsoftnetwork"></a>Microsoft.Network

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | applicationgateways | Non | 
> | applicationgatewaywebapplicationfirewallpolicies | Non | 
> | applicationsecuritygroups |  Non |  
> | azurefirewalls |  Non |  
> | bastionhosts | Non | 
> | connections |  Non | 
> | ddoscustompolicies |  Non | 
> | ddosprotectionplans | Non | 
> | dnszones |  Non | 
> | expressroutecircuits | Non | 
> | expressroutecrossconnections | Non | 
> | expressroutegateways | Non | 
> | expressrouteports | Non | 
> | frontdoors | Non | 
> | frontdoorwebapplicationfirewallpolicies | Non | 
> | loadbalancers | Oui <br/><br/> Utilisez [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) pour déplacer des équilibreurs de charge internes et externes. |
> | localnetworkgateways |  Non | 
> | natgateways |  Non | 
> | networkintentpolicies |  Non | 
> | networkinterfaces | Oui <br/><br/> Utilisez [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) pour déplacer des cartes réseau. | 
> | networkprofiles | Non | 
> | networksecuritygroups | Oui <br/><br/> Utilisez [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) pour déplacer des groupes de sécurité réseau (NGS). | 
> | networkwatchers |  Non |  
> | networkwatchers / connectionmonitors |  Non | 
> | networkwatchers / lenses |  Non | 
> | networkwatchers / pingmeshes |  Non | 
> | p2svpngateways | Non | 
> | privatednszones |  Non |  
> | privatednszones / virtualnetworklinks |  Non |  
> | privateendpoints | Non | 
> | privatelinkservices | Non | 
> | publicipaddresses | Oui<br/><br/> Utilisez [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) pour déplacer des adresses IP publiques. |
> | publicipprefixes | Non | 
> | routefilters | Non | 
> | routetables |  Non | 
> | serviceendpointpolicies |  Non | 
> | trafficmanagerprofiles |  Non | 
> | virtualhubs | Non | 
> | virtualnetworkgateways |  Non |  
> | virtualnetworks |  Non | 
> | virtualnetworktaps | Non | 
> | virtualwans | Non | 
> | vpngateways (Virtual WAN) | Non | 
> | vpnsites (Virtual WAN) | Non | 
> | webapplicationfirewallpolicies |  Non | 


## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | espaces de noms |  Non | 
> | namespaces / notificationhubs |  Non |  

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | workspaces |  Non | 



## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | managementconfigurations |  Non | 
> | views |  Non | 

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | peerings | Non | 

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | dashboards | Non | 

## <a name="microsoftportalsdk"></a>Microsoft.PortalSdk

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | rootresources | Non | 

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | workspacecollections |  Non | 

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | capacities |  Non | 

## <a name="microsoftprojectoxford"></a>Microsoft.ProjectOxford

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | accounts | Non | 

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | vaults | Non.<br/><br/> Le déplacement de coffres Recovery Services pour Sauvegarde Azure d’une région Azure à une autre n’est pas pris en charge.<br/><br/> Dans les coffres Recovery Services pour Azure Site Recovery, vous pouvez [désactiver et recréer le coffre](../../site-recovery/move-vaults-across-regions.md) dans la région cible. | 


## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | espaces de noms |  Non | 

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | queries |  Non |  

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région |
> | ------------- | ----------- |
> | deploymentScripts |  Oui<br/><br/>[Déplacer des ressources Microsoft.Resources vers une nouvelle région](microsoft-resources-move-regions.md) |
> | templateSpecs |  Oui<br/><br/>[Déplacer des ressources Microsoft.Resources vers une nouvelle région](microsoft-resources-move-regions.md) |  

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | applications |  Non | 

## <a name="microsoftscheduler"></a>Microsoft.Scheduler

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | flows |  Non |  
> | jobcollections |  Non | 

## <a name="microsoftsearch"></a>Microsoft.Search

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | searchservices |  Non | 


## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | iotsecuritysolutions |  Non | 
> | playbookconfigurations | Non | 

## <a name="microsoftservermanagement"></a>Microsoft.ServerManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | gateways | Non | 
> | nœuds | Non | 

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | espaces de noms |  Non | 

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | applications | Non | 
> | clusters |  Non | 
> | clusters / applications | Non | 
> | containergroups | Non | 
> | containergroupsets | Non | 
> | edgeclusters | Non | 
> | networks | Non | 
> | secretstores | Non | 
> | volumes | Non | 

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | applications |  Non | 
> | containergroups | Non | 
> | gateways |  Non | 
> | networks |  Non | 
> | secrets |  Non | 
> | volumes |  Non |  

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | signalr |  Non |  

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | appliancedefinitions | Non | 
> | appliances | Non | 
> | applicationdefinitions | Non | 
> | applications | Non | 
> | jitrequests | Non | 

## <a name="microsoftsql"></a>Microsoft.Sql

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | instancepools | Non | 
> | managedinstances | Oui <br/><br/> [Apprenez-en davantage](../../azure-sql/database/move-resources-across-regions.md) sur le déplacement des instances managées entre les régions. | 
> | managedinstances / databases | Oui | 
> | servers | Oui | 
> | servers / databases | Oui <br/><br/> [Apprenez-en davantage](../../azure-sql/database/move-resources-across-regions.md) sur le déplacement des bases de données entre les régions.<br/><br/> [Apprenez-en davantage](../../resource-mover/tutorial-move-region-sql.md) sur l’utilisation d’Azure Resource Mover pour déplacer des bases de données Azure SQL.  | 
> | servers / elasticpools | Oui <br/><br/> [Apprenez-en davantage](../../azure-sql/database/move-resources-across-regions.md) sur le déplacement des pools élastiques entre les régions.<br/><br/> [Apprenez-en davantage](../../resource-mover/tutorial-move-region-sql.md) sur l’utilisation d’Azure Resource Mover pour déplacer des pools élastiques Azure SQL.  | 
> | virtualclusters | Oui | 

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | sqlvirtualmachinegroups |  Non |  
> | sqlvirtualmachines |  Non |  

## <a name="microsoftsqlvm"></a>Microsoft.SqlVM

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | dwvm | Non | 

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | storageaccounts | Oui<br/><br/> [Déplacer un compte Stockage Azure vers une autre région](../../storage/common/storage-account-move.md) | 

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | caches | Non | 

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | storagesyncservices |  Non | 

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | storagesyncservices | Non | 

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | storagesyncservices | Non | 

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | managers | Non | 

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | streamingjobs |  Non |  


## <a name="microsoftstreamanalyticsexplorer"></a>Microsoft.StreamAnalyticsExplorer

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | environments | Non | 
> | environments / eventsources | Non | 
> | instances | Non | 
> | instances / environments | Non | 
> | instances / environments / eventsources | Non | 

## <a name="microsoftterraformoss"></a>Microsoft.TerraformOSS

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | providerregistrations | Non | 
> | les ressources | Non | 

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | environments |  Non | 
> | environments / eventsources |  Non |  
> | environments / referencedatasets |  Non | 

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | stores | Non | 

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | imagetemplates | Non | 

## <a name="microsoftvisualstudio"></a>microsoft.visualstudio

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | account |  Non | 
> | account / extension |  Non | 
> | account / project |  Non | 



## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | dedicatedcloudnodes | Non | 
> | dedicatedcloudservices | Non | 
> | virtualmachines | Non | 

## <a name="microsoftweb"></a>Microsoft.Web

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | certificates | Non | 
> | connectiongateways |  Non |  
> | connections |  Non |  
> | customapis |  Non | 
> | hostingenvironments | Non | 
> | serverfarms |  Non |  
> | sites |  Non | 
> | sites / premieraddons |  Non |  
> | sites / slots |  Non |  


## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | deviceservices | Non | 

## <a name="microsoftwindowsvirtualdesktop"></a>Microsoft.WindowsVirtualDesktop

> [!div class="mx-tableFixed"]
> | Type de ressource | Changement de région | 
> | ------------- | ----------- |
> | applicationgroups | Non | 
> | hostpools | Non | 
> | workspaces | Non | 

## <a name="third-party-services"></a>Services tiers

Actuellement, les services tiers ne prennent pas en charge l’opération de déplacement.