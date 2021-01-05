---
title: Étiquettes de service de groupe de sécurité réseau (NSG) pour Azure HDInsight
description: Utilisez des étiquettes de service HDInsight pour autoriser le trafic entrant vers votre cluster à partir des nœuds des services de contrôle d'intégrité et de gestion, sans ajouter d'adresses IP à vos groupes de sécurité réseau.
author: hrasheed-msft
ms.author: hrasheed
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 08/11/2020
ms.openlocfilehash: 1a90bc6636dcb3aa81f09b0489850c1a95b3256d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022749"
---
# <a name="nsg-service-tags-for-azure-hdinsight"></a>Étiquettes de service Azure HDInsight des groupes de sécurité réseau

Les étiquettes de service Azure HDInsight des groupes de sécurité réseau sont des groupes d'adresses IP destinées aux services de contrôle d'intégrité et de gestion. Ces groupes permettent de réduire la complexité de la création de règles de sécurité. Les [étiquettes de service](../virtual-network/network-security-groups-overview.md#service-tags) autorisent le trafic entrant à partir d'adresses IP spécifiques, sans avoir à entrer chacune des [adresses IP de gestion](hdinsight-management-ip-addresses.md) dans vos groupes de sécurité réseau.

Le service HDInsight gère ces étiquettes de service. Vous ne pouvez pas créer votre propre étiquette de service ou modifier une étiquette existante. Microsoft gère les préfixes d'adresse correspondant à l'étiquette de service, et met automatiquement à jour l'étiquette de service lorsque les adresses changent.

Si vous souhaitez utiliser une région particulière et que l’étiquette de service n’est pas encore documentée dans cette page, vous pouvez utiliser l’[API Service Tag Discovery](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api-public-preview) pour rechercher votre étiquette de service. Vous pouvez également télécharger le [fichier JSON d’étiquette de service](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) et recherchez la région de votre choix.

## <a name="get-started-with-service-tags"></a>Prise en main des étiquettes de service

Vous avez le choix entre deux options pour utiliser les balises de service dans vos groupes de sécurité réseau :

- **Utiliser une étiquette de service HDInsight globale unique** : cette option ouvre votre réseau virtuel à toutes les adresses IP que le service HDInsight utilise pour analyser les clusters dans l'ensemble des régions. Il s'agit de la méthode la plus simple, mais elle peut ne pas vous convenir si vous avez des exigences de sécurité restrictives.

- **Utiliser plusieurs étiquettes de service régionales** : cette option ouvre exclusivement votre réseau virtuel aux adresses IP que HDInsight utilise dans cette région spécifique. Toutefois, si vous utilisez plusieurs régions, vous devez ajouter plusieurs étiquettes de service à votre réseau virtuel.

## <a name="use-a-single-global-hdinsight-service-tag"></a>Utiliser une balise de service HDInsight globale unique

Pour commencer à utiliser des étiquettes de service avec votre cluster HDInsight, le plus simple consiste à ajouter l'étiquette globale `HDInsight` à une règle de groupe de sécurité réseau.

1. Dans le [Portail Azure](https://portal.azure.com/), sélectionnez votre groupe de sécurité réseau.

1. Sous **Paramètres**, sélectionnez **Règles de sécurité de trafic entrant**, puis sélectionnez **+ Ajouter**.

1. Dans la liste déroulante **Source**, sélectionnez **Étiquette de service**.

1. Dans la liste déroulante **Étiquette du service source**, sélectionnez **HDInsight**.

    ![Ajouter une étiquette de service à partir du portail Azure](./media/hdinsight-service-tags/azure-portal-add-service-tag.png)

Cette étiquette contient les adresses IP des services de contrôle d'intégrité et de gestion de toutes les régions où HDInsight est disponible. L'étiquette garantit que votre cluster peut communiquer avec les services de contrôle d'intégrité et de gestion nécessaires, quel que soit l'emplacement où il est créé.

## <a name="use-regional-hdinsight-service-tags"></a>Utiliser les balises de service régional HDInsight

Si l'option d'étiquette globale ne fonctionne pas parce que vous avez besoin d'autorisations plus restrictives, vous pouvez autoriser uniquement les étiquettes de service applicables à votre région. Il peut y avoir plusieurs étiquettes de service selon la région dans laquelle votre cluster est créé.

Pour savoir quelles étiquettes de service ajouter à votre région, consultez les sections suivantes de l'article.

### <a name="use-a-single-regional-service-tag"></a>Utiliser une seule balise de service régional

Si votre cluster se trouve dans une région répertoriée dans ce tableau, il vous suffit d’ajouter une balise de service régional à votre groupe de sécurité réseau.

| Country | Région | Balise du service |
| ---- | ---- | ---- |
| Australie | Australie Est | HDInsight.AustraliaEast |
| &nbsp; | Sud-Australie Est | HDInsight.AustraliaSoutheast |
| &nbsp; | Centre de l’Australie | HDInsight.AustraliaCentral |
| Chine | Chine orientale 2 | HDInsight.ChinaEast2 |
| &nbsp; | Chine Nord 2 | HDInsight.ChinaNorth2 |
| États-Unis | Centre-Nord des États-Unis | HDInsight.NorthCentralUS |
| &nbsp; | USA Ouest 2 | HDInsight.WestUS2 |
| &nbsp; | Centre-USA Ouest | HDInsight.WestCentralUS |
| Canada | Est du Canada | HDInsight.CanadaEast |
| Brésil | Brésil Sud | HDInsight.BrazilSouth |
| Corée du Sud | Centre de la Corée | HDInsight.KoreaCentral |
| &nbsp; | Corée du Sud | HDInsight.KoreaSouth |
| Inde | Inde centrale | HDInsight.CentralIndia |
| &nbsp; | Inde Sud | HDInsight.SouthIndia |
| Japon | OuJapon Est | HDInsight.JapanWest |
| France | France Centre| HDInsight.FranceCentral |
| Royaume-Uni | Sud du Royaume-Uni | HDInsight.UKSouth |
| Azure Government | US DoD - Centre | HDInsight.USDoDCentral |
| &nbsp; | Gouvernement des États-Unis - Texas | HDInsight.USGovTexas |
| &nbsp; | US DoD - Est | HDInsight.USDoDEast |
| &nbsp; | Gouvernement des États-Unis - Arizona | HDInsight.USGovArizona |

### <a name="use-multiple-regional-service-tags"></a>Utiliser plusieurs balises de service régional

Si la région dans laquelle votre cluster a été créé ne figure pas dans le tableau précédent, vous devez autoriser plusieurs étiquettes de service régionales. La nécessité d’en utiliser plusieurs repose sur les différences de disposition des fournisseurs de ressources pour les différentes régions.

Les régions restantes sont divisées en groupes en fonction des balises de service régional qu’elles utilisent.

#### <a name="group-1"></a>Groupe 1

Si votre cluster est créé dans une des régions du tableau suivant, autorisez les étiquettes de service `HDInsight.WestUS` et `HDInsight.EastUS`. En outre, la balise de service régionale est répertoriée. Les régions de cette section nécessitent trois balises de service.

Par exemple, si votre cluster est créé dans la région `East US 2`, vous devez ajouter les étiquettes de service suivantes à votre groupe de sécurité réseau :

- `HDInsight.EastUS2`
- `HDInsight.WestUS`
- `HDInsight.EastUS`

| Country | Région | Balise du service |
| ---- | ---- | ---- |
| États-Unis | USA Est 2 | HDInsight.EastUS2 |
| &nbsp; | USA Centre | HDInsight.CentralUS |
| &nbsp; | NorthCentral US | HDInsight. NorthCentralUS |
| &nbsp; | États-Unis - partie centrale méridionale | HDInsight.SouthCentralUS |
| &nbsp; | USA Est | HDInsight.EastUS |
| &nbsp; | USA Ouest | HDInsight.WestUS |
| Japon | Japon Est | HDInsight.JapanEast |
| Europe | Europe Nord | HDInsight.NorthEurope |
| &nbsp; | Europe Ouest| HDInsight.WestEurope |
| Asia | Asie Est | HDInsight.EastAsia |
| &nbsp; | Asie Sud-Est | HDInsight.SoutheastAsia |
| Australie | Australie Est | HDInsight.AustraliaEast |

#### <a name="group-2"></a>Groupe 2

Les clusters des régions *Chine Nord* et *Chine Est* doivent autoriser deux étiquettes de service : `HDInsight.ChinaNorth` et `HDInsight.ChinaEast`.

#### <a name="group-3"></a>Groupe 3

Les clusters des régions *US Gov Iowa* et *US Gov Virginie* doivent autoriser deux étiquettes de service : `HDInsight.USGovIowa` et `HDInsight.USGovVirginia`.

#### <a name="group-4"></a>Groupe 4

Les clusters des régions *Allemagne Centre* et *Allemagne Nord-Est* doivent autoriser deux étiquettes de service : `HDInsight.GermanyCentral` et `HDInsight.GermanyNortheast`.

## <a name="next-steps"></a>Étapes suivantes

- [Groupes de sécurité réseau - Étiquettes de service](../virtual-network/network-security-groups-overview.md#security-rules)
- [Créer des réseaux virtuels pour les clusters Azure HDInsight](hdinsight-create-virtual-network.md)