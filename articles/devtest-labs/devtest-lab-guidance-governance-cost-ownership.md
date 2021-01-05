---
title: Gérer les coûts et la propriété dans Azure DevTest Labs
description: Cet article vous aidera à optimiser les coûts et à aligner l’appartenance au sein de votre environnement.
ms.topic: article
ms.date: 06/26/2020
ms.reviewer: christianreddington,anthdela,juselph
ms.openlocfilehash: dfac69a055c9b0c75032622caf7fb8502fad3406
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92328264"
---
# <a name="governance-of-azure-devtest-labs-infrastructure---manage-cost-and-ownership"></a>Gouvernance de l’infrastructure Azure DevTest Labs – Gérer le coût et l’appartenance
Le coût et l’appartenance sont les principaux éléments à prendre en compte lors de la création de vos environnements de développement et de test. Dans cette section, vous trouvez des informations qui vous aideront à optimiser les coûts et à aligner l’appartenance au sein de votre environnement.

## <a name="optimize-for-cost"></a>Optimiser le coût

### <a name="question"></a>Question
Comment puis-je optimiser le coût au sein de mon environnement DevTest Labs ?

### <a name="answer"></a>Réponse
DevTest Labs propose de nombreuses fonctionnalités intégrées qui vous aident à optimiser le coût. Consultez les articles sur [la gestion du coût, les seuils](devtest-lab-configure-cost-management.md) [et les stratégies](devtest-lab-set-lab-policy.md) afin de limiter des activités des utilisateurs. 

Lorsque vous utilisez DevTest Labs pour des charges de travail de développement et de test, vous pouvez envisager d’exploiter l’[avantage de l’abonnement Enterprise Dev/Test](https://azure.microsoft.com/offers/ms-azr-0148p/) dans le cadre de votre contrat Entreprise. Si vous payez à l’utilisation, vous pouvez également considérer l’[offre DevTest de paiement à l’utilisation](https://azure.microsoft.com/offers/ms-azr-0023p/).

Cette approche vous offre de nombreux avantages :

- Tarifs Dev/Test inférieurs préférentiels sur les machines virtuelles, les services cloud, HDInsight, App Service et Logic Apps sous Windows
- Tarifs de contrat Entreprise (EA) avantageux sur d’autres services Azure
- Accès aux images Dev/Test exclusives de la galerie, par exemple Windows 8.1 et Windows 10
 
Seuls les abonnés Visual Studio actifs (abonnements standard, abonnements cloud annuels et mensuels) peuvent utiliser les ressources Azure exécutées dans le cadre d’un abonnement Enterprise Dev/Test. Toutefois, les utilisateurs finaux peuvent également accéder à l’application pour fournir leurs commentaires ou effectuer des tests d’acceptation. L’utilisation des ressources au sein de cet abonnement est limitée au développement et au test d’applications. Aucune garantie n’est offerte concernant le temps d’activité.

Si vous décidez d’utiliser l’offre DevTest, notez que cet avantage est exclusivement réservé au développement et au test de vos applications. L’utilisation dans le cadre de l’abonnement n’est pas associée à un contrat de niveau de service avec compensations financières, sauf au sein d’Azure DevOps et de HockeyApp.

## <a name="define-role-based-access-across-your-organization"></a>Définir un accès en fonction du rôle au sein de votre organisation
### <a name="question"></a>Question
Comment définir le contrôle d’accès en fonction du rôle Azure pour les environnements DevTest Labs afin de garantir la gouvernance du service informatique pendant le travail des développeurs/testeurs ? 

### <a name="answer"></a>Réponse
Il existe un modèle assez large, mais le détail dépend de votre organisation.

Le centre informatique doit posséder uniquement ce qui est nécessaire et donner aux équipes de projet et d’application le niveau de contrôle dont elles ont besoin. En règle générale, cela signifie que le centre informatique possède l’abonnement et gère les fonctions informatiques clés telles que les configurations de mise en réseau. L’ensemble de **propriétaires** pour un abonnement doit être réduit. Ces propriétaires peuvent désigner des propriétaires supplémentaires, si nécessaire, ou appliquer des stratégies de niveau abonnement, par exemple « aucune adresse IP publique ».

Un sous-ensemble d’utilisateurs peut nécessiter un accès à un abonnement, comme le support de niveau 1 ou 2. Dans ce cas, nous vous recommandons d’accorder à ces utilisateurs l’accès **contributeur** afin qu’ils puissent gérer les ressources, mais pas attribuer des accès aux utilisateurs ou ajuster les stratégies.

La ressource DevTest Labs doit appartenir à des propriétaires proches de l’équipe de projet/d’application, car ils comprennent leurs exigences en matière de machines et de logiciels requis. Dans la plupart des organisations, le propriétaire de cette ressource DevTest Labs est souvent le responsable de projet/développement. Ce propriétaire peut gérer les utilisateurs et les stratégies dans l’environnement de laboratoire, ainsi que toutes les machines virtuelles dans l’environnement DevTest Labs.

Les membres de l’équipe de projet/d’application doivent être ajoutés au rôle Utilisateurs de DevTest Labs. Ces utilisateurs peuvent créer des machines virtuelles (alignées sur les stratégies du laboratoire et de l’abonnement). Ils peuvent également gérer leurs propres machines virtuelles. Ils ne peuvent pas gérer les machines virtuelles qui appartiennent à d’autres utilisateurs.

Pour plus d’informations, consultez la documentation [Structure d’entreprise Azure : gouvernance normative de l’abonnement](/azure/architecture/cloud-adoption/appendix/azure-scaffold).


## <a name="next-steps"></a>Étapes suivantes
Voir [Politique et conformité d’entreprise](devtest-lab-guidance-governance-policy-compliance.md).
