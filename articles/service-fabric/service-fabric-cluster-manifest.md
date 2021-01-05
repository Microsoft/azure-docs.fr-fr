---
title: Configurer votre cluster Azure Service Fabric autonome
description: Découvrez comment configurer votre cluster Azure Service Fabric local ou autonome.
ms.topic: conceptual
ms.date: 11/12/2018
ms.openlocfilehash: fd93263b38340ce080cca1aecb98f3a599ff1861
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91843156"
---
# <a name="configuration-settings-for-a-standalone-windows-cluster"></a>Paramètres de configuration pour un cluster Windows autonome
Cet article décrit les paramètres de configuration d’un cluster Azure Service Fabric autonome qui peuvent être définis dans le fichier *ClusterConfig.json*. Utilisez ce fichier pour spécifier les informations sur les nœuds de cluster, les configurations de la sécurité ainsi que la topologie du réseau en termes de domaines d’erreur et de mise à niveau.  Après la modification ou l’ajout de paramètres de configuration, vous pouvez [créer un cluster autonome](service-fabric-cluster-creation-for-windows-server.md) ou [mettre à niveau la configuration d’un cluster autonome](service-fabric-cluster-config-upgrade-windows-server.md).

Lorsque vous [téléchargez le package Service Fabric autonome](service-fabric-cluster-creation-for-windows-server.md#downloadpackage),les exemples ClusterConfig.json sont également inclus. Les exemples comprenant « DevCluster » dans leurs noms créent un cluster avec les trois nœuds sur le même ordinateur, à l’aide de nœuds logiques. Parmi ces nœuds, au moins un doit être marqué comme nœud principal. Ce type de cluster est utile dans des environnements de développement ou de test. Il n’est pas pris en charge comme un cluster de production. Les exemples comprenant « MultiMachine » dans leurs noms vous permettent de créer des clusters de niveau de production, avec chaque nœud sur un ordinateur distinct. Le nombre de nœuds principaux pour ces clusters dépend du [niveau de fiabilité](#reliability) du cluster. Dans la version de l’API 5.7 de mai 2017, nous avons supprimé la propriété du niveau de fiabilité. À la place, notre code calcule le niveau de fiabilité le plus optimisé pour votre cluster. N’essayez pas de définir la valeur de cette propriété à partir des versions 5.7.

* ClusterConfig.Unsecure.DevCluster.json et ClusterConfig.Unsecure.MultiMachine.json montrent comment créer, respectivement, un cluster de test ou de production non sécurisé.

* ClusterConfig.Windows.DevCluster.json et ClusterConfig.Windows.MultiMachine.json montrent comment créer des clusters de test ou de production sécurisés en utilisant la [sécurité Windows](service-fabric-windows-cluster-windows-security.md).

* ClusterConfig.X509.DevCluster.JSON et ClusterConfig.X509.MultiMachine.JSON montrent comment créer des clusters de test ou de production sécurisés à l’aide de la [sécurité basée sur un certificat X509](service-fabric-windows-cluster-x509-security.md).

Nous allons maintenant examiner les différentes sections d’un fichier ClusterConfig.json.

## <a name="general-cluster-configurations"></a>Configurations de cluster générales
Cette section couvre les configurations spécifiques à de larges clusters, comme indiqué dans l’extrait de code JSON suivant :

```json
    "name": "SampleCluster",
    "clusterConfigurationVersion": "1.0.0",
    "apiVersion": "01-2017",
```

Vous pouvez attribuer un nom convivial à votre cluster Service Fabric en lui assignant la variable name. La valeur clusterConfigurationVersion représente le numéro de version de votre cluster. Elle augmente à chaque fois que vous mettez à niveau votre cluster Service Fabric. Conservez la valeur par défaut pour le champ apiVersion.

## <a name="nodes-on-the-cluster"></a>Nœuds sur le cluster
Vous pouvez configurer les nœuds de votre cluster Service Fabric à l’aide de la section nœuds, comme le montre l’extrait de code suivant :
```json
"nodes": [{
    "nodeName": "vm0",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType0",
    "faultDomain": "fd:/dc1/r0",
    "upgradeDomain": "UD0"
}, {
    "nodeName": "vm1",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType1",
    "faultDomain": "fd:/dc1/r1",
    "upgradeDomain": "UD1"
}, {
    "nodeName": "vm2",
    "iPAddress": "localhost",
    "nodeTypeRef": "NodeType2",
    "faultDomain": "fd:/dc1/r2",
    "upgradeDomain": "UD2"
}],
```

Un cluster Service Fabric doit contenir au moins trois nœuds. Vous pouvez ajouter d’autres nœuds à cette section, selon votre configuration. Le tableau suivant décrit les paramètres de configuration de chaque nœud :

| **Configuration de nœuds** | **Description** |
| --- | --- |
| nodeName |Vous pouvez attribuer n’importe quel nom convivial au nœud. |
| iPAddress |Identifiez l’adresse IP de votre nœud en ouvrant une fenêtre de commande puis en tapant `ipconfig`. Notez l’adresse IPV4 et assignez-la à la variable iPAddress. |
| nodeTypeRef |Chaque nœud peut être associé à un type de nœud différent. Les [types de nœuds](#node-types) sont définis dans la section suivante. |
| faultDomain |Les domaines d'erreur permettent aux administrateurs de cluster de définir les nœuds physiques qui sont susceptibles de rencontrer un échec en même temps en raison des dépendances physiques partagées. |
| upgradeDomain |Les domaines de mise à niveau décrivent des ensembles de nœuds qui sont arrêtés pour les mises à niveau Service Fabric, à peu près au même moment. Vous pouvez choisir les nœuds à attribuer aux domaines de mise à niveau car ils ne sont pas limités par des exigences physiques. |

## <a name="cluster-properties"></a>Propriétés du cluster
La section Propriétés du fichier ClusterConfig.json permet de configurer le cluster comme indiqué :

### <a name="reliability"></a>Fiabilité
Le concept de reliabilityLevel définit le nombre de répliques ou instances des services système Service Fabric qui peuvent s’exécuter sur les nœuds principaux du cluster. Il détermine augmente la fiabilité de ces services et, par conséquent, du cluster. La valeur est calculée par le système au moment de la création et de la mise à niveau du cluster.

### <a name="diagnostics"></a>Diagnostics
La section diagnosticsStore vous permet de configurer des paramètres pour activer les diagnostics et corriger les défaillances de nœud ou du cluster, comme illustré dans l’extrait de code suivant : 

```json
"diagnosticsStore": {
    "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
    "dataDeletionAgeInDays": "7",
    "storeType": "FileShare",
    "IsEncrypted": "false",
    "connectionstring": "c:\\ProgramData\\SF\\DiagnosticsStore"
}
```

La section metadata est une description du diagnostic de votre cluster et peut être définie selon votre installation. Ces variables vous aident à collecter les journaux d’activité de suivi ETW les vidages sur incident ainsi que les compteurs de performance. Consultez les sections [Tracelog](/windows-hardware/drivers/devtest/tracelog) (Journal de suivi) et [ETW Tracing](/dotnet/framework/wcf/samples/etw-tracing) pour plus d’informations sur les journaux d’activité de suivi ETW. Tous les journaux d’activité, notamment les [vidages sur incident](https://techcommunity.microsoft.com/t5/ask-the-performance-team/bg-p/AskPerf) et les [compteurs de performance](/windows/win32/perfctrs/performance-counters-portal) peuvent être dirigés vers le dossier connectionString sur votre ordinateur. Vous pouvez également utiliser AzureStorage pour stocker les diagnostics. Reportez-vous à l’extrait de code suivant :

```json
"diagnosticsStore": {
    "metadata":  "Please replace the diagnostics store with an actual file share accessible from all cluster machines.",
    "dataDeletionAgeInDays": "7",
    "storeType": "AzureStorage",
    "IsEncrypted": "false",
    "connectionstring": "xstore:DefaultEndpointsProtocol=https;AccountName=[AzureAccountName];AccountKey=[AzureAccountKey]"
}
```

### <a name="security"></a>Sécurité
La section security est nécessaire pour garantir la sécurité d’un cluster Service Fabric autonome. L’extrait de code suivant montre une partie de cette section :

```json
"security": {
    "metadata": "This cluster is secured using X509 certificates.",
    "ClusterCredentialType": "X509",
    "ServerCredentialType": "X509",
    . . .
}
```

La section metadata est une description de votre cluster sécurisé et peut être définie selon votre installation. Les propriétés ClusterCredentialType et ServerCredentialType déterminent le type de sécurité que le cluster et les nœuds implémenteront. Elles peuvent avoir la valeur *X509* pour une sécurité basée sur un certificat ou *Windows* pour une sécurité basée sur Active Directory. Le reste de la section security varie selon le type de sécurité. Pour plus d’informations sur la façon de remplir le reste de la section security, consultez les rubriques [Sécuriser un cluster autonome sur Windows à l’aide de certificats X.509](service-fabric-windows-cluster-x509-security.md) ou [Sécuriser un cluster autonome sur Windows à l’aide de la sécurité Windows](service-fabric-windows-cluster-windows-security.md).

### <a name="node-types"></a>Types de nœuds
La section nodeTypes décrit le type des nœuds de votre cluster. Au moins un type de nœud doit être spécifié pour un cluster, comme indiqué dans l’extrait de code suivant : 

```json
"nodeTypes": [{
    "name": "NodeType0",
    "clientConnectionEndpointPort": "19000",
    "clusterConnectionEndpointPort": "19001",
    "leaseDriverEndpointPort": "19002"
    "serviceConnectionEndpointPort": "19003",
    "httpGatewayEndpointPort": "19080",
    "reverseProxyEndpointPort": "19081",
    "applicationPorts": {
        "startPort": "20575",
        "endPort": "20605"
    },
    "ephemeralPorts": {
        "startPort": "20606",
        "endPort": "20861"
    },
    "isPrimary": true
}]
```

La valeur name représente le nom convivial de ce type de nœud particulier. Pour créer un nœud de ce type, attribuez son nom convivial à la variable nodeTypeRef pour ce nœud, comme [indiqué précédemment](#nodes-on-the-cluster). Pour chaque type de nœud, définissez les points de terminaison de connexion qui sont utilisés. Vous pouvez choisir n’importe quel numéro de port pour ces points de terminaison de connexion, à condition qu’ils n’entrent pas en conflit avec d’autres points de terminaison de ce cluster. Un cluster à plusieurs nœuds contient un ou plusieurs nœuds principaux (c'est-à-dire que isPrimary a la valeur *true*) en fonction du niveau de fiabilité ([reliabilityLevel](#reliability)). Pour en savoir plus sur les propriétés nodeTypes et reliabilityLevel, et sur les types de nœud principal et non principal, consultez la rubrique [Considérations en matière de planification de la capacité du cluster Service Fabric](service-fabric-cluster-capacity.md). 

#### <a name="endpoints-used-to-configure-the-node-types"></a>Points de terminaison utilisés pour configurer les types de nœuds
* clientConnectionEndpointPort est le port utilisé par le client pour se connecter au cluster lors de l’utilisation d’API clientes. 
* clusterConnectionEndpointPort est le port sur lequel les nœuds communiquent entre eux.
* leaseDriverEndpointPort est le port utilisé par le pilote de lease cluster pour savoir si les nœuds sont toujours actifs. 
* serviceConnectionEndpointPort est le port utilisé par les applications et les services déployés sur un nœud pour communiquer avec le client Service Fabric sur ce nœud spécifique.
* httpGatewayEndpointPort est le port utilisé par le Service Fabric Explorer pour se connecter au cluster.
* Les ports ephemeralPorts remplacent les [ports dynamiques utilisés par le système d’exploitation](https://support.microsoft.com/kb/929851). Service Fabric utilise une partie de ces ports comme ports d’application et le reste est disponible pour le système d’exploitation. Il mappe également cette plage sur la plage existante dans le système d’exploitation. Vous pouvez donc utiliser en toutes circonstances les plages spécifiées dans les exemples de fichiers JSON. Assurez-vous que la différence entre les ports de début et de fin est d’au moins 255. Vous pouvez rencontrer des conflits si cette différence est trop faible, étant donné que cette plage est partagée avec le système d’exploitation. Consultez la plage de ports dynamiques configurée en exécutant `netsh int ipv4 show dynamicport tcp`.
* Les ports applicationPorts sont les ports utilisés par les applications Service Fabric. La plage des ports d’application doit suffire à couvrir les exigences en matière de points de terminaison de vos applications. Cette plage doit être exclusive à partir de la plage de ports dynamiques de la machine, c’est-à-dire la plage ephemeralPorts comme défini dans la configuration. Service Fabric les utilise chaque fois que des nouveaux ports sont nécessaires et prend également en charge l’ouverture du pare-feu pour ces ports. 
* reverseProxyEndpointPort est un point de terminaison proxy inverse facultatif. Pour en savoir plus, consultez [Proxy inverse Service Fabric](service-fabric-reverseproxy.md). 

### <a name="log-settings"></a>Paramètres du journal
La section fabricSettings vous permet de définir les répertoires racine des données et journaux d’activité Service Fabric. Vous pouvez personnaliser ces répertoires uniquement lors de la création initiale du cluster. Reportez-vous à l’extrait de code suivant de cette section :

```json
"fabricSettings": [{
    "name": "Setup",
    "parameters": [{
        "name": "FabricDataRoot",
        "value": "C:\\ProgramData\\SF"
    }, {
        "name": "FabricLogRoot",
        "value": "C:\\ProgramData\\SF\\Log"
}]
```

Nous vous recommandons d’utiliser un lecteur autre que celui du système d’exploitation pour FabricDataRoot et FabricLogRoot pour plus de fiabilité en cas de défaillance du système d’exploitation. Si vous personnalisez uniquement la racine des données, la racine du journal sera placée un niveau en dessous de la racine des données.

### <a name="stateful-reliable-services-settings"></a>Paramètres Services fiables avec état
La section KtlLogger vous permet de définir les paramètres de configuration globaux pour les services fiables (Reliable Services). Pour plus d’informations sur ces paramètres, consultez [Configuration de services fiables (Reliable Services) avec état](service-fabric-reliable-services-configuration.md). L’exemple suivant montre comment modifier le journal des transactions partagé qui est créé afin de sauvegarder toutes les collections fiables pour les services avec état :

```json
"fabricSettings": [{
    "name": "KtlLogger",
    "parameters": [{
        "name": "SharedLogSizeInMB",
        "value": "4096"
    }]
}]
```

### <a name="add-on-features"></a>Fonctionnalités supplémentaires
Pour configurer des fonctionnalités supplémentaires, définissez la valeur apiVersion sur 04-2017 ou une version ultérieure, et configurez la propriété addonFeatures tel que suit :

```json
"apiVersion": "04-2017",
"properties": {
    "addOnFeatures": [
        "DnsService",
        "RepairManager"
    ]
}
```
Toutes les fonctionnalités de module complémentaire disponibles sont décrites dans les [Informations de référence sur l’API REST de Service Fabric](/rest/api/servicefabric/sfrp-model-addonfeatures).

### <a name="container-support"></a>Support pour les conteneurs
Pour activer le support pour les conteneurs Windows Server et Hyper-V pour les clusters autonomes, la fonctionnalité supplémentaire DnsService doit être activée.

## <a name="next-steps"></a>Étapes suivantes
Une fois que vous avez configuré un fichier *ClusterConfig.json* complet, adapté à votre installation de cluster autonome, vous pouvez déployer votre cluster. Suivez les étapes décrites dans [Créer un cluster Service Fabric autonome](service-fabric-cluster-creation-for-windows-server.md). 

Si vous avez déployé un cluster autonome, vous pouvez également [mettre à niveau la configuration d’un cluster autonome](service-fabric-cluster-config-upgrade-windows-server.md). 

Découvrez comment [visualiser votre cluster à l’aide de l’outil Service Fabric Explorer](service-fabric-visualizing-your-cluster.md).
