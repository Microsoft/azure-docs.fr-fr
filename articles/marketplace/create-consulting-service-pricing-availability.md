---
title: Comment configurer la tarification et de la disponibilité de votre service de conseil dans l’Espace partenaires Microsoft
description: Découvrez comment configurer les détails de prix de votre offre de service de conseil sur la place de marché commerciale Microsoft à l’aide de l’Espace partenaires.
author: Microsoft-BradleyWright
ms.author: brwrigh
ms.reviewer: anbene
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 10/27/2020
ms.openlocfilehash: 6b386238bd759714bc0c8ad81d67c29aa1774aba
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96780227"
---
# <a name="how-to-configure-your-consulting-service-pricing-and-availability"></a>Comment configurer la tarification et de la disponibilité de votre service de conseil

Cet article explique comment définir la disponibilité sur le marché et les détails de tarification de votre offre de service de conseil sur le marketplace commercial de Microsoft.

> [!NOTE]
> Les offres de services de conseil ne sont que du référencement. Toutes les transactions doivent être effectuées entre vos clients et vous en dehors de la place de marché commerciale.

## <a name="markets"></a>Marchés

Dans la section **Marchés**, sélectionnez les pays ou les régions dans lesquels votre service de conseil sera disponible.

1. Sous **Marchés**, sélectionnez le lien **Modifier les marchés**.
2. Dans la boîte de dialogue qui s’affiche, sélectionnez les emplacements de marché où vous souhaitez que votre offre soit disponible. Vous devez sélectionner au minimum 1 et au maximum 141 marchés.
3. Sélectionnez **Enregistrer** pour fermer la boîte de dialogue.

## <a name="preview-audience"></a>Public de préversion

Lorsque vous publiez ou mettez à jour votre offre de service de conseil, l’Espace partenaires crée une préversion du référencement. Cette préversion n’est visible que d’utilisateurs disposant d’une **clé de masquage**.

Dans le champ **Masquer la clé**, entrez une chaîne alphanumérique pour accéder à la préversion de votre offre.

## <a name="pricing-informational-only"></a>Tarification (à titre purement informatif)

Dans la section **Tarification**, spécifiez si votre offre est gratuite ou payante.

Pour une offre payante, indiquez s’il s’agit d’un prix fixe ou d’une estimation. S’il s’agit d’une estimation, la description de votre offre doit décrire les facteurs déterminant le prix.

Si vous avez choisi un seul pays ou une seule région dans la section **Marchés**, fournissez le prix dans une devise prise en charge pour ce marché, puis sélectionnez **Enregistrer le brouillon**. Pour accéder à la liste des devises prises en charge, accédez à la page, consultez [Disponibilité géographique et prise en charge des devises pour le marketplace commercial](./marketplace-geo-availability-currencies.md).

Si vous avez choisi plusieurs pays ou régions dans la section **Marchés**, indiquez le prix en dollars américains (USD) et sélectionnez **Enregistrer le brouillon**. Espace partenaires convertit ce prix en devise locale pour tous les marchés sélectionnés à l’aide des taux de change disponibles lorsque vous avez enregistré le brouillon.

Pour valider la conversion ou fixer des prix personnalisés dans un marché individuel, vous devez exporter, modifier puis importer la feuille de calcul de tarification :

1. Sous **Tarification**, sélectionnez le lien **Exporter les données de tarification**. Cela permet de télécharger un fichier sur votre appareil.
1. Ouvrez le fichier exportedPrice.xlsx dans Microsoft Excel.
1. Dans la feuille de calcul, vous pouvez ajuster les prix et les devises pour chaque marché. Pour accéder à la liste des devises prises en charge, accédez à la page, consultez [Disponibilité géographique et prise en charge des devises pour le marketplace commercial](./marketplace-geo-availability-currencies.md). Quand vous avez terminé, enregistrez le fichier.
1. Dans Espace partenaires, sous **Tarification**, sélectionnez le lien **Exporter les données de tarification**. L’importation du fichier remplacera les informations tarifaires précédentes.

> [!IMPORTANT]
> Les prix que vous définissez dans Espace partenaires sont statiques et ne suivent pas les variations des taux de change. Pour modifier le prix d’un ou de plusieurs marchés après la publication, mettez à jour votre offre et soumettez-la à nouveau dans Espace partenaires.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="next-steps"></a>Étapes suivantes

* [Vérifier et publier](review-publish-offer.md)