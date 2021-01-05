---
title: Concevoir des solutions résilientes avec Azure DDoS Protection
description: Découvrez comment utiliser les données de journalisation pour obtenir des informations détaillées sur votre application.
services: security
author: terrylanfear
manager: RKarlin
editor: TomSh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/18/2018
ms.author: terrylan
ms.openlocfilehash: e298cb0d1a2c510a096f8ead03f8af7e39c206a8
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498929"
---
# <a name="azure-ddos-protection---designing-resilient-solutions"></a>Azure DDoS Protection - Concevoir des solutions résilientes

Cet article est destiné aux décideurs informatiques et aux équipes de sécurité. Il sous-tend que vous connaissez Azure, la mise en réseau et la sécurité.
DDoS est un type d’attaque qui tente d’épuiser les ressources d’une application. Son objectif est d’affecter la disponibilité de l’application et sa capacité à gérer des demandes légitimes. Ces attaques de plus en plus sophistiquées gagnent en importance et en impact. Les attaques DDoS peuvent être ciblées sur n’importe quel point de terminaison qui est publiquement accessible via Internet. La conception d’applications capables de résister à des attaques par déni de service distribué (DDoS) nécessite un travail de planification et la prise en compte de divers modes d’échec. Azure fournit une protection contre les attaques DDoS. Cette protection est intégrée à la plateforme Azure par défaut sans frais additionnels.

Outre la protection DDoS de base sur la plateforme, [Azure DDoS Protection Standard](https://azure.microsoft.com/services/ddos-protection/) fournit des fonctions d’atténuation DDoS avancées contre les attaques réseau. Cette solution s’adapte automatiquement pour protéger vos ressources Azure spécifiques. Vous pouvez facilement activer cette protection durant la création de réseaux virtuels. Vous pouvez également l’activer après la création ; aucun changement au niveau de l’application ou de la ressource ne s’impose.

![Rôle d’Azure DDoS Protection dans la protection des clients et d’un réseau virtuel face à un attaquant](./media/ddos-best-practices/image1.png)


## <a name="fundamental-best-practices"></a>Bonnes pratiques de base

La section suivante fournit des conseils normatifs pour générer des services résistants aux attaques DDoS sur Azure.

### <a name="design-for-security"></a>Conception dans l’optique de la sécurité

Assurez-vous que la sécurité est une priorité tout au long du cycle de vie d’une application, de la conception et de l’implémentation au déploiement et aux opérations. Les applications peuvent contenir des bogues qui laissent passer un volume relativement faible de requêtes conçues pour utiliser une quantité anormale de ressources, provoquant une panne de service. 

Pour permettre de protéger un service fonctionnant sur Microsoft Azure, vous devez bien comprendre l’architecture de votre application et respecter les [cinq piliers de la qualité logicielle](/azure/architecture/guide/pillars).
Vous devez avoir connaissance des volumes de trafic habituels, du modèle de connectivité entre l’application et d’autres applications, et des points de terminaison de service exposés à l’Internet public.

Il est extrêmement important que vous conceviez une application suffisamment résiliente pour surmonter une attaque ciblée par déni de service. La plateforme Azure intègre des fonctionnalités de sécurité et de confidentialité, à commencer par [SDL (Security Development Lifecycle)](https://www.microsoft.com/sdl/default.aspx). SDL tient compte de la sécurité à chaque phase de développement et vérifie qu’Azure est continuellement mis à jour pour le rendre encore plus sécurisé.

### <a name="design-for-scalability"></a>Conception dans l’optique de la scalabilité

L’extensibilité correspond à la capacité d’un système à traiter une charge accrue. Concevez vos applications de sorte qu’elles puissent être [mises à l’échelle horizontalement](/azure/architecture/guide/design-principles/scale-out) pour répondre à la demande d’une charge amplifiée, en particulier dans le cadre d’une attaque DDoS. Si votre application repose sur une seule instance unique d’un service, cela crée un point de défaillance unique. L’approvisionnement de plusieurs instances rend votre système plus résilient et plus évolutif.

Pour [Azure App Service](../../app-service/overview.md), sélectionnez un [plan App Service](../../app-service/overview-hosting-plans.md) qui offre plusieurs instances. Pour les Azure Cloud Services, configurez chacun de vos rôles de manière à utiliser [plusieurs instances](../../cloud-services/cloud-services-choose-me.md). Pour [Machines virtuelles Azure](../../virtual-machines/index.yml), vérifiez que votre architecture de machines virtuelles inclut plusieurs machines virtuelles et que chacune d’elles figure dans un [groupe à haute disponibilité](../../virtual-machines/windows/tutorial-availability-sets.md). Nous vous recommandons d’utiliser des [groupes de machines virtuelles identiques](../../virtual-machine-scale-sets/overview.md) pour les fonctionnalités de mise à l’échelle automatique.

### <a name="defense-in-depth"></a>Défense en profondeur

L’idée derrière la défense en profondeur est de gérer les risques en utilisant des stratégies de protection variées. Le fait de superposer des défenses dans une application réduit les chances de réussite d’une attaque. Nous vous recommandons d’implémenter des conceptions sécurisées pour vos applications par le biais des fonctionnalités intégrées à la plateforme Azure.

Par exemple, plus la taille (*surface d’exposition*) de l’application est importante, plus le risque d’attaque est élevé. Vous pouvez réduire la surface d’exposition en créant une liste d’approbation permettant de limiter l’espace d’adressage IP exposé et les ports d’écoute qui ne sont pas nécessaires sur les équilibreurs de charge ([Azure Load Balancer](../../load-balancer/quickstart-load-balancer-standard-public-portal.md) et [Azure Application Gateway](../../application-gateway/application-gateway-create-probe-portal.md)). Les [groupes de sécurité réseau](../../virtual-network/network-security-groups-overview.md) permettent également de réduire la surface d’attaque.
Vous pouvez utiliser des [balises de service](../../virtual-network/network-security-groups-overview.md#service-tags) et des [groupes de sécurité d’application](../../virtual-network/network-security-groups-overview.md#application-security-groups) pour simplifier la création de règles de sécurité et configurer la sécurité réseau comme prolongement naturel de la structure d’une application.

Vous devez déployer les services Azure dans un [réseau virtuel](../../virtual-network/virtual-networks-overview.md) dans la mesure du possible. Les ressources de service peuvent ainsi communiquer par le biais d’adresses IP privées. Le trafic du service Azure à partir d’un réseau virtuel utilise des adresses IP publiques comme adresses IP source par défaut. Le fait d’utiliser des [points de terminaison de service](../../virtual-network/virtual-network-service-endpoints-overview.md) force le trafic de service à utiliser des adresses privées de réseau virtuel comme adresses IP source lors de l’accès au service Azure à partir d’un réseau virtuel.

Il arrive fréquemment que les ressources locales d’un client soient attaquées en même temps que ses ressources dans Azure. Si vous connectez un environnement local à Azure, nous vous recommandons de réduire l’exposition des ressources locales à l’Internet public. Pour bénéficier des fonctionnalités de mise à l’échelle et de protection DDoS avancée d’Azure, déployez vos entités publiques connues dans Azure. Ces entités accessibles publiquement étant souvent la cible d’attaques DDoS, le fait de les placer dans Azure réduit l’impact sur vos ressources locales.

## <a name="azure-offerings-for-ddos-protection"></a>Offres Azure pour la protection DDoS

Azure a deux offres de service DDoS qui fournissent une protection contre les attaques réseau (couche 3 et 4) : DDoS Protection Basic et DDoS Protection Standard. 

### <a name="ddos-protection-basic"></a>DDoS Protection Basic

La protection de base est intégrée à Azure par défaut sans frais additionnels. De par son échelle et sa capacité, le réseau Azure déployé à l’échelle mondiale assure une défense contre les attaques courantes de la couche réseau. Cette défense est par ailleurs renforcée par le monitoring continu du trafic et l’atténuation en temps réel. DDoS Protection Basic ne nécessite aucun changement de la part de l’utilisateur au niveau de la configuration ou de l’application. DDoS Protection Basic permet de protéger tous les services Azure, notamment les services PaaS comme Azure DNS.

![Mappage de la représentation du réseau Azure, avec le texte « Global DDoS mitigation presence » (Présence d’atténuation DDoS globale) et « Leading DDoS mitigation capacity » (Principale capacité d’atténuation DDoS)](./media/ddos-best-practices/image3.png)

Dans Azure, la protection DDoS de base comprend des composants logiciels et matériels. Un plan de contrôle logiciel détermine le type de trafic acheminé vers des appliances matérielles qui analysent et suppriment le trafic d’attaque, le moment où l’acheminent a lieu ainsi que l’emplacement. Le plan de contrôle prend cette décision en fonction d’une *stratégie* de protection DDoS à l’échelle de l’infrastructure. Cette stratégie est définie de manière statique et appliquée universellement à tous les clients Azure.

Par exemple, la stratégie de protection DDoS indique le volume de trafic auquel la protection doit être *déclenchée.* (Autrement dit, le trafic du locataire doit être acheminé via des appliances de lecture à vitesse variable.) La stratégie spécifie ensuite comment les appliances de lecture à vitesse variable doivent *atténuer* l’attaque.

Le service Azure DDoS Protection Basic a pour but de protéger l’infrastructure et la plateforme Azure. Il atténue le trafic quand celui-ci dépasse un débit important susceptible d’impacter plusieurs clients dans un environnement multi-locataire. Il ne prend en charge ni les alertes ni les stratégies personnalisées par client.

### <a name="ddos-protection-standard"></a>DDoS Protection Standard

La protection standard fournit des fonctionnalités d’atténuation DDoS améliorées. Cette solution s’adapte automatiquement pour protéger vos ressources Azure spécifiques dans un réseau virtuel. La protection est simple à activer sur n’importe quel réseau virtuel, nouveau ou existant, et ne nécessite aucun changement au niveau de l’application ou des ressources. Elle offre plusieurs avantages par rapport au service de base, notamment la journalisation, la création d’alertes et la télémétrie. Les sections suivantes décrivent les principales fonctionnalités du service Azure DDoS Protection Standard.

#### <a name="adaptive-real-time-tuning"></a>Réglage adaptatif en temps réel

Le service Azure DDoS Protection Basic contribue à protéger les clients et évite d’impacter d’autres clients. Par exemple, si un service est provisionné pour un volume de trafic entrant légitime inférieur au *taux de déclenchement* de la stratégie de protection DDoS à l’échelle de l’infrastructure, une attaque DDoS sur les ressources du client peut passer inaperçue. De manière plus générale, la complexité des récentes attaques (comme les attaques DDoS multivectorielles) et les comportements spécifiques aux applications des locataires demandent des stratégies de protection personnalisées par client. Le service effectue cette personnalisation en s’appuyant sur deux éléments :

- L’apprentissage automatique des modèles de trafic par client (par adresse IP) pour les couches 3 et 4.

- La réduction des faux positifs dans la mesure où l’échelle d’Azure permet d’absorber une quantité importante de trafic.

![Schéma du fonctionnement de DDoS Protection Standard, avec la génération de stratégie entourée](./media/ddos-best-practices/image5.png)

#### <a name="ddos-protection-telemetry-monitoring-and-alerting"></a>Protection DDoS : télémétrie, monitoring et génération d’alertes

DDoS Protection Standard expose des données de télémétrie riches par le biais [d’Azure Monitor](../../azure-monitor/overview.md) pendant la durée d’une attaque DDoS. Vous pouvez configurer des alertes pour une des mesures d’Azure Monitor utilisée par DDoS Protection. Vous pouvez intégrer la journalisation à Splunk (Azure Event Hubs), Journaux Azure Monitor et Stockage Azure pour l’analyse avancée via l’interface des diagnostics d’Azure Monitor.

##### <a name="ddos-mitigation-policies"></a>Stratégies d’atténuation des risques liés à DDoS

Dans le portail Azure, cliquez sur **Surveiller** > **Métriques**. Dans le volet **Métriques**, sélectionnez le groupe de ressources, le type de ressource **Adresse IP publique** et votre adresse IP publique Azure. Les métriques DDoS sont visibles dans le volet **Métriques disponibles**.

DDoS Protection Standard applique trois stratégies de prévention réglées automatiquement (TCP SYN, TCP et UDP) pour chaque adresse IP publique de la ressource protégée, dans le réseau virtuel sur lequel la protection DDoS est activée. Vous pouvez afficher les seuils de stratégie en sélectionnant la métrique **Paquets entrants pour déclencher l’atténuation DDoS**.

![Métriques disponibles et graphique des métriques](./media/ddos-best-practices/image7.png)

Les seuils de stratégie sont configurés automatiquement par le biais du système de profilage du trafic réseau basé sur l’apprentissage automatique. L’atténuation des risques liés à DDoS pour une adresse IP n’a lieu que si le seuil de stratégie est dépassé.

##### <a name="metric-for-an-ip-address-under-ddos-attack"></a>Métrique d’une adresse IP faisant l’objet d’une attaque DDoS

Si l’adresse IP publique est attaquée, la valeur de la métrique **Sous attaque DDoS ou non** passe à 1 à mesure que DDoS Protection effectue les opérations d’atténuation sur le trafic de l’attaque.

![Métrique « Sous attaque DDoS ou non » et graphique](./media/ddos-best-practices/image8.png)

Nous vous recommandons de configurer une alerte sur cette métrique. Vous serez alors averti en cas d’atténuation DDoS active sur votre adresse IP publique.

Pour plus d’informations, consultez [Gérer Azure DDoS Protection Standard à l’aide du portail Azure](../../ddos-protection/manage-ddos-protection.md).

#### <a name="web-application-firewall-for-resource-attacks"></a>Pare-feu d’applications web pour les attaques sur les ressources

Pour lutter spécifiquement contre les attaques sur les ressources au niveau de la couche application et renforcer la sécurité des applications web, vous devez configurer le pare-feu d’applications web (WAF). WAF inspecte le trafic web entrant pour bloquer les injections SQL, les scripts intersites, les attaques DDoS et autres attaques au niveau de la couche 7. [WAF est une fonctionnalité d’Application Gateway](../../web-application-firewall/ag/ag-overview.md) fournie par Azure pour protéger de manière centralisée vos applications web contre les vulnérabilités et exploits courants. Les partenaires Azure proposent d’autres offres WAF. Pour trouver celle la mieux adaptée à vos besoins, visitez la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps?search=WAF&page=1).

Même les pare-feu d’application web sont vulnérables aux attaques volumétriques et d’épuisement d’état. Nous vous recommandons fortement d’activer DDoS Protection Standard sur le réseau virtuel WAF pour contribuer à vous protéger contre les attaques volumétriques et de protocole. Pour plus d’informations, consultez la section sur [les architectures de référence DDoS Protection](#ddos-protection-reference-architectures).

### <a name="protection-planning"></a>Planification de la protection

La planification et la préparation sont des étapes cruciales pour comprendre comment un système se comporte en cas d’attaque DDoS. La conception d’un plan de réponse de gestion des incidents fait partie de cet effort.

Si vous disposez de DDoS Protection Standard, assurez-vous qu’il est activé sur le réseau virtuel des points de terminaison accessibles sur Internet. La configuration d’alertes DDoS vous permet d’être en permanence attentif aux attaques potentielles sur votre infrastructure. 

Surveillez vos applications de manière indépendante. Vous devez comprendre le comportement normal d’une application. Préparez-vous à intervenir si l’application ne se comporte pas comme prévu durant une attaque DDoS.

#### <a name="testing-through-simulations"></a>Test à l’aide de simulations

Nous vous conseillons de tester vos hypothèses sur la façon dont vos services répondront à une attaque au moyen de simulations périodiques. Ces tests permettent de vérifier que vos services ou applications continuent à fonctionner comme prévu et que l’expérience des utilisateurs n’est pas interrompue. Identifiez les lacunes du point de vue de la technologie et du processus, et incorporez-les dans la stratégie de réponse DDoS. Nous vous recommandons d’effectuer ces tests dans des environnements intermédiaires ou durant les heures creuses afin de minimiser l’impact sur l’environnement de production.

Nous travaillons en collaboration avec [BreakingPoint Cloud](https://www.ixiacom.com/products/breakingpoint-cloud) pour créer une interface à l’aide de laquelle les clients Azure peuvent générer du trafic sur les points de terminaison publics dotés de DDoS Protection à des fins de simulation. Vous pouvez utiliser la simulation [BreakingPoint Cloud](https://www.ixiacom.com/products/breakingpoint-cloud) pour :

- Valider la façon dont Azure DDoS Protection contribue à protéger vos ressources Azure contre les attaques DDoS.

- Optimiser votre processus de réponse aux incidents en cas d’attaque DDoS.

- Documenter la conformité DDoS.

- Former des équipes de sécurité réseau.

La cybersécurité nécessite une innovation constante en matière de défense. Azure DDoS Protection Standard est une solution de pointe qui permet aux clients d’atténuer de manière efficace les attaques DDoS de plus en plus complexes.

## <a name="components-of-a-ddos-response-strategy"></a>Composants d’une stratégie de réponse DDoS

Une attaque DDoS ciblant des ressources Azure nécessite généralement une intervention minimale de l’utilisateur. Néanmoins, le fait d’incorporer l’atténuation des risques liés à DDoS à la stratégie de réponse aux incidents aide à limiter l’impact sur l’activité.

### <a name="microsoft-threat-intelligence"></a>Microsoft Threat Intelligence

Microsoft dispose d’un réseau complet d’informations sur les menaces. Celui-ci s’appuie sur les connaissances d’une communauté de spécialistes de la sécurité prenant en charge les services en ligne de Microsoft, sur les partenaires Microsoft et sur les relations au sein de la communauté dédiée à la sécurité d’Internet. 

En tant que fournisseur d’infrastructures critiques, Microsoft reçoit à l’avance des avertissements sur les menaces. Microsoft collecte des informations sur les menaces à partir de ses services en ligne et de sa base client globale. Microsoft incorpore tous ces renseignements sur les menaces dans les produits Azure DDoS Protection.

En outre, Microsoft Digital Crimes Unit (DCU) met en place des stratégies offensives contre les botnets. Les botnets constituent une source courante de commande et de contrôle pour les attaques DDoS.

### <a name="risk-evaluation-of-your-azure-resources"></a>Évaluation des risques de vos ressources Azure

Il est impératif de comprendre en permanence la portée des risques liés à une attaque DDoS. Demandez-vous régulièrement :

- Quelles nouvelles ressources Azure disponibles publiquement doivent être protégées ?

- Existe-t-il un point de défaillance unique dans le service ? 

- Comment faire pour isoler les services de manière à limiter l’impact d’une attaque tout en les rendant accessibles aux clients valides ?

- Y a-t-il des réseaux virtuels dans lesquels le service DDoS Protection Standard n’est pas activé alors qu’il devrait l’être ? 

- Mes services sont-ils actifs/actifs avec basculement dans plusieurs régions ?

### <a name="customer-ddos-response-team"></a>Équipe de réponse aux attaques DDoS du client

La création d’une équipe de réponse DDoS est essentielle pour garantir une réponse rapide et efficace en cas d’attaque. Dans votre organisation, identifiez plusieurs contacts qui seront en charge de la planification et de l’exécution. Cette équipe de réponse DDoS doit comprendre en détail le service Azure DDoS Protection Standard. Assurez-vous que l’équipe est en mesure d’identifier et d’atténuer une attaque en coordination avec les clients internes et externes, y compris l’équipe de support Microsoft.

Pour votre équipe de réponse DDoS, nous vous recommandons d’utiliser des exercices de simulation comme composant normal de la planification de la continuité et de la disponibilité de votre service. Ces exercices doivent inclure des tests d’échelle.

### <a name="alerts-during-an-attack"></a>Alertes durant une attaque

Azure DDoS Protection Standard identifie et atténue les attaques DDoS sans aucune intervention de l’utilisateur. Pour être averti en cas d’atténuation active pour une adresse IP publique protégée, vous pouvez [configurer une alerte](../../ddos-protection/manage-ddos-protection.md) sur la métrique **Sous attaque DDoS ou non**. Vous pouvez choisir de créer des alertes pour les autres métriques DDoS afin de comprendre l’échelle des attaques, le trafic abandonné, etc.

#### <a name="when-to-contact-microsoft-support"></a>Quand contacter le support Microsoft

- Au cours d’une attaque DDoS, vous constatez que les performances de la ressource protégée sont fortement dégradées ou que la ressource n’est pas disponible.

- Vous pensez que le service DDoS Protection ne se comporte pas comme prévu. 

  Le service DDoS Protection déclenche l’atténuation uniquement si la valeur de la métrique **Stratégie pour déclencher l’atténuation DDoS (TCP/TCP SYN/UDP)** est inférieure au trafic reçu sur la ressource IP publique protégée.

- Vous planifiez un événement viral qui peut accroître considérablement votre trafic réseau.

- Un acteur menace de lancer une attaque DDoS sur vos ressources.

- Si vous devez autoriser la liste d’une adresse IP ou d’une plage d’adresses IP depuis une norme Azure DDoS Protection. Un scénario courant consiste à autoriser une liste d’adresses IP si le trafic est acheminé à partir d’un pare-feu d’applications web de cloud externe vers Azure. 

Pour les attaques ayant un impact critique sur l’entreprise, créez un [ticket de support](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) avec un niveau de gravité A.

### <a name="post-attack-steps"></a>Étapes à suivre après l’attaque

Il est toujours bon d’effectuer un compte-rendu après une attaque et d’ajuster la stratégie de réponse DDoS au besoin. Points importants à prendre en compte :

- Le manque d’extensibilité de l’architecture a-t-il perturbé le service ou l’expérience utilisateur ?

- Quels sont les services ou les applications qui ont souffert le plus ?

- Quel a été le degré d’efficacité de la stratégie de réponse DDoS et comment peut-il être amélioré ?

Si vous pensez que vous faites l’objet d’une attaque DDoS, transmettez le problème aux canaux de support Azure habituels.

## <a name="ddos-protection-reference-architectures"></a>Architectures de référence de la protection DDoS

DDoS Protection Standard est conçu [pour les services déployés dans un réseau virtuel](../../virtual-network/virtual-network-for-azure-services.md). Pour les autres services, le service DDoS Protection Basic par défaut s’applique. Les architectures de référence suivantes sont organisées par scénarios, avec regroupement des modèles d’architecture.

### <a name="virtual-machine-windowslinux-workloads"></a>Charges de travail de machines virtuelles (Windows/Linux)

#### <a name="application-running-on-load-balanced-vms"></a>Application exécutée sur des machines virtuelles à charge équilibrée

Cette architecture de référence présente un ensemble de pratiques éprouvées pour exécuter plusieurs machines virtuelles Windows dans un groupe identique derrière un équilibreur de charge, afin d’améliorer la disponibilité et l’extensibilité. Cette architecture convient à n’importe quelle charge de travail sans état, par exemple un serveur web.

![Schéma de l’architecture de référence pour une application en cours d’exécution sur des machines virtuelles à charge équilibrée](./media/ddos-best-practices/image9.png)

Dans cette architecture, une charge de travail est répartie entre plusieurs instances de machine virtuelle. Il existe une seule adresse IP publique, et le trafic Internet est réparti entre les machines virtuelles à l’aide d’un équilibreur de charge. DDoS Protection Standard est activé sur le réseau virtuel de l’équilibreur de charge Azure (Internet) associé à l’adresse IP publique.

L’équilibreur de charge distribue les demandes Internet entrantes aux instances de machine virtuelle. Les groupes de machines virtuelles identiques permettent d’augmenter ou de réduire le nombre de machines virtuelles manuellement ou automatiquement en fonction de règles prédéfinies. Ceci est important si la ressource fait l’objet d’une attaque DDoS. Pour plus d’informations sur cette architecture de référence, consultez [cet article](/azure/architecture/reference-architectures/virtual-machines-windows/multi-vm).

#### <a name="application-running-on-windows-n-tier"></a>Application s’exécutant sur une architecture Windows multiniveau

Il existe de nombreuses façons d’implémenter une architecture multiniveau. Le schéma suivant présente une application web à trois niveaux standard. Cette architecture repose sur celle décrite dans l’article [Exécuter des machines virtuelles à charge équilibrée pour la scalabilité et la disponibilité](/azure/architecture/reference-architectures/virtual-machines-windows/multi-vm). Les niveaux Web et Business utilisent des machines virtuelles à charge équilibrée.

![Schéma de l’architecture de référence pour une application en cours d’exécution sur une architecture Windows multiniveau](./media/ddos-best-practices/image10.png)

Dans cette architecture, DDoS Protection Standard est activé sur le réseau virtuel. Toutes les adresses IP publiques dans le réseau virtuel bénéficient d’une protection DDoS au niveau des couches 3 et 4. Pour protéger la couche 7, déployez Application Gateway dans la référence SKU WAF. Pour plus d’informations sur cette architecture de référence, consultez [cet article](/azure/architecture/reference-architectures/virtual-machines-windows/n-tier).

#### <a name="paas-web-application"></a>Application web PaaS

Cette architecture de référence montre l’exécution d’une application Azure App Service dans une seule région. Cette architecture présente un ensemble de pratiques éprouvées pour une application web utilisant [Azure App Service](https://azure.microsoft.com/documentation/services/app-service/) et [Azure SQL Database](https://azure.microsoft.com/documentation/services/sql-database/).
La région de secours est configurée pour les scénarios de basculement.

![Schéma de l’architecture de référence pour une application web PaaS](./media/ddos-best-practices/image11.png)

Azure Traffic Manager achemine les requêtes entrantes à Application Gateway dans l’une des régions. Pendant le fonctionnement normal, il achemine les requêtes à Application Gateway dans la région active. Si cette région n’est plus disponible, Traffic Manager bascule sur Application Gateway dans la région de secours.

Tout le trafic provenant d’Internet à destination de l’application web est acheminé à [l’adresse IP publique d’Application Gateway](../../application-gateway/application-gateway-web-app-overview.md) par le biais de Traffic Manager. Dans ce scénario, App Service (application web) n’est pas directement accessible de l’extérieur et est protégé par Application Gateway. 

Nous vous recommandons de configurer la référence SKU WAF Application Gateway (mode prévention) pour mettre en place une protection contre les attaques de couche 7 (HTTP/HTTPS/Web Socket). Par ailleurs, les applications web sont configurées pour [accepter uniquement le trafic provenant de l’adresse IP d’Application Gateway](https://azure.microsoft.com/blog/ip-and-domain-restrictions-for-windows-azure-web-sites/).

Pour plus d’informations sur cette architecture de référence, consultez [cet article](/azure/architecture/reference-architectures/app-service-web-app/multi-region).

### <a name="mitigation-for-non-web-paas-services"></a>Atténuation pour les services PaaS non web

#### <a name="hdinsight-on-azure"></a>HDInsight sur Azure

Cette architecture de référence montre la configuration de DDoS Protection Standard pour un [cluster Azure HDInsight](../../hdinsight/index.yml). Assurez-vous que le cluster HDInsight est lié à un réseau virtuel et que DDoS Protection est activé sur ce réseau virtuel.

![Volets « HDInsight » et « Paramètres avancés », avec les paramètres de réseau virtuel](./media/ddos-best-practices/image12.png)

![Sélection pour activer DDoS Protection](./media/ddos-best-practices/image13.png)

Dans cette architecture, le trafic destiné au cluster HDInsight en provenance d’Internet est acheminé à l’adresse IP publique associée à l’équilibreur de charge de la passerelle HDInsight. L’équilibreur de charge de la passerelle envoie ensuite directement le trafic aux nœuds principaux ou aux nœuds worker. Dans la mesure où DDoS Protection Standard est activé sur le réseau virtuel HDInsight, toutes les adresses IP publiques dans le réseau virtuel bénéficient d’une protection DDoS au niveau des couches 3 et 4. Cette architecture de référence peut être combinée avec les architectures de référence multiniveau/multirégion.

Pour plus d’informations sur cette architecture de référence, consultez la documentation [Étendre HDInsight à l’aide d’un réseau virtuel Azure](../../hdinsight/hdinsight-plan-virtual-network-deployment.md?toc=%2fazure%2fvirtual-network%2ftoc.json).


> [!NOTE]
> Azure App Service Environment pour PowerApps ou la gestion des API dans un réseau virtuel avec une adresse IP publique ne sont pas pris en charge en mode natif.

## <a name="next-steps"></a>Étapes suivantes

* [Responsabilité partagée dans le cloud](shared-responsibility.md)
* [Azure DDoS Protection : page produit](https://azure.microsoft.com/services/ddos-protection/)
* [Documentation sur Azure DDoS Protection](../../ddos-protection/ddos-protection-overview.md)