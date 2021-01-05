---
title: Métriques personnalisées dans Azure Monitor (Préversion)
description: Découvrez les métriques personnalisées dans Azure Monitor et comment elles sont modélisées.
author: ancav
ms.author: ancav
services: azure-monitor
ms.topic: conceptual
ms.date: 06/01/2020
ms.subservice: metrics
ms.openlocfilehash: 73c9b2bf8cf88ca5e8576c451c9d9ac5f0eae8a3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88639900"
---
# <a name="custom-metrics-in-azure-monitor-preview"></a>Métriques personnalisées dans Azure Monitor (Préversion)

Quand vous déployez des ressources et des applications dans Azure, il est généralement utile de collecter des données de télémétrie pour obtenir des insights sur leurs performances et leur intégrité. Azure met à votre disposition des métriques prêtes à l’emploi. Il s’agit de métriques [standard ou de plateforme](./metrics-supported.md). Toutefois, ces métriques sont, par leur nature, limitées. 

Vous pouvez avoir besoin de collecter certains indicateurs de performance personnalisés ou des métriques métier pour obtenir des insights plus approfondis. Ces métriques **personnalisées** peuvent être collectées par le biais des données de télémétrie de votre application, d’un agent s’exécutant sur vos ressources Azure ou même d’un système de supervision d’interaction indirecte, puis soumises directement à Azure Monitor. Après leur publication dans Azure Monitor, vous pouvez parcourir et interroger les métriques personnalisées relatives à vos ressources et applications Azure (et créer des alertes sur ces métriques) parallèlement aux métriques standard émises par Azure.

Les métriques personnalisées d’Azure Monitor sont actuelles en préversion publique. 

## <a name="methods-to-send-custom-metrics"></a>Méthodes d’envoi de métriques personnalisées

Les métriques personnalisées peuvent être envoyées à Azure Monitor à l’aide de plusieurs méthodes :
- Instrumenter votre application en utilisant le SDK Azure Application Insights et envoyer des données de télémétrie personnalisées à Azure Monitor 
- Installez Azure Monitor Agent (préversion) sur votre [machine virtuelle Azure Windows ou Linux](azure-monitor-agent-overview.md) et utilisez une [règle de collecte de données](data-collection-rule-azure-monitor-agent.md) pour envoyer des compteurs de performances aux indicateurs de performance Azure Monitor.
- Installer l’extension Microsoft Azure Diagnostics (WAD) sur votre [machine virtuelle Azure](collect-custom-metrics-guestos-resource-manager-vm.md), votre [groupe de machines virtuelles identiques](collect-custom-metrics-guestos-resource-manager-vmss.md), votre [machine virtuelle classique](collect-custom-metrics-guestos-vm-classic.md) ou votre [instance Cloud Services classique](collect-custom-metrics-guestos-vm-cloud-service-classic.md), et envoyer des compteurs de performances à Azure Monitor 
- Installer [l’agent InfluxData Telegraf](collect-custom-metrics-linux-telegraf.md) sur votre machine virtuelle Linux Azure et envoyer les métriques à l’aide du plug-in de sortie Azure Monitor
- Envoyer des métriques personnalisées [directement à l’API REST Azure Monitor](./metrics-store-custom-rest-api.md) : `https://<azureregion>.monitoring.azure.com/<AzureResourceID>/metrics`

## <a name="pricing-model-and-retention"></a>Modèle de tarification et rétention

Pour plus d’informations sur l’activation de la facturation des requêtes de métriques personnalisées et des requêtes de métriques, consultez la [page de tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). Des détails spécifiques sur la grille tarifaire de toutes les métriques, y compris des métriques personnalisées et des requêtes de métriques, sont disponibles sur cette page. En résumé, il n’y a aucun coût d’ingestion pour les métriques standard (métriques de plateforme) dans le magasin de métriques d’Azure Monitor, mais les métriques personnalisées entraînent des coûts lorsqu’elles seront mises en disponibilité générale. Les requêtes d’API de métrique seront facturées.

Les métriques personnalisées sont conservées pendant la [même durée que les métriques de plateforme](data-platform-metrics.md#retention-of-metrics). 

> [!NOTE]  
> Les métriques envoyées à Azure Monitor via le Kit de développement logiciel (SDK) Application Insights sont facturées en tant que données de journal ingérées. Elles n’entraînent des frais supplémentaires que si la fonctionnalité [Activer les alertes sur les dimensions des métriques personnalisées](../app/pre-aggregated-metrics-log-metrics.md#custom-metrics-dimensions-and-pre-aggregation) d’Application Insights a été sélectionnée. Cette case à cocher détermine l’envoi de données à la base de données de métriques d’Azure Monitor à l’aide de l’API de métriques personnalisées pour permettre la création d’alertes plus complexes.  En savoir plus sur le [modèle de tarification Application Insights](../app/pricing.md#pricing-model) et les [prix dans votre région](https://azure.microsoft.com/pricing/details/monitor/).


## <a name="how-to-send-custom-metrics"></a>Comment envoyer des métriques personnalisées

Lorsque vous envoyez des métriques personnalisées à Azure Monitor, chaque point de données (ou valeur) rapporté doit inclure les informations qui suivent.

### <a name="authentication"></a>Authentification
Pour soumettre des métriques personnalisées à Azure Monitor, l’entité qui soumet la métrique doit disposer d’un jeton Azure Active Directory (Azure AD) valide, figurant dans l’en-tête **porteur** de la requête. Il existe plusieurs moyens d’acquérir un jeton du porteur valide :
1. [Identités managées pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md). Donne une identité à une ressource Azure, par exemple, une machine virtuelle. Managed Service Identity (MSI) est conçu pour accorder aux ressources des autorisations permettant d’effectuer certaines opérations. Il peut s’agir, par exemple, d’autoriser une ressource à générer des métriques à propos d’elle-même. Une ressource (ou son MSI) peut recevoir des autorisations **Surveillance de l’éditeur de métriques** pour une autre ressource. Avec cette autorisation, le MSI peut également générer des métriques sur d’autres ressources.
2. [Principal du service Azure AD](../../active-directory/develop/app-objects-and-service-principals.md). Dans ce scénario, une application (ou service) Azure AD peut se voir accorder les autorisations nécessaires pour générer des métriques concernant une ressource Azure.
Pour authentifier la requête, Azure Monitor valide le jeton d’application à l’aide de clés publiques Azure AD. Le rôle **Surveillance de l’éditeur de métriques** dispose déjà de cette autorisation. Cette autorisation est disponible dans le portail Azure. En fonction des ressources pour lesquelles il émettra des métriques personnalisées, le principal de service peut se voir accorder le rôle **Surveillance de l’éditeur de métriques** selon la portée nécessaire. Il peut s’agir d’un abonnement, d’un groupe de ressources ou d’une ressource.

> [!TIP]  
> Lorsque vous demandez à un jeton Azure AD de générer des métriques personnalisées, vérifiez que l’audience ou la ressource pour laquelle le jeton est demandé est `https://monitoring.azure.com/`. Veillez à inclure la barre oblique (/) à la fin.

### <a name="subject"></a>Objet
Cette propriété capture l’ID de ressource Azure pour lequel la métrique personnalisée est rapportée. Cette information sera codée dans l’URL de l’appel d’API effectué. Chaque API peut soumettre des valeurs de métrique pour une ressource Azure uniquement.

> [!NOTE]  
> Vous ne pouvez pas générer de métriques personnalisées pour l’ID de ressource d’un abonnement ou d’un groupe de ressources.


### <a name="region"></a>Région
Cette propriété capture la région Azure dans laquelle est déployée la ressource pour laquelle vous émettez des métriques. Les métriques doivent être émises vers le point de terminaison régional Azure Monitor correspondant à la région dans laquelle la ressource est déployée. Par exemple, les métriques personnalisées concernant une machine virtuelle déployée dans la région USA Ouest doivent être envoyées au point de terminaison Azure Monitor régional WestUS. Les informations de région sont également codées dans l’URL de l’appel d’API.

> [!NOTE]  
> Dans le cadre de la préversion publique, les métriques personnalisées sont disponibles dans un sous-ensemble de régions Azure uniquement. Une liste des régions prises en charge figure dans une section ultérieure de cet article.
>
>

### <a name="timestamp"></a>Timestamp
Chaque point de données envoyé à Azure Monitor doit être marqué par un timestamp. Cet horodatage capture la date et l’heure auxquelles la valeur de métrique a été mesurée ou collectée. Azure Monitor accepte les données métriques dont les horodatages ne datent pas de plus de 20 minutes et ne dépassent pas les 5 minutes à venir. Le timestamp doit être au format ISO 8601.

### <a name="namespace"></a>Espace de noms
Les espaces de noms offrent un moyen de grouper ou classer par catégorie des métriques similaires. Les espaces de noms permettent d’isoler les groupes de métriques collectant différents insights ou indicateurs de performances. On peut avoir par exemple un espace de noms nommé **contosomemorymetrics** qui effectue le suivi des métriques d’utilisation de la mémoire caractéristiques de l’application. Un autre espace de noms appelé **contosoapptransaction** pourrait effectuer le suivi de toutes les métriques relatives aux transactions utilisateur de l’application.

### <a name="name"></a>Nom
**Nom** correspond au nom de la métrique rapportée. Généralement, le nom est suffisamment descriptif pour identifier ce qui est mesuré. Par exemple, il peut s’agir d’une métrique qui mesure le nombre d’octets de mémoire utilisés sur une machine virtuelle donnée. Le nom de cette métrique pourrait être **Memory Bytes In Use** (Octets de mémoire en cours d’utilisation).

### <a name="dimension-keys"></a>Clés de dimension
Une dimension est une paire de clés ou de valeurs qui décrivent des caractéristiques supplémentaires concernant la métrique collectée. Ces caractéristiques supplémentaires permettent de collecter plus d’informations sur la métrique, offrant des insights plus approfondis. Par exemple, la métrique **Memory Bytes In Use** peut disposer d’une clé de dimension nommée **Process**, qui capture le nombre d’octets de mémoire consommés par chaque processus sur une machine virtuelle. Cette clé vous permet de filtrer les résultats de la métrique pour connaître la quantité de mémoire utilisée par certains processus ou identifier les cinq processus utilisant le plus de mémoire.
Les dimensions sont facultatives ; certaines métriques peuvent ne pas avoir de dimensions. Une métrique personnalisée peut avoir jusqu'à 10 dimensions.

### <a name="dimension-values"></a>Valeurs de dimension
Lorsqu’un point de données de métrique est rapporté, chaque clé de dimension rapportée est associée à une valeur de dimension. Par exemple, vous pouvez souhaiter que la mémoire utilisée par l’application ContosoApp sur votre machine virtuelle soit rapportée :

* Le nom de la métrique sera **Memory Bytes In Use**.
* La clé de dimension sera **Process**.
* La valeur de dimension sera **ContosoApp.exe**.

Lorsque vous publiez une valeur de métrique, vous pouvez spécifier une valeur de dimension par clé de dimension uniquement. Si vous collectez une même utilisation de la mémoire pour plusieurs processus sur la machine virtuelle, vous pouvez rapporter plusieurs valeurs de métrique pour cet horodatage. Chaque valeur de métrique spécifiera une valeur de dimension différente pour la clé de dimension **Process**.
Les dimensions sont facultatives ; certaines métriques peuvent ne pas avoir de dimensions. Si une publication de métriques définit des clés de dimension, les valeurs de dimension correspondantes sont obligatoires.

### <a name="metric-values"></a>Valeurs de métrique
Azure Monitor stocke toutes les métriques à intervalles réguliers (avec une granularité d’une minute). Nous savons qu’il peut être nécessaire d’échantillonner une métrique plusieurs fois au cours d’une minute donnée. C’est le cas, par exemple, avec l’utilisation du processeur. Vous pouvez également avoir besoin de mesurer une métrique pour de nombreux événements discrets. C’est le cas, par exemple, avec les latences de transaction de connexion. Pour limiter le nombre de valeurs brutes que vous devez émettre et payer dans Azure Monitor, vous pouvez pré-agréger les valeurs localement et les émettre :

* **Min** : valeur minimale observée parmi tous les échantillons et mesures au cours d’une minute donnée.
* **Max** : valeur maximale observée parmi tous les échantillons et mesures au cours d’une minute donnée.
* **Sum** : somme de toutes les valeurs observées parmi tous les échantillons et mesures au cours d’une minute donnée.
* **Nombre** : nombre d’échantillons et de mesures relevés au cours d’une minute donnée.

Par exemple, si quatre transactions de connexion à votre application ont été effectuées pendant une minute donnée, les latences mesurées pour chacune d’elles sont les suivantes :

|Transaction 1|Transaction 2|Transaction 3|Transaction 4|
|---|---|---|---|
|7 ms|4 ms|13 ms|16 ms|
|

La publication des métriques dans Azure Monitor se présentera ainsi :
* Min : 4
* Max : 16
* Somme : 40
* Nombre : 4

Si votre application ne parvient pas à pré-agréger les valeurs localement et doit émettre chaque événement ou échantillon discret immédiatement après collecte, vous pouvez émettre les valeurs de mesure brutes. Par exemple, à chaque exécution d’une transaction de connexion à votre application, vous publiez une métrique dans Azure Monitor avec une seule mesure. Par conséquent, pour une transaction de connexion qui a duré 12 ms, la publication de métrique se présentera ainsi :
* Min : 12
* Max : 12
* Somme : 12
* Nombre : 1

Ce processus vous permet de générer plusieurs valeurs pour la même combinaison métrique + dimension au cours d’une minute donnée. Azure Monitor prend ensuite toutes les valeurs brutes émises pendant une minute donnée, et les agrège.

### <a name="sample-custom-metric-publication"></a>Exemple de publication de métrique personnalisée
Dans l’exemple suivant, vous créez une métrique personnalisée appelée **Memory Bytes in Use** sous l’espace de noms de métrique **Memory Profile** pour une machine virtuelle. La métrique a une seule dimension appelée **Process**. Pour l’horodatage donné, nous émettons des valeurs de métrique pour deux processus différents :

```json
{
    "time": "2018-08-20T11:25:20-7:00",
    "data": {

      "baseData": {

        "metric": "Memory Bytes in Use",
        "namespace": "Memory Profile",
        "dimNames": [
          "Process"        ],
        "series": [
          {
            "dimValues": [
              "ContosoApp.exe"
            ],
            "min": 10,
            "max": 89,
            "sum": 190,
            "count": 4
          },
          {
            "dimValues": [
              "SalesApp.exe"
            ],
            "min": 10,
            "max": 23,
            "sum": 86,
            "count": 4
          }
        ]
      }
    }
  }
```
> [!NOTE]  
> Application Insights, l’extension de diagnostics et l’agent InfluxData Telegraf sont déjà configurés pour générer des valeurs de métrique sur le bon point de terminaison régional, ainsi que pour intégrer toutes les propriétés précédentes dans chaque émission.
>
>

## <a name="custom-metric-definitions"></a>Définitions de métriques personnalisées
Il n’est pas nécessaire de prédéfinir une métrique personnalisée dans Azure Monitor avant son émission. Chaque point de données de métrique publié contient un espace de noms, un nom et des informations de dimension. Ainsi, la première fois qu’une métrique personnalisée est émise vers Azure Monitor, une définition de métrique est automatiquement créée. Cette définition de métrique est ensuite découvrable sur n’importe quelle ressource pour laquelle la métrique a été émise via les définitions de métriques.

> [!NOTE]  
> Azure Monitor ne prend pas encore en charge la définition des **unités** pour une métrique personnalisée.

## <a name="using-custom-metrics"></a>Utilisation de métriques personnalisées
Une fois les métriques personnalisées envoyées à Azure Monitor, vous pouvez les parcourir dans le portail Azure et les interroger via les API REST Azure Monitor. Vous pouvez également créer des alertes les concernant, afin d’être averti lorsque certaines conditions sont remplies.

> [!NOTE]
> Vous devez avoir un rôle Lecteur ou Contributeur pour afficher des métriques personnalisées.

### <a name="browse-your-custom-metrics-via-the-azure-portal"></a>Parcourir vos métriques personnalisées dans le Portail Azure
1.    Accédez au [portail Azure](https://portal.azure.com).
2.    Sélectionnez le volet **Surveiller**.
3.    Sélectionnez **Métriques**.
4.    Sélectionnez une ressource pour laquelle vous avez émis des métriques personnalisées.
5.    Sélectionnez l’espace de noms de métrique de votre métrique personnalisée.
6.    Sélectionnez la métrique personnalisée.

## <a name="supported-regions"></a>Régions prises en charge
Dans la préversion publique, la publication des métriques personnalisées n’est possible que dans un sous-ensemble de régions Azure. Cette restriction signifie que vous ne pouvez publier des métriques que pour les ressources qui sont situées dans l’une de ces régions. Le tableau suivant répertorie les régions Azure qui prennent en charge les métriques personnalisées. Il répertorie également les points de terminaison où peuvent être publiées les métriques concernant les ressources situées dans ces régions :

|Région Azure |Préfixe du point de terminaison régional|
|---|---|
| **États-Unis et Canada** | |
|Centre-USA Ouest | https:\//westcentralus.monitoring.azure.com |
|USA Ouest 2       | https:\//westus2.monitoring.azure.com |
|Centre-Nord des États-Unis | https:\//northcentralus.monitoring.azure.com
|États-Unis - partie centrale méridionale| https:\//southcentralus.monitoring.azure.com |
|USA Centre      | https:\//centralus.monitoring.azure.com |
|Centre du Canada | https:\//canadacentral.monitoring.azure.com |
|USA Est| https:\//eastus.monitoring.azure.com |
|USA Est 2 | https:\//eastus2.monitoring.azure.com |
| **Europe** | |
|Europe Nord    | https:\//northeurope.monitoring.azure.com |
|Europe Ouest     | https:\//westeurope.monitoring.azure.com |
|Sud du Royaume-Uni | https:\//uksouth.monitoring.azure.com
|France Centre | https:\//francecentral.monitoring.azure.com |
| **Afrique** | |
|Afrique du Sud Nord | https:\//southafricanorth.monitoring.azure.com |
| **Asie** | |
|Inde centrale | https:\//centralindia.monitoring.azure.com |
|Australie Est | https:\//australiaeast.monitoring.azure.com |
|Japon Est | https:\//japaneast.monitoring.azure.com |
|Asie Sud-Est  | https:\//southeastasia.monitoring.azure.com |
|Asie Est | https:\//eastasia.monitoring.azure.com |
|Centre de la Corée   | https:\//koreacentral.monitoring.azure.com |

## <a name="latency-and-storage-retention"></a>Conservation de stockage et latence

L’apparition de l’ajout d’une nouvelle métrique ou d’une nouvelle dimension à une métrique peut prendre de deux à trois minutes. Une fois dans le système, les données doivent apparaître en moins de 30 secondes 99 % du temps. 

Si vous supprimez une métrique ou une dimension, il faudra peut-être d’une semaine à un mois pour que la modification soit supprimée du système.

## <a name="quotas-and-limits"></a>Quotas et limites
Azure Monitor impose les limites d’utilisation suivantes quant aux métriques personnalisées :

|Category|Limite|
|---|---|
|Série chronologique active/abonnements/région|50 000|
|Clés de dimension par métrique|10|
|Longueur de chaîne pour les espaces de noms de métrique, les noms de métrique, les clés de dimension et les valeurs de dimension|256 caractères|

Une série chronologique active se définit comme toute combinaison unique de métriques, clés de dimension ou valeurs de dimension pour laquelle des valeurs de métrique ont été publiées au cours des 12 dernières heures.

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez utiliser les métriques personnalisées à partir de différents services : 
 - [Machines virtuelles](collect-custom-metrics-guestos-resource-manager-vm.md)
 - [Groupe de machines virtuelles identiques](collect-custom-metrics-guestos-resource-manager-vmss.md)
 - [Machines virtuelles Azure (Classic)](collect-custom-metrics-guestos-vm-classic.md)
 - [Machine virtuelle Linux utilisant l’agent Telegraf](collect-custom-metrics-linux-telegraf.md)
 - [REST API](./metrics-store-custom-rest-api.md)
 - [Services cloud (classiques)](collect-custom-metrics-guestos-vm-cloud-service-classic.md)
 

