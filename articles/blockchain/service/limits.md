---
title: Limites d’Azure Blockchain Service
description: Vue d’ensemble des limites de service et fonctionnelles dans Azure Blockchain Service
ms.date: 04/02/2020
ms.topic: conceptual
ms.reviewer: ravastra
ms.openlocfilehash: 71e1bebf10fa0142870d03977182472da1ad031f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80676515"
---
# <a name="limits-in-azure-blockchain-service"></a>Limites dans Azure Blockchain Service

Azure Blockchain Service présente des limites de service et fonctionnelles telles que le nombre de nœuds qu’un membre peut avoir, des restrictions de consortium et des quantités de stockage.

## <a name="pricing-tier"></a>Niveau tarifaire

Les limites maximales sur les transactions et les nœuds validateurs varient selon que vous approvisionnez Azure Blockchain Service avec le niveau tarifaire de base ou standard.

| Niveau tarifaire | Nombre maximal de nœuds de transaction | Nombre maximal de nœuds validateurs |
|:---|:---:|:---:|
| De base | 10 | 1 |
| standard | 10 | 2 |

Le réseau de votre consortium doit comporter au moins deux nœuds Azure Blockchain Service de niveau standard. Les nœuds de niveau standard incluent deux nœuds validateurs. Quatre nœuds validateurs sont requis pour satisfaire le [consensus IBFT (Istanbul Byzantine Fault Tolerant)](https://github.com/jpmorganchase/quorum/wiki/Quorum-Consensus).

Utilisez le niveau de base pour le développement, les tests et les preuves de concept. Utilisez le niveau standard pour les déploiements en production. Vous devez également avoirs recours au niveau *Standard* si vous utilisez Blockchain Data Manager ou si vous envoyez un volume élevé de transactions privées.

Le passage du niveau tarifaire de base au niveau standard (ou inversement) après la création du membre n'est pas pris en charge.

## <a name="storage-capacity"></a>Capacité de stockage

La quantité de stockage maximale qui peut être utilisée par nœud pour les journaux et données de registre est 1.8 To.

La diminution de la taille de stockage des registres et journaux n’est pas prise en charge.
## <a name="consortium-limits"></a>Limites de consortium

* **Les noms de membre et de consortium doivent être uniques** et différents des autres noms de membre et de consortium dans Azure Blockchain Service.

* **Les noms de membre et de consortium ne peuvent pas être modifiés**

* **Tous les membres d’un consortium doivent s’inscrire dans le même niveau tarifaire**

* **Tous les membres qui participent à un consortium doivent résider dans la même région**

    Le premier membre créé dans un consortium détermine la région. Les membres invités au consortium doivent résider dans la même région que le premier membre. La limitation de tous les membres à la même région permet de garantir que le consensus du réseau n’est pas affecté.

* **Un consortium doit avoir au moins un administrateur**

    Si un consortium n’a qu’un administrateur, celui-ci ne peut pas se retirer du consortium ou supprimer ses membres tant qu’un autre administrateur n’est pas ajouté ou promu dans le consortium.

* **Les membres supprimés du consortium ne peuvent pas être ajoutés à nouveau**

    Au lieu de cela, ils doivent être réinvités à rejoindre le consortium et créer un nouveau membre. Leurs ressources de membre existantes ne sont pas supprimées afin de préserver les transactions historiques.

* **Tous les membres d’un consortium doivent utiliser la même version de registre**

    Pour plus d’informations sur l’application de correctifs, les mises à jour et les versions de registre disponibles dans Azure Blockchain Service, consultez [Patching, updates, and versions](ledger-versions.md) (Application de correctifs, mises à jour et versions).

## <a name="performance"></a>Performances

N'utilisez pas la fonction de gaz *eth.estimate* pour chaque soumission de transaction. La fonction *eth.estimate* utilise beaucoup de mémoire. Le fait d'appeler la fonction plusieurs fois réduit considérablement le nombre de transactions par seconde.

Si possible, utilisez une valeur de gaz prudente pour la soumission des transactions et n'abusez pas de l'utilisation de *eth.estimate*.

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur les stratégies relatives aux mises à niveau des systèmes et l’application de correctifs à ces derniers  : [Application de correctifs, mises à jour et versions](ledger-versions.md).
