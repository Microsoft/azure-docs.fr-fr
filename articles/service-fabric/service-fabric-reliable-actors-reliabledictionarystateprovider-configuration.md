---
title: Modifier les paramètres ReliableDictionaryActorStateProvider
description: Découvrez comment configurer les acteurs avec état Azure Service Fabric de type ReliableDictionaryActorStateProvider.
author: sumukhs
ms.topic: conceptual
ms.date: 10/2/2017
ms.author: sumukhs
ms.openlocfilehash: fbd6f7cd3ade753c659464522408aa715cce48f9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75609738"
---
# <a name="configuring-reliable-actors--reliabledictionaryactorstateprovider"></a>Configuration de Reliable Actors - ReliableDictionaryActorStateProvider
Vous pouvez modifier la configuration par défaut de ReliableDictionaryActorStateProvider en modifiant le fichier settings.xml généré dans la racine du package Visual Studio sous le dossier Config de l’acteur spécifié.

Le runtime Azure Service Fabric recherche des noms de sections prédéfinis dans le fichier settings.xml et utilise les valeurs de configuration pendant la création des composants runtime sous-jacents.

> [!NOTE]
> Veillez à **ne pas** supprimer ou modifier les noms de sections des configurations suivantes dans le fichier settings.xml généré dans la solution Visual Studio.
> 
> 

Il existe également des paramètres globaux qui affectent la configuration de ReliableDictionaryActorStateProvider.

## <a name="global-configuration"></a>Configuration globale
La configuration globale est spécifiée dans le manifeste de cluster sous la section KtlLogger. Elle permet la configuration de l’emplacement et de la taille du journal partagé, ainsi que des limites de mémoire globales utilisées par l’enregistreur d’événements. Notez que les modifications apportées au manifeste de cluster affectent l’ensemble des services qui utilisent ReliableDictionaryActorStateProvider et des services fiables avec état.

Le manifeste de cluster est un fichier XML simple qui contient les paramètres et les configurations qui s’appliquent à l’ensemble des nœuds et des services du cluster. Le fichier a généralement pour nom ClusterManifest.xml. Vous pouvez voir le manifeste de cluster de votre cluster à l’aide de la commande PowerShell Get-ServiceFabricClusterManifest.

### <a name="configuration-names"></a>Noms de configuration
| Nom | Unité | Valeur par défaut | Notes |
| --- | --- | --- | --- |
| WriteBufferMemoryPoolMinimumInKB |Ko |8388608 |Nombre minimal de Ko à allouer en mode noyau pour le pool de mémoire tampon d’écriture de l’enregistreur d’événements. Ce pool de mémoire est utilisé pour la mise en cache des informations d’état avant l’écriture sur le disque. |
| WriteBufferMemoryPoolMaximumInKB |Ko |Aucune limite |Taille maximale que peut atteindre le pool de mémoire tampon d’écriture de l’enregistreur d’événements. |
| SharedLogId |GUID |"" |Spécifie un GUID unique à utiliser pour identifier le fichier journal partagé par défaut utilisé par tous les services fiables sur tous les nœuds du cluster qui ne spécifient pas l’élément SharedLogId dans leur configuration de service spécifique. Si SharedLogId est spécifié, SharedLogPath doit l’être aussi. |
| SharedLogPath |Nom de chemin complet |"" |Spécifie le chemin d’accès complet du fichier journal partagé utilisé par tous les services fiables sur tous les nœuds du cluster qui ne spécifient pas l’élément SharedLogPath dans leur configuration de service spécifique. Toutefois, si SharedLogPath est spécifié, SharedLogId doit l'être aussi. |
| SharedLogSizeInMB |Mo |8 192 |Spécifie le nombre de Mo d’espace disque à allouer de manière statique pour le journal partagé. La valeur doit être supérieure ou égale à 2 048. |

### <a name="sample-cluster-manifest-section"></a>Exemple de section du manifeste de cluster
```xml
   <Section Name="KtlLogger">
     <Parameter Name="WriteBufferMemoryPoolMinimumInKB" Value="8192" />
     <Parameter Name="WriteBufferMemoryPoolMaximumInKB" Value="8192" />
     <Parameter Name="SharedLogId" Value="{7668BB54-FE9C-48ed-81AC-FF89E60ED2EF}"/>
     <Parameter Name="SharedLogPath" Value="f:\SharedLog.Log"/>
     <Parameter Name="SharedLogSizeInMB" Value="16383"/>
   </Section>
```

### <a name="remarks"></a>Notes
L’enregistreur dispose d’un pool global de mémoire allouée provenant de la mémoire du noyau non paginée disponible pour tous les services fiables sur un nœud pour mettre en cache les données d’état avant l’écriture dans le journal dédié associé du réplica de service fiable. La taille du pool est contrôlée par les paramètres WriteBufferMemoryPoolMinimumInKB et WriteBufferMemoryPoolMaximumInKB. WriteBufferMemoryPoolMinimumInKB spécifie la taille initiale de ce pool de mémoire et la taille minimale à laquelle peut se réduire le pool de mémoire. WriteBufferMemoryPoolMaximumInKB est la taille maximale que peut atteindre le pool de mémoire. Chaque réplica de service fiable qui est ouvert peut augmenter la taille du pool de mémoire d’une quantité déterminée par le système jusqu’à WriteBufferMemoryPoolMaximumInKB. Si la demande de mémoire au pool de mémoire dépasse la quantité disponible, les requêtes de mémoire sont retardées jusqu’à ce que la mémoire soit disponible. Par conséquent, si le pool de mémoire tampon d’écriture est trop petit pour une configuration particulière, les performances peuvent être affectées.

Les paramètres SharedLogId et SharedLogPath sont toujours utilisés ensemble pour définir le GUID et l’emplacement du journal partagé par défaut pour tous les nœuds du cluster. Le journal partagé par défaut est utilisé pour tous les services fiables qui ne spécifient pas les paramètres dans le fichier settings.xml pour le service spécifique. Pour des performances optimales, les fichiers journaux partagés doivent être placés sur des disques uniquement utilisés pour le fichier journal partagé afin de réduire la contention.

SharedLogSizeInMB spécifie la quantité d’espace disque à préallouer pour le journal partagé par défaut sur tous les nœuds.  Il n’est pas nécessaire de spécifier SharedLogId et SharedLogPath pour spécifier SharedLogSizeInMB.

## <a name="replicator-security-configuration"></a>Configuration de la sécurité du réplicateur
Les configurations de sécurité du réplicateur sont utilisées pour sécuriser le canal de communication utilisé lors de la réplication. Un service ne peut donc pas afficher le trafic de réplication d’un autre service, ce qui garantit la sécurité des données rendues hautement disponibles.
Par défaut, une section de configuration de sécurité vide empêche de sécuriser la réplication.

> [!IMPORTANT]
> Sur les nœuds Linux, les certificats doivent être au format PEM. Pour en savoir plus sur la localisation et la configuration des certificats pour Linux, consultez [Configurer des certificats sur Linux](./service-fabric-configure-certificates-linux.md). 
> 

### <a name="section-name"></a>Nom de la section
&lt;ActorName&gt;ServiceReplicatorSecurityConfig

## <a name="replicator-configuration"></a>Configuration du réplicateur
Les configurations de réplicateur servent à configurer le réplicateur en charge de la haute fiabilité de l’état du fournisseur d’état d’acteur par la réplication et la persistance de l’état au niveau local.
La configuration par défaut est générée par le modèle Visual Studio et devrait suffire. Cette section décrit les configurations supplémentaires disponibles pour paramétrer le réplicateur.

### <a name="section-name"></a>Nom de la section
&lt;ActorName&gt;ServiceReplicatorConfig

### <a name="configuration-names"></a>Noms de configuration
| Nom | Unité | Valeur par défaut | Notes |
| --- | --- | --- | --- |
| BatchAcknowledgementInterval |Secondes |0.015 |Durée d'attente du réplicateur secondaire après la réception d'une opération et avant de renvoyer un accusé de réception au réplicateur principal. Tous les autres accusés de réception à envoyer pour les opérations traitées durant cet intervalle sont envoyés sous la forme d'une réponse. |
| ReplicatorEndpoint |N/A |Aucune valeur par défaut (paramètre obligatoire) |Adresse IP et port que le réplicateur principal/secondaire utilise pour communiquer avec d'autres réplicateurs dans le jeu de réplicas. Doit faire référence à un point de terminaison de ressource TCP dans le manifeste de service. Pour en savoir plus sur la définition de ressources de point de terminaison dans le manifeste de service, consultez [Ressources du manifeste de service](service-fabric-service-manifest-resources.md) . |
| MaxReplicationMessageSize |Octets |50 Mo |Taille maximale des données de réplication pouvant être transmises dans un même message. |
| MaxPrimaryReplicationQueueSize |Nombre d'opérations |8 192 |Nombre maximal d'opérations dans la file d'attente principale. Une opération est libérée quand le réplicateur principal reçoit un accusé de réception de tous les réplicateurs secondaires. Cette valeur doit être supérieure à 64 et être une puissance de 2. |
| MaxSecondaryReplicationQueueSize |Nombre d'opérations |16384 |Nombre maximal d'opérations dans la file d'attente secondaire. Une opération est libérée une fois son état devenu hautement disponible grâce à la persistance. Cette valeur doit être supérieure à 64 et être une puissance de 2. |
| CheckpointThresholdInMB |Mo |200 |Quantité d'espace du fichier journal après lequel l'état est vérifié. |
| MaxRecordSizeInKB |Ko |1 024 |Taille maximale de l'enregistrement que le réplicateur peut écrire dans le journal. Cette valeur doit être un multiple de 4 et supérieure à 16. |
| OptimizeLogForLowerDiskUsage |Boolean |true |Si la valeur est true, le journal est configuré de sorte que le fichier journal dédié au réplica est créé à l’aide d’un fichier partiellement alloué NTFS. Cela réduit l'utilisation d'espace disque réel du fichier. Si la valeur est false, le fichier est créé avec des allocations fixes qui offrent les meilleures performances en écriture. |
| SharedLogId |guid |"" |Spécifie une valeur guid unique à utiliser pour identifier le fichier journal partagé utilisé avec ce réplica. En règle générale, les services ne doivent pas utiliser ce paramètre. Toutefois, si SharedLogId est spécifié, SharedLogPath doit l'être aussi. |
| SharedLogPath |Nom de chemin complet |"" |Spécifie le chemin d'accès complet où sera créé le fichier journal partagé pour ce réplica. En règle générale, les services ne doivent pas utiliser ce paramètre. Toutefois, si SharedLogPath est spécifié, SharedLogId doit l'être aussi. |

## <a name="sample-configuration-file"></a>Exemple de fichier de configuration
```xml
<?xml version="1.0" encoding="utf-8"?>
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Section Name="MyActorServiceReplicatorConfig">
      <Parameter Name="ReplicatorEndpoint" Value="MyActorServiceReplicatorEndpoint" />
      <Parameter Name="BatchAcknowledgementInterval" Value="0.05"/>
      <Parameter Name="CheckpointThresholdInMB" Value="180" />
   </Section>
   <Section Name="MyActorServiceReplicatorSecurityConfig">
      <Parameter Name="CredentialType" Value="X509" />
      <Parameter Name="FindType" Value="FindByThumbprint" />
      <Parameter Name="FindValue" Value="9d c9 06 b1 69 dc 4f af fd 16 97 ac 78 1e 80 67 90 74 9d 2f" />
      <Parameter Name="StoreLocation" Value="LocalMachine" />
      <Parameter Name="StoreName" Value="My" />
      <Parameter Name="ProtectionLevel" Value="EncryptAndSign" />
      <Parameter Name="AllowedCommonNames" Value="My-Test-SAN1-Alice,My-Test-SAN1-Bob" />
   </Section>
</Settings>
```

## <a name="remarks"></a>Notes
Le paramètre BatchAcknowledgementInterval contrôle la latence de la réplication. La valeur « 0 » entraîne la latence la plus faible possible, au détriment du débit (car davantage de messages d'accusé de réception doivent être envoyés et traités, chacun contenant moins d'accusés de réception).
Plus la valeur de BatchAcknowledgementInterval est élevée, plus le débit de réplication général est élevé, au détriment d'une plus grande latence de l'opération. Cela se traduit directement par une latence dans la validation des transactions.

Le paramètre CheckpointThresholdInMB contrôle la quantité d’espace disque que le réplicateur peut utiliser pour stocker des informations d’état dans le fichier journal dédié au réplica. Son augmentation pour une valeur supérieure à celle par défaut peut entraîner des temps de reconfiguration plus rapides quand un nouveau réplica est ajouté à l'ensemble. Cela est dû au transfert d'état partiel qui intervient suite à la disponibilité d'un historique des opérations plus important dans le journal. Cela peut potentiellement accroître le temps de récupération d’un réplica après un blocage.

Si vous affectez à OptimizeForLowerDiskUsage la valeur true, l’espace du fichier journal est surapprovisionné. Les réplicas actifs peuvent ainsi stocker davantage d’informations d’état dans leurs fichiers journaux, tandis que les réplicas inactifs utilisent moins d’espace disque. Il est donc possible d’héberger davantage de réplicas sur un nœud. Si vous affectez à OptimizeForLowerDiskUsage la valeur false, les informations d’état sont écrites dans les fichiers journaux plus rapidement.

Le paramètre MaxRecordSizeInKB définit la taille maximale d’un enregistrement que le réplicateur peut écrire dans le fichier journal. Dans la plupart des cas, la taille d’enregistrement par défaut de 1 024 Ko est optimale. Toutefois, si le service ajoute des données plus volumineuses aux informations d’état, cette valeur devra éventuellement être augmentée. Il n'est pas recommandé de choisir une valeur MaxRecordSizeInKB inférieure à 1 024 Ko, car des enregistrements plus petits utilisent uniquement l'espace nécessaire à l'enregistrement le plus petit. Cette valeur ne doit être modifiée qu’en de rares occasions.

Les paramètres SharedLogId et SharedLogPath sont toujours utilisés ensemble pour permettre à un service d'utiliser un journal partagé distinct du journal partagé par défaut pour le nœud. Pour plus d'efficacité, vous devriez spécifier autant de services que possible dans le même journal partagé. Les fichiers journaux partagés doivent être placés sur des disques uniquement utilisés pour le fichier journal partagé afin de réduire la contention des mouvements de la tête. Ces valeurs ne doivent être modifiées qu’en de rares occasions.

