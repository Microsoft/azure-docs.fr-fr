---
title: Meilleures pratiques pour le référencement d’offres – Place de marché commerciale Microsoft
description: Découvrez les meilleures pratiques de référencement pour la commercialisation de vos offres sur Microsoft AppSource et dans la Place de marché Azure.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 07/06/2020
ms.openlocfilehash: 3ea6a0035a9f9354be5c14699936c6a07dea1150
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94492087"
---
# <a name="offer-listing-best-practices"></a>Bonnes pratiques pour le référencement des offres

Cet article propose des suggestions pour la création et l’engagement d’offres de la Place de marché commerciale Microsoft. Les tableaux suivants décrivent les meilleures pratiques pour entrer les informations d’offre dans l’Espace partenaires. Pour une analyse des performances de vos offres, accédez au [tableau de bord Marketplace Insights](https://partner.microsoft.com/dashboard/commercial-marketplace/analytics/marketplaceinsights) dans l’Espace partenaires. 

## <a name="online-store-offer-details"></a>Détails de l’offre du magasin en ligne

| Paramètre | Bonne pratique |
|:--- |:--- |  
| Nom de l’offre | Pour les applications, fournissez un titre clair qui comprend des mots clés de recherche pour aider les clients à découvrir votre offre. <br> <br> Pour les services de conseil, suivez ce format : [nom de l’offre : [durée] [type d’offre] (par exemple, Contoso : implémentation de 2 semaines) |
| Description de l’offre | Fournissez une description claire qui décrit la proposition de valeur de votre offre dès les premières phrases.  Gardez à l’esprit que ces phrases peuvent être utilisées dans les résultats du moteur de recherche. Votre proposition de valeur doit comprendre les principaux composants suivants : <ul> <li>Description du produit ou de la solution. </li> <li> Personnage utilisateur qui tire parti du produit ou de la solution. </li> <li> Besoin ou problèmes du client auxquels le produit ou la solution répond. </li> </ul> <br> Dans la mesure du possible, utilisez le vocabulaire standard du secteur ainsi que des termes décrivant les avantages proposés.  Ne comptez pas sur les fonctionnalités de votre produit pour convaincre.  Concentrez-vous plutôt sur ce que votre application peut apporter aux clients. <br> <br> Pour les annonces de service de conseil, indiquez clairement le service professionnel que vous fournissez. |

> [!IMPORTANT]
> Vérifiez que votre nom d’offre et votre description d’offre répondent aux **[directives des marques Microsoft](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general.aspx)** ainsi qu’aux noms des logiciels, produits et services Microsoft, suivez les Directives relatives aux marques et aux marques déposées Microsoft ainsi que d’autres directives pertinentes et propres aux produits.

## <a name="online-store-listing-details"></a>Détails du listing du magasin en ligne

Suivant le magasin en ligne, les catégories et les secteurs s’appliquent à différents types d’offres.

| Magasin en ligne | Catégories <br>par magasin en ligne | Catégories <br>par magasin en ligne | Secteurs d’activité <br> pour AppSource |
| :------------------- |:----------------:|:------:|:-------------:|
| **Type d’offre**   |  **Azure Marketplace**  | **AppSource**  |
| Azure App | X | |
| Conteneur | X | |
| Services de conseil | | | X |
| Dynamics 365 Customer Engagement & Power Platform | | X | X |
| Dynamics 365 Finance & Supply Chain Management | | X | X | 
| Dynamics 365 Business Central | | X | X |
| Modules IoT Edge | X | |
| Power BI | | X | X |
| SaaS | X | X | X |
| Machine virtuelle Azure |  X |    |

### <a name="categories"></a>Catégories

Microsoft AppSource et la Place de marché Azure sont des magasins en ligne qui offrent différents types de solutions. La Place de marché Azure propose des solutions informatiques créées sur ou pour Azure.  Microsoft AppSource propose des solutions d’entreprise, telles que des applications SaaS métier et/ou sectorielles, de compléments Dynamics 365, de compléments Microsoft 365 et d’applications Power Platform.

Les catégories et sous-catégories sont mappées à chaque magasin en ligne en fonction du type de votre solution. Votre offre sera publiée sur Microsoft AppSource ou la Place de marché Azure en fonction du type d’offre, des possibilités de transaction de l’offre et du choix de catégorie/sous-catégorie. 

Sélectionnez les catégories et sous-catégories qui correspondent le mieux à votre type de solution. Vous pouvez sélectionner :

* Jusqu’à deux catégories, y compris une catégorie principale et une catégorie secondaire (facultatif).
* Jusqu’à deux sous-catégories pour chaque catégorie principale et/ou secondaire. Si aucune sous-catégorie n’est sélectionnée, votre offre sera tout de même découvrable sur la catégorie sélectionnée uniquement.

[!INCLUDE [categories and subcategories](./includes/categories.md)]

#### <a name="important-saas-offers-and-microsoft-365-add-ins"></a>IMPORTANT : Offres SaaS et compléments Microsoft 365

Pour plus d’informations sur l’impact des fonctionnalités de transaction sur la visualisation et l’achat de votre offre par les clients de la place de marché, consultez [Transaction dans la Place de marché commerciale](marketplace-commercial-transaction-capabilities-and-considerations.md). Pour les offres SaaS, la fonctionnalité de transaction de l’offre ainsi que le choix de la catégorie déterminent le magasin en ligne dans laquelle votre offre est publiée.


| Offre SaaS    | Offre SaaS   | Offre SaaS  | Offre SaaS   | Offre SaaS   | Offre SaaS   | Offre SaaS    | Magasin en ligne applicable| Magasin en ligne applicable |
|:-------------:|:---:|:--------:|:---------:|:--:|:--:|:---:|:---------------------:|:-------------:|
| Facturation à l’usage | Compléments Microsoft 365 | Me contacter | Transaction (au moins 1 plan) | Plan privé uniquement | Plan public uniquement | Plans privés et publics | AppSource | Place de marché Azure |
|  | X |  |  |  |  |  | X |  |
| X |  |  | X | X |  |  |  | X |
| X |  |  | X |  | X |  |  | X |
| X |  |  | X |  |  | X |  | X<sup>2</sup> |
|  |  |  | X | X |  |  |  | X |
|  |  |  | X |  | X |  | X<sup>1</sup> | X<sup>1</sup> |
|  |  |  | X |  |  | X | X<sup>1</sup> | X<sup>1,2</sup> |
|  |  | X |  |  |  |  | x<sup>1</sup> | X<sup>1</sup> | 

1. En fonction de la catégorie/sous-catégorie et du choix du secteur
2. Les offres avec des plans privés sont publiées sur le portail Azure


### <a name="industries"></a>Secteurs d’activité

Le choix de secteur s’applique uniquement aux offres publiées sur AppSource et aux services de Conseil publiés dans la Place de marché Azure.  Sélectionnez les secteurs et/ou secteurs verticaux si votre offre répond à des besoins spécifiques du secteur, en appelant des fonctionnalités spécifiques du secteur dans la description de votre offre. Vous pouvez sélectionner jusqu’à deux (2) secteurs et deux (2) secteurs verticaux par secteur d’activité sélectionné.

>[!Note]
>Pour les offres de service de conseil dans la Place de marché Azure, il n’y a pas de segments verticaux.

| **Secteurs d’activité** |  **Segments** |
| :------------------- | :----------------|
| **Agriculture** | |
| **Architecture et construction** | |
| **Automobile** | |
| **Distribution** | Vente en gros <br> Expédition de paquets et colis |  
| **Education** | Enseignement supérieur <br> Enseignement primaire et secondaire/M-12 <br> Bibliothèques et musées |
| **Services financiers** | Banques et marchés de capitaux <br> Assurance | 
| **Secteur public** |  Défense et intelligence <br> Gouvernement civil <br> Sécurité publique et justice |
| **Santé** | Organisme de sécurité sociale <br> Professionnel de santé <br> Produits pharmaceutiques | 
| **Tourisme et voyage** | Transport et voyages <br> Hôtels et loisirs <br> Restaurants et services traiteur | 
| **Industrie et ressources** | Produits chimiques et agrochimiques <br> Fabrication discrète <br> Énergie | 
| **Médias et communications** | Multimédia et divertissement <br> Télécommunications | 
| **Autres secteurs publics** | Foresterie et pêche <br> Association sans but lucratif | 
| **Services professionnels** | Services professionnels partenaires <br> Informations juridiques | 
| **Immobilier** | |

Secteur pour Microsoft AppSource uniquement :

| **Secteur d’activité** |  **Segments** |
| :------------------- | :----------------|
| **Commerce et produits de consommation** | Détaillants <br> Produits de consommation |

### <a name="applicable-products"></a>Produits applicables

Sélectionnez les produits applicables utilisés par votre application pour que l’offre s’affiche sous les produits sélectionnés dans AppSource.

### <a name="search-keywords"></a>Mots clés de recherche

Les mots clés peuvent aider les clients à trouver votre offre lorsqu’ils effectuent une recherche. Identifiez les principaux mots clés de recherche pour votre offre, intégrez-les dans le résumé et la description de votre offre, ainsi que dans la section mot clé de la section de détails de la liste d’offres.

## <a name="online-store-marketing-details"></a>Détails marketing du magasin en ligne
| Paramètre | Bonne pratique |
|:--- |:--- |  
| Logo de l’offre (format PNG, de 216 × 216 à 350 x 350 px) : page Détails de l’application | Concevez et optimisez votre logo pour un support numérique :<br>Chargez le logo au format PNG vers la page détaillant les applications de votre offre. L’Espace partenaires le redimensionne sur les tailles de logo requises. |
| Logo de l’offre (format PNG, 48 x 48 pixels) : page de recherche | L’Espace partenaires génère ce logo à partir du grand logo que vous avez chargé. Si vous le souhaitez, vous pouvez par la suite le remplacer par une autre image. |
| Documents « Plus d’informations » | Incluez dans la section « Plus d’informations » des ressources de ventes et marketing de support, par exemple :<ul><li>livres blancs</li><li> brochures</li><li>Checklists</li><li> présentations PowerPoint</li></ul><br>Enregistrez tous les fichiers au format PDF. Votre objectif avec ces documents doit être de renseigner les clients, pas de leur faire acheter.<br><br>Ajoutez un lien à votre application vers la page d’accueil avec tous vos documents, et ajoutez des paramètres d’URL pour vous assister dans le suivi des visiteurs et des essais gratuits. |
| Vidéos : offres AppSource, de services de conseil et SaaS uniquement | Les vidéos les plus percutantes expriment la valeur de votre offre sous forme narrative :<ul> <li> Faites du client le héros de l’histoire, et non votre entreprise. </li> <li> Votre vidéo doit évoquer les principaux défis et objectifs de votre client cible. </li> <li> Durée recommandée : 60 à 90 secondes.</li> <li> Incorporez des mots clés de recherche similaires aux noms des vidéos. </li> <li> Vous pouvez ajouter d’autres vidéos, telles qu’une vidéo explicative, un tutoriel ou des témoignages client. </li> </ul> |
| Captures d’écran (1280&nbsp;&times;&nbsp;720) | Ajoutez jusqu'à cinq captures d’écran :<br>Incorporez les mots clés de recherche dans le nom des fichiers. |

## <a name="link-to-your-offer-page-from-your-website"></a>Lien vers la page de votre offre à partir de votre site web

Quand vous créez un lien entre le badge AppSource ou Place de marché Azure de votre site et votre annonce sur la place de marché commerciale, vous pouvez inclure les paramètres de requête suivants à la fin de l’URL pour prendre en charge de puissantes fonctionnalités d’analyse et de création de rapports :
* **src** : ajoute la source à partir de laquelle le trafic est routé jusqu’à AppSource (par exemple, un site web, LinkedIn ou Facebook).
* **mktcmpid**  : votre ID de campagne marketing, qui peut contenir jusqu’à 16 caractères dans n’importe quelle combinaison de lettres, chiffres, tirets bas et traits d’union (par exemple, *blogpost_12* ).

L’exemple d’URL suivant contient à la fois les deux paramètres de requête précédents : `https://appsource.microsoft.com/product/dynamics-365/mscrm.04931187-431c-415d-8777-f7f482ba8095?src=website&mktcmpid=blogpost_12`

L’ajout de ces paramètres dans votre URL AppSource vous permet d’observer l’efficacité de votre campagne dans le tableau de bord d’analytique de l’[Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace/).

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en plus sur les [avantages de la place de marché commerciale](./gtm-your-marketplace-benefits.md).

Connectez-vous à l’[Espace partenaires](https://partner.microsoft.com/dashboard/account/v3/enrollment/introduction/partnership) pour créer et configurer votre offre.
