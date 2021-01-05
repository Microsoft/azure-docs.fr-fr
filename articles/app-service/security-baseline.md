---
title: Base de référence de sécurité Azure pour App Service
description: La base de référence de sécurité App Service fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: app-service
ms.topic: conceptual
ms.date: 11/17/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 3d84f0df44f17bdc7dd68aa9cae1f01b06d261b4
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96601639"
---
# <a name="azure-security-baseline-for-app-service"></a>Base de référence de sécurité Azure pour App Service

La base de référence de sécurité App Service contient des recommandations qui vous aideront à améliorer l’état de la sécurité de votre déploiement. La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview-v1.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques. Le contenu est regroupé par les **contrôles de sécurité** définis par le benchmark de sécurité Azure et les conseils associés applicables à App Service. Les **contrôles** non applicables à App Service ont été exclus.

Pour voir comment App Service est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité App Service](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-azure-resources-within-virtual-networks"></a>1.1 : Protéger les ressources Azure au sein des réseaux virtuels

**Conseils** : Lorsque vous utilisez App Service au niveau tarifaire Isolé, également appelé App Service Environment (ASE), vous pouvez le déployer directement dans un sous-réseau au sein de votre réseau virtuel Azure. Utilisez des groupes de sécurité réseau pour sécuriser votre instance Azure App Service Environment en bloquant le trafic entrant et sortant vers les ressources de votre réseau virtuel ou pour restreindre l’accès aux applications dans une instance App Service Environment.

Par défaut, les groupes de sécurité réseau incluent une règle de refus implicite au niveau de priorité le plus bas, ce qui vous oblige à ajouter explicitement les règles d’autorisation. Ajoutez des règles d’autorisation pour votre groupe de sécurité réseau selon une approche de mise en réseau du moindre privilège. Les machines virtuelles sous-jacentes utilisées pour héberger l’instance App Service Environment ne sont pas directement accessibles, car elles se trouvent dans un abonnement géré par Microsoft.

Protégez un environnement ASE en acheminant le trafic via Azure Application Gateway avec pare-feu d’applications web (WAF). Utilisez les points de terminaison de service conjointement avec Application Gateway pour sécuriser le trafic de publication entrant vers votre application.  

Dans l’instance App Service mutualisée (une application qui n’est pas au niveau isolé), utilisez des groupes de sécurité réseau pour bloquer le trafic sortant de votre application. Permettez à vos applications d’accéder aux ressources dans ou via un réseau virtuel grâce à la fonctionnalité Intégration du réseau virtuel. Cette fonctionnalité peut également être utilisée pour bloquer le trafic sortant de l’application vers des adresses publiques. La fonctionnalité Intégration du réseau virtuel ne peut pas être utilisée pour fournir un accès entrant à une application.  

Sécurisez le trafic entrant dans votre application avec :
- des restrictions d’accès : série de règles d’autorisation ou de refus qui contrôlent l’accès entrant.
- des points de terminaison de service : peuvent refuser le trafic entrant provenant de l’extérieur de réseaux virtuels ou sous-réseaux spécifiés.
- des points de terminaison privés : exposent votre application à votre réseau virtuel à l’aide d’une adresse IP privée. Lorsque les points de terminaison privés sont activés sur votre application, celle-ci n’est plus accessible sur Internet.

Lorsque vous utilisez la fonctionnalité Intégration du réseau virtuel avec des réseaux virtuels dans la même région, utilisez des groupes de sécurité réseau et des tables de routage avec des itinéraires définis par l’utilisateur. Les itinéraires définis par l’utilisateur peuvent être placés sur le sous-réseau d’intégration pour envoyer le trafic sortant comme prévu.  

Envisagez d’implémenter un pare-feu Azure pour créer, appliquer et consigner de manière centralisée des stratégies de connectivité réseau et d’application dans vos abonnements et réseaux virtuels. Pare-feu Azure utilise une IP publique statique pour vos ressources de réseau virtuel, ce qui permet aux pare-feu extérieurs d’identifier le trafic qui provient de votre réseau virtuel. 

- [Verrouiller un environnement App Service](environment/firewall-integration.md)

- [Protection contre les 10 principales vulnérabilités de l’OWASP (Open Web Application Security Project)](https://owasp.org/www-project-top-ten/)

- [Groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md)

- [Intégrer votre application à un réseau virtuel Azure](web-sites-integrate-with-vnet.md)

- [Considérations relatives à la mise en réseau pour un environnement App Service](environment/network-info.md)

- [Guide pratique pour créer un ASE externe](environment/create-external-ase.md)

- [Guide pratique pour créer un ASE interne](environment/create-ilb-ase.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-virtual-networks-subnets-and-network-interfaces"></a>1.2 : Superviser et journaliser la configuration et le trafic des réseaux virtuels, des sous-réseaux et des interfaces réseau

**Conseils** : Implémentez les recommandations de protection réseau d’Azure Security Center pour sécuriser les ressources réseau et les configurations associées à vos applications et API App Service.

Utilisez Pare-feu Azure pour envoyer du trafic et créer, appliquer et consigner des stratégies de connectivité réseau et d’application de façon centralisée entre les abonnements et les réseaux virtuels. Pare-feu Azure utilise une IP publique statique pour vos ressources de réseau virtuel, ce qui permet aux pare-feu extérieurs d’identifier le trafic qui provient de votre réseau virtuel. Le service Pare-feu Azure est également entièrement intégré à Azure Monitor pour la journalisation et l’analytique.

- [Vue d’ensemble du Pare-feu Azure](../firewall/overview.md)

- [Présentation de la sécurité réseau assurée par Azure Security Center](../security-center/security-center-network-recommendations.md)

- [Guide pratique pour activer la surveillance et la protection d’App Service](/azure/security-center/defender-for-app-service-introduction)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Pour sécuriser une application accessible sur Internet dans un environnement App Service Environment (ASE) :
- Déployez un pare-feu d’applications web (WAF) avec Azure Application Gateway devant une application accessible sur Internet.
- Utilisez des restrictions d’accès pour sécuriser le trafic entrant sur Application Gateway.
- Sécurisez l’application avec Azure Active Directory (Azure AD) pour garantir l’authentification.
- Définissez la version 1.2 comme version TLS minimale.
- Définissez l’application sur HTTPS uniquement.

Acheminez tout le trafic d’application sortant via un appareil Pare-feu Azure et surveillez les journaux. 

Pour sécuriser une application accessible sur Internet dans l’instance App Service mutualisée (par exemple, une application qui n’est pas dans le niveau isolé) :
- Déployez un appareil Web Application Firewall devant une application.
- Utilisez des restrictions d’accès ou des points de terminaison de service pour sécuriser le trafic entrant sur l’appareil Web Application Firewall (WAF).
- Sécurisez l’application avec Azure AD pour garantir l’authentification.
- Définissez la version 1.2 comme version TLS minimale.
- Définissez l’application sur HTTPS uniquement.
- Utilisez Intégration du réseau virtuel et le paramètre d’application WEBSITE_VIRTUAL NETWORK_ROUTE_ALL pour soumettre tout le trafic sortant aux groupes de sécurité réseau et aux itinéraires définis par l’utilisateur sur le sous-réseau d’intégration.

Comme pour l’application Application Service Environment, faites passer tout le trafic d’application sortant par un appareil Pare-feu Azure et surveillez les journaux de l’application.

En outre, examinez et suivez les recommandations du document « Verrouiller un environnement App Service ».

- [Verrouiller un environnement App Service](environment/firewall-integration.md)

- [Pare-feu d’applications web Azure sur Azure Application Gateway](../web-application-firewall/ag/ag-overview.md)

- [Restrictions d’accès dans Azure App Service](app-service-ip-restrictions.md)

- [Suivre les alertes WAF et surveiller facilement les tendances grâce à Azure Monitor](../azure-monitor/overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications avec des adresses IP connues comme étant malveillantes

**Conseils** : Sécurisez l’environnement App Service Environment comme décrit dans la documentation « Verrouiller un environnement App Service ». Appliquez la fonctionnalité Threat Intelligence intégrée d’Azure Security Center pour refuser les communications avec des IP publiques inutilisées ou connues comme étant malveillantes. Utilisez des restrictions d’accès pour sécuriser le trafic entrant sur Application Gateway. 

Sécurisez l’instance App Service mutualisée (une application qui n’est pas dans un niveau isolé) à l’aide d’un point de terminaison orienté vers l’Internet public. Il autorise le trafic provenant uniquement d’un sous-réseau spécifique au sein de votre réseau virtuel et bloque tout le reste. Utilisez les restrictions d’accès pour configurer les listes de contrôle d’accès (restrictions IP) du réseau pour verrouiller le trafic entrant autorisé.

Définissez la priorité dans la liste d’autorisation ou de refus ordonnée pour gérer l’accès réseau à votre application. Cette liste peut inclure des adresses IP ou des sous-réseaux de réseau virtuel. Une règle implicite « Tout refuser » se trouve à la fin de la liste lorsqu’elle contient une ou plusieurs entrées. La capacité peut être utilisée avec toutes les charges de travail hébergées par App Service, notamment les applications web, les applications API, les applications Linux, les applications de conteneur Linux et les fonctions. 

Utilisez des points de terminaison de service pour restreindre l’accès à votre application web à partir d’une instance Réseau virtuel Microsoft Azure. Limitez l’accès à une instance App Service mutualisée (une application qui n’est pas dans un niveau isolé) à partir de sous-réseaux sélectionnés à l’aide de points de terminaison de service. 

- [Restrictions d’adresse IP statique avec Azure App Service](app-service-ip-restrictions.md)

- [Pare-feu d’applications web Azure sur Azure Application Gateway](../web-application-firewall/ag/ag-overview.md)

- [Comment configurer un pare-feu d’applications web (WAF) pour App Service Environment](environment/app-service-app-service-environment-web-application-firewall.md)

- [Sécuriser l’environnement ASE comme décrit dans Verrouiller un environnement App Service](environment/firewall-integration.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="15-record-network-packets"></a>1.5 : Enregistrer les paquets réseau

**Conseils** : Supervisez les requêtes et les réponses envoyées vers et depuis des applications App Service avec Security Center. Les attaques contre une application web peuvent être surveillées à l’aide d’une passerelle applicative en temps réel dotée de Web Application Firewall, activée avec la journalisation intégrée à partir d’Azure Monitor pour suivre les alertes Web Application Firewall et surveiller facilement les tendances.

- [Pare-feu d’applications web Azure sur Azure Application Gateway](../web-application-firewall/ag/ag-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Gérez le trafic pour une application dans un environnement App Service Environment :

- Sécurisez l’environnement App Service Environment comme décrit dans « Verrouiller un environnement App Service ».
- Déployez une passerelle applicative dotée d’Azure Web Application Firewall devant vos applications accessibles sur Internet.
- Définissez l’application pour qu’elle soit accessible uniquement via HTTPS.

Gérez le trafic pour une application accessible sur Internet dans l’instance App Service mutualisée (pas dans le niveau isolé) : 

- Déployez une passerelle applicative sur laquelle Azure Web Application Firewall est activé devant vos applications accessibles sur Internet.
- Utilisez des restrictions d’accès ou des points de terminaison de service pour sécuriser le trafic entrant sur le pare-feu d’applications web. La capacité Restrictions d’accès peut être utilisée avec toutes les charges de travail hébergées par App Service, notamment les applications web, les applications API, les applications Linux, les applications de conteneur Linux et les fonctions.

- Définissez l’application pour qu’elle soit accessible uniquement via HTTPS.
- Limitez l’accès à votre application App Service avec des restrictions d’adresses IP statiques pour qu’elle reçoive uniquement le trafic provenant de l’adresse IP virtuelle sur une passerelle applicative comme seule adresse ayant accès.

Pour plus d’informations, consultez les liens référencés.

- [Restrictions d’adresse IP statique avec Azure App Service](app-service-ip-restrictions.md)

- [Pare-feu d’applications web Azure sur Azure Application Gateway](../web-application-firewall/ag/ag-overview.md)

- [Guide pratique pour la configuration du chiffrement TLS de bout en bout avec Application Gateway et le portail](../application-gateway/end-to-end-ssl-portal.md)

- [Sécuriser l’environnement ASE comme décrit dans Verrouiller un environnement App Service](/azure/app-service/environment/firewall-integration)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : App Service a plusieurs points de terminaison qui sont utilisés pour gérer le service. Ces adresses de point de terminaison sont également incluses dans l’étiquette de service IP AppServiceManagement. L’étiquette AppServiceManagement est utilisée avec un App Service Environnement uniquement dans le but d’autoriser ce trafic. 

Vous pouvez autoriser ou refuser le trafic pour le service correspondant en spécifiant le nom de l’étiquette de service dans le champ de source ou de destination approprié d’une règle. Les adresses App Service entrantes sont suivies dans l’étiquette de service d’adresse IP App Service. Il n’y a aucune balise de service d’adresse IP qui contient les adresses sortantes utilisées par App Service.

Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

- [Balises de service du réseau virtuel](../virtual-network/service-tags-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les paramètres réseau liés à vos applications App Service. 

Gérez les configurations de sécurité à l’aide d’alias Azure Policy dans les espaces de noms « Microsoft.Web » et « Microsoft.Network ». Créez des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos applications App Service. 

Utilisez des définitions de stratégie intégrées pour App Service, par exemple :
- L’application doit utiliser un point de terminaison de service de réseau virtuel.
- L’application doit uniquement être accessible via HTTPS.
- Définissez la version actuelle comme version TLS actuelle.

Pour plus d’informations, consultez les liens référencés.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour la configuration du chiffrement TLS de bout en bout avec Application Gateway et le portail](../application-gateway/end-to-end-ssl-portal.md)

- [Sécuriser l’environnement ASE comme décrit dans Verrouiller un environnement App Service](/azure/app-service/environment/firewall-integration)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Utilisez des étiquettes pour les groupes de sécurité réseau et d’autres ressources associées, notamment le flux de trafic dans App Service.

Utilisez le champ « Description » afin de spécifier le besoin métier, la durée, etc., pour toutes les règles qui autorisent le trafic vers et depuis un réseau pour les règles des différents groupes de sécurité du réseau.

Appliquez n’importe laquelle des définitions Azure Policy intégrées en lien avec les effets de l’étiquetage, comme « Exiger une étiquette et sa valeur », pour vous assurer que toutes les ressources sont créées avec étiquettes et pour vous informer de l’existence de toute ressource non étiquetée. Utilisez Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

- [Restrictions d’accès dans Azure App Service](/azure/app-service/app-service-ip-restrictions)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal d’activité Azure pour surveiller les configurations des ressources réseau et détecter les modifications apportées aux paramètres réseau et à toutes les ressources relatives à App Service. 

Appliquez l’une des définitions Azure Policy intégrées pour App Service, par exemple une stratégie qui audite les applications pour l’utilisation du service de point de terminaison de réseau virtuel. Créez des alertes dans Azure Monitor qui se déclenchent lorsque des modifications sont apportées à des paramètres ou ressources réseau critiques. 

Passez en revue les alertes de sécurité détaillées et les recommandations dans Security Center, sur le portail ou par le biais d’outils de programmation. Exportez ces informations ou envoyez-les à d’autres outils de supervision dans votre environnement. Des outils sont disponibles pour exporter les alertes et les recommandations manuellement ou automatiquement et de manière continue. Grâce à ces outils, vous pouvez :
 
- Exporter en continu vers un espace de travail Log Analytics
- Exporter en continu vers des Azure Event Hubs (pour les intégrations à des solutions SIEM tierces)
- Exporter dans un fichier CSV (une fois)

Il est recommandé de créer un processus avec des outils automatisés pour surveiller les configurations des ressources réseau et détecter rapidement les modifications.

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

- [Exporter les alertes et recommandations de sécurité](../security-center/continuous-export.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Intégrez votre App Service Environment (ASE) à Azure Monitor pour envoyer des journaux à Stockage Azure, Azure Event Hubs ou Log Analytics. Activez les paramètres de diagnostic des journaux d’activité Azure pour la journalisation d’audit du plan de contrôle. Les alertes de sécurité de Security Center sont publiées dans le journal d’activité Azure. Auditez les données des journaux d’activité Azure, ce qui vous permet de déterminer les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle pour Azure App Service et d’autres ressources Azure. Enregistrez vos requêtes pour une utilisation ultérieure, épinglez les résultats des requêtes aux tableaux de bord Azure et créez des alertes de journal. Utilisez également l’API REST d’accès aux données dans Application Insights pour accéder à vos données de télémétrie par programmation.

Utilisez Microsoft Azure Sentinel, une solution d’Informations de sécurité et gestion d’événements (SIEM) évolutive et native Cloud, pour vous connecter à diverses sources de données et à divers connecteurs, en fonction des besoins de votre entreprise. Vous pouvez également activer et intégrer des données à un système SIEM tiers, tel que Barracuda dans Place de marché Azure.

- [Journalisation de l’activité ASE](environment/using-an-ase.md#logging)

- [Guide pratique pour activer les paramètres de diagnostic pour Azure App Service](troubleshoot-diagnostic-logs.md)

- [Procédure d’activation d’Application Insights](../azure-monitor/app/app-insights-overview.md)

- [Exporter la télémétrie depuis Application Insights](../azure-monitor/app/export-telemetry.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Activez les paramètres de diagnostic des journaux d’activité Azure pour la journalisation d’audit du plan de contrôle d’App Service. Envoyez les journaux à un espace de travail Log Analytics, un Event Hub Azure ou un compte de stockage Azure.
Les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle peuvent être déterminés à l’aide des données des journaux d’activité Azure pour App Service et d’autres ressources Azure.

En outre, Azure Key Vault fournit une gestion centralisée des secrets avec des stratégies d’accès et un historique d’audit. 

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Guide pratique pour activer les paramètres de diagnostic pour Azure App Service](troubleshoot-diagnostic-logs.md)

- [Opérations Azure Resource Manager](../role-based-access-control/resource-provider-operations.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Dans Azure Monitor, définissez la période de conservation des journaux pour les espaces de travail Log Analytics associés à vos ressources App Service conformément aux règles de conformité de votre organisation.
- [Guide pratique pour définir les paramètres de conservation des journaux](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Passez en revue les paramètres de diagnostic des journaux d’activité Azure dans vos ressources App Service avec les journaux envoyés à un espace de travail Log Analytics. Effectuez des requêtes dans Log Analytics pour rechercher des termes, identifier des tendances, analyser des modèles et fournir de nombreux autres insights basés sur les données collectées.

Utilisez Application Insights pour vos applications App Service et afin de collecter les données de journal, de performances et d’erreur. Affichez les données de télémétrie collectées par Application Insights dans le portail Azure.

Si vous avez déployé un pare-feu d’applications web (WAF), vous pouvez surveiller les attaques contre vos applications App Service à l’aide d’un journal en temps réel du pare-feu d’applications web. Ce journal est intégré à Azure Monitor pour suivre les alertes WAF et analyser facilement les tendances.

Utilisez Azure Sentinel, une solution SIEM évolutive et native Cloud, pour l’intégration avec diverses sources de données et connecteurs, selon les besoins. Si vous le souhaitez, vous pouvez activer et intégrer des données à une solution SIEM tierce dans Place de marché Azure.

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Procédure d’activation d’Application Insights](../azure-monitor/app/app-insights-overview.md)

- [Comment intégrer votre ASE à l’aide d’Azure Application Gateway (WAF)](environment/integrate-with-application-gateway.md)

- [Procédure d’intégration d’Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : Configurez Security Center dans votre abonnement Azure et examinez les alertes générées. Utilisez Azure Monitor pour transmettre les données de votre journal d’activité à un Event Hub où elles peuvent être lues par une solution SIEM, telle qu’Azure Sentinel. 

Surveillez les attaques contre vos applications App Service en utilisant un journal en temps réel du pare-feu d’applications web avec une instance Azure Web Application Firewall (WAF) déployée. Ce journal est intégré à Azure Monitor pour suivre les alertes WAF et analyser facilement les tendances.

- [Comment intégrer votre ASE à l’aide d’Azure Application Gateway (WAF)](environment/integrate-with-application-gateway.md)

- [Exporter les alertes et recommandations de sécurité](../security-center/continuous-export.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Azure Active Directory (Azure AD) comporte des rôles intégrés qui doivent être explicitement attribués et pouvoir être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration.

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?preserve-view=true&view=azureadps-2.0)

- [Guide pratique pour utiliser des identités managées avec App Service et Azure Functions](overview-managed-identity.md?context=azure%2Factive-directory%2Fmanaged-identities-azure-resources%2Fcontext%2Fmsi-context&amp;tabs=dotnet)

- [Ajouter ou supprimer des attributions de rôles Azure avec le portail Azure](../role-based-access-control/role-assignments-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Aide** : Azure Active Directory (Azure AD) n’intègre pas le concept des mots de passe par défaut. Il fournit au plan de contrôle un accès à App Service.

En règle générale, évitez d’implémenter des mots de passe par défaut pour l’accès utilisateur lors de la création de vos propres applications. Utilisez l’un des fournisseurs d’identité disponibles par défaut pour App Service, comme Azure AD, un compte Microsoft, Facebook, Google ou Twitter.

Désactivez l’accès anonyme, sauf si vous avez besoin de le prendre en charge. 

- [Fournisseurs d’identité disponibles par défaut dans Azure App Service](overview-authentication-authorization.md#identity-providers)

- [Authentification et autorisation dans Azure App Service et Azure Functions](overview-authentication-authorization.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez les fonctionnalités de gestion des identités et des accès de Security Center pour superviser et suivre le nombre de comptes d’administration. 

Utilisez les recommandations de Security Center ou des stratégies Azure intégrées. Par exemple :

- Plusieurs propriétaires doivent être attribués à votre abonnement 
- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement
- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

Créez un processus pour surveiller les configurations des ressources réseau et détecter les modifications apportées aux comptes d’administration.

- [Utilisation d’Azure Security Center pour surveiller l’identité et l’accès](../security-center/security-center-identity-access.md)

- [Utilisation d’Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [En savoir plus sur l’octroi de l’accès aux applications aux utilisateurs](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="34-use-azure-active-directory-single-sign-on-sso"></a>3.4 : Utiliser l’authentification unique (SSO) Azure Active Directory

**Conseils** : Authentifiez App Service via Azure Active Directory (Azure AD). Il fournit un service OAuth 2.0 pour votre fournisseur d’identité et permet un accès autorisé aux applications mobiles et web. 

Les applications App Service utilisent l’identité fédérée, dans laquelle un fournisseur d’identité tiers gère automatiquement l’identité des utilisateurs et le flux d’authentification. Ces fournisseurs d’identité sont disponibles par défaut :

- Azure AD
- Compte Microsoft

- Facebook

- Google

- Twitter

Lorsque l’authentification et l’autorisation sont activées avec un de ces fournisseurs, son point de terminaison de connexion est accessible à des fins d’authentification de l’utilisateur et de validation des jetons d’authentification provenant du fournisseur.

- [Comprendre l’authentification et l’autorisation dans Azure App Service](overview-authentication-authorization.md#identity-providers)

- [En savoir plus sur l’authentification et l’autorisation dans Azure App Service](overview-authentication-authorization.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Activez la fonctionnalité d’authentification multifacteur dans Azure Active Directory (Azure AD) et suivez les recommandations relatives à la gestion des identités et des accès dans Security Center.

Implémentez l’authentification multifacteur pour Azure AD. Les administrateurs doivent s’assurer que les comptes d’abonnement du portail sont protégés. L'abonnement gérant les ressources que vous créez, il est vulnérable aux attaques. 

- [Sécurité Azure : l’authentification multifacteur](/previous-versions/azure/security/develop/secure-aad-app)

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-secure-azure-managed-workstations-for-administrative-tasks"></a>3.6 : Utiliser des stations de travail sécurisées et gérées par Azure pour les tâches administratives

**Conseils** : Utilisez des stations de travail disposant d’un accès privilégié avec l’authentification multifacteur configurée pour vous connecter aux ressources Azure et les configurer.

- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations)

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Conseils** : Utilisez Privileged Identity Management (PIM) dans Azure Active Directory (Azure AD) pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez également les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

La protection contre les menaces dans Security Center fournit des défenses complètes pour votre environnement, notamment la protection contre les menaces pour les ressources de calcul Azure telles que les machines Windows, les machines Linux, App Service et les conteneurs Azure.

- [Déploiement de Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-deployment-plan.md)

- [Présentation des détections de risques Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Protection contre les menaces pour les ressources de calcul Azure](../security-center/azure-defender.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Utilisez des emplacements nommés avec accès conditionnel pour autoriser l’accès au portail Azure uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP, de pays ou de régions.

- [Guide pratique pour configurer des emplacements nommés dans Azure](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils** : Utilisez Azure Active Directory (Azure AD) comme système central d’authentification et d’autorisation pour vos applications App Service. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit, mais aussi en salant, en hachant et en stockant de manière sécurisée les informations d’identification des utilisateurs.

- [Guide pratique pour configurer vos applications Azure App Service afin d’utiliser la connexion Azure AD](configure-authentication-provider-aad.md)

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Conseils** : Découvrez les comptes obsolètes à l’aide des journaux fournis par Azure Active Directory (Azure AD). Utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. Vérifiez régulièrement l’accès utilisateur pour vous assurer que seuls les utilisateurs prévus disposent d’un accès continu. 

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Conseils** : Utilisez Azure Active Directory (Azure AD) comme système central d’authentification et d’autorisation pour vos applications App Service. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit, et il sale, hache et stocke de manière sécurisée les informations d’identification des utilisateurs.

L’accès aux sources de journaux d’activité, d’audit et d’événement à risque vous permet de les intégrer à Azure Sentinel ou à une solution SIEM tierce. Rationalisez le processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et de connexion à un espace de travail Log Analytics. Les alertes de journal souhaitées peuvent être configurées dans Log Analytics.

- [Guide pratique pour configurer vos applications Azure App Service afin d’utiliser la connexion Azure AD](configure-authentication-provider-aad.md)

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Procédure d’intégration d’Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12 : Alerter en cas d’écart de comportement de connexion à un compte

**Conseils** : Utilisez Azure Active Directory (Azure AD) comme système central d’authentification et d’autorisation pour vos applications App Service. 

Utilisez Azure AD Identity Protection pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur, telles que les écarts du comportement de connexion au compte sur le plan de contrôle avec le portail Azure. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen. 

- [Guide pratique pour configurer votre application Azure App Service afin d’utiliser la connexion Azure AD](configure-authentication-provider-aad.md)

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Conseils** : Non disponible pour App Service. Customer Lockbox n’est pas pris en charge pour Azure App Service.

- [Liste des services pris en charge pour Customer Lockbox](../security/fundamentals/customer-lockbox-overview.md#supported-services-and-scenarios-in-general-availability)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources App Service qui stockent ou traitent des informations sensibles.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Pour une instance App Service Environment, implémentez des abonnements et/ou des groupes d’administration distincts pour les environnements de développement, de test et de production. Vous pouvez isoler les applications qui traitent des informations sensibles des autres applications de la même manière. Déployez votre application App Service dans un réseau virtuel. Utilisez des groupes de sécurité réseau et des sous-réseaux pour isoler davantage les applications. 

Il existe deux types de déploiement pour un environnement App Service (ASE) : Les deux vous permettent d’isoler le trafic en fonction des besoins de votre entreprise.

- Environnement de service d’application externe : expose les applications hébergées par App Service Environment sur une adresse IP accessible via Internet.

-  Environnement de service d’application d’équilibreur de charge interne (ILB) : expose les applications hébergées par App Service Environment sur une adresse IP au sein de votre réseau virtuel. Le point de terminaison interne est un équilibreur de charge interne (ILB), d’où cette appellation d’ASE ILB. 

Pour l’instance App Service mutualisée (une application qui n’est pas au niveau isolé), utilisez Intégration du réseau virtuel pour l’accès de votre application aux ressources de votre réseau virtuel.  Utilisez un accès au site privé pour rendre une application accessible uniquement à partir d’un réseau privé, par exemple, à partir d’un réseau virtuel Azure. La fonctionnalité Intégration du réseau virtuel sert uniquement à passer des appels sortants de votre application vers votre réseau virtuel. La fonctionnalité Intégration du réseau virtuel se comporte différemment selon qu’elle est utilisée avec un réseau virtuel de la même région ou avec ceux d’autres régions. 
 
- [Considérations relatives à la mise en réseau pour un environnement App Service](environment/network-info.md)

- [Guide pratique pour créer un ASE externe](environment/create-external-ase.md)

- [Guide pratique pour créer un ASE interne](environment/create-ilb-ase.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils** : Bien que les fonctionnalités d’identification, de classification et de prévention contre la perte de données ne soient pas encore disponibles pour App Service, vous pouvez réduire le risque d’exfiltration des données du réseau virtuel en supprimant toutes les règles dans lesquelles la destination utilise une « étiquette » pour les services Internet ou Azure. 

Microsoft gère l’infrastructure sous-jacente d’App Service et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de vos données.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Conseils** : Utilisez la version minimale par défaut de TLS 1.2, configurée dans les paramètres TLS/SSL, pour chiffrer toutes les informations en transit. Assurez-vous également que toutes les demandes de connexion HTTP sont redirigées vers le protocole HTTPS.

- [Comprendre le chiffrement en transit pour les applications web Azure App Service](security-recommendations.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Conseils** : Actuellement non disponible. Les fonctionnalités d’identification, de classification et de protection contre la perte des données ne sont pas encore disponibles pour App Service. 

Étiquetez les applications App Service susceptibles de traiter des informations sensibles. Implémentez une solution tierce, si nécessaire à des fins de conformité.

Microsoft gère la plateforme sous-jacente, traite toutes les données client comme sensibles et déploie des efforts importants pour se prémunir contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="46-use-role-based-access-control-to-control-access-to-resources"></a>4.6 : Utiliser le contrôle d’accès en fonction du rôle pour contrôler l’accès aux ressources

**Conseils** : Utilisez le contrôle d’accès en fonction du rôle (Azure RBAC) dans Azure Active Directory (Azure AD) pour contrôler l’accès au plan de contrôle App Service au niveau du portail Azure.

- [Guide pratique pour configurer le contrôle RBAC dans Azure](../role-based-access-control/role-assignments-portal.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Le contenu de site web dans une application App Service, comme les fichiers, est stocké dans Stockage Azure, qui chiffre automatiquement le contenu au repos. Choisissez de stocker les secrets d’application dans Key Vault et de les récupérer au moment de l’exécution.

Les secrets fournis par le client sont chiffrés au repos lorsqu’ils sont stockés dans des bases de données de configuration d’App Service.

Notez que, même si les disques attachés localement peuvent être utilisés en option par des sites web comme stockage temporaire (par exemple, D:\local et %TMP%), ils ne sont pas chiffrés au repos.

- [Présentation des contrôles de protection des données pour Azure App Service]()

- [Présentation du chiffrement Stockage Azure au repos](../storage/common/storage-service-encryption.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modification des applications de production App Service et d’autres ressources critiques ou associées.

- [Guide pratique pour créer des alertes sur les événements du journal d’activité Azure](../azure-monitor/platform/alerts-activity-log.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Conseils** : Adoptez une pratique DevSecOps pour garantir que vos applications App Service sont sécurisées et le restent pendant toute la durée de leur cycle de vie. DevSecOps intègre l’équipe de sécurité de votre organisation et ses capacités dans vos pratiques DevOps, faisant ainsi de la sécurité la responsabilité de tous les membres de l’équipe.

Examinez et suivez les recommandations de Security Center pour sécuriser vos applications App Service.

- [Guide pratique pour ajouter une validation de sécurité continue à votre pipeline CI/CD](/azure/devops/migrate/security-validation-cicd-pipeline?preserve-view=true&view=azure-devops)

- [Implémenter les recommandations d'évaluation des vulnérabilités d'Azure Security Center](../security-center/deploy-vulnerability-assessment-vm.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils** : Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge App Service. Toutefois, vous pouvez utiliser la gravité des recommandations dans Security Center ainsi que le score de sécurité pour mesurer les risques au sein de votre environnement. Votre niveau de sécurité est basé sur le nombre de recommandations Security Center que vous avez appliquées pour atténuer les risques. Pour classer par ordre de priorité les recommandations à résoudre en premier, réfléchissez à la gravité de chacune d’entre elles.

- [Guide de référence des recommandations de sécurité](../security-center/recommendations-reference.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : Utilisez Azure Resource Graph pour interroger ou découvrir toutes les ressources (telles que le calcul, le stockage, le réseau, les ports, les protocoles, etc.) dans vos abonnements. Assurez-vous que les autorisations appropriées sont appliquées à votre locataire et que vous pouvez répertorier tous les abonnements Azure, ainsi que les ressources qu’ils contiennent.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?preserve-view=true&view=azps-4.8.0)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Appliquez des balises aux ressources Azure en utilisant des métadonnées pour les organiser de façon logique par catégories.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration et des abonnements distincts, le cas échéant, pour organiser et suivre les ressources Azure. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de vos abonnements dans le cadre de ce processus.

Choisissez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements client en utilisant les définitions de stratégies intégrées suivantes :

- Types de ressources non autorisés
- Types de ressources autorisés

Pour plus d’informations, consultez les liens référencés.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Conseils** : Créez un inventaire des ressources Azure et logiciels approuvés pour les ressources de calcul en fonction des besoins de votre organisation.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Aide** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.

Utilisez Azure Resource Graph pour interroger ou découvrir des ressources dans leurs abonnements.  Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées. 

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils** : Utilisez Azure Resource Graph pour interroger ou découvrir des ressources au sein de vos abonnements et assurez-vous que les ressources Azure découvertes sont approuvées conformément aux stratégies de votre organisation.

Utilisez WebJobs dans App Service pour surveiller les applications logicielles non approuvées qui sont déployées dans des ressources de calcul. Utilisez WebJobs pour exécuter un programme ou un script dans une même instance, en tant qu’application web, application API ou application mobile. Définissez des configurations WebJobs et la supervision à l’aide de journaux. À la page WebJob Run Details, sélectionnez Activer/désactiver la sortie pour afficher le texte du contenu du journal. Notez que WebJobs n’est pas encore pris en charge pour App Service sur Linux.

- [Exécuter des tâches en arrière-plan avec WebJobs dans Azure App Service](webjobs-create.md)

- [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de l’explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md).

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées. Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements. Supprimez toutes les applications logicielles déployées qui n’ont pas été approuvées conformément aux stratégies de votre organisation.

- [Exécuter des tâches en arrière-plan avec WebJobs dans Azure App Service](webjobs-create.md)

- [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de l’explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md).

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils** : Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées. Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements. Supprimez toutes les applications logicielles déployées qui n’ont pas été approuvées conformément aux stratégies de votre organisation. 

- [Exécuter des tâches en arrière-plan avec WebJobs dans Azure App Service](webjobs-create.md)

- [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de l’explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md).

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Conseils** : Créez un processus pour réviser périodiquement les services Azure non autorisés pour vous assurer que seuls les services Azure autorisés sont utilisés dans vos abonnements.

Utilisez Azure Resource Graph, dans le cadre de ce processus, pour interroger ou découvrir des ressources dans vos abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

Configurez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements en utilisant les définitions de stratégie intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Utilisez WebJobs dans App Service pour surveiller les applications logicielles non approuvées qui sont déployées dans des ressources de calcul. Utilisez WebJobs pour exécuter un programme ou un script dans une même instance, en tant qu’application web, application API ou application mobile. Définissez des configurations WebJobs et la supervision à l’aide de journaux. À la page WebJob Run Details, sélectionnez Activer/désactiver la sortie pour afficher le texte du contenu du journal. Notez que WebJobs n’est pas encore pris en charge pour App Service sur Linux.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/built-in-policies.md#general)

- [Exécuter des tâches en arrière-plan avec WebJobs dans Azure App Service](webjobs-create.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="610-maintain-an-inventory-of-approved-software-titles"></a>6.10 : Tenir un inventaire des titres de logiciels approuvés

**Conseils** : Mettez en place un processus périodique d’inventaire et d’examen des titres de logiciels dans vos abonnements pour vous assurer que seuls les services Azure autorisés sont utilisés dans vos abonnements.

Utilisez Azure Resource Graph, dans le cadre de ce processus, pour interroger ou découvrir des ressources dans vos abonnements. Vérifiez que toutes les ressources Azure découvertes dans l’environnement sont approuvées.

Configurez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans les abonnements client en fonction des définitions de stratégies intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

De même, utilisez WebJobs dans App Service pour inventorier les applications logicielles non approuvées qui sont déployées dans des ressources de calcul. Définissez leur configuration et leur supervision à l’aide de journaux. À la page WebJob Run Details, sélectionnez Activer/désactiver la sortie pour afficher le texte du contenu du journal. Notez que WebJobs n’est pas encore pris en charge pour App Service sur Linux.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/built-in-policies.md#general)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Conseils** : Configurez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils** : WebJobs dans App Service permet aux clients d’exécuter un programme ou un script dans une même instance, en tant qu’application web, application API ou application mobile. Vous êtes tenu de définir votre configuration pour restreindre ou limiter les scripts qui ne sont pas autorisés par l’organisation. App Service ne fournit pas de mécanisme permettant de limiter l’exécution de scripts en mode natif. Notez que WebJobs n’est pas encore pris en charge pour App Service sur Linux.

- [Exécuter des tâches en arrière-plan avec WebJobs dans Azure App Service](webjobs-create.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Implémentez des abonnements ou des groupes d’administration distincts pour assurer l’isolation des applications App Service à risque élevé. Déployez une application à risque plus élevé dans son propre réseau virtuel, car la sécurité du périmètre dans App Service est obtenue par l’utilisation de réseaux virtuels. L’instance App Service Environment est un déploiement d’App Service dans un sous-réseau de votre réseau virtuel Azure. 

Il existe deux types d’environnement App Service Environment, l’environnement App Service Environment externe et l’environnement App Service Environment ILB (équilibreur de charge interne). Choisissez la meilleure architecture en fonction de vos besoins.

- [Considérations relatives à la mise en réseau pour un environnement App Service](environment/network-info.md)

- [Créer un environnement App Service externe](environment/create-external-ase.md)

- [Créer et utiliser un environnement App Service Environment avec équilibreur de charge interne](environment/create-ilb-ase.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour vos applications déployées App Service avec Azure Policy.

Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.Web » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de vos applications web App Service.

Appliquez des définitions de stratégie intégrées, telles que :
- App Service doit utiliser un point de terminaison de service de réseau virtuel
- Les applications web doivent être accessibles uniquement via HTTPS

- Utilisez la version TLS la plus récente dans vos applications

Il est recommandé de documenter le processus d’application des définitions de stratégie intégrées pour une utilisation standardisée.   

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?preserve-view=true&view=azps-4.8.0)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Utilisez les effets Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos applications Azure App Service.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des effets d’Azure Policy](../governance/policy/concepts/effects.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils** : Choisissez Azure DevOps ou Azure Repos pour stocker et gérer votre code en toute sécurité lorsque vous utilisez des définitions Azure Policy personnalisées.

Utilisez votre pipeline d’intégration continue (CI) et de déploiement continu (CD) existant pour déployer une configuration sécurisée éprouvée.

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?preserve-view=true&view=azure-devops)

- [Documentation Azure Repos](/azure/devops/repos/?preserve-view=true&view=azure-devops)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Conseils** : Utilisez les définitions Azure Policy intégrées ainsi que les alias Azure Policy dans l’espace de noms « Microsoft.Web » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. Développez un processus et un pipeline pour la gestion des exceptions de stratégie.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Conseils** : Utilisez les définitions Azure Policy intégrées ainsi que les alias Azure Policy dans l’espace de noms « Microsoft.Web » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. 

Appliquez les effets Azure Policy [auditer], [refuser] et [déployer s’il n’existe pas] afin d’appliquer automatiquement des configurations pour vos ressources Azure.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Utilisez des identités managées pour fournir aux applications App Service une identité gérée automatiquement dans Azure Active Directory (Azure AD). Les identités managées permettent à vos applications de s’authentifier auprès d’un service qui prend en charge l’authentification Azure AD, notamment Key Vault, sans informations d’identification dans votre code. Assurez-vous que la suppression réversible est activée dans Azure Key Vault.

- [Guide pratique pour activer la suppression réversible dans Azure Key Vault](../key-vault/general/key-vault-recovery.md)

- [Guide pratique pour utiliser des identités managées pour App Service](overview-managed-identity.md)

- [Fournir une authentification Key Vault avec une identité managée](../key-vault/general/assign-access-policy-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils** : Utilisez des identités managées pour fournir aux applications déployées App Service une identité gérée automatiquement dans Azure Active Directory (Azure AD). Les identités managées permettent à vos applications de s’authentifier auprès d’un service qui prend en charge l’authentification Azure AD, notamment Key Vault, sans informations d’identification dans votre code.

- [Guide pratique pour utiliser des identités managées pour App Service](overview-managed-identity.md)

- [Fournir une authentification Key Vault avec une identité managée](../key-vault/general/assign-access-policy-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-recovery"></a>Récupération de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : Garantir des sauvegardes automatiques régulières

**Conseils** : La fonctionnalité de sauvegarde et de restauration d’App Service vous permet de créer facilement des sauvegardes d’applications manuelles ou planifiées. Vous pouvez configurer les sauvegardes pour qu’elles soient conservées pendant une durée indéfinie. Vous pouvez restaurer l’application d’après la capture instantanée d’un état précédent en remplaçant l’application existante ou en restaurant sur une autre application.

App Service peut sauvegarder les informations suivantes dans un compte de stockage et un conteneur Azure, que vous avez configurés pour l’utilisation de votre application :
- la configuration d’une application ;
- le contenu d’un fichier ;
- la base de données connectée à votre application.

Assurez-vous que des sauvegardes régulières et automatisées sont effectuées à une fréquence définie par les stratégies de votre organisation.

- [Comprendre la capacité de sauvegarde d’Azure App Service](manage-backup.md)

- [Clés gérées par le client pour le chiffrement du service Stockage Azure](../storage/common/customer-managed-keys-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : Effectuer des sauvegardes complètes du système et sauvegarder les clés managées par le client

**Conseils** : Utilisez la fonctionnalité de sauvegarde et de restauration d’App Service pour sauvegarder vos applications. Les fonctionnalités de sauvegarde nécessitent un compte de stockage Azure pour stocker les informations de sauvegarde de votre application.

- Stockage Azure fournit un chiffrement au repos : utilisez des clés fournies par le système ou vos propres clés gérées par le client. C’est là que sont stockées vos données d’application lorsqu’elles ne sont pas exécutées dans une application web sur Azure.
- L’exécution à partir d’un package de déploiement est une fonctionnalité de déploiement d’App Service. Elle vous permet de déployer le contenu de votre site à partir d’un compte de stockage Azure, à l’aide d’une URL de signature d’accès partagé (SAS).

- Les références Key Vault sont une fonctionnalité de sécurité d’App Service. Elle vous permet d’importer des secrets au moment de l’exécution comme des paramètres d’application. Utilisez cette valeur pour chiffrer l’URL SAS de votre compte de stockage Azure.

Pour plus d’informations, consultez les liens référencés.

- [Sauvegarde de votre application dans Azure](manage-backup.md)

- [Restaurer une application en cours d’exécution dans Azure App Service](web-sites-restore.md)

- [Présentation du chiffrement au repos dans Azure](../security/fundamentals/encryption-atrest.md#encryption-at-rest-in-microsoft-cloud-services) 

- [Modèle de chiffrement et table de gestion des clés](../security/fundamentals/encryption-atrest.md)

- [Chiffrement au repos à l’aide de clés gérées par le client](configure-encrypt-at-rest-using-cmk.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Testez régulièrement le processus de restauration pour toutes les sauvegardes de vos applications App Service.

- [Sauvegarde de votre application dans Azure](manage-backup.md)

- [Comment restaurer une application web Azure App Service](web-sites-restore.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés managées par le client

**Conseils** : Les sauvegardes d’App Service sont stockées dans un compte de stockage Azure. Les données dans Stockage Azure sont chiffrées et déchiffrées en toute transparence à l’aide du chiffrement AES 256 bits, un des chiffrements par blocs les plus puissants actuellement disponibles, et sont conformes à la norme FIPS 140-2. Le chiffrement du Stockage Azure est similaire au chiffrement BitLocker sur Windows.

Le chiffrement de Stockage Azure est activé pour tous les comptes de stockage, y compris les comptes de stockage classiques et Resource Manager. Le chiffrement de Stockage Azure ne peut pas être désactivé. Étant donné que vos données sont sécurisées par défaut, vous n’avez pas besoin de modifier votre code ou vos applications pour tirer parti du chiffrement du Stockage Azure.

Par défaut, les données d’un compte de stockage sont chiffrées à l’aide de clés managées par Microsoft. Vous pouvez vous reposer sur les clés managées par Microsoft pour le chiffrement des vos données, ou vous pouvez gérer le chiffrement avec vos propres clés. Assurez-vous que la suppression réversible est activée dans Azure Key Vault.

- [Présentation du chiffrement de Stockage Azure pour les données au repos](../storage/common/storage-service-encryption.md)

- [Guide pratique pour activer la suppression réversible dans Azure Key Vault](../key-vault/general/key-vault-recovery.md)

**Supervision d’Azure Security Center** : Oui

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

**Conseils** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez clairement les abonnements (par exemple, production, non-production) et créez un système de nommage pour identifier et classer les ressources Azure de façon claire.

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les capacités de réponse aux incidents de votre système de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

- [Consultez la publication du NIST, Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Conseils** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé.  Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Aide** : Exportez vos alertes et recommandations de Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Utilisez le connecteur de données de Security Center pour diffuser en continu les alertes Sentinel en fonction des besoins de l’entreprise.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans le Centre de sécurité pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de Microsoft pour garantir que vos tests d’intrusion sont conformes aux stratégies de Microsoft : https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

Vous trouverez ici plus d’informations sur la stratégie de Microsoft, sur l’exécution de Red Teaming et sur les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)