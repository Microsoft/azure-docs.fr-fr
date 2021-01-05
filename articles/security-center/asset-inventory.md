---
title: Inventaire des ressources d’Azure Security Center
description: En savoir plus sur l’expérience de gestion des ressources d’Azure Security Center, qui vous offre une visibilité complète sur toutes vos ressources analysées par Security Center.
author: memildin
manager: rkarlin
services: security-center
ms.author: memildin
ms.date: 09/22/2020
ms.service: security-center
ms.topic: how-to
ms.openlocfilehash: d15d73b0f2b87b8e6f66c7bd4e7fb34f6b06e1a0
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92341921"
---
# <a name="explore-and-manage-your-resources-with-asset-inventory-and-management-tools"></a>Explorez et gérez vos ressources à l’aide des outils d’inventaire et de gestion des ressources

La page d’inventaire des ressources d’Azure Security Center fournit une page unique pour visualiser la posture de sécurité des ressources que vous avez connectées à Security Center. 

Security Center analyse périodiquement l’état de sécurité de vos ressources Azure pour identifier les vulnérabilités de sécurité potentielles. Il fournit ensuite des recommandations sur la façon de corriger ces vulnérabilités.

Lorsqu’une ressource contient des recommandations en suspens, celles-ci apparaissent dans l’inventaire.

Utilisez cette vue et ses filtres pour répondre à des questions telles que les suivantes :

- Parmi mes abonnements avec Azure Defender activé, lesquels ont des recommandations en suspens ?
- Parmi mes machines ayant la balise « Production », lesquelles n’ont pas l’agent Log Analytics ?
- Combien de mes machines, marquées d’une balise spécifique, ont des recommandations en suspens ?
- Combien de ressources dans un groupe de ressources spécifique ont des résultats de sécurité provenant d’un service d’évaluation des vulnérabilités ?

Les possibilités de gestion des ressources de cet outil sont considérables et continuent à se développer. 

> [!TIP]
> Les recommandations de sécurité sur la page d’inventaire des ressources sont les mêmes que celles de la page **Recommandations**, mais ici, elles sont affichées selon la ressource affectée. Pour plus d’informations sur l’application de recommandations, consultez l’article [Implémentation des recommandations de sécurité dans Azure Security Center](security-center-recommendations.md).


## <a name="availability"></a>Disponibilité

|Aspect|Détails|
|----|:----|
|État de sortie :|Disponibilité générale (GA)|
|Prix :|Gratuit|
|Rôles et autorisations obligatoires :|tous les utilisateurs|
|Clouds :|![Oui](./media/icons/yes-icon.png) Clouds commerciaux<br>![Non](./media/icons/no-icon.png) National/souverain (US Gov, Chine Gov, autres Gov)|
|||


## <a name="what-are-the-key-features-of-asset-inventory"></a>Quelles sont les principales fonctionnalités de l’inventaire des ressources ?

La page d’inventaire fournit les outils suivants :

- **Résumés** : Avant de définir des filtres, une bande de valeurs bien visible en haut de la vue de l’inventaire affiche les éléments suivants :

    - **Nombre total de ressources** : Nombre total de ressources connectées à Security Center.
    - **Ressources non saines** : Ressources ayant des recommandations de sécurité actives. [En savoir plus sur les recommandations de sécurité.](security-center-recommendations.md)
    - **Ressources non analysées** : Ressources ayant des problèmes d’analyse de l’agent (l’agent Log Analytics est déployé, mais il n’envoie pas de données ou a d’autres problèmes d’intégrité).

- **Filtres** : Les différents filtres en haut de la page permettent d’affiner rapidement la liste des ressources en fonction de la question à laquelle vous tentez de répondre. Par exemple, si vous souhaitez répondre à la question *Parmi mes machines ayant la balise « Production », lesquelles n’ont pas l’agent Log Analytics ?* , vous pouvez combiner les filtres **Analyse de l’agent** et **Balises**, comme le montre le clip suivant :

    :::image type="content" source="./media/asset-inventory/filtering-to-prod-unmonitored.gif" alt-text="Filtrage des ressources de production qui ne sont pas analysées":::

    Dès que vous avez appliqué des filtres, les valeurs récapitulatives sont mises à jour pour se rapporter aux résultats de la requête. 

- **Options d’exportation** : L’inventaire offre la possibilité d’exporter les résultats des options de filtrage que vous avez sélectionnées dans un fichier CSV. En outre, vous pouvez exporter la requête elle-même vers Azure Resource Graph Explorer pour affiner, enregistrer ou modifier la requête KQL.

    ![Options d’exportation de l’inventaire](./media/asset-inventory/inventory-export-options.png)

    > [!TIP]
    > La documentation KQL fournit une base de données avec quelques exemples de données ainsi que quelques requêtes simples pour vous faire une idée du langage. [Pour en savoir plus, consultez ce didacticiel KQL](/azure/data-explorer/kusto/query/tutorial?pivots=azuredataexplorer).

- **Options de gestion des ressources** : L’inventaire vous permet d’effectuer des requêtes de détection complexes. Une fois que vous avez trouvé les ressources qui correspondent à vos requêtes, l’inventaire fournit des raccourcis pour les opérations telles que :

    - Attribuer des balises aux ressources filtrées : activez les cases à cocher à côté des ressources que vous souhaitez baliser.
    - Intégrer de nouveaux serveurs à Security Center : utilisez le bouton de barre d’outils **Ajouter des serveurs non-Azure**.
    - Automatiser les charges de travail avec Azure Logic Apps : utilisez le bouton **Déclencher l’application logique** pour exécuter une application logique sur une ou plusieurs ressources. Vos applications logiques doivent être préparées à l’avance et accepter le type de déclencheur approprié (requête HTTP). [En savoir plus sur Logic Apps](../logic-apps/logic-apps-overview.md).


## <a name="how-does-asset-inventory-work"></a>Fonctionnement de l’inventaire des ressources

L’inventaire des ressources utilise [Azure Resource Graph (ARG)](../governance/resource-graph/index.yml), un service Azure qui permet d’interroger les données relatives à la posture de sécurité de Security Center sur plusieurs abonnements.

ARG est conçu pour permettre une exploration efficace des ressources avec la possibilité d’interroger à grande échelle.

Grâce au [langage de requête Kusto (KQL)](/azure/data-explorer/kusto/query/), l’inventaire des ressources peut rapidement produire des informations détaillées en croisant les données ASC avec les propriétés d’autres ressources.


## <a name="how-to-use-asset-inventory"></a>Utilisation de l’inventaire des ressources

1. Dans la barre latérale de Security Center, sélectionnez **Inventaire**.

1. Utilisez l’option **Filtrer par nom** pour afficher une ressource spécifique, ou utilisez les filtres comme décrit ci-dessous.

1. Sélectionnez les options pertinentes dans les filtres pour créer la requête spécifique que vous souhaitez effectuer.

    :::image type="content" source="./media/asset-inventory/inventory-filters.png" alt-text="Filtrage des ressources de production qui ne sont pas analysées" lightbox="./media/asset-inventory/inventory-filters.png":::

    Par défaut, les ressources sont triées selon le nombre de recommandations de sécurité actives.

    > [!IMPORTANT]
    > Les options de chaque filtre sont spécifiques aux ressources des abonnements actuellement sélectionnés **et** à vos sélections dans les autres filtres.
    >
    > Par exemple, si vous avez sélectionné un seul abonnement et que celui-ci ne comporte aucune ressource avec des recommandations de sécurité à corriger (0 ressource non saine), le filtre **Recommandations** n’aura aucune option. 

1. Pour utiliser le filtre **Les résultats de sécurité contiennent**, entrez en texte libre l’ID, la vérification de sécurité ou le nom CVE du résultat d’une vulnérabilité pour filtrer les ressources concernées :

    ![Filtre « Les résultats de sécurité contiennent »](./media/asset-inventory/security-findings-contain-elements.png)

    > [!TIP]
    > Les filtres **Les résultats de sécurité contiennent** et **Balises** n’acceptent qu’une seule valeur. Pour filtrer sur plusieurs valeurs, utilisez **Ajouter des filtres**.

1. Pour utiliser le filtre **Azure Defender**, sélectionnez une ou plusieurs options (Désactivé, Activé ou Partiel) :

    - **Désactivé** : ressources non protégées par un plan Azure Defender. Vous pouvez cliquer avec le bouton droit sur l’un de ces éléments et les mettre à niveau :

        :::image type="content" source="./media/asset-inventory/upgrade-resource-inventory.png" alt-text="Filtrage des ressources de production qui ne sont pas analysées" lightbox="./media/asset-inventory/upgrade-resource-inventory.png":::

    - **Activé** : ressources protégées par un plan Azure Defender
    - **Partiel** : s’applique aux **abonnements** pour lesquels certains plans Azure Defender ont été désactivés (mais pas en totalité). Par exemple, l’abonnement suivant comporte cinq plans Azure Defender désactivés. 

        :::image type="content" source="./media/asset-inventory/pricing-tier-partial.png" alt-text="Filtrage des ressources de production qui ne sont pas analysées":::

1. Pour examiner plus en détail les résultats de votre requête, sélectionnez les ressources qui vous intéressent.

1. Pour afficher les options de filtre actuellement sélectionnées en tant que requête dans l’Explorateur Resource Graph, sélectionnez **Afficher dans l’Explorateur Resource Graph**.

    ![Requête d’inventaire dans ARG](./media/asset-inventory/inventory-query-in-resource-graph-explorer.png)

1. Pour exécuter une application logique précédemment définie avec 

1. Si vous avez défini des filtres et que vous laissez la page ouverte, Security Center ne mettra pas les résultats à jour automatiquement. Les modifications apportées aux ressources n’ont pas d’impact sur les résultats affichés, sauf si vous rechargez manuellement la page ou si vous sélectionnez **Actualiser**.


## <a name="faq---inventory"></a>FAQ – Inventaire

### <a name="why-arent-all-of-my-subscriptions-machines-storage-accounts-etc-shown"></a>Pourquoi tous mes abonnements, ordinateurs, comptes de stockage, etc. ne sont-ils pas affichés ?

La vue de l’inventaire répertorie vos ressources connectées Security Center du point de vue de la gestion de la posture de sécurité cloud (CSPM). Les filtres ne renvoient pas toutes les ressources de votre environnement, mais seulement celles qui ont des recommandations en suspens (ou « actives »). 

Par exemple, la capture d’écran suivante montre un utilisateur ayant accès à 38 abonnements, mais seulement 10 de ces abonnements ont des recommandations. Ainsi, si l’on applique le filtre **Type de ressource = Abonnements**, seuls ces 10 abonnements avec des recommandations actives apparaissent dans l’inventaire :

:::image type="content" source="./media/asset-inventory/filtered-subscriptions-some.png" alt-text="Filtrage des ressources de production qui ne sont pas analysées":::

### <a name="why-do-some-of-my-resources-show-blank-values-in-the-azure-defender-or-agent-monitoring-columns"></a>Pourquoi certaines de mes ressources affichent-elles des valeurs vides dans les colonnes d’Azure Defender ou d’analyse de l’agent ?

Toutes les ressources analysées par Security Center n’ont pas d’agents. Par exemple, les comptes Stockage Azure ou les ressources PaaS, telles que les disques, les applications logiques, l’analyse du lac de données et les Event Hubs.

Lorsque la tarification ou l’analyse de l’agent ne sont pas pertinentes pour une ressource, rien n’est indiqué dans ces colonnes d’inventaire.

:::image type="content" source="./media/asset-inventory/agent-pricing-blanks.png" alt-text="Filtrage des ressources de production qui ne sont pas analysées":::

## <a name="next-steps"></a>Étapes suivantes

Cet article a décrit la page d’inventaire des ressources d’Azure Security Center.

Pour plus d’informations sur les outils connexes, consultez les pages suivantes :

- [Azure Resource Graph (ARG)](../governance/resource-graph/index.yml)
- [Langage de requête Kusto (KQL)](/azure/data-explorer/kusto/query/)