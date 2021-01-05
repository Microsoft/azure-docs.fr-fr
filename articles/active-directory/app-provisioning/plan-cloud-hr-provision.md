---
title: Planifier une application RH cloud pour l’approvisionnement d’utilisateurs Azure Active Directory
description: Cet article décrit le processus de déploiement de l’intégration des systèmes RH cloud, tels que Workday et SucessFactors avec Azure Active Directory. L’intégration d’Azure AD à votre système RH cloud permet d’obtenir un système complet de gestion du cycle de vie des identités.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: conceptual
ms.workload: identity
ms.date: 11/22/2019
ms.author: kenwith
ms.reviewer: arvindha, celested
ms.openlocfilehash: 64418a727ecb9a300912a4766a9ea2066328ad31
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96174898"
---
# <a name="plan-cloud-hr-application-to-azure-active-directory-user-provisioning"></a>Planifier une application RH cloud pour l’approvisionnement d’utilisateurs Azure Active Directory

Historiquement, le personnel informatique s’appuyait sur des méthodes manuelles pour créer, mettre à jour et supprimer des employés. Ils ont utilisé des méthodes telles que le téléchargement de fichiers CSV ou de scripts personnalisés pour synchroniser les données des employés. Ces processus d’approvisionnement sont sujets aux erreurs, non sécurisés et difficiles à gérer.

Pour gérer les cycles de vie des identités des employés, des fournisseurs ou des travailleurs occasionnels, le [service d’approvisionnement d’utilisateurs Azure Active Directory (Azure AD)](../app-provisioning/user-provisioning.md) offre une intégration avec des applications de ressources humaines (RH) informatiques basées sur le cloud. Exemples d’applications : Workday ou SuccessFactors.

Azure AD utilise cette intégration pour activer les workflows suivants d’applications RH cloud :

- **Approvisionnement des utilisateurs vers Active Directory :** approvisionnez les ensembles d’utilisateurs sélectionnés dans une application RH cloud dans un ou plusieurs domaines Active Directory (AD).
- **Approvisionnement des utilisateurs du cloud uniquement vers Azure AD :** dans les scénarios où Active Directory n’est pas utilisé, approvisionnez les utilisateurs directement à partir de l’application RH du cloud vers Azure AD.
- **Mise à jour de l’application RH cloud** : mettez à jour les adresses e-mail et les attributs de nom d’utilisateur à partir d’Azure AD sur l’application RH cloud.

> [!NOTE]
> Ce plan de déploiement vous indique comment déployer vos workflows d’application RH cloud à l’aide de l’approvisionnement d’utilisateurs Azure AD. Pour plus d’informations sur la façon de déployer l’approvisionnement automatique d’utilisateurs vers des applications SaaS (Software-as-a-Service), consultez [Planifier un déploiement de l’approvisionnement automatique d’utilisateurs](./plan-auto-user-provisioning.md).

## <a name="enabled-hr-scenarios"></a>Scénarios RH activés

Le service d’approvisionnement d’utilisateurs Azure AD permet d’automatiser les scénarios suivants de gestion du cycle de vie des identités basée sur les RH :

- **Embauche de nouveaux employés :** lorsqu’un nouvel employé est ajouté à l’application RH cloud, un compte d’utilisateur est automatiquement créé dans Active Directory et Azure AD avec la possibilité de mettre à jour l’adresse e-mail et les attributs du nom d’utilisateur dans l’application RH cloud.
- **Mises à jour des profils et des attributs des employés :** lorsqu’un enregistrement d’employé, tel qu’un nom, un titre ou un responsable, est mis à jour dans l’application RH cloud, son compte d’utilisateur est automatiquement mis à jour dans Active Directory et Azure AD.
- **Licenciement des employés :** lorsqu’un employé est licencié dans l’application RH cloud, son compte d’utilisateur est automatiquement désactivé dans Active Directory et Azure AD.
- **Réembauche d’employés :** lorsqu’un employé est réembauché dans l’application RH cloud, son ancien compte peut être automatiquement réactivé ou réapprovisionné dans Active Directory et Azure AD.

## <a name="who-is-this-integration-best-suited-for"></a>À qui cette intégration convient-elle le mieux ?

L’intégration de l’application RH cloud avec l’approvisionnement d’utilisateurs Azure AD est idéale pour les organisations qui :

- veulent une solution informatique prédéfinie basée sur le cloud pour l’approvisionnement d’utilisateurs RH cloud ;
- nécessitent un approvisionnement d’utilisateurs direct de l’application RH cloud vers Active Directory ou Azure AD ;
- exigent que les utilisateurs soient approvisionnés à l’aide des données obtenues à partir de l’application RH cloud ;
- ont besoin de joindre, de déplacer et de laisser les utilisateurs à synchroniser avec une ou plusieurs unités d’organisation, domaines et forêts Active Directory seulement sur la base d’informations de modifications détectées dans l’application RH cloud ;
- Utilisez Microsoft 365 pour la messagerie électronique.

## <a name="learn"></a>Découvrir

L’approvisionnement des utilisateurs crée une base pour la gouvernance continue des identités. Cela améliore la qualité des processus d’entreprise qui reposent sur les données d’identité faisant autorité.

### <a name="terms"></a>Termes

Cet article utilise les termes suivants :

- **Système source** : référentiel d’utilisateurs à partir duquel Azure AD approvisionne. C’est le cas, par exemple, d’une application RH cloud telle que Workday ou SuccessFactors.
- **Système cible** : référentiel d’utilisateurs vers lequel Azure AD approvisionne. Active Directory, Azure AD, Microsoft 365 ou d’autres applications SaaS en sont des exemples.
- **Processus entrants-changements de poste-sortants :** terme utilisé pour les nouvelles embauches, les transferts et les licenciements avec une application RH cloud comme système d’enregistrement. Le processus se termine lorsque le service approvisionne correctement les attributs nécessaires sur le système cible.

### <a name="key-benefits"></a>Principaux avantages

Cette fonctionnalité d’approvisionnement informatique axé sur les ressources humaines offre les avantages significatifs suivants pour l’entreprise :

- **Augmenter la productivité :** vous pouvez désormais automatiser l’attribution de comptes d’utilisateur et de licences Microsoft 365 et fournir un accès aux groupes de clés. L’automatisation des attributions offre aux nouveaux employés un accès immédiat à leurs outils de travail et augmente la productivité.
- **Gérer le risque :** vous pouvez augmenter la sécurité en automatisant les changements, en fonction du statut des employés ou des appartenances aux groupes grâce aux données transmises à partir de l’application RH cloud. L’automatisation des modifications permet de s’assurer que les identités des utilisateurs et l’accès aux applications clés sont automatiquement mis à jour lorsque les utilisateurs sont transférés ou quittent l’organisation.
- **Conformité et gouvernance des adresses :** Azure AD prend en charge les journaux d’audit natifs pour les demandes d’approvisionnement d’utilisateurs effectuées par les applications des systèmes source et cible. Lors d’audit, vous pouvez effectuer le suivi des personnes qui ont accès aux applications à partir d’un seul écran.
- **Gérer les coûts :** l’approvisionnement automatique réduit les coûts tout en évitant l’inefficacité et les erreurs humaines qui sont inhérents à l’approvisionnement manuel. Cela réduit le besoin de solutions d’approvisionnement d’utilisateurs personnalisées créées au fil du temps à l’aide de plateformes héritées et obsolètes.

### <a name="licensing"></a>Licence

Pour configurer l’application RH cloud à l’intégration de l’approvisionnement d’utilisateurs Azure AD, vous devez disposer d’une [licence Premium Azure AD](https://azure.microsoft.com/pricing/details/active-directory/) valide et d’une licence pour l’application RH cloud telle que Workday ou SuccessFactors.

Vous avez également besoin d’une licence valide à un abonnement Azure AD Premium P1 ou supérieur pour chaque utilisateur provenant de l’application RH cloud et approvisionné dans Active Directory ou Azure AD. Tout nombre incorrect de licences détenues dans l’application RH cloud peut entraîner des erreurs pendant l’approvisionnement d’utilisateurs.

### <a name="prerequisites"></a>Prérequis

- [Administrateur d’identités hybrides](../roles/permissions-reference.md#hybrid-identity-administrator) Azure AD pour configurer l’agent d’approvisionnement Azure AD Connect.
- Rôle d’[administrateur d’application](../roles/permissions-reference.md#application-administrator) Azure AD pour configurer l’application d’approvisionnement dans le portail Azure.
- Une instance de test et de production de l’application RH cloud.
- Autorisations d’administrateur dans l’application RH cloud pour créer un utilisateur de l’intégration système et apporter des modifications afin de tester les informations de l’employé.
- Pour l’approvisionnement d’utilisateurs vers Active Directory, un serveur exécutant Windows Server 2012 ou version ultérieure et doté d’un runtime .NET 4.7.1+ est nécessaire afin d’héberger l’agent d’approvisionnement Azure AD Connect.
- [Azure AD Connect](../hybrid/whatis-azure-ad-connect.md) pour la synchronisation des utilisateurs entre Active Directory et Azure AD.

### <a name="training-resources"></a>Ressources de formation

| **Ressources** | **Lien et description** |
|:-|:-|
| Vidéos | [Présentation de l’attribution d’utilisateurs dans Azure Active Directory](https://youtu.be/_ZjARPpI6NI) |
| | [Comment déployer l’approvisionnement d’utilisateurs dans Azure Active Directory](https://youtu.be/pKzyts6kfrw) |
| Tutoriels | [Liste de tutoriels sur l’intégration d’applications SaaS à Azure AD](../saas-apps/tutorial-list.md) |
| | [Tutoriel : Configurer Workday pour le provisionnement automatique d’utilisateurs](../saas-apps/workday-inbound-tutorial.md#frequently-asked-questions-faq) |
| Questions fréquentes (FAQ) | [Approvisionnement automatique d’utilisateurs](../app-provisioning/user-provisioning.md#what-applications-and-systems-can-i-use-with-azure-ad-automatic-user-provisioning) |
| | [Approvisionnement de Workday vers Azure AD](../saas-apps/workday-inbound-tutorial.md#frequently-asked-questions-faq) |

### <a name="solution-architecture"></a>Architecture de solution

L’exemple suivant décrit l’architecture de la solution d’approvisionnement d’utilisateurs de bout en bout pour les environnements hybrides courants et comprend :

- **Flux de données RH faisant autorité de l’application RH cloud vers Active Directory.** Dans ce flux, l’événement RH (processus entrants-changements de poste-sortants) est initié dans le locataire de l’application RH cloud. Le service d’approvisionnement Azure AD et l’agent d’approvisionnement Azure AD Connect approvisionnent les données utilisateur du locataire de l’application RH cloud vers Active Directory. Selon l’événement, cela peut provoquer des opérations de création, de mise à jour, d’activation et de désactivation dans Active Directory.
- **Synchronisez avec Azure AD et mettez à jour l’e-mail et le nom d’utilisateur d’Active Directory en local vers une application RH cloud.** Une fois les comptes mis à jour dans Active Directory, cela est synchronisé avec Azure AD via Azure AD Connect. Les adresses e-mail et les attributs de nom d’utilisateur peuvent être mis à jour dans le locataire de l’application RH cloud.

![Diagramme du workflow](media/plan-cloud-hr-provision/plan-cloudhr-provisioning-img1.png)

#### <a name="description-of-workflow"></a>Description du workflow

Les étapes de clé suivantes sont indiquées dans le diagramme :  

1. **L’équipe RH** effectue les transactions dans le locataire de l’application RH cloud.
2. **Le service d’approvisionnement Azure AD** exécute les cycles planifiés à partir du locataire de l’application RH cloud et identifie les modifications qui doivent être traitées pour la synchronisation avec Active Directory.
3. **Le service d’approvisionnement Azure AD** appelle l’agent d’approvisionnement Azure AD Connect avec une charge utile de demandes contenant des opérations de création, de mise à jour, d’activation et de désactivation de compte Active Directory.
4. **L’agent d’approvisionnement Azure AD Connect** utilise un compte de service pour gérer des données de compte Active Directory.
5. **Azure AD Connect** exécute la [synchronisation](../hybrid/how-to-connect-sync-whatis.md) différentielle pour extraire les mises à jour dans Active Directory.
6. Les mises à jour d’**Active Directory** sont synchronisées avec Azure AD.
7. **Le service d’approvisionnement Azure AD** met à jour l’attribut d’e-mail et le nom d’utilisateur à partir d’Azure AD vers le locataire de l’application RH cloud.

## <a name="plan-the-deployment-project"></a>Planifier le projet de déploiement

Tenez compte des besoins de votre organisation lorsque vous déterminez la stratégie de ce déploiement dans votre environnement.

### <a name="engage-the-right-stakeholders"></a>Impliquer les parties prenantes appropriées

Lorsque des projets technologiques échouent, cela est généralement dû à des attentes qui ne correspondent pas à l’impact, aux résultats et aux responsabilités réels. Pour éviter un tel cas de figure, [veillez à faire appel aux bonnes personnes](../fundamentals/active-directory-deployment-plans.md). Assurez-vous également que les rôles des parties prenantes dans le projet sont bien compris. Documentez les parties prenantes et leurs informations et responsabilités de projet.

Incluez un représentant de l’organisation RH qui peut fournir des informations sur les processus opérationnels RH existants et les exigences en matière de traitement des données relatives à l’identité du travailleur et à ses tâches.

### <a name="plan-communications"></a>Planifier les communications

La communication est essentielle à la réussite de tout nouveau service. Communiquez de manière proactive avec vos utilisateurs sur le moment où et la façon dont leur expérience va changer. Dites-leur comment bénéficier d’un support s’ils rencontrent des problèmes.

### <a name="plan-a-pilot"></a>Prévoir un pilote

L’intégration des processus opérationnels RH et des workflows d’identité de l’application RH cloud vers les systèmes cibles requiert une quantité considérable de validation, transformation et nettoyage des données et de test de bout en bout avant de pouvoir déployer la solution en production.

Exécutez la configuration initiale dans un [environnement pilote](../fundamentals/active-directory-deployment-plans.md#best-practices-for-a-pilot) avant de la mettre à l’échelle de tous les utilisateurs en production.

## <a name="select-cloud-hr-provisioning-connector-apps"></a>Sélectionner les applications du connecteur d’approvisionnement RH cloud

Pour faciliter les workflows d’approvisionnement Azure AD entre l’application RH cloud et Active Directory, vous pouvez ajouter plusieurs applications du connecteur d’approvisionnement à partir de la galerie d’applications Azure AD :

- **Approvisionnement d’utilisateurs de l’application RH cloud vers Active Directory** : cette application du connecteur d’approvisionnement facilite l’approvisionnement des comptes d’utilisateur à partir de l’application RH cloud vers un domaine Active Directory unique. Si vous avez de plusieurs domaines, vous pouvez ajouter une instance de cette application à partir de la galerie d’applications Azure AD pour chaque domaine Active Directory vers lequel vous devez provisionner.
- **Approvisionnement d’utilisateurs de l’application RH cloud vers Azure AD** : Azure AD Connect est l’outil à utiliser pour synchroniser les utilisateurs Active Directory avec Azure AD, mais cette application du connecteur d’approvisionnement peut aussi faciliter l’approvisionnement d’utilisateurs cloud uniquement de l’application RH cloud vers un locataire Azure AD unique.
- **Mise à jour de l’application RH Cloud** : cette application du connecteur d’approvisionnement facilite la mise à jour des adresses e-mail de l’utilisateur d’Azure AD vers l’application RH cloud.

Par exemple, l’image suivante répertorie les applications du connecteur Workday qui sont disponibles dans la galerie d’applications Azure AD.

![Galerie d’applications du portail Azure Active Directory](media/plan-cloud-hr-provision/plan-cloudhr-provisioning-img2.png)

### <a name="decision-flow-chart"></a>Organigramme décisionnel

Utilisez l’organigramme décisionnel suivant pour identifier les applications d’approvisionnement RH cloud pertinentes pour votre scénario.

![Organigramme décisionnel](media/plan-cloud-hr-provision/plan-cloudhr-provisioning-img3.png)

## <a name="design-the-azure-ad-connect-provisioning-agent-deployment-topology"></a>Concevoir la topologie de déploiement de l’agent d’approvisionnement Azure AD Connect

L’intégration de l’approvisionnement entre l’application RH cloud et Active Directory requiert quatre composants :

- Locataire de l’application RH cloud
- Application du connecteur d’approvisionnement
- Agent d’approvisionnement Azure AD Connect
- Domaine Active Directory

La topologie de déploiement de l’agent d’approvisionnement Azure AD Connect dépend du nombre de locataires de l’application RH cloud et des sous-domaines Active Directory que vous prévoyez d’intégrer. Si vous avez plusieurs domaines Active Directory, cela dépend si les domaines Active Directory sont contigus ou [disjoint](/windows-server/identity/ad-ds/plan/disjoint-namespace).

En fonction de votre décision, choisissez l’un des scénarios de déploiement suivants :

- Locataire unique de l’application RH cloud -> ciblez un ou plusieurs domaines enfants Active Directory dans une forêt approuvée
- Locataire unique de l’application RH cloud -> ciblez plusieurs domaines enfants dans une forêt Active Directory disjointe

### <a name="single-cloud-hr-app-tenant---target-single-or-multiple-active-directory-child-domains-in-a-trusted-forest"></a>Locataire unique de l’application RH cloud -> ciblez un ou plusieurs domaines enfants Active Directory dans une forêt approuvée

Nous vous recommandons la configuration de production suivante :

|Condition requise|Recommandation|
|:-|:-|
|Nombre d’agents d’approvisionnement Azure AD Connect à déployer|Deux (pour haute disponibilité et basculement)
|Nombre d’applications du connecteur d’approvisionnement à configurer|Une application par sous-domaine|
|Serveur hôte pour l’agent d’approvisionnement Azure AD Connect|Windows 2012 R2+ avec ligne de vue sur les contrôleurs de domaine Active Directory géo-localisés</br>Peut coexister avec le service Azure AD Connect|

![Flux vers les agents locaux](media/plan-cloud-hr-provision/plan-cloudhr-provisioning-img4.png)

### <a name="single-cloud-hr-app-tenant---target-multiple-child-domains-in-a-disjoint-active-directory-forest"></a>Locataire unique de l’application RH cloud -> ciblez plusieurs domaines enfants dans une forêt Active Directory disjointe

Ce scénario implique l’approvisionnement d’utilisateurs de l’application RH cloud vers des domaines situés dans des forêts Active Directory disjointes.

Nous vous recommandons la configuration de production suivante :

|Condition requise|Recommandation|
|:-|:-|
|Nombre d’agents d’approvisionnement Azure AD Connect à déployer localement|Deux par forêt Active Directory disjointe|
|Nombre d’applications du connecteur d’approvisionnement à configurer|Une application par sous-domaine|
|Serveur hôte pour l’agent d’approvisionnement Azure AD Connect|Windows 2012 R2+ avec ligne de vue sur les contrôleurs de domaine Active Directory géo-localisés</br>Peut coexister avec le service Azure AD Connect|

![Une seule forêt Active Directory disjointe du locataire de l’application RH cloud](media/plan-cloud-hr-provision/plan-cloudhr-provisioning-img5.png)

### <a name="azure-ad-connect-provisioning-agent-requirements"></a>Configuration requise pour l’agent d’approvisionnement Azure AD Connect

La solution d’approvisionnement d’utilisateurs de l’application RH cloud vers Active Directory nécessite le déploiement d’un ou plusieurs agents d’approvisionnement Azure AD Connect sur des serveurs exécutant Windows 2012 R2 (ou versions ultérieures). Les serveurs doivent avoir au minimum 4 Go de RAM et runtime .NET 4.7.1+. Vérifiez que le serveur hôte a un accès réseau au domaine Active Directory cible.

Pour préparer l’environnement local, l’Assistant de configuration de l’agent d’approvisionnement Azure AD Connect inscrit l’agent auprès de votre locataire Azure AD, [ouvre les ports](../manage-apps/application-proxy-add-on-premises-application.md#open-ports), [autorise l’accès aux URL](../manage-apps/application-proxy-add-on-premises-application.md#allow-access-to-urls) et prend en charge [la configuration du proxy HTTPS sortant](../saas-apps/workday-inbound-tutorial.md#how-do-i-configure-the-provisioning-agent-to-use-a-proxy-server-for-outbound-http-communication).

L’agent d’approvisionnement utilise un compte de service pour communiquer avec le ou les domaines Active Directory. Avant d’installer l’agent, créez un compte de service dans les utilisateurs et les ordinateurs Active Directory qui répondent aux exigences suivantes :

- Un mot de passe qui n’expire pas
- Des autorisations de contrôle délégué pour lire, créer, supprimer et gérer des comptes d’utilisateur

Vous pouvez sélectionner des contrôleurs de domaine qui doivent gérer les demandes d’approvisionnement. Si vous avez plusieurs contrôleurs de domaine géographiquement répartis, installez l’agent d’approvisionnement dans le même site que vos contrôleurs de domaine par défaut. Ce positionnement améliore la fiabilité et les performances de la solution de bout en bout.

Pour une haute disponibilité, vous pouvez déployer plusieurs agents d’approvisionnement Azure AD Connect. Enregistrez l’agent pour gérer le même ensemble de domaines Active Directory locaux.

## <a name="plan-scoping-filters-and-attribute-mapping"></a>Planifier les filtres d’étendue et le mappage des attributs

Lorsque vous activez l’approvisionnement à partir de l’application RH cloud vers Active Directory ou Azure AD, le Portail Azure contrôle les valeurs d’attribut par le biais du mappage d’attributs.

### <a name="define-scoping-filters"></a>Définir des filtres d’étendue

Utilisez des [filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md) pour définir les règles basées sur les attributs qui déterminent les utilisateurs qui doivent être approvisionnés à partir de l’application de ressources humaines cloud vers Active Directory ou Azure AD.

Lorsque vous lancez le processus Entrants, rassemblez les spécifications suivantes :

- L’application RH cloud est-elle utilisée pour intégrer à la fois les employés et les travailleurs occasionnels ?
- Envisagez-vous d’utiliser l’approvisionnement d’utilisateurs de l’application de ressources humaines cloud vers Azure AD pour gérer à la fois les employés et les travailleurs occasionnels ?
- Prévoyez-vous de déployer l’approvisionnement d’utilisateurs de l’application RH cloud vers Azure AD uniquement pour un sous-ensemble d’utilisateurs de l’application RH cloud ? Pour les employés uniquement, par exemple.

En fonction de vos besoins, lorsque vous configurez des mappages d’attributs, vous pouvez définir le champ **Étendue de l’objet source** pour sélectionner les ensembles d’utilisateurs de l’application RH cloud qui doivent être dans l’étendue de l’approvisionnement vers Active Directory. Pour plus d’informations, voir le tutoriel sur les applications de ressources humaines cloud pour obtenir les filtres d’étendue couramment utilisés.

### <a name="determine-matching-attributes"></a>Déterminer les attributs correspondants

Grâce à l’approvisionnement, vous avez la possibilité de mettre en correspondance les comptes existants entre le système source et le système cible. Lorsque vous intégrez l’application RH cloud au service d’approvisionnement Azure AD, vous pouvez [configurer le mappage d’attributs](../app-provisioning/configure-automatic-user-provisioning-portal.md#mappings) pour déterminer quelles données utilisateur doivent transiter de l’application RH cloud vers Active Directory ou Azure AD.

Lorsque vous lancez le processus Entrants, rassemblez les spécifications suivantes :

- Quel est l’ID unique de cette application RH cloud qui est utilisé pour identifier chaque utilisateur ?
- Du point de vue du cycle de vie des identités, comment gérez-vous les employés réembauchés ? Les employés réembauchés gardent-ils leur ancien ID d’employé ?
- Traitez-vous les embauches futures et créez-vous pour eux des comptes Active Directory à l’avance ?
- Du point de vue du cycle de vie des identités, comment gérez-vous la conversion employé à travailleur occasionnel ou autre ?
- Les utilisateurs convertis gardent-ils leurs anciens comptes Active Directory ou en ont-ils de nouveaux ?

En fonction de vos besoins, Azure AD prend en charge le mappage direct d’attribut à attribut, en fournissant des valeurs constantes ou [en écrivant des expressions pour les mappages d’attributs](../app-provisioning/functions-for-customizing-application-data.md). Cette flexibilité vous permet de contrôler très précisément ce qui sera renseigné dans l’attribut de l’application ciblée. Vous pouvez utiliser l’[API Microsoft Graph](../app-provisioning/export-import-provisioning-configuration.md) et l’Afficheur Graph pour exporter vos mappages d’attributs et votre schéma d’attribution d’utilisateurs dans un fichier JSON, et le réimporter dans Azure AD.

Par défaut, l’attribut de l’application RH cloud qui représente l’ID d’employé unique est utilisé comme attribut correspondant *mappé à l’attribut unique dans Active Directory.* Par exemple, dans le scénario de l’application Workday, l’attribut **WorkerID** de **Workday** est mappé à l’attribut **employeeID** d’Active Directory.

Vous pouvez définir plusieurs attributs de correspondance et attribuer la priorité de correspondance. Ils sont évalués sur la priorité de correspondance. Dès qu’une correspondance est trouvée, aucun autre attribut correspondant n’est évalué.

Vous pouvez également [personnaliser les mappages d’attributs par défaut](../app-provisioning/customize-application-attributes.md#understanding-attribute-mapping-types), tels que la modification ou la suppression de mappages d’attributs existants. Vous pouvez aussi créer des mappages d’attributs en fonction des besoins de votre organisation. Pour plus d’informations, consultez le tutoriel sur l’application RH cloud (par exemple, [Workday](../saas-apps/workday-inbound-tutorial.md#managing-your-configuration)) pour obtenir la liste des attributs personnalisés à mapper.

### <a name="determine-user-account-status"></a>Déterminer l’état du compte d’utilisateur

Par défaut, l’application du connecteur d’approvisionnement mappe l’état du profil utilisateur RH à l’état du compte d’utilisateur dans Active Directory ou Azure AD pour déterminer s’il faut activer ou désactiver le compte d’utilisateur.

Lorsque vous lancez le processus entrants-sortants, rassemblez les spécifications suivantes.

| Process | Spécifications |
| - | - |
| **Entrants** | Du point de vue du cycle de vie des identités, comment gérez-vous les employés réembauchés ? Les employés réembauchés gardent-ils leur ancien ID d’employé ? |
| | Traitez-vous les embauches futures et créez-vous pour eux des comptes Active Directory à l’avance ? Ces comptes sont-ils créés dans un état activé ou désactivé ? |
| | Du point de vue du cycle de vie des identités, comment gérez-vous la conversion employé à travailleur occasionnel ou autre ? |
| | Les utilisateurs convertis gardent-ils leurs anciens comptes Active Directory ou en ont-ils de nouveaux ? |
| **Sortants** | Les licenciements sont-ils gérés différemment pour les employés et les travailleurs occasionnels dans Active Directory ? |
| | Quelles sont les dates d’effet prises en compte pour le traitement du licenciement des utilisateurs ? |
| | Quel est l’impact des conversions des employés et des travailleurs occasionnels sur les comptes Active Directory existants ? |
| | Comment traitez-vous l’opération « Annuler » dans Active Directory ? Les opérations d’annulation doivent être gérées si des embauches futures sont créées dans Active Directory dans le cadre du processus entrants. |

En fonction de vos besoins, vous pouvez personnaliser la logique de mappage à l’aide d’[expressions Azure AD](../app-provisioning/functions-for-customizing-application-data.md) afin que le compte Active Directory soit activé ou désactivé selon une combinaison de points de données.

### <a name="map-cloud-hr-app-to-active-directory-user-attributes"></a>Mapper des attributs d’utilisateur de l’application RH cloud vers Active Directory

Chaque application RH cloud est fournie avec des mappages par défaut de l’application RH cloud vers Active Directory.

Lorsque vous lancez le processus Entrants-Changements de poste-Sortants, rassemblez les spécifications suivantes.

| Process | Spécifications |
| - | - |
| **Entrants** | Le processus de création de compte Active Directory est-il manuel, automatisé ou partiellement automatisé ? |
| | Envisagez-vous de propager des attributs personnalisés de l’application RH cloud vers Active Directory ? |
| **Changements de poste** | Quels attributs voulez-vous traiter quand une opération Changements de poste a lieu dans l’application RH cloud ? |
| | Effectuez-vous des validations d’attributs spécifiques au moment des mises à jour de l’utilisateur ? Si c’est le cas, fournissez des détails. |
| **Sortants** | Les licenciements sont-ils gérés différemment pour les employés et les travailleurs occasionnels dans Active Directory ? |
| | Quelles sont les dates d’effet prises en compte pour le traitement du licenciement des utilisateurs ? |
| | Quel est l’impact des conversions des employés et des travailleurs occasionnels sur les comptes Active Directory existants ? |

En fonction de vos besoins, vous pouvez modifier les mappages pour répondre à vos objectifs d’intégration. Pour plus d’informations, consultez le tutoriel spécifique sur l’application RH cloud (par exemple, [Workday](../saas-apps/workday-inbound-tutorial.md#part-4-configure-attribute-mappings)) pour obtenir la liste des attributs personnalisés à mapper.

### <a name="generate-a-unique-attribute-value"></a>Générer une valeur d’attribut unique

Lorsque vous lancez le processus Entrants, vous devrez peut-être générer des valeurs d’attribut uniques lorsque vous définissez des attributs tels que CN, samAccountName et l’UPN qui ont des contraintes uniques.

La fonction [SelectUniqueValues](../app-provisioning/functions-for-customizing-application-data.md#selectuniquevalue) d’Azure AD évalue chaque règle, puis vérifie la valeur générée pour l’unicité dans le système cible. Pour un exemple, voir [Générer une valeur unique pour l’attribut userPrincipalName (UPN)](../app-provisioning/functions-for-customizing-application-data.md#generate-unique-value-for-userprincipalname-upn-attribute).

> [!NOTE]
> Cette fonction est actuellement prise en charge uniquement pour l’approvisionnement d’utilisateurs de Workday vers Active Directory. Elle ne peut pas être utilisée avec d’autres applications d’approvisionnement.

### <a name="configure-active-directory-ou-container-assignment"></a>Configurer l’attribution de conteneur d’UO Active Directory

Il est souvent nécessaire de placer les comptes d’utilisateur Active Directory dans des conteneurs basés sur des unités commerciales, des emplacements et des services. Lorsque vous lancez un processus Changements de poste et que l’organisme de supervision change, vous devrez peut-être déplacer l’utilisateur d’une unité d’organisation vers une autre dans Active Directory.

Utilisez la fonction [Switch ()](../app-provisioning/functions-for-customizing-application-data.md#switch) pour configurer la logique métier de l’attribution de l’unité d’organisation et la mapper à l’attribut Active Directory **parentDistinguishedName**.

Par exemple, si vous souhaitez créer des utilisateurs dans l’unité d’organisation en fonction de l’attribut RH **Municipalité**, vous pouvez utiliser l’expression suivante :

`
Switch([Municipality], "OU=Default,OU=Users,DC=contoso,DC=com", "Dallas", "OU=Dallas,OU=Users,DC=contoso,DC=com", "Austin", "OU=Austin,OU=Users,DC=contoso,DC=com", "Seattle", "OU=Seattle,OU=Users,DC=contoso,DC=com", "London", "OU=London,OU=Users,DC=contoso,DC=com")
`

Avec cette expression, si la valeur Municipalité est Dallas, Austin, Seattle ou Londres, le compte d’utilisateur est créé dans l’unité d’organisation correspondante. Si aucune correspondance n’est trouvée, le compte est créé dans l’unité d’organisation par défaut.

## <a name="plan-for-password-delivery-of-new-user-accounts"></a>Planifier la remise de mot de passe pour de nouveaux comptes d’utilisateur

Lorsque vous lancez le processus Entrants, vous devez définir et remettre un mot de passe temporaire pour les nouveaux comptes d’utilisateur. Grâce à l’approvisionnement d’utilisateurs de l’application RH cloud vers Azure AD, vous pouvez déployer la fonction de [réinitialisation de mot de passe en libre-service](../authentication/tutorial-enable-sspr.md) (SSPR) Azure AD pour l’utilisateur dès le premier jour.

La SSPR est un moyen simple pour les administrateurs informatiques de permettre aux utilisateurs de réinitialiser ou de déverrouiller leurs comptes. Vous pouvez approvisionner l’attribut **Numéro de téléphone** de l’application RH cloud vers Active Directory, puis le synchroniser avec Azure AD. Une fois que l’attribut **Numéro de téléphone mobile** est dans Azure AD, vous pouvez activer SSPR pour le compte de l’utilisateur. Ensuite, le premier jour, le nouvel utilisateur peut utiliser le numéro de téléphone mobile enregistré et vérifié pour l’authentification.

## <a name="plan-for-initial-cycle"></a>Planifier le cycle initial

Lorsque le service d’approvisionnement Azure AD s’exécute pour la première fois, il effectue un [cycle initial](../app-provisioning/how-provisioning-works.md#initial-cycle) sur l’application RH cloud pour créer un instantané de tous les objets utilisateur se trouvant dans l’application RH cloud. Le temps nécessaire pour les cycles initiaux dépend directement du nombre d’utilisateurs présents dans le système source. Le cycle initial pour certains locataires de l’application RH cloud ayant plus de 100 000 utilisateurs peut prendre beaucoup de temps.

**Pour les locataires de l’application RH Cloud importants (> 30 000 utilisateurs),** exécutez le cycle initial par étapes progressives. Démarrez les mises à jour incrémentielles uniquement après avoir validé que les attributs corrects sont définis dans Active Directory pour différents scénarios d’approvisionnement d’utilisateurs. Suivez l’ordre ici.

1. Exécutez le cycle initial uniquement pour un ensemble limité d’utilisateurs en paramétrant le [filtre d’étendue](#plan-scoping-filters-and-attribute-mapping).
2. Vérifiez que les valeurs d’attribut et d’approvisionnement du compte Active Directory sont définies pour les utilisateurs sélectionnés pour la première exécution. Si le résultat répond à vos attentes, développez le filtre d’étendue pour inclure progressivement plus d’utilisateurs et vérifier les résultats de la deuxième exécution.

Lorsque vous êtes satisfait des résultats du cycle initial pour les utilisateurs de test, lancez les [mises à jour incrémentielles](../app-provisioning/how-provisioning-works.md#incremental-cycles).

## <a name="plan-testing-and-security"></a>Planifier les tests et la sécurité

À chaque étape de votre déploiement, du pilote initial à l’activation de l’approvisionnement d’utilisateurs, assurez-vous que les tests que vous effectuez donnent les résultats attendus et que vous auditez les cycles d’approvisionnement.

### <a name="plan-testing"></a>Planifier les tests

Après avoir configuré l’approvisionnement d’utilisateurs de l’application RH cloud vers Azure AD, exécutez des cas de test pour vérifier que cette solution répond aux besoins de votre organisation.

|Scénarios|Résultats attendus|
|:-|:-|
|Un nouvel employé est embauché dans l’application RH Cloud.| - Le compte d’utilisateur est approvisionné dans Active Directory.</br>- L’utilisateur peut se connecter à des applications de domaine Active Directory et effectuer les actions souhaitées.</br>- Si la synchronisation Azure AD Connect est configurée, le compte d’utilisateur est également créé dans Azure AD.
|L’utilisateur est licencié dans l’application RH cloud.|- Le compte d’utilisateur est désactivé dans Active Directory.</br>- L’utilisateur ne peut pas se connecter aux applications de l’entreprise qui sont protégées par Active Directory.
|L’organisme de supervision des utilisateurs est mise à jour dans l’application RH cloud.|En fonction du mappage des attributs, le compte d’utilisateur passe d’une unité d’organisation à une autre dans Active Directory.|
|Les RH mettent à jour le responsable de l’utilisateur dans l’application RH cloud.|Le champ Responsable dans Active Directory est mis à jour pour refléter le nom du nouveau responsable.|
|Les RH réembauchent un employé à un nouveau poste.|Le comportement dépend de la façon dont l’application RH cloud est configurée pour générer des ID d’employés :</br>- Si l’ancien ID d’employé est réutilisé pour un employé réembauché, le connecteur active le compte Active Directory existant pour l’utilisateur.</br>-Si l’employé réembauché obtient un nouvel ID d’employé, le connecteur crée un nouveau compte Active Directory pour l’utilisateur.|
|Les RH convertissent l’employé en travailleur occasionnel ou vice versa.|Un nouveau compte Active Directory est créé pour le nouveau personnage et l’ancien compte est désactivé à la date d’effet de la conversion.|

Utilisez les résultats précédents pour déterminer comment faire passer votre implémentation automatique de l’approvisionnement d’utilisateurs en production en fonction de vos délais établis.

> [!TIP]
> Utilisez des techniques telles que la réduction des données et le nettoyage des données lorsque vous actualisez l’environnement de test à l’aide de données de production pour supprimer ou masquer les données personnelles sensibles pour respecter les normes de sécurité et de confidentialité. 

### <a name="plan-security"></a>Planifier la sécurité

Il est fréquent qu’une révision de sécurité soit nécessaire dans le cadre du déploiement d’un nouveau service. Si une révision de sécurité est nécessaire ou n’a pas été effectuée, consultez les nombreux [livres blancs](https://www.microsoft.com/download/details.aspx?id=36391) Azure AD qui fournissent une vue d’ensemble de l’identité en tant que service.

### <a name="plan-rollback"></a>Planifier la restauration

L’implémentation de l’approvisionnement d’utilisateurs des RH dans le cloud peut ne pas fonctionner comme vous le souhaitez dans l’environnement de production. Dans ce cas, les étapes de restauration suivantes peuvent vous aider à revenir à un état correct connu précédent.

1. Consultez le [rapport de synthèse sur l’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md#getting-provisioning-reports-from-the-azure-portal) et les [journaux d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md#provisioning-logs-preview) pour déterminer les opérations incorrectes qui ont été effectuées sur les utilisateurs ou les groupes concernés. Pour plus d’informations sur le rapport de synthèse de l’approvisionnement et les journaux, consultez [Gérer l’approvisionnement d’utilisateurs d’applications RH cloud](#manage-your-configuration).
2. Le dernier état correct connu des utilisateurs ou des groupes concernés peut être déterminé par l’intermédiaire des journaux d’audit de l’approvisionnement ou en examinant les systèmes cibles (Azure AD ou Active Directory).
3. Collaborez avec le propriétaire de l’application pour mettre à jour les utilisateurs ou groupes concernés directement dans l’application, à l’aide des dernières valeurs d’état connues pour être correctes.

## <a name="deploy-the-cloud-hr-app"></a>Déployer l’application RH cloud

Choisissez l’application RH cloud qui correspond aux exigences de votre solution.

**Workday** : Pour importer des profils de travail depuis Workday dans Active Directory et Azure AD, consultez [Didacticiel : Configurer Workday pour l’attribution automatique d’utilisateurs](../saas-apps/workday-inbound-tutorial.md#planning-your-deployment). Si vous le souhaitez, vous pouvez mettre à jour l’adresse e-mail, le nom d’utilisateur et le numéro de téléphone dans Workday.

**SAP SuccessFactors** : Pour importer des profils de Worker depuis SuccessFactors dans Active Directory et Azure AD, consultez [Didacticiel : Configurer SAP SuccessFactors pour l’approvisionnement automatique d’utilisateurs](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md). Si vous le souhaitez, vous pouvez mettre à jour l’adresse e-mail et le nom d’utilisateur dans SuccessFactors.

## <a name="manage-your-configuration"></a>Gérer votre configuration

Azure AD peut fournir des insights supplémentaires sur l’exploitation de l’approvisionnement d’utilisateurs et son intégrité opérationnelle au sein de votre organisation, par le biais des journaux d’audit et des rapports.

### <a name="gain-insights-from-reports-and-logs"></a>Obtenir des informations à partir des rapports et des journaux

Après l’exécution d’un [cycle initial](../app-provisioning/how-provisioning-works.md#initial-cycle) réussi, le service d’approvisionnement Azure AD continue d’exécuter indéfiniment des mises à jour incrémentielles dos à dos, à des intervalles définis dans les tutoriels propres à chaque application, jusqu’à ce que l’un des événements suivants se produise :

- Le service est arrêté manuellement. Un nouveau cycle initial est déclenché à l’aide du [portail Azure](https://portal.azure.com/) ou de la commande [API Microsoft Graph](/graph/api/resources/synchronization-overview) appropriée.
- Un nouveau cycle initial est déclenché en raison d’une modification dans les mappages d’attributs ou les filtres d’étendue.
- Le processus d’approvisionnement passe en quarantaine en raison d’un taux d’erreur élevé. Il reste en quarantaine pendant plus de quatre semaines, au bout desquelles il est automatiquement désactivé.

Pour passer en revue ces événements, et toutes les autres activités effectuées par le service d’approvisionnement, [découvrez comment consulter les journaux d’activité et obtenir des rapports sur l’activité d’approvisionnement](../app-provisioning/check-status-user-account-provisioning.md).

#### <a name="azure-monitor-logs"></a>Journaux d’activité Azure Monitor

Toutes les activités effectuées par le service de provisionnement sont enregistrées dans les journaux d’audit Azure AD. Vous pouvez acheminer les journaux d’audit Azure AD vers des journaux Azure Monitor pour une analyse plus poussée. Avec les journaux d’activité Azure Monitor (également appelés espace de travail Log Analytics), vous pouvez interroger les données pour trouver des événements, analyser les tendances et effectuer une corrélation entre différentes sources de données. Regardez cette [vidéo](https://youtu.be/MP5IaCTwkQg) pour découvrir les avantages de l’utilisation de journaux d’activité Azure Monitor pour Azure AD dans des scénarios utilisateur pratique.

Installez les [affichages analytiques des journaux d'activité pour les journaux d’activité Azure AD](../reports-monitoring/howto-install-use-log-analytics-views.md). Vous avez ainsi accès à différents [rapports prédéfinis](https://github.com/AzureAD/Deployment-Plans/tree/master/Log%20Analytics%20Views) qui affichent des événements d’approvisionnement qui se produisent dans votre environnement.

Pour plus d’informations, découvrez comment [analyser les journaux d’activité Azure AD à l’aide des journaux Azure Monitor](../reports-monitoring/howto-analyze-activity-logs-log-analytics.md).

### <a name="manage-personal-data"></a>Gérer des données personnelles

L’agent d’approvisionnement Azure AD Connect installé sur le serveur Windows crée des journaux dans le journal des événements Windows qui peuvent contenir des données personnelles selon vos mappages d’attributs de l’application RH cloud vers Active Directory. Pour respecter les obligations de confidentialité de l’utilisateur, configurez une tâche planifiée Windows pour effacer le journal des événements et veiller à ce qu’aucune donnée ne soit conservée plus de 48 heures.

Le service d’approvisionnement Azure AD ne génère pas de rapports, n’effectue pas d’analyses ou ne fournit pas d’insights au-delà de 30 jours, car le service ne stocke pas, ne traite pas et ne conserve aucune donnée au-delà de 30 jours.

### <a name="troubleshoot"></a>Dépanner

Pour résoudre les problèmes susceptibles de survenir au cours de l’approvisionnement, consultez les articles suivants :

- [Problèmes lors de la configuration de l’approvisionnement des utilisateurs pour une application relevant de la galerie Azure AD](application-provisioning-config-problem.md)
- [Synchroniser un attribut entre votre instance Active Directory local et Azure AD pour le provisionnement d’une application](user-provisioning-sync-attributes-for-mapping.md)
- [Problème d’enregistrement des informations d’identification d’administrateur lors de la configuration de l’approvisionnement des utilisateurs pour une application de galerie Azure Active Directory](./user-provisioning.md)
- [Aucun utilisateur n’est attribué à une application de la galerie Azure AD](application-provisioning-config-problem-no-users-provisioned.md)
- [Un mauvais ensemble d’utilisateurs est attribué à une application de la galerie Azure AD](../manage-apps/add-application-portal-assign-users.md)
- [Configuration de l'observateur d'événements Windows pour résoudre les problèmes liés à l'agent](../saas-apps/workday-inbound-tutorial.md#setting-up-windows-event-viewer-for-agent-troubleshooting)
- [Configuration des journaux d’activité d’audit du portail Azure pour résoudre les problèmes de service](../saas-apps/workday-inbound-tutorial.md#setting-up-azure-portal-audit-logs-for-service-troubleshooting)
- [Familiarisation avec les journaux d’activité pour les opérations de création de compte d’utilisateur AD](../saas-apps/workday-inbound-tutorial.md#understanding-logs-for-ad-user-account-create-operations)
- [Familiarisation avec les journaux d’activité pour les opérations de mise à jour de l’attribut manager](../saas-apps/workday-inbound-tutorial.md#understanding-logs-for-manager-update-operations)
- [Résolution des erreurs les plus fréquentes](../saas-apps/workday-inbound-tutorial.md#resolving-commonly-encountered-errors)

### <a name="next-steps"></a>Étapes suivantes

- [Écriture d’expressions pour les mappages d’attributs](functions-for-customizing-application-data.md)
- [Azure AD synchronization API overview](/graph/api/resources/synchronization-overview) (Vue d’ensemble de l’API de synchronisation Azure AD)
- [Ignorer la suppression des comptes d’utilisateurs qui sortent de l’étendue](skip-out-of-scope-deletions.md)
- [Agent de provisionnement Azure AD Connect : Historique de publication des versions](provisioning-agent-release-version-history.md)