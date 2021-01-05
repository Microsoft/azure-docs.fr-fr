---
title: 'Démarrage rapide : Créer un catalogue de données Azure'
description: Ce démarrage rapide explique comment créer un catalogue Azure Data Catalog à l’aide du portail Azure.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: quickstart
ms.date: 05/26/2020
ms.openlocfilehash: a7d575b92bff69c13ded70187005ce1db018ea4c
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92152000"
---
# <a name="quickstart-create-an-azure-data-catalog"></a>Démarrage rapide : Créer un catalogue de données Azure

Azure Data Catalog est un service cloud entièrement géré qui sert de système d’inscription et de découverte des ressources de données d’entreprise. Pour une présentation détaillée, consultez l’article [Qu’est-ce qu’Azure Data Catalog ?](overview.md).

Ce guide de démarrage rapide vous guide dans la création d’un catalogue Azure Data Catalog.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

> [!Note]
> En raison des exigences de sécurité d’Azure, Azure Data Catalog a appliqué le protocole TLS (Transport Layer Security) 1.2. Les protocoles TLS 1.0 et TLS 1.1 ont été désactivés. Vous pouvez rencontrer des erreurs lors de l’exécution de l’outil d’inscription si votre ordinateur n’est pas mis à jour pour le protocole TLS 1.2. Consultez [Activer le protocole TLS (1.2)](/mem/configmgr/core/plan-design/security/enable-tls-1-2) pour mettre à jour votre ordinateur.

Pour commencer, vous avez besoin des éléments suivants :

* Un abonnement [Microsoft Azure](https://azure.microsoft.com/).
* Vous devez avoir votre propre [locataire Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

Pour configurer Data Catalog, vous devez être le propriétaire ou le copropriétaire d’un abonnement Azure.

## <a name="create-a-data-catalog"></a>Créer un catalogue de données

Vous ne pouvez approvisionner qu’un seul catalogue de données par organisation (domaine Azure Active Directory). Par conséquent, si le propriétaire ou copropriétaire d’un abonnement Azure qui fait partie de ce domaine Azure Active Directory a déjà créé un catalogue, vous ne pourrez pas en créer un autre, même si vous disposez de plusieurs abonnements Azure. Pour tester si un catalogue de données a été créé par un utilisateur dans votre domaine Azure Active Directory, accédez à la [page d’accueil Azure Data Catalog](http://azuredatacatalog.com) et vérifiez si le catalogue y figure. Ignorez la procédure suivante et passez à la section suivante si un catalogue a déjà été créé pour vous.

1. Accédez au [portail Azure](https://portal.azure.com) > **Créer une ressource**, puis sélectionnez **Data Catalog**.

    ![Bouton Créer un catalogue Azure Data Catalog](media/data-catalog-get-started/data-catalog-create.png)

2. Spécifiez un **nom** pour le catalogue de données, l’**abonnement** à utiliser, **l’emplacement** du catalogue, ainsi que le **niveau tarifaire**. Sélectionnez ensuite **Créer**.

3. Accédez à la [page d’accueil Azure Data Catalog](http://azuredatacatalog.com) , puis cliquez sur **Publier des données**.

   ![Azure Data Catalog--bouton Publier des données](media/data-catalog-get-started/data-catalog-publish-data.png)

   Vous pouvez également accéder à la page d’accueil de Data Catalog à partir de la [page du service Data Catalog](https://azure.microsoft.com/services/data-catalog), en sélectionnant **Prise en main**.

   ![Azure Data Catalog--page d’accueil marketing](media/data-catalog-get-started/data-catalog-marketing-landing-page.png)

4. Accédez à la page **Paramètres**.

    ![Azure Data Catalog--approvisionner data catalog](media/data-catalog-get-started/data-catalog-create-azure-data-catalog.png)

5. Développez **Prix appliqués**, puis vérifiez votre **édition** d’Azure Data Catalog (gratuite ou standard).

    ![Azure Data Catalog--Sélection de l’édition](media/data-catalog-get-started/data-catalog-create-catalog-select-edition.png)

6. Si vous choisissez l’édition *Standard* comme niveau tarifaire, vous pouvez développer **Groupes de sécurité** afin d’autoriser les groupes de sécurité Active Directory à accéder à Data Catalog et de permettre l’ajustement automatique de la facturation.

    ![Groupes de sécurité Azure Data Catalog](media/data-catalog-get-started/data-catalog-standard-security-groups.png)

7. Développez **Utilisateurs du catalogue** et cliquez sur **Ajouter** pour ajouter des utilisateurs au catalogue de données. Vous êtes automatiquement ajouté à ce groupe.

    ![Azure Data Catalog--Utilisateurs](media/data-catalog-get-started/data-catalog-add-catalog-user.png)

8. Si vous choisissez l’édition *Standard* comme niveau tarifaire, vous pouvez développer **Administrateurs de glossaire** et cliquer sur **Ajouter** pour ajouter des administrateurs de glossaire. Vous êtes automatiquement ajouté à ce groupe.

    ![Administrateurs de glossaire Azure Data Catalog](media/data-catalog-get-started/data-catalog-standard-glossary-admin.png)

9. Développez **Administrateurs du catalogue** et cliquez sur **Ajouter** pour ajouter des administrateurs supplémentaires au catalogue de données. Vous êtes automatiquement ajouté à ce groupe.

    ![Azure Data Catalog--Administrateurs](media/data-catalog-get-started/data-catalog-add-catalog-admins.png)

10. Développez **Titre du portail** et ajoutez le texte qui doit être affiché dans le titre du portail.

    ![Azure Data Catalog - Titre du portail](media/data-catalog-get-started/data-catalog-portal-title.png)

11. Lorsque vous en avez terminé avec la page **Paramètres**, accédez à la page **Publier**.

    ![Azure Data Catalog--Créé](media/data-catalog-get-started/data-catalog-created.png)

## <a name="find-a-data-catalog-in-the-azure-portal"></a>Rechercher un catalogue de données dans le portail Azure

1. Dans un onglet distinct du navigateur web ou dans une fenêtre de navigateur web distincte, accédez au [portail Azure](https://portal.azure.com) et connectez-vous en utilisant le compte que vous avez utilisé pour créer le catalogue de données à l’étape précédente.

2. Sélectionnez **Tous les services**, puis cliquez sur **Data Catalog**.

    ![Azure Data Catalog--parcourir Azure](media/data-catalog-get-started/data-catalog-browse-azure-portal.png)

    Le catalogue de données que vous avez créé s’affiche.

    ![Azure Data Catalog--afficher le catalogue dans une liste](media/data-catalog-get-started/data-catalog-azure-portal-show-catalog.png)

3. Cliquez sur le catalogue que vous avez créé. Le panneau **Catalogue de données** s’affiche dans le portail.

   ![Azure Data Catalog--panneau dans le portail](media/data-catalog-get-started/data-catalog-blade-azure-portal.png)

4. Vous pouvez afficher les propriétés du catalogue de données et les mettre à jour. Par exemple, cliquez sur **Niveau tarifaire** et modifiez l’édition.

    ![Azure Data Catalog--niveau tarifaire](media/data-catalog-get-started/data-catalog-change-pricing-tier.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez vu comment créer un catalogue Azure Data Catalog pour votre organisation. Vous pouvez désormais inscrire des sources de données dans votre catalogue de données.

> [!div class="nextstepaction"]
> [Inscrire des sources de données dans Azure Data Catalog](data-catalog-how-to-register.md)