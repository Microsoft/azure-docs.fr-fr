---
title: Guide pratique pour sécuriser l’accès à Azure Data Catalog
description: Cet article explique comment sécuriser un catalogue de données et ses ressources de données dans Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: 6a429c09b6f8082c95e29bcea62d27ec4fb46fd3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017302"
---
# <a name="how-to-secure-access-to-data-catalog-and-data-assets"></a>Guide pratique pour sécuriser l’accès à un catalogue de données et des ressources de données

> [!IMPORTANT]
> Cette fonctionnalité est disponible uniquement dans l’édition Standard d’Azure Data Catalog.

Azure Data Catalog vous permet de spécifier qui peut accéder au catalogue de données et quelles opérations (inscrire, annoter, prendre possession) ils peuvent effectuer sur les métadonnées dans le catalogue. 

## <a name="catalog-users-and-permissions"></a>Utilisateurs du catalogue et autorisations

Pour accorder à un utilisateur ou à un groupe l’accès à un catalogue de données et définir des autorisations

1. Dans la [page d’accueil de votre catalogue de données](https://www.azuredatacatalog.com), cliquez sur **Paramètres** dans la barre d’outils.

   ![Bouton Paramètres de la page d’accueil Azure Data Catalog](media/data-catalog-how-to-secure-catalog/data-catalog-settings.png)

2. Dans la page Paramètres, développez la section **Utilisateurs du catalogue**.

   ![Bouton Ajouter des utilisateurs d’Azure Data Catalog](media/data-catalog-how-to-secure-catalog/data-catalog-add-button.png)

3. Cliquez sur **Ajouter**.

4. Entrez le **nom d’utilisateur** qualifié complet ou le nom du **groupe de sécurité** dans l’annuaire Azure Active Directory (AAD) associé au catalogue. Utilisez une virgule (« , ») comme séparateur si vous ajoutez plusieurs utilisateurs ou groupes.

   ![Utilisateurs d’Azure Data Catalog - utilisateurs ou groupes](media/data-catalog-how-to-secure-catalog/data-catalog-users-groups.png)

5. Appuyez sur **Entrée** ou **Tab** en dehors de la zone de texte. 

6. Vérifiez que toutes les autorisations (**Annoter**, **Inscrire**, et **Prendre possession**) sont affectées à ces utilisateurs ou ces groupes par défaut. Autrement dit, l’utilisateur ou le groupe peut [inscrire des ressources de données]( data-catalog-how-to-register.md), [annoter des ressources de données]( data-catalog-how-to-annotate.md) et [prendre possession des ressources de données]( data-catalog-how-to-manage.md). 

   ![Utilisateurs d’Azure Data Catalog - autorisations par défaut](media/data-catalog-how-to-secure-catalog/data-catalog-default-permissions.png)

7. Pour accorder à un utilisateur ou à un groupe uniquement l’accès en lecture au catalogue, désactivez l’option **annoter** pour cet utilisateur ou groupe. Dans ce cas, l’utilisateur ou le groupe ne peut pas annoter de ressources de données dans le catalogue, mais il peut les visualiser. 

8. Pour interdire à un utilisateur ou à un groupe d’inscrire des ressources de données, désactivez l’option **inscrire** pour cet utilisateur ou groupe.

9. Pour interdire à un utilisateur de prendre possession d’une ressource de données, désactivez l’option **prendre possession** pour cet utilisateur ou groupe. 

10. Pour supprimer un utilisateur/groupe des utilisateurs du catalogue, cliquez sur **x** pour l’utilisateur/groupe en bas de la liste. 

   ![Utilisateurs du catalogue Azure Data Catalog - icône X de suppression d’utilisateur](media/data-catalog-how-to-secure-catalog/data-catalog-delete-user.png)

   > [!IMPORTANT]
   > Nous vous recommandons d’ajouter des groupes de sécurité aux utilisateurs du catalogue plutôt que d’ajouter directement des utilisateurs et d’affecter des autorisations. Ensuite, ajoutez des utilisateurs aux groupes de sécurité qui correspondent à leurs rôles et à l’accès au catalogue dont ils ont besoin.

## <a name="special-considerations"></a>Considérations spéciales

- Les autorisations affectées aux groupes de sécurité sont additives. Imaginez par exemple qu’un utilisateur appartient à deux groupes. L’un des groupes a des autorisations d’annotation, tandis que l’autre n’en a pas. Dans ce cas de figure, l’utilisateur a des autorisations d’annotation. 
- Les autorisations affectées explicitement à un utilisateur remplacent celles affectées aux groupes auxquels il appartient. Reprenons l’exemple précédent. Imaginez que vous avez explicitement ajouté l’utilisateur aux utilisateurs du catalogue et que vous n’affectez pas d’autorisation d’annotation. L’utilisateur ne pourra pas annoter les ressources de données même s’il est membre d’un groupe qui a des autorisations d’annotation.

## <a name="next-steps"></a>Étapes suivantes

- [Prise en main d’Azure Data Catalog](data-catalog-get-started.md)
