---
title: Mise à l’échelle automatique d’un service cloud dans le portail | Microsoft Docs
description: Découvrez comment utiliser le portail pour configurer des règles de mise à l’échelle automatique pour le rôle web ou de travail d’un service cloud dans Azure.
services: cloud-services
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 05/18/2017
ms.author: tagore
ms.openlocfilehash: 06a0209c2bbd0982054d33c199685d016f405b0c
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92165483"
---
# <a name="how-to-configure-auto-scaling-for-a-cloud-service-in-the-portal"></a>Configuration de la mise à l’échelle automatique d’un service cloud dans le portail

Des conditions peuvent être définies pour un rôle de travail de service cloud qui déclenchent une opération de scale-in ou de scale-out. Les conditions pour le rôle peuvent être basées sur le processeur, le disque ou la charge réseau du rôle. Vous pouvez également définir une condition basée sur une file d’attente de messages ou sur des mesures d’une autre ressource Azure associée à votre abonnement.

> [!NOTE]
> Cet article porte essentiellement sur les rôles web et de travail d’un service cloud. Lorsque vous créez directement une machine virtuelle (Classic), elle est hébergée dans un service cloud. Vous pouvez mettre à l’échelle une machine virtuelle standard en l’associant à un [groupe à haute disponibilité](/previous-versions/azure/virtual-machines/windows/classic/configure-availability-classic) et en l’activant ou la désactivant manuellement.

## <a name="considerations"></a>Considérations
Vous devez tenir compte des informations suivantes avant de configurer la mise à l'échelle de votre application :

* L'utilisation des cœurs a une incidence sur la mise à l'échelle.

    Le nombre de cœurs utilisés varie en fonction de la taille des instances de rôle. Vous pouvez mettre à l’échelle une application uniquement dans la limite des cœurs de votre abonnement. Par exemple, si la limite de votre abonnement est de 20 cœurs. Si vous exécutez une application avec deux services cloud de taille moyenne (4 cœurs au total), vous pouvez seulement faire monter en charge les autres déploiements de service cloud de votre abonnement des 16 cœurs restants. Pour plus d’informations sur les tailles, consultez [Tailles de services cloud](cloud-services-sizes-specs.md).

* Vous pouvez mettre à l’échelle en fonction d’un seuil de messages de file d’attente. Pour plus d’informations sur l’utilisation des files d’attente, consultez la rubrique sur [l’utilisation du service Queue Storage](../storage/queues/storage-dotnet-how-to-use-queues.md).

* Vous pouvez également mettre à l’échelle d’autres ressources associées à votre abonnement.

* Pour activer la fonction de haute disponibilité de votre application, vous devez vous assurer qu’elle est déployée avec plusieurs instances de rôle. Pour plus d'informations, consultez la page [Contrats de niveau de service](https://azure.microsoft.com/support/legal/sla/).

* La mise à l’échelle automatique se produit uniquement lorsque tous les rôles sont dans un état **Prêt** .  


## <a name="where-scale-is-located"></a>Emplacement de la mise à l’échelle
Une fois votre service cloud sélectionné, le panneau du service cloud doit s’afficher.

1. Dans le panneau du service cloud, sélectionnez le nom du service cloud dans la vignette **Rôles et instances** .   
   **IMPORTANT**  : veillez à cliquer sur le rôle de service cloud, non sur l’instance de rôle qui se trouve sous le rôle.

    ![Capture d’écran de la vignette des Rôles et des instances avec l’option Worker Role With S B Queue 1 indiquée en rouge.](./media/cloud-services-how-to-scale-portal/roles-instances.png)
2. Sélectionnez la vignette **Mise à l’échelle** .

    ![Capture d’écran de la page Opérations avec la vignette Vente indiquée en rouge.](./media/cloud-services-how-to-scale-portal/scale-tile.png)

## <a name="automatic-scale"></a>Mise à l’échelle automatique
Vous pouvez configurer les paramètres de mise à l’échelle d’un rôle avec deux modes : **manuel** ou **automatique** . Le mode Manuel permet définir le nombre absolu d’instances. Le mode Automatique vous permet toutefois de définir des règles qui déterminent le type et le volume de la mise à l’échelle.

Définissez l’option **Mettre à l’échelle selon** sur **Règles de planification et de performance** .

![Capture d’écran montrant l’option de planification et de règles de performance.](./media/cloud-services-how-to-scale-portal/schedule-basics.png)

1. Un profil existant.
2. Ajouter une règle pour le profil parent.
3. Ajoutez un autre profil.

Sélectionnez **Ajouter un profil** . Le profil détermine le mode que vous souhaitez utiliser pour la mise à l’échelle : **toujours** , **périodicité** , **date fixe** .

Après avoir configuré le profil et les règles, sélectionnez l’icône **Enregistrer** en haut.

#### <a name="profile"></a>Profil
Le profil définit un nombre minimum et maximum d’instance pour la mise à l’échelle, et également lorsque cette plage de mise à l’échelle est active.

* **Toujours**

    Toujours conserver cette plage d’instances disponible.  

    ![Service cloud toujours mis à l’échelle](./media/cloud-services-how-to-scale-portal/select-always.png)
* **Périodicité**

    Choisissez un ensemble de jours de la semaine pour la mise à l’échelle.

    ![Mise à l’échelle du service cloud avec une planification périodique](./media/cloud-services-how-to-scale-portal/select-recurrence.png)
* **Date fixe**

    Une plage de dates fixe pour la mise à l’échelle du rôle.

    ![Mise à l’échelle du service cloud avec une date fixe](./media/cloud-services-how-to-scale-portal/select-fixed.png)

Après avoir configuré le profil, sélectionnez le bouton **OK** en bas du panneau du profil.

#### <a name="rule"></a>Règle
Des règles sont ajoutées à un profil et représentent une condition de déclenchement de la mise à l’échelle.

Le déclencheur de la règle est basé sur une mesure du service cloud (utilisation de l’UC, activité du disque ou activité réseau) à laquelle vous pouvez ajouter une valeur conditionnelle. Vous pouvez également définir le déclencheur en fonction d’une file d’attente de messages ou de mesures d’une autre ressource Azure associée à votre abonnement.

![Capture d’écran de la boîte de dialogue Règle avec l’option de nom de la Métrique en rouge.](./media/cloud-services-how-to-scale-portal/rule-settings.png)

Après avoir configuré la règle, sélectionnez le bouton **OK** en bas du panneau de la règle.

## <a name="back-to-manual-scale"></a>Retour à la mise à l’échelle manuelle
Accédez aux [Paramètres de mise à l’échelle](#where-scale-is-located) et définissez l’option **Mettre à l’échelle selon** sur **un nombre d’instances que j’entre manuellement** .

![Paramètres de mise à l’échelle de services cloud avec profil et règle](./media/cloud-services-how-to-scale-portal/manual-basics.png)

Ce paramètre supprime la mise à l’échelle automatique du rôle, et vous pouvez définir le nombre d’instances directement.

1. L’option de mise à l’échelle (manuelle ou automatique).
2. Un curseur de nombre d’instances de rôle pour définir les instances de mise à l’échelle.
3. Nombre d’instances pour la mise à l’échelle du rôle.

Après avoir configuré les paramètres de mise à l’échelle, sélectionnez l’icône **Enregistrer** en haut.