---
title: Consortium Azure Blockchain Service
description: Vue d’ensemble de la façon dont le service Azure Blockchain implémente les réseaux blockchain en consortium.
ms.date: 11/21/2019
ms.topic: conceptual
ms.reviewer: zeyadr
ms.openlocfilehash: 7b8885ba08d35db20d1eb7e75141cb173913b386
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84712528"
---
# <a name="azure-blockchain-service-consortium"></a>Consortium Azure Blockchain Service

Avec Azure Blockchain Service, vous pouvez créer des réseaux blockchain en consortium privé où chaque réseau blockchain peut être limité à des participants spécifiques dans le réseau. Seuls les participants dans le réseau blockchain en consortium privé peuvent afficher et interagir avec la blockchain. Les réseaux en consortium dans Azure Blockchain Service peuvent contenir deux types de rôle de participant :

* **Administrateur** : participants privilégiés pouvant effectuer des actions de gestion de consortium et participer aux transactions de blockchain.

* **Utilisateur** : participants ne pouvant pas effectuer d’actions de gestion de consortium mais pouvant participer aux transactions de blockchain.

Les réseaux en consortium peuvent être une combinaison de rôles participants et peuvent avoir un nombre arbitraire de chaque type de rôle. Il doit exister au moins un administrateur.

Le diagramme suivant illustre un réseau en consortiums avec plusieurs participants :

![Diagramme du réseau privé en consortium](./media/consortium/network-diagram.png)

Avec la gestion du consortium dans Azure Blockchain Service, vous pouvez gérer les participants dans le réseau en consortium. La gestion du consortium est basée sur le modèle de consensus du réseau. Dans la préversion actuelle, Azure Blockchain Service fournit un modèle de consensus centralisé pour la gestion du consortium. Tout participant privilégié avec un rôle d’administrateur peut effectuer des actions de gestion du consortium, comme l’ajout ou suppression des participants d’un réseau.

## <a name="roles"></a>Rôles

Les participants à un consortium peuvent être des individus ou des organisations et vous pouvez leur affecter un rôle d’utilisateur ou d’administrateur. Le tableau suivant répertorie les principales différences entre ces deux rôles :

| Action | Rôle utilisateur | Rôle administrateur
|--------|:----:|:------------:|
| Créer un nouveau membre | Oui | Oui |
| Inviter de nouveaux membres | Non | Oui |
| Définir ou modifier le rôle d’un participant membre | Non | Oui |
| Modifier le nom d’affichage d’un membre | Uniquement pour ses propres membres | Uniquement pour ses propres membres |
| Supprimer des membres | Uniquement pour ses propres membres | Oui |
| Participer aux transactions de blockchain | Oui | Oui |

### <a name="user-role"></a>Rôle utilisateur

Les utilisateurs sont des participants du consortium dépourvus des fonctions d’administrateur. Ils ne peuvent pas participer à la gestion des membres liés au consortium. Les utilisateurs peuvent modifier leur nom d’affichage et se retirer d’un consortium.

### <a name="administrator"></a>Administrateur

Un administrateur peut gérer les membres au sein du consortium. Un administrateur peut inviter des membres, les supprimer ou mettre à jour les rôles des membres au sein du consortium.
Il doit toujours y avoir au moins un administrateur au sein d’un consortium. Le dernier administrateur doit attribuer le rôle d’administrateur à un autre participant avant de quitter un consortium.

## <a name="managing-members"></a>Gestion des membres

Seuls les administrateurs peuvent inviter d’autres participants au sein du consortium. Les administrateurs invitent les participants à l’aide de leur ID d’abonnement Azure.

Une fois invités, les participants peuvent rejoindre le consortium de blockchain en déployant un nouveau membre dans Azure Blockchain Service. Pour afficher et rejoindre le consortium invité, vous devez spécifier le même ID d’abonnement Azure utilisé dans l’invitation par l’administrateur réseau.

Les administrateurs peuvent retirer n’importe quel participant du consortium, y compris les autres administrateurs. Les membres peuvent uniquement se retirer eux-même d’un consortium.

## <a name="consortium-management-smart-contract"></a>Contrat intelligent de gestion du consortium

La gestion du consortium dans Azure Blockchain Service s’effectue par le biais de contrats intelligents de gestion du consortium. Lorsque vous déployez un nouveau membre de blockchain, les contrats intelligents sont déployés automatiquement sur vos nœuds.

L’adresse du contrat racine intelligent de gestion du consortium peut être affichée dans le portail Azure. L’**adresse RootContract** est dans la section de vue d’ensemble du membre blockchain.

![Adresse RootContract](./media/consortium/rootcontract-address.png)

Vous pouvez interagir avec le contrat intelligent de gestion du consortium à l’aide du [module PowerShell](manage-consortium-powershell.md) de gestion du consortium, d’Azure portail, ou directement via le contrat intelligent à l’aide du compte Ethereum généré par Azure Blockchain Service.

## <a name="ethereum-account"></a>Compte Ethereum

Lorsqu’un membre est créé, une clé de compte Ethereum l’est également. Azure Blockchain Service utilise la clé pour créer des transactions liées à la gestion du consortium. La clé du compte Ethereum est gérée automatiquement par Azure Blockchain Service.

Le compte de membre peut être affiché dans le portail Azure. Le compte de membre est dans la section de vue d’ensemble du membre blockchain.

![Compte de membre](./media/consortium/member-account.png)

Vous pouvez réinitialiser votre compte Ethereum en cliquant sur votre compte de membre et en entrant un nouveau mot de passe. L’adresse du compte Ethereum et le mot de passe seront réinitialisés.  

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez accéder aux actions de gestion de consortium via PowerShell. Pour plus d’informations, consultez [Gérer les membres d’un consortium dans Azure Blockchain Service à l’aide de PowerShell](manage-consortium-powershell.md).
