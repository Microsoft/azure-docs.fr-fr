---
title: Adresses IP de gestion Azure HDInsight
description: Découvrez les adresses IP à partir desquelles vous devez autoriser le trafic entrant afin de configurer correctement les groupes de sécurité réseau et routes définies par l’utilisateur pour la mise en réseau virtuelle avec Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 08/11/2020
ms.openlocfilehash: 9fa38e045bbe29e5d45587adf0d277c1414fee4c
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96549036"
---
# <a name="hdinsight-management-ip-addresses"></a>Adresses IP de gestion HDInsight

Cet article répertorie les adresses IP utilisées par les services d’intégrité et de gestion Azure HDInsight. Si vous utilisez des groupes de sécurité réseau (NSG) ou des itinéraires définis par l’utilisateur (UDR), vous devrez peut-être ajouter une partie de ces adresses IP à la liste d’autorisation pour le trafic réseau entrant.

## <a name="introduction"></a>Introduction
 
> [!Important]
> Dans la plupart des cas, vous pouvez désormais utiliser des [balises de service](hdinsight-service-tags.md) pour les groupes de sécurité réseau au lieu d’ajouter manuellement des adresses IP. Les adresses IP ne seront pas publiées pour les nouvelles régions Azure, seules des balises de service seront publiées. Les adresses IP statiques pour les adresses IP de gestion seront finalement dépréciées.

Si vous utilisez des groupes de sécurité réseau ou des routes définies par l’utilisateur pour contrôler le trafic vers votre cluster HDInsight, vérifiez que votre cluster peut communiquer avec les services de gestion et d’intégrité Azure.  Certaines adresses IP sont propres à une région, et d’autres s’appliquent à toutes les régions Azure. Vous devrez peut-être autoriser le trafic provenant du service Azure DNS si vous n’utilisez pas de DNS personnalisé.

Si vous avez besoin d’adresses IP pour une région qui n’est pas mentionnée ici, vous pouvez utiliser l’[API Service Tag Discovery](../virtual-network/service-tags-overview.md#use-the-service-tag-discovery-api-public-preview) pour rechercher des adresses IP pour votre région. Si vous ne parvenez pas à utiliser l’API, téléchargez le [fichier JSON de balises de service](../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files) et recherchez la région de votre choix.

HDInsight effectue une validation pour ces règles avec la création et la mise à l’échelle du cluster afin d’éviter d’autres erreurs. Si la validation ne réussit pas, la création et la mise à l’échelle échouent.

Les sections suivantes traitent des adresses IP spécifiques qui doivent être autorisées.

## <a name="azure-dns-service"></a>Service Azure DNS

Si vous utilisez le service DNS fourni par Azure, autorisez l’accès à l’adresse __168.63.129.16__ sur le port 53 à la fois pour TCP et UDP. Pour plus d’informations, voir le document [Résolution de noms pour les machines virtuelles et les instances de rôle](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md). Si vous utilisez un service DNS personnalisé, ignorez cette étape.

## <a name="health-and-management-services-all-regions"></a>Services de gestion et d'intégrité : Toutes les régions

Autorisez le trafic provenant des adresses IP suivantes pour les services de gestion et d’intégrité Azure HDInsight, qui s’appliquent à toutes les régions Azure :

| Adresse IP source | Destination  | Sens |
| ---- | ----- | ----- |
| 168.61.49.99 | \*:443 | Trafic entrant |
| 23.99.5.239 | \*:443 | Trafic entrant |
| 168.61.48.131 | \*:443 | Trafic entrant |
| 138.91.141.162 | \*:443 | Trafic entrant |

## <a name="health-and-management-services-specific-regions"></a>Services de gestion et d'intégrité : Régions spécifiques

Autorisez le trafic provenant des adresses IP répertoriées pour les services de gestion et d’intégrité Azure HDInsight dans la région Azure spécifique où vos ressources sont situées :

> [!IMPORTANT]  
> Si la région Azure que vous utilisez n’est pas répertoriée, utilisez la fonctionnalité [balise de service](hdinsight-service-tags.md) pour les groupes de sécurité réseau.

| Country | Région | Adresses IP sources autorisées | Destination autorisée | Sens |
| ---- | ---- | ---- | ---- | ----- |
| Asia | Asie Est | 23.102.235.122</br>52.175.38.134 | \*:443 | Trafic entrant |
| &nbsp; | Asie Sud-Est | 13.76.245.160</br>13.76.136.249 | \*:443 | Trafic entrant |
| Australie | Australie Est | 104.210.84.115</br>13.75.152.195 | \*:443 | Trafic entrant |
| &nbsp; | Sud-Australie Est | 13.77.2.56</br>13.77.2.94 | \*:443 | Trafic entrant |
| Brésil | Brésil Sud | 191.235.84.104</br>191.235.87.113 | \*:443 | Trafic entrant |
| Canada | Est du Canada | 52.229.127.96</br>52.229.123.172 | \*:443 | Trafic entrant |
| &nbsp; | Centre du Canada | 52.228.37.66</br>52.228.45.222 |\*: 443 | Trafic entrant |
| Chine | Chine du Nord | 42.159.96.170</br>139.217.2.219</br></br>42.159.198.178</br>42.159.234.157 | \*:443 | Trafic entrant |
| &nbsp; | Chine orientale | 42.159.198.178</br>42.159.234.157</br></br>42.159.96.170</br>139.217.2.219 | \*:443 | Trafic entrant |
| &nbsp; | Chine Nord 2 | 40.73.37.141</br>40.73.38.172 | \*:443 | Trafic entrant |
| &nbsp; | Chine orientale 2 | 139.217.227.106</br>139.217.228.187 | \*:443 | Trafic entrant |
| Europe | Europe Nord | 52.164.210.96</br>13.74.153.132 | \*:443 | Trafic entrant |
| &nbsp; | Europe Ouest| 52.166.243.90</br>52.174.36.244 | \*:443 | Trafic entrant |
| France | France Centre| 20.188.39.64</br>40.89.157.135 | \*:443 | Trafic entrant |
| Allemagne | Centre de l’Allemagne | 51.4.146.68</br>51.4.146.80 | \*:443 | Trafic entrant |
| &nbsp; | Nord-Est de l’Allemagne | 51.5.150.132</br>51.5.144.101 | \*:443 | Trafic entrant |
| Inde | Inde centrale | 52.172.153.209</br>52.172.152.49 | \*:443 | Trafic entrant |
| &nbsp; | Inde Sud | 104.211.223.67<br/>104.211.216.210 | \*:443 | Trafic entrant |
| Japon | Japon Est | 13.78.125.90</br>13.78.89.60 | \*:443 | Trafic entrant |
| &nbsp; | OuJapon Est | 40.74.125.69</br>138.91.29.150 | \*:443 | Trafic entrant |
| Corée du Sud | Centre de la Corée | 52.231.39.142</br>52.231.36.209 | \*:443 | Trafic entrant |
| &nbsp; | Corée du Sud | 52.231.203.16</br>52.231.205.214 | \*:443 | Trafic entrant
| Royaume-Uni | Ouest du Royaume-Uni | 51.141.13.110</br>51.141.7.20 | \*:443 | Trafic entrant |
| &nbsp; | Sud du Royaume-Uni | 51.140.47.39</br>51.140.52.16 | \*:443 | Trafic entrant |
| États-Unis | USA Centre | 13.89.171.122</br>13.89.171.124 | \*:443 | Trafic entrant |
| &nbsp; | USA Est | 13.82.225.233</br>40.71.175.99 | \*:443 | Trafic entrant |
| &nbsp; | Centre-Nord des États-Unis | 157.56.8.38</br>157.55.213.99 | \*:443 | Trafic entrant |
| &nbsp; | Centre-USA Ouest | 52.161.23.15</br>52.161.10.167 | \*:443 | Trafic entrant |
| &nbsp; | USA Ouest | 13.64.254.98</br>23.101.196.19 | \*:443 | Trafic entrant |
| &nbsp; | USA Ouest 2 | 52.175.211.210</br>52.175.222.222 | \*:443 | Trafic entrant |
| &nbsp; | Émirats arabes unis Nord | 65.52.252.96</br>65.52.252.97 | \*:443 | Trafic entrant |
| &nbsp; | Émirats arabes unis Centre | 20.37.76.96</br>20.37.76.99 | \*:443 | Trafic entrant |

Pour plus d’informations sur les adresses IP à utiliser pour Azure Government, voir le document [Intelligence et analyse Azure Government](../azure-government/compare-azure-government-global-azure.md).

Pour plus d’informations, consultez [Contrôler le trafic réseau](./control-network-traffic.md).

Si vous utilisez des routes définies par l’utilisateur, vous devez spécifier une route et autoriser le trafic sortant du réseau virtuel vers les adresses IP ci-dessus avec le tronçon suivant défini sur « Internet ».

## <a name="next-steps"></a>Étapes suivantes

* [Créer des réseaux virtuels pour les clusters Azure HDInsight](hdinsight-create-virtual-network.md)
* [Étiquettes de service de groupe de sécurité réseau (NSG) pour Azure HDInsight](hdinsight-service-tags.md)
