---
title: Gérer le coût et l’utilisation d’AWS dans Azure Cost Management
description: Cet article explique comment utiliser les fonctionnalités d’analyse des budgets et des coûts dans Cost Management pour gérer le coût et votre utilisation d’AWS.
author: bandersmsft
ms.author: banders
ms.date: 10/16/2020
ms.topic: how-to
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: matrive
ms.custom: ''
ms.openlocfilehash: 5fed70ccdbebbd178412c416f37c2e9001a81f38
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96188103"
---
# <a name="manage-aws-costs-and-usage-in-azure"></a>Gérer le coût et l’utilisation d’AWS dans Azure

Après avoir configuré l’intégration du rapport sur le coût et l’utilisation d’AWS pour Azure Cost Management, vous pouvez gérer votre coût et utilisation d’AWS. Cet article explique comment utiliser les fonctionnalités d’analyse des budgets et des coûts dans Cost Management pour gérer le coût et votre utilisation d’AWS.

Si vous n’avez pas déjà configuré l’intégration, consultez [Configurer l’intégration du rapport d’utilisation d’AWS](aws-integration-set-up-configure.md).

_Avant de commencer_ : Si vous n’êtes pas familiarisé avec l’analyse des coûts, consultez le guide de démarrage rapide [Explorer et analyser les coûts avec l’analyse du coût](quick-acm-cost-analysis.md). Enfin, si vous n’êtes pas familiarisé avec les budgets dans Azure, consultez le didacticiel [Créer et gérer des budgets Azure](tutorial-acm-create-budgets.md).

## <a name="view-aws-costs-in-cost-analysis"></a>Afficher les coûts d’AWS dans l’analyse du coût

Aux coûts d’AWS sont disponibles dans les portées suivantes de la fonctionnalité Analyse du coût :

- Comptes AWS liés sous un groupe d’administration
- Coûts de compte AWS lié
- Coûts de compte AWS consolidés

Les sections suivantes décrivent comment utiliser les portées pour afficher chaque donnée d’utilisation et de coût.

### <a name="view-aws-linked-accounts-under-a-management-group"></a>Afficher les comptes AWS liés sous un groupe d’administration

Le seul moyen de voir les coûts agrégés provenant de différents abonnements Azure et comptes liés AWS consiste à afficher les coûts avec l’étendue du groupe d’administration, qui permet d’obtenir une vue multicloud des coûts d’Azure et AWS.

Dans Analyse du coût, ouvrez le sélecteur de portée, puis sélectionnez le groupe d’administration qui contient vos comptes AWS liés. Voici une capture d’écran d’exemple dans le portail Microsoft Azure :

:::image type="content" source="./media/aws-integration-manage/select-scope01.png" alt-text="Exemple de vue Sélectionnez une étendue avec des comptes liés sous un groupe d’administration" :::

Voici un exemple indiquant le coût de groupe d’administration dans Analyse du coût, classé par fournisseur (Azure et AWS).

:::image type="content" source="./media/aws-integration-manage/cost-analysis-aws-azure.png" alt-text="Exemple illustrant les coûts d’Azure et d’AWS pour un trimestre dans Analyse du coût" lightbox="./media/aws-integration-manage/cost-analysis-aws-azure.png" :::

> [!NOTE]
> Les groupes d’administration ne sont actuellement pas pris en charge pour les clients disposant d’un Contrat client Microsoft (MCA). Ces clients peuvent créer le connecteur et visualiser leurs données AWS. Cependant, ils ne peuvent pas voir conjointement leurs coûts Azure et leurs coûts AWS dans un groupe d’administration.

### <a name="view-aws-linked-account-costs"></a>Afficher les coûts d’un compte AWS lié

Pour afficher les coûts d’un compte AWS lié, ouvrez le sélecteur de portée et sélectionnez le compte AWS lié. Notez que les comptes liés sont associés à un groupe d’administration, tel que défini dans le connecteur AWS.

Voici un exemple qui présente la sélection d’une portée d’un compte AWS lié.

:::image type="content" source="./media/aws-integration-manage/select-scope02.png" alt-text="Exemple de vue Sélectionnez une étendue montrant des comptes liés AWS" :::

### <a name="view-aws-consolidated-account-costs"></a>Afficher les coûts d’un compte AWS consolidé

Pour afficher les coûts d’un compte AWS consolidé, ouvrez le sélecteur de portée, puis sélectionnez le compte AWS consolidé. Voici un exemple qui présente la sélection d’une portée d’un compte AWS consolidés.

:::image type="content" source="./media/aws-integration-manage/select-scope03.png" alt-text="Exemple de vue Sélectionnez une étendue avec des comptes consolidés" :::

Cette portée présente une vue agrégée de tous les comptes AWS liés associés au compte AWS consolidé. Voici un exemple illustrant les coûts d’un compte AWS consolidé, classés par nom de service.

:::image type="content" source="./media/aws-integration-manage/cost-analysis-aws-consolidated.png" alt-text="Exemple illustrant les coûts d’un compte AWS consolidé dans Analyse du coût" lightbox="./media/aws-integration-manage/cost-analysis-aws-consolidated.png" :::

### <a name="dimensions-available-for-filtering-and-grouping"></a>Dimensions disponibles pour le filtrage et regroupement

Le tableau suivant décrit les dimensions disponibles pour le regroupement et le filtrage par analyse de coût.

| Dimension | En-tête Amazon CUR | Étendues | Commentaires |
| --- | --- | --- | --- |
| Zone de disponibilité | élémentdeligne/ZoneDisponibilité | Tous |   |
| Emplacement | produit/région | Tous |   |
| Compteur |   | Tous |   |
| Catégorie du compteur | élémentdeligne/CodeProduit | Tous |   |
| Sous-catégorie du compteur | élémentdeligne/TypeUtilisation | Tous |   |
| Opération | élémentdeligne/Opération | Tous |   |
| Ressource | élémentdeligne/IDRessource | Tous |   |
| Type de ressource | produit/TypeInstance | Tous | Si le champ produit/TypeInstance a la valeur NULL, alors le champ élémentdeligne/TypeUtilisation est utilisé. |
| ResourceGuid | N/A | Tous | GUID de compteur Azure. |
| Nom du service | produit/NomProduit | Tous | Si le champ produit/NomProduit est NULL, élémentdeligne/CodeProduit est utilisé. |
| Niveau de service |   |   |   |
| Identifiant d’abonnement | élémentdeligne/IDCompteUtilisation | Compte consolidé et groupe d’administration |   |
| Nom d’abonnement | N/A | Compte consolidé et groupe d’administration | Les noms des comptes sont collectés à l’aide de l’API Organisation AWS. |
| Tag | resourceTags | Tous | Le préfixe _utilisateur:_ est supprimé des balises définies par l’utilisateur pour autoriser les balises dans le cloud. Le préfixe _aws :_ est n’est pas supprimé. |
| ID de compte de facturation | facture/IDComptePayeur | Groupe d’administration |   |
| Nom du compte de facturation | N/A | Groupe d’administration | Les noms des comptes sont collectés à l’aide de l’API Organisation AWS. |
| Fournisseur | N/A | Groupe d’administration | AWS ou Azure. |

## <a name="set-budgets-on-aws-scopes"></a>Définir des budgets sur des portées AWS

Utilisez des budgets pour gérer les coûts de manière proactive et responsabiliser les équipes au sein de votre organisation. Les budgets sont définis sur les portées des comptes AWS consolidés et liés. Voici un exemple de budgets d’un compte AWS consolidé affiché dans Cost Management :

:::image type="content" source="./media/aws-integration-manage/budgets-aws-consolidated-account01.png" alt-text="Exemple présentant des budgets pour un compte AWS consolidé" :::

## <a name="aws-data-collection-process"></a>Processus de collecte de données AWS

Lorsque vous avez configuré le connecteur AWS, les processus de collecte et de découverte des données démarrent. La collecte des données d’utilisation peut prendre quelques heures. La durée dépend :

- Du temps requis pour traiter les fichiers CUR qui se trouvent dans le compartiment AWS S3.
- Du temps requis pour créer le compte AWS consolidé et les portées de comptes AWS liés.
- Du temps et de et de la fréquence des écritures des fichiers de rapports de coût et d’utilisation d’AWS dans le compartiment S3

## <a name="aws-integration-pricing"></a>Tarification de l’intégration d’AWS

Chaque connecteur AWS obtient 90 jours d’essai gratuits.

Le prix catalogue est de 1 % des coûts mensuels d’AWS. Chaque mois, vous êtes facturé en fonction de vos coûts facturés le mois précédent.

L’accès aux API d’AWS peut occasionner des coûts supplémentaires sur AWS.

## <a name="aws-integration-limitations"></a>Limitations de l’intégration d’AWS

- Les budgets de Cost Management ne prennent pas en charge les groupes d’administration comportant plusieurs devises, qui ne comportent pas d’évaluation du budget. Un message d’erreur s’affiche si vous sélectionnez un groupe d’administration multidevise lorsque vous créez un budget.
- Les connecteurs cloud ne prennent pas en charge AWS GovCloud (US), AWS Gov ou AWS China.
- Cost Management affiche uniquement les _coûts d’utilisation_ d’AWS. Les impôts/taxes, remboursements, réassurances, crédits et autres types de frais ne sont pas pris en charge pour le moment.

## <a name="troubleshooting-aws-integration"></a>Résolution des problèmes d’intégration d’AWS

Utilisez les informations de dépannage suivantes pour résoudre les problèmes courants.

### <a name="no-permission-to-aws-linked-accounts"></a>Absence d’autorisation de comptes AWS liés

**Code d’erreur :** _Non autorisé_

Il existe deux façons d’obtenir les autorisations d’accès aux coûts de comptes AWS liés :

- Obtenir l’accès au groupe d’administration qui contient les comptes AWS liés.
- Demandez à une personne de vous accorder l’autorisation d’accès à un compte AWS lié.

Par défaut, le créateur du connecteur AWS est le propriétaire de tous les objets créés par ce connecteur. Notamment le compte AWS consolidé et le compte AWS lié.

Pour pouvoir vérifier les paramètres du connecteur, vous devez disposer au moins d’un rôle de contributeur. Le rôle Lecteur ne peut pas vérifier les paramètres de connecteur

### <a name="collection-failed-with-assumerole"></a>Échec de la collecte avec l’erreur AssumeRole

**Code d’erreur :** _FailedToAssumeRole_

Cette erreur signifie que Cost Management ne peut pas appeler l’API AssumeRole AWS. Ce problème peut se produire en raison d’un problème avec la définition de rôle. Vérifiez que les conditions suivantes sont remplies :

- L’ID externe est identique à celui de la définition de rôle et de la définition de connecteur.
- Le type de rôle est défini sur **un autre compte AWS qui vous appartient ou appartient à un tiers.**
- La case à cocher **MFA requise** est décochée.
- Le compte AWS approuvé dans le rôle AWS est _432263259397_.

### <a name="collection-failed-with-access-denied---cur-report-definitions"></a>Échec de la collecte avec les définitions de rapport Accès refusé - Rapport d’utilisation et de coût

**Code d’erreur :** _AccessDeniedReportDefinitions_

Cette erreur signifie que Cost Management ne peut pas voir les définitions de rapport d’utilisation et de coût. Cette autorisation est utilisée pour valider le fait que le rapport d’utilisation et de coût est défini comme prévu par Azure Cost Management. Consultez [Créer un rapport sur les coûts et l’utilisation dans AWS](aws-integration-set-up-configure.md#create-a-cost-and-usage-report-in-aws).

### <a name="collection-failed-with-access-denied---list-reports"></a>Échec de la collecte avec l’erreur Accès refusé - Lister les rapports

**Code d’erreur :** _AccessDeniedListReports_

Cette erreur signifie que Cost Management ne peut pas répertorier l’objet dans le compartiment S3 où se trouve le rapport d’utilisation et de coût. La stratégie IAM AWS requiert une autorisation sur le compartiment et sur les objets dans le compartiment. Consultez [Créer un rôle et une stratégie dans AWS](aws-integration-set-up-configure.md#create-a-role-and-policy-in-aws).

### <a name="collection-failed-with-access-denied---download-report"></a>Échec de la collecte avec l’erreur Accès refusé - Télécharger le rapport

**Code d’erreur :** _AccessDeniedDownloadReport_

Ce message d’erreur signifie que Cost Management ne peut pas accéder aux fichiers de rapport d’utilisation et de coût stockés dans le compartiment Amazon S3 ni les télécharger. Assurez-vous que la stratégie JSON AWS jointe au rôle ressemble à l’exemple indiqué au bas de la section [Créer un rôle et une stratégie dans AWS](aws-integration-set-up-configure.md#create-a-role-and-policy-in-aws).

### <a name="collection-failed-since-we-did-not-find-the-cost-and-usage-report"></a>Échec de la collecte car nous n’avons pas trouvé le rapport sur le coût et l’utilisation

**Code d’erreur :** _FailedToFindReport_

Cette erreur indique que Cost Management ne peut pas trouver le rapport de coût et d’utilisation qui a été défini dans le connecteur. Assurez-vous qu’il n’est pas supprimé et que la stratégie JSON AWS liée au rôle ressemble à l’exemple indiqué au bas de la section [Créer un rôle et une stratégie dans AWS](aws-integration-set-up-configure.md#create-a-role-and-policy-in-aws).

### <a name="unable-to-create-or-verify-connector-due-to-cost-and-usage-report-definitions-mismatch"></a>Impossible de créer ou de vérifier le connecteur en raison d’une incompatibilité de définitions de rapport sur le coût et l’utilisation

**Code d’erreur :** _ReportIsNotValid_

Cette erreur est liée à la définition du rapport sur les coûts et l’utilisation d’AWS. Nous exigeons des paramètres spécifiques pour ce rapport. Consultez les spécifications dans [Créer un rapport sur les coûts et l’utilisation dans AWS](aws-integration-set-up-configure.md#create-a-cost-and-usage-report-in-aws).

### <a name="internal-error-when-creating-connector"></a>Erreur interne lors de la création du connecteur

**Code d’erreur :** _Créer un connecteur – Impossible de créer le connecteur &lt;ConnectorName&gt;. Motif : Erreur interne. Vérifiez que les propriétés AWS correctes ont été fournies._

Cette erreur peut se produire lorsque votre connecteur AWS et votre abonnement figurent dans des groupes d’administration différents. L’abonnement et le connecteur AWS doivent être dans le même groupe d’administration.

## <a name="next-steps"></a>Étapes suivantes

- Si vous n’avez pas déjà configuré votre environnement Azure avec des groupes d’administration, veuillez consulter la section [Configuration initiale des groupes d’administration](../../governance/management-groups/overview.md#initial-setup-of-management-groups).
