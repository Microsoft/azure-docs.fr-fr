---
title: Fichier include
description: Fichier include
author: tfitzmac
ms.service: governance
ms.topic: include
ms.date: 03/26/2020
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 204ca121045d1bd53fd507acc7ea9691bfeac12e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88748880"
---
| Ressource | Limite |
| --- | --- |
| Groupes d’administration par locataire Azure AD | 10 000 |
| Abonnements par groupe d’administration | Illimité. |
| Niveaux de hiérarchie des groupes d’administration | Niveau racine plus six niveaux<sup>1</sup> |
| Groupe d’administration parent direct par groupe d’administration | Un |
| [Déploiements au niveau du groupe d’administration](../articles/azure-resource-manager/templates/deploy-to-management-group.md) par emplacement | 800<sup>2</sup> |

<sup>1</sup>Les six niveaux n’incluent pas le niveau d’abonnement.

<sup>2</sup> Si vous atteignez la limite des 800 déploiements, supprimez les déploiements inutiles dans l’historique. Pour supprimer des déploiements au niveau du groupe d’administration, utilisez [Remove-AzManagementGroupDeployment](/powershell/module/az.resources/Remove-AzManagementGroupDeployment) ou [az deployment mg delete](/cli/azure/deployment/mg?view=azure-cli-latest#az-deployment-mg-delete).
