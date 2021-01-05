---
title: Visualisation des journaux de flux de groupe de sécurité réseau Azure - Power BI
titleSuffix: Azure Network Watcher
description: Découvrez comment utiliser Power BI pour visualiser des journaux de flux de groupe de sécurité réseau pour vous permettre d’afficher des informations sur le trafic IP dans Azure Network Watcher.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: a4c97b8baaf5265f984622e2dc47e6c8c714172a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995724"
---
# <a name="visualizing-network-security-group-flow-logs-with-power-bi"></a>Visualisation des journaux de flux des groupes de sécurité réseau Azure avec Power BI

Les journaux de flux des groupes de sécurité réseau (NSG) vous permettent d’afficher des informations relatives au trafic IP entrant et sortant sur les groupes de sécurité réseau. Ces journaux de flux affichent les flux entrants et sortants en fonction de la règle, de la carte réseau à laquelle le flux s’applique, des informations à 5 tuples sur le flux (adresse IP source/de destination, port source/de destination, protocole), et de l’autorisation ou du refus du trafic.

Il peut être difficile de comprendre les données de journalisation des flux en effectuant une simple recherche manuelle dans les fichiers journaux. Dans cet article, nous vous proposons une solution permettant de visualiser vos flux de journaux d’activité les plus récents et d’en savoir plus sur le trafic de votre réseau.

> [!Warning]  
> Les étapes suivantes fonctionnent avec les journaux de flux version 1. Pour plus d’informations, consultez [Présentation de la journalisation des flux pour les groupes de sécurité réseau](network-watcher-nsg-flow-logging-overview.md). Les instructions suivantes ne fonctionnent pas avec la version 2 des fichiers journaux, sans modification.

## <a name="scenario"></a>Scénario

Dans le scénario suivant, nous connectons Power BI Desktop au compte de stockage que nous avons configuré comme récepteur pour nos données de consignation de flux NSG. Une fois connecté à notre compte de stockage, Power BI télécharge et analyse les journaux d’activité pour fournir une représentation visuelle du trafic consigné par les groupes de sécurité réseau.

Grâce aux représentations visuelles fournies dans le modèle, vous pouvez examiner les informations suivantes :

* Principaux consommateurs
* Données de flux chronologiques par direction et par règle de décision
* Flux par adresse MAC d’interface réseau
* Flux par groupe de sécurité réseau et par règle
* Flux par port de destination

Le modèle fourni est modifiable afin de vous permettre d’ajouter de nouvelles données ou de nouveaux graphiques, ou encore d’adapter vos requêtes à vos besoins.

## <a name="setup"></a>Programme d’installation

Avant de commencer, la journalisation des flux de groupe de sécurité réseau doit être activée sur un ou plusieurs groupes de sécurité réseau de votre compte. Pour obtenir des instructions sur l’activation des journaux de flux de sécurité réseau, consultez l’article suivant [Introduction to flow logging for Network Security Groups](network-watcher-nsg-flow-logging-overview.md) (Introduction à la journalisation des flux pour les groupes de sécurité réseau).

Le client Power BI Desktop doit également être installé sur votre ordinateur, lequel doit posséder suffisamment d’espace libre pour pouvoir télécharger et charger les données de journaux existant dans votre compte de stockage.

![Diagramme Visio][1]

### <a name="steps"></a>Étapes

1. Téléchargez et ouvrez le modèle Power BI suivant dans l’application Power BI Desktop : [Modèle de journaux de flux Network Watcher PowerBI](https://aka.ms/networkwatcherpowerbiflowlogstemplate)
1. Entrez les paramètres de requête obligatoires
   1. **StorageAccountName** : spécifie le nom du compte de stockage contenant les journaux de flux NSG que vous souhaitez charger et visualiser.
   1. **NumberOfLogFiles** : spécifie le nombre de fichiers journaux que vous souhaitez télécharger et visualiser dans Power BI. Par exemple, si vous spécifiez 50, il s’agit des 50 derniers fichiers journaux. Si vous avez activé 2 groupes de sécurité réseau et que vous les avez configurés pour envoyer des journaux de flux NSG sur ce compte, vous pourrez visualiser les 25 dernières heures de journaux d’activité.

      ![page d’accueil Power BI][2]

1. Entrez la clé d'accès de votre compte de stockage. Pour trouver les clés d’accès valides, accédez à votre compte de stockage dans le portail Azure et sélectionnez **Clés d’accès** dans le menu Paramètres. Cliquez sur **Connecter**, puis appliquez les modifications.

    ![clés d'accès][3]

    ![clé d’accès 2][4]

4. Vos journaux d’activité sont téléchargés et analysés. Vous pouvez maintenant utiliser les éléments visuels créés au préalable.

## <a name="understanding-the-visuals"></a>Vue d’ensemble des éléments visuels

Le modèle contient un ensemble d’éléments visuels qui permettent de mieux comprendre les données des journaux de flux NSG. Les illustrations suivantes montrent un exemple du tableau de bord complété par des données. Nous allons examiner ci-dessous en détail chaque représentation visuelle 

![powerbi][5]
 
Le graphique Top Talkers (Principaux consommateurs) présente les adresses IP qui ont initié le plus de connexions sur la période spécifiée. La taille des zones correspond au nombre relatif de connexions. 

![toptalkers][6]

Les graphiques chronologiques suivants indiquent le nombre de flux sur la période. Le graphique supérieur est segmenté en fonction de la direction du flux, et le graphique inférieur est segmenté en fonction de la décision prise (autorisation ou refus). Avec cet élément visuel, vous pouvez examiner les tendances du trafic au fil du temps et repérer les pics ou baisses d’activité anormaux au niveau du trafic ou de la segmentation du trafic.

![flowsoverperiod][7]

Les graphiques suivants décrivent les flux par interface réseau, le graphique supérieur étant segmenté en fonction de la direction du flux et le graphique inférieur en fonction de la décision prise. Avec ces informations, vous pouvez identifier parmi vos machines virtuelles celles qui ont le plus communiqué par rapport aux autres et déterminer si le trafic vers une machine virtuelle spécifique est autorisé ou refusé.

![flowspernic][8]

Le graphique circulaire ci-dessous montre une répartition des flux par port de destination. Avec ces informations, vous pouvez afficher les ports de destination les plus fréquemment utilisés pendant la période spécifiée.

![donut][9]

Le diagramme ci-dessous illustre le flux par groupe de sécurité réseau et par règle. Ces informations vous permettent d’identifier les groupes de sécurité associés au trafic le plus important et de voir la répartition du trafic sur un groupe de sécurité réseau par règle.

![barchart][10]
 
Les graphiques d’informations suivants affichent des informations sur les groupes de sécurité réseau présents dans les journaux d’activité, le nombre de flux capturés au cours de la période et la date du premier journal d’activité capturé. Ces informations vous donnent une idée des groupes de sécurité réseau consignés ainsi que de la plage de dates des flux.

![infochart1][11]

![infochart2][12]

Ce modèle inclut les segments suivants afin que vous puissiez afficher uniquement les données qui vous intéressent le plus. Vous pouvez appliquer un filtre sur vos groupes de ressources, sur les groupes de sécurité réseau et sur les règles. Vous pouvez également filtrer sur les informations à 5 tuples, sur la décision et sur l’heure à laquelle des données ont été écrites dans le journal.

![slicers][13]

## <a name="conclusion"></a>Conclusion

Dans ce scénario, nous vous avons montré comment visualiser et comprendre le trafic à l’aide des journaux de flux des groupes de sécurité réseau fournis par Network Watcher et par Power BI. À l’aide du modèle fourni, Power BI télécharge les journaux d’activité directement à partir du stockage et les traite en local. Le temps nécessaire au chargement du modèle varie en fonction du nombre de fichiers requis et de la taille totale des fichiers téléchargés.

N’hésitez pas à personnaliser ce modèle selon vos besoins. Il existe de nombreuses manières d’utiliser Power BI avec les journaux de flux des groupes de sécurité réseau. 

## <a name="notes"></a>Notes

* Par défaut, les journaux d’activité sont stockés dans `https://{storageAccountName}.blob.core.windows.net/insights-logs-networksecuritygroupflowevent/`

    * S’il existe d’autres données dans un autre répertoire, vous devez modifier les requêtes permettant d’extraire et traiter les données.

* Le modèle fourni n’est pas recommandé pour des journaux d’activité d’une taille supérieure à 1 Go.

* Si vous disposez d’une grande quantité de journaux d’activité, nous vous recommandons de rechercher une solution à l’aide d’un autre magasin de données tel que Data Lake ou SQL Server.

## <a name="next-steps"></a>Étapes suivantes

Consultez l’article [Visualiser les journaux de flux NSG d’Azure Network Watcher à l’aide d’outils open source](network-watcher-visualize-nsg-flow-logs-open-source-tools.md) pour apprendre à visualiser vos journaux de flux NSG avec Elastic Stack.

[1]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure1.png
[2]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure2.png
[3]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure3.png
[4]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure4.png
[5]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure5.png
[6]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure6.png
[7]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure7.png
[8]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure8.png
[9]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure9.png
[10]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure10.png
[11]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure11.png
[12]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure12.png
[13]: ./media/network-watcher-visualize-nsg-flow-logs-power-bi/figure13.png
