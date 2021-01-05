---
title: Restrictions concernant le nommage des ressources
description: Affiche les règles et restrictions concernant le nommage des ressources Azure.
ms.topic: conceptual
ms.date: 12/04/2020
ms.openlocfilehash: a6aac03277a11cf671f5a618bc85ff987cfdc2dd
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608314"
---
# <a name="naming-rules-and-restrictions-for-azure-resources"></a>Règles de nommage et restrictions pour les ressources Azure

Cet article récapitule les règles et restrictions concernant le nommage des ressources Azure. Pour obtenir des recommandations sur la façon de nommer des ressources, consultez [Conventions de nommage et de catégorisation recommandées](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging).

Cet article répertorie les ressources par espace de noms de fournisseur de ressources. Pour obtenir une liste qui mappe les fournisseurs de ressources et les services Azure, voir [Fournisseurs de ressources pour les services Azure](azure-services-resource-providers.md).

Les noms de ressources ne respectent pas la casse, sauf indication contraire dans la colonne relative aux caractères valides.

Dans les tableaux suivants, le terme « alphanumérique » fait référence aux caractères suivants :

* **a** à **z** (lettres minuscules)
* **A** à **Z** (lettres majuscules)
* **0** à **9** (chiffres)

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | servers | resource group | 3-63 | Lettres minuscules et chiffres.<br><br>Doit commencer par une lettre minuscule. |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | service | globale | 1-50 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par une lettre et se terminer par un caractère alphanumérique. |
> | service / apis | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / apis / issues | api | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / apis / issues / attachments | problème | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / apis / issues / comments | problème | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / apis / operations | api | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / apis / operations / tags | opération | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / apis / releases | api | 1-80 | Caractères alphanumériques, traits de soulignement et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | service / apis / schemas | api | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / apis / tagDescriptions | api | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / apis / tags | api | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / api-version-sets | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / authorizationServers | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / backends | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / certificates | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / diagnostics | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / groups | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / groups / users | group | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / identityProviders | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / loggers | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / notifications | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / notifications / recipientEmails | notification | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / openidConnectProviders | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / policies | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / products | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / products / apis | product | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / products / groups | product | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / products / tags | product | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / properties | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / subscriptions | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / tags | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / templates | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |
> | service / users | service | 1-256 | Impossibilité d’utiliser :<br> `*#&+:<>?` |

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | configurationStores | resource group | 5 à 50 | Caractères alphanumériques, traits de soulignement et traits d’union. |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | locks | étendue de l’affectation | 1-90 | Caractères alphanumériques, points, traits de soulignement, traits d’union et parenthèses.<br><br>Ne peut pas se terminer par un point. |
> | policyAssignments | étendue de l’affectation | 1-128 (nom d’affichage)<br><br>1-64 (nom de la ressource) | Le nom d’affichage peut contenir n’importe quel caractère.<br><br>Le nom de la ressource ne peut pas contenir `%` et ne peut pas se terminer par un point ou un espace. |
> | policyDefinitions | étendue de la définition | 1-128 (nom d’affichage)<br><br>1-64 (nom de la ressource) | Le nom d’affichage peut contenir n’importe quel caractère.<br><br>Le nom de la ressource ne peut pas contenir `%` et ne peut pas se terminer par un point ou un espace. |
> | policySetDefinitions | étendue de la définition | 1-128 (nom d’affichage)<br><br>1-64 (nom de la ressource)<br><br>1-24 (nom de la ressource dans l’étendue du groupe d’administration) | Le nom d’affichage peut contenir n’importe quel caractère.<br><br>Le nom de la ressource ne peut pas contenir `%` et ne peut pas se terminer par un point ou un espace.  |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | automationAccounts | resource group | 6-50 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par une lettre et se terminer par un caractère alphanumérique. |
> | automationAccounts / certificates | compte Automation | 1-128 | Impossibilité d’utiliser :<br> `<>*%&:\?.+/` <br><br>Ne peut pas se terminer par un espace.  |
> | automationAccounts / connections | compte Automation | 1-128 | Impossibilité d’utiliser :<br> `<>*%&:\?.+/` <br><br>Ne peut pas se terminer par un espace. |
> | automationAccounts / credentials | compte Automation | 1-128 | Impossibilité d’utiliser :<br> `<>*%&:\?.+/` <br><br>Ne peut pas se terminer par un espace. |
> | automationAccounts / runbooks | compte Automation | 1-63 | Caractères alphanumériques, traits de soulignement et traits d’union.<br><br>Doit commencer par une lettre.  |
> | automationAccounts / schedules | compte Automation | 1-128 | Impossibilité d’utiliser :<br> `<>*%&:\?.+/` <br><br>Ne peut pas se terminer par un espace. |
> | automationAccounts / variables | compte Automation | 1-128 | Impossibilité d’utiliser :<br> `<>*%&:\?.+/` <br><br>Ne peut pas se terminer par un espace. |
> | automationAccounts / watchers | compte Automation | 1-63 |  Caractères alphanumériques, traits de soulignement et traits d’union.<br><br>Doit commencer par une lettre. |
> | automationAccounts / webhooks | compte Automation | 1-128 | Impossibilité d’utiliser :<br> `<>*%&:\?.+/` <br><br>Ne peut pas se terminer par un espace. |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | batchAccounts | Région | 3-24 | Lettres minuscules et chiffres. |
> | batchAccounts / applications | compte Batch | 1-64 | Caractères alphanumériques, traits de soulignement et traits d’union. |
> | batchAccounts / certificates | compte Batch | 5-45 | Caractères alphanumériques, traits de soulignement et traits d’union. |
> | batchAccounts / pools | compte Batch | 1-64 | Caractères alphanumériques, traits de soulignement et traits d’union. |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | blockchainMembers | globale | 2-20 | Lettres minuscules et chiffres.<br><br>Doit commencer par une lettre minuscule. |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | botServices | globale | 2-64 |  Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. |
> | botServices / channels | service Bot | 2-64 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. |
> | botServices / Connections | service Bot | 2-64 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. |
> | enterpriseChannels | resource group | 2-64 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | Redis | globale | 1-63 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique. Traits d’union consécutifs non autorisés. |
> | Redis / firewallRules | Redis | 1-256 | Caractères alphanumériques |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | profiles | resource group | 1-260 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | profiles / endpoints | globale | 1-50 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | certificateOrders | resource group | 3-30 | Caractères alphanumériques. |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | accounts | resource group | 2-64 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | availabilitySets | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | diskEncryptionSets | resource group | 1-80 | Caractères alphanumériques et traits de soulignement. |
> | disks | resource group | 1-80 | Caractères alphanumériques et traits de soulignement. |
> | galleries | resource group | 1-80 | Caractères alphanumériques et points.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | galleries / applications | galerie | 1-80 | Caractères alphanumériques, traits d’union et points.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | galleries / applications/versions | application | Entier de 32 bits | Chiffres et points. |
> | galleries / images | galerie | 1-80 | Caractères alphanumériques, traits d’union et points.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | galleries / images / versions | image | Entier de 32 bits | Chiffres et points. |
> | images | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | snapshots | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | virtualMachines | resource group | 1-15 (Windows)<br>1-64 (Linux)<br><br>Voir la remarque ci-dessous. | Impossible d’utiliser un espace ou ces caractères :<br> `\/"'[]:|<>+=;,?*@&_`<br><br>Les machines virtuelles Windows ne peuvent pas inclure de point ni se terminer par un tiret.<br><br>Les machines virtuelles Linux ne peuvent pas se terminer par un point ou un trait d’union. |
> | virtualMachineScaleSets | resource group | 1-15 (Windows)<br>1-64 (Linux)<br><br>Voir la remarque ci-dessous. | Impossible d’utiliser un espace ou ces caractères :<br> `\/"'[]:|<>+=;,?*@&`<br><br>Ne peut pas commencer par un trait de soulignement. Ne peut pas se terminer par un point ou un trait d’union. |

> [!NOTE]
> Les machines virtuelles Azure portent deux noms distincts : un nom de ressource et un nom d’hôte. Quand vous créez une machine virtuelle dans le portail, la même valeur est utilisée pour les deux noms. Les restrictions mentionnées dans le tableau précédent concernent le nom d’hôte. Le nom de ressource proprement dit peut comporter jusqu’à 64 caractères.

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | containerGroups | resource group | 1-63 | Lettres minuscules, chiffres et traits d’union.<br><br>Ne peut pas commencer ou se terminer par un trait d’union. Les traits d’union consécutifs ne sont pas autorisés. |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | registries | globale | 5 à 50 | Caractères alphanumériques. |
> | registries / buildTasks | Registre | 5 à 50 | Caractères alphanumériques. |
> | registries / buildTasks/steps | tâche de génération | 5 à 50 | Caractères alphanumériques. |
> | registries / replications | Registre | 5 à 50 | Caractères alphanumériques. |
> | registries / scopeMaps | Registre | 5 à 50 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | registries / tasks | Registre | 5 à 50 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | registries / tokens | Registre | 5 à 50 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | registries / webhooks | Registre | 5 à 50 | Caractères alphanumériques. |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | managedclusters | resource group | 1-63 | Caractères alphanumériques, traits de soulignement et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | openShiftManagedClusters | resource group | 1-30 | Caractères alphanumériques. |

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | hubs | resource group | 1-64 | Caractères alphanumériques.<br><br>Doit commencer par une lettre.  |
> | hubs / authorizationPolicies | hub | 1-50 | Caractères alphanumériques, traits de soulignement et points.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | hubs / connectors | hub | 1-128 | Caractères alphanumériques et traits de soulignement.<br><br>Doit commencer par une lettre. |
> | hubs / connectors/mappings | connecteur | 1-128 | Caractères alphanumériques et traits de soulignement.<br><br>Doit commencer par une lettre. |
> | hubs / interactions | hub | 1-128 | Caractères alphanumériques et traits de soulignement.<br><br>Doit commencer par une lettre. |
> | hubs / kpi | hub | 1-512 | Caractères alphanumériques et traits de soulignement.<br><br>Doit commencer par une lettre. |
> | hubs / links | hub | 1-512 | Caractères alphanumériques et traits de soulignement.<br><br>Doit commencer par une lettre. |
> | hubs / predictions | hub | 1-512 | Caractères alphanumériques et traits de soulignement.<br><br>Doit commencer par une lettre. |
> | hubs / profiles | hub | 1-128 | Caractères alphanumériques et traits de soulignement.<br><br>Doit commencer par une lettre. |
> | hubs / relationshipLinks | hub | 1-512 | Caractères alphanumériques et traits de soulignement.<br><br>Doit commencer par une lettre. |
> | hubs / relationships | hub | 1-512 | Caractères alphanumériques et traits de soulignement.<br><br>Doit commencer par une lettre. |
> | hubs / roleAssignments | hub | 1-128 | Caractères alphanumériques et traits de soulignement.<br><br>Doit commencer par une lettre. |
> | hubs / views | hub | 1-512 | Caractères alphanumériques et traits de soulignement.<br><br>Doit commencer par une lettre. |

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | associations | resource group | 1-180 | Impossibilité d’utiliser :<br>`%&\\?/`<br><br>Ne peut pas se terminer par un point ou un espace. |
> | resourceProviders | resource group | 3-64 | Impossibilité d’utiliser :<br>`%&\\?/`<br><br>Ne peut pas se terminer par un point ou un espace. |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | jobs | resource group | 3-24 | Caractères alphanumériques, traits d’union, traits de soulignement et points. |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | workspaces | resource group | 3-30 | Caractères alphanumériques, traits de soulignement et traits d’union |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | factories | globale | 3-63 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | factories / dataflows | usine | 1-260 | Impossibilité d’utiliser :<br>`<>*#.%&:\\+?/`<br><br>Doit commencer par un caractère alphanumérique. |
> | factories / datasets | usine | 1-260 | Impossibilité d’utiliser :<br>`<>*#.%&:\\+?/`<br><br>Doit commencer par un caractère alphanumérique. |
> | factories / integrationRuntimes | usine | 3-63 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | factories / linkedservices | usine | 1-260 | Impossibilité d’utiliser :<br>`<>*#.%&:\\+?/`<br><br>Doit commencer par un caractère alphanumérique. |
> | factories / pipelines | usine | 1-260 | Impossibilité d’utiliser :<br>`<>*#.%&:\\+?/`<br><br>Doit commencer par un caractère alphanumérique. |
> | factories / triggers | usine | 1-260 | Impossibilité d’utiliser :<br>`<>*#.%&:\\+?/`<br><br>Doit commencer par un caractère alphanumérique. |
> | factories / triggers / rerunTriggers | déclencheur | 1-260 | Impossibilité d’utiliser :<br>`<>*#.%&:\\+?/`<br><br>Doit commencer par un caractère alphanumérique. |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | accounts | globale | 3-24 | Lettres minuscules et chiffres. |
> | accounts / computePolicies | account | 3-60 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | accounts / dataLakeStoreAccounts | account | 3-24 | Lettres minuscules et chiffres. |
> | accounts / firewallRules | account | 3-50 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | accounts / storageAccounts | account | 3-60 | Caractères alphanumériques, traits d’union et traits de soulignement. |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | accounts | globale | 3-24 | Lettres minuscules et chiffres. |
> | accounts / firewallRules | account | 3-50 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | accounts / virtualNetworkRules | account | 3-50 | Caractères alphanumériques, traits d’union et traits de soulignement. |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | services | resource group | 2-62 | Caractères alphanumériques, traits d’union, points et traits de soulignement.<br><br>Doit commencer par un caractère alphanumérique. |
> | services / projects | service | 2-57 | Caractères alphanumériques, traits d’union, points et traits de soulignement.<br><br>Doit commencer par un caractère alphanumérique. |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | servers | globale | 3-63 | Lettres minuscules, traits d’union et chiffres.<br><br>Ne peut pas commencer ou se terminer par un trait d’union. |
> | servers / databases | servers | 1-63 | Caractères alphanumériques et traits d’union. |
> | servers / firewallRules | servers | 1-128 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | servers / virtualNetworkRules | servers | 1-128 | Caractères alphanumériques et traits d’union. |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | servers | globale | 3-63 | Lettres minuscules, traits d’union et chiffres.<br><br>Ne peut pas commencer ou se terminer par un trait d’union. |
> | servers / databases | servers | 1-63 | Caractères alphanumériques et traits d’union. |
> | servers / firewallRules | servers | 1-128 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | servers / virtualNetworkRules | servers | 1-128 | Caractères alphanumériques et traits d’union. |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | servers | globale | 3-63 | Lettres minuscules, traits d’union et chiffres.<br><br>Ne peut pas commencer ou se terminer par un trait d’union. |
> | servers / databases | servers | 1-63 | Caractères alphanumériques et traits d’union. |
> | servers / firewallRules | servers | 1-128 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | servers / virtualNetworkRules | servers | 1-128 | Caractères alphanumériques et traits d’union. |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | IoTHubs | globale | 3-50 | Caractères alphanumériques et traits d’union.<br><br>Ne peut pas se terminer par un trait d’union. |
> | IotHubs / certificates | hub IOT | 1-64 | Caractères alphanumériques, traits d’union, points et traits de soulignement. |
> | IotHubs / eventHubEndpoints / ConsumerGroups | eventHubEndpoints | 1-50 | Caractères alphanumériques, traits d’union, points et traits de soulignement. |
> | provisioningServices | resource group | 3-64 | Caractères alphanumériques et traits d’union.<br><br>Doit se terminer par un caractère alphanumérique. |
> | provisioningServices / certificates | provisioningServices | 1-64 | Caractères alphanumériques, traits d’union, points et traits de soulignement. |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | labs | resource group | 1-50 | Caractères alphanumériques, traits de soulignement et traits d’union. |
> | labs / customimages | labo | 1-80 | Caractères alphanumériques, traits de soulignement, traits d’union et parenthèses. |
> | labs / formulas | labo | 1-80 | Caractères alphanumériques, traits de soulignement, traits d’union et parenthèses. |
> | labs / virtualmachines | labo | 1-15 (Windows)<br>1-64 (Linux) | Caractères alphanumériques et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique. Ne peut pas être constitué uniquement de chiffres. |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | databaseAccounts | globale | 3-44 | Lettres minuscules, chiffres et traits d’union.<br><br>Doit commencer par une lettre minuscule ou un chiffre. |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | domaines | resource group | 3-50 | Caractères alphanumériques et traits d’union. |
> | domains / topics | domaine | 3-50 | Caractères alphanumériques et traits d’union. |
> | eventSubscriptions | resource group | 3-64 | Caractères alphanumériques et traits d’union. |
> | topics | resource group | 3-50 | Caractères alphanumériques et traits d’union. |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | clusters | resource group | 6-50 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par une lettre. Doit se terminer par une lettre ou un chiffre. |
> | espaces de noms | globale | 6-50 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par une lettre. Doit se terminer par une lettre ou un chiffre. |
> | namespaces / AuthorizationRules | espace de noms | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par une lettre ou un chiffre. |
> | namespaces / disasterRecoveryConfigs | espace de noms | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par une lettre ou un chiffre. |
> | namespaces / eventhubs | espace de noms | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par une lettre ou un chiffre. |
> | namespaces / eventhubs / authorizationRules | hub d’événements | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par une lettre ou un chiffre. |
> | namespaces / eventhubs / consumergroups | hub d’événements | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par une lettre ou un chiffre. |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | clusters | globale | 3-59 | Caractères alphanumériques et traits d’union<br><br>Doit commencer et se terminer par une lettre ou un chiffre. |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | jobs | resource group | 2-64 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par une lettre. |

## <a name="microsoftinsights"></a>Microsoft.Insights

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | actionGroups | resource group | 1-260 | Impossibilité d’utiliser :<br>`/&%\?` <br><br>Ne peut pas se terminer par un espace ou un point.  |
> | components | resource group | 1-260 | Impossibilité d’utiliser :<br>`%&\?/` <br><br>Ne peut pas se terminer par un espace ou un point.  |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | IoTApps | globale | 2-63 | Lettres minuscules, chiffres et traits d’union.<br><br>Doit commencer par une lettre minuscule ou un chiffre. |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | vaults | globale | 3-24 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par une lettre. Doit se terminer par une lettre ou un chiffre. Ne peut pas contenir des traits d’union consécutifs. |
> | vaults / secrets | Coffre | 1-127 | Caractères alphanumériques et traits d’union. |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | clusters | globale | 4-22 | Lettres minuscules et chiffres.<br><br>Doit commencer par une lettre. |
> | /clusters / databases | cluster | 1-260 | Caractères alphanumériques, traits d’union, espaces et points. |
> | /clusters / databases / dataConnections | database | 1-40 | Caractères alphanumériques, traits d’union, espaces et points. |
> | /clusters / databases / eventhubconnections | database | 1-40 | Caractères alphanumériques, traits d’union, espaces et points. |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | integrationAccounts | resource group | 1-80 | Caractères alphanumériques, traits d’union, traits de soulignement, points et parenthèses. |
> | integrationAccounts / assemblies | compte d’intégration | 1-80 | Caractères alphanumériques, traits d’union, traits de soulignement, points et parenthèses. |
> | integrationAccounts / batchConfigurations | compte d’intégration | 1-20 | Caractères alphanumériques. |
> | integrationAccounts / certificates | compte d’intégration | 1-80 | Caractères alphanumériques, traits d’union, traits de soulignement, points et parenthèses. |
> | integrationAccounts / maps | compte d’intégration | 1-80 | Caractères alphanumériques, traits d’union, traits de soulignement, points et parenthèses. |
> | integrationAccounts / partners | compte d’intégration | 1-80 | Caractères alphanumériques, traits d’union, traits de soulignement, points et parenthèses. |
> | integrationAccounts / rosettanetprocessconfigurations | compte d’intégration | 1-80 | Caractères alphanumériques, traits d’union, traits de soulignement, points et parenthèses. |
> | integrationAccounts / schemas | compte d’intégration | 1-80 | Caractères alphanumériques, traits d’union, traits de soulignement, points et parenthèses. |
> | integrationAccounts / sessions | compte d’intégration | 1-80 | Caractères alphanumériques, traits d’union, traits de soulignement, points et parenthèses. |
> | integrationServiceEnvironments | resource group | 1-80 | Caractères alphanumériques, traits d’union, points et traits de soulignement. |
> | integrationServiceEnvironments / managedApis | environnement de service d’intégration | 1-80 | Caractères alphanumériques, traits d’union, points et traits de soulignement. |
> | workflows | resource group | 1-80 | Caractères alphanumériques, traits d’union, traits de soulignement, points et parenthèses. |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | commitmentPlans | resource group | 1-260 | Impossibilité d’utiliser :<br>`<>*%&:?+/\\`<br><br>Ne peut pas se terminer par un espace. |
> | webServices | resource group | 1-260 | Impossibilité d’utiliser :<br>`<>*%&:?+/\\`<br><br>Ne peut pas se terminer par un espace. |
> | workspaces | resource group | 1-260 | Impossibilité d’utiliser :<br>`<>*%&:?+/\\`<br><br>Ne peut pas se terminer par un espace. |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | workspaces | resource group | 3-33 | Caractères alphanumériques et traits d’union. |
> | workspaces / computes | espace de travail | 2-16 | Caractères alphanumériques et traits d’union. |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | userAssignedIdentities | resource group | 3-128 | Caractères alphanumériques, traits d’union et traits de soulignement<br><br>Doit commencer par une lettre ou un chiffre. |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | accounts | resource group | 1-98 (pour le nom du groupe de ressources et le nom du compte) | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | mediaservices | resource group | 3-24 | Lettres minuscules et chiffres. |
> | mediaservices / liveEvents | service multimédia | 1-32 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. |
> | mediaservices / liveEvents / liveOutputs | événement en temps réel | 1-256 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. |
> | mediaservices / streamingEndpoints | service multimédia | 1-24 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. |

## <a name="microsoftnetwork"></a>Microsoft.Network

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | applicationGateways | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | applicationSecurityGroups | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | azureFirewalls | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | bastionHosts | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | connections | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | dnsZones | resource group | 1-63 caractères<br><br>2 à 34 étiquettes<br><br>Chaque étiquette est un ensemble de caractères séparés par un point. Par exemple **contoso.com** est composé de 2 étiquettes. | Chaque étiquette peut contenir des caractères alphanumériques, des traits de soulignement et des traits d’union.<br><br>Chaque étiquette est séparée par un point. |
> | expressRouteCircuits | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | firewallPolicies | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | firewallPolicies / ruleGroups | stratégie de pare-feu | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | frontDoors | globale | 5-64 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | frontdoorWebApplicationFirewallPolicies | resource group | 1-128 | Caractères alphanumériques.<br><br>Doit commencer par une lettre. |
> | loadBalancers | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | loadBalancers / inboundNatRules | équilibreur de charge | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | localNetworkGateways | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | networkInterfaces | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | networkSecurityGroups | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | networkSecurityGroups / securityRules | groupe de sécurité réseau | 1-80 |  Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | networkWatchers | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | privateDnsZones | resource group | 1-63 caractères<br><br>2 à 34 étiquettes<br><br>Chaque étiquette est un ensemble de caractères séparés par un point. Par exemple **contoso.com** est composé de 2 étiquettes. | Chaque étiquette peut contenir des caractères alphanumériques, des traits de soulignement et des traits d’union.<br><br>Chaque étiquette est séparée par un point. |
> | privateDnsZones / virtualNetworkLinks | zone DNS privée | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | publicIPAddresses | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | publicIPPrefixes | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | routeFilters | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | routeFilters / routeFilterRules | Filtre de routage | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | routeTables | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | routeTables / routes | table de routage | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | serviceEndpointPolicies | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | trafficmanagerprofiles | globale | 1-63 | Caractères alphanumériques, traits d’union et points.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | virtualNetworkGateways | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | virtualNetworks | resource group | 2-64 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | virtualnetworks / subnets | réseau virtuel | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | virtualNetworks / virtualNetworkPeerings | réseau virtuel | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | virtualWans | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | vpnGateways | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | vpnGateways / vpnConnections | passerelle VPN | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |
> | vpnSites | resource group | 1-80 | Caractères alphanumériques, traits de soulignement, points et traits d’union.<br><br>Doit commencer par un caractère alphanumérique. Doit se terminer par un caractère alphanumérique ou un trait de soulignement. |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | espaces de noms | globale | 6-50 | Caractères alphanumériques et traits d’union<br><br>Doit commencer par une lettre. Doit se terminer par un caractère alphanumérique. |
> | namespaces / AuthorizationRules | espace de noms | 1-256 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer par un caractère alphanumérique. |
> | namespaces / notificationHubs | espace de noms | 1-260 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer par un caractère alphanumérique. |
> | namespaces / notificationHubs / AuthorizationRules | hub de notification | 1-256 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer par un caractère alphanumérique. |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | clusters | resource group | 4-63 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | workspaces | resource group | 4-63 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | solutions | espace de travail | N/A | Pour les solutions créées par Microsoft, le nom doit suivre le modèle suivant :<br>`SolutionType(WorkspaceName)`<br><br>Pour les solutions créées par des tiers, le nom doit suivre le modèle suivant :<br>`SolutionType[WorkspaceName]`<br><br>Par exemple, un nom valide est :<br>`AntiMalware(contoso-IT)`<br><br>Le type de solution respecte la casse. |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | dashboards | resource group | 3-160 | Caractères alphanumériques et traits d’union.<br><br>Pour utiliser des caractères interdits, ajoutez une balise nommée **hidden-title** avec le nom du tableau de bord que vous souhaitez utiliser. Le portail affiche ce nom lors de l'affichage du tableau de bord. |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | workspaceCollections | region | 3-63 | Caractères alphanumériques et traits d’union.<br><br>Ne peut pas commencer par un tiret. Impossibilité d’utiliser des traits d’union consécutifs. |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | capacities | region | 3-63 | Lettres minuscules ou chiffres<br><br>Doit commencer par une lettre minuscule. |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | vaults | resource group | 2-50 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par une lettre. |
> | vaults / backupPolicies | coffre | 3-150 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par une lettre. Ne peut pas se terminer par un trait d’union. |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | espaces de noms | globale | 6-50 | Caractères alphanumériques et traits d’union.<br><br>Le nom doit commencer par une lettre. Doit se terminer par une lettre ou un chiffre. |
> | namespaces / AuthorizationRules | espace de noms | 1-50 |  Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | namespaces / HybridConnections | espace de noms | 1-260 | Caractères alphanumériques, points, traits d’union, traits de soulignement et barres obliques.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | namespaces / HybridConnections/authorizationRules | connexion hybride | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | namespaces / WcfRelays | espace de noms | 1-260 | Caractères alphanumériques, points, traits d’union, traits de soulignement et barres obliques.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | namespaces / WcfRelays / authorizationRules | relais WCF | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | deployments | resource group | 1-64 | Caractères alphanumériques, traits de soulignement, parenthèses, traits d’union et points. |
> | resourcegroups | subscription | 1-90 | Caractères alphanumériques, traits de soulignement, parenthèses, traits d’union, points et caractères Unicode correspondant à la [documentation sur l’expression régulière](/rest/api/resources/resourcegroups/createorupdate).<br><br>Ne peut pas se terminer par un point. |
> | tagNames | resource | 1-512 | Impossibilité d’utiliser :<br>`<>%&\?/` |
> | tagNames / tagValues | nom d’étiquette | 1-256 | Tous les caractères. |
> | templateSpecs | resource group | 1-90 | Caractères alphanumériques, traits de soulignement, parenthèses, traits d’union et points. |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | espaces de noms | globale | 6-50 | Caractères alphanumériques et traits d’union.<br><br>Le nom doit commencer par une lettre. Doit se terminer par une lettre ou un chiffre.<br><br>Pour plus d’informations, consultez [Créer un espace de noms](/rest/api/servicebus/create-namespace). |
> | namespaces / AuthorizationRules | espace de noms | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | namespaces / disasterRecoveryConfigs | globale | 6-50 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par une lettre. Doit se terminer par un caractère alphanumérique. |
> | namespaces / migrationConfigurations | espace de noms |  | La valeur doit toujours être **$default**. |
> | namespaces / queues | espace de noms | 1-260 | Caractères alphanumériques, points, traits d’union, traits de soulignement et barres obliques.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | namespaces / queues / authorizationRules | queue | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | namespaces / topics | espace de noms | 1-260 | Caractères alphanumériques, points, traits d’union, traits de soulignement et barres obliques.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | namespaces / topics / authorizationRules | topic | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | namespaces / topics / subscriptions | topic | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |
> | namespaces / topics / subscriptions / rules | subscription | 1-50 | Caractères alphanumériques, points, traits d’union et traits de soulignement.<br><br>Doit commencer et se terminer par un caractère alphanumérique. |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | clusters | region | 4-23 | Lettres minuscules, chiffres et traits d’union.<br><br>Doit commencer par une lettre minuscule. Doit se terminer par une lettre minuscule ou un chiffre. |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | signalR | globale | 3-63 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par une lettre. Doit se terminer par une lettre ou un chiffre.  |

## <a name="microsoftsql"></a>Microsoft.Sql

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | managedInstances | globale | 1-63 | Lettres minuscules, chiffres et traits d’union.<br><br>Ne peut pas commencer ou se terminer par un trait d’union. |
> | servers | globale | 1-63 | Lettres minuscules, chiffres et traits d’union.<br><br>Ne peut pas commencer ou se terminer par un trait d’union. |
> | servers / administrators | server |  | Doit être `ActiveDirectory`. |
> | servers / databases | server | 1-128 | Impossibilité d’utiliser :<br>`<>*%&:\/?`<br><br>Ne peut pas se terminer par un point ou un espace. |
> | servers / databases / syncGroups | database | 1-150 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | servers / elasticPools | server | 1-128 | Impossibilité d’utiliser :<br>`<>*%&:\/?`<br><br>Ne peut pas se terminer par un point ou un espace. |
> | servers / failoverGroups | globale | 1-63 | Lettres minuscules, chiffres et traits d’union.<br><br>Ne peut pas commencer ou se terminer par un trait d’union. |
> | servers / firewallRules | server | 1-128 | Impossibilité d’utiliser :<br>`<>*%&:;\/?`<br><br>Ne peut pas se terminer par un point. |

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | storageAccounts | globale | 3-24 | Lettres minuscules et chiffres. |
> | storageAccounts / blobServices | compte de stockage |  | Doit être `default`. |
> | storageAccounts / blobServices / containers | compte de stockage | 3-63 | Lettres minuscules, chiffres et traits d’union.<br><br>Doit commencer par une lettre minuscule ou un chiffre. Impossibilité d’utiliser des traits d’union consécutifs. |
> | storageAccounts / fileServices | compte de stockage |  | Doit être `default`. |
> | storageAccounts / fileServices / shares | compte de stockage | 3-63 | Lettres minuscules, chiffres et traits d’union.<br><br>Ne peut pas commencer ou se terminer par un trait d’union. Impossibilité d’utiliser des traits d’union consécutifs. |
> | storageAccounts / managementPolicies | compte de stockage |  | Doit être `default`. |
> | objet BLOB | conteneur | 1-1024 | Tout caractère d’URL respectant la casse |
> | queue | compte de stockage | 3-63 | Lettres minuscules, chiffres et traits d’union.<br><br>Ne peut pas commencer ou se terminer par un trait d’union. Impossibilité d’utiliser des traits d’union consécutifs. |
> | table | compte de stockage | 3-63 | Caractères alphanumériques.<br><br>Doit commencer par une lettre. |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | storageSyncServices | resource group | 1-260 | Caractères alphanumériques, espaces, points, traits d’union et traits de soulignement.<br><br>Ne peut pas se terminer par un point ou un espace. |
> | storageSyncServices / syncGroups | service de synchronisation de stockage | 1-260 | Caractères alphanumériques, espaces, points, traits d’union et traits de soulignement.<br><br>Ne peut pas se terminer par un point ou un espace. |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | managers | resource group | 2-50 | Caractères alphanumériques et traits d’union.<br><br>Doit commencer par une lettre. Doit se terminer par un caractère alphanumérique. |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | streamingjobs | resource group | 3-63 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | streamingjobs / functions | tâche de streaming | 3-63 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | streamingjobs / inputs | tâche de streaming | 3-63 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | streamingjobs / outputs | tâche de streaming | 3-63 | Caractères alphanumériques, traits d’union et traits de soulignement. |
> | streamingjobs / transformations | tâche de streaming | 3-63 | Caractères alphanumériques, traits d’union et traits de soulignement. |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | environments | resource group | 1-90 | Impossibilité d’utiliser :<br>`'<>%&:\?/#` |
> | environments / accessPolicies | Environnement | 1-90 | Impossibilité d’utiliser :<br> `'<>%&:\?/#` |
> | environments / eventSources | Environnement | 1-90 | Impossibilité d’utiliser :<br>`'<>%&:\?/#` |
> | environments / referenceDataSets | Environnement | 3-63 | Caractères alphanumériques |

## <a name="microsoftweb"></a>Microsoft.Web

> [!div class="mx-tableFixed"]
> | Entité | Étendue | Longueur | Caractères valides |
> | --- | --- | --- | --- |
> | certificates | resource group | 1-260 | Impossibilité d’utiliser :<br>`/` <br><br>Ne peut pas se terminer par un espace ou un point.  | 
> | serverfarms | resource group | 1-40 | Caractères alphanumériques et traits d’union. |
> | sites | globale | 2-60 | Contient des caractères alphanumériques et des traits d’union.<br><br>Ne peut pas commencer ou se terminer par un trait d’union. |
> | sites / slots | site | 2-59 | Caractères alphanumériques et traits d’union. |

> [!NOTE]
> Azure Functions a les mêmes règles d’affectation de noms et restrictions que Microsoft.Web/sites.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des recommandations sur la façon de nommer des ressources, consultez [Prêt : Conventions de nommage et de catégorisation recommandées](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging).
