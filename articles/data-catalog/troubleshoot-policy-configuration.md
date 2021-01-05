---
title: Guide pratique pour dépanner Azure Data Catalog
description: Cet article décrit les considérations courantes concernant la résolution des problèmes pour les ressources Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: troubleshooting
ms.date: 08/01/2019
ms.openlocfilehash: 84bd14f8ae18527b4f6e9d8509a12555baec8771
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68879557"
---
# <a name="troubleshooting-azure-data-catalog"></a>Résolution des problèmes liés à Azure Data Catalog

Cet article décrit les considérations courantes concernant la résolution des problèmes pour les ressources Azure Data Catalog. 

## <a name="functionality-limitations"></a>Limitations des fonctionnalités

Quand vous utilisez Azure Data Catalog, les fonctionnalités suivantes sont limitées :

- Les comptes avec le type **Rôle Invité** ne sont pas pris en charge. Vous ne pouvez pas ajouter de comptes Invité en tant qu’utilisateurs d’Azure Data Catalog, et les utilisateurs invités ne peuvent pas utiliser le portail sur [https://www.azuredatacatalog.com](https://www.azuredatacatalog.com).

- La création de ressources Azure Data Catalog à l’aide de modèles Azure Resource Manager ou de commandes Azure PowerShell n’est pas prise en charge.

- La ressource Azure Data Catalog ne peut pas être déplacée entre locataires Azure.

## <a name="azure-active-directory-policy-configuration"></a>Configuration de la stratégie Azure Active Directory

Il arrive que vous puissiez vous connecter au portail d’Azure Data Catalog, mais que lorsque vous tentez de vous connecter à l’outil de référencement de la source de données, un message d’erreur s’affiche et vous empêche de vous connecter. Cette erreur peut se produire lorsque vous êtes sur le réseau d'entreprise ou lorsque vous vous connectez depuis l'extérieur de celui-ci.

L’outil de référencement utilise *l’authentification par formulaire* pour valider les connexions des utilisateurs avec Azure Active Directory. Pour une connexion réussie, un administrateur Azure Active Directory doit activer l’authentification par formulaire dans la *stratégie d’authentification globale*.

La stratégie d’authentification globale vous permet d’activer séparément des méthodes d’authentification pour les connexions intranet et extranet, comme illustré dans l’image suivante. Des erreurs de connexion peuvent survenir si l'authentification par formulaire n'est pas activée pour le réseau à partir duquel vous vous connectez.

 ![Stratégie d’authentification globale Azure Active Directory](./media/troubleshoot-policy-configuration/global-auth-policy.png)

Pour plus d’informations, consultez [Configuration des stratégies d’authentification](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn486781(v=ws.11)).

## <a name="next-steps"></a>Étapes suivantes

* [Créer un catalogue Azure Data Catalog](data-catalog-get-started.md)
