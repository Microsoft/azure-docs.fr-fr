---
title: Application Gateway v2 avec mise à l’échelle automatique et redondance interzone
description: Cet article présente la référence SKU Azure Application Standard_v2 et WAF_v2, qui inclut des fonctionnalités de mise à l’échelle automatique et de redondance de zone.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 06/06/2020
ms.author: victorh
ms.custom: fasttrack-edit, references_regions
ms.openlocfilehash: 8c989e426faa77025a84515fe0a19424cb6bfa89
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91826636"
---
# <a name="autoscaling-and-zone-redundant-application-gateway-v2"></a>Application Gateway v2 avec mise à l’échelle automatique et redondance interzone 

Application Gateway est disponible sous une référence SKU Standard_v2. Le pare-feu d’applications web (WAF) est disponible sous une référence WAF_v2. La référence SKU v2 offre des performances améliorées et prend en charge de nouvelles fonctionnalités critiques telles que la mise à l’échelle automatique, la redondance de zone et la prise en charge d'adresses IP virtuelles statiques. Les fonctionnalités existantes sous la référence SKU Standard et WAF continuent à être prises en charge dans la nouvelle référence SKU v2, à quelques exceptions près que vous trouverez dans la section relative aux [comparaisons](#differences-from-v1-sku).

La nouvelle référence SKU v2 inclut les améliorations suivantes :

- **Mise à l’échelle automatique** : les déploiements d’Application Gateway ou du WAF sous la référence SKU de la mise à l’échelle automatique peuvent augmenter ou diminuer en fonction de l’évolution des modèles de charge du trafic. La mise à l’échelle automatique vous évite aussi d’avoir à choisir une taille de déploiement ou un nombre d’instances au moment du provisionnement. Cette référence SKU offre une véritable élasticité. Dans la référence SKU Standard_v2 et WAF_v2, Application Gateway peut fonctionner à la fois en mode de capacité fixe (mise à l’échelle automatique désactivée) et de mise à l’échelle automatique. Le mode de capacité fixe est utile pour les scénarios avec des charges de travail cohérentes et prévisibles. Le mode de mise à l’échelle automatique est utile dans les applications qui connaissent un trafic variable.
- **Redondance de zone** : un déploiement d’Application Gateway ou de WAF peut désormais couvrir plusieurs zones de disponibilité, ce qui évite d’avoir à approvisionner plusieurs instances d’Application Gateway dans chaque zone à l’aide d’une instance Traffic Manager. Vous pouvez choisir une ou plusieurs zones où les instances d’Application Gateway sont déployées, ce qui renforce la résilience en cas d’échec de la zone. Le pool principal pour les applications peut être distribué de la même façon entre les différentes zones de disponibilité.

  La redondance de zone est uniquement disponible dans les régions où les zones Azure sont disponibles. Dans les autres régions, toutes les autres fonctionnalités sont prises en charge. Pour plus d’informations, consultez [Régions et zones de disponibilité dans Azure](../availability-zones/az-overview.md).
- **Adresse IP virtuelle statique** : la référence SKU Application Gateway v2 prend exclusivement en charge le type d’adresse IP virtuelle statique. Ainsi, l’adresse IP virtuelle associée à Application Gateway ne change pas pendant toute la durée de vie du déploiement, même après un redémarrage.  Il n'existe pas d'adresse IP virtuelle statique dans v1, et vous devez donc utiliser l’URL Application Gateway plutôt que l’adresse IP pour router le nom de domaine vers App Services via Application Gateway.
- **Réécriture d’en-tête** : Application Gateway vous permet d'ajouter, de supprimer ou de mettre à jour les en-têtes de requête et de réponse HTTP avec la référence SKU v2. Pour plus d’informations, consultez [Réécrire des en-têtes HTTP avec Application Gateway](rewrite-http-headers.md).
- **Intégration dans Key Vault** : Application Gateway v2 prend en charge l'intégration dans Key Vault des certificats de serveur associés à des écouteurs HTTPS. Pour plus d'informations, consultez [Arrêt de TLS avec des certificats Key Vault](key-vault-certs.md).
- **Contrôleur d'entrée Azure Kubernetes Service** : Le contrôleur d’entrée Application Gateway v2 permet l'utilisation d'Azure Application Gateway en tant qu'entrée pour un service Azure Kubernetes Service (AKS) appelé Cluster AKS. Pour plus d'informations, consultez [Présentation des contrôleurs d'entrée Application Gateway](ingress-controller-overview.md).
- **Amélioration des performances** : la référence SKU v2 offre jusqu’à 5 fois plus de déchargement TLS que la référence SKU Standard/WAF.
- **Déploiement et mise à jour plus rapides** La référence SKU v2 permet un déploiement et une mise à jour plus rapides que la référence SKU Standard/WAF. Elle inclut également des modifications de configuration WAF.

![Diagramme de la zone de mise à l’échelle automatique.](./media/application-gateway-autoscaling-zone-redundant/application-gateway-autoscaling-zone-redundant.png)

## <a name="supported-regions"></a>Régions prises en charge

La référence SKU Standard_v2 et WAF_v2 est disponible dans les régions suivantes : USA Centre Nord, USA Centre Sud, USA Ouest, USA Ouest 2, USA Est, USA Est 2, USA Centre, Europe Nord, Europe Ouest, Asie Sud-Est, France Centre, Royaume-Uni Ouest, Japon Est, Japon Ouest, Australie Est, Australie Sud-Est, Brésil Sud, Canada Centre, Canada Est, Asie Est, Corée Centre, Corée Sud, Royaume-Uni Sud, Inde Centre, Inde Ouest, Inde Sud.

## <a name="pricing"></a>Tarifs

Avec la référence SKU v2, le modèle de tarification est basé sur la consommation et n’est plus lié au nombre ou à la taille des instances. La tarification de la référence SKU v2 inclut deux composants :

- **Prix fixe** : correspond au prix horaire (ou heure entamée) pour approvisionner une passerelle Standard_v2 ou WAF_v2. Notez que l’option 0 instance minimale supplémentaire garantit quand même une haute disponibilité du service, ce qui est toujours inclus avec le prix fixe.
- **Prix des unités de capacité** : correspond à un coût basé sur la consommation qui est facturé en plus du coût fixe. Les frais liés aux unités de capacité sont également calculés à l’heure ou partiellement à l’heure. Les unités de capacité incluent 3 dimensions : l’unité Compute, les connexions persistantes et le débit. L’unité Compute est une mesure de la capacité consommée du processeur. Les facteurs affectant l’unité Compute sont les connexions TLS/s, les calculs de réécriture d’URL et le traitement des règles WAF. Une connexion permanente est une mesure de connexions TCP établies vers la passerelle d’application à un intervalle de facturation donné. Le débit correspond à la moyenne des mégabits/s traités par le système à un intervalle de facturation donné.  La facturation est effectuée à un niveau d’unité de capacité pour tout ce qui se trouve au-dessus du nombre d’instances réservées.

Chaque unité de capacité est composée au maximum de ce qui suit : 1 unité Compute, 2 500 connexions permanentes et 2,22 Mbits/s de débit.

Pour plus d’informations, consultez [Compréhension de la tarification](understanding-pricing.md).

## <a name="scaling-application-gateway-and-waf-v2"></a>Mise à l’échelle d'Application Gateway et de WAF v2

Application Gateway et WAF peuvent être configurés pour être mis à l'échelle dans deux modes :

- **Mise à l’échelle automatique** - Lorsque la mise à l’échelle automatique est activée, les références SKU Application Gateway et WAF v2 sont mis à l'échelle (augmentation ou réduction) en fonction des besoins de trafic de l'application. Ce mode offre une meilleure élasticité à votre application et vous évite de devoir estimer la taille d'Application Gateway ou le nombre d'instances. En outre, ce mode vous évite de devoir exécuter des passerelles à la capacité maximale approvisionnée pour une charge de trafic maximale anticipée. Vous devez spécifier un nombre minimal d’instances et éventuellement un nombre maximal d'instances. Moyennant une capacité minimale, Application Gateway et WAF v2 ne descendent pas en dessous du nombre minimal d'instances spécifié, même en l’absence de trafic. Chaque instance est à peu près équivalente à 10 unités de capacité de réserve supplémentaires. Zéro signifie aucune capacité réservée, avec une mise à l’échelle purement automatique par nature. Vous pouvez également spécifier un nombre maximal d’instances pour vous assurer qu'Application Gateway n'effectue pas de mise à l'échelle au-delà du nombre d'instances spécifié. Vous ne serez facturé que pour la quantité de trafic desservie par l passerelle. Le nombre d’instances peut varier de 0 à 125. Si elle n'est pas spécifiée, la valeur par défaut correspondant au nombre maximal d'instances est de 20.
- **Manuel** - Vous pouvez aussi choisir le mode Manuel, sans mise à l’échelle automatique de la passerelle. Dans ce mode, un trafic supérieur à ce que Application Gateway ou WAF peut gérer est susceptible d'entraîner une perte de trafic. En mode Manuel, vous êtes tenu de spécifier un nombre d'instances. Le nombre d’instances peut varier de 1 à 125.

## <a name="autoscaling-and-high-availability"></a>Mise à l’échelle automatique et haute disponibilité

Les passerelles Azure Application sont toujours déployées de façon hautement disponible. Le service est constitué de plusieurs instances créées conformément à la configuration (si la mise à l’échelle automatique est désactivée) ou requises par la charge de l’application (si la mise à l’échelle automatique est activée). Notez que, du point de vue de l’utilisateur, vous n’avez pas nécessairement de visibilité des instances individuelles, mais uniquement du service Application Gateway dans son ensemble. Si une instance présente un problème et cesse d’être fonctionnelle, Azure Application Gateway crée sans difficulté une nouvelle instance.

Notez que, même si vous configurez la mise à l’échelle automatique sans minimum d’instances, le service offrira toujours une haute disponibilité qui est toujours incluse dans le prix fixe.

Cependant, la création d’une instance peut prendre un certain temps (six ou sept minutes). Par conséquent, si vous ne souhaitez pas composer avec ce temps d’arrêt, vous pouvez configurer un nombre minimal d’instances égal à 2, idéalement avec prise en charge de zone de disponibilité. De cette façon, dans des conditions normales, vous avez au moins deux instances à l’intérieur de votre Application Gateway, de sorte que, si l’une d’elles rencontre un problème, l’autre tente de s’accommoder du trafic le temps de créer une nouvelle instance. Notez qu’une instance Azure Application Gateway peut prendre en charge environ 10 unités de capacité. Par conséquent, selon le volume du trafic en général, vous pouvez configurer le paramètre de mise à l’échelle automatique minimale d’instance sur une valeur supérieure à 2.

## <a name="feature-comparison-between-v1-sku-and-v2-sku"></a>Comparaison des fonctionnalités des références SKU v1 et v2

Le tableau suivant répertorie les fonctionnalités disponibles avec chaque référence SKU.

| Fonctionnalité                                           | Référence SKU v1   | Référence SKU v2   |
| ------------------------------------------------- | -------- | -------- |
| Mise à l’échelle automatique                                       |          | &#x2713; |
| Redondance de zone                                   |          | &#x2713; |
| Adresse IP virtuelle statique                                        |          | &#x2713; |
| Contrôleur d’entrée Azure Kubernetes Service (AKS) |          | &#x2713; |
| Intégration du coffre de clés Azure                       |          | &#x2713; |
| Réécrire les en-têtes HTTP(S)                           |          | &#x2713; |
| Routage basé sur des URL                                 | &#x2713; | &#x2713; |
| Hébergement de plusieurs sites                             | &#x2713; | &#x2713; |
| Redirection du trafic                               | &#x2713; | &#x2713; |
| Pare-feu d’applications web (WAF)                    | &#x2713; | &#x2713; |
| Règles personnalisées WAF                                  |          | &#x2713; |
| Arrêt de TLS (Transport Layer Security)/SSL (Secure Sockets Layer)            | &#x2713; | &#x2713; |
| Chiffrement TSL de bout en bout                         | &#x2713; | &#x2713; |
| Affinité de session                                  | &#x2713; | &#x2713; |
| Pages d’erreur personnalisées                                | &#x2713; | &#x2713; |
| Prise en charge de WebSocket                                 | &#x2713; | &#x2713; |
| Assistance HTTP/2                                    | &#x2713; | &#x2713; |
| Vidage des connexions                               | &#x2713; | &#x2713; |

> [!NOTE]
> La référence SKU v2 avec mise à l’échelle automatique prend désormais en charge les [sondes d’intégrité par défaut](application-gateway-probe-overview.md#default-health-probe) afin de superviser automatiquement l’intégrité de toutes les ressources dans son pool principal et de mettre en évidence les membres principaux considérés comme non sains. La sonde d’intégrité par défaut est automatiquement configurée pour les serveurs principaux ne disposant d'aucune configuration de sonde personnalisée. Pour plus d’informations, consultez [Sondes d’intégrité dans Application Gateway](application-gateway-probe-overview.md).

## <a name="differences-from-v1-sku"></a>Différences par rapport à la référence (SKU) v1

Cette section décrit les fonctionnalités et les limitations de la référence (SKU) v2 qui diffèrent de celles de la référence (SKU) v1.

|Différence|Détails|
|--|--|
|Certificat d’authentification|Non pris en charge.<br>Pour plus d’informations, consultez [Présentation du chiffrement TLS de bout en bout sur la passerelle Application Gateway](ssl-overview.md#end-to-end-tls-with-the-v2-sku).|
|Combinaison de Standard_v2 et Standard Application Gateway sur le même sous-réseau|Non pris en charge|
|Itinéraire défini par l’utilisateur sur le sous-réseau d’Application Gateway|Pris en charge (scénarios spécifiques). En préversion.<br> Pour plus d’informations sur les scénarios pris en charge, voir [Présentation de la configuration d’Application Gateway](configuration-infrastructure.md#supported-user-defined-routes).|
|Groupe de sécurité réseau pour plage de ports entrants| - 65 200 à 65 535 pour référence (SKU) Standard_v2<br>- 65 503 à 65 534 pour référence (SKU) Standard<br>Pour plus d’informations, visitez le [FAQ](application-gateway-faq.md#are-network-security-groups-supported-on-the-application-gateway-subnet).|
|Journaux d’activité de performances dans les diagnostics Azure|Non pris en charge.<br>Les métriques Azure doivent être utilisées.|
|Facturation|Facturation planifiée pour le 1er juillet 2019.|
|Mode FIPS|Ils ne sont pas pris en charge.|
|Mode ILB uniquement|Non pris en charge actuellement. Les modes public et ILB sont pris en charge ensemble.|
|Intégration de l’observateur réseau|Non pris en charge.|
|Intégration Azure Security Center|Pas encore disponible.

## <a name="migrate-from-v1-to-v2"></a>Effectuer la migration de la version 1 à la version 2

Un script Azure PowerShell est disponible dans la galerie PowerShell pour vous aider à migrer de v1 Application Gateway/WAF vers la référence SKU de mise à l'échelle automatique v2. Ce script vous permet de copier la configuration à partir de votre passerelle v1. La migration de trafic relève toujours de votre responsabilité. Pour plus d’informations, consultez [Migrer Azure Application Gateway de la version v1 vers la version v2](migrate-v1-v2.md).

## <a name="next-steps"></a>Étapes suivantes

- [Démarrage rapide : diriger le trafic web avec Azure Application Gateway - Portail Azure](quick-create-portal.md)
- [Créer une passerelle d’application redondante dans une zone, avec mise à l’échelle automatique et avec une adresse IP virtuelle réservée à l’aide d’Azure PowerShell](tutorial-autoscale-ps.md)
- Découvrez [Application Gateway](overview.md).
- Découvrez le [Pare-feu Azure](../firewall/overview.md).
