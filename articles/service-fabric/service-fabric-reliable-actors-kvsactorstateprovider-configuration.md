---
title: Modifier les paramètres KVSActorStateProvider
description: Découvrez comment configurer les acteurs avec état Azure Service Fabric de type KVSActorStateProvider.
author: sumukhs
ms.topic: conceptual
ms.date: 10/2/2017
ms.author: sumukhs
ms.openlocfilehash: cdb115bd57cf3d5af4388f4efa03c2522feef9ca
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75609772"
---
# <a name="configuring-reliable-actors--kvsactorstateprovider"></a>Configuration de Reliable Actors - KVSActorStateProvider
Vous pouvez modifier la configuration par défaut de KVSActorStateProvider en modifiant le fichier settings.xml généré dans la racine du package Microsoft Visual Studio sous le dossier Config de l’acteur spécifié.

Le runtime Azure Service Fabric recherche des noms de sections prédéfinis dans le fichier settings.xml et utilise les valeurs de configuration pendant la création des composants runtime sous-jacents.

> [!NOTE]
> Veillez à **ne pas** supprimer ou modifier les noms de sections des configurations suivantes dans le fichier settings.xml généré dans la solution Visual Studio.
> 
> 

## <a name="replicator-security-configuration"></a>Configuration de la sécurité du réplicateur
Les configurations de sécurité du réplicateur sont utilisées pour sécuriser le canal de communication utilisé lors de la réplication. Un service ne peut donc pas afficher le trafic de réplication d’un autre service, ce qui garantit la sécurité des données rendues hautement disponibles.
Par défaut, une section de configuration de sécurité vide empêche de sécuriser la réplication.

> [!IMPORTANT]
> Sur les nœuds Linux, les certificats doivent être au format PEM. Pour en savoir plus sur la localisation et la configuration des certificats pour Linux, consultez [Configurer des certificats sur Linux](./service-fabric-configure-certificates-linux.md). 
> 

### <a name="section-name"></a>Nom de la section
&lt;ActorName&gt;ServiceReplicatorSecurityConfig

## <a name="replicator-configuration"></a>Configuration du réplicateur
Les configurations de réplicateur servent à configurer le réplicateur responsable de la haute fiabilité de l’état du fournisseur d’état d’acteur.
La configuration par défaut est générée par le modèle Visual Studio et devrait suffire. Cette section décrit les configurations supplémentaires disponibles pour paramétrer le réplicateur.

### <a name="section-name"></a>Nom de la section
&lt;ActorName&gt;ServiceReplicatorConfig

### <a name="configuration-names"></a>Noms de configuration
| Nom | Unité | Valeur par défaut | Notes |
| --- | --- | --- | --- |
| BatchAcknowledgementInterval |Secondes |0.015 |Durée d'attente du réplicateur secondaire après la réception d'une opération et avant de renvoyer un accusé de réception au réplicateur principal. Tous les autres accusés de réception à envoyer pour les opérations traitées durant cet intervalle sont envoyés sous la forme d'une réponse. |
| ReplicatorEndpoint |N/A |Aucune valeur par défaut (paramètre obligatoire) |Adresse IP et port que le réplicateur principal/secondaire utilise pour communiquer avec d'autres réplicateurs dans le jeu de réplicas. Doit faire référence à un point de terminaison de ressource TCP dans le manifeste de service. Pour en savoir plus sur la définition de ressources de point de terminaison dans le manifeste de service, consultez [Ressources du manifeste de service](service-fabric-service-manifest-resources.md) . |
| RetryInterval |Secondes |5 |Période après laquelle le réplicateur retransmet un message s'il ne reçoit aucun accusé de réception pour une opération. |
| MaxReplicationMessageSize |Octets |50 Mo |Taille maximale des données de réplication pouvant être transmises dans un même message. |
| MaxPrimaryReplicationQueueSize |Nombre d'opérations |1 024 |Nombre maximal d'opérations dans la file d'attente principale. Une opération est libérée quand le réplicateur principal reçoit un accusé de réception de tous les réplicateurs secondaires. Cette valeur doit être supérieure à 64 et être une puissance de 2. |
| MaxSecondaryReplicationQueueSize |Nombre d'opérations |2 048 |Nombre maximal d'opérations dans la file d'attente secondaire. Une opération est libérée une fois son état devenu hautement disponible grâce à la persistance. Cette valeur doit être supérieure à 64 et être une puissance de 2. |

## <a name="store-configuration"></a>Configuration du magasin
Les configurations de magasin permettent de configurer le magasin local utilisé pour conserver l’état en cours de réplication.
La configuration par défaut est générée par le modèle Visual Studio et devrait suffire. Cette section décrit les configurations supplémentaires disponibles pour paramétrer le magasin local.

### <a name="section-name"></a>Nom de la section
&lt;ActorName&gt;ServiceLocalStoreConfig

### <a name="configuration-names"></a>Noms de configuration
| Nom | Unité | Valeur par défaut | Notes |
| --- | --- | --- | --- |
| MaxAsyncCommitDelayInMilliseconds |Millisecondes |200 |Définit l'intervalle maximal de traitement par lot pour les validations de magasin local durables. |
| MaxVerPages |Nombre de pages |16384 |Nombre maximal de pages de version dans la base de données du magasin local. Il détermine le nombre maximal de transactions en attente. |

## <a name="sample-configuration-file"></a>Exemple de fichier de configuration
```xml
<?xml version="1.0" encoding="utf-8"?>
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
   <Section Name="MyActorServiceReplicatorConfig">
      <Parameter Name="ReplicatorEndpoint" Value="MyActorServiceReplicatorEndpoint" />
      <Parameter Name="BatchAcknowledgementInterval" Value="0.05"/>
   </Section>
   <Section Name="MyActorServiceLocalStoreConfig">
      <Parameter Name="MaxVerPages" Value="8192" />
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

