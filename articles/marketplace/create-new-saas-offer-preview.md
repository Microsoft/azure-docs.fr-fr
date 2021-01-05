---
title: Comment ajouter un public de préversion pour votre offre SaaS dans la place de marché commerciale Microsoft
description: Comment ajouter un public de préversion pour votre offre SaaS dans l’Espace partenaires Microsoft.
author: mingshen-ms
ms.author: mingshen
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 09/02/2020
ms.openlocfilehash: 041df9b77e85f09d2cf680773edc995c6288acaa
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96746418"
---
# <a name="how-to-add-a-preview-audience-for-your-saas-offer"></a>Comment ajouter un public de préversion pour votre offre SaaS

Lorsque vous créez votre offre SaaS dans l’Espace partenaires, vous devez définir un public de préversion qui peut consulter le référencement de l’offre avant sa mise en ligne. Cet article explique comment configurer un public de préversion.

> [!NOTE]
> Si vous décidez de traiter les transactions de façon indépendante, vous ne verrez pas cette option. Au lieu de cela, passez à [Comment vendre votre offre SaaS](create-new-saas-offer-marketing.md).

## <a name="define-a-preview-audience"></a>Définir un public de préversion

Sous l’onglet **Public de préversion**, vous pouvez définir un public limité pouvant consulter votre offre SaaS avant sa publication en ligne pour le public le plus large de la place de marché. Vous pouvez envoyer des invitations à des adresses e-mail de Compte Microsoft (MSA) ou d’Azure Active Directory (Azure AD). Ajoutez de 1 à 10 adresses e-mail manuellement, ou importez-en jusqu’à 20 à l’aide d’un fichier .csv. Vous pouvez mettre à jour la liste du public de préversion à tout moment.

> [!NOTE]
> Le public pour la préversion n’est pas le même que le public privé. Un public de préversion peut accéder à votre offre avant sa publication en direct dans les magasins en ligne. Vous pouvez aussi choisir de créer un plan et de le rendre disponible uniquement à un public privé. Les plans privés sont expliqués dans [Comment créer des plans pour votre offre SaaS](create-new-saas-offer-plans.md).

### <a name="add-email-addresses-manually"></a>Ajouter des adresses e-mail manuellement

1. Sur la page **Public de préversion**, ajoutez une adresse e-mail Azure AD ou MSA, et une description facultative dans les zones fournies.
1. Pour ajouter une autre adresse e-mail, sélectionnez le lien **Ajouter une autre adresse e-mail**.
1. Avant de passer à l’onglet suivant, sélectionnez **Enregistrer le brouillon** : **Configuration technique**.
1. Passez à [Comment ajouter des détails techniques pour votre offre SaaS](create-new-saas-offer-technical.md).

### <a name="add-email-addresses-using-the-csv-file"></a>Ajouter des adresses e-mail à l’aide du fichier CSV

1. Sur la page **Public de préversion**, sélectionnez le lien **Exporter le public (CSV)** .
1. Ouvrez le fichier .CSV dans une application, par exemple Microsoft Excel.
1. Dans le fichier .CSV, dans la colonne **ID**, entrez les adresses e-mail que vous souhaitez ajouter au public de préversion.
1. Dans la colonne **Description**, vous pouvez ajouter une description pour chaque adresse e-mail.
1. Dans la colonne **Type** , ajoutez **MixedAadMsaId** à chaque ligne contenant une adresse e-mail.
1. Enregistrez le fichier au format .CSV.
1. Sur la page **Public de préversion**, sélectionnez le lien **Importer le public (csv)** .
1. Dans la boîte de dialogue **Confirmer**, sélectionnez **Oui**.
1. Sélectionnez le fichier .CSV, puis **Ouvrir**.
1. Avant de passer à l’onglet suivant, sélectionnez **Enregistrer le brouillon** : **Configuration technique**.

## <a name="next-steps"></a>Étapes suivantes

- [Comment ajouter des détails techniques pour votre offre SaaS](create-new-saas-offer-technical.md)
