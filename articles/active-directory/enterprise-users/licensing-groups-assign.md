---
title: Attribuer des licences à un groupe - Azure Active Directory | Microsoft Docs
description: Comment affecter des licences à l’aide d’une gestion des licences de groupe Azure Active Directory
services: active-directory
keywords: Gestion des licences Azure AD
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: enterprise-users
ms.topic: how-to
ms.workload: identity
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: sumitp
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1c1a6957d347681f1c7dd9979429fffb153ccb39
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96546605"
---
# <a name="assign-licenses-to-users-by-group-membership-in-azure-active-directory"></a>Affecter des licences aux utilisateurs par appartenance aux groupes dans Azure Active Directory

Cet article présente l’affectation de licences de produit à un groupe d’utilisateurs, puis la vérification que les licences ont été correctement reçues dans Azure Active Directory (Azure AD).

Dans cet exemple, l’organisation Azure AD contient un groupe de sécurité appelé **HR Department**. Ce groupe inclut tous les membres du département des ressources humaines (environ 1 000 utilisateurs). Vous souhaitez attribuer des licences Office 365 Entreprise E3 à l’ensemble du département. Le service Yammer Enterprise inclus dans le produit doit être temporairement désactivé jusqu’à ce que le département soit prêt à l’utiliser. L’administrateur souhaite également déployer des licences Enterprise Mobility + Security vers le même groupe d’utilisateurs.

> [!NOTE]
> Certains services Microsoft ne sont pas disponibles dans tous les emplacements. Avant de pouvoir attribuer une licence à un utilisateur, l’administrateur doit spécifier la propriété Emplacement d’utilisation sur l’utilisateur.
>
> Pour l’affectation d’une licence à un groupe, tous les utilisateurs sans emplacement d’utilisation spécifié héritent de l’emplacement du répertoire. Si vos utilisateurs sont situés à plusieurs emplacements, nous vous recommandons de définir systématiquement l’emplacement d’utilisation dans le cadre de votre flux de création d’utilisateur dans Azure AD (par exemple, par le biais d’une configuration AAD Connect). Cela garantit que le résultat de l’affectation de licence est toujours correct et que les utilisateurs ne reçoivent pas de services à des emplacements non autorisés.

## <a name="step-1-assign-the-required-licenses"></a>Étape 1 : Affecter les licences requises

1. Connectez-vous au [**Centre d’administration Azure AD**](https://aad.portal.azure.com) avec un compte administrateur de licences. Pour gérer les licences, le compte doit être un rôle administrateur de licences, administrateur d’utilisateurs ou administrateur général.

1. Sélectionnez **Licences** afin d’ouvrir une page vous permettant de voir et gérer tous les produits sous licence dans l’organisation.

1. Sous **Tous les produits**, sélectionnez Office 365 Enterprise E5 et Enterprise Mobility + Security E3 en sélectionnant les noms de produits. Pour démarrer l’affectation, sélectionnez **Affecter** en haut de la page.

   ![Sélectionnez les produits pour attribuer les licences](./media/licensing-groups-assign/licenses-all-products-assign.png)
  
1. Dans la page **Affecter une licence**, sélectionnez **Utilisateurs et groupes** pour ouvrir la liste des utilisateurs et groupes.

1. Sélectionnez un utilisateur ou groupe, puis utilisez le bouton **Sélectionner** en bas de la page pour confirmer votre sélection.

1. Dans la page **Affecter une licence**, cliquez sur **Options d’affectation**, ce qui affiche tous les plans de services inclus dans les deux produits que nous avons sélectionnés précédemment. Recherchez **Yammer Enterprise** et définissez-le sur **Désactivé** pour désactiver ce service à partir de la licence du produit. Confirmez en cliquant sur **OK** en bas des **options de licence**.

   ![Sélectionnez les plans de service pour les licences](./media/licensing-groups-assign/assignment-options.png)
  
1. Pour procéder à l’affectation, au bas de la page **Affecter une licence**, cliquez sur **Affecter**.

1. Une notification indiquant l’état et le résultat du processus s’affiche dans l’angle supérieur droit. Si l’affectation au groupe n’a pas pu être effectuée (par exemple en raison de licences existantes dans le groupe), cliquez sur la notification pour afficher les détails de l’échec.

Lorsque vous attribuez des licences à un groupe, Azure AD traite tous les membres existants de ce groupe. Ce processus peut prendre un certain temps, en fonction de la taille du groupe. L’étape suivante décrit comment vérifier que le processus est terminé, et comment déterminer si une attention supplémentaire est requise pour résoudre des problèmes.

## <a name="step-2-verify-that-the-initial-assignment-has-finished"></a>Étape 2 : Vérifier que l’affectation initiale est terminée

1. Accédez à **Azure Active Directory** > **Groupes**. Sélectionnez le groupe auquel les licences ont été assignées.

1. Dans la page du groupe, sélectionnez **Licences**. Cela vous permet de vérifier rapidement si les licences ont été entièrement affectées aux utilisateurs et s’il existe des erreurs que vous devez examiner. Les informations suivantes sont disponibles :

   - Licences de services actuellement affectées au groupe. Sélectionnez une entrée pour afficher les services spécifiques qui ont été activés et pour apporter des modifications.

   - Mises à jour de l’état des dernières modifications de licences, disponibles si les modifications sont en cours de traitement ou si le traitement a été effectué sur tous les membres utilisateurs.

   - Informations sur les affectations de licences utilisateur en état d’erreur.

   ![erreurs de licence et état des licences](./media/licensing-groups-assign/assignment-errors.png)

1. Pour des informations plus détaillées sur le traitement des licences, consultez **Azure Active Directory** > **Utilisateurs et groupes** > *nom du groupe* > **Journaux d’audit**. Vérifiez les activités suivantes :

   - Activité : `Start applying group based license to users`. Elle est consignée lorsque le système sélectionne la modification d’affectation de licences sur le groupe et commence à l’appliquer à tous les membres de l’utilisateur. Elle contient des informations sur la modification qui a été apportée.

   - Activité : `Finish applying group based license to users`. Elle est consignée lorsque le système termine le traitement de tous les utilisateurs du groupe. Elle contient un résumé du nombre d’utilisateurs qui ont été traités avec succès et du nombre d’utilisateurs auxquels les licences du groupe n’ont pas pu être affectées.

   [Lisez cette section](licensing-group-advanced.md#use-audit-logs-to-monitor-group-based-licensing-activity) pour en savoir plus sur l’utilisation des journaux d’audit pour analyser les modifications apportées par une gestion des licences basée sur un groupe.

## <a name="step-3-check-for-license-problems-and-resolve-them"></a>Étape 3 : Rechercher des problèmes de licences et les résoudre

1. Accédez à **Azure Active Directory** > **Groupes** et recherchez les groupes auquel les licences ont été affectées.
1. Dans la page du groupe, sélectionnez **Licences**. La notification en haut de la page indique que des licences n’ont pas pu être affectées à 10 utilisateurs. Ouvrez cette notification pour accéder à une liste de tous les utilisateurs associés à une erreur de licence pour ce groupe.
1. La colonne **Échec d’affectations** indique que les deux licences de produit n’ont pas pu être affectées aux utilisateurs. La colonne **Raison principale de l’échec** contient la cause de l’échec. Dans ce cas, il s’agit de **Plans de service en conflit**.

   ![licences qui n’ont pas pu être affectées](./media/licensing-groups-assign/failed-assignments.png)

1. Sélectionnez un utilisateur pour ouvrir la page **Licences** de l’utilisateur. Cette page montre toutes les licences actuellement affectées à l’utilisateur. Dans cet exemple, l’utilisateur a hérité de la licence Office 365 Enterprise E1 du groupe **Kiosk users**. Cette licence est en conflit avec la licence E3 que le système a tenté d’appliquer à partir du groupe **HR Department**. Par conséquent, aucune des licences de ce groupe n’a été assignée à l’utilisateur.

   ![Voir tous les conflits de licence pour un utilisateur](./media/licensing-groups-assign/user-licence-conflicting-service-plans.png)

1. Pour résoudre ce conflit, nous supprimons l’utilisateur du groupe **Kiosk users**. Lorsqu’Azure AD a traité la modification, les licences **HR Department** sont affectées correctement.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’ensemble de fonctionnalités pour l’affectation des licences avec des groupes, consultez les articles suivants :

- [What is group-based licensing in Azure Active Directory? (Présentation des licences basées sur le groupe dans Azure Active Directory)](../fundamentals/active-directory-licensing-whatis-azure-portal.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)
- [Identification et résolution des problèmes de licence pour un groupe dans Azure Active Directory](licensing-groups-resolve-problems.md)
- [Migration des utilisateurs individuels sous licence vers les licences basées sur les groupes dans Azure Active Directory](licensing-groups-migrate-users.md)
- [Guide pratique pour migrer des utilisateurs entre des licences de produit à l’aide de la gestion de licences basée sur des groupes dans Azure Active Directory](licensing-groups-change-licenses.md)
- [Azure Active Directory group-based licensing additional scenarios (Autres scénarios de licence basée sur le groupe Azure Active Directory)](licensing-group-advanced.md)
- [Exemples PowerShell pour les licences basées sur les groupes dans Azure Active Directory](licensing-ps-examples.md)