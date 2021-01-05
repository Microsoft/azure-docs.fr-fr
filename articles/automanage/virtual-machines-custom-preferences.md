---
title: Créer une préférence personnalisée dans Azure Automanage pour machines virtuelles
description: Découvrez comment ajuster le profil de configuration dans Azure Automanage pour les machines virtuelles, et définir vos propres préférences.
author: ju-shim
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: how-to
ms.date: 09/04/2020
ms.author: jushiman
ms.openlocfilehash: 377677c9e5e81487059241db68baff639a3de033
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91715039"
---
# <a name="create-a-custom-preference-in-azure-automanage-for-vms"></a>Créer une préférence personnalisée dans Azure Automanage pour machines virtuelles

Les meilleures pratiques en lien avec Azure Automanage pour machines virtuelles proposent des profils de configuration par défaut que vous pouvez modifier si nécessaire. Cet article explique comment vous pouvez définir vos propres préférences de profil de configuration lorsque vous activez la gestion automatique sur une machine virtuelle nouvelle ou existante.

Nous prenons actuellement en charge les personnalisations de [Sauvegarde Azure](..\backup\backup-azure-arm-vms-prepare.md#create-a-custom-policy) et de [Microsoft Antimalware](../security/fundamentals/antimalware.md#default-and-custom-antimalware-configuration).


> [!NOTE]
> Vous ne pouvez pas modifier le profil ou la préférence de configuration sur votre machine virtuelle tant que le service Automanage est activé. Vous devez désactiver le service Automanage pour cette machine virtuelle, puis le réactiver avec le profil de configuration et les préférences souhaités.


## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) avant de commencer.

> [!NOTE]
> Les comptes associés à un essai gratuit n’ont pas accès aux machines virtuelles utilisées dans ce tutoriel. Veuillez passer à un abonnement avec paiement à l’utilisation.

> [!IMPORTANT]
> L’autorisation Azure RBAC suivante est nécessaire pour activer Automanage : Rôle de **Propriétaire**, ou rôle de **Contributeur** associé au rôle d’**Administrateur de l’accès utilisateur**.


## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com/).


## <a name="enable-automanage-for-vms-on-an-existing-vm"></a>Activer Automanage pour machines virtuelles sur une machine virtuelle existante

1. Dans la barre de recherche, recherchez puis sélectionnez **Automanage – Meilleures pratiques pour les machines virtuelles**.

2. Sélectionnez l’option **Enable on existing VM** (Activer sur une machine virtuelle existante).

3. Dans le panneau **Sélectionner des machines** :
    1. Filtrez la liste des machines virtuelles sur votre **abonnement** et votre **groupe de ressources**.
    1. Activez la case à cocher de chaque machine virtuelle à intégrer.
    1. Cliquez sur le bouton **Sélectionner**.

    :::image type="content" source="media\virtual-machine-custom-preferences\existing-vm-select-machine.png" alt-text="Sélectionnez une machine virtuelle existante dans la liste des machines virtuelles disponibles.":::

4. Sous **Profil de configuration**, cliquez sur **Browse and change profiles and preferences** (Parcourir et modifier les profils et préférences).

    :::image type="content" source="media\virtual-machine-custom-preferences\existing-vm-quick-create.png" alt-text="Sélectionnez une machine virtuelle existante dans la liste des machines virtuelles disponibles.":::

5. Dans le panneau **Sélectionnez le profil et les préférences de configuration**, sélectionnez un profil à gauche : *Dev/Test* à des fins de test, *Prod* à des fins de production.

    :::image type="content" source="media\virtual-machine-custom-preferences\browse-production-profile.png" alt-text="Sélectionnez une machine virtuelle existante dans la liste des machines virtuelles disponibles.":::

6. Dans le profil choisi, sous **Préférences de configuration**, il existe une liste déroulante dans laquelle vous pouvez ajuster certains services.
    1. Cliquez sur **Créer des préférences**.
    1. Dans le panneau **Créer une préférence de configuration**, sous l’onglet De base, entrez les informations suivantes :
        1. Abonnement
        1. Resource group
        1. Nom de la préférence
        1. Région

    :::image type="content" source="media\virtual-machine-custom-preferences\create-preference.png" alt-text="Sélectionnez une machine virtuelle existante dans la liste des machines virtuelles disponibles.":::

7. Accédez à l’onglet Préférences, puis ajustez les préférences de configuration comme vous le souhaitez.
        
    > [!NOTE]
    > Lors de la modification des configurations de profil, seuls les ajustements qui s’inscrivent dans les limites supérieure et inférieure de nos meilleures pratiques sont autorisés.

8. Examinez votre profil de configuration.
9. Cliquez sur le bouton **Créer**.

10. Cliquez sur le bouton **Activer**.


## <a name="disable-automanage-for-vms"></a>Désactiver Automanage pour machines virtuelles

Cessez rapidement d’utiliser Azure Automanage pour machines virtuelles en désactivant la gestion automatique.

:::image type="content" source="media\virtual-machine-custom-preferences\disable-step-1.png" alt-text="Sélectionnez une machine virtuelle existante dans la liste des machines virtuelles disponibles.":::

1. Accédez à la page **Automanage – Meilleures pratiques pour les machines virtuelles** qui répertorie toutes vos machines virtuelles gérées automatiquement.
1. Activez la case à cocher en regard des machines virtuelles que vous souhaitez désactiver.
1. Cliquez sur le bouton **Désactiver la gestion automatique**.
1. Lisez attentivement le message dans la fenêtre contextuelle qui s’affiche avant d’accepter de **Désactiver**.


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si, pour essayer Azure Automanage pour machines virtuelles, vous avez créé un groupe de ressources dont vous n’avez plus besoin, vous pouvez le supprimer. La suppression du groupe a également pour effet de supprimer la machine virtuelle, ainsi que toutes les ressources du groupe de ressources.

Azure Automanage crée des groupes de ressources par défaut dans lesquels stocker les ressources. Vérifiez les groupes de ressources qui suivent la convention d’affectation de noms « DefaultResourceGroupRegionName » ou « AzureBackupRGRegionName » afin de nettoyer toutes les ressources.

1. Sélectionnez le **groupe de ressources**.
1. Dans la page du groupe de ressources, sélectionnez **Supprimer**.
1. Lorsque vous y êtes invité, confirmez le nom du groupe de ressources, puis sélectionnez **Supprimer**.


## <a name="next-steps"></a>Étapes suivantes 

Pour obtenir les réponses aux questions les plus fréquemment posées, consultez notre FAQ. 

> [!div class="nextstepaction"]
> [Questions fréquentes (FAQ)](faq.md)