---
title: Base de référence de sécurité Azure pour Azure Kubernetes Service
description: La base de référence de sécurité Azure Kubernetes Service fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans Azure Security Benchmark.
author: msmbaldwin
ms.service: container-service
ms.topic: conceptual
ms.date: 10/01/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 29bcf2fdd3e20b28b1f2cb014137cfba37ccaf2e
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96931807"
---
# <a name="azure-security-baseline-for-azure-kubernetes-service"></a>Base de référence de sécurité Azure pour Azure Kubernetes Service

La base de référence de sécurité Azure pour Azure Kubernetes Service contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Conseils** : Par défaut, un groupe de sécurité réseau et une table de routage sont créés automatiquement lors de la création d’un cluster Microsoft Azure Kubernetes Service (AKS). AKS modifie automatiquement les groupes de sécurité réseau afin que le trafic transite de manière appropriée lorsque des services sont créés avec des équilibreurs de charge, des mappages de ports ou des routes d’entrée. Le groupe de sécurité réseau est associé automatiquement aux cartes réseau virtuelles sur les nœuds clients et à la table de routage avec le sous-réseau sur le réseau virtuel. 

Utilisez des stratégies de réseau AKS pour limiter le trafic réseau en définissant des règles pour le trafic entrant et sortant entre les pods Linux d’un cluster en fonction du choix des espaces de noms et des sélecteurs d’étiquettes. L’utilisation de la stratégie réseau requiert le plug-in Azure CNI avec un réseau virtuel et des sous-réseaux définis, et ne peut être activée qu’au moment de la création du cluster. Ils ne peuvent pas être déployés sur un cluster AKS existant.

Vous pouvez implémenter un cluster AKS privé pour vous assurer que le trafic réseau entre votre serveur d’API AKS et les pools de nœuds restent uniquement sur le réseau privé. Le plan de contrôle ou le serveur d’API se trouve dans un abonnement Azure géré par AKS et utilise des adresses IP internes (RFC1918), tandis que le pool de clusters ou de nœuds du client se trouve dans son propre abonnement. Le serveur et le cluster ou le pool de nœuds communiquent entre eux à l’aide du service Azure Private Link dans le réseau virtuel du serveur d’API et d’un point de terminaison privé exposé dans le sous-réseau du cluster AKS du client.  Vous pouvez également utiliser un point de terminaison public pour le serveur d’API AKS, mais limitez l’accès à l’aide de la fonctionnalité de plages d’adresses IP autorisées du serveur d’API AKS. 

- [Concepts de sécurité pour les applications et les clusters dans AKS (Azure Kubernetes Service)](concepts-security.md)

- [Sécuriser le trafic entre les pods avec des stratégies réseau dans Azure Kubernetes Service (AKS)](use-network-policies.md)

- [Créer un cluster Azure Kubernetes Service privé](private-clusters.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : Utilisez Security Center et suivez ses recommandations de protection réseau pour sécuriser les ressources réseau utilisées par vos clusters Azure Kubernetes Service (AKS). 

Activez les journaux de flux de groupe de sécurité réseau et envoyez ceux-ci à un compte Stockage Azure à des fins d’audit. Vous pouvez également envoyer les journaux de flux à un espace de travail Log Analytics, puis utiliser Traffic Analytics pour fournir des informations sur les modèles de trafic dans votre cloud Azure afin de visualiser l’activité du réseau, identifier les zones réactives et les menaces de sécurité, comprendre les modèles de flux de trafic et identifier les problèmes de mauvaise configuration du réseau.

- [Présentation de la sécurité réseau assurée par Azure Security Center](../security-center/security-center-network-recommendations.md)

- [Guide pratique pour activer les journaux des flux de groupe de sécurité réseau](../network-watcher/network-watcher-nsg-flow-logging-portal.md)

- [Guide pratique pour activer et utiliser Traffic Analytics](../network-watcher/traffic-analytics.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Utilisez un Web Application Firewall (WAF) prenant en charge Azure Application Gateway devant un cluster AKS pour fournir une couche supplémentaire de sécurité en filtrant le trafic entrant sur vos applications web. Azure WAF utilise un ensemble de règles, fourni par The Open Web Application Security Project (OWASP), pour les attaques telles que les scripts inter-sites ou le « cookie poisoning » contre ce trafic. 

Utilisez une passerelle d’API pour l’authentification, l’autorisation, la limitation, la mise en cache, la transformation et la surveillance des API utilisées dans votre environnement AKS. Une passerelle API sert de porte d’entrée pour les microservices, dissocie les clients de vos microservices et réduit la complexité de vos microservices en supprimant la charge liée à la gestion des problèmes communs.

- [Comprendre les meilleures pratiques relatives à la connectivité réseau et à la sécurité dans AKS](operator-best-practices-network.md)

- [Contrôleur d’entrée Application Gateway ](../application-gateway/ingress-controller-overview.md)

- [Utiliser Gestion des API Azure avec des microservices déployés dans Azure Kubernetes Service](../api-management/api-management-kubernetes.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Conseils** : Activez la protection standard contre l’attaque par déni de service (DDoS) Microsoft sur les réseaux virtuels où des composants Azure Kubernetes Service (AKS) sont déployés pour obtenir des protections contre les attaques DDoS.
Installez le moteur de stratégie réseau et créez des stratégies réseau Kubernetes pour contrôler le flux du trafic entre les pods dans AKS puisque, par défaut, tout le trafic est autorisé entre ces pods. Une stratégie réseau doit uniquement être utilisée pour les nœuds et pods Linux dans AKS. Définissez des règles qui limitent la communication des pods pour une sécurité renforcée. 

Choisissez d’autoriser ou de refuser un trafic en fonction de paramètres, tels que des étiquettes attribuées, un espace de noms ou un port de trafic. Les stratégies réseau nécessaires peuvent être appliquées automatiquement car les pods sont créés de façon dynamique dans un cluster AKS. 

- [Sécuriser le trafic entre les pods avec des stratégies réseau dans Azure Kubernetes Service (AKS)](use-network-policies.md)

- [Guide pratique pour configurer la protection DDoS](../ddos-protection/manage-ddos-protection.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : Utilisez la capture de paquets Network Watcher si cela s’avère nécessaire pour analyser une activité anormale. 

Network Watcher est activé automatiquement dans la région de votre réseau virtuel lorsque vous créez ou mettez à jour un réseau virtuel dans votre abonnement. Vous pouvez également créer des instances de Network Watcher à l’aide de PowerShell, d’Azure CLI, de l’API REST ou de la méthode ARMClient

- [Guide pratique pour activer Network Watcher](../network-watcher/network-watcher-create.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS) basés sur le réseau

**Conseils** : Sécurisez votre cluster Azure Kubernetes Service (AKS) avec une Azure Application Gateway activée avec un Web Application Firewall (WAF). 

Si la détection des intrusions et/ou la prévention basée sur l’inspection de la charge utile ou l’analyse du comportement n’est pas obligatoire, une Azure Application Gateway avec WAF peut être utilisée et configurée en « mode de détection » pour consigner les alertes et les menaces ou en « mode de prévention » pour bloquer activement les intrusions et les attaques détectées.

- [Comprendre les meilleures pratiques de sécurisation de votre cluster AKS avec un WAF](operator-best-practices-network.md#secure-traffic-with-a-web-application-firewall-waf)

- [Guide pratique de déploiement d’Azure Application Gateway (Azure WAF)](../web-application-firewall/ag/application-gateway-web-application-firewall-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Utilisez un Web Application Firewall (WAF) prenant en charge Azure Application Gateway devant un cluster AKS pour filtrer le trafic entrant. Le projet OWASP (Open Web Application Security Project) propose un ensemble de règles qui sont utilisées dans Azure AWF pour surveiller les attaques de type script inter-sites ou cookie poisoning.

Appliquez des étiquettes de nom de domaine complet (FQDN) aux applications afin de simplifier leur utilisation dans la configuration de règles d’application au sein d’un groupe de sécurité réseau. Après la configuration des règles réseau. Ajoutez une règle d’application utilisant une étiquette de FQDN, par exemple AzureKubernetesService, qui comprend tous les FQDN requis accessibles via le port TCP 443 et le port 80. 

- [Comprendre les meilleures pratiques relatives à la connectivité réseau et à la sécurité dans AKS](operator-best-practices-network.md)

- [Sécuriser le trafic entre les pods avec des stratégies réseau dans Azure Kubernetes Service (AKS)](use-network-policies.md)

- [Guide pratique de déploiement d’Azure Application Gateway (Azure WAF)](../web-application-firewall/ag/application-gateway-web-application-firewall-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : Utilisez des balises de service de réseau virtuel pour définir des contrôles d’accès réseau sur les groupes de sécurité réseau associés à des instances Azure Kubernetes Service (AKS). Des balises de service peuvent être utilisées à la place d’adresses IP spécifiques lors de la création de règles de sécurité pour autoriser ou refuser le trafic du service correspondant en spécifiant le nom de l’étiquette de service. 

Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Appliquez une balise Azure à des pools de nœuds de votre cluster AKS. Elles sont différentes des balises de service de réseau virtuel, s’appliquent à chaque nœud du pool de nœuds et sont conservées dans les mises à niveau. 

- [Présentation et usage des balises de service](../virtual-network/service-tags-overview.md)

- [Présentation des NSG pour AKS](support-policies.md)

- [Contrôler le trafic de sortie pour les nœuds de cluster dans Azure Kubernetes Service (AKS)](limit-egress-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Définissez et implémentez des configurations de sécurité standard avec Azure Policy pour les ressources réseau associées à vos clusters Azure Kubernetes Service (AKS). Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.ContainerService » et « Microsoft.Network » pour créer des stratégies personnalisées d’audit ou d’application de la configuration réseau de vos clusters AKS. 

Utilisez également les définitions de stratégie intégrées associées à AKS, telles que :

•   Des plages d’adresses IP autorisées doivent être définies sur les services Kubernetes

•   Appliquer une entrée HTTPS dans un cluster Kubernetes

•   Garantir que les services écoutent uniquement sur les ports autorisés dans un cluster Kubernetes

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Exemples Azure Policy pour le réseau](../governance/policy/samples/built-in-policies.md#network)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Utilisez des étiquettes pour les groupes de sécurité réseau et d’autres ressources pour le trafic entrant et sortant de clusters Azure Kubernetes Service (AKS). Utilisez le champ « Description » des règles de groupe de sécurité réseau individuelles afin de spécifier le besoin métier et/ou la durée, etc., pour toutes les règles qui autorisent le trafic vers et depuis un réseau.
Utilisez l’une des définitions en lien avec l’étiquetage Azure Policy intégrées, par exemple « Exiger une étiquette et sa valeur », qui garantissent que toutes les ressources sont créées avec des étiquettes et pour recevoir des notifications sur l’existence de ressources non étiquetées.

Choisissez d’autoriser ou de refuser des chemins d’accès réseau spécifiques au sein du cluster en fonction des espaces de noms et des sélecteurs d’étiquettes avec des stratégies de réseau. Utilisez ces espaces de noms et étiquettes comme des descripteurs des règles de configuration du trafic. Utilisez Azure PowerShell ou l’interface de ligne de commande (CLI) Azure pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

- [Azure Policy avec CLI](/cli/azure/policy)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

- [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les changements des ressources réseau associées aux clusters Azure Kubernetes Service (AKS). 

Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques. Toutes les entrées de l’utilisateur AzureContainerService dans les journaux d’activité sont consignées comme des actions de plateforme. 

Utilisez les journaux Azure Monitor pour activer et interroger les journaux des principaux composants d’AKS, kube-apiserver et kube-controller-manager. Créez et gérez les nœuds qui exécutent le kubelet et le runtime de conteneurs, et déployez leurs applications par le biais du serveur d’API de Kubernetes managé. 

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

- [Activer et consulter les journaux d'activité du nœud principal Kubernetes dans Azure Kubernetes Service (AKS)](view-master-logs.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : Les nœuds Azure Kubernetes Service (AKS) utilisent ntp.ubuntu.com pour la synchronisation de l’heure, ainsi que le port UDP 123 et le protocole NTP (Network Time Protocol). 

Vérifiez que les serveurs NTP sont accessibles par les nœuds de cluster si vous utilisez des serveurs DNS personnalisés. 

- [Présentation des exigences relatives au domaine et au port NTP pour les nœuds de cluster AKS](limit-egress-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Activez les journaux d’audit des principaux composants d’Azure Kubernetes Service (AKS), kube-apiserver et kube-controller-manager, qui sont fournis sous la forme d’un service géré. 

•   kube-auditaksService : Nom d’affichage dans le journal d’audit de l’opération de plan de contrôle (à partir de hcpService) 

•   masterclient : Nom d’affichage dans le journal d’audit pour MasterClientCertificate, le certificat obtenu à partir de la commande az aks get-credentials 

•   nodeclient : Nom d’affichage du ClientCertificate utilisé par les nœuds d’agent

Activez également d’autres journaux d’audit, tels que kube-audit. 

Exportez ces journaux vers Log Analytics ou une autre plateforme de stockage. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et utilisez des comptes Stockage Azure pour le stockage à long terme et l’archivage.

Activez ces données et intégrez-les à Azure Sentinel ou à une solution SIEM tierce, en fonction des besoins de votre organisation.

- [Passez en revue le schéma de journal, y compris les rôles de journal, ici](view-master-logs.md)

- [Présentation d’Azure Monitor pour conteneurs](../azure-monitor/insights/container-insights-overview.md)

- [Procédure d’activation d’Azure Monitor pour conteneurs](../azure-monitor/insights/container-insights-onboard.md)

- [Activer et consulter les journaux d'activité du nœud principal Kubernetes dans Azure Kubernetes Service (AKS)](view-master-logs.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Utilisez les journaux d’activité pour surveiller les actions sur les ressources Azure Kubernetes Service (AKS) afin d’afficher l’ensemble de l’activité et des états. Déterminez les opérations qui ont été effectuées sur les ressources de votre abonnement avec les journaux d’activité : Qui a démarré l’opération

Le moment où a eu lieu l’opération

L’état de l’opération

Les valeurs d’autres propriétés qui peuvent vous aider à effectuer des recherches sur l’opération

Récupérez les informations du journal d'activité par le biais d'Azure PowerShell, de l'interface de ligne de commande (CLI) Azure, de l'API REST Azure ou du portail Azure. 

Activez les journaux d’audit sur les principaux composants d’AKS, tels que : 

•   kube-auditaksService : Nom d’affichage dans le journal d’audit de l’opération de plan de contrôle (à partir de hcpService) 

•   masterclient : Nom d’affichage dans le journal d’audit pour MasterClientCertificate, le certificat obtenu à partir de la commande az aks get-credentials 

•   nodeclient : Nom d’affichage du ClientCertificate utilisé par les nœuds d’agent

Activez également d’autres journaux d’audit, tels que kube-audit. 

- [Procédure d’activation et de consultation des journaux d’activité du nœud principal Kubernetes dans AKS](view-master-logs.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils** : Activez l’installation automatique d’agents Log Analytics pour la collecte de données à partir des nœuds de cluster AKS. Activez également l’approvisionnement automatique de l’agent de surveillance Azure Log Analytics à partir d’Azure Security Center, car l’approvisionnement automatique est désactivé par défaut. L’agent peut également être installé manuellement. Lorsque l’approvisionnement automatique est activé, Security Center déploie l’agent Log Analytics sur toutes les machines virtuelles Azure prises en charge et toutes celles nouvellement créées. Security Center collecte des données sur vos machines virtuelles Azure, groupes de machines virtuelles identiques et conteneurs IaaS tels que les nœuds de cluster Kubernetes, pour surveiller les menaces et vulnérabilités de sécurité. Les données sont collectées à l’aide de l’agent Azure Log Analytics, qui lit divers journaux d’événements et configurations liées à la sécurité de l’ordinateur et copie les données dans votre espace de travail à des fins d’analyse. 

La collecte de données est requise pour fournir une visibilité des mises à jour manquantes, des paramètres de sécurité du système d’exploitation mal configurés, de l’état du point de terminaison ainsi que de l’intégrité et des détections de menaces.

- [Procédure d’activation de l’approvisionnement automatique de l’agent Log Analytics](../security-center/security-center-enable-data-collection.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Intégrez vos instances Azure Kubernetes Service (AKS) à Azure Monitor et définissez la période de rétention de l’espace de travail Azure Log Analytics correspondante en fonction des exigences de conformité de votre organisation. 

- [Définir les paramètres de conservation des journaux pour les espaces de travail Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Intégrez vos instances Azure Kubernetes Service (AKS) à Azure Monitor et configurez les paramètres de diagnostic de votre cluster. 

Utilisez l’espace de travail Log Analytics d’Azure Monitor pour examiner les journaux et effectuer des requêtes sur leurs données. Les journaux Azure Monitor sont activés et gérés dans le portail Azure ou via l’interface CLI, et ils fonctionnent avec le contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes), RBAC Azure et les clusters AKS non-RBAC.

Affichez les journaux générés par les principaux composants d’AKS (kube-apiserver et kube-controllermanager) pour la résolution des problèmes de votre application et de vos services. Activez des données et intégrez-les à Azure Sentinel ou une solution SIEM tierce pour la gestion et la surveillance centralisées des journaux.

- [Procédure d’activation et de consultation des journaux d’activité du nœud principal Kubernetes dans AKS](view-master-logs.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

- [Guide pratique pour effectuer des requêtes personnalisées dans Azure Monitor](../azure-monitor/log-query/get-started-queries.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : Utilisez Azure Kubernetes Service (AKS) avec Security Center pour une meilleure visibilité sur les nœuds AKS. Passez en revue les alertes Security Center sur les menaces et les activités malveillantes détectées au niveau de l’hôte et du cluster. Security Center implémente une analyse continue des événements de sécurité bruts survenant dans un cluster AKS, tels que les données réseau, la création de processus et le journal d’audit Kubernetes. Déterminez si cette activité est un comportement attendu ou si l’application ne fonctionne pas correctement. Utilisez des métriques et des journaux d’activité dans Azure Monitor pour justifier vos découvertes. 

- [Présentation de l’intégration d’Azure Kubernetes Service à Security Center](../security-center/defender-for-kubernetes-introduction.md)

- [Procédure d’activation du niveau Standard d’Azure Security Center](../security-center/security-center-get-started.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Conseils** : Installez et activez Microsoft Antimalware pour Azure sur les machines virtuelles Azure Kubernetes Service (AKS) et les nœuds de groupe de machines virtuelles identiques. Passez en revue les alertes dans Security Center à des fins de correction.

- [Microsoft Antimalware pour Azure Cloud Services et les machines virtuelles](../security/fundamentals/antimalware.md)

- [Guide de référence des alertes de sécurité](../security-center/alerts-reference.md)

- [Alertes pour les conteneurs : clusters Azure Kubernetes Service](../security-center/alerts-reference.md#alerts-akscluster)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Conseils** : AKS (Azure Kubernetes Service) utilise le projet CoreDNS pour la gestion et la résolution DNS de cluster.

Activez la journalisation de requête DNS en appliquant la configuration documentée dans votre ConfigMap coredns-custom. 

- [Personnaliser CoreDNS avec Azure Kubernetes Service](coredns-custom.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Conseils** : Utilisez kubectl, un client de ligne de commande, dans Azure Kubernetes Service (AKS) pour gérer un cluster Kubernetes et obtenir ses journaux à partir du nœud AKS à des fins de résolution des problèmes. Kubectl est déjà installé si vous utilisez Azure Cloud Shell. Pour installer kubectl localement, utilisez la cmdlet « Install-AzAksKubectl ».

- [Démarrage rapide : déployer un cluster Azure Kubernetes Service à l’aide de PowerShell](kubernetes-walkthrough-powershell.md)

- [Accéder aux journaux d’activité kubelet à partir de nœuds serveur Azure Kubernetes Service (AKS)](kubelet-logs.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Azure Kubernetes Service (AKS) à lui tout seul ne fournit pas de solution de gestion des identités qui stocke les comptes et les mots de passe d’utilisateurs standard. Avec l’intégration Azure Active Directory (Azure AD), vous pouvez accorder aux utilisateurs ou aux groupes l’accès aux ressources Kubernetes dans un espace de noms ou au sein du cluster. 

Effectuez des requêtes ad hoc afin de découvrir les comptes qui sont membres de groupes d’administration AKS à l’aide du module Azure AD PowerShell

Utilisez Azure CLI pour des opérations telles que « Obtenir les informations d’identification d’accès pour un cluster Kubernetes managé » afin de faciliter le rapprochement régulier de l’accès. Implémentez ce processus pour conserver un inventaire à jour des comptes de service qui sont un autre type d’utilisateur principal dans AKS. Appliquez les recommandations liées à la gestion des identités et des accès dans Security Center.

- [Guide pratique pour intégrer AKS à Azure AD](./azure-ad-integration-cli.md)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

- [Comment superviser les identités et les accès avec Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils** : Azure Kubernetes Service (AKS) n’inclut pas le concept de mots de passe par défaut communs et ne fournit pas de solution de gestion des identités dans laquelle les comptes et les mots de passe d’utilisateurs standard peuvent être stockés. Avec l’intégration Azure Active Directory (Azure AD), vous pouvez accorder un accès en fonction du rôle aux ressources AKS dans un espace de noms ou au sein du cluster. 

Effectuez des requêtes ad hoc afin de découvrir les comptes qui sont membres de groupes d’administration AKS à l’aide du module Azure AD PowerShell

- [Présentation des options d’accès et d’identité pour AKS](concepts-identity.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Intégrez l’authentification utilisateur pour vos clusters Azure Kubernetes Service (AKS) avec Azure Active Directory (Azure AD). Connectez-vous à un cluster AKS en utilisant un jeton d’authentification Azure AD. Configurez le contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes) pour l’accès d’administration aux informations de configuration de Kubernetes (kubeconfig) et aux autorisations, aux espaces de noms et aux ressources de cluster. 

Créez des stratégies et des procédures concernant l’utilisation de comptes d’administration dédiés. Implémentez les recommandations liées à la gestion des identités et des accès dans Security Center.

- [Comment superviser les identités et les accès avec Azure Security Center](../security-center/security-center-identity-access.md)

- [Contrôler l’accès aux ressources de cluster](azure-ad-rbac.md)

- [Utiliser des contrôles d’accès en fonction du rôle Azure](control-kubeconfig-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : Utilisez l’authentification unique pour l’authentification intégrée d’Azure Kubernetes Service (AKS) avec Azure Active Directory (Azure AD) pour un cluster AKS.

- [Comment voir les journaux, métriques de pod et événements Kubernetes en temps réel](../azure-monitor/insights/container-insights-livedata-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Intégrez l’authentification pour Azure Kubernetes Service (AKS) avec Azure Active Directory (Azure AD). 

Activez l’authentification multifacteur (MFA) Azure AD et suivez les recommandations de gestion des identités et des accès de Security Center.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md) 

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : Utilisez une station de travail à accès privilégié (PAW), avec l’authentification multifacteur (MFA) configurée pour la connexion à vos clusters Azure Kubernetes Service (AKS) et aux ressources associées.
- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : Utilisez des rapports de sécurité Azure Active Directory (Azure AD) avec l’authentification intégrée Azure AD pour Azure Kubernetes Service (AKS). Des alertes peuvent être générées lorsque des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement. Utilisez Security Center pour superviser l’activité liée aux identités et aux accès.

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-only-from-approved-locations"></a>3.8 : Gérer les ressources Azure uniquement à partir d’emplacements approuvés

**Conseils** : Utilisez des emplacements nommés avec accès conditionnel pour autoriser l’accès aux clusters Azure Kubernetes Service (AKS) uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions. Cela requiert l’authentification intégrée pour (AKS) avec Azure Active Directory (Azure AD).

Limitez l’accès au serveur d’API AKS à partir d’un ensemble limité de plages d’adresses IP, car il reçoit des requêtes pour effectuer des actions dans le cluster afin de créer des ressources ou de mettre à l’échelle le nombre de nœuds. 

- [Sécuriser l’accès au serveur d’API à l’aide de plages d’adresses IP autorisées dans Azure Kubernetes Service (AKS)](api-server-authorized-ip-ranges.md)

- [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils** : Utilisez Azure Active Directory (Azure AD) comme système central d’authentification et d’autorisation pour Azure Kubernetes Service (AKS). Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit, et il sale, hache et stocke de manière sécurisée les informations d’identification des utilisateurs.

Utilisez les rôles intégrés AKS avec le contrôle d’accès en fonction du rôle Azure (Azure RBAC), Contributeur et Propriétaire de la stratégie de ressource, pour les opérations d’attribution de stratégie à votre cluster Kubernetes

- [Vue d’ensemble d’Azure Policy](../governance/policy/overview.md)

- [Guide pratique pour intégrer Azure AD à AKS](./azure-ad-integration-cli.md) 

- [Intégrer Azure AD managé par AKS](managed-aad.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Conseils** : Utilisez des rapports de sécurité Azure Active Directory (Azure AD) avec l’authentification intégrée Azure AD pour Azure Kubernetes Service (AKS). Recherchez des journaux Azure AD pour vous aider à découvrir les comptes obsolètes. 

Vérifiez l’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. Corrigez les recommandations liées aux identités et aux accès de Security Center.

Tenez compte des rôles utilisés à des fins de support ou de résolution des problèmes. Par exemple, toutes les actions de cluster effectuées par le support de Microsoft (avec le consentement de l’utilisateur) sont effectuées sous le rôle Kubernetes intégré « edit » nommé aks-support-rolebinding. Le support AKS est activé avec ce rôle pour modifier la configuration du cluster et des ressources afin de résoudre ou de diagnostiquer les problèmes de cluster. Ce rôle ne peut toutefois pas modifier d’autorisations ni créer de rôles ou de liaisons de rôle. Cet accès aux rôles est activé uniquement sous les tickets de support actifs avec l’accès juste-à-temps (JIT).
 
- [Options d’accès et d’identité pour Azure Kubernetes Service (AKS)](concepts-identity.md)

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Conseils** : Intégrez l’authentification utilisateur pour Azure Kubernetes Service (AKS) avec Azure Active Directory (Azure AD). Créez des paramètres de diagnostic pour Azure AD en envoyant les journaux d’audit et de connexion à un espace de travail Azure Log Analytics. Configurez les alertes souhaitées (par exemple, lorsqu’un compte désactivé tente de se connecter) au sein d’un espace de travail Azure Log Analytics.
- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Guide pratique pour créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor](../azure-monitor/platform/alerts-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Conseils** : Intégrez l’authentification utilisateur pour Azure Kubernetes Service (AKS) avec Azure Active Directory (Azure AD). Utilisez les détections de risques et la fonctionnalité Identity Protection d’Azure AD pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. Ingérez des données dans Azure Sentinel pour approfondir votre examen en fonction des besoins de l’entreprise.

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Non applicable à Azure Kubernetes Service (AKS), car non pris en charge par Customer Lockbox.
- [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Non applicable

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes sur des déploiements Azure Kubernetes Service (AKS) pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

- [Mettre à jour les étiquettes de clusters managés](/rest/api/aks/managedclusters/updatetags)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Isolez logiquement les équipes et les charges de travail dans le même cluster avec Azure Kubernetes Service (AKS) pour fournir le plus petit nombre de privilèges, limités aux ressources nécessaires à chaque équipe. 

Utilisez l’espace de noms dans Kubernetes pour créer une limite d’isolation logique. Envisagez d’implémenter des fonctionnalités Kubernetes supplémentaires pour l’isolation et l’architecture multilocataires, telles que la planification, la mise en réseau, l’authentification/autorisation et les conteneurs.

Implémentez des abonnements et/ou des groupes d’administration distincts pour les environnements de développement, de test et de production. Séparez les clusters AKS avec mise en réseau en les déployant sur des réseaux virtuels distincts qui sont étiquetés correctement.

- [En savoir plus sur les meilleurs pratiques relatives à l’isolation de cluster dans AKS](operator-best-practices-cluster-isolation.md)

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Comprendre les meilleures pratiques relatives à la connectivité réseau et à la sécurité dans AKS](operator-best-practices-network.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : Utilisez une solution tierce issue de la Place de marché Azure sur les périmètres du réseau, qui surveille et bloque les transferts non autorisés d’informations sensibles tout en alertant les professionnels de la sécurité de l’information.

Microsoft gère la plateforme sous-jacente et traite tout le contenu des clients en tant que contenu sensible et déploie d’importants efforts pour assurer une protection contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Liste des ports, adresses et noms de domaines requis pour la fonctionnalité AKS](limit-egress-traffic.md)

- [Procédure de configuration des paramètres de diagnostic pour Pare-feu Azure](../firewall/firewall-diagnostics.md)

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Créez un contrôleur d’entrée HTTPS et utilisez vos propres certificats TLS (ou éventuellement des certificats Let’s Encrypt) pour vos déploiements Azure Kubernetes Service (AKS). 

Le trafic de sortie Kubernetes est chiffré sur HTTPS/TLS par défaut. Examinez le trafic sortant potentiellement non chiffré provenant de vos instances AKS pour une surveillance supplémentaire. Cela peut inclure le trafic NTP, le trafic DNS et le trafic HTTP pour la récupération des mises à jour dans certains cas. 

- [Procédure de création d’un contrôleur d’entrée HTTPS sur AKS et d’utilisation de vos propres certificats TLS](ingress-own-tls.md)

- [Procédure de création d’un contrôleur d’entrée HTTPS sur AKS avec Let’s Encrypt](ingress-tls.md)

- [Liste des ports et protocoles sortants potentiels utilisés par AKS](limit-egress-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Instructions** : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour le Stockage Azure ou les ressources de calcul. Implémentez une solution tierce si nécessaire à des fins de conformité.
Microsoft gère la plateforme sous-jacente et traite tout le contenu des clients en tant que contenu sensible et déploie d’importants efforts pour assurer une protection contre la perte et l’exposition des données client. 

Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="46-use-azure-rbac-to-manage-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Utilisez le système d’autorisation de contrôle d’accès en fonction du rôle Azure (Azure RBAC) basé sur Azure Resource Manager pour permettre une gestion précise des accès des ressources Azure.

Configurez Azure Kubernetes Service (AKS) pour utiliser Azure Active Directory (Azure AD) pour l’authentification utilisateur. Connectez-vous à un cluster AKS en utilisant un jeton d’authentification Azure AD utilisant cette configuration. 

Utilisez les rôles intégrés AKS avec Azure RBAC, Contributeur et Propriétaire de la stratégie de ressource, pour les opérations d’attribution de stratégie à votre cluster AKS

- [Procédure de contrôle de l’accès aux ressources de cluster à l’aide d’Azure RBAC et des identités Azure AD dans AKS](azure-ad-rbac.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Aide** : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour le Stockage Azure ou les ressources de calcul. Implémentez une solution tierce si nécessaire à des fins de conformité.
Microsoft gère la plateforme sous-jacente et traite tout le contenu des clients en tant que contenu sensible et déploie d’importants efforts pour assurer une protection contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Les deux principaux types de stockage fournis pour les volumes dans Azure Kubernetes Service (AKS) reposent sur des disques Azure ou Azure Files. Ces deux types de stockage utilisent Azure Storage Service Encryption (SSE), qui chiffre les données au repos, pour améliorer la sécurité. Par défaut, les données sont chiffrées avec des clés managées par Microsoft

Le chiffrement au repos à l’aide de clés gérées par le client est disponible pour chiffrer le système d’exploitation et des disques de données sur des clusters AKS afin de renforcer le contrôle sur les clés de chiffrement. Les clients sont responsables des activités de gestion des clés, telles que la sauvegarde et la rotation des clés. Les disques ne peuvent actuellement pas être chiffrés à l’aide d’Azure Disk Encryption au niveau des nœuds AKS.

- [Meilleures pratiques relatives au stockage et aux sauvegardes dans Azure Kubernetes Service (AKS)](operator-best-practices-storage.md)

- [BYOK (Bring Your Own Keys) avec des disques Azure dans Azure Kubernetes Service (AKS)](azure-disk-customer-managed-keys.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : Utilisez Azure Monitor pour conteneurs pour surveiller les performances des charges de travail de conteneur déployées sur des clusters Kubernetes managés hébergés sur Azure Kubernetes Service (AKS). 

Configurez des alertes pour recevoir une notification proactive ou créer un journal lorsque l’utilisation de l’UC et de la mémoire sur des nœuds ou des conteneurs dépasse des seuils définis, ou encore lorsqu’un changement d’état d’intégrité se produit dans le cluster au niveau de l’infrastructure ou du cumul d’intégrité des nœuds. 

Utilisez le journal d’activité Azure pour surveiller vos clusters AKS et les ressources associées à un niveau élevé. Bénéficier d’une intégration à Prometheus pour afficher les métriques d’application et de charge de travail qu’il collecte à partir des nœuds et Kubernetes à l’aide de requêtes pour créer des alertes personnalisées, des tableaux de bord et une analyse détaillée des performances.

- [Présentation d’Azure Monitor pour conteneurs](../azure-monitor/insights/container-insights-overview.md)

- [Procédure d’activation d’Azure Monitor pour les conteneurs](../azure-monitor/insights/container-insights-onboard.md)

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Utilisez Security Center pour surveiller votre Azure Container Registry, y compris les instances Azure Kubernetes Service (AKS) pour les vulnérabilités. Activez le pack de registres de conteneurs dans Security Center pour s’assurer que ce dernier est prêt à analyser les images qui sont envoyées au registre.

Recevez une notification dans le tableau de bord de Security Center si des problèmes sont détectés lorsque Security Center a analysé l’image à l’aide de Qualys. Le pack de registres de conteneurs offre une meilleure visibilité des vulnérabilités des images utilisées dans les registres basés sur Azure Resource Manager. 

Utilisez Security Center pour obtenir des recommandations pratiques pour chaque vulnérabilité. Ces recommandations incluent une classification de la gravité et des conseils de correction. 

- [Meilleures pratiques relatives à la gestion et à la sécurisation des images de conteneur dans Azure Kubernetes Services (AKS)](../security-center/defender-for-container-registries-introduction.md)

- [Présentation des meilleures pratiques relatives à la gestion et à la sécurité des images conteneur dans AKS](operator-best-practices-container-image-management.md)

- [Présentation de l’intégration de registre de conteneurs à Azure Security Center](../security-center/defender-for-container-registries-introduction.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : Les mises à jour de sécurité sont appliquées automatiquement aux nœuds Linux pour protéger les clusters Azure Kubernetes Service (AKS) du client. Ces mises à jour incluent des correctifs de sécurité ou des mises à jour du noyau. 

Notez que le processus de conservation des nœuds Windows Server à jour est différents des nœuds exécutant Linux, car les nœuds Windows Server ne reçoivent pas de mises à jour quotidiennes. Au lieu de cela, les clients doivent effectuer une mise à niveau sur les pools de nœuds Windows Server dans leurs clusters AKS qui déploie de nouveaux nœuds avec les dernières images et correctifs Windows Server à l’aide du panneau de configuration Azure ou d’Azure CLI. Ces mises à jour contiennent des améliorations en matière de sécurité ou de fonctionnalités pour AKS.

- [Comprendre comment les mises à jour sont appliquées aux nœuds de cluster AKS exécutant Linux](node-updates-kured.md)

- [Procédure de mise à niveau d’un pool de nœuds AKS pour des clusters AKS qui utilisent des nœuds Windows Server](use-multiple-node-pools.md#upgrade-a-node-pool)

- [Mises à niveau des images de nœud Azure Kubernetes Service (AKS)](node-image-upgrade.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="53-deploy-an-automated-patch-management-solution-for-third-party-software-titles"></a>5.3 : Déployer une solution de gestion automatisée des correctifs de logiciels tiers

**Conseils** : Implémentez un processus manuel pour garantir que les applications tierces du nœud de cluster Azure Kubernetes Service (AKS) restent à jour pendant toute la durée de la durée de vie du cluster. Cela peut nécessiter l’activation des mises à jour automatiques, la surveillance des nœuds ou l’exécution de redémarrages périodiques.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils** : Exportez les résultats de l’analyse de Security Center à intervalles réguliers et comparez les résultats pour vérifier que les vulnérabilités ont été corrigées. 

Utilisez la cmdlet PowerShell « Get-AzSecurityTask » pour automatiser la récupération des tâches de sécurité que Security Center vous recommande d’effectuer afin de votre posture de sécurité et les résultats de l’analyse des vulnérabilités de mise à jour.

- [Procédure d’utilisation de PowerShell pour afficher les vulnérabilités découvertes par Azure Security Center](/powershell/module/az.security/get-azsecuritytask)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Utilisez le classement de la gravité fourni par Security Center pour hiérarchiser la correction des vulnérabilités. 

Utilisez Common Vulnerability Score System (CVSS) (ou un autre système de scoring fourni par votre outil d’analyse) si vous utilisez un outil d’évaluation des vulnérabilités intégré (par exemple, Qualys ou Rapid7, proposé par Azure).

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (calcul, stockage, réseau, etc.) dans vos abonnements. Vérifiez que vous avez les autorisations (lecture) appropriées dans votre locataire et que vous êtes en mesure d’établir une liste de tous les abonnements Azure et de toutes les ressources dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources sur Azure Resource Manager (ARM) à l’avenir.

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en utilisant des métadonnées pour les organiser de façon logique dans une taxonomie.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration et diviser des abonnements, le cas échéant, pour organiser et suivre les ressources. 

Appliquez des teintes, des étiquettes ou des balises lors de la création d’un pool de nœuds Azure Kubernetes Service (AKS). Tous les nœuds de ce pool de nœuds hériteront également de cette teinte, étiquette ou balise.

Des teintes, étiquettes ou balises peuvent également être utilisées pour rapprocher régulièrement l’inventaire et vous assurer que les ressources non autorisées sont supprimées des abonnements en temps utile.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

- [Clusters managés - Mettre à jour les étiquettes](/rest/api/aks/managedclusters/updatetags)

- [Spécifier une teinte, un intitulé ou une étiquette pour un pool de nœuds](use-multiple-node-pools.md#specify-a-taint-label-or-tag-for-a-node-pool)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Dresser et tenir un inventaire des ressources Azure approuvées

**Conseils** : Définissez une liste de ressources Azure et logiciels approuvés pour les ressources de calcul en fonction des besoins de votre entreprise.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes :
-   Types de ressources non autorisés 

-   Types de ressources autorisés

Utilisez Azure Resource Graph pour interroger/découvrir les ressources dans vos abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées selon les besoins de l’entreprise.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Utilisez les fonctionnalités Suivi des modifications et inventaire d’Azure Automation pour déterminer les logiciels installés dans votre environnement. 

Collectez et affichez l’inventaire des logiciels, fichiers, démons Linux, services Windows et clés de Registre Windows présents sur vos ordinateurs, et surveillez les applications logicielles non approuvées. 

Suivez les configurations de vos ordinateurs pour vous aider à identifier les problèmes opérationnels au sein de votre environnement et à mieux comprendre l’état de vos ordinateurs.

- [Guide pratique pour activer l’inventaire des machines virtuelles Azure](../automation/automation-tutorial-installed-software.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Utilisez les fonctionnalités Suivi des modifications et inventaire d’Azure Automation pour déterminer les logiciels installés dans votre environnement. 

Collectez et affichez l’inventaire des logiciels, fichiers, démons Linux, services Windows et clés de Registre Windows présents sur vos ordinateurs, et surveillez les applications logicielles non approuvées. 

Suivez les configurations de vos ordinateurs pour vous aider à identifier les problèmes opérationnels au sein de votre environnement et à mieux comprendre l’état de vos ordinateurs.

- [Guide pratique pour activer l’inventaire des machines virtuelles Azure](../automation/automation-tutorial-installed-software.md)

- [Guide pratique pour utiliser le monitoring d’intégrité des fichiers](../security-center/security-center-file-integrity-monitoring.md)

- [Présentation d’Azure Change Tracking](../automation/change-tracking/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Utilisez les fonctionnalités Suivi des modifications et inventaire d’Azure Automation pour déterminer les logiciels installés dans votre environnement. 

Collectez et affichez l’inventaire des logiciels, fichiers, démons Linux, services Windows et clés de Registre Windows présents sur vos ordinateurs, et surveillez les applications logicielles non approuvées. 

Suivez les configurations de vos ordinateurs pour vous aider à identifier les problèmes opérationnels au sein de votre environnement et à mieux comprendre l’état de vos ordinateurs.

Activez l’analyse adaptative des applications dans Security Center pour les applications qui existent dans votre environnement.

- [Guide pratique pour activer l’inventaire des machines virtuelles Azure](../automation/automation-tutorial-installed-software.md)

 
Guide pratique pour utiliser les contrôles d’application adaptatifs d’Azure Security Center
- [Contrôles](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements clients selon les définitions de stratégies intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Utilisez Azure Resource Graph pour interroger/découvrir les ressources dans vos abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Conseils** : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements, en utilisant Azure Policy avec des définitions de stratégies intégrées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».
- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-in-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : Azure Kubernetes Service (AKS) ne fournit pas de solution de gestion des identités dans laquelle les comptes et les mots de passe d’utilisateurs standard sont stockés. Utilisez plutôt Azure Active Directory (Azure AD) comme solution d’identité intégrée pour vos clusters AKS. 

Accordez aux utilisateurs ou aux groupes l’accès aux ressources Kubernetes dans un espace de noms ou au sein du cluster à l’aide de l’intégration d’Azure AD. 

Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir des comptes membres de vos groupes d’administration AKS, et rapprocher les accès à intervalles réguliers. Utilisez Azure CLI pour des opérations telles que l’obtention des informations d’identification d’accès pour un cluster Kubernetes managé. Implémentez les recommandations liées à la gestion des identités et des accès dans Security Center.

- [Gérer AKS avec Azure CLI](/cli/azure/aks)

- [Présentation de l’intégration d’AKS et d’Azure AD](concepts-identity.md)

- [Guide pratique pour intégrer AKS à Azure AD](./azure-ad-integration-cli.md)

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember)

- [Comment superviser les identités et les accès avec Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Utilisez des fonctionnalités d’Azure Kubernetes Service (AKS) pour isoler logiquement les équipes et les charges de travail dans le même cluster afin de fournir le plus petit nombre de privilèges, limités aux ressources nécessaires à chaque équipe. 

Implémentez l’espace de noms dans Kubernetes pour créer une limite d’isolation logique. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ContainerService » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de vos instances Azure Kubernetes Service (AKS). 

Examinez et implémentez des fonctionnalités et recommandations Kubernetes supplémentaires pour l’isolation et la mutualisation, en lien avec les aspects suivants : planification, mise en réseau, authentification/autorisation et conteneurs. Utilisez également des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Séparez les clusters AKS avec des réseaux virtuels, des sous-réseaux qui sont étiquetés correctement et sécurisés avec un Web Application Firewall (WAF).

- [En savoir plus sur les meilleurs pratiques relatives à l’isolation de cluster dans AKS](operator-best-practices-cluster-isolation.md)

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Comprendre les meilleures pratiques relatives à la connectivité réseau et à la sécurité dans AKS](operator-best-practices-network.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.ContainerService » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de vos instances Azure Kubernetes Service (AKS). Utilisez les définitions Azure Policy intégrées.

Voici des exemples de définitions de stratégie intégrées pour AKS :

•   Appliquer une entrée HTTPS dans un cluster Kubernetes

•   Des plages d’adresses IP autorisées doivent être définies sur les services Kubernetes

•   Un contrôle d’accès en fonction du rôle (RBAC) doit être utilisé sur les services Kubernetes

•   Garantir que les images conteneur autorisées s’exécutent dans un cluster Kubernetes

Exportez un modèle de votre configuration AKS en JavaScript Object Notation (JSON) avec Azure Resource Manager. Vérifiez-le régulièrement pour vous assurer que ces configurations satisfont les exigences de sécurité de votre organisation. Utilisez les recommandations d’Azure Security Center comme ligne de base de configuration sécurisée pour vos ressources Azure. 

- [Guide pratique pour configurer et gérer les stratégies de sécurité du des pods AKS](use-pod-security-policies.md)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Conseils** : Les clusters Azure Kubernetes Service (AKS) sont déployés sur des machines virtuelles hôtes avec un système d’exploitation doté d’une sécurité optimisée. Le système d’exploitation hôte comprend des étapes de durcissement de la sécurité intégrées afin de réduire la surface d’attaque et de permettre le déploiement de conteneurs de manière sécurisée. 

Azure applique des correctifs quotidiens (y compris des correctifs de sécurité) aux hôtes de machines virtuelles AKS, certains correctifs nécessitant un redémarrage. Les clients sont responsables de la planification des redémarrages des hôtes de machines virtuelles AKS en fonction des besoins. 

- [Durcissement de la sécurité du système d’exploitation hôte du nœud de l’agent AKS](security-hardened-vm-host-image.md)

- [Présentation du durcissement de la sécurité des hôtes de machines virtuelles AKS](security-hardened-vm-host-image.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Sécurisez votre cluster Azure Kubernetes Service (AKS) à l’aide de stratégies de sécurité des pods.  Limitez les pods pouvant être planifiés pour améliorer la sécurité de votre cluster. 

Les pods qui demandent des ressources non autorisées ne peuvent pas s’exécuter dans le cluster AKS. 

Utilisez également les effets Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés aux ressources Azure associées à vos déploiements AKS (tels que les réseaux virtuels, les sous-réseaux, les Pare-feu Azure, les comptes de stockage, etc.). 

Créez des définitions Azure Policy personnalisées à l’aide d’alias à partir des espaces de noms suivants : 

•   Microsoft.ContainerService

•   Microsoft.Network

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Conseils** : Les clusters Azure Kubernetes Service (AKS) sont déployés sur des machines virtuelles hôtes avec un système d’exploitation doté d’une sécurité optimisée. Le système d’exploitation hôte comprend des étapes de durcissement de la sécurité intégrées afin de réduire la surface d’attaque et de permettre le déploiement de conteneurs de manière sécurisée. 

Reportez-vous à la liste des contrôles du CIS (Center for Internet Security) qui sont intégrés au système d’exploitation hôte.  

- [Durcissement de la sécurité du système d’exploitation hôte du nœud de l’agent AKS](security-hardened-vm-host-image.md)

- [Présentation de la configuration de l’état des clusters AKS](concepts-clusters-workloads.md#control-plane)

- [Présentation du durcissement de la sécurité des hôtes de machines virtuelles AKS](security-hardened-vm-host-image.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils** : Utilisez Azure Repos pour stocker et gérer vos configurations en toute sécurité si vous utilisez des définitions Azure Policy personnalisées. Exportez un modèle de votre configuration Azure Kubernetes Service (AKS) en JavaScript Object Notation (JSON) avec Azure Resource Manager. Vérifiez-le régulièrement pour vous assurer que ces configurations satisfont les exigences de sécurité de votre organisation. 

Implémentez des solutions tierces telles que Terraform pour créer un fichier de configuration qui déclare les ressources du cluster Kubernetes. Vous pouvez renforcer votre déploiement AKS en implémentant les meilleures pratiques de sécurité et stocker votre configuration en tant que code à un emplacement sécurisé.

- [Définir un cluster Kubernetes](/azure/developer/terraform/create-k8s-cluster-with-tf-and-aks#define-a-kubernetes-cluster)

Durcissement de la sécurité du système d’exploitation hôte du nœud de l’agent AKS

security-hardened-vm-host-image.md

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils** : Non applicable à Azure Kubernetes Service (AKS). AKS fournit un système d’exploitation hôte à sécurité optimisée par défaut. Il n’existe actuellement aucune option pour sélectionner un autre système d’exploitation ou un système d’exploitation personnalisé.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements, en utilisant des définitions de stratégie intégrées ainsi que des alias Azure Policy dans l’espace de noms « Microsoft.ContainerService ». 

Créez des stratégies personnalisées pour auditer et appliquer des configurations système. Développez un processus et un pipeline pour la gestion des exceptions de stratégie.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Utiliser des alias](../governance/policy/concepts/definition-structure.md#aliases)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer des outils de gestion de la configuration pour les systèmes d'exploitation

**Conseils** : Les clusters Azure Kubernetes Service (AKS) sont déployés sur des machines virtuelles hôtes avec un système d’exploitation doté d’une sécurité optimisée. Le système d’exploitation hôte comprend des étapes de durcissement de la sécurité intégrées afin de réduire la surface d’attaque et de permettre le déploiement de conteneurs de manière sécurisée. 

Reportez-vous à la liste des contrôles du CIS (Center for Internet Security) qui sont intégrés à l’hôte AKS.  

- [Durcissement de la sécurité du système d’exploitation hôte du nœud de l’agent AKS](security-hardened-vm-host-image.md)

- [Présentation du durcissement de la sécurité des hôtes de machines virtuelles AKS](security-hardened-vm-host-image.md)

- [Présentation de la configuration de l’état des clusters AKS](concepts-clusters-workloads.md#control-plane)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils** : Utilisez Security Center pour effectuer des analyses de ligne de base pour vos déploiements Azure Kubernetes Service (AKS). Entre autres exemples de ressources, citons par exemple le cluster AKS lui-même, le réseau virtuel sur lequel le cluster AKS a été déployé, le compte de stockage Azure utilisé pour assurer le suivi de l’état de Terraform ou les instances Azure Key Vault utilisées pour les clés de chiffrement pour le système d’exploitation et les disques de données de votre cluster AKS.

- [Corriger les recommandations dans Azure Security Center](../security-center/security-center-remediate-recommendations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : Utilisez les recommandations de conteneur de Security Center dans la section « Compute &amp; apps » pour effectuer des analyses de ligne de base pour vos clusters Azure Kubernetes Service (AKS). Recevez une notification dans le tableau de bord de Security Center lorsque des problèmes de configuration ou des vulnérabilités sont détectés. Cela nécessite l’activation du pack de registres de conteneurs facultatif qui permet à Security Center d’analyser l’image.  

- [Comprendre les recommandations concernant les conteneurs dans Azure Security Center](../security-center/container-security.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Intégrez Azure Key Vault à un cluster Azure Kubernetes Service (AKS) à l’aide d’un pilote FlexVolume. Utilisez Azure Key Vault pour stocker et faire tourner régulièrement les secrets, tels que les informations d’identification, les clés de compte de stockage ou les certificats. Le pilote FlexVolume permet au cluster AKS de récupérer en mode natif les informations d’identification dans Key Vault et de les remettre uniquement au pod demandeur, en toute sécurité. Utilisez une identité de pod managée pour demander l’accès à Key Vault et récupérer les informations d’identification requises via le pilote FlexVolume. Vérifiez que la suppression réversible est activée dans Key Vault. 

Limitez l’exposition des informations d’identification en ne les définissant pas dans le code de votre application. 

Évitez d’utiliser des informations d’identification fixes ou partagées. 

- [Concepts de sécurité pour les applications et les clusters dans AKS (Azure Kubernetes Service)](concepts-security.md)

- [Guide pratique pour utiliser Key Vault avec votre cluster AKS](developer-best-practices-pod-security.md#limit-credential-exposure)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : Comme meilleure pratique, ne définissez pas d’informations d’identification dans le code de votre application. Utilisez des identités managées pour les ressources Azure pour permettre à un pod de s’authentifier dans Azure auprès d’un service qui le prend en charge, notamment Azure Key Vault. Le pod se voit attribuer une identité Azure pour s’authentifier auprès d’Azure Active Directory (Azure AD) et recevoir un jeton numérique qui peut être présenté à d’autres services Azure qui vérifient si le pod est autorisé à accéder au service et à effectuer les actions requises. 

Notez que les identités managées sont conçues pour être utilisées avec des pods Linux et des images conteneur uniquement. Approvisionnez Azure Key Vault pour qu’il stocke et récupère les clés numériques et les informations d’identification. Des clés telles que celles utilisées pour chiffrer les disques de système d’exploitation, les données de cluster AKS, peuvent être stockées dans Azure Key Vault.

Des principaux du service peuvent également être utilisés dans des clusters AKS. Toutefois, les clusters utilisant des principaux du service peuvent finir par atteindre un état dans lequel le principal du service doit être renouvelé pour que le cluster continue de fonctionner. La gestion des principaux de service ajoute de la complexité : c’est pourquoi il est plus facile d’utiliser à la place des identités managées. Les mêmes exigences d’autorisation s’appliquent aux principaux de service et aux identités managées.

- [Présentation des identités managées et de Key Vault avec Azure Kubernetes Service (AKS)](developer-best-practices-pod-security.md#limit-credential-exposure)

- [Identité de pod Azure Active Directory](https://github.com/Azure/aad-pod-identity)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault, avec des recommandations.

Limitez l’exposition des informations d’identification en ne les définissant pas dans le code de votre application. Évitez également d’utiliser des informations d’identification partagées. Azure Key Vault doit être utilisé pour stocker et récupérer les clés numériques et les informations d’identification. Utilisez des identités managées pour les ressources Azure pour permettre à votre pod de demander l’accès à d’autres ressources. 

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

- [Meilleures pratiques pour les développeurs relatives à la sécurité des pods](developer-best-practices-pod-security.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-antimalware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils** : AKS gère le cycle de vie et les opérations des nœuds d’agent pour votre compte ; la modification des ressources IaaS associées aux nœuds d’agent n’est pas prise en charge. Toutefois, pour les nœuds Linux, vous pouvez utiliser des ensembles de démons pour installer des logiciels personnalisés comme une solution anti-programme malveillant.

- [Guide de référence des alertes de sécurité](../security-center/alerts-reference.md)

- [Alertes pour les conteneurs : clusters Azure Kubernetes Service](../security-center/alerts-reference.md#alerts-akscluster)

- [Responsabilité partagée d’AKS et ensembles de démons](support-policies.md#shared-responsibility)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Conseils** : Effectuez une analyse préliminaire des fichiers chargés sur vos ressources AKS. Utilisez la détection des menaces de Security Center pour les services de données afin de détecter les logiciels malveillants chargés sur les comptes de stockage si vous utilisez un compte de stockage Azure comme magasin de données ou pour suivre l’état de Terraform pour votre cluster AKS. 

- [Présentation de la détection des menaces pour les services de données d’Azure Security Center](../security-center/azure-defender.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="83-ensure-antimalware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont à jour

**Conseils** : AKS gère le cycle de vie et les opérations des nœuds d’agent pour votre compte ; la modification des ressources IaaS associées aux nœuds d’agent n’est pas prise en charge. Toutefois, pour les nœuds Linux, vous pouvez utiliser des ensembles de démons pour installer des logiciels personnalisés comme une solution anti-programme malveillant.

- [Guide de référence des alertes de sécurité](../security-center/alerts-reference.md)

- [Alertes pour les conteneurs : clusters Azure Kubernetes Service](../security-center/alerts-reference.md#alerts-akscluster)

- [Responsabilité partagée d’AKS et ensembles de démons](support-policies.md#shared-responsibility)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="data-recovery"></a>Récupération de données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Conseils** : Sauvegardez vos données à l’aide d’un outil approprié pour votre type de stockage, tel que Velero, qui peut sauvegarder des volumes persistants avec des ressources et des configurations de cluster supplémentaires. Vérifiez régulièrement l’intégrité et la sécurité de ces sauvegardes. 

Supprimez l’état de vos applications avant de sauvegarder. Si vous ne pouvez pas le faire, sauvegardez les données de volumes persistants et testez régulièrement les opérations de restauration pour vérifier l’intégrité des données et les processus requis.

- [Meilleures pratiques relatives au stockage et aux sauvegardes dans AKS](operator-best-practices-storage.md)

- [Meilleures pratiques relatives à la continuité d’activité et reprise d’activité dans AKS](operator-best-practices-multi-region.md)

- [Présentation d’Azure Site Recovery](../site-recovery/site-recovery-overview.md)

- [Guide pratique pour configurer Velero sur Azure](https://github.com/vmware-tanzu/velero-plugin-for-microsoft-azure/blob/master/README.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Conseils** : Sauvegardez vos données à l’aide d’un outil approprié pour votre type de stockage, tel que Velero, qui peut sauvegarder des volumes persistants avec des ressources et des configurations de cluster supplémentaires. 

Effectuez des sauvegardes automatisées régulières des certificats, clés, comptes de stockage managés et secrets Key Vault avec les commandes PowerShell. 

Par exemple :

Backup-AzKeyVaultCertificate Backup-AzKeyVaultKey Backup-AzKeyVaultManagedStorageAccount Backup-AzKeyVaultSecret

- [Comment sauvegarder des certificats Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultcertificate)

- [Comment sauvegarder des clés Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey)

- [Comment sauvegarder des comptes de stockage managés Key Vault](/powershell/module/az.keyvault/add-azkeyvaultmanagedstorageaccount)

- [Comment sauvegarder des secrets Key Vault](/powershell/module/azurerm.keyvault/backup-azurekeyvaultsecret)

- [Guide pratique pour activer la Sauvegarde Azure](../backup/index.yml)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Effectuez régulièrement une restauration du contenu dans la sauvegarde Velero. Si nécessaire, testez la restauration sur un réseau virtuel isolé.

Effectuez régulièrement une restauration de données des certificats, clés, comptes de stockage managés et secrets Key Vault avec des commandes PowerShell. 

Par exemple :

Restore-AzKeyVaultCertificate ; Restore-AzKeyVaultKey ; Restore-AzKeyVaultManagedStorageAccount ; Restore-AzKeyVaultSecret

- [Restauration de certificats Key Vault](/powershell/module/azurerm.keyvault/restore-azurekeyvaultcertificate?view=azurermps-6.13.0)

- [Restauration de clés Key Vault](/powershell/module/azurerm.keyvault/restore-azurekeyvaultkey?view=azurermps-6.13.0)

- [Restauration des comptes de stockage gérés par Key Vault](/powershell/module/az.keyvault/backup-azkeyvaultmanagedstorageaccount)

- [Restauration de secrets Key Vault](/powershell/module/azurerm.keyvault/restore-azurekeyvaultsecret?view=azurermps-6.13.0)

- [Guide pratique pour récupérer des fichiers à partir d’une sauvegarde de machines virtuelles Azure](../backup/backup-azure-restore-files-from-vm.md)

**Supervision Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés gérées par le client

**Conseils** : Sauvegardez vos données à l’aide d’un outil approprié pour votre type de stockage, tel que Velero, qui peut sauvegarder des volumes persistants avec des ressources et des configurations de cluster supplémentaires. 

Activez la suppression réversible dans Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante si Azure Key Vault est utilisé avec des déploiements Azure Kubernetes Service (AKS).

- [Présentation d’Azure Storage Service Encryption](../storage/common/storage-service-encryption.md)

- [Activer la suppression réversible dans Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

- [Comment configurer des automatisations de workflow dans Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/)

- [Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils** : Classez par ordre de priorité les alertes qui doivent être examinées en premier avec un niveau de gravité attribué par Security Center aux alertes. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.
Marquez clairement les abonnements (par exemple, production, non-production) et créez un système de nommage pour identifier et classer les ressources Azure de façon claire.

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Reportez-vous à la publication du NIST pour identifier les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- [Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf) (Guide de test, de formation et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez les alertes et recommandations de Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez également choisir le connecteur de données Security Center pour diffuser en continu les alertes vers Azure Sentinel en fonction des exigences de votre organisation.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de Microsoft pour garantir que vos tests d’intrusion sont conformes aux stratégies de Microsoft : https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

- [Vous trouverez ici plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming et sur les tests d’intrusion de site actif dans l’infrastructure, les services et les applications cloud gérés par Microsoft.](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)
