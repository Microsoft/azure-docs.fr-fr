---
title: Diagnostics avec les métriques, les alertes et l’intégrité des ressources – Azure Standard Load Balancer
description: Utilisez les métriques, les alertes et les informations d’intégrité des ressources disponibles pour diagnostiquer Azure Standard Load Balancer.
services: load-balancer
documentationcenter: na
author: asudbring
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/14/2019
ms.author: allensu
ms.openlocfilehash: 9c322620e1d66182937be41bb02d48fd1469f459
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94697558"
---
# <a name="standard-load-balancer-diagnostics-with-metrics-alerts-and-resource-health"></a>Diagnostics Azure Standard Load Balancer avec les métriques, les alertes et l’intégrité des ressources

Azure Load Balancer Standard expose les fonctionnalités de diagnostic suivantes :

* **Métriques multidimensionnelles et alertes** : des fonctionnalités de diagnostic multidimensionnel sont proposées par le biais d’[Azure Monitor](../azure-monitor/overview.md) pour les configurations d’équilibreur standard. Vous pouvez surveiller, gérer et résoudre les problèmes de vos ressources d’équilibreur de charge standard.

* **Intégrité des ressources** : l’état d’intégrité des ressources de votre équilibreur de charge est disponible dans la page Resource Health sous Superviser. Cette vérification automatique vous informe de la disponibilité actuelle de votre ressource Load Balancer.

Cet article propose une présentation rapide de ces fonctionnalités et des méthodes pour les utiliser dans Load Balancer Standard. 

## <a name="multi-dimensional-metrics"></a><a name = "MultiDimensionalMetrics"></a>Métriques multidimensionnelles

Azure Load Balancer fournit des métriques multidimensionnelles par le biais des métriques Azure dans le portail Azure, et vous permet d’obtenir des informations de diagnostic en temps réel sur vos ressources d’équilibreur de charge. 

Les différentes configurations de Load Balancer Standard fournissent les métriques suivantes :

| Métrique | Type de ressource | Description | Agrégation recommandée |
| --- | --- | --- | --- |
| Disponibilité du chemin d’accès aux données | Équilibreur de charge interne et public | Load Balancer Standard teste en continu le chemin de données d’une région vers le serveur frontal de l’équilibreur de charge, jusqu’à la pile SDN qui prend en charge votre machine virtuelle. Tant que les instances saines restent, la mesure suit le même chemin que le trafic à charge équilibrée de vos applications. Le chemin de données utilisé par vos clients est également validé. La mesure est invisible pour votre application et n’interfère pas avec les autres opérations.| Average |
| État de la sonde d’intégrité | Équilibreur de charge interne et public | Load Balancer Standard utilise un service de détection d’intégrité distribué qui surveille l’intégrité du point de terminaison de votre application en fonction de vos paramètres de configuration. Cette métrique fournit un agrégat ou une vue filtrée par point de terminaison de chaque point de terminaison d’instance dans le pool de l’équilibreur de charge. Vous pouvez observer comment Load Balancer voit l’intégrité de votre application comme indiqué par votre configuration de sonde d’intégrité. |  Average |
| Paquets SYN (synchroniser) | Équilibreur de charge interne et public | Load Balancer Standard ne termine pas les connexions Transmission Control Protocol (TCP) et n’interagit pas avec les flux de paquets UDP ou TCP. Les flux et leurs établissements de liaisons sont toujours entre la source et l’instance de machine virtuelle. Pour mieux résoudre les problèmes posés par vos scénarios de protocole TCP, vous pouvez utiliser les compteurs de paquets SYN pour comprendre le nombre de tentatives de connexion TCP effectuées. La métrique indique le nombre de paquets SYN TCP reçus.| Average |
| Connexions SNAT | Équilibreur de charge public |Load Balancer Standard indique le nombre de flux sortants usurpés sur le serveur frontal d’adresse IP public. Les ports de traduction d'adresses réseau source (SNAT) constituent une ressource épuisable. Cette métrique peut donner une idée de l’importance du rôle joué par SNAT dans votre application pour les flux sortants. Les compteurs relatifs aux flux SNAT sortants réussis et mis en échec sont indiqués et peuvent être utilisés pour comprendre l’intégrité de vos flux sortants et résoudre les problèmes associés.| Average |
| Ports SNAT alloués | Équilibreur de charge public | Standard Load Balancer indique le nombre de ports SNAT alloués par instance de back-end | Moyenne. |
| Ports SNAT utilisés | Équilibreur de charge public | Standard Load Balancer indique le nombre de ports SNAT utilisés par instance de back-end. | Average | 
| Compteurs d’octets |  Équilibreur de charge interne et public | Load Balancer Standard indique les données traitées par serveur frontal. Vous pouvez remarquer que les octets ne sont pas répartis de manière égale entre les instances du serveur principal. Cela est normal, car l’algorithme d’Azure Load Balance est basé sur les flux | Average |
| Compteurs de paquets |  Équilibreur de charge interne et public | Load Balancer Standard indique les paquets traités par serveur frontal.| Average |

  >[!NOTE]
  >Lorsque vous utilisez la distribution du trafic à partir d’un équilibreur de charge interne via un appliance virtuelle réseau ou un pare-feu, les métriques de paquet Syn, de compteur d’octets et de compteur de paquets ne sont pas disponibles et affichent zéro. 
  
### <a name="view-your-load-balancer-metrics-in-the-azure-portal"></a>Afficher vos métriques d’équilibreur de charge dans le portail Azure

Le Portail Azure présente les métriques d’équilibreur de charge via la page Métriques, qui est disponible sur la page de ressource d’équilibreur de charge pour une ressource spécifique, ainsi que sur la page Azure Monitor. 

Pour afficher les métriques de vos ressources de Load Balancer Standard :
1. Accédez à la page Métriques et effectuez l’une des opérations suivantes :
   * Sur la page de ressource d’équilibreur de charge, sélectionnez le type de métrique dans la liste déroulante.
   * Sur la page Azure Monitor, sélectionnez la ressource d’équilibreur de charge.
2. Définissez le type d’agrégation de métriques qui convient.
3. Éventuellement, configurez le filtrage et le regroupement requis.
4. Vous pouvez également configurer l’intervalle de temps et l’agrégation. Par défaut, l’heure s'affiche au format UTC.

  >[!NOTE] 
  >L’agrégation de temps est importante lors de l’interprétation de certaines métriques, car les données sont échantillonnées toutes les minutes. Si l’agrégation de temps est définie sur cinq minutes et que le type d’agrégation de métriques Sum est utilisé pour les métriques telles que l’allocation SNAT, votre graphique affichera cinq fois le nombre total de ports SNAT alloués. 

![Métriques pour Standard Load Balancer](./media/load-balancer-standard-diagnostics/lbmetrics1anew.png)

*Figure : Métrique de disponibilité du chemin d’accès aux données pour Standard Load Balancer*

### <a name="retrieve-multi-dimensional-metrics-programmatically-via-apis"></a>Récupérer les métriques multidimensionnelles par programme via des API

Pour obtenir des instructions relatives à l’API permettant de récupérer les définitions et valeurs de métrique multidimensionnelle, consultez [Procédure pas à pas d’utilisation de l’API REST d’Azure Monitor](../azure-monitor/platform/rest-api-walkthrough.md#retrieve-metric-definitions-multi-dimensional-api). Ces indicateurs de performance peuvent être écrits dans un compte de stockage à l’aide de l’option « Tous les indicateurs de performance » uniquement. 

### <a name="configure-alerts-for-multi-dimensional-metrics"></a>Configurer des alertes pour des métriques multidimensionnelles ###

Azure Standard Load Balancer prend en charge des alertes facilement configurables pour les métriques multidimensionnelles. Configurez des seuils personnalisés pour des métriques spécifiques afin de déclencher des alertes avec différents niveaux de gravité et de bénéficier d’une expérience de supervision des ressources sans toucher.

Pour configurer des alertes :
1. Accédez au sous-panneau d’alerte pour l’équilibreur de charge.
1. Créer une nouvelle règle d’alerte
    1.  Configurez une condition d’alerte.
    1.  (Facultatif) Ajoutez un groupe d’actions pour la réparation automatisée.
    1.  Affectez la gravité, le nom et la description de l’alerte qui active la réaction intuitive.

  >[!NOTE]
  >La fenêtre de configuration des conditions d’alerte affiche les séries chronologiques pour l’historique des signaux. Il existe une option permettant de filtrer cette série chronologique par dimensions, par exemple par adresse IP back-end. Cela permet de filtrer le graphique de série chronologique, mais **pas** l’alerte elle-même. Vous ne pouvez pas configurer d’alertes pour des adresses IP back-end spécifiques.

### <a name="common-diagnostic-scenarios-and-recommended-views"></a><a name = "DiagnosticScenarios"></a>Scénarios de diagnostic courants et vues recommandées

#### <a name="is-the-data-path-up-and-available-for-my-load-balancer-frontend"></a>Le chemin d’accès aux données est-il opérationnel et disponible pour mon serveur frontal d’équilibrage de charge ?
<details><summary>Développez</summary>

La métrique de disponibilité du chemin de données décrit l’intégrité de ce dernier dans la région jusqu’à l’hôte de calcul où résident vos machines virtuelles. La métrique reflète l’intégrité de l’infrastructure Azure. Vous pouvez utiliser la métrique pour :
- Surveiller la disponibilité externe de votre service
- Aller plus loin et savoir si la plateforme sur laquelle votre service est déployé est intègre ou si votre système d’exploitation invité ou instance d’application est intègre.
- Déterminez si un événement est associé à votre service ou au plan de données sous-jacent. Ne confondez pas cette métrique avec l’état de la sonde d’intégrité (« disponibilité de l’instance de serveur principal »).

Pour obtenir la disponibilité du chemin d’accès aux données pour vos ressources Load Balancer Standard :
1. Assurez-vous que la ressource d’équilibreur de charge correcte est sélectionnée. 
2. Dans la liste déroulante **Métrique**, sélectionnez **Disponibilité du chemin d’accès aux données**. 
3. Dans la liste déroulante **Agrégation**, sélectionnez **Moy**. 
4. De plus, ajoutez un filtre sur l’adresse IP du serveur frontal ou le port frontal comme dimension avec l’adresse IP de serveur frontal ou le port de serveur frontal requis, puis regroupez-les en fonction de la dimension sélectionnée.

![Détection d’adresse IP virtuelle](./media/load-balancer-standard-diagnostics/LBMetrics-VIPProbing.png)

*Figure : Détails de la détection d’adresse IP frontale du Load Balancer*

La métrique est générée par une mesure intrabande active. Un service de détection dans la région provient du trafic pour la mesure. Le service est activé dès que vous créez un déploiement avec un serveur frontal public, et il continue de fonctionner tant que vous ne supprimez pas le serveur frontal. 

Un paquet correspondant au serveur frontal et à la règle de votre déploiement est généré régulièrement. Il traverse la région depuis la source vers l’hôte où une machine virtuelle dans le pool de serveur principal se trouve. L’infrastructure de l’équilibreur de charge effectue les mêmes opérations d’équilibrage de charge et de conversion que pour tout autre trafic. Cette sonde est intrabande sur votre point de terminaison équilibré en charge. Lorsque la sonde arrive sur l’hôte de calcul sur lequel se trouve une machine virtuelle intègre du pool principal, l’hôte de calcul génère une réponse au service de détection. Votre machine virtuelle ne voit pas ce trafic.

La disponibilité du chemin de données fait défaut pour les raisons suivantes :
- Il ne reste plus de machines virtuelles intègres du pool principal dans votre déploiement. 
- Une panne s’est produite au niveau de l’infrastructure.

À des fins de diagnostics, vous pouvez utiliser la [métrique de disponibilité du chemin d’accès aux données avec l’état de la sonde d’intégrité](#vipavailabilityandhealthprobes).

Utilisez une agrégation **Moyenne** pour la plupart des scénarios.
</details>

#### <a name="are-the-backend-instances-for-my-load-balancer-responding-to-probes"></a>Les instances de serveur principal pour mon équilibreur de charge répondent-elles aux sondes ?
<details>
  <summary>Développez</summary>
La métrique d’état de la sonde d’intégrité indique l’intégrité de votre déploiement d’application tel que vous l’avez configuré lors de la configuration de la sonde d’intégrité de votre équilibreur de charge. L’équilibreur de charge utilise l’état de la sonde d’intégrité pour déterminer où envoyer les nouveaux flux. Les sondes d’intégrité proviennent d’une adresse d’infrastructure Azure et sont visibles dans le système d’exploitation invité de la machine virtuelle.

Pour obtenir l’état de la sonde d’intégrité pour vos ressources Load Balancer Standard :
1. Sélectionnez la métrique **État de la sonde d’intégrité** avec le type d’agrégation **Moy**. 
2. Appliquez un filtre sur l’adresse IP frontale ou le port requis (ou les deux).

Les sondes d’intégrité échouent pour les raisons suivantes :
- Vous configurez une sonde d’intégrité pour un port qui n’écoute pas ou ne répond pas ou qui utilise le mauvais protocole. Si votre service utilise des règles de retour direct du serveur (DSR, ou adresse IP flottante), assurez-vous que le service écoute sur l’adresse IP de la configuration IP de la carte d’interface réseau, et pas seulement sur le bouclage configuré avec l’adresse IP de serveur frontal.
- Votre sonde n’est pas autorisée par le groupe de sécurité réseau, le pare-feu du système d’exploitation invité de la machine virtuelle ou les filtres de la couche Application.

Utilisez une agrégation **Moyenne** pour la plupart des scénarios.
</details>

#### <a name="how-do-i-check-my-outbound-connection-statistics"></a>Comment faire pour vérifier les statistiques de ma connexion sortante ? 
<details>
  <summary>Développez</summary>
La métrique des connexions SNAT indique le volume de connexions ayant réussi et ayant échoué pour les [flux sortants](./load-balancer-outbound-connections.md).

Un volume de connexions ayant échoué supérieur à zéro indique un épuisement du port SNAT. Vous devez mener un examen précis pour déterminer les causes de ces échecs. L’épuisement du port SNAT se traduit par un problème d’établissement de [flux sortant](./load-balancer-outbound-connections.md). Consultez l’article sur les connexions sortantes pour comprendre les scénarios, les mécanismes au travail et pour apprendre à atténuer et éviter l’épuisement du port SNAT. 

Pour obtenir des statistiques de connexion SNAT :
1. Sélectionnez le type de métrique **Connexions SNAT** et l’agrégation **Somme**. 
2. Regroupez par **État de la connexion** pour le nombre de connexions SNAT ayant réussi et échoué représenté par les différentes lignes. 

![Connexion SNAT](./media/load-balancer-standard-diagnostics/LBMetrics-SNATConnection.png)

*Figure : Nombre de connexions SNAT pour l’équilibreur de charge*
</details>


#### <a name="how-do-i-check-my-snat-port-usage-and-allocation"></a>Comment vérifier l’utilisation et l’allocation des ports SNAT ?
<details>
  <summary>Développez</summary>
La métrique Ports SNAT utilisés assure le suivi du nombre de ports SNAT consommés pour maintenir les flux sortants. Elle indique le nombre de flux uniques établis entre une source Internet et une machine virtuelle back-end ou un groupe de machines virtuelles identiques qui se trouve derrière un équilibreur de charge et qui n’a pas d’adresse IP publique. En comparant le nombre de ports SNAT que vous utilisez à la métrique Ports SNAT alloués, vous pouvez déterminer si votre service présente des problèmes ou des risques d’épuisement SNAT se traduisant par un échec de flux sortant. 

Si vos métriques indiquent un risque d'échec de [flux sortant](./load-balancer-outbound-connections.md), reportez-vous à l’article et prenez les mesures qui s'imposent pour l’atténuer afin de garantir l’intégrité du service.

Pour afficher l’utilisation et l’allocation des ports SNAT :
1. Définissez l’agrégation de temps du graphique sur une minute pour vous assurer que les données souhaitées s'affichent.
1. Sélectionnez **Ports SNAT utilisés** et/ou **Ports SNAT alloués** comme type de métrique et **Moyenne** comme agrégation
    * Par défaut, ces métriques représentent le nombre moyen de ports SNAT alloués ou utilisés par chaque VMSS ou machine virtuelle back-end, correspondant à toutes les adresses IP publiques front-end mappées à l’équilibreur de charge, agrégées sur TCP et UDP.
    * Pour afficher le nombre total de ports SNAT utilisés ou alloués pour l’équilibreur de charge, utilisez l’agrégation de métriques **Sum**.
1. Filtrez sur un **Type de protocole** spécifique, un ensemble d'**Adresses IP principales** et/ou des **Adresses IP frontales**.
1. Pour surveiller l’intégrité par instance principale ou frontale, utilisez le fractionnement. 
    * Notez que le fractionnement permet l'affichage d'une seule métrique à la fois. 
1. Par exemple, pour surveiller l’utilisation SNAT pour les flux TCP par machine, agrégez par **Moyenne**, fractionnez par **Adresses IP principales** et filtrez par **Type de protocole**. 

![Allocation et utilisation SNAT](./media/load-balancer-standard-diagnostics/snat-usage-and-allocation.png)

*Figure : Utilisation et allocation de ports TCP SNAT moyennes pour un ensemble de machines virtuelles principales*

![Utilisation SNAT par instance principale](./media/load-balancer-standard-diagnostics/snat-usage-split.png)

*Figure : Utilisation de ports SNAT TCP par instance de serveur principal*
</details>

#### <a name="how-do-i-check-inboundoutbound-connection-attempts-for-my-service"></a>Comment faire pour vérifier les tentatives de connexions entrantes/sortantes pour mon service ?
<details>
  <summary>Développez</summary>
Une métrique de paquets SYN indique le volume de paquets TCP SYN qui ont été reçus ou qui ont été envoyés (pour les [flux sortants](./load-balancer-outbound-connections.md)) associés à un serveur frontal spécifique. Vous pouvez utiliser cette métrique pour comprendre les tentatives de connexions TCP vers votre service.

Utilisez une agrégation **Totale** pour la plupart des scénarios.

![Connexion SYN](./media/load-balancer-standard-diagnostics/LBMetrics-SYNCount.png)

*Figure : Nombre de SYN de l’équilibreur de charge*
</details>


#### <a name="how-do-i-check-my-network-bandwidth-consumption"></a>Comment faire pour vérifier la consommation de ma bande passante réseau ? 
<details>
  <summary>Développez</summary>
La métrique de compteurs d’octets et de paquets indique le volume d’octets et de paquets envoyés ou reçus par votre service par serveur frontal.

Utilisez une agrégation **Totale** pour la plupart des scénarios.

Pour obtenir les statistiques de nombre d’octets ou de paquets :
1. Sélectionnez le type de métrique **Bytes Count** (Nombre d’octets) et/ou **Nombre de paquets** avec l’agrégation **Moy**. 
2. Effectuez l'une des opérations suivantes :
   * Appliquez un filtre sur une adresse IP frontale, un port frontal, une adresse IP de serveur principal ou un port de serveur principal spécifique.
   * Récupérez des statistiques globales pour votre ressource d’équilibreur de charge sans aucun filtrage.

![Nombre d’octets](./media/load-balancer-standard-diagnostics/LBMetrics-ByteCount.png)

*Figure : Nombre d’octets de l’équilibreur de charge*
</details>

#### <a name="how-do-i-diagnose-my-load-balancer-deployment"></a><a name = "vipavailabilityandhealthprobes"></a>Comment faire pour diagnostiquer mon déploiement d’équilibreur de charge ?
<details>
  <summary>Développez</summary>
La combinaison des métriques de disponibilité du chemin d’accès et d’état des sondes d’intégrité sur un même graphique vous permet de déterminer où rechercher et résoudre le problème. Vous pouvez vous assurer qu’Azure fonctionne correctement et utiliser ces informations pour déterminer si la configuration ou l’application est la cause racine.

Vous pouvez utiliser des métriques de sonde d’intégrité pour comprendre comment Azure affiche l’intégrité de votre déploiement en fonction de la configuration que vous avez fournie. Examiner les sondes d’intégrité est toujours une première étape très utile dans l’analyse ou la détermination d’une cause.

Vous pouvez aller plus loin et utiliser la métrique de disponibilité du chemin d’accès pour comprendre comment Azure affiche l’intégrité du plan de données sous-jacent responsable de votre déploiement. Lorsque vous combinez les deux métriques, vous pouvez déterminer où l’erreur peut se situer comme illustré dans cet exemple :

![Combinaison des métriques de disponibilité du chemin d’accès aux données et de l’état de la sonde d’intégrité](./media/load-balancer-standard-diagnostics/lbmetrics-dipnvipavailability-2bnew.png)

*Figure : Combinaison des métriques de disponibilité du chemin d’accès aux données et de l’état de la sonde d’intégrité*

Ce graphique affiche les informations suivantes :
- L’infrastructure qui héberge vos machines virtuelles n’était pas disponible et à 0 pour cent au début du graphique. Plus tard, l’infrastructure était intègre, les machines virtuelles étaient accessibles et plusieurs machines virtuelles ont été placées dans le serveur principal. Ces informations sont représentées par le trait bleu pour la disponibilité du chemin d’accès aux données, qui a été plus tard de 100 %. 
- L’état de la sonde d’intégrité, indiqué par le trait violet, est à 0 % au début du graphique. La zone dans le cercle vert indique où l’état de la sonde d’intégrité est devenu intègre, et à quel point le déploiement du client a été en mesure d’accepter de nouveaux flux.

Le graphique permet aux clients de dépanner eux-mêmes le déploiement sans devoir deviner ou demander de l’aide sur d’autres problèmes. Le service n’était pas disponible, car les sondes d’intégrité ont échoué en raison d’une configuration incorrecte ou de l’échec d’une application.
</details>

## <a name="resource-health-status"></a><a name = "ResourceHealth"></a>État d’intégrité des ressources

L’état d’intégrité des ressources de niveau Standard de Load Balancer est indiqué dans la page **Intégrité des ressources** existante sous **Monitor > État du service**.

Pour afficher l’intégrité de vos ressources Load Balancer Standard public :
1. Sélectionnez **Monitor** > **État du service**.

   ![Port Monitor](./media/load-balancer-standard-diagnostics/LBHealth1.png)

   *Figure : Lien vers État du service sur Azure Monitor*

2. Sélectionnez **Intégrité des ressources** et vérifiez que **l’ID d’abonnement** et le **Type de ressource = Load Balancer** sont sélectionnés.

   ![État d’intégrité des ressources](./media/load-balancer-standard-diagnostics/LBHealth3.png)

   *Figure : Sélectionner une ressource pour la vue d’intégrité*

3. Dans la liste, cliquez sur la ressource Load Balancer pour afficher son état d’intégrité historique.

    ![État d’intégrité de Load Balancer](./media/load-balancer-standard-diagnostics/LBHealth4.png)

   *Figure : Vue d’intégrité de la ressource Load Balancer*
 
La description de l’état d’intégrité de ressource générique est disponible dans la [documentation RHC](../service-health/resource-health-overview.md). Les états spécifiques d’Azure Load Balancer sont listés dans le tableau ci-dessous : 

| État d’intégrité des ressources | Description |
| --- | --- |
| Disponible | Votre ressource d’équilibreur de charge standard est intègre et disponible. |
| Détérioré | Votre équilibreur de charge standard présente des événements lancés par la plateforme ou l’utilisateur qui nuisent aux performances. La métrique Disponibilité du chemin de données a fait état d’une intégrité inférieure à 90 %, mais supérieure à 25 % pendant au moins deux minutes. L’impact sur les performances que vous allez subir sera de niveau modéré à grave. [Suivez le guide de résolution des problèmes de disponibilité du chemin de données] pour déterminer si les événements ayant un impact sur votre disponibilité sont lancés par l’utilisateur.
| Non disponible | Votre ressource d’équilibreur de charge standard n’est pas intègre. La métrique Disponibilité du chemin des données a fait état d’une intégrité inférieure à 25 % pendant au moins deux minutes. Vous allez subir un impact sur les performances significatif ou un défaut de disponibilité pour la connectivité entrante. Des événements utilisateur ou plateforme peuvent être à l’origine de l’indisponibilité. [Suivez le guide de résolution des problèmes de disponibilité du chemin de données] pour déterminer si les événements ayant un impact sur votre disponibilité sont lancés par l’utilisateur. |
| Unknown | L’état d’intégrité de votre ressource d’équilibrage de charge standard n’a pas encore été mis à jour ou n’a pas encore reçu d’informations de disponibilité du chemin de données au cours des 10 dernières minutes. Cet état devrait être transitoire et passer à un état correct dès que des données seront reçues. |

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur l’[équilibreur de charge standard](./load-balancer-overview.md).
- En savoir plus sur la [connectivité sortante de votre équilibreur de charge](./load-balancer-outbound-connections.md).
- Découvrez [Azure Monitor](../azure-monitor/overview.md).
- Découvrez l’[API REST Azure Monitor](/rest/api/monitor/) et [comment récupérer des métriques par le biais de l’API REST](/rest/api/monitor/metrics/list).