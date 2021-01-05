---
title: Optimiser les charges de travail Azure à l’aide du score Advisor
description: Utilisez le score Advisor pour tirer le meilleur parti d’Azure.
ms.topic: article
ms.date: 09/09/2020
ms.openlocfilehash: c4c401a8b532c247806c5fe30728a1f4afd334a4
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92074020"
---
# <a name="optimize-azure-workloads-using-advisor-score"></a>Optimiser les charges de travail Azure à l’aide du score Advisor

## <a name="introduction-to-advisor-score"></a>Présentation du score Advisor

Azure Advisor fournit des recommandations sur les bonnes pratiques en matière de charges de travail. Ces recommandations sont personnalisées et actionnables pour vous aider à :
* Améliorer la posture de vos charges de travail et optimiser vos déploiements Azure.
* Prévenir de manière proactive les principaux problèmes en respectant les bonnes pratiques.
* Évaluer vos charges de travail Azure par rapport aux cinq piliers de [Microsoft Azure Well-Architected Framework](/azure/architecture/framework/).

En tant que fonctionnalité de base d’Advisor, le **score Advisor** est conçu pour vous aider à atteindre efficacement ces objectifs. 

Pour tirer le meilleur parti d’Azure, il est essentiel de comprendre où vous vous trouvez dans votre parcours d’optimisation des charges de travail, et d’identifier les services/ressources qui sont bien utilisés et ceux qui ne le sont pas. En outre, vous souhaiterez savoir comment hiérarchiser vos actions, en fonction des recommandations, afin d’optimiser les résultats. Il est également important de suivre et de rapporter la progression durant ce parcours d’optimisation. Avec le **score Advisor**, vous pouvez facilement effectuer toutes ces opérations avec notre nouvelle expérience de gamification. En tant que consultant cloud personnalisé, Azure Advisor évalue continuellement vos données de télémétrie d’utilisation et la configuration des ressources afin de vérifier que les bonnes pratiques du secteur d’activité sont respectées. Advisor combine ensuite ses résultats pour obtenir un score, ce qui vous permet de savoir, en un clin d’œil, si vous prenez les mesures nécessaires pour créer des solutions fiables, sécurisées et rentables. Le score Advisor est constitué d’un score global, qui peut être divisé en cinq scores de catégorie, un pour chaque catégorie d’Azure Advisor qui représente les cinq piliers de Well-Architected Framework. Vous pouvez suivre la progression dans le temps en affichant votre score global et votre score de catégorie avec une tendance quotidienne, hebdomadaire et mensuelle, et vous pouvez définir des points de référence pour vous aider à atteindre vos objectifs. 

 ![Expérience de score Advisor](./media/advisor-score-1.png)

## <a name="how-to-consume-advisor-score"></a>Comment consommer le score Advisor ?
Advisor affiche le score Advisor global et la répartition pour les catégories Advisor en pourcentages. Un score de 100 % dans une catégorie signifie que toutes vos ressources évaluées par Advisor respectent les bonnes pratiques recommandées par Advisor. À l’autre extrémité du spectre, un score de 0 % signifie qu’aucune de vos ressources évaluée par Advisor ne respecte les recommandations d’Advisor. Vous bénéficiez ainsi des scores suivants :
* Le **score Advisor** vous aide à définir une base de référence des performances de vos charges de travail/abonnements. Vous pouvez également voir les tendances historiques afin de mieux comprendre la tendance.
* Les **scores par catégorie Advisor** des différentes recommandations vous indiquent les recommandations en suspens qui amélioreront le plus votre score. Ces valeurs reflètent à la fois le poids de la recommandation et notre facilité d’implémentation prédite pour garantir que vous tirerez le meilleur parti de votre temps et vous aider à hiérarchiser.
* L’**impact sur le score par catégorie** de chaque recommandation vous aide à hiérarchiser vos actions correctives pour chaque catégorie.

La contribution de chaque recommandation à votre score par catégorie est clairement visible dans la page Score Advisor du portail Azure. Vous pouvez augmenter chaque score par catégorie du pourcentage indiqué dans la colonne d’impact sur le score par catégorie. Cette valeur reflète à la fois le poids de la recommandation au sein de la catégorie et la facilité d’implémentation prévue des éventuelles opportunités à portée de main. Le fait de vous concentrer sur les recommandations présentant l’impact le plus élevé vous aidera à progresser le plus avec le temps.  

![Impact du score Advisor](./media/advisor-score-2.png)

Si l’une des recommandations Advisor n’est pas pertinente pour une ressource, vous pouvez reporter ou ignorer ces recommandations afin de les exclure du calcul du score à compter de l’actualisation suivante. Advisor utilisera également cette entrée comme une suggestion supplémentaire pour améliorer le modèle.

## <a name="how-is-advisor-score-calculated"></a>Comment le score Advisor est-il calculé ?
Advisor affiche vos scores de catégorie et votre score Advisor global sous forme de pourcentages. Un score de 100 % dans une catégorie signifie que toutes vos ressources *évaluées par Advisor* respectent les bonnes pratiques recommandées par Advisor. À l’autre extrémité du spectre, un score de 0 % signifie qu’aucune de vos ressources évaluée par Advisor ne respecte les recommandations d’Advisor. 
**Chacune des cinq catégories a un score potentiel le plus élevé de 100.** Votre score Advisor global est calculé sous la forme d’une somme de chaque score de catégorie applicable, divisée par la somme des scores potentiels les plus élevés de toutes les catégories applicables. Pour la plupart des abonnements, cela signifie qu’Advisor ajoute le score de chaque catégorie et divise le total par 500. Toutefois, **chaque score de catégorie est calculé uniquement si vous utilisez des ressources évaluées par Advisor.**

**Exemple de calcul du score Advisor**
* Score pour un abonnement : Il s’agit de la moyenne simple de tous les scores par catégorie Advisor pour votre abonnement. Si les scores par catégorie Advisor sont : Coût = 73, Fiabilité = 85, Excellence opérationnelle = 77, Performance = 100 ; le **score Advisor** est (73 + 85 + 77 + 100)/(4x100) = 0,84 ou 84 %.
* Score pour plusieurs abonnements : Lorsque plusieurs abonnements sont sélectionnés, le score Advisor global que nous générons correspond au total des scores par catégorie pondérés. Ici, chaque score par catégorie Advisor est cumulé en fonction des ressources consommées par les abonnements. Une fois que nous disposons des scores par catégorie pondérés cumulés, nous en effectuons une moyenne simple pour fournir le score global pour les abonnements. 


### <a name="scoring-methodology"></a>Méthodologie de scoring : 
Le calcul du score Advisor peut être résumé en quatre étapes :
1. Advisor calcule le **coût à l’achat des ressources affectées**, à savoir les ressources de vos abonnements qui ont au moins une recommandation dans Advisor.
2. Advisor calcule le **coût à l’achat des ressources évaluées**, à savoir les ressources qui sont supervisées par Advisor, qu’elles aient ou non des recommandations. 
3. Pour chaque type de recommandation, Advisor calcule le **taux de ressources saines**, à savoir le coût à l’achat des ressources affectées divisé par le coût à l’achat des ressources évaluées.
4. Advisor applique trois pondérations supplémentaires au taux de ressources saines dans chaque catégorie :
  * Les recommandations ayant un impact plus important sont pondérées plus lourdement que celles ayant un faible impact.
  * Les ressources ayant des recommandations à long terme sont davantage prises en compte dans le calcul du score.
  * Les ressources que vous reportez ou ignorez dans Advisor sont retirées entièrement du calcul de votre score. 
    
Advisor applique ce modèle au niveau de la catégorie Advisor (Sécurité utilise le modèle [Niveau de sécurité](../security-center/secure-score-security-controls.md#introduction-to-secure-score)), ce qui donne un score Advisor pour chaque catégorie, et une moyenne simple produit le score final d’Advisor.


## <a name="advisor-score-faq"></a>FAQ sur le score Advisor
* **Quelle est la fréquence de mise à jour de mon score ?**
Votre score est actualisé au moins une fois par jour. 
* **Pourquoi certaines recommandations ont-elles une valeur « - » vide dans la colonne d’impact sur le score par catégorie ?** Advisor n’inclut pas immédiatement les nouvelles recommandations ou celles avec des changements récents dans le modèle de scoring. Après une brève période d’évaluation, en général quelques semaines, elles sont incluses dans le score. 
* **Pourquoi le score Coût a-t-il un impact plus important pour certaines recommandations même si elles présentent des économies potentielles inférieures ?**
Votre score de coût reflète à la fois vos économies potentielles liées à des ressources sous-exploitées et la facilité d’implémentation prévue de ces recommandations. Par exemple, une pondération supplémentaire est appliquée aux ressources affectées qui sont restées inactives plus longtemps, même si les économies potentielles sont inférieures. 
* **Pourquoi n’ai-je pas de score pour une ou plusieurs catégories ou pour un ou plusieurs abonnements ?**
Advisor génère un score uniquement pour les catégories et les abonnements dont il évalue des ressources.
* **Que se passe-t-il si une recommandation n’est pas pertinente ?**
Si vous ignorez une recommandation d’Advisor, elle est omise du calcul de votre score. Le fait d’ignorer des recommandations aide également Advisor à améliorer la qualité des recommandations.
* **Pourquoi mon score a-t-il changé ?** Votre score peut changer si vous corrigez des ressources affectées en adoptant les bonnes pratiques recommandées par Advisor. Si vous ou une personne disposant d’autorisations sur votre abonnement a modifié ou créé de nouvelles ressources, vous pouvez également observer des fluctuations de votre score, car celui-ci est basé sur le rapport entre le coût des ressources affectées et le coût total de toutes les ressources.
* **Comment Advisor calcule-t-il le coût à l’achat des ressources sur un abonnement ?**
Advisor utilise les tarifs de paiement à l’utilisation publiés dans la page des tarifs sur Azure.com, qui ne reflètent pas les remises applicables, multipliés par la quantité d’utilisation au cours du dernier jour où la ressource a été allouée. Le fait d’omettre les remises dans le calcul du coût de la ressource rend le score Advisor comparable d’un abonnement, locataire ou inscription à un autre où les remises peuvent varier. 
* **Dois-je consulter les recommandations dans Advisor pour obtenir des points pour mon score ?** Non. Votre score indique si vous adoptez les bonnes pratiques recommandées par Advisor, même si vous adoptez ces bonnes pratiques de manière proactive sans jamais consulter les recommandations dans Advisor.
* **La méthodologie de scoring fait-elle la différence entre les charges de travail de production et de développement/test ?**
Non, pas pour l’instant, mais vous pouvez ignorer les recommandations sur certaines ressources si elles sont utilisées à des fins de développement et de test et si la recommandation n’est pas applicable.
* **Puis-je comparer les scores entre un abonnement ayant 100 ressources et un abonnement ayant 100 000 ressources ?**
La méthodologie de scoring est conçue pour prendre en considération le nombre de ressources sur un abonnement et la combinaison de services ; ainsi, les abonnements ayant moins de ressources peuvent avoir des scores supérieurs ou inférieurs aux abonnements ayant davantage de ressources. 
* **Que signifie la mention « Bientôt disponible » dans la colonne d’impact sur le score ?**
Elle signifie qu’il s’agit d’une nouvelle recommandation et que nous travaillons encore à son intégration dans notre modèle de score Advisor. Quand cette nouvelle recommandation sera prise en compte dans le calcul du score, vous verrez la valeur d’impact sur le score pour votre recommandation.  
* **Mon score dépend-t-il des sommes d’argent que je consacre à Azure ?**
Non, votre score ne reflète pas nécessairement les sommes que vous consacrez. Au contraire, des dépenses inutiles entraînent une baisse du score de coût.

## <a name="how-to-access-advisor"></a>Comment accéder à Advisor ?
Le score Advisor est en préversion publique dans le portail Azure. Vous devez accéder à la section Advisor. Le score Advisor est le deuxième élément de menu dans le volet de navigation gauche. 

![Point d’entrée du score Advisor](./media/advisor-score-3.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les recommandations d’Advisor, consultez :
* [Présentation du conseiller](advisor-overview.md)
* [Prise en main d’Advisor](advisor-get-started.md)
* [Recommandations d’Advisor en matière de coûts](advisor-cost-recommendations.md)
* [Recommandations d’Advisor en matière de performances](advisor-performance-recommendations.md)
* [Recommandations d’Advisor en matière de sécurité](advisor-security-recommendations.md)
* [Recommandations d’Advisor en matière d’excellence opérationnelle](advisor-operational-excellence-recommendations.md)