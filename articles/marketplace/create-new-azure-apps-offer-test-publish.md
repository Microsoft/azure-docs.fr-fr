---
title: Comment tester et publier une offre Azure Application
description: Utilisez l’Espace Partenaires pour soumettre votre offre Azure Application en préversion, afficher un aperçu de votre offre, la tester, puis la publier sur la Place de marché commerciale Microsoft.
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 11/06/2020
ms.openlocfilehash: 461f9354bb3a6eae0af186de8fe9f39c6b5fff2c
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620917"
---
# <a name="how-to-test-and-publish-an-azure-application-offer"></a>Comment tester et publier une offre Azure Application

Cet article explique comment utiliser l’Espace partenaires afin de soumettre votre offre Azure Application pour publication, afficher un aperçu de votre offre, la tester, puis la mettre en ligne sur la Place de marché commerciale. Vous devez avoir déjà créé une offre que vous souhaitez publier.

## <a name="submit-your-offer-for-publishing"></a>Soumettre votre offre pour publication

1. Connectez-vous au tableau de bord de la place de marché commerciale dans l’[Espace partenaires](https://partner.microsoft.com/dashboard/commercial-marketplace/overview).
1. Sur la page **Vue d’ensemble**, sélectionnez l’offre que vous souhaitez publier.
1. Dans l’angle supérieur droit du portail, sélectionnez **Vérifier et publier**.
1. Assurez-vous que la colonne **État** de chaque page indique **Complète**. Les états possibles sont les suivants :
    - **Non démarré** : la page n’est pas démarrée.
    - **Incomplète** : des informations requises sont manquantes dans la page ou des erreurs doivent être corrigées. Vous devez revenir à la page et la mettre à jour.
    - **Complète** : la page est complète. Toutes les données requises ont été fournies et il n’existe aucune erreur.
1. Si l’un des pages présente un État autre que **Complète**, sélectionnez le nom de la page, corrigez le problème, enregistrez la page, puis sélectionnez **Vérifier et publier** à nouveau pour revenir à cette page.
1. Après avoir compléter toutes les pages, dans la boîte **Notes pour la certification**, fournissez des instructions de test à l’équipe de certification pour vous assurer que votre application est correctement testée. Fournissez toutes les notes supplémentaires utiles pour comprendre votre demande.
1. Pour démarrer le processus de publication de votre offre, sélectionnez **Publier**. La page **Vue d’ensemble de l’offre** s’affiche et indique l’**État de publication** de l’offre.

L’état de publication de votre offre changera à mesure qu’il progressera dans le processus de publication. Pour plus d’informations sur ce processus, consultez [Étapes de validation et de publication](review-publish-offer.md#validation-and-publishing-steps).

## <a name="preview-and-test-your-offer"></a>Afficher un aperçu de votre offre et la tester

Une fois que l’offre est prête à être approuvée, nous vous enverrons un e-mail pour vous demander de consulter et d’approuver la préversion de votre offre. Vous pouvez également actualiser la page **Vue d’ensemble de l’offre** depuis votre navigateur pour voir si votre offre a atteint la phase d’approbation de publication. Si c’est le cas, le bouton **Démarrer** et un lien d’aperçu seront disponibles. Si vous avez choisi de vendre votre offre par le biais de Microsoft, quiconque a été ajouté au public de la préversion peut tester son acquisition et son déploiement pour vérifier qu’elle répond à vos exigences au cours de cette phase.

La capture d’écran suivante montre la page **Vue d’ensemble de l’offre** pour une offre, avec deux liens d’aperçu sous le bouton **Démarrer**. Les étapes de validation que vous verrez sur cette page varient en fonction du type d’offre et des sélections que vous avez faites lors de la création de l’offre.

[![Illustre la page Vue d’ensemble de l’offre pour une offre dans l’Espace partenaires. Le bouton Démarrer et les liens d’aperçu sont affichés.](media/create-new-azure-app-offer/azure-app-publish-status.png)](media/create-new-azure-app-offer/azure-app-publish-status.png#lightbox)

Pour afficher un aperçu de votre offre, procédez comme suit.

1. Sur la page **Vue d’ensemble de l’offre**, sélectionnez un lien d’aperçu sous le bouton **Démarrer**. 

1. Pour valider le flux d’achat et d’installation de bout en bout, achetez votre offre en préversion. Commencez par informer Microsoft avec un [ticket de support](https://aka.ms/marketplacesupport) pour vous assurer que nous ne vous imputions pas de frais.

1. Si votre offre Azure Application prend en charge la [facturation à l’utilisation à l’aide du service de contrôle de la Place de marché commerciale](./partner-center-portal/azure-app-metered-billing.md), examinez et suivez les meilleures pratiques de test détaillées dans les [API de facturation à l’utilisation de la Place de marché](./partner-center-portal/marketplace-metering-service-apis.md#development-and-testing-best-practices).

1. Si vous devez apporter des changements à l’offre après en avoir affiché l’aperçu et l’avoir testée, vous pouvez la modifier et soumettre à la publication une nouvelle préversion. Pour plus d’informations, consultez [Mettre à jour une offre existante dans la Place de marché commerciale](./partner-center-portal/update-existing-offer.md).

## <a name="publish-your-offer-live"></a>Publier votre offre en ligne

Après avoir complètement testé votre préversion, sélectionnez **Démarrer** pour publier votre offre sur le place de marché commerciale.

   > [!TIP]
   > Si votre offre est déjà en ligne dans la place de marché commerciale, les mises à jour que vous effectuez ne sont pas mises en ligne tant que vous ne sélectionnez pas **Démarrer**.

Maintenant que vous avez choisi de rendre votre offre disponible dans la place de marché commerciale, nous effectuons une série de contrôles de validation finaux pour nous assurer que l’offre en ligne est configurée exactement comme sa préversion. Pour plus d’informations sur ces contrôles de validation, consultez [Phase de publication](review-publish-offer.md#publish-phase).

Une fois ces vérifications terminées, votre offre est publiée sur la place de marché.

### <a name="errors-and-review-feedback"></a>Erreurs et commentaires de révision

L’étape **Validation manuelle** dans le processus de publication équivaut à une revue complète de votre offre et des ressources techniques associées à celle-ci (en particulier, le modèle Azure Resource Manager). Les problèmes sont généralement présentés sous la forme de liens de demande de tirage (PR). Pour savoir comment afficher ces demandes de tirage (pull requests) et y répondre, consultez [Gestion des commentaires de révision](partner-center-portal/azure-apps-review-feedback.md).

Si vous rencontrez des erreurs durant une ou plusieurs des étapes de publication, corrigez-les avant de republier votre offre.

## <a name="next-step"></a>Étape suivante

- [Accéder aux rapports analytiques de la Place de marché commerciale dans l’Espace partenaires](partner-center-portal/analytics.md)
- Découvrez [comment vendre votre offre Azure Application](create-new-azure-apps-offer-marketing.md) par le biais des programmes de Co-vente avec Microsoft et de Revente via des fournisseurs de solutions Cloud.
