---
title: Création d’une demande de support Azure
description: Les clients qui ont besoin d’aide peuvent utiliser le portail Azure pour trouver des solutions en libre-service et pour créer et gérer des demandes de support.
ms.assetid: fd6841ea-c1d5-4bb7-86bd-0c708d193b89
ms.topic: how-to
ms.date: 06/25/2020
ms.openlocfilehash: 11ca7925ce1664b5586ab8ec0fb523a2d562ee80
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745585"
---
# <a name="create-an-azure-support-request"></a>Créer une demande de support Azure

Azure vous permet de créer et gérer les demandes de support, également appelées tickets de support. Vous pouvez créer et gérer des demandes dans le [portail Azure](https://portal.azure.com), comme décrit dans cet article. Vous pouvez également créer et gérer des demandes par programme, à l’aide de l’[API REST de ticket de support Azure](/rest/api/support).

> [!NOTE]
> L’URL du portail Azure est spécifique au cloud Azure sur lequel votre organisation est déployée.
>
>* Portail Azure pour une utilisation commerciale : [https://portal.azure.com](https://portal.azure.com)
>* Portail Azure pour l’Allemagne : [https://portal.microsoftazure.de](https://portal.microsoftazure.de)
>* Portail Azure pour le gouvernement des États-Unis : [https://portal.azure.us](https://portal.azure.us)

L’expérience de demande de support repose sur trois objectifs principaux :

* **Optimisation** : nous avons facilité la recherche de support et la résolution des problèmes et simplifié la soumission d’une demande de support.
* **Intégration** : vous pouvez facilement ouvrir une demande de support quand vous travaillez à la résolution d’un problème lié à une ressource Azure, sans changer de contexte.
* **Efficacité** : collectez les informations clés dont votre ingénieur de support a besoin pour résoudre votre problème.

Azure offre une prise en charge illimitée de la gestion des abonnements, ce qui inclut la facturation, le réglage des quotas et les transferts de compte. Pour bénéficier du support, vous devez disposer d’un plan de support. Pour plus d’informations, consultez [Comparer les plans de support](https://azure.microsoft.com/support/plans).

## <a name="getting-started"></a>Prise en main

Vous pouvez accéder à **Aide + support** dans le portail Azure. Cette fonctionnalité est disponible dans le menu du portail Azure, dans l’en-tête global ou dans le menu des ressources d’un service. Avant de pouvoir faire une demande de support, vous devez disposer des autorisations appropriées.

### <a name="azure-role-based-access-control"></a>Contrôle d'accès en fonction du rôle Azure

Pour créer une demande de support, vous devez être [propriétaire](../../role-based-access-control/built-in-roles.md#owner) ou [contributeur](../../role-based-access-control/built-in-roles.md#contributor) ou le rôle [Contributeur de demande de support](../../role-based-access-control/built-in-roles.md#support-request-contributor) doit vous avoir été attribuer au niveau de l’abonnement. Pour créer une demande de support sans abonnement, par exemple dans un scénario Azure Active Directory, vous devez être un [administrateur](../../active-directory/roles/permissions-reference.md).

### <a name="go-to-help--support-from-the-global-header"></a>Accédez à Aide + support dans l’en-tête global

Pour démarrer une demande de support à partir de n’importe où dans le portail Azure :

1. Sélectionnez **?** dans l’en-tête global. Ensuite, sélectionnez **Aide + support**.

   ![Aide et support](./media/how-to-create-azure-support-request/helpandsupportnewlower.png)

1. Sélectionnez **Nouvelle demande de support**. Suivez les invites pour fournir des informations sur votre problème. Nous vous suggérerons des solutions possibles, collecterons des détails sur le problème et vous aiderons à soumettre et à suivre la demande de support.

   ![Nouvelle demande de support](./media/how-to-create-azure-support-request/newsupportrequest2lower.png)

### <a name="go-to-help--support-from-a-resource-menu"></a>Accéder à Aide + support dans le menu d’une ressource

Pour démarrer une demande de support dans le contexte de la ressource en cours d’utilisation :

1. Dans le menu de la ressource, dans la section **Support + dépannage**, sélectionnez **Nouvelle demande de support**.

   ![En contexte](./media/how-to-create-azure-support-request/incontext2lower.png)

1. Suivez les invites pour nous fournir des informations sur le problème que vous rencontrez. Quand vous démarrez le processus de demande de support à partir de la ressource, certaines options sont présélectionnées pour vous.

## <a name="create-a-support-request"></a>Création d’une demande de support

Nous allons vous guider tout au long des étapes à suivre pour recueillir des informations sur votre problème et vous aider à le résoudre. Chaque étape est décrite dans les sections suivantes.

### <a name="basics"></a>Concepts de base

La première étape du processus de demande de support collecte des informations sur votre problème et votre plan de support.

Sous l’onglet **Bases** de **Nouvelle demande de support**, utilisez les sélecteurs pour commencer à nous faire part du problème. Tout d’abord, vous allez identifier des catégories générales pour le type de problème et choisir l’abonnement associé. Sélectionnez le service, par exemple **Machine virtuelle exécutant Windows**. Sélectionnez la ressource, par exemple le nom de votre machine virtuelle. Décrivez le problème avec vos propres mots, puis sélectionnez **Type de problème** et **Sous-type de problème** pour être plus spécifique.

![Panneau Informations de base](./media/how-to-create-azure-support-request/basics2lower.png)

### <a name="solutions"></a>Solutions

Après avoir collecté les informations de base, nous vous présenterons des solutions à essayer vous-même. Dans certains cas, nous pouvons même exécuter un diagnostic rapide. Écrites par des ingénieurs Azure, les solutions résolvent les problèmes les plus courants.

### <a name="details"></a>Détails

Nous recueillons ensuite des détails supplémentaires sur le problème. En fournissant des informations complètes et détaillées à cette étape, vous nous aidez à acheminer votre demande de support vers l’ingénieur approprié.

1. Si possible, indiquez-nous quand le problème a commencé et les étapes permettant de le reproduire. Vous pouvez charger un fichier, tel qu’un fichier journal ou le résultat d’un diagnostic. Pour plus d’informations sur les chargements de fichiers, consultez [Instructions de chargement de fichier](how-to-manage-azure-support-request.md#file-upload-guidelines).

1. Une fois que vous disposez de toutes les informations sur le problème, choisissez comment obtenir de l’aide. Dans la section **Méthode de support** sous **Détails**, sélectionnez la gravité de l’impact. Le niveau de gravité maximale dépend de votre [plan de support](https://azure.microsoft.com/support/plans).

    Par défaut, l’option **Partager les informations de diagnostic** est sélectionnée. Cela permet au support Azure de collecter des [informations de diagnostic](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) à partir de vos ressources Azure. Dans certains cas, il existe une deuxième question qui n’est pas sélectionnée par défaut, par exemple une demande d’accès à la mémoire d’une machine virtuelle.

1. Indiquez votre méthode de contact préférée, l’heure à laquelle nous pouvons vous contacter et la langue de votre support.

1. Ensuite, complétez la section **Informations de contact** afin que nous sachions comment vous contacter.

### <a name="review--create"></a>Vérifier + créer

Renseignez toutes les informations requises sous chaque onglet, puis sélectionnez **Vérifier + créer**. Vérifiez les détails que vous allez envoyer au support. Si nécessaire, vous pouvez revenir à n’importe quel onglet pour apporter une modification. Une fois que la demande de support est complète, sélectionnez **Créer**.

Un ingénieur de support vous contactera en utilisant la méthode que vous avez indiquée. Pour plus d’informations sur le temps de réponse initial, consultez [Étendue du support et réactivité](https://azure.microsoft.com/support/plans/response/).


## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les options de support en libre-service dans Azure, regardez cette vidéo :

> [!VIDEO https://www.youtube.com/embed/gNhzR5FE9DY]

Suivez ces liens pour en savoir plus :

* [Comment gérer une demande de support Azure](how-to-manage-azure-support-request.md)
* [API REST de ticket de support Azure](/rest/api/support)
* [Nous envoyer vos commentaires et suggestions](https://feedback.azure.com/forums/266794-support-feedback)
* Nous contacter sur [Twitter](https://twitter.com/azuresupport)
* Obtenez l’aide de vos pairs sur la [page de questions Microsoft Q&A](/answers/products/azure)
* En découvrir plus dans le [Forum aux questions du support Azure](https://azure.microsoft.com/support/faq)