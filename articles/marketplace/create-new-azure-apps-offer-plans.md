---
title: Comment créer des plans pour votre offre Azure Application
description: Découvrez comment créer des plans pour votre offre Azure Application dans l’Espace partenaires.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 11/06/2020
ms.openlocfilehash: fd5ee9085cf716128c5b3ae073f963c76d2bd17a
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94369742"
---
# <a name="how-to-create-plans-for-your-azure-application-offer"></a>Comment créer des plans pour votre offre Azure Application

Les offres vendues par le biais de la Place de marché commerciale Microsoft doivent avoir au moins un plan pour répertorier votre offre sur la place de marché commerciale. Vous pouvez créer divers plans avec différentes options dans la même offre. Ces plans (parfois appelés « références SKU ») peuvent se différencier par le type de plan (_modèle de solution_ ou _application managée_), la monétisation ou le public. Pour obtenir des conseils d’ordre général sur les plans, consultez [Plans et tarification des offres de la Place de marché commerciale](plans-pricing.md).

## <a name="create-a-plan"></a>Créer un plan

1. Vers le haut de l’onglet **Vue d’ensemble du plan** , sélectionnez **+ Créer un plan**.
1. Dans la boîte de dialogue qui s’affiche, dans la zone **ID du plan**, entrez un ID de plan unique. Cet ID sera visible par les clients dans l’URL du produit. Utilisez jusqu’à 50 caractères alphanumériques en minuscules, tirets ou traits de soulignement. Vous ne pouvez pas modifier l’ID de plan une fois que vous avez sélectionné **Créer**.
1. Dans la zone **Nom du plan**, entrez un nom unique pour ce plan. Les clients verront ce nom lorsqu’ils décideront du plan à sélectionner dans votre offre. Utilisez un maximum de 50 caractères.
1. Sélectionnez **Create** (Créer).

## <a name="define-the-plan-setup"></a>Définir la configuration des plans

L’onglet **Configuration du plan** vous permet de définir le type de plan, si le plan réutilisera la configuration technique d’un autre plan et dans quelles régions Azure le plan sera disponible. Les choix effectués dans cet onglet déterminent les champs affichés dans les autres onglets de ce plan.

### <a name="select-the-plan-type"></a>Sélectionner le type de plan

- Dans la liste **Type de plan**, sélectionnez **Modèle de solution** ou **Application managée**.

Un plan **modèle de solution** est entièrement managé par le client. Un plan d’**application managée** permet aux éditeurs de manager l’application pour le compte du client. Pour plus d’informations sur ces deux types de plans, consultez [Types de plans](plan-azure-application-offer.md#types-of-plans).

### <a name="re-use-technical-configuration-optional"></a>Réutiliser la configuration technique (facultatif)

Si vous avez créé plusieurs plans du même type au sein de cette offre et que la configuration technique est identique, vous pouvez réutiliser la configuration technique d’un autre plan. Ce paramètre n’est plus modifiable une fois le plan publié.

#### <a name="to-re-use-technical-configuration"></a>Réutiliser la configuration technique

1. Activez la case à cocher **Ce plan réutilise la configuration technique d’un autre plan du même type**.
1. Dans la liste qui s’affiche, sélectionnez le plan de base souhaité.

> [!NOTE]
> Si vous réutilisez les packages d’un autre plan, l’onglet **Configuration technique** ne s’affichera plus pour ce plan. En effet, tous les paramètres de la configuration technique de l’autre plan, y compris les mises à jour effectuées ultérieurement, sont également appliqués à ce plan.

### <a name="select-azure-regions-clouds"></a>Sélectionner des régions Azure (clouds)

Votre plan doit être rendu disponible dans au moins une région Azure. Une fois votre plan publié et disponible dans une région Azure spécifique, vous ne pouvez pas supprimer cette région de votre offre.

#### <a name="azure-global-region"></a>Région globale Azure

La case à cocher **Azure global** est sélectionnée par défaut. Cela mettra votre plan à la disposition des clients de toutes les régions Azure qui disposent d’une intégration de la Place de marché commerciale. Pour les plans d’application managée, vous pouvez sélectionner les marchés sur lesquels vous souhaitez mettre votre plan à disposition.

Pour supprimer votre offre de cette région, désactivez la case à cocher **Azure global**.

#### <a name="azure-government-region"></a>Région Azure Government

Cette région fournit un accès contrôlé aux clients d’entités gouvernementales fédérales, étatiques, locales ou tribales des États-Unis, ainsi qu’aux partenaires éligibles pour les servir. En tant qu’éditeur, il vous incombe de mettre en place l’ensemble des contrôles de conformité, des mesures de sécurité et des bonnes pratiques nécessaires. Azure Government utilise des réseaux et des centres de données qui sont physiquement isolés (situés aux États-Unis uniquement).

Les services Azure Government gèrent des données soumises à certaines réglementations et exigences gouvernementales. Par exemple, FedRAMP, NIST 800.171 (DIB), ITAR, IRS 1075, DoD L4 et CJIS. Afin de renseigner les clients sur vos certifications pour ces programmes, vous pouvez fournir jusqu’à 100 liens décrivant ces certifications. Ces liens peuvent être des liens vers votre liste de plans directement sur le programme, ou des liens vers des descriptions de votre conformité avec ces certifications sur vos propres sites web. Ces liens sont visibles uniquement par les clients Azure Government.

##### <a name="to-select-the-azure-government-region"></a>Pour sélectionner la région Azure Government

1. Activez la case à cocher **Azure Government**.
1. Sous **Certifications Azure Government**, sélectionnez **+ Ajouter une certification (max. 100)** .
1. Dans les zones qui s’affichent, fournissez un nom et un lien vers une certification.
1. Pour ajouter une autre certification, répétez les étapes 2 et 3.

Avant de passer à l’onglet suivant, sélectionnez **Enregistrer le brouillon** : Liste des plans.

## <a name="define-the-plan-listing"></a>Définir la liste des plans

L’onglet **Liste des plans** est l’endroit où vous configurez les détails de la description du plan. Cet onglet affiche des informations spécifiques qui indique la différence entre les plans au sein de la même offre. Vous pouvez définir le nom, le résumé et la description du plan comme vous souhaitez qu’ils apparaissent dans la Place de marché commerciale.

1. Dans la zone **Nom du plan**, le nom que vous avez fourni précédemment pour ce plan s’affiche. Vous pouvez le modifier à tout moment. Ce nom apparaîtra dans la Place de marché commerciale comme titre du plan logiciel de votre offre et est limité à 100 caractères.
1. Dans la zone **Résumé du plan**, fournissez un bref résumé de votre plan (pas de l’offre). Ce résumé est limité à 100 caractères.
1. La zone **Description du plan** permet d’expliquer ce qui rend ce plan logiciel unique et de pointer les différences par rapport aux autres plans dans votre offre. Ne décrivez pas l’offre, mais simplement le plan. Cette description peut contenir jusqu’à 2 000 caractères.
1. Sélectionnez **Enregistrer le brouillon** avant de continuer.

## <a name="next-steps"></a>Étapes suivantes

Effectuez l’une des actions suivantes :

- Si vous configurez un plan de modèle de solution, accédez à [Configurer un plan de modèle de solution](create-new-azure-apps-offer-solution.md).
- Si vous configurez un plan d’application managée, reportez-vous à [Configurer un plan d’application managée](create-new-azure-apps-offer-managed.md).
