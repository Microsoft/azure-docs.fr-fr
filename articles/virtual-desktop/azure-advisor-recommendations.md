---
title: Procédure pas à pas Azure Advisor Windows Virtual Desktop – Azure
description: Résolution des recommandations Azure Advisor pour Windows Virtual Desktop.
author: Heidilohr
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 760e0212d2d863e6b869c23c2e523a0e056a28ed
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90033028"
---
# <a name="how-to-resolve-azure-advisor-recommendations"></a>Résolution des recommandations Azure Advisor

Cet article explique comment vous pouvez résoudre les recommandations qui s’affichent dans Azure Advisor pour Windows Virtual Desktop.

## <a name="no-validation-environment-enabled"></a>« Aucun environnement de validation activé »

>[!div class="mx-imgBorder"]
>![A screenshot of the Azure Advisor Operational Excellence page. The "no validation environment enabled" recommendation is highlighted in red.](media/no-validation-environment.png)

Cette recommandation s’affiche sous Excellence opérationnelle. La recommandation doit également vous présenter un message d’avertissement de ce type :

« Aucun environnement de validation n’est activé dans cet abonnement. Lors de la création de vos pools d’hôtes, vous avez sélectionné **Non** pour "Environnement de validation" sous l’onglet Propriétés. Pour garantir la continuité des activités via les déploiements du service Windows Virtual Desktop, assurez-vous d’avoir au moins un pool d’hôtes avec un environnement de validation dans lequel vous pouvez tester les problèmes potentiels. »

Vous pouvez faire disparaître ce message d’avertissement en activant un environnement de validation dans l’un de vos pools hôtes.

Pour activer un environnement de validation :

1. Accédez à la page d’accueil de Portail Azure et sélectionnez le pool d’hôtes que vous souhaitez modifier.

2. Ensuite, sélectionnez le pool d’hôtes que vous souhaitez passer d’un environnement de production à un environnement de validation.

3. Dans votre pool d’hôtes, sélectionnez **Propriétés** dans la colonne de gauche. Ensuite, faites défiler la page vers le bas jusqu’à voir « Environnement de validation ». Sélectionnez **Oui**, puis **Appliquer**.

>[!div class="mx-imgBorder"]
>![A screenshot of the Properties menu. "Validation environment" is highlighted in red, and the "Yes" bubble is selected.](media/validation-yes.png)

Ces modifications ne feront pas disparaître l’avertissement immédiatement, mais celui-ci devrait finir par disparaître. Azure Advisor effectue une mise à jour deux fois par jour. En attendant, vous pouvez reporter ou ignorer la recommandation manuellement. Nous vous recommandons de laisser la recommandation disparaître d’elle-même. De cette façon, Azure Advisor peut vous faire savoir s’il rencontre des problèmes lors du changement de paramètres.

## <a name="not-enough-production-non-validation-environments-enabled"></a>« Nombre d’environnements de production (non-validation) activés insuffisants »

Cette recommandation s’affiche sous Excellence opérationnelle.

Pour cette recommandation, le message d’avertissement s’affiche pour l’une des raisons suivantes :

- Vous avez trop de pools d’hôtes dans votre environnement de validation.
- Vous n’avez aucun pool d’hôtes de production.

Nous recommandons aux utilisateurs d’avoir moins de la moitié de leurs pools d’hôtes dans un environnement de validation.

Pour résoudre cet avertissement :

1. Accédez à la page d’accueil du portail Azure.

2. Sélectionnez les pools d’hôtes que vous souhaitez passer de la validation à la production.

3. Dans votre pool d’hôtes, sélectionnez l’onglet **Propriétés** dans la colonne à droite de l’écran. Ensuite, faites défiler la page vers le bas jusqu’à voir « Environnement de validation ». Sélectionnez **Non**, puis **Appliquer**.

>[!div class="mx-imgBorder"]
>![A screenshot of the Properties menu. "Validation environment" is highlighted in red, and the "No" bubble is selected.](media/validation-no.png)

Ces modifications ne feront pas disparaître l’avertissement immédiatement, mais celui-ci devrait finir par disparaître. Azure Advisor effectue une mise à jour deux fois par jour. En attendant, vous pouvez reporter ou ignorer la recommandation manuellement. Nous vous recommandons de laisser la recommandation disparaître d’elle-même. De cette façon, Azure Advisor peut vous faire savoir s’il rencontre des problèmes lors du changement de paramètres.

## <a name="not-enough-links-are-unblocked-to-successfully-implement-your-vm"></a>« Trop peu de liens sont débloqués pour implémenter correctement votre machine virtuelle »

Cette recommandation s’affiche sous Excellence opérationnelle.

Vous devez débloquer des URL spécifiques pour vous assurer que votre machine virtuelle fonctionne correctement. Vous pouvez consulter la liste sur le site [Liste des URL sécurisées](safe-url-list.md). Si les URL ne sont pas débloquées, votre machine virtuelle ne fonctionnera pas correctement.

Pour résoudre cette recommandation, veillez à débloquer toutes les URL figurant dans la [liste des URL sécurisées](safe-url-list.md). Vous pouvez également utiliser la balise de service ou les balises de nom de domaine complet pour débloquer des URL.

## <a name="propose-new-recommendations"></a>Proposer de nouvelles recommandations

Vous pouvez nous aider à améliorer Azure Advisor en nous soumettant des idées de recommandations. Votre recommandation pourrait aider un autre utilisateur à se sortir d’une situation difficile. Pour soumettre une suggestion, accédez à [notre forum UserVoice](https://windowsvirtualdesktop.uservoice.com/forums/930847-azure-advisor-recommendations) et remplissez le formulaire de soumission. Lorsque vous remplissez le formulaire, veillez à nous donner le plus de détails possible.

## <a name="next-steps"></a>Étapes suivantes

Si vous recherchez des guides plus détaillés sur la façon de résoudre des problèmes courants, consultez [Vue d’ensemble de la résolution des problèmes, commentaires et support pour Windows Virtual Desktop](troubleshoot-set-up-overview.md).
