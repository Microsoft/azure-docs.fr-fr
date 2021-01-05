---
title: Base de référence de sécurité Azure pour Azure App Configuration
description: La base de référence de sécurité Azure App Configuration propose des instructions procédurales et des ressources pour la mise en œuvre des recommandations de sécurité spécifiées dans le Benchmark de sécurité Azure.
author: msmbaldwin
ms.service: azure-app-configuration
ms.topic: conceptual
ms.date: 11/20/2020
ms.author: mbaldwin
ms.custom: subject-security-benchmark
ms.openlocfilehash: 79a99ad37c526103fc3068562d62ed40defc983b
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96532378"
---
# <a name="azure-security-baseline-for-azure-app-configuration"></a>Base de référence de sécurité Azure pour Azure App Configuration

Cette base de référence de sécurité applique les instructions du [Benchmark de sécurité Azure version 2.0](../security/benchmarks/overview.md) à Azure App Configuration. Le benchmark de sécurité Azure fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Le contenu est regroupé selon les **contrôles de sécurité** définis par le Benchmark de sécurité Azure et les instructions associées applicables à Azure App Configuration. Les **contrôles** non applicables à Azure App Configuration ont été exclus.

Pour voir la correspondance complète entre Azure App Configuration et le Benchmark de sécurité Azure, consultez le [fichier complet de mise en correspondance de la base de référence de sécurité Azure App Configuration](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

## <a name="network-security"></a>Sécurité réseau

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : sécurité réseau](../security/benchmarks/security-controls-v2-network-security.md).*

### <a name="ns-1-implement-security-for-internal-traffic"></a>NS-1 : Implémenter la sécurité pour le trafic interne

**Conseils** : Azure App Configuration ne déploie pas directement de ressources dans un réseau virtuel. Étant donné que le service n’est pas déployé dans un réseau virtuel, il n’est pas possible de tirer parti de certaines fonctionnalités réseau pour sécuriser le trafic interne du service, par exemple les groupes de sécurité réseau, les tables de route et d’autres appliances réseau comme un Pare-feu Azure. Toutefois, App Configuration offre la possibilité d’utiliser des points de terminaison privés pour se connecter de manière sécurisée à Azure App Configuration à partir d’un réseau virtuel.

Utilisez Azure Sentinel pour découvrir l’utilisation de protocoles non sécurisés hérités, tels que SSL/TLSv1, SMBv1, LM/NTLMv1, wDigest, liaisons LDAP non signées et chiffrements faibles dans Kerberos.

- [Utilisation de points de terminaison privés pour Azure App Configuration](concept-private-endpoint.md)

- [Classeur des protocoles non sécurisés Azure Sentinel](../sentinel/quickstart-get-visibility.md#use-built-in-workbooks)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="ns-2-connect-private-networks-together"></a>NS-2 : Interconnecter des réseaux privés

**Conseils** : Azure App Configuration prend en charge l’utilisation de points de terminaison privés pour envoyer des données de manière sécurisée au moyen d’une liaison privée. Azure ExpressRoute ou le réseau privé virtuel (VPN) Azure permettent de créer des connexions privées entre les centres de données Azure et l’infrastructure locale dans un environnement de colocation. Les connexions ExpressRoute ne transitent pas par l’Internet public. Elles offrent davantage de fiabilité, des vitesses supérieures et des latences inférieures par rapport aux connexions Internet classiques. Pour un VPN point à site et un VPN site à site, vous pouvez connecter des appareils ou des réseaux locaux à un réseau virtuel à l’aide de n’importe quelle combinaison de ces options VPN et d’Azure ExpressRoute.

Pour interconnecter deux réseaux virtuels ou plus dans Azure, utilisez le peering de réseaux virtuels. Le trafic réseau entre les réseaux virtuels appairés est privé et conservé sur le réseau principal Azure.

- [Quels sont les modèles de connectivité ExpressRoute ?](../expressroute/expressroute-connectivity-models.md)

- [Vue d’ensemble des VPN Azure](../vpn-gateway/vpn-gateway-about-vpngateways.md)

- [Peering de réseau virtuel](../virtual-network/virtual-network-peering-overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="ns-3-establish-private-network-access-to-azure-services"></a>NS-3 : Établir un accès réseau privé aux services Azure

**Conseils** : Utilisez Azure Private Link pour permettre un accès privé à Azure App Configuration à partir de vos réseaux virtuels, sans passer par Internet.

L’accès privé est une mesure de défense renforcée qui vient s’ajouter à la sécurité de l’authentification et du trafic proposée par les services Azure.

- [Présentation d’Azure Private Link](../private-link/private-link-overview.md)

- [Guide pratique pour configurer une liaison privée dans Azure App Configuration](concept-private-endpoint.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

### <a name="ns-4-protect-applications-and-services-from-external-network-attacks"></a>NS-4 : Protégez les applications et les services contre les attaques réseau externes

**Conseils** : Lorsque vous accédez à des valeurs de configuration via un réseau virtuel, protégez vos ressources contre les attaques provenant de réseaux externes, notamment les attaques par déni de service distribué (DDoS), les attaques propres aux applications et le trafic Internet non sollicité, potentiellement malveillant. Utilisez le Pare-feu Azure pour protéger les applications et les services contre le trafic potentiellement malveillant provenant d’Internet et d’autres emplacements externes. Protégez vos ressources contre les attaques DDoS en activant la protection DDoS Standard sur vos réseaux virtuels Azure. Utilisez Azure Security Center pour détecter les risques de configuration incorrecte liés aux ressources associées à votre réseau.

Azure App Configuration n’est pas destiné à exécuter des applications web. Il en fournit simplement la configuration. Il n’est pas nécessaire de configurer des paramètres supplémentaires ni de déployer d’autres services réseau pour les protéger contre les attaques de réseau externe qui les visent.

- [Documentation sur le pare-feu Azure](../firewall/index.yml)

- [Gérer la norme Azure DDoS Protection à l’aide du portail Azure](../ddos-protection/manage-ddos-protection.md)

- [Recommandations relatives à Azure Security Center](../security-center/recommendations-reference.md#recs-network)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="ns-5-deploy-intrusion-detectionintrusion-prevention-systems-idsips"></a>NS-5 : Déployer des systèmes de détection et de prévention des intrusions (IDS/IPS)

**Conseils** : Utilisez le Pare-feu Azure avec filtrage basé sur le renseignement sur les menaces pour générer des alertes et bloquer le trafic en provenance et à destination d’adresses IP et de domaines malveillants connus. Ces adresses IP et domaines proviennent du flux Microsoft Threat Intelligence. Si l’inspection de la charge utile est obligatoire, vous pouvez déployer, à partir de la Place de marché Azure, une solution IDS/IPS tierce dotée de fonctionnalités d’inspection de charge utile. Vous pouvez également utiliser un système IDS/IPS basé sur un hôte ou une solution de protection évolutive des points de terminaison (PEPT) basée sur un hôte avec ou au lieu d’une solution IDS/IPS réseau.

Remarque : Si l’utilisation des systèmes IDS/IPS s’accompagne d’une exigence réglementaire ou autre, vérifiez qu’elle est toujours paramétrée de façon à fournir des alertes de haute qualité à votre solution SIEM.

- [Guide pratique pour déployer le Pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md)

- [La Place de marché Azure comprend des fonctionnalités IDS tierces](https://azuremarketplace.microsoft.com/marketplace?search=IDS)

- [Fonctionnalité de détection de point de terminaison et réponse (EDR) de Microsoft Defender ATP](/windows/security/threat-protection/microsoft-defender-atp/overview-endpoint-detection-response)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="ns-6-simplify-network-security-rules"></a>NS-6 : Simplifier les règles de sécurité réseau

**Conseils** : Utilisez les étiquettes de service Réseau virtuel Azure pour définir des contrôles d’accès réseau sur les groupes de sécurité réseau ou le Pare-feu Azure configurés pour vos ressources App Configuration. Vous pouvez utiliser l’étiquette de service « AppConfiguration » à la place d’adresses IP spécifiques lors de la création de règles de sécurité pour le trafic sortant dans le réseau de votre application. En spécifiant le nom de l’étiquette de service dans le champ de source ou de destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service correspondant. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent.

- [Présentation et usage des balises de service](../virtual-network/service-tags-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="identity-management"></a>Gestion des identités

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des identités](../security/benchmarks/security-controls-v2-identity-management.md).*

### <a name="im-1-standardize-azure-active-directory-as-the-central-identity-and-authentication-system"></a>IM-1 : Normaliser Azure Active Directory comme système d’authentification et d’identité central

**Conseils** : Azure App Configuration est compatible avec Azure Active Directory (Azure AD), le service de gestion des identités et des accès par défaut d’Azure. Vous devez normaliser Azure AD pour régir la gestion des identités et des accès de votre organisation dans :
- Les ressources cloud Microsoft, comme le portail Azure, le stockage Azure, les machines virtuelles Azure (Linux et Windows), les applications Azure Key Vault, PaaS et SaaS.
- Les ressources de votre organisation, comme les applications sur Azure ou les ressources réseau de votre entreprise.

La sécurisation d’Azure AD doit être d’une priorité élevée dans les pratiques de sécurité cloud de votre organisation. Azure AD fournit un score d'identité sécurisée pour vous aider à évaluer la sécurité des identités par rapport aux recommandations de Microsoft en matière de meilleures pratiques. Utilisez le score pour évaluer avec précision votre configuration par rapport aux meilleures pratiques recommandées et apporter des améliorations à votre posture de sécurité.

Azure fournit les rôles Azure intégrés suivants pour autoriser l’accès aux données App Configuration à l’aide d’Azure AD et d’OAuth :

- Propriétaire des données App Configuration : Utilisez ce rôle pour accorder un accès en lecture/écriture/suppression aux données App Configuration. Cela n’accorde pas l’accès à la ressource App Configuration.

- Lecteur des données App Configuration : Utilisez ce rôle pour accorder un accès en lecture aux données App Configuration. Cela n’accorde pas l’accès à la ressource App Configuration.

- Collaborateur : Utilisez ce rôle pour gérer la ressource App Configuration. Alors que les données d’App Configuration sont accessibles à l’aide de clés d’accès, ce rôle n’accorde pas l’accès aux données avec Azure AD.

- Lecteur : Utilisez ce rôle pour accorder un accès en lecture à la ressource App Configuration. Cela n’accorde pas l’accès aux clés d’accès de la ressource ni aux données stockées dans App Configuration.

Pour plus d’informations, consultez les références suivantes :

- [Création et configuration d’une instance Azure AD](../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

- [Qu’est-ce que le degré de sécurisation Identity Secure Score dans Azure Active Directory](../active-directory/fundamentals/identity-secure-score.md)

- [Autoriser l’accès à Azure App Configuration avec Azure AD](concept-enable-rbac.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-2-manage-application-identities-securely-and-automatically"></a>IM-2 : Gérer les identités d’application de façon sécurisée et automatique

**Conseils** : Utilisez les identités managées Azure pour accéder à Azure App Configuration à partir de comptes non humains, par exemple d’autres services Azure. Il est recommandé de recourir à la fonctionnalité d’identité managée Azure plutôt que de créer un compte humain plus puissant pour accéder aux ressources ou les exécuter. En effet, cela permet de limiter les informations d’identification à gérer. Azure App Configuration peut également se voir affecter une identité managée pour s’authentifier en mode natif auprès d’autres services/ressources Azure prenant en charge l’authentification Azure AD. Cela peut être utile pour faciliter l’accès d’App Configuration à Azure Key Vault lors de la récupération des secrets. Les identités managées sont gérées par la plateforme Azure, ce qui vous évite d’avoir à provisionner et à renouveler des secrets.

Azure App Configuration prend en charge l’octroi de deux types d’identités à votre application :
- Une identité affectée par le système est liée à votre ressource de configuration. Elle est supprimée en cas de suppression de cette ressource. Une ressource de configuration ne peut comporter qu’une seule identité affectée par le système.
- Une identité affectée par l’utilisateur est une ressource Azure autonome qui peut être affectée à une ressource de configuration. Une ressource de configuration peut comporter plusieurs identités affectées par l’utilisateur.

Si vous ne pouvez pas tirer parti des identités managées, créez un principal de service disposant d’autorisations restreintes au niveau de la ressource Azure App Configuration. Configurez ces principaux de service avec des informations d’identification de type certificat, et des clés secrètes client en secours seulement. Dans les deux cas, Azure Key Vault peut être utilisé conjointement avec des identités managées Azure, afin que l’environnement d’exécution (par exemple, une fonction Azure) puisse récupérer les informations d’identification dans le coffre de clés.

- [Découvrez comment utiliser des identités managées pour Azure App Configuration](overview-managed-identity.md)

- [Identités managées Azure](../active-directory/managed-identities-azure-resources/overview.md)

- [Services prenant en charge les identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

- [Utiliser des identités managées pour accéder à App Configuration](howto-integrate-azure-managed-service-identity.md)

- [Principal de service Azure](/powershell/azure/create-azure-service-principal-azureps) 

- [Créer un principal du service avec un certificat](../active-directory/develop/howto-authenticate-service-principal-powershell.md) 

- [Utiliser Azure Key Vault pour l’inscription du principal de sécurité](../key-vault/general/authentication.md#app-identity-and-security-principals)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-3-use-azure-ad-single-sign-on-sso-for-application-access"></a>IM-3 : Utiliser l’authentification unique Azure AD pour l’accès aux applications

**Conseils** : Azure App Configuration utilise Azure Active Directory (Azure AD) pour fournir la gestion des identités et des accès aux ressources Azure, aux applications cloud et aux applications locales. Cela inclut les identités d’entreprise, comme les employés, ainsi que les identités externes, comme les partenaires et les fournisseurs. Azure AD permet à l’authentification unique (SSO) de gérer le service App Configuration par le biais du Portail Azure à l’aide des identités d’entreprise Active Directory synchronisées. Connectez l’ensemble de vos utilisateurs, applications et appareils à Azure AD pour un accès transparent et sécurisé, ainsi qu’une visibilité et un contrôle accrus.

- [Comprendre l’authentification unique des applications avec Azure AD](../active-directory/manage-apps/what-is-single-sign-on.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-4-use-strong-authentication-controls-for-all-azure-active-directory-based-access"></a>IM-4 : Utiliser des contrôles d’authentification renforcés pour tous les accès basés sur Azure Active Directory

**Conseils** : Azure App Configuration utilise Azure Active Directory, qui prend en charge des contrôles d’authentification forts (multifacteur, ou MFA) et des méthodes sans mot de passe fortes.
- Authentification multifacteur – Activez l’authentification multifacteur Azure AD et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center pour appliquer quelques bonnes pratiques dans votre configuration MFA. L’authentification multifacteur peut être appliquée à tous les utilisateurs, à des utilisateurs sélectionnés ou au niveau de chaque utilisateur en fonction des conditions de connexion et des facteurs de risque.
- Authentification sans mot de passe – Trois options d’authentification sans mot de passe sont disponibles : Windows Hello Entreprise, l’application Microsoft Authenticator et les méthodes d’authentification locales, comme les cartes à puce.

Pour les utilisateurs administrateurs et privilégiés, veillez à ce que soit utilisé le niveau le plus élevé de méthode d’authentification forte. Ensuite, déployez la stratégie d’authentification forte appropriée auprès des autres utilisateurs.

Remarque : L’authentification MFA peut être appliquée aux comptes d’utilisateur qui accèdent à App Configuration et le gèrent, et non aux comptes de service programmatiques. Utilisez l’authentification sans mot de passe (par exemple les identités managées) dans la mesure du possible, et appliquez l’authentification MFA à tous les comptes d’utilisateur.

- [Guide pratique pour activer l’authentification MFA dans Azure](../active-directory/authentication/howto-mfa-getstarted.md)

- [Introduction aux options d’authentification sans mot de passe pour Azure Active Directory](../active-directory/authentication/concept-authentication-passwordless.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-5-monitor-and-alert-on-account-anomalies"></a>IM-5 : Surveiller et alerter en cas d’anomalies de compte

**Conseils** : Azure App Configuration est compatible avec Azure Active Directory, qui fournit les sources de données suivantes :

-   Connexions – le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.

-   Journaux d’audit : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Les exemples de modifications journalisées dans les journaux d’audit incluent l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles et de stratégies.

-   Connexions risquées : une connexion risquée est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.

-   Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Ces sources de données peuvent être intégrées à Azure Monitor, à Azure Sentinel ou à des systèmes SIEM tiers.

Azure Security Center pouvez également alerter en cas de certaines activités suspectes, comme un nombre excessif de tentatives d’authentification ayant échoué et la présence de comptes dépréciés dans l’abonnement. 

Azure Advanced Threat Protection (ATP) est une solution de sécurité qui peut utiliser vos signaux Active Directory locaux pour identifier, détecter et examiner les menaces avancées, les identités compromises et les actions malveillantes internes.

- [Rapports d’activité d’audit dans Azure AD](../active-directory/reports-monitoring/concept-audit-logs.md)

- [Guide pratique pour afficher les connexions risquées Azure AD](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../security-center/security-center-identity-access.md)

- [Alertes dans le module de protection de renseignement sur les menaces d’Azure Security Center](../security-center/alerts-reference.md)

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Connecter des données depuis Azure AD Identity Protection](../sentinel/connect-azure-ad-identity-protection.md)

- [Azure Advanced Threat Protection](/azure-advanced-threat-protection/what-is-atp)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-6-restrict-azure-resource-access-based-on-conditions"></a>IM-6 : Restreindre l’accès aux ressources Azure en fonction des conditions

**Conseils** : Azure App Configuration prend en charge l’accès conditionnel Azure Active Directory (Azure AD) pour un contrôle d’accès plus granulaire en fonction des conditions définies par l’utilisateur (par exemple, les utilisateurs de certaines plages d’adresses IP devront utiliser l’authentification MFA pour se connecter). Une stratégie de gestion granulaire des sessions d’authentification peut également être utilisée dans différents cas d’usage. Ces stratégies d’accès conditionnel s’appliquent uniquement aux comptes d’utilisateurs qui s’authentifient auprès d’Azure AD pour accéder au service App Configuration et le gérer, et non aux principaux de service ni aux chaînes de connexion qui se connectent à votre ressource de configuration.

- [Présentation de l’accès conditionnel Azure](../active-directory/conditional-access/overview.md)

- [Stratégies d’accès conditionnel courantes](../active-directory/conditional-access/concept-conditional-access-policy-common.md)

- [Configurer la gestion des sessions d’authentification avec l’accès conditionnel](../active-directory/conditional-access/howto-conditional-access-session-lifetime.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="im-7-eliminate-unintended-credential-exposure"></a>IM-7 : Éliminer l’exposition involontaire des informations d’identification

**Conseils** : Azure App Configuration permet aux clients de stocker des configurations qui peuvent éventuellement contenir des identités ou des secrets. Il est recommandé d’implémenter le moteur d’analyse des informations d’identification pour identifier les informations d’identification dans les configurations. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

Utilisez le service Azure App Configuration avec Azure Key Vault. Stockez les informations d’identification dans Key Vault, puis établissez un lien vers celles-ci en créant une référence Key Vault dans votre ressource App Configuration. Lorsque le service App Configuration crée ces références, il stocke l’URI des valeurs Key Vault plutôt que les valeurs proprement dites. Les applications peuvent se connecter à App Configuration pour récupérer les informations d’identification auprès de Key Vault.

Pour GitHub, vous pouvez utiliser la fonctionnalité native d’analyse de secret pour identifier les informations d’identification ou d’autres formes de secrets dans le code.

- [Tutoriel d’utilisation de références Key Vault dans une application ASP.NET Core](use-key-vault-references-dotnet-core.md)

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

- [Analyse du secret GitHub](https://docs.github.com/github/administering-a-repository/about-secret-scanning)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="privileged-access"></a>Accès privilégié

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Accès privilégié](../security/benchmarks/security-controls-v2-privileged-access.md).*

### <a name="pa-1-protect-and-limit-highly-privileged-users"></a>PA-1 : Protéger et limiter les utilisateurs disposant de privilèges élevés

**Conseils** : Limitez le nombre de comptes ou rôles très privilégiés et protégez ces comptes à un niveau élevé, car les utilisateurs disposant de ces privilèges peuvent lire et modifier, directement ou non, toutes les ressources de votre environnement Azure.

Vous pouvez activer l’accès privilégié juste-à-temps (JAT) aux ressources Azure et Azure AD en utilisant Azure AD Privileged Identity Management (PIM). JAT accorde des autorisations temporaires pour effectuer des tâches privilégiées uniquement lorsque les utilisateurs en ont besoin. PIM peut également générer des alertes de sécurité en cas d’activité suspecte ou non sécurisée dans votre organisation Azure AD.

Les clés d’accès sont très privilégiées. Elles doivent être renouvelées régulièrement pour des raisons de sécurité. Elles contiennent des chaînes de connexion qui comportent des informations d’identification. Elles sont donc considérées comme des secrets. Ces secrets doivent être stockés dans Azure Key Vault, et votre code doit s’authentifier auprès de Key Vault pour les récupérer. Les clés d’accès peuvent fournir un accès en lecture et en écriture ou simplement en lecture à une application. Veillez à ce que la clé d’accès émise soit du bon type pour empêcher tout accès non autorisé. Pour plus de sécurité, utilisez la fonctionnalité d’identités managées d’Azure AD. La seule condition est que les applications disposent de l’URL du point de terminaison de configuration pour accéder aux valeurs de configuration.

- [Meilleures pratiques App Configuration](howto-best-practices.md#app-configuration-bootstrap)

- [Utiliser des identités managées pour accéder à App Configuration](howto-integrate-azure-managed-service-identity.md)
- [Autorisations de rôle Administrateur dans Azure AD](../active-directory/roles/permissions-reference.md)

- [Utiliser les alertes de sécurité Azure Privileged Identity Management](../active-directory/privileged-identity-management/pim-how-to-configure-security-alerts.md)

- [Sécurisation de l’accès privilégié pour les déploiements hybrides et cloud dans Azure AD](../active-directory/roles/security-planning.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="pa-2-restrict-administrative-access-to-business-critical-systems"></a>PA-2 : Limiter l’accès administratif aux systèmes critiques de l’entreprise

**Conseils** : Azure App Configuration utilise le contrôle RBAC Azure afin d’isoler l’accès aux systèmes critiques pour l’entreprise en restreignant les comptes disposant d’un accès privilégié. Azure RBAC est pris en charge par App Configuration au niveau de la ressource. Pour compartimenter les configurations critiques pour l’entreprise de manière sécurisée, stockez ces informations dans leur propre ressource App Configuration. Au sein d’une ressource, un accès granulaire est également possible au moyen de clés ou de comptes d’accès en lecture seule, ainsi qu’avec l’étiquetage et le balisage.

Pour assurer un contrôle d’accès cohérent, tous les types de contrôle d’accès doivent être alignés sur la stratégie de segmentation de votre entreprise.

- [Accès aux groupes d’administration](../governance/management-groups/overview.md#management-group-access)

- [Administrateurs d’abonnements Azure](../cost-management-billing/manage/add-change-subscription-administrator.md)

- [Intégrer le contrôle RBAC à l’aide d’Azure AD avec App Configuration](concept-enable-rbac.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="pa-3-review-and-reconcile-user-access-regularly"></a>PA-3 : Examiner et rapprocher régulièrement les accès utilisateur

**Conseils** : Azure App Configuration utilise des comptes Azure Active Directory (Azure AD) pour gérer ses ressources, et examiner régulièrement les comptes d’utilisateur et l’attribution des accès pour veiller à ce que les comptes et leur accès soient valides. 

Azure fournit les rôles Azure intégrés suivants pour autoriser l’accès aux données App Configuration à l’aide d’Azure AD et d’OAuth :

- Propriétaire des données App Configuration : Utilisez ce rôle pour accorder un accès en lecture/écriture/suppression aux données App Configuration. Cela n’accorde pas l’accès à la ressource App Configuration.

- Lecteur des données App Configuration : Utilisez ce rôle pour accorder un accès en lecture aux données App Configuration. Il ne donne pas accès à la ressource App Configuration.

Vous pouvez utiliser les révisions d’accès Azure AD pour examiner les appartenances aux groupes, l’accès aux applications d’entreprise et les attributions de rôles (par exemple les rôles App Configuration ci-dessus). Les rapports Azure AD peuvent fournir des journaux pour vous aider à découvrir les comptes obsolètes. Vous pouvez également utiliser Azure AD Privileged Identity Management pour créer un workflow de rapport de révision d’accès afin de faciliter le processus de révision.

Remarque : Les identités managées sont suggérées dans la mesure du possible pour l’authentification auprès d’App Configuration à partir d’un autre service ou d’une autre application. Si vous les utilisez, vous devrez gérer séparément les principaux de service et les chaînes de connexion configurés avec un accès à App Configuration.

- [Créer une révision d’accès des rôles de ressources Azure dans Privileged Identity Management (PIM)](../active-directory/privileged-identity-management/pim-resource-roles-start-access-review.md) 

- [Utilisation des révisions d’accès et des identités Azure AD](/azure/active-directory/governance/access-reviews-overview)

- [Autoriser l’accès à Azure App Configuration avec Azure AD](concept-enable-rbac.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="pa-4-set-up-emergency-access-in-azure-ad"></a>PA-4 : Configurer l’accès d’urgence dans Azure AD

**Conseils** : Azure App Configuration est compatible avec Azure Active Directory pour gérer ses ressources. Pour empêcher le verrouillage accidentel de votre organisation Azure AD, configurez un compte d’accès d’urgence pour conserver un accès lorsque les comptes administratifs normaux ne peuvent pas être utilisés. Les comptes d’accès d’urgence sont généralement hautement privilégiés et ne doivent pas être attribués à des personnes spécifiques. Les comptes d’accès d’urgence sont limités à des cas d’urgence ou à des scénarios « de secours » où il est impossible d’utiliser des comptes d’administration normaux.

Vous devez vous assurer que les informations d’identification (telles que le mot de passe, le certificat ou la carte à puce) des comptes d’accès d’urgence restent sécurisées et connues des seules personnes autorisées à les utiliser en cas d’urgence.

- [Gérer des comptes d’accès d’urgence dans Azure AD](../active-directory/roles/security-emergency-access.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="pa-5-automate-entitlement-management"></a>PA-5 : Automatiser la gestion des droits d'utilisation 

**Conseils** : Azure App Configuration est compatible avec Azure Active Directory pour gérer ses ressources. Utilisez les fonctionnalités de gestion des droits d’utilisation d’Azure AD pour automatiser les workflows de demandes d’accès, notamment les attributions, les révisions et l’expiration des accès. L’approbation en deux ou plusieurs étapes est également prise en charge.

- [Présentation des révisions d’accès Azure AD](../active-directory/governance/access-reviews-overview.md)

- [Présentation de la gestion des droits d’utilisation Azure AD](../active-directory/governance/entitlement-management-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="pa-6-use-privileged-access-workstations"></a>PA-6 : Utiliser des stations de travail d’accès privilégié

**Conseils** : Les stations de travail sécurisées et isolées sont extrêmement importantes pour la sécurité des rôles sensibles comme les administrateurs, développeurs et opérateurs de service critique. Utilisez des stations de travail utilisateur hautement sécurisées ou Azure Bastion pour les tâches d’administration liées à App Configuration. Utilisez Azure Active Directory, Microsoft Defender Advanced Threat Protection (MDATP) et/ou Microsoft Intune pour déployer une station de travail utilisateur sécurisée et gérée pour les tâches d’administration. Les stations de travail sécurisées peuvent être gérées de manière centralisée pour appliquer une configuration sécurisée, notamment une authentification forte, des lignes de base logicielles et matérielles et un accès réseau et logique restreint.

- [Comprendre les stations de travail d’accès privilégié](../active-directory/devices/concept-azure-managed-workstation.md) 

- [Déployer une station de travail d’accès privilégié](../active-directory/devices/howto-azure-managed-workstation.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="pa-7-follow-just-enough-administration-least-privilege-principle"></a>PA-7 : Suivre le principe Just Enough Administration (privilèges minimum) 

**Conseils** : Azure App Configuration est compatible avec le contrôle d’accès en fonction du rôle (RBAC) Azure pour gérer ses ressources. Azure RBAC vous permet de gérer l’accès aux ressources Azure via des attributions de rôles. Vous pouvez affecter ces rôles à des utilisateurs, regrouper des principaux de service et des identités managées. Il existe des rôles intégrés prédéfinis pour Azure App Configuration, que vous pouvez lister ou interroger par le biais d’outils tels qu’Azure CLI, Azure PowerShell ou le Portail Azure. Les privilèges que vous affectez aux ressources via Azure RBAC doivent toujours être limités à ce qui est requis par les rôles. Ils complètent l’approche juste-à-temps (JIT) d’Azure AD Privileged Identity Management (PIM) et doivent être révisés régulièrement.

Azure fournit les rôles Azure intégrés suivants pour autoriser l’accès aux données App Configuration à l’aide d’Azure AD et d’OAuth :
- Propriétaire des données App Configuration : Utilisez ce rôle pour accorder un accès en lecture/écriture/suppression aux données App Configuration. Cela n’accorde pas l’accès à la ressource App Configuration.
- Lecteur des données App Configuration : Utilisez ce rôle pour accorder un accès en lecture aux données App Configuration. Cela n’accorde pas l’accès à la ressource App Configuration.

Utilisez des rôles intégrés pour allouer les autorisations. Ne créez des rôles personnalisés que si nécessaire. 

App Configuration prend en charge le stockage de la configuration de plusieurs applications dans une ressource App Configuration. Pour limiter l’accès aux informations entre les applications, créez une ressource App Configuration par application et configurez Azure RBAC en conséquence.

- [Présentation du contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md)

- [Configurer le contrôle d'accès en fonction du rôle dans Azure](../role-based-access-control/role-assignments-portal.md)

- [Utilisation des révisions d’accès et des identités Azure AD](../active-directory/governance/access-reviews-overview.md)

- [Autoriser l’accès à Azure App Configuration avec Azure AD](concept-enable-rbac.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="pa-8-choose-approval-process-for-microsoft-support"></a>PA-8 : Choisir le processus d’approbation pour le support Microsoft  

**Conseils** : Implémentez un processus d’approbation organisationnel pour les scénarios de support dans lesquels Microsoft est susceptible d’avoir besoin d’accéder à vos données App Configuration. Customer Lockbox n’est à l’heure actuelle pas disponible pour les scénarios de support d’App Configuration.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="data-protection"></a>Protection des données

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : protection des données](../security/benchmarks/security-controls-v2-data-protection.md).*

### <a name="dp-1-discover-classify-and-label-sensitive-data"></a>DP-1 : Découvrir, classer et étiqueter des données sensibles

**Conseils** : Découvrez, classifiez et étiquetez vos données sensibles de façon à concevoir les contrôles appropriés pour faire en sorte que le stockage, le traitement et la transmission sécurisées des informations sensibles soient assurés par les systèmes technologiques de l’organisation. L’étiquetage des informations sensibles, sous forme de marquage, est pris en charge pour les ressources App Configuration, et non pour les valeurs de configuration qu’elles contiennent. Une fois titulaires d’un accès en lecture ou en lecture et en écriture à un magasin de configuration, les applications disposent d’un accès total à toutes les configurations de ce magasin.

- [Étiqueter des informations sensibles à l’aide d’Azure Information Protection](/azure/information-protection/what-is-information-protection)

- [Marquage des classifications des données dans Azure](/azure/cloud-adoption-framework/govern/policy-compliance/data-classification#tagging-data-classification-in-azure)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="dp-2-protect-sensitive-data"></a>DP-2 : Protection des données sensibles

**Conseils** : Pour la plateforme sous-jacente gérée par Microsoft, Microsoft traite tout le contenu client en tant que contenu sensible, et assure une protection contre la perte et l’exposition des données client. Pour assurer la sécurité des données des clients dans Azure, Microsoft a implémenté certains contrôles et fonctionnalités de protection des données par défaut. Veillez à renouveler régulièrement les clés d’accès à vos ressources App Configuration. Les informations d’identification issues des chaînes de connexion doivent être stockées dans Azure Key Vault. Votre code doit s’authentifier auprès de Key Vault pour les récupérer. Les clés d’accès peuvent fournir un accès en lecture et en écriture ou simplement en lecture à une application. Veillez à ce que la clé d’accès émise soit du bon type pour empêcher tout accès non autorisé. Pour plus de sécurité, utilisez la fonctionnalité d’identités managées d’Azure AD. La seule condition est que les applications disposent de l’URL du point de terminaison de configuration pour accéder aux valeurs de configuration.

Restreignez l’accès à l’aide du contrôle d’accès en fonction du rôle Azure (Azure RBAC) :

- Séparez les données sensibles dans leur propre ressource App Configuration, puis allouez les stratégies RBAC en conséquence pour que seul l’accès autorisé soit activé. 

- Utilisez des contrôles d’accès réseau.

- Des contrôles spécifiques dans les services Azure (par exemple le chiffrement dans SQL et d’autres bases de données) garantissent un contrôle d’accès cohérent. Tous les types de contrôle d’accès doivent être alignés sur votre stratégie de segmentation d’entreprise.

- La stratégie de segmentation de l’entreprise doit aussi être informée de l’emplacement des systèmes et données sensibles ou vitaux de l’entreprise.

Pour plus d’informations, consultez les références suivantes :

- [Autoriser l’accès à Azure App Configuration Azure avec Azure Active Directory](concept-enable-rbac.md)

- [Chiffrement de données App Configuration](faq.md#does-app-configuration-encrypt-my-data)

- [Contrôle d’accès en fonction du rôle (RBAC) Azure](../role-based-access-control/overview.md) 

- [Présentation de la protection des données client dans Azure](../security/fundamentals/protection-customer-data.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="dp-4-encrypt-sensitive-information-in-transit"></a>DP-4 : Chiffrement des informations sensibles en transit

**Conseils** : En complément des contrôles d’accès, les données en transit doivent être protégées contre les attaques « hors bande » à l’aide du chiffrement. Cela vise à empêcher que les attaquants puissent facilement lire ou modifier les données.

Azure App Configuration utilise le chiffrement TLS pour toutes les requêtes HTTP. L’infrastructure Azure fournit une couche supplémentaire de chiffrement au niveau du réseau pour toutes les demandes effectuées entre les centres de données Azure. Pour le trafic HTTP, veillez à ce que les clients qui se connectent à vos ressources App Configuration puissent négocier la version 1.2 ou une version supérieure du protocole TLS.

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Partagé

### <a name="dp-5-encrypt-sensitive-data-at-rest"></a>DP-5 : Chiffrement des données sensibles au repos

**Conseils** : En complément des contrôles d’accès, les données au repos doivent être protégées contre les attaques « hors bande » (telles que l’accès au stockage sous-jacent) à l’aide du chiffrement. Cela vise à empêcher que les attaquants puissent facilement lire ou modifier les données.

Azure assure par défaut un chiffrement des données au repos. Pour les données hautement sensibles, vous avez le choix entre plusieurs options pour implémenter un chiffrement supplémentaire au repos sur toutes les ressources Azure, le cas échéant. Azure gère par défaut vos clés de chiffrement, mais vous offre la possibilité de gérer vos propres clés (clés gérées par le client) pour Azure App Configuration.

- [Utiliser des clés gérées par le client pour chiffrer des données dans Azure App Configuration](concept-customer-managed-keys.md)

- [Présentation du chiffrement au repos dans Azure](../security/fundamentals/encryption-atrest.md#encryption-at-rest-in-microsoft-cloud-services) 

- [Modèle de chiffrement et table de gestion des clés](../security/fundamentals/encryption-atrest.md#azure-resource-providers-encryption-model-support)

- [Double chiffrement des données au repos dans Azure](../security/fundamentals/double-encryption.md#data-at-rest)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Customer

## <a name="asset-management"></a>Gestion des ressources

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Gestion des ressources](/azure/security/benchmarks/security-controls-v2-asset-management).*

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

**Conseils** : Veillez à ce que les équipes de sécurité aient accès à un inventaire des ressources mis à jour en permanence sur Azure, comme Azure App Configuration. Les équipes de sécurité ont souvent besoin de cet inventaire pour évaluer l’exposition potentielle de leur organisation à des risques émergents, et en tant qu’entrée pour des améliorations de sécurité continues. Créez un groupe Azure Active Directory, qui contiendra l’équipe de sécurité autorisée de votre organisation, et attribuez-lui un accès en lecture à toutes les ressources Azure App Configuration. Cette procédure peut être simplifiée par une seule attribution de rôle de haut niveau au sein de votre abonnement.

La fonctionnalité d’inventaire d’Azure Security Center et Azure Resource Graph peuvent rechercher et découvrir toutes les ressources de vos abonnements, notamment les services Azure, les applications et les ressources réseau.

Appliquez des étiquettes à vos ressources Azure, groupes de ressources et abonnements pour les organiser de façon logique dans une taxonomie. Chaque balise se compose d’une paire nom-valeur. Par exemple, vous pouvez appliquer le nom « Environnement » et la valeur « Production » à toutes les ressources en production.

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

- [Gestion de l’inventaire de sécurité dans Azure Security Center](../security-center/asset-inventory.md)

- [Pour plus d’informations sur l’étiquetage des ressources, consultez le guide de décision concernant le nommage et l’étiquetage des ressources](/azure/cloud-adoption-framework/decision-guides/resource-tagging/?toc=%2fazure%2fazure-resource-manager%2fmanagement%2ftoc.json)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="am-3-use-only-approved-azure-services"></a>AM-3 : Utiliser des services Azure approuvés uniquement

**Conseils** : Azure App Configuration prend en charge les déploiements basés sur Azure Resource Manager et la mise en œuvre de la configuration à l’aide d’Azure Policy. Utilisez Azure Policy pour auditer et limiter les services que les utilisateurs peuvent approvisionner dans votre environnement. Utilisez Azure Resource Graph pour interroger et découvrir des ressources dans leurs abonnements. Vous pouvez également utiliser Azure Monitor pour créer des règles afin de déclencher des alertes lorsqu’un service non approuvé est détecté.

- [Guide pratique pour configurer et gérer Azure Policy](../governance/policy/tutorials/create-and-manage.md)

- [Guide pratique pour refuser un type de ressource spécifique avec Azure Policy](../governance/policy/samples/built-in-policies.md#general)

- [Procédure pour créer des requêtes avec l’Explorateur Azure Resource Graph](../governance/resource-graph/first-query-portal.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="am-4-ensure-security-of-asset-lifecycle-management"></a>AM-4 : Garantir la sécurité de la gestion du cycle de vie des actifs

**Conseils** : Établissez ou mettez à jour des stratégies de sécurité qui traitent les processus de gestion du cycle de vie pour les modifications à fort impact potentiel. Il s’agit notamment des modifications apportées aux fournisseurs d’identité et aux accès, au niveau de confidentialité des données, à la configuration réseau et à l’attribution de privilèges administratifs.

Supprimez les ressources Azure qui ne sont plus nécessaires. Veillez à ce que les administrateurs renouvellent régulièrement les clés d’accès pour que seuls les utilisateurs authentifiés aient accès à leur ressource de configuration.

- [Renouveler les clés de chiffrement utilisées pour App Configuration](concept-customer-managed-keys.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="am-5-limit-users-ability-to-interact-with-azure-resource-manager"></a>AM-5 : Limiter la capacité des utilisateurs à interagir avec Azure Resource Manager

**Aide** : Utilisez l’accès conditionnel Azure pour limiter la capacité des utilisateurs à interagir avec Azure Resource Manager en configurant « Bloquer l’accès » pour l’application « Gestion Microsoft Azure ».

- [Guide pratique pour configurer l’accès conditionnel de façon à bloquer l’accès à Azure Resource Manager](../role-based-access-control/conditional-access-azure-management.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="logging-and-threat-detection"></a>Journalisation et détection des menaces

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : Journalisation et détection des menaces](/azure/security/benchmarks/security-controls-v2-logging-threat-detection).*

### <a name="lt-2-enable-threat-detection-for-azure-identity-and-access-management"></a>LT-2 : Activer la détection des menaces pour la gestion des identités et des accès Azure

**Conseils** : App Configuration est compatible avec Azure Active Directory (Azure AD). Sont ainsi fournis les journaux d’utilisateur suivants, qui peuvent être consultés dans les rapports Azure AD ou intégrés à Azure Monitor, à Azure Sentinel ou à d’autres outils de SIEM/monitoring pour des cas d’usage plus complexes de monitoring et d’analytique :
- Connexions – le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.
- Journaux d’audit : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Par exemple, les journaux d’audit peuvent inclure les modifications apportées à des ressources dans Azure AD comme l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles, de stratégies, etc.
- Connexions risquées : une connexion risquée est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.
- Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Azure Security Center peut également alerter dans le cas de certaines activités suspectes, comme un nombre excessif de tentatives d’authentification ayant échoué et la présence de comptes dépréciés dans l’abonnement. En plus du monitoring de base de la gestion de la sécurité, le module Protection contre les menaces d’Azure Security Center peut collecter des alertes de sécurité plus approfondies sur les couches de service Azure. Cette capacité offre une bonne visibilité sur les anomalies des comptes à l’intérieur des différentes ressources.

Il existe une autre méthode pour accéder à la ressource de configuration App Configuration, qui consiste à utiliser des clés d’accès. Celles-ci doivent être renouvelées régulièrement pour qu’aucun agent non autorisé n’ait accès à la ressource de configuration. Le renouvellement peut être effectué directement sur le portail, sous « clés d’accès ».

- [Permettre à Azure App Configuration d’accéder à d’autres ressources protégées Azure AD à l’aide d’une identité managée](overview-managed-identity.md)

- [Utiliser les identités managées avec Azure App Configuration](howto-integrate-azure-managed-service-identity.md)

- [Rapports d’activité d’audit dans Azure Active Directory](../active-directory/reports-monitoring/concept-audit-logs.md)

- [Activer Azure AD Identity Protection](../active-directory/identity-protection/overview-identity-protection.md)

- [Autoriser l’accès à Azure App Configuration avec Azure AD](concept-enable-rbac.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="lt-3-enable-logging-for-azure-network-activities"></a>LT-3 : Activer la journalisation pour les activités réseau Azure

**Conseils** : Azure App Configuration ne déploie pas directement de ressources dans un réseau virtuel. Toutefois, App Configuration offre la possibilité d’utiliser des points de terminaison privés pour se connecter de manière sécurisée à Azure App Configuration à partir d’un réseau virtuel. Par ailleurs, Azure App Configuration ne produit ni ne traite les journaux de requêtes DNS qui doivent être activés.

Activez la journalisation sur vos points de terminaison privés App Configuration configurés pour capturer les données suivantes :
- Données traitées par le point de terminaison privé (entrée/sortie)
- Données traitées par le service Liaison privée (entrée/sortie)
- Disponibilité du port NAT

Pour plus d’informations, consultez les références suivantes :

- [Monitoring Azure Private Link](../private-link/private-link-overview.md#logging-and-monitoring)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="lt-4-enable-logging-for-azure-resources"></a>LT-4 : Activer la journalisation pour les ressources Azure

**Conseils** : Les journaux d’activité, disponibles automatiquement, contiennent toutes les opérations d’écriture (PUT, POST, DELETE) des ressources App Configuration, et non les opérations de lecture (GET). Les journaux d’activité peuvent être utilisés pour rechercher une erreur lors de la résolution de problèmes ou pour surveiller la manière dont un utilisateur de votre organisation a modifié une ressource. Pour App Configuration, les journaux d’activité ne sont disponibles que dans le plan de contrôle. Ils sont exposés par Azure Resource Manager (ARM). La journalisation du plan de données côté client pour App Configuration n’est pas prise en charge actuellement. Les journaux de ressources Azure ne sont pas non plus configurables.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md)

- [Présentation de la journalisation et des différents types de journaux dans Azure](../azure-monitor/platform/platform-logs-overview.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="lt-5-centralize-security-log-management-and-analysis"></a>LT-5 : Centraliser la gestion et l’analyse des journaux de sécurité

**Conseils** : Centralisez le stockage et l’analyse de la journalisation pour activer la corrélation. Pour chaque source de journal, assurez-vous d’avoir attribué un propriétaire de données, des conseils d’accès, un emplacement de stockage, les outils utilisés pour traiter les données et y accéder, ainsi que les exigences de conservation des données.

Veillez à intégrer les journaux d’activité Azure dans votre journalisation centralisée. Ingérez des journaux par le biais d’Azure Monitor pour agréger les données de sécurité générées par les appareils de point de terminaison, les ressources réseau et d’autres systèmes de sécurité. Dans Azure Monitor, utilisez des espaces de travail Log Analytics pour interroger et effectuer l’analytique, et utilisez des comptes Stockage Azure pour le stockage à long terme et l’archivage.

En outre, activez et intégrez les données dans Azure Sentinel ou une solution SIEM tierce. De nombreuses organisations choisissent d’utiliser Azure Sentinel pour les données « chaudes » qui sont utilisées fréquemment et Stockage Azure pour les données « froides » qui sont utilisées moins fréquemment.

- [Guide pratique pour collecter des journaux et des métriques de plateforme avec Azure Monitor](../azure-monitor/platform/diagnostic-settings.md) 

- [Guide pratique pour intégrer Azure Sentinel](../sentinel/quickstart-onboard.md)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

### <a name="lt-6-configure-log-storage-retention"></a>LT-6 : Configurer la rétention du stockage des journaux

**Conseils** : Veillez à ce que la période de rétention des journaux définie dans les comptes de stockage ou les espaces de travail Log Analytics utilisés pour le stockage des journaux App Configuration soit conforme aux obligations réglementaires de votre organisation. Utilisez des comptes Stockage Azure, Data Lake ou d’espace de travail Log Analytics pour le stockage à long terme et l’archivage.

Dans Azure Monitor, vous pouvez définir la période de rétention de votre espace de travail Log Analytics en fonction des règles de conformité de votre organisation.

- [Comment configurer la période de conservation d’un espace de travail Log Analytics](../azure-monitor/platform/manage-cost-storage.md)

- [Stockage des journaux des ressources dans un compte de stockage Azure](../azure-monitor/platform/resource-logs.md#send-to-azure-storage)

**Supervision d’Azure Security Center** : Non applicable

**Responsabilité** : Customer

## <a name="incident-response"></a>Réponse aux incidents

*Pour plus d’informations, consultez [Benchmark de sécurité Azure : réponse aux incidents](../security/benchmarks/security-controls-v2-incident-response.md).*

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

Azure Security Center attribue un niveau de gravité à chaque alerte pour vous aider à hiérarchiser celles devant être examinées en premier. La gravité dépend du niveau de confiance accordé par Security Center à la découverte ou à l’analytique ayant servi à émettre l’alerte, ainsi que du niveau de confiance quant à l’existence d’une intention malveillante sous-jacente dans l’activité à l’origine de l’alerte.

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

### <a name="pv-1-establish-secure-configurations-for-azure-services"></a>PV-1 : Établir des configurations sécurisées pour les services Azure 

**Conseils** : Azure App Configuration prend en charge les stratégies propres aux services ci-dessous, disponibles dans Azure Security Center pour auditer et appliquer les configurations de vos ressources Azure. Cet aspect peut être configuré dans les initiatives Azure Security Center ou Azure Policy.
- App Configuration doit utiliser une clé gérée par le client : Les clés gérées par le client offrent une protection améliorée des données en vous permettant de gérer vos clés de chiffrement. Cela est souvent nécessaire pour répondre aux exigences de conformité.
- App Configuration doit utiliser une liaison privée : Les connexions de points de terminaison privés permettent aux clients d’un réseau virtuel d’accéder de manière sécurisée à Azure App Configuration par le biais d’une liaison privée.

Vous pouvez utiliser Azure Blueprints pour automatiser le déploiement et la configuration de services et d’environnements d’application (par exemple des modèles Azure Resource Manager, des contrôles Azure RBAC et des stratégies) dans une même définition de blueprint.

- [En savoir plus sur les stratégies App Configuration](../governance/policy/samples/built-in-policies.md#app-configuration)

- [Utiliser des stratégies de sécurité dans Azure Security Center](../security-center/tutorial-security-policy.md)

- [Illustration de l’implémentation de GuardRails dans une zone d’atterrissage à l’échelle de l’entreprise](/azure/cloud-adoption-framework/ready/enterprise-scale/architecture#landing-zone-expanded-definition)

- [Tutoriel : Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

- [Azure Blueprints](../governance/blueprints/overview.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

### <a name="pv-2-sustain-secure-configurations-for-azure-services"></a>PV-2 : Supporter des configurations sécurisées pour les services Azure

**Conseils** : Utilisez Azure Security Center pour superviser votre base de référence de configuration et appliquer l’utilisation d’Azure Policy. Azure Policy pour App Configuration comprend les éléments suivants : 
- App Configuration doit utiliser une clé gérée par le client : Les clés gérées par le client offrent une protection améliorée des données en vous permettant de gérer vos clés de chiffrement. Cela est souvent nécessaire pour répondre aux exigences de conformité.
- App Configuration doit utiliser une liaison privée : Les connexions de points de terminaison privés permettent aux clients d’un réseau virtuel d’accéder de manière sécurisée à Azure App Configuration par le biais d’une liaison privée.

- [Comprendre les effets d'Azure Policy](../governance/policy/concepts/effects.md) 

- [Créer et gérer des stratégies pour assurer la conformité](../governance/policy/tutorials/create-and-manage.md)

**Supervision d’Azure Security Center** : Oui

**Responsabilité** : Partagé

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

- Consultez [Vue d’ensemble d’Azure Security Benchmark V2](../security/benchmarks/overview.md)
- En savoir plus sur les [bases de référence de la sécurité Azure](../security/benchmarks/security-baselines-overview.md)