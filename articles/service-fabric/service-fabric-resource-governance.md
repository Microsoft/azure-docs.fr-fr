---
title: La gestion des ressources pour les conteneurs et les services
description: Azure Service Fabric vous permet de spécifier des limites et des requêtes de ressources pour les services en cours d’exécution en tant que processus ou conteneurs.
ms.topic: conceptual
ms.date: 8/9/2017
ms.openlocfilehash: 889fce77c1a3a743e9805ec482a9c87b9bf8da65
ms.sourcegitcommit: 2989396c328c70832dcadc8f435270522c113229
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92172868"
---
# <a name="resource-governance"></a>Gouvernance des ressources

Lors de l’exécution de plusieurs services sur le même nœud ou cluster, il est possible qu’un service consomme davantage de ressources au détriment des autres services du processus. C’est ce que l’on appelle le problème du « voisin bruyant ». Azure Service Fabric permet au développeur de contrôler ce comportement en spécifiant des requêtes et des limites par service afin limiter l’utilisation des ressources.

> Avant de poursuivre la lecture de cet article, nous vous recommandons de vous familiariser avec le [modèle d’application Service Fabric][application-model-link] et le [modèle d’hébergement Service Fabric][hosting-model-link].
>

## <a name="resource-governance-metrics"></a>Mesures de la gouvernance des ressources

Dans Service Fabric, la gouvernance des ressources est prise en charge conformément au [package de service][application-model-link]. Les ressources qui sont affectées au package de service peuvent être subdivisées entre les packages de code. Service Fabric prend en charge la gouvernance des ressources processeur et mémoire par package de service à l’aide de deux [mesures](service-fabric-cluster-resource-manager-metrics.md) intégrées :

* *Processeur* (nom de la métrique `servicefabric:/_CpuCores`) : cœur logique qui est disponible sur la machine hôte. Tous les cœurs de tous les nœuds sont pondérés de la même façon.

* *Mémoire* (nom de la métrique `servicefabric:/_MemoryInMB`) : la mémoire est exprimée en mégaoctets et elle est mise en correspondance avec la mémoire physique disponible sur la machine.

Pour ces deux mesures, [Cluster Resource Manager][cluster-resource-manager-description-link] (CRM) surveille la capacité totale du cluster, la charge sur chaque nœud du cluster, ainsi que les ressources restantes dans le cluster. Ces deux mesures sont équivalentes à un autre utilisateur ou une mesure personnalisée. Toutes les fonctionnalités existantes peuvent être utilisées avec ces mesures :

* Le cluster peut être [équilibré](service-fabric-cluster-resource-manager-balancing.md) en fonction de ces deux mesures (comportement par défaut).
* Le cluster peut être [défragmenté](service-fabric-cluster-resource-manager-defragmentation-metrics.md) en fonction de ces deux mesures.
* Lors de la [description d’un cluster][cluster-resource-manager-description-link], la capacité mise en mémoire tampon peut être définie pour ces deux mesures.

> [!NOTE]
> Le [rapport de charge dynamique](service-fabric-cluster-resource-manager-metrics.md) n’est pas pris en charge pour ces métriques ; les charges de ces métriques sont définies lors de la création.

## <a name="resource-governance-mechanism"></a>Mécanisme de la gouvernance des ressources

À partir de la version 7.2, le runtime Service Fabric prend en charge la spécification des demandes et des limites pour les ressources processeur et mémoire.

> [!NOTE]
> Les versions du runtime Service Fabric antérieures à la v7.2 prennent uniquement en charge un modèle dans lequel une seule valeur sert à la fois de **requête** et de **limite** pour une ressource particulière (processeur ou mémoire). Cela est décrit comme la spécification **RequestsOnly** dans ce document.

* *Requêtes :* Les valeurs des requêtes de processeur et de mémoire représentent les charges utilisées par [Cluster Resource Manager (CRM)][cluster-resource-manager-description-link] pour les métriques `servicefabric:/_CpuCores` et `servicefabric:/_MemoryInMB` . En d’autres termes, CRM considère que la consommation de ressources d’un service est égale à ses valeurs de demande et utilise ces valeurs lors de la prise de décisions de placement.

* *Limites :* Les valeurs limites de processeur et de la mémoire représentent les limites de ressources réelles appliquées lorsqu’un processus ou un conteneur est activé sur un nœud.

Service Fabric autorise les spécifications **RequestsOnly, LimitsOnly** et **RequestsAndLimits** pour le processeur et la mémoire.
* Lorsque la spécification RequestsOnly est utilisée, Service Fabric utilise également les valeurs de requête comme limites.
* Lorsque la spécification LimitsOnly est utilisée, Service Fabric considère que les valeurs de la requête sont égales à 0.
* Lorsque la spécification RequestsAndLimits est utilisée, les valeurs limites doivent être supérieures ou égales aux valeurs de la requête.

Pour mieux comprendre le mécanisme de gouvernance des ressources, examinons un exemple de scénario de placement avec une spécification **RequestsOnly** pour la ressource de processeur (le mécanisme de gouvernance de mémoire est équivalent). Prenons l’exemple d’un nœud avec deux cœurs d’unité centrale et deux packages de service qui seront placés sur celui-ci. Le premier package de service à placer est constitué d’un seul package de code de conteneur et spécifie une requête d’un seul cœur de processeur. Le deuxième package de service à placer est constitué d’un seul package de code basé sur un processus et spécifie également une requête d’un seul cœur de processeur. Étant donné que les deux packages de service ont une spécification RequestsOnly, leurs valeurs limites sont définies sur leurs valeurs de requête.

1. Tout d’abord, le package de service basé sur un conteneur qui demande un cœur de processeur est placé sur le nœud. Le runtime active le conteneur et définit la limite de processeur à un seul cœur. Le conteneur ne pourra pas utiliser plus d’un cœur.

2. Ensuite, le package de service basé sur un processus qui demande un cœur de processeur est placé sur le nœud. Le runtime active le processus de service et définit sa limite de processeur à un seul cœur.

À ce stade, la somme des requêtes est égale à la capacité du nœud. CRM ne placera plus de conteneurs ou de processus de service avec des demandes d’UC sur ce nœud. Sur le nœud, un conteneur et un processus sont en cours d’exécution avec un seul cœur chacun et n’interfèrent pas entre eux pour le processeur.

Nous allons maintenant revisiter notre exemple avec une spécification **RequestsAndLimits** . Cette fois, le package de service basé sur un conteneur spécifie une requête d’un cœur de processeur et une limite de deux cœurs de processeur. Le package de service basé sur un processus spécifie une requête et une limite d’un cœur de processeur.
  1. Tout d’abord, le package de service basé sur un conteneur est placé sur le nœud. Le runtime active le conteneur et définit la limite de processeur à deux cœurs. Le conteneur ne pourra pas utiliser plus de deux cœurs.
  2. Ensuite, le package de service basé sur un processus est placé sur le nœud. Le runtime active le processus de service et définit sa limite de processeur à un seul cœur.

  À ce stade, la somme des demandes de processeur des packages de service placés sur le nœud est égale à la capacité de l’UC du nœud. CRM ne placera plus de conteneurs ou de processus de service avec des demandes d’UC sur ce nœud. Toutefois, sur le nœud, la somme des limites (deux cœurs pour le conteneur + un cœur pour le processus) dépasse la capacité de deux cœurs. Si le conteneur et le processus s’éclatent en même temps, il existe une possibilité de contention de la ressource processeur. Cette contention sera managée par le système d’exploitation sous-jacent pour la plateforme. Pour cet exemple, le conteneur peut augmenter jusqu’à deux cœurs de processeur, ce qui entraîne la demande de processus d’un cœur de processeur qui n’est pas garanti.

> [!NOTE]
> Comme illustré dans l’exemple précédent, les valeurs de requête pour le processeur et la mémoire **n’entraînent pas la réservation de ressources sur un nœud** . Ces valeurs représentent la consommation de ressources que le gestionnaire des ressources clusters prend en compte lors de la prise de décisions de placement. Les valeurs limites représentent les limites de ressources réelles appliquées lorsqu’un processus ou un conteneur est activé sur un nœud.


Il existe quelques situations dans lesquelles il peut y avoir une contention pour l’UC. Dans ces situations, le processus et le conteneur de notre exemple peuvent être confrontés au problème du voisin bruyant :

* *Combinaison de conteneurs et de services gouvernés et non gouvernés*  : si un utilisateur crée un service sans spécifier de gouvernance des ressources, le runtime considère qu’il ne consomme pas de ressources et peut donc le placer sur le nœud de notre exemple. Dans ce cas, ce nouveau processus consomme en réalité certaines ressources de processeur aux dépens des services qui s’exécutent déjà sur le nœud. Il existe deux solutions à ce problème. Il suffit de ne pas combiner de services régis et non régis sur le même cluster ou d’utiliser des [contraintes de placement](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md) pour éviter que les deux types de services se trouvent sur le même ensemble de nœuds.

* *Quand un autre processus est démarré sur le nœud, en dehors de Service Fabric (par exemple, un service du système d’exploitation)*  : dans ce cas, le processus en dehors de Service Fabric entre aussi en concurrence avec les services existants au niveau processeur. La solution à ce problème consiste à configurer correctement la capacité des nœuds de façon à prendre en compte la surcharge du système d’exploitation, comme indiqué dans la section suivante.

* *Lorsque les requêtes ne sont pas égales aux limites*  : Comme décrit dans l’exemple RequestsAndLimits précédent, les demandes n’entraînent pas la réservation de ressources sur un nœud. Lorsqu’un service avec des limites supérieures aux demandes est placé sur un nœud, il peut consommer des ressources (le cas échéant) jusqu’aux limites. Dans ce cas, d’autres services sur le nœud peuvent ne pas être en mesure de consommer des ressources jusqu’à leur valeur de requête.

## <a name="cluster-setup-for-enabling-resource-governance"></a>Configuration du cluster pour l’activation de la gouvernance des ressources

Quand un nœud démarre et rejoint le cluster, Service Fabric détecte la quantité de mémoire disponible, ainsi que le nombre de cœurs disponibles et définit la capacité des nœuds pour ces deux ressources.

Pour laisser de l’espace de mémoire tampon au système d’exploitation et aux autres processus susceptibles de s’exécuter sur le nœud, Service Fabric utilise seulement 80 % des ressources disponibles sur le nœud. Ce pourcentage est configurable et peut être modifié dans le manifeste du cluster.

Voici un exemple qui montre comment ordonner à Service Fabric d’utiliser 50 % de la capacité de processeur disponible et 70 % de la quantité de mémoire disponible :

```xml
<Section Name="PlacementAndLoadBalancing">
    <!-- 0.0 means 0%, and 1.0 means 100%-->
    <Parameter Name="CpuPercentageNodeCapacity" Value="0.5" />
    <Parameter Name="MemoryPercentageNodeCapacity" Value="0.7" />
</Section>
```

Pour la plupart des clients et des scénarios, la détection automatique des capacités de nœud pour l’UC et la mémoire est la configuration recommandée (la détection automatique est activée par défaut). Toutefois, si vous avez besoin de procéder à une configuration entièrement manuelle des capacités des nœuds, vous pouvez les configurer par type de nœud à l’aide du mécanisme permettant de décrire les nœuds du cluster. Voici un exemple de configuration du type de nœud avec 4 cœurs et 2 Go de mémoire :

```xml
    <NodeType Name="MyNodeType">
      <Capacities>
        <Capacity Name="servicefabric:/_CpuCores" Value="4"/>
        <Capacity Name="servicefabric:/_MemoryInMB" Value="2048"/>
      </Capacities>
    </NodeType>
```

Quand la détection automatique des ressources disponibles est activée et que la capacité du nœud est définie manuellement dans le manifeste de cluster, Service Fabric vérifie que le nœud a suffisamment de ressources pour prendre en charge la capacité définie par l’utilisateur :

* Si la capacité du nœud définie dans le manifeste est inférieure ou égale aux ressources disponibles sur le nœud, Service Fabric utilise la capacité spécifiée dans le manifeste.

* Si la capacité du nœud définie dans le manifeste est supérieure aux ressources disponibles, Service Fabric utilise les ressources disponibles comme capacité du nœud.

La détection automatique des ressources disponibles peut être désactivée si elle n’est pas nécessaire. Pour la désactiver, modifiez le paramètre suivant :

```xml
<Section Name="PlacementAndLoadBalancing">
    <Parameter Name="AutoDetectAvailableResources" Value="false" />
</Section>
```

Pour des performances optimales, le paramètre suivant doit également être activé dans le manifeste du cluster :

```xml
<Section Name="PlacementAndLoadBalancing">
    <Parameter Name="PreventTransientOvercommit" Value="true" />
    <Parameter Name="AllowConstraintCheckFixesDuringApplicationUpgrade" Value="true" />
</Section>
```

> [!IMPORTANT]
> À compter de Service Fabric version 7.0, nous avons mis à jour la règle de calcul de la capacité des ressources de nœud dans les cas où l’utilisateur fournit manuellement la valeur de cette capacité. Considérons le scénario suivant :
>
> * Il y a au total 10 cœurs de processeur sur le nœud
> * SF est configuré pour utiliser 80 % du total des ressources pour les services d’utilisateurs (paramètre par défaut), ce qui laisse une mémoire tampon de 20 % pour les autres services qui s’exécutent sur le nœud (y compris les services système Service Fabric).
> * L’utilisateur décide de remplacer manuellement la capacité des ressources de nœud pour la métrique des cœurs de processeur et de lui affecter la valeur 5 cœurs
>
> Nous avons modifié ainsi la règle de calcul de la capacité disponible pour les services d’utilisateurs Service Fabric :
>
> * Avant Service Fabric 7.0, la capacité disponible pour les services utilisateur était évaluée à **5 cœurs** (la mémoire tampon de capacité de 20 % était ignorée).
> * À compter de Service Fabric 7.0, la capacité disponible pour les services d’utilisateurs est évaluée à **4 cœurs** (la mémoire tampon de capacité de 20 % n’est pas ignorée).

## <a name="specify-resource-governance"></a>Spécifier la gouvernance des ressources

Les limites et les requêtes de gouvernance des ressources sont spécifiées dans le manifeste d’application (section ServiceManifestImport). Prenons quelques exemples :

**Exemple 1 : Spécification RequestsOnly**
```xml
<?xml version='1.0' encoding='UTF-8'?>
<ApplicationManifest ApplicationTypeName='TestAppTC1' ApplicationTypeVersion='vTC1' xsi:schemaLocation='http://schemas.microsoft.com/2011/01/fabric ServiceFabricServiceModel.xsd' xmlns='http://schemas.microsoft.com/2011/01/fabric' xmlns:xsi='https://www.w3.org/2001/XMLSchema-instance'>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName='ServicePackageA' ServiceManifestVersion='v1'/>
    <Policies>
      <ServicePackageResourceGovernancePolicy CpuCores="1"/>
      <ResourceGovernancePolicy CodePackageRef="CodeA1" CpuShares="512" MemoryInMB="1024" />
      <ResourceGovernancePolicy CodePackageRef="CodeA2" CpuShares="256" MemoryInMB="1024" />
    </Policies>
  </ServiceManifestImport>
```

Dans cet exemple, l’attribut `CpuCores` est utilisé pour spécifier une requête de 1 cœur de processeur pour **ServicePackageA** . Étant donné que la limite de l’UC (attribut `CpuCoresLimit`) n’est pas spécifiée, Service Fabric utilise également la valeur de demande spécifiée de 1 cœur comme limite d’UC pour le package de services.

**ServicePackageA** sera placé sur un nœud dont la capacité restante de l’UC après la soustraction de la **somme des demandes d’UC de tous les packages de service placés sur ce nœud** est supérieure ou égale à 1 cœur. Sur le nœud, le package de service sera limité à un cœur. Ce package de service contient deux packages de code ( **CodeA1** et **CodeA2** ), et tous deux spécifient l’attribut `CpuShares`. La proportion de CpuShares 512:256 est utilisée pour calculer les limites du processeur pour les packages de code individuels. Par conséquent, CodeA1 sera limité à deux tiers d’un cœur et CodeA2 sera limité à un tiers d’un cœur. Si les CpuShares ne sont pas spécifiés pour tous les packages de code, Service Fabric divise la limite d’UC équitablement entre eux.

Tandis que les CpuShares spécifiés pour les packages de code représentent leur proportion relative de la limite globale de l’UC du package de service, les valeurs de mémoire pour les packages de code sont spécifiées en termes absolus. Dans cet exemple, l’attribut `MemoryInMB` est utilisé pour spécifier les demandes de mémoire de 1024 Mo pour CodeA1 et CodeA2. Étant donné que la limite de mémoire (attribut `MemoryInMBLimit`) n’est pas spécifiée, Service Fabric utilise également les valeurs de demande spécifiées comme limites pour les packages de code. La demande de mémoire (et la limite) pour le package de services est calculée comme la somme des valeurs de requête (et limite) de la mémoire de ses packages de code constitutifs. Par conséquent, pour **ServicePackageA** , la requête et la limite de mémoire sont calculées comme 2048 Mo.

**ServicePackageA** sera placé sur un nœud dont la capacité restante de mémoire après la soustraction de la **somme des demandes de mémoire de tous les packages de service placés sur ce nœud** est supérieure ou égale à 2048 Mo. Sur le nœud, les deux packages de code sont limités à 1024 Mo de mémoire chacun. Les packages de code (conteneurs ou processus) seront pas en mesure d’allouer plus de mémoire que cette limite. Toute tentative en ce sens conduit à des exceptions de mémoire insuffisante.

**Exemple 2 : Spécification LimitsOnly**
```xml
<?xml version='1.0' encoding='UTF-8'?>
<ApplicationManifest ApplicationTypeName='TestAppTC1' ApplicationTypeVersion='vTC1' xsi:schemaLocation='http://schemas.microsoft.com/2011/01/fabric ServiceFabricServiceModel.xsd' xmlns='http://schemas.microsoft.com/2011/01/fabric' xmlns:xsi='https://www.w3.org/2001/XMLSchema-instance'>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName='ServicePackageA' ServiceManifestVersion='v1'/>
    <Policies>
      <ServicePackageResourceGovernancePolicy CpuCoresLimit="1"/>
      <ResourceGovernancePolicy CodePackageRef="CodeA1" CpuShares="512" MemoryInMBLimit="1024" />
      <ResourceGovernancePolicy CodePackageRef="CodeA2" CpuShares="256" MemoryInMBLimit="1024" />
    </Policies>
  </ServiceManifestImport>
```
Cet exemple utilise des attributs `CpuCoresLimit` et `MemoryInMBLimit`, qui sont uniquement disponibles dans les versions SF 7.2 et ultérieures. L’attribut CpuCoresLimit est utilisé pour spécifier une limite d’UC de 1 cœur pour **ServicePackageA** . Étant donné que la demande d’UC (attribut `CpuCores`) n’est pas spécifiée, elle est considérée comme égale à 0. L’attribut `MemoryInMBLimit` est utilisé pour spécifier des limites de mémoire de 1024 Mo pour CodeA1 et CodeA2 et puisque les demandes (attribut `MemoryInMB`) ne sont pas spécifiées, elles sont considérées comme égales à 0. Par conséquent, pour **ServicePackageA** , la requête et la limite de mémoire sont calculées comme 0 Mo et 2048 Mo respectivement. Étant donné que les demandes d’UC et de mémoire pour **ServicePackageA** sont égales à 0, il n’y a aucune charge à prendre en compte pour le positionnement de CRM, pour les métriques `servicefabric:/_CpuCores` et `servicefabric:/_MemoryInMB`. Par conséquent, du point de vue de la gouvernance des ressources, **ServicePackageA** peut être placé sur n’importe quel nœud **indépendamment de la capacité restante** . Comme dans l’exemple 1, sur le nœud, CodeA1 sera limité à deux tiers d’un cœur et 1024 Mo de mémoire, et CodeA2 sera limité à un tiers d’un cœur et à 1024 Mo de mémoire.

**Exemple 3 : Spécification RequestsAndLimits**
```xml
<?xml version='1.0' encoding='UTF-8'?>
<ApplicationManifest ApplicationTypeName='TestAppTC1' ApplicationTypeVersion='vTC1' xsi:schemaLocation='http://schemas.microsoft.com/2011/01/fabric ServiceFabricServiceModel.xsd' xmlns='http://schemas.microsoft.com/2011/01/fabric' xmlns:xsi='https://www.w3.org/2001/XMLSchema-instance'>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName='ServicePackageA' ServiceManifestVersion='v1'/>
    <Policies>
      <ServicePackageResourceGovernancePolicy CpuCores="1" CpuCoresLimit="2"/>
      <ResourceGovernancePolicy CodePackageRef="CodeA1" CpuShares="512" MemoryInMB="1024" MemoryInMBLimit="3072" />
      <ResourceGovernancePolicy CodePackageRef="CodeA2" CpuShares="256" MemoryInMB="2048" MemoryInMBLimit="4096" />
    </Policies>
  </ServiceManifestImport>
```
En s’appuyant sur les deux premiers exemples, cet exemple montre comment spécifier les demandes et les limites de l’UC et de la mémoire. **ServicePackageA** a des demandes d’UC et de mémoire de 1 cœur et de 3072 Mo (1024 + 2048) respectivement. Il ne peut être placé que sur un nœud qui a au moins 1 cœur (et 3072 Mo) de capacité restante après avoir soustrait la somme de toutes les demandes de processeur (et de mémoire) de tous les packages de service placés sur le nœud à partir de la capacité totale du processeur (et de la mémoire) du nœud. Sur le nœud, CodeA1 sera limité à deux tiers de 2 cœurs et à 3072 Mo de mémoire, tandis que CodeA2 sera limité à un tiers de 2 cœurs et à 4096 Mo de mémoire.

### <a name="using-application-parameters"></a>Utilisation des paramètres d'application

Lorsque l’on spécifie les paramètres de gouvernance des ressources, il est possible d’utiliser les [Paramètres de l’application](service-fabric-manage-multiple-environment-app-configuration.md) pour gérer plusieurs configurations d’application. L’exemple suivant illustre l’utilisation de paramètres d’application :

```xml
<?xml version='1.0' encoding='UTF-8'?>
<ApplicationManifest ApplicationTypeName='TestAppTC1' ApplicationTypeVersion='vTC1' xsi:schemaLocation='http://schemas.microsoft.com/2011/01/fabric ServiceFabricServiceModel.xsd' xmlns='http://schemas.microsoft.com/2011/01/fabric' xmlns:xsi='https://www.w3.org/2001/XMLSchema-instance'>

  <Parameters>
    <Parameter Name="CpuCores" DefaultValue="4" />
    <Parameter Name="CpuSharesA" DefaultValue="512" />
    <Parameter Name="CpuSharesB" DefaultValue="512" />
    <Parameter Name="MemoryA" DefaultValue="2048" />
    <Parameter Name="MemoryB" DefaultValue="2048" />
  </Parameters>

  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName='ServicePackageA' ServiceManifestVersion='v1'/>
    <Policies>
      <ServicePackageResourceGovernancePolicy CpuCores="[CpuCores]"/>
      <ResourceGovernancePolicy CodePackageRef="CodeA1" CpuShares="[CpuSharesA]" MemoryInMB="[MemoryA]" />
      <ResourceGovernancePolicy CodePackageRef="CodeA2" CpuShares="[CpuSharesB]" MemoryInMB="[MemoryB]" />
    </Policies>
  </ServiceManifestImport>
```

Dans cet exemple, les valeurs par défaut sont définies pour l’environnement de production, où chaque package de service obtient 4 cœurs et 2 Go de mémoire. Il est possible de modifier les valeurs par défaut avec des fichiers de paramètres d’application. Dans cet exemple, un fichier de paramètre peut être utilisé pour tester l’application localement, là où elle obtiendrait moins de ressources qu’en production :

```xml
<!-- ApplicationParameters\Local.xml -->

<Application Name="fabric:/TestApplication1" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="CpuCores" DefaultValue="2" />
    <Parameter Name="CpuSharesA" DefaultValue="512" />
    <Parameter Name="CpuSharesB" DefaultValue="512" />
    <Parameter Name="MemoryA" DefaultValue="1024" />
    <Parameter Name="MemoryB" DefaultValue="1024" />
  </Parameters>
</Application>
```

> [!IMPORTANT]
> La spécification de la gouvernance des ressources avec des paramètres d’application est disponible à partir de Service Fabric version 6.1.<br>
>
> Lorsque les paramètres de l’application sont utilisés pour spécifier la gouvernance des ressources, Service Fabric ne peut pas être rétrogradé à une version antérieure à la version 6.1.

## <a name="enforcing-the-resource-limits-for-user-services"></a>Appliquer les limites de ressources des services d’utilisateurs

Bien que l’application de la gouvernance des ressources à des services Service Fabric garantit que ces services régis par les ressources ne peuvent pas dépasser leur quota de ressources, de nombreux utilisateurs ont besoin d’exécuter certains de leurs services Service Fabric en mode non régi. Avec des services Service Fabric non régis, il est possible de rencontrer des situations dans lesquelles des services non régis « en perte de contrôle » consomment toutes les ressources disponibles sur les nœuds Service Fabric, ce qui peut entraîner de graves problèmes :

* insuffisance de ressources d’autres services qui s’exécutent sur les nœuds (y compris les services système Service Fabric) ;
* nœuds se retrouvant dans un état non sain ;
* API de gestion de clusters Service Fabric qui ne répondent pas.

Pour éviter ces cas de figure, Service Fabric offre la possibilité  *d’appliquer les limites de ressources pour tous les services d’utilisateurs Service Fabric qui s’exécutent sur le nœud* (régis ou non) afin de garantir qu’ils n’utiliseront jamais plus de ressources que la quantité spécifiée. Pour cela, définissez sur true la valeur de la configuration EnforceUserServiceMetricCapacities dans la section PlacementAndLoadBalancing de ClusterManifest. Le paramètre est désactivé par défaut.

```xml
<SectionName="PlacementAndLoadBalancing">
    <ParameterName="EnforceUserServiceMetricCapacities" Value="false"/>
</Section>
```

Remarques supplémentaires :

* L’application de limites de ressources s’applique uniquement aux métriques de ressource `servicefabric:/_CpuCores` et `servicefabric:/_MemoryInMB`.
* L’application de limites de ressources ne fonctionne que si les capacités de nœud des métriques de ressource sont accessibles à Service Fabric, par le biais d’un mécanisme de détection automatique ou d’utilisateurs qui les spécifient manuellement (comme l’explique la section [Configuration de cluster pour activer la gouvernance des ressources](service-fabric-resource-governance.md#cluster-setup-for-enabling-resource-governance)).  Si les capacités de nœud ne sont pas configurées, la capacité d’application de limites de ressources n’est pas utilisable, car Service Fabric ne connaît pas le volume de ressources à réserver pour les services d’utilisateurs.  Service Fabric émet un avertissement d’intégrité si « EnforceUserServiceMetricCapacities » a la valeur true, mais que les capacités de nœud ne sont pas configurées.

## <a name="other-resources-for-containers"></a>Autres ressources pour les conteneurs

En plus du processeur et de la mémoire, il est possible de spécifier d’autres limites de ressources pour les conteneurs. Ces limites sont spécifiées au niveau du package de code et s’appliquent au démarrage du conteneur. Contrairement au processeur et à la mémoire, Cluster Resource Manager ne perçoit pas ces ressources et n’assure aucune tâche de vérification de ressources ou d’équilibrage de charge sur celles-ci.

* *MemorySwapInMB*  : quantité de mémoire d’échange qu’un conteneur peut utiliser.
* *MemoryReservationInMB*  : Limite non stricte de la gouvernance de mémoire qui s’applique uniquement quand une contention de mémoire est détectée sur le nœud.
* *CpuPercent*  : pourcentage de la capacité du processeur que le conteneur peut utiliser. Si des demandes ou limites de processeur sont spécifiées pour le package de services, ce paramètre est en réalité ignoré.
* *MaximumIOps*  : nombre maximal d’E/S par seconde qu’un conteneur peut utiliser (en lecture et écriture).
* *MaximumIOBytesps*  : nombre maximal d’E/S (octets par seconde) qu’un conteneur peut utiliser (en lecture et écriture).
* *BlockIOWeight*  : poids des E/S de bloc par rapport aux autres conteneurs.

Ces ressources peuvent être combinées avec le processeur et la mémoire. Voici un exemple illustration la définition de ressources supplémentaires pour des conteneurs :

```xml
    <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
        <Policies>
            <ResourceGovernancePolicy CodePackageRef="FrontendService.Code" CpuPercent="5"
            MemorySwapInMB="4084" MemoryReservationInMB="1024" MaximumIOPS="20" />
        </Policies>
    </ServiceManifestImport>
```

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur Cluster Resource Manager, consultez [Présentation de Service Fabric Cluster Resource Manager](service-fabric-cluster-resource-manager-introduction.md).
* Pour en savoir plus sur le modèle d’application, les packages de service, les packages de code et leur mappage avec les réplicas, consultez [Modéliser une application dans Service Fabric][application-model-link].

<!-- Links -->
[application-model-link]: service-fabric-application-model.md
[hosting-model-link]: service-fabric-hosting-model.md
[cluster-resource-manager-description-link]: service-fabric-cluster-resource-manager-cluster-description.md
