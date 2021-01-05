---
title: Bonnes pratiques pour Azure AD B2C
titleSuffix: Azure AD B2C
description: Suggestions et bonnes pratiques à prendre en compte lors de l’utilisation d’Azure Active Directory B2C (Azure AD B2C).
services: active-directory-b2c
author: vigunase
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 06/06/2020
ms.author: vigunase
ms.subservice: B2C
ms.openlocfilehash: 1c3c3d38ac0d8334f70f681d8ef86c0d6f86ecfa
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96750218"
---
# <a name="recommendations-and-best-practices-for-azure-active-directory-b2c"></a>Suggestions et bonnes pratiques pour Azure Active Directory B2C

Les bonnes pratiques et suggestions suivantes couvrent certains aspects principaux de l’intégration d’Azure Active Directory (Azure AD) B2C dans des environnements d’application existants ou nouveaux.

## <a name="fundamentals"></a>Notions de base

| Bonne pratique | Description |
|--|--|
| Choisir des flux d’utilisateurs pour la plupart des scénarios | L’infrastructure d’expérience d’identité IEF (Identity Experience Framework) d’Azure AD B2C constitue le point fort du service. Les stratégies décrivent entièrement les expériences relatives à l’identité, comme l’inscription, la connexion ou la modification de profil. Pour vous aider à configurer les tâches d’identité les plus courantes, le portail Azure AD B2C inclut des stratégies configurables prédéfinies, appelées flux d’utilisateurs. Les flux d’utilisateurs vous permettent de créer des expériences utilisateur exceptionnelles en quelques minutes, en quelques clics seulement. [Apprenez quand utiliser des flux d’utilisateurs et des stratégies personnalisées](custom-policy-overview.md#comparing-user-flows-and-custom-policies).|
| Inscriptions des applications | Toute application (web, native) ou API sécurisée doit être inscrite dans Azure AD B2C. Si une application a une version web et une version native d’iOS et d’Android, vous pouvez les inscrire comme une seule application dans Azure AD B2C avec le même ID client. Découvrez comment [inscrire des applications OIDC, SAML, web et natives](./tutorial-register-applications.md?tabs=applications). Apprenez-en davantage sur les [types d’application pouvant être utilisés dans Azure AD B2C](./application-types.md). |
| Passer à une facturation pour les utilisateurs actifs mensuels | Azure AD B2C a évolué d’une facturation des authentifications actives mensuelles à la facturation des utilisateurs actifs mensuels. La plupart des clients trouveront ce modèle économique. [Apprenez-en davantage sur la facturation des utilisateurs actifs mensuels](https://azure.microsoft.com/updates/mau-billing/). [Enregistrer ce lien](b2clogin.md) |

## <a name="planning-and-design"></a>Planification et conception

Définissez l’architecture de vos applications et services, inventoriez les systèmes actuels et planifiez votre migration vers Azure AD B2C.

| Bonne pratique | Description |
|--|--|
| Créer l’architecture d’une solution de bout en bout | Incluez toutes les dépendances de vos applications lors de la planification d’une intégration d’Azure AD B2C. Tenez compte de tous les services et produits figurant actuellement dans votre environnement ou qui peuvent avoir besoin d’être ajoutés à la solution, tels qu’Azure Functions, les systèmes de gestion de la relation client (CRM), la passerelle de gestion des API Azure et les services de stockage. Prenez en compte la sécurité et la scalabilité pour tous les services. |
| Documenter les expériences de vos utilisateurs | Détaillez tous les parcours utilisateur que vos clients peuvent rencontrer dans votre application. Incluez tous les écrans et tous les flux de branchement qu’ils peuvent rencontrer en interagissant avec les aspects d’identité et de profil de votre application. Incluez la facilité d’utilisation, l’accessibilité et la localisation dans votre planification. |
| Choisir le protocole d’authentification approprié |  Pour une répartition des différents scénarios d’application et de leurs flux d’authentification recommandés, consultez [Scénarios et flux d’authentification pris en charge](../active-directory/develop/authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows). |
| Piloter une expérience utilisateur de bout en bout de type preuve de concept (POC) | Commencez avec nos [exemples de code Microsoft](code-samples.md) et nos [exemples de la communauté](https://github.com/azure-ad-b2c/samples). |
| Créer un plan de migration |Une planification anticipée peut rendre la migration plus fluide. Apprenez-en davantage sur la [migration utilisateur](user-migration.md).|
| Facilité d’utilisation et sécurité | Votre solution doit bien équilibrer la facilité d’utilisation des applications et le niveau de risque que votre organisation peut accepter. |
| Déplacer les dépendances locales vers le cloud | Pour garantir une solution résiliente, envisagez de déplacer les dépendances des applications existantes vers le cloud. |
| Migrer les applications existantes vers b2clogin.com | La dépréciation de login.microsoftonline.com prendra effet pour tous les locataires Azure AD B2C le 4 décembre 2020. [Plus d’informations](b2clogin.md) |
| Utiliser Identity Protection et l’accès conditionnel | Utilisez ces capacités pour mieux contrôler les authentifications risquées et les stratégies d’accès. Azure AD B2C Premium P2 est requis. [Plus d’informations](conditional-access-identity-protection-overview.md) |

## <a name="implementation"></a>Implémentation

Au cours de la phase d’implémentation, tenez compte des suggestions suivantes.

| Bonne pratique | Description |
|--|--|
| Modifier les stratégies personnalisées avec l’extension Azure AD B2C pour Visual Studio Code | Téléchargez Visual Studio Code et cette extension générée par la communauté [à partir de la Place de marché Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=AzureADB2CTools.aadb2c). Bien qu’il ne s’agisse pas d’un produit Microsoft officiel, l’extension Azure AD B2C pour Visual Studio Code comprend plusieurs fonctionnalités qui facilitent l’utilisation de stratégies personnalisées. |
| En savoir plus sur la résolution des problèmes liés à Azure AD B2C | Découvrez comment [résoudre les problèmes de stratégies personnalisées](./troubleshoot-custom-policies.md?tabs=applications) au cours du développement. Découvrez à quoi ressemble un flux d’authentification normal et utilisez des outils pour découvrir les anomalies et les erreurs. Par exemple, utilisez [Application Insights](troubleshoot-with-application-insights.md) pour passer en revue les journaux de sortie des parcours utilisateur. |
| Tirer parti de notre bibliothèque de modèles de stratégie personnalisés éprouvés | Recherchez des [exemples](https://github.com/azure-ad-b2c/samples) pour plusieurs parcours utilisateur de gestion des accès et des identités des clients Azure AD B2C améliorés. |

## <a name="testing"></a>Test

Testez et automatisez votre implémentation d’Azure AD B2C.

| Bonne pratique | Description |
|--|--|
| Compte pour trafic global | Utilisez des sources de trafic provenant de différentes adresses globales pour tester les exigences en matière de performances et de localisation. Assurez-vous que tous les éléments HTML, les feuilles de style en cascade et les dépendances peuvent répondre à vos besoins en matière de performances. |
| Tests fonctionnels et de l’interface utilisateur | Testez les flux d’utilisateurs de bout en bout. Ajoutez des tests synthétiques à quelques minutes d’intervalle avec les tests web VS, Selenium, etc. |
| Test du stylet | Avant de mettre en ligne votre solution, effectuez des exercices de test de pénétration pour vérifier que tous les composants sont sécurisés, y compris les dépendances tierces. Vérifiez que vous avez sécurisé vos API avec des jetons d’accès et utilisé le protocole d’authentification adapté à votre scénario d’application. Apprenez-en davantage sur les [test de pénétration](../security/fundamentals/pen-testing.md) et les [règles d’engagement pour les tests de pénétration du cloud Microsoft](https://www.microsoft.com/msrc/pentest-rules-of-engagement?rtc=1). |
| Test A/B | Soumettez vos nouvelles fonctionnalités avec un petit nombre d’utilisateurs aléatoires avant de procéder au déploiement sur l’ensemble de votre population. JavaScript étant activé dans Azure AD B2C, vous pouvez intégrer avec des outils de test A/B tels qu’Optimizely, Clarity et autres. |
| Test de charge | Azure AD B2C peut être mis à l’échelle, mais votre application peut être mise à l’échelle uniquement si toutes ses dépendances peuvent l’être aussi. Effectuez un test de charge de vos API et CDN. |
| Limitation |  Azure AD B2C limite le trafic si trop de requêtes sont envoyées à partir de la même source dans un laps de temps réduit. Utilisez plusieurs sources de trafic pendant le test de charge et gérez de manière appropriée le code d’erreur `AADB2C90229` dans vos applications. |
| Automatisation | Utilisez des pipelines d’intégration et de livraison continues (CI/CD) pour automatiser les tests et les déploiements, par exemple, [Azure DevOps](deploy-custom-policies-devops.md). |

## <a name="operations"></a>Opérations

Gérez votre environnement Azure AD B2C.

| Bonne pratique | Description |
|--|--|
| Créer plusieurs environnements | Pour faciliter le déploiement et le fonctionnement, créez des environnements distincts pour le développement, les tests, la pré-production et la production. Créez des locataires Azure AD B2C pour chacun d’eux. |
| Utiliser la gestion de version pour vos stratégies personnalisées | Envisagez d’utiliser GitHub, Azure Repos ou un autre système de gestion de versions basé sur le cloud pour vos stratégies personnalisées Azure AD B2C. |
| Utiliser l’API Microsoft Graph pour automatiser la gestion de vos locataires B2C | API Microsoft Graph :<br/>Gérer [Identity Experience Framework](/graph/api/resources/trustframeworkpolicy?preserve-view=true&view=graph-rest-beta) (stratégies personnalisées)<br/>[Clés](/graph/api/resources/trustframeworkkeyset?preserve-view=true&view=graph-rest-beta)<br/>[Flux d’utilisateurs](/graph/api/resources/identityuserflow?preserve-view=true&view=graph-rest-beta) |
| Intégrer à Azure DevOps | Un [pipeline CI/CD](deploy-custom-policies-devops.md) permet de déplacer facilement du code entre différents environnements et garantit la préparation de la production à tout moment.   |
| Intégrer à Azure Monitor | Les [événements des journaux d’audit](view-audit-logs.md) sont uniquement conservés pendant sept jours. [Intégrez à Azure Monitor](azure-monitor.md) pour conserver les journaux pour les utiliser sur le long terme ou les intégrer à des outils de gestion d’événements et d’informations de sécurité tiers pour obtenir des insights sur votre environnement. |
| Alertes et supervision actives de la configuration | [Suivez le comportement des utilisateurs](./analytics-with-application-insights.md) dans Azure AD B2C à l’aide d’Application Insights. |

## <a name="support-and-status-updates"></a>Mises à jour de support et d’état
## <a name="todays-support-and-status-updates"></a>Mises à jour actuelles de support et d’état
Restez à jour avec l’état du service et recherchez des options de support.

| Bonne pratique | Description |
|--|--|
| [Mises à jour de service](https://azure.microsoft.com/updates/?product=active-directory-b2c) |  Restez informé des mises à jour et des annonces du produit Azure AD B2C. |
| [Support Microsoft](support-options.md) | Soumettez une demande de support pour des problèmes techniques avec Azure AD B2C. La gestion de la facturation et des abonnements est fournie gratuitement. |
| [Statut Azure](https://status.azure.com/status) | Affichez l’état d’intégrité actuel de tous les services Azure. |