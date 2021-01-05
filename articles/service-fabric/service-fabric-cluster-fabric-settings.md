---
title: Modifier les paramètres de cluster Azure Service Fabric
description: Cet article décrit les paramètres de structure et les stratégies de mise à niveau de la structure que vous pouvez personnaliser.
ms.topic: reference
ms.date: 08/30/2019
ms.openlocfilehash: c055ad1dad8b9574c8d811284a34619ee3648a10
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97095268"
---
# <a name="customize-service-fabric-cluster-settings"></a>Personnaliser les paramètres de cluster Service Fabric
Cet article décrit les différents paramètres de structure personnalisables d’un cluster Service Fabric. Pour des clusters hébergés dans Azure, vous pouvez personnaliser les paramètres via le [portail Azure](https://portal.azure.com) ou en utilisant un modèle Azure Resource Manager. Pour plus d’informations, voir [Mettre à niveau la configuration d’un cluster Azure](service-fabric-cluster-config-upgrade-azure.md). Pour personnaliser les paramètres d’un cluster autonome, mettez à jour le fichier *ClusterConfig.json* et effectuez une mise à niveau de configuration sur le cluster. Pour plus d’informations, voir [Mettre à niveau la configuration d’un cluster autonome](service-fabric-cluster-config-upgrade-windows-server.md).

Il existe trois stratégies de mise à niveau différentes :

- **Dynamique** – Les modifications apportées à une configuration dynamique ne provoquent pas de redémarrage de processus, qu’il s’agisse des processus de Service Fabric ou de vos processus hôte de service. 
- **Statique** – Les modifications apportées à une configuration statique provoquent le redémarrage du nœud Service Fabric afin de consommer la modification. Les services sur les nœuds sont redémarrés.
- **Non autorisé** – Ces paramètres ne peuvent pas être modifiés. La modification de ces paramètres implique de détruire le cluster pour en créer un nouveau. 

Voici une liste des paramètres Fabric que vous pouvez personnaliser, classés par section.

## <a name="applicationgatewayhttp"></a>ApplicationGateway/Http

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|ApplicationCertificateValidationPolicy|Chaîne (valeur par défaut : "None")|statique| Cela ne valide pas le certificat de serveur ; succès de la requête. Reportez-vous à la configuration ServiceCertificateThumbprints pour obtenir la liste séparée par des virgules des empreintes de certificats distants auxquels le proxy inversé peut faire confiance. Reportez-vous à la configuration ServiceCommonNameAndIssuer pour le nom de l’objet et l’empreinte de l’émetteur des certificats distants auxquels le proxy inversé peut faire confiance. Pour plus d’informations, consultez [Connexion sécurisée du proxy inversé](service-fabric-reverseproxy-configure-secure-communication.md#secure-connection-establishment-between-the-reverse-proxy-and-services). |
|BodyChunkSize |Valeur Uint (valeur par défaut : 16384) |Dynamique| Indique la taille en octets du bloc utilisé pour lire le corps. |
|CrlCheckingFlag|uint, valeur par défaut : 0x40000000 |Dynamique| Indicateurs pour la validation de la chaîne de certificats de l’application/service, par exemple CRL checking 0x10000000 CERT_CHAIN_REVOCATION_CHECK_END_CERT 0x20000000 CERT_CHAIN_REVOCATION_CHECK_CHAIN 0x40000000 CERT_CHAIN_REVOCATION_CHECK_CHAIN_EXCLUDE_ROOT 0x80000000 CERT_CHAIN_REVOCATION_CHECK_CACHE_ONLY Si définie sur 0, cette valeur désactive la vérification CRL La liste complète des valeurs prises en charge est documentée par dwFlags de CertGetCertificateChain : https://msdn.microsoft.com/library/windows/desktop/aa376078(v=vs.85).aspx  |
|DefaultHttpRequestTimeout |Durée en secondes. La valeur par défaut est 120 |Dynamique|Spécifiez la durée en secondes.  Indique le délai d’expiration par défaut des requêtes HTTP traitées par la passerelle HTTP. |
|ForwardClientCertificate|valeur booléenne, valeur par défaut : FALSE|Dynamique|Quand la valeur définie est false, le proxy inversé ne demande pas le certificat client. Quand la valeur définie est true, il demande le certificat client pendant la négociation TLS et transfère la chaîne de format PEM encodée en base 64 au service dans un en-tête nommé X-Client-Certificate. Ce service peut faire échouer la requête avec le code d’état approprié après avoir inspecté les données du certificat. Si la valeur est TRUE et que le client ne présente pas de certificat, le proxy inversé transfère un en-tête vide et laisse le service gérer le cas. Le proxy inverse agit comme une couche transparente. Pour plus d’informations, consultez [Configurer l’authentification du certificat client](service-fabric-reverseproxy-configure-secure-communication.md#setting-up-client-certificate-authentication-through-the-reverse-proxy). |
|GatewayAuthCredentialType |Chaîne (valeur par défaut : "None") |statique| Indique le type d’informations d’identification de sécurité à utiliser au niveau du point de terminaison de passerelle d’application HTTP. Les valeurs valides sont None/X509. |
|GatewayX509CertificateFindType |Chaîne (valeur par défaut : "FindByThumbprint") |Dynamique| Indique comment rechercher un certificat dans le magasin spécifié par GatewayX509CertificateStoreName. Valeur prise en charge : FindByThumbprint; FindBySubjectName. |
|GatewayX509CertificateFindValue | Chaîne (valeur par défaut : "") |Dynamique| Valeur du filtre de recherche utilisée pour localiser le certificat de la passerelle d’application HTTP. Ce certificat est configuré sur le point d’extrémité HTTPS et peut également servir à vérifier l’identité de l’application au besoin par les services. Le paramètre FindValue est consulté en premier. S’il n’existe pas, FindValueSecondary est consulté. |
|GatewayX509CertificateFindValueSecondary | Chaîne (valeur par défaut : "") |Dynamique|Valeur du filtre de recherche utilisée pour localiser le certificat de la passerelle d’application HTTP. Ce certificat est configuré sur le point d’extrémité HTTPS et peut également servir à vérifier l’identité de l’application au besoin par les services. Le paramètre FindValue est consulté en premier. S’il n’existe pas, FindValueSecondary est consulté.|
|GatewayX509CertificateStoreName |Chaîne (valeur par défaut : "My") |Dynamique| Nom du magasin de certificats X.509 qui contient le certificat de la passerelle d’application HTTP. |
|HttpRequestConnectTimeout|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(5)|Dynamique|Spécifiez la durée en secondes.  Indique le délai d’expiration de la connexion des requêtes HTTP envoyées depuis la passerelle d’application HTTP.  |
|IgnoreCrlOfflineError|Valeur booléenne, valeur par défaut : TRUE|Dynamique|Indique s’il faut ou non ignorer l’erreur en mode hors connexion de la liste CRL pour la vérification du certificat de l’application/service. |
|IsEnabled |Valeur booléenne (valeur par défaut : false) |statique| Active/désactive le paramètre HttpApplicationGateway. Le paramètre Httpgateway est désactivé par défaut et cette configuration doit être définie pour l’activer. |
|NumberOfParallelOperations | Valeur Uint (par défaut : 5000) |statique|Nombre de lectures à publier dans la file d’attente du serveur HTTP. Ce paramètre contrôle le nombre de requêtes concurrentes que la passerelle HTTP peut satisfaire. |
|RemoveServiceResponseHeaders|chaîne, valeur par défaut : « Date; Server »|statique|Liste séparée par des points-virgules/virgules d’en-têtes de réponse qui sera supprimée de la réponse du service avant son transfert vers le client. Si cette valeur est une chaîne vide, transmettre tels quels tous les en-têtes retournés par le service. Par exemple, ne pas remplacer la date et le serveur |
|ResolveServiceBackoffInterval |Durée en secondes (valeur par défaut : 5) |Dynamique|Spécifiez la durée en secondes.  Indique l’interface d’interruption par défaut avant de retenter une opération du service de résolution ayant échoué. |
|SecureOnlyMode|valeur booléenne, valeur par défaut : FALSE|Dynamique| SecureOnlyMode: true : le proxy inversé transfère uniquement les demandes vers les services qui publient des points de terminaison sécurisés. false : le proxy inversé peut transférer les demandes vers des points de terminaison sécurisés/non sécurisés. Pour plus d’informations, consultez [Logique de sélection du point de terminaison proxy inversé](service-fabric-reverseproxy-configure-secure-communication.md#endpoint-selection-logic-when-services-expose-secure-as-well-as-unsecured-endpoints).  |
|ServiceCertificateThumbprints|Chaîne (valeur par défaut : "")|Dynamique|Liste séparée par des virgules des empreintes de certificats distants auxquels le proxy inversé peut faire confiance. Pour plus d’informations, consultez [Connexion sécurisée du proxy inversé](service-fabric-reverseproxy-configure-secure-communication.md#secure-connection-establishment-between-the-reverse-proxy-and-services). |

## <a name="applicationgatewayhttpservicecommonnameandissuer"></a>ApplicationGateway/Http/ServiceCommonNameAndIssuer

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|X509NameMap, valeur par défaut : None|Dynamique| Nom de l’objet et empreinte de l’émetteur des certificats distants auxquels le proxy inversé peut faire confiance. Pour plus d’informations, consultez [Connexion sécurisée du proxy inversé](service-fabric-reverseproxy-configure-secure-communication.md#secure-connection-establishment-between-the-reverse-proxy-and-services). |

## <a name="backuprestoreservice"></a>BackupRestoreService

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|MinReplicaSetSize|entier (valeur par défaut : 0)|statique|Paramètre MinReplicaSetSize pour BackupRestoreService |
|PlacementConstraints|chaîne, valeur par défaut : « »|statique|    Paramètre PlacementConstraints pour le service BackupRestore |
|SecretEncryptionCertThumbprint|chaîne, valeur par défaut : « »|Dynamique|Empreinte numérique du certificat X509 de chiffrement de secret |
|SecretEncryptionCertX509StoreName|chaîne, valeur recommandée « My » (pas de valeur par défaut) |    Dynamique|    Ce paramètre indique le certificat à utiliser pour le chiffrement et le déchiffrement des informations d’identification. Nom du magasin de certificats X.509 utilisé pour le chiffrement et le déchiffrement des informations d’identification de magasin qu’utilise le service de restauration de sauvegarde |
|TargetReplicaSetSize|entier, valeur par défaut : 0|statique| Paramètre TargetReplicaSetSize pour BackupRestoreService |

## <a name="clustermanager"></a>ClusterManager

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|AllowCustomUpgradeSortPolicies | Valeur booléenne (valeur par défaut : false) |Dynamique|Indique si les stratégies de tri de mise à niveau personnalisées sont autorisées ou non. Il est utilisé pour effectuer une mise à niveau en 2 phases qui active cette fonctionnalité. Service Fabric 6.5 ajoute le support de la spécification de la stratégie de tri pour les domaines de mise à niveau lors des mises à niveau de cluster ou d’application. Les stratégies prises en charge sont Numeric, Lexicographical, ReverseNumeric et ReverseLexicographical. La valeur par défaut est Numeric. Pour pouvoir utiliser cette fonctionnalité, le paramètre de manifeste de cluster ClusterManager/ AllowCustomUpgradeSortPolicies doit être défini sur true comme deuxième étape de mise à niveau de configuration après la mise à niveau du code SF 6.5. Il est important que cette opération soit effectuée en deux phases. Dans le cas contraire, l’ordre de mise à niveau risque d’être confus dans la mise à niveau du code au cours de la première mise à niveau.|
|EnableDefaultServicesUpgrade | Valeur booléenne (valeur par défaut : false) |Dynamique|Active la mise à niveau des services par défaut lors de la mise à niveau de l’application. Les descriptions de service par défaut sont remplacées après la mise à niveau. |
|FabricUpgradeHealthCheckInterval |Durée en secondes, la valeur par défaut est 60 |Dynamique|Fréquence des contrôles de l’état d’intégrité durant une mise à niveau de la structure surveillée |
|FabricUpgradeStatusPollInterval |Durée en secondes, la valeur par défaut est 60 |Dynamique|Fréquence d’interrogation de l’état de mise à niveau de la structure. Cette valeur détermine le taux de mise à jour pour tout appel GetFabricUpgradeProgress |
|ImageBuilderTimeoutBuffer |Durée en secondes (valeur par défaut : 3) |Dynamique|Spécifiez la durée en secondes. Délai à respecter pour autoriser le renvoi au client des erreurs de délai d’expiration propres à Image Builder. Si cette mémoire tampon est trop petite, le délai d’expiration du client s’écoule avant le serveur et reçoit une erreur générique de délai d’expiration. |
|InfrastructureTaskHealthCheckRetryTimeout | Durée en secondes, la valeur par défaut est 60 |Dynamique|Spécifiez la durée en secondes. Délai à respecter pour relancer les contrôles d’intégrité ayant échoué, après le post-traitement d’une tâche d’infrastructure. L’examen d’un contrôle d’intégrité réussi réinitialise ce minuteur. |
|InfrastructureTaskHealthCheckStableDuration | Durée en secondes (valeur par défaut : 0)|Dynamique| Spécifiez la durée en secondes. Délai à respecter pour examiner les contrôles d’intégrité passés avec succès consécutivement avant que le post-traitement d’une tâche d’infrastructure aboutisse. L’observation d’un contrôle d’intégrité ayant échoué réinitialise ce minuteur. |
|InfrastructureTaskHealthCheckWaitDuration |Durée en secondes (valeur par défaut : 0)|Dynamique| Spécifiez la durée en secondes. Délai à respecter avant de commencer les vérifications d’intégrité après le post-traitement d’une tâche d’infrastructure. |
|InfrastructureTaskProcessingInterval | Durée en secondes, la valeur par défaut est 10 |Dynamique|Spécifiez la durée en secondes. Intervalle de traitement utilisé par l’ordinateur d’état de traitement de la tâche infrastructure. |
|MaxCommunicationTimeout |Durée en secondes (valeur par défaut : 600) |Dynamique|Spécifiez la durée en secondes. Délai d’expiration maximum pour les communications internes entre ClusterManager et d’autres services système (comme le Service d’attribution de noms, le Gestionnaire de basculement, etc.). Ce délai d’expiration doit être inférieur au MaxOperationTimeout global (car il peut y avoir plusieurs communications entre les composants du système pour chaque opération de client). |
|MaxDataMigrationTimeout |Durée en secondes (valeur par défaut : 600) |Dynamique|Spécifiez la durée en secondes. Délai d’expiration maximum pour les opérations de récupération de migration de données après une mise à niveau de l’infrastructure. |
|MaxOperationRetryDelay |Durée en secondes (valeur par défaut : 5)|Dynamique| Spécifiez la durée en secondes. Délai maximum des nouvelles tentatives internes lorsque des défaillances sont rencontrées. |
|MaxOperationTimeout |Durée en secondes (valeur par défaut : MaxValue) |Dynamique| Spécifiez la durée en secondes. Délai d’expiration global maximum pour le traitement interne des opérations sur ClusterManager. |
|MaxTimeoutRetryBuffer | Durée en secondes (valeur par défaut : 600) |Dynamique|Spécifiez la durée en secondes. Délai d’expiration maximal de l’opération quand une nouvelle tentative interne due aux délais d’expiration est `<Original Time out> + <MaxTimeoutRetryBuffer>`. Un délai d’expiration supplémentaire est ajouté en incréments de MinOperationTimeout. |
|MinOperationTimeout | Durée en secondes, la valeur par défaut est 60 |Dynamique|Spécifiez la durée en secondes. Délai d’expiration global minimum pour le traitement interne des opérations sur ClusterManager. |
|MinReplicaSetSize |Entier (valeur par défaut : 3) |Non autorisée|Paramètre MinReplicaSetSize pour ClusterManager. |
|PlacementConstraints | Chaîne (valeur par défaut : "") |Non autorisée|Paramètre PlacementConstraints pour ClusterManager. |
|QuorumLossWaitDuration |Durée en secondes (valeur par défaut : MaxValue) |Non autorisée| Spécifiez la durée en secondes. Paramètre QuorumLossWaitDuration pour ClusterManager. |
|ReplicaRestartWaitDuration |Durée en secondes, la valeur par défaut est (60.0 \* 30)|Non autorisée|Spécifiez la durée en secondes. Paramètre ReplicaRestartWaitDuration pour ClusterManager. |
|ReplicaSetCheckTimeoutRollbackOverride |Durée en secondes (valeur par défaut : 1200) |Dynamique| Spécifiez la durée en secondes. Si ReplicaSetCheckTimeout est défini sur la valeur maximale de DWORD, il est remplacé par la valeur de cette configuration dans le cadre de la restauration. La valeur utilisée pour la restauration par progression n’est pas remplacée. |
|SkipRollbackUpdateDefaultService | Valeur booléenne (valeur par défaut : false) |Dynamique|CM ne rétablira pas les services par défaut mis à jour, lors de l’annulation de la mise à niveau de l’application. |
|StandByReplicaKeepDuration | Durée en seconde, la valeur par défaut est (3600.0 \* 2)|Non autorisée|Spécifiez la durée en secondes. Paramètre StandByReplicaKeepDuration pour ClusterManager. |
|TargetReplicaSetSize |Entier (valeur par défaut : 7) |Non autorisée|Paramètre TargetReplicaSetSize pour ClusterManager. |
|UpgradeHealthCheckInterval |Durée en secondes, la valeur par défaut est 60 |Dynamique|Fréquence des contrôles de l’état d’intégrité durant une mise à niveau de l’application surveillée |
|UpgradeStatusPollInterval |Durée en secondes, la valeur par défaut est 60 |Dynamique|Fréquence d’interrogation de l’état de mise à niveau de l’application. Cette valeur détermine le taux de mise à jour pour tout appel GetApplicationUpgradeProgress |
|CompleteClientRequest | Valeur booléenne (valeur par défaut : false) |Dynamique| Termine la requête du client lorsqu’elle est acceptée par CM. |

## <a name="common"></a>Courant

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PerfMonitorInterval |Durée en secondes (valeur par défaut : 1) |Dynamique|Spécifiez la durée en secondes. Interface de surveillance des performances. Une valeur nulle ou négative désactive la surveillance. |

## <a name="defragmentationemptynodedistributionpolicy"></a>DefragmentationEmptyNodeDistributionPolicy
| **Paramètre** | **Valeurs autorisées** |**Stratégie de mise à niveau**| **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|KeyIntegerValueMap, valeur par défaut : None|Dynamique|Spécifie la défragmentation de la stratégie qui survient lorsque vous videz des nœuds. Pour une métrique donnée, la valeur 0 indique que SF doit tenter de défragmenter les nœuds équitablement entre les domaines de mise à niveau et les domaines d’erreur ; la valeur 1 indique uniquement que les nœuds doivent être défragmentés |

## <a name="defragmentationmetrics"></a>DefragmentationMetrics
| **Paramètre** | **Valeurs autorisées** |**Stratégie de mise à niveau**| **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|KeyBoolValueMap, valeur par défaut : None|Dynamique|Détermine le jeu de métriques qui doit être utilisé pour la défragmentation et pas pour l’équilibrage de charge. |

## <a name="defragmentationmetricspercentornumberofemptynodestriggeringthreshold"></a>DefragmentationMetricsPercentOrNumberOfEmptyNodesTriggeringThreshold
| **Paramètre** | **Valeurs autorisées** |**Stratégie de mise à niveau**| **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|KeyDoubleValueMap, valeur par défaut : None|Dynamique|Détermine le nombre de nœuds libres nécessaires pour défragmenter le cluster en spécifiant un pourcentage dans la plage [0.0 - 1.0] ou le nombre de nœuds vides lorsque le nombre >= 1.0 |

## <a name="diagnostics"></a>Diagnostics

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|AdminOnlyHttpAudit |Valeur booléenne (valeur par défaut : true) | Dynamique | Excluez les requêtes HTTP qui n’affectent pas l’état du cluster de l’audit. Actuellement, seules les requêtes de type « GET » sont exclues, mais cela est susceptible de changer. |
|AppDiagnosticStoreAccessRequiresImpersonation |Valeur booléenne (valeur par défaut : true) | Dynamique |Indique si l’emprunt d’identité est requis lors de l’accès aux magasins de diagnostics pour le compte de l’application. |
|AppEtwTraceDeletionAgeInDays |Entier (valeur par défaut : 3) | Dynamique |Délai en jours à l’issue duquel nous supprimons les anciens fichiers ETL contenant les traces ETW d’application. |
|ApplicationLogsFormatVersion |Entier (valeur par défaut : 0) | Dynamique |Version du format des journaux d’activité de l’application. Valeurs prises en charge : 0 et 1. La version 1 comprend davantage de champs de l’enregistrement d’événement ETW que la version 0. |
|AuditHttpRequests |Valeur booléenne (valeur par défaut : false) | Dynamique | Activez ou désactivez l’audit HTTP. L’objectif de l’audit est de voir les activités qui ont été effectuées sur le cluster, notamment l’utilisateur qui a lancé la requête. Notez qu’il s’agit d’un enregistrement de meilleure tentative ; une perte de trace peut se produire. Les requêtes HTTP avec l’authentification « Utilisateur » ne sont pas enregistrées. |
|CaptureHttpTelemetry|Valeur booléenne (valeur par défaut : true) | Dynamique | Activez ou désactivez la télémétrie HTTP. L’objectif de la télémétrie pour Service Fabric est d’être en mesure de capturer des données de télémétrie pour mieux planifier les travaux futurs et identifier les aspects problématiques. La télémétrie n’enregistre aucune donnée personnelle ni le corps de la requête. La télémétrie capture toutes les requêtes HTTP, sauf si elle est configurée d’une autre manière. |
|ClusterId |String | Dynamique |ID unique du cluster. Valeur générée lors de la création du cluster. |
|ConsumerInstances |String | Dynamique |Liste des instances de consommateur DCA. |
|DiskFullSafetySpaceInMB |Entier (valeur par défaut : 1024) | Dynamique |Espace disque restant (en Mo) à protéger contre toute utilisation par DCA. |
|EnableCircularTraceSession |Valeur booléenne (valeur par défaut : false) | statique |Indicateur spécifiant si les sessions de trace circulaire doivent être utilisées. |
|EnablePlatformEventsFileSink |Valeur booléenne (valeur par défaut : false) | statique |Activer/désactiver les événements de plateforme écrits sur le disque |
|EnableTelemetry |Valeur booléenne (valeur par défaut : true) | Dynamique |Ce paramètre active ou désactive la télémétrie. |
|FailuresOnlyHttpTelemetry | Valeur booléenne (valeur par défaut : false) | Dynamique | Si la capture de télémétrie HTTP est activée, capturez uniquement les requêtes ayant échoué. Cela a pour but de réduire le nombre d’événements générés pour la télémétrie. |
|HttpTelemetryCapturePercentage | Entier, valeur par défaut : 50 | Dynamique | Si la capture de télémétrie HTTP est activée, capturez uniquement un pourcentage aléatoire de requêtes. Cela a pour but de réduire le nombre d’événements générés pour la télémétrie. |
|MaxDiskQuotaInMB |Entier (valeur par défaut : 65536) | Dynamique |Quota de disque (en Mo) pour les fichiers journaux de Windows Fabric. |
|ProducerInstances |String | Dynamique |Liste des instances de producteur DCA. |

## <a name="dnsservice"></a>DnsService
| **Paramètre** | **Valeurs autorisées** |**Stratégie de mise à niveau**| **Conseils ou brève description** |
| --- | --- | --- | --- |
|EnablePartitionedQuery|valeur booléenne, valeur par défaut : FALSE|statique|Indicateur permettant d’activer la prise en charge des requêtes DNS pour les services partitionnés. Cette fonctionnalité est désactivée par défaut. Pour en savoir plus, voir [Service DNS Service Fabric.](service-fabric-dnsservice.md)|
|InstanceCount|entier, valeur par défaut : -1|statique|La valeur par défaut est 1, ce qui signifie que DnsService est en cours d’exécution sur chaque nœud. OneBox a besoin de cette option ait la valeur 1, dans la mesure où DnsService utilise le port 53 bien connu. Donc, il ne peut pas avoir plusieurs instances sur le même ordinateur.|
|IsEnabled|valeur booléenne, valeur par défaut : FALSE|statique|Active/désactive DnsService. DnsService est désactivé par défaut, et cette configuration doit être définie pour l’activer. |
|PartitionPrefix|Chaîne (valeur par défaut : "--")|statique|Contrôle la valeur de la chaîne de préfixe de partition dans les requêtes DNS pour les services partitionnés. La valeur : <ul><li>doit être conforme à RFC, car elle fera partie d’une requête DNS ;</li><li>ne doit pas contenir de point (« . »), car le point interfère avec le comportement de suffixe DNS ;</li><li>ne doit pas comporter plus de 5 caractères ;</li><li>ne peut pas être une chaîne vide.</li><li>Si le paramètre PartitionPrefix est remplacé, PartitionSuffix doit être substitué et vice versa.</li></ul>Pour en savoir plus, voir [Service DNS Service Fabric](service-fabric-dnsservice.md).|
|PartitionSuffix|Chaîne (valeur par défaut : "")|statique|Contrôle la valeur de la chaîne de suffixe de partition dans les requêtes DNS pour les services partitionnés. La valeur : <ul><li>doit être conforme à RFC, car elle fera partie d’une requête DNS ;</li><li>ne doit pas contenir de point (« . »), car le point interfère avec le comportement de suffixe DNS ;</li><li>ne doit pas comporter plus de 5 caractères ;</li><li>Si le paramètre PartitionPrefix est remplacé, PartitionSuffix doit être substitué et vice versa.</li></ul>Pour en savoir plus, voir [Service DNS Service Fabric](service-fabric-dnsservice.md). |
|RetryTransientFabricErrors|Valeur booléenne (valeur par défaut : true)|statique|Le paramètre contrôle les fonctionnalités de nouvelle tentative lors de l’appel des API Service Fabric à partir de DnsService. Lorsqu’il est activé, il effectue jusqu’à trois tentatives en cas d’erreur temporaire.|

## <a name="eventstoreservice"></a>EventStoreService

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|MinReplicaSetSize|entier (valeur par défaut : 0)|statique|MinReplicaSetSize pour le service EventStore |
|PlacementConstraints|chaîne, valeur par défaut : « »|statique|    PlacementConstraints pour le service EventStore |
|TargetReplicaSetSize|entier, valeur par défaut : 0|statique| TargetReplicaSetSize pour le service EventStore |

## <a name="fabricclient"></a>FabricClient

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|ConnectionInitializationTimeout |Durée en secondes (valeur par défaut : 2) |Dynamique|Spécifiez la durée en secondes. Délai d’expiration de connexion à respecter pour chaque client avant de tenter d’établir une connexion à la passerelle.|
|HealthOperationTimeout |Durée en secondes (valeur par défaut : 120) |Dynamique|Spécifiez la durée en secondes. Délai d’expiration d’un message de rapport envoyé au Gestionnaire d’intégrité. |
|HealthReportRetrySendInterval |Durée en secondes, valeur par défaut : 30, valeur minimale : 1 |Dynamique|Spécifiez la durée en secondes. Délai à l’issue duquel le composant concerné renvoie les rapports d’intégrité cumulés à Health Manager. |
|HealthReportSendInterval |Durée en secondes, la valeur par défaut est 30 |Dynamique|Spécifiez la durée en secondes. Délai à l’issue duquel le composant concerné envoie les rapports d’intégrité cumulés au Gestionnaire d’intégrité. |
|KeepAliveIntervalInSeconds |Entier (valeur par défaut : 20) |statique|Durée pendant laquelle le transport FabricClient envoie des messages de conservation d’activité à la passerelle. À 0, le paramètre keepAlive est désactivé. Cette valeur doit être un entier positif. |
|MaxFileSenderThreads |Valeur Uint (valeur par défaut : 10) |statique|Nombre maximum de fichiers transférés en parallèle. |
|NodeAddresses |Chaîne (valeur par défaut : "") |statique|Collection d’adresses (chaînes de connexion) sur différents nœuds pouvant servir à communiquer avec le service d’attribution de noms. Au début, le client se connecte en sélectionnant une des adresses de façon aléatoire. Si plusieurs chaînes de connexion sont fournies et qu’une échoue en raison d’une erreur de communication ou de délai d’attente, le client utilise l’adresse suivante dans la liste. Pour plus d’informations sur la sémantique des nouvelles tentatives, consultez la section sur les nouvelles tentatives concernant l’adresse du service d’attribution de noms. |
|PartitionLocationCacheLimit |Entier (valeur par défaut : 100000) |statique|Nombre de partitions mises en cache pour la résolution de service (0 pour aucune limite). |
|RetryBackoffInterval |Durée en secondes (valeur par défaut : 3) |Dynamique|Spécifiez la durée en secondes. Délai de temporisation à respecter avant de retenter l’opération. |
|ServiceChangePollInterval |Durée en secondes (valeur par défaut : 120) |Dynamique|Spécifiez la durée en secondes. L’intervalle entre deux interrogations consécutives de modifications de service est différent entre le client et la passerelle pour les rappels de notifications de modification de service enregistrés. |

## <a name="fabrichost"></a>FabricHost

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|ActivationMaxFailureCount |Entier (valeur par défaut : 10) |Dynamique|Nombre maximum pour lequel le système va relancer l’activation ayant échoué avant d’abandonner. |
|ActivationMaxRetryInterval |Durée en secondes, la valeur par défaut est 300 |Dynamique|Spécifiez la durée en secondes. Intervalle maximum avant une nouvelle tentative d’activation. À chaque échec continu, l’intervalle avant nouvelle tentative est calculé de la manière suivante : Min (ActivationMaxRetryInterval; Nombre d’échecs continus * ActivationRetryBackoffInterval). |
|ActivationRetryBackoffInterval |Durée en secondes (valeur par défaut : 5) |Dynamique|Spécifiez la durée en secondes. Intervalle d’interruption à chaque échec d’activation. À chaque échec d’activation continue, le système retente l’activation jusqu’à atteindre la valeur du paramètre MaxActivationFailureCount. L’intervalle avant nouvelle tentative pour chaque tentative est une combinaison de l’échec de l’activation continue et de l’intervalle d’interruption d’activation. |
|EnableRestartManagement |Valeur booléenne (valeur par défaut : false) |Dynamique|Ce paramètre active le redémarrage du serveur. |
|EnableServiceFabricAutomaticUpdates |Valeur booléenne (valeur par défaut : false) |Dynamique|Ce paramètre permet d’activer les mises à jour automatiques de l’infrastructure via Windows Update. |
|EnableServiceFabricBaseUpgrade |Valeur booléenne (valeur par défaut : false) |Dynamique|Ce paramètre active la mise à jour de base du serveur. |
|FailureReportingExpeditedReportingIntervalEnabled | Valeur booléenne (valeur par défaut : true) | statique | Assure une vitesse de chargement supérieure dans DCA quand FabricHost est en mode de création de rapports d’échec. |
|FailureReportingTimeout | TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(60) | statique |Spécifiez la durée en secondes. Délai d’expiration des rapports d’échec DCA dans le cas où FabricHost rencontre un échec de démarrage en phase initiale. | 
|RunDCAOnStartupFailure | Valeur booléenne (valeur par défaut : true) | statique |Détermine s’il faut lancer DCA pour charger les journaux en cas de problèmes de démarrage dans FabricHost. | 
|StartTimeout |Durée en secondes, la valeur par défaut est 300 |Dynamique|Spécifiez la durée en secondes. Délai d’expiration pour le démarrage de fabricactivationmanager. |
|StopTimeout |Durée en secondes, la valeur par défaut est 300 |Dynamique|Spécifiez la durée en secondes. Délai d’attente pour l’activation, la désactivation et la mise à niveau du service hébergé. |

## <a name="fabricnode"></a>FabricNode

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|ClientAuthX509FindType |Chaîne (valeur par défaut : "FindByThumbprint") |Dynamique|Indique comment rechercher le certificat de serveur dans le magasin spécifié par la valeur ClientAuthX509StoreName prise en charge : FindByThumbprint; FindBySubjectName. |
|ClientAuthX509FindValue |Chaîne (valeur par défaut : "") | Dynamique|Valeur de filtre de recherche utilisée pour localiser le certificat du rôle d’administrateur par défaut FabricClient. |
|ClientAuthX509FindValueSecondary |Chaîne (valeur par défaut : "") |Dynamique|Valeur de filtre de recherche utilisée pour localiser le certificat du rôle d’administrateur par défaut FabricClient. |
|ClientAuthX509StoreName |Chaîne (valeur par défaut : "My") |Dynamique|Nom du magasin de certificats X.509 qui contient le certificat pour le rôle d’administrateur par défaut FabricClient. |
|ClusterX509FindType |Chaîne (valeur par défaut : "FindByThumbprint") |Dynamique|Indique comment rechercher le certificat de cluster dans le magasin spécifié par les valeurs ClusterX509StoreName prises en charge : "FindByThumbprint"; "FindBySubjectName" avec "FindBySubjectName"; quand il y a plusieurs correspondances ; celle avec l’expiration la plus loin est utilisée. |
|ClusterX509FindValue |Chaîne (valeur par défaut : "") |Dynamique|Valeur de filtre de recherche utilisée pour localiser le certificat de cluster. |
|ClusterX509FindValueSecondary |Chaîne (valeur par défaut : "") |Dynamique|Valeur de filtre de recherche utilisée pour localiser le certificat de cluster. |
|ClusterX509StoreName |Chaîne (valeur par défaut : "My") |Dynamique|Nom du magasin de certificats X.509 qui contient le certificat de cluster utilisé pour sécuriser les communications internes au cluster. |
|EndApplicationPortRange |Entier (valeur par défaut : 0) |statique|Fin (non inclusive) des ports d’application gérés par le sous-système d’hébergement. Requis si EndpointFilteringEnabled a la valeur true dans Hosting. |
|ServerAuthX509FindType |Chaîne (valeur par défaut : "FindByThumbprint") |Dynamique|Indique comment rechercher le certificat de serveur dans le magasin spécifié par la valeur ServerAuthX509StoreName prise en charge : FindByThumbprint; FindBySubjectName. |
|ServerAuthX509FindValue |Chaîne (valeur par défaut : "") |Dynamique|Valeur de filtre de recherche utilisée pour localiser le certificat de serveur. |
|ServerAuthX509FindValueSecondary |Chaîne (valeur par défaut : "") |Dynamique|Valeur de filtre de recherche utilisée pour localiser le certificat de serveur. |
|ServerAuthX509StoreName |Chaîne (valeur par défaut : "My") |Dynamique|Nom du magasin de certificats X.509 qui contient le certificat de serveur pour le service d’entrée. |
|StartApplicationPortRange |Entier (valeur par défaut : 0) |statique|Début des ports d’application gérés par le sous-système d’hébergement. Requis si EndpointFilteringEnabled a la valeur true dans Hosting. |
|StateTraceInterval |Durée en secondes, la valeur par défaut est 300 |statique|Spécifiez la durée en secondes. Durée du suivi d’état sur chaque nœud et les nœuds actifs sur FM/FMM. |
|UserRoleClientX509FindType |Chaîne (valeur par défaut : "FindByThumbprint") |Dynamique|Indique comment rechercher le certificat dans le magasin spécifié par la valeur UserRoleClientX509StoreName prise en charge : FindByThumbprint; FindBySubjectName. |
|UserRoleClientX509FindValue |Chaîne (valeur par défaut : "") |Dynamique|Valeur de filtre de recherche utilisée pour localiser le certificat du rôle d’utilisateur par défaut FabricClient. |
|UserRoleClientX509FindValueSecondary |Chaîne (valeur par défaut : "") |Dynamique|Valeur de filtre de recherche utilisée pour localiser le certificat du rôle d’utilisateur par défaut FabricClient. |
|UserRoleClientX509StoreName |Chaîne (valeur par défaut : "My") |Dynamique|Nom du magasin de certificats X.509 qui contient le certificat pour le rôle d’utilisateur par défaut FabricClient. |

## <a name="failovermanager"></a>FailoverManager

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|AllowNodeStateRemovedForSeedNode|Valeur booléenne, valeur par défaut : FALSE |Dynamique|Indicateur qui spécifie s’il est autorisé de supprimer l’état du nœud pour un nœud de départ |
|BuildReplicaTimeLimit|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(3600)|Dynamique|Spécifiez la durée en secondes. Le délai de construction d’un réplica avec état après lequel un rapport d’intégrité Warning est généré |
|ClusterPauseThreshold|entier, valeur par défaut : 1|Dynamique|Si le nombre de nœuds dans le système passe sous cette valeur, le placement, l’équilibrage de charge et le basculement sont arrêtés. |
|CreateInstanceTimeLimit|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(300)|Dynamique|Spécifiez la durée en secondes. Le délai de création d’un réplica sans état après lequel un rapport d’intégrité Warning est généré |
|ExpectedClusterSize|entier, valeur par défaut : 1|Dynamique|Lorsque le cluster est démarré pour la première fois, le FM attendra que ce nombre de nœuds soit atteint avant de placer d’autres services, notamment les services système comme l’affectation de noms. Accroître cette valeur augmente le temps nécessaire au démarrage d’un cluster mais empêche toute surcharge des premiers nœuds ainsi que les déplacements supplémentaires qui sont nécessaires à mesure que d’autres nœuds sont mis en ligne. Cette valeur doit être définie en général sur une petite fraction de la taille initiale du cluster. |
|ExpectedNodeDeactivationDuration|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(60.0 \* 30)|Dynamique|Spécifiez la durée en secondes. La durée prévue de la désactivation complète d’un nœud. |
|ExpectedNodeFabricUpgradeDuration|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(60.0 \* 30)|Dynamique|Spécifiez la durée en secondes. La durée prévue de la mise à niveau d’un nœud lors de la mise à niveau de Windows Fabric. |
|ExpectedReplicaUpgradeDuration|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(60.0 \* 30)|Dynamique|Spécifiez la durée en secondes. La durée prévue de la mise à niveau de tous les réplicas sur un nœud lors de la mise à niveau de l’application. |
|IsSingletonReplicaMoveAllowedDuringUpgrade|Valeur booléenne, valeur par défaut : TRUE|Dynamique|Si cette valeur est définie sur true, les réplicas avec une taille de jeu de réplicas cible de 1 seront autorisés à se déplacer lors de la mise à niveau. |
|MinReplicaSetSize|entier, valeur par défaut : 3|Non autorisée|Taille minimale du jeu de réplicas pour le FM. Si le nombre de réplicas FM actifs passe sous cette valeur, le FM rejette toutes les modifications apportées au cluster tant que le nombre minimal de réplicas n’a pas été récupéré |
|PlacementConstraints|Chaîne (valeur par défaut : "")|Non autorisée|Toutes les contraintes de placement pour les réplicas de Failover Manager |
|PlacementTimeLimit|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(600)|Dynamique|Spécifiez la durée en secondes. Le délai pour atteindre le nom de réplicas cible après lequel un rapport d’intégrité Warning est généré |
|QuorumLossWaitDuration |Durée en secondes (valeur par défaut : MaxValue) |Dynamique|Spécifiez la durée en secondes. Il s’agit de la durée maximale pour laquelle nous autorisons une partition à être dans un état de perte de quorum. Si la partition est toujours en perte de quorum au-delà de cette durée, elle est rétablie en considérant les réplicas arrêtés comme perdus. Notez que cela peut potentiellement provoquer une perte de données. |
|ReconfigurationTimeLimit|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(300)|Dynamique|Spécifiez la durée en secondes. Le délai de reconfiguration après lequel un rapport d’intégrité Warning est généré |
|ReplicaRestartWaitDuration|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(60.0 \* 30)|Non autorisée|Spécifiez la durée en secondes. Paramètre ReplicaRestartWaitDuration pour FMService |
| SeedNodeQuorumAdditionalBufferNodes | entier (valeur par défaut : 0) | Dynamique | Mémoire tampon des nœuds initiaux qui doivent fonctionner (avec le quorum des nœuds initiaux). FM doit autoriser la défaillance d’au maximum (totalNumSeedNodes - (seedNodeQuorum + SeedNodeQuorumAdditionalBufferNodes)) nœuds initiaux. |
|StandByReplicaKeepDuration|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(3600.0 \* 24 \* 7)|Non autorisée|Spécifiez la durée en secondes. Paramètre StandByReplicaKeepDuration pour FMService |
|TargetReplicaSetSize|entier, valeur par défaut : 7|Non autorisée|Il s’agit du nombre cible de réplicas FM que Windows Fabric conservera. Un nombre plus élevé entraîne une plus grande fiabilité des données FM, avec peu d’impact sur les performances. |
|UserMaxStandByReplicaCount |Entier (valeur par défaut : 1) |Dynamique|Nombre maximum par défaut de réplicas en attente que le système conserve pour les services de l’utilisateur. |
|UserReplicaRestartWaitDuration |Durée en secondes, la valeur par défaut est 60.0 \* 30 |Dynamique|Spécifiez la durée en secondes. Lorsqu’un réplica persistant tombe en panne, Windows Fabric attend pendant cette durée que le réplica revienne en ligne avant de créer d’autres réplicas de remplacement (ce qui nécessite une copie de l’état). |
|UserStandByReplicaKeepDuration |Durée en secondes (valeur par défaut : 3600.0 \* 24 \* 7) |Dynamique|Spécifiez la durée en secondes. Lorsqu’un réplica persistant n’est plus défaillant, il peut avoir déjà été remplacé. Ce minuteur détermine la durée pendant laquelle le FM conserve le réplica en attente avant de le rejeter. |

## <a name="faultanalysisservice"></a>FaultAnalysisService

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|CompletedActionKeepDurationInSeconds | Entier (valeur par défaut : 604800) |statique| Durée approximative de conservation des actions dans un état terminal. Cela dépend également de StoredActionCleanupIntervalInSeconds, car le travail de nettoyage ne s’effectue que pendant cette durée. 604800 correspond à 7 jours. |
|DataLossCheckPollIntervalInSeconds|entier, valeur par défaut : 5|statique|Il s’agit du délai d’attente du système entre les vérifications avant qu’une perte de données ne se produise. Le nombre de fois où le nombre de pertes de données sera vérifié par itération interne est DataLossCheckWaitDurationInSeconds/this. |
|DataLossCheckWaitDurationInSeconds|entier, valeur par défaut : 25|statique|Le délai total, en secondes, que le système attendra avant qu’une perte de données se produise. Cette valeur est utilisée en interne lors de l’appel de l’api StartPartitionDataLossAsync(). |
|MinReplicaSetSize |Entier (valeur par défaut : 0) |statique|Paramètre MinReplicaSetSize pour FaultAnalysisService. |
|PlacementConstraints | Chaîne (valeur par défaut : "")|statique| Paramètre PlacementConstraints pour FaultAnalysisService. |
|QuorumLossWaitDuration | Durée en secondes (valeur par défaut : MaxValue) |statique|Spécifiez la durée en secondes. Paramètre QuorumLossWaitDuration pour FaultAnalysisService. |
|ReplicaDropWaitDurationInSeconds|entier, valeur par défaut : 600|statique|Ce paramètre est utilisé lorsque l’api de perte de données est appelée. Il contrôle le délai que le système attend avant qu’un réplica soit abandonné après la suppression d’un réplica appelé en interne. |
|ReplicaRestartWaitDuration |Durée en secondes (valeur par défaut : 60 minutes)|statique|Spécifiez la durée en secondes. Paramètre ReplicaRestartWaitDuration pour FaultAnalysisService. |
|StandByReplicaKeepDuration| Durée en secondes (valeur par défaut : 60 *24* 7 minutes) |statique|Spécifiez la durée en secondes. Paramètre StandByReplicaKeepDuration pour FaultAnalysisService. |
|StoredActionCleanupIntervalInSeconds | Entier (valeur par défaut : 3600) |statique|Fréquence de nettoyage du magasin. Seules les actions dans un état terminal et celles qui exécutées il y a au moins CompletedActionKeepDurationInSeconds secondes seront supprimées. |
|StoredChaosEventCleanupIntervalInSeconds | Entier (valeur par défaut : 3600) |statique|Fréquence à laquelle le système évalue la nécessité du nettoyage. Si le nombre d’événements est supérieur à 30000, le nettoyage se déclenche. |
|TargetReplicaSetSize |Entier (valeur par défaut : 0) |statique|NOT_PLATFORM_UNIX_START Paramètre TargetReplicaSetSize pour FaultAnalysisService. |

## <a name="federation"></a>Fédération

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|LeaseDuration |Durée en secondes, la valeur par défaut est 30 |Dynamique|Durée de bail entre un nœud et ses voisins. |
|LeaseDurationAcrossFaultDomain |Durée en secondes, la valeur par défaut est 30 |Dynamique|Durée de bail entre un nœud et ses voisins dans des domaines d’erreur. |

## <a name="filestoreservice"></a>FileStoreService

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|AcceptChunkUpload|Valeur booléenne, valeur par défaut : TRUE|Dynamique|Configurez ce paramètre pour déterminer si le service de magasin de fichiers accepte le chargement de fichiers en fonction du bloc lors de la copie du package d’application. |
|AnonymousAccessEnabled | Valeur booléenne (valeur par défaut : true) |statique|Activez ou désactivez l’accès anonyme aux partages FileStoreService. |
|CommonName1Ntlmx509CommonName|Chaîne (valeur par défaut : "")|statique| Nom commun du certificat X509 utilisé pour générer le code HMAC sur le CommonName1NtlmPasswordSecret en cas d’authentification NTLM |
|CommonName1Ntlmx509StoreLocation|Chaîne (valeur par défaut : "LocalMachine")|statique|Emplacement du magasin du certificat X509 utilisé pour générer le code HMAC sur le CommonName1NtlmPasswordSecret en cas d’authentification NTLM |
|CommonName1Ntlmx509StoreName|Chaîne (valeur par défaut : "MY")| statique|Nom du magasin du certificat X509 utilisé pour générer le code HMAC sur le CommonName1NtlmPasswordSecret en cas d’authentification NTLM |
|CommonName2Ntlmx509CommonName|Chaîne (valeur par défaut : "")|statique|Nom commun du certificat X509 utilisé pour générer le code HMAC sur le CommonName2NtlmPasswordSecret en cas d’authentification NTLM |
|CommonName2Ntlmx509StoreLocation|Chaîne (valeur par défaut : "LocalMachine")| statique|Emplacement du magasin du certificat X509 utilisé pour générer le code HMAC sur le CommonName2NtlmPasswordSecret en cas d’authentification NTLM |
|CommonName2Ntlmx509StoreName|Chaîne (valeur par défaut : "MY")|statique| Nom du magasin du certificat X509 utilisé pour générer le code HMAC sur le CommonName2NtlmPasswordSecret en cas d’authentification NTLM |
|CommonNameNtlmPasswordSecret|SecureString, valeur par défaut : Common::SecureString("")| statique|Secret de mot de passe utilisé comme valeur initiale pour générer le même mot de passe en cas d’authentification NTLM |
|DiskSpaceHealthReportingIntervalWhenCloseToOutOfDiskSpace |TimeSpan, valeur par défaut : Common::TimeSpan::FromMinutes(5)|Dynamique|Spécifiez la durée en secondes. Intervalle de temps entre la vérification de l’espace disque et le signalement d’un événement d’intégrité lorsque le disque est proche de l’espace insuffisant. |
|DiskSpaceHealthReportingIntervalWhenEnoughDiskSpace |TimeSpan, la valeur par défaut est Common::TimeSpan::FromMinutes(15)|Dynamique|Spécifiez la durée en secondes. Intervalle de temps entre la vérification de l’espace disque et le signalement d’un événement d’intégrité lorsque l’espace sur le disque est suffisant. |
|EnableImageStoreHealthReporting |Valeur booléenne, valeur par défaut : TRUE    |statique|Configuration pour déterminer si le service de magasin de fichiers doit signaler son intégrité. |
|FreeDiskSpaceNotificationSizeInKB|int64, la valeur par défaut est 25\*1024 |Dynamique|Taille de l’espace disque disponible au-dessous de laquelle un avertissement d’intégrité peut se produire. La valeur minimale de cette configuration et de la configuration de FreeDiskSpaceNotificationThresholdPercentage est utilisée pour déterminer l’envoi de l’avertissement d’intégrité. |
|FreeDiskSpaceNotificationThresholdPercentage|double, la valeur par défaut est 0.02 |Dynamique|Pourcentage d’espace disque disponible au-dessous duquel un avertissement d’intégrité peut se produire. La valeur minimale de cette configuration et de la configuration de FreeDiskSpaceNotificationInMB est utilisée pour déterminer l’envoi de l’avertissement d’intégrité. |
|GenerateV1CommonNameAccount| Valeur booléenne, valeur par défaut : TRUE|statique|Spécifie s’il faut générer un compte avec l’algorithme de génération V1 de nom d’utilisateur. Depuis Service Fabric version 6.1, un compte avec la génération v2 est toujours créé. Le compte V1 est nécessaire pour les mises à niveau à partir de/vers des versions qui ne prennent pas en charge la génération V2 (avant 6.1).|
|MaxCopyOperationThreads | Valeur Uint (valeur par défaut : 0) |Dynamique| Nombre maximum de fichiers parallèles que le réplica secondaire peut copier à partir du réplica principal. 0 = nombre de cœurs. |
|MaxFileOperationThreads | Valeur Uint (valeur par défaut : 100) |statique| Nombre maximum de threads parallèles autorisés à effectuer des opérations sur les fichiers (copie/déplacement) dans le réplica principal. 0 = nombre de cœurs. |
|MaxRequestProcessingThreads | Valeur Uint (valeur par défaut : 200) |statique|Nombre maximum de threads parallèles autorisés pour traiter les requêtes dans le réplica principal. 0 = nombre de cœurs. |
|MaxSecondaryFileCopyFailureThreshold | Valeur Uint (valeur par défaut : 25)|Dynamique|Nombre maximum de nouvelles tentatives de copie de fichier sur le réplica secondaire avant d’abandonner. |
|MaxStoreOperations | Valeur Uint (valeur par défaut : 4096) |statique|Nombre maximum d’opérations parallèles de transaction de magasin autorisées sur le réplica principal. 0 = nombre de cœurs. |
|NamingOperationTimeout |Durée en secondes, la valeur par défaut est 60 |Dynamique|Spécifiez la durée en secondes. Délai pour effectuer l’opération d’attribution de noms. |
|PrimaryAccountNTLMPasswordSecret | Valeur SecureString (valeur par défaut : vide) |statique| Secret de mot de passe utilisé comme valeur initiale pour générer le même mot de passe en cas d’authentification NTLM. |
|PrimaryAccountNTLMX509StoreLocation | Chaîne (valeur par défaut : "LocalMachine")|statique| Emplacement du magasin du certificat X509 utilisé pour générer le code HMAC sur le PrimaryAccountNTLMPasswordSecret en cas d’authentification NTLM. |
|PrimaryAccountNTLMX509StoreName | Chaîne (valeur par défaut : "MY")|statique| Nom du magasin du certificat X509 utilisé pour générer le code HMAC sur le PrimaryAccountNTLMPasswordSecret en cas d’authentification NTLM. |
|PrimaryAccountNTLMX509Thumbprint | Chaîne (valeur par défaut : "")|statique|Empreinte du certificat X509 utilisé pour générer le code HMAC sur le PrimaryAccountNTLMPasswordSecret en cas d’authentification NTLM. |
|PrimaryAccountType | Chaîne (valeur par défaut : "") |statique|Type de compte principal du principal utilisé pour contrôler l’accès aux partages FileStoreService. |
|PrimaryAccountUserName | Chaîne (valeur par défaut : "") |statique|Nom d’utilisateur du compte principal du principal utilisé pour contrôler l’accès aux partages FileStoreService. |
|PrimaryAccountUserPassword | Valeur SecureString (valeur par défaut : vide) |statique|Mot de passe du compte principal du principal utilisé pour contrôler l’accès aux partages FileStoreService. |
|QueryOperationTimeout | Durée en secondes, la valeur par défaut est 60 |Dynamique|Spécifiez la durée en secondes. Délai pour effectuer l’opération de requête. |
|SecondaryAccountNTLMPasswordSecret | Valeur SecureString (valeur par défaut : vide) |statique| Secret de mot de passe utilisé comme valeur initiale pour générer le même mot de passe en cas d’authentification NTLM. |
|SecondaryAccountNTLMX509StoreLocation | Chaîne (valeur par défaut : "LocalMachine") |statique|Emplacement du magasin du certificat X509 utilisé pour générer le code HMAC sur le SecondaryAccountNTLMPasswordSecret en cas d’authentification NTLM. |
|SecondaryAccountNTLMX509StoreName | Chaîne (valeur par défaut : "MY") |statique|Nom du magasin du certificat X509 utilisé pour générer le code HMAC sur le SecondaryAccountNTLMPasswordSecret en cas d’authentification NTLM. |
|SecondaryAccountNTLMX509Thumbprint | Chaîne (valeur par défaut : "")| statique|Empreinte du certificat X509 utilisé pour générer le code HMAC sur le SecondaryAccountNTLMPasswordSecret en cas d’authentification NTLM. |
|SecondaryAccountType | Chaîne (valeur par défaut : "")|statique| Type de compte secondaire du principal utilisé pour contrôler l’accès aux partages FileStoreService. |
|SecondaryAccountUserName | Chaîne (valeur par défaut : "")| statique|Nom d’utilisateur du compte secondaire du principal utilisé pour contrôler l’accès aux partages FileStoreService. |
|SecondaryAccountUserPassword | Valeur SecureString (valeur par défaut : vide) |statique|Mot de passe du compte secondaire du principal utilisé pour contrôler l’accès aux partages FileStoreService. |
|SecondaryFileCopyRetryDelayMilliseconds|valeur uint, valeur par défaut : 500|Dynamique|Délai de nouvelle tentative de copie de fichier (en millisecondes).|
|UseChunkContentInTransportMessage|Valeur booléenne, valeur par défaut : TRUE|Dynamique|Indicateur pour l’utilisation de la nouvelle version du protocole de chargement introduite dans v6.4. Cette version du protocole utilise Service Fabric Transport pour charger des fichiers dans le magasin d’images, ce qui offre de meilleures performances que le protocole SMB utilisé dans les versions précédentes. |

## <a name="healthmanager"></a>HealthManager

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|EnableApplicationTypeHealthEvaluation |Valeur booléenne (valeur par défaut : false) |statique|Stratégie d’évaluation de l’intégrité du cluster : activer pour chaque évaluation de l’intégrité du type d’application. |
|MaxSuggestedNumberOfEntityHealthReports|Entier (valeur par défaut : 100) |Dynamique|Nombre maximal de rapports d’intégrité qu’une entité peut avoir avant de susciter des inquiétudes concernant la logique de création de rapports d’intégrité de la surveillance. Chaque entité d’intégrité est supposée avoir un nombre relativement faible de rapports d’intégrité. Si le nombre de rapports dépasse ce nombre, des problèmes peuvent se poser avec l’implémentation de la surveillance. Une entité avec un trop grand nombre de rapports est marquée d’un rapport d’intégrité d’avertissement quand l’entité est évaluée. |

## <a name="healthmanagerclusterhealthpolicy"></a>HealthManager/ClusterHealthPolicy

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|ConsiderWarningAsError |Valeur booléenne (valeur par défaut : false) |statique|Stratégie d’évaluation d’intégrité de cluster : avertissements traités comme des erreurs. |
|MaxPercentUnhealthyApplications | Entier (valeur par défaut : 0) |statique|Stratégie d’évaluation d’intégrité de cluster : pourcentage maximum d’applications défaillantes autorisées pour que le cluster soit fonctionnel. |
|MaxPercentUnhealthyNodes | Entier (valeur par défaut : 0) |statique|Stratégie d’évaluation d’intégrité de cluster : pourcentage maximum de nœuds défaillants autorisés pour que le cluster soit fonctionnel. |

## <a name="healthmanagerclusterupgradehealthpolicy"></a>HealthManager/ClusterUpgradeHealthPolicy

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|MaxPercentDeltaUnhealthyNodes|entier, valeur par défaut : 10|statique|Stratégie d’évaluation d’intégrité de mise à niveau de cluster : pourcentage maximum du delta de nœuds défaillants autorisés pour que le cluster soit fonctionnel |
|MaxPercentUpgradeDomainDeltaUnhealthyNodes|entier, valeur par défaut : 15|statique|Stratégie d’évaluation d’intégrité de mise à niveau de cluster : pourcentage maximum de delta de nœuds défaillants autorisés pour que le cluster soit fonctionnel |

## <a name="hosting"></a>Hébergement

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|ActivationMaxFailureCount |Nombre entier, la valeur par défaut est 10 |Dynamique|Nombre de fois où les nouvelles tentatives système ont échoué à l’activation avant d’abandonner |
|ActivationMaxRetryInterval |Durée en secondes, la valeur par défaut est 300 |Dynamique|Pour chaque cas d’échec d’activation continue, le système retente l’activation pour une durée équivalente à ActivationMaxFailureCount. ActivationMaxRetryInterval spécifie l’intervalle de temps d’attente avant une nouvelle tentative après chaque échec d’activation |
|ActivationRetryBackoffInterval |Durée en secondes, la valeur par défaut est 5 |Dynamique|Intervalle d’interruption sur chaque échec d’activation. Sur chaque cas d’échec d’activation continue, le système retente l’activation pour une durée équivalente à MaxActivationFailureCount. L’intervalle avant nouvelle tentative pour chaque tentative est une combinaison de l’échec de l’activation continue et de l’intervalle d’interruption d’activation. |
|ActivationTimeout| TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(180)|Dynamique| Spécifiez la durée en secondes. Délai d’attente pour l’activation, la désactivation et la mise à niveau de l’application. |
|ApplicationHostCloseTimeout| TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(120)|Dynamique| Spécifiez la durée en secondes. Lorsqu’une sortie de Fabric est détectée dans des processus auto-activés ; FabricRuntime ferme tous les réplicas dans le processus de l’hôte de l’utilisateur (applicationhost). Il s’agit du délai d’attente pour l’opération de fermeture. |
| CnsNetworkPluginCnmUrlPort | Valeur wstring (valeur par défaut : L"48080") | statique | Port de l’URL de l’API Azure CNM. |
| CnsNetworkPluginCnsUrlPort | Valeur wstring (valeur par défaut : L"10090") | statique | Port de l’URL Azure CNS. |
|ContainerServiceArguments|chaîne, valeur par défaut : « -H localhost:2375 -H npipe:// »|statique|Service Fabric (SF) gère le démon Docker (sauf sur les ordinateurs clients Windows comme Win10). Cette configuration permet à l’utilisateur de spécifier des arguments personnalisés qui doivent être passés au démon Docker quand vous le démarrez. Quand des arguments personnalisés sont spécifiés, Service Fabric ne transmet pas d’autres arguments au moteur Docker, à l’exception de l’argument « --pidfile ». Par conséquent, les utilisateurs ne doivent pas spécifier l’argument « --pidfile » dans le cadre de leurs arguments personnalisés. Les arguments personnalisés doivent également vérifier que le démon Docker écoute sur le canal de nom par défaut sur Windows (ou le socket de domaine Unix sur Linux) pour que Service Fabric puisse communiquer avec lui.|
|ContainerServiceLogFileMaxSizeInKb|entier, valeur par défaut : 32768|statique|Taille maximale du fichier journal généré par les conteneurs Docker.  Windows uniquement.|
|ContainerImageDownloadTimeout|valeur int, nombre de secondes, valeur par défaut : 1 200 (20 minutes)|Dynamique|Nombre de secondes avant l’expiration du chargement de l’image.|
|ContainerImagesToSkip|chaîne, noms d’images séparés par le caractère de la ligne verticale, la valeur par défaut est ""|statique|Nom d’une ou plusieurs images de conteneur qui ne doit pas être supprimé.  Utilisé avec le paramètre PruneContainerImages.|
|ContainerServiceLogFileNamePrefix|chaîne, valeur par défaut : sfcontainerlogs|statique|Préfixe du nom de fichier pour les fichiers journaux générés par les conteneurs Docker.  Windows uniquement.|
|ContainerServiceLogFileRetentionCount|entier, valeur par défaut : 10|statique|Nombre de fichiers journaux générés par les conteneurs Docker avant le remplacement des fichiers journaux.  Windows uniquement.|
|CreateFabricRuntimeTimeout|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(120)|Dynamique| Spécifiez la durée en secondes. La valeur du délai d’expiration pour l’appel de synchronisation FabricCreateRuntime |
|DefaultContainerRepositoryAccountName|Chaîne (valeur par défaut : "")|statique|Informations d’identification par défaut utilisées à la place des informations d’identification spécifiées dans ApplicationManifest.xml |
|DefaultContainerRepositoryPassword|Chaîne (valeur par défaut : "")|statique|Informations d’identification de mot de passe par défaut utilisées à la place des informations d’identification spécifiées dans ApplicationManifest.xml|
|DefaultContainerRepositoryPasswordType|Chaîne (valeur par défaut : "")|statique|Quand la valeur n’est pas une chaîne vide, elle peut être « Encrypted » ou « SecretsStoreRef ».|
|DefaultDnsSearchSuffixEmpty|valeur booléenne, valeur par défaut : FALSE|statique|Par défaut, le nom du service est ajouté au nom DNS SF pour les services de conteneur. Cette fonctionnalité met fin à ce comportement, si bien que rien n’est ajouté au nom DNS SF par défaut dans le cheminement de la résolution.|
|DeploymentMaxFailureCount|entier, valeur par défaut : 20| Dynamique|Le déploiement de l’application sera retenté DeploymentMaxFailureCount fois avant l’échec du déploiement de cette application sur le nœud.| 
|DeploymentMaxRetryInterval| TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(3600)|Dynamique| Spécifiez la durée en secondes. Intervalle maximum avant une nouvelle tentative de déploiement. À chaque échec continu, l’intervalle avant nouvelle tentative est calculé de la manière suivante : Min( DeploymentMaxRetryInterval; Nombre d’échecs continus * DeploymentRetryBackoffInterval) |
|DeploymentRetryBackoffInterval| TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(10)|Dynamique|Spécifiez la durée en secondes. Intervalle de temporisation pour l’échec du déploiement. À chaque échec de déploiement continu, le système retentera le déploiement jusqu'à la valeur MaxDeploymentFailureCount. L’intervalle entre chaque tentative est un produit de l’échec du déploiement continu et de l’intervalle de temporisation de déploiement. |
|DisableContainers|valeur booléenne, valeur par défaut : FALSE|statique|Configuration pour la désactivation des conteneurs - utilisé à la place de DisableContainerServiceStartOnContainerActivatorOpen qui est un paramètre de configuration déprécié |
|DisableDockerRequestRetry|valeur booléenne, valeur par défaut : FALSE |Dynamique| Par défaut SF communique avec DD (docker dameon) avec un délai d’expiration « DockerRequestTimeout » pour chaque requête http qui lui est envoyée. Si DD ne répond pas au cours de cette période, SF renvoie la requête s’il reste du temps dans l’opération de niveau supérieur.  Avec le conteneur Hyper-V ; DD prend parfois beaucoup plus de temps pour afficher le conteneur ou le désactiver. Dans ce cas, DD demande un délai d’expiration à partir de la perspective de SF et SF retente l’opération. Parfois, cela semble ajoute plus de pression sur DD. Cette configuration permet de désactiver cette nouvelle tentative et d’attendre la réponse de DD. |
|DnsServerListTwoIps | Valeur booléenne, valeur par défaut : FALSE | statique | Cet indicateur ajoute deux fois le serveur DNS local pour aider à atténuer les problèmes intermittents. |
| DoNotInjectLocalDnsServer | valeur booléenne, valeur par défaut : FALSE | statique | Empêche le runtime d’injecter l’adresse IP locale en tant que serveur DNS pour les conteneurs. |
|EnableActivateNoWindow| valeur booléenne, valeur par défaut : FALSE|Dynamique| Le processus activé est créé en arrière-plan sans aucune console. |
|EnableContainerServiceDebugMode|Valeur booléenne, valeur par défaut : TRUE|statique|Activer/désactiver la journalisation pour les conteneurs Docker.  Windows uniquement.|
|EnableDockerHealthCheckIntegration|Valeur booléenne, valeur par défaut : TRUE|statique|Permet l’intégration des événements de vérification d’intégrité Docker avec le rapport d’intégrité du système Service Fabric |
|EnableProcessDebugging|valeur booléenne, valeur par défaut : FALSE|Dynamique| Permet de lancer des hôtes d’application dans le débogueur |
|EndpointProviderEnabled| valeur booléenne, valeur par défaut : FALSE|statique| Permet la gestion des ressources de points de terminaison par Fabric. Requiert la spécification de la plage de ports d’application de début et de fin dans FabricNode. |
|FabricContainerAppsEnabled| valeur booléenne, valeur par défaut : FALSE|statique| |
|FirewallPolicyEnabled|valeur booléenne, valeur par défaut : FALSE|statique| Permet l’ouverture des ports du pare-feu pour les ressources de points de terminaison avec des ports explicites spécifiés dans ServiceManifest |
|GetCodePackageActivationContextTimeout|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(120)|Dynamique|Spécifiez la durée en secondes. La valeur du délai d’expiration pour les appels CodePackageActivationContext. Cela ne s’applique pas aux services ad hoc. |
|GovernOnlyMainMemoryForProcesses|valeur booléenne, valeur par défaut : FALSE|statique|Le comportement par défaut de la gouvernance des ressources consiste à placer la limite spécifiée dans MemoryInMB sur la quantité de mémoire totale (RAM + permutation) utilisée par le processus. Si la limite est dépassée, le processus reçoit l’exception OutOfMemory. Si ce paramètre est défini sur true, la limite est appliquée seulement à la quantité de mémoire RAM utilisée par un processus. Si cette limite est dépassée et si ce paramètre est true, le système d’exploitation permute la mémoire principale sur le disque. |
|IPProviderEnabled|valeur booléenne, valeur par défaut : FALSE|statique|Permet la gestion des adresses IP. |
|IsDefaultContainerRepositoryPasswordEncrypted|valeur booléenne, valeur par défaut : FALSE|statique|Indique si DefaultContainerRepositoryPassword est chiffré ou non.|
|LinuxExternalExecutablePath|chaîne, valeur par défaut : « /usr/bin/ » |statique|Répertoire principal des commandes exécutables externes sur le nœud.|
|NTLMAuthenticationEnabled|valeur booléenne, valeur par défaut : FALSE|statique| Active la prise en charge pour l’utilisation de NTLM par les packages de code exécutés comme d’autres utilisateurs afin que les processus sur plusieurs ordinateurs puissent communiquer en toute sécurité. |
|NTLMAuthenticationPasswordSecret|SecureString, valeur par défaut : Common::SecureString("")|statique|Valeur chiffrée utilisée pour générer le mot de passe des utilisateurs NTLM. Doit être définie si NTLMAuthenticationEnabled a la valeur true. Validé par le déploiement. |
|NTLMSecurityUsersByX509CommonNamesRefreshInterval|TimeSpan, la valeur par défaut est Common::TimeSpan::FromMinutes(3)|Dynamique|Spécifiez la durée en secondes. Paramètres spécifiques à l’environnement. L’intervalle périodique selon lequel l’hébergement recherche de nouveaux certificats à utiliser pour la configuration NTLM FileStoreService. |
|NTLMSecurityUsersByX509CommonNamesRefreshTimeout|TimeSpan, la valeur par défaut est Common::TimeSpan::FromMinutes(4)|Dynamique| Spécifiez la durée en secondes. Le délai d’attente pour la configuration des utilisateurs NTLM qui se servent de noms de certificats communs. Les utilisateurs NTLM sont nécessaires pour les partages FileStoreService. |
|PruneContainerImages|valeur booléenne, valeur par défaut : FALSE|Dynamique| Supprimer des nœuds les images du conteneur d’application inutilisées. Quand un ApplicationType est désinscrit à partir du cluster Service Fabric, les images du conteneur qui ont été utilisées par cette application seront supprimées sur les nœuds où elle a été téléchargée par Service Fabric. Le nettoyage s’exécute toutes les heures, donc la suppression des images du conteneur peut prendre jusqu’à une heure (plus le temps de nettoyer l’image).<br>Service Fabric ne télécharge jamais ni ne supprime des images non liées à une application.  Les images non liées qui ont été téléchargées manuellement ou autre doivent être supprimées de manière explicite.<br>Les images qui ne doivent pas être supprimées peuvent être spécifiées dans le paramètre ContainerImagesToSkip.| 
|RegisterCodePackageHostTimeout|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(120)|Dynamique| Spécifiez la durée en secondes. La valeur du délai d’expiration pour l’appel de synchronisation FabricRegisterCodePackageHost. Applicable uniquement pour les hôtes d’application avec plusieurs packages de code, par exemple FWP |
|RequestTimeout|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(30)|Dynamique| Spécifiez la durée en secondes. Représente le délai d’expiration de la communication entre l’hôte d’application de l’utilisateur et le processus Fabric pour différentes opérations associées à l’hébergement, notamment l’inscription des paramètres d’usine et l’inscription du runtime. |
|RunAsPolicyEnabled| valeur booléenne, valeur par défaut : FALSE|statique| Permet l’exécution des packages de code en tant qu’un utilisateur local autre que l’utilisateur sous lequel le processus Fabric est en cours d’exécution. Pour activer cette stratégie, Fabric doit être exécuté comme SYSTEM ou en tant qu’utilisateur avec SeAssignPrimaryTokenPrivilege. |
|ServiceFactoryRegistrationTimeout| TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(120)|Dynamique|Spécifiez la durée en secondes. La valeur du délai d’expiration pour l’appel de synchronisation Register(Stateless/Stateful)ServiceFactory |
|ServiceTypeDisableFailureThreshold |Nombre entier, la valeur par défaut est 1 |Dynamique|Il s’agit du seuil pour le nombre d’échecs après lequel FailoverManager (FM) reçoit une notification de désactiver le type de service sur ce nœud et d’essayer un autre nœud pour le positionnement. |
|ServiceTypeDisableGraceInterval|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(30)|Dynamique|Spécifiez la durée en secondes. Intervalle de temps après lequel le type de service peut être désactivé |
|ServiceTypeRegistrationTimeout |Durée en secondes, la valeur par défaut est 300 |Dynamique|Durée maximale autorisée pour que ServiceType soit inscrit auprès de la structure |
|UseContainerServiceArguments|Valeur booléenne, valeur par défaut : TRUE|statique|Cette configuration indique à l’hébergement d’ignorer le passage d’arguments (spécifié dans la configuration ContainerServiceArguments) pour le démon Docker.|

## <a name="httpgateway"></a>HttpGateway

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|ActiveListeners |Valeur Uint (valeur par défaut : 50) |statique| Nombre de lectures à publier dans la file d’attente du serveur HTTP. Ce paramètre contrôle le nombre de requêtes concurrentes que la passerelle HTTP peut satisfaire. |
|HttpGatewayHealthReportSendInterval |Durée en secondes, la valeur par défaut est 30 |statique|Spécifiez la durée en secondes. Délai à l’issue duquel la passerelle HTTP envoie les rapports d’intégrité cumulés à Health Manager. |
|HttpStrictTransportSecurityHeader|Valeur de chaîne, par défaut : ""|Dynamique| Spécifiez la valeur d’en-tête STS HTTP à inclure dans chaque réponse envoyée par HttpGateway. Quand la valeur est une chaîne vide, cet en-tête n’est pas inclus dans la réponse de la passerelle.|
|IsEnabled|Valeur booléenne (valeur par défaut : false) |statique| Active/désactive le paramètre HttpGateway. HttpGateway est désactivé par défaut. |
|MaxEntityBodySize |Valeur Uint (valeur par défaut : 4194304) |Dynamique|Ce paramètre indique la taille maximum du corps que l’on peut attendre d’une requête HTTP. Valeur par défaut : 4 Mo. Le paramètre httpgateway ne peut pas traiter une requête si son corps a une taille supérieure à cette valeur. La taille minimum du bloc de lectures est de 4096 octets. Cette valeur doit donc être supérieure ou égale à 4096. |

## <a name="imagestoreservice"></a>ImageStoreService

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|activé |Valeur booléenne (valeur par défaut : false) |statique|Indicateur d’activation d’ImageStoreService. Par défaut : false |
|MinReplicaSetSize | Entier (valeur par défaut : 3) |statique|Paramètre MinReplicaSetSize pour ImageStoreService. |
|PlacementConstraints | Chaîne (valeur par défaut : "") |statique| Paramètre PlacementConstraints pour ImageStoreService. |
|QuorumLossWaitDuration | Durée en secondes (valeur par défaut : MaxValue) |statique| Spécifiez la durée en secondes. Paramètre QuorumLossWaitDuration pour ImageStoreService. |
|ReplicaRestartWaitDuration | Durée en secondes, la valeur par défaut est 60.0 \* 30 |statique|Spécifiez la durée en secondes. Paramètre ReplicaRestartWaitDuration pour ImageStoreService. |
|StandByReplicaKeepDuration | Durée en secondes, la valeur par défaut est 3600.0 \* 2 |statique| Spécifiez la durée en secondes. Paramètre StandByReplicaKeepDuration pour ImageStoreService. |
|TargetReplicaSetSize | Entier (valeur par défaut : 7) |statique|Paramètre TargetReplicaSetSize pour ImageStoreService. |

## <a name="ktllogger"></a>KtlLogger

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|AutomaticMemoryConfiguration |Entier (valeur par défaut : 1) |Dynamique|Indicateur spécifiant si les paramètres de mémoire doivent être configurés automatiquement et dynamiquement. Si vous spécifiez 0, les paramètres de configuration de la mémoire sont utilisés directement et ne sont pas modifiés par les conditions du système. Si vous spécifiez 1, les paramètres de mémoire sont configurés automatiquement et peuvent varier selon les conditions du système. |
|MaximumDestagingWriteOutstandingInKB | Entier (valeur par défaut : 0) |Dynamique|Nombre de Ko que le journal partagé peut atteindre, en plus du journal dédié. Spécifiez 0 pour indiquer aucune limite.
|SharedLogId |Chaîne (valeur par défaut : "") |statique|GUID unique du conteneur du journal partagé. Utilisez "" si vous choisissez le chemin par défaut sous la racine des données de structure. |
|SharedLogPath |Chaîne (valeur par défaut : "") |statique|Chemin et nom de fichier du conteneur du journal partagé. Utilisez "" pour utiliser le chemin par défaut sous la racine des données de structure. |
|SharedLogSizeInMB |Entier (valeur par défaut : 8192) |statique|Nombre de Mo à allouer dans le conteneur de journal partagé. |
|SharedLogThrottleLimitInPercentUsed|entier (valeur par défaut : 0) | statique | Le pourcentage d’utilisation du journal partagé qui provoque la limitation. La valeur doit être comprise entre 0 et 100. La valeur 0 indique l’utilisation de la valeur de pourcentage par défaut. La valeur 100 indique aucune limitation. Une valeur comprise entre 1 et 99 spécifie le pourcentage d’utilisation de journal au-delà duquel la limitation se produit ; par exemple, si le journal partagé est de 10 Go et la valeur est de 90, la limitation se produit lorsque 9 Go sont utilisés. L’utilisation de la valeur par défaut est recommandée.|
|WriteBufferMemoryPoolMaximumInKB | Entier (valeur par défaut : 0) |Dynamique|Nombre de Ko maximum que le pool de mémoires-tampons d’écritures peut atteindre. Spécifiez 0 pour indiquer aucune limite. |
|WriteBufferMemoryPoolMinimumInKB |Entier (valeur par défaut : 8388608) |Dynamique|Nombre de Ko à attribuer initialement au pool de mémoires-tampons d’écritures. Utilisez 0 pour spécifier aucune limite. La valeur par défaut doit être cohérente avec la valeur du paramètre SharedLogSizeInMB ci-dessous. |

## <a name="managedidentitytokenservice"></a>ManagedIdentityTokenService
| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|IsEnabled|valeur booléenne, valeur par défaut : FALSE|statique|Indicateur contrôlant la présence et l’état du Service de jetons des identités managées dans le cluster. Il s’agit d’une condition préalable à l’utilisation des fonctionnalités d’identité managée des applications Service Fabric.|

## <a name="management"></a>Gestion

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|AutomaticUnprovisionInterval|TimeSpan, valeur par défaut : Common::TimeSpan::FromMinutes(5)|Dynamique|Spécifiez la durée en secondes. Intervalle de nettoyage autorisé pour désinscrire le type d’application au cours du nettoyage de type d’application automatique.|
|AzureStorageMaxConnections | Entier (valeur par défaut : 5000) |Dynamique|Nombre maximum de connexions simultanées au stockage Azure. |
|AzureStorageMaxWorkerThreads | Entier (valeur par défaut : 25) |Dynamique|Nombre maximum de threads de travail en parallèle. |
|AzureStorageOperationTimeout | Durée en secondes (valeur par défaut : 6000) |Dynamique|Spécifiez la durée en secondes. Délai permettant de finaliser l’opération xstore. |
|CleanupApplicationPackageOnProvisionSuccess|Valeur booléenne, valeur par défaut : true |Dynamique|Active ou désactive le nettoyage automatique du package d’application en cas de provisionnement réussi.
|CleanupUnusedApplicationTypes|Valeur booléenne, valeur par défaut : FALSE |Dynamique|Cette configuration, si elle est activée, permet de désinscrire automatiquement des versions inutilisées de types d’applications en ignorant les trois dernières versions inutilisées, ce qui réduit l’espace disque occupé par le magasin d’images. Le nettoyage automatique est déclenché à la fin d’un provisionnement réussi pour ce type d’application spécifique et s’exécute aussi régulièrement une fois par jour pour tous les types d’applications. Le nombre de versions inutilisées à ignorer est configurable à l’aide du paramètre « MaxUnusedAppTypeVersionsToKeep ». <br/> *La bonne pratique est d’utiliser `true`.*
|DisableChecksumValidation | Valeur booléenne (valeur par défaut : false) |statique| Cette configuration nous permet d’activer ou de désactiver la validation de la somme de contrôle pendant l’approvisionnement de l’application. |
|DisableServerSideCopy | Valeur booléenne (valeur par défaut : false) |statique|Cette configuration active ou désactive la copie côté serveur du package d’application sur le magasin ImageStore pendant l’approvisionnement de l’application. |
|ImageCachingEnabled | Valeur booléenne (valeur par défaut : true) |statique|Cette configuration nous permet d’activer ou de désactiver la mise en cache. |
|ImageStoreConnectionString |SecureString |statique|Chaîne de connexion à la racine d’ImageStore. |
|ImageStoreMinimumTransferBPS | Entier (valeur par défaut : 1024) |Dynamique|Débit de transfert minimum entre le cluster et ImageStore. Cette valeur permet de déterminer le délai d’expiration lors de l’accès au magasin ImageStore externe. Ne modifiez cette valeur que si la latence entre le cluster et ImageStore est élevée afin de laisser davantage de temps au cluster pour effectuer des téléchargements à partir du magasin ImageStore externe. |
|MaxUnusedAppTypeVersionsToKeep | Entier (valeur par défaut : 3) |Dynamique|Cette configuration définit le nombre de versions inutilisées de types d’applications à ignorer pour le nettoyage. Ce paramètre s’applique uniquement si le paramètre CleanupUnusedApplicationTypes est activé. <br/>*La meilleure pratique générale consiste à utiliser la valeur par défaut (`3`). Les valeurs inférieures à 1 ne sont pas valides.*|


## <a name="metricactivitythresholds"></a>MetricActivityThresholds
| **Paramètre** | **Valeurs autorisées** |**Stratégie de mise à niveau**| **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|KeyIntegerValueMap, valeur par défaut : None|Dynamique|Détermine l’ensemble de valeurs MetricActivityThresholds pour les métriques dans le cluster. L’équilibrage fonctionnera si maxNodeLoad est supérieure à MetricActivityThresholds. Pour les mesures de défragmentation, il définit la quantité de charge égale à inférieure à laquelle Service Fabric considère que le nœud est vide |

## <a name="metricbalancingthresholds"></a>MetricBalancingThresholds
| **Paramètre** | **Valeurs autorisées** |**Stratégie de mise à niveau**| **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|KeyDoubleValueMap, valeur par défaut : None|Dynamique|Détermine l’ensemble de valeurs MetricBalancingThresholds pour les métriques dans le cluster. L’équilibrage fonctionnera si maxNodeLoad/minNodeLoad est supérieure à MetricBalancingThresholds. La défragmentation fonctionnera si la maxNodeLoad/minNodeLoad dans au moins un FD ou UD est inférieure à MetricBalancingThresholds. |

## <a name="metricloadstickinessforswap"></a>MetricLoadStickinessForSwap
| **Paramètre** | **Valeurs autorisées** |**Stratégie de mise à niveau**| **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|KeyDoubleValueMap, valeur par défaut : None|Dynamique|Détermine la partie de la charge qui s’exécute avec le réplica lorsqu’elle est permutée. La valeur est comprise entre 0 (la charge ne s’exécute pas avec le réplica) et 1 (la charge s’exécute avec le réplica - par défaut) |

## <a name="namingservice"></a>NamingService

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|GatewayServiceDescriptionCacheLimit |Entier (valeur par défaut : 0) |statique|Nombre maximum d’entrées maintenues dans le cache de description du service LRU au niveau de la passerelle d’attribution de noms (0 correspond à aucune limite). |
|MaxClientConnections |Entier, valeur par défaut : 1000 |Dynamique|Nombre maximum autorisé de connexions clientes par passerelle. |
|MaxFileOperationTimeout |Durée en secondes, la valeur par défaut est 30 |Dynamique|Spécifiez la durée en secondes. Délai d’attente maximum autorisé pour l’opération du service de magasin de fichiers. Les requêtes spécifiant un délai d’expiration supérieur seront rejetées. |
|MaxIndexedEmptyPartitions |Entier, valeur par défaut : 1000 |Dynamique|Nombre maximum de partitions vides qui resteront indexées dans le cache de notifications pour synchroniser les clients qui se reconnectent. Toutes les partitions vides au-delà de ce nombre seront supprimées de l’index dans l’ordre croissant des versions de recherche. Les clients qui se reconnectent peuvent toujours synchroniser et recevoir les mises à jour manquantes des partitions vides, mais le protocole de synchronisation devient plus coûteux. |
|MaxMessageSize |Entier, valeur par défaut : 4\*1024\*1024 |statique|Taille maximale des messages pour la communication de nœuds clients lors de l’utilisation du service d’attribution de noms. Atténuation d’attaque par déni de service. Valeur par défaut : 4 Mo. |
|MaxNamingServiceHealthReports | Entier (valeur par défaut : 10) |Dynamique|Nombre maximum d’opérations lentes que le magasin du Service d’attribution de noms signale comme défectueuses à la fois. Si vous spécifiez 0, toutes les opérations lentes sont envoyées. |
|MaxOperationTimeout |Durée en secondes (valeur par défaut : 600) |Dynamique|Spécifiez la durée en secondes. Délai d’attente maximum autorisé pour les opérations du client. Les requêtes spécifiant un délai d’expiration supérieur seront rejetées. |
|MaxOutstandingNotificationsPerClient |Entier, valeur par défaut : 1000 |Dynamique|Nombre maximum de notifications en attente avant la fermeture forcée de l’inscription du client par la passerelle. |
|MinReplicaSetSize | Entier (valeur par défaut : 3) |Non autorisée| Nombre minimum de réplicas du Service d’attribution de noms requis dans lesquels effectuer des écritures pour finaliser une mise à jour. Si le nombre de réplicas actifs dans le système est inférieur à cette valeur, le système de fiabilité refuse les mises à jour du magasin du Service d’attribution de noms jusqu’à ce que les réplicas soient restaurés. Cette valeur ne doit jamais être supérieure à TargetReplicaSetSize. |
|PartitionCount |Entier (valeur par défaut : 3) |Non autorisée|Nombre de partitions du magasin du Service d’attribution de noms à créer. Chaque partition possède sa clé de partition unique qui correspond à son index. Les clés de partition [0; PartitionCount] existent donc. En augmentant le nombre de partitions du Services d’attribution de noms, vous augmentez l’étendue sur laquelle intervient le Service d’attribution de noms, tout en diminuant le volume moyen de données stockées par un jeu de réplicas de secours, moyennant une utilisation accrue des ressources (car les réplicas de service PartitionCount*ReplicaSetSize doivent être maintenus).|
|PlacementConstraints | Chaîne (valeur par défaut : "") |Non autorisée| Contrainte de placement pour le Service d’attribution de noms. |
|QuorumLossWaitDuration | Durée en secondes (valeur par défaut : MaxValue) |Non autorisée| Spécifiez la durée en secondes. Ce minuteur démarre lorsqu’un service d’attribution de noms passe en perte de quorum. Lorsque ce délai est écoulé, le FM considère les réplicas arrêtés comme perdus et tente de rétablir le quorum. Notez que cela peut entraîner une perte de données. |
|RepairInterval | Durée en secondes (valeur par défaut : 5) |statique| Spécifiez la durée en secondes. Durée pendant laquelle la réparation d’une incohérence d’attribution de noms entre le propriétaire de l’autorité et le propriétaire du nom démarre. |
|ReplicaRestartWaitDuration | Durée en secondes (valeur par défaut : 60.0 * 30)|Non autorisée| Spécifiez la durée en secondes. Ce minuteur démarre lorsqu’un réplica du service d’attribution de noms tombe en panne. Lorsqu’il arrive à expiration, le FM commence à remplacer les réplicas qui sont arrêtés (il ne les considère pas encore comme perdus). |
|ServiceDescriptionCacheLimit | Entier (valeur par défaut : 0) |statique| Nombre maximum d’entrées maintenues dans le cache de description du service LRU au niveau du magasin du Service d’attribution de noms (0 correspond à aucune limite). |
|ServiceNotificationTimeout |Durée en secondes, la valeur par défaut est 30 |Dynamique|Spécifiez la durée en secondes. Délai d’expiration utilisé lors de l’envoi des notifications du service au client. |
|StandByReplicaKeepDuration | Durée en secondes (valeur par défaut : 3600.0*2) |Non autorisée| Spécifiez la durée en secondes. Lorsqu’un réplica du Service d’attribution de noms n’est plus défaillant, il peut avoir déjà été remplacé. Ce minuteur détermine la durée pendant laquelle le FM conserve le réplica en attente avant de le rejeter. |
|TargetReplicaSetSize |Entier (valeur par défaut : 7) |Non autorisée|Nombre de jeux de réplicas pour chaque partition de la banque du magasin du service d’attribution de noms. En augmentant le nombre de jeux de réplicas, vous augmentez le niveau de fiabilité des informations contenues dans le magasin du Service d’attribution de noms, ce qui diminue le risque de perte d’informations suite à des défaillances de nœud, moyennant une charge accrue sur Windows Fabric et le temps nécessaire pour mettre à jour les données d’attribution de noms.|

## <a name="nodebufferpercentage"></a>NodeBufferPercentage
| **Paramètre** | **Valeurs autorisées** |**Stratégie de mise à niveau**| **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|KeyDoubleValueMap, valeur par défaut : None|Dynamique|Pourcentage de capacité de nœud par nom de mesure ; utilisé comme mémoire tampon pour maintenir un emplacement disponible sur un nœud en cas de basculement. |

## <a name="nodecapacities"></a>NodeCapacities

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup |NodeCapacityCollectionMap |statique|Collection des capacités du nœud pour les différentes mesures. |

## <a name="nodedomainids"></a>NodeDomainIds

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup |NodeFaultDomainIdCollection |statique|Décrit les domaines d’erreur auquel un nœud appartient. Le domaine par défaut est défini par un URI qui décrit l’emplacement du nœud dans le centre de données.  Les URI de domaine d’erreur sont au format de:/de suivi d’un segment de chemin d’accès d’URI.|
|UpgradeDomainId |Chaîne (valeur par défaut : "") |statique|Décrit le domaine de mise à niveau auquel un nœud appartient. |

## <a name="nodeproperties"></a>NodeProperties

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup |NodePropertyCollectionMap |statique|Collection de paires clé-valeur de chaîne pour les propriétés du nœud. |

## <a name="paas"></a>PaaS

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|ClusterId |Chaîne (valeur par défaut : "") |Non autorisée|Magasin de certificats X509 utilisé par l’infrastructure pour protéger la configuration. |

## <a name="performancecounterlocalstore"></a>PerformanceCounterLocalStore

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|Counters |String | Dynamique |Liste séparée par des virgules, des compteurs de performance à collecter. |
|IsEnabled |Valeur booléenne (valeur par défaut : true) | Dynamique |Indicateur spécifiant si la collection de compteurs de performance sur le nœud local est activée. |
|MaxCounterBinaryFileSizeInMB |Entier (valeur par défaut : 1) | Dynamique |Taille maximale (en Mo) de chaque fichier binaire de compteur de performances. |
|NewCounterBinaryFileCreationIntervalInMinutes |Entier (valeur par défaut : 10) | Dynamique |Durée maximale (en secondes) après lequel un fichier binaire de compteur de performances est créé. |
|SamplingIntervalInSeconds |Entier (valeur par défaut : 60) | Dynamique |Intervalle d’échantillonnage des compteurs de performances collectés. |

## <a name="placementandloadbalancing"></a>PlacementAndLoadBalancing

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|AffinityConstraintPriority | Entier (valeur par défaut : 0) | Dynamique|Détermine la priorité de la contrainte d’affinité : 0 : Stricte ; 1 : Souple ; valeur négative : à ignorer. |
|ApplicationCapacityConstraintPriority | Entier (valeur par défaut : 0) | Dynamique|Détermine la priorité de la contrainte de capacité : 0 : Stricte ; 1 : Souple ; valeur négative : à ignorer. |
|AutoDetectAvailableResources|Valeur booléenne, valeur par défaut : TRUE|statique|Cette configuration déclenchera la détection automatique des ressources disponibles sur le nœud (processeur et mémoire). Si cette configuration est définie sur true, nous lirons les capacités réelles et les corrigerons si l’utilisateur a spécifié des capacités de nœud incorrectes ou ne les a pas définies du tout. Si cette configuration est définie sur false, nous enverrons un avertissement que l’utilisateur a spécifié des capacités de nœud incorrectes, mais nous ne les corrigerons pas. Ce qui signifie que cet utilisateur souhaite que les capacités spécifiées soient > à celles du nœud ou si les capacités n’ont pas été définies ; cette valeur suppose une capacité illimitée |
|BalancingDelayAfterNewNode | Durée en secondes (valeur par défaut : 120) |Dynamique|Spécifiez la durée en secondes. Ne démarrez pas l’équilibrage des activités pendant cette période après l’ajout d’un nouveau nœud. |
|BalancingDelayAfterNodeDown | Durée en secondes (valeur par défaut : 120) |Dynamique|Spécifiez la durée en secondes. Ne démarrez pas l’équilibrage des activités pendant cette période après un événement d’arrêt de nœud. |
|BlockNodeInUpgradeConstraintPriority | Entier (valeur par défaut : 0) |Dynamique|Détermine la priorité de la contrainte de capacité : 0 : Stricte ; 1 : Souple ; valeur négative : Ignorer  |
|CapacityConstraintPriority | Entier (valeur par défaut : 0) | Dynamique|Détermine la priorité de la contrainte de capacité : 0 : Stricte ; 1 : Souple ; valeur négative : à ignorer. |
|ConsecutiveDroppedMovementsHealthReportLimit | Entier (valeur par défaut : 20) | Dynamique|Définit le nombre de fois consécutives que des mouvements émis par ResourceBalancer sont supprimés avant que des diagnostics soient effectués et que des avertissements d’intégrité soient émis. Négatif : aucun avertissement émis sous cette condition. |
|ConstraintFixPartialDelayAfterNewNode | Durée en secondes (valeur par défaut : 120) |Dynamique| Spécifiez la durée en secondes. Ne corrigez pas les violations de contrainte FaultDomain et UpgradeDomain pendant cette période après l’ajout d’un nouveau nœud. |
|ConstraintFixPartialDelayAfterNodeDown | Durée en secondes (valeur par défaut : 120) |Dynamique| Spécifiez la durée en secondes. Ne corrigez pas les violations de contrainte FaultDomain et UpgradeDomain pendant cette période après un événement d’arrêt de nœud. |
|ConstraintViolationHealthReportLimit | Entier (valeur par défaut : 50) |Dynamique| Définit le nombre de fois qu’un réplica ne respectant pas la contrainte ne doit pas être corrigé de façon permanente avant que des diagnostics soient effectués et que des rapports d’intégrité soient émis. |
|DetailedConstraintViolationHealthReportLimit | Entier (valeur par défaut : 200) |Dynamique| Définit le nombre de fois qu’un réplica ne respectant pas la contrainte ne doit pas être corrigé de façon permanente avant que des diagnostics soient effectués et que des rapports d’intégrité détaillés soient émis. |
|DetailedDiagnosticsInfoListLimit | Entier (valeur par défaut : 15) |Dynamique| Définit le nombre d’entrées de diagnostic (avec informations détaillées) par contrainte à inclure avant la troncation dans Diagnostics.|
|DetailedNodeListLimit | Entier (valeur par défaut : 15) |Dynamique| Définit le nombre de nœuds par contrainte à inclure avant la troncation dans les rapports de réplica non placé. |
|DetailedPartitionListLimit | Entier (valeur par défaut : 15) |Dynamique| Définit le nombre de partitions par entrée de diagnostic pour une contrainte à inclure avant la troncation dans Diagnostics. |
|DetailedVerboseHealthReportLimit | Entier (valeur par défaut : 200) | Dynamique|Définit le nombre de fois qu’un réplica non placé doit être non placé de façon permanente avant que des rapports d’intégrité détaillés soient émis. |
|EnforceUserServiceMetricCapacities|valeur booléenne, valeur par défaut : FALSE | statique |Active la protection des services de structure. Tous les services utilisateur relèvent d’un même objet de traitement/groupe de travaux et sont limités à une quantité spécifiée de ressources. Il doit être statique (nécessite le redémarrage de FabricHost), car la création/suppression de l’objet de traitement utilisateur et des limites de paramètres s’effectue pendant l’ouverture de Fabric Host. |
|FaultDomainConstraintPriority | Entier (valeur par défaut : 0) |Dynamique| Détermine la priorité de la contrainte de domaine d’erreur : 0 : Stricte ; 1 : Souple ; valeur négative : à ignorer. |
|GlobalMovementThrottleCountingInterval | Durée en secondes (valeur par défaut : 600) |statique| Spécifiez la durée en secondes. Indiquez la durée de l’intervalle écoulé pendant lequel effectuer le suivi des mouvements de réplica de domaine (utilisés avec GlobalMovementThrottleThreshold). Réglez cette valeur sur 0 pour ignorer complètement la limitation globale. |
|GlobalMovementThrottleThreshold | Valeur Uint (valeur par défaut : 1000) |Dynamique| Nombre maximum de mouvements autorisés dans la phase d’équilibrage de l’intervalle écoulé spécifié par GlobalMovementThrottleCountingInterval. |
|GlobalMovementThrottleThresholdForBalancing | Valeur Uint (valeur par défaut : 0) | Dynamique|Nombre maximum de mouvements autorisés dans la phase d’équilibrage de l’intervalle écoulé spécifié par GlobalMovementThrottleCountingInterval. 0 indique aucune limite. |
|GlobalMovementThrottleThresholdForPlacement | Valeur Uint (valeur par défaut : 0) |Dynamique| Nombre maximum de mouvements autorisés dans la phase d’équilibrage de l’intervalle écoulé spécifié par GlobalMovementThrottleCountingInterval. 0 indique aucune limite.|
|GlobalMovementThrottleThresholdPercentage|double, valeur par défaut : 0|Dynamique|Nombre maximal de mouvements totaux autorisés dans les phases d’équilibrage et de placement (exprimé en pourcentage du nombre total de réplicas dans le cluster) dans l’intervalle écoulé indiqué par GlobalMovementThrottleCountingInterval. 0 indique aucune limite. Si ce paramètre et le paramètre GlobalMovementThrottleThreshold sont tous deux spécifiés, la limite la plus conservatrice est utilisée.|
|GlobalMovementThrottleThresholdPercentageForBalancing|double, valeur par défaut : 0|Dynamique|Nombre maximal de mouvements autorisés dans la phase d’équilibrage (exprimé en pourcentage du nombre total de réplicas dans PLB) dans l’intervalle écoulé indiqué par GlobalMovementThrottleCountingInterval. 0 indique aucune limite. Si ce paramètre et le paramètre GlobalMovementThrottleThresholdForBalancing sont tous deux spécifiés, la limite la plus conservatrice est utilisée.|
|InBuildThrottlingAssociatedMetric | Chaîne (valeur par défaut : "") |statique| Nom de la mesure associée à cette limitation. |
|InBuildThrottlingEnabled | Valeur booléenne (valeur par défaut : false) |Dynamique| Détermine si la limitation intégrée est activée. |
|InBuildThrottlingGlobalMaxValue | Entier (valeur par défaut : 0) |Dynamique|Nombre maximum de réplicas intégrés autorisés globalement. |
|InterruptBalancingForAllFailoverUnitUpdates | Valeur booléenne (valeur par défaut : false) | Dynamique|Détermine si n’importe quel type de mise à jour des unités de basculement doit interrompre l’exécution lente ou rapide de l’équilibrage. Avec la valeur « false », l’exécution de l’équilibrage est interrompue si FailoverUnit est créé/supprimé, a des réplicas manquants, a modifié l’emplacement du réplica principal ou a modifié le nombre de réplicas. L’exécution de l’équilibrage n’est PAS interrompue si FailoverUnit a des réplicas supplémentaires, a modifié un indicateur de réplica, n’a modifié que la version de la partition ou dans tout autre cas. |
|MinConstraintCheckInterval | Durée en secondes (valeur par défaut : 1) |Dynamique| Spécifiez la durée en secondes. Définit le délai minimum à respecter avant deux passes de vérification de contrainte consécutives. |
|MinLoadBalancingInterval | Durée en secondes (valeur par défaut : 5) |Dynamique| Spécifiez la durée en secondes. Définit le délai minimum à respecter avant deux passes d’équilibrage consécutives. |
|MinPlacementInterval | Durée en secondes (valeur par défaut : 1) |Dynamique| Spécifiez la durée en secondes. Définit le délai minimum à respecter avant deux passes de placement consécutives. |
|MoveExistingReplicaForPlacement | Valeur booléenne (valeur par défaut : true) |Dynamique|Paramètre déterminant si le réplica doit être déplacé pendant le placement. |
|MovementPerPartitionThrottleCountingInterval | Durée en secondes (valeur par défaut : 600) |statique| Spécifiez la durée en secondes. Indiquez la durée de l’intervalle écoulé pendant lequel effectuer le suivi des mouvements de réplica pour chaque partition (utilisée avec MovementPerPartitionThrottleThreshold). |
|MovementPerPartitionThrottleThreshold | Valeur Uint (valeur par défaut : 50) |Dynamique| Aucun mouvement d’équilibrage n’est effectué pour une partition si le nombre d’équilibrages des réplicas de cette partition a atteint ou dépassé la valeur MovementPerFailoverUnitThrottleThreshold pendant la période écoulée spécifiée par MovementPerPartitionThrottleCountingInterval. |
|MoveParentToFixAffinityViolation | Valeur booléenne (valeur par défaut : false) |Dynamique| Paramètre déterminant si les réplicas parents peuvent être déplacés pour corriger les contraintes d’affinité.|
|PartiallyPlaceServices | Valeur booléenne (valeur par défaut : true) |Dynamique| Détermine si tous les réplicas de service dans le cluster seront placés selon le principe du « tout ou rien », en fonction du nombre limité de nœuds appropriés pour eux.|
|PlaceChildWithoutParent | Valeur booléenne (valeur par défaut : true) | Dynamique|Paramètre déterminant si le réplica de service enfant peut être placé si aucun réplica parent n’est actif. |
|PlacementConstraintPriority | Entier (valeur par défaut : 0) | Dynamique|Détermine la priorité de la contrainte de placement : 0 : Stricte ; 1 : Souple ; valeur négative : à ignorer. |
|PlacementConstraintValidationCacheSize | Entier (valeur par défaut : 10000) |Dynamique| Limite la taille de la table utilisée pour la validation et la mise en cache rapides des expressions de contrainte de placement. |
|PlacementSearchTimeout | Durée en secondes (valeur par défaut est 0.5) |Dynamique| Spécifiez la durée en secondes. Lors du placement de services, effectuez une recherche au moins aussi longue avant de renvoyer un résultat. |
|PLBRefreshGap | Durée en secondes (valeur par défaut : 1) |Dynamique| Spécifiez la durée en secondes. Définit le délai minimum à respecter avant que PLB ne réactualise l’état. |
|PreferredLocationConstraintPriority | Entier (valeur par défaut : 2)| Dynamique|Détermine la priorité de la contrainte d’emplacement préféré : 0 : Stricte ; 1 : Souple ; 2 : optimisation ; valeur négative : Ignorer |
|PreferredPrimaryDomainsConstraintPriority| Entier (valeur par défaut : 1) | Dynamique| Détermine la priorité de la contrainte de domaine principal préféré : 0 : Stricte ; 1 : Souple ; valeur négative : Ignorer |
|PreferUpgradedUDs|valeur booléenne,valeur par défaut : FALSE|Dynamique|Active ou désactive la logique qui préfère le déplacement vers des UD déjà mis à niveau. À partir de SF 7.0, la valeur par défaut de ce paramètre passe de TRUE à FALSE.|
|PreventTransientOvercommit | Valeur booléenne (valeur par défaut : false) | Dynamique|Détermine si PLB doit immédiatement compter sur les ressources qui seront libérées par les déplacements initiés. Par défaut, PLB peut initier un mouvement sortant et un mouvement entrant sur le même nœud, ce qui peut créer une survalidation temporaire. Le réglage de ce paramètre sur true empêche ces types de survalidation et désactive la défragmentation à la demande (également appelée placementWithMove). |
|ScaleoutCountConstraintPriority | Entier (valeur par défaut : 0) |Dynamique| Détermine la priorité de la contrainte de nombre d’opérations de scale-out : 0 : Stricte ; 1 : Souple ; valeur négative : à ignorer. |
|SubclusteringEnabled|Valeur booléenne, valeur par défaut : FALSE | Dynamique |Reconnaît le sous-clustering lors du calcul de l’écart-type pour l’équilibrage |
|SubclusteringReportingPolicy| Entier (valeur par défaut : 1) |Dynamique|Définit l’activation et le mode d’envoi des rapports d’intégrité du sous-clustering : 0 : Ne pas envoyer de rapport ; 1 : Avertissement ; 2 : OK |
|SwapPrimaryThrottlingAssociatedMetric | Chaîne (valeur par défaut : "")|statique| Nom de la mesure associée à cette limitation. |
|SwapPrimaryThrottlingEnabled | Valeur booléenne (valeur par défaut : false)|Dynamique| Détermine si la limitation de basculement vers le réplica principal est activée. |
|SwapPrimaryThrottlingGlobalMaxValue | Entier (valeur par défaut : 0) |Dynamique| Nombre maximum de réplicas principaux basculés autorisés globalement. |
|TraceCRMReasons |Valeur booléenne (valeur par défaut : true) |Dynamique|Spécifie si les raisons des mouvements effectués par CRM doivent être envoyés au canal des événements opérationnels. |
|UpgradeDomainConstraintPriority | Entier (valeur par défaut : 1)| Dynamique|Détermine la priorité de la contrainte de domaine de mise à niveau : 0 : Stricte ; 1 : Souple ; valeur négative : à ignorer. |
|UseMoveCostReports | Valeur booléenne (valeur par défaut : false) | Dynamique|Demande à LB d’ignorer l’élément de coût de la fonction de score, ce qui génère un nombre potentiellement important de déplacements et, donc, un placement mieux équilibré. |
|UseSeparateSecondaryLoad | Valeur booléenne (valeur par défaut : true) | Dynamique|Paramètre déterminant si une charge distincte doit être utilisée pour les réplicas secondaires. |
|UseSeparateSecondaryMoveCost | Valeur booléenne (valeur par défaut : false) | Dynamique|Paramètre déterminant si un coût de déplacement distinct doit être utilisé pour les réplicas secondaires. |
|ValidatePlacementConstraint | Valeur booléenne (valeur par défaut : true) |Dynamique| Spécifie si l’expression PlacementConstraint d’un service est validée lors de la mise à jour du paramètre ServiceDescription d’un service. |
|ValidatePrimaryPlacementConstraintOnPromote| Valeur booléenne, valeur par défaut : TRUE |Dynamique|Spécifie si l’expression PlacementConstraint pour un service est évaluée ou non pour la préférence principale lors du basculement. |
|VerboseHealthReportLimit | Entier (valeur par défaut : 20) | Dynamique|Définit le nombre de fois que le placement d’un réplica doit être annulé avant qu’un avertissement d’intégrité ne soit émis (si le rapport d’intégrité détaillé est activé). |
|NodeLoadsOperationalTracingEnabled | Valeur booléenne (valeur par défaut : true) |Dynamique|Configuration qui active la trace structurelle opérationnelle du chargement de nœuds dans le magasin d’événements. |
|NodeLoadsOperationalTracingInterval | TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(20) | Dynamique|Spécifiez la durée en secondes. Intervalle avec lequel tracer les chargements de nœuds dans le magasin d’événements pour chaque domaine de service. |

## <a name="reconfigurationagent"></a>ReconfigurationAgent

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|ApplicationUpgradeMaxReplicaCloseDuration | Durée en secondes (valeur par défaut : 900) |Dynamique|Spécifiez la durée en secondes. Délai respecté par le système avant de mettre fin à des hôtes de service dont des réplicas sont bloqués en fermeture lors de la mise à niveau d’une application.|
|FabricUpgradeMaxReplicaCloseDuration | Durée en secondes (valeur par défaut : 900) |Dynamique| Spécifiez la durée en secondes. Délai respecté par le système avant de mettre fin à des hôtes de service dont des réplicas sont bloqués en fermeture lors de la mise à niveau de Fabric. |
|GracefulReplicaShutdownMaxDuration|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(120)|Dynamique|Spécifiez la durée en secondes. Délai respecté par le système avant de mettre fin à des hôtes de service dont des réplicas sont bloqués en fermeture. Si cette valeur est définie sur 0, les réplicas ne reçoivent pas d’instruction de fermeture.|
|NodeDeactivationMaxReplicaCloseDuration | Durée en secondes (valeur par défaut : 900) |Dynamique|Spécifiez la durée en secondes. Délai respecté par le système avant de mettre fin à des hôtes de service dont des réplicas sont bloqués en fermeture lors de la désactivation d’un nœud. |
|PeriodicApiSlowTraceInterval | Durée en secondes (valeur par défaut : 5 minutes) |Dynamique| Spécifiez la durée en secondes. PeriodicApiSlowTraceInterval définit la période pendant laquelle les appels d’API lents sont retracés par le moniteur d’API. |
|ReplicaChangeRoleFailureRestartThreshold|entier, valeur par défaut : 10|Dynamique| Entier. Spécifiez le nombre d’échecs d’API pendant la promotion principale après lesquels une action d’atténuation d’automatique (redémarrage du réplica) sera appliquée. |
|ReplicaChangeRoleFailureWarningReportThreshold|entier, valeur par défaut : 2147483647|Dynamique| Entier. Spécifiez le nombre d’échecs d’API pendant la promotion principale après lesquels des rapports d’intégrité Warning seront générés.|
|ServiceApiHealthDuration | Durée en secondes (valeur par défaut : 30 minutes) |Dynamique| Spécifiez la durée en secondes. ServiceApiHealthDuration définit le délai d’attente à respecter pour exécuter une API de service avant qu’elle soit signalée comme défectueuse. |
|ServiceReconfigurationApiHealthDuration | Durée en secondes, la valeur par défaut est 30 |Dynamique| Spécifiez la durée en secondes. ServiceReconfigurationApiHealthDuration définit le délai d’attente à respecter pour exécuter une API de service avant qu’elle soit signalée comme défectueuse. S’applique aux appels d’API qui affectent la disponibilité.|

## <a name="replication"></a>Réplication
| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau**| **Conseils ou brève description** |
| --- | --- | --- | --- |
|BatchAcknowledgementInterval|TimeSpan, la valeur par défaut est Common::TimeSpan::FromMilliseconds(15)|statique|Spécifiez la durée en secondes. Détermine le délai respecté par le réplicateur après la réception d’une opération et avant de renvoyer un accusé de réception. Les autres opérations reçues pendant cette période verront leur accusé de réception renvoyé dans un seul message, ce qui diminue le trafic réseau mais peut potentiellement réduire le débit du réplicateur.|
|MaxCopyQueueSize|uint, valeur par défaut : 1024|statique|Valeur maximale définissant la taille initiale de la file d’attente qui gère les opérations de réplication. Ce nombre doit être une puissance de 2. Si, pendant l’exécution, la file d’attente atteint cette taille, les opérations sont limitées entre les réplicateurs principal et secondaire.|
|MaxPrimaryReplicationQueueMemorySize|uint, valeur par défaut : 0|statique|Valeur maximale de la file d’attente de réplication principale en octets.|
|MaxPrimaryReplicationQueueSize|uint, valeur par défaut : 1024|statique|Nombre maximum d’opérations pouvant exister dans la file d’attente de réplication principale. Ce nombre doit être une puissance de 2.|
|MaxReplicationMessageSize|uint, valeur par défaut : 52428800|statique|Taille maximale des messages des opérations de réplication. Valeur par défaut : 50 Mo.|
|MaxSecondaryReplicationQueueMemorySize|uint, valeur par défaut : 0|statique|Valeur maximale de la file d’attente de réplication secondaire en octets.|
|MaxSecondaryReplicationQueueSize|uint, valeur par défaut : 2048|statique|Nombre maximum d’opérations pouvant exister dans la file d’attente de réplication secondaire. Ce nombre doit être une puissance de 2.|
|QueueHealthMonitoringInterval|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(30)|statique|Spécifiez la durée en secondes. Cette valeur détermine la période de temps utilisée par le réplicateur pour surveiller tous les événements de contrôle d’intégrité d’erreur/avertissement dans les files d’attente des opérations de réplication. Une valeur '0' désactive le contrôle d’intégrité |
|QueueHealthWarningAtUsagePercent|uint, valeur par défaut : 80|statique|Cette valeur détermine l’utilisation de la file d’attente de réplication (en pourcentage) après laquelle nous envoyons un avertissement pour utilisation intensive de la file d’attente. Nous effectuons cette opération après un délai de grâce de QueueHealthMonitoringInterval. Si l’utilisation de la file d’attente est inférieure au pourcentage de ce délai de grâce|
|ReplicatorAddress|Chaîne (valeur par défaut : "localhost:0")|statique|Point de terminaison sous la forme d’une chaîne « IP:port» utilisée par le réplicateur Windows Fabric pour se connecter à d’autres réplicas afin d’envoyer/recevoir des opérations.|
|ReplicatorListenAddress|Chaîne (valeur par défaut : "localhost:0")|statique|Point de terminaison sous la forme d’une chaîne 'IP:Port' utilisée par le réplicateur Windows Fabric pour recevoir des opérations d’autres réplicas.|
|ReplicatorPublishAddress|Chaîne (valeur par défaut : "localhost:0")|statique|Point de terminaison sous la forme d’une chaîne 'IP:Port' utilisée par le réplicateur Windows Fabric pour envoyer des opérations à d’autres réplicas.|
|RetryInterval|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(5)|statique|Spécifiez la durée en secondes. Lorsqu’une opération est perdue ou rejetée, cette minuterie détermine la fréquence à laquelle le réplicateur réessaiera d’envoyer l’opération.|

## <a name="resourcemonitorservice"></a>ResourceMonitorService
| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau**| **Conseils ou brève description** |
| --- | --- | --- | --- |
|IsEnabled|valeur booléenne, valeur par défaut : FALSE |statique|Contrôle si le service est activé, ou non, dans le cluster. |

## <a name="runas"></a>RunAs

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|RunAsAccountName |Chaîne (valeur par défaut : "") |Dynamique|Indique le nom du compte RunAs. Ce paramètre n’est nécessaire que pour le type de compte DomainUser ou ManagedServiceAccount. Les valeurs autorisées sont "domain\user" ou "user@domain". |
|RunAsAccountType|Chaîne (valeur par défaut : "") |Dynamique|Indique le type du compte RunAs. Ce paramètre est nécessaire pour toutes les sections RunAs. Les valeurs autorisées DomainUser/NetworkService/ManagedServiceAccount/LocalSystem.|
|RunAsPassword|Chaîne (valeur par défaut : "") |Dynamique|Indique le mot de passe du compte RunAs. Ce paramètre n’est nécessaire que pour le type de compte DomainUser. |

## <a name="runas_dca"></a>RunAs_DCA

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|RunAsAccountName |Chaîne (valeur par défaut : "") |Dynamique|Indique le nom du compte RunAs. Ce paramètre n’est nécessaire que pour le type de compte DomainUser ou ManagedServiceAccount. Les valeurs autorisées sont "domain\user" ou "user@domain". |
|RunAsAccountType|Chaîne (valeur par défaut : "") |Dynamique|Indique le type du compte RunAs. Ce paramètre est nécessaire pour toutes les sections RunAs. Les valeurs autorisées sont LocalUser/DomainUser/NetworkService/ManagedServiceAccount/LocalSystem. |
|RunAsPassword|Chaîne (valeur par défaut : "") |Dynamique|Indique le mot de passe du compte RunAs. Ce paramètre n’est nécessaire que pour le type de compte DomainUser. |

## <a name="runas_fabric"></a>RunAs_Fabric

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|RunAsAccountName |Chaîne (valeur par défaut : "") |Dynamique|Indique le nom du compte RunAs. Ce paramètre n’est nécessaire que pour le type de compte DomainUser ou ManagedServiceAccount. Les valeurs autorisées sont "domain\user" ou "user@domain". |
|RunAsAccountType|Chaîne (valeur par défaut : "") |Dynamique|Indique le type du compte RunAs. Ce paramètre est nécessaire pour toutes les sections RunAs. Les valeurs autorisées sont LocalUser/DomainUser/NetworkService/ManagedServiceAccount/LocalSystem. |
|RunAsPassword|Chaîne (valeur par défaut : "") |Dynamique|Indique le mot de passe du compte RunAs. Ce paramètre n’est nécessaire que pour le type de compte DomainUser. |

## <a name="runas_httpgateway"></a>RunAs_HttpGateway

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|RunAsAccountName |Chaîne (valeur par défaut : "") |Dynamique|Indique le nom du compte RunAs. Ce paramètre n’est nécessaire que pour le type de compte DomainUser ou ManagedServiceAccount. Les valeurs autorisées sont "domain\user" ou "user@domain". |
|RunAsAccountType|Chaîne (valeur par défaut : "") |Dynamique|Indique le type du compte RunAs. Ce paramètre est nécessaire pour toutes les sections RunAs. Les valeurs autorisées sont LocalUser/DomainUser/NetworkService/ManagedServiceAccount/LocalSystem. |
|RunAsPassword|Chaîne (valeur par défaut : "") |Dynamique|Indique le mot de passe du compte RunAs. Ce paramètre n’est nécessaire que pour le type de compte DomainUser. |

## <a name="security"></a>Sécurité
| **Paramètre** | **Valeurs autorisées** |**Stratégie de mise à niveau**| **Conseils ou brève description** |
| --- | --- | --- | --- |
|AADCertEndpointFormat|Chaîne (valeur par défaut : "")|statique|Format du point de terminaison du certificat AAD, Azure Commercial par défaut, spécifié pour un environnement autre que par défaut comme Azure Government « https:\//login.microsoftonline.us/{0}/federationmetadata/2007-06/federationmetadata.xml » |
|AADClientApplication|Chaîne (valeur par défaut : "")|statique|Nom de l’application cliente native ou ID représentant les clients Fabric |
|AADClusterApplication|Chaîne (valeur par défaut : "")|statique|Nom de l’application API Web ou ID représentant le cluster |
|AADLoginEndpoint|Chaîne (valeur par défaut : "")|statique|Point de terminaison de connexion AAD, Azure Commercial par défaut, spécifié pour un environnement autre que par défaut comme Azure Government « https:\//login.microsoftonline.us » |
|AADTenantId|Chaîne (valeur par défaut : "")|statique|ID de client (GUID) |
|AcceptExpiredPinnedClusterCertificate|valeur booléenne, valeur par défaut : FALSE|Dynamique|Indicateur déterminant si les certificats de cluster expirés déclarés par empreinte doivent être acceptés. S'applique uniquement aux certificats de cluster afin de maintenir le cluster actif. |
|AdminClientCertThumbprints|Chaîne (valeur par défaut : "")|Dynamique|Empreintes de certificats utilisées par les clients dans le rôle d’administrateur. Il s’agit d’une liste de noms séparés par des virgules. |
|AADTokenEndpointFormat|Chaîne (valeur par défaut : "")|statique|Point de terminaison de jeton AAD, Azure Commercial par défaut, spécifié pour un environnement autre que par défaut comme Azure Government « https:\//login.microsoftonline.us/{0} » |
|AdminClientClaims|Chaîne (valeur par défaut : "")|Dynamique|Toutes les revendications possibles attendues des clients d’administration ; même format que ClientClaims ; cette liste est ajoutée en interne à ClientClaims ; par conséquent, pas besoin d’ajouter également les mêmes entrées à ClientClaims. |
|AdminClientIdentities|Chaîne (valeur par défaut : "")|Dynamique|Identités Windows des clients de la structure dans le rôle d’administrateur ; utilisé pour autoriser des opérations de structure privilégiées. Il s’agit d’une liste séparée par des virgules ; chaque entrée est un nom de compte de domaine ou nom de groupe. Pour des raisons pratiques ; le compte qui exécute fabric.exe reçoit automatiquement un rôle d’administrateur ; par conséquent, le groupe ServiceFabricAdministrators l’est aussi. |
|AppRunAsAccountGroupX509Folder|valeur de chaîne, valeur par défaut : /home/sfuser/sfusercerts |statique|Dossier où se trouvent les certificats X509 et les clés privées de AppRunAsAccountGroup |
|CertificateExpirySafetyMargin|TimeSpan, la valeur par défaut est Common::TimeSpan::FromMinutes(43200)|statique|Spécifiez la durée en secondes. Marge de sécurité pour l’expiration du certificat ; l’état d’intégrité du certificat passe de OK à Warning lorsque le délai d’expiration est plus proche que cette valeur. La valeur par défaut est 30 jours. |
|CertificateHealthReportingInterval|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(3600 * 8)|statique|Spécifiez la durée en secondes. Spécifiez l’intervalle de génération des rapports d’intégrité de certificats ; la valeur par défaut est 8 heures ; Si défini sur 0, désactive le rapport de contrôle d’intégrité de certificat |
|ClientCertThumbprints|Chaîne (valeur par défaut : "")|Dynamique|Empreintes de certificats utilisés par les clients pour communiquer avec le cluster ; le cluster utilise ce paramètre pour autoriser la connexion entrante. Il s’agit d’une liste de noms séparés par des virgules. |
|ClientClaimAuthEnabled|valeur booléenne, valeur par défaut : FALSE|statique|Indique si l’authentification basée sur les revendications est activée sur les clients ; si définie sur true, définit implicitement ClientRoleEnabled. |
|ClientClaims|Chaîne (valeur par défaut : "")|Dynamique|Toutes les revendications possibles attendues des clients pour se connecter à la passerelle. Il s’agit d’une liste 'OR' : ClaimsEntry \|\| ClaimsEntry \|\| ClaimsEntry ... chaque ClaimsEntry est une liste "AND" : ClaimType=ClaimValue && ClaimType=ClaimValue && ClaimType=ClaimValue ... |
|ClientIdentities|Chaîne (valeur par défaut : "")|Dynamique|Identités Windows de FabricClient ; la passerelle d’affectation de noms utilise ce paramètre pour autoriser les connexions entrantes. Il s’agit d’une liste séparée par des virgules ; chaque entrée est un nom de compte de domaine ou nom de groupe. Pour des raisons pratiques ; le compte qui exécute fabric.exe est autorisé automatiquement ; par conséquent, les groupes ServiceFabricAllowedUsers et ServiceFabricAdministrators le sont aussi. |
|ClientRoleEnabled|valeur booléenne, valeur par défaut : FALSE|statique|Indique si le rôle de client est activé ; si cette valeur est définie True ; les clients reçoivent des rôles selon leur identité. Pour V2 ; activer cette option signifie que le client qui ne figure pas dans AdminClientCommonNames/AdminClientIdentities peut uniquement exécuter des opérations en lecture seule. |
|ClusterCertThumbprints|Chaîne (valeur par défaut : "")|Dynamique|Empreintes de certificats autorisées à rejoindre le cluster ; une liste de noms séparés par des virgules. |
|ClusterCredentialType|Chaîne (valeur par défaut : "None")|Non autorisée|Indique le type d’informations d’identification de sécurité à utiliser pour sécuriser le cluster. Les valeurs valides sont "None/X509/Windows" |
|ClusterIdentities|Chaîne (valeur par défaut : "")|Dynamique|Identités Windows des nœuds de cluster ; utilisé pour l’autorisation de l’appartenance au cluster. Il s’agit d’une liste séparée par des virgules ; chaque entrée est un nom de compte de domaine ou nom de groupe |
|ClusterSpn|Chaîne (valeur par défaut : "")|Non autorisée|Nom de principal du service du cluster ; lorsque Fabric s’exécute en tant qu’utilisateur de domaine unique (gMSA/compte d’utilisateur de domaine). Il s’agit du nom de principal de service (SPN) des écouteurs de bail et écouteurs dans fabric.exe : écouteurs de fédération ; écouteurs de réplication internes ; écouteur de service runtime et écouteur de la passerelle d’affectation de noms. Doit être vide lorsque la structure s’exécute en tant que comptes d’ordinateur ; dans ce cas, connecter le SPN de l’écouteur côté ordinateur à partir de l’adresse de transport de l’écouteur. |
|CrlCheckingFlag|uint, valeur par défaut : 0x40000000|Dynamique|Indicateur de validation de chaîne de certificats par défaut ; peut être remplacé par un indicateur spécifique au composant ; par exemple Federation/X509CertChainFlags 0x10000000 CERT_CHAIN_REVOCATION_CHECK_END_CERT 0x20000000 CERT_CHAIN_REVOCATION_CHECK_CHAIN 0x40000000 CERT_CHAIN_REVOCATION_CHECK_CHAIN_EXCLUDE_ROOT 0x80000000 CERT_CHAIN_REVOCATION_CHECK_CACHE_ONLY Si définie sur 0, cette valeur désactive la vérification CRL La liste complète des valeurs prises en charge est documentée par dwFlags de CertGetCertificateChain : https://msdn.microsoft.com/library/windows/desktop/aa376078(v=vs.85).aspx |
|CrlDisablePeriod|TimeSpan, la valeur par défaut est Common::TimeSpan::FromMinutes(15)|Dynamique|Spécifiez la durée en secondes. Durée de la désactivation de la vérification de la liste CRL pour un certificat donné après une erreur en mode hors connexion ; l’erreur en mode hors connexion de la liste CRL peut être ignorée. |
|CrlOfflineHealthReportTtl|TimeSpan, la valeur par défaut est Common::TimeSpan::FromMinutes(1440)|Dynamique|Spécifiez la durée en secondes. |
|DisableFirewallRuleForDomainProfile| Valeur booléenne, valeur par défaut : TRUE |statique| Indique si la règle de pare-feu ne doit pas être activée pour le profil de domaine |
|DisableFirewallRuleForPrivateProfile| Valeur booléenne, valeur par défaut : TRUE |statique| Indique si la règle de pare-feu ne doit pas être activée pour le profil privé | 
|DisableFirewallRuleForPublicProfile| Valeur booléenne, valeur par défaut : TRUE | statique|Indique si la règle de pare-feu ne doit pas être activée pour le profil public |
| EnforceLinuxMinTlsVersion | valeur booléenne, valeur par défaut : FALSE | statique | Si la valeur est true, seule la version 1.2 et les versions ultérieures de TLS sont prises en charge.  Si la valeur est false, les versions antérieures de TLS sont prises en charge. S’applique à Linux uniquement. |
| EnforcePrevalidationOnSecurityChanges | valeur booléenne, valeur par défaut : FALSE| Dynamique | Indicateur contrôlant le comportement de mise à niveau du cluster lors de la détection de modification des paramètres de sécurité. S’il est défini sur « true », la mise à niveau du cluster tente de s’assurer qu’au moins l’un des certificats répondant à l’une des règles de présentation peut transmettre une règle de validation correspondante. La validation préalable est exécutée avant que les nouveaux paramètres ne soient appliqués à un nœud, mais elle n’est exécutée que sur le nœud qui héberge le réplica principal du service Cluster Manager au démarrage de la mise à niveau. La valeur par défaut est actuellement « false » ; à partir de la version 7.1, le paramètre sera défini sur « true » pour les nouveaux clusters Azure Service Fabric.|
|FabricHostSpn| Chaîne (valeur par défaut : "") |statique| Nom de principal du service de FabricHost ; lorsque Fabric s’exécute en tant qu’utilisateur de domaine unique (gMSA/compte d’utilisateur de domaine) et que FabricHost s’exécute sous le compte d’ordinateur. Il s’agit du SPN de l’écouteur IPC pour FabricHost ; qui par défaut doit être vide car FabricHost s’exécute sous le compte d’ordinateur |
|IgnoreCrlOfflineError|valeur booléenne, valeur par défaut : FALSE|Dynamique|Indique s’il faut ou non ignorer l’erreur en mode hors connexion de la liste CRL lorsque le côté serveur vérifie des certificats de client entrants |
|IgnoreSvrCrlOfflineError|Valeur booléenne, valeur par défaut : TRUE|Dynamique|Indique s’il faut ou non ignorer l’erreur en mode hors connexion de la liste CRL lorsque le côté client vérifie des certificats clients entrants ; valeur par défaut : true. Les attaques avec des certificats serveurs révoqués exigent de compromettre le DNS ; plus difficile qu’avec des certificats clients révoqués. |
|ServerAuthCredentialType|Chaîne (valeur par défaut : "None")|statique|Indique le type d’informations d’identification de sécurité à utiliser pour sécuriser la communication entre FabricClient et le cluster. Les valeurs valides sont "None/X509/Windows" |
|ServerCertThumbprints|Chaîne (valeur par défaut : "")|Dynamique|Empreintes de certificats de serveur utilisés par le cluster pour communiquer avec les clients ; les clients l’utilisent pour authentifier le cluster. Il s’agit d’une liste de noms séparés par des virgules. |
|SettingsX509StoreName| Chaîne (valeur par défaut : "MY")| Dynamique|Magasin de certificats X509 utilisé par l’infrastructure pour protéger la configuration |
|UseClusterCertForIpcServerTlsSecurity|valeur booléenne, valeur par défaut : FALSE|statique|Indique s’il faut utiliser le certificat de cluster pour sécuriser l’unité de transport TLS du serveur IPC |
|X509Folder|chaîne, valeur par défaut : /var/lib/waagent|statique|Dossier dans lequel se trouvent les certificats X509 et les clés privées |
|TLS1_2_CipherList| string| statique|Si la valeur est une chaîne non vide : remplace la liste de chiffrement prise en charge par TLS 1.2 ou une version antérieure. Consultez la documentation « openssl-ciphers » pour récupérer la liste de chiffrement prise en charge et le format de la liste. Exemple de liste de chiffrement forte pour TLS 1.2 : « ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES-128-GCM-SHA256:ECDHE-ECDSA-AES256-CBC-SHA384:ECDHE-ECDSA-AES128-CBC-SHA256:ECDHE-RSA-AES256-CBC-SHA384:ECDHE-RSA-AES128-CBC-SHA256 » ; s’applique à Linux uniquement. |

## <a name="securityadminclientx509names"></a>Security/AdminClientX509Names

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|X509NameMap, valeur par défaut : None|Dynamique|Il s’agit d’une liste de paires « Nom » et « Valeur ». Chaque « Nom » a pour objet un nom commun ou un nom DNS de certificats X509 autorisés pour les opérations du client d’administration. Pour un « Nom » donné, une « Valeur » est une liste séparée par des virgules d’empreintes de certificat pour l’épinglage d’émetteur. Si cette chaîne est non vide, l’émetteur direct de certificats clients d’administration doit être dans la liste. |

## <a name="securityclientaccess"></a>Security/ClientAccess

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|ActivateNode |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour activer un nœud. |
|CancelTestCommand |Chaîne (valeur par défault : "Admin") |Dynamique| Annule une commande de test si elle est en cours. |
|CodePackageControl |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour redémarrer des packages de code. |
|CreateApplication |Chaîne (valeur par défault : "Admin") | Dynamique|Configuration de la sécurité pour la création d’applications. |
|CreateComposeDeployment|Chaîne (valeur par défault : "Admin")| Dynamique|Crée un déploiement compose décrit par les fichiers compose |
|CreateGatewayResource|Chaîne (valeur par défault : "Admin")| Dynamique|Créer une ressource de passerelle |
|CreateName |Chaîne (valeur par défault : "Admin") |Dynamique|Configuration de la sécurité pour la création d’URI d’attribution de noms. |
|CreateNetwork|Chaîne (valeur par défault : "Admin") |Dynamique|Crée un réseau de conteneurs |
|CreateService |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour la création de services. |
|CreateServiceFromTemplate |Chaîne (valeur par défault : "Admin") |Dynamique|Configuration de la sécurité pour la création de services à partir d’un modèle. |
|CreateVolume|Chaîne (valeur par défault : "Admin")|Dynamique|Crée un volume. |
|DeactivateNode |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour désactiver un nœud. |
|DeactivateNodesBatch |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour désactiver plusieurs nœuds. |
|DELETE |Chaîne (valeur par défault : "Admin") |Dynamique| Configurations de la sécurité pour l’opération de suppression de clients du magasin d’images (interne). |
|DeleteApplication |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour la suppression d’applications. |
|DeleteComposeDeployment|Chaîne (valeur par défault : "Admin")| Dynamique|Supprime le déploiement compose |
|DeleteGatewayResource|Chaîne (valeur par défault : "Admin")| Dynamique|Supprime une ressource de passerelle |
|DeleteName |Chaîne (valeur par défault : "Admin") |Dynamique|Configuration de la sécurité pour la suppression d’URI d’attribution de noms. |
|DeleteNetwork|Chaîne (valeur par défault : "Admin") |Dynamique|Supprime un réseau de conteneurs |
|DeleteService |Chaîne (valeur par défault : "Admin") |Dynamique|Configuration de la sécurité pour la suppression de services. |
|DeleteVolume|Chaîne (valeur par défault : "Admin")|Dynamique|Supprime un volume.| 
|EnumerateProperties |Chaîne, la valeur par défaut est « Admin\|\|User » | Dynamique|Configuration de la sécurité pour l’énumération de propriétés d’attribution de noms. |
|EnumerateSubnames |Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour l’énumération d’URI d’attribution de noms. |
|FileContent |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour le transfert de fichiers clients du magasin d’images (externe vers le cluster). |
|FileDownload |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour le lancement du téléchargement de fichiers clients du magasin d’images (externe vers le cluster). |
|FinishInfrastructureTask |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour terminer des tâches d’infrastructure. |
|GetChaosReport | Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Récupère l’état de Chaos pendant une période donnée. |
|GetClusterConfiguration | Chaîne, la valeur par défaut est « Admin\|\|User » | Dynamique|Déclenche GetClusterConfiguration sur une partition. |
|GetClusterConfigurationUpgradeStatus | Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Déclenche GetClusterConfigurationUpgradeStatus sur une partition. |
|GetFabricUpgradeStatus |Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour interroger l’état de mise à niveau du cluster. |
|GetFolderSize |Chaîne (valeur par défault : "Admin") |Dynamique|Configuration de la sécurité pour la taille du dossier d’obtention de FileStoreService |
|GetNodeDeactivationStatus |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour vérifier l’état de désactivation. |
|GetNodeTransitionProgress | Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour obtenir l’état d’avancement d’une commande de transition de nœud. |
|GetPartitionDataLossProgress | Chaîne, la valeur par défaut est « Admin\|\|User » | Dynamique|Récupère l’état d’avancement de l’invocation d’un appel d’API de perte de données. |
|GetPartitionQuorumLossProgress | Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Récupère l’état d’avancement de l’invocation d’un appel d’API de perte de quorum. |
|GetPartitionRestartProgress | Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Récupère l’état d’avancement de l’invocation d’un appel d’API de redémarrage de partition. |
|GetSecrets|Chaîne (valeur par défault : "Admin")|Dynamique|Obtient les valeurs du secret |
|GetServiceDescription |Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour la lecture des descriptions de service et les notifications de sondage de longue durée. |
|GetStagingLocation |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour récupérer l’emplacement intermédiaire du client du magasin d’images. |
|GetStoreLocation |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour récupérer l’emplacement du client du magasin d’images. |
|GetUpgradeOrchestrationServiceState|Chaîne (valeur par défault : "Admin")| Dynamique|Déclenche GetUpgradeOrchestrationServiceState sur une partition |
|GetUpgradesPendingApproval |Chaîne (valeur par défault : "Admin") |Dynamique| Déclenche GetUpgradesPendingApproval sur une partition. |
|GetUpgradeStatus |Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour interroger l’état de mise à niveau de l’application. |
|InternalList |Chaîne (valeur par défault : "Admin") | Dynamique|Configuration de la sécurité pour l’opération de liste de fichiers clients deumagasin d’images (interne). |
|InvokeContainerApi|chaîne, valeur par : « Admin »|Dynamique|Invoke container API |
|InvokeInfrastructureCommand |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour les commandes de gestion des tâches d’infrastructure. |
|InvokeInfrastructureQuery |Chaîne, la valeur par défaut est « Admin\|\|User » | Dynamique|Configuration de la sécurité pour interroger les tâches d’infrastructure. |
|List |Chaîne, la valeur par défaut est « Admin\|\|User » | Dynamique|Configuration de la sécurité pour l’opération de liste de fichiers clients du magasin d’images. |
|MoveNextFabricUpgradeDomain |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour reprendre des mises à niveau de cluster avec un domaine de mise à niveau explicite. |
|MoveNextUpgradeDomain |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour reprendre des mises à niveau d’application avec un domaine de mise à niveau explicite. |
|MoveReplicaControl |Chaîne (valeur par défault : "Admin") | Dynamique|Déplacez un réplica. |
|NameExists |Chaîne, la valeur par défaut est « Admin\|\|User » | Dynamique|Configuration de la sécurité pour la vérification de l’existence des URI d’attribution de noms. |
|NodeControl |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour démarrer, arrêter et redémarrer des nœuds. |
|NodeStateRemoved |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour signaler l’état de nœud supprimé. |
|Ping |Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour les tests ping de clients |
|PredeployPackageToNode |Chaîne (valeur par défault : "Admin") |Dynamique| API de pré-déploiement. |
|PrefixResolveService |Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour la résolution de préfixe de service en fonction de la réclamation. |
|PropertyReadBatch |Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour les opérations de lecture de propriétés d’attribution de noms. |
|PropertyWriteBatch |Chaîne (valeur par défault : "Admin") |Dynamique|Configurations de la sécurité pour les opérations d’écriture de la propriété d’attribution de noms. |
|ProvisionApplicationType |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour l’approvisionnement du type d’application. |
|ProvisionFabric |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour l’approvisionnement de MSI et/ou du manifeste de cluster. |
|Requête |Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour les requêtes. |
|RecoverPartition |Chaîne (valeur par défault : "Admin") | Dynamique|Configuration de la sécurité pour restaurer une partition. |
|RecoverPartitions |Chaîne (valeur par défault : "Admin") | Dynamique|Configuration de la sécurité pour restaurer des partitions. |
|RecoverServicePartitions |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour restaurer des partitions de service. |
|RecoverSystemPartitions |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour restaurer des partitions du service système. |
|RemoveNodeDeactivations |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour annuler la désactivation sur plusieurs nœuds. |
|ReportFabricUpgradeHealth |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour reprendre des mises à niveau de cluster à leur état d’avancement actuel. |
|ReportFault |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour signaler une défaillance. |
|ReportHealth |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour créer un rapport d’intégrité. |
|ReportUpgradeHealth |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour reprendre des mises à niveau d’application à leur état d’avancement actuel. |
|ResetPartitionLoad |Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour la charge de réinitialisation d’une unité failoverUnit. |
|ResolveNameOwner |Chaîne, la valeur par défaut est « Admin\|\|User » | Dynamique|Configuration de la sécurité pour résoudre le propriétaire de l’URI d’attribution de noms. |
|ResolvePartition |Chaîne, la valeur par défaut est « Admin\|\|User » | Dynamique|Configuration de la sécurité pour résoudre les services du système. |
|ResolveService |Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour la résolution de services en fonction de la réclamation. |
|ResolveSystemService|Chaîne, la valeur par défaut est « Admin\|\|User »|Dynamique| Configuration de la sécurité pour résoudre les services du système |
|RollbackApplicationUpgrade |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour annuler des mises à niveau d’application. |
|RollbackFabricUpgrade |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour annuler des mises à niveau de cluster. |
|ServiceNotifications |Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour les notifications de service basées sur l’événement. |
|SetUpgradeOrchestrationServiceState|Chaîne (valeur par défault : "Admin")| Dynamique|Déclenche SetUpgradeOrchestrationServiceState sur une partition |
|StartApprovedUpgrades |Chaîne (valeur par défault : "Admin") |Dynamique| Déclenche StartApprovedUpgrades sur une partition. |
|StartChaos |Chaîne (valeur par défault : "Admin") |Dynamique| Démarre Chaos si ce n’est déjà fait. |
|StartClusterConfigurationUpgrade |Chaîne (valeur par défault : "Admin") |Dynamique| Déclenche StartClusterConfigurationUpgrade sur une partition. |
|StartInfrastructureTask |Chaîne (valeur par défault : "Admin") | Dynamique|Configuration de la sécurité pour démarrer des tâches d’infrastructure. |
|StartNodeTransition |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour démarrer une transition de nœud. |
|StartPartitionDataLoss |Chaîne (valeur par défault : "Admin") |Dynamique| Provoque la perte de données sur une partition. |
|StartPartitionQuorumLoss |Chaîne (valeur par défault : "Admin") |Dynamique| Provoque la perte de quorum sur une partition. |
|StartPartitionRestart |Chaîne (valeur par défault : "Admin") |Dynamique| Redémarre simultanément certains ou l’ensemble des réplicas d’une partition. |
|StopChaos |Chaîne (valeur par défault : "Admin") |Dynamique| Arrête Chaos s’il a été démarré. |
|ToggleVerboseServicePlacementHealthReporting | Chaîne, la valeur par défaut est « Admin\|\|User » |Dynamique| Configuration de la sécurité pour l’activation ou la désactivation du rapport d’intégrité détaillé sur le placement du service. |
|UnprovisionApplicationType |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour annuler l’approvisionnement du type d’application. |
|UnprovisionFabric |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour annuler l’approvisionnement de MSI et/ou du manifeste de cluster. |
|UnreliableTransportControl |Chaîne (valeur par défault : "Admin") |Dynamique| Transport non fiable pour ajouter et supprimer des comportements. |
|UpdateService |Chaîne (valeur par défault : "Admin") |Dynamique|Configuration de la sécurité pour les mises à niveau de services. |
|UpgradeApplication |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour démarrer ou arrêter des mises à niveau d’application. |
|UpgradeComposeDeployment|Chaîne (valeur par défault : "Admin")| Dynamique|Met à niveau le déploiement compose |
|UpgradeFabric |Chaîne (valeur par défault : "Admin") |Dynamique| Configuration de la sécurité pour démarrer des mises à niveau de cluster. |
|Télécharger |Chaîne (valeur par défault : "Admin") | Dynamique|Configuration de la sécurité pour l’opération de chargement de clients du magasin d’images. |

## <a name="securityclientcertificateissuerstores"></a>Security/ClientCertificateIssuerStores

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|IssuerStoreKeyValueMap, la valeur par défaut est None |Dynamique|Magasins de certificats de l’émetteur X509 pour les certificats de clients ; Nom = clientIssuerCN ; Valeur = liste des magasins séparée par des virgules |

## <a name="securityclientx509names"></a>Security/ClientX509Names

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|X509NameMap, valeur par défaut : None|Dynamique|Il s’agit d’une liste de paires « Nom » et « Valeur ». Chaque « Nom » a pour objet un nom commun ou un nom DNS de certificats X509 autorisés pour les opérations de client. Pour un « Nom » donné, une « Valeur » est une liste séparée par des virgules d’empreintes de certificat pour l’épinglage d’émetteur. Si cette chaîne est non vide, l’émetteur direct de certificats clients doit être dans la liste.|

## <a name="securityclustercertificateissuerstores"></a>Security/ClusterCertificateIssuerStores

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|IssuerStoreKeyValueMap, la valeur par défaut est None |Dynamique|Magasins de certificats de l’émetteur X509 pour les certificats de cluster ; Nom = clusterIssuerCN ; Valeur = liste des magasins séparée par des virgules |

## <a name="securityclusterx509names"></a>Security/ClusterX509Names

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|X509NameMap, valeur par défaut : None|Dynamique|Il s’agit d’une liste de paires « Nom » et « Valeur ». Chaque « Nom » a pour objet un nom commun ou un nom DNS de certificats X509 autorisés pour les opérations de cluster. Pour un « Nom » donné, une « Valeur » est une liste séparée par des virgules d’empreintes de certificat pour l’épinglage d’émetteur. Si cette chaîne est non vide, l’émetteur direct de certificats de cluster doit être dans la liste.|

## <a name="securityservercertificateissuerstores"></a>Security/ServerCertificateIssuerStores

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|IssuerStoreKeyValueMap, la valeur par défaut est None |Dynamique|Magasins de certificats de l’émetteur X509 pour les certificats de serveurs ; Nom = serverIssuerCN ; Valeur = liste des magasins séparée par des virgules |

## <a name="securityserverx509names"></a>Security/ServerX509Names

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup|X509NameMap, valeur par défaut : None|Dynamique|Il s’agit d’une liste de paires « Nom » et « Valeur ». Chaque « Nom » a pour objet un nom commun ou un nom DNS de certificats X509 autorisés pour les opérations de serveur. Pour un « Nom » donné, une « Valeur » est une liste séparée par des virgules d’empreintes de certificat pour l’épinglage d’émetteur. Si cette chaîne est non vide, l’émetteur direct de certificats de serveur doit être dans la liste.|

## <a name="setup"></a>Programme d’installation

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|ContainerNetworkName|Chaîne (valeur par défaut : "")| statique |Le nom du réseau à utiliser lors de la configuration d’un réseau de conteneur.|
|ContainerNetworkSetup|bool, la valeur par défaut est FALSE pour Linux et TRUE pour Windows| statique |Spécifie si un réseau de conteneur doit être configuré.|
|FabricDataRoot |String | Non autorisée |Répertoire racine des données Service Fabric. La valeur par défaut pour Azure est d:\svcfab |
|FabricDataRoot |String | Non autorisée |Répertoire racine du journal Service Fabric. Il s’agit de l’emplacement des journaux d’activité et des traces de SF. |
|NodesToBeRemoved|Chaîne (valeur par défaut : "")| Dynamique |Les nœuds qui doivent être supprimés dans le cadre de la mise à niveau de la configuration. (Uniquement pour les déploiements autonomes)|
|ServiceRunAsAccountName |String | Non autorisée |Nom du compte sous lequel exécuter le service hôte Fabric. |
|SkipContainerNetworkResetOnReboot|valeur booléenne, valeur par défaut : FALSE|Non autorisé|Indique s’il faut ou non ignorer la réinitialisation du réseau de conteneur au redémarrage.|
|SkipFirewallConfiguration |Valeur booléenne (valeur par défaut : false) | Non autorisée |Indique si les paramètres de pare-feu doivent être définis par le système ou non. Cela s’applique uniquement si vous utilisez le pare-feu Windows. Si vous utilisez des pare-feu tiers, vous devez alors ouvrir les ports que le système et les applications doivent utiliser |

## <a name="tokenvalidationservice"></a>TokenValidationService

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|Fournisseurs |Chaîne (valeur par défault : "DSTS") |statique|Liste séparée par des virgules des fournisseurs de validation de jeton à activer (fournisseurs valides : DSTS ; AAD). Pour l’instant, un seul fournisseur peut être activé à la fois. |

## <a name="traceetw"></a>Trace/Etw

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|Level |Entier (valeur par défaut : 4) | Dynamique |Le niveau etw de la trace accepte les valeurs 1, 2, 3, 4. Pour assurer la prise en charge, vous devez conserver le niveau de trace sur 4 |

## <a name="transactionalreplicator"></a>TransactionalReplicator

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|BatchAcknowledgementInterval | Durée en secondes (valeur par défaut : 0.015) | statique | Spécifiez la durée en secondes. Détermine le délai respecté par le réplicateur après la réception d’une opération et avant de renvoyer un accusé de réception. Les autres opérations reçues pendant cette période verront leur accusé de réception renvoyé dans un seul message, ce qui diminue le trafic réseau mais peut potentiellement réduire le débit du réplicateur. |
|MaxCopyQueueSize |Valeur Uint (valeur par défaut : 16384) | statique |Valeur maximale définissant la taille initiale de la file d’attente qui gère les opérations de réplication. Ce nombre doit être une puissance de 2. Si, pendant l’exécution, la file d’attente atteint cette taille, les opérations sont limitées entre les réplicateurs principal et secondaire. |
|MaxPrimaryReplicationQueueMemorySize |Valeur Uint (valeur par défaut : 0) | statique |Valeur maximale de la file d’attente de réplication principale en octets. |
|MaxPrimaryReplicationQueueSize |Valeur Uint (valeur par défaut : 8192) | statique |Nombre maximum d’opérations pouvant exister dans la file d’attente de réplication principale. Ce nombre doit être une puissance de 2. |
|MaxReplicationMessageSize |Valeur Uint (valeur par défaut : 52428800) | statique | Taille maximale des messages des opérations de réplication. Valeur par défaut : 50 Mo. |
|MaxSecondaryReplicationQueueMemorySize |Valeur Uint (valeur par défaut : 0) | statique |Valeur maximale de la file d’attente de réplication secondaire en octets. |
|MaxSecondaryReplicationQueueSize |Valeur Uint (valeur par défaut : 16384) | statique |Nombre maximum d’opérations pouvant exister dans la file d’attente de réplication secondaire. Ce nombre doit être une puissance de 2. |
|ReplicatorAddress |Chaîne (valeur par défaut : "localhost:0") | statique | Point de terminaison sous la forme d’une chaîne « IP:port» utilisée par le réplicateur Windows Fabric pour se connecter à d’autres réplicas afin d’envoyer/recevoir des opérations. |

## <a name="transport"></a>Transport
| **Paramètre** | **Valeurs autorisées** |**Stratégie de mise à niveau** |**Conseils ou brève description** |
| --- | --- | --- | --- |
|ConnectionOpenTimeout|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(60)|statique|Spécifiez la durée en secondes. Délai d’expiration pour la configuration de la connexion côté entrant et acceptant (y compris la négociation de sécurité en mode sécurisé) |
|FrameHeaderErrorCheckingEnabled|Valeur booléenne, valeur par défaut : TRUE|statique|Paramètre par défaut pour la vérification d’erreurs dans l’en-tête d’image en mode non sécurisé ; remplacé par le paramètre du composant. |
|MessageErrorCheckingEnabled|Valeur booléenne, valeur par défaut : TRUE|statique|Paramètre par défaut pour la vérification d’erreurs dans l’en-tête et le corps de message en mode non sécurisé ; remplacé par le paramètre du composant. |
|ResolveOption|chaîne, valeur par défaut : « unspecified »|statique|Détermine la façon dont le nom de domaine complet est résolu.  Les valeurs valides sont "unspecified/ipv4/ipv6". |
|SendTimeout|TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(300)|Dynamique|Spécifiez la durée en secondes. Envoyez un délai d'expiration pour détecter une connexion bloquée. Les rapports d’échec TCP ne sont pas fiables dans certains environnements. Cela peut être ajusté en fonction de la bande passante réseau disponible et de la taille des données sortantes (\*MaxMessageSize\/\*SendQueueSizeLimit). |

## <a name="upgradeorchestrationservice"></a>UpgradeOrchestrationService

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|AutoupgradeEnabled | Valeur booléenne (valeur par défaut : true) |statique| Interrogation automatique et action de mise à niveau basée sur un fichier d’état d’objectif. |
|AutoupgradeInstallEnabled|Valeur booléenne, valeur par défaut : FALSE|statique|Interrogation automatique, provisionnement et installation de l’action de mise à niveau du code basée sur un fichier d’état d’objectif.|
|GoalStateExpirationReminderInDays|valeur int, valeur par défaut : 30|statique|Définit le nombre de jours restants après lequel le rappel d’état d’objectif doit être montré.|
|MinReplicaSetSize |Entier (valeur par défaut : 0) |statique |Paramètre MinReplicaSetSize pour UpgradeOrchestrationService.|
|PlacementConstraints | Chaîne (valeur par défaut : "") |statique| Paramètre PlacementConstraints pour UpgradeOrchestrationService. |
|QuorumLossWaitDuration | Durée en secondes (valeur par défaut : MaxValue) |statique| Spécifiez la durée en secondes. Paramètre QuorumLossWaitDuration pour UpgradeOrchestrationService. |
|ReplicaRestartWaitDuration | Durée en secondes (valeur par défaut : 60 minutes)|statique| Spécifiez la durée en secondes. Paramètre ReplicaRestartWaitDuration pour UpgradeOrchestrationService. |
|StandByReplicaKeepDuration | Durée en secondes (valeur par défaut : 60 *24* 7 minutes) |statique| Spécifiez la durée en secondes. Paramètre StandByReplicaKeepDuration pour UpgradeOrchestrationService. |
|TargetReplicaSetSize |Entier (valeur par défaut : 0) |statique |Paramètre TargetReplicaSetSize pour UpgradeOrchestrationService. |
|UpgradeApprovalRequired | Valeur booléenne (valeur par défaut : false) | statique|La configuration de la mise à niveau du code requiert une autorisation de l’administrateur. |

## <a name="upgradeservice"></a>UpgradeService

| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|BaseUrl | Chaîne (valeur par défaut : "") |statique|Paramètre BaseUrl pour UpgradeService. |
|ClusterId | Chaîne (valeur par défaut : "") |statique|Paramètre ClusterId pour UpgradeService. |
|CoordinatorType | Chaîne (valeur par défault : "WUTest")|Non autorisée|Paramètre CoordinatorType pour UpgradeService. |
|MinReplicaSetSize | Entier (valeur par défaut : 2) |Non autorisée| Paramètre MinReplicaSetSize pour UpgradeService. |
|OnlyBaseUpgrade | Valeur booléenne (valeur par défaut : false) |Dynamique|Paramètre OnlyBaseUpgrade pour UpgradeService. |
|PlacementConstraints |Chaîne (valeur par défaut : "") |Non autorisée|Paramètre PlacementConstraints pour le service de mise à niveau. |
|PollIntervalInSeconds|Valeur de type TimeSpan, la valeur par défaut est Common::TimeSpan::FromSeconds(60) |Dynamique|Spécifiez la durée en secondes. Intervalle entre les interrogations d’UpgradeService pour les opérations de gestion ARM. |
|TargetReplicaSetSize | Entier (valeur par défaut : 3) |Non autorisée| Paramètre TargetReplicaSetSize pour UpgradeService. |
|TestCabFolder | Chaîne (valeur par défaut : "") |statique| Paramètre TestCabFolder pour UpgradeService. |
|X509FindType | Chaîne (valeur par défaut : "")|Dynamique| Paramètre X509FindType pour UpgradeService. |
|X509FindValue | Chaîne (valeur par défaut : "") |Dynamique| Paramètre X509FindValue pour UpgradeService. |
|X509SecondaryFindValue | Chaîne (valeur par défaut : "") |Dynamique| Paramètre X509SecondaryFindValue pour UpgradeService. |
|X509StoreLocation | Chaîne (valeur par défaut : "") |Dynamique| Paramètre X509StoreLocation pour UpgradeService. |
|X509StoreName | Chaîne (valeur par défaut : "My")|Dynamique|Paramètre X509StoreName pour UpgradeService. |

## <a name="userservicemetriccapacities"></a>UserServiceMetricCapacities
| **Paramètre** | **Valeurs autorisées** | **Stratégie de mise à niveau** | **Conseils ou brève description** |
| --- | --- | --- | --- |
|PropertyGroup| UserServiceMetricCapacitiesMap, la valeur par défaut est None | statique | Collection de limites de gouvernance des ressources de services utilisateur. Doit être statique car elle affecte la logique de détection automatique |

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations, voir [Mettre à niveau la configuration d’un cluster Azure](service-fabric-cluster-config-upgrade-azure.md) et [Mettre à niveau la configuration d’un cluster autonome](service-fabric-cluster-config-upgrade-windows-server.md).
