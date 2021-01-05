---
title: 'Questions fréquentes : SRE et codage | Microsoft Docs'
titleSuffix: Azure
description: 'Questions fréquentes : Comprendre la relation entre la SRE et le codage'
author: dnblankedelman
manager: ScottCa
ms.service: site-reliability-engineering
ms.topic: article
ms.date: 09/14/2020
ms.author: dnb
ms.openlocfilehash: b8865fdd53f4947b17a3621a128fc83f3d93d3e2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90089069"
---
# <a name="frequently-asked-questions-do-i-need-to-know-how-to-code-to-get-involved-with-sre"></a>Forum Aux Questions : Dois-je savoir comment coder pour m’impliquer dans la SRE ?

Quand des personnes envisagent de s’impliquer dans la SRE et que les équipes projettent d’introduire des pratiques SRE, une question courante est « Faut-il savoir coder ? »

En bref : oui. 

Mais la réponse complète est un peu plus nuancée. Jetons un coup d’œil à trois situations où le codage intervient dans l’ingénierie de fiabilité des sites, ainsi que le niveau d’expertise requis pour chacune. Cette liste n’est pas complète, mais ces scénarios sont des cas d’usage les plus courants.

## <a name="scenario-1-removing-toil-through-automation"></a>Scénario 1 : Suppression du labeur grâce à l’automatisation

Les ingénieurs en fiabilité de site (SRE) et d’autres personnes qui utilisent des pratiques SRE essaient dans la mesure du possible de supprimer tout labeur. « Labeur » désigne quelque chose spécifique en SRE. Le labeur fait référence au travail opérationnel effectué par un humain ayant certaines caractéristiques. Il n’a pas de valeur à long terme. Cela ne fait pas progresser le service de manière significative. Il est souvent répétitif et en grande partie manuel (même s’il pourrait être automatisé). À mesure que le service ou les systèmes prendront de l’ampleur, le nombre de demandes pour ce système augmentera probablement également en quantité à un rythme proportionnel et nécessitera encore plus de travail manuel.

Par exemple, si un service nécessite que l’équipe SRE réinitialise quelque chose chaque semaine, approvisionne de nouveaux comptes et l’espace disque manuellement ou le redémarre à plusieurs reprises manuellement, il s’agit d’une charge opérationnelle pénible. La réalisation de ces actions n’a pas amélioré le service de manière durable et persistante. Ces actions devront probablement être répétées encore et encore.

Les SRE détestent le labeur. Ils travaillent à l’éliminer chaque fois que cela est possible et approprié. C’est l’une des situations où l’automatisation entre en jeu en SRE. Si ces demandes peuvent être gérées automatiquement, cela permet à l’équipe de travailler sur des choses plus gratifiantes et plus importantes.

*Expertise en codage* : l’automatisation nécessite une certaine expertise en matière de codage, mais elle ne requiert pas nécessairement des compétences complètes en génie logiciel. Si vous pouvez écrire des scripts de petite taille (peut-être dans PowerShell ou l’interpréteur de commandes Bourne), ou même si vous créez une [application logique Azure https://docs.microsoft.com/azure/logic-apps/logic-apps-overview](Azure Logic app) avec très peu de code, cette application peut quand même aider à éliminer le labeur.

## <a name="scenario-2-control-through-apisdomain-specific-languages-dslstemplates"></a>Scénario 2 : Contrôle via des API, des langages spécifiques à un domaine (DSL) ou des modèles

Bien que cela ne soit pas strictement nécessaire pour le travail en SRE, le fait de pouvoir contrôler des environnements par le biais d’API, de DSL et de modèles (en particulier les environnements cloud) permet aux SRE d’intensifier leur travail. L’infrastructure d’approvisionnement et de déprovisionnement, la configuration de la surveillance et l’intégration de plusieurs services deviennent toutes beaucoup plus efficaces grâce au codage.

*Expertise en codage* : comme dans le scénario précédent, cela nécessite une certaine expertise en matière de codage, mais ne requiert pas nécessairement des compétences complètes en génie logiciel. Outre les scripts et les applications logiques mentionnés précédemment, des [modèles Resource Manager https://docs.microsoft.com/azure/azure-resource-manager/templates/overview](Azure Resource Manager Templates) peuvent également être utilisés avec un minimum d’expérience de codage.

## <a name="scenario-3-fixing-the-code"></a>Scénario 3 : Correction du code

Les ingénieurs en fiabilité de site cherchent à améliorer la fiabilité d’un système. Cet objectif nécessite parfois de plonger dans le code source d’un système, de déterminer le problème et souvent de proposer un correctif à la base du code. Bien que le niveau de sophistication de ce travail puisse varier considérablement en fonction de la situation, une expertise en matière de codage est absolument nécessaire dans ces cas.

*Expertise en codage* : une maîtrise complète en matière de génie logiciel est souvent requise dans ce scénario.


## <a name="next-steps"></a>Étapes suivantes

Vous souhaitez en savoir plus sur l’ingénierie de fiabilité des sites ou sur le travail avec peu de code ? Consultez notre [hub d’ingénierie de fiabilité des sites](../index.yml), la documentation du produit liée ci-dessus.
