---
title: Base de référence de sécurité pour Azure Web Application Firewall
description: La base de référence de sécurité Azure Web Application Firewall fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le benchmark de sécurité Azure.
author: msmbaldwin
ms.service: web-application-firewall
ms.topic: conceptual
ms.date: 10/13/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: cb5fae022957dd5fdc56058ed08b92734929d13e
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94562296"
---
# <a name="azure-security-baseline-for-azure-web-application-firewall"></a>Base de référence de sécurité pour Azure Web Application Firewall

Cette base de référence de sécurité applique les conseils du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview-v1.md) à Azure Web Application Firewall. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé par les **contrôles de sécurité** définis par le benchmark de sécurité Azure et les conseils associés applicables à Azure Web Application Firewall. Les **contrôles** non applicables à Azure Web Application Firewall ont été exclus. 

Pour voir comment Azure Web Application Firewall est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité Azure Web Application Firewall](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité du réseau

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils** : Utilisez Microsoft Azure Web Application Firewall (WAF) pour une protection centralisée des applications web contre les codes malveillants exploitant une faille de sécurité et les vulnérabilités telles que les injections de code SQL et le script de site à site. 

- Mode de détection : Utilisez ce mode pour comprendre le trafic réseau, comprendre et évaluer les faux positifs. Il surveille et journalise toutes les alertes de menace. Assurez-vous que Diagnostics et le journal WAF sont sélectionnés et activés. Notez que le WAF ne bloque pas les requêtes entrantes lorsqu’il opère en mode de détection.
- Mode de prévention : Bloque les intrusions et les attaques détectées par les règles. Un attaquant reçoit une exception « 403 Accès non autorisé » et la connexion est fermée. Le mode de prévention enregistre de telles attaques dans les journaux WAF.

Référencez votre trafic réseau avec le mode de détection de WAF. Après avoir déterminé vos besoins de trafic, configurez le WAF en mode de prévention pour bloquer le trafic indésirable.

Suivez les recommandations à gravité élevée du Centre de sécurité pour les ressources web qui ne sont pas protégées par le WAF.  

- [Règles et groupes de règles CRS de Web Application Firewall](ag/application-gateway-crs-rulegroups-rules.md) 

- [Modes WAF sur Application Gateway](ag/ag-overview.md#waf-modes)

- [Modes WAF sur Front Door](afds/afds-overview.md#waf-modes)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications avec des adresses IP connues comme étant malveillantes

**Conseils** : Utilisez des règles personnalisées avec Azure Web Application Firewall (WAF) pour autoriser et bloquer le trafic. Par exemple, tout le trafic entrant à partir d’une plage d’adresses IP peut être bloqué. Configurez Azure WAF pour qu’il s’exécute en mode de prévention, ce qui bloque les intrusions et les attaques détectées par les règles. Un attaquant reçoit une exception « 403 Accès non autorisé » et la connexion est fermée. Le mode de prévention enregistre de telles attaques dans les journaux WAF.

- [Modes WAF sur Application Gateway](ag/ag-overview.md#waf-modes)

- [Modes WAF sur Front Door](afds/afds-overview.md#waf-modes)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils** : Azure Web Application Firewall (WAF) est un composant essentiel des protections des applications web d’Azure. Utilisez Azure WAF pour assurer une protection centralisée des applications web contre les codes malveillants exploitant une faille de sécurité et les vulnérabilités avec des ensembles de règles gérées préconfigurées sur des signatures d’attaques connues issues de catégories OWASP TOP 10.

Personnalisez Azure WAF avec des règles et des groupes de règles pour les besoins d’application web et pour éliminer les faux positifs. Associez une stratégie Azure WAF pour chaque site situé derrière un WAF pour permettre une configuration spécifique du site. Azure WAF prend en charge le filtrage géographique, la limitation du débit et les règles d’ensemble de règles par défaut gérées par Azure. et des règles personnalisées peuvent être créées pour répondre aux besoins d’une application. 

Configurez Azure WAF pour qu’il s’exécute en mode de prévention après avoir référencé le trafic réseau en mode de détection pendant une période déterminée. Azure WAF bloque les intrusions et les attaques détectées par les règles en mode de prévention. Un attaquant reçoit une exception « 403 Accès non autorisé » et la connexion est fermée. Le mode de prévention enregistre de telles attaques dans les journaux WAF.

- [Modes WAF sur Application Gateway](ag/ag-overview.md#waf-modes)

- [Modes WAF sur Front Door](afds/afds-overview.md#waf-modes)

- [Règles et groupes de règles CRS de pare-feu d’applications web](ag/application-gateway-crs-rulegroups-rules.md?tabs=owasp31)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Conseils** : Créez, associez et organisez logiquement des ressources dans un abonnement Azure avec des balises pour la détection de mauvaises configurations d’application courantes (par exemple, Apache et IIS). 

Appliquez des règles et des groupes de règles aux stratégies Azure Web Application Firewall (WAF) en fonction des métadonnées de balise appliquées.

- [Stratégie WAF sur Application Gateway](/cli/azure/network/application-gateway/waf-policy?view=azure-cli-latest) 

- [Stratégie WAF sur Front Door](/cli/azure/ext/front-door/network/front-door/waf-policy?view=azure-cli-latest)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Conseils** : Utilisez des étiquettes pour les groupes de sécurité réseau associés à Azure Web Application Firewall (WAF) dans votre sous-réseau Azure Application Gateway et d’autres ressources liées à la sécurité réseau et au flux de trafic. Concernant les règles de groupe de sécurité réseau individuelles, utilisez le champ « Description » afin de spécifier le besoin métier, la durée, etc., pour toutes les règles qui autorisent le trafic vers et depuis un réseau.

Utilisez l’une des définitions Azure Policy intégrées en lien avec l’étiquetage, comme « Exiger une étiquette et sa valeur », pour vous assurer que toutes les ressources créées sont étiquetées et être informé de l’existence de ressources non étiquetées.

Choisissez Azure PowerShell ou Azure CLI pour rechercher des ressources ou effectuer des actions sur des ressources en fonction de leurs étiquettes.

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

- [Guide pratique pour créer un réseau virtuel](../virtual-network/quick-create-portal.md)

- [Guide pratique pour créer un groupe NSG avec une configuration de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils** : Utilisez le journal d’activité Azure pour superviser les configurations des ressources réseau et détecter les changements apportés aux paramètres et ressources réseau liés à vos déploiements d’Azure Web Application Firewall (WAF). Créez des alertes dans Azure Monitor qui se déclenchent lorsque des modifications sont apportées à des paramètres ou ressources réseau critiques.

- [Guide pratique pour consulter et récupérer les événements du journal d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log)

- [Guide pratique pour créer des alertes dans Azure Monitor](../azure-monitor/platform/alerts-activity-log.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-monitoring"></a>Enregistrement et surveillance

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Conseils** : Créez une règle de réseau pour Azure Web Application Firewall (WAF) afin d’autoriser l’accès à un serveur NTP avec le port et le protocole appropriés, tel que le port 123 sur UDP.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Conseils** : Configurez les journaux d’Azure Web Application Firewall (WAF) à envoyer à une solution de gestion des journaux de sécurité centralisée, telle qu’Azure Sentinel, ou une application SIEM tierce. Ces journaux incluent les journaux d’activité Azure, de diagnostic et de WAF en temps réel. Ces journaux peuvent ensuite être affichés dans différents outils, tels qu’Azure Monitor, Excel et Power BI. Les journaux d’Azure Web Application Firewall fournissent des informations sur ce qu’Azure WAF évalue, met en correspondance et bloque.

Azure Sentinel possède un classeur Azure WAF intégré qui fournit une vue d’ensemble des événements de sécurité sur Azure WAF. Ce classeur inclut les événements, les règles de correspondance et les règles bloquées ainsi que tout le reste de ce qui est consigné dans les journaux du pare-feu. Cette télémétrie peut être utilisée pour lancer l’automatisation du playbook pour notifier ou prendre des mesures correctives en fonction des événements WAF collectés par Sentinel.

- [Afficher les journaux d’activité](../azure-resource-manager/management/view-activity-logs.md)

- [Journaux de diagnostic pour Application Gateway](../application-gateway/application-gateway-diagnostics.md)

- [Connecter des données de pare-feu d’application web Microsoft à Azure Sentinel](../sentinel/connect-azure-waf.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Conseils** : Activez la journalisation sur vos ressources Azure Web Application Firewall (WAF) pour accéder aux journaux d’audit, de sécurité et de diagnostic. Azure Web Application Firewall fournit des rapports détaillés sur chacune des menaces détectées qui sont mises à disposition dans les journaux de diagnostic configurés. Les journaux d’activité, automatiquement disponibles, incluent la source de l’événement, la date, l’utilisateur, l’horodatage, les adresses sources, les adresses de destination et d’autres éléments utiles.

- [Vue d’ensemble de la journalisation](ag/ag-overview.md#logging)

- [Vue d’ensemble de la requête de journal Azure Monitor](../azure-monitor/log-query/log-query-overview.md)

- [Vue d’ensemble des journaux de plateforme Azure](../azure-monitor/platform/platform-logs-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils** : Envoyez les journaux d’Azure Web Application Firewall (WAF) à un compte de stockage personnalisé et définissez la stratégie de rétention. Utilisez Azure Monitor pour définir la période de rétention de votre espace de travail Log Analytics en fonction des exigences de conformité de votre organisation.
- [Configuration de la surveillance d'un compte de stockage](../storage/common/storage-monitor-storage-account.md#configure-logging)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Conseils** : Azure Web Application Firewall (WAF) fournit des rapports détaillés sur chaque menace détectée. La journalisation est intégrée aux journaux Diagnostics Azure qui fournisseur des informations détaillées sur les opérations et erreurs qui sont importantes pour l’audit et le dépannage. 

Les instances Azure WAF sont intégrées au Centre de sécurité pour envoyer des alertes et des informations d’intégrité pour la création de rapports. Utilisez les recommandations du Centre de sécurité pour détecter les applications web non protégées et protéger ces ressources vulnérables. 

Azure Sentinel possède un classeur d’événements WAF, de pare-feu, intégré qui fournit une vue d’ensemble des événements de sécurité sur WAF. Ceux-ci inclut les événements, les règles de correspondance et les règles bloquées ainsi que tout le reste de ce qui est consigné dans les journaux du pare-feu.

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/index.yml) 

- [Procédure d’activation des paramètres de diagnostic pour Azure Application Gateway](../application-gateway/application-gateway-diagnostics.md)

- [Supervision des journaux et des métriques dans Azure Front Door](../frontdoor/front-door-diagnostics.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="27-enable-alerts-for-anomalous-activities"></a>2.7 : Activer les alertes d’activité anormale

**Conseils** : Activez les paramètres de diagnostic du journal d’activité Azure pour votre Azure WAF et envoyez les journaux dans un espace de travail Log Analytics. Effectuez des requêtes dans Log Analytics pour rechercher des termes, identifier des tendances, analyser des modèles et fournir de nombreux autres insights basés sur les données collectées. Créez des alertes d’activité anormale en fonction des mesures WAF. Par exemple, si le nombre de requêtes bloquées dépasse « X », faire « Y ».

- [Guide pratique pour activer les paramètres de diagnostic du journal d’activité Azure](../azure-monitor/platform/activity-log.md)

- [Guide pratique pour créer des alertes dans Azure](../azure-monitor/learn/tutorial-response.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide** : Déployez un pare-feu d'applications web (WAF) Azure devant les applications web critiques pour bénéficier d'un contrôle supplémentaire du trafic entrant. 

Azure WAF fournit une protection centralisée de vos applications web contre les codes malveillants exploitant une faille de sécurité et les vulnérabilités, et sécurise vos applications en inspectant le trafic web entrant pour bloquer les attaques telles que les injections SQL, les scripts inter-sites, les téléchargements de programmes malveillants et les attaques DDoS.

- [Guide pratique pour déployer Azure WAF](ag/create-waf-policy-ag.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Conseils** : Azure Active Directory (Azure AD) comporte des rôles intégrés qui peuvent être interrogés et qui doivent être explicitement attribués. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration.

- [Guide pratique pour obtenir un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0)

- [Guide pratique pour obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils** : Aucune attribution d’administrateur local n’est disponible dans le WAF. Il peut toutefois y avoir des rôles d’administrateur Azure Active Directory (Azure AD) dans l’environnement, ce qui pourrait permettre de contrôler la gestion des ressources Azure WAF.
Une bonne pratique consiste à créer des procédures de fonctionnement standard autour de l’utilisation de comptes d’administration dédiés ayant accès aux instances Azure Web Application Firewall (WAF). Utilisez les fonctionnalités de gestion des identités et des accès de Security Center pour superviser le nombre de comptes d’administration.

- [Présentation de l’identité et de l’accès Azure Security Center](../security-center/security-center-identity-access.md)

- [Comprendre comment créer des utilisateurs administrateurs dans Azure Database pour PostgreSQL](../postgresql/howto-create-users.md#the-server-admin-account)

- [Utilisation d’Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Conseils** : Activez l’authentification multifacteur (MFA) Azure Active Directory (Azure AD) et suivez les recommandations relatives à la gestion des identités et des accès dans le Centre de sécurité.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Guide pratique pour superviser les identités et les accès dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils** : Utilisez des stations de travail avec accès privilégié (PAW) avec l’authentification multifacteur (MFA) configurée pour se connecter à Azure Web Application Firewall (WAF) et ressources associées et les configurer. 

- [En savoir plus sur les stations de travail à accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations) 

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="37-log-and-alert-on-suspicious-activities-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes sur des comptes d’administration

**Aide** : Utilisez les rapports de sécurité Azure Active Directory (Azure AD) pour générer des journaux et des alertes quand une activité suspecte ou potentiellement dangereuse se produit dans l’environnement. Utilisez le Centre de sécurité pour surveiller l’activité liée aux identités et aux accès.

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Conseils** : Configurez la condition d’emplacement d’une stratégie d’accès conditionnel et gérez vos emplacements nommés. 

Créez des regroupements logiques de plages d’adresses IP ou de pays et de régions avec des emplacements nommés. Restreignez l’accès à vos ressources sensibles, telles que vos secrets Key Vault, à vos emplacements nommés configurés.

- [Présentation de la condition d’emplacement dans l’accès conditionnel Azure Active Directory](../active-directory/reports-monitoring/quickstart-configure-named-locations.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Aide** : Utiliser Azure Active Directory (Azure AD) comme système d’authentification et d’autorisation central. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit, mais aussi en salant, en hachant et en stockant de manière sécurisée les informations d’identification des utilisateurs.
- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md) 

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide** : Azure Active Directory (Azure AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. Utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. Examinez régulièrement l’accès des utilisateurs pour vous assurer que seuls les utilisateurs actifs disposent d’un accès continu.

- [Présentation des rapports Azure AD](../active-directory/reports-monitoring/index.yml)

- [Comment utiliser les révisions d’accès des identités Azure](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="311-monitor-attempts-to-access-deactivated-credentials"></a>3.11 : Superviser les tentatives d’accès à des informations d’identification désactivées

**Conseils** : Implémentez l’intégration avec un outil SIEM ou de supervision quelconque tel qu’Azure Sentinel sur votre accès aux activités de connexion Azure Active Directory (Azure AD) et aux sources des journaux d’événements à risque et d’audit.

Simplifiez ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure Active Directory (Azure AD) et en envoyant les journaux d’audit et les journaux de connexion dans un espace de travail Log Analytics. Configurez les alertes souhaitées dans un espace de travail Log Analytics.

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="312-alert-on-account-sign-in-behavior-deviation"></a>3.12 : Alerter en cas d’écart de comportement de connexion à un compte

**Conseils** : Utilisez les détections de risque et la fonctionnalité Risk and Identity Protection d’Azure Active Directory (Azure AD) pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. Ingérez des données dans Azure Sentinel pour approfondir votre examen.

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour configurer et activer des stratégies de risque Identity Protection](../active-directory/identity-protection/howto-identity-protection-configure-risk-policies.md)

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils** : Utilisez des étiquettes pour faciliter le suivi des ressources d’Azure Web Application Firewall (WAF) et des ressources associées qui stockent ou traitent des informations sensibles.
- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils** : Implémentez une isolation en utilisant des abonnements et groupes d’administration distincts selon les domaines de sécurité, tels que le type d’environnement et le niveau de confidentialité des données pour, par exemple, les environnements de développement, de test et de production. 

Contrôlez l’accès aux ressources Azure à l’aide du contrôle d’accès en fonction du rôle Azure (Azure RBAC).

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Instructions** : Chiffrer toutes les informations sensibles en transit. Assurez-vous que les clients qui se connectent à vos instances Azure Web Application Firewall (WAF) et ressources associées peuvent négocier TLS 1.2 ou une version ultérieure.

Suivez les recommandations du Centre de sécurité relatives au chiffrement au repos et au chiffrement en transit le cas échéant.

- [Présentation du chiffrement en transit avec Azure](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Conseils** : Contrôlez l’accès aux ressources Azure à l’aide du contrôle d’accès en fonction du rôle Azure (Azure RBAC).
- [Comment configurer Azure RBAC](../role-based-access-control/role-assignments-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Conseils** : Utilisez le chiffrement au repos pour toutes les ressources Azure, notamment Azure Web Application Firewall (WAF) Azure et les ressources associées. Microsoft recommande d’autoriser Azure à gérer vos clés de chiffrement, mais il existe une option vous permettant de gérer vos propres clés dans certains cas.

- [Présentation du chiffrement au repos dans Azure](../security/fundamentals/encryption-atrest.md)

- [Guide pratique pour configurer des clés de chiffrement gérées par le client](../storage/common/customer-managed-keys-configure-key-vault.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Conseils** : Configurez Azure Web Application Firewall (WAF) pour qu’il s’exécute en mode de prévention après avoir référencé le trafic réseau en mode de détection pendant une période prédéterminée. 

En mode de prévention, Azure WAF bloque les intrusions et les attaques détectées par les règles. L’attaquant reçoit une exception « 403 Accès non autorisé » et la connexion est fermée. Le mode de prévention enregistre de telles attaques dans les journaux WAF.

- [Vue d’ensemble de l’intégration entre Application Gateway et Azure Security Center](../application-gateway/application-gateway-integration-security-center.md#overview)

- [Modes WAF sur Application Gateway](ag/ag-overview.md#waf-modes)

- [Modes WAF sur Front Door](afds/afds-overview.md#waf-modes)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-automated-asset-discovery-solution"></a>6.1 : Utiliser la solution de détection automatisée des ressources

**Conseils** : Utilisez Azure Resource Graph pour interroger ou découvrir toutes les ressources telles que calcul, stockage, réseau, ports, protocoles, etc., dans vos abonnements. 

Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements. Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Guide pratique pour afficher ses abonnements Azure](/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0)

- [Présentation d’Azure RBAC](../role-based-access-control/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils** : Utilisez des étiquettes sur des stratégies Azure Web Application Firewall (WAF). Les étiquettes peuvent être associées à des ressources et appliquées logiquement pour organiser l’accès à ces ressources dans un abonnement client. 

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Conseils** : Utilisez des étiquettes, des groupes d’administration et des abonnements distincts, le cas échéant, pour organiser et suivre Azure WAF et les ressources associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

- [Guide pratique pour créer des abonnements Azure supplémentaires](../cost-management-billing/manage/create-subscription.md)

- [Guide pratique pour créer des groupes d’administration](../governance/management-groups/create-management-group-portal.md)

- [Guide pratique pour créer et utiliser des étiquettes](../azure-resource-manager/management/tag-resources.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="64-define-and-maintain-inventory-of-approved-azure-resources"></a>6.4 : Définir et tenir un inventaire des ressources Azure approuvées

**Conseils** : Créez un inventaire des ressources approuvées, notamment de configuration, en fonction des besoins de votre organisation.

Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements. Utilisez Azure Resource Graph pour interroger et découvrir des ressources dans leurs abonnements. Vérifiez que toutes les ressources Azure présentes dans l’environnement sont approuvées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Aide** : Utilisez Azure Policy pour appliquer des restrictions quant au type de ressources pouvant être créées dans vos abonnements.
Utilisez Azure Resource Graph pour interroger ou découvrir des ressources Azure Web Application Firewall (WAF) dans leurs abonnements. Vérifiez que toutes les ressources Azure WAF et ressources associées présentes dans l’environnement sont approuvées.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour créer des requêtes avec Azure Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Conseils** : Surveillez et supprimez les ressources Azure WAF non approuvées avec Azure Policy pour refuser le déploiement d’Azure WAF, ou un certain type de WAF, par exemple, Azure WAF v1 et v2.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Aide** : Utilisez Azure Policy pour limiter les services que vous pouvez approvisionner dans votre environnement.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="611-limit-users-ability-to-interact-with-azure-resource-manager"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Conseils** : Utilisez l’accès conditionnel Azure pour limiter la capacité d’un utilisateur à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Guide pratique pour configurer l’accès conditionnel de façon à bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils** : Azure Web Application Firewall (WAF) peut être associé à différents environnements via des réseaux, des groupes de ressources ou des abonnements pour séparer les applications à haut risque.

- [Vue d’ensemble du réseau virtuel Azure](../virtual-network/virtual-networks-overview.md)

- [Organiser vos ressources avec des groupes d’administration Azure](../governance/management-groups/overview.md)

- [Guide de décision concernant les abonnements](/azure/cloud-adoption-framework/decision-guides/subscriptions/)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Conseils** : Définissez et implémentez des configurations de sécurité standard pour les paramètres réseau relatifs à vos déploiements d’Azure Web Application Firewall (WAF).
Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.Network » pour créer des stratégies personnalisées d’audit ou d’application de la configuration réseau de vos passerelles Azure Application Gateway, de vos réseaux virtuels et de vos groupes de sécurité réseau, et utilisez des définitions de stratégie intégrées.

- [Affichage des alias Azure Policy disponibles](/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0)

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Conseils** : Utilisez les effets Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à votre Azure Web Application Firewall (WAF) et ressources associées. 

Utilisez des modèles Azure Resource Manager pour appliquer la configuration de sécurité de votre Azure WAF et ressources associées requise par votre organisation.

- [Comprendre les effets d'Azure Policy](../governance/policy/concepts/effects.md)

- [Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Présentation des modèles Azure Resource Manager](../azure-resource-manager/templates/overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Conseils** : Utilisez Azure DevOps pour stocker et gérer de manière sécurisée votre code, tel que les stratégies Azure personnalisées et les modèles Azure Resource Manager. 

Accordez ou refusez des autorisations à des utilisateurs spécifiques, à des groupes de sécurité intégrés ou à des groupes définis dans Azure Active Directory (Azure AD) s’ils sont intégrés à Azure DevOps ou dans Active Directory s’ils sont intégrés à Team Foundation Server (TFS).

- [Stocker du code dans Azure DevOps](/azure/devops/repos/git/gitworkflow?view=azure-devops)

- [À propos des autorisations et des groupes dans Azure DevOps](/azure/devops/organizations/security/about-permissions)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="77-deploy-configuration-management-tools-for-azure-resources"></a>7.7 : Déployer des outils de gestion de la configuration pour les ressources Azure

**Aide** : Utilisez les définitions Azure Policy intégrées ainsi que les alias Azure Policy dans l’espace de noms « Microsoft.Network » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. Développez un processus et un pipeline pour la gestion des exceptions de stratégie.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Documentation Azure Policy](../governance/policy/index.yml)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="79-implement-automated-configuration-monitoring-for-azure-resources"></a>7.9 : Mettre en place une supervision automatisée de la configuration pour les ressources Azure

**Aide** : Utilisez les définitions Azure Policy intégrées ainsi que les alias Azure Policy dans l’espace de noms « Microsoft.Network » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. 

Utilisez les effets Azure Policy [auditer], [refuser] et [déployer s’il n’existe pas] afin d’appliquer automatiquement des configurations pour vos ressources Azure.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Documentation Azure Policy](../governance/policy/index.yml)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Conseils** : Utilisez Azure Key Vault pour stocker des certificats en toute sécurité. Azure Key Vault est un magasin de secrets managé par une plateforme, que vous pouvez utiliser pour protéger des secrets, des clés et des certificats SSL. 

Azure Application Gateway prend en charge l’intégration dans Key Vault des certificats de serveur associés à des écouteurs HTTPS. 

- [Procédure de configuration de l’arrêt SSL avec les certificats du coffre de clés à l’aide d’Azure PowerShell](../application-gateway/configure-keyvault-ps.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Implémentez le moteur d’analyse pour identifier des informations d’identification dans du code qui encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.
- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-recovery"></a>Récupération de données

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : Garantir la protection des sauvegardes et des clés managées par le client

**Conseils** : Assurez-vous que la suppression réversible est activée pour Azure Key Vault. La suppression réversible permet de récupérer des coffres de clés et des objets de coffre supprimés, tels que des clés, secrets et autres certificats.

- [Utilisation de la suppression réversible d’Azure Key Vault](../key-vault/general/key-vault-recovery.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Aide** : Développez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé. 

- [Aide sur la création de votre propre processus de réponse aux incidents de sécurité](https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/) 

- [Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft](https://msrc-blog.microsoft.com/2019/06/27/inside-the-msrc-anatomy-of-a-ssirp-incident/) 

- [Utiliser le guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de votre propre plan de réponse aux incidents](https://csrc.nist.gov/publications/detail/sp/800-61/rev-2/final)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Aide** : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à classer par ordre de priorité celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.
Marquez clairement les abonnements (par exemple, production, non-production) et créez un système de nommage pour identifier et classer les ressources Azure de façon claire.

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils** : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.
- [Consultez la publication du NIST, Guide to Test, Training, and Exercise Programs for IT Plans and Capabilities (Guide de test, d’entraînement et d’utilisation des programmes destinés aux plans et fonctionnalités informatiques)](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Aide** : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé. Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.
- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Aide** : Exportez vos alertes et recommandations de Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Utilisez le connecteur de données Azure Security Center pour diffuser en continu les alertes vers Azure Sentinel, en fonction des exigences de votre organisation.

- [Comment configurer l’exportation continue](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils** : Utilisez la fonctionnalité d’automatisation du workflow dans le Centre de sécurité pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.
- [Comment configurer l’automatisation des workflows et Logic Apps](../security-center/workflow-automation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d'informations, consultez [Benchmark de sécurité Azure : Tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings"></a>11.1 : Procéder régulièrement à des tests d’intrusion des ressources Azure et veiller à corriger tous les problèmes de sécurité critiques détectés

**Aide** : Suivez les règles d’engagement de Microsoft pour que vos tests d’intrusion soient conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft. 

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consulter le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)