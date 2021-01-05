---
title: Ligne de base de sécurité Azure Virtual WAN
description: La base de référence de sécurité de Virtual WAN fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans Azure Security Benchmark.
author: msmbaldwin
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 11/24/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 994cb250aff2079e32eeef6a0d22466a284481f5
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602425"
---
# <a name="azure-security-baseline-for-virtual-wan"></a>Ligne de base de sécurité Azure Virtual WAN

Cette base de référence de sécurité applique les conseils du[Benchmark de sécurité Azure version 2.0](../security/benchmarks/overview.md) à Microsoft Azure Virtual WAN. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé par les **contrôles de sécurité** définis par le benchmark de sécurité Azure et les conseils associés applicables à Virtual WAN. Les **contrôles** non applicables à Virtual WAN ont été exclus.

Pour voir comment Virtual WAN est entièrement mappé au benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité Virtual WAN](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : sécurité réseau](/azure/security/benchmarks/security-controls-v2-network-security).*

### <a name="ns-1-implement-security-for-internal-traffic"></a>NS-1 : Implémenter la sécurité pour le trafic interne

**Conseils** : Microsoft Azure Virtual WAN fournit des fonctionnalités de routage personnalisées et offre un chiffrement pour votre trafic ExpressRoute. La gestion des routes est assurée en totalité par le routeur de hub virtuel, qui permet également la connectivité de transit entre les réseaux virtuels. Le chiffrement de votre trafic ExpressRoute avec Virtual WAN assure un transit chiffré entre les réseaux locaux et les réseaux virtuels Azure sur ExpressRoute, sans passer par l’Internet public ni utiliser des adresses IP publiques. 

- [Chiffrement du trafic ExpressRoute](virtual-wan-about.md#encryption)

- [Utiliser Azure Private Link dans Azure Virtual WAN](howto-private-link.md)

- [Scénarios de routage personnalisés](scenario-any-to-any.md)

- [Documentation sur la table de routage personnalisée](how-to-virtual-hub-routing.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="ns-2-connect-private-networks-together"></a>NS-2 : Interconnecter des réseaux privés 

**Conseils** : Microsoft Azure ExpressRoute offre une connectivité privée à Azure Virtual WAN. Les connexions ExpressRoute ne transitant pas par l’Internet public, ExpressRoute offre plus de fiabilité, des vitesses supérieures et des latences inférieures par rapport aux connexions Internet classiques. Vous pouvez également utiliser un réseau privé virtuel pour vous connecter à Azure via un VPN site à site (S2S) ou un VPN point à site (P2S).

- [ExpressRoute dans Virtual WAN](virtual-wan-expressroute-portal.md)

- [Vue d’ensemble des VPN site à site](virtual-wan-site-to-site-portal.md)

- [Vue d’ensemble des VPN point à site](virtual-wan-point-to-site-portal.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="ns-4-protect-applications-and-services-from-external-network-attacks"></a>NS-4 : Protégez les applications et les services contre les attaques réseau externes

**Conseils** : Virtual WAN n’expose pas de points de terminaison aux réseaux externes qui leur imposent d’être sécurisés avec des protections réseau conventionnelles. Vous êtes libre de protéger des ressources de réseaux virtuels spoke (tout réseau virtuel connecté à un hub virtuel) en utilisant des services de protection de réseau virtuel. 

Utilisez le Pare-feu Azure pour protéger les applications et les services contre le trafic potentiellement malveillant provenant d’Internet et d’autres emplacements externes. 

Choisissez la protection DDoS fournie par Azure pour protéger vos ressources contre les attaques sur vos réseaux virtuels Azure. Utilisez Azure Security Center pour détecter les risques de configuration incorrecte liés aux ressources associées à votre réseau.

- [Documentation sur le pare-feu Azure](/azure/firewall)

- [Gérer la norme Azure DDoS Protection à l’aide du portail Azure](/azure/virtual-network/manage-ddos-protection) 

- [Recommandations relatives à Azure Security Center](../security-center/recommendations-reference.md#recs-network)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="ns-5-deploy-intrusion-detectionintrusion-prevention-systems-idsips"></a>NS-5 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS)

**Conseils** : Azure Virtual WAN est un service managé par Microsoft. Il n’offre pas de fonctionnalités natives de détection ou de prévention des intrusions. Cependant, il existe des fonctionnalités de sécurité fournies à Virtual WAN via le Pare-feu Azure pour activer un point de contrôle de stratégie unifié. Vous pouvez créer une stratégie de Pare-feu Azure et lier la stratégie à un concentrateur Virtual WAN pour permettre au hub Virtual WAN existant de fonctionner comme un hub virtuel sécurisé, avec les ressources de Pare-feu Azure nécessaires déployées.

- [Configurer un Pare-feu Azure dans un hub de réseau étendu virtuel](howto-firewall.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="ns-6-simplify-network-security-rules"></a>NS-6 : Simplifier les règles de sécurité réseau

**Conseils** : Simplifiez les règles de sécurité réseau en tirant parti des étiquettes du service Réseau virtuel pour définir des contrôles d’accès réseau sur des groupes de sécurité réseau ou sur le Pare-feu Azure. Les étiquettes de service peuvent être utilisées à la place d’adresses IP spécifiques pendant la création de règles de sécurité. En spécifiant le nom de l’étiquette de service dans le champ source ou de destination d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

- [Présentation et utilisation des balises de service](../virtual-network/service-tags-overview.md)

- [Présentation et utilisation des groupes de sécurité d’application](/azure/virtual-network/security-overview#application-security-groups)

- [Documentation sur le pare-feu Azure](/azure/firewall/)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="ns-7-secure-domain-name-service-dns"></a>NS-7 : Système DNS (Domain Name System) sécurisé

**Conseils** : Des fonctionnalités DNS sécurisées sont fournies à Virtual WAN avec le Pare-feu Azure. Configurez le Pare-feu Azure pour qu’il agisse en tant que proxy DNS, qui devient un intermédiaire pour les requêtes DNS adressées par les machines virtuelles clientes à un serveur DNS. Pour les configurations de serveur DNS personnalisées, activez le proxy DNS pour éviter les non-correspondances de la résolution DNS et activez le filtrage de nom de domaine complet (FQDN) dans les règles de réseau. 

- [Documentation sur le pare-feu Azure](/azure/firewall/)

- [Paramètres DNS du Pare-feu Azure](/azure/firewall/dns-settings)

- [Utiliser le Pare-feu Azure comme redirecteur DNS avec Private Link](https://github.com/adstuart/azure-privatelink-dns-azurefirewall)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="identity-management"></a>Gestion des identités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des identités](/azure/security/benchmarks/security-controls-v2-identity-management).*

### <a name="im-1-standardize-azure-active-directory-as-the-central-identity-and-authentication-system"></a>IM-1 : Normaliser Azure Active Directory comme système d’authentification et d’identité central

**Conseils** : Azure Active Directory (Azure AD) est le service par défaut de gestion des identités et des accès pour les services Azure, y compris Virtual WAN. Normalisez Azure AD pour régir la gestion des identités et des accès de votre organisation dans :

- Les ressources cloud Microsoft, comme le portail Azure, Stockage Azure, les machines virtuelles Azure (Linux et Windows), Azure Key Vault, PaaS et les applications SaaS.
- Les ressources de votre organisation, comme les applications sur Azure ou les ressources réseau de votre entreprise.

Sécurisez Azure AD avec une priorité élevée dans les pratiques de sécurité cloud de votre organisation. Évaluez votre identité et votre niveau de sécurité avec la fonctionnalité de score de sécurité d’Azure Security Center, pour estimer dans quelle mesure votre configuration applique les recommandations des bonnes pratiques de Microsoft. Implémentez selon les nécessités les recommandations des bonnes pratiques de Microsoft pour améliorer votre niveau de sécurité.

Azure AD prend également en charge les identités externes, qui permettent aux utilisateurs sans compte Microsoft de se connecter à leurs applications et ressources avec leur identité externe. 

Consultez les informations sur l’utilisation d’Azure AD dans les scénarios VPN point à site disponibles dans les liens référencés.

- [Créer un locataire Azure Active Directory (AD) pour les connexions P2S de protocole OpenVPN](openvpn-azure-ad-tenant-multi-app.md)

- [Configurer l’authentification Azure Active Directory pour une connexion VPN utilisateur](virtual-wan-point-to-site-azure-ad.md)

- [Locataires dans Azure Active Directory](../active-directory/develop/single-and-multi-tenant-apps.md) 

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="im-4-use-strong-authentication-controls-for-all-azure-active-directory-based-access"></a>IM-4 : Utiliser des contrôles d’authentification renforcés pour tous les accès basés sur Azure Active Directory

**Conseils** : Actuellement, l’authentification Azure Active Directory (Azure AD) est fournie via l’intégration à un VPN point à site Virtual WAN.

Azure Active Directory (Azure AD) est le service par défaut de gestion des identités et des accès pour les services Azure, Azure AD prend en charge des contrôles d’authentification forte via l’authentification multifacteur et des méthodes fortes sans mot de passe.

Azure AD recommande ce qui suit pour les contrôles d’authentification forte :

- Authentification multifacteur : activez l’authentification multifacteur Azure AD et suivez les recommandations relatives à la gestion des identités et des accès dans Azure Security Center pour les bonnes pratiques de sécurité. Appliquez l’authentification multifacteur à tous les utilisateurs, à certains utilisateurs ou au niveau de chaque utilisateur, en fonction des conditions de connexion et des facteurs de risque.

- Authentification sans mot de passe : trois options d’authentification sans mot de passe sont proposées. Il s’agit de Windows Hello Entreprise, de l’application Microsoft Authenticator et des méthodes d’authentification locales, comme les cartes à puce.

Vérifiez que le niveau le plus élevé des méthodes d’authentification forte est utilisé pour les administrateurs et les utilisateurs privilégiés, puis déployez une stratégie d’authentification forte pour les autres utilisateurs.

- [Procédure d’activation de l’authentification multifacteur dans P2S VPN pour Virtual WAN](openvpn-azure-ad-mfa.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="im-6-restrict-azure-resource-access-based-on-conditions"></a>IM-6 : Restreindre l’accès aux ressources Azure en fonction des conditions

**Conseils** : Activez l’authentification multifacteur Azure Active Directory (Azure AD) pour les utilisateurs du VPN (point à site) avec l’authentification Azure AD. Configurez l’authentification multifacteur par utilisateur ou tirez parti de l’authentification multifacteur avec accès conditionnel. L’accès conditionnel autorise un contrôle plus précis sur la façon dont le deuxième facteur doit être promu. Il peut autoriser l’affectation de l’authentification MFA seulement au VPN et exclure les autres applications liées au locataire Azure AD. 

Notez que l’authentification Azure AD est disponible seulement pour les passerelles utilisant le protocole OpenVPN et les clients exécutant Windows.

- [Qu’est-ce que l’accès conditionnel](../active-directory/conditional-access/overview.md)

- [Configurer l’authentification Azure Active Directory pour une connexion VPN utilisateur](virtual-wan-point-to-site-azure-ad.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-7-eliminate-unintended-credential-exposure"></a>IM-7 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Le réseau VPN de site à site dans Virtual WAN utilise des clés prépartagées (PSK) qui sont découvertes, créées et gérées par le client dans son coffre Azure Key Vault. Exécuter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

Pour GitHub, vous pouvez utiliser la fonctionnalité native d’analyse de secret pour identifier les informations d’identification ou d’autres formes de secrets dans le code.

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html) 

- [Analyse du secret GitHub](https://docs.github.com/github/administering-a-repository/about-secret-scanning)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="privileged-access"></a>Accès privilégié

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Accès privilégié](/azure/security/benchmarks/security-controls-v2-privileged-access).*

### <a name="pa-2-restrict-administrative-access-to-business-critical-systems"></a>PA-2 : Limiter l’accès administratif aux systèmes critiques de l’entreprise

**Conseils** : Azure Virtual WAN utilise les contrôles d’accès en fonction du rôle Azure (Azure RBAC) pour isoler l’accès aux systèmes vitaux pour l'entreprise en restreignant les comptes disposant d’un accès privilégié aux abonnements et aux groupes d’administration dans lesquels ils se trouvent.

Limitez également l’accès aux systèmes de gestion, d’identité et de sécurité qui ont un accès administratif à vos ressources critiques, comme les contrôleurs de domaine Active Directory, les outils de sécurité et les outils de gestion du système avec les agents installés sur les systèmes critiques de l’entreprise. Les attaquants qui compromettent ces systèmes de gestion et de sécurité peuvent immédiatement les armer pour compromettre les ressources stratégiques de l’entreprise.

Pour assurer un contrôle d’accès cohérent, tous les types de contrôle d’accès doivent être alignés sur la stratégie de segmentation de votre entreprise.

- [Composants Azure et modèle de référence](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151) 

- [Accès aux groupes d’administration](../governance/management-groups/overview.md#management-group-access) 

- [Administrateurs d’abonnements Azure](../cost-management-billing/manage/add-change-subscription-administrator.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : protection des données](/azure/security/benchmarks/security-controls-v2-data-protection).*

### <a name="dp-4-encrypt-sensitive-information-in-transit"></a>DP-4 : Chiffrement des informations sensibles en transit

**Conseils** : Utilisez un VPN de point à site, un VPN de site à site et une route ExpressRoute chiffrée avec Virtual WAN pour vos besoins de connectivité. Le chiffrement VPN protège les données en transit contre les attaques « hors bande » (comme la capture de trafic) pour empêcher les attaquants de lire ou de modifier les données. 

- [VPN point à site](virtual-wan-point-to-site-portal.md)

- [VPN site à site](virtual-wan-site-to-site-portal.md)

- [ExpressRoute chiffré](vpn-over-expressroute.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

## <a name="asset-management"></a>Gestion des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des ressources](/azure/security/benchmarks/security-controls-v2-asset-management).*

### <a name="am-1-ensure-security-team-has-visibility-into-risks-for-assets"></a>AM-1 : S’assurer que l’équipe de sécurité a une visibilité sur les risques pour les ressources

**Conseils** : Assurez-vous que les équipes de sécurité reçoivent des autorisations de lecteur de sécurité dans votre locataire et vos abonnements Azure afin qu’elles puissent surveiller les risques de sécurité à l’aide d’Azure Security Center. 

Selon la façon dont les responsabilités de l’équipe de sécurité sont structurées, la surveillance des risques de sécurité peut tenir de la responsabilité d’une équipe de sécurité centrale ou d’une équipe locale. Avec cela, les insights et les risques liés à la sécurité doivent toujours être agrégés de façon centralisée au sein d’une organisation. 

Les autorisations du Lecteur Sécurité peuvent être appliquées globalement à tout un locataire (groupe d’administration racine), ou délimitées à des groupes d’administration ou des abonnements spécifiques. 

Remarque : Des autorisations supplémentaires peuvent être nécessaires pour obtenir une visibilité sur les charges de travail et services. 

- [Vue d’ensemble du rôle lecteur de sécurité](../role-based-access-control/built-in-roles.md#security-reader)

- [Vue d'ensemble des groupes d'administration Azure](../governance/management-groups/overview.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="am-2-ensure-security-team-has-access-to-asset-inventory-and-metadata"></a>AM-2 : S’assurer que l’équipe de sécurité a accès à l’inventaire des ressources et aux métadonnées

**Conseils** : Appliquez des étiquettes à vos ressources Azure, groupes de ressources et abonnements pour les organiser de façon logique dans une taxonomie. Chaque balise se compose d’une paire nom-valeur. Par exemple, vous pouvez appliquer le nom « Environnement » et la valeur « Production » à toutes les ressources en production. Azure Virtual WAN prend également en charge les déploiements de ressources basés sur Azure Resource Manager, avec lesquels vous pouvez exporter des modèles de ressources. 

- [Guides de décision concernant le nommage et l’étiquetage des ressources](https://docs.microsoft.com/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=/azure/azure-resource-manager/management/toc.json)

- [Gestion de l’inventaire de sécurité dans Azure Security Center](../security-center/asset-inventory.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="am-3-use-only-approved-azure-services"></a>AM-3 : Utiliser des services Azure approuvés uniquement

**Conseils** : Utilisez Azure Monitor pour créer des règles afin de déclencher des alertes lorsqu’un service non approuvé est détecté. Virtual WAN combine un grand nombre de fonctionnalités réseau, de sécurité et de routage pour fournir une interface opérationnelle unique. Les passerelles VPN Virtual WAN, les passerelles ExpressRoute et Pare-feu Azure disposent de journaux et de métriques disponibles via Azure Monitor. 
 

- [Journaux et métriques de Virtual WAN](logs-metrics.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="am-5-limit-users-ability-to-interact-with-azure-resource-manager"></a>AM-5 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Conseils** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Configuration de l’accès conditionnel pour bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="logging-and-threat-detection"></a>Journalisation et détection des menaces

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Journalisation et détection des menaces](/azure/security/benchmarks/security-controls-v2-logging-threat-detection).*

### <a name="lt-1-enable-threat-detection-for-azure-resources"></a>LT-1 : Activer la détection des menaces pour les ressources Azure

**Conseils** : Le VPN de point à site avec Virtual WAN est intégré à Azure Active Directory (Azure AD). Azure AD fournit les journaux utilisateur suivants qui peuvent être consultés dans les rapports Azure AD ou intégrés à Azure Monitor, à Azure Sentinel ou à d’autres outils SIEM de surveillance pour des cas d’usage plus sophistiqués de supervision et d’analytique des menaces. Ces règles sont les suivantes :

- Connexion : le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.
- Journaux d’audit : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Par exemple, les journaux d’audit peuvent inclure les modifications apportées à des ressources dans Azure AD comme l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles et de stratégies.
- Connexion risquée : une connexion risquée indique une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.
- Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Azure Security Center permet de créer des alertes en cas d’activités suspectes, comme un nombre excessif de tentatives d’authentification ayant échoué, notamment de comptes déconseillés dans l’abonnement. En plus de la supervision de base de l’hygiène de sécurité, utilisez le module Protection contre les menaces de Security Center pour collecter des alertes de sécurité plus approfondies auprès des ressources de calcul Azure individuelles (machines virtuelles, conteneurs, service d’application), de ressources de données (base de données SQL et stockage) et de couches de service Azure. Cette capacité vous permet d’avoir une visibilité sur les anomalies de compte à l’intérieur des ressources individuelles.

- [Rapports d’activité d’audit dans Azure Active Directory](../active-directory/reports-monitoring/concept-audit-logs.md) 

- [Activer Azure AD Identity Protection](../active-directory/identity-protection/overview-identity-protection.md)

- [Configurer un Pare-feu Azure dans un hub de réseau étendu virtuel](howto-firewall.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="lt-2-enable-threat-detection-for-azure-identity-and-access-management"></a>LT-2 : Activer la détection des menaces pour la gestion des identités et des accès Azure

**Conseils** : Le VPN de point à site avec Virtual WAN est intégré à Azure Active Directory (Azure AD). Azure AD fournit les journaux utilisateur suivants qui peuvent être consultés dans les rapports Azure AD ou intégrés à Azure Monitor, à Azure Sentinel ou à d’autres outils SIEM de surveillance pour des cas d’usage plus sophistiqués de supervision et d’analytique des menaces. Ces règles sont les suivantes :

- Connexion : le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.
- Journaux d’audit : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Par exemple, les journaux d’audit peuvent inclure les modifications apportées à des ressources dans Azure AD comme l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles et de stratégies.
- Connexion risquée : une connexion risquée indique une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.
- Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Azure Security Center permet de créer des alertes en cas d’activités suspectes, comme un nombre excessif de tentatives d’authentification ayant échoué, notamment de comptes déconseillés dans l’abonnement. En plus de la supervision de base de l’hygiène de sécurité, utilisez le module Protection contre les menaces de Security Center pour collecter des alertes de sécurité plus approfondies auprès des ressources de calcul Azure individuelles (machines virtuelles, conteneurs, service d’application), de ressources de données (base de données SQL et stockage) et de couches de service Azure. Cette capacité vous permet d’avoir une visibilité sur les anomalies de compte à l’intérieur des ressources individuelles.

- [Rapports d’activité d’audit dans Azure Active Directory](../active-directory/reports-monitoring/concept-audit-logs.md) 

- [Activer Azure AD Identity Protection](../active-directory/identity-protection/overview-identity-protection.md)

- [Configurer un Pare-feu Azure dans un hub de réseau étendu virtuel](howto-firewall.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="lt-3-enable-logging-for-azure-network-activities"></a>LT-3 : Activer la journalisation pour les activités réseau Azure

**Conseils** : Supervisez Azure Virtual WAN avec Azure Monitor. Virtual WAN combine un grand nombre de fonctionnalités réseau, de sécurité et de routage pour fournir une interface opérationnelle unique. Les passerelles VPN Virtual WAN, les passerelles ExpressRoute et Pare-feu Azure disposent de journaux et de métriques disponibles via Azure Monitor. Les entrées du journal d’activité sont recueillies par défaut et peuvent être affichées dans le portail Azure. Vous pouvez utiliser les journaux d’activité Azure (anciennement journaux des opérations et journaux d’audit) pour afficher toutes les opérations soumises à votre abonnement Azure.

Un grand nombre de journaux de diagnostic sont également disponibles pour Virtual WAN et peuvent être configurés pour la ressource Virtual WAN avec le portail Azure.  Vous pouvez choisir d’envoyer à Log Analytics, de diffuser vers un Event Hub ou de simplement archiver dans un compte de stockage. 
 

- [Journaux et métriques de Virtual WAN](logs-metrics.md)

- [Journaux et métriques du pare-feu Azure](/azure/firewall/logs-and-metrics)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="lt-4-enable-logging-for-azure-resources"></a>LT-4 : Activer la journalisation pour les ressources Azure

**Conseils** : Les journaux d’activité Azure, activés automatiquement, contiennent toutes les opérations d’écriture (PUT, POST, DELETE) pour vos ressources Azure Virtual WAN, à l’exception des opérations de lecture (GET). Les journaux d’activité peuvent être utilisés pour rechercher une erreur pendant la résolution de problèmes ou pour surveiller la manière dont un utilisateur de votre organisation a modifié une ressource.

Activez les journaux de ressources Azure pour Virtual WAN. Vous pouvez utiliser Azure Security Center et Azure Policy pour activer la collecte des journaux de ressources et des données de journaux. Ces journaux peuvent être essentiels pour l’examen ultérieur des incidents de sécurité et l’exécution d’exercices légaux.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

- [Présentation de la journalisation et des différents types de journaux dans Azure](../azure-monitor/platform/platform-logs-overview.md) 

- [Présentation de la collecte de données Azure Security Center](../security-center/security-center-enable-data-collection.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="lt-5-centralize-security-log-management-and-analysis"></a>LT-5 : Centraliser la gestion et l’analyse des journaux de sécurité

**Conseils** : Activez la journalisation de la sécurité pour Virtual WAN avec Azure Monitor. Virtual WAN combine un grand nombre de fonctionnalités réseau, de sécurité et de routage pour fournir une interface opérationnelle unique. Les passerelles VPN Virtual WAN, les passerelles ExpressRoute et Pare-feu Azure disposent de journaux et de métriques disponibles via Azure Monitor. Les entrées du journal d’activité sont recueillies par défaut et peuvent être affichées dans le portail Azure. Vous pouvez utiliser les journaux d’activité Azure (anciennement journaux des opérations et journaux d’audit) pour afficher toutes les opérations soumises à votre abonnement Azure. 

Un grand nombre de journaux de diagnostic sont également disponibles pour Virtual WAN et peuvent être configurés pour la ressource Virtual WAN avec le portail Azure. Envoyez à Log Analytics, diffusez en continu vers un hub d’événements ou archivez simplement dans un compte de stockage. En outre, activez et intégrez des données à Azure Sentinel ou à une solution SIEM (Security Information and Event Management) de tiers. 
 

- [Journaux et métriques de Virtual WAN](logs-metrics.md)

- [Journaux et métriques du pare-feu Azure](/azure/firewall/logs-and-metrics)

La sécurité d’Azure Virtual WAN est fournie via le Pare-feu Azure. 

- [Documentation sur le pare-feu Azure](/azure/firewall/overview)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Partagé

### <a name="lt-6-configure-log-storage-retention"></a>LT-6 : Configurer la rétention du stockage des journaux

**Conseils** : Configurez la rétention des journaux en fonction de vos besoins en matière de conformité, de réglementation et d’exigences métier. Dans Azure Monitor, vous pouvez définir la période de rétention de votre espace de travail Log Analytics en fonction des règles de conformité de votre organisation. Utilisez des comptes Stockage Azure, Data Lake ou d’espace de travail Log Analytics pour le stockage à long terme et l’archivage.

- [Modification de la période de conservation des données dans Log Analytics](../azure-monitor/platform/manage-cost-storage.md#change-the-data-retention-period)

- [Guide pratique pour configurer la stratégie de conservation des journaux de compte de Stockage Azure](../storage/common/storage-monitor-storage-account.md#configure-logging)

- [Exportation des alertes et recommandations Azure Security Center](../security-center/continuous-export.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](/azure/security/benchmarks/security-controls-v2-incident-response).*

### <a name="ir-1-preparation--update-incident-response-process-for-azure"></a>IR-1 : Préparation – mettre à jour le processus de réponse aux incidents pour Azure

**Conseils** : Assurez-vous que votre organisation dispose de processus pour répondre aux incidents de sécurité, qu’elle a mis à jour ces processus pour Azure et qu’elle les exerce régulièrement pour garantir la préparation.

- [Implémenter la sécurité dans l’environnement de l’entreprise](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

- [Guide de référence sur les réponses aux incidents](/microsoft-365/downloads/IR-Reference-Guide.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="ir-2-preparation--setup-incident-notification"></a>IR-2 : Préparation – configurer la notification d’incident

**Conseils** : Configurez les coordonnées des personnes à contacter en cas d’incident de sécurité dans Azure Security Center. Microsoft utilisera ces coordonnées afin de vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Vous avez également la possibilité de personnaliser les alertes et les notifications d’incidents dans différents services Azure en fonction de vos besoins en matière de réponse aux incidents. 

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="ir-3-detection-and-analysis--create-incidents-based-on-high-quality-alerts"></a>IR-3 : Détection et analyse – créer des incidents en fonction d’alertes de haute qualité

**Conseils** : Assurez-vous de disposer d’un processus permettant de créer des alertes de haute qualité et de mesurer la qualité des alertes. Cela vous permet de tirer les leçons des incidents passés et de classer par ordre de priorité les alertes pour les analystes, afin qu’ils ne perdent pas de temps sur les faux positifs. 

Les alertes de haute qualité peuvent être créées en fonction de l’expérience acquise lors des incidents passés, des sources validées de la communauté et des outils conçus pour générer et nettoyer les alertes en fusionnant et en mettant en corrélation diverses sources de signaux. 

Azure Security Center fournit des alertes de haute qualité sur de nombreuses ressources Azure. Vous pouvez utiliser le connecteur de données ASC pour diffuser en continu les alertes vers Azure Sentinel. Azure Sentinel vous permet de créer des règles d’alerte avancées pour générer automatiquement des incidents à des fins d’enquête. 

Exportez vos alertes et recommandations Azure Security Center en utilisant la fonctionnalité d’exportation pour identifier les risques pesant sur les ressources Azure. Exportez les alertes et les recommandations manuellement ou automatiquement de manière continue.

- [Procédure de configuration de l’exportation](../security-center/continuous-export.md)

- [Comment envoyer des alertes à Azure Sentinel](../sentinel/connect-azure-security-center.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="ir-4-detection-and-analysis--investigate-an-incident"></a>IR-4 : Détection et analyse – enquêter sur un incident

**Conseils** : Veiller à ce que les analystes puissent interroger et utiliser diverses sources de données lorsqu’ils enquêtent sur des incidents potentiels, afin d’obtenir une vision complète de ce qui s’est passé. Différents journaux doivent être collectés pour suivre les activités d’un attaquant potentiel tout au long de la chaîne de destruction afin d’éviter les angles morts.  Vous devez également vous assurer que les insights et les enseignements sont capturés pour d’autres analystes et pour une référence historique future.  

Les sources de données à examiner comprennent les sources de journalisation centralisées qui sont déjà collectées auprès des services et des systèmes en fonctionnement concernés, mais elles peuvent également inclure les éléments suivants :

- Données réseau : utilisez les journaux de flux des groupes de sécurité réseau, Azure Network Watcher et Azure Monitor pour capturer des journaux de flux réseau et d’autres informations analytiques. 

- Captures instantanées des systèmes en fonctionnement : 

    - Utilisez la capacité de capture instantanée de la machine virtuelle Azure pour créer un instantané du disque du système en fonctionnement. 

    - Utilisez la capacité native de sauvegarde de la mémoire du système d’exploitation pour créer un instantané de la mémoire du système en fonctionnement.

    - Utilisez la capacité de capture instantanée des services Azure ou celle de votre logiciel pour créer des instantanés des systèmes en fonctionnement.

Azure Sentinel fournit des analyses de données approfondies sur pratiquement toutes les sources de journal et un portail de gestion des cas pour gérer le cycle de vie complet des incidents. Les renseignements obtenus au cours d’une enquête peuvent être associés à un incident à des fins de suivi et de rapport. 

- [Capture instantanée du disque d’un ordinateur Windows](../virtual-machines/windows/snapshot-copy-managed-disk.md)

- [Capture instantanée du disque d’un ordinateur Linux](../virtual-machines/linux/snapshot-copy-managed-disk.md)

- [Collecte de l’image mémoire et des informations de diagnostic par le support Microsoft Azure](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) 

- [Examiner les incidents avec Azure Sentinel](../sentinel/tutorial-investigate-cases.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="ir-5-detection-and-analysis--prioritize-incidents"></a>IR-5 : Détection et analyse – classer par ordre de priorité les incidents

**Conseils** : Donnez aux analystes le contexte sur lequel les incidents doivent se concentrer en premier lieu en fonction de la gravité de l’alerte et de la sensibilité des ressources. 

Azure Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou aux données analytiques utilisées pour émettre l’alerte, mais aussi de l’intention malveillante estimée de l’activité à l’origine de l’alerte.

En outre, marquez les ressources à l’aide d’étiquettes et créez un système de nommage pour identifier et classer les ressources Azure, en particulier celles qui traitent des données sensibles.  Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

- [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md)

- [Organisation des ressources Azure à l’aide de catégories](/azure/azure-resource-manager/resource-group-using-tags)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="ir-6-containment-eradication-and-recovery--automate-the-incident-handling"></a>IR-6 : Confinement, éradication et récupération – automatiser la gestion des incidents

**Conseils** : Automatisez les tâches manuelles répétitives pour accélérer le temps de réponse et réduire la charge de travail des analystes. Les tâches manuelles sont plus longues à exécuter, ce qui ralentit chaque incident et réduit le nombre d’incidents qu’un analyste peut traiter. Les tâches manuelles augmentent également la fatigue des analystes, ce qui accroît le risque d’erreur humaine entraînant des retards et dégrade la capacité des analystes à se concentrer efficacement sur des tâches complexes. Utilisez les fonctionnalités d’automatisation des workflows dans Azure Security Center et Azure Sentinel pour déclencher automatiquement des actions ou exécuter un playbook pour répondre aux alertes de sécurité entrantes. Le playbook prend des mesures, telles que l’envoi de notifications, la désactivation de comptes et l’isolement des réseaux problématiques. 

- [Configurer l’automatisation du workflow dans Security Center](../security-center/workflow-automation.md)

- [Configurer des réponses automatisées aux menaces dans Azure Security Center](../security-center/tutorial-security-incident.md#triage-security-alerts)

- [Configurer des réponses automatisées aux menaces dans Azure Sentinel](../sentinel/tutorial-respond-threats-playbook.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="posture-and-vulnerability-management"></a>Gestion de la posture et des vulnérabilités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion de la posture et des vulnérabilités](/azure/security/benchmarks/security-controls-v2-posture-vulnerability-management).*

### <a name="pv-8-conduct-regular-attack-simulation"></a>PV-8 : Effectuer une simulation d’attaque régulière

**Conseils** : Selon les besoins, effectuez un test d’intrusion ou des activités Red Team sur vos ressources Azure et résolvez tous les problèmes de sécurité critiques détectés.
Suivez les règles d’engagement de pénétration du cloud Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

- [Test d’intrusion dans Azure](../security/fundamentals/pen-testing.md)

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="endpoint-security"></a>Sécurité des points de terminaison

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Sécurité des points de terminaison](/azure/security/benchmarks/security-controls-v2-endpoint-security).*

### <a name="es-1-use-endpoint-detection-and-response-edr"></a>ES-1 : Utiliser la détection de point de terminaison et réponse (EDR)

**Conseils** : Les clients ne sont pas explicitement autorisés à configurer les paramètres de détection et de réponse des points de terminaison. Toutefois, les machines virtuelles utilisées dans l’offre Azure Virtual WAN utilisent ces fonctionnalités. En savoir plus sur ces fonctionnalités générales à partir des liens référencés. 

- [Vue d’ensemble de Microsoft Defender Advanced Threat Protection](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection)

- [Service Microsoft Defender ATP pour les serveurs Windows](/windows/security/threat-protection/microsoft-defender-atp/configure-server-endpoints)

- [Service Microsoft Defender ATP pour les serveurs non Windows](/windows/security/threat-protection/microsoft-defender-atp/configure-endpoints-non-windows)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

## <a name="governance-and-strategy"></a>Gouvernance et stratégie

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gouvernance et stratégie](/azure/security/benchmarks/security-controls-v2-governance-strategy).*

### <a name="gs-1-define-asset-management-and-data-protection-strategy"></a>GS-1 : Définir la stratégie de gestion des ressources et de protection des données 

**Conseils** : Veillez à documenter et à communiquer une stratégie claire pour la surveillance et la protection continues des systèmes et des données. Hiérarchisez la découverte, l’évaluation, la protection et la surveillance des données et systèmes critiques de l’entreprise. 

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

-   Norme de classification des données en fonction des risques pour l’entreprise

-   Visibilité de l’organisation de sécurité sur les risques et l’inventaire des actifs 

-   Approbation de l’organisation de sécurité pour les services Azure en vue de leur utilisation 

-   Sécurité des ressources tout au long de leur cycle de vie

-   Stratégie de contrôle d’accès requise conformément à la classification des données organisationnelles

-   Utilisation des fonctionnalités de protection des données natives et tierces d’Azure

-   Exigences de chiffrement des données pour les cas d’utilisation en transit et au repos

-   Normes de chiffrement appropriées

Pour plus d’informations, consultez les références suivantes :
- [Recommandation d’architecture de sécurité Azure - Stockage, données et chiffrement](https://docs.microsoft.com/azure/architecture/framework/security/storage-data-encryption?toc=/security/compass/toc.json&amp;bc=/security/compass/breadcrumb/toc.json)

- [Notions de base de la sécurité Azure - Sécurité, chiffrement et stockage des données Azure](../security/fundamentals/encryption-overview.md)

- [Cloud Adoption Framework - Meilleures pratiques en matière de chiffrement et de sécurité des données Azure](https://docs.microsoft.com/azure/security/fundamentals/data-encryption-best-practices?toc=/azure/cloud-adoption-framework/toc.json&amp;bc=/azure/cloud-adoption-framework/_bread/toc.json)

- [Benchmark de sécurité Azure - Gestion des ressources](/azure/security/benchmarks/security-controls-v2-asset-management)

- [Benchmark de sécurité Azure - Protection des données](/azure/security/benchmarks/security-controls-v2-data-protection)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-2-define-enterprise-segmentation-strategy"></a>GS-2 : Définir la stratégie de segmentation d'entreprise 

**Conseils** : Établissez une stratégie à l'échelle de l'entreprise pour segmenter l'accès aux ressources à l'aide d'une combinaison de contrôles d'identité, de réseau, d'application, d'abonnement, de groupe de gestion et autres.

Trouvez le bon équilibre entre la nécessité de séparation sur le plan de la sécurité et la nécessité d'exécuter quotidiennement les systèmes qui doivent communiquer entre eux et accéder aux données.

Veillez à ce que la stratégie de segmentation soit implémentée de manière cohérente pour tous les types de contrôle, y compris pour les modèles d'identité, d'accès et de sécurité du réseau, les modèles d'autorisation/d'accès aux applications et les contrôles des processus humains.

- [Aide relative à la stratégie de segmentation dans Azure (vidéo)](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151)

- [Aide relative à la stratégie de segmentation dans Azure (document)](/security/compass/governance#enterprise-segmentation-strategy)

- [Aligner la segmentation du réseau avec la stratégie de segmentation d’entreprise](/security/compass/network-security-containment#align-network-segmentation-with-enterprise-segmentation-strategy)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-3-define-security-posture-management-strategy"></a>GS-3 : Définir la stratégie de gestion de la posture de la sécurité

**Conseils** : Mesurez et atténuez en permanence les risques liés à vos ressources individuelles et à l’environnement dans lequel elles sont hébergées. Priorisez les ressources à valeur élevée et les surfaces d’attaque hautement exposées, comme les applications publiées, les points d’entrée et de sortie du réseau, les points de terminaison utilisateur et administrateur, etc.

- [Benchmark de sécurité Azure - Gestion de la posture et des vulnérabilités](/azure/security/benchmarks/security-controls-v2-posture-vulnerability-management)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-4-align-organization-roles-responsibilities-and-accountabilities"></a>GS-4 : Aligner les rôles et les responsabilités de l’organisation

**Conseils** : Veillez à documenter et à communiquer une stratégie claire pour les rôles et les responsabilités de votre organisation de sécurité. Veillez à définir clairement les responsabilités pour les décisions relatives à la sécurité, à former tout le monde au modèle de responsabilité partagée et à former les équipes techniques à la technologie permettant de sécuriser le cloud.

- [Meilleures pratiques pour la sécurité Azure 1 – Personnes : Former les équipes pour le parcours vers la sécurité dans le cloud](/azure/cloud-adoption-framework/security/security-top-10#1-people-educate-teams-about-the-cloud-security-journey)

- [Meilleures pratiques pour la sécurité Azure 2 – Personnes : Former les équipes pour les technologies de sécurité dans le cloud](/azure/cloud-adoption-framework/security/security-top-10#2-people-educate-teams-on-cloud-security-technology)

- [Meilleures pratiques pour la sécurité Azure 3 – Processus : Affecter les responsabilités pour les décisions de sécurité dans le cloud](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-5-define-network-security-strategy"></a>GS-5 : Définir la stratégie de sécurité réseau

**Conseils** : Établissez une approche de sécurité réseau Azure dans le cadre de la stratégie de contrôle d’accès de sécurité globale de votre organisation.  

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

-   Responsabilité centralisée pour la gestion et la sécurité du réseau

-   Modèle de segmentation de réseau virtuel aligné avec la stratégie de segmentation de l’entreprise

-   Stratégie de correction dans différents scénarios de menaces et d’attaques

-   Stratégie de périphérie d’Internet et d’entrée et de sortie

-   Stratégie de cloud hybride et d’interconnexion locale

-   Artefacts de sécurité réseau à jour (par exemple diagrammes réseau, architecture de réseau de référence)

Pour plus d’informations, consultez les références suivantes :
- [Meilleures pratiques pour la sécurité Azure 11 – Architecture. Stratégie de sécurité unifiée unique](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy)

- [Benchmark de sécurité Azure – Sécurité réseau](/azure/security/benchmarks/security-controls-v2-network-security)

- [Vue d’ensemble de la sécurité réseau d’Azure](../security/fundamentals/network-overview.md)

- [Stratégie d’architecture de réseau d’entreprise](/azure/cloud-adoption-framework/ready/enterprise-scale/architecture)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-6-define-identity-and-privileged-access-strategy"></a>GS-6 : Définir une stratégie d’accès privilégié et d’identité

**Conseils** : Établissez une approche d’identité Azure et d’accès privilégié dans le cadre de la stratégie de contrôle d’accès de sécurité globale de votre organisation.  

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

-   Un système centralisé d’identité et d’authentification et son interconnexion avec d’autres systèmes d’identité internes et externes

-   Méthodes d’authentification fortes dans différents cas d’usage et différentes conditions

-   Protection des utilisateurs disposant de privilèges élevés

-   Surveillance et gestion des activités anormales des utilisateurs  

-   Vérification de l’identité et de l’accès des utilisateurs et processus de rapprochement

Pour plus d’informations, consultez les références suivantes :

- [Benchmark de sécurité Azure - Gestion des identités](/azure/security/benchmarks/security-controls-v2-identity-management)

- [Benchmark de sécurité Azure - Accès privilégié](/azure/security/benchmarks//security-controls-v2-privileged-access)

- [Meilleures pratiques pour la sécurité Azure 11 – Architecture. Stratégie de sécurité unifiée unique](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy)

- [Vue d’ensemble de la sécurité et de la gestion des identités Azure](../security/fundamentals/identity-management-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-7-define-logging-and-threat-response-strategy"></a>GS-7 : Définir la stratégie de journalisation et de réponse aux menaces

**Conseils** : Établissez une stratégie de journalisation et de réponse aux menaces pour détecter et corriger rapidement les menaces tout en répondant aux exigences de conformité. Veillez à ce que les analystes reçoivent des alertes de haute qualité et des expériences homogènes afin qu’ils puissent se concentrer sur les menaces plutôt que sur l’intégration et les étapes manuelles. 

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

-   Rôle et responsabilités de l’organisation d’opérations de sécurité (SecOP) 

-   Un processus de réponse aux incidents bien défini, aligné avec NIST ou autre cadre réglementaire du secteur 

-   Capture et rétention des journaux pour prendre en charge la détection des menaces, la réponse aux incidents et les besoins de conformité

-   Visibilité centralisée des informations de corrélation sur les menaces, avec SIEM, les fonctionnalités Azure natives et d’autres sources 

-   Plan de communication et de notification avec vos clients, fournisseurs et les parties publiques pertinentes

-   Utilisation de plateformes Azure natives et tierces pour la gestion des incidents, comme la journalisation et la détection des menaces, les investigations et la correction et l’éradication des attaques

-   Processus de gestion des incidents et des activités postérieures aux incidents, comme les leçons apprises et la rétention des preuves

Pour plus d’informations, consultez les références suivantes :

- [Benchmark de sécurité Azure - Journalisation et détection des menaces](/azure/security/benchmarks/security-controls-v2-logging-threat-detection)

- [Benchmark de sécurité Azure - Réponse aux incidents](/azure/security/benchmarks/security-controls-v2-incident-response)

- [Meilleures pratiques pour la sécurité Azure 4 - Processus. Mise à jour des processus de réponse aux incidents pour le cloud](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

- [Guide pour le cadre d’adoption d’Azure, la journalisation et la prise de décision pour les rapports](/azure/cloud-adoption-framework/decision-guides/logging-and-reporting/)

- [Mise à l’échelle, gestion et surveillance d’entreprise Azure](/azure/cloud-adoption-framework/ready/enterprise-scale/management-and-monitoring)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Vue d’ensemble d’Azure Security Benchmark V2](/azure/security/benchmarks/overview)
- En savoir plus sur les [bases de référence de la sécurité Azure](/azure/security/benchmarks/security-baselines-overview)
