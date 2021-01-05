---
title: Planifier le déploiement de votre appareil Azure Active Directory
description: Choisissez les stratégies d’intégration des appareils Azure AD qui répondent aux besoins de votre organisation.
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: how-to
ms.date: 06/15/2020
ms.author: baselden
author: BarbaraSelden
manager: daveba
ms.reviewer: sandeo
ms.collection: M365-identity-device-management
ms.openlocfilehash: bf30570f10f176c47aed0f99127e03a027eff775
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92093087"
---
# <a name="plan-your-azure-active-directory-device-deployment"></a>Planifier le déploiement de votre appareil Azure Active Directory

Cet article vous aide à évaluer les méthodes d’intégration de votre appareil avec Azure AD, à choisir le plan d’implémentation et fournit des liens clés vers les outils de gestion des appareils pris en charge.

La gamme des appareils à partir desquels les utilisateurs se connectent s’étend. Les organisations peuvent fournir des ordinateurs de bureau, des ordinateurs portables, des téléphones, des tablettes et d’autres appareils. Vos utilisateurs peuvent apporter leurs propres appareils et accéder à des informations à partir d’emplacements variés. Dans cet environnement, votre travail en tant qu’administrateur consiste à sécuriser vos ressources organisationnelles sur tous les appareils.

Azure Active Directory (Azure AD) permet à votre organisation de répondre à ces objectifs avec la gestion des identités des appareils. Vous pouvez maintenant intégrer vos appareils à Azure AD et les contrôler à partir d’un emplacement central dans le [portail Azure](https://portal.azure.com/). Vous bénéficiez ainsi d’une expérience unifiée, d’une sécurité renforcée et d’une réduction du temps nécessaire à la configuration d’un nouvel appareil.

Il existe plusieurs méthodes pour intégrer vos appareils à Azure AD :

* Vous pouvez [inscrire des appareils](concept-azure-ad-register.md) auprès d’Azure AD

* [Joindre des appareils](concept-azure-ad-join.md) à Azure AD (cloud seulement) ou

* [Créer une jonction Azure AD Hybride](concept-azure-ad-join-hybrid.md) entre des appareils de votre Active Directory local et Azure AD. 

## <a name="learn"></a>Découvrir

Avant de commencer, assurez-vous que vous êtes familiarisé avec la [vue d’ensemble de la gestion des identités des appareils](overview.md).

### <a name="benefits"></a>Avantages

Voici les principaux avantages d’attribuer à vos appareils une identité Azure AD :

* Augmenter la productivité : avec Azure AD, vos utilisateurs peuvent effectuer une [authentification unique fluide](./azuread-join-sso.md) à vos ressources locales et cloud, ce qui leur permet d’être productifs où qu’ils se trouvent.

* Renforcer la sécurité : les appareils Azure AD vous permettent d’appliquer des [stratégies d’accès conditionnel](../conditional-access/require-managed-devices.md) aux ressources en fonction de l’identité de l’appareil ou de l’utilisateur. Les stratégies d’accès conditionnel peuvent offrir une protection supplémentaire à l’aide [d’Azure AD Identity Protection](../identity-protection/overview-identity-protection.md). La jonction d’un appareil à Azure AD est un prérequis au renforcement de votre sécurité avec une stratégie [d’authentification sans mot de passe](../authentication/concept-authentication-passwordless.md).

* Améliorer l’expérience utilisateur : avec les identités des appareils dans Azure AD, vous pouvez fournir à vos utilisateurs un accès facile aux ressources cloud de votre organisation à partir d’appareils personnels et d’entreprise. Les administrateurs peuvent activer [Enterprise State Roaming](enterprise-state-roaming-overview.md) pour une expérience unifiée sur tous les appareils Windows.

* Simplifier le déploiement et la gestion : la gestion des identités des appareils simplifie le processus d’intégration d’appareils à Azure AD avec [Windows Autopilot](/windows/deployment/windows-autopilot/windows-10-autopilot), le [provisionnement en bloc](/mem/intune/enrollment/windows-bulk-enroll) et le [libre-service : OOBE (out-of-box experience)](../user-help/user-help-join-device-on-network.md). Vous pouvez gérer ces appareils avec des outils de gestion des périphériques mobiles, comme [Microsoft Intune](/mem/intune/fundamentals/what-is-intune) et leurs identités dans le [portail Azure](https://portal.azure.com/).

### <a name="training-resources"></a>Ressources de formation

Vidéo :  [Accès conditionnel et les contrôles d’appareil](https://youtu.be/NcONUf-jeS4)

Questions fréquentes (FAQ) : [FAQ sur la gestion des appareils Azure AD](faq.md) et [FAQ sur l’itinérance des paramètres et des données](enterprise-state-roaming-faqs.md) 

## <a name="plan-the-deployment-project"></a>Planifier le projet de déploiement

Tenez compte des besoins de votre organisation lorsque vous déterminez la stratégie de ce déploiement dans votre environnement.

### <a name="engage-the-right-stakeholders"></a>Impliquer les parties prenantes appropriées

Lorsque des projets technologiques échouent, cela est généralement dû à des attentes qui ne correspondent pas à l’impact, aux résultats et aux responsabilités réels. Pour éviter ces écueils, [assurez-vous de faire appel aux bonnes personnes](../fundamentals/active-directory-deployment-plans.md), et que les rôles des parties prenantes soient bien compris. 

Pour ce plan, ajoutez les parties prenantes suivantes à votre liste :

| Role| Description |
| - | - |
| Administrateur de l’appareil| Représentant de l’équipe en charge des appareils qui peut vérifier que le plan répondra aux exigences de votre organisation en termes d’appareils. |
| Administrateur réseau| Représentant de l’équipe en charge du réseau qui peut s’assurer que les exigences réseau sont respectées. |
| Équipe de gestion des appareils| Équipe qui gère l’inventaire des appareils. |
| Équipes d’administration spécifiques au système d’exploitation| Équipes qui prennent en charge et gèrent des versions de système d’exploitation spécifiques. Par exemple, il peut y avoir une équipe dédiée à Mac ou à iOS. |

### <a name="plan-communications"></a>Planifier les communications

La communication est essentielle à la réussite de tout nouveau service. Communiquez de manière proactive avec vos utilisateurs sur ce qui va changer, à quel moment les changements seront appliqués et comment ils peuvent obtenir de l’aide en cas de problème.

### <a name="plan-a-pilot"></a>Prévoir un pilote

Nous vous recommandons de procéder à la configuration initiale de votre méthode d’intégration dans un environnement de test ou avec un petit groupe d’appareils de test. Consultez [Meilleures pratiques pour un pilote](../fundamentals/active-directory-deployment-plans.md).

Le déploiement de jonction Azure AD Hybride est simple et il s’agit d’une tâche entièrement réservée à l’administrateur sans intervention de l’utilisateur final nécessaire. Vous pouvez [effectuer une validation contrôlée de la jonction Azure AD Hybride](hybrid-azuread-join-control.md) avant de l’activer dans toute l’organisation en une fois.

## <a name="choose-your-integration-methods"></a>Choisir vos méthodes d’intégration

Votre organisation peut utiliser plusieurs méthodes d’intégration d’appareils dans un seul locataire Azure AD. L’objectif est de choisir la ou les méthodes appropriées pour que vos appareils soient managés en toute sécurité dans Azure AD. Il existe de nombreux paramètres à l’origine de cette décision, notamment la propriété, les types d’appareils, les audiences principales et l’infrastructure de votre organisation.

Les informations suivantes peuvent vous aider à choisir les méthodes d’intégration à utiliser.

### <a name="decision-tree-for-devices-integration"></a>Arbre de décision pour l’intégration des appareils

Utilisez cet arbre afin de déterminer les options disponibles pour les appareils appartenant à l’organisation. 

> [!NOTE]
> Les scénarios personnels ou BYOD (Bring-Your-Own Device) ne sont pas décrits dans ce diagramme. Ils entraînent toujours l’inscription à Azure AD.

 ![Arbre de décision](./media/plan-device-deployment/flowchart.png)

### <a name="comparison-matrix"></a>Matrice de comparaison

Les appareils iOS et Android peuvent uniquement être inscrits auprès d’Azure AD. Le tableau suivant présente les considérations de haut niveau pour les appareils clients Windows. Utilisez-le comme vue d’ensemble, puis explorez les différentes méthodes d’intégration en détail.

| Considération | Appareils inscrits sur Azure AD| jointure Azure AD ;| jointure Azure AD hybride ; |
| - | - | - | - |
| **Systèmes d’exploitation clients**| | |  |
| Appareils Windows 10| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |
| Appareils utilisant une version antérieure de Windows (Windows 8.1 ou Windows 7)| | | ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |
|**Options de connexion**| | |  |
| Informations d’identification locales de l’utilisateur final| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| |  |
| Mot de passe| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |
| Code PIN d’appareil| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| |  |
| Windows Hello| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| |  |
| Windows Hello Entreprise| | ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |
| Clés de sécurité FIDO 2.0| | ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |
| Application Microsoft Authenticator (sans mot de passe)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |
|**Fonctionnalités clés**| | |  |
| Ressources de l’authentification unique au cloud| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |
| Authentification unique auprès de ressources locales| | ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |
| Accès conditionnel <br> (Les appareils doivent être marqués comme étant conformes) <br> (Administration par la solution de gestion des périphériques mobiles)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) | ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)|![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |
Accès conditionnel <br>(Appareils joints Azure AD Hybride requis)| | | ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)
| Réinitialisation du mot de passe en libre-service à partir de l’écran de connexion Windows| | ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |
| Réinitialisation du code PIN Windows Hello| | ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |
| Enterprise State Roaming sur les appareils| | ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png) |


## <a name="azure-ad-registration"></a>Inscription à Azure AD 

Les appareils inscrits sont souvent managés avec [Microsoft Intune](/mem/intune/enrollment/device-enrollment). Les appareils sont inscrits dans Intune de plusieurs façons, selon le système d’exploitation. 

Les appareils inscrits à Azure AD offrent une prise en charge pour l’authentification unique (SSO) des appareils BYOD et des appareils d’entreprise sur les ressources cloud. L’accès aux ressources est basé sur les [stratégies d’accès conditionnel](../conditional-access/require-managed-devices.md) Azure AD appliquées à l’appareil et à l’utilisateur.

### <a name="registering-devices"></a>Inscription des appareils

Les appareils inscrits sont souvent managés avec [Microsoft Intune](/mem/intune/enrollment/device-enrollment). Les appareils sont inscrits dans Intune de plusieurs façons, selon le système d’exploitation. 

Les appareils BYOD et les appareils mobiles appartenant à l’entreprise sont enregistrés par les utilisateurs qui installent l’application Portail d’entreprise.

* [iOS](/mem/intune/user-help/install-and-sign-in-to-the-intune-company-portal-app-ios)

* [Android](/mem/intune/user-help/enroll-device-android-company-portal)

* [Windows 10](/mem/intune/user-help/enroll-windows-10-device)

Si l’inscription de vos appareils est la meilleure option pour votre organisation, consultez les ressources suivantes :

* Cette vue d’ensemble des [appareils inscrits à Azure AD](concept-azure-ad-register.md).

* Cette documentation pour l’utilisateur final expliquant comment [Inscrire votre appareil personnel au réseau de votre organisation](../user-help/user-help-register-device-on-network.md).

## <a name="azure-ad-join"></a>jointure Azure AD ;

La jonction Azure AD vous permet d’effectuer la transition vers un modèle privilégiant le cloud avec Windows. Elle offre une bonne base si vous envisagez de moderniser votre gestion des appareils et de réduire les coûts informatiques liés aux appareils. La jonction Azure AD fonctionne avec les appareils Windows 10 uniquement. Considérez-la comme le premier choix pour les nouveaux appareils.

Toutefois, les [appareils joints à Azure AD peuvent utiliser l’authentification unique sur des ressources locales](azuread-join-sso.md) lorsqu’ils se trouvent sur le réseau de l’organisation, peuvent s’authentifier sur des serveurs locaux, comme des serveurs de fichiers, d’impression et d’autres applications.

S’il s’agit de la meilleure option pour votre organisation, consultez les ressources suivantes :

* Cette vue d’ensemble des [appareils joints à Azure AD](concept-azure-ad-join.md).

* Familiarisez-vous avec le [plan d’implémentation de la jonction Azure AD](azureadjoin-plan.md).

### <a name="provisioning-azure-ad-join-to-your-devices"></a>Provisionnement de la jonction Azure AD sur vos appareils

Vous pouvez provisionner la jonction Azure AD en utilisant les approches suivantes :

* Libre-service : [Expérience de première exécution de Windows 10](azuread-joined-devices-frx.md)

Si Windows 10 Professionnel ou Windows 10 Entreprise est installé sur un appareil, le processus exécuté par défaut est l’installation pour les appareils d’entreprise.

* [Expérience OOBE Windows ou à partir des Paramètres Windows](../user-help/user-help-join-device-on-network.md)

* [Windows Autopilot](/windows/deployment/windows-autopilot/windows-autopilot)

* [Inscription en bloc](/mem/intune/enrollment/windows-bulk-enroll)

Choisissez votre procédure de déploiement après avoir effectué une [comparaison détaillée de ces approches](azureadjoin-plan.md).

Vous pouvez déterminer que la jonction Azure AD est la meilleure solution pour un appareil, et cet appareil peut être déjà dans un état différent. Voici les points importants relatifs à la mise à niveau

| État actuel de l’appareil| État souhaité de l’appareil| Procédures |
| - | - | - |
| Jonction au domaine local| Azure AD Join| Annulez la jonction de l’appareil au domaine local avant de le joindre à Azure AD |
| Jonction Azure AD Hybride| Azure AD Join| Annulez la jonction de l’appareil au domaine local et à Azure AD avant de le joindre à Azure AD |
| Appareils inscrits sur Azure AD| Azure AD Join| Annulez l’inscription de l’appareil avant de le joindre à Azure AD |


## <a name="hybrid-azure-ad-join"></a>jointure Azure AD hybride ;

Si vous disposez d’un environnement Active Directory local et que vous souhaitez joindre vos ordinateurs joints au domaine Active Directory à Azure AD, vous pouvez y parvenir en effectuant une jonction Azure AD Hybride. Elle prend en charge un [large éventail d’appareils Windows](hybrid-azuread-join-plan.md), notamment les appareils Windows actuels et les appareils exécutant une version antérieure de Windows.

La plupart des organisations possèdent déjà des appareils joints à un domaine et les gèrent via une Stratégie de groupe ou System Center Configuration Manager (SCCM). Dans ce cas, nous vous recommandons de configurer la jonction Azure AD Hybride pour commencer à profiter des avantages tout en tirant parti des investissements existants.

Si la jonction Azure AD Hybride est la meilleure option pour votre organisation, consultez les ressources suivantes :

* Cette vue d’ensemble des [appareils joints à Azure AD Hybride](concept-azure-ad-join-hybrid.md).

* Familiarisez-vous avec le [plan d’implémentation de la jonction Azure AD Hybride](hybrid-azuread-join-plan.md).

### <a name="provisioning-hybrid-azure-ad-join-to-your-devices"></a>Provisionnement de la jonction Azure AD Hybride sur vos appareils

[Passez en revue votre infrastructure d’identité](hybrid-azuread-join-plan.md). Azure AD Connect comporte un Assistant permettant de configurer la jonction Azure AD Hybride pour les éléments suivants :

* [Domaines fédérés](hybrid-azuread-join-federated-domains.md)

* [Domaines managés](hybrid-azuread-join-managed-domains.md)

Si vous n’envisagez pas d’installer la version requise d’Azure AD Connect, consultez [comment configurer manuellement la jonction Azure AD Hybride](hybrid-azuread-join-manual.md). 

> [!NOTE] 
> L’appareil Windows 10 joint à un domaine local tente de se joindre automatiquement à Azure AD pour devenir joint à Azure AD Hybride par défaut. Cela ne fonctionnera que si vous avez configuré l’environnement approprié. 

Vous pouvez déterminer que la jonction Azure AD Hybride est la meilleure solution pour un appareil, et cet appareil peut être déjà dans un état différent. Voici les points importants relatifs à la mise à niveau.

| État actuel de l’appareil| État souhaité de l’appareil| Procédures |
| - | - | - |
| Jonction à un domaine local| Jonction Azure AD Hybride| Utiliser Azure AD Connect ou AD FS pour la jonction à Azure |
| Groupe de travail local joint ou nouveau| Jonction Azure AD Hybride| Prise en charge assurée avec [Windows Autopilot](/windows/deployment/windows-autopilot/windows-autopilot). Sinon, l’appareil doit être joint au domaine local avant d’effectuer la jonction Azure AD Hybride |
| Appareil joints Azure AD| Jonction Azure AD Hybride| Annulez la jonction avec Azure AD, ce qui le place dans le groupe de travail local ou dans un nouvel état. |
| Inscrit auprès d’Azure AD| Jonction Azure AD Hybride| Dépend de la version de Windows. [Voir ces remarques](hybrid-azuread-join-plan.md). |

## <a name="manage-your-devices"></a>Gestion de vos appareils

Une fois que vous avez inscrit ou joint vos appareils à Azure AD, utilisez le [portail Azure](https://portal.azure.com/) comme emplacement central pour manager les identités des appareils. La page des appareils Azure Active Directory vous permet d’effectuer les opérations suivantes :

* [Configurer les paramètres de vos appareils](device-management-azure-portal.md#configure-device-settings)
* Vous devez être un administrateur local pour manager les appareils Windows. [Azure AD met à jour cette appartenance pour les appareils joints à Azure AD](assign-local-admin.md), les ajoutant automatiquement avec le rôle de gestionnaire d’appareils en tant qu’administrateurs de tous les appareils joints.

Veillez à préserver l’intégrité de l’environnement en [gérant les appareils obsolètes](manage-stale-devices.md) et à concentrer vos ressources sur la gestion des appareils actuels.

* [Examiner les journaux d’audit liés à l’appareil](device-management-azure-portal.md#audit-logs)

### <a name="supported-device-management-tools"></a>Outils de gestion des appareils pris en charge

Les administrateurs peuvent sécuriser et mieux contrôler ces appareils inscrits et joints à l’aide d’autres outils de gestion des appareils. Ces outils permettent d’appliquer les configurations requises par l’organisation, comme l’exigence du chiffrement du stockage, la complexité des mots de passe, les installations de logiciels et les mises à jour logicielles. 

Passez en revue les plateformes prises en charge et non prises en charge pour les appareils intégrés :

| Outils de gestion des appareils| Appareils inscrits sur Azure AD| jointure Azure AD ;| jointure Azure AD hybride ;|
| - | - | - | - |
| [Gestion des périphériques mobiles](/windows/client-management/mdm/azure-active-directory-integration-with-mdm) <br>Exemple : Microsoft Intune| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)|  |
| [Cogestion avec Microsoft Intune et Microsoft Endpoint Configuration Manager](/mem/configmgr/comanage/overview) <br>(Windows 10 et versions ultérieures)| | ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)| ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)|  |
| [Stratégie de groupe](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh831791(v=ws.11))<br>(Windows uniquement)| | | ![Coche pour ces valeurs.](./media/plan-device-deployment/check.png)|  |



 Nous vous recommandons de prendre en compte la [gestion des applications mobiles Microsoft Intune](/mem/intune/apps/app-management) avec ou sans la gestion des appareils pour les appareils iOS ou Android inscrits.

 Les administrateurs peuvent également [déployer des plateformes VDI (Virtual Desktop Infrastructure)](howto-device-identity-virtual-desktop-infrastructure.md) hébergeant des systèmes d’exploitation Windows dans leurs organisations afin de simplifier la gestion et de réduire les coûts grâce à la consolidation et à la centralisation des ressources. 

### <a name="troubleshoot-device-identities"></a>Résoudre les problèmes d’identités d’appareil

* [Résolution des problèmes des appareils à l’aide de la commande dsregcmd](troubleshoot-device-dsregcmd.md)

* [Résolution des problèmes de paramètres Enterprise State Roaming dans Azure Active Directory](enterprise-state-roaming-troubleshooting.md)

Si vous rencontrez des problèmes durant l’exécution d’une jonction Azure AD Hybride pour des appareils Windows appartenant à un domaine, consultez :

* [Résolution des problèmes de jonction Azure AD Hybride pour les appareils Windows actuels](troubleshoot-hybrid-join-windows-current.md)

* [Résolution des problèmes de jonction Azure AD Hybride pour les appareils Windows de version plus ancienne](troubleshoot-hybrid-join-windows-legacy.md)

## <a name="next-steps"></a>Étapes suivantes

* [Planifier votre implémentation de la jonction Azure AD](azureadjoin-plan.md)
* [Planifier votre implémentation de la jonction Azure AD Hybride](hybrid-azuread-join-plan.md)
* [Gérer les identités des appareils](device-management-azure-portal.md)