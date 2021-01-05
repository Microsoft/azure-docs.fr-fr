---
title: Tutoriel - Gérer les coûts avec Cloudyn dans Azure
description: Dans ce didacticiel, vous allez apprendre à gérer les coûts en utilisant des rapports de récupération des données de facturation, de facturation interne et d’allocation des coûts.
author: bandersmsft
ms.author: banders
ms.date: 03/12/2020
ms.topic: tutorial
ms.service: cost-management-billing
ms.subservice: cloudyn
ms.custom: seodec18
ms.reviewer: benshy
ROBOTS: NOINDEX
ms.openlocfilehash: 52858b93c61de8ee3d6416915c1eba407622a1d3
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 08/20/2020
ms.locfileid: "88683553"
---
# <a name="tutorial-manage-costs-by-using-cloudyn"></a>Tutoriel : Gérer les coûts à l’aide de Cloudyn

Vous gérez les coûts et générez des rapports de récupération des données de facturation dans Cloudyn en allouant les coûts en fonction des étiquettes. Le processus d’allocation des coûts affecte des coûts à vos ressources cloud consommées. Les coûts sont entièrement alloués quand toutes les ressources sont classées avec des balises. Une fois les coûts alloués, vous pouvez fournir à vos utilisateurs des informations de récupération des données de facturation ou de facturation interne sous la forme de rapports et de tableaux de bord. Toutefois, il arrive que de nombreuses ressources ne soient pas étiquetées ou étiquetables quand vous commencez à utiliser Cloudyn.

Par exemple, vous pouvez souhaiter être remboursé de coûts d’ingénierie. Vous devez pouvoir montrer à votre équipe d’ingénierie que vous avez besoin d’un montant spécifique, basé sur les coûts des ressources. Vous pouvez leur montrer un rapport pour toutes les ressources consommées qui portent la balise *ingénierie*.

Dans cet article, les étiquettes et les catégories sont parfois synonymes. Les catégories sont de vastes collections qui peuvent correspondre à beaucoup de choses. Elles peuvent inclure des unités commerciales, des centres de coûts, des services web ou tout élément étiqueté. Les étiquettes sont des paires nom/valeur qui vous permettent de catégoriser les ressources, ainsi que d’afficher et de gérer les informations de facturation consolidées en appliquant la même étiquette à plusieurs ressources et groupes de ressources. Dans les versions antérieures du portail Azure, un *nom d’étiquette* s’appelait une *clé*. Les étiquettes sont créées et stockées par un seul abonnement Azure. Les étiquettes dans AWS se composent de paires clé/valeur. Dans la mesure où Azure et AWS ont tous deux utilisé le terme *clé*, Cloudyn utilise ce terme. Le Gestionnaire de catégories utilise des clés (noms d’étiquettes) pour fusionner les étiquettes.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Utiliser des balises personnalisées pour allouer les coûts.
> * Créer des rapports de récupération des données de facturation et de facturation interne.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [cloudyn-note](../../../includes/cloudyn-note.md)]

## <a name="prerequisites"></a>Prérequis

- Vous devez disposer d’un compte Azure.
- Vous devez disposer d’une inscription à un essai gratuit ou d’un abonnement payant pour Cloudyn.
- Les [comptes non activés doivent être activés](activate-subs-accounts.md) dans le portail Cloudyn.
- La [supervision d’invités](azure-vm-extended-metrics.md) doit être activée sur vos machines virtuelles.


## <a name="use-custom-tags-to-allocate-costs"></a>Utiliser des balises personnalisées pour allouer les coûts

Cloudyn récupère les données d’étiquette de groupe de ressources auprès d’Azure et les propage automatiquement vers les ressources. Dans l’affectation des coûts, vous pouvez voir le coût en fonction des étiquettes de ressources.

À l’aide du modèle d’affectation des coûts, vous définissez des catégories (étiquettes) qui sont appliquées de façon interne aux ressources non catégorisées (non étiquetées) pour regrouper vos coûts et définir des règles permettant de gérer les coûts non étiquetés. Les règles d’affectation des coûts représentent vos instructions sauvegardées dans lesquelles les coûts d’un service sont distribués à un autre service. Ces ressources affichent ensuite les étiquettes/catégories dans des rapports d’*affectation des coûts* via la sélection du modèle que vous avez créé.

N’oubliez pas que les informations sur les étiquettes n’apparaissent pas pour ces ressources dans les rapports d’*analyse des coûts*. De plus, les étiquettes appliquées dans Cloudyn à l’aide de l’affectation des coûts ne sont pas envoyées à Azure. Vous ne les voyez donc pas dans le portail Azure.

Quand vous démarrez l’allocation des coûts, vous devez commencer par définir l’étendue à l’aide d’un modèle de coût. Le modèle de coût ne change pas les coûts, mais les distribue. Quand vous créez un modèle de coût, vous segmentez vos données par entité de coût, compte ou abonnement et par plusieurs balises. Parmi les exemples de balise courants citons un code de facturation, un centre de coûts ou un nom de groupe. En outre, les balises vous aident à effectuer des opérations de récupération des données de facturation ou de facturation interne sur d’autres parties de votre organisation.

Pour créer un modèle d’affectation des coûts personnalisé, sélectionnez **Costs** (Coûts) &gt; **Cost Management** (Gestion des coûts) &gt; **Cost Allocation 360°** (Affectation des coûts 360°) dans le menu du rapport.

![Exemple illustrant un tableau de bord dans lequel vous sélectionnez Cost Allocation 360 (Allocation des coûts 360)](./media/tutorial-manage-costs/cost-allocation-360.png)

Dans la page **Cost Allocation 360** (Allocation des coûts 360), sélectionnez **Add** (Ajouter), puis entrez un nom et une description pour votre modèle de coût. Sélectionnez tous les comptes ou des comptes individuels. Si vous souhaitez utiliser des comptes individuels, vous pouvez sélectionner plusieurs comptes à partir de plusieurs fournisseurs de services cloud. Ensuite, cliquez sur **Categorization** (Catégorisation) pour choisir les balises découvertes qui classent les données de coût. Choisissez les balises (catégories) que vous souhaitez inclure dans votre modèle. Dans l’exemple suivant, la balise **Unit** (Unité) est sélectionnée.

![Exemple illustrant une catégorisation avec modèle de coût](./media/tutorial-manage-costs/cost-model01.png)

L’exemple montre qu’un montant de 19 680 $ n’entre dans aucune des catégories (aucune balise spécifique associée).

Ensuite, sélectionnez **Uncategorized Resources** (Ressources sans catégorie) et sélectionnez les services qui ont des coûts non alloués. Ensuite, définissez des règles pour allouer les coûts.

Par exemple, vous pouvez traiter les coûts de stockage Azure en les répartissant en parts égales entre les machines virtuelles. Pour ce faire, sélectionnez le service **Azure/Storage**, sélectionnez **Proportional to Categorized** (Proportionnel aux ressources classées), puis sélectionnez **Azure/VM** (Azure/MV). Sélectionnez ensuite **Create** (Créer).

![Exemple de règle d’allocation avec modèle de coût pour une distribution égale](./media/tutorial-manage-costs/cost-model02.png)



Dans un autre exemple, vous pouvez allouer tous vos coûts de réseau Azure à une division spécifique dans votre organisation. Pour ce faire, sélectionnez le service **Azure/Network** (Azure/Réseau), puis sous **Define Allocation Rule** (Définir une règle d’affectation), sélectionnez **Explicit Distribution** (Distribution explicite). Ensuite, définissez le pourcentage de distribution sur 100 et sélectionnez la division (**G&amp;A** dans l’image suivante) :

![Exemple de règle d’allocation avec modèle de coût pour une division spécifique](./media/tutorial-manage-costs/cost-model03.png)



Pour toutes les ressources sans catégorie restantes, créez des règles d’allocation supplémentaires.

Si vous avez des instances réservées AWS (Amazon Web Services) non allouées, vous pouvez les affecter à des catégories balisées avec **Reserved Instances** (Instances réservées).

Pour afficher des d’informations sur les choix que vous avez effectués pour allouer les coûts, sélectionnez **Summary** (Récapitulatif). Pour enregistrer vos informations en vue de travailler ultérieurement sur des règles supplémentaires, sélectionnez **Save As Draft** (Enregistrer en tant que brouillon). Ou bien, pour enregistrer vos informations afin que Cloudyn commence à traiter votre modèle d’allocation des coûts, sélectionnez **Save and Activate** (Enregistrer et activer).

La liste des modèles de coût affiche votre nouveau modèle de coût comme étant en cours de traitement (**Processing status**). La mise à jour de la base de données Cloudyn avec votre modèle de coût peut prendre un certain temps. Quand le traitement est terminé, l’état passe à **Completed** (Terminé). Vous pouvez ensuite afficher les données de votre modèle de coût dans le rapport Cost Analysis (Analyse des coûts) sous **Extended Filters** (Filtres étendus) &gt; **Cost Model** (Modèle de coût).

### <a name="category-manager"></a>Gestionnaire de catégories

Category Manager (Gestionnaire de catégories) est un outil de nettoyage des données qui vous permet de fusionner les valeurs de plusieurs catégories (balises) pour en créer d’autres. Avec cet outil simple basé sur les règles, vous pouvez sélectionner une catégorie et créer des règles pour fusionner des valeurs existantes. Par exemple, vous pouvez avoir des catégories existantes pour **R&amp;D** et **dev**, qui représentent toutes deux le groupe de développement.

Dans le portail Cloudyn, cliquez sur le symbole d’engrenage dans le coin supérieur droit et sélectionnez **Category Manager** (Gestionnaire de catégories). Pour créer une catégorie, sélectionnez le signe plus ( **+** ). Entrez un nom pour la catégorie, puis sous **Keys** (Clés), entrez les clés de catégorie que vous souhaitez inclure dans la nouvelle catégorie.

Quand vous définissez une règle, vous pouvez ajouter plusieurs valeurs avec une condition OR. Vous pouvez également effectuer certaines opérations de chaîne de base. Dans les deux cas, cliquez sur le symbole points de suspension ( **…** ) à droite de **Rule** (Règle).

Pour définir une nouvelle règle, dans la zone **Rules** (règles), créez une règle. Par exemple, entrez **dev** sous **Rules** (Règles), puis entrez **R&amp;D** sous **Actions**. Quand vous avez terminé, enregistrez votre nouvelle catégorie.

L’image suivante montre un exemple de règles créées pour une nouvelle catégorie nommée **Work-Load** (Charge de travail) :

![Exemple illustrant la nouvelle catégorie work-load (charge de travail)](./media/tutorial-manage-costs/category01.png)

### <a name="tag-sources-and-reports"></a>Sources et rapports de balise

Les données de balise présentes dans les rapports Cloudyn proviennent de trois emplacements :

- API de ressources de fournisseur de cloud
- API de facturation de fournisseur de cloud
- Balises créées manuellement à partir des sources suivantes :
    - Balises d’entité Cloudyn : métadonnées définies par l’utilisateur appliquées aux entités Cloudyn
    - Category Manager : outil de nettoyage de données qui crée de nouvelles balises basées sur des règles appliquées aux balises existantes

Pour afficher des balises de fournisseur de cloud dans des rapports de coût Cloudyn, vous devez créer un modèle de répartition de coût personnalisé à l’aide de Cost Allocation 360. Pour ce faire, accédez à **Cost** (Coût) > **Cost Management** (Gestion des coûts) > **Cost Allocation 360°** (Affectation des coûts 360°), sélectionnez les étiquettes souhaitées, puis définissez des règles permettant de gérer les coûts non étiquetés. Créez ensuite un nouveau modèle de coût. Par la suite, vous pouvez afficher des rapports dans Cost Allocation Analysis pour consulter, filtrer et trier sur vos balises de ressources Azure.

Les étiquettes de ressources Azure apparaissent uniquement dans les rapports **Costs** (Coûts) > **Cost Allocation Analysis** (Analyse d’affectation des coûts).

Les balises de facturation de fournisseur de cloud apparaissent dans tous les rapports de coût.

Les balises d’entités Cloudyn et les balises que vous créez manuellement apparaissent dans tous les rapports de coût.


## <a name="create-showback-and-chargeback-reports"></a>Créer des rapports de récupération des données de facturation et de facturation interne

La méthode qu’utilisent les organisations pour effectuer des opérations de récupération des données de facturation et de facturation interne varie considérablement. Toutefois, dans les deux cas, vous pouvez vous baser sur les tableaux de bord et rapports mis à votre disposition dans le portail Cloudyn. Vous pouvez fournir l’accès utilisateur à toute personne de votre organisation afin qu’elle puisse afficher les tableaux de bord et les rapports à la demande. Tous les rapports d’analyse des coûts prennent en charge les opérations de récupération des données de facturation, car ils montrent aux utilisateurs les ressources qu’ils ont consommées. En outre, ils permettent aux utilisateurs d’explorer les données de coût ou d’utilisation propres à leur groupe au sein de votre organisation.

Pour afficher les résultats de l’allocation des coûts, ouvrez le rapport d’analyse des coûts et sélectionnez le modèle de coût que vous avez créé. Ensuite, ajoutez un regroupement en fonction d’une ou de plusieurs balises sélectionnées dans le modèle de coût.

![Rapport d’analyse des coûts illustrant un exemple de données issues du nouveau coût](./media/tutorial-manage-costs/cost-analysis.png)

Vous pouvez facilement créer et enregistrer des rapports axés sur des services spécifiques utilisés par des groupes donnés. Par exemple, vous pouvez avoir un département qui utilise beaucoup les machines virtuelles Azure. Vous pouvez créer un rapport qui est filtré sur les machines virtuelles Azure pour afficher la consommation et les coûts.

Si vous avez besoin de fournir des données de capture instantanée à d’autres équipes, vous pouvez exporter un rapport au format PDF ou CSV.


## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Utiliser des balises personnalisées pour allouer les coûts.
> * Créer des rapports de récupération des données de facturation et de facturation interne.



Passez au didacticiel suivant pour en savoir plus sur le contrôle de l’accès aux données.

> [!div class="nextstepaction"]
> [Contrôler l'accès aux données](tutorial-user-access.md)
