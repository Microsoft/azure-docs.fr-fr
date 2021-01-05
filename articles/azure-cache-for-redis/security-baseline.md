---
title: Base de référence de sécurité Azure pour Azure Cache Redis
description: Base de référence de sécurité Azure pour Azure Cache Redis
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 03/16/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 24000ef4a4dba7a0d03c5e12fbe070f8871b55be
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92546498"
---
# <a name="azure-security-baseline-for-azure-cache-for-redis"></a>Base de référence de sécurité Azure pour Azure Cache Redis

La ligne de base de sécurité Azure pour Azure Cache Redis contient des recommandations qui vous aideront à améliorer la posture de sécurité de votre déploiement.

La base de référence pour ce service est tirée du [benchmark de sécurité Azure version 1.0](../security/benchmarks/overview.md), qui fournit des recommandations sur la façon de sécuriser vos solutions cloud sur Azure en suivant nos conseils sur les bonnes pratiques.

Pour plus d’informations, consultez [Vue d’ensemble des lignes de base de sécurité Azure](../security/benchmarks/security-baselines-overview.md).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Contrôle de sécurité : sécurité réseau](../security/benchmarks/security-control-network-security.md).*

### <a name="11-protect-resources-using-network-security-groups-or-azure-firewall-on-your-virtual-network"></a>1.1 : Protéger les ressources à l'aide de groupes de sécurité réseau ou du Pare-feu Azure sur votre réseau virtuel

**Aide**  : Déployez votre instance Azure Cache pour Redis au sein d’un réseau virtuel (VNet). Un réseau VNet est un réseau privé dans le cloud. Lorsqu’une instance de Cache Azure pour Redis est configurée avec un réseau virtuel, elle n’est pas adressable publiquement et est accessible uniquement à partir de machines virtuelles et d’applications sur le réseau virtuel.

Vous pouvez également spécifier des règles de pare-feu avec une des valeurs de début et de fin de plage d’adresses IP. Lorsque des règles de pare-feu sont configurées, seules les connexions client à partir des plages d’adresses IP spécifiées peuvent se connecter au cache.

Comment configurer la prise en charge de réseau virtuel pour un Cache Azure pour Redis Premium :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-premium-vnet

Configuration des règles de pare-feu Azure Cache pour Redis :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-configure#firewall

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="12-monitor-and-log-the-configuration-and-traffic-of-vnets-subnets-and-nics"></a>1.2 : Surveiller et consigner la configuration et le trafic des réseaux virtuels, des sous-réseaux et des cartes réseau

**Aide**  : Lorsque des machines virtuelles sont déployées dans le même réseau virtuel que votre instance Azure Cache pour Redis, vous pouvez utiliser des groupes de sécurité réseau (NSG) pour réduire le risque d’exfiltration de données. Activez les journaux de flux NSG et transférez-les vers un compte de stockage Azure pour l'audit du trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

Activer les journaux de flux NSG :

https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Activer et utiliser Traffic Analytics :

https://docs.microsoft.com/azure/network-watcher/traffic-analytics

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="13-protect-critical-web-applications"></a>1.3 : Protéger les applications web critiques

**Conseils**  : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="14-deny-communications-with-known-malicious-ip-addresses"></a>1.4 : Refuser les communications présentant des adresses IP connues comme étant malveillantes

**Conseils**  : Le déploiement du Réseau virtuel Azure (VNet) fournit une sécurité et une isolation améliorées pour le cache Azure pour Redis, ainsi que des sous-réseaux, des stratégies de contrôle d’accès et d’autres fonctionnalités permettant de restreindre davantage l’accès. Lors du déploiement dans un réseau virtuel, Azure Cache pour Redis n’est pas adressable publiquement et est accessible uniquement à partir de machines virtuelles et d’applications au sein du réseau virtuel.

Activez le service Protection DDoS Standard sur les réseaux virtuels associés à vos instances Azure Cache pour Redis à des fins de protection contre les attaques par déni de service distribué (DDoS). Utilisez la fonctionnalité de renseignement sur les menaces intégrée à Azure Security Center pour refuser les communications avec des adresses IP Internet connues comme étant malveillantes ou inutilisées.

Comment configurer la prise en charge de réseau virtuel pour un Cache Azure pour Redis Premium :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-premium-vnet

Gérer le Service Protection DDoS Standard Azure à l’aide du portail Azure :

https://docs.microsoft.com/azure/virtual-network/manage-ddos-protection

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="15-record-network-packets-and-flow-logs"></a>1.5 : Consigner les paquets réseau et les journaux de flux

**Conseils**  : Lorsque des machines virtuelles sont déployées dans le même réseau virtuel que votre instance Azure Cache pour Redis, vous pouvez utiliser des groupes de sécurité réseau (NSG) pour réduire le risque d’exfiltration de données. Activez les journaux de flux NSG et transférez-les vers un compte de stockage Azure pour l'audit du trafic. Vous pouvez aussi envoyer ces journaux vers un espace de travail Log Analytics et utiliser Traffic Analytics pour fournir des insights sur le flux de trafic dans votre cloud Azure. Parmi les avantages de Traffic Analytics figure la possibilité de visualiser l’activité réseau et d’identifier les zones réactives, d’identifier les menaces de sécurité, de comprendre les modèles de flux de trafic et de repérer les mauvaises configurations du réseau.

Activer les journaux de flux NSG :

https://docs.microsoft.com/azure/network-watcher/network-watcher-nsg-flow-logging-portal

Activer et utiliser Traffic Analytics :

https://docs.microsoft.com/azure/network-watcher/traffic-analytics

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="16-deploy-network-based-intrusion-detectionintrusion-prevention-systems-idsips"></a>1.6 : Déployer des systèmes de détection et de prévention des intrusions basés sur le réseau (IDS/IPS)

**Aide**  : Lorsque vous utilisez Azure Cache pour Redis avec vos applications Web s’exécutant sur des instances Azure App Service ou de calcul, déployez toutes les ressources au sein d’un réseau virtuel Azure (VNet) et sécurisez-les avec un pare-feu d’applications web (WAF) Azure sur Web Application Gateway. Configurez WAF pour une exécution en « mode de prévention ». Le mode de prévention bloque les intrusions et les attaques détectées par les règles. L’attaquant reçoit une exception « 403 Accès non autorisé » et la connexion est fermée. Le mode de prévention enregistre de telles attaques dans les journaux WAF.

Vous pouvez également sélectionner une offre de la place de marché Azure qui prend en charge les fonctionnalités IDS/IPS avec des fonctionnalités d’inspection de charge utile et/ou de détection d'anomalie.

Comprendre les fonctionnalités WAF d’Azure :

https://docs.microsoft.com/azure/web-application-firewall/ag/ag-overview

Déployer Azure WAF :

https://docs.microsoft.com/azure/web-application-firewall/ag/create-waf-policy-ag

Place de marché Azure :

https://azuremarketplace.microsoft.com/marketplace/?term=Firewall

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="17-manage-traffic-to-web-applications"></a>1.7 : Gérer le trafic à destination des applications web

**Conseils**  : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="18-minimize-complexity-and-administrative-overhead-of-network-security-rules"></a>1.8 : Réduire la complexité et les frais administratifs liés aux règles de sécurité réseau

**Aide**  : Utilisez des balises de service de réseau virtuel pour définir des contrôles d’accès réseau sur les groupes de sécurité réseau (NSG) ou le pare-feu Azure. Vous pouvez utiliser des balises de service à la place des adresses IP spécifiques lors de la création de règles de sécurité. En spécifiant le nom de la balise de service (par exemple, ApiManagement) dans le champ Source ou Destination approprié d'une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

Vous pouvez également utiliser des groupes de sécurité d’application (ASG) pour simplifier la configuration de la sécurité complexe. Les groupes de sécurité d’application permettent de configurer la sécurité réseau comme un prolongement naturel de la structure de l’application, et donc de regrouper les machines virtuelles et définir des stratégies de sécurité réseau basés sur ces groupes.

Balises de service du réseau virtuel :

https://docs.microsoft.com/azure/virtual-network/service-tags-overview

Groupes de sécurité d’application :

https://docs.microsoft.com/azure/virtual-network/security-overview#application-security-groups

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="19-maintain-standard-security-configurations-for-network-devices"></a>1.9 : Gérer les configurations de sécurité standard pour les périphériques réseau

**Aide**  : Définissez et implémentez des configurations de sécurité standard pour les ressources réseau associées à vos instances Azure Cache pour Redis avec Azure Policy. Utilisez des alias Azure Policy dans les espaces de noms « Microsoft.Cache » et « Microsoft. Network » afin de créer des stratégies personnalisées pour auditer ou appliquer la configuration réseau de vos instances Azure Cache pour Redis. Vous pouvez également utiliser des définitions de stratégie intégrée, par exemple :

Seules les connexions sécurisées à votre cache Redis doivent être activées

DDoS Protection Standard doit être activé

Vous pouvez aussi utiliser Azure Blueprints pour simplifier les déploiements Azure à grande échelle en regroupant les artefacts d’environnement clés, comme les modèles Azure Resource Manager (ARM), le contrôle d’accès en fonction du rôle Azure (Azure RBAC) et les stratégies dans une même définition de blueprint. Appliquez facilement le blueprint aux nouveaux abonnements et environnements, et ajustez le contrôle et la gestion par le biais du versioning.

Configurer et gérer Azure Policy :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Créer une instance d'Azure Blueprint :

https://docs.microsoft.com/azure/governance/blueprints/create-blueprint-portal

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="110-document-traffic-configuration-rules"></a>1.10 : Règles de configuration du trafic de documents

**Aide**  : Utilisez des balises pour les ressources réseau associées à votre déploiement Azure Cache pour Redis afin de les organiser logiquement dans une taxonomie.

Créer et utiliser des étiquettes :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="111-use-automated-tools-to-monitor-network-resource-configurations-and-detect-changes"></a>1.11 : Utiliser des outils automatisés pour superviser les configurations des ressources réseau et détecter les modifications

**Conseils**  : Utilisez le journal d’activité Azure pour surveiller les configurations des ressources réseau et détecter les modifications de celles-ci associées à vos instances Azure Cache pour Redis. Créez des alertes dans Azure Monitor, qui se déclenchent lors de la modification de ressources réseau critiques.

Comment consulter et récupérer les événements du journal d’activité Azure :

https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-view

Comment créer des alertes dans Azure Monitor :

https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

## <a name="logging-and-monitoring"></a>Journalisation et supervision

*Pour plus d’informations, consultez [Contrôle de sécurité : journalisation et supervision](../security/benchmarks/security-control-logging-monitoring.md).*

### <a name="21-use-approved-time-synchronization-sources"></a>2.1 : Utiliser des sources de synchronisation date/heure approuvées

**Aide**  : Microsoft conserve la source de temps utilisée pour les ressources Azure, par exemple Azure Cache pour Redis, pour les horodatages dans les journaux.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Microsoft

### <a name="22-configure-central-security-log-management"></a>2.2 : Configurer la gestion des journaux de sécurité centrale

**Aide**  : activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux vers un espace de travail Log Analytics, Event Hub ou un compte de stockage Azure pour archivage. Les journaux d’activité fournissent des insights sur les opérations qui ont été effectuées sur vos instances Azure Cache pour Redis au niveau du plan de contrôle. À l’aide des données des journaux d’activité Azure, vous pouvez déterminer les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle pour vos instances Azure Cache pour Redis.

Comment activer les paramètres de diagnostic pour le journal d'activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="23-enable-audit-logging-for-azure-resources"></a>2.3 : Activer la journalisation d’audit pour les ressources Azure

**Aide**  : activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux vers un espace de travail Log Analytics, Event Hub ou un compte de stockage Azure pour archivage. Les journaux d’activité fournissent des insights sur les opérations qui ont été effectuées sur vos instances Azure Cache pour Redis au niveau du plan de contrôle. À l’aide des données des journaux d’activité Azure, vous pouvez déterminer les éléments « qui, quand et quoi » pour toutes les opérations d’écriture (PUT, POST, DELETE) effectuées au niveau du plan de contrôle pour vos instances Azure Cache pour Redis.

Alors que les indicateurs de performance sont disponibles en activant les paramètres de diagnostic, la journalisation d’audit au niveau du plan de données n’est pas encore disponible pour Azure Cache Redis.

Comment activer les paramètres de diagnostic pour le journal d'activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="24-collect-security-logs-from-operating-systems"></a>2.4 : Collecter les journaux de sécurité des systèmes d’exploitation

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="25-configure-security-log-storage-retention"></a>2.5 : Configurer la conservation du stockage des journaux de sécurité

**Conseils**  : Dans Azure Monitor, définissez la période de rétention des journaux pour les espaces de travail Log Analytics associés à vos instances Azure Cache pour Redis conformément aux réglementations de conformité de votre organisation.

Notez que la journalisation d’audit au niveau du plan de données n’est pas encore disponible pour Azure Cache pour Redis.

Définir des paramètres de rétention des journaux :

https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#change-the-data-retention-period

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="26-monitor-and-review-logs"></a>2.6 : Superviser et examiner les journaux

**Aide**  : Activez les paramètres de diagnostic des journaux d’activité Azure et envoyez les journaux à un espace de travail Log Analytics. Exécutez des requêtes dans Log Analytics pour rechercher des termes, identifier des tendances, analyser des modèles et fournir de nombreuses autres informations basées sur les données du journal d’activité qui ont pu être collectées pour Azure Cache Redis.

Notez que la journalisation d’audit au niveau du plan de données n’est pas encore disponible pour Azure Cache pour Redis.

Comment activer les paramètres de diagnostic pour le journal d'activité Azure : https://docs.microsoft.com/azure/azure-monitor/platform/diagnostic-settings-legacy

Comment collecter et analyser les journaux d’activité Azure dans l’espace de travail Log Analytics dans Azure Monitor : https://docs.microsoft.com/azure/azure-monitor/platform/activity-log-collect

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="27-enable-alerts-for-anomalous-activity"></a>2.7 : Activer des alertes en cas d’activité anormale

**Aide**  : Vous pouvez effectuer la configuration de manière à recevoir des alertes basées sur les indicateurs de performance et les journaux d’activité liés à vos instances Azure Cache pour Redis. Azure Monitor vous permet de configurer une alerte pour envoyer une notification par courrier électronique, appeler un webhook ou appeler une application logique Azure.

Alors que les indicateurs de performance sont disponibles en activant les paramètres de diagnostic, la journalisation d’audit au niveau du plan de données n’est pas encore disponible pour Azure Cache Redis.

Comment configurer des alertes pour Azure Cache Redis : https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-monitor#alerts

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="28-centralize-anti-malware-logging"></a>2.8 : Centraliser la journalisation anti-programme malveillant

**Aide**  : Non applicable. Azure Cache pour Redis ne traite pas et ne produit pas de journaux liés aux logiciels anti-programmes malveillants.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="29-enable-dns-query-logging"></a>2.9 : Activer la journalisation des requêtes DNS

**Aide**  : Non applicable. Azure Cache pour Redis ne traite pas et ne produit pas de journaux liés au DNS.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="210-enable-command-line-audit-logging"></a>2.10 : Activer l’enregistrement d’audit en ligne de commande

**Aide**  : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

## <a name="identity-and-access-control"></a>Contrôle des accès et des identités

*Pour plus d’informations, consultez [Contrôle de sécurité : contrôle des accès et des identités](../security/benchmarks/security-control-identity-access-control.md).*

### <a name="31-maintain-an-inventory-of-administrative-accounts"></a>3.1 : Tenir un inventaire des comptes d’administration

**Aide**  : Azure Active Directory (AD) comporte des rôles intégrés qui doivent être explicitement attribués et qui peuvent être interrogés. Utilisez le module Azure AD PowerShell pour effectuer des requêtes ad hoc afin de découvrir les comptes membres de groupes d’administration.

Comment obtenir un rôle d’annuaire dans Azure AD avec PowerShell : https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrole?view=azureadps-2.0

Comment obtenir les membres d’un rôle d’annuaire dans Azure AD avec PowerShell : https://docs.microsoft.com/powershell/module/azuread/get-azureaddirectoryrolemember?view=azureadps-2.0

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="32-change-default-passwords-where-applicable"></a>3.2 : Modifier les mots de passe par défaut lorsque cela est possible

**Conseils**  : L’accès au plan de contrôle pour Azure Cache Redis est contrôlé par le biais d’Azure Active Directory (AD). Azure AD n’intègre pas le concept des mots de passe par défaut. 

L’accès du plan de données à Azure Cache pour Redis est contrôlé par le biais de clés d’accès. Ces clés sont utilisées par les clients qui se connectent à votre cache et peuvent être regénérées à tout moment.

Il n’est pas recommandé de créer des mots de passe par défaut dans votre application. Au lieu de cela, vous pouvez stocker vos mots de passe dans Azure Key Vault, puis utiliser Azure Active Directory pour les récupérer.

Comment regénérer les clés d’accès pour Azure Cache Redis : https://docs.microsoft.com/azure/azure-cache-for-redis/cache-configure#settings

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Partagé

### <a name="33-use-dedicated-administrative-accounts"></a>3.3 : Utiliser des comptes d’administration dédiés

**Conseils**  : Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Utilisez la gestion des identités et des accès dans Azure Security Center pour superviser le nombre de comptes d’administration.

En outre, pour vous aider à suivre les comptes d’administration dédiés, vous pouvez utiliser des recommandations d’Azure Security Center ou des stratégies Azure intégrées, telles que les suivantes :

- Plusieurs propriétaires doivent être affectés à votre abonnement

- Les comptes dépréciés disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

- Les comptes externes disposant d’autorisations de propriétaire doivent être supprimés de votre abonnement

Comment utiliser Azure Security Center pour superviser l’identité et l’accès (préversion) : https://docs.microsoft.com/azure/security-center/security-center-identity-access

Comment utiliser Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage


**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="34-use-single-sign-on-sso-with-azure-active-directory"></a>3.4 : Utiliser l’authentification unique (SSO) avec Azure Active Directory

**Aide**  : Azure Cache pour Redis utilise des clés d’accès pour authentifier les utilisateurs et ne prend pas en charge l’authentification unique (SSO) au niveau du plan de données. L’accès au plan de contrôle pour Azure Cache Redis est disponible via l’API REST et prend en charge l’authentification unique. Pour vous authentifier, définissez l’en-tête d’autorisation pour vos demandes sur un jeton web JSON que vous avez obtenu auprès d’Azure Active Directory.

Comprendre l’API REST Azure Cache pour Redis : https://docs.microsoft.com/rest/api/redis/

Comprendre l’authentification unique (SSO) avec Azure AD : https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on


**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="35-use-multi-factor-authentication-for-all-azure-active-directory-based-access"></a>3.5 : Utiliser l’authentification multifacteur pour tous les accès basés sur Azure Active Directory

**Aide**  : Activez l’authentification multifacteur (MFA) Azure Active Directory (AAD) et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center.

Comment activer l’authentification multifacteur dans Azure : https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

Comment surveiller l’identité et l’accès dans Azure Security Center : https://docs.microsoft.com/azure/security-center/security-center-identity-access

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="36-use-dedicated-machines-privileged-access-workstations-for-all-administrative-tasks"></a>3.6 : Utiliser des ordinateurs dédiés (stations de travail avec accès privilégié) pour toutes les tâches administratives

**Conseils**  : Utilisez des stations de travail disposant d’un accès privilégié avec Multi-Factor Authentication (MFA) configuré pour vous connecter aux ressources Azure et les configurer.

En savoir plus sur les stations de travail avec accès privilégié :

https://docs.microsoft.com/windows-server/identity/securing-privileged-access/privileged-access-workstations

Comment activer l’authentification multifacteur dans Azure :

https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-getstarted

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="37-log-and-alert-on-suspicious-activity-from-administrative-accounts"></a>3.7 : Journaliser et générer des alertes en cas d’activités suspectes depuis les comptes d’administration

**Aide**  : Utilisez Azure Active Directory (AD) Privileged Identity Management pour générer des journaux et des alertes quand des activités suspectes ou potentiellement dangereuses se produisent dans l’environnement.

Utilisez également les détections de risque Azure AD pour visualiser les alertes et des rapports sur les comportements à risque des utilisateurs.

Comment déployer Privileged Identity Management (PIM) : https://docs.microsoft.com/azure/active-directory/privileged-identity-management/pim-deployment-plan

Comprendre les détections de risques Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risk-events

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="38-manage-azure-resources-from-only-approved-locations"></a>3.8 : Gérer les ressources Azure à partir des emplacements approuvés uniquement

**Aide**  : Configurez des emplacements nommés dans l’accès conditionnel Azure Active Directory (AD) pour autoriser l’accès uniquement à partir de regroupements logiques spécifiques de plages d’adresses IP ou de pays/régions.

Comment configurer des emplacements nommés dans Azure : https://docs.microsoft.com/azure/active-directory/reports-monitoring/quickstart-configure-named-locations

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="39-use-azure-active-directory"></a>3.9 : Utiliser Azure Active Directory

**Conseils**  : Utiliser Azure Active Directory (AD) comme système d’authentification et d’autorisation central. Azure AD protège les données en utilisant un chiffrement fort pour les données au repos et en transit. De plus, AAD sale, hache et stocke de manière sécurisée les informations d’identification utilisateur.

L’authentification Azure AD ne peut pas être utilisée pour un accès direct au plan de données Azure Cache pour Redis. Les informations d’identification Azure AD peuvent cependant être utilisées pour l’administration au niveau du plan de contrôle (c’est-à-dire le portail Azure) pour contrôler les clés d’accès Azure Cache pour Redis.


**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="310-regularly-review-and-reconcile-user-access"></a>3.10 : Examiner et rapprocher régulièrement l’accès utilisateur

**Aide**  : Azure Active Directory (AD) fournit des journaux pour vous aider à découvrir les comptes obsolètes. De plus, utilisez les révisions d’accès des identités Azure pour gérer efficacement les appartenances aux groupes, les accès aux applications d’entreprise et les attributions de rôles. L’accès des utilisateurs peut être passé en revue régulièrement pour vérifier que seules les utilisateurs appropriés continuent de bénéficier d’un accès. 

Comprendre la génération de rapports Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/

Comment utiliser les révisions d’accès des identités Azure : https://docs.microsoft.com/azure/active-directory/governance/access-reviews-overview

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="311-monitor-attempts-to-access-deactivated-accounts"></a>3.11 : Superviser les tentatives d’accès aux comptes désactivés

**Aide**  : Vous avez accès aux activités de connexion Azure Active Directory (AD), aux sources des journaux d’événements à risque et d’audit, ce qui vous permet de les intégrer à Azure Sentinel ou à un outil SIEM tiers.

Vous pouvez simplifier ce processus en créant des paramètres de diagnostic pour les comptes d’utilisateur Azure AD et en envoyant les journaux d’audit et les journaux de connexion à un espace de travail Log Analytics. Vous pouvez configurer les alertes de journal souhaitées dans un espace de travail Log Analytics.

Comment intégrer des journaux d’activité Azure dans Azure Monitor : https://docs.microsoft.com/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="312-alert-on-account-login-behavior-deviation"></a>3.12 : Alerte en cas d’écart de comportement de connexion à un compte

**Aide**  : Pour un écart de comportement de connexion de compte, utilisez Azure Active Directory (AAD) Identity Protection et les fonctionnalités de détections de risques pour configurer des réponses automatiques aux actions suspectes détectées liées aux identités d’utilisateur. Vous pouvez également ingérer des données dans Azure Sentinel pour approfondir votre examen.

Comment afficher les connexions risquées à Azure AD : https://docs.microsoft.com/azure/active-directory/reports-monitoring/concept-risky-sign-ins

Comment configurer et activer des stratégies de protection des identités : https://docs.microsoft.com/azure/active-directory/identity-protection/howto-identity-protection-configure-risk-policies

Comment intégrer Azure Sentinel : https://docs.microsoft.com/azure/sentinel/quickstart-onboard

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="313-provide-microsoft-with-access-to-relevant-customer-data-during-support-scenarios"></a>3.13 : Fournir à Microsoft un accès aux données client pertinentes pendant les scénarios de support

**Aide**  : Pas encore disponible. Customer Lockbox n’est actuellement pas pris en charge pour Azure Cache Redis.

Liste des services pris en charge pour Customer Lockbox :

https://docs.microsoft.com/azure/security/fundamentals/customer-lockbox-overview#supported-services-and-scenarios-in-general-availability

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Non applicable

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Contrôle de sécurité : protection des données](../security/benchmarks/security-control-data-protection.md).*

### <a name="41-maintain-an-inventory-of-sensitive-information"></a>4.1 : Conserver un inventaire des informations sensibles

**Conseils**  : Utilisez des étiquettes pour faciliter le suivi des ressources Azure qui stockent ou traitent des informations sensibles.

Créer et utiliser des étiquettes :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="42-isolate-systems-storing-or-processing-sensitive-information"></a>4.2 : Isoler les systèmes qui stockent ou traitent les informations sensibles

**Conseils**  : Implémentez des abonnements et/ou des groupes d’administration distincts pour le développement, les tests et la production. Les instances Azure Cache pour Redis doivent être séparées par un réseau virtuel/sous-réseau et balisées de manière appropriée. Si vous le souhaitez, utilisez Azure Cache Redis pour définir des règles, de manière à ce que seules les connexions client à partir des plages d’adresses IP spécifiées puissent se connecter au cache.

Créer des abonnements Azure supplémentaires :

https://docs.microsoft.com/azure/billing/billing-create-subscription

Créer des groupes d’administration :

https://docs.microsoft.com/azure/governance/management-groups/create

Comment déployer Azure Cache pour Redis dans un réseau virtuel :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-premium-vnet

Configuration des règles de pare-feu Azure Cache pour Redis :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-configure#firewall

Créer et utiliser des étiquettes :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="43-monitor-and-block-unauthorized-transfer-of-sensitive-information"></a>4.3. : Surveiller et bloquer le transfert non autorisé d’informations sensibles

**Conseils**  : Pas encore disponible. Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour Azure Cache Redis.

Microsoft gère l’infrastructure sous-jacente d’Azure Cache pour Redis et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Partagé

### <a name="44-encrypt-all-sensitive-information-in-transit"></a>4.4 : Chiffrer toutes les informations sensibles en transit

**Aide**  : Azure Cache pour Redis nécessite des communications chiffrées avec le protocole TLS par défaut. Les versions 1.0, 1.1 et 1.2 de TLS sont actuellement prises en charge. Toutefois, TLS 1.0 et 1.1 étant en passe de dépréciation dans l’ensemble du secteur, utilisez TLS 1.2 dans la mesure du possible. Si votre outil ou bibliothèque de client ne prend pas en charge TLS, vous pouvez activer des connexions non chiffrées par le biais du portail Azure ou des API de gestion. Dans les cas où il est impossible d’établir des connexions chiffrées, nous vous recommandons de placer votre cache et votre application cliente dans un réseau virtuel.

Comprendre le chiffrement en transit pour Azure Cache Redis :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-best-practices

Comprendre les ports requis utilisés dans les scénarios de cache de réseau virtuel :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-premium-vnet#outbound-port-requirements

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Partagé

### <a name="45-use-an-active-discovery-tool-to-identify-sensitive-data"></a>4.5 : Utiliser un outil de découverte actif pour identifier les données sensibles

**Aide**  : Les fonctionnalités d’identification des données, de classification des données et de protection contre la perte de données ne sont pas encore disponibles pour Azure Cache Redis. Baliser des instances contenant des informations sensibles en tant que telles, et implémenter une solution tierce à des fins de conformité si nécessaire.

Pour la plateforme sous-jacente qui est gérée par Microsoft, Microsoft traite tout le contenu client comme sensible et déploie d'importants efforts pour vous protéger contre la perte et l’exposition des données client. Pour garantir la sécurité des données client dans Azure, Microsoft a implémenté et tient à jour une suite de contrôles et de fonctionnalités de protection des données robustes.

Comprendre la protection des données client dans Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="46-use-azure-rbac-to-control-access-to-resources"></a>4.6 : Utiliser Azure RBAC pour contrôler l’accès aux ressources

**Aide**  : Utilisez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour contrôler l’accès au plan de contrôle Azure Cache pour Redis (c.-à-d., le portail Azure). 

Comment configurer Azure RBAC :

https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="47-use-host-based-data-loss-prevention-to-enforce-access-control"></a>4.7 : Utiliser la protection contre la perte de données basée sur l’hôte pour appliquer le contrôle d’accès

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

Microsoft gère l’infrastructure sous-jacente d’Azure Cache pour Redis et a implémenté des contrôles stricts pour empêcher la perte ou l’exposition de données client.

Comprendre la protection des données client dans Azure :

https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="48-encrypt-sensitive-information-at-rest"></a>4.8 : Chiffrer des informations sensibles au repos

**Aide**  : Azure Cache pour Redis stocke les données client en mémoire et, malgré une protection efficace grâce à de nombreux contrôles implémentés par Microsoft, la mémoire n’est pas chiffrée par défaut. Si cela est requis par votre organisation, chiffrez le contenu avant de le stocker dans Azure Cache pour Redis.

Si vous utilisez la fonctionnalité Azure Cache pour Redis « Persistance des données Redis », les données sont envoyées à un compte de stockage Azure que vous possédez et gérez. Vous pouvez configurer la persistance à partir du panneau « Nouvelle instance Azure Cache pour Redis » lors de la création du cache et dans le menu Ressources pour les caches Premium existants.

Les données dans Stockage Azure sont chiffrées et déchiffrées en toute transparence à l’aide du chiffrement AES 256 bits, un des chiffrements par blocs les plus puissants actuellement disponibles, et sont conformes à la norme FIPS 140-2. Le chiffrement de Stockage Azure ne peut pas être désactivé. Vous pouvez vous reposer sur les clés managées par Microsoft pour le chiffrement de votre compte de stockage, ou vous pouvez gérer le chiffrement avec vos propres clés.

Comment configurer la persistance des données dans Azure Cache pour Redis : https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-premium-persistence

Comprendre le chiffrement pour les comptes de stockage Azure : https://docs.microsoft.com/azure/storage/common/storage-service-encryption

Comprendre la protection des données client Azure : https://docs.microsoft.com/azure/security/fundamentals/protection-customer-data

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Partagé

### <a name="49-log-and-alert-on-changes-to-critical-azure-resources"></a>4.9 : Consigner et alerter les modifications apportées aux ressources Azure critiques

**Aide**  : Utilisez Azure Monitor avec le journal d’activité Azure pour créer des alertes en cas de modifications d’instances de production Azure Cache pour Redis et d’autres ressources critiques ou associées.

Comment créer des alertes pour les événements du journal des activités Azure :

https://docs.microsoft.com/azure/azure-monitor/platform/alerts-activity-log

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

## <a name="vulnerability-management"></a>Gestion des vulnérabilités

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des vulnérabilités](../security/benchmarks/security-control-vulnerability-management.md).*

### <a name="51-run-automated-vulnerability-scanning-tools"></a>5.1 : Exécuter les outils d’analyse des vulnérabilités automatisés

**Aide**  : Suivez les recommandations d’Azure Security Center relatives à la sécurisation de vos instances Azure Cache pour Redis et des ressources associées.

Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Azure Cache pour Redis.

Comprendre les recommandations d’Azure Security Center : https://docs.microsoft.com/azure/security-center/recommendations-reference

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Partagé

### <a name="52-deploy-automated-operating-system-patch-management-solution"></a>5.2 : Déployer une solution de gestion des correctifs de système d’exploitation automatisée

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="53-deploy-automated-third-party-software-patch-management-solution"></a>5.3 : Déployer une solution de gestion de correctif logiciel tiers automatisée

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="54-compare-back-to-back-vulnerability-scans"></a>5.4 : Comparer les analyses de vulnérabilités dos à dos

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="55-use-a-risk-rating-process-to-prioritize-the-remediation-of-discovered-vulnerabilities"></a>5.5 : Utilisez un processus de classement des risques pour classer par ordre de priorité la correction des vulnérabilités découvertes.

**Conseils**  : Microsoft assure la gestion des vulnérabilités sur les systèmes sous-jacents prenant en charge Azure Cache pour Redis.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Microsoft

## <a name="inventory-and-asset-management"></a>Gestion des stocks et des ressources

*Pour plus d’informations, consultez [Contrôle de sécurité : Gestion des stocks et des ressources](../security/benchmarks/security-control-inventory-asset-management.md).*

### <a name="61-use-azure-asset-discovery"></a>6.1 : Utiliser la découverte de ressources Azure

**Aide**  : Utilisez Azure Resource Graph pour interroger/découvrir toutes les ressources (telles que calcul, stockage, réseau, ports et protocoles) dans vos abonnements.  Vérifiez les autorisations (lecture) appropriées dans votre locataire et répertoriez tous les abonnements Azure, ainsi que les ressources dans vos abonnements.

Bien que les ressources Azure classiques puissent être découvertes via Resource Graph, il est vivement recommandé de créer et d’utiliser des ressources Azure Resource Manager à l’avenir.

Créer des requêtes avec Azure Resource Graph : https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

Afficher vos abonnements Azure : https://docs.microsoft.com/powershell/module/az.accounts/get-azsubscription?view=azps-3.0.0

Comprendre Azure RBAC : https://docs.microsoft.com/azure/role-based-access-control/overview

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="62-maintain-asset-metadata"></a>6.2 : Gérer les métadonnées de ressources

**Conseils**  : Appliquez des balises aux ressources Azure en fournissant des métadonnées pour les organiser de façon logique par catégories.

Comment créer et utiliser des étiquettes :

https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="63-delete-unauthorized-azure-resources"></a>6.3 : Supprimer des ressources Azure non autorisées

**Aide**  : Utilisez des étiquettes, des groupes d’administration, voire des abonnements séparés, pour organiser et suivre les instances Azure Cache pour Redis et les ressources associées. Rapprochez régulièrement l’inventaire et assurez-vous que les ressources non autorisées sont supprimées de l’abonnement en temps utile.

Appliquez également des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions de stratégie intégrées suivantes :

- Types de ressources non autorisés

- Types de ressources autorisés

Créer des abonnements Azure supplémentaires : https://docs.microsoft.com/azure/billing/billing-create-subscription

Créer des groupes d’administration : https://docs.microsoft.com/azure/governance/management-groups/create

Créer et utiliser des balises : https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="64-maintain-an-inventory-of-approved-azure-resources-and-software-titles"></a>6.4 : Tenir un inventaire des ressources Azure approuvées et titres des logiciels

**Aide**  : Non applicable. Cette recommandation concerne les ressources de calcul et Azure dans son ensemble.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="65-monitor-for-unapproved-azure-resources"></a>6.5 : Analyser les ressources Azure non approuvées

**Conseils**  : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

Types de ressources non autorisés

Types de ressources autorisés

Utilisez également Azure Resource Graph pour interroger/découvrir des ressources dans les abonnements.

Configurer et gérer Azure Policy :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Créer des requêtes avec Azure Graph :

https://docs.microsoft.com/azure/governance/resource-graph/first-query-portal

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="66-monitor-for-unapproved-software-applications-within-compute-resources"></a>6.6 : Analyser les applications logicielles non approuvées dans des ressources de calcul

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="67-remove-unapproved-azure-resources-and-software-applications"></a>6.7 : Supprimer des ressources et applications logicielles Azure non approuvées

**Aide**  : Non applicable. Cette recommandation concerne les ressources de calcul et Azure dans son ensemble.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="68-use-only-approved-applications"></a>6.8 : Utiliser des applications approuvées uniquement

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="69-use-only-approved-azure-services"></a>6.9 : Utiliser des services Azure approuvés uniquement

**Aide**  : Appliquez des restrictions quant au type de ressources pouvant être créées dans les abonnements clients, en utilisant Azure Policy avec les définitions intégrées suivantes :

Types de ressources non autorisés

Types de ressources autorisés

Configurer et gérer Azure Policy :

https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Refuser un type de ressource spécifique avec Azure Policy :

https://docs.microsoft.com/azure/governance/policy/samples/not-allowed-resource-types

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="610-implement-approved-application-list"></a>6.10 : Implémenter une liste d’applications approuvées

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="611-limit-users-ability-to-interact-with-azure-resources-manager-via-scripts"></a>6.11 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager par le biais de scripts

**Conseils**  : Configurez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager (ARM) en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

Configurer l’accès conditionnel pour bloquer l’accès à ARM :

https://docs.microsoft.com/azure/role-based-access-control/conditional-access-azure-management

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="612-limit-users-ability-to-execute-scripts-within-compute-resources"></a>6.12 : Limiter la capacité des utilisateurs à exécuter des scripts dans des ressources de calcul

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="613-physically-or-logically-segregate-high-risk-applications"></a>6.13 : Séparer physiquement ou logiquement des applications à risque élevé

**Conseils**  : Non applicable. Cette recommandation a trait aux applications web s’exécutant sur Azure App Service ou des ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

## <a name="secure-configuration"></a>Configuration sécurisée

*Pour plus d’informations, consultez [Contrôle de sécurité : Configuration sécurisée](../security/benchmarks/security-control-secure-configuration.md).*

### <a name="71-establish-secure-configurations-for-all-azure-resources"></a>7.1 : Établir des configurations sécurisées pour toutes les ressources Azure

**Aide**  : Définir et implémenter des configurations de sécurité standard pour vos instances Azure Cache Redis avec Azure Policy. Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.Cache » pour créer des stratégies personnalisées d’audit ou d’application de la configuration de vos instances Azure Cache pour Redis. Vous pouvez aussi utiliser des définitions de stratégie intégrées relatives à vos instances Azure Cache pour Redis comme :

Seules les connexions sécurisées à votre cache Redis doivent être activées

Affichage des alias Azure Policy disponibles : https://docs.microsoft.com/powershell/module/az.resources/get-azpolicyalias?view=azps-3.3.0

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="72-establish-secure-operating-system-configurations"></a>7.2 : Établir des configurations sécurisées du système d’exploitation

**Aide**  : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="73-maintain-secure-azure-resource-configurations"></a>7.3 : Gérer les configurations de ressources Azure sécurisées

**Aide**  : Utilisez les stratégies Azure Policy [refuser] et [déployer s’il n’existe pas] pour appliquer des paramètres sécurisés à vos ressources Azure.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

Comprendre les effets d’Azure Policy : https://docs.microsoft.com/azure/governance/policy/concepts/effects

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="74-maintain-secure-operating-system-configurations"></a>7.4 : Préserver la sécurité des configurations du système d'exploitation

**Aide**  : Non applicable. Ces conseils concernent les ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="75-securely-store-configuration-of-azure-resources"></a>7.5 : Stocker en toute sécurité la configuration des ressources Azure

**Aide**  : Si vous utilisez des définitions d’Azure Policy personnalisées pour vos instances Azure Resource Manager pour Azure Cache Redis et des ressources associées, utilisez des Azure Repos pour stocker et gérer votre code de façon sécurisée.

Stocker du code dans Azure DevOps : https://docs.microsoft.com/azure/devops/repos/git/gitworkflow?view=azure-devops

Documentation Azure Repos : https://docs.microsoft.com/azure/devops/repos/index?view=azure-devops

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="76-securely-store-custom-operating-system-images"></a>7.6 Stocker en toute sécurité des images de système d’exploitation personnalisées

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="77-deploy-system-configuration-management-tools"></a>7.7 : Déployer les outils de gestion de configuration système

**Aide**  : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.Cache » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. En outre, développez un processus et un pipeline pour la gestion des exceptions de stratégie.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="78-deploy-system-configuration-management-tools-for-operating-systems"></a>7.8 : Déployer les outils de gestion de la configuration système pour les systèmes d’exploitation

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="79-implement-automated-configuration-monitoring-for-azure-services"></a>7.9 : Mettre en place la surveillance de la configuration automatique pour les services Azure

**Aide**  : Utilisez des alias Azure Policy dans l’espace de noms « Microsoft.Cache » pour créer des stratégies personnalisées d’alerte, d’audit ou d’application de configurations système. Utilisez les stratégies Azure Policy [auditer], [refuser] et [déployer s’il n’existe pas] pour appliquer automatiquement des configurations pour vos instances Azure Cache pour Redis et les ressources associées.

Configurer et gérer Azure Policy : https://docs.microsoft.com/azure/governance/policy/tutorials/create-and-manage

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="710-implement-automated-configuration-monitoring-for-operating-systems"></a>7.10 : Implémenter la surveillance de la configuration automatique pour les systèmes d’exploitation

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="711-manage-azure-secrets-securely"></a>7.11 : Gérer les secrets Azure en toute sécurité

**Aide**  : Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à vos instances Azure Cache pour Redis, utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets Azure Cache pour Redis. Vérifiez que la suppression réversible est activée dans Key Vault.

Intégration aux identités managées Azure :

https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

Pour créer un coffre de clés :

https://docs.microsoft.com/azure/key-vault/general/quick-create-portal

Comment s’authentifier auprès de Key Vault :

https://docs.microsoft.com/azure/key-vault/general/authentication

Comment attribuer une stratégie d’accès Key Vault :

https://docs.microsoft.com/azure/key-vault/general/assign-access-policy-portal

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="712-manage-identities-securely-and-automatically"></a>7.12 : Gérer les identités de façon sécurisée et automatique

**Conseils**  : Pour les machines virtuelles Azure ou les applications web s’exécutant sur Azure App Service utilisées pour accéder à vos instances Azure Cache pour Redis, utilisez Managed Service Identity conjointement avec Azure Key Vault pour simplifier et sécuriser la gestion des secrets Azure Cache pour Redis. Vérifiez que la suppression réversible est activée dans Key Vault.

Utilisez des identités managées pour fournir aux services Azure une identité gérée automatiquement dans Azure Active Directory. Les identités managées vous permettent de vous authentifier auprès d’un service qui prend en charge l’authentification AAD, y compris Azure Key Vault, sans informations d’identification dans votre code.

Configuration d’identités managées :

https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm

Intégration aux identités managées Azure :

https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="713-eliminate-unintended-credential-exposure"></a>7.13 : Éliminer l’exposition involontaire des informations d’identification

**Conseils**  : Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

Configuration de Credential Scanner : https://secdevtools.azurewebsites.net/helpcredscan.html

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

## <a name="malware-defense"></a>Défense contre les programmes malveillants

*Pour plus d’informations, consultez [Contrôle de sécurité : Défense contre les programmes malveillants](../security/benchmarks/security-control-malware-defense.md).*

### <a name="81-use-centrally-managed-anti-malware-software"></a>8.1 : Utiliser un logiciel anti-programme malveillant géré de manière centralisée

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure App Service), mais il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

### <a name="82-pre-scan-files-to-be-uploaded-to-non-compute-azure-resources"></a>8.2 : Pré-analyser les fichiers à charger sur des ressources Azure non liées au calcul

**Aide**  : Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure Cache pour Redis), mais il ne s’exécute pas sur du contenu client.

Pré-analysez tout contenu chargé sur des ressources Azure non liées au calcul, comme App Service, Data Lake Storage, Stockage Blob, Azure Database pour PostgreSQL, etc. Microsoft ne peut pas accéder à vos données dans ces instances.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="83-ensure-anti-malware-software-and-signatures-are-updated"></a>8.3 : Vérifier que les logiciels et signatures anti-programme malveillant sont mis à jour

**Conseils**  : Non applicable. Cette recommandation a trait aux ressources de calcul.

Microsoft Antimalware est activé sur l’hôte sous-jacent qui prend en charge les services Azure (par exemple, Azure Cache pour Redis), mais il ne s’exécute pas sur du contenu client.

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Non applicable

## <a name="data-recovery"></a>Récupération des données

*Pour plus d’informations, consultez [Contrôle de sécurité : récupération de données](../security/benchmarks/security-control-data-recovery.md).*

### <a name="91-ensure-regular-automated-back-ups"></a>9.1 : garantir des sauvegardes automatisées régulières

**Aide**  : Activation de la persistance Redis. La persistance Redis vous permet de conserver les données stockées dans Redis. Vous pouvez également prendre des instantanés et sauvegarder les données que vous pouvez charger en cas de défaillance matérielle. Il s’agit d’un avantage substantiel par rapport au niveau De base ou Standard, où toutes les données sont stockées en mémoire et il existe un risque de perte de données en cas de défaillance des nœuds de cache.

Vous pouvez également utiliser l’exportation Azure Cache pour Redis. L’exportation vous permet d’exporter les données stockées dans le Cache Azure pour Redis vers un ou plusieurs fichiers RDB compatibles. Vous pouvez utiliser cette fonctionnalité pour déplacer des données d’une instance de Cache Azure pour Redis à une autre, ou vers un autre serveur Redis. Pendant le processus d’exportation, un fichier temporaire est créé sur la machine virtuelle qui héberge l’instance de serveur Azure Cache pour Redis, puis téléchargé vers le compte de stockage désigné. Lorsque l’opération d’exportation se termine avec un état de réussite ou d’échec, le fichier temporaire est supprimé.

Comment activer la persistance Redis :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-premium-persistence

Comment utiliser l’exportation Azure Cache pour Redis :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-import-export-data

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="92-perform-complete-system-backups-and-backup-any-customer-managed-keys"></a>9.2 : effectuer des sauvegardes complètes du système et sauvegarder les clés gérées par le client

**Aide**  : Activation de la persistance Redis. La persistance Redis vous permet de conserver les données stockées dans Redis. Vous pouvez également prendre des instantanés et sauvegarder les données que vous pouvez charger en cas de défaillance matérielle. Il s’agit d’un avantage substantiel par rapport au niveau De base ou Standard, où toutes les données sont stockées en mémoire et il existe un risque de perte de données en cas de défaillance des nœuds de cache.

Vous pouvez également utiliser l’exportation Azure Cache pour Redis. L’exportation vous permet d’exporter les données stockées dans le Cache Azure pour Redis vers un ou plusieurs fichiers RDB compatibles. Vous pouvez utiliser cette fonctionnalité pour déplacer des données d’une instance de Cache Azure pour Redis à une autre, ou vers un autre serveur Redis. Pendant le processus d’exportation, un fichier temporaire est créé sur la machine virtuelle qui héberge l’instance de serveur Azure Cache pour Redis, puis téléchargé vers le compte de stockage désigné. Lorsque l’opération d’exportation se termine avec un état de réussite ou d’échec, le fichier temporaire est supprimé.

Si vous utilisez Azure Key Vault pour stocker les informations d’identification de vos instances Azure Cache pour Redis, veillez à faire des sauvegardes automatisées régulières de vos clés.

Comment activer la persistance Redis :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-premium-persistence

Comment utiliser l’exportation Azure Cache pour Redis :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-import-export-data

Comment sauvegarder des clés Key Vault :

https://docs.microsoft.com/powershell/module/azurerm.keyvault/backup-azurekeyvaultkey

**Supervision d’Azure Security Center**  : actuellement non disponible

**Responsabilité**  : Customer

### <a name="93-validate-all-backups-including-customer-managed-keys"></a>9.3 : valider toutes les sauvegardes, y compris les clés gérées par le client

**Aide**  : Utiliser l’importation Azure Cache pour Redis. L’importation peut servir à récupérer les fichiers RDB compatibles Redis depuis un serveur Redis en cours d’exécution dans un environnement ou dans un cloud, y compris si Redis est exécuté sur Linux, Windows ou n’importe quel fournisseur de cloud tel qu’Amazon Web Services. Importer des données est un moyen simple de créer un cache pré-rempli de données. Pendant le processus d’importation, le Cache Azure pour Redis charge les fichiers RDB du stockage Azure dans la mémoire, puis insère les clés dans le cache.

Testez régulièrement la restauration des données de vos secrets Azure Key Vault.

Comment utiliser l’importation Azure Cache pour Redis :

https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-import-export-data

Comment restaurer des secrets Key Vault :

https://docs.microsoft.com/powershell/module/azurerm.keyvault/restore-azurekeyvaultsecret?view=azurermps-6.13.0

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="94-ensure-protection-of-backups-and-customer-managed-keys"></a>9.4 : garantir la protection des sauvegardes et des clés gérées par le client

**Aide**  : Les sauvegardes Azure Cache pour Redis de l’exportation Redis et la persistance Redis sont stockées dans votre compte de stockage Azure sélectionné. Les données dans Stockage Azure sont chiffrées et déchiffrées en toute transparence à l’aide du chiffrement AES 256 bits, un des chiffrements par blocs les plus puissants actuellement disponibles, et sont conformes à la norme FIPS 140-2. Le chiffrement de Stockage Azure ne peut pas être désactivé. Vous pouvez vous reposer sur les clés managées par Microsoft pour le chiffrement de votre compte de stockage, ou vous pouvez gérer le chiffrement avec vos propres clés.

Comprendre le chiffrement pour les comptes de stockage Azure : https://docs.microsoft.com/azure/storage/common/storage-service-encryption

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Microsoft

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Contrôle de sécurité : réponse aux incidents](../security/benchmarks/security-control-incident-response.md).*

### <a name="101-create-an-incident-response-guide"></a>10.1 : Créer un guide de réponse aux incidents

**Conseils**  : Créez un guide de réponse aux incidents pour votre organisation. Assurez-vous qu’il existe des plans de réponse aux incidents écrits qui définissent tous les rôles du personnel, ainsi que les phases de gestion des incidents, depuis la détection jusqu’à la revue une fois l’incident terminé.

Comment configurer des automations de flux de travail dans Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-planning-and-operations-guide

Aide sur la création de votre propre processus de réponse aux incidents de sécurité :

https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/

Anatomie d’un incident dans le centre de réponse aux incidents de sécurité Microsoft :

https://msrc-blog.microsoft.com/2019/07/01/inside-the-msrc-building-your-own-security-incident-response-process/

Le client peut également tirer parti du guide de gestion des incidents de sécurité informatique du NIST pour faciliter la création de son propre plan de réponse aux incidents :

https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="102-create-an-incident-scoring-and-prioritization-procedure"></a>10.2 : Créer une procédure de notation et de classement des incidents

**Conseils**  : Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez clairement les abonnements (par ex. production, non production) et créez un système d’attribution de noms pour identifier et classer les ressources Azure de façon claire.

**Supervision d’Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="103-test-security-response-procedures"></a>10.3 : Tester les procédures de réponse de sécurité

**Conseils**  : Exécutez des exercices pour tester les fonctionnalités de réponse aux incidents de vos systèmes de façon régulière. Identifiez les points faibles et les lacunes, et révisez le plan en fonction des besoins.

Reportez-vous à la publication du NIST : Guide de test, de formation et programmes d’exercice pour les plans informatiques et les fonctionnalités :

https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-84.pdf

**Supervision Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="104-provide-security-incident-contact-details-and-configure-alert-notifications-for-security-incidents"></a>10.4 : Fournir des informations de contact pour les incidents de sécurité et configurer des notifications d’alerte pour les incidents de sécurité

**Aide**  : Les informations de contact d’incident de sécurité seront utilisées par Microsoft pour vous contacter si Microsoft Security Response Center (MSRC) découvre que les données du client ont été utilisées par un tiers illégal ou non autorisé.  Examinez les incidents après les faits pour vous assurer que les problèmes sont résolus.

Comment définir le contact de sécurité Azure Security Center :

https://docs.microsoft.com/azure/security-center/security-center-provide-security-contact-details

**Supervision Azure Security Center**  : Oui

**Responsabilité**  : Customer

### <a name="105-incorporate-security-alerts-into-your-incident-response-system"></a>10.5 : Intégrer des alertes de sécurité à votre système de réponse aux incidents

**Conseils**  : Exportez vos alertes et recommandations d’Azure Security Center à l’aide de la fonctionnalité d’exportation continue. L’exportation continue vous permet d’exporter les alertes et les recommandations manuellement, ou automatiquement de manière continue. Vous pouvez utiliser le connecteur de données Azure Security Center pour diffuser la sentinelle des alertes.

Comment configurer l’exportation continue :

https://docs.microsoft.com/azure/security-center/continuous-export

Comment diffuser des alertes dans Azure Sentinel :

https://docs.microsoft.com/azure/sentinel/connect-azure-security-center

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

### <a name="106-automate-the-response-to-security-alerts"></a>10.6 : Automatiser la réponse aux alertes de sécurité

**Conseils**  : Utilisez la fonctionnalité d’automatisation du workflow dans Azure Security Center pour déclencher automatiquement des réponses via « Logic Apps » sur les alertes et recommandations de sécurité.

Comment configurer l’automatisation du flux de travail et Logic Apps :

https://docs.microsoft.com/azure/security-center/workflow-automation

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Customer

## <a name="penetration-tests-and-red-team-exercises"></a>Tests d’intrusion et exercices Red Team

*Pour plus d’informations, consultez [Contrôle de sécurité : tests d’intrusion et exercices Red Team](../security/benchmarks/security-control-penetration-tests-red-team-exercises.md).*

### <a name="111-conduct-regular-penetration-testing-of-your-azure-resources-and-ensure-remediation-of-all-critical-security-findings-within-60-days"></a>11.1 : Procédez régulièrement à des tests d’intrusion de vos ressources Azure et veillez à résoudre tous les problèmes de sécurité critiques détectés dans un délai de 60 jours

**Aide**  : Suivez les règles d’engagement de Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft :

https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1

Vous trouverez plus d’informations sur la stratégie de Microsoft et l’exécution des tests Red Teaming et de pénétration de sites en direct sur l’infrastructure, les services et les applications cloud gérés par Microsoft ici : 

https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e

**Supervision d’Azure Security Center**  : Non applicable

**Responsabilité**  : Partagé

## <a name="next-steps"></a>Étapes suivantes

- Consultez le [benchmark de sécurité Azure](../security/benchmarks/overview.md)
- Apprenez-en davantage sur les [Bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)