---
title: Utiliser Purview Studio
description: Cet article conceptuel explique comment utiliser Azure Purview Studio.
author: nayenama
ms.author: nayenama
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/12/2020
ms.openlocfilehash: d8e6c4b2addf9745b2ddabe8f6fdad9d82dce59f
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97503948"
---
# <a name="use-purview-studio"></a>Utiliser Purview Studio

Cet article donne une vue d’ensemble des principales fonctionnalités d’Azure Purview.

## <a name="prerequisites"></a>Prérequis

* Un compte Purview actif est déjà créé sur le Portail Azure. L’utilisateur dispose des autorisations nécessaires pour accéder à Purview Studio.

## <a name="launch-purview-account"></a>Lancement du compte Purview

* Pour lancer votre compte Purview, accédez à Comptes Purview sur le Portail Azure, sélectionnez le compte souhaité et lancez-le.

   :::image type="content" source="./media/use-purview-studio/launch-from-portal.png" alt-text="Capture d’écran de la sélection permettant de lancer le catalogue du compte Azure Purview.":::

* Pour lancer un compte Purview, vous pouvez également accéder à `https://web.purview.azure.com`, puis sélectionner **Azure Active Directory** et le nom du compte.

## <a name="home-page"></a>page d'accueil

**Accueil** est la page de démarrage du client Azure Purview.

 :::image type="content" source="./media/use-purview-studio/purview-homepage.png" alt-text="Capture d’écran de la page d’accueil":::

Dans la liste suivante figurent les principales fonctionnalités de la **page d’accueil**. Chacun des numéros de la liste correspond à un nombre encadré dans la capture d’écran précédente.

1. Nom convivial du catalogue. Vous pouvez définir le nom du catalogue dans **Centre de gestion** > * *Informations sur le compte*.

2. L’analytique du catalogue indique les volumes suivants :
    - Utilisateurs, groupes et applications
    - Sources de données
    - Éléments multimédias
    - Termes de glossaire

3. La zone de recherche permet de rechercher des ressources de données dans le catalogue de données.

4. Les boutons Accès rapide donnent accès aux fonctions fréquemment utilisées de l’application. Les boutons présentés dépendent du rôle affecté à votre compte d’utilisateur.

    - Pour *Organisateur de données* : **Centre des connaissances**, **Parcourir les ressources**, **Gérer le glossaire** et **Afficher des insights**.
    - Pour *Lecteur de données* : **Centre des connaissances**, **Parcourir les ressources**, **Afficher le glossaire** et **Afficher des insights**.
    - Pour *Administrateur de sources de données* + *Organisateur de données*, les boutons proposés sont **Centre des connaissances**, **Inscrire des sources de données**, **Parcourir les ressources** et **Gérer le glossaire**.
    - Pour *Administrateur de sources de données* + *Lecteur de données*, les boutons proposés sont **Centre des connaissances**, **Inscrire des sources de données**, **Parcourir les ressources** et **Afficher le glossaire**.

5. La barre de navigation de gauche aide à localiser les pages principales de l’application. Les boutons présentés dépendent du rôle affecté à votre compte d’utilisateur.

    - Pour *Organisateur de données* : **Accueil**, **Glossaire**, **Insights** et **Centre de gestion**.
    - Pour *Lecteur de données* : **Accueil**, **Glossaire**, **Insights** et **Centre de gestion**.
    - Pour *Administrateur de sources de données* + *Organisateur/lecteur de données*, les boutons sont **Accueil**, **Sources**, **Glossaire**, **Insights** et **Centre de gestion**.
  
6. L’onglet **Récemment consultés** donne la liste des ressources de données récemment consultées. Pour plus d’informations sur l’accès aux ressources, consultez [Rechercher dans le catalogue de données](how-to-search-catalog.md) et [Parcourir par type de ressource](how-to-browse-catalog.md#browse-experience).  L’onglet **Mes éléments** correspond à la liste des ressources de données détenues par l’utilisateur connecté.
7. **Liens utiles** contient des liens vers l’état de la région, la documentation, les tarifs, la vue d’ensemble et l’état Purview.
8. La barre de navigation supérieure contient des informations sur les sections Notes de publication/mises à jour, Modifier le compte Purview, Notifications, Aide et Commentaires.

## <a name="knowledge-center"></a>Centre des connaissances

Dans le Centre des connaissances se trouvent toutes les vidéos et tous les tutoriels relatifs à Purview.

## <a name="guided-tours"></a>Visites guidées

Chacune des expériences utilisateur d’Azure Purview Studio comporte une visite guidée qui donne une vue d’ensemble de la page. Pour commencer la visite guidée, sélectionnez **Aide** dans la barre supérieure, puis sélectionnez **Visites guidées**.

:::image type="content" source="./media/use-purview-studio/guided-tour.png" alt-text="Capture d’écran de la visite guidée":::

> [!Important]
   > Le rôle Administrateur de sources de données n’a pas accès à Purview Studio.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Ajout d’un principal de sécurité](tutorial-scan-data.md)