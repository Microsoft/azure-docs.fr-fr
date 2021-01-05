---
title: Inscrire le fournisseur de ressources Azure VMware Solution
description: Procédure d’inscription du fournisseur de ressources Azure VMware Solution.
ms.topic: include
ms.date: 09/21/2020
ms.openlocfilehash: 96d15546c5102a69a0b19f92de33d35d2e9ab6c2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91575739"
---
<!-- Used in avs-deployment.md and tutorial-create-private-cloud.md -->

Pour utiliser Azure VMware Solution, vous devez d’abord inscrire le fournisseur de ressources pour votre abonnement.

```azurecli-interactive
az provider register -n Microsoft.AVS --subscription <your subscription ID>
```

>[!TIP]
>Vous pouvez également utiliser l’interface graphique utilisateur pour inscrire le fournisseur de ressources **Microsoft.AVS**.  Pour plus d’informations, consultez l’article [Inscrire un fournisseur de ressources et des types](../../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider).  
