---
title: Base de référence de sécurité Azure pour Azure HPC Cache
description: La base de référence de sécurité pour Azure HPC Cache fournit des instructions et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le Benchmark de sécurité Azure.
author: msmbaldwin
ms.service: hpc-cache
ms.topic: conceptual
ms.date: 11/19/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 036cab033e5b70a8940a488c07374e9092d52b33
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96533653"
---
# <a name="azure-security-baseline-for-azure-hpc-cache"></a>Base de référence de sécurité Azure pour Azure HPC Cache

Cette base de référence de sécurité applique les conseils du[Benchmark de sécurité Azure version 2.0](../security/benchmarks/overview.md) à Microsoft Azure HPC Cache. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé par les **contrôles de sécurité** définis par le Benchmark de sécurité Azure et les conseils associés applicables à Azure HPC Cache. **Les contrôles** non applicables à Azure HPC Cache ont été exclus.

Pour voir comment Azure HPC Cache est entièrement mappé au Benchmark de sécurité Azure, consultez le [fichier de mappage complet de la base de référence de sécurité Azure HPC Cache](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : sécurité réseau](../security/benchmarks/security-controls-v2-network-security.md).*

### <a name="ns-1-implement-security-for-internal-traffic"></a>NS-1 : Implémenter la sécurité pour le trafic interne

**Conseils** : Lorsque vous déployez des ressources Azure HPC Cache, vous devez créer ou utiliser un réseau virtuel existant. 

Vérifiez que tous les réseaux virtuels Azure suivent un principe de segmentation d’entreprise qui s’aligne sur les risques métier. Tout système qui peut entraîner des risques plus élevés pour l’organisation doit être isolé sur son propre réseau virtuel et être suffisamment sécurisé à l’aide d’un groupe de sécurité réseau (NSG) et/ou du Pare-feu Azure.

Avant de pouvoir utiliser votre cache, vous devez configurer deux prérequis liés au réseau : 

- Un sous-réseau dédié pour l’instance Azure HPC Cache

- La prise en charge du DNS afin que le cache puisse accéder au stockage et aux autres ressources

Azure HPC Cache a besoin d’un sous-réseau dédié avec les qualités suivantes : 

- Le sous-réseau doit disposer d’au moins 64 adresses IP.

- Le sous-réseau ne peut pas héberger d’autres machines virtuelles, même pour les services associés tels que les ordinateurs clients.

- Si vous utilisez plusieurs instances Azure HPC Cache, chacune d’elles doit avoir son propre sous-réseau.

La méthode recommandée consiste à créer un nouveau sous-réseau pour chaque cache. Vous pouvez créer un réseau virtuel et un sous-réseau dans le cadre de la création du cache.

Pour utiliser Azure HPC Cache avec un stockage NAS local, vous devez vous assurer que certains ports du réseau local autorisent un trafic illimité depuis le sous-réseau d’Azure HPC Cache. 

- [Guide pratique pour configurer des ports pour l’accès au stockage NFS](hpc-cache-prerequisites.md#nfs-storage-requirements)

- [Comment créer un groupe de sécurité réseau avec des règles de sécurité](../virtual-network/tutorial-filter-network-traffic.md)

 

- [Guide pratique pour déployer et configurer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="ns-2-connect-private-networks-together"></a>NS-2 : Interconnecter des réseaux privés

**Conseils** : Azure ExpressRoute ou le réseau privé virtuel (VPN) Azure permettent de créer des connexions privées entre les centres de données Azure et l’infrastructure locale dans un environnement de colocation. Les connexions ExpressRoute ne transitent pas par l’Internet public et offrent davantage de fiabilité, des vitesses supérieures et des latences inférieures par rapport aux connexions Internet classiques. Pour un VPN point à site et un VPN site à site, vous pouvez connecter des appareils ou des réseaux locaux à un réseau virtuel à l’aide de n’importe quelle combinaison de ces options VPN et d’Azure ExpressRoute. 

Pour interconnecter deux réseaux virtuels ou plus dans Azure, utilisez le peering de réseaux virtuels. Le trafic réseau entre les réseaux virtuels appairés est privé et conservé sur le réseau principal Azure.

- [Quels sont les modèles de connectivité ExpressRoute ?](../expressroute/expressroute-connectivity-models.md) 

- [Vue d’ensemble des VPN Azure](../vpn-gateway/vpn-gateway-about-vpngateways.md) 

- [Peering de réseau virtuel](../virtual-network/virtual-network-peering-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="ns-3-establish-private-network-access-to-azure-services"></a>NS-3 : Établir un accès réseau privé aux services Azure

**Conseils** : Utilisez des points de terminaison de service de réseau virtuel Azure pour fournir un accès sécurisé à Azure HPC Cache. Les points de terminaison de service constituent un itinéraire optimisé sur le réseau principal Azure sans traverser Internet. 

Azure HPC Cache ne prend pas en charge l’utilisation d’Azure Private Link pour sécuriser ses points de terminaison de gestion sur un réseau privé. 

L’accès privé est une mesure de défense renforcée supplémentaire en plus de la sécurité de l’authentification et du trafic proposée par les services Azure.

- [Présentation des points de terminaison de service de réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md) 

- [Comprendre comment monter Azure HPC Cache](hpc-cache-mount.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="ns-4-protect-applications-and-services-from-external-network-attacks"></a>NS-4 : Protégez les applications et les services contre les attaques réseau externes

**Conseils** : Protégez vos ressources Azure HPC Cache des attaques de réseaux externes, notamment les attaques par déni de service distribué (DDoS), les attaques spécifiques aux applications et le trafic Internet non sollicité et potentiellement malveillant. 

Pour cette protection, Azure intègre des capacités natives : 

- Utilisez le Pare-feu Azure pour protéger les applications et les services contre le trafic potentiellement malveillant provenant d’Internet et d’autres emplacements externes. 
- Protégez vos ressources contre les attaques DDoS en activant la protection DDoS Standard sur vos réseaux virtuels Azure. 
- Utilisez Azure Security Center pour détecter les risques de configuration incorrecte liés à vos ressources réseau.

Azure HPC Cache n’est pas destiné à exécuter des applications web et ne nécessite pas de configuration de paramètres supplémentaires ni de déploiement de services réseau supplémentaires pour le protéger des attaques de réseau externe qui ciblent les applications web.

- [Documentation sur le pare-feu Azure](../firewall/index.yml) 

- [Gérer la norme Azure DDoS Protection à l’aide du portail Azure](../ddos-protection/manage-ddos-protection.md) 

- [Recommandations relatives à Azure Security Center](../security-center/recommendations-reference.md#recs-network)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="ns-5-deploy-intrusion-detectionintrusion-prevention-systems-idsips"></a>NS-5 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS)

**Conseils** : Utilisez Pare-feu Azure avec un filtrage basé sur le renseignement sur les menaces pour générer des alertes ou bloquer le trafic en provenance et à destination d’adresses IP et de domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence. 

Quand l’inspection de la charge utile est obligatoire, vous pouvez déployer une solution tierce de détection et de prévention d’intrusion (IDS/IPS) à partir de Place de marché Azure avec des capacités d’inspection de charge utile. Vous pouvez également utiliser un système IDS/IPS basé sur un hôte ou une solution de détection de point de terminaison et réponse (EDR) basée sur un hôte avec ou au lieu d’une solution IDS/IPS réseau.

Remarque : Si l’utilisation des systèmes IDS/IPS s’accompagne d’une exigence réglementaire ou autre, vérifiez qu’elle est toujours paramétrée de façon à fournir des alertes de haute qualité à votre solution SIEM.

- [Guide pratique pour déployer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md) 

- [Capacités IDS de tiers sur Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace?search=IDS) 

- [Fonctionnalité de détection de point de terminaison et réponse (EDR) de Microsoft Defender ATP](/windows/security/threat-protection/microsoft-defender-atp/overview-endpoint-detection-response)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="ns-6-simplify-network-security-rules"></a>NS-6 : Simplifier les règles de sécurité réseau

**Conseils** : Non applicable ; cette recommandation est destinée à des offres qui peuvent être déployées dans des réseaux virtuels Azure ou qui ont la capacité de définir des regroupements de plages d’adresses IP autorisées pour une gestion efficace. Azure HPC Cache ne prend pas actuellement en charge les étiquettes de service. 

La méthode recommandée consiste à créer un nouveau sous-réseau pour chaque cache. Vous pouvez créer un réseau virtuel et un sous-réseau dans le cadre de la création du cache.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="ns-7-secure-domain-name-service-dns"></a>NS-7 : Système DNS (Domain Name System) sécurisé

**Conseils** : Suivez les meilleures pratiques concernant la sécurité DNS afin d’atténuer les attaques courantes comme les entrées DNS non résolues, les attaques d’amplifications DNS, l’empoisonnement et l’usurpation d’identité DNS, entre autres.

Azure HPC Cache a besoin d’un accès DNS pour accéder aux ressources situées en dehors du réseau virtuel privé du cache. Si votre workflow comprend des ressources externes à Azure, vous devez configurer et sécuriser votre propre serveur DNS en plus d’utiliser Azure DNS.

- Pour accéder aux points de terminaison Stockage Blob Azure, aux ordinateurs client basés sur Azure ou à d’autres ressources Azure, utilisez Azure DNS.
- Pour accéder au stockage local ou pour vous connecter au cache à partir de clients externes à Azure, vous devez créer un serveur DNS personnalisé qui peut résoudre ces noms d’hôte.
- Si votre workflow comprend des ressources internes et externes, configurez votre serveur DNS personnalisé pour qu’il transfère les demandes de résolution spécifiques à Azure vers le serveur Azure DNS. 

Quand Azure DNS est utilisé comme votre service DNS faisant autorité, vérifiez que les zones et les enregistrements DNS sont protégés contre toute modification accidentelle ou malveillante en utilisant Azure RBAC et des verrous de ressources.

Si vous configurez votre propre serveur DNS, veillez à suivre les instructions de sécurité suivantes :

- [Guide de déploiement d’un système DNS (Domain Name System) sécurisé](https://csrc.nist.gov/publications/detail/sp/800-81/2/final) 

- [Prévention des entrées DNS non résolues et de l’acquisition de sous-domaine](../security/fundamentals/subdomain-takeover.md) 

- [En savoir plus sur les exigences d’accès DNS pour Azure HPC Cache](hpc-cache-prerequisites.md#dns-access)

- [Vue d’ensemble d’Azure DNS](../dns/dns-overview.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="identity-management"></a>Gestion des identités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des identités](../security/benchmarks/security-controls-v2-identity-management.md).*

### <a name="im-1-standardize-azure-active-directory-as-the-central-identity-and-authentication-system"></a>IM-1 : Normaliser Azure Active Directory comme système d’authentification et d’identité central

**Conseils** : Azure HPC Cache n’est pas intégré à Azure Active Directory pour les opérations internes. Toutefois, Azure AD peut être utilisé pour authentifier les utilisateurs dans le portail Azure ou l’interface CLI afin de créer, d’afficher et de gérer les déploiements d’Azure HPC Cache et les composants connexes.

Azure Active Directory (Azure AD) est le service par défaut de gestion des identités et des accès dans Azure. Vous devez normaliser Azure AD pour régir la gestion des identités et des accès de votre organisation dans :

- Les ressources cloud Microsoft, comme le portail Azure, le stockage Azure, les machines virtuelles Azure (Linux et Windows), les applications Azure Key Vault, PaaS et SaaS.

- Les ressources de votre organisation, comme les applications sur Azure ou les ressources réseau de votre entreprise.

La sécurisation d’Azure AD doit être d’une priorité élevée dans les pratiques de sécurité cloud de votre organisation. Azure AD fournit un score d'identité sécurisée pour vous aider à évaluer la sécurité des identités par rapport aux recommandations de Microsoft en matière de meilleures pratiques. Utilisez le score pour évaluer avec précision votre configuration par rapport aux meilleures pratiques recommandées et apporter des améliorations à votre posture de sécurité.

Remarque : Azure AD prend en charge l’identité externe, ce qui permet aux utilisateurs sans compte Microsoft de se connecter à leurs applications et ressources avec leur identité externe.

- [Locataires dans Azure Active Directory](../active-directory/develop/single-and-multi-tenant-apps.md) 

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md) 

- [Utiliser des fournisseurs d’identité externes pour une application](../active-directory/external-identities/identity-providers.md) 

- [Qu’est-ce que le degré de sécurisation Identity Secure Score dans Azure Active Directory](../active-directory/fundamentals/identity-secure-score.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-2-manage-application-identities-securely-and-automatically"></a>IM-2 : Gérer les identités d’application de façon sécurisée et automatique

**Conseils** : Azure HPC Cache utilise des identités managées Azure pour les comptes non humains tels que les services ou l’automatisation. Il est recommandé de recourir à la fonctionnalité d’identité managée Azure plutôt que de créer un compte humain plus puissant pour accéder aux ressources ou les exécuter. 

Azure HPC Cache peut s’authentifier en mode natif auprès des services/ressources Azure qui prennent en charge l’authentification Azure AD par le biais de règles prédéfinies d’octroi d’accès. Cela évite d’avoir à utiliser des informations d’identification codées en dur dans le code source ou les fichiers config.

- [Identités managées Azure](../active-directory/managed-identities-azure-resources/overview.md) 

- [Services prenant en charge les identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-3-use-azure-ad-single-sign-on-sso-for-application-access"></a>IM-3 : Utiliser l’authentification unique Azure AD pour l’accès aux applications

**Conseils** : Azure HPC Cache ne s’intègre pas à Azure AD pour les opérations internes. Toutefois, Azure AD peut être utilisé pour authentifier les utilisateurs dans le portail Azure ou l’interface CLI afin de créer, d’afficher et de gérer les déploiements d’Azure HPC Cache et les composants connexes.

Azure Active Directory fournit la gestion des identités et des accès aux ressources Azure, aux applications cloud et aux applications locales. Cela inclut les identités d’entreprise, comme les employés, ainsi que les identités externes, comme les partenaires et les fournisseurs. Cela permet à l’authentification unique de gérer et sécuriser l’accès aux données et ressources de votre organisation localement et dans le cloud. Connectez l’ensemble de vos utilisateurs, applications et appareils à Azure AD pour un accès transparent et sécurisé, ainsi qu’une visibilité et un contrôle accrus.

- [Comprendre l’authentification unique des applications avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-4-use-strong-authentication-controls-for-all-azure-active-directory-based-access"></a>IM-4 : Utiliser des contrôles d’authentification renforcés pour tous les accès basés sur Azure Active Directory

**Conseils** : Bien qu’Azure HPC Cache ne s’intègre pas à Azure AD pour les opérations internes, Azure AD peut être utilisé pour authentifier les utilisateurs dans le portail Azure ou l’interface CLI afin de créer, d’afficher et de gérer les déploiements d’Azure HPC Cache et les composants connexes.  

Azure AD prend en charge des contrôles d’authentification renforcés via l’authentification multifacteur (MFA) et des méthodes fortes sans mot de passe.

- Authentification multifacteur : Activez Azure AD MFA et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center pour appliquer quelques-unes des meilleures pratiques dans votre configuration MFA. L’authentification multifacteur peut être appliquée à tous les utilisateurs, seulement certains utilisateurs ou au niveau de chaque utilisateur en fonction des conditions de connexion et des facteurs de risque.
- Authentification sans mot de passe – Trois options d’authentification sans mot de passe sont disponibles : Windows Hello Entreprise, l’application Microsoft Authenticator et les méthodes d’authentification locales, comme les cartes à puce.

Pour les administrateurs et les utilisateurs privilégiés, veillez à utiliser le niveau le plus élevé de la méthode d’authentification forte. Déployez la stratégie d’authentification forte appropriée auprès des autres utilisateurs.

Azure HPC Cache prend également en charge l’authentification par mot de passe héritée pour les systèmes clients qui se connectent au cache. Ces types de connexions comprennent les comptes cloud uniquement (comptes d’utilisateur créés directement dans Azure) qui ont une stratégie de mot de passe de base ou les comptes hybrides (comptes d’utilisateur provenant de Windows Server Active Directory) qui suivront les stratégies de mot de passe locales. 

Lors de l’utilisation de l’authentification par mot de passe, Azure AD fournit une fonctionnalité de protection par mot de passe qui empêche les utilisateurs de définir des mots de passe faciles à deviner. Microsoft fournit une liste globale de mots de passe interdits mise à jour en fonction de la télémétrie, et les clients peuvent compléter la liste en fonction de leurs besoins (par exemple avec des noms de marque, des références culturelles, etc.). Cette protection par mot de passe peut être utilisée pour les comptes cloud et hybrides uniquement.

Remarque : La seule authentification basée sur les informations de mot de passe est vulnérable aux méthodes d’attaque populaires. Pour une sécurité accrue, utilisez une authentification forte telle que l’authentification multifacteur et une stratégie de mot de passe forte. Si vous utilisez des applications tierces et des services de marketplace qui ont des mots de passe par défaut, définissez un nouveau mot de passe sécurisé la première fois que vous configurez le service. 

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md) 

- [Introduction aux options d’authentification sans mot de passe pour Azure Active Directory](../active-directory/authentication/concept-authentication-passwordless.md)

- [Stratégie de mot de passe par défaut d’Azure AD](../active-directory/authentication/concept-sspr-policy.md#password-policies-that-only-apply-to-cloud-user-accounts)

- [Éliminer les mauvais mots de passe à l’aide de Protection de mots de passe d’Azure AD](../active-directory/authentication/concept-password-ban-bad.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="im-5-monitor-and-alert-on-account-anomalies"></a>IM-5 : Surveiller et alerter en cas d’anomalies de compte

**Conseils** : Azure HPC Cache peut utiliser Azure Active Directory pour la gestion des utilisateurs à partir du portail Azure.  Azure Active Directory fournit les sources de données suivantes :

- Connexions – Le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.

- Journaux d’audit : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Par exemple, les journaux d’audit peuvent inclure les modifications apportées à des ressources dans Azure AD comme l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles, de stratégies, etc.

- Connexions risquées : une connexion risquée est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.

- Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Ces sources de données peuvent être intégrées à Azure Monitor, à Azure Sentinel ou à des systèmes SIEM tiers.

Azure Security Center peut également alerter en cas de certaines activités suspectes, comme un nombre excessif de tentatives d’authentification ayant échoué et la présence de comptes dépréciés dans l’abonnement.

Azure Advanced Threat Protection (ATP) est une solution de sécurité qui peut utiliser les signaux Active Directory pour identifier, détecter et examiner les menaces avancées, les identités compromises et les actions malveillantes internes.

- [Audit des rapports d’activité dans Azure Active Directory](../active-directory/reports-monitoring/concept-audit-logs.md)

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md) 

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md) 

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)

 

- [Alertes dans le module de protection de renseignement sur les menaces d’Azure Security Center](../security-center/alerts-reference.md) 

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="im-7-eliminate-unintended-credential-exposure"></a>IM-7 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Non applicable ; Azure HPC Cache ne permet pas aux clients de déployer des données persistantes dans l’environnement en cours d’exécution.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="privileged-access"></a>Accès privilégié

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Accès privilégié](../security/benchmarks/security-controls-v2-privileged-access.md).*

### <a name="pa-2-restrict-administrative-access-to-business-critical-systems"></a>PA-2 : Limiter l’accès administratif aux systèmes critiques de l’entreprise

**Conseils** : Azure HPC Cache utilise Azure RBAC pour isoler l’accès aux systèmes vitaux pour l’entreprise en limitant les comptes qui bénéficient d’un accès privilégié aux abonnements et aux groupes d’administration auxquels ils appartiennent.

Créez des procédures standard autour de l’utilisation de comptes d’administration dédiés. Pour créer un cache, Azure HPC Cache exige que les utilisateurs disposent des privilèges suffisants dans l’abonnement pour créer des cartes réseau. Si vous utilisez Stockage Blob, les rôles Azure Contributeur au compte de stockage et Contributeur aux données Blob du stockage sont requis pour qu’Azure HPC Cache accède au stockage. 

Veillez également à limiter l’accès aux systèmes de gestion, d’identité et de sécurité qui ont un accès administratif à vos ressources critiques, comme les contrôleurs de domaine (DC) Active Directory, les outils de sécurité et les outils de gestion du système avec les agents installés sur les systèmes critiques de l’entreprise. Les attaquants qui compromettent ces systèmes de gestion et de sécurité peuvent immédiatement les armer pour compromettre les ressources stratégiques de l’entreprise.

Pour assurer un contrôle d’accès cohérent, tous les types de contrôle d’accès doivent être alignés sur la stratégie de segmentation de votre entreprise.

- [Autorisations RBAC d’Azure HPC Cache](hpc-cache-prerequisites.md#permissions)

- [Composants Azure et modèle de référence](/security/compass/microsoft-security-compass-introduction#azure-components-and-reference-model-2151) 

- [Accès aux groupes d’administration](../governance/management-groups/overview.md#management-group-access)

- [Administrateurs d’abonnements Azure](../cost-management-billing/manage/add-change-subscription-administrator.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="pa-3-review-and-reconcile-user-access-regularly"></a>PA-3 : Examiner et rapprocher régulièrement les accès utilisateur

**Conseils** : Examinez régulièrement les comptes d’utilisateur et l’attribution des accès pour veiller à ce que les comptes et leur niveau d’accès soient valides. 

Azure HPC Cache peut utiliser des comptes Azure Active Directory (Azure AD) pour gérer l’accès utilisateur via le portail Azure et les interfaces connexes. Azure AD propose des révisions d’accès qui vous aident à réviser les appartenances aux groupes, l’accès aux applications d’entreprise et les attributions de rôles. Les rapports Azure AD peuvent fournir des journaux pour vous aider à découvrir les comptes obsolètes. Vous pouvez également utiliser Azure AD Privileged Identity Management pour créer un workflow de rapport de révision d’accès afin de faciliter le processus de révision.

En outre, Azure Privileged Identity Management peut être configuré pour alerter en cas de création d’un nombre excessif de comptes d’administrateur et pour identifier les comptes d’administrateur obsolètes ou mal configurés.

Remarque : Certains services Azure prennent en charge des utilisateurs et des rôles locaux qui ne sont pas gérés par le biais d’Azure AD. Vous devrez gérer ces utilisateurs séparément.

Lorsque vous utilisez des cibles de stockage NFS, vous devez collaborer avec les administrateurs réseau et les gestionnaires de pare-feu pour vérifier les paramètres d’accès et vous assurer qu’Azure HPC cache pourra communiquer avec les systèmes de stockage NFS.

- [Pour obtenir la liste des autorisations Azure HPC Cache, consultez la configuration requise pour Azure HPC Cache.](hpc-cache-prerequisites.md) 

- [Créer une révision d’accès des rôles de ressources Azure dans Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-resource-roles-start-access-review.md) 

- [Utilisation des révisions d’accès et des identités Azure AD](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="pa-4-set-up-emergency-access-in-azure-ad"></a>PA-4 : Configurer l’accès d’urgence dans Azure AD

**Conseils** : Azure HPC Cache peut utiliser Azure Active Directory pour gérer l’accès aux ressources via le portail Azure. Pour empêcher le verrouillage accidentel de votre organisation Azure AD, configurez un compte d’accès d’urgence pour conserver un accès lorsque les comptes administratifs normaux ne peuvent pas être utilisés. Les comptes d’accès d’urgence sont généralement hautement privilégiés et ne doivent pas être attribués à des personnes spécifiques. Les comptes d’accès d’urgence sont limités à des cas d’urgence ou à des scénarios « de secours » où il est impossible d’utiliser des comptes d’administration normaux.

Vous devez vous assurer que les informations d’identification (telles que le mot de passe, le certificat ou la carte à puce) des comptes d’accès d’urgence restent sécurisées et connues des seules personnes autorisées à les utiliser en cas d’urgence.

- [Gérer des comptes d’accès d’urgence dans Azure AD](../active-directory/roles/security-emergency-access.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="pa-5-automate-entitlement-management"></a>PA-5 : Automatiser la gestion des droits d'utilisation 

**Conseils** : Azure HPC Cache peut utiliser Azure Active Directory pour gérer l’accès aux ressources du cache via le portail Azure. 

Utilisez les fonctionnalités de gestion des droits d’utilisation d’Azure AD pour automatiser les workflows de demandes d’accès, notamment les attributions, les révisions et l’expiration des accès. L’approbation en deux ou plusieurs étapes est également prise en charge. 

- [Présentation des révisions d’accès Azure AD](../active-directory/governance/access-reviews-overview.md) 

- [Présentation de la gestion des droits d’utilisation Azure AD](../active-directory/governance/entitlement-management-overview.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="pa-7-follow-just-enough-administration-least-privilege-principle"></a>PA-7 : Suivre le principe Just Enough Administration (privilèges minimum) 

**Conseils** : Azure HPC Cache est intégré au contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour gérer ses ressources. Azure RBAC vous permet de gérer l’accès aux ressources Azure via des attributions de rôles. Vous pouvez affecter ces rôles à des utilisateurs, regrouper des principaux de service et des identités managées. Les rôles intégrés prédéfinis pour certaines ressources peuvent être inventoriés ou interrogés par le biais d’outils tels que Azure CLI, Azure PowerShell ou le Portail Azure. 

Les privilèges que vous attribués aux ressources via Azure RBAC doivent toujours être limités à ce qui est requis par les rôles. Ils complètent l’approche juste-à-temps (JAT) d’Azure AD Privileged Identity Management (PIM) et doivent être révisés régulièrement.

Utilisez des rôles intégrés pour allouer les autorisations et créez un rôle personnalisé uniquement lorsque cela est nécessaire.

- [Présentation du contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) 

- [Configurer le contrôle d'accès en fonction du rôle dans Azure](../role-based-access-control/role-assignments-portal.md) 

- [Utilisation des révisions d’accès et des identités Azure AD](../active-directory/governance/access-reviews-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : protection des données](../security/benchmarks/security-controls-v2-data-protection.md).*

### <a name="dp-1-discover-classify-and-label-sensitive-data"></a>DP-1 : Découvrir, classer et étiqueter des données sensibles 

**Conseils** : Azure HPC Cache gère les données sensibles, mais n’a pas la capacité de les détecter, de les classer et de les étiqueter.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="dp-2-protect-sensitive-data"></a>DP-2 : Protection des données sensibles

**Conseils** : Protégez les données sensibles en limitant l’accès avec le contrôle d’accès en fonction du rôle Azure (Azure RBAC), des contrôles d’accès basés sur le réseau et des contrôles spécifiques dans les services Azure (comme le chiffrement dans les bases de données SQL et autres).

Pour assurer un contrôle d’accès cohérent, tous les types de contrôle d’accès doivent être alignés sur la stratégie de segmentation de votre entreprise. La stratégie de segmentation de l’entreprise doit aussi être informée de l’emplacement des systèmes et données sensibles ou vitaux de l’entreprise.

Pour la plateforme sous-jacente gérée par Microsoft, Microsoft traite tout le contenu client en tant que contenu sensible, et assure une protection contre la perte et l’exposition des données client. Pour assurer la sécurité des données des clients dans Azure, Microsoft a implémenté certains contrôles et fonctionnalités de protection des données par défaut.

- [Contrôle d’accès en fonction du rôle (RBAC) Azure](../role-based-access-control/overview.md) 

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="dp-3-monitor-for-unauthorized-transfer-of-sensitive-data"></a>DP-3 : Détection des transferts non autorisés de données sensibles

**Conseils** : Azure HPC Cache transmet des données sensibles, mais ne permet pas de surveiller le transfert non autorisé de données sensibles.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="dp-4-encrypt-sensitive-information-in-transit"></a>DP-4 : Chiffrement des informations sensibles en transit

**Conseils** : Azure HPC Cache prend en charge le chiffrement des données en transit avec TLS 1.2 ou version ultérieure.

C’est certes facultatif pour le trafic sur les réseaux privés, mais essentiel pour le trafic sur les réseaux externes et publics. Pour le trafic HTTP, vérifiez que les clients se connectant à vos ressources Azure peuvent négocier TLS v1.2 ou une version ultérieure. Pour la gestion à distance, utilisez SSH (pour Linux) ou RDP/TLS (pour Windows) plutôt qu’un protocole non chiffré. Les versions et protocoles SSL, TLS et SSH rendus obsolètes et les chiffrements faibles doivent être désactivés.

Par défaut, Azure assure le chiffrement des données en transit entre les centres de données Azure.

- [Présentation du chiffrement en transit avec Azure](../security/fundamentals/encryption-overview.md#encryption-of-data-in-transit) 

- [Informations sur la sécurité TLS](/security/engineering/solving-tls1-problem) 

- [Double chiffrement pour les données Azure en transit](../security/fundamentals/double-encryption.md#data-in-transit)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="dp-5-encrypt-sensitive-data-at-rest"></a>DP-5 : Chiffrement des données sensibles au repos

**Conseils** : En complément des contrôles d’accès, les données au repos doivent être protégées contre les attaques « hors bande » (par exemple, l’accès au stockage sous-jacent) à l’aide du chiffrement. Cela vise à empêcher que les attaquants puissent facilement lire ou modifier les données.

Azure assure par défaut un chiffrement des données au repos. Pour les données hautement sensibles, vous avez le choix entre plusieurs options pour implémenter un chiffrement supplémentaire au repos sur toutes les ressources Azure, le cas échéant. Azure gère vos clés de chiffrement par défaut, mais met à votre disposition des options pour gérer vos propres clés (clés gérées par le client) pour certains services Azure.

Toutes les données stockées dans Azure, y compris sur les caches de disque, sont chiffrées au repos à l’aide de clés gérées par Microsoft par défaut. Vous devez seulement personnaliser les paramètres d’Azure HPC Cache si vous souhaitez gérer les clés utilisées pour chiffrer vos données.

Si nécessaire à des fins de conformité sur les ressources de calcul, implémentez un outil tiers (par exemple, une solution automatisée de protection contre la perte de données basée sur un hôte) pour appliquer des contrôles d’accès aux données même lorsque celles-ci sont copiées à partir d’un système.

- [Comment utiliser des clés de chiffrement gérées par le client avec Azure HPC Cache](./hpc-cache-create.md?tabs=azure-portal#enable-azure-key-vault-encryption-optional)

- [Présentation du chiffrement au repos dans Azure](../security/fundamentals/encryption-atrest.md#encryption-at-rest-in-microsoft-cloud-services) 

- [Guide pratique pour configurer des clés de chiffrement gérées par le client](../storage/common/customer-managed-keys-configure-key-vault.md) 

- [Modèle de chiffrement et table de gestion des clés](../security/fundamentals/encryption-atrest.md)

 

- [Double chiffrement des données au repos dans Azure](../security/fundamentals/double-encryption.md#data-at-rest)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="asset-management"></a>Gestion des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des ressources](../security/benchmarks/security-controls-v2-asset-management.md).*

### <a name="am-1-ensure-security-team-has-visibility-into-risks-for-assets"></a>AM-1 : S’assurer que l’équipe de sécurité a une visibilité sur les risques pour les ressources

**Conseils** : Assurez-vous que les équipes de sécurité reçoivent des autorisations de lecteur de sécurité dans votre locataire et vos abonnements Azure afin qu’elles puissent surveiller les risques de sécurité à l’aide d’Azure Security Center. 

Selon la façon dont les responsabilités de l’équipe de sécurité sont structurées, la surveillance des risques de sécurité peut tenir de la responsabilité d’une équipe de sécurité centrale ou d’une équipe locale. Cela dit, les insights et les risques liés à la sécurité doivent toujours être agrégés de manière centralisée au sein d’une organisation. 

Les autorisations de lecteur de sécurité peuvent être appliquées globalement à un locataire entier (groupe d’administration racine) ou étendues à des groupes d’administration ou à des abonnements spécifiques. 

Remarque : Des autorisations supplémentaires peuvent être nécessaires pour obtenir une visibilité sur les charges de travail et services. 

- [Vue d’ensemble du rôle lecteur de sécurité](../role-based-access-control/built-in-roles.md#security-reader)

- [Vue d'ensemble des groupes d'administration Azure](../governance/management-groups/overview.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="am-2-ensure-security-team-has-access-to-asset-inventory-and-metadata"></a>AM-2 : S’assurer que l’équipe de sécurité a accès à l’inventaire des ressources et aux métadonnées

**Conseils** : Azure HPC Cache prend en charge l’utilisation d’étiquettes. Appliquez des étiquettes à vos ressources Azure, groupes de ressources et abonnements pour les organiser de façon logique dans une taxonomie. Chaque balise se compose d’une paire nom-valeur. 

Par exemple, vous pouvez appliquer le nom « Environnement » et la valeur « Production » à toutes les ressources en production. Vous pouvez ajouter des étiquettes lors de la création d’un cache, ainsi qu’après le déploiement du cache. 

Utilisez l’inventaire des machines virtuelles Azure pour automatiser la collecte d’informations relatives aux logiciels présents sur les machines virtuelles. Le nom, la version, l’éditeur et l’heure d’actualisation du logiciel sont disponibles sur le portail Azure. Pour avoir accès à la date d’installation et à d’autres informations, activez les diagnostics au niveau de l’invité et placez les journaux des événements Windows dans un espace de travail Log Analytics.
Azure HPC Cache ne permet pas d’exécuter une application ni d’installer un logiciel sur ses ressources. 

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md) 

- [Gestion de l’inventaire de sécurité dans Azure Security Center](../security-center/asset-inventory.md) 

- [Guides de décision concernant le nommage et l’étiquetage des ressources](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=%2fazure%2fazure-resource-manager%2fmanagement%2ftoc.json) 

- [Guide pratique pour activer l’inventaire des machines virtuelles Azure](../automation/automation-tutorial-installed-software.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="am-3-use-only-approved-azure-services"></a>AM-3 : Utiliser des services Azure approuvés uniquement

**Conseils** : Azure HPC Cache prend en charge les déploiements d’Azure Resource Manager. Utilisez Azure Policy pour auditer et limiter les services que les utilisateurs peuvent approvisionner dans votre environnement. Utilisez Azure Resource Graph pour interroger et découvrir des ressources dans leurs abonnements. Vous pouvez également utiliser Azure Monitor pour créer des règles afin de déclencher des alertes lorsqu’un service non approuvé est détecté.

- [Configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md) 

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/index.md) 

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="am-4-ensure-security-of-asset-lifecycle-management"></a>AM-4 : Garantir la sécurité de la gestion du cycle de vie des actifs

**Aide** : Non applicable. Azure HPC Cache ne peut pas être utilisé pour garantir la sécurité des ressources dans un processus de gestion du cycle de vie. Il incombe au client de gérer les attributs et les configurations réseau des ressources considérées comme ayant un impact élevé. 

Il est recommandé que le client crée un processus permettant de capturer les modifications des attributs et des configurations réseau, mesurer l’impact des modifications et créer des tâches de correction, le cas échéant.

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

## <a name="logging-and-threat-detection"></a>Journalisation et détection des menaces

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Journalisation et détection des menaces](/azure/security/benchmarks/security-controls-v2-logging-threat-detection).*

### <a name="lt-1-enable-threat-detection-for-azure-resources"></a>LT-1 : Activer la détection des menaces pour les ressources Azure

**Conseils** : Utilisez la capacité de détection des menaces intégrée à Azure Security Center et activez Azure Defender (officiellement Azure Advanced Threat Protection) pour vos ressources Azure HPC Cache. Azure Defender pour Azure HPC Cache offre une couche supplémentaire de sécurité intelligente qui détecte les tentatives d’accès ou d’exploitation inhabituelles et potentiellement dangereuses de vos ressources de cache.

Transférez tous les journaux d’Azure HPC Cache à votre solution SIEM, qui peut être utilisée pour configurer des détections de menaces personnalisées. Veillez à surveiller les différents types de ressources Azure pour identifier les menaces et anomalies potentielles. Concentrez-vous sur l’obtention d’alertes de haute qualité afin de réduire les faux positifs que les analystes doivent trier. Les alertes peuvent provenir de données de journal, d’agents ou d’autres données.

- [Protection contre les menaces dans Azure Security Center](../security-center/azure-defender.md) 

- [Guide de référence des alertes de sécurité Azure Security Center](../security-center/alerts-reference.md) 

- [Créer des règles d’analytique personnalisées pour détecter des menaces](../sentinel/tutorial-detect-threats-custom.md) 

- [Renseignement sur les menaces informatiques dans Azure Sentinel](/azure/architecture/example-scenario/data/sentinel-threat-intelligence)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="lt-2-enable-threat-detection-for-azure-identity-and-access-management"></a>LT-2 : Activer la détection des menaces pour la gestion des identités et des accès Azure

**Conseils** : Azure AD fournit les journaux d’utilisateur suivants qui peuvent être consultés dans les rapports Azure AD ou intégrés à Azure Monitor, Azure Sentinel ou d’autres outils de surveillance/SIEM pour des cas d’usage plus sophistiqués de surveillance et d’analytique :
- Connexions – le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.

- Journaux d’audit : fournissent une traçabilité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Par exemple, les journaux d’audit peuvent inclure les modifications apportées à des ressources dans Azure AD, comme l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles, de stratégies, etc.

- Connexions risquées : une connexion risquée est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.

- Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Azure Security Center peut également alerter dans le cas de certaines activités suspectes, comme un nombre excessif de tentatives d’authentification ayant échoué ou la présence de comptes dépréciés dans l’abonnement. En plus de la surveillance de base de l’hygiène de sécurité, le module Protection contre les menaces d’Azure Security Center peut également collecter des alertes de sécurité plus approfondies à partir de ressources de calcul Azure individuelles (machines virtuelles, conteneurs ou service d’application), de ressources de données (base de données SQL et stockage) et de couches de service Azure. Cette capacité vous permet d’avoir une visibilité sur les anomalies de compte à l’intérieur des ressources individuelles.

- [Audit des rapports d’activité dans Azure Active Directory](../active-directory/reports-monitoring/concept-audit-logs.md) 

- [Activer Azure AD Identity Protection](../active-directory/identity-protection/overview-identity-protection.md) 

- [Protection contre les menaces dans Azure Security Center](../security-center/azure-defender.md)

**Supervision Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="lt-3-enable-logging-for-azure-network-activities"></a>LT-3 : Activer la journalisation pour les activités réseau Azure

**Conseils** : Vous pouvez utiliser les passerelles VPN et leur capacité de capture de paquets, en plus des outils de capture de paquets généralement disponibles, pour enregistrer les paquets de réseau circulant entre vos réseaux virtuels.

Déployez un groupe de sécurité réseau sur le réseau sur lequel vos ressources Azure HPC Cache sont déployées. Activez les journaux de flux du groupe de sécurité réseau sur vos groupes de sécurité réseau pour l’audit du trafic.

Vos journaux de flux sont conservés dans un compte de stockage. Activez la solution Traffic Analytics pour traiter et envoyer ces journaux de flux à un espace de travail Log Analytics. Traffic Analytics fournit des informations supplémentaires sur le flux de trafic pour vos réseaux Azure. Traffic Analytics peut vous aider à visualiser l’activité réseau et à identifier les zones réactives, à identifier les menaces de sécurité, à comprendre les modèles de flux de trafic et à repérer les mauvaises configurations du réseau.

Le cache a besoin d’un accès DNS pour accéder aux ressources situées en dehors de son réseau virtuel. Selon les ressources que vous utilisez, vous devrez peut-être configurer un serveur DNS personnalisé et configurer un transfert entre ce serveur et les serveurs Azure DNS. 

Implémentez une solution tierce de journalisation DNS à partir de la Place de marché Azure en fonction des besoins de votre organisation.

- [Configurer les captures de paquets pour les passerelles VPN](../vpn-gateway/packet-capture.md) 

- [Tutoriel : journaliser le trafic réseau à destination et en provenance d’une machine virtuelle à l’aide du portail Azure](../network-watcher/network-watcher-nsg-flow-logging-portal.md) 

- [Traffic Analytics](../network-watcher/traffic-analytics.md) 

- [Protéger vos ressources réseau](../security-center/security-center-network-recommendations.md) 

- [En savoir plus sur les prérequis pour DNS](hpc-cache-prerequisites.md).

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="lt-4-enable-logging-for-azure-resources"></a>LT-4 : Activer la journalisation pour les ressources Azure

**Conseils** : Les ressources Azure HPC Cache créent automatiquement des journaux d’activité. Ces journaux contiennent toutes les opérations d’écriture (PUT, POST, DELETE), mais n’incluent pas les opérations de lecture (GET). Les journaux d’activité peuvent être utilisés pour rechercher une erreur lors de la résolution de problèmes ou pour surveiller la manière dont un utilisateur de votre organisation a modifié une ressource.

Vous pouvez également utiliser Azure Security Center et Azure Policy pour activer les journaux de ressources Azure pour HPC Cache et pour collecter les données de journaux. Ces journaux peuvent être essentiels pour l’examen ultérieur des incidents de sécurité et l’exécution d’exercices légaux.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

- [Présentation de la journalisation et des différents types de journaux dans Azure](../azure-monitor/platform/platform-logs-overview.md) 

- [Présentation de la collecte de données Azure Security Center](../security-center/security-center-enable-data-collection.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="lt-5-centralize-security-log-management-and-analysis"></a>LT-5 : Centraliser la gestion et l’analyse des journaux de sécurité

**Conseils** : Centralisez le stockage et l’analyse de la journalisation pour activer la corrélation. Pour chaque source de journal, assurez-vous d’avoir attribué un propriétaire de données, des conseils d’accès, un emplacement de stockage, les outils utilisés pour traiter les données et y accéder, ainsi que les exigences en matière de conservation des données.

Veillez à intégrer les journaux d’activité Azure dans votre journalisation centralisée. Ingérez des journaux par le biais d’Azure Monitor pour agréger les données de sécurité générées par les appareils de point de terminaison, les ressources réseau et d’autres systèmes de sécurité. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et utilisez des comptes Stockage Azure pour le stockage à long terme et l’archivage.

En outre, activez et intégrez les données dans Azure Sentinel ou une solution SIEM tierce.

De nombreuses organisations choisissent d’utiliser Azure Sentinel pour les données « chaudes » qui sont utilisées fréquemment et Stockage Azure pour les données « froides » qui sont utilisées moins fréquemment.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-controls-v2-incident-response.md).*

### <a name="ir-1-preparation--update-incident-response-process-for-azure"></a>IR-1 : Préparation – mettre à jour le processus de réponse aux incidents pour Azure

**Conseils** : Assurez-vous que votre organisation dispose de processus pour répondre aux incidents de sécurité, qu’elle a mis à jour ces processus pour Azure et qu’elle les exerce régulièrement pour garantir la préparation.

- [Implémenter la sécurité dans l’environnement de l’entreprise](/azure/cloud-adoption-framework/security/security-top-10#4-process-update-incident-response-ir-processes-for-cloud)

- [Guide de référence sur les réponses aux incidents](/microsoft-365/downloads/IR-Reference-Guide.pdf)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="ir-2-preparation--set-up-incident-notification"></a>IR-2 : Préparation – configurer la notification d’incident 

**Conseils** : Configurez les coordonnées des personnes à contacter en cas d’incident de sécurité dans Azure Security Center. Microsoft utilisera ces coordonnées afin de vous contacter si le Microsoft Security Response Center (MSRC) découvre que vos données ont été consultées de manière illégale ou par un tiers non autorisé. Vous avez également la possibilité de personnaliser les alertes et les notifications d’incidents dans différents services Azure en fonction de vos besoins en matière de réponse aux incidents. 

- [Comment définir le contact de sécurité d’Azure Security Center](../security-center/security-center-provide-security-contact-details.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="ir-3-detection-and-analysis--create-incidents-based-on-high-quality-alerts"></a>IR-3 : Détection et analyse – créer des incidents en fonction d’alertes de haute qualité

**Instructions** : Veillez à avoir un processus de création d’alertes de bonne qualité et de mesure de la qualité des alertes. Cela vous permet de tirer les leçons des incidents passés et de classer par ordre de priorité les alertes pour les analystes, afin qu’ils ne perdent pas de temps sur les faux positifs. 

Vous pouvez créer des alertes de bonne qualité en vous basant sur l’expérience des incidents passés, sur les sources validées par la communauté, et sur des outils conçus pour générer et nettoyer les alertes en fusionnant et en mettant en corrélation différentes sources de signaux. 

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

Azure Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance que Security Center accorde à la recherche ou à l’analytique utilisées pour émettre l’alerte, ainsi qu’à l’intention malveillante estimée de l’activité à l’origine du déclenchement de l’alerte.

En outre, marquez les ressources à l’aide d’étiquettes et créez un système de nommage pour identifier et classer les ressources Azure, en particulier celles qui traitent des données sensibles.  Il vous incombe de hiérarchiser le traitement des alertes en fonction de la criticité des ressources et de l’environnement Azure où l’incident s’est produit.

- [Alertes de sécurité dans le Centre de sécurité Azure](../security-center/security-center-alerts-overview.md)

- [Organisation des ressources Azure à l’aide de catégories](../azure-resource-manager/management/tag-resources.md)

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

### <a name="pv-3-establish-secure-configurations-for-compute-resources"></a>PV-3 : Établir des configurations sécurisées pour des ressources de calcul

**Conseils** : Utilisez Azure Security Center et Azure Policy pour établir des configurations sécurisées sur toutes les ressources de calcul, notamment les machines virtuelles, les conteneurs, etc.

- [Suivi des recommandations d'Azure Security Center](../security-center/security-center-recommendations.md) 

- [Recommandations de sécurité - Guide de référence](../security-center/recommendations-reference.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="pv-8-conduct-regular-attack-simulation"></a>PV-8 : Effectuer une simulation d’attaque régulière

**Conseils** : Selon les besoins, effectuez un test d’intrusion ou des activités Red Team sur vos ressources Azure et résolvez tous les problèmes de sécurité critiques détectés.
Suivez les règles d’engagement de pénétration du cloud Microsoft pour vous assurer que vos tests d’intrusion sont conformes aux stratégies de Microsoft. Utilisez la stratégie et l’exécution de Red Teaming de Microsoft ainsi que les tests d’intrusion de site actif sur l’infrastructure cloud, les services et les applications gérés par Microsoft.

- [Test d’intrusion dans Azure](../security/fundamentals/pen-testing.md)

- [Règles d’engagement des tests d’intrusion](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1) 

- [Microsoft Cloud Red Teaming](https://gallery.technet.microsoft.com/Cloud-Red-Teaming-b837392e)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

## <a name="backup-and-recovery"></a>Sauvegarde et récupération

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Sauvegarde et récupération](../security/benchmarks/security-controls-v2-backup-recovery.md).*

### <a name="br-1-ensure-regular-automated-backups"></a>BR-1 : Garantir des sauvegardes automatiques régulières

**Conseils** : Étant donné qu’Azure HPC Cache est une solution de mise en cache et non un système de stockage, veillez à ce que les données de ses cibles de stockage soient régulièrement sauvegardées. Suivez les procédures standard pour les conteneurs de blobs Azure et pour sauvegarder toutes les cibles de stockage locales. 

Pour réduire l’interruption en cas de panne régionale, vous pouvez prendre des mesures pour garantir l’accès aux données entre les régions.  

Chaque instance Azure HPC Cache s’exécute dans un abonnement et dans une région. Cela signifie que votre workflow de cache peut être interrompu si la région connaît une panne totale. Pour réduire cette interruption, l’organisation doit utiliser un stockage principal accessible à partir de plusieurs régions. Ce stockage peut être soit un système NAS local avec une prise en charge DNS appropriée, soit un Stockage Blob Azure qui réside dans une autre région que celle du cache.

À mesure que votre workflow progresse dans votre région primaire, les données sont enregistrées dans le stockage à long terme en dehors de la région. Si la zone de cache n’est plus disponible, vous pouvez dupliquer une instance Azure HPC Cache dans une région secondaire, vous connecter au même stockage et reprendre le travail à partir du nouveau cache.

- [En savoir plus sur le basculement régional](hpc-region-recovery.md)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="br-2-encrypt-backup-data"></a>BR-2 : Chiffrer les données de sauvegarde

**Conseils** : Assurez-vous que vos sauvegardes sont protégées contre les attaques. Cela doit inclure le chiffrement des sauvegardes afin de vous protéger contre la perte de confidentialité.

Pour une sauvegarde locale à l’aide du service Sauvegarde Azure, le chiffrement au repos est assuré à l’aide de la phrase secrète que vous fournissez. Pour une sauvegarde de service Azure normale, les données de sauvegarde sont automatiquement chiffrées à l’aide de clés gérées par la plateforme Azure. Vous pouvez choisir de chiffrer la sauvegarde à l’aide de clés gérées par le client. Dans ce cas, assurez-vous que cette clé gérée par le client dans le coffre de clés se trouve également dans l’étendue de la sauvegarde.

Azure HPC Cache est également protégé par le chiffrement de l’hôte de machine virtuelle sur les disques gérés qui contiennent vos données mises en cache, même si vous ajoutez une clé de client pour les caches de disque. L’ajout d’une clé gérée par le client pour le double chiffrement offre un niveau supplémentaire de sécurité aux clients ayant des besoins élevés en matière de sécurité. Pour plus d’informations, lisez Chiffrement côté serveur de stockage sur disque Azure.

Utilisez le contrôle d’accès en fonction du rôle dans Sauvegarde Azure, Azure Key Vault ou d’autres ressources pour protéger les sauvegardes et les clés gérées par le client. En outre, vous pouvez activer des fonctionnalités de sécurité avancées pour exiger une authentification multifacteur avant que les sauvegardes soient modifiées ou supprimées.

- [Chiffrement de l’hôte de machine virtuelle](../virtual-machines/disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data)

- [Chiffrement côté serveur du stockage sur disque Azure](../virtual-machines/disk-encryption.md)

- [Vue d’ensemble des fonctionnalités de sécurité de Sauvegarde Azure](../backup/security-overview.md) 

- [Chiffrement des données de sauvegarde à l’aide de clés gérées par le client](../backup/encryption-at-rest-with-cmk.md)  

- [Guide pratique pour sauvegarder des clés de coffre de clés dans Azure](/powershell/module/az.keyvault/backup-azkeyvaultkey?preserve-view=true&view=azps-5.1.0)

**Supervision Azure Security Center** : actuellement non disponible

**Responsabilité** : Customer

### <a name="br-3-validate-all-backups-including-customer-managed-keys"></a>BR-3 : Valider toutes les sauvegardes, y compris les clés gérées par le client

**Conseils** : Vérifiez régulièrement que vous pouvez restaurer les clés gérées par le client qui sont sauvegardées.

- [Guide pratique pour restaurer des clés du coffre de clés dans Azure](/powershell/module/az.keyvault/restore-azkeyvaultkey?preserve-view=true&view=azps-5.1.0)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="br-4-mitigate-risk-of-lost-keys"></a>BR-4 : Atténuer les risques liés aux clés perdues

**Conseils** : Assurez-vous que vous avez pris les mesures nécessaires pour empêcher la perte de clés et récupérer d’une telle perte. Activez la suppression réversible et la protection contre la purge dans Azure Key Vault pour protéger les clés contre une suppression accidentelle ou malveillante.

- [Guide pratique pour activer la suppression réversible et la protection contre la purge dans Key Vault](../storage/blobs/soft-delete-blob-overview.md?tabs=azure-portal)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="governance-and-strategy"></a>Gouvernance et stratégie

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gouvernance et stratégie](../security/benchmarks/security-controls-v2-governance-strategy.md).*

### <a name="gs-1-define-asset-management-and-data-protection-strategy"></a>GS-1 : Définir la stratégie de gestion des ressources et de protection des données 

**Conseils** : Veillez à documenter et à communiquer une stratégie claire pour la surveillance et la protection continues des systèmes et des données. Hiérarchisez la découverte, l’évaluation, la protection et la surveillance des données et systèmes critiques de l’entreprise. 

Cette stratégie doit inclure les recommandations, stratégies et normes documentées pour les éléments suivants : 

-   Norme de classification des données en fonction des risques pour l’entreprise

-   Visibilité de l’organisation de sécurité sur les risques et l’inventaire des actifs 

-   Approbation de l’organisation de sécurité pour les services Azure en vue de leur utilisation 

-   Sécurité des ressources tout au long de leur cycle de vie

-   Stratégie de contrôle d’accès requise conformément à la classification des données organisationnelles

-   Utilisation des fonctionnalités de protection des données Azure natives et de tiers

-   Exigences de chiffrement des données pour les cas d’utilisation en transit et au repos

-   Normes de chiffrement appropriées

Pour plus d’informations, consultez les références suivantes :
- [Recommandation d’architecture de sécurité Azure - Stockage, données et chiffrement](/azure/architecture/framework/security/storage-data-encryption?bc=%2fsecurity%2fcompass%2fbreadcrumb%2ftoc.json&toc=%2fsecurity%2fcompass%2ftoc.json)

- [Notions de base de la sécurité Azure - Sécurité, chiffrement et stockage des données Azure](../security/fundamentals/encryption-overview.md)

- [Cloud Adoption Framework - Meilleures pratiques en matière de chiffrement et de sécurité des données Azure](../security/fundamentals/data-encryption-best-practices.md?bc=%2fazure%2fcloud-adoption-framework%2f_bread%2ftoc.json&toc=%2fazure%2fcloud-adoption-framework%2ftoc.json)

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

- [Benchmark de sécurité Azure - Accès privilégié](/azure/security/benchmarks/security-controls-v2-privileged-access)

- [Meilleures pratiques pour la sécurité Azure 11 – Architecture. Stratégie de sécurité unifiée unique](/azure/cloud-adoption-framework/security/security-top-10#11-architecture-establish-a-single-unified-security-strategy)

- [Vue d’ensemble de la sécurité et de la gestion des identités Azure](../security/fundamentals/identity-management-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="gs-7-define-logging-and-threat-response-strategy"></a>GS-7 : Définir la stratégie de journalisation et de réponse aux menaces

**Conseils** : Établissez une stratégie de journalisation et de réponse aux menaces pour détecter et corriger rapidement les menaces tout en répondant aux exigences de conformité. En priorité, fournissez aux analystes des alertes de bonne qualité et des expériences homogènes afin qu’ils puissent se concentrer sur les menaces plutôt que sur l’intégration et les étapes manuelles. 

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

- Consultez [Vue d’ensemble d’Azure Security Benchmark V2](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)