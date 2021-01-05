---
title: Comment tester et publier une offre SaaS sur la place de marché commerciale de Microsoft
description: Utilisez l’Espace Partenaires pour soumettre votre offre SaaS en préversion, afficher un aperçu de votre offre, la tester, puis la publier sur la place de marché commerciale Microsoft.
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: mingshen-ms
ms.author: mingshen
ms.date: 09/02/2020
ms.openlocfilehash: bef59ea9e7de77e7f9a80cc3950762ea70238b87
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89380377"
---
# <a name="how-to-test-and-publish-a-saas-offer-to-the-commercial-marketplace"></a>Comment tester et publier une offre SaaS sur la place de marché commerciale

Cet article explique comment utiliser l’Espace partenaires afin de soumettre votre offre SaaS pour publication, afficher un aperçu de votre offre, la tester, puis la publier en direct sur la place de marché commerciale. Vous devez avoir déjà créé une offre que vous souhaitez publier.

## <a name="submit-your-offer-for-publishing"></a>Soumettre votre offre pour publication

1. Connectez-vous au tableau de bord de la place de marché commerciale dans l’[Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace/overview).
1. Sur la page **Vue d’ensemble**, sélectionnez l’offre que vous souhaitez publier.
1. Dans l’angle supérieur droit du portail, sélectionnez **Vérifier et publier**.
2. Assurez-vous que la colonne **État** de chaque page indique **Complète**. Les états possibles sont les suivants :

   - **Non démarré** : la page n’est pas démarrée.
   - **Incomplète** : des informations requises sont manquantes dans la page ou des erreurs doivent être corrigées. Vous devez revenir à la page et la mettre à jour.
   - **Complète** : la page est complète. Toutes les données requises ont été fournies et il n’existe aucune erreur.

1. Si l’un des pages présente un État autre que **Complète**, sélectionnez le nom de la page, corrigez le problème, enregistrez la page, puis sélectionnez **Vérifier et publier** à nouveau pour revenir à cette page.
1. Après avoir compléter toutes les pages, dans la boîte **Notes pour la certification**, fournissez des instructions de test à l’équipe de certification pour vous assurer que votre application est correctement testée. Fournissez toutes les notes supplémentaires utiles pour comprendre votre demande.
1. Pour démarrer le processus de publication de votre offre, sélectionnez **Publier**. La page **Vue d’ensemble de l’offre** s’affiche et indique l’**État de publication** de l’offre.

L’état de publication de votre offre changera à mesure qu’il progressera dans le processus de publication. Pour plus d’informations sur ce processus, consultez [Étapes de validation et de publication](review-publish-offer.md#validation-and-publishing-steps).

## <a name="preview-and-test-your-offer"></a>Afficher un aperçu de votre offre et la tester

Une fois que l’offre est prête à être approuvée, nous vous enverrons un e-mail pour vous demander de consulter et d’approuver la préversion de votre offre. Vous pouvez également actualiser la page **Vue d’ensemble de l’offre** depuis votre navigateur pour voir si votre offre a atteint la phase d’approbation de publication. Si c’est le cas, le bouton **Go Live** et les liens d’aperçu seront disponibles. Il y aura un lien soit pour l’aperçu Microsoft AppSource, soit pour l’aperçu de la Place de marché Azure, ou les deux, selon les options que vous avez choisies lors de la création de votre offre. Si vous avez choisi de vendre votre offre par le biais de Microsoft, quiconque a été ajouté au public de la préversion peut tester son acquisition et son déploiement pour vérifier qu’elle répond à vos exigences au cours de cette phase.

La capture d’écran suivante montre la page **Vue d’ensemble de l’offre** pour une offre SaaS, avec deux liens d’aperçu sous le bouton **Démarrer**. Les étapes de validation que vous verrez sur cette page varient en fonction du type d’offre et des sélections que vous avez faites lors de la création de l’offre.

![Illustre la page Vue d’ensemble de l’offre pour une offre dans l’Espace partenaires. Le bouton Go Live et les liens d’aperçu s’affichent.](media/publish-status-publisher-signoff.png)

Pour afficher un aperçu de votre offre, procédez comme suit.

1. Sur la page **Vue d’ensemble de l’offre**, sélectionnez un lien d’aperçu sous le bouton **Démarrer**. 

1. Pour valider le flux d’achat et d’installation de bout en bout, achetez votre offre en préversion. Commencez par informer Microsoft avec un [ticket de support](https://aka.ms/marketplacesupport) pour vous assurer que nous ne vous imputions pas de frais.

1. Si votre offre SaaS prend en charge la [facturation contrôlée à l’aide du service de contrôle de la place de marché commerciale](./partner-center-portal/saas-metered-billing.md), examinez et suivez les meilleures pratiques de test détaillées dans les [API de facturation contrôlée de la place de marché](./partner-center-portal/marketplace-metering-service-apis.md#development-and-testing-best-practices).

1. Passez en revue et suivez les instructions de test dans [API de traitement SaaS version 2 dans la place de marché commerciale de Microsoft](./partner-center-portal/pc-saas-fulfillment-api-v2.md#development-and-testing) pour vous assurer que votre offre est correctement intégrée avec les API avant de la publier en ligne.

1. Si vous devez apporter des changements à l’offre après en avoir affiché l’aperçu et l’avoir testée, vous pouvez la modifier et soumettre à la publication une nouvelle préversion. Pour plus d’informations, consultez [Mettre à jour une offre existante dans la Place de marché commerciale](./partner-center-portal/update-existing-offer.md).

## <a name="publish-your-offer-live"></a>Publier votre offre en ligne

Après avoir complètement testé votre préversion, sélectionnez **Démarrer** pour publier votre offre sur le place de marché commerciale.

   > [!TIP]
   > Si votre offre est déjà en ligne dans la place de marché commerciale, les mises à jour que vous effectuez ne sont pas mises en ligne tant que vous ne sélectionnez pas **Démarrer**.

Maintenant que vous avez choisi de rendre votre offre disponible dans la place de marché commerciale, nous effectuons une série de contrôles de validation finaux pour nous assurer que l’offre en ligne est configurée exactement comme sa préversion. Pour plus d’informations sur ces contrôles de validation, consultez [Phase de publication](review-publish-offer.md#publish-phase).

Une fois ces vérifications terminées, votre offre est publiée sur la place de marché.

## <a name="next-step"></a>Étape suivante

[Accéder aux rapports analytiques de la Place de marché commerciale dans l’Espace partenaires](./partner-center-portal/analytics.md)