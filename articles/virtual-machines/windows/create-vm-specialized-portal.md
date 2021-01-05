---
title: Créer une machine virtuelle Windows à partir d’un disque dur virtuel spécialisé dans le Portail Azure
description: Créez une machine virtuelle Windows à partir d’un disque dur virtuel dans le portail Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 01/18/2019
ms.author: cynthn
ms.openlocfilehash: 31677482660a48e2bb4c71b81b04681eba725fcd
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96455132"
---
# <a name="create-a-vm-from-a-vhd-by-using-the-azure-portal"></a>Créer une machine virtuelle à partir d’un disque dur virtuel à l’aide du portail Azure

Vous pouvez créer une machine virtuelle dans Azure de plusieurs façons : 

- Si vous souhaitez réutiliser un disque dur virtuel existant ou copier le disque dur virtuel à utiliser à partir d’une machine virtuelle existante, vous pouvez créer une machine virtuelle en *attachant* le disque dur virtuel à la nouvelle machine virtuelle comme disque du système d’exploitation. 

- Vous pouvez créer une machine virtuelle à partir du disque dur virtuel d’une machine virtuelle qui a été supprimée. Par exemple, si vous avez une machine virtuelle Azure qui ne fonctionne pas correctement, vous pouvez la supprimer et utiliser son disque dur virtuel pour créer une autre machine virtuelle. Vous pouvez réutiliser le même disque dur virtuel ou créer une copie du disque dur virtuel en créant une capture instantanée, puis en créant un disque managé à partir de la capture instantanée. Bien que la création d’une capture instantanée nécessite quelques étapes supplémentaires, elle conserve le disque dur virtuel d’origine et offre une solution de secours.

- Prenez une machine virtuelle classique et utilisez le disque dur virtuel pour en créer une nouvelle qui utilise le modèle de déploiement Resource Manager et des disques managés. Pour obtenir de meilleurs résultats, **arrêtez** la machine virtuelle classique dans le portail Azure avant de créer l’instantané.
 
- Vous pouvez créer une machine virtuelle Azure à partir d’un disque dur virtuel local en chargeant le disque dur virtuel local et en l’attachant à une nouvelle machine virtuelle. Utilisez PowerShell ou un autre outil pour charger le disque dur virtuel sur un compte de stockage, puis créez un disque managé à partir du disque dur virtuel. Pour plus d’informations, consultez [Charger un disque dur virtuel spécialisé](create-vm-specialized.md#option-2-upload-a-specialized-vhd). 

> [!IMPORTANT]
> 
> Quand vous utilisez un disque spécialisé pour créer une machine virtuelle, la nouvelle machine conserve le nom d’ordinateur de la machine virtuelle d’origine. D’autres informations spécifiques à l’ordinateur (par exemple le CMID) sont également conservées. Dans certains cas, ces informations en double peuvent provoquer des problèmes. Quand vous copiez une machine virtuelle, vous devez connaître les types d’informations spécifiques à l’ordinateur dont vos applications dépendent.  
> Donc, n’utilisez pas un disque spécialisé si vous voulez créer plusieurs machines virtuelles. Pour les déploiements plus importants, [créez une image](capture-image-resource.md), puis [utilisez-la pour créer plusieurs machines virtuelles](create-vm-generalized-managed.md).

Nous vous recommandons de limiter le nombre de déploiements simultanés à 20 machines virtuelles provenant d’une seule capture instantanée ou d’un seul disque dur virtuel. 

## <a name="copy-a-disk"></a>Copier un disque

Créez une capture instantanée, puis créez un disque à partir de cette capture instantanée. Cette stratégie vous permet de conserver le disque dur virtuel d’origine comme solution de secours :

1. Dans le menu de gauche du [portail Azure](https://portal.azure.com), sélectionnez **Tous les services**.
2. Dans la zone de recherche **Tous les services**, entrez **disques**, puis sélectionnez **Disques** pour afficher la liste des disques disponibles.
3. Sélectionnez le disque à utiliser. La page **Disque** correspondante s’affiche.
4. Dans le menu du haut, sélectionnez **Créer une capture instantanée**. 
5. Entrez un **nom** pour la capture instantanée.
6. Choisissez un **groupe de ressources** pour l’instantané. Vous pouvez utiliser un groupe de ressources existant ou en créer un.
7. Pour **Type de compte**, choisissez le stockage **Standard (HDD)** ou **Premium (SSD)** .
8. Quand vous avez terminé, sélectionnez **Créer** pour créer la capture instantanée.
9. Une fois la capture instantanée créée, sélectionnez **Créer une ressource** dans le menu de gauche.
10. Dans la zone de recherche, entrez **disque managé**, puis sélectionnez **Disques managés** dans la liste.
11. Dans la page **Disques managés**, sélectionnez **Créer**.
12. Donnez un **Nom** au disque.
13. Choisissez un **groupe de ressources** pour le disque. Vous pouvez utiliser un groupe de ressources existant ou en créer un. Cette sélection est également utilisée comme groupe de ressources où vous créez la machine virtuelle à partir du disque.
14. Pour **Type de compte**, choisissez le stockage **Standard (HDD)** ou **Premium (SSD)** .
15. Dans **Type de source**, vérifiez que l’option **Capture instantanée** est sélectionnée.
16. Dans la liste déroulante **Capture instantanée source**, sélectionnez l’instantané que vous souhaitez utiliser.
17. Effectuez tout autre changement nécessaire, puis sélectionnez **Créer** pour créer le disque.

## <a name="create-a-vm-from-a-disk"></a>Créer une machine virtuelle à partir d’un disque

Quand vous avez le disque dur virtuel du disque managé que vous souhaitez utiliser, vous pouvez créer la machine virtuelle dans le portail :

1. Dans le menu de gauche du [portail Azure](https://portal.azure.com), sélectionnez **Tous les services**.
2. Dans la zone de recherche **Tous les services**, entrez **disques**, puis sélectionnez **Disques** pour afficher la liste des disques disponibles.
3. Sélectionnez le disque à utiliser. La page **Disque** correspondante s’ouvre.
4. Dans la page **Vue d’ensemble**, vérifiez que **ÉTAT DU DISQUE** affiche **Non attaché**. Si ce n’est pas le cas, essayez de détacher le disque de la machine virtuelle ou de supprimer la machine virtuelle pour libérer le disque.
4. Dans le menu situé en haut de la page, sélectionnez **Créer une machine virtuelle**.
5. Dans la page **De base** pour la nouvelle machine virtuelle, entrez un **Nom de machine virtuelle**, puis sélectionnez un **groupe de ressources** existant ou créez-en un.
6. En ce qui concerne **Taille**, sélectionnez **Changer la taille** pour accéder à la page **Taille**.
7. Sélectionnez une ligne de taille de machine virtuelle, puis choisissez **Sélectionner**.
8. Dans la page **Mise en réseau**, vous pouvez soit laisser le portail créer automatiquement toutes les ressources, soit sélectionner vous-même un **Réseau virtuel** et un **Groupe de sécurité réseau** existants. Le portail crée toujours une interface réseau et une adresse IP publique pour la nouvelle machine virtuelle. 
9. Dans la page **Gestion**, changez éventuellement les options de supervision.
10. Dans la page **Configuration de l’invité**, ajoutez les extensions nécessaires.
11. Quand vous avez terminé, sélectionnez **Vérifier + créer**. 
12. Si la configuration de la machine virtuelle est validée, sélectionnez **Créer** pour démarrer le déploiement.


## <a name="next-steps"></a>Étapes suivantes

Vous pouvez également utiliser PowerShell pour [charger un disque dur virtuel dans Azure et créer une machine virtuelle spécialisée](create-vm-specialized.md).


