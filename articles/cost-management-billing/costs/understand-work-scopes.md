---
title: Comprendre et utiliser les étendues d’Azure Cost Management
description: Cet article vous explique les étendues de facturation et de gestion des ressources disponibles dans Azure, et comment les utiliser dans Cost Management et les API.
author: bandersmsft
ms.author: banders
ms.date: 08/12/2020
ms.topic: conceptual
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: micflan
ms.custom: ''
ms.openlocfilehash: 61a8dbbdde87af792bb4337660d4d3c34f7cf802
ms.sourcegitcommit: 33368ca1684106cb0e215e3280b828b54f7e73e8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92132293"
---
# <a name="understand-and-work-with-scopes"></a>Comprendre et utiliser des étendues

Cet article vous explique les étendues de facturation et de gestion des ressources disponibles dans Azure, et comment les utiliser dans Cost Management et les API.

## <a name="scopes"></a>Étendues

Une _étendue_ est un nœud de la hiérarchie des ressources Azure dans lequel les utilisateurs Azure AD peuvent accéder à des services et les gérer. La plupart des ressources Azure sont créées et déployées dans des groupes de ressources, qui font partie des abonnements. Microsoft propose également deux hiérarchies en plus des abonnements Azure qui offrent des rôles spécialisés pour gérer les données de facturation :
- Les données de facturation, telles que les factures et paiements
- Des services cloud, comme la gouvernance des coûts et des stratégies

Les étendues vous permettent de gérer les données de facturation, de définir des rôles propres aux paiements, d’afficher les factures et de gérer les comptes de manière générale. Les rôles de facturation et de gestion des comptes sont gérés séparément des rôles utilisés pour la gestion des ressources, qui utilisent [Azure RBAC](../../role-based-access-control/overview.md). Pour distinguer clairement l’intention des différentes étendues, y compris les différences en termes de contrôle d’accès, elles sont désignées comme _étendues de facturation_ et _étendues Azure RBAC_, respectivement.

Pour en savoir plus sur les étendues, regardez la vidéo [Cost Management - Configuration de hiérarchies](https://www.youtube.com/watch?v=n3TLRaYJ1NY). Pour regarder d’autres vidéos, consultez la [chaîne YouTube relative à Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/n3TLRaYJ1NY]

## <a name="how-cost-management-uses-scopes"></a>Utilisation des étendues par Cost Management

Cost Management s’applique à toutes les étendues au-dessus des ressources pour permettre aux organisations de gérer les coûts au niveau auquel elles ont accès, qu’il s’agisse du compte de facturation dans son ensemble ou d’un groupe de ressources unique. Contrairement aux étendues de facturation qui varient en fonction de votre contrat Microsoft (type d’abonnement), les étendues Azure RBAC ne changent pas.

## <a name="azure-rbac-scopes"></a>Étendues RBAC Azure

Azure prend en charge trois étendues pour la gestion des ressources. Chaque étendue prend en charge la gestion des accès et de la gouvernance, y compris, mais pas seulement, la gestion des coûts.

- [**Groupes d’administration**](../../governance/management-groups/overview.md): conteneurs hiérarchiques, jusqu’à huit niveaux, pour organiser les abonnements Azure.

    Type de ressource : [Microsoft.Management/managementGroups](/rest/api/resources/managementgroups)

- **Abonnements** : conteneurs principaux pour les ressources Azure.

    Type de ressource : [Microsoft.Resources/subscriptions](/rest/api/resources/subscriptions)

- [**Groupes de ressources**](../../azure-resource-manager/management/overview.md#resource-groups) : regroupements logiques de ressources liées à une solution Azure qui partagent le même cycle de vie. Par exemple, des ressources qui sont déployées et supprimées ensemble.

    Type de ressource : [Microsoft.Resources/subscriptions/resourceGroups](/rest/api/resources/resourcegroups)

Les groupes d’administration vous permettent d’organiser les abonnements dans une hiérarchie. Par exemple, vous pouvez créer une hiérarchie d’organisation logique à l’aide de groupes d’administration. Ensuite, vous pouvez distribuer des abonnements aux équipes pour les charges de travail de production et de développement/test. Puis vous pouvez créer des groupes de ressources dans les abonnements pour gérer chaque sous-système ou composant.

La création d'une hiérarchie organisationnelle permet de déployer une conformité des coûts et des stratégies au niveau de l'organisation. Par la suite, chaque responsable peut afficher et analyser ses coûts actuels. Ils peuvent ensuite créer des budgets pour maîtriser les mauvaises habitudes de dépenses et optimiser les coûts à l’aide des recommandations Advisor au niveau le plus bas.

L'octroi de l'accès à l'affichage des coûts et éventuellement à la gestion de la configuration des coûts, par exemple les budgets et les exportations, s'effectue au niveau des étendues de gouvernance à l'aide des rôles Azure RBAC. Les rôles Azure RBAC permettent d'accorder aux utilisateurs et aux groupes Azure AD la permission d'effectuer un ensemble d'actions prédéterminées, définies dans un rôle sur une étendue spécifique et sur les niveaux inférieurs. Par exemple, un rôle affecté à une étendue de groupes de gestion accorde également les mêmes autorisations aux abonnements et aux groupes de ressources imbriqués.

Cost Management prend en charge les rôles intégrés indiqués ci-dessous pour chacune des étendues suivantes :

- [**Propriétaire**](../../role-based-access-control/built-in-roles.md#owner) : peut afficher les coûts et tout gérer, y compris la configuration des coûts.
- [**Collaborateur**](../../role-based-access-control/built-in-roles.md#contributor) : peut afficher les coûts et tout gérer, y compris la configuration des coûts, à l’exclusion du contrôle d’accès.
- [**Lecteur**](../../role-based-access-control/built-in-roles.md#reader) : peut tout afficher, y compris les données et la configuration des coûts, mais ne peut pas apporter de modifications.
- [**Contributeur Cost Management**](../../role-based-access-control/built-in-roles.md#cost-management-contributor) : peut afficher les coûts, gérer la configuration des coûts et afficher les recommandations.
- [**Lecteur Cost Management**](../../role-based-access-control/built-in-roles.md#cost-management-reader) : peut afficher les données de coûts, la configuration des coûts et afficher les recommandations.

Le rôle recommandé avec le niveau de privilège minimum est celui de Contributeur Cost Management. Le rôle permet aux utilisateurs de créer et de gérer les budgets et les exportations pour une surveillance et une création de rapports plus efficaces sur les coûts. Les Contributeurs Cost Management peuvent également avoir besoin de rôles supplémentaires pour prendre en charge les scénarios complexes de gestion des coûts. Examinez les scénarios suivants :

- **Création de rapports sur l'utilisation des ressources** : Azure Cost Management affiche le coût sur le portail Azure. Il comprend l'utilisation telle qu'elle se rapporte au coût dans les modèles d'utilisation complets. Ce rapport peut également afficher les frais relatifs aux API et aux téléchargements, mais vous pouvez aussi explorer les métriques d’utilisation détaillées dans Azure Monitor pour une compréhension approfondie. Envisagez d'accorder l'autorisation [Lecteur de surveillance](../../role-based-access-control/built-in-roles.md#monitoring-reader) à toute étendue pour laquelle vous devez également signaler des métriques d'utilisation détaillées.
- **Agir de cas de dépassement des budgets** : les Contributeurs Cost Management ont également besoin d'un accès pour créer et gérer des groupes d'actions permettant de réagir automatiquement aux dépassements. Envisagez d’accorder le rôle [Contributeur de surveillance](../../role-based-access-control/built-in-roles.md#monitoring-contributor) à un groupe de ressources contenant le groupe d’actions à utiliser en cas de dépassement des seuils budgétaires. L’automatisation d’actions données nécessite des rôles supplémentaires pour les services spécifiques utilisés, comme Automation et Azure Functions.
- **Planifier l’exportation des données de coût** : les Contributeurs Cost Management ont également besoin d’un accès à la gestion des comptes de stockage afin de planifier une exportation pour copier les données dans un compte de stockage. Envisagez d’accorder le rôle [Collaborateur de compte de stockage](../../role-based-access-control/built-in-roles.md#storage-account-contributor) à un groupe de ressources contenant le compte de stockage vers lequel les données de coût de coûts seront exportées.
- **Affichage des recommandations pour réaliser des économies** : les Lecteurs Cost Management et les Contributeurs Cost Management peuvent *afficher* les recommandations en matière de coûts par défaut. Toutefois, la possibilité de suivre ces recommandations requiert l’accès aux ressources individuelles. Envisagez d’accorder un [rôle propre au service](../../role-based-access-control/built-in-roles.md#all) si vous souhaitez suivre une recommandation en matière de coûts.

Les groupes d’administration sont pris en charge uniquement s’ils contiennent des abonnements Accord Entreprise (EA), Paiement à l’utilisation (PAYG) ou un abonnement interne Microsoft. Les groupes d'administration associés à d'autres types d'abonnements, comme le Contrat client Microsoft ou les abonnements Azure Active Directory, ne peuvent pas afficher les coûts. Si vous disposez d’un mélange d’abonnements, déplacez les abonnements non pris en charge vers une autre branche de la hiérarchie du groupe d’administration pour activer Cost Management sur les abonnements pris en charge. Par exemple, créez deux groupes d’administration sous le groupe d’administration racine : **Azure AD** et **Mon organisation**. Déplacez votre abonnement Azure AD vers le groupe d’administration **Azure AD**, puis affichez et gérez les coûts à l’aide du groupe d’administration **Mon organisation**.

## <a name="enterprise-agreement-scopes"></a>Étendues Contrat Entreprise

Les comptes de facturation Contrat Entreprise (EA), également appelés inscriptions, offrent les étendues suivantes :

- [**Compte de facturation**](../manage/view-all-accounts.md) : représente une inscription EA. Les factures sont générées dans cette étendue. Les achats qui ne sont pas basés sur l’utilisation (par exemple Place de marché et réservations), sont uniquement disponibles dans cette étendue. Ils ne sont pas représentés dans les départements ou les comptes d’inscription. L’utilisation de la réservation, de même que toute autre utilisation, est appliquée à des ressources individuelles. L’utilisation est cumulée dans les abonnements présents le compte de facturation. Pour voir les coûts de réservation répartis entre chaque ressource, basculez vers la vue **Coût amorti** dans l’analyse des coûts.

    Type de ressource : `Microsoft.Billing/billingAccounts (accountType = Enrollment)`
- **Département** : regroupement facultatif des comptes d’inscription.

    Type de ressource : `Billing/billingAccounts/departments`

- **Compte d’inscription** : représente un seul propriétaire de compte. Ne prend pas en charge l’octroi d’un accès à plusieurs personnes.

    Type de ressource : `Microsoft.Billing/billingAccounts/enrollmentAccounts`

Bien que les étendues de gouvernance soient liées à un seul répertoire, ce n’est pas le cas pour les étendues de facturation EA. Un compte de facturation EA peut compter des abonnements sur n’importe quelle quantité de répertoires Azure AD.

Les étendues de facturation EA prennent en charge les rôles suivants :

- **Administrateur d’entreprise** : peut gérer les paramètres des comptes de facturation et l’accès à ces derniers, peut afficher tous les coûts et peut gérer la configuration des coûts. Par exemple, les budgets et les exportations. En pratique, l’étendue de facturation de Contrat Entreprise est identique au [rôle Azure Contributeur Cost Management](../../role-based-access-control/built-in-roles.md#cost-management-contributor).
- **Utilisateur d’entreprise en lecture seule** : peut afficher les paramètres de compte de facturation, les données de coûts et la configuration des coûts. Par exemple, les budgets et les exportations. En pratique, l’étendue de facturation de Contrat Entreprise est identique au [rôle Azure Lecteur Cost Management](../../role-based-access-control/built-in-roles.md#cost-management-reader).
- **Administrateur de service** : peut gérer les paramètres des départements, comme le centre de coûts, peut afficher tous les coûts et gérer la configuration des coûts. Par exemple, les budgets et les exportations.  Le paramètre de compte de facturation **Affichage des frais pour l’administrateur de service** doit être activé pour que les administrateurs de service et les utilisateurs en lecture seule puissent afficher les coûts. Si l'option **Affichage des frais pour l'administrateur de service** est désactivée, les utilisateurs du département ne peuvent afficher les coûts à aucun niveau, même s'ils sont propriétaires de compte ou d'abonnement.
- **Utilisateur de service en lecture seule** : peut afficher les paramètres de département, les données de coûts et la configuration des coûts. Par exemple, les budgets et les exportations. Si l'option **Affichage des frais pour l'administrateur de service** est désactivée, les utilisateurs du département ne peuvent afficher les coûts à aucun niveau, même s'ils sont propriétaires de compte ou d'abonnement.
- **Propriétaire du compte** : peut gérer les paramètres de compte d’inscription (comme le centre de coûts), afficher tous les coûts et gérer la configuration des coûts (par exemple les budgets et les exportations) pour le compte d’inscription. Le paramètre de compte de facturation **Affichage des frais pour le propriétaire du compte** doit être activé pour que les propriétaires de compte et les utilisateurs Azure RBAC puissent afficher les coûts.

Les utilisateurs de compte de facturation EA n’ont pas d’accès direct aux factures. Les factures sont disponibles à partir d’un système externe de gestion des licences en volume.

Les abonnements Azure sont imbriqués sous des comptes d’inscription. Les utilisateurs de facturation ont accès aux données de coûts concernant les abonnements et les groupes de ressources se trouvant dans leurs étendues respectives. Ils ne peuvent pas afficher ou gérer les ressources dans le Portail Azure. Les utilisateurs peuvent afficher les coûts en accédant à **Gestion des coûts + facturation** dans la liste de services du portail Azure. Ensuite, ils peuvent filtrer les coûts sur les abonnements et groupes de ressources spécifiques dont ils ont besoin pour créer leurs rapports.

Les utilisateurs de facturation n’ont pas accès aux groupes d’administration, car ils ne relèvent pas explicitement d’un compte de facturation spécifique. Cet accès doit être accordé explicitement aux groupes d’administration. Les groupes d’administration déploient les coûts à partir de tous les abonnements imbriqués. Toutefois, ils ne comprennent que les achats basés sur l’utilisation. Ils n’incluent pas les achats comme les réservations et les offres tierces de la Place de marché. Pour afficher ces coûts, utilisez le compte de facturation EA.

## <a name="individual-agreement-scopes"></a>Étendues Contrat individuel

Les abonnements Azure créés à partir d’offres individuelles, comme le paiement à l’utilisation et les types d’offres connexes comme les essais gratuits et les offres dev/test, n’ont pas d’étendue de compte de facturation explicite. Au lieu de cela, chaque abonnement possède un propriétaire de compte ou un administrateur de compte, comme le propriétaire de compte EA.

- [**Compte de facturation**](../manage/view-all-accounts.md) : représente un seul propriétaire de compte pour un ou plusieurs abonnements Azure. Il ne prend actuellement pas en charge l’octroi de l’accès à plusieurs personnes ou l’accès aux vues agrégées des coûts.

    Type de ressource : Non applicable

Les administrateurs de comptes d’abonnement Azure individuels peuvent afficher et gérer les données de facturation, telles que les factures et les paiements, à partir du [Centre des comptes Azure](https://account.azure.com/subscriptions). Toutefois, ils ne peuvent pas afficher les données de coûts ou gérer les ressources dans le Portail Azure. Pour accorder l’accès à l’administrateur de compte, utilisez les rôles Cost Management mentionnés précédemment.

Contrairement au Contrat Entreprise, les administrateurs de comptes d’abonnement Azure individuels peuvent voir leurs factures dans le Portail Azure. N’oubliez pas que les rôles Lecteur Cost Management et Contributeur Cost Management ne donnent pas accès aux factures. Pour plus d’informations, consultez l’article expliquant [comment accorder l’accès aux factures](../manage/manage-billing-access.md#give-read-only-access-to-billing).

## <a name="microsoft-customer-agreement-scopes"></a>Étendues Contrat client Microsoft

Les comptes de facturation Contrat client Microsoft présentent les étendues suivantes :

- **Compte de facturation** : représente un contrat client pour plusieurs produits et services Microsoft. En pratique, les comptes de facturation de contrat client ne sont pas identiques aux inscriptions EA. Les inscriptions EA sont plus proches des profils de facturation.

    Type de ressource : `Microsoft.Billing/billingAccounts (accountType = Organization)`

- **Profil de facturation** : définit les abonnements inclus dans une facture. Les profils de facturation sont l’équivalent fonctionnel d’une inscription EA, car il s’agit de l’étendue dans laquelle les factures sont générées. De même, les achats qui ne sont pas basés sur l’utilisation (par exemple Place de marché et réservations), sont uniquement disponibles dans cette étendue. Ils ne sont pas inclus dans les sections de facture.

    Type de ressource : `Microsoft.Billing/billingAccounts/billingProfiles`

- **Section de facture** : représente un groupe d’abonnements dans une facture ou un profil de facturation. Les sections de facture fonctionnent comme des départements : plusieurs personnes peuvent avoir accès à une section de facture.

    Type de ressource : `Microsoft.Billing/billingAccounts/invoiceSections`

- **Client**  : représente un groupe d’abonnements associés à un client spécifique qui est intégré à un contrat client Microsoft par un partenaire. Cette étendue est spécifique aux fournisseurs de solutions Cloud (CSP).

Contrairement aux étendues de facturation EA, les comptes de facturation de contrat client _sont_ liés à un répertoire unique et ne peuvent pas posséder des abonnements sur plusieurs annuaires Azure AD.

Les étendues de facturation de Contrat client ne s’appliquent pas aux partenaires. Les autorisations et rôles des partenaires sont décrits dans [Affecter des rôles et des autorisations aux utilisateurs](/partner-center/permissions-overview).

Les étendues de facturation de contrat client prennent en charge les rôles suivants :

- **Propriétaire** : peut gérer les paramètres de facturation et l’accès à cette dernière, afficher tous les coûts et gérer la configuration des coûts. Par exemple, les budgets et les exportations. En pratique, cette étendue de facturation de Contrat client est identique au [rôle Azure Contributeur Cost Management](../../role-based-access-control/built-in-roles.md#cost-management-contributor).
- **Collaborateur** : peut gérer les paramètres de facturation, mais pas l’accès à cette dernière, afficher tous les coûts et gérer la configuration des coûts. Par exemple, les budgets et les exportations. En pratique, cette étendue de facturation de Contrat client est identique au [rôle Azure Contributeur Cost Management](../../role-based-access-control/built-in-roles.md#cost-management-contributor).
- **Lecteur** : peut afficher les paramètres de facturation, les données de coûts et la configuration des coûts. Par exemple, les budgets et les exportations. En pratique, cette étendue de facturation de Contrat client est identique au [rôle Azure Lecteur Cost Management](../../role-based-access-control/built-in-roles.md#cost-management-reader).
- **Gestionnaire de factures** : peut afficher et payer les factures, et peut afficher les données et la configuration des coûts. Par exemple, les budgets et les exportations. En pratique, cette étendue de facturation de Contrat client est identique au [rôle Azure Lecteur Cost Management](../../role-based-access-control/built-in-roles.md#cost-management-reader).
- **Créateur de l’abonnement Azure** : peut créer des abonnements Azure, afficher les coûts et gérer la configuration des coûts. Par exemple, les budgets et les exportations. En pratique, cette étendue de facturation de contrat client est identique au rôle de propriétaire de compte d’inscription EA.

Les abonnements Azure sont imbriqués sous les sections de facture, comme ils le sont dans les comptes d’inscription EA. Les utilisateurs de facturation ont accès aux données de coûts concernant les abonnements et les groupes de ressources se trouvant dans leurs étendues respectives. Toutefois, ils ne peuvent pas afficher ou gérer les ressources dans le Portail Azure. Les utilisateurs de facturation peuvent afficher les coûts en accédant à **Gestion des coûts + facturation** dans la liste de services du Portail Azure. Ensuite, ils peuvent filtrer les coûts sur les abonnements et groupes de ressources spécifiques dont ils ont besoin pour créer leurs rapports.

Les utilisateurs de facturation n’ont pas accès aux groupes d’administration, car ils ne relèvent pas explicitement du compte de facturation. Toutefois, lorsque des groupes d'administration sont activés pour l'organisation, tous les coûts d'abonnement sont cumulés sur le compte de facturation et dans le groupe d'administration racine, car ils sont tous deux limités à un répertoire unique. Les groupes d’administration incluent uniquement les achats basés sur l’utilisation. Les achats tels que les réservations et les offres tierces de la Place de marché ne sont pas inclus dans les groupes d’administration. Par conséquent, le compte de facturation et le groupe d’administration racine peuvent signaler des totaux différents. Pour afficher ces coûts, utilisez le compte de facturation ou le profil de facturation correspondant.

## <a name="aws-scopes"></a>Étendues AWS

Une fois l’intégration AWS terminée, consultez la section [Installer et configurer l'intégration d’AWS](aws-integration-set-up-configure.md). Les étendues disponibles sont les suivantes :

- **Compte de facturation externe** : représente un contrat de client avec un fournisseur tiers. Semblable au compte de facturation EA.

    Type de ressource : `Microsoft.CostManagement/externalBillingAccounts`

- **Abonnement externe** : représente un compte opérationnel client avec un fournisseur tiers. Semblable à un abonnement Azure.

    Type de ressource : `Microsoft.CostManagement/externalSubscriptions`

## <a name="cloud-solution-provider-csp-scopes"></a>Étendues Fournisseur de solutions cloud (CSP)

Les étendues suivantes sont prises en charge pour les CSP ayant des clients sous Contrat client Microsoft :

- **Compte de facturation** : représente un contrat client pour plusieurs produits et services Microsoft. En pratique, les comptes de facturation de contrat client ne sont pas identiques aux inscriptions EA. Les inscriptions EA sont plus proches des profils de facturation.

    Type de ressource : `Microsoft.Billing/billingAccounts (accountType = Organization)`

- **Profil de facturation** : définit les abonnements inclus dans une facture. Les profils de facturation sont l’équivalent fonctionnel d’une inscription EA, car il s’agit de l’étendue dans laquelle les factures sont générées. De même, les achats qui ne sont pas basés sur l’utilisation (par exemple Place de marché et réservations), sont uniquement disponibles dans cette étendue.

    Type de ressource : `Microsoft.Billing/billingAccounts/billingProfiles`

- **Client**  : représente un groupe d’abonnements associés à un client spécifique qui est intégré à un Contrat client Microsoft par un partenaire.

Seuls les utilisateurs ayant les rôles *Administrateur général* et *Agent d’administration* peuvent gérer et voir les coûts pour les comptes de facturation, les profils de facturation et les clients directement dans le locataire Azure du partenaire. Pour plus d’informations sur les rôles de l’espace Partenaires, consultez [Affecter des rôles et des autorisations aux utilisateurs](/partner-center/permissions-overview).

Azure Cost Management prend en charge uniquement les clients partenaires CSP si ces clients disposent d’un Contrat client Microsoft. Pour les clients pris en charge par un CSP qui ne sont pas encore sous Contrat client Microsoft, consultez l'[Espace partenaires](/azure/cloud-solution-provider/overview/partner-center-overview).

Les groupes d'administration des étendues CSP ne sont pas pris en charge par Cost Management. Si vous disposez d'un abonnement CSP et que vous définissez l'étendue sur un groupe d'administration dans l'analyse des coûts, une erreur semblable à la suivante s'affiche :

`Management group <ManagementGroupName> does not have any valid subscriptions`

## <a name="switch-between-scopes-in-cost-management"></a>Passer d’une étendue à l’autre dans Cost Management

Toutes les vues de Cost Management du Portail Azure incluent un bouton de sélection **Étendue** en haut à gauche. Il permet de modifier rapidement l’étendue. Sélectionnez le bouton **Étendue** pour ouvrir le sélecteur. Il contient les comptes de facturation, le groupe d’administration racine et les abonnements éventuels qui ne sont pas imbriqués sous le groupe d’administration racine. Pour sélectionner une étendue, sélectionnez son arrière-plan afin de la mettre en surbrillance, puis choisissez **Sélectionner** en bas. Pour explorer les étendues imbriquées, comme les groupes de ressources d'un abonnement, sélectionnez le lien du nom de l'étendue. Pour sélectionner l'étendue parente à tous les niveaux imbriqués, choisissez **Sélectionner cette &lt;étendue&gt;** en haut du sélecteur.

## <a name="identify-the-resource-id-for-a-scope"></a>Identifier l’ID de ressource d’une étendue

Lorsque vous utilisez les API Cost Management, il est essentiel d’en connaître les étendues. Utilisez les informations suivantes pour générer l’URI d’étendue appropriée pour les API Cost Management.

### <a name="billing-accounts"></a>Comptes de facturation

1. Ouvrez le Portail Azure et accédez à **Gestion des coûts + facturation** dans la liste des services.
2. Dans le menu du compte de facturation, sélectionnez **Propriétés**.
3. Copiez l’ID du compte de facturation.
4. Votre étendue est : `"/providers/Microsoft.Billing/billingAccounts/{billingAccountId}"`

### <a name="billing-profiles"></a>Profils de facturation

1. Ouvrez le Portail Azure et accédez à **Gestion des coûts + facturation** dans la liste des services.
2. Dans le menu du compte de facturation, sélectionnez **Profils de facturation**.
3. Sélectionnez le nom du profil de facturation.
4. Dans le menu du profil de facturation, sélectionnez **Propriétés**.
5. Copiez les ID du compte de facturation et du profil de facturation.
6. Votre étendue est : `"/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/billingProfiles/{billingProfileId}"`

### <a name="invoice-sections"></a>Sections de facture

1. Ouvrez le Portail Azure et accédez à **Gestion des coûts + facturation** dans la liste des services.
2. Dans le menu du compte de facturation, sélectionnez **Sections de facture**.
3. Sélectionnez le nom de la section de facture.
4. Dans le menu de la section de facture, sélectionnez **Propriétés**.
5. Copiez les ID du compte de facturation et de la section de facture.
6. Votre étendue est : `"/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/invoiceSections/{invoiceSectionId}"`

### <a name="ea-departments"></a>Départements EA

1. Ouvrez le Portail Azure et accédez à **Gestion des coûts + facturation** dans la liste des services.
2. Dans le menu du compte de facturation, sélectionnez **Départements**.
3. Sélectionnez le nom du département.
4. Dans le menu du département, sélectionnez **Propriétés**.
5. Copiez les ID du compte de facturation et du département.
6. Votre étendue est : `"/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/departments/{departmentId}"`

### <a name="ea-enrollment-account"></a>Compte d’inscription EA

1. Ouvrez le Portail Azure et accédez à **Gestion des coûts + facturation** dans la liste des services.
2. Dans le menu du compte de facturation, sélectionnez **Comptes d’inscription**.
3. Sélectionnez le nom du compte d'inscription.
4. Dans le menu du compte d’inscription, sélectionnez **Propriétés**.
5. Copiez les ID du compte de facturation et du compte d’inscription.
6. Votre étendue est : `"/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/enrollmentAccounts/{enrollmentAccountId}"`

### <a name="management-group"></a>Groupe d’administration

1. Ouvrez le Portail Azure et accédez à **Groupes d’administration** dans la liste des services.
2. Accédez au groupe d'administration.
3. Copiez l’ID du groupe d’administration affiché dans le tableau.
4. Votre étendue est : `"/providers/Microsoft.Management/managementGroups/{id}"`

### <a name="subscription"></a>Abonnement

1. Ouvrez le Portail Azure et accédez à **Abonnements** dans la liste des services.
2. Copiez l’ID de l’abonnement affiché dans le tableau.
3. Votre étendue est : `"/subscriptions/{id}"`

### <a name="resource-groups"></a>Groupes de ressources

1. Ouvrez le Portail Azure et accédez à **Groupes de ressources** dans la liste des services.
2. Sélectionnez le nom du groupe de ressources.
3. Dans le menu du groupe de ressources, sélectionnez **Propriétés**.
4. Copiez la valeur du champ ID de la ressource.
5. Votre étendue est : `"/subscriptions/{id}/resourceGroups/{name}"`

Cost Management est actuellement pris en charge dans [Azure Global](https://management.azure.com) et [Azure Government](https://management.usgovcloudapi.net). Pour plus d’informations sur Azure Government, consultez l’article sur les [points de terminaison d’API Azure Global et Government](../../azure-government/documentation-government-developer-guide.md#endpoint-mapping).

## <a name="next-steps"></a>Étapes suivantes

- Si vous n’avez pas encore effectué le premier guide de démarrage rapide relatif à Cost Management, lisez-le à partir de [Démarrer l’analyse des coûts](quick-acm-cost-analysis.md).