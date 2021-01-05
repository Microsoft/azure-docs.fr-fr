---
title: 'Benchmark de sécurité Azure v2 : Gestion des identités'
description: Benchmark de sécurité Azure v2, gestion des identités
author: msmbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 09/20/2020
ms.author: mbaldwin
ms.custom: security-benchmark
ms.openlocfilehash: 33f5dff65fa7ad8274051f784f2e61dc8366d389
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368849"
---
# <a name="security-control-v2-identity-management"></a>Contrôle de sécurité V2 : Gestion des identités

La gestion des identités couvre les contrôles permettant d’établir une identité et des contrôles d’accès sécurisés à l’aide d’Azure Active Directory. Cela comprend l’utilisation de l’authentification unique, des authentifications fortes, des identités gérées (et des principes de service) pour les applications, l’accès conditionnel et la surveillance des anomalies de compte.

## <a name="im-1-standardize-azure-active-directory-as-the-central-identity-and-authentication-system"></a>IM-1 : Normaliser Azure Active Directory comme système d’authentification et d’identité central

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| IM-1 | 16.1, 16.2, 16.4, 16.5 | IA-2, IA-8, AC-2, AC-3 |

Azure Active Directory (Azure AD) est le service centralisé de gestion des identités et des accès par défaut d’Azure. Vous devez normaliser Azure AD pour régir la gestion des identités et des accès de votre organisation dans :
- Les ressources cloud Microsoft, comme le Portail Azure, le stockage Azure, les machines virtuelles Azure (Linux et Windows), les applications Azure Key Vault, PaaS et SaaS.

- Les ressources de votre organisation, comme les applications sur Azure ou les ressources réseau de votre entreprise.

La sécurisation d’Azure AD doit être d’une priorité élevée dans les pratiques de sécurité cloud de votre organisation. Azure AD fournit un score d'identité sécurisée pour vous aider à évaluer votre posture de sécurité des identités par rapport aux recommandations de Microsoft en matière de meilleures pratiques. Utilisez le score pour évaluer avec précision votre configuration par rapport aux meilleures pratiques recommandées et apporter des améliorations à votre posture de sécurité.

Remarque : Azure AD prend en charge les fournisseurs d’identité externes, qui permettent aux utilisateurs sans compte Microsoft de se connecter à leurs applications et ressources avec leur identité externe.

- [Location dans Azure AD](../../active-directory/develop/single-and-multi-tenant-apps.md)

- [Création et configuration d’une instance Azure AD](../../active-directory/fundamentals/active-directory-access-create-new-tenant.md)

- [Définir des locataires Azure AD](https://azure.microsoft.com/resources/securing-azure-environments-with-azure-active-directory/)  

- [Utiliser des fournisseurs d’identité externes pour une application](../../active-directory/external-identities/identity-providers.md)

- [Qu’est-ce que le degré de sécurisation Identity Secure Score dans Azure AD](../../active-directory/fundamentals/identity-secure-score.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys) 

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

## <a name="im-2-manage-application-identities-securely-and-automatically"></a>IM-2 : Gérer les identités d’application de façon sécurisée et automatique

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| IM-2 | N/A | AC-2, AC-3, IA-2, IA-4, IA-9 |

Pour les comptes non humains, tels que les services ou l’automatisation, utilisez les identités gérées Azure au lieu de créer un compte humain plus puissant pour accéder aux ressources ou exécuter du code. Les identités gérées par Azure peuvent s’authentifier auprès des services et ressources Azure qui prennent en charge l’authentification Azure AD. L’authentification est activée à l’aide de règles d’octroi d’accès prédéfinies, évitant les informations d’identification codées en dur dans le code source ou les fichiers de configuration. 

Pour les services qui ne prennent pas en charge les identités gérées, utilisez Azure AD pour créer un principal de service avec des autorisations restreintes au niveau de la ressource à la place.  Il est recommandé de configurer des principaux de service avec des informations d’identification de certificat et de se replier sur les secrets des clients. Dans les deux cas, Azure Key Vault peut être utilisé conjointement avec des identités gérées par Azure, afin que l’environnement d’exécution (par exemple, une fonction Azure) puisse récupérer les informations d’identification du coffre de clés.

- [Identités managées Azure](../../active-directory/managed-identities-azure-resources/overview.md)

- [Services prenant en charge les identités managées pour les ressources Azure](../../active-directory/managed-identities-azure-resources/services-support-managed-identities.md)

- [Principal de service Azure](/powershell/azure/create-azure-service-principal-azureps)

- [Créer un principal du service avec un certificat](../../active-directory/develop/howto-authenticate-service-principal-powershell.md)

Utiliser Azure Key Vault pour l’inscription d’un principal de sécurité : authentication#authorize-a-security-principal-to-access-key-vault

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="im-3-use-azure-ad-single-sign-on-sso-for-application-access"></a>IM-3 : Utiliser l’authentification unique Azure AD pour l’accès aux applications

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| IM-3 | 4.4 | IA-2, IA-4 |

Azure AD fournit la gestion des identités et des accès aux ressources Azure, aux applications cloud et aux applications locales. La gestion des identités et des accès s’applique aux identités d’entreprise, comme les employés, ainsi qu’aux identités externes, comme les partenaires et les fournisseurs.

Utilisez l’authentification unique Azure AD pour gérer et sécuriser l’accès aux données et ressources de votre organisation localement et dans le cloud. Connectez l’ensemble de vos utilisateurs, applications et appareils à Azure AD pour un accès transparent et sécurisé, ainsi qu’une visibilité et un contrôle accrus. 

- [Comprendre l’authentification unique d’application avec Azure AD](../../active-directory/manage-apps/what-is-single-sign-on.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="im-4-use-strong-authentication-controls-for-all-azure-active-directory-based-access"></a>IM-4 : Utiliser des contrôles d’authentification renforcés pour tous les accès basés sur Azure Active Directory

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| IM-4 | 4.2, 4.4 4.5, 11.5, 12.11, 16.3 | AC-2, AC-3, IA-2, IA-4 |

Azure AD prend en charge des contrôles d’authentification renforcés via l’authentification multifacteur (MFA) et des méthodes de mot de passe fort.  
- Authentification multifacteur : Activez Azure AD MFA et suivez les recommandations liées à la gestion des identités et des accès dans Azure Security Center pour votre configuration MFA. L’authentification multifacteur peut être appliquée à tous les utilisateurs, seulement certains utilisateurs ou au niveau de chaque utilisateur en fonction des conditions de connexion et des facteurs de risque. 

- Authentification sans mot de passe : Trois options d’authentification sans mot de passe sont proposées : Windows Hello Entreprise, l’application Microsoft Authenticator et les méthodes d’authentification locales, comme les cartes à puce. 

Pour les utilisateurs administrateurs et privilégiés, assurez-vous que le niveau le plus élevé de méthode d’authentification forte est utilisé, puis déployez la stratégie d’authentification forte appropriée pour les autres utilisateurs.

Si l’authentification par mot de passe héritée est toujours utilisée pour l’authentification Azure AD, sachez que les comptes cloud uniquement (comptes d’utilisateur créés directement dans Azure) ont une stratégie de mot de passe de base par défaut. Et les comptes hybrides (comptes d’utilisateur provenant d’Active Directory locaux) suivent les stratégies de mot de passe locales. Lors de l’utilisation de l’authentification par mot de passe, Azure AD fournit une fonctionnalité de protection par mot de passe qui empêche les utilisateurs de définir des mots de passe faciles à deviner. Microsoft fournit une liste globale de mots de passe interdits mise à jour en fonction des données de télémétrie, et les clients peuvent compléter la liste en fonction de leurs besoins (par exemple, noms de marque, références culturelles, etc.). Cette protection par mot de passe peut être utilisée pour les comptes cloud et hybrides uniquement. 

Remarque : La seule authentification basée sur les informations de mot de passe est vulnérable aux méthodes d’attaque populaires. Pour une sécurité accrue, utilisez une authentification forte telle que l’authentification multifacteur et une stratégie de mot de passe forte. Pour les applications tierces et les services de la Place de marché qui peuvent avoir des mots de passe par défaut, vous devez les modifier lors de la configuration initiale du service. 

- [Guide pratique pour activer l’authentification MFA dans Azure](../../active-directory/authentication/howto-mfa-getstarted.md)

- [Introduction aux options d’authentification sans mot de passe pour Azure Active Directory](../../active-directory/authentication/concept-authentication-passwordless.md)

- [Stratégie de mot de passe par défaut d’Azure AD](../../active-directory/authentication/concept-sspr-policy.md#password-policies-that-only-apply-to-cloud-user-accounts)

- [Éliminer les mauvais mots de passe à l’aide de Protection de mots de passe d’Azure AD](../../active-directory/authentication/concept-password-ban-bad.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

## <a name="im-5-monitor-and-alert-on-account-anomalies"></a>IM-5 : Surveiller et alerter en cas d’anomalies de compte

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| IM-5 | 4.8, 4.9, 16.12, 16.13 | AC-2, AC-3, AC-7, AU-6 |

Azure AD fournit les sources de données suivantes : 
-   Connexions – le rapport de connexions fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs.

-   Journaux d’audit : traçabilité proposée via des journaux d’activité pour toutes les modifications effectuées par diverses fonctionnalités au sein d’Azure AD. Les exemples de modifications journalisées dans les journaux d’audit incluent l’ajout ou la suppression d’utilisateurs, d’applications, de groupes, de rôles et de stratégies.

-   Connexions risquées : une connexion risquée est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur.

-   Utilisateurs avec indicateur de risque : un utilisateur à risque correspond à un indicateur de compte d’utilisateur susceptible d’être compromis.

Ces sources de données peuvent être intégrées à Azure Monitor, Azure Sentinel ou des systèmes SIEM tiers.

Azure Security Center pouvez également alerter en cas de certaines activités suspectes, comme un nombre excessif de tentatives d’authentification ayant échoué et la présence de comptes dépréciés dans l’abonnement. 

Azure Advanced Threat Protection (ATP) est une solution de sécurité qui peut utiliser vos signaux Active Directory locaux pour identifier, détecter et examiner les menaces avancées, les identités compromises et les actions malveillantes internes.

- [Rapports d’activité d’audit dans Azure AD](../../active-directory/reports-monitoring/concept-audit-logs.md)

- [Guide pratique pour afficher les connexions risquées Azure AD](../../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour identifier les utilisateurs Azure AD pour lesquels une activité à risque a été signalée](../../active-directory/identity-protection/overview-identity-protection.md)

- [Guide pratique pour superviser l’activité liée aux identités et aux accès des utilisateurs dans Azure Security Center](../../security-center/security-center-identity-access.md)

- [Alertes dans le module de protection de renseignement sur les menaces d’Azure Security Center](../../security-center/alerts-reference.md)

- [Guide pratique pour intégrer des journaux d’activité Azure dans Azure Monitor](../../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md)

- [Connecter des données depuis Azure AD Identity Protection](../../sentinel/connect-azure-ad-identity-protection.md)

- [Azure Advanced Threat Protection](/azure-advanced-threat-protection/what-is-atp)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

## <a name="im-6-restrict-azure-resource-access-based-on-conditions"></a>IM-6 : Restreindre l’accès aux ressources Azure en fonction des conditions

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| IM-6 | N/A | AC-2, AC-3 |

Utilisez l’accès conditionnel Azure AD pour un contrôle d’accès plus granulaire en fonction des conditions définies par l’utilisateur, comme la nécessité de connecter des utilisateurs à partir de certaines plages d’adresses IP pour utiliser l’authentification multifacteur. Une gestion granulaire des sessions d’authentification peut également être utilisée via une stratégie d’accès conditionnel Azure AD pour différents cas d’usage. 

- [Présentation de l’accès conditionnel Azure](../../active-directory/conditional-access/overview.md)

- [Stratégies d’accès conditionnel courantes](../../active-directory/conditional-access/concept-conditional-access-policy-common.md)

- [Configurer la gestion de session d’authentification avec l’accès conditionnel](../../active-directory/conditional-access/howto-conditional-access-session-lifetime.md)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Gestion des identités et des clés](/azure/cloud-adoption-framework/organize/cloud-security-identity-keys)

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

- [Renseignement sur les menaces](/azure/cloud-adoption-framework/organize/cloud-security-threat-intelligence)

## <a name="im-7-eliminate-unintended-credential-exposure"></a>IM-7 : Éliminer l’exposition involontaire des informations d’identification

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| IM-7 | 18.1, 18.7 | IA-5 |

Implémenter le moteur d’analyse des informations d’identification Azure DevOps pour identifier les informations d’identification dans le code. Le moteur d’analyse des informations d’identification encourage également le déplacement des informations d’identification découvertes vers des emplacements plus sécurisés, tels qu’Azure Key Vault.

Pour GitHub, vous pouvez utiliser la fonctionnalité native d’analyse de secret pour identifier les informations d’identification ou d’autres formes de secrets dans le code.

- [Configurer Credential Scanner](https://secdevtools.azurewebsites.net/helpcredscan.html)

- [Analyse du secret GitHub](https://docs.github.com/github/administering-a-repository/about-secret-scanning)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)

- [Gestion de la posture](/azure/cloud-adoption-framework/organize/cloud-security-posture-management)

## <a name="im-8-secure-user-access-to-legacy-applications"></a>IM-8 : Sécuriser l’accès des utilisateurs aux applications héritées

| Identifiant Azure | ID des contrôles CIS v7.1 | ID NIST SP 800-53 r4 |
|--|--|--|--|
| IM-8 | 14.6 | AC-2, AC-3, SC-11 |

Vérifiez que vous disposez de contrôles d’accès et d’une surveillance de session modernes pour les applications héritées et les données qu’elles stockent et traitent. Bien que les VPN soient couramment utilisés pour accéder aux applications héritées, elles ont souvent uniquement un contrôle d’accès de base et une surveillance limitée des sessions.

Le proxy d’application Azure AD vous permet de publier des applications locales héritées sur des utilisateurs distants avec l’authentification unique tout en validant de manière explicite la fiabilité des utilisateurs et des appareils distants avec l’accès conditionnel Azure AD. 

Microsoft Cloud App Security est également un service CASB (Cloud Access Security Broker) qui peut fournir des contrôles pour surveiller les sessions d’application d’un utilisateur et les actions bloquantes (pour les applications locales héritées et les applications cloud Software as a Service (SaaS)). 

- [Proxy d’application Azure AD](../../active-directory/manage-apps/application-proxy.md#what-is-application-proxy)

- [Meilleures pratiques avec Microsoft Cloud App Security](/cloud-app-security/best-practices)

**Responsabilité** : Customer

**Parties prenantes de la sécurité des clients** ([En savoir plus](/azure/cloud-adoption-framework/organize/cloud-security#security-functions)) :

- [Architecture de la sécurité](/azure/cloud-adoption-framework/organize/cloud-security-architecture) 

- [Sécurité d’infrastructure et de point de terminaison](/azure/cloud-adoption-framework/organize/cloud-security-architecture)

- [Sécurité des applications et DevSecOps](/azure/cloud-adoption-framework/organize/cloud-security-application-security-devsecops)