---
title: Récupération d’urgence à l’aide d’Azure DNS et Traffic Manager | Microsoft Docs
description: Présentation des solutions de récupération d’urgence à l’aide d’Azure DNS et de Traffic Manager
services: dns
documentationcenter: na
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: dns
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/08/2018
ms.author: kumud
ms.openlocfilehash: 6eab1803bf5adab42be87b5f8567682c6d75947e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74483525"
---
# <a name="disaster-recovery-using-azure-dns-and-traffic-manager"></a>Récupération d’urgence à l’aide d’Azure DNS et Traffic Manager

La récupération d’urgence se concentre sur la récupération des fonctionnalités des applications en cas de perte grave. Pour choisir une solution de récupération d’urgence, les propriétaires de sociétés et de produits technologiques doivent d’abord déterminer le niveau de fonctionnalité requis lors d’un incident (indisponible, partiellement disponible avec fonctionnalités réduites, disponibilité retardée, ou entièrement disponible).
La plupart des clients d’entreprise choisissent une architecture sur plusieurs régions pour assurer la résilience en cas de basculement au niveau de l’application ou de l’infrastructure. Les clients peuvent choisir différentes approches pour effectuer le basculement en conservant une haute disponibilité via une architecture redondante. Ils disposent notamment des approches suivantes :

- **Mode actif/passif avec reprise progressive** : dans cette solution de basculement, les machines virtuelles et autres appliances exécutées dans la région de secours ne sont pas actives tant que le basculement n’est pas nécessaire. Cependant, l’environnement de production est répliqué vers une autre région sous la forme de sauvegardes, d’images de machine virtuelle ou de modèles Resource Manager. Ce mécanisme de basculement est économique mais son exécution complète prend plus de temps.
 
    ![Mode actif/passif avec reprise progressive](./media/disaster-recovery-dns-traffic-manager/active-passive-with-cold-standby.png)
    
    *Figure - Mode actif/passif avec configuration de récupération d’urgence Reprise progressive*

- **Mode actif/passif avec veilleuse** : dans cette solution de basculement, l’environnement de secours est configuré avec une configuration minimale. Le programme d’installation n’exécute que les services nécessaires pour prendre en charge un jeu d’applications obligatoire minimal. Dans sa forme native, ce scénario peut uniquement exécuter des fonctionnalités minimales, mais il peut évoluer et générer des services supplémentaires pour prendre en bloc la charge de production si un basculement se produit.
    
    ![Mode actif/passif avec veilleuse](./media/disaster-recovery-dns-traffic-manager/active-passive-with-pilot-light.png)
    
    *Figure : Mode actif/passif avec configuration de récupération d’urgence Veilleuse*

- **Mode actif/passif avec secours semi-automatique** : dans cette solution de basculement, la région de secours est préparée et prête à assumer la charge de base, la mise à l’échelle automatique est activée et toutes les instances sont opérationnelles. Cette solution n’est pas prévue pour assumer la totalité de la charge de production, mais elle est fonctionnelle, et tous les services sont opérationnels. Cette solution est une version augmentée de l’approche avec veilleuse.
    
    ![Mode actif/passif avec secours semi-automatique](./media/disaster-recovery-dns-traffic-manager/active-passive-with-warm-standby.png)
    
    *Figure : Mode actif/passif avec configuration de récupération d’urgence Secours semi-automatique*
    
Pour plus d’informations sur le basculement et la haute disponibilité, consultez [Récupération d’urgence pour les applications Azure](https://docs.microsoft.com/azure/architecture/resiliency/disaster-recovery-azure-applications).


## <a name="planning-your-disaster-recovery-architecture"></a>Planification de votre architecture de récupération d’urgence

Deux aspects techniques sont à prendre en considération lors de la configuration de votre architecture de récupération d’urgence :
-  Utiliser un mécanisme de déploiement pour répliquer les instances, les données et les configurations entre les environnements primaire et de secours. Ce type de récupération d’urgence est possible en mode natif dans Azure Site-Recovery via les dispositifs/services de partenaires de Microsoft Azure comme Veritas ou NetApp. 
- Développer une solution afin de transférer le trafic réseau/web du site principal vers le site de secours. Ce type de récupération d’urgence peut être mis en œuvre via Azure DNS, Azure Trafic Manager (DNS) ou des équilibreurs de charge globale tiers.

Cet article concerne seulement les approches via la redirection du trafic réseau et web. Pour connaître les instructions relatives à la configuration d’Azure Site Recovery, consultez la [Documentation d’Azure Site Recovery](https://docs.microsoft.com/azure/site-recovery/).
DNS est l’un des mécanismes les plus efficaces pour rediriger le trafic réseau, car il est généralement mondial et externe au centre de données, et ainsi isolé des défaillances au niveau de la région ou de la zone de disponibilité. On peut utiliser un mécanisme de basculement basé sur DNS et, dans Azure, deux services DNS peuvent accomplir la même chose d’une certaine manière : Azure DNS (DNS faisant autorité) et Azure Traffic Manager (routage du trafic intelligent basé sur DNS). 

Il est important de comprendre certains concepts DNS largement utilisés pour parler des solutions décrites dans cet article :
- **Enregistrement A DNS** : les enregistrements A sont des pointeurs qui pointent un domaine vers une adresse IPv4. 
- **Nom CNAME ou canonique** : ce type d’enregistrement est utilisé pour pointer vers un autre enregistrement DNS. CNAME ne répond pas par une adresse IP, mais par le pointeur vers l’enregistrement qui contient l’adresse IP. 
- **Routage basé sur la pondération** : on peut choisir d’associer un poids à des points de terminaison de service, puis répartir le trafic en fonction des poids attribués. Cette méthode de routage est l’un des quatre mécanismes de routage du trafic disponibles dans Traffic Manager. Pour plus d’informations, consultez la section [Méthode de routage basé sur la pondération](../traffic-manager/traffic-manager-routing-methods.md#weighted).
- **Routage par priorité** : le routage par priorité est basé sur le contrôle de l’intégrité des points de terminaison. Par défaut, Azure Traffic Manager envoie tout le trafic vers le point de terminaison ayant la priorité la plus élevée et, en cas de défaillance ou d’incident, achemine le trafic vers le point de terminaison secondaire. Pour plus d’informations, consultez la section [Méthode de routage basé sur la priorité](../traffic-manager/traffic-manager-routing-methods.md#priority-traffic-routing-method).

## <a name="manual-failover-using-azure-dns"></a>Basculement manuel à l’aide d’Azure DNS
La solution de basculement manuel Azure DNS pour la récupération d’urgence utilise le mécanisme DNS standard pour effectuer le basculement vers le site de sauvegarde. L’option manuelle via Azure DNS fonctionne de manière optimale lorsqu’elle est utilisée conjointement avec l’approche Reprise progressive ou Veilleuse. 

![Basculement manuel à l’aide d’Azure DNS](./media/disaster-recovery-dns-traffic-manager/manual-failover-using-dns.png)

*Figure - Basculement manuel à l’aide d’Azure DNS*

Les hypothèses formulées pour la solution sont :
- Les points de terminaison principal et secondaire ont des adresses IP statiques qui ne changent pas souvent. Par exemple, l’adresse IP du site principal est 100.168.124.44 et celle du site secondaire 100.168.124.43.
- Il existe une zone Azure DNS pour le site principal et pour le site secondaire. Par exemple, le point de terminaison pour le site principal est prod.contoso.com et pour le site de sauvegarde dr.contoso.com. Il existe également un enregistrement DNS pour l’application principale appelé \.www.contoso.com.   
- La durée de vie est égale ou inférieure à la valeur RTO du SLA définie dans l’organisation. Par exemple, si une entreprise définit le RTO de réponse à l’incident de l’application sur 60 minutes, la durée de vie doit être inférieure à 60 minutes, de préférence la plus faible valeur possible. 
  Vous pouvez configurer Azure DNS de la façon suivante pour le basculement manuel :
- Création d’une zone DNS
- Création d’enregistrements de zone DNS
- Mise à jour de l’enregistrement CNAME

### <a name="step-1-create-a-dns"></a>Étape 1 : Création d’une zone DNS
Procédez comme suit pour créer une zone DNS (par exemple, www\.contoso.com) :

![Création d’une zone DNS dans Azure](./media/disaster-recovery-dns-traffic-manager/create-dns-zone.png)

*Figure - Création d’une zone DNS dans Azure*

### <a name="step-2-create-dns-zone-records"></a>Étape 2 : Création d’enregistrements de zone DNS

Dans cette zone, créez trois enregistrements (par exemple, www\.contoso.com, prod.contoso.com et dr.consoto.com), comme illustré ci-dessous.

![Création d’enregistrements de zone DNS](./media/disaster-recovery-dns-traffic-manager/create-dns-zone-records.png)

*Figure - Création d’enregistrements de zone DNS dans Azure*

Dans ce scénario, le site www\.contoso.com a une durée de vie de 30 minutes, ce qui est bien inférieur au RTO annoncé, et pointe vers le site de production prod.contoso.com. Cette configuration est vraie pour les opérations commerciales normales. La durée de vie de prod.contoso.com et de dr.contoso.com a été définie sur 300 secondes soit 5 minutes. Vous pouvez utiliser un service de surveillance tel qu’Azure Monitor ou Azure App Insights, ou encore une solution de surveillance de partenaire comme Dynatrace. Vous pouvez également utiliser votre propre solution capable de surveiller ou de détecter des défaillances au niveau de l’application ou de l’infrastructure virtuelle.

### <a name="step-3-update-the-cname-record"></a>Étape 3 : Mettre à jour l’enregistrement CNAME

Une fois la défaillance détectée, modifiez la valeur de l’enregistrement pour qu’elle pointe vers dr.contoso.com, comme illustré ci-dessous :
       
![Mise à jour de l’enregistrement CNAME](./media/disaster-recovery-dns-traffic-manager/update-cname-record.png)

*Figure - Mise à jour de l’enregistrement CNAME dans Azure*

Dans les 30 minutes, pendant lesquelles la plupart des programmes de résolution vont actualiser le fichier de zone mis en cache, toute requête sur www\.contoso.com sera redirigée vers dr.contoso.com.
Vous pouvez également exécuter la commande d’interface de ligne de commande Azure suivante pour modifier la valeur CNAME :
 ```azurecli
   az network dns record-set cname set-record \
   --resource-group 123 \
   --zone-name contoso.com \
   --record-set-name www \
   --cname dr.contoso.com
```
Cette étape peut être exécutée manuellement ou de manière automatique. Elle peut être effectuée manuellement via la console ou par l’interface de ligne de commande Azure. Le Kit de développement logiciel (SDK) et l’API Azure peuvent être utilisés pour automatiser la mise à jour du CNAME de sorte qu’aucune intervention manuelle ne soit nécessaire. L’automatisation peut être créée à l’aide de fonctions Azure ou au sein d’une application de surveillance tierce, ou encore en local.

### <a name="how-manual-failover-works-using-azure-dns"></a>Fonctionnement du basculement manuel à l’aide d’Azure DNS
Le serveur DNS étant situé en dehors de la zone de basculement ou d’incident, il est protégé d’un éventuel temps d’arrêt. Cela permet à l’utilisateur de concevoir un scénario de basculement simple et économique qui fonctionne en continu, en admettant que l’opérateur possède une connectivité réseau pendant l’incident et puisse effectuer le changement. Si la solution est basée sur un script, il faut s’assurer que le serveur ou service exécutant le script soit isolé du problème qui affecte l’environnement de production. Pensez également à mémoriser la faible durée de vie qui était définie sur la zone de sorte qu’aucun programme de résolution dans le monde ne garde longtemps le point de terminaison mis en cache et que les clients puissent accéder au site dans le RTO prévu. Pour les méthodes de reprise progressive et de veilleuse, une préparation et une autre activité administrative pouvant être nécessaires, il faut également prévoir suffisamment de temps pour effectuer le changement.

## <a name="automatic-failover-using-azure-traffic-manager"></a>Basculement automatique à l’aide d’Azure Traffic Manager
Lorsque vous avez des architectures complexes et plusieurs ensembles de ressources capables d’exécuter la même fonction, vous pouvez configurer Azure Traffic Manager (basé sur DNS) pour vérifier l’intégrité de vos ressources et acheminer le trafic de la ressource défectueuse vers la ressource saine. Dans l’exemple suivant, la région principale et la région secondaire ont un déploiement complet. Ce déploiement inclut les services cloud et une base de données synchronisée. 

![Basculement automatique à l’aide d’Azure Traffic Manager](./media/disaster-recovery-dns-traffic-manager/automatic-failover-using-traffic-manager.png)

*Figure - Basculement automatique à l’aide d’Azure Traffic Manager*

Toutefois, seule la région primaire traite activement les requêtes réseau des utilisateurs. La région secondaire devient active uniquement lorsque la région primaire subit une interruption de service. Dans ce cas, toutes les nouvelles requêtes réseau effectuent le routage vers la région secondaire. La sauvegarde de la base de données étant quasiment instantanée, les deux équilibreurs de charge ont des adresses IP dont l’intégrité peut être vérifiée, et les instances sont toujours opérationnelles. Cette topologie permet de choisir un RTO faible et un basculement sans intervention manuelle. La région de basculement secondaire doit être opérationnelle dès que la région primaire rencontre une défaillance.
Ce scénario est idéal pour l’utilisation d’Azure Traffic Manager qui dispose de sondes intégrées pour différents types de contrôles d’intégrité, y compris http / https et TCP. Azure Traffic Manager est également doté d’un moteur de règle qui peut être configuré de manière à basculer en cas de défaillance, comme décrit ci-dessous. Prenons la solution suivante utilisant Traffic Manager :
- Le client a le point de terminaison Région 1 appelé prod.contoso.com ayant l’adresse IP statique 100.168.124.44, et un point de terminaison Région 2 appelé dr.contoso.com ayant l’adresse IP statique 100.168.124.43. 
-   Chacun de ces environnements est exposé via une propriété d’accès public telle qu’un équilibreur de charge. L’équilibreur de charge peut être configuré de manière à avoir un point de terminaison DNS ou un nom de domaine complet (FQDN), comme indiqué ci-dessus.
-   Toutes les instances de la Région 2 sont répliquées quasiment en temps réel avec la Région 1. De plus, les images de machine sont à jour et toutes les données logicielles/de configuration sont corrigées et alignées avec la Région 1.  
-   La mise à l’échelle automatique est configurée à l’avance. 

La procédure pour configurer le basculement avec Azure Traffic Manager est la suivante :
1. Création d’un profil Azure Traffic Manager
2. Création des points de terminaison dans le profil Traffic Manager
3. Configuration du contrôle d’intégrité et vérification du basculement

### <a name="step-1-create-a-new-azure-traffic-manager-profile"></a>Étape 1 : Création d’un profil Azure Traffic Manager
Créez un profil Azure Traffic Manager avec le nom contoso123 et sélectionnez la méthode de routage Priorité. Si vous voulez associer un groupe de ressources préexistant, vous pouvez en sélectionner un ou encore créer un nouveau groupe de ressources.

![Créer un profil Traffic Manager](./media/disaster-recovery-dns-traffic-manager/create-traffic-manager-profile.png)

*Figure : Créer un profil Traffic Manager*

### <a name="step-2-create-endpoints-within-the-traffic-manager-profile"></a>Étape 2 : Création de points de terminaison dans le profil Traffic Manager

Au cours de cette étape, vous créez des points de terminaison qui pointent vers les sites de production et de récupération d’urgence. Ici, choisissez **Type** comme point de terminaison externe, mais si la ressource est hébergée dans Azure, vous pouvez également sélectionner **Point de terminaison Azure**. Si vous choisissez **Point de terminaison Azure**, sélectionnez une **Ressource cible** qui soit **Service d’application** ou **Adresse IP publique** allouée par Azure. La priorité est définie sur **1** car il s’agit du service principal pour la Région 1.
De la même manière, créez le point de terminaison de récupération d’urgence dans Traffic Manager.

![Création de points de terminaison de récupération d’urgence](./media/disaster-recovery-dns-traffic-manager/create-disaster-recovery-endpoint.png)

*Figure - Création de points de terminaison de récupération d’urgence*

### <a name="step-3-set-up-health-check-and-failover-configuration"></a>Étape 3 : Configuration du contrôle d’intégrité et vérification du basculement

Au cours de cette étape, vous définissez la durée de vie du DNS sur 10 secondes, une durée respectée par la plupart des programmes de résolution récursifs sur Internet. Cette configuration signifie qu’aucun programme de résolution DNS ne mettra en cache les informations pendant plus de 10 secondes. Pour les paramètres de surveillance du point de terminaison, le chemin d’accès est défini au / ou à la racine, mais vous pouvez personnaliser les paramètres du point de terminaison pour évaluer un chemin d’accès, par exemple, prod.contoso.com/index. Dans l’exemple ci-dessous, **https** est défini comme protocole de détection. Vous pouvez cependant choisir **http** ou **tcp**. Le choix du protocole dépend de l’application finale. L’intervalle de sondage est défini sur 10 secondes, ce qui permet une détection rapide. La nouvelle tentative est définie sur 3. Par conséquent, Traffic Manager bascule vers le deuxième point de terminaison si trois intervalles consécutifs enregistrent une défaillance. La formule suivante définit la durée totale d’un basculement automatique : Durée du basculement = Durée de vie + Nouvelle tentative * Intervalle de sondage. En l’occurrence, la valeur est 10 + 3 * 10 = 40 secondes (max).
Si Nouvelle tentative est défini sur 1 et Durée de vie sur 10 secondes, la durée du basculement sera 10 + 1 * 10 = 20 secondes. Définissez une valeur de Nouvelle tentative supérieure à **1** pour éliminer les risques de défaillances dues à des faux positifs ou à des spots réseau mineurs. 


![Configuration du contrôle d’intégrité](./media/disaster-recovery-dns-traffic-manager/set-up-health-check.png)

*Figure - Configuration du contrôle d’intégrité et vérification du basculement*

### <a name="how-automatic-failover-works-using-traffic-manager"></a>Fonctionnement du basculement automatique à l’aide d’Azure Traffic Manager

Lors d’un incident, le point de terminaison principal est interrogé et l’état passe à **détérioré**. Le site de récupération d’urgence reste **Online**. Par défaut, Traffic Manager envoie tout le trafic vers le point de terminaison principal (priorité la plus élevée). Si le point de terminaison principal apparaît détérioré, Traffic Manager achemine le trafic au deuxième point de terminaison tant qu’il reste sain. Il est possible de configurer davantage de points de terminaison dans Traffic Manager, qui peuvent servir de points de terminaison de basculement supplémentaires ou d’équilibreurs de charge qui partagent la charge entre les points de terminaison.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md).
- En savoir plus sur [Azure DNS](../dns/dns-overview.md)









