---
title: 'Azure AD Connect Sync : extensions d’annuaire | Microsoft Docs'
description: Cette rubrique décrit la fonctionnalité d’extensions d’annuaire dans Azure AD Connect.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: 995ee876-4415-4bb0-a258-cca3cbb02193
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 11/12/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 74bc659c11c4f43ab3cf85cdc53f704cd07a1cde
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96172365"
---
# <a name="azure-ad-connect-sync-directory-extensions"></a>Azure AD Connect Sync : extensions d’annuaire
Vous pouvez utiliser les extensions d’annuaire pour étendre le schéma dans Azure Active Directory (Azure AD) avec vos propres attributs à partir d’un annuaire Active Directory local. Cette fonctionnalité vous permet de générer des applications métiers en consommant les attributs que vous continuez à gérer en local. Ces attributs peuvent être utilisés via des [extensions](/graph/extensibility-overview
). Vous pouvez voir les attributs disponibles à l’aide de [Microsoft Graph Explorer](https://developer.microsoft.com/graph/graph-explorer). Vous pouvez également utiliser cette fonctionnalité pour créer des groupes dynamiques dans Azure AD.

Actuellement, aucune charge de travail Microsoft 365 n’utilise ces attributs.

## <a name="customize-which-attributes-to-synchronize-with-azure-ad"></a>Personnaliser les attributs à synchroniser avec Azure AD

Vous pouvez configurer les attributs supplémentaires à synchroniser dans le chemin d’accès des paramètres personnalisés dans l’Assistant d’installation.

> [!NOTE]
> Dans les versions d’Azure AD Connect antérieures à 1.2.65.0, la zone de recherche pour les **Attributs disponibles** respecte la casse.

![Assistant d’extension de schéma](./media/how-to-connect-sync-feature-directory-extensions/extension2.png)  

L’installation affiche les attributs suivants, qui sont des candidats valides :

* Types d’utilisateur et d’objet de groupe
* Attributs à valeur unique : chaîne, booléen, entier, binaire
* Attributs à valeurs multiples : chaîne, binaire


>[!NOTE]
> Même si Azure AD Connect prend en charge la synchronisation d’attributs Active Directory à valeurs multiples dans Azure AD en tant qu’extensions d’annuaire à valeurs multiples, il n’existe actuellement aucun moyen de récupérer/consommer les données chargées dans les attributs d’extensions d’annuaire à valeurs multiples.

La liste des attributs est lue à partir du cache du schéma qui est créé pendant l’installation d’Azure AD Connect. Si vous avez étendu le schéma Active Directory avec des attributs supplémentaires, vous devez [actualiser le schéma](how-to-connect-installation-wizard.md#refresh-directory-schema) pour que ces nouveaux attributs soient visibles.

Un objet dans Azure AD peut comporter jusqu’à 100 attributs pour extensions d’annuaire. La longueur maximale est de 250 caractères. Si une valeur d’attribut est plus longue, le moteur de synchronisation la tronque.

## <a name="configuration-changes-in-azure-ad-made-by-the-wizard"></a>Changements de configuration apportés par l’Assistant dans Azure AD

Lors de l’installation d’Azure AD Connect, une application dans laquelle ces attributs sont disponibles est enregistrée. Vous pouvez voir cette application dans le portail Azure. Son nom est toujours **Tenant Schema Extension App** (Application d’extension de schéma de locataire).

![Application d’extension de schéma](./media/how-to-connect-sync-feature-directory-extensions/extension3new.png)

Veillez à sélectionner **Toutes les applications** pour voir cette application.

Les attributs ont pour préfixe **extension \_{ApplicationId}\_** . ApplicationId a la même valeur pour tous les attributs de votre locataire Azure AD. Vous aurez besoin de cette valeur pour tous les autres scénarios de cette rubrique.

## <a name="viewing-attributes-using-the-microsoft-graph-api"></a>Affichage des attributs à l’aide de l’API Microsoft Graph

Ces attributs sont désormais disponibles par le biais de l’API Microsoft Graph, en utilisant [Microsoft Graph Explorer](https://developer.microsoft.com/graph/graph-explorer#).

>[!NOTE]
> Dans l’API Microsoft Graph, vous devez demander que les attributs soient retournés. Sélectionnez explicitement les attributs comme suit : `https://graph.microsoft.com/beta/users/abbie.spencer@fabrikamonline.com?$select=extension_9d98ed114c4840d298fad781915f27e4_employeeID,extension_9d98ed114c4840d298fad781915f27e4_division`.
>
> Pour plus d’informations, consultez l’article [Microsoft Graph : Utilisation de paramètres de requête](/graph/query-parameters#select-parameter).

>[!NOTE]
> La synchronisation des valeurs d’attribut d’AADConnect vers des attributs d’extension qui ne sont pas créés par AADConnect n’est pas prise en charge. Cela peut entraîner des problèmes de performances et des résultats inattendus. Seuls les attributs d’extension créés comme indiqué ci-dessus sont pris en charge pour la synchronisation.

## <a name="use-the-attributes-in-dynamic-groups"></a>Utiliser les attributs dans des groupes dynamiques

L’un des scénarios les plus utiles consiste à utiliser ces attributs dans la sécurité dynamique ou dans des groupes Microsoft 365.

1. Créez un groupe dans Azure AD. Donnez-lui un bon nom et vérifiez que le **Type d’appartenance** est **Utilisateur dynamique**.

   ![Capture d’écran avec un nouveau groupe](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup1.png)

2. Sélectionnez l’option permettant d’**Ajouter une requête dynamique**. Si vous examinez les propriétés, vous ne voyez pas ces attributs étendus. Vous devez d’abord les ajouter. Cliquez sur **Obtenir des propriétés d’extension personnalisée**, entrez l’ID d’application, puis cliquez sur **Actualiser les propriétés**.

   ![Capture d’écran où des extensions d’annuaire ont été ajoutées](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup2.png) 

3. Ouvrez la liste déroulante des propriétés et notez que les attributs que vous avez ajoutés sont désormais visibles.

   ![Capture d’écran avec de nouveaux attributs affichés dans l’interface utilisateur](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup3.png)

   Complétez l’expression pour qu’elle réponde à vos besoins. Dans notre exemple, la règle est définie sur **(user.extension_9d98ed114c4840d298fad781915f27e4_division -eq "Ventes et marketing")** .

4. Une fois le groupe créé, laissez un peu de temps à Azure AD pour remplir les membres, puis passez-les en revue.

   ![Capture d’écran avec les membres du groupe dynamique](./media/how-to-connect-sync-feature-directory-extensions/dynamicgroup4.png)  

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur la configuration de la [synchronisation Azure AD Connect](how-to-connect-sync-whatis.md) .

En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).
