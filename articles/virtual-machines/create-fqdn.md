---
title: Créer un nom de domaine complet pour une machine virtuelle sur le Portail Azure
description: Apprenez à créer un nom de domaine complet (FQDN) pour une machine virtuelle dans le portail Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: networking
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 11/03/2020
ms.author: cynthn
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 08d5e20bf0755a71e70a0e446cf96d33bb42ad59
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93351837"
---
# <a name="create-a-fully-qualified-domain-name-in-the-azure-portal-for-a-linux-vm"></a>Créer un nom de domaine complet dans le portail Azure pour une machine virtuelle Linux

Lorsque vous créez une machine virtuelle dans le [portail Azure](https://portal.azure.com), une ressource d’adresse IP publique est créée automatiquement pour la machine virtuelle. Vous utilisez cette adresse IP pour accéder à distance à la machine virtuelle. Bien que le portail ne crée pas de [nom de domaine complet](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) (FQDN), vous pouvez en ajouter un après la création de la machine virtuelle. Cet article explique les étapes pour créer un nom DNS ou un nom de domaine complet. 

## <a name="create-a-fqdn"></a>Créer un nom de domaine complet
Cet article suppose que vous avez déjà créé une machine virtuelle. Si nécessaire, vous pouvez créer une machine virtuelle [Linux](./linux/quick-create-portal.md) ou [Windows](./windows/quick-create-portal.md) dans le portail. Une fois que votre machine virtuelle est en cours d’exécution, procédez comme suit :


1. Sélectionnez votre machine virtuelle dans le portail. Sous **Nom DNS** , sélectionnez **Configurer**.
2. Entrez le nom DNS, puis sélectionnez **Enregistrer** en haut de la page.
3. Pour revenir au panneau Vue d’ensemble de la machine virtuelle, fermez le panneau **Configuration** en sélectionnant le **X** dans l’angle supérieur droit. 
4. Vérifiez que le *Nom DNS* s’affiche correctement.
   



## <a name="next-steps"></a>Étapes suivantes
Maintenant que votre machine virtuelle a un nom DNS et une adresse IP publique, vous pouvez déployer des infrastructures d’applications courantes ou des services, tels que nginx, MongoDB et Docker.

Vous pouvez également lire un autre article sur [l’utilisation de Resource Manager](../azure-resource-manager/management/overview.md) pour obtenir des conseils sur la création de vos déploiements Azure.

