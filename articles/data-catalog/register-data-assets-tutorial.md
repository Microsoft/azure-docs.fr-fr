---
title: 'Tutoriel : Enregistrer des éléments de données dans Azure Data Catalog'
description: Ce tutoriel explique comment inscrire des ressources de données dans votre catalogue Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: tutorial
ms.date: 08/01/2019
ms.openlocfilehash: 66bacdf27b2ef9cf624b645cb6e4c123d9c3fa2a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88135876"
---
# <a name="tutorial-register-data-assets-in-azure-data-catalog"></a>Tutoriel : Enregistrer des éléments de données dans Azure Data Catalog

Dans ce tutoriel, vous utilisez l’outil de référencement pour inscrire des ressources de données de l’exemple de base de données avec le catalogue. L’inscription est le processus par lequel les métadonnées structurelles clés (telles que les noms, les types et les emplacements) sont extraites de la source de données et des ressources qu’elle contient, puis copiées dans le catalogue. La source de données et les ressources de données restent au même endroit, mais les métadonnées sont utilisées par le catalogue pour faciliter leur découverte et leur compréhension.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Inscrire des ressources de données 
> * Rechercher des ressources de données
> * Annoter les ressources de données
> * Se connecter aux ressources de données
> * Gérer les ressources de données
> * Supprimer des ressources de données

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous devez suivre le guide de [démarrage rapide](register-data-assets-tutorial.md).

* Un abonnement [Microsoft Azure](https://azure.microsoft.com/).
* Vous devez avoir votre propre [locataire Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

Pour configurer Data Catalog, vous devez être le propriétaire ou le copropriétaire d’un abonnement Azure.

## <a name="register-data-assets"></a>Inscrire des ressources de données

### <a name="register-a-data-source"></a>Référencer une source de données

Vous inscrivez des ressources de données (tables) d’un [exemple de base de données](../azure-sql/database/single-database-create-quickstart.md) pour Azure SQL Database. Vous pouvez cependant utiliser n’importe quelle source de données prise en charge si vous préférez travailler avec des données plus familières et mieux adaptées à votre rôle. Pour obtenir la liste des sources de données prises en charge, consultez l’article [Sources de données prises en charge](data-catalog-dsr.md).

Le nom de la base de données que nous utilisons dans ce tutoriel est *RLSTest*.

Vous pouvez désormais inscrire des ressources de données de l’exemple de base de données à l’aide d’Azure Data Catalog.

1. Accédez à la [page d’accueil Azure Data Catalog](http://azuredatacatalog.com), puis sélectionnez **Publier des données**.

   ![Azure Data Catalog--bouton Publier des données](media/register-data-assets-tutorial/data-catalog-publish-data.png)

2. Sélectionnez **Lancer l’application** pour télécharger, installer et exécuter l’outil de référencement sur votre ordinateur.

   ![Azure Data Catalog--bouton de lancement](media/register-data-assets-tutorial/data-catalog-launch-application.png)

3. Dans la page **Bienvenue**, sélectionnez **Se connecter**, puis entrez vos informations d’identification.

    ![Azure Data Catalog--page d’accueil](media/register-data-assets-tutorial/data-catalog-welcome-dialog.png)

4. Dans la page **Microsoft Azure Data Catalog**, sélectionnez **SQL Server**, puis **Suivant**.

    ![Azure Data Catalog--sources de données](media/register-data-assets-tutorial/data-catalog-data-sources.png)

5. Entrez les propriétés de connexion SQL Server pour votre exemple de base de données dans Azure SQL Database, puis sélectionnez **SE CONNECTER**.

   ![Azure Data Catalog--paramètres de connexion à SQL Server](media/register-data-assets-tutorial/data-catalog-sql-server-connection.png)

6. Enregistrez les métadonnées de votre ressource de données. Dans cet exemple, vous inscrivez les objets **Product** à partir de l’exemple d’espace de noms :

   1. Dans l’arborescence **Hiérarchie du serveur**, développez votre exemple de base de données, puis sélectionnez **SalesLT**.

   2. Appuyez sur la touche CTRL, puis sélectionnez **Product**, **ProductCategory**, **ProductDescription** et **ProductModel**.

   3. Sélectionnez la **flèche de déplacement de la sélection** ( **>** ). Cette action déplace tous les objets sélectionnés dans la liste **Objets à inscrire** .

      ![Didacticiel Azure Data Catalog--parcourir et sélectionner des objets](media/register-data-assets-tutorial/data-catalog-server-hierarchy.png)

   4. Sélectionnez **Include a Preview (Inclure un aperçu)** pour inclure un aperçu instantané des données. L’instantané inclut jusqu’à 20 enregistrements de chaque table et est copié dans le catalogue.

   5. Sélectionnez **Inclure le profil de données** pour inclure un instantané des statistiques des objets du profil de données (par exemple, les valeurs minimale, maximale et moyenne d’une colonne ou le nombre de lignes).

   6. Dans le champ **Ajouter des étiquettes**, entrez **sales, product, azure sql**. Cette action ajoute des étiquettes de recherche à ces ressources de données. Les étiquettes sont un excellent moyen d’aider les utilisateurs à trouver une source de données inscrite.

   7. Spécifiez le nom d’un **expert** pour ces données (facultatif).

      ![Didacticiel Azure Data Catalog--objets à inscrire](media/register-data-assets-tutorial/data-catalog-objects-register.png)

   8. Sélectionnez **S’INSCRIRE**. Azure Data Catalog enregistre les objets que vous avez sélectionnés. Dans cet exercice, les objets sélectionnés dans votre exemple de base de données sont inscrits. L’outil de référencement extrait les métadonnées de la ressource de données et les copie dans le service Azure Data Catalog. Les données restent où elles se trouvent actuellement. Les données restent sous le contrôle des administrateurs et des stratégies du système d’origine.

      ![Azure Data Catalog--objets inscrits](media/register-data-assets-tutorial/data-catalog-registered-objects.png)

    9. Pour voir les objets sources de données que vous avez inscrits, sélectionnez **Afficher le portail**. Dans le portail Azure Data Catalog, vérifiez que les quatre tables et la base de données sont affichées en mode Grille (la barre de recherche doit être vide).

       ![Objets dans le portail Azure Data Catalog](media/register-data-assets-tutorial/data-catalog-view-portal.png)

Dans cet exercice, vous avez inscrit des objets de l’exemple de base de données pour Azure SQL Database afin que les utilisateurs de votre organisation puissent les découvrir facilement.

Dans l’exercice suivant, vous apprenez à découvrir les ressources de données inscrites.

## <a name="discover-data-assets"></a>Découvrir les ressources de données

Dans Azure Data Catalog, la découverte utilise deux mécanismes principaux : la recherche et le filtrage.

La recherche est conçue pour être intuitive et puissante. Par défaut, les termes de recherche sont comparés à toutes les propriétés du catalogue, notamment aux annotations fournies par l’utilisateur.

Le filtrage est conçu pour compléter la recherche. Vous pouvez sélectionner des caractéristiques spécifiques telles que les experts, le type de source de données, le type d’objet et les balises, pour afficher les ressources de données correspondantes et pour limiter les résultats de la recherche aux ressources correspondantes.

En combinant la recherche et le filtrage, vous pouvez parcourir rapidement les sources de données qui sont inscrites auprès d’Azure Data Catalog.

Dans cet exercice, vous utilisez le portail Azure Data Catalog pour découvrir les ressources de données que vous avez enregistrées dans l’exercice précédent. Pour plus d’informations sur la syntaxe de recherche, consultez l’article [Data Catalog Search syntax reference (Informations de référence sur la syntaxe de recherche dans Data Catalog)](/rest/api/datacatalog/#search-syntax-reference) .

Voici quelques exemples de découverte des ressources de données dans le catalogue.  

### <a name="discover-data-assets-with-basic-search"></a>Découvrir les ressources de données à l’aide de la fonction de recherche de base

La recherche de base vous permet d’effectuer des recherches dans le catalogue en utilisant un ou plusieurs termes de recherche. Les résultats sont toutes les ressources correspondant à une propriété quelconque avec un ou plusieurs termes spécifiés.

1. Sélectionnez **Accueil** dans le portail Azure Data Catalog. Si vous avez fermé le navigateur web, accédez à la [page d’accueil Azure Data Catalog](https://www.azuredatacatalog.com).

2. Dans la zone de recherche, entrez `product` et appuyez sur **ENTRÉE**.

   ![Azure Data Catalog--recherche de texte de base](media/register-data-assets-tutorial/data-catalog-basic-text-search.png)

3. Vérifiez que les quatre tables et la base de données s’affichent dans les résultats. Vous pouvez basculer entre le **mode Grille** et le **mode Liste** en sélectionnant les boutons de la barre d’outils, comme l’illustre l’image suivante. Le mot clé de recherche est mis en surbrillance dans les résultats de recherche, car l’option **Mettre en surbrillance** est **ACTIVÉE**. Vous pouvez également spécifier le nombre de **résultats par page** dans les résultats de recherche.

   ![Azure Data Catalog--résultats de recherche de texte de base](media/register-data-assets-tutorial/data-catalog-basic-text-search-results.png)

   Le panneau **Recherches** et le panneau **Propriétés** s’affichent respectivement à gauche et à droite. Le panneau **Recherches** vous permet de modifier les critères de recherche et de filtrer les résultats. Le panneau **Propriétés** affiche les propriétés d’un objet sélectionné dans la grille ou liste.

4. Sélectionnez **Product** dans les résultats de la recherche. Sélectionnez les onglets **Aperçu**, **Colonnes**, **Profil de données** et **Documentation**, ou sélectionnez la flèche pour développer le volet inférieur.  

   ![Azure Data Catalog--volet inférieur](media/register-data-assets-tutorial/data-catalog-data-asset-preview.png)

   L’aperçu des données de la table **Product** s’affiche dans l’onglet **Aperçu**.

5. Sélectionnez l’onglet **Colonnes** pour rechercher des informations sur les colonnes (comme le **nom** et le **type de données**) dans la ressource de données.

6. Sélectionnez l’onglet **Profil de données** pour voir le profilage des données (par exemple, le nombre de lignes, la taille des données ou la valeur minimale dans une colonne) dans la ressource de données.

### <a name="discover-data-assets-with-property-scoping"></a>Découvrir les ressources de données à l’aide de la fonction de recherche d’étendue de la propriété

L’étendue de la propriété vous permet de découvrir les ressources de données dans lesquelles le terme recherché correspond à la propriété spécifiée.

1. Désactivez le filtre **Table** sous **Type d’objet** dans **Filtres**.  

2. Dans la zone de recherche, entrez `tags:product` et appuyez sur **ENTRÉE**. Consultez l’article [Data Catalog Search syntax reference (Informations de référence sur la syntaxe de recherche dans Data Catalog)](/rest/api/datacatalog/#search-syntax-reference) pour connaître toutes les propriétés que vous pouvez utiliser pour effectuer des recherches dans le catalogue de données.

3. Vérifiez que les tables et la base de données s’affichent dans les résultats.  

   ![Data Catalog--résultats de recherche d’étendue de la propriété](media/register-data-assets-tutorial/data-catalog-property-scoping-results.png)

### <a name="save-the-search"></a>Enregistrer la recherche

1. Dans le volet **Recherches** de la section **Recherche actuelle**, entrez un nom pour la recherche, puis sélectionnez **Enregistrer**.

   ![Azure Data Catalog--enregistrer la recherche](media/register-data-assets-tutorial/data-catalog-save-search.png)

2. Vérifiez que la recherche enregistrée s’affiche sous **Recherches enregistrées**.

3. Sélectionnez l’une des actions que vous pouvez effectuer sur la recherche enregistrée (**Renommer**, **Supprimer**, **Définir comme recherche par défaut**).

### <a name="grouping-with-parentheses"></a>Parenthèses de regroupement

Les parenthèses de regroupement vous permettent de grouper les parties de la requête à isoler logiquement, notamment avec des opérateurs booléens.

1. Dans la zone de recherche, entrez `name:product AND (tags:product AND objectType:table)` et appuyez sur **ENTRÉE**.

2. Vérifiez que seule la table **Product** s’affiche dans les résultats de la recherche.

   ![Azure Data Catalog--recherche par regroupement](media/register-data-assets-tutorial/data-catalog-grouping-search.png)

### <a name="comparison-operators"></a>Opérateurs de comparaison

Les opérateurs de comparaison vous permettent d’utiliser des comparaisons autres que l’égalité pour les propriétés comportant des types de données numériques et de date.

1. Dans la zone de recherche, entrez `lastRegisteredTime:>"06/09/2016"`.

2. Désactivez le filtre **Table** sous **Type d’objet**.

3. Appuyez sur **Entrée**.

4. Vérifiez que les tables **Product**, **ProductCategory** et **ProductDescription** ainsi que la base de données SQL que vous avez inscrites s’affichent dans les résultats de la recherche.

   ![Azure Data Catalog--résultats de recherche par comparaison](media/register-data-assets-tutorial/data-catalog-comparison-operator-results.png)

Consultez [Guide pratique pour découvrir les ressources de données](data-catalog-how-to-discover.md) pour obtenir des informations détaillées sur la découverte des ressources de données. Pour plus d’informations sur la syntaxe de recherche, consultez [Informations de référence sur la syntaxe de recherche de Data Catalog](/rest/api/datacatalog/#search-syntax-reference).

## <a name="annotate-data-assets"></a>Annoter les ressources de données

Dans cet exercice, vous utilisez le portail Azure Data Catalog pour annoter (ajouter des informations comme des descriptions, des mots clés ou des experts) des ressources de données existantes dans le catalogue. Les annotations complètent les métadonnées structurelles extraites de la source de données pendant l’inscription. Elles facilitent considérablement la découverte et la compréhension des ressources de données.

Dans cet exercice, vous annotez une ressource de données unique (ProductPhoto). Vous ajoutez un nom convivial et une description à la ressource de données ProductPhoto.  

1. Accédez à la [page d’accueil Azure Data Catalog](https://www.azuredatacatalog.com) et recherchez avec `tags:product` les ressources de données que vous avez inscrites.

2. Sélectionnez **ProductModel** dans les résultats de la recherche.  

3. Entrez **Images de produit** comme **Nom convivial** et **Photos de produit pour les documents marketing** comme **Description**.

    ![Azure Data Catalog--description d’une photo de produit](media/register-data-assets-tutorial/data-catalog-productmodel-description.png)

    La **Description** aide les autres utilisateurs à découvrir et à comprendre pourquoi et comment utiliser la ressource de données sélectionnée. Vous pouvez également ajouter des balises supplémentaires et afficher les colonnes. Vous pouvez rechercher et filtrer les ressources de données à l’aide des métadonnées descriptives que vous avez ajoutées au catalogue.

Vous pouvez également effectuer les étapes suivantes de cette page :

* Ajouter des experts pour la ressource de données. Sélectionnez **Ajouter** dans la zone **Experts**.

* Ajouter des balises au niveau du jeu de données. Sélectionnez **Ajouter** dans la zone **Balises**. Une balise peut être une balise utilisateur ou une balise glossaire. L’édition Standard de Data Catalog comprend un glossaire métier qui permet aux administrateurs de catalogue de définir une taxonomie commerciale centrale. Les utilisateurs du catalogue peuvent ensuite annoter les ressources de données avec la terminologie du glossaire. Pour plus d’informations, consultez l’article [Comment configurer le glossaire métier pour un balisage géré](data-catalog-how-to-business-glossary.md)

* Ajouter des balises au niveau des colonnes. Sélectionnez **Ajouter** sous **Balises** pour la colonne que vous souhaitez annoter.

* Ajouter une description au niveau des colonnes. Entrez une **description** pour une colonne. Vous pouvez également afficher les métadonnées de description extraites de la source de données.

* Ajoutez des informations sur le champ **Demander l’accès** pour expliquer aux utilisateurs comment demander un accès à la ressource de données.
  
* Sélectionnez l’onglet **Documentation** et fournissez une documentation pour la ressource de données. La documentation Azure Data Catalog vous permet d’utiliser votre catalogue de données comme référentiel de contenu pour créer une narration complète de vos ressources de données.
  
Vous pouvez également ajouter une annotation à plusieurs ressources de données. Par exemple, vous pouvez sélectionner toutes les ressources de données que vous avez inscrites et leur spécifier un expert.

![Azure Data Catalog--annoter plusieurs ressources de données](media/register-data-assets-tutorial/data-catalog-multi-select-annotate.png)

Azure Data Catalog prend en charge une approche basée sur le crowdsourcing pour les annotations. Tous les utilisateurs de Data Catalog peuvent ajouter des balises (utilisateur ou de glossaire), des descriptions et d’autres métadonnées. Ainsi, les utilisateurs ajoutent une perspective sur une ressource de données et son utilisation, et la partagent avec d’autres utilisateurs.

Consultez l’article [How to annotate data assets (Annotation des ressources de données)](data-catalog-how-to-annotate.md) pour obtenir des informations détaillées sur l’annotation des ressources de données.

## <a name="connect-to-data-assets"></a>Se connecter aux ressources de données

Dans cet exercice, vous ouvrez les ressources de données dans un outil client intégré (Excel) et dans un outil non intégré (SQL Server Management Studio) à l’aide des informations de connexion.

> [!NOTE]
> Il est important de rappeler qu’Azure Data Catalog ne vous donne pas accès à la source de données elle-même : il vous permet simplement de découvrir et de comprendre plus facilement cette source de données. Lorsque vous vous connectez à une source de données, l’application cliente que vous choisissez utilise vos informations d’identification Windows ou vous invite, si nécessaire, à saisir vos informations d’identification. Si l’accès à la source de données ne vous a pas été préalablement accordé, vous devez obtenir cet accès avant de pouvoir vous connecter.

### <a name="connect-to-a-data-asset-from-excel"></a>Se connecter à une ressource de données à partir d’Excel

1. Sélectionnez **Produit** dans les résultats de la recherche. Dans la barre d’outils, sélectionnez **Ouvrir dans**, puis **Excel**.

   ![Azure Data Catalog--se connecter à une ressource de données](media/register-data-assets-tutorial/data-catalog-connect1.png)

2. Sélectionnez **Ouvrir** dans la fenêtre contextuelle de téléchargement. Cette expérience peut varier en fonction du navigateur.

3. Dans la fenêtre **Avis de sécurité Microsoft Excel**, sélectionnez **Activer**.

   ![Azure Data Catalog--fenêtre contextuelle de sécurité Excel](media/register-data-assets-tutorial/data-catalog-excel-security-popup.png)

4. Conservez les valeurs par défaut dans la boîte de dialogue **Importer des données**, puis sélectionnez **OK**.

   ![Azure Data Catalog--données d’importation Excel](media/register-data-assets-tutorial/data-catalog-excel-import-data.png)

5. La source de données s’affiche dans Excel.

   ![Azure Data Catalog--table de produits dans Excel](media/register-data-assets-tutorial/data-catalog-connect2.png)

### <a name="sql-server-management-studio"></a>SQL Server Management Studio

Dans cet exercice, vous vous êtes connecté aux ressources de données découvertes à l’aide d’Azure Data Catalog. Le portail Azure Data Catalog vous permet de vous connecter directement à l’aide d’applications clientes intégrées au menu **Ouvrir dans** . Vous pouvez également vous connecter avec l’application de votre choix en utilisant les informations d’emplacement de connexion incluses dans les métadonnées des ressources. Par exemple, vous pouvez utiliser SQL Server Management Studio pour vous connecter à Azure SQL Database afin d’accéder aux données des ressources de données inscrites dans ce tutoriel.

1. Ouvrez **SQL Server Management Studio**.

2. Dans la boîte de dialogue **Se connecter au serveur**, entrez le nom du serveur dans le volet **Propriétés** du portail Azure Data Catalog.

3. Utilisez l’authentification et les informations d’identification appropriées pour accéder à la ressource de données. Si vous n’avez pas accès à la ressource, utilisez les informations du champ **Demander l’accès** pour l’obtenir.

   ![Azure Data Catalog--demander l’accès](media/register-data-assets-tutorial/data-catalog-request-access.png)

Sélectionnez **Afficher les chaînes de connexion** pour afficher et copier les chaînes de connexion ADO.NET, ODBC et OLEDB dans le Presse-papiers pour les utiliser dans votre application.

## <a name="manage-data-assets"></a>Gérer les ressources de données

Dans cette étape, vous apprenez à configurer la sécurité de vos ressources de données. Data Catalog ne permet pas aux utilisateurs d’accéder aux données proprement dites. Le propriétaire de la source de données contrôle l’accès aux données.

Data Catalog vous permet de découvrir les sources de données et d’afficher les métadonnées relatives aux sources inscrites dans le catalogue. Il peut arriver, cependant, que certaines sources de données ne doivent être visibles que pour des utilisateurs spécifiques ou des membres de groupes spécifiques. Pour ces scénarios, vous pouvez utiliser Data Catalog pour vous approprier les ressources de données inscrites et contrôler la visibilité des ressources que vous détenez.

> [!NOTE]
> Les fonctionnalités de gestion décrites dans cet exercice sont disponibles uniquement dans l’édition Standard d’Azure Data Catalog, et non dans l’édition gratuite.
> Dans Azure Data Catalog, vous pouvez vous approprier des ressources de données, ajouter des copropriétaires aux ressources de données et définir la visibilité de ces ressources.

### <a name="take-ownership-of-data-assets-and-restrict-visibility"></a>S’approprier les ressources de données et restreindre leur visibilité

1. Accédez à la [page d’accueil Azure Data Catalog](https://www.azuredatacatalog.com). Dans la zone de texte **Recherche**, entrez `tags:cycles` et appuyez sur **ENTRÉE**.

2. Sélectionnez un élément dans la liste des résultats, puis **Appropriation** dans la barre d’outils.

3. Dans la section **Gestion** du panneau **Propriétés**, sélectionnez **Appropriation**.

    ![Azure Data Catalog--prendre possession](media/register-data-assets-tutorial/data-catalog-take-ownership.png)

4. Pour restreindre la visibilité, choisissez **Propriétaires et ces utilisateurs** dans la section **Visibilité**, puis sélectionnez **Ajouter**. Entrez les adresses de messagerie des utilisateurs dans la zone de texte et appuyez sur **ENTRÉE**.

    ![Azure Data Catalog--restreindre l’accès](media/register-data-assets-tutorial/data-catalog-ownership.png)

## <a name="remove-data-assets"></a>Supprimer les ressources de données

Dans cet exercice, vous utilisez le portail Azure Data Catalog pour supprimer les aperçus de données des ressources de données inscrites et pour supprimer les ressources de données du catalogue.

Dans Azure Data Catalog, vous pouvez supprimer une ou plusieurs ressources.

1. Accédez à la [page d’accueil Azure Data Catalog](https://www.azuredatacatalog.com).

2. Dans la zone de texte **Recherche**, entrez `tags:cycles`, puis sélectionnez **ENTRÉE**.

3. Sélectionnez un élément dans la liste des résultats, puis sélectionnez **Supprimer** dans la barre d’outils, comme illustré dans l’image suivante :

   ![Azure Data Catalog--supprimer un élément de grille](media/register-data-assets-tutorial/data-catalog-delete-grid-item.png)

   Si vous utilisez le mode Liste, la case à cocher se situe à gauche de l’élément, comme indiqué dans l’image suivante :

   ![Azure Data Catalog--supprimer un élément de liste](media/register-data-assets-tutorial/data-catalog-delete-list-item.png)

   Vous pouvez également sélectionner et supprimer plusieurs ressources de données comme indiqué dans l’image suivante :

   ![Azure Data Catalog--supprimer plusieurs ressources de données](media/register-data-assets-tutorial/data-catalog-delete-assets.png)

> [!NOTE]
> Par défaut, le catalogue permet aux utilisateurs d’inscrire n’importe quelle source de données et de supprimer toutes les ressources de données préalablement inscrites. Les fonctionnalités de gestion incluses dans l’édition Standard d’Azure Data Catalog offrent des options supplémentaires qui permettent l’appropriation de ressources, ce qui limite les fonctions de découverte et de suppression des ressources à certains utilisateurs.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Procédez de la manière décrite dans [Supprimer les ressources de données](#remove-data-assets) pour nettoyer les ressources que vous avez utilisées en suivant ce tutoriel.

## <a name="summary"></a>Résumé

Dans ce didacticiel, vous avez exploré les fonctionnalités essentielles d’Azure Data Catalog, notamment l’inscription, l’annotation, la découverte et la gestion des ressources de données d’entreprise. Maintenant que vous avez terminé ce didacticiel, il est temps de passer à la pratique. Vous pouvez commencer dès aujourd’hui en enregistrant les sources de données que vous et votre équipe utilisez, et en invitant vos collègues à utiliser le catalogue.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Sources de données prises en charge](data-catalog-dsr.md)
