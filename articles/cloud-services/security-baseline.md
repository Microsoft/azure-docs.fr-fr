---
title: Ligne de base de sécurité Azure pour Azure Cloud Services
description: La ligne de base de sécurité pour Azure Cloud Services fournit des instructions et ressources pour l’implémentation des recommandations de sécurité spécifiées dans le Benchmark de sécurité Azure.
author: msmbaldwin
ms.service: cloud-services
ms.topic: conceptual
ms.date: 11/02/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 100871ab2188aa638c4c9f2b058021706c7741bb
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96531902"
---
# <a name="azure-security-baseline-for-azure-cloud-services"></a>Ligne de base de sécurité Azure pour Azure Cloud Services

Cette ligne de base de sécurité applique les instructions du [ Benchmark de sécurité Azure version 1.0](../security/benchmarks/overview-v1.md) à l’offre Microsoft Azure Cloud Services. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure.
Le contenu est regroupé par les **contrôles de sécurité** définis par le Benchmark de sécurité Azure et les conseils associés applicables à l’offre Azure Cloud Services. Les **contrôles** non applicables à l’offre Azure Cloud Services ont été exclus.

 
Pour voir comment l’offre Azure Cloud Services est entièrement mappée au Benchmark de sécurité Azure, consultez le[fichier de mappage complet de la ligne de base de sécurité de l’offre Azure Cloud Services](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Conseils** : créez un réseau virtuel Azure classique avec des sous-réseaux public et privé distincts pour appliquer une isolation basée sur des ports et des plages d’adresses IP de confiance. Ce réseau virtuel et ces sous-réseaux doivent être des ressources basées sur le réseau virtuel classique (déploiement classique), non sur les ressources Azure Resource Manager actuelles.  

Autorisez ou refusez le trafic à l’aide d’un groupe de sécurité réseau contenant des règles de contrôle d’accès en fonction de la direction du trafic, du protocole, de l’adresse et du port sources, ainsi que de l’adresse et du port de destination. Vous pouvez modifier les règles d’un groupe de sécurité réseau à tout moment, et les modifications sont appliquées à toutes les instances associées.

L’offre Microsoft Azure Cloud Services (classique) ne peut pas être placée dans des réseaux virtuels Azure Resource Manager. Toutefois, des réseaux virtuels basés sur Resource Manager et des réseaux virtuels basés sur un déploiement classiques peuvent être connectés par le biais d’un appairage. 

- [Vue d’ensemble du groupe de sécurité réseau](../virtual-network/network-security-groups-overview.md)

- [Appairage de réseaux virtuels](./cloud-services-connectivity-and-networking-faq.md?preserve-view=true#how-can-i-use-azure-resource-manager-virtual-networks-with-cloud-services)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-nics"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Conseils** : documentez votre configuration Azure Cloud Services et surveillez ses modifications. Utilisez le fichier de configuration de service pour spécifier le nombre d’instances de rôle à déployer pour chaque rôle dans le service, les valeurs des paramètres de configuration et les empreintes des certificats associés à un rôle. 

Si le service fait partie d’un réseau virtuel, les informations de configuration du réseau doivent être fournies dans le fichier de configuration du service, ainsi que dans le fichier de configuration du réseau virtuel. L’extension par défaut du fichier de configuration de service est .cscfg. Notez qu’Azure Policy n’est pas pris en charge pour des déploiements classiques destinés à une application de configuration.

Définissez les valeurs de configuration d’un service cloud dans le fichier de configuration de service (.cscfg), et la définition dans un fichier de définition de service (.csdef). Utilisez le fichier de définition de service pour définir le modèle de service d’une application. Définissez les rôles disponibles pour un service cloud, et spécifiez les points de terminaison de service. Consignez la configuration pour Azure Cloud Services avec un fichier de configuration de service. Toute reconfiguration peut être effectuée à l’aide du fichier ServiceConfig.cscfg. 

Surveillez la définition du service de l’élément facultatif NetworkTrafficRules, qui limite les rôles pouvant communiquer avec des points de terminaison internes spécifiés. Configurez le nœud NetworkTrafficRules, un élément facultatif dans le fichier de définition de service, pour spécifier le mode de communication entre les rôles. Fixez des limites aux rôles pouvant accéder aux points de terminaison internes du rôle spécifique.  Notez qu’il n’est pas possible de modifier la définition de service. 

Activez les journaux de flux de groupe de sécurité réseau et envoyez ceux-ci à un compte Stockage Azure à des fins d’audit. Envoyez les journaux de flux à un espace de travail Log Analytics, puis utilisez Traffic Analytics pour obtenir des insights sur les modèles de trafic au sein de votre locataire Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau, d’identifier les zones réactives et des menaces de sécurité, de comprendre les modèles de flux de trafic, ainsi que d’épingler de mauvaises configurations du réseau.

- [Déploiement Azure Resource Manager et déploiement classique : comprendre les modèles de déploiement et l’état de vos ressources](../azure-resource-manager/management/deployment-models.md)

- [Fichier de configuration d’Azure Cloud Services](schema-cscfg-file.md)

- [Liste des services qu’Azure Policy prend en charge](/cli/azure/azure-services-the-azure-cli-can-manage?preserve-view=)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Microsoft utilise le protocole TLS (Transport Layer Security) v1.2 pour protéger les données transitant entre Azure Cloud Services et les clients. Les centres de données Microsoft négocient une connexion TLS avec les systèmes clients qui se connectent aux services Azure. TLS fournit une authentification forte, la confidentialité et l’intégrité des messages (activation de la détection de falsification et d’interception des messages), l’interopérabilité, la flexibilité des algorithmes, ainsi que la facilité de déploiement et d’utilisation.

- [Notions de base du chiffrement](../security/fundamentals/encryption-overview.md)

- [Configurer des certificats TLS/SSL](cloud-services-configure-ssl-certificate-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Conseils** : le cloud Azure implémente une sécurité réseau multicouche pour protéger ses services de plateforme contre les attaques par déni de service distribué (DDoS). Le service Azure DDoS Protection fait partie du processus de surveillance continue d’Azure, qui est constamment amélioré à l’aide de tests de pénétration. Cette Protection DDos est conçue pour résister non seulement aux attaques extérieures, mais aussi à celles émanant d’autres locataires Azure. 

Il existe plusieurs façons de bloquer ou de refuser la communication à côté de la protection au niveau de la plateforme au sein d’Azure Cloud Services. Celles-ci sont les suivantes : 

-  créer une tâche de démarrage pour bloquer de manière sélective certaines adresses IP spécifiques ;
-  limiter l’accès d’un rôle web Azure à un ensemble d’adresses IP spécifiques en modifiant votre fichier IIS web.config.

Empêchez le trafic entrant via l’URL ou le nom par défaut de vos services cloud (par exemple, *.cloudapp.net). Définissez l’en-tête de l’hôte sur un nom DNS personnalisé, sous la configuration de liaison de site dans le fichier de définition des services cloud (* .csdef).

Configurez une règle de refus d’application pour les attributions d’administrateur d’abonnement classique. Par défaut, une fois un point de terminaison interne défini, la communication peut s’effectuer à partir de n’importe quel rôle vers le point de terminaison interne d’un rôle sans restriction. Pour restreindre la communication, vous devez ajouter un élément NetworkTrafficRules à l'élément ServiceDefinition dans le fichier de définition de service.

- [Comment bloquer/désactiver le trafic entrant via l’URL par défaut de mon service cloud ?](./cloud-services-connectivity-and-networking-faq.md?preserve-view=true#how-can-i-blockdisable-incoming-traffic-to-the-default-url-of-my-cloud-service)

- [Azure DDoS Protection](./cloud-services-connectivity-and-networking-faq.md?preserve-view=true#how-do-i-prevent-receiving-thousands-of-hits-from-unknown-ip-addresses-that-might-indicate-a-malicious-attack-to-the-cloud-service)

- [Bloquer une adresse IP spécifique](cloud-services-startup-tasks-common.md#block-a-specific-ip-address)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : utilisez Azure Network Watcher, service de surveillance, de diagnostic et d’analytique des performances réseau permettant de surveiller les réseaux Azure. L’extension de machine virtuelle Agent Network Watcher est obligatoire pour capturer le trafic réseau à la demande et pour assurer la prise en charge d’autres fonctionnalités avancées sur des machines virtuelles Azure. Installez l’extension de machine virtuelle Agent Network Watcher et activez les journaux de flux de groupe de sécurité réseau.

Configurez la journalisation de flux sur un groupe de sécurité réseau. Examinez les détails du déploiement de l’extension de machine virtuelle Network Watcher sur une machine virtuelle existante déployée via le modèle de déploiement classique.

- [Configurer la journalisation de flux sur un groupe de sécurité réseau](../virtual-machines/extensions/network-watcher-linux.md)

- [Pour plus d’informations sur la configuration des journaux de flux, visitez](/cli/azure/azure-services-the-azure-cli-can-manage?preserve-view=)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Conseils** : l’offre Azure Cloud Services n’intègre pas de fonctionnalité IDS ou IPS. Les clients peuvent sélectionner et déployer une solution IDS ou IPS réseau supplémentaire à partir de la Place de marché Azure en fonction de leurs besoins organisationnels. Lorsque vous utilisez des solutions tierces, veillez à tester minutieusement votre solution IDS ou IPS sélectionnée avec l’offre Azure Cloud Services pour garantir un fonctionnement et une fonctionnalité corrects.

- [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/?term=Firewall) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : les certificats de service associés à l’offre Azure Cloud Services permettent une communication sécurisée avec le service. Ces certificats de service sont définis dans la définition des services, et déployés automatiquement sur la machine virtuelle exécutant une instance de rôle web. Par exemple, pour un rôle web, vous pouvez utiliser un certificat de service capable d’authentifier un point de terminaison HTTPS exposé. 

Pour mettre à jour le certificat, il est uniquement nécessaire de charger un nouveau certificat et de modifier la valeur d’empreinte numérique dans le fichier de configuration de service.

Utilisez le protocole TLS 1.2, la méthode de sécurisation des données la plus couramment utilisée, afin de garantir la confidentialité et la protection de l’intégrité. 

En règle générale, pour protéger des applications web et les protéger contre des attaques telles que celles dirigées contre les 10 principales vulnérabilités identifiées par l’OWASP, vous pouvez déployer l’équilibreur de charge Azure Application Gateway prenant en charge Azure Web Application Firewall pour protéger les applications web. 

- [Certificats de service](cloud-services-certs-create.md)

- [Configuration de TLS pour une application dans Azure](cloud-services-configure-ssl-certificate-portal.md)

- [Guide pratique pour déployer Application Gateway](../application-gateway/quick-create-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : renforcez votre configuration Azure Cloud Services et surveillez les modifications apportées à celle-ci. Le fichier de configuration de service spécifie le nombre d’instances de rôle à déployer pour chaque rôle dans le service, les valeurs des paramètres de configuration et les empreintes des certificats associés à un rôle. 

Si votre service fait partie d’un réseau virtuel, les informations de configuration pour le réseau doivent être fournies dans le fichier de configuration de service, ainsi que dans le fichier de configuration de réseau virtuel. L’extension par défaut du fichier de configuration de service est .cscfg.

Notez qu’Azure Policy n’est pas pris en charge avec l’offre Azure Cloud Services pour l’application la configuration.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : vous pouvez utiliser des groupes de sécurité réseau Azure pour filtrer le trafic réseau à destination et en provenance de ressources Azure dans un réseau virtuel Azure. Un groupe de sécurité réseau contient des règles de sécurité qui autorisent ou refusent le trafic réseau entrant ou sortant à destination ou en provenance de différents types de ressources Azure. Pour chaque règle, vous pouvez spécifier la source et la destination, le port et le protocole.

Utilisez le champ « Description » pour les règles de groupe de sécurité réseau individuelles dans l’offre Azure Cloud Services afin de documenter les règles qui autorisent le trafic à destination et à partir d’un réseau.

- [Comment filtrer le trafic réseau avec les règles de groupes de sécurité réseau](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : utilisez les fonctionnalités intégrées de surveillance de point de terminaison et de basculement automatique de point de terminaison d’Azure Traffic Manager. Elles vous aident à fournir des applications haute disponibilité résilientes par rapport aux défaillances de point de terminaison et de région Azure. Pour configurer une surveillance de point de terminaison, vous devez spécifier certains paramètres sur votre profil Traffic Manager.

Rassemblez des informations du journal d’activité, un journal de plateforme dans Azure, dans des événements de niveau abonnement. Ces informations indiquent, par exemple, à quel moment une ressource a été modifiée ou une machine virtuelle démarrée. Afficher le journal d’activité dans le portail Azure, ou récupérez des entrées avec PowerShell et Azure CLI. 

Créez un paramètre de diagnostic pour envoyer le journal d’activité à Azure Monitor, à Azure Event Hubs afin de le transférer hors d’Azure ou au service Stockage Azure à des fins d’archivage. Configurez Azure Monitor pour les alertes de notification quand des ressources critiques dans Azure Cloud Services sont modifiées. 

- [Journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Créer, afficher et gérer des alertes de journal d’activité à l’aide d’Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

- [Surveillance avec Traffic Manager](../traffic-manager/traffic-manager-monitoring.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : Microsoft gère les sources de temps des ressources Azure pour Azure Cloud Services. Les clients peuvent créer une règle de réseau pour autoriser l’accès à un serveur de temps utilisé dans leur environnement, sur le port 123 avec le protocole UDP.

- [Accès au serveur NTP](../firewall/protect-windows-virtual-desktop.md#additional-considerations)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : utilisez les données de streaming de votre service cloud par programmation avec Azure Event Hubs. Intégrez et envoyez toutes ces données à Azure Sentinel pour surveiller et examiner vos journaux, ou utilisez un ensemble de technologies SIEM tiers. Pour la gestion des journaux de sécurité centralisée, configurez l’exportation continue des données d’Azure Security Center que vous avez choisies vers Azure Event Hubs, et configurez le connecteur approprié pour votre solution SIEM. Voici quelques options pour Azure Sentinel, notamment des outils tiers :

- Azure Sentinel : utilisez le connecteur de données d’alertes Security Center natif
- Splunk : utilisez le module complémentaire Azure Monitor pour Splunk
- IBM QRadar : utilisez une source de journal configurée manuellement
- ArcSight : utilisez SmartConnector

Consultez la documentation Azure Sentinel pour plus d’informations sur les connecteurs disponibles avec Azure Sentinel. 

- [Connecter des sources de données](../sentinel/connect-data-sources.md)

- [Intégrer avec une solution SIEM](../security-center/continuous-export.md)

- [Stockez les données de diagnostic](diagnostics-extension-to-storage.md)

- [Configuration de l’intégration SIEM à l’aide d’Azure Event Hubs](../security-center/continuous-export.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : configurez Visual Studio afin de définir l’agent Diagnostics Azure pour la résolution des problèmes liés à l’offre Azure Cloud Services qui capture les données système et de journalisation sur les machines virtuelles, y compris les instances de machines virtuelles exécutant vos Azure Cloud Services. Les données de diagnostic sont transférées vers un compte de stockage de votre choix. Activez les diagnostics dans les projets Azure Cloud Services avant leur déploiement.

 
Affichez l’historique des modifications pour certains événements dans le journal d’activité d’Azure Monitor. Auditez les modifications qui se sont produites pendant une période d’événement. Choisissez un événement dans le journal d’activité pour une inspection plus poussée avec l’onglet Historique des modifications (préversion). Envoyez les données de diagnostic à Application Insights lorsque vous publiez un service Azure Cloud Services à partir de Visual Studio. Créez la ressource Azure Application Insights à ce moment-là ou envoyez les données à une ressource Azure existante. 

La disponibilité, les performances, les échecs et l’utilisation d’Azure Cloud Services peuvent être surveillés par Application Insights. Vous pouvez ajouter des graphiques personnalisés à Application Insights afin d’afficher les données qui vous intéressent le plus. Vous pouvez collecter les données d’instance de rôle à l’aide du Kit de développement logiciel (SDK) Application Insights dans votre projet Azure Cloud Services. 

- [Activer les diagnostics dans Visual Studio avant le déploiement](/visualstudio/azure/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines?preserve-view=true#to-turn-on-diagnostics-in-visual-studio-before-deployment)

- [Afficher l'historique des modifications](../azure-monitor/platform/activity-log.md#view-change-history)

- [Application Insights pour le service cloud Azure (classique)](../azure-monitor/app/cloudservices.md)

- [Configurer les diagnostics pour le service cloud (classique) et les machines virtuelles Azure](/visualstudio/azure/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines?preserve-view=true&toc=%2fazure%2fcloud-services%2ftoc.json)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : vous pouvez utiliser une surveillance avancée avec Azure Cloud Services, ce qui permet d’échantillonner des mesures supplémentaires et de les collecter à intervalles de 5 minutes, de 1 heure et de 12 heures. Les données agrégées sont stockées dans des tables au sein d’un compte de stockage, et purgées après 10 jours. Cependant, le compte de stockage utilisé est configuré par rôle et vous pouvez utiliser différents comptes de stockage pour différents rôles. Vous configurez cela avec une chaîne de connexion dans les fichiers .csdef et .cscfg.

Notez que la surveillance avancée implique l’utilisation de l’extension Diagnostics Azure (et éventuellement du Kit de développement logiciel (SDK) Application Insights) sur le rôle que vous souhaitez surveiller. L’extension Diagnostics utilise un fichier de configuration (par rôle) nommé diagnostics.wadcfgx pour configurer les mesures de diagnostic analysées. L’extension Azure diagnostics collecte et stocke des données dans un compte de stockage Azure. Ces paramètres sont configurés dans les fichiers .wadcfgx, .csdef et .cscfg.

- [Introduction à la surveillance des services cloud](cloud-services-how-to-monitor.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Les modes de surveillance de base ou avancé sont disponibles pour Azure Cloud Services. Azure Cloud Services collecte automatiquement les données de surveillance de base (pourcentage de processeur, trafic réseau entrant/sortant, et lecture/écriture de disque) à partir d’une machine virtuelle hôte. Affichez les données d’analyse collectées sur les pages de vue d’ensemble et de métriques d’un service cloud dans le portail Azure. 

Activez les diagnostics dans Azure Cloud Services pour collecter des données de diagnostic telles que des journaux d’applications, des compteurs de performances, etc., tout en utilisant l’extension Diagnostics Azure. Activez ou mettez à jour la configuration des diagnostics sur un service cloud en cours d’exécution avec la cmdlet Set-AzureServiceDiagnosticsExtension, ou déployez automatiquement un service cloud avec l’extension Diagnostics. Vous pouvez également installer le Kit de développement logiciel (SDK) Application Insights. Envoyez les compteurs de performances à Azure Monitor.

L’extension Azure diagnostics collecte et stocke des données dans un compte de stockage Azure. Transférez les données de diagnostic vers l’Émulateur de stockage Microsoft Azure ou le service Stockage Azure, car elles ne sont pas stockées de manière permanente. Une fois les données dans le stockage, vous pouvez les afficher avec l’un des outils disponibles, tels que l’Explorateur de serveurs dans Visual Studio, l’Explorateur Stockage Microsoft Azure et Management Studio Azure. Configurez les métriques de diagnostics à surveiller à l’aide d’un fichier de configuration (par rôle) nommé diagnostics.wadcfgx dans l’extension Diagnostics. 

- [Introduction à la surveillance des services cloud](cloud-services-how-to-monitor.md)

- [Comment activer les diagnostics dans un rôle de travail – Intégrer avec une solution SIEM](../security-center/continuous-export.md)

- [Activer les diagnostics dans Azure Cloud Services à l’aide de PowerShell](cloud-services-diagnostics-powershell.md)

- [Stocker et afficher des données de diagnostic dans le stockage Azure](diagnostics-extension-to-storage.md?&amp;preserve-view=true)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : vous pouvez surveiller les données de journal Azure Cloud Services en les intégrant avec Azure Sentinel ou avec une solution SIEM tierce, en activant les alertes pour les activités anormales.

- [Intégrer avec une solution SIEM](../security-center/continuous-export.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Conseils** : Microsoft Antimalware pour Azure protège Azure Cloud Services et les machines virtuelles. Vous avez la possibilité de déployer des solutions de sécurité tierces supplémentaires, telles que des pare-feu d’applications web, des pare-feu réseau, des logiciels anti-programmes malveillants, des systèmes de détection et de prévention des intrusions (IDS ou IPS), etc.

- [Quelles sont les fonctionnalités et capacités IPS/ID et DDOS de base sur Azure ?](./cloud-services-configuration-and-management-faq.md?preserve-view=true#what-are-the-features-and-capabilities-that-azure-basic-ipsids-and-ddos-provides)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Microsoft vous recommande de gérer l’accès aux ressources Azure à l’aide du contrôle d’accès en fonction du rôle Azure (Azure RBAC). Toutefois, Azure Cloud Services ne prend pas en charge le modèle RBAC Azure, car il ne s’agit pas d’un service Azure Resource Manager et vous devez utiliser un abonnement classique.

Par défaut, dans Azure, les trois rôles d’administrateur d’abonnements classique sont Administrateur de comptes, Administrateur de services et Coadministrateur. 

Les administrateurs d’abonnements classiques ont un accès total à l’abonnement Azure. Ils peuvent gérer les ressources en utilisant le portail Azure, les API Azure Resource Manager et les API du modèle de déploiement classique. Le compte qui est utilisé pour l’inscription à Azure est automatiquement défini en tant qu’administrateur de compte et administrateur de services fédérés. Il est possible d’ajouter des coadministrateurs ultérieurement. 

L’administrateur de services et les coadministrateurs ont un accès équivalent aux utilisateurs qui ont reçu le rôle Propriétaire (rôle Azure) dans l’étendue de l’abonnement. Gérer les coadministrateurs ou affichez l’administrateur de service sous l’onglet Administrateurs classiques sur le portail Azure. 

Répertoriez les attributions de rôles pour l’administrateur de service et les coadministrateurs classiques avec la commande PowerShell suivante :

Get-AzRoleAssignment -IncludeClassicAdministrators

Passez en revue les différences entre les rôles d’administration d’abonnement classiques. 

- [Différences entre les trois rôles d’administration d’abonnement classiques](../role-based-access-control/rbac-and-directory-admin-roles.md#classic-subscription-administrator-roles)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : il est recommandé de créer des procédures de fonctionnement standard autour de l’utilisation de comptes d’administration dédiés, en fonction des rôles disponibles et des autorisations nécessaires pour exploiter et gérer les ressources Azure Cloud Services.

- [Différences entre les rôles d’administration d’abonnement classiques](../role-based-access-control/rbac-and-directory-admin-roles.md#classic-subscription-administrator-roles)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Conseils** : évitez de gérer des identités distinctes pour les applications qui s’exécutent sur Azure Cloud Services. Implémentez l’authentification unique pour éviter de demander aux utilisateurs de gérer plusieurs identités et informations d’identification.

- [Qu’est-ce que l’authentification unique ? (SSO)](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : il est recommandé d’utiliser une station de travail sécurisée et managée par Azure (également appelée station de travail à accès privilégié) pour les tâches d’administration qui requièrent des privilèges élevés.

- [Comprendre les stations de travail sécurisées gérées par Azure](../active-directory/devices/concept-azure-managed-workstation.md)

- [Procédure d’activation d’Azure AD MFA](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : utilisez les API REST du service cloud Azure pour inventorier vos ressources de service cloud Azure afin d’obtenir des informations sensibles. Interrogez les ressources de service cloud déployées pour obtenir les ressources de configuration et .pkg.

 Par exemple, quelques API sont répertoriées ci-dessous :

- Get Deployment : l’opération Get Deployment renvoie les informations de configuration, l’état, et les propriétés système d’un déploiement.
- Get Package : l’opération Get Package récupère un package de service cloud pour un déploiement, et stocke les fichiers du package dans le Stockage Blob Microsoft Azure.
- Get Cloud Service Properties : l’opération Get Cloud Service Properties récupère les propriétés du service cloud spécifié.

Consultez la documentation sur les API REST de service cloud Azure, et créez un processus pour la protection des données d’informations sensibles, en fonction des besoins de votre organisation.

- [Get Deployment](/rest/api/compute/cloudservices/rest-get-deployment)

- [Obtention des propriétés d'un service cloud](/rest/api/compute/cloudservices/rest-get-cloud-service-properties)

- [Obtention de package](/rest/api/compute/cloudservices/rest-get-package)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : implémentez une isolation en utilisant des abonnements et groupes d’administration distincts selon les domaines de sécurité, par exemple, le type d’environnement et le niveau de sensibilité des données pour Azure Cloud Services.

Vous pouvez également modifier l’élément « permissionLevel » dans l’élément de certificat du service cloud Azure pour spécifier les autorisations d’accès accordées aux processus de rôle. Si vous souhaitez que seuls les processus élevés puissent accéder à la clé privée, spécifiez une autorisation elevated. Une autorisation limitedOrElevated permet à tous les processus de rôle d’accéder à la clé privée. Les valeurs possibles sont limitedOrElevated ou elevated. La valeur par défaut est limitedOrElevated.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Schéma WebRole](schema-csdef-webrole.md#Certificate)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : il est recommandé d’utiliser une solution tierce de la Place de marché Azure dans les périmètres du réseau, qui surveille et bloque les transferts non autorisés d’informations sensibles tout en alertant les professionnels de la sécurité des informations.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : configurez TLS v2 pour Azure Cloud Services. Utilisez le portail Azure pour ajouter le certificat à votre déploiement Azure Cloud Services intermédiaire, et ajoutez les informations de certificat aux fichiers CSDEF et CSCFG des services. Ré-empaquetez votre application et mettez à jour votre déploiement intermédiaire pour utiliser le nouveau package. 

Utilisez des certificats de service dans Azure, qui sont associés à l’offre Azure Cloud Services pour permettre des communications sécurisées vers et depuis le service. Fournissez un certificat capable d’authentifier un point de terminaison HTTPS exposé. Définissez des certificats de service dans la définition de service du service cloud, et déployez-les automatiquement sur la machine virtuelle en exécutant une instance de votre rôle.

Authentifiez avec l’API de gestion avec des certificats de gestion. Ceux-ci vous permettent d’authentifier avec le modèle de déploiement classique. Un grand nombre de programmes et d’outils (comme Visual Studio ou le SDK Azure) utilisent ces certificats pour automatiser la configuration et le déploiement de divers services Azure. 

Pour plus d’informations, l’API Gestion des services Azure fournit un accès par programmation à la fonctionnalité de management des services disponible sur le portail Azure. Le Kit de développement logiciel (SDK) Azure pour Python peut être utilisé pour gérer l’offre Azure Cloud Services et les comptes de stockage Azure. Le Kit de développement logiciel (SDK) Azure pour Python inclut l’API Gestion des services qui est une API REST. Toutes les opérations de l’API sont effectuées au moyen du protocole TLS et sont mutuellement authentifiées au moyen de certificats X.509 v3. Le service de gestion est accessible à partir d’un service s’exécutant dans Azure. Il est également accessible directement via Internet à partir de toute application capable d’envoyer une demande HTTPS et de recevoir une réponse HTTPS.

- [Configurer TLS pour une application dans Azure](cloud-services-configure-ssl-certificate-portal.md)

- [Utiliser le management des services de Python](cloud-services-python-how-to-use-service-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : il est recommandé d’utiliser un outil de découverte actif tiers pour identifier toutes les informations sensibles stockées, traitées ou transmises par les systèmes technologiques de l’organisation, qu’elles soient situées sur site ou chez un fournisseur de services distant, puis de mettre à jour l’inventaire des informations sensibles de l’organisation.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils** : non applicable au service cloud (classique). Cette recommandation n’applique pas de protection contre la perte de données.

Il est recommandé d’implémenter un outil tiers, tel qu’une solution automatisée de prévention contre la perte de données basée sur un hôte, pour appliquer des contrôles d’accès aux données même lorsque celles-ci sont copiées à partir d’un système.

Pour la plateforme sous-jacente managée par Microsoft, Microsoft considère tout le contenu client comme sensible et met tout en œuvre pour empêcher la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : l’offre Azure Cloud Services ne prend pas en charge le chiffrement au repos. Cela est dû au fait qu’elle est conçue pour être sans état. L’offre Azure Cloud Services prend en charge le stockage externe, par exemple, le service Stockage Azure qui est par défaut chiffré au repos.  

Les données d’application stockées sur des disques temporaires ne sont pas chiffrées. Le client est chargé de gérer et de chiffrer ces données en fonction des besoins.  

- [Présentation du chiffrement au repos dans Azure](../security/fundamentals/encryption-atrest.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : vous pouvez utiliser des alertes métriques classiques dans Azure Monitor pour être averti quand une de vos mesures appliquées à des ressources critiques franchit un seuil. Les alertes métriques classiques sont une ancienne fonctionnalité permettant de générer des alertes uniquement sur des métriques sans dimensions. Il existe une fonctionnalité plus récente et plus avancée : alertes de métrique. 

En outre, la fonctionnalité Application Insights vous permet de surveiller la disponibilité, les performances, les échecs et l’utilisation des applications Azure Cloud Services. Elle utilise des données combinées de kits de développement logiciel (SDK) Application Insights avec des données de Diagnostics Azure provenant de vos Azure Cloud Services.

- [Créer, afficher et gérer des alertes de métrique classiques à l’aide d’Azure Monitor](../azure-monitor/platform/alerts-classic-portal.md)

- [Vue d’ensemble des alertes métriques](../azure-monitor/platform/alerts-metric-overview.md) 

- [Application Insights pour le service cloud Azure (classique)](../azure-monitor/app/cloudservices.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils** : notez que ces informations ont trait au système d’exploitation invité Azure pour les rôles web et de travail Azure Cloud Services avec PaaS (Platform as a Service). Toutefois, elles ne s’appliquent pas aux machines virtuelles avec IaaS (Infrastructure as a Service).

Par défaut, Azure met à jour périodiquement le système d’exploitation invité du client vers la dernière image prise en charge au sein de la famille de systèmes d’exploitation spécifiée dans sa configuration de service (.cscfg), par exemple, Windows Server 2016.

Quand un client choisit une version de système d’exploitation spécifique pour son déploiement d’Azure Cloud Services, il désactive les mises à jour automatiques du système d’exploitation et assume la responsabilité de la mise à jour corrective. Le client doit veiller à que ses instances de rôle reçoivent des mises à jour ou puissent exposer leur application à des failles de sécurité.

- [Système d’exploitation invité d'Azure](cloud-services-guestos-msrc-releases.md)

- [Prise en charge et stratégie de suppression du SE invité d’Azure](cloud-services-guestos-retirement-policy.md)

- [Configuration de service cloud (classique)](cloud-services-how-to-configure-portal.md)

- [Gestion de la version de SE invité](cloud-services-how-to-configure-portal.md#manage-guest-os-version)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="53-deploy-an-automated-patch-management-solution-for-third-party-software-titles"></a>5.3 : Déployer une solution de gestion automatisée des correctifs de logiciels tiers

**Conseils** : Utilisez une solution tierce de gestion des correctifs. Les clients qui utilisent déjà Configuration Manager dans leur environnement peuvent également tirer parti de l’éditeur de mise à jour de System Center, qui leur permet de publier des mises à jour personnalisées dans le service Windows Server Update. 

Cela permet à la solution Update Management d’appliquer des correctifs aux machines qui utilisent Configuration Manager en tant que référentiel de mise à jour avec des logiciels tiers.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : il est recommandé qu’un client comprenne l’étendue de son exposition à une attaque DDoS de manière continue. 

Nous vous suggérons de considérer les scénarios suivants :

- Quelles nouvelles ressources Azure disponibles publiquement doivent être protégées ?
- Existe-t-il un point de défaillance unique dans le service ?
- Comment faire pour isoler les services de manière à limiter l’impact d’une attaque tout en les rendant accessibles aux clients valides ?
- Y a-t-il des réseaux virtuels dans lesquels le service DDoS Protection Standard n’est pas activé alors qu’il devrait l’être ?
- Mes services sont-ils actifs/actifs avec basculement dans plusieurs régions ?

Documents associés :

- [Évaluation des risques de vos ressources Azure](../security/fundamentals/ddos-best-practices.md#risk-evaluation-of-your-azure-resources)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : non applicable à l’offre Azure Cloud Services. Cette recommandation s’applique aux ressources de calcul IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : il est recommandé de rapprocher régulièrement l’inventaire et de veiller à ce que les ressources non autorisées soient supprimées de l’abonnement en temps utile.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-an-inventory-of-approved-azure-resources"></a>6.4 : Dresser et tenir un inventaire des ressources Azure approuvées

**Conseils** : Le client doit définir des ressources Azure et des logiciels approuvés pour les ressources de calcul.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils** : utilisez la fonctionnalité de contrôle d’application adaptatif disponible dans Azure Security Center. Il s’agit d’une solution de bout en bout intelligente et automatisée d’Azure Security Center qui vous permet de contrôler les applications qui peuvent s’exécuter sur vos ordinateurs Windows et Linux, tant Azure que non-Azure. Elle permet également de renforcer vos ordinateurs contre les programmes malveillants. 

Cette fonctionnalité est disponible pour les machines et les serveurs Azure et non-Azure Windows (toutes versions, classique ou Azure Resource Manager), mais aussi Linux.

Security Center utilise le Machine Learning pour analyser les applications en cours d’exécution sur vos machines et tire parti de ces informations pour créer une liste verte. Cette fonctionnalité simplifie considérablement le processus de configuration et de gestion des stratégies de mise en liste verte des applications, ce qui vous permet d’effectuer les applications suivantes :
- Bloquer ou vous alerter si une application malveillante tente de s’exécuter, y compris celles pouvant être omises par les solutions de logiciels anti-programme malveillant.

- Respecter la stratégie de sécurité de votre organisation qui autorise uniquement l’utilisation de certains logiciels licenciés.
- Empêcher l’utilisation de logiciels indésirables sur votre environnement.
- Empêcher l’exécution des applications obsolètes et non prises en charge.
- Bloquer des outils logiciels spécifiques qui ne sont pas autorisés dans votre organisation.
- Autoriser le service informatique à contrôler l’accès aux données sensibles en utilisant l’application.

Pour plus d’informations, consultez les liens référencés.

- [Contrôles d’application adaptative](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : utilisez la fonctionnalité de contrôle d’application adaptatif disponible dans Azure Security Center. Il s’agit d’une solution de bout en bout intelligente et automatisée d’Azure Security Center qui vous permet de contrôler les applications qui peuvent s’exécuter sur vos ordinateurs Windows et Linux, tant Azure que non-Azure. Elle permet également de renforcer vos ordinateurs contre les programmes malveillants. 

Cette fonctionnalité est disponible pour les machines et les serveurs Azure et non-Azure Windows (toutes versions, classique ou Azure Resource Manager), mais aussi Linux.

Security Center utilise le Machine Learning pour analyser les applications en cours d’exécution sur vos machines et tire parti de ces informations pour créer une liste verte. Cette fonctionnalité simplifie considérablement le processus de configuration et de gestion des stratégies de mise en liste verte des applications, ce qui vous permet d’effectuer les applications suivantes :

- Bloquer ou vous alerter si une application malveillante tente de s’exécuter, y compris celles pouvant être omises par les solutions de logiciels anti-programme malveillant.

- Respecter la stratégie de sécurité de votre organisation qui autorise uniquement l’utilisation de certains logiciels licenciés.

- Empêcher l’utilisation de logiciels indésirables sur votre environnement.

- Empêcher l’exécution des applications obsolètes et non prises en charge.

- Bloquer des outils logiciels spécifiques qui ne sont pas autorisés dans votre organisation.

- Autoriser le service informatique à contrôler l’accès aux données sensibles en utilisant l’application.

Pour plus d’informations, consultez les liens référencés.

- [Contrôles d’application adaptative](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : utilisez la fonctionnalité de contrôle d’application adaptatif disponible dans Azure Security Center. Il s’agit d’une solution de bout en bout intelligente et automatisée d’Azure Security Center qui vous permet de contrôler les applications qui peuvent s’exécuter sur vos ordinateurs Windows et Linux, tant Azure que non-Azure. Elle permet également de renforcer vos ordinateurs contre les programmes malveillants. 

Cette fonctionnalité est disponible pour les machines et les serveurs Azure et non-Azure Windows (toutes versions, classique ou Azure Resource Manager), mais aussi Linux.

Security Center utilise le Machine Learning pour analyser les applications en cours d’exécution sur vos machines et tire parti de ces informations pour créer une liste verte. Cette fonctionnalité simplifie considérablement le processus de configuration et de gestion des stratégies de mise en liste verte des applications, ce qui vous permet d’effectuer les applications suivantes :

- Bloquer ou vous alerter si une application malveillante tente de s’exécuter, y compris celles pouvant être omises par les solutions de logiciels anti-programme malveillant.

- Respecter la stratégie de sécurité de votre organisation qui autorise uniquement l’utilisation de certains logiciels licenciés.

- Empêcher l’utilisation de logiciels indésirables sur votre environnement.

- Empêcher l’exécution des applications obsolètes et non prises en charge.

- Bloquer des outils logiciels spécifiques qui ne sont pas autorisés dans votre organisation.

- Autoriser le service informatique à contrôler l’accès aux données sensibles en utilisant l’application.

Pour plus d’informations, consultez les liens référencés.

- [Contrôles d’application adaptative](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : utilisez la fonctionnalité de contrôle d’application adaptatif disponible dans Azure Security Center. Il s’agit d’une solution de bout en bout intelligente et automatisée d’Azure Security Center qui vous permet de contrôler les applications qui peuvent s’exécuter sur vos ordinateurs Windows et Linux, tant Azure que non-Azure. Elle permet également de renforcer vos ordinateurs contre les programmes malveillants. 

Cette fonctionnalité est disponible pour les machines et les serveurs Azure et non-Azure Windows (toutes versions, classique ou Azure Resource Manager), mais aussi Linux.

Security Center utilise le Machine Learning pour analyser les applications en cours d’exécution sur vos machines et tire parti de ces informations pour créer une liste verte. Cette fonctionnalité simplifie considérablement le processus de configuration et de gestion des stratégies de mise en liste verte des applications, ce qui vous permet d’effectuer les applications suivantes :

- Bloquer ou vous alerter si une application malveillante tente de s’exécuter, y compris celles pouvant être omises par les solutions de logiciels anti-programme malveillant.

- Respecter la stratégie de sécurité de votre organisation qui autorise uniquement l’utilisation de certains logiciels licenciés.

- Empêcher l’utilisation de logiciels indésirables sur votre environnement.

- Empêcher l’exécution des applications obsolètes et non prises en charge.

- Bloquer des outils logiciels spécifiques qui ne sont pas autorisés dans votre organisation.

- Autoriser le service informatique à contrôler l’accès aux données sensibles en utilisant l’application.

Pour plus d’informations, consultez les liens référencés.

- [Contrôles d’application adaptative](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Conseils** : utilisez la fonctionnalité de contrôle d’application adaptatif disponible dans Azure Security Center. Il s’agit d’une solution de bout en bout intelligente et automatisée d’Azure Security Center qui vous permet de contrôler les applications qui peuvent s’exécuter sur vos ordinateurs Windows et Linux, tant Azure que non-Azure. Elle permet également de renforcer vos ordinateurs contre les programmes malveillants. 

Cette fonctionnalité est disponible pour les machines et les serveurs Azure et non-Azure Windows (toutes versions, classique ou Azure Resource Manager), mais aussi Linux.

Security Center utilise le Machine Learning pour analyser les applications en cours d’exécution sur vos machines et tire parti de ces informations pour créer une liste verte. Cette fonctionnalité simplifie considérablement le processus de configuration et de gestion des stratégies de mise en liste verte des applications, ce qui vous permet d’effectuer les applications suivantes :
- Bloquer ou vous alerter si une application malveillante tente de s’exécuter, y compris celles pouvant être omises par les solutions de logiciels anti-programme malveillant.

- Respecter la stratégie de sécurité de votre organisation qui autorise uniquement l’utilisation de certains logiciels licenciés.
- Empêcher l’utilisation de logiciels indésirables sur votre environnement.
- Empêcher l’exécution des applications obsolètes et non prises en charge.
- Bloquer des outils logiciels spécifiques qui ne sont pas autorisés dans votre organisation.
- Autoriser le service informatique à contrôler l’accès aux données sensibles en utilisant l’application.

Pour plus d’informations, consultez les liens référencés.

- [Contrôles d’application adaptative](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-in-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : utilisez la fonctionnalité de contrôle d’application adaptatif disponible dans Azure Security Center. Il s’agit d’une solution de bout en bout intelligente et automatisée d’Azure Security Center qui vous permet de contrôler les applications qui peuvent s’exécuter sur vos ordinateurs Windows et Linux, tant Azure que non-Azure. Elle permet également de renforcer vos ordinateurs contre les programmes malveillants. 

Cette fonctionnalité est disponible pour les machines et les serveurs Azure et non-Azure Windows (toutes versions, classique ou Azure Resource Manager), mais aussi Linux.

Security Center utilise le Machine Learning pour analyser les applications en cours d’exécution sur vos machines et tire parti de ces informations pour créer une liste verte. Cette fonctionnalité simplifie considérablement le processus de configuration et de gestion des stratégies de mise en liste verte des applications, ce qui vous permet d’effectuer les applications suivantes :

- Bloquer ou vous alerter si une application malveillante tente de s’exécuter, y compris celles pouvant être omises par les solutions de logiciels anti-programme malveillant.

- Respecter la stratégie de sécurité de votre organisation qui autorise uniquement l’utilisation de certains logiciels licenciés.

- Empêcher l’utilisation de logiciels indésirables sur votre environnement.

- Empêcher l’exécution des applications obsolètes et non prises en charge.

- Bloquer des outils logiciels spécifiques qui ne sont pas autorisés dans votre organisation.

- Autoriser le service informatique à contrôler l’accès aux données sensibles en utilisant l’application.

Pour plus d’informations, consultez les liens référencés.

- [Contrôles d’application adaptative](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : pour des applications sensibles ou à haut risque avec Azure Cloud Services, implémentez des abonnements ou des groupes d’administration séparés pour assurer l’isolation.

Utilisez un groupe de sécurité réseau, créez une règle de sécurité de trafic entrant, choisissez un service tel que http, choisissez également un port personnalisé, et attribuez-lui une priorité et un nom. La priorité affecte l’ordre dans lequel les règles sont appliquées : plus la valeur numérique est faible, plus tôt la règle est appliquée. Vous devrez associer votre groupe de sécurité réseau à un sous-réseau ou à une interface réseau spécifique pour isoler ou segmenter le trafic réseau en fonction de vos besoins métier.

Pour plus d’informations, consultez les liens référencés.

- [Tutoriel : filtrer le trafic réseau avec un groupe de sécurité réseau à l’aide du portail Azure](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : suivez les recommandations d’Azure Security Center comme ligne de base de configuration sécurisée pour vos ressources Azure Cloud Services. 

Sur le portail Azure, choisissez Security Center, Calcul et applications, puis Azure Cloud Services pour voir les recommandations applicables à vos ressources de service.

- [Recommandations de sécurité - Guide de référence](../security-center/recommendations-reference.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : non applicable à l’offre Azure Cloud Services. Cette recommandation est basée sur le modèle de déploiement classique. Nous vous recommandons d’utiliser une solution tierce pour gérer les configurations de ressources Azure sécurisées.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils** : le fichier de configuration du service cloud Azure stocke les attributs d’opération d’une ressource. Vous pouvez stocker une copie des fichiers de configuration dans un compte de stockage sécurisé.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : non applicable à l’offre Azure Cloud Services. Cette recommandation étant basée sur le modèle de déploiement classique, elle ne peut pas être gérée par les outils de configuration basés sur le déploiement d’Azure Resource Manager.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="78-deploy-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer des outils de gestion de la configuration pour les systèmes d'exploitation

**Conseils** : non applicable à l’offre Azure Cloud Services. Cette recommandation s’applique aux ressources de calcul basées sur l’IaaS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Aide** : Utiliser Azure Security Center pour effectuer des analyses de ligne de base pour vos ressources Azure.  

- [Corriger les recommandations dans Azure Security Center](../security-center/security-center-remediate-recommendations.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils** : dans Azure Security Center, choisissez la fonctionnalité Calcul et applications, puis suivez les recommandations pour les machines virtuelles, les serveurs et les conteneurs.

- [Comprendre les recommandations concernant les conteneurs dans Azure Security Center](../security-center/container-security.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : l’offre Azure Cloud Services est basée sur un modèle de déploiement classique et ne s’intègre pas avec Azure Key Vault.

Vous pouvez sécuriser des secrets, telles les informations d’identification utilisées dans Azure Cloud Services, afin de ne pas avoir à taper un mot de passe à chaque fois. Pour commencer, spécifiez un mot de passe en texte brut, puis convertissez-le en chaîne sécurisée à l’aide de la cmdlet PowerShell ConvertTo-SecureString. Convertissez ensuite cette chaîne sécurisée en chaîne standard chiffrée à l’aide de la cmdlet ConvertFrom-SecureString.  Vous pouvez maintenant enregistrer cette chaîne standard chiffrée dans un fichier à l’aide de la cmdlet Set-Content.

En outre, il est recommandé de stocker les clés privées des certificats utilisés dans Azure Cloud Services dans un stockage sécurisé.

- [Configurer le Bureau à distance à partir de PowerShell](cloud-services-role-enable-remote-desktop-powershell.md#configure-remote-desktop-from-powershell)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : sécurisez des secrets, telles les informations d’identification utilisées dans Azure Cloud Services, afin de ne pas avoir à taper un mot de passe à chaque fois. 
 

Pour commencer, spécifiez un mot de passe en texte brut, puis convertissez-le en chaîne sécurisée à l’aide de la cmdlet PowerShell ConvertTo-SecureString. Convertissez ensuite cette chaîne sécurisée en chaîne standard chiffrée à l’aide de la cmdlet ConvertFrom-SecureString. Enregistrez à présent cette chaîne standard chiffrée dans un fichier à l’aide de la cmdlet Set-Content.

Stockez les clés privées des certificats utilisés dans Azure Cloud Services dans un emplacement de stockage sécurisé.

- [Configurer le Bureau à distance à partir de PowerShell](cloud-services-role-enable-remote-desktop-powershell.md#configure-remote-desktop-from-powershell)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-antimalware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils** : Microsoft Antimalware pour Azure est disponible les offres Azure Cloud Services et Machines virtuelles. La solution offre une protection en temps réel gratuite qui permet d’identifier et de supprimer les virus, logiciels espions et autres logiciels malveillants. Il fournit des alertes lorsqu’un logiciel malveillant ou indésirable connu essaye de s’installer ou de s’exécuter sur vos systèmes Azure. 

Utilisez la cmdlet Antimalware « Get-AzureServiceAntimalwareConfig » basée sur PowerShell pour obtenir la configuration du logiciel anti-programme malveillant.

Activez l’extension Antimalware avec un script PowerShell dans la tâche de démarrage dans Azure Cloud Services.

Choisissez la fonctionnalité de contrôle d’application adaptative dans Azure Security Center, une solution de bout en bout intelligente et automatisée. Elle renforce la sécurité de vos machines contre les programmes malveillants, et vous permet de bloquer ou de signaler des tentatives d’exécution d’applications malveillantes, y compris celles pouvant être omises par les solutions de logiciels anti-programme malveillant.

- [Comment puis-je ajouter une extension Antimalware de manière automatisée pour mon offre Azure Cloud Services ?](./cloud-services-configuration-and-management-faq.md?preserve-view=true#how-can-i-add-an-antimalware-extension-for-my-cloud-services-in-an-automated-way)

- [Scénarios de déploiement Antimalware](../security/fundamentals/antimalware.md#antimalware-deployment-scenarios)

- [Contrôles d’application adaptative](../security-center/security-center-adaptive-application.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils** : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

- [Comment configurer des automatisations de workflow dans Azure Security Center](../security-center/security-center-planning-and-operations-guide.md)

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process)

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process)

- [Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Aide** : Azure Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte. 

Marquez clairement les abonnements (par exemple, production, non-production) et créez un système de nommage pour identifier et classer les ressources Azure de façon claire.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins. 

- [Reportez-vous à la publication du NIST : Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus. 

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils** : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Security Center pour diffuser en continu les alertes vers Azure Sentinel. 

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