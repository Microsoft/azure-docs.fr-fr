---
title: Prendre en charge le trafic élevé Application Gateway
description: Cet article fournit des conseils pour configurer Azure Application Gateway en vue de la prise en charge de grands volumes de trafic réseau.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: conceptual
ms.date: 03/24/2020
ms.author: caya
ms.openlocfilehash: a5f7569fc46d4678ca0c12299e33caa3c78df849
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182918"
---
# <a name="application-gateway-high-traffic-support"></a>Prendre en charge le trafic élevé avec Application Gateway

>[!NOTE]
> Cet article décrit quelques recommandations pour vous aider à configurer votre Application Gateway afin de gérer le trafic supplémentaire résultant d’un volume de trafic élevé qui peut se produire. Les seuils d’alerte sont purement des suggestions et donc génériques par nature. Les utilisateurs peuvent déterminer les seuils d’alerte en fonction de leurs attentes en matière de charge de travail et d’utilisation.

Vous pouvez utiliser Application Gateway avec le pare-feu d’applications Web (WAF) pour disposer d’un moyen évolutif et sécurisé de gérer le trafic vers vos applications web.

Il est important de mettre à l’échelle votre Application Gateway en fonction de votre trafic et de prévoir un peu de mémoire tampon afin d’anticiper les pics de trafic et de réduire l’impact que cela peut avoir dans votre QoS. Les suggestions suivantes vous aident à configurer Application Gateway avec WAF pour gérer le trafic supplémentaire.

Pour obtenir la liste complète des métriques fournies par Application Gateway, consultez la [documentation sur les métriques](./application-gateway-metrics.md). Pour plus d’informations sur la façon de définir des alertes pour les métriques, consultez [Visualiser des métriques](./application-gateway-metrics.md#metrics-visualization) dans le portail Azure et la [documentation Azure Monitor](../azure-monitor/platform/alerts-metric.md).

## <a name="scaling-for-application-gateway-v1-sku-standardwaf-sku"></a>Mise à l'échelle pour la référence SKU Application Gateway v1 (SKU Standard/WAF)

### <a name="set-your-instance-count-based-on-your-peak-cpu-usage"></a>Définir le nombre d’instances en fonction de vos pics d’utilisation du processeur
Si vous utilisez une passerelle SKU v1, vous avez la possibilité de définir jusqu’à 32 instances Application Gateway pour la mise à l’échelle. Vérifiez l’utilisation du processeur de votre Application Gateway au cours du mois passé pour détecter les pics au-delà de 80 %. Cette information est disponible en tant que métrique à surveiller. Il est recommandé de définir le nombre d’instances en fonction de votre pic d’utilisation, eavec une mémoire tampon supplémentaire de 10 à 20 % pour tenir compte des pics de trafic.

:::image type="content" source="./media/application-gateway-covid-guidelines/v1-cpu-utilization-inline.png" alt-text="Métriques d'utilisation du processeur V1" lightbox="./media/application-gateway-covid-guidelines/v1-cpu-utilization-exp.png":::

### <a name="use-the-v2-sku-over-v1-for-its-autoscaling-capabilities-and-performance-benefits"></a>Utiliser la référence (SKU) v2 plutôt que la v1 pour ses capacités de mise à l’échelle automatiques et ses avantages en matière de performances
La v2 applique la mise à l’échelle automatique pour s’assurer que votre Application Gateway peut monter en puissance à mesure que le trafic augmente. Elle offre également d’autres avantages significatifs en matière de performances, par exemple des performances 5 fois supérieures de déchargement TLS, des temps de déploiement et de mise à jour plus courts, la redondance de zone, etc., par rapport à la v1. Pour plus d’informations, consultez notre [documentation v2](./application-gateway-autoscaling-zone-redundant.md) et notre [documentation sur la migration](./migrate-v1-v2.md) de v1 à v2 pour savoir comment migrer vos passerelles SKU v1 existantes vers la référence SKU v2. 

## <a name="autoscaling-for-application-gateway-v2-sku-standard_v2waf_v2-sku"></a>Mise à l'échelle automatique pour la référence SKU Application Gateway v2 (SKU Standard_v2/WAF_v2)

### <a name="set-maximum-instance-count-to-the-maximum-possible-125"></a>Définir le nombre d’instances maximal sur la valeur maximale possible (125)
 
Pour une SKU Application Gateway v2, la définition du nombre d’instances maximal sur la valeur maximale possible de 125 permet à Application Gateway de monter en charge en fonction des besoins. Cela lui permet de gérer l’augmentation possible du trafic vers vos applications. Seules les unités de capacité (CU) que vous utilisez vous seront facturées. 

Veillez à vérifier la taille de votre sous-réseau et le nombre d’adresses IP disponibles dans votre sous-réseau, puis définissez le nombre d’instances maximal selon cette valeur. Si votre sous-réseau n’a pas suffisamment d’espace, vous devrez recréer votre passerelle dans le même sous-réseau ou dans un autre sous-réseau qui dispose d’une capacité suffisante. 

:::image type="content" source="./media/application-gateway-covid-guidelines/v2-autoscaling-max-instances-inline.png" alt-text="Configuration de la mise à l’échelle automatique V2" lightbox="./media/application-gateway-covid-guidelines/v2-autoscaling-max-instances-exp.png":::

### <a name="set-your-minimum-instance-count-based-on-your-average-compute-unit-usage"></a>Définir le nombre minimal d’instances en fonction de votre utilisation moyenne d’unités de calcul

Pour la référence SKU Application Gateway v2, la mise à l’échelle automatique prend six à sept minutes pour effectuer un scale-out et préparer un ensemble supplémentaire d’instances prêtes à gérer le trafic. En attendant, si le trafic connaît de courts pics, vos instances de passerelle existantes peuvent être fortement sollicitées, entraînant une latence ou une perte inattendue du trafic. 

Il est recommandé de définir le nombre minimal d’instances sur un niveau optimal. Par exemple, si vous avez besoin de 50 instances pour gérer le trafic lors des périodes de charge maximale, réduire la valeur minimale de 25 à 30 plutôt qu’à < 10 est judicieux, de sorte que même en cas de pics de trafic, Application Gateway est capable de gérer ce trafic, laissant suffisamment de temps pour déclencher et mettre en œuvre la mise à l’échelle automatique.

Vérifiez la métrique de votre unité de calcul pour le mois passé. La métrique d’unité de calcul est une représentation de l’utilisation du processeur de votre passerelle, en fonction de votre utilisation maximale, divisée par 10. Vous pouvez définir le nombre minimal d’instances requises. Notez qu’une instance Application Gateway peut gérer un minimum de 10 unités de calcul

:::image type="content" source="./media/application-gateway-covid-guidelines/compute-unit-metrics-inline.png" alt-text="Métriques des unités de calcul V2" lightbox="./media/application-gateway-covid-guidelines/compute-unit-metrics-exp.png":::

## <a name="manual-scaling-for-application-gateway-v2-sku-standard_v2waf_v2"></a>Mise à l'échelle manuelle pour la référence SKU Application Gateway v2 (Standard_v2/WAF_v2)

### <a name="set-your-instance-count-based-on-your-peak-compute-unit-usage"></a>Définir le nombre d’instances en fonction de vos pics d’utilisation d’unités de calcul 

Contrairement à la mise à l’échelle automatique, dans le cas d’une mise à l’échelle manuelle, vous devez définir manuellement le nombre d’instances de votre passerelle d’application en fonction des exigences du trafic. Il est recommandé de définir le nombre d’instances en fonction de votre pic d’utilisation, eavec une mémoire tampon supplémentaire de 10 à 20 % pour tenir compte des pics de trafic. Par exemple, si votre trafic requiert 50 instances au niveau de charge maximale, provisionnez 55 à 60 instances pour gérer les pics de trafic inattendus qui peuvent se produire.

Vérifiez la métrique de votre unité de calcul pour le mois passé. La métrique d’unité de calcul est une représentation de l’utilisation du processeur de votre passerelle, en fonction de votre utilisation maximale, divisée par 10. Vous pouvez définir le nombre d’instances nécessaires, car une instance Application Gateway peut gérer un minimum de 10 unités de calcul

## <a name="monitoring-and-alerting"></a>Surveillance et alerte

Pour être informé des anomalies de trafic ou d’utilisation, vous pouvez configurer des alertes sur certaines métriques. Pour obtenir la liste complète des métriques fournies par Application Gateway, consultez la [documentation sur les métriques](./application-gateway-metrics.md). Pour plus d’informations sur la façon de définir des alertes pour les métriques, consultez [Visualiser des métriques](./application-gateway-metrics.md#metrics-visualization) dans le portail Azure et la [documentation Azure Monitor](../azure-monitor/platform/alerts-metric.md).

## <a name="alerts-for-application-gateway-v1-sku-standardwaf"></a>Alertes pour la référence SKU Application Gateway v1 (Standard/WAF)

### <a name="alert-if-average-cpu-utilization-crosses-80"></a>Alerter si l’utilisation moyenne du processeur dépasse 80 %

Dans des conditions normales, l’utilisation du processeur ne doit pas dépasser 90 %, car cela peut entraîner une latence dans les sites Web hébergés derrière Application Gateway et perturber l’expérience du client. Vous pouvez indirectement contrôler ou améliorer l'utilisation du processeur en modifiant la configuration d'Application Gateway en augmentant le nombre d'instances ou en passant à une taille SKU plus grande, ou en faisant les deux. Définissez une alerte si la métrique d’utilisation du processeur dépasse 80 % de la valeur moyenne.

### <a name="alert-if-unhealthy-host-count-crosses-threshold"></a>Alerter si le nombre d’hôtes non sains dépasse le seuil

Cette métrique indique le nombre de serveurs back-end que la passerelle Application Gateway ne peut pas détecter correctement. Cela permet de détecter les problèmes où les instances de passerelle d’application ne peuvent pas se connecter au back-end. Alerter si ce nombre dépasse 20 % de la capacité du back-end. Par exemple, Si vous disposez actuellement de 30 serveurs back-end dans leur pool back-end, définissez une alerte si le nombre d’hôtes non sains est supérieur à 6.

### <a name="alert-if-response-status-4xx-5xx-crosses-threshold"></a>Alerter si l’état de réponse (4xx, 5xx) dépasse le seuil 

Créer une alerte lorsque l’état de la réponse Application Gateway est 4xx ou 5xx. Des réponses 4xx ou 5xx peuvent parfois s’afficher en raison de problèmes temporaires. Vous devez observer la passerelle en production pour déterminer le seuil statique, ou utiliser le seuil dynamique pour l’alerte.

### <a name="alert-if-failed-requests-crosses-threshold"></a>Alerter si les échecs de requêtes franchissent le seuil 

Créer une alerte lorsque la métrique des échecs de requêtes franchit le seuil. Vous devez observer la passerelle en production pour déterminer le seuil statique, ou utiliser le seuil dynamique pour l’alerte.

### <a name="example-setting-up-an-alert-for-more-than-100-failed-requests-in-the-last-5-minutes"></a>Exemple : Configuration d’une alerte pour plus de 100 échecs de requêtes au cours des 5 dernières minutes

Cet exemple montre comment utiliser le portail Azure pour configurer une alerte lorsque le nombre d’échecs de requêtes au cours des 5 dernières minutes est supérieur à 100.
1. Accédez à votre **Application Gateway**.
2. Dans le volet de gauche, sélectionnez **Mesures** sous l’onglet **Surveillance**. 
3. Ajoutez une métrique pour les **échecs de requêtes**.
4. Cliquez sur **Nouvelle règle d’alerte** et définissez votre condition et vos actions
5. Cliquez sur **Créer une règle d’alerte** pour créer et activer l’alerte

:::image type="content" source="./media/application-gateway-covid-guidelines/create-alerts-inline.png" alt-text="Créer des alertes V2" lightbox="./media/application-gateway-covid-guidelines/create-alerts-exp.png":::

## <a name="alerts-for-application-gateway-v2-sku-standard_v2waf_v2"></a>Alertes pour Application Gateway v2 SKU (Standard_v2/WAF_v2)

### <a name="alert-if-compute-unit-utilization-crosses-75-of-average-usage"></a>Alerter si l’utilisation de l’unité de calcul dépasse 75 % de l’utilisation moyenne 

L’unité de calcul mesure l’utilisation du calcul de votre Application Gateway. Vérifiez l’utilisation moyenne des unités de calcul au cours du dernier mois et définissez une alerte si elle dépasse 75 % de cette valeur. Par exemple, si votre utilisation moyenne est de 10 unités de calcul, définissez une alerte sur 7,5 CU. Cela vous avertit si votre utilisation augmente et cela vous donne le temps de réagir en fonction. Vous pouvez augmenter la valeur minimale si vous pensez que ce niveau de trafic sera maintenu, afin de vous avertir que le trafic peut augmenter. Suivez les suggestions de mise à l’échelle ci-dessus pour procéder à un scale-out si nécessaire.

### <a name="example-setting-up-an-alert-on-75-of-average-cu-usage"></a>Exemple : configuration d’une alerte à 75 % de l’utilisation moyenne des unités de capacité

Cet exemple montre comment utiliser le portail Azure pour configurer une alerte quand l’utilisation moyenne des unités de capacité atteint 75 %. 
1. Accédez à votre **Application Gateway**.
2. Dans le volet de gauche, sélectionnez **Mesures** sous l’onglet **Surveillance**. 
3. Ajoutez une mesure pour la **moyenne des unités de calcul actuelles**. 
4. Si vous avez défini le nombre minimal d’instances comme étant l’utilisation moyenne des unités de capacité, vous pouvez définir une alerte quand 75 % de vos instances minimales sont en cours d’utilisation. Par exemple, si votre utilisation moyenne est de 10 CU, définissez une alerte sur 7,5 CU. Cela vous avertit si votre utilisation augmente et cela vous donne le temps de réagir en fonction. Vous pouvez augmenter la valeur minimale si vous pensez que ce niveau de trafic sera maintenu, afin de vous avertir que le trafic peut augmenter. 

:::image type="content" source="./media/application-gateway-covid-guidelines/compute-unit-alert-inline.png" alt-text="Alertes des unités de calcul V2" lightbox="./media/application-gateway-covid-guidelines/compute-unit-alert-exp.png":::

> [!NOTE]
> Vous pouvez définir l’alerte pour qu’elle se produise un pourcentage d’utilisation des unités de capacité inférieur ou supérieur, en fonction de la sensibilité que vous souhaitez pour les pics de trafic potentiels.

### <a name="alert-if-capacity-unit-utilization-crosses-75-of-peak-usage"></a>Alerter si l’utilisation de l’unité de capacité dépasse 75 % de l’utilisation maximale 

Les unités de capacité représentent l’utilisation globale de la passerelle en termes de débit, de calcul et de nombre de connexions. Vérifiez l’utilisation de votre unité de capacité maximale au cours du dernier mois et définissez une alerte si elle dépasse 75 % de cette valeur. Par exemple, si votre utilisation maximale est de 100 unités de capacité, définissez une alerte sur 75 CU. Suivez les deux suggestions ci-dessus pour effectuer un scale-out, si nécessaire.

### <a name="alert-if-unhealthy-host-count-crosses-threshold"></a>Alerter si le nombre d’hôtes non sains dépasse le seuil 

Cette métrique indique le nombre de serveurs back-end que la passerelle Application Gateway ne peut pas détecter correctement. Cela permet de détecter les problèmes où les instances de passerelle d’application ne peuvent pas se connecter au back-end. Alerter si ce nombre dépasse 20 % de la capacité du back-end. Par exemple, Si vous disposez actuellement de 30 serveurs back-end dans leur pool back-end, définissez une alerte si le nombre d’hôtes non sains est supérieur à 6.

### <a name="alert-if-response-status-4xx-5xx-crosses-threshold"></a>Alerter si l’état de réponse (4xx, 5xx) dépasse le seuil 

Créer une alerte lorsque l’état de la réponse Application Gateway est 4xx ou 5xx. Des réponses 4xx ou 5xx peuvent parfois s’afficher en raison de problèmes temporaires. Vous devez observer la passerelle en production pour déterminer le seuil statique, ou utiliser le seuil dynamique pour l’alerte.

### <a name="alert-if-failed-requests-crosses-threshold"></a>Alerter si les échecs de requêtes franchissent le seuil 

Créer une alerte lorsque la métrique des échecs de requêtes franchit le seuil. Vous devez observer la passerelle en production pour déterminer le seuil statique, ou utiliser le seuil dynamique pour l’alerte.

### <a name="alert-if-backend-last-byte-response-time-crosses-threshold"></a>Alerter si le temps de réponse du dernier octet back-end franchit le seuil 

Cette métrique indique l’intervalle de temps entre le début de l’établissement d’une connexion au serveur principal et la réception du dernier octet du corps de la réponse. Créez une alerte si la latence de la réponse du back-end est supérieure à celle d’un seuil normal. Par exemple, définissez cette valeur afin d’être averti lorsque la latence de réponse du back-end augmente de plus de 30 % par rapport à la valeur habituelle.

### <a name="alert-if-application-gateway-total-time-crosses-threshold"></a>Alerter si le temps total Application Gateway dépasse le seuil

Il s’agit de l’intervalle entre le moment où Application Gateway reçoit le premier octet de la requête HTTP et le moment où le dernier octet de la réponse a été envoyé au client. Doit créer alerte si la latence de la réponse du back-end est supérieure à celle d’un seuil normal. Par exemple, ils peuvent définir cette valeur afin d’être avertis lorsque la latence totale augmente de plus de 30 % par rapport à la valeur habituelle.

## <a name="set-up-waf-with-geo-filtering-and-bot-protection-to-stop-attacks"></a>Configurer WAF avec le géofiltrage et la protection bot pour bloquer les attaques
Si vous souhaitez ajouter une couche de sécurité devant votre application, utilisez Application Gateway WAF_v2 pour les fonctionnalités WAF. Vous pouvez configurer la référence v2 pour autoriser uniquement l’accès à vos applications à partir d’un ou plusieurs pays/régions. Vous configurez une règle personnalisée WAF pour autoriser ou bloquer explicitement le trafic en fonction de la géolocalisation. Pour plus d’informations, consultez [Règles personnalisées de géofiltrage](../web-application-firewall/ag/geomatch-custom-rules.md) et [Comment configurer des règles personnalisées sur Application Gateway WAF_v2 via PowerShell](../web-application-firewall/ag/configure-waf-custom-rules.md).

Activez la protection bot pour bloquer les mauvais robots connus. Cela devrait réduire le volume de trafic qui arrive jusqu’à votre application. Pour plus d’informations, consultez [Protection bot avec instructions de configuration](../web-application-firewall/ag/configure-waf-custom-rules.md).

## <a name="turn-on-diagnostics-on-application-gateway-and-waf"></a>Activer les diagnostics sur Application Gateway et WAF

Les journaux de diagnostic vous permettent de consulter les journaux de pare-feu, les journaux de performances et les journaux d’accès. Vous pouvez utiliser ces journaux d’activité dans Azure pour gérer les passerelles Application Gateway et résoudre les problèmes associés. Pour plus d’informations, consultez la [documentation des diagnostics](./application-gateway-diagnostics.md#diagnostic-logging). 

## <a name="set-up-an-tls-policy-for-extra-security"></a>Configurer une stratégie TLS pour plus de sécurité
Vérifiez que vous utilisez la dernière version de la stratégie TLS ([AppGwSslPolicy20170401S](./application-gateway-ssl-policy-overview.md#appgwsslpolicy20170401s)). Cela garantit le chiffrement TLS 1.2 et les chiffrements renforcés. Pour plus d’informations, consultez [Configuration des versions de stratégie TLS et des suites de chiffrement via PowerShell](./application-gateway-configure-ssl-policy-powershell.md).