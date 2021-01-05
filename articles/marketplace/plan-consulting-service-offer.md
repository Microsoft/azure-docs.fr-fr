---
title: Comment planifier une offre de services de conseil pour le marketplace commercial de Microsoft
description: Comment planifier une nouvelle offre de service de conseil pour Microsoft AppSource ou Place de marché Azure à l’aide du programme de marketplace commercial dans Espace partenaires Microsoft.
author: Microsoft-BradleyWright
ms.author: brwrigh
ms.reviewer: anbene
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 10/27/2020
ms.openlocfilehash: 10c06f5b36de880302f17dc17c1d47d5209dd902
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97588469"
---
# <a name="how-to-plan-a-consulting-service-offer-in-the-commercial-marketplace"></a>Comment planifier une offre de services de conseil dans le marketplace commercial

Cet article présente les différentes options et exigences relatives à la publication d’une offre de services de conseil sur le marketplace commercial de Microsoft. Lisez cet article pour préparer votre offre en vue de sa publication par le biais d’Espace partenaires.

## <a name="whats-a-consulting-service"></a>Qu’est-ce qu’un service de conseil ?

Les services de conseil sont des contrats clients virtuels ou en personne qui prennent en charge et étendent l’utilisation des produits Microsoft par les clients. Grâce à un service de conseil, les clients peuvent évaluer et déployer des solutions qui leur permettent d’atteindre leurs objectifs commerciaux. Vous choisissez la portée, la durée et la structure tarifaire (prix fixe ou gratuit) de votre service.

Les offres de service de conseil sont publiées sous la forme de publications **Contactez-moi**. Cela signifie que les clients vous contacteront directement à partir des informations que vous publiez dans votre offre. Bien que Microsoft héberge la publication sur le marketplace commercial, vous gérez toutes les interactions avec les clients. Vous êtes responsable du service, de la facturation et du règlement par le client.

## <a name="primary-products-and-online-stores"></a>Produits principaux et magasins en ligne

Chaque service de conseil doit se concentrer sur l’un de ces produits Microsoft, appelé le **produit principal** de l’offre :

:::row:::
    :::column:::
        Azure

        Dynamics 365 Business Central

        Dynamics 365 Commerce

        Dynamics 365 Customer Insights

        Dynamics 365 Customer Service

        Dynamics 365 Field Service

        Dynamics 365 Finance

        Dynamics 365 Marketing
    :::column-end:::
    :::column:::
        Dynamics 365 Project Service Automation

        Dynamics 365 Sales

        Dynamics 365 Supply Chain Management

        Microsoft 365

        Power Apps

        Power Automate

        Power BI

        Power Virtual Agents
    :::column-end:::
:::row-end:::

Le produit principal que vous sélectionnez dans Espace partenaires détermine si votre offre sera publiée sur Microsoft AppSource ou sur Place de marché Azure.

* Si le produit principal de votre service de conseil est Azure, l’offre est publiée sur Place de marché Azure.
* Si le produit principal n’est pas Azure, l’offre est publiée sur AppSource.

Pour en savoir plus sur les différences entre AppSource et Place de marché Azure, consultez [Magasins en ligne du marketplace commercial](./overview.md#commercial-marketplace-online-stores).

## <a name="eligibility-requirements"></a>Conditions d’éligibilité

Pour démontrer aux clients votre expertise dans un domaine, vous devez remplir un ensemble de conditions d’éligibilité avant de publier une offre de service de conseil. Ces conditions dépendent du produit au cœur de votre offre. La liste complète des conditions d’éligibilité pour chaque produit principal est accessible dans les [stratégies de certification pour les services de conseil](/legal/marketplace/certification-policies#800-consulting-services).

> [!NOTE]
> Pour certains produits principaux, vous devez disposer d’une compétence Microsoft Gold ou Silver dans votre domaine de solution. Pour plus d’informations, consultez [Compétences Microsoft Partner Network](https://partner.microsoft.com/membership/competencies).

## <a name="service-type-and-duration"></a>Type de service et durée

Le marketplace commercial prend en charge cinq types de service de conseil :

* **Évaluation** : évaluation de l’environnement d’un client pour déterminer la mise en application d’une solution et estimer le coût et la chronologie de son implémentation.
* **Réunion d’information** : présentation d’une solution ou d’un service à l’aide d’infrastructures, de versions de démonstration et d’exemples de clients.
* **Implémentation** : installation complète qui résulte en une solution entièrement fonctionnelle.
* **Preuve de concept** : implémentation de portée limitée pour déterminer si une solution répond aux exigences du client.
* **Atelier** : engagement interactif réalisé dans les locaux du client. Il peut inclure une formation, des réunions d’information, des évaluations ou des démonstrations reposant sur les données ou l’environnement du client.

Votre service doit avoir une durée fixe et prédéterminée pouvant atteindre 10 semaines. La durée du service doit être rendue explicite dans la publication de l’offre.

## <a name="customer-leads"></a>Prospects

Vous devez connecter votre offre à votre système de gestion des relations avec la clientèle (Customer Relationship Management, CRM) pour collecter des informations sur les clients. Le client devra autoriser le partage de ses informations. Ces détails du client, ainsi que le nom de l’offre, son ID et le magasin en ligne dans lequel il a trouvé votre offre, seront envoyés au système CRM que vous avez configuré. Le marketplace commercial prend en charge différents types de systèmes CRM, ainsi que la possibilité d’utiliser une table Azure ou de configurer un point de terminaison HTTPS à l’aide de Power Automate.

Vous pouvez ajouter ou modifier une connexion CRM à tout moment pendant ou après la création de l’offre. Pour obtenir des instructions détaillées, consultez [Prospects de votre offre de marketplace commercial](partner-center-portal/commercial-marketplace-get-customer-leads.md).

## <a name="offer-listing-details"></a>Détails du référencement de l’offre

Lorsque vous créez votre offre de service de conseil dans Espace partenaires, vous entrez du texte, des images, des documents et d’autres détails relatifs à l’offre. C’est ce que les clients verront lorsqu’ils découvriront votre offre dans AppSource ou Place de marché Azure. Voir l’exemple suivant :

![Illustre la façon dont une offre de service de conseil apparaît dans les magasins en ligne](./media/example-consulting-service.png)

**Descriptions de légende**

1. Logo
2. Type de service
3. Compétences
4. Domaines de solution (Place de marché Azure) ou Produits (AppSource)
5. Secteurs d’activité
6. Pays/région
7. Nom de l’offre
8. Résumé des résultats de la recherche
9. Description
10. Captures d’écran/vidéos
11. Documents de soutien

> [!NOTE]
> Si vous livrez votre service dans une langue autre que l’anglais, la publication de l’offre peut être dans cette langue, mais la description doit commencer ou se terminer par la phrase en anglais « This service is available in &lt;language of your offer content> » (Ce service est disponible en <langue du contenu de votre offre>). Vous pouvez également fournir un complément de documentation dans une langue différente de celle utilisée dans les détails de l’offre.

Pour faciliter la création de votre offre, préparez certains de ces éléments à l’avance. Sauf indication contraire, les éléments suivants sont requis.

**Name** : ce nom apparaîtra en tant que titre du référencement de votre annonce dans la place de marché commerciale. Le nom peut être une marque. Il ne peut pas contenir d’emojis (sauf s’il s’agit du symbole de marque ou de copyright) et ne doit pas dépasser 50 caractères. Le nom doit inclure la durée et le type de service de l’offre afin de favoriser l’optimisation du référencement d’un site auprès d’un moteur de recherche (SEO). Le format requis est *Nom : durée + type*. N’incluez pas le nom de votre société, sauf s’il s’agit également du nom du produit. Voici quelques exemples :

|Ne pas dire |Dire  |
|---|---|
|Prise en main d’Azure IoT pour la fabrication |Fabrication IoT : Évaluation de 2 jours |
|Atelier portant sur les grille-pain intelligents |Grille-pain intelligents : Atelier d’une semaine |
|POC sur la migration de SQL Server par Contoso |Migration SQL : Preuve de concept de 3 semaines |
| | |

**Résultats de recherche récapitulatifs** : Décrivez l’objectif ou le but de votre offre en 200 caractères maximum. Ce résumé est utilisé dans les résultats de recherche de la publication sur le marketplace commercial. Il ne doit pas être identique au titre. Pensez à inclure vos principaux mots clés de SEO.

**Description** : Cette description s’affichera dans la publication sur le marketplace commercial. Pensez à inclure une proposition de valeur, les principaux avantages, la base d’utilisateurs visée et toute catégorie ou association de secteur d’activité.

Lors de la rédaction de la description, suivez ces critères, en fonction de votre type de service :

|Type de service |Exigences en matière de description |
|---|---|
|Évaluation |Incluez un calendrier détaillé des évaluations d’une durée supérieure à une journée et formulez les livrables dont le client bénéficiera. |
|Réunion d’information |Expliquez les livrables dont le client bénéficiera.|
|Implémentation |Incluez un calendrier détaillé des implémentations d’une durée supérieure à une journée et décrivez les modifications d’ingénierie, les artefacts techniques ou les autres artefacts que vous vous engagez à fournir à un client. |
|Preuve de concept |Décrivez les modifications d’ingénierie, les artefacts techniques ou les autres artefacts que vous vous engagez à fournir à un client. |
|Atelier |Incluez un calendrier détaillé quotidien, hebdomadaire ou mensuel, selon la durée de votre offre. Formulez les objectifs d’apprentissage ou d’autres livrables de votre atelier. |
| | |

Voici quelques conseils pour rédiger votre description :

* Décrivez clairement la valeur de votre offre dès les premières phrases, à savoir :
    * Le type d’utilisateur qui bénéficie de l’offre.
    * Les besoins ou problèmes auxquels l’offre répond.
* Gardez à l’esprit que les premières phrases peuvent être affichées dans les résultats de recherche.
* Si le prix de votre offre est estimé, expliquez les variables qui détermineront le prix final.
* Utilisez le vocabulaire spécifique au secteur d’activité.

Vous pouvez également utiliser des balises HTML pour mettre en forme votre description. Vous pouvez entrer dans cette zone de texte jusqu’à 2 000 caractères, balisage HTML et espaces compris. Pour plus d’informations sur la mise en forme HTML, consultez [Balises HTML prises en charge dans les descriptions d’offres de la place de marché commerciale](./supported-html-tags.md).

**Mots clés de recherche** (facultatif) : Entrez jusqu’à trois mots clés que les clients pourront utiliser pour rechercher votre offre dans les magasins en ligne. Vous n’avez pas besoin d’inclure le **nom** et la **description** de l’offre.

**Durée** : Votre offre de service de conseil doit avoir une durée prédéterminée de 10 semaines maximum.

**Coordonnées** : Dans Espace partenaires, vous êtes invité à fournir le nom, l’adresse e-mail et le numéro de téléphone de deux personnes de votre entreprise (vous pouvez être l’un des deux contacts). Nous utiliserons ces informations pour communiquer avec vous concernant votre offre. Elles ne sont pas présentées aux clients, mais peuvent être communiquées à des partenaires fournisseurs de solutions cloud (CSP).

**Documents d’accompagnement** : Chargez au moins un et jusqu’à trois documents PDF destinés aux clients qui fournissent des informations sur votre offre. Par exemple, il peut s’agir de livres blancs ou de brochures.

**Médias de marketplace – logos** : Fournissez un fichier PNG pour le logo de grande taille. Espace partenaires l’utilisera pour créer un petit logo. Si vous le souhaitez, vous pouvez remplacer le petit logo par une autre image ultérieurement.

* Grande taille (de 216 x 216 à 350 x 350 px, obligatoire)
* Petite taille (48 x 48 px, facultatif)

Le grand logo apparaît sur la page de référencement de votre offre dans Place de marché Azure ou AppSource. Le petit logo s’affiche dans les résultats de recherche sur Place de marché Azure, ainsi que dans la page principale et celle des résultats de recherche de Microsoft AppSource.

Suivez ces instructions pour vos logos:

* Assurez-vous que l’image n’est pas floue ni étirée.
* Le design Azure a une palette de couleurs simple. Limitez le nombre de couleurs primaires et secondaires sur votre logo.
* Si vous utilisez un arrière-plan transparent, assurez-vous que le logo et le texte ne sont pas blancs, noirs ou bleus.
* Évitez les dégradés dans le logo ou l’arrière-plan. Ne placez pas de texte sur le logo, pas même le nom de votre marque ou de votre entreprise.

**Médias de marketplace – captures d’écran** : Ajoutez entre une et cinq images qui illustrent votre offre. Chacune image doit avoir une taille de 1280 x 720 pixels et être au format .PNG.

**Médias de marketplace – vidéos** (facultatif) : Vous pouvez ajouter jusqu’à quatre vidéos qui illustrent votre offre. Les vidéos doivent être hébergées sur YouTube ou Vimeo et avoir une miniature (fichier PNG 1280 x 720).

## <a name="pricing-and-availability"></a>Tarification et disponibilité

Votre offre de service de conseil peut être mise à disposition dans un ou plusieurs pays ou régions. Dans Espace partenaires, vous pouvez décider du prix pour chaque marché que vous sélectionnez. Pour obtenir la liste complète des marchés et devises pris en charge, consultez [Disponibilité géographique et prise en charge des devises pour le marketplace commercial](./marketplace-geo-availability-currencies.md).


## <a name="next-steps"></a>Étapes suivantes

* [Créer une offre de services de conseil dans le marketplace commercial](./create-consulting-service-offer.md)
* [Bonnes pratiques pour le référencement des offres](./gtm-offer-listing-best-practices.md)