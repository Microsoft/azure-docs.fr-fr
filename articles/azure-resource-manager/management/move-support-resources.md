---
title: Prise en charge des opérations de déplacement par type de ressource
description: Liste les types de ressources Azure qui peuvent être déplacés vers un nouveau groupe de ressources ou un nouvel abonnement.
ms.topic: conceptual
ms.date: 12/07/2020
ms.openlocfilehash: 0b1304b273523d42ad62f4e392fb803e1c174b69
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96937523"
---
# <a name="move-operation-support-for-resources"></a>Prise en charge de l’opération de déplacement pour les ressources

Cet article indique si un type de ressource Azure donné prend en charge l’opération de déplacement. Il fournit également des informations sur les conditions spéciales à prendre en compte lors du déplacement d’une ressource.

> [!IMPORTANT]
> Dans la plupart des cas, une ressource enfant ne peut pas être déplacée indépendamment de sa ressource parente. Les ressources enfants ont un type de ressource au format `<resource-provider-namespace>/<parent-resource>/<child-resource>`. Par exemple, `Microsoft.ServiceBus/namespaces/queues` est une ressource enfant de `Microsoft.ServiceBus/namespaces`. Lorsque vous déplacez la ressource parente, la ressource enfant est automatiquement déplacée avec elle. Si vous ne voyez pas de ressource enfant dans cet article, vous pouvez supposer qu’elle est déplacée avec la ressource parente. Si la ressource parente ne prend pas en charge le déplacement, la ressource enfant ne peut pas être déplacée.

Accédez à un espace de noms du fournisseur de ressources :
> [!div class="op_single_selector"]
> - [Microsoft.AAD](#microsoftaad)
> - [microsoft.aadiam](#microsoftaadiam)
> - [Microsoft.Addons](#microsoftaddons)
> - [Microsoft.ADHybridHealthService](#microsoftadhybridhealthservice)
> - [Microsoft.Advisor](#microsoftadvisor)
> - [Microsoft.AlertsManagement](#microsoftalertsmanagement)
> - [Microsoft.AnalysisServices](#microsoftanalysisservices)
> - [Microsoft.ApiManagement](#microsoftapimanagement)
> - [Microsoft.AppConfiguration](#microsoftappconfiguration)
> - [Microsoft.AppPlatform](#microsoftappplatform)
> - [Microsoft.AppService](#microsoftappservice)
> - [Microsoft.Attestation](#microsoftattestation)
> - [Microsoft.Authorization](#microsoftauthorization)
> - [Microsoft.Automation](#microsoftautomation)
> - [Microsoft.AVS](#microsoftavs)
> - [Microsoft.AzureActiveDirectory](#microsoftazureactivedirectory)
> - [Microsoft.AzureData](#microsoftazuredata)
> - [Microsoft.AzureStack](#microsoftazurestack)
> - [Microsoft.AzureStackHCI](#microsoftazurestackhci)
> - [Microsoft.Batch](#microsoftbatch)
> - [Microsoft.Billing](#microsoftbilling)
> - [Microsoft.BingMaps](#microsoftbingmaps)
> - [Microsoft.BizTalkServices](#microsoftbiztalkservices)
> - [Microsoft.Blockchain](#microsoftblockchain)
> - [Microsoft.BlockchainTokens](#microsoftblockchaintokens)
> - [Microsoft.Blueprint](#microsoftblueprint)
> - [Microsoft.BotService](#microsoftbotservice)
> - [Microsoft.Cache](#microsoftcache)
> - [Microsoft.Capacity](#microsoftcapacity)
> - [Microsoft.Cdn](#microsoftcdn)
> - [Microsoft.CertificateRegistration](#microsoftcertificateregistration)
> - [Microsoft.ClassicCompute](#microsoftclassiccompute)
> - [Microsoft.ClassicInfrastructureMigrate](#microsoftclassicinfrastructuremigrate)
> - [Microsoft.ClassicNetwork](#microsoftclassicnetwork)
> - [Microsoft.ClassicStorage](#microsoftclassicstorage)
> - [Microsoft.ClassicSubscription](#microsoftclassicsubscription)
> - [Microsoft.CognitiveServices](#microsoftcognitiveservices)
> - [Microsoft.Commerce](#microsoftcommerce)
> - [Microsoft.Compute](#microsoftcompute)
> - [Microsoft.Consumption](#microsoftconsumption)
> - [Microsoft.ContainerInstance](#microsoftcontainerinstance)
> - [Microsoft.ContainerRegistry](#microsoftcontainerregistry)
> - [Microsoft.ContainerService](#microsoftcontainerservice)
> - [Microsoft.ContentModerator](#microsoftcontentmoderator)
> - [Microsoft.CortanaAnalytics](#microsoftcortanaanalytics)
> - [Microsoft.CostManagement](#microsoftcostmanagement)
> - [Microsoft.CustomerInsights](#microsoftcustomerinsights)
> - [Microsoft.CustomerLockbox](#microsoftcustomerlockbox)
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
> - [Microsoft.DataProtection](#microsoftdataprotection)
> - [Microsoft.DataShare](#microsoftdatashare)
> - [Microsoft.DBforMariaDB](#microsoftdbformariadb)
> - [Microsoft.DBforMySQL](#microsoftdbformysql)
> - [Microsoft.DBforPostgreSQL](#microsoftdbforpostgresql)
> - [Microsoft.DeploymentManager](#microsoftdeploymentmanager)
> - [Microsoft.DesktopVirtualization](#microsoftdesktopvirtualization)
> - [Microsoft.Devices](#microsoftdevices)
> - [Microsoft.DevOps](#microsoftdevops)
> - [Microsoft.DevSpaces](#microsoftdevspaces)
> - [Microsoft.DevTestLab](#microsoftdevtestlab)
> - [Microsoft.DigitalTwins](#microsoftdigitaltwins)
> - [Microsoft.DocumentDB](#microsoftdocumentdb)
> - [Microsoft.DomainRegistration](#microsoftdomainregistration)
> - [Microsoft.EnterpriseKnowledgeGraph](#microsoftenterpriseknowledgegraph)
> - [Microsoft.EventGrid](#microsofteventgrid)
> - [Microsoft.EventHub](#microsofteventhub)
> - [Microsoft.Experimentation](#microsoftexperimentation)
> - [Microsoft.Falcon](#microsoftfalcon)
> - [Microsoft.Features](#microsoftfeatures)
> - [Microsoft.Genomics](#microsoftgenomics)
> - [Microsoft.GuestConfiguration](#microsoftguestconfiguration)
> - [Microsoft.HanaOnAzure](#microsofthanaonazure)
> - [Microsoft.HardwareSecurityModules](#microsofthardwaresecuritymodules)
> - [Microsoft.HDInsight](#microsofthdinsight)
> - [Microsoft.HealthcareApis](#microsofthealthcareapis)
> - [Microsoft.HybridCompute](#microsofthybridcompute)
> - [Microsoft.HybridData](#microsofthybriddata)
> - [Microsoft.HybridNetwork](#microsofthybridnetwork)
> - [Microsoft.Hydra](#microsofthydra)
> - [Microsoft.ImportExport](#microsoftimportexport)
> - [microsoft.insights](#microsoftinsights)
> - [Microsoft.IoTCentral](#microsoftiotcentral)
> - [Microsoft.IoTSpaces](#microsoftiotspaces)
> - [Microsoft.KeyVault](#microsoftkeyvault)
> - [Microsoft.Kubernetes](#microsoftkubernetes)
> - [Microsoft.KubernetesConfiguration](#microsoftkubernetesconfiguration)
> - [Microsoft.Kusto](#microsoftkusto)
> - [Microsoft.LabServices](#microsoftlabservices)
> - [Microsoft.LocationBasedServices](#microsoftlocationbasedservices)
> - [Microsoft.LocationServices](#microsoftlocationservices)
> - [Microsoft.Logic](#microsoftlogic)
> - [Microsoft.MachineLearning](#microsoftmachinelearning)
> - [Microsoft.MachineLearningCompute](#microsoftmachinelearningcompute)
> - [Microsoft.MachineLearningExperimentation](#microsoftmachinelearningexperimentation)
> - [Microsoft.MachineLearningModelManagement](#microsoftmachinelearningmodelmanagement)
> - [Microsoft.MachineLearningServices](#microsoftmachinelearningservices)
> - [Microsoft.Maintenance](#microsoftmaintenance)
> - [Microsoft.ManagedIdentity](#microsoftmanagedidentity)
> - [Microsoft.ManagedNetwork](#microsoftmanagednetwork)
> - [Microsoft.ManagedServices](#microsoftmanagedservices)
> - [Microsoft.Management](#microsoftmanagement)
> - [Microsoft.Maps](#microsoftmaps)
> - [Microsoft.Marketplace](#microsoftmarketplace)
> - [Microsoft.MarketplaceApps](#microsoftmarketplaceapps)
> - [Microsoft.MarketplaceOrdering](#microsoftmarketplaceordering)
> - [Microsoft.Media](#microsoftmedia)
> - [Microsoft.Microservices4Spring](#microsoftmicroservices4spring)
> - [Microsoft.Migrate](#microsoftmigrate)
> - [Microsoft.MixedReality](#microsoftmixedreality)
> - [Microsoft.NetApp](#microsoftnetapp)
> - [Microsoft.Network](#microsoftnetwork)
> - [Microsoft.NotificationHubs](#microsoftnotificationhubs)
> - [Microsoft.ObjectStore](#microsoftobjectstore)
> - [Microsoft.OffAzure](#microsoftoffazure)
> - [Microsoft.OperationalInsights](#microsoftoperationalinsights)
> - [Microsoft.OperationsManagement](#microsoftoperationsmanagement)
> - [Microsoft.Peering](#microsoftpeering)
> - [Microsoft.PolicyInsights](#microsoftpolicyinsights)
> - [Microsoft.Portal](#microsoftportal)
> - [Microsoft.PowerBI](#microsoftpowerbi)
> - [Microsoft.PowerBIDedicated](#microsoftpowerbidedicated)
> - [Microsoft.ProjectBabylon](#microsoftprojectbabylon)
> - [Microsoft.ProviderHub](#microsoftproviderhub)
> - [Microsoft.Quantum](#microsoftquantum)
> - [Microsoft.RecoveryServices](#microsoftrecoveryservices)
> - [Microsoft.RedHatOpenShift](#microsoftredhatopenshift)
> - [Microsoft.Relay](#microsoftrelay)
> - [Microsoft.ResourceGraph](#microsoftresourcegraph)
> - [Microsoft.ResourceHealth](#microsoftresourcehealth)
> - [Microsoft.Resources](#microsoftresources)
> - [Microsoft.SaaS](#microsoftsaas)
> - [Microsoft.Search](#microsoftsearch)
> - [Microsoft.Security](#microsoftsecurity)
> - [Microsoft.SecurityInsights](#microsoftsecurityinsights)
> - [Microsoft.SerialConsole](#microsoftserialconsole)
> - [Microsoft.ServerManagement](#microsoftservermanagement)
> - [Microsoft.ServiceBus](#microsoftservicebus)
> - [Microsoft.ServiceFabric](#microsoftservicefabric)
> - [Microsoft.ServiceFabricMesh](#microsoftservicefabricmesh)
> - [Microsoft.Services](#microsoftservices)
> - [Microsoft.SignalRService](#microsoftsignalrservice)
> - [Microsoft.SoftwarePlan](#microsoftsoftwareplan)
> - [Microsoft.Solutions](#microsoftsolutions)
> - [Microsoft.Sql](#microsoftsql)
> - [Microsoft.SqlVirtualMachine](#microsoftsqlvirtualmachine)
> - [Microsoft.Storage](#microsoftstorage)
> - [Microsoft.StorageCache](#microsoftstoragecache)
> - [Microsoft.StorageSync](#microsoftstoragesync)
> - [Microsoft.StorageSyncDev](#microsoftstoragesyncdev)
> - [Microsoft.StorageSyncInt](#microsoftstoragesyncint)
> - [Microsoft.StorSimple](#microsoftstorsimple)
> - [Microsoft.StreamAnalytics](#microsoftstreamanalytics)
> - [Microsoft.StreamAnalyticsExplorer](#microsoftstreamanalyticsexplorer)
> - [Microsoft.Subscription](#microsoftsubscription)
> - [microsoft.support](#microsoftsupport)
> - [Microsoft.Synapse](#microsoftsynapse)
> - [Microsoft.TimeSeriesInsights](#microsofttimeseriesinsights)
> - [Microsoft.Token](#microsofttoken)
> - [Microsoft.VirtualMachineImages](#microsoftvirtualmachineimages)
> - [microsoft.visualstudio](#microsoftvisualstudio)
> - [Microsoft.VMware](#microsoftvmware)
> - [Microsoft.VMwareCloudSimple](#microsoftvmwarecloudsimple)
> - [Microsoft.VnfManager](#microsoftvnfmanager)
> - [Microsoft.VSOnline](#microsoftvsonline)
> - [Microsoft.Web](#microsoftweb)
> - [Microsoft.WindowsESU](#microsoftwindowsesu)
> - [Microsoft.WindowsIoT](#microsoftwindowsiot)
> - [Microsoft.WorkloadBuilder](#microsoftworkloadbuilder)
> - [Microsoft.WorkloadMonitor](#microsoftworkloadmonitor)

## <a name="microsoftaad"></a>Microsoft.AAD

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | domainservices | Non | Non |

## <a name="microsoftaadiam"></a>microsoft.aadiam

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | diagnosticsettings | Non | Non |
> | diagnosticsettingscategories | Non | Non |
> | privatelinkforazuread | Oui | Oui |
> | tenants | Oui | Oui |

## <a name="microsoftaddons"></a>Microsoft.Addons

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | supportproviders | Non | Non |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | aadsupportcases | Non | Non |
> | addsservices | Non | Non |
> | agents | Non | Non |
> | anonymousapiusers | Non | Non |
> | configuration | Non | Non |
> | logs | Non | Non |
> | reports | Non | Non |
> | servicehealthmetrics | Non | Non |
> | services | Non | Non |

## <a name="microsoftadvisor"></a>Microsoft.Advisor

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | configurations | Non | Non |
> | generaterecommendations | Non | Non |
> | metadata | Non | Non |
> | de films | Non | Non |
> | suppressions | Non | Non |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | actionrules | Oui | Oui |
> | alertes | Non | Non |
> | alertslist | Non | Non |
> | alertsmetadata | Non | Non |
> | alertssummary | Non | Non |
> | alertssummarylist | Non | Non |
> | smartdetectoralertrules | Oui | Oui |
> | smartgroups | Non | Non |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | servers | Oui | Oui |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!IMPORTANT]
> Un service Gestion des API dont la référence SKU définie est Consommation ne peut pas être déplacé.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | reportfeedback | Non | Non |
> | service | Oui | Oui |

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | configurationstores | Oui | Oui |
> | configurationstores / eventgridfilters | Non | Non |

## <a name="microsoftappplatform"></a>Microsoft.AppPlatform

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | spring | Oui | Oui |

## <a name="microsoftappservice"></a>Microsoft.AppService

> [!IMPORTANT]
> Consultez les [Conseils de déplacement d’App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | apiapps | Non | Non |
> | appidentities | Non | Non |
> | gateways | Non | Non |

## <a name="microsoftattestation"></a>Microsoft.Attestation

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | attestationproviders | Oui | Oui |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | classicadministrators | Non | Non |
> | dataaliases | Non | Non |
> | denyassignments | Non | Non |
> | elevateaccess | Non | Non |
> | findorphanroleassignments | Non | Non |
> | locks | Non | Non |
> | autorisations | Non | Non |
> | policyassignments | Non | Non |
> | policydefinitions | Non | Non |
> | policysetdefinitions | Non | Non |
> | privatelinkassociations | Non | Non |
> | resourcemanagementprivatelinks | Non | Non |
> | roleassignments | Non | Non |
> | roleassignmentsusagemetrics | Non | Non |
> | roledefinitions | Non | Non |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!IMPORTANT]
> Les runbooks doivent exister dans le même groupe de ressources que le compte Automation.
>
> Pour plus d'informations, consultez [Déplacer votre compte Azure Automation vers un autre abonnement](../../automation/how-to/move-account.md?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | automationaccounts | Oui | Oui |
> | automationaccounts / configurations | Oui | Oui |
> | automationaccounts / runbooks | Oui | Oui |

## <a name="microsoftavs"></a>Microsoft.AVS

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | privateclouds | Oui | Oui |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | b2cdirectories | Oui | Oui |
> | b2ctenants | Non | Non |

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | datacontrollers | Non | Non |
> | hybriddatamanagers | Non | Non |
> | postgresinstances | Non | Non |
> | sqlinstances | Non | Non |
> | sqlmanagedinstances | Non | Non |
> | sqlserverinstances | Non | Non |
> | sqlserverregistrations | Oui | Oui |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | cloudmanifestfiles | Non | Non |
> | registrations | Oui | Oui |

## <a name="microsoftazurestackhci"></a>Microsoft.AzureStackHCI

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | clusters | Non | Non |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | batchaccounts | Oui | Oui |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | billingaccounts | Non | Non |
> | billingperiods | Non | Non |
> | billingpermissions | Non | Non |
> | billingproperty | Non | Non |
> | billingroleassignments | Non | Non |
> | billingroledefinitions | Non | Non |
> | departments | Non | Non |
> | enrollmentaccounts | Non | Non |
> | invoices | Non | Non |
> | transfers | Non | Non |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | mapapis | Non | Non |

## <a name="microsoftbiztalkservices"></a>Microsoft.BizTalkServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | biztalk | Non | Non |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | blockchainmembers | Non | Non |
> | cordamembers | Non | Non |
> | watchers | Non | Non |

## <a name="microsoftblockchaintokens"></a>Microsoft.BlockchainTokens

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | tokenservices | Non | Non |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | blueprintassignments | Non | Non |
> | blueprints | Non | Non |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | botservices | Oui | Oui |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!IMPORTANT]
> Si l’instance du Cache Azure pour Redis est configurée avec un réseau virtuel, l’instance ne peut pas être déplacée vers un autre abonnement. Consultez [Limitations relatives au déplacement réseau](./move-limitations/networking-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | redis | Oui | Oui |
> | redisenterprise | Non | Non |

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | appliedreservations | Non | Non |
> | calculateexchange | Non | Non |
> | calculateprice | Non | Non |
> | calculatepurchaseprice | Non | Non |
> | catalogs | Non | Non |
> | commercialreservationorders | Non | Non |
> | change | Non | Non |
> | reservationorders | Non | Non |
> | reservations | Non | Non |
> | les ressources | Non | Non |
> | validatereservationorder | Non | Non |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | cdnwebapplicationfirewallmanagedrulesets | Non | Non |
> | cdnwebapplicationfirewallpolicies | Oui | Oui |
> | edgenodes | Non | Non |
> | profiles | Oui | Oui |
> | profiles / endpoints | Oui | Oui |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!IMPORTANT]
> Consultez les [Conseils de déplacement d’App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | certificateorders | Oui | Oui |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!IMPORTANT]
> Consultez les [Conseils de déplacement de déploiement classique](./move-limitations/classic-model-move-limitations.md). Les ressources de déploiement classiques peuvent être déplacées entre abonnements avec une opération spécifique à ce scénario.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | capabilities | Non | Non |
> | domainnames | Oui | Non |
> | quotas | Non | Non |
> | resourcetypes | Non | Non |
> | validatesubscriptionmoveavailability | Non | Non |
> | virtualmachines | Oui | Oui |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!IMPORTANT]
> Consultez les [Conseils de déplacement de déploiement classique](./move-limitations/classic-model-move-limitations.md). Les ressources de déploiement classiques peuvent être déplacées entre abonnements avec une opération spécifique à ce scénario.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | classicinfrastructureresources | Non | Non |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!IMPORTANT]
> Consultez les [Conseils de déplacement de déploiement classique](./move-limitations/classic-model-move-limitations.md). Les ressources de déploiement classiques peuvent être déplacées entre abonnements avec une opération spécifique à ce scénario.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | capabilities | Non | Non |
> | expressroutecrossconnections | Non | Non |
> | expressroutecrossconnections / peerings | Non | Non |
> | gatewaysupporteddevices | Non | Non |
> | networksecuritygroups | Non | Non |
> | quotas | Non | Non |
> | reservedips | Non | Non |
> | virtualnetworks | Non | Non |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!IMPORTANT]
> Consultez les [Conseils de déplacement de déploiement classique](./move-limitations/classic-model-move-limitations.md). Les ressources de déploiement classiques peuvent être déplacées entre abonnements avec une opération spécifique à ce scénario.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | disks | Non | Non |
> | images | Non | Non |
> | osimages | Non | Non |
> | osplatformimages | Non | Non |
> | publicimages | Non | Non |
> | quotas | Non | Non |
> | storageaccounts | Oui | Non |
> | vmimages | Non | Non |

## <a name="microsoftclassicsubscription"></a>Microsoft.ClassicSubscription

> [!IMPORTANT]
> Consultez les [Conseils de déplacement de déploiement classique](./move-limitations/classic-model-move-limitations.md). Les ressources de déploiement classiques peuvent être déplacées entre abonnements avec une opération spécifique à ce scénario.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | opérations | Non | Non |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Oui | Oui |

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | ratecard | Non | Non |
> | usageaggregates | Non | Non |

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!IMPORTANT]
> Consultez [Conseils pour le déplacement de machines virtuelles](./move-limitations/virtual-machines-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | availabilitysets | Oui | Oui |
> | diskaccesses | Non | Non |
> | diskencryptionsets | Non | Non |
> | disks | Oui | Oui |
> | galleries | Non | Non |
> | galleries / images | Non | Non |
> | galleries / images / versions | Non | Non |
> | hostgroups | Non | Non |
> | hostgroups / hosts | Non | Non |
> | images | Oui | Oui |
> | proximityplacementgroups | Oui | Oui |
> | restorepointcollections | Non | Non |
> | restorepointcollections / restorepoints | Non | Non |
> | sharedvmextensions | Non | Non |
> | sharedvmimages | Non | Non |
> | sharedvmimages / versions | Non | Non |
> | snapshots | Oui | Oui |
> | sshpublickeys | Non | Non |
> | virtualmachines | Oui | Oui |
> | virtualmachines / extensions | Oui | Oui |
> | virtualmachinescalesets | Oui | Oui |

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | aggregatedcost | Non | Non |
> | balances | Non | Non |
> | budgets | Non | Non |
> | charges | Non | Non |
> | costtags | Non | Non |
> | credits | Non | Non |
> | événements | Non | Non |
> | forecasts | Non | Non |
> | lots | Non | Non |
> | marketplaces | Non | Non |
> | pricesheets | Non | Non |
> | products | Non | Non |
> | reservationdetails | Non | Non |
> | reservationrecommendationdetails | Non | Non |
> | reservationrecommendations | Non | Non |
> | reservationsummaries | Non | Non |
> | reservationtransactions | Non | Non |
> | tags | Non | Non |
> | tenants | Non | Non |
> | terms | Non | Non |
> | usagedetails | Non | Non |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | containergroups | Non | Non |
> | serviceassociationlinks | Non | Non |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | registries | Oui | Oui |
> | registries / agentpools | Oui | Oui |
> | registries / buildtasks | Oui | Oui |
> | registries / replications | Oui | Oui |
> | registries / tasks | Oui | Oui |
> | registries / webhooks | Oui | Oui |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | containerservices | Non | Non |
> | managedclusters | Non | Non |
> | openshiftmanagedclusters | Non | Non |

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | applications | Non | Non |

## <a name="microsoftcortanaanalytics"></a>Microsoft.CortanaAnalytics

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Non | Non |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | alertes | Non | Non |
> | billingaccounts | Non | Non |
> | budgets | Non | Non |
> | cloudconnectors | Non | Non |
> | connecteurs | Oui | Oui |
> | departments | Non | Non |
> | dimensions | Non | Non |
> | enrollmentaccounts | Non | Non |
> | exports | Non | Non |
> | externalbillingaccounts | Non | Non |
> | forecast | Non | Non |
> | query | Non | Non |
> | inscription | Non | Non |
> | reportconfigs | Non | Non |
> | reports | Non | Non |
> | paramètres | Non | Non |
> | showbackrules | Non | Non |
> | views | Non | Non |

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | hubs | Non | Non |

## <a name="microsoftcustomerlockbox"></a>Microsoft.CustomerLockbox

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | requêtes | Non | Non |

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | associations | Non | Non |
> | resourceproviders | Oui | Oui |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | jobs | Non | Non |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | availableskus | Non | Non |
> | databoxedgedevices | Oui | Oui |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | workspaces | Non | Non |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | catalogs | Oui | Oui |
> | datacatalogs | Non | Non |

## <a name="microsoftdataconnect"></a>Microsoft.DataConnect

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | connectionmanagers | Non | Non |

## <a name="microsoftdataexchange"></a>Microsoft.DataExchange

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | packages | Non | Non |
> | plans | Non | Non |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | datafactories | Oui | Oui |
> | factories | Oui | Oui |

## <a name="microsoftdatalake"></a>Microsoft.DataLake

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | datalakeaccounts | Non | Non |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Oui | Oui |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Oui | Oui |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | services | Non | Non |
> | services / projects | Non | Non |
> | slots | Non | Non |

## <a name="microsoftdataprotection"></a>Microsoft.DataProtection

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | backupvaults | Non | Non |

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Oui | Oui |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | servers | Oui | Oui |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | flexibleServers | Oui | Oui |
> | servers | Oui | Oui |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | flexibleServers | Oui | Oui |
> | servergroups | Non | Non |
> | servers | Oui | Oui |
> | serversv2 | Oui | Oui |
> | singleservers | Oui | Oui |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | artifactsources | Oui | Oui |
> | rollouts | Oui | Oui |
> | servicetopologies | Oui | Oui |
> | servicetopologies / services | Oui | Oui |
> | servicetopologies / services / serviceunits | Oui | Oui |
> | steps | Oui | Oui |

## <a name="microsoftdesktopvirtualization"></a>Microsoft.DesktopVirtualization

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | applicationgroups | Oui | Oui |
> | hostpools | Oui | Oui |
> | workspaces | Oui | Oui |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | elasticpools | Non | Non |
> | elasticpools / iothubtenants | Non | Non |
> | iothubs | Oui | Oui |
> | provisioningservices | Oui | Oui |

## <a name="microsoftdevops"></a>Microsoft.DevOps

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | pipelines | Oui | Oui |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | controllers | Oui | Oui |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | labcenters | Non | Non |
> | labs | Oui | Non |
> | labs / environments | Oui | Oui |
> | labs / servicerunners | Oui | Oui |
> | labs / virtualmachines | Oui | Non |
> | schedules | Oui | Oui |

## <a name="microsoftdigitaltwins"></a>Microsoft.DigitalTwins

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | digitaltwinsinstances | Non | Non |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | databaseaccountnames | Non | Non |
> | databaseaccounts | Oui | Oui |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | domaines | Oui | Oui |
> | generatessorequest | Non | Non |
> | topleveldomains | Non | Non |
> | validatedomainregistrationinformation | Non | Non |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | services | Oui | Oui |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | domaines | Oui | Oui |
> | eventsubscriptions | Non (non déplaçable indépendamment, mais déplacé automatiquement avec la ressource abonnée) | Non (non déplaçable indépendamment, mais déplacé automatiquement avec la ressource abonnée) |
> | extensiontopics | Non | Non |
> | partnernamespaces | Oui | Oui |
> | partnerregistrations | Non | Non |
> | partnertopics | Oui | Oui |
> | systemtopics | Oui | Oui |
> | topics | Oui | Oui |
> | topictypes | Non | Non |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | clusters | Oui | Oui |
> | espaces de noms | Oui | Oui |
> | sku | Non | Non |

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | experimentworkspaces | Non | Non |

## <a name="microsoftfalcon"></a>Microsoft.Falcon

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | espaces de noms | Oui | Oui |

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | featureproviders | Non | Non |
> | features | Non | Non |
> | fournisseurs | Non | Non |
> | subscriptionfeatureregistrations | Non | Non |

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Non | Non |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | automanagedaccounts | Non | Non |
> | automanagedvmconfigurationprofiles | Non | Non |
> | guestconfigurationassignments | Non | Non |
> | software | Non | Non |
> | softwareupdateprofile | Non | Non |
> | softwareupdates | Non | Non |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | hanainstances | Non | Non |
> | sapmonitors | Non | Non |

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | dedicatedhsms | Non | Non |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!IMPORTANT]
> Vous pouvez déplacer des clusters HDInsight vers un nouvel abonnement ou groupe de ressources. Toutefois, vous ne pouvez pas déplacer sur différents abonnements les ressources réseau liées au cluster HDInsight (par exemple le réseau virtuel, une carte réseau ou un équilibrage de charge). En outre, vous ne pouvez pas déplacer vers un nouveau groupe de ressources une carte réseau connectée à une machine virtuelle pour le cluster.
>
> Lorsque vous déplacez un cluster HDInsight vers un nouvel abonnement, déplacez tout d’abord les autres ressources (le compte de stockage, par exemple). Puis, déplacez le cluster HDInsight par lui-même.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | clusters | Oui | Oui |

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | services | Oui | Oui |

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | machines | Oui | Oui |
> | machines / extensions | Oui | Oui |

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | datamanagers | Oui | Oui |

## <a name="microsofthybridnetwork"></a>Microsoft.HybridNetwork

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | périphériques | Non | Non |
> | vnfs | Non | Non |

## <a name="microsofthydra"></a>Microsoft.Hydra

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | components | Non | Non |
> | networkscopes | Non | Non |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | jobs | Oui | Oui |

## <a name="microsoftinsights"></a>microsoft.insights

> [!IMPORTANT]
> Vérifiez que le déplacement vers le nouvel abonnement ne dépasse pas les [quotas d’abonnement](azure-subscription-service-limits.md#azure-monitor-limits).

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | actiongroups | Oui | Oui |
> | activitylogalerts | Non | Non |
> | alertrules | Oui | Oui |
> | autoscalesettings | Oui | Oui |
> | baseline | Non | Non |
> | components | Oui | Oui |
> | datacollectionrules | Non | Non |
> | diagnosticsettings | Non | Non |
> | diagnosticsettingscategories | Non | Non |
> | eventcategories | Non | Non |
> | eventtypes | Non | Non |
> | extendeddiagnosticsettings | Non | Non |
> | guestdiagnosticsettings | Non | Non |
> | listmigrationdate | Non | Non |
> | logdefinitions | Non | Non |
> | logprofiles | Non | Non |
> | logs | Non | Non |
> | metricalerts | Non | Non |
> | metricbaselines | Non | Non |
> | metricbatch | Non | Non |
> | metricdefinitions | Non | Non |
> | metricnamespaces | Non | Non |
> | Mesures | Non | Non |
> | migratealertrules | Non | Non |
> | migratetonewpricingmodel | Non | Non |
> | myworkbooks | Non | Non |
> | notificationgroups | Non | Non |
> | privatelinkscopes | Non | Non |
> | rollbacktolegacypricingmodel | Non | Non |
> | scheduledqueryrules | Oui | Oui |
> | Topologie | Non | Non |
> | transactions | Non | Non |
> | vminsightsonboardingstatuses | Non | Non |
> | webtests | Oui | Oui |
> | webtests / gettestresultfile | Non | Non |
> | workbooks | Oui | Oui |
> | workbooktemplates | Oui | Oui |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | apptemplates | Non | Non |
> | iotapps | Oui | Oui |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | graph | Oui | Oui |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!IMPORTANT]
> Les coffres de clés utilisés pour le chiffrement de disques ne peuvent pas être déplacés vers des groupes de ressources situés dans le même abonnement ni d’un abonnement à l’autre.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | deletedvaults | Non | Non |
> | hsmpools | Non | Non |
> | managedhsms | Non | Non |
> | vaults | Oui | Oui |

## <a name="microsoftkubernetes"></a>Microsoft.Kubernetes

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | connectedclusters | Oui | Oui |
> | registeredsubscriptions | Non | Non |

## <a name="microsoftkubernetesconfiguration"></a>Microsoft.KubernetesConfiguration

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | sourcecontrolconfigurations | Non | Non |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | clusters | Oui | Oui |

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | labaccounts | Non | Non |
> | users | Non | Non |

## <a name="microsoftlocationbasedservices"></a>Microsoft.LocationBasedServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Non | Non |

## <a name="microsoftlocationservices"></a>Microsoft.LocationServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Non | Non |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | hostingenvironments | Non | Non |
> | integrationaccounts | Oui | Oui |
> | integrationserviceenvironments | Oui | Non |
> | integrationserviceenvironments / managedapis | Oui | Non |
> | isolatedenvironments | Non | Non |
> | workflows | Oui | Oui |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | commitmentplans | Non | Non |
> | webservices | Oui | Non |
> | workspaces | Oui | Oui |

## <a name="microsoftmachinelearningcompute"></a>Microsoft.MachineLearningCompute

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | operationalizationclusters | Non | Non |

## <a name="microsoftmachinelearningexperimentation"></a>Microsoft.MachineLearningExperimentation

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Non | Non |
> | teamaccounts | Non | Non |

## <a name="microsoftmachinelearningmodelmanagement"></a>Microsoft.MachineLearningModelManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Non | Non |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | workspaces | Non | Non |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | configurationassignments | Non | Non |
> | maintenanceconfigurations | Oui | Oui |
> | updates | Non | Non |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | Identités | Non | Non |
> | userassignedidentities | Non | Non |

## <a name="microsoftmanagednetwork"></a>Microsoft.ManagedNetwork

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | managednetworks | Non | Non |
> | managednetworks / managednetworkgroups | Non | Non |
> | managednetworks / managednetworkpeeringpolicies | Non | Non |
> | notification | Non | Non |

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | marketplaceregistrationdefinitions | Non | Non |
> | registrationassignments | Non | Non |
> | registrationdefinitions | Non | Non |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | getentities | Non | Non |
> | managementgroups | Non | Non |
> | managementgroups / settings | Non | Non |
> | les ressources | Non | Non |
> | starttenantbackfill | Non | Non |
> | tenantbackfillstatus | Non | Non |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Oui | Oui |
> | accounts / privateatlases | Oui | Oui |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | offers | Non | Non |
> | offertypes | Non | Non |
> | privategalleryitems | Non | Non |
> | privatestoreclient | Non | Non |
> | privatestores | Non | Non |
> | products | Non | Non |
> | publishers | Non | Non |
> | inscription | Non | Non |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | classicdevservices | Non | Non |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | agreements | Non | Non |
> | offertypes | Non | Non |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | mediaservices | Oui | Oui |
> | mediaservices / liveevents | Oui | Oui |
> | mediaservices / streamingendpoints | Oui | Oui |

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | appclusters | Non | Non |

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | assessmentprojects | Non | Non |
> | migrateprojects | Non | Non |
> | movecollections | Non | Non |
> | projects | Non | Non |

## <a name="microsoftmixedreality"></a>Microsoft.MixedReality

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | holographicsbroadcastaccounts | Non | Non |
> | objectunderstandingaccounts | Non | Non |
> | remoterenderingaccounts | Oui | Oui |
> | spatialanchorsaccounts | Oui | Oui |

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | netappaccounts | Non | Non |
> | netappaccounts / capacitypools | Non | Non |
> | netappaccounts / capacitypools / volumes | Non | Non |

## <a name="microsoftnetwork"></a>Microsoft.Network

> [!IMPORTANT]
> Consultez l’[Aide pour le déplacement du réseau](./move-limitations/networking-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | applicationgateways | Non | Non |
> | applicationgatewaywebapplicationfirewallpolicies | Non | Non |
> | applicationsecuritygroups | Oui | Oui |
> | azurefirewalls | Non | Non |
> | bastionhosts | Non | Non |
> | bgpservicecommunities | Non | Non |
> | connections | Oui | Oui |
> | ddoscustompolicies | Oui | Oui |
> | ddosprotectionplans | Non | Non |
> | dnszones | Oui | Oui |
> | expressroutecircuits | Non | Non |
> | expressroutegateways | Non | Non |
> | expressrouteserviceproviders | Non | Non |
> | firewallpolicies | Oui | Oui |
> | frontdoors | Non | Non |
> | ipallocations | Oui | Oui |
> | ipgroups | Oui | Oui |
> | loadbalancers | Oui - Référence SKU de base<br> Oui - Référence SKU Standard | Oui - Référence SKU de base<br>Non - Référence SKU Standard |
> | localnetworkgateways | Oui | Oui |
> | natgateways | Non | Non |
> | networkexperimentprofiles | Non | Non |
> | networkintentpolicies | Oui | Oui |
> | networkinterfaces | Oui | Oui |
> | networkprofiles | Non | Non |
> | networksecuritygroups | Oui | Oui |
> | networkwatchers | Non | Non |
> | networkwatchers / connectionmonitors | Oui | Non |
> | networkwatchers / flowlogs | Oui | Non |
> | networkwatchers / pingmeshes | Oui | Non |
> | p2svpngateways | Non | Non |
> | privatednszones | Oui | Oui |
> | privatednszones / virtualnetworklinks | Oui | Oui |
> | privatednszonesinternal | Non | Non |
> | privateendpointredirectmaps | Non | Non |
> | privateendpoints | Non | Non |
> | privatelinkservices | Non | Non |
> | publicipaddresses | Oui - Référence SKU de base<br>Oui - Référence SKU Standard | Oui - Référence SKU de base<br>Non - Référence SKU Standard |
> | publicipprefixes | Oui | Oui |
> | routefilters | Non | Non |
> | routetables | Oui | Oui |
> | securitypartnerproviders | Oui | Oui |
> | serviceendpointpolicies | Oui | Oui |
> | trafficmanagergeographichierarchies | Non | Non |
> | trafficmanagerprofiles | Oui | Oui |
> | trafficmanagerprofiles / heatmaps | Non | Non |
> | trafficmanagerusermetricskeys | Non | Non |
> | virtualhubs | Non | Non |
> | virtualnetworkgateways | Oui | Oui |
> | virtualnetworks | Oui | Oui |
> | virtualnetworktaps | Non | Non |
> | virtualrouters | Oui | Oui |
> | virtualwans | Non | Non |
> | vpngateways (Virtual WAN) | Non | Non |
> | vpnserverconfigurations | Non | Non |
> | vpnsites (Virtual WAN) | Non | Non |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | espaces de noms | Oui | Oui |
> | namespaces / notificationhubs | Oui | Oui |

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | osnamespaces | Oui | Oui |

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | hypervsites | Non | Non |
> | importsites | Non | Non |
> | serversites | Non | Non |
> | vmwaresites | Non | Non |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!IMPORTANT]
> Vérifiez que le déplacement vers un nouvel abonnement ne dépasse pas les [quotas d’abonnement](azure-subscription-service-limits.md#azure-monitor-limits).
>
> Les espaces de travail qui ont un compte Automation lié ne peuvent pas être déplacés. Avant de commencer une opération de déplacement, veillez à dissocier tous les comptes Automation.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | clusters | Non | Non |
> | deletedworkspaces | Non | Non |
> | linktargets | Non | Non |
> | storageinsightconfigs | Non | Non |
> | workspaces | Oui | Oui |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | managementassociations | Non | Non |
> | managementconfigurations | Oui | Oui |
> | solutions | Oui | Oui |
> | views | Oui | Oui |

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | legacypeerings | Non | Non |
> | peerasns | Non | Non |
> | peeringlocations | Non | Non |
> | peerings | Non | Non |
> | peeringservicecountries | Non | Non |
> | peeringservicelocations | Non | Non |
> | peeringserviceproviders | Non | Non |
> | peeringservices | Non | Non |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | policyevents | Non | Non |
> | policystates | Non | Non |
> | policytrackedresources | Non | Non |
> | remediations | Non | Non |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | consoles | Non | Non |
> | dashboards | Oui | Oui |
> | usersettings | Non | Non |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | workspacecollections | Oui | Oui |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | capacities | Oui | Oui |

## <a name="microsoftprojectbabylon"></a>Microsoft.ProjectBabylon

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Non | Non |

## <a name="microsoftproviderhub"></a>Microsoft.ProviderHub

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | availableaccounts | Non | Non |
> | providerregistrations | Non | Non |
> | rollouts | Non | Non |

## <a name="microsoftquantum"></a>Microsoft.Quantum

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | workspaces | Non | Non |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!IMPORTANT]
> Voir [Conseils sur le déplacement de Recovery Services](../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | replicationeligibilityresults | Non | Non |
> | vaults | Oui | Oui |

## <a name="microsoftredhatopenshift"></a>Microsoft.RedHatOpenShift

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | openshiftclusters | Non | Non |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | espaces de noms | Oui | Oui |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | queries | Oui | Oui |
> | resourcechangedetails | Non | Non |
> | resourcechanges | Non | Non |
> | les ressources | Non | Non |
> | resourceshistory | Non | Non |
> | subscriptionsstatus | Non | Non |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | childresources | Non | Non |
> | emergingissues | Non | Non |
> | événements | Non | Non |
> | metadata | Non | Non |
> | Notifications | Non | Non |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | deployments | Non | Non |
> | deploymentscripts | Non | Non |
> | deploymentscripts / logs | Non | Non |
> | liens | Non | Non |
> | fournisseurs | Non | Non |
> | resourcegroups | Non | Non |
> | les ressources | Non | Non |
> | subscriptions | Non | Non |
> | tags | Non | Non |
> | templatespecs | Non | Non |
> | templatespecs / versions | Non | Non |
> | tenants | Non | Non |

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | applications | Oui | Non |
> | saasresources | Non | Non |

## <a name="microsoftsearch"></a>Microsoft.Search

> [!IMPORTANT]
> Vous ne pouvez pas déplacer simultanément plusieurs ressources de recherche dans des régions différentes. Déplacez-les plutôt dans des opérations distinctes.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | resourcehealthmetadata | Non | Non |
> | searchservices | Oui | Oui |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | adaptivenetworkhardenings | Non | Non |
> | advancedthreatprotectionsettings | Non | Non |
> | alertes | Non | Non |
> | allowedconnections | Non | Non |
> | applicationwhitelistings | Non | Non |
> | assessmentmetadata | Non | Non |
> | assessments | Non | Non |
> | autodismissalertsrules | Non | Non |
> | automations | Oui | Oui |
> | autoprovisioningsettings | Non | Non |
> | complianceresults | Non | Non |
> | compliances | Non | Non |
> | datacollectionagents | Non | Non |
> | devicesecuritygroups | Non | Non |
> | discoveredsecuritysolutions | Non | Non |
> | externalsecuritysolutions | Non | Non |
> | informationprotectionpolicies | Non | Non |
> | iotsecuritysolutions | Oui | Oui |
> | iotsecuritysolutions / analyticsmodels | Non | Non |
> | iotsecuritysolutions / analyticsmodels / aggregatedalerts | Non | Non |
> | iotsecuritysolutions / analyticsmodels / aggregatedrecommendations | Non | Non |
> | jitnetworkaccesspolicies | Non | Non |
> | stratégies | Non | Non |
> | pricings | Non | Non |
> | regulatorycompliancestandards | Non | Non |
> | regulatorycompliancestandards / regulatorycompliancecontrols | Non | Non |
> | regulatorycompliancestandards / regulatorycompliancecontrols / regulatorycomplianceassessments | Non | Non |
> | securitycontacts | Non | Non |
> | securitysolutions | Non | Non |
> | securitysolutionsreferencedata | Non | Non |
> | securitystatuses | Non | Non |
> | securitystatusessummaries | Non | Non |
> | servervulnerabilityassessments | Non | Non |
> | paramètres | Non | Non |
> | subassessments | Non | Non |
> | tâches | Non | Non |
> | topologies | Non | Non |
> | workspacesettings | Non | Non |

## <a name="microsoftsecurityinsights"></a>Microsoft.SecurityInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | aggregations | Non | Non |
> | alertrules | Non | Non |
> | alertruletemplates | Non | Non |
> | automationrules | Non | Non |
> | bookmarks | Non | Non |
> | cas | Non | Non |
> | dataconnectors | Non | Non |
> | entities | Non | Non |
> | entityqueries | Non | Non |
> | incidents | Non | Non |
> | officeconsents | Non | Non |
> | paramètres | Non | Non |
> | threatintelligence | Non | Non |

## <a name="microsoftserialconsole"></a>Microsoft.SerialConsole

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | consoleservices | Non | Non |

## <a name="microsoftservermanagement"></a>Microsoft.ServerManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | gateways | Non | Non |
> | nœuds | Non | Non |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | espaces de noms | Oui | Oui |
> | premiummessagingregions | Non | Non |
> | sku | Non | Non |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | applications | Non | Non |
> | clusters | Oui | Oui |
> | containergroups | Non | Non |
> | containergroupsets | Non | Non |
> | edgeclusters | Non | Non |
> | managedclusters | Non | Non |
> | networks | Non | Non |
> | secretstores | Non | Non |
> | volumes | Non | Non |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | applications | Oui | Oui |
> | containergroups | Non | Non |
> | gateways | Oui | Oui |
> | networks | Oui | Oui |
> | secrets | Oui | Oui |
> | volumes | Oui | Oui |

## <a name="microsoftservices"></a>Microsoft.Services

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | rollouts | Non | Non |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | signalr | Oui | Oui |

## <a name="microsoftsoftwareplan"></a>Microsoft.SoftwarePlan

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | hybridusebenefits | Non | Non |

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | applicationdefinitions | Non | Non |
> | applications | Non | Non |
> | jitrequests | Non | Non |

## <a name="microsoftsql"></a>Microsoft.Sql

> [!IMPORTANT]
> La base de données et le serveur doivent résider dans le même groupe de ressources. Lorsque vous déplacez un serveur SQL, toutes ses bases de données sont également déplacées. Ce comportement s’applique aux bases de données Azure SQL Database et Azure Synapse Analytics.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | instancepools | Non | Non |
> | locations | Oui | Oui |
> | managedinstances | Non | Non |
> | servers | Oui | Oui |
> | servers / databases | Oui | Oui |
> | servers / databases / backuplongtermretentionpolicies | Oui | Oui |
> | servers / elasticpools | Oui | Oui |
> | servers / jobaccounts | Oui | Oui |
> | servers / jobagents | Oui | Oui |
> | virtualclusters | Oui | Oui |

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | sqlvirtualmachinegroups | Oui | Oui |
> | sqlvirtualmachines | Oui | Oui |

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | storageaccounts | Oui | Oui |

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | caches | Non | Non |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | storagesyncservices | Oui | Oui |

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | storagesyncservices | Non | Non |

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | storagesyncservices | Non | Non |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | managers | Non | Non |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!IMPORTANT]
> Les tâches Stream Analytics ne peuvent pas être déplacées lorsqu’elles sont en cours d’exécution.

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | clusters | Non | Non |
> | streamingjobs | Oui | Oui |

## <a name="microsoftstreamanalyticsexplorer"></a>Microsoft.StreamAnalyticsExplorer

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | environments | Non | Non |
> | instances | Non | Non |

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | subscriptions | Non | Non |

## <a name="microsoftsupport"></a>microsoft.support

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | services | Non | Non |
> | supporttickets | Non | Non |

## <a name="microsoftsynapse"></a>Microsoft.Synapse

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | workspaces | Oui | Oui |
> | workspaces / bigdatapools | Oui | Oui |
> | workspaces / sqlpools | Oui | Oui |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | environments | Oui | Oui |
> | environments / eventsources | Oui | Oui |
> | environments / referencedatasets | Oui | Oui |

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | stores | Oui | Oui |

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | imagetemplates | Non | Non |

## <a name="microsoftvisualstudio"></a>microsoft.visualstudio

> [!IMPORTANT]
> Pour modifier l’abonnement pour Azure DevOps, consultez [Modifier l’abonnement Azure utilisé pour la facturation](/azure/devops/organizations/billing/change-azure-subscription?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | account | Non | Non |
> | account / extension | Non | Non |
> | account / project | Non | Non |

## <a name="microsoftvmware"></a>Microsoft.VMware

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | arczones | Non | Non |
> | resourcepools | Non | Non |
> | vcenters | Non | Non |
> | virtualmachines | Non | Non |
> | virtualmachinetemplates | Non | Non |
> | virtualnetworks | Non | Non |

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | dedicatedcloudnodes | Non | Non |
> | dedicatedcloudservices | Non | Non |
> | virtualmachines | Non | Non |

## <a name="microsoftvnfmanager"></a>Microsoft.VnfManager

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | périphériques | Non | Non |
> | vnfs | Non | Non |

## <a name="microsoftvsonline"></a>Microsoft.VSOnline

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | accounts | Non | Non |
> | plans | Non | Non |
> | registeredsubscriptions | Non | Non |

## <a name="microsoftweb"></a>Microsoft.Web

> [!IMPORTANT]
> Consultez les [Conseils de déplacement d’App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | availablestacks | Non | Non |
> | billingmeters | Non | Non |
> | certificates | Non | Oui |
> | connectiongateways | Oui | Oui |
> | connections | Oui | Oui |
> | customapis | Oui | Oui |
> | deletedsites | Non | Non |
> | deploymentlocations | Non | Non |
> | georegions | Non | Non |
> | hostingenvironments | Non | Non |
> | kubeenvironments | Oui | Oui |
> | publishingusers | Non | Non |
> | de films | Non | Non |
> | resourcehealthmetadata | Non | Non |
> | runtimes | Non | Non |
> | serverfarms | Oui | Oui |
> | serverfarms / eventgridfilters | Non | Non |
> | sites | Oui | Oui |
> | sites / premieraddons | Oui | Oui |
> | sites / slots | Oui | Oui |
> | sourcecontrols | Non | Non |
> | staticsites | Non | Non |

## <a name="microsoftwindowsesu"></a>Microsoft.WindowsESU

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | multipleactivationkeys | Non | Non |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | deviceservices | Non | Non |

## <a name="microsoftworkloadbuilder"></a>Microsoft.WorkloadBuilder

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | workloads | Non | Non |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tableFixed"]
> | Type de ressource | Resource group | Abonnement |
> | ------------- | ----------- | ---------- |
> | components | Non | Non |
> | componentssummary | Non | Non |
> | monitorinstances | Non | Non |
> | monitorinstancessummary | Non | Non |
> | monitors | Non | Non |

## <a name="third-party-services"></a>Services tiers

Actuellement, les services tiers ne prennent pas en charge l’opération de déplacement.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir les commandes permettant de déplacer des ressources, consultez [Déplacer des ressources vers un nouveau groupe de ressources ou un nouvel abonnement](move-resource-group-and-subscription.md).

Pour obtenir les mêmes données qu'un fichier de valeurs séparées par des virgules, téléchargez [move-support-resources.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/move-support-resources.csv).
