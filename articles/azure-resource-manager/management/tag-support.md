---
title: Prise en charge des étiquettes pour les ressources
description: Indique les types de ressources Azure qui prennent en charge les étiquettes. Fournit des détails pour tous les services Azure.
ms.topic: conceptual
ms.date: 10/21/2020
ms.openlocfilehash: fb81ed303cf27add3a033021f017c03f4b9e9750
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005722"
---
# <a name="tag-support-for-azure-resources"></a>Prise en charge des étiquettes pour les ressources Azure
Cet article indique si un type de ressource prend en charge les [étiquettes](tag-resources.md). La colonne intitulée **Prend en charge les balises** indique si le type de ressource a une propriété pour la balise. La colonne intitulée **Balise dans le rapport des coûts** indique si ce type de ressource transmet la balise au rapport des coûts. Vous pouvez afficher les coûts à l’aide d’étiquettes dans l’[analyse Azure Cost Management](../../cost-management-billing/costs/group-filter.md) et les [données de facturation et d’utilisation quotidienne Azure](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md).

Pour obtenir les mêmes données qu’un fichier de valeurs séparées par des virgules, téléchargez [tag-tag-support.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/tag-support.csv).

Accédez à un espace de noms du fournisseur de ressources :
> [!div class="op_single_selector"]
> - [Microsoft.AAD](#microsoftaad)
> - [Microsoft.Addons](#microsoftaddons)
> - [Microsoft.ADHybridHealthService](#microsoftadhybridhealthservice)
> - [Microsoft.Advisor](#microsoftadvisor)
> - [Microsoft.AgFoodPlatform](#microsoftagfoodplatform)
> - [Microsoft.AlertsManagement](#microsoftalertsmanagement)
> - [Microsoft.AnalysisServices](#microsoftanalysisservices)
> - [Microsoft.ApiManagement](#microsoftapimanagement)
> - [Microsoft.AppConfiguration](#microsoftappconfiguration)
> - [Microsoft.AppPlatform](#microsoftappplatform)
> - [Microsoft.Attestation](#microsoftattestation)
> - [Microsoft.Authorization](#microsoftauthorization)
> - [Microsoft.Automanage](#microsoftautomanage)
> - [Microsoft.Automation](#microsoftautomation)
> - [Microsoft.AVS](#microsoftavs)
> - [Microsoft.Azure.Geneva](#microsoftazuregeneva)
> - [Microsoft.AzureActiveDirectory](#microsoftazureactivedirectory)
> - [Microsoft.AzureData](#microsoftazuredata)
> - [Microsoft.AzureStack](#microsoftazurestack)
> - [Microsoft.AzureStackHCI](#microsoftazurestackhci)
> - [Microsoft.BareMetalInfrastructure](#microsoftbaremetalinfrastructure)
> - [Microsoft.Batch](#microsoftbatch)
> - [Microsoft.Billing](#microsoftbilling)
> - [Microsoft.BingMaps](#microsoftbingmaps)
> - [Microsoft.Blockchain](#microsoftblockchain)
> - [Microsoft.BlockchainTokens](#microsoftblockchaintokens)
> - [Microsoft.Blueprint](#microsoftblueprint)
> - [Microsoft.BotService](#microsoftbotservice)
> - [Microsoft.Cache](#microsoftcache)
> - [Microsoft.Capacity](#microsoftcapacity)
> - [Microsoft.Cdn](#microsoftcdn)
> - [Microsoft.CertificateRegistration](#microsoftcertificateregistration)
> - [Microsoft.ChangeAnalysis](#microsoftchangeanalysis)
> - [Microsoft.ClassicCompute](#microsoftclassiccompute)
> - [Microsoft.ClassicInfrastructureMigrate](#microsoftclassicinfrastructuremigrate)
> - [Microsoft.ClassicNetwork](#microsoftclassicnetwork)
> - [Microsoft.ClassicStorage](#microsoftclassicstorage)
> - [Microsoft.Codespaces](#microsoftcodespaces)
> - [Microsoft.CognitiveServices](#microsoftcognitiveservices)
> - [Microsoft.Commerce](#microsoftcommerce)
> - [Microsoft.Compute](#microsoftcompute)
> - [Microsoft.ConnectedCache](#microsoftconnectedcache)
> - [Microsoft.Consumption](#microsoftconsumption)
> - [Microsoft.ContainerInstance](#microsoftcontainerinstance)
> - [Microsoft.ContainerRegistry](#microsoftcontainerregistry)
> - [Microsoft.ContainerService](#microsoftcontainerservice)
> - [Microsoft.CostManagement](#microsoftcostmanagement)
> - [Microsoft.CustomerLockbox](#microsoftcustomerlockbox)
> - [Microsoft.CustomProviders](#microsoftcustomproviders)
> - [Microsoft.D365CustomerInsights](#microsoftd365customerinsights)
> - [Microsoft.DataBox](#microsoftdatabox)
> - [Microsoft.DataBoxEdge](#microsoftdataboxedge)
> - [Microsoft.Databricks](#microsoftdatabricks)
> - [Microsoft.DataCatalog](#microsoftdatacatalog)
> - [Microsoft.DataFactory](#microsoftdatafactory)
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
> - [Microsoft.DeviceUpdate](#microsoftdeviceupdate)
> - [Microsoft.DevOps](#microsoftdevops)
> - [Microsoft.DevSpaces](#microsoftdevspaces)
> - [Microsoft.DevTestLab](#microsoftdevtestlab)
> - [Microsoft.DigitalTwins](#microsoftdigitaltwins)
> - [Microsoft.DocumentDB](#microsoftdocumentdb)
> - [Microsoft.DomainRegistration](#microsoftdomainregistration)
> - [Microsoft.DynamicsLcs](#microsoftdynamicslcs)
> - [Microsoft.EnterpriseKnowledgeGraph](#microsoftenterpriseknowledgegraph)
> - [Microsoft.EventGrid](#microsofteventgrid)
> - [Microsoft.EventHub](#microsofteventhub)
> - [Microsoft.Experimentation](#microsoftexperimentation)
> - [Microsoft.Falcon](#microsoftfalcon)
> - [Microsoft.Features](#microsoftfeatures)
> - [Microsoft.Gallery](#microsoftgallery)
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
> - [Microsoft.Insights](#microsoftinsights)
> - [Microsoft.Intune](#microsoftintune)
> - [Microsoft.IoTCentral](#microsoftiotcentral)
> - [Microsoft.IoTSpaces](#microsoftiotspaces)
> - [Microsoft.KeyVault](#microsoftkeyvault)
> - [Microsoft.Kubernetes](#microsoftkubernetes)
> - [Microsoft.KubernetesConfiguration](#microsoftkubernetesconfiguration)
> - [Microsoft.Kusto](#microsoftkusto)
> - [Microsoft.LabServices](#microsoftlabservices)
> - [Microsoft.Logic](#microsoftlogic)
> - [Microsoft.MachineLearning](#microsoftmachinelearning)
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
> - [Microsoft.Notebooks](#microsoftnotebooks)
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
> - [Microsoft.ScVmm](#microsoftscvmm)
> - [Microsoft.Search](#microsoftsearch)
> - [Microsoft.Security](#microsoftsecurity)
> - [Microsoft.SecurityGraph](#microsoftsecuritygraph)
> - [Microsoft.SecurityInsights](#microsoftsecurityinsights)
> - [Microsoft.SerialConsole](#microsoftserialconsole)
> - [Microsoft.ServiceBus](#microsoftservicebus)
> - [Microsoft.ServiceFabric](#microsoftservicefabric)
> - [Microsoft.ServiceFabricMesh](#microsoftservicefabricmesh)
> - [Microsoft.Services](#microsoftservices)
> - [Microsoft.SignalRService](#microsoftsignalrservice)
> - [Microsoft.Singularity](#microsoftsingularity)
> - [Microsoft.SoftwarePlan](#microsoftsoftwareplan)
> - [Microsoft.Solutions](#microsoftsolutions)
> - [Microsoft.SQL](#microsoftsql)
> - [Microsoft.SqlVirtualMachine](#microsoftsqlvirtualmachine)
> - [Microsoft.Storage](#microsoftstorage)
> - [Microsoft.StorageCache](#microsoftstoragecache)
> - [Microsoft.StorageReplication](#microsoftstoragereplication)
> - [Microsoft.StorageSync](#microsoftstoragesync)
> - [Microsoft.StorageSyncDev](#microsoftstoragesyncdev)
> - [Microsoft.StorageSyncInt](#microsoftstoragesyncint)
> - [Microsoft.StorSimple](#microsoftstorsimple)
> - [Microsoft.StreamAnalytics](#microsoftstreamanalytics)
> - [Microsoft.Subscription](#microsoftsubscription)
> - [Microsoft.Synapse](#microsoftsynapse)
> - [Microsoft.TimeSeriesInsights](#microsofttimeseriesinsights)
> - [Microsoft.Token](#microsofttoken)
> - [Microsoft.VirtualMachineImages](#microsoftvirtualmachineimages)
> - [Microsoft.VMware](#microsoftvmware)
> - [Microsoft.VMwareCloudSimple](#microsoftvmwarecloudsimple)
> - [Microsoft.VnfManager](#microsoftvnfmanager)
> - [Microsoft.VSOnline](#microsoftvsonline)
> - [Microsoft.Web](#microsoftweb)
> - [Microsoft.WindowsDefenderATP](#microsoftwindowsdefenderatp)
> - [Microsoft.WindowsESU](#microsoftwindowsesu)
> - [Microsoft.WindowsIoT](#microsoftwindowsiot)
> - [Microsoft.WorkloadBuilder](#microsoftworkloadbuilder)
> - [Microsoft.WorkloadMonitor](#microsoftworkloadmonitor)

## <a name="microsoftaad"></a>Microsoft.AAD

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | DomainServices | Oui | Oui |
> | DomainServices / oucontainer | Non | Non |

## <a name="microsoftaddons"></a>Microsoft.Addons

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | supportProviders | Non | Non |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
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
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | advisorScore | Non | Non |
> | configurations | Non | Non |
> | generateRecommendations | Non | Non |
> | metadata | Non | Non |
> | de films | Non | Non |
> | suppressions | Non | Non |

## <a name="microsoftagfoodplatform"></a>Microsoft.AgFoodPlatform

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | farmBeats | Oui | Oui |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | actionRules | Oui | Oui |
> | alertes | Non | Non |
> | alertsList | Non | Non |
> | alertsMetaData | Non | Non |
> | alertsSummary | Non | Non |
> | alertsSummaryList | Non | Non |
> | smartDetectorAlertRules | Oui | Oui |
> | smartGroups | Non | Non |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | servers | Oui | Oui |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | reportFeedback | Non | Non |
> | service | Oui | Oui |
> | validateServiceName | Non | Non |

> [!NOTE]
> Gestion des API Azure prend en charge uniquement la création d’un maximum de 15 paires nom-valeur de balise pour chaque service.

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | configurationStores | Oui | Non |
> | configurationStores / eventGridFilters | Non | Non |
> | configurationStores / keyValues | Non | Non |

## <a name="microsoftappplatform"></a>Microsoft.AppPlatform

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | Spring | Oui | Oui |
> | Spring / apps | Non | Non |
> | Spring / apps / deployments | Non | Non |

## <a name="microsoftattestation"></a>Microsoft.Attestation

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | attestationProviders | Oui | Oui |
> | defaultProviders | Non | Non |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accessReviewScheduleDefinitions | Non | Non |
> | accessReviewScheduleSettings | Non | Non |
> | classicAdministrators | Non | Non |
> | dataAliases | Non | Non |
> | denyAssignments | Non | Non |
> | elevateAccess | Non | Non |
> | findOrphanRoleAssignments | Non | Non |
> | locks | Non | Non |
> | autorisations | Non | Non |
> | policyAssignments | Non | Non |
> | policyDefinitions | Non | Non |
> | policyExemptions | Non | Non |
> | policySetDefinitions | Non | Non |
> | privateLinkAssociations | Non | Non |
> | providerOperations | Non | Non |
> | resourceManagementPrivateLinks | Oui | Oui |
> | roleAssignments | Non | Non |
> | roleAssignmentsUsageMetrics | Non | Non |
> | roleDefinitions | Non | Non |

## <a name="microsoftautomanage"></a>Microsoft.Automanage

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accounts | Oui | Oui |
> | configurationProfileAssignments | Non | Non |
> | configurationProfilePreferences | Oui | Oui |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | automationAccounts | Oui | Oui |
> | automationAccounts / configurations | Oui | Oui |
> | automationAccounts / jobs | Non | Non |
> | automationAccounts / privateEndpointConnectionProxies | Non | Non |
> | automationAccounts / privateEndpointConnections | Non | Non |
> | automationAccounts / privateLinkResources | Non | Non |
> | automationAccounts / runbooks | Oui | Oui |
> | automationAccounts / softwareUpdateConfigurations | Non | Non |
> | automationAccounts / webhooks | Non | Non |

> [!NOTE]
> Azure Automation prend en charge uniquement la création d’un maximum de 15 paires nom-valeur de balise pour chaque ressource Automation.

## <a name="microsoftavs"></a>Microsoft.AVS

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | privateClouds | Oui | Oui |
> | privateClouds / addons | Non | Non |
> | Clouds privés / autorisations | Non | Non |
> | Clouds privés / clusters | Non | Non |
> | privateClouds / globalReachConnections | Non | Non |
> | Clouds privés / hcxEnterpriseSites | Non | Non |
> | privateClouds / workloadNetworks | Non | Non |
> | privateClouds / workloadNetworks / dhcpConfigurations | Non | Non |
> | privateClouds / workloadNetworks / gateways | Non | Non |
> | privateClouds / workloadNetworks / portMirroringProfiles | Non | Non |
> | privateClouds / workloadNetworks / segments | Non | Non |
> | privateClouds / workloadNetworks / virtualMachines | Non | Non |
> | privateClouds / workloadNetworks / vmGroups | Non | Non |

## <a name="microsoftazuregeneva"></a>Microsoft.Azure.Geneva

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | environments | Non | Non |
> | environments / accounts | Non | Non |
> | environments / accounts / namespaces | Non | Non |
> | environments / accounts / namespaces / configurations | Non | Non |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | b2cDirectories | Oui | Non |
> | b2ctenants | Non | Non |
> | guestUsages | Oui | Oui |

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | dataControllers | Oui | Oui |
> | postgresInstances | Oui | Oui |
> | sqlManagedInstances | Oui | Oui |
> | sqlServerInstances | Oui | Oui |
> | sqlServerRegistrations | Oui | Oui |
> | sqlServerRegistrations / sqlServers | Non | Non |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | cloudManifestFiles | Non | Non |
> | edgeSubscriptions | Oui | Oui |
> | linkedSubscriptions | Oui | Oui |
> | registrations | Oui | Oui |
> | registrations / customerSubscriptions | Non | Non |
> | registrations / products | Non | Non |

## <a name="microsoftazurestackhci"></a>Microsoft.AzureStackHCI

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | clusters | Oui | Oui |

## <a name="microsoftbaremetalinfrastructure"></a>Microsoft.BareMetalInfrastructure

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | bareMetalInstances | Oui | Oui |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | batchAccounts | Oui | Oui |
> | batchAccounts / certificates | Non | Non |
> | batchAccounts / pools | Non | Non |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | billingAccounts | Non | Non |
> | billingAccounts / agreements | Non | Non |
> | billingAccounts / billingPermissions | Non | Non |
> | billingAccounts / billingProfiles | Non | Non |
> | billingAccounts / billingProfiles / billingPermissions | Non | Non |
> | billingAccounts / billingProfiles / billingRoleAssignments | Non | Non |
> | billingAccounts / billingProfiles / billingRoleDefinitions | Non | Non |
> | billingAccounts / billingProfiles / billingSubscriptions | Non | Non |
> | billingAccounts / billingProfiles / createBillingRoleAssignment | Non | Non |
> | billingAccounts / billingProfiles / customers | Non | Non |
> | billingAccounts / billingProfiles / instructions | Non | Non |
> | billingAccounts / billingProfiles / invoices | Non | Non |
> | billingAccounts / billingProfiles / invoices / pricesheet | Non | Non |
> | billingAccounts / billingProfiles / invoices / transactions | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections / billingPermissions | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections / billingRoleAssignments | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections / billingRoleDefinitions | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections / billingSubscriptions | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections / createBillingRoleAssignment | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections / initiateTransfer | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections / products | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections / products / transfer | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections / products / updateAutoRenew | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections / transactions | Non | Non |
> | billingAccounts / billingProfiles / invoiceSections / transfers | Non | Non |
> | billingAccounts / BillingProfiles / patchOperations | Non | Non |
> | billingAccounts / billingProfiles / paymentMethods | Non | Non |
> | billingAccounts / billingProfiles / policies | Non | Non |
> | billingAccounts / billingProfiles / pricesheet | Non | Non |
> | billingAccounts / billingProfiles / pricesheetDownloadOperations | Non | Non |
> | billingAccounts / billingProfiles / products | Non | Non |
> | billingAccounts / billingProfiles / reservations | Non | Non |
> | billingAccounts / billingProfiles / transactions | Non | Non |
> | billingAccounts / billingProfiles / validateDetachPaymentMethodEligibility | Non | Non |
> | billingAccounts / billingRoleAssignments | Non | Non |
> | billingAccounts / billingRoleDefinitions | Non | Non |
> | billingAccounts / billingSubscriptions | Non | Non |
> | billingAccounts / billingSubscriptions / invoices | Non | Non |
> | billingAccounts / createBillingRoleAssignment | Non | Non |
> | billingAccounts / createInvoiceSectionOperations | Non | Non |
> | billingAccounts / customers | Non | Non |
> | billingAccounts / customers / billingPermissions | Non | Non |
> | billingAccounts / customers / billingSubscriptions | Non | Non |
> | billingAccounts / customers / initiateTransfer | Non | Non |
> | billingAccounts / customers / policies | Non | Non |
> | billingAccounts / customers / products | Non | Non |
> | billingAccounts / customers / transactions | Non | Non |
> | billingAccounts / customers / transfers | Non | Non |
> | billingAccounts / departments | Non | Non |
> | billingAccounts / departments / billingPermissions | Non | Non |
> | billingAccounts / departments / billingRoleAssignments | Non | Non |
> | billingAccounts / departments / billingRoleDefinitions | Non | Non |
> | billingAccounts / enrollmentAccounts | Non | Non |
> | billingAccounts / enrollmentAccounts / billingPermissions | Non | Non |
> | billingAccounts / enrollmentAccounts / billingRoleAssignments | Non | Non |
> | billingAccounts / enrollmentAccounts / billingRoleDefinitions | Non | Non |
> | billingAccounts / invoices | Non | Non |
> | billingAccounts / invoices / transactions | Non | Non |
> | billingAccounts / invoiceSections | Non | Non |
> | billingAccounts / invoiceSections / billingSubscriptionMoveOperations | Non | Non |
> | billingAccounts / invoiceSections / billingSubscriptions | Non | Non |
> | billingAccounts / invoiceSections / billingSubscriptions / transfer | Non | Non |
> | billingAccounts / invoiceSections / elevate | Non | Non |
> | billingAccounts / invoiceSections / initiateTransfer | Non | Non |
> | billingAccounts / invoiceSections / patchOperations | Non | Non |
> | billingAccounts / invoiceSections / productMoveOperations | Non | Non |
> | billingAccounts / invoiceSections / products | Non | Non |
> | billingAccounts / invoiceSections / products / transfer | Non | Non |
> | billingAccounts / invoiceSections / products / updateAutoRenew | Non | Non |
> | billingAccounts / invoiceSections / transactions | Non | Non |
> | billingAccounts / invoiceSections / transfers | Non | Non |
> | billingAccounts / lineOfCredit | Non | Non |
> | billingAccounts / patchOperations | Non | Non |
> | billingAccounts / paymentMethods | Non | Non |
> | billingAccounts / products | Non | Non |
> | billingAccounts / reservations | Non | Non |
> | billingAccounts / transactions | Non | Non |
> | billingPeriods | Non | Non |
> | billingPermissions | Non | Non |
> | billingProperty | Non | Non |
> | billingRoleAssignments | Non | Non |
> | billingRoleDefinitions | Non | Non |
> | createBillingRoleAssignment | Non | Non |
> | departments | Non | Non |
> | enrollmentAccounts | Non | Non |
> | invoices | Non | Non |
> | transfers | Non | Non |
> | transfers / acceptTransfer | Non | Non |
> | transfers / declineTransfer | Non | Non |
> | transfers / operationStatus | Non | Non |
> | transfers / validateTransfer | Non | Non |
> | validateAddress | Non | Non |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | mapApis | Oui | Oui |
> | updateCommunicationPreference | Non | Non |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | blockchainMembers | Oui | Oui |
> | cordaMembers | Oui | Oui |
> | watchers | Oui | Oui |

## <a name="microsoftblockchaintokens"></a>Microsoft.BlockchainTokens

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | TokenServices | Oui | Oui |
> | TokenServices / BlockchainNetworks | Non | Non |
> | TokenServices / Groups | Non | Non |
> | TokenServices / Groups / Accounts | Non | Non |
> | TokenServices / TokenTemplates | Non | Non |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | blueprintAssignments | Non | Non |
> | blueprintAssignments / assignmentOperations | Non | Non |
> | blueprintAssignments / operations | Non | Non |
> | blueprints | Non | Non |
> | blueprints / artifacts | Non | Non |
> | blueprints / versions | Non | Non |
> | blueprints / versions / artifacts | Non | Non |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | botServices | Oui | Oui |
> | botServices / channels | Non | Non |
> | botServices / connections | Non | Non |
> | languages | Non | Non |
> | modèles | Non | Non |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | Redis | Oui | Oui |
> | Redis / EventGridFilters | Non | Non |
> | Redis / privateEndpointConnectionProxies | Non | Non |
> | Redis / privateEndpointConnectionProxies / validate | Non | Non |
> | Redis / privateEndpointConnections | Non | Non |
> | Redis / privateLinkResources | Non | Non |
> | redisEnterprise | Oui | Oui |
> | RedisEnterprise / privateEndpointConnectionProxies | Non | Non |
> | RedisEnterprise / privateEndpointConnectionProxies / validate | Non | Non |
> | RedisEnterprise / privateEndpointConnections | Non | Non |
> | RedisEnterprise / privateLinkResources | Non | Non |

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | appliedReservations | Non | Non |
> | autoQuotaIncrease | Non | Non |
> | calculateExchange | Non | Non |
> | calculatePrice | Non | Non |
> | calculatePurchasePrice | Non | Non |
> | catalogs | Non | Non |
> | commercialReservationOrders | Non | Non |
> | change | Non | Non |
> | ownReservations | Non | Non |
> | placePurchaseOrder | Non | Non |
> | reservationOrders | Non | Non |
> | reservationOrders / calculateRefund | Non | Non |
> | reservationOrders / merge | Non | Non |
> | reservationOrders / reservations | Non | Non |
> | reservationOrders / reservations / revisions | Non | Non |
> | reservationOrders / return | Non | Non |
> | reservationOrders / split | Non | Non |
> | reservationOrders / swap | Non | Non |
> | reservations | Non | Non |
> | resourceProviders | Non | Non |
> | les ressources | Non | Non |
> | validateReservationOrder | Non | Non |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | CdnWebApplicationFirewallManagedRuleSets | Non | Non |
> | CdnWebApplicationFirewallPolicies | Oui | Oui |
> | edgenodes | Non | Non |
> | profiles | Oui | Oui |
> | profiles / endpoints | Oui | Oui |
> | profiles / endpoints / customdomains | Non | Non |
> | profiles / endpoints / origingroups | Non | Non |
> | profiles / endpoints / origins | Non | Non |
> | validateProbe | Non | Non |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | certificateOrders | Oui | Oui |
> | certificateOrders / certificates | Non | Non |
> | validateCertificateRegistrationInformation | Non | Non |

## <a name="microsoftchangeanalysis"></a>Microsoft.ChangeAnalysis

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | profile | Non | Non |
> | resourceChanges | Non | Non |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | capabilities | Non | Non |
> | domainNames | Non | Non |
> | domainNames / capabilities | Non | Non |
> | domainNames / internalLoadBalancers | Non | Non |
> | domainNames / serviceCertificates | Non | Non |
> | domainNames / slots | Non | Non |
> | domainNames / slots / roles | Non | Non |
> | domainNames / slots / roles / metricDefinitions | Non | Non |
> | domainNames / slots / roles / metrics | Non | Non |
> | moveSubscriptionResources | Non | Non |
> | operatingSystemFamilies | Non | Non |
> | operatingSystems | Non | Non |
> | quotas | Non | Non |
> | resourceTypes | Non | Non |
> | validateSubscriptionMoveAvailability | Non | Non |
> | virtualMachines | Non | Non |
> | virtualMachines / diagnosticSettings | Non | Non |
> | virtualMachines / metricDefinitions | Non | Non |
> | virtualMachines / metrics | Non | Non |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | classicInfrastructureResources | Non | Non |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | capabilities | Non | Non |
> | expressRouteCrossConnections | Non | Non |
> | expressRouteCrossConnections / peerings | Non | Non |
> | gatewaySupportedDevices | Non | Non |
> | networkSecurityGroups | Non | Non |
> | quotas | Non | Non |
> | reservedIps | Non | Non |
> | virtualNetworks | Non | Non |
> | virtualNetworks / remoteVirtualNetworkPeeringProxies | Non | Non |
> | virtualNetworks / virtualNetworkPeerings | Non | Non |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | capabilities | Non | Non |
> | disks | Non | Non |
> | images | Non | Non |
> | osImages | Non | Non |
> | osPlatformImages | Non | Non |
> | publicImages | Non | Non |
> | quotas | Non | Non |
> | storageAccounts | Non | Non |
> | storageAccounts / blobServices | Non | Non |
> | storageAccounts / fileServices | Non | Non |
> | storageAccounts / metricDefinitions | Non | Non |
> | storageAccounts / metrics | Non | Non |
> | storageAccounts / queueServices | Non | Non |
> | storageAccounts / services | Non | Non |
> | storageAccounts / services / diagnosticSettings | Non | Non |
> | storageAccounts / services / metricDefinitions | Non | Non |
> | storageAccounts / services / metrics | Non | Non |
> | storageAccounts / tableServices | Non | Non |
> | storageAccounts / vmImages | Non | Non |
> | vmImages | Non | Non |

## <a name="microsoftcodespaces"></a>Microsoft.Codespaces

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | plans | Oui | Non |
> | registeredSubscriptions | Non | Non |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accounts | Oui | Oui |
> | accounts / privateEndpointConnectionProxies | Non | Non |
> | accounts / privateEndpointConnections | Non | Non |
> | accounts / privateLinkResources | Non | Non |

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | RateCard | Non | Non |
> | UsageAggregates | Non | Non |

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | availabilitySets | Oui | Oui |
> | cloudServices | Oui | Oui |
> | cloudServices / networkInterfaces | Non | Non |
> | cloudServices / publicIPAddresses | Non | Non |
> | cloudServices / roleInstances | Non | Non |
> | cloudServices / roleInstances / networkInterfaces | Non | Non |
> | cloudServices / roles | Non | Non |
> | diskAccesses | Oui | Oui |
> | diskEncryptionSets | Oui | Oui |
> | disks | Oui | Oui |
> | galleries | Oui | Oui |
> | galleries / applications | Non | Non |
> | galleries / applications / versions | Non | Non |
> | galleries / images | Non | Non |
> | galleries / images / versions | Non | Non |
> | hostGroups | Oui | Oui |
> | hostGroups / hosts | Oui | Oui |
> | images | Oui | Oui |
> | proximityPlacementGroups | Oui | Oui |
> | restorePointCollections | Oui | Oui |
> | restorePointCollections / restorePoints | Non | Non |
> | sharedVMExtensions | Oui | Oui |
> | sharedVMExtensions / versions | Non | Non |
> | sharedVMImages | Oui | Oui |
> | sharedVMImages / versions | Non | Non |
> | snapshots | Oui | Oui |
> | sshPublicKeys | Oui | Oui |
> | virtualMachines | Oui | Oui |
> | virtualMachines / extensions | Oui | Oui |
> | virtualMachines / metricDefinitions | Non | Non |
> | virtualMachines / runCommands | Oui | Oui |
> | virtualMachineScaleSets | Oui | Oui |
> | virtualMachineScaleSets / extensions | Non | Non |
> | virtualMachineScaleSets / networkInterfaces | Non | Non |
> | virtualMachineScaleSets / publicIPAddresses | Oui | Non |
> | virtualMachineScaleSets / virtualMachines | Non | Non |
> | virtualMachineScaleSets / virtualMachines / networkInterfaces | Non | Non |

> [!NOTE]
> Vous ne pouvez pas ajouter une balise à une machine virtuelle qui a été marquée comme généralisée. Vous marquez machine virtuelle comme généralisée avec [Set-AzVm-Generalized](/powershell/module/Az.Compute/Set-AzVM) ou [az vm generalize](/cli/azure/vm#az-vm-generalize).

## <a name="microsoftconnectedcache"></a>Microsoft.ConnectedCache

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | CacheNodes | Oui | Oui |

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | AggregatedCost | Non | Non |
> | Balances | Non | Non |
> | Budgets | Non | Non |
> | Charges | Non | Non |
> | CostTags | Non | Non |
> | credits | Non | Non |
> | événements | Non | Non |
> | Prévisions | Non | Non |
> | lots | Non | Non |
> | Marketplaces | Non | Non |
> | Pricesheets | Non | Non |
> | products | Non | Non |
> | ReservationDetails | Non | Non |
> | ReservationRecommendationDetails | Non | Non |
> | ReservationRecommendations | Non | Non |
> | ReservationSummaries | Non | Non |
> | ReservationTransactions | Non | Non |
> | Balises | Non | Non |
> | tenants | Non | Non |
> | Termes | Non | Non |
> | UsageDetails | Non | Non |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | containerGroups | Oui | Oui |
> | serviceAssociationLinks | Non | Non |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | registries | Oui | Oui |
> | registries / agentPools | Oui | Oui |
> | registries / builds | Non | Non |
> | registries / builds / cancel | Non | Non |
> | registries / builds / getLogLink | Non | Non |
> | registries / buildTasks | Oui | Oui |
> | registries / buildTasks / steps | Non | Non |
> | registries / eventGridFilters | Non | Non |
> | registries / exportPipelines | Non | Non |
> | registries / generateCredentials | Non | Non |
> | registries / getBuildSourceUploadUrl | Non | Non |
> | registries / GetCredentials | Non | Non |
> | registries / importImage | Non | Non |
> | registries / importPipelines | Non | Non |
> | registries / pipelineRuns | Non | Non |
> | registries / privateEndpointConnectionProxies | Non | Non |
> | registries / privateEndpointConnectionProxies / validate | Non | Non |
> | registries / privateEndpointConnections | Non | Non |
> | registries / privateLinkResources | Non | Non |
> | registries / queueBuild | Non | Non |
> | registries / regenerateCredential | Non | Non |
> | registries / regenerateCredentials | Non | Non |
> | registries / replications | Oui | Oui |
> | registries / runs | Non | Non |
> | registries / runs / cancel | Non | Non |
> | registries / scheduleRun | Non | Non |
> | registries / scopeMaps | Non | Non |
> | registries / taskRuns | Non | Non |
> | registries / tasks | Oui | Oui |
> | registries / tokens | Non | Non |
> | registries / updatePolicies | Non | Non |
> | registries / webhooks | Oui | Oui |
> | registries / webhooks / getCallbackConfig | Non | Non |
> | registries / webhooks / ping | Non | Non |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | containerServices | Oui | Oui |
> | managedclusters | Oui | Oui |
> | openShiftManagedClusters | Oui | Oui |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | Alertes | Non | Non |
> | BillingAccounts | Non | Non |
> | Budgets | Non | Non |
> | CloudConnectors | Non | Non |
> | Connecteurs | Oui | Oui |
> | costAllocationRules | Non | Non |
> | Departments | Non | Non |
> | Dimensions | Non | Non |
> | EnrollmentAccounts | Non | Non |
> | Exports | Non | Non |
> | ExternalBillingAccounts | Non | Non |
> | ExternalBillingAccounts / Alerts | Non | Non |
> | ExternalBillingAccounts / Dimensions | Non | Non |
> | ExternalBillingAccounts / Forecast | Non | Non |
> | ExternalBillingAccounts / Query | Non | Non |
> | ExternalSubscriptions | Non | Non |
> | ExternalSubscriptions / Alerts | Non | Non |
> | ExternalSubscriptions / Dimensions | Non | Non |
> | ExternalSubscriptions / Forecast | Non | Non |
> | ExternalSubscriptions / Query | Non | Non |
> | Forecast | Non | Non |
> | Insights | Non | Non |
> | Requête | Non | Non |
> | inscription | Non | Non |
> | Reportconfigs | Non | Non |
> | Rapports | Non | Non |
> | Paramètres | Non | Non |
> | showbackRules | Non | Non |
> | Les vues | Non | Non |

## <a name="microsoftcustomerlockbox"></a>Microsoft.CustomerLockbox

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | requêtes | Non | Non |

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | associations | Non | Non |
> | resourceProviders | Oui | Oui |

## <a name="microsoftd365customerinsights"></a>Microsoft.D365CustomerInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | instances | Oui | Oui |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | jobs | Oui | Oui |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | DataBoxEdgeDevices | Oui | Oui |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | workspaces | Oui | Oui |
> | workspaces / dbWorkspaces | Non | Non |
> | workspaces / virtualNetworkPeerings | Non | Non |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | catalogs | Oui | Oui |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | dataFactories | Oui | Oui |
> | dataFactories / diagnosticSettings | Non | Non |
> | dataFactories / metricDefinitions | Non | Non |
> | dataFactorySchema | Non | Non |
> | factories | Oui | Oui |
> | factories / integrationRuntimes | Non | Non |

> [!NOTE]
> Si vous disposez de runtimes d’intégration Azure-SSIS dans votre fabrique de données, leur coût d’exécution sera marqué par des balises Data Factory. L’exécution des runtimes d’intégration Azure-SSIS doit être arrêtée et redémarrée pour permettre l’application des nouvelles balises de la fabrique de données à leur coût d’exécution.

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accounts | Oui | Oui |
> | accounts / dataLakeStoreAccounts | Non | Non |
> | accounts / storageAccounts | Non | Non |
> | accounts / storageAccounts / containers | Non | Non |
> | accounts / transferAnalyticsUnits | Non | Non |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accounts | Oui | Oui |
> | accounts / eventGridFilters | Non | Non |
> | accounts / firewallRules | Non | Non |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | services | Non | Non |
> | services / projects | Non | Non |

## <a name="microsoftdataprotection"></a>Microsoft.DataProtection

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | BackupVaults | Oui | Oui |
> | ResourceOperationGateKeepers | Oui | Oui |

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accounts | Oui | Oui |
> | accounts / shares | Non | Non |
> | accounts / shares / datasets | Non | Non |
> | accounts / shares / invitations | Non | Non |
> | accounts / shares / providersharesubscriptions | Non | Non |
> | accounts / shares / synchronizationSettings | Non | Non |
> | accounts / sharesubscriptions | Non | Non |
> | accounts / sharesubscriptions / consumerSourceDataSets | Non | Non |
> | accounts / sharesubscriptions / datasetmappings | Non | Non |
> | accounts / sharesubscriptions / triggers | Non | Non |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | servers | Oui | Oui |
> | servers / advisors | Non | Non |
> | servers / keys | Non | Non |
> | servers / privateEndpointConnectionProxies | Non | Non |
> | servers / privateEndpointConnections | Non | Non |
> | servers / privateLinkResources | Non | Non |
> | servers / queryTexts | Non | Non |
> | servers / recoverableServers | Non | Non |
> | servers / start | Non | Non |
> | servers / stop | Non | Non |
> | servers / topQueryStatistics | Non | Non |
> | servers / virtualNetworkRules | Non | Non |
> | servers / waitStatistics | Non | Non |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | flexibleServers | Oui | Oui |
> | servers | Oui | Oui |
> | servers / advisors | Non | Non |
> | servers / keys | Non | Non |
> | servers / privateEndpointConnectionProxies | Non | Non |
> | servers / privateEndpointConnections | Non | Non |
> | servers / privateLinkResources | Non | Non |
> | servers / queryTexts | Non | Non |
> | servers / recoverableServers | Non | Non |
> | servers / start | Non | Non |
> | servers / stop | Non | Non |
> | servers / topQueryStatistics | Non | Non |
> | servers / upgrade | Non | Non |
> | servers / virtualNetworkRules | Non | Non |
> | servers / waitStatistics | Non | Non |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | flexibleServers | Oui | Oui |
> | serverGroups | Oui | Oui |
> | servers | Oui | Oui |
> | servers / advisors | Non | Non |
> | servers / keys | Non | Non |
> | servers / privateEndpointConnectionProxies | Non | Non |
> | servers / privateEndpointConnections | Non | Non |
> | servers / privateLinkResources | Non | Non |
> | servers / queryTexts | Non | Non |
> | servers / recoverableServers | Non | Non |
> | servers / topQueryStatistics | Non | Non |
> | servers / virtualNetworkRules | Non | Non |
> | servers / waitStatistics | Non | Non |
> | serversv2 | Oui | Oui |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | artifactSources | Oui | Oui |
> | rollouts | Oui | Oui |
> | serviceTopologies | Oui | Oui |
> | serviceTopologies / services | Oui | Oui |
> | serviceTopologies / services / serviceUnits | Oui | Oui |
> | steps | Oui | Oui |

## <a name="microsoftdesktopvirtualization"></a>Microsoft.DesktopVirtualization

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | applicationgroups | Oui | Oui |
> | applicationgroups / applications | Non | Non |
> | applicationgroups / desktops | Non | Non |
> | applicationgroups / startmenuitems | Non | Non |
> | hostpools | Oui | Oui |
> | hostpools / msixpackages | Non | Non |
> | hostpools / sessionhosts | Non | Non |
> | hostpools / sessionhosts / usersessions | Non | Non |
> | hostpools / usersessions | Non | Non |
> | workspaces | Oui | Oui |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | ElasticPools | Oui | Oui |
> | ElasticPools / IotHubTenants | Oui | Oui |
> | ElasticPools / IotHubTenants / securitySettings | Non | Non |
> | IoTHubs | Oui | Oui |
> | IotHubs / eventGridFilters | Non | Non |
> | IotHubs / securitySettings | Non | Non |
> | ProvisioningServices | Oui | Oui |
> | usages | Non | Non |

## <a name="microsoftdeviceupdate"></a>Microsoft.DeviceUpdate

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accounts | Oui | Oui |
> | accounts / instances | Oui | Oui |

## <a name="microsoftdevops"></a>Microsoft.DevOps

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | pipelines | Oui | Oui |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | controllers | Oui | Oui |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | labcenters | Oui | Oui |
> | labs | Oui | Oui |
> | labs / environments | Oui | Oui |
> | labs / serviceRunners | Oui | Oui |
> | labs / virtualMachines | Oui | Oui |
> | schedules | Oui | Oui |

## <a name="microsoftdigitaltwins"></a>Microsoft.DigitalTwins

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | digitalTwinsInstances | Oui | Oui |
> | digitalTwinsInstances / endpoints | Non | Non |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | databaseAccountNames | Non | Non |
> | databaseAccounts | Oui | Oui |
> | restorableDatabaseAccounts | Non | Non |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | domaines | Oui | Oui |
> | domains / domainOwnershipIdentifiers | Non | Non |
> | generateSsoRequest | Non | Non |
> | topLevelDomains | Non | Non |
> | validateDomainRegistrationInformation | Non | Non |

## <a name="microsoftdynamicslcs"></a>Microsoft.DynamicsLcs

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | lcsprojects | Non | Non |
> | lcsprojects / clouddeployments | Non | Non |
> | lcsprojects / connectors | Non | Non |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | services | Oui | Oui |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | domaines | Oui | Oui |
> | domains / topics | Non | Non |
> | eventSubscriptions | Non | Non |
> | extensionTopics | Non | Non |
> | partnerNamespaces | Oui | Oui |
> | partnerNamespaces / eventChannels | Non | Non |
> | partnerRegistrations | Oui | Oui |
> | partnerTopics | Oui | Oui |
> | partnerTopics / eventSubscriptions | Non | Non |
> | systemTopics | Oui | Oui |
> | systemTopics / eventSubscriptions | Non | Non |
> | topics | Oui | Oui |
> | topicTypes | Non | Non |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | clusters | Oui | Oui |
> | espaces de noms | Oui | Oui |
> | namespaces / authorizationrules | Non | Non |
> | namespaces / disasterrecoveryconfigs | Non | Non |
> | namespaces / eventhubs | Non | Non |
> | namespaces / eventhubs / authorizationrules | Non | Non |
> | namespaces / eventhubs / consumergroups | Non | Non |
> | namespaces / networkrulesets | Non | Non |
> | namespaces / privateEndpointConnections | Non | Non |

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | experimentWorkspaces | Oui | Oui |

## <a name="microsoftfalcon"></a>Microsoft.Falcon

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | espaces de noms | Oui | Oui |

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | featureProviders | Non | Non |
> | features | Non | Non |
> | fournisseurs | Non | Non |
> | subscriptionFeatureRegistrations | Non | Non |

## <a name="microsoftgallery"></a>Microsoft.Gallery

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | enroll | Non | Non |
> | galleryitems | Non | Non |
> | generateartifactaccessuri | Non | Non |
> | myareas | Non | Non |
> | myareas / areas | Non | Non |
> | myareas / areas / areas | Non | Non |
> | myareas / areas / areas / galleryitems | Non | Non |
> | myareas / areas / galleryitems | Non | Non |
> | myareas / galleryitems | Non | Non |
> | inscription | Non | Non |
> | les ressources | Non | Non |
> | retrieveresourcesbyid | Non | Non |

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accounts | Oui | Oui |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | autoManagedAccounts | Oui | Oui |
> | autoManagedVmConfigurationProfiles | Oui | Oui |
> | configurationProfileAssignments | Non | Non |
> | guestConfigurationAssignments | Non | Non |
> | software | Non | Non |
> | softwareUpdateProfile | Non | Non |
> | softwareUpdates | Non | Non |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | hanaInstances | Oui | Oui |
> | sapMonitors | Oui | Oui |

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | dedicatedHSMs | Oui | Oui |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | clusters | Oui | Oui |
> | clusters / applications | Non | Non |

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | services | Oui | Oui |
> | services / iomtconnectors | Non | Non |
> | services / iomtconnectors / connections | Non | Non |
> | services / iomtconnectors / mappings | Non | Non |
> | services / privateEndpointConnectionProxies | Non | Non |
> | services / privateEndpointConnections | Non | Non |
> | services / privateLinkResources | Non | Non |

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | machines | Oui | Oui |
> | machines / assessPatches | Non | Non |
> | machines / extensions | Oui | Oui |
> | machines / installPatches | Non | Non |

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | dataManagers | Oui | Oui |

## <a name="microsofthybridnetwork"></a>Microsoft.HybridNetwork

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | périphériques | Oui | Oui |
> | networkFunctions | Oui | Oui |
> | networkFunctionVendors | Non | Non |
> | registeredSubscriptions | Non | Non |
> | vendors | Non | Non |
> | vendors / vendorSkus | Non | Non |
> | vendors / vendorSkus / previewSubscriptions | Non | Non |
> | virtualNetworkFunctions | Oui | Oui |
> | virtualNetworkFunctionVendors | Non | Non |

## <a name="microsofthydra"></a>Microsoft.Hydra

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | components | Oui | Oui |
> | networkScopes | Oui | Oui |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | jobs | Oui | Oui |

## <a name="microsoftinsights"></a>Microsoft.Insights

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | actionGroups | Oui | Oui |
> | activityLogAlerts | Oui | Oui |
> | alertrules | Oui | Oui |
> | autoscalesettings | Oui | Oui |
> | components | Oui | Oui |
> | components / linkedStorageAccounts | Non | Non |
> | components / ProactiveDetectionConfigs | Non | Non |
> | diagnosticSettings | Non | Non |
> | guestDiagnosticSettings | Oui | Oui |
> | guestDiagnosticSettingsAssociation | Oui | Oui |
> | logprofiles | Oui | Oui |
> | metricAlerts | Oui | Oui |
> | privateLinkScopes | Oui | Oui |
> | privateLinkScopes / privateEndpointConnections | Non | Non |
> | privateLinkScopes / scopedResources | Non | Non |
> | queryPacks | Oui | Oui |
> | queryPacks / queries | Non | Non |
> | scheduledQueryRules | Oui | Oui |
> | webtests | Oui | Oui |
> | workbooks | Oui | Oui |
> | workbooktemplates | Oui | Oui |

## <a name="microsoftintune"></a>Microsoft.Intune

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | diagnosticSettings | Non | Non |
> | diagnosticSettingsCategories | Non | Non |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | appTemplates | Non | Non |
> | IoTApps | Oui | Oui |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | Graph | Oui | Oui |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | deletedVaults | Non | Non |
> | hsmPools | Oui | Oui |
> | managedHSMs | Oui | Oui |
> | vaults | Oui | Oui |
> | vaults / accessPolicies | Non | Non |
> | vaults / eventGridFilters | Non | Non |
> | vaults / keys | Non | Non |
> | vaults / keys / versions | Non | Non |
> | vaults / secrets | Non | Non |

## <a name="microsoftkubernetes"></a>Microsoft.Kubernetes

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | connectedClusters | Oui | Oui |
> | registeredSubscriptions | Non | Non |

## <a name="microsoftkubernetesconfiguration"></a>Microsoft.KubernetesConfiguration

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | extensions | Non | Non |
> | sourceControlConfigurations | Non | Non |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | clusters | Oui | Oui |
> | clusters / attacheddatabaseconfigurations | Non | Non |
> | clusters / databases | Non | Non |
> | clusters / databases / dataconnections | Non | Non |
> | clusters / databases / eventhubconnections | Non | Non |
> | clusters / databases / principalassignments | Non | Non |
> | clusters / dataconnections | Non | Non |
> | clusters / principalassignments | Non | Non |
> | clusters / sharedidentities | Non | Non |

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | labaccounts | Oui | Non |
> | users | Non | Non |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | hostingEnvironments | Oui | Oui |
> | integrationAccounts | Oui | Oui |
> | integrationServiceEnvironments | Oui | Oui |
> | integrationServiceEnvironments / managedApis | Non | Non |
> | isolatedEnvironments | Oui | Oui |
> | workflows | Oui | Oui |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | commitmentPlans | Oui | Oui |
> | webServices | Oui | Oui |
> | Workspaces | Oui | Oui |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | workspaces | Oui | Oui |
> | workspaces / batchEndpoints | Oui | Oui |
> | workspaces / batchEndpoints / deployments | Oui | Oui |
> | workspaces / codes | Non | Non |
> | workspaces / codes / versions | Non | Non |
> | workspaces / computes | Non | Non |
> | workspaces / datastores | Non | Non |
> | workspaces / eventGridFilters | Non | Non |
> | workspaces / jobs | Non | Non |
> | workspaces / labelingJobs | Non | Non |
> | workspaces / linkedServices | Non | Non |
> | workspaces / models | Non | Non |
> | workspaces / models / versions | Non | Non |
> | workspaces / onlineEndpoints | Oui | Oui |
> | workspaces / onlineEndpoints / deployments | Oui | Oui |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | applyUpdates | Non | Non |
> | configurationAssignments | Non | Non |
> | maintenanceConfigurations | Oui | Oui |
> | publicMaintenanceConfigurations | Non | Non |
> | updates | Non | Non |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | Identities | Non | Non |
> | userAssignedIdentities | Oui | Oui |

## <a name="microsoftmanagednetwork"></a>Microsoft.ManagedNetwork

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | managedNetworks | Oui | Oui |
> | managedNetworks / managedNetworkGroups | Oui | Oui |
> | managedNetworks / managedNetworkPeeringPolicies | Oui | Oui |
> | notification | Oui | Oui |

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | marketplaceRegistrationDefinitions | Non | Non |
> | registrationAssignments | Non | Non |
> | registrationDefinitions | Non | Non |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | getEntities | Non | Non |
> | managementGroups | Non | Non |
> | managementGroups / settings | Non | Non |
> | les ressources | Non | Non |
> | startTenantBackfill | Non | Non |
> | tenantBackfillStatus | Non | Non |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accounts | Oui | Oui |
> | accounts / eventGridFilters | Non | Non |
> | accounts / privateAtlases | Oui | Oui |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | macc | Non | Non |
> | offers | Non | Non |
> | offerTypes | Non | Non |
> | offerTypes / publishers | Non | Non |
> | offerTypes / publishers / offers | Non | Non |
> | offerTypes / publishers / offers / plans | Non | Non |
> | offerTypes / publishers / offers / plans / agreements | Non | Non |
> | offerTypes / publishers / offers / plans / configs | Non | Non |
> | offerTypes / publishers / offers / plans / configs / importImage | Non | Non |
> | privategalleryitems | Non | Non |
> | privateStoreClient | Non | Non |
> | privateStores | Non | Non |
> | privateStores / offers | Non | Non |
> | products | Non | Non |
> | publishers | Non | Non |
> | publishers / offers | Non | Non |
> | publishers / offers / amendments | Non | Non |
> | inscription | Non | Non |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | classicDevServices | Oui | Oui |
> | updateCommunicationPreference | Non | Non |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | agreements | Non | Non |
> | offertypes | Non | Non |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | mediaservices | Oui | Oui |
> | mediaservices / accountFilters | Non | Non |
> | mediaservices / assets | Non | Non |
> | mediaservices / assets / assetFilters | Non | Non |
> | mediaservices / contentKeyPolicies | Non | Non |
> | mediaservices / eventGridFilters | Non | Non |
> | mediaservices / liveEventOperations | Non | Non |
> | mediaservices / liveEvents | Oui | Oui |
> | mediaservices / liveEvents / liveOutputs | Non | Non |
> | mediaservices / liveOutputOperations | Non | Non |
> | mediaservices / mediaGraphs | Non | Non |
> | mediaservices / privateEndpointConnectionOperations | Non | Non |
> | mediaservices / privateEndpointConnectionProxies | Non | Non |
> | mediaservices / privateEndpointConnections | Non | Non |
> | mediaservices / streamingEndpointOperations | Non | Non |
> | mediaservices / streamingEndpoints | Oui | Oui |
> | mediaservices / streamingLocators | Non | Non |
> | mediaservices / streamingPolicies | Non | Non |
> | mediaservices / transforms | Non | Non |
> | mediaservices / transforms / jobs | Non | Non |

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | appClusters | Oui | Oui |

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | assessmentProjects | Oui | Oui |
> | migrateprojects | Oui | Oui |
> | moveCollections | Oui | Oui |
> | projects | Oui | Oui |

## <a name="microsoftmixedreality"></a>Microsoft.MixedReality

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | holographicsBroadcastAccounts | Oui | Oui |
> | objectUnderstandingAccounts | Oui | Oui |
> | remoteRenderingAccounts | Oui | Oui |
> | spatialAnchorsAccounts | Oui | Oui |

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | netAppAccounts | Oui | Non |
> | netAppAccounts / accountBackups | Non | Non |
> | netAppAccounts / capacityPools | Oui | Non |
> | netAppAccounts / capacityPools / volumes | Oui | Non |
> | netAppAccounts / capacityPools / volumes / snapshots | Non | Non |

## <a name="microsoftnetwork"></a>Microsoft.Network

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | applicationGateways | Oui | Oui |
> | applicationGatewayWebApplicationFirewallPolicies | Oui | Oui |
> | applicationSecurityGroups | Oui | Oui |
> | azureFirewallFqdnTags | Non | Non |
> | azureFirewalls | Oui | Non |
> | bastionHosts | Oui | Non |
> | bgpServiceCommunities | Non | Non |
> | connections | Oui | Oui |
> | ddosCustomPolicies | Oui | Oui |
> | ddosProtectionPlans | Oui | Oui |
> | dnsOperationStatuses | Non | Non |
> | dnszones | Oui | Oui |
> | dnszones / A | Non | Non |
> | dnszones / AAAA | Non | Non |
> | dnszones / all | Non | Non |
> | dnszones / CAA | Non | Non |
> | dnszones / CNAME | Non | Non |
> | dnszones / MX | Non | Non |
> | dnszones / NS | Non | Non |
> | dnszones / PTR | Non | Non |
> | dnszones / recordsets | Non | Non |
> | dnszones / SOA | Non | Non |
> | dnszones / SRV | Non | Non |
> | dnszones / TXT | Non | Non |
> | expressRouteCircuits | Oui | Oui |
> | expressRouteCrossConnections | Oui | Oui |
> | expressRouteGateways | Oui | Oui |
> | expressRoutePorts | Oui | Oui |
> | expressRouteServiceProviders | Non | Non |
> | firewallPolicies | Oui | Oui |
> | frontdoors | Oui, mais limitée (voir la [remarque ci-dessous](#frontdoor)) | Oui |
> | frontdoorWebApplicationFirewallManagedRuleSets | Oui, mais limitée (voir la [remarque ci-dessous](#frontdoor)) | Non |
> | frontdoorWebApplicationFirewallPolicies | Oui, mais limitée (voir la [remarque ci-dessous](#frontdoor)) | Oui |
> | getDnsResourceReference | Non | Non |
> | internalNotify | Non | Non |
> | ipGroups | Oui | Oui |
> | loadBalancers | Oui | Oui |
> | localNetworkGateways | Oui | Oui |
> | natGateways | Oui | Oui |
> | networkIntentPolicies | Oui | Oui |
> | networkInterfaces | Oui | Oui |
> | networkProfiles | Oui | Oui |
> | networkSecurityGroups | Oui | Oui |
> | networkWatchers | Oui | Oui |
> | networkWatchers / connectionMonitors | Oui | Non |
> | networkWatchers / flowLogs | Non | Non |
> | networkWatchers / lenses | Oui | Non |
> | networkWatchers / pingMeshes | Oui | Non |
> | p2sVpnGateways | Oui | Oui |
> | privateDnsOperationStatuses | Non | Non |
> | privateDnsZones | Oui | Oui |
> | privateDnsZones / A | Non | Non |
> | privateDnsZones / AAAA | Non | Non |
> | privateDnsZones / all | Non | Non |
> | privateDnsZones / CNAME | Non | Non |
> | privateDnsZones / MX | Non | Non |
> | privateDnsZones / PTR | Non | Non |
> | privateDnsZones / SOA | Non | Non |
> | privateDnsZones / SRV | Non | Non |
> | privateDnsZones / TXT | Non | Non |
> | privateDnsZones / virtualNetworkLinks | Oui | Oui |
> | privateEndpoints | Oui | Oui |
> | privateLinkServices | Oui | Oui |
> | publicIPAddresses | Oui | Oui |
> | publicIPPrefixes | Oui | Oui |
> | routeFilters | Oui | Oui |
> | routeTables | Oui | Oui |
> | serviceEndpointPolicies | Oui | Oui |
> | trafficManagerGeographicHierarchies | Non | Non |
> | trafficmanagerprofiles | Oui | Oui |
> | trafficmanagerprofiles/heatMaps | Non | Non |
> | trafficManagerUserMetricsKeys | Non | Non |
> | virtualHubs | Oui | Oui |
> | virtualNetworkGateways | Oui | Oui |
> | virtualNetworks | Oui | Oui |
> | virtualnetworks / subnets | Non | Non |
> | virtualNetworkTaps | Oui | Oui |
> | virtualWans | Oui | Non |
> | vpnGateways | Oui | Oui |
> | vpnSites | Oui | Oui |
> | webApplicationFirewallPolicies | Oui | Oui |

<a id="frontdoor"></a>

> [!NOTE]
> Pour Azure Front Door Service, vous pouvez appliquer des balises lors de la création de la ressource, mais la mise à jour ou l’ajout de balises n’est actuellement pas pris en charge.


## <a name="microsoftnotebooks"></a>Microsoft.Notebooks

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | NotebookProxies | Non | Non |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | espaces de noms | Oui | Non |
> | namespaces / notificationHubs | Oui | Non |

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | osNamespaces | Oui | Oui |

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | HyperVSites | Oui | Oui |
> | ImportSites | Oui | Oui |
> | MasterSites | Oui | Oui |
> | ServerSites | Oui | Oui |
> | VMwareSites | Oui | Oui |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | clusters | Oui | Oui |
> | deletedWorkspaces | Non | Non |
> | linkTargets | Non | Non |
> | storageInsightConfigs | Non | Non |
> | workspaces | Oui | Oui |
> | workspaces / dataExports | Non | Non |
> | workspaces / dataSources | Non | Non |
> | workspaces / linkedServices | Non | Non |
> | workspaces / linkedStorageAccounts | Non | Non |
> | workspaces / metadata | Non | Non |
> | workspaces / query | Non | Non |
> | workspaces / scopedPrivateLinkProxies | Non | Non |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | managementassociations | Non | Non |
> | managementconfigurations | Oui | Oui |
> | solutions | Oui | Oui |
> | views | Oui | Oui |

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | legacyPeerings | Non | Non |
> | peerAsns | Non | Non |
> | peerings | Oui | Oui |
> | peeringServiceCountries | Non | Non |
> | peeringServiceProviders | Non | Non |
> | peeringServices | Oui | Oui |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | attestations | Non | Non |
> | policyEvents | Non | Non |
> | policyMetadata | Non | Non |
> | policyStates | Non | Non |
> | policyTrackedResources | Non | Non |
> | remediations | Non | Non |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | consoles | Non | Non |
> | dashboards | Oui | Oui |
> | userSettings | Non | Non |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | privateLinkServicesForPowerBI | Oui | Oui |
> | tenants | Oui | Oui |
> | locataires / espaces de travail | Non | Non |
> | workspaceCollections | Oui | Oui |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | capacities | Oui | Oui |

## <a name="microsoftprojectbabylon"></a>Microsoft.ProjectBabylon

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accounts | Oui | Oui |
> | deletedAccounts | Non | Non |

## <a name="microsoftproviderhub"></a>Microsoft.ProviderHub

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | providerRegistrations | Non | Non |
> | providerRegistrations / defaultRollouts | Non | Non |
> | providerRegistrations / resourceTypeRegistrations | Non | Non |
> | rollouts | Oui | Oui |

## <a name="microsoftquantum"></a>Microsoft.Quantum

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | Workspaces | Oui | Oui |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | backupProtectedItems | Non | Non |
> | vaults | Oui | Oui |

## <a name="microsoftredhatopenshift"></a>Microsoft.RedHatOpenShift

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | OpenShiftClusters | Oui | Oui |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | espaces de noms | Oui | Oui |
> | namespaces / authorizationrules | Non | Non |
> | namespaces / hybridconnections | Non | Non |
> | namespaces / hybridconnections / authorizationrules | Non | Non |
> | namespaces / privateEndpointConnections | Non | Non |
> | namespaces / wcfrelays | Non | Non |
> | namespaces / wcfrelays / authorizationrules | Non | Non |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | queries | Oui | Oui |
> | resourceChangeDetails | Non | Non |
> | resourceChanges | Non | Non |
> | les ressources | Non | Non |
> | resourcesHistory | Non | Non |
> | subscriptionsStatus | Non | Non |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | availabilityStatuses | Non | Non |
> | childAvailabilityStatuses | Non | Non |
> | childResources | Non | Non |
> | emergingissues | Non | Non |
> | événements | Non | Non |
> | impactedResources | Non | Non |
> | metadata | Non | Non |
> | Notifications | Non | Non |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | calculateTemplateHash | Non | Non |
> | deployments | Oui | Non |
> | deployments / operations | Non | Non |
> | deploymentScripts | Oui | Oui |
> | deploymentScripts / logs | Non | Non |
> | liens | Non | Non |
> | notifyResourceJobs | Non | Non |
> | fournisseurs | Non | Non |
> | resourceGroups | Oui | Non |
> | subscriptions | Oui | Non |
> | templateSpecs | Oui | Oui |
> | templateSpecs / versions | Oui | Oui |
> | tenants | Non | Non |

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | applications | Oui | Oui |
> | saasresources | Non | Non |

## <a name="microsoftscvmm"></a>Microsoft.ScVmm

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | clouds | Oui | Oui |
> | VirtualMachines | Oui | Oui |
> | VirtualMachineTemplates | Oui | Oui |
> | VirtualNetworks | Oui | Oui |
> | vmmservers | Oui | Oui |

## <a name="microsoftsearch"></a>Microsoft.Search

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | resourceHealthMetadata | Non | Non |
> | searchServices | Oui | Oui |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | adaptiveNetworkHardenings | Non | Non |
> | advancedThreatProtectionSettings | Non | Non |
> | alertes | Non | Non |
> | alertsSuppressionRules | Non | Non |
> | allowedConnections | Non | Non |
> | applicationWhitelistings | Non | Non |
> | assessmentMetadata | Non | Non |
> | assessments | Non | Non |
> | autoDismissAlertsRules | Non | Non |
> | automations | Oui | Oui |
> | AutoProvisioningSettings | Non | Non |
> | Compliances | Non | Non |
> | connecteurs | Non | Non |
> | dataCollectionAgents | Non | Non |
> | deviceSecurityGroups | Non | Non |
> | discoveredSecuritySolutions | Non | Non |
> | externalSecuritySolutions | Non | Non |
> | InformationProtectionPolicies | Non | Non |
> | iotDefenderSettings | Non | Non |
> | iotSecuritySolutions | Oui | Oui |
> | iotSecuritySolutions / analyticsModels | Non | Non |
> | iotSecuritySolutions / analyticsModels / aggregatedAlerts | Non | Non |
> | iotSecuritySolutions / analyticsModels / aggregatedRecommendations | Non | Non |
> | iotSecuritySolutions / iotAlerts | Non | Non |
> | iotSecuritySolutions / iotAlertTypes | Non | Non |
> | iotSecuritySolutions / iotRecommendations | Non | Non |
> | iotSecuritySolutions / iotRecommendationTypes | Non | Non |
> | iotSensors | Non | Non |
> | jitNetworkAccessPolicies | Non | Non |
> | jitPolicies | Non | Non |
> | stratégies | Non | Non |
> | pricings | Non | Non |
> | regulatoryComplianceStandards | Non | Non |
> | regulatoryComplianceStandards / regulatoryComplianceControls | Non | Non |
> | regulatoryComplianceStandards / regulatoryComplianceControls / regulatoryComplianceAssessments | Non | Non |
> | secureScoreControlDefinitions | Non | Non |
> | secureScoreControls | Non | Non |
> | secureScores | Non | Non |
> | secureScores / secureScoreControls | Non | Non |
> | securityContacts | Non | Non |
> | securitySolutions | Non | Non |
> | securitySolutionsReferenceData | Non | Non |
> | securityStatuses | Non | Non |
> | securityStatusesSummaries | Non | Non |
> | serverVulnerabilityAssessments | Non | Non |
> | paramètres | Non | Non |
> | sqlVulnerabilityAssessments | Non | Non |
> | subAssessments | Non | Non |
> | tâches | Non | Non |
> | topologies | Non | Non |
> | workspaceSettings | Non | Non |

## <a name="microsoftsecuritygraph"></a>Microsoft.SecurityGraph

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | diagnosticSettings | Non | Non |
> | diagnosticSettingsCategories | Non | Non |

## <a name="microsoftsecurityinsights"></a>Microsoft.SecurityInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | aggregations | Non | Non |
> | alertRules | Non | Non |
> | alertRuleTemplates | Non | Non |
> | automationRules | Non | Non |
> | bookmarks | Non | Non |
> | cas | Non | Non |
> | dataConnectors | Non | Non |
> | dataConnectorsCheckRequirements | Non | Non |
> | entities | Non | Non |
> | entityQueries | Non | Non |
> | incidents | Non | Non |
> | officeConsents | Non | Non |
> | paramètres | Non | Non |
> | threatIntelligence | Non | Non |
> | watchlists | Non | Non |

## <a name="microsoftserialconsole"></a>Microsoft.SerialConsole

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | consoleServices | Non | Non |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | espaces de noms | Oui | Oui |
> | namespaces / authorizationrules | Non | Non |
> | namespaces / disasterrecoveryconfigs | Non | Non |
> | namespaces / eventgridfilters | Non | Non |
> | namespaces / networkrulesets | Non | Non |
> | namespaces / privateEndpointConnections | Non | Non |
> | namespaces / queues | Non | Non |
> | namespaces / queues / authorizationrules | Non | Non |
> | namespaces / topics | Non | Non |
> | namespaces / topics / authorizationrules | Non | Non |
> | namespaces / topics / subscriptions | Non | Non |
> | namespaces / topics / subscriptions / rules | Non | Non |
> | premiumMessagingRegions | Non | Non |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | applications | Oui | Oui |
> | clusters | Oui | Oui |
> | clusters / applications | Non | Non |
> | containerGroups | Oui | Oui |
> | containerGroupSets | Oui | Oui |
> | edgeclusters | Oui | Oui |
> | edgeclusters / applications | Non | Non |
> | managedclusters | Oui | Oui |
> | managedclusters / nodetypes | Non | Non |
> | networks | Oui | Oui |
> | secretstores | Oui | Oui |
> | secretstores / certificates | Non | Non |
> | secretstores / secrets | Non | Non |
> | volumes | Oui | Oui |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | applications | Oui | Oui |
> | containerGroups | Oui | Oui |
> | gateways | Oui | Oui |
> | networks | Oui | Oui |
> | secrets | Oui | Oui |
> | volumes | Oui | Oui |

## <a name="microsoftservices"></a>Microsoft.Services

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | providerRegistrations | Non | Non |
> | providerRegistrations / resourceTypeRegistrations | Non | Non |
> | rollouts | Oui | Oui |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | SignalR | Oui | Oui |
> | SignalR / eventGridFilters | Non | Non |

## <a name="microsoftsingularity"></a>Microsoft.Singularity

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accounts | Oui | Oui |
> | accounts / accountQuotaPolicies | Non | Non |
> | accounts / groupPolicies | Non | Non |
> | accounts / jobs | Non | Non |
> | accounts / storageContainers | Non | Non |

## <a name="microsoftsoftwareplan"></a>Microsoft.SoftwarePlan

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | hybridUseBenefits | Non | Non |

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | applicationDefinitions | Oui | Oui |
> | applications | Oui | Oui |
> | jitRequests | Oui | Oui |


## <a name="microsoftsql"></a>Microsoft.SQL

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | managedInstances | Oui | Oui |
> | managedInstances / databases | Oui (voir la [remarque ci-dessous](#sqlnote)) | Oui |
> | managedInstances / databases / backupShortTermRetentionPolicies | Non | Non |
> | managedInstances / databases / schemas / tables / columns / sensitivityLabels | Non | Non |
> | managedInstances / databases / vulnerabilityAssessments | Non | Non |
> | managedInstances / databases / vulnerabilityAssessments / rules / baselines | Non | Non |
> | managedInstances / encryptionProtector | Non | Non |
> | managedInstances / keys | Non | Non |
> | managedInstances / restorableDroppedDatabases / backupShortTermRetentionPolicies | Non | Non |
> | managedInstances / vulnerabilityAssessments | Non | Non |
> | servers | Oui | Oui |
> | servers / administrators | Non | Non |
> | servers / communicationLinks | Non | Non |
> | servers / databases | Oui (voir la [remarque ci-dessous](#sqlnote)) | Oui |
> | servers / encryptionProtector | Non | Non |
> | servers / firewallRules | Non | Non |
> | servers / keys | Non | Non |
> | servers / restorableDroppedDatabases | Non | Non |
> | servers / serviceobjectives | Non | Non |
> | servers / tdeCertificates | Non | Non |
> | virtualClusters | Non | Non |

<a id="sqlnote"></a>

> [!NOTE]
> La base de données MASTER ne prend pas en charge les balises, à la différence d’autres bases de données, comme les bases de données Azure Synapse Analytics. Les bases de données Azure Synapse Analytics doivent avoir l’état Actif (pas en pause).

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | SqlVirtualMachineGroups | Oui | Oui |
> | SqlVirtualMachineGroups / AvailabilityGroupListeners | Non | Non |
> | SqlVirtualMachines | Oui | Oui |

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | deletedAccounts | Non | Non |
> | storageAccounts | Oui | Oui |
> | storageAccounts / blobServices | Non | Non |
> | storageAccounts / fileServices | Non | Non |
> | storageAccounts / queueServices | Non | Non |
> | storageAccounts / services | Non | Non |
> | storageAccounts / services / metricDefinitions | Non | Non |
> | storageAccounts / tableServices | Non | Non |
> | usages | Non | Non |

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | caches | Oui | Oui |
> | caches / storageTargets | Non | Non |
> | usageModels | Non | Non |

## <a name="microsoftstoragereplication"></a>Microsoft.StorageReplication

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | replicationGroups | Non | Non |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | storageSyncServices | Oui | Oui |
> | storageSyncServices / registeredServers | Non | Non |
> | storageSyncServices / syncGroups | Non | Non |
> | storageSyncServices / syncGroups / cloudEndpoints | Non | Non |
> | storageSyncServices / syncGroups / serverEndpoints | Non | Non |
> | storageSyncServices / workflows | Non | Non |

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | storageSyncServices | Oui | Oui |
> | storageSyncServices / registeredServers | Non | Non |
> | storageSyncServices / syncGroups | Non | Non |
> | storageSyncServices / syncGroups / cloudEndpoints | Non | Non |
> | storageSyncServices / syncGroups / serverEndpoints | Non | Non |
> | storageSyncServices / workflows | Non | Non |

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | storageSyncServices | Oui | Oui |
> | storageSyncServices / registeredServers | Non | Non |
> | storageSyncServices / syncGroups | Non | Non |
> | storageSyncServices / syncGroups / cloudEndpoints | Non | Non |
> | storageSyncServices / syncGroups / serverEndpoints | Non | Non |
> | storageSyncServices / workflows | Non | Non |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | managers | Oui | Oui |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | clusters | Oui | Oui |
> | clusters / privateEndpoints | Non | Non |
> | streamingjobs | Oui (voir la remarque ci-dessous) | Oui |

> [!NOTE]
> Vous ne pouvez pas ajouter une balise lorsque streamingjobs est en cours d’exécution. Arrêtez la ressource pour pouvoir ajouter une balise.

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | acceptChangeTenant | Non | Non |
> | aliases | Non | Non |
> | annuler | Non | Non |
> | changeTenantRequest | Non | Non |
> | changeTenantStatus | Non | Non |
> | CreateSubscription | Non | Non |
> | enable | Non | Non |
> | renommer | Non | Non |
> | SubscriptionDefinitions | Non | Non |
> | SubscriptionOperations | Non | Non |
> | subscriptions | Non | Non |

## <a name="microsoftsynapse"></a>Microsoft.Synapse

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | privateLinkHubs | Oui | Oui |
> | workspaces | Oui | Oui |
> | workspaces / bigDataPools | Oui | Oui |
> | workspaces / operationStatuses | Non | Non |
> | workspaces / sqlDatabases | Oui | Oui |
> | workspaces / sqlPools | Oui | Oui |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | environments | Oui | Non |
> | environments / accessPolicies | Non | Non |
> | environments / eventsources | Oui | Non |
> | environments / referenceDataSets | Oui | Non |

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | stores | Oui | Oui |
> | stores / accessPolicies | Non | Non |
> | stores / services | Non | Non |
> | stores / services / tokens | Non | Non |

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | imageTemplates | Oui | Oui |
> | imageTemplates / runOutputs | Non | Non |

## <a name="microsoftvmware"></a>Microsoft.VMware

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | ArcZones | Oui | Oui |
> | ResourcePools | Oui | Oui |
> | VCenters | Oui | Oui |
> | VirtualMachines | Oui | Oui |
> | VirtualMachineTemplates | Oui | Oui |
> | VirtualNetworks | Oui | Oui |

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | dedicatedCloudNodes | Oui | Oui |
> | dedicatedCloudServices | Oui | Oui |
> | virtualMachines | Oui | Oui |

## <a name="microsoftvnfmanager"></a>Microsoft.VnfManager

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | périphériques | Oui | Oui |
> | registeredSubscriptions | Non | Non |
> | vendors | Non | Non |
> | vendors / skus | Non | Non |
> | vendors / vnfs | Non | Non |
> | virtualNetworkFunctionSkus | Non | Non |
> | vnfs | Oui | Oui |

## <a name="microsoftvsonline"></a>Microsoft.VSOnline

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | accounts | Oui | Non |
> | plans | Oui | Non |
> | registeredSubscriptions | Non | Non |

## <a name="microsoftweb"></a>Microsoft.Web

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | apiManagementAccounts | Non | Non |
> | apiManagementAccounts / apiAcls | Non | Non |
> | apiManagementAccounts / apis | Non | Non |
> | apiManagementAccounts / apis / apiAcls | Non | Non |
> | apiManagementAccounts / apis / connectionAcls | Non | Non |
> | apiManagementAccounts / apis / connections | Non | Non |
> | apiManagementAccounts / apis / connections / connectionAcls | Non | Non |
> | apiManagementAccounts / apis / localizedDefinitions | Non | Non |
> | apiManagementAccounts / connectionAcls | Non | Non |
> | apiManagementAccounts / connections | Non | Non |
> | billingMeters | Non | Non |
> | certificates | Oui | Oui |
> | connectionGateways | Oui | Oui |
> | connections | Oui | Oui |
> | customApis | Oui | Oui |
> | deletedSites | Non | Non |
> | hostingEnvironments | Oui | Oui |
> | hostingEnvironments / eventGridFilters | Non | Non |
> | hostingEnvironments / multiRolePools | Non | Non |
> | hostingEnvironments / workerPools | Non | Non |
> | kubeEnvironments | Oui | Oui |
> | publishingUsers | Non | Non |
> | de films | Non | Non |
> | resourceHealthMetadata | Non | Non |
> | runtimes | Non | Non |
> | serverFarms | Oui | Oui |
> | serverFarms / eventGridFilters | Non | Non |
> | serverFarms / firstPartyApps | Non | Non |
> | serverFarms / firstPartyApps / keyVaultSettings | Non | Non |
> | sites | Oui | Oui |
> | sites / config  | Non | Non |
> | sites / eventGridFilters | Non | Non |
> | sites / hostNameBindings | Non | Non |
> | sites / networkConfig | Non | Non |
> | sites / premieraddons | Oui | Oui |
> | sites / slots | Oui | Oui |
> | sites / slots / eventGridFilters | Non | Non |
> | sites / slots / hostNameBindings | Non | Non |
> | sites / slots / networkConfig | Non | Non |
> | sourceControls | Non | Non |
> | staticSites | Oui | Oui |
> | validate | Non | Non |
> | verifyHostingEnvironmentVnet | Non | Non |

## <a name="microsoftwindowsdefenderatp"></a>Microsoft.WindowsDefenderATP

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | diagnosticSettings | Non | Non |
> | diagnosticSettingsCategories | Non | Non |

## <a name="microsoftwindowsesu"></a>Microsoft.WindowsESU

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | multipleActivationKeys | Oui | Oui |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | DeviceServices | Oui | Oui |

## <a name="microsoftworkloadbuilder"></a>Microsoft.WorkloadBuilder

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | workloads | Oui | Oui |
> | workloads / instances | Non | Non |
> | workloads / versions | Non | Non |
> | workloads / versions / artifacts | Non | Non |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tableFixed"]
> | Type de ressource | Prend en charge les étiquettes | Balise dans le rapport des coûts |
> | ------------- | ----------- | ----------- |
> | components | Non | Non |
> | componentsSummary | Non | Non |
> | monitorInstances | Non | Non |
> | monitorInstancesSummary | Non | Non |
> | monitors | Non | Non |
> | notificationSettings | Non | Non |

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment appliquer des étiquettes aux ressources, consultez [Utiliser des étiquettes pour organiser vos ressources Azure](tag-resources.md).
