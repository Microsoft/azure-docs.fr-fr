---
title: Azure Automanage pour machines virtuelles
description: Découvrez Azure Automanage pour machines virtuelles.
author: deanwe
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: deanwe
ms.custom: references_regions
ms.openlocfilehash: ffbf3fa3e2d404b7bc5888ef01707cbd816600b3
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182850"
---
# <a name="azure-automanage-for-virtual-machines"></a>Azure Automanage pour machines virtuelles

Cet article contient des informations sur le service Azure Automanage pour machines virtuelles, qui offre les avantages suivants :

- intègre intelligemment des machines virtuelles afin de sélectionner les meilleures pratiques pour les services Azure ;
- configure automatiquement chaque service conformément aux meilleures pratiques Azure ;
- surveille la dérive et la corrige en cas de détection ;
- offre une interface simple (pointer, cliquer, définir, oublier).


## <a name="overview"></a>Vue d’ensemble

Azure Automanage pour machines virtuelles est un service qui élimine la nécessité de découvrir, de savoir comment intégrer et de configurer certains services Azure susceptibles de tirer parti de votre machine virtuelle. Ces services contribuent à améliorer la fiabilité, la sécurité et la gestion des machines virtuelles, et sont considérés comme des services conformes aux meilleures pratiques Azure, tels [Azure Update Management](../automation/update-management/overview.md) et [Sauvegarde Azure](../backup/backup-overview.md).

Une fois vos machines virtuelles intégrées au service Azure Automanage, celui-ci configure automatiquement les paramètres recommandés pour chaque service conformément aux meilleures pratiques. Les meilleures pratiques sont distinctes pour chaque service. Par exemple, dans Sauvegarde Azure, la meilleure pratique peut être de sauvegarder la machine virtuelle quotidiennement et de respecter une période de rétention de six mois.

Le service Azure Automanage surveille également automatiquement la dérive et la corrige en cas de détection. Cela signifie que, si votre machine virtuelle est intégrée au service Azure Automanage, nous allons non seulement la configurer conformément aux meilleures pratiques Azure, mais aussi la surveiller en veillant à ce qu’elle reste conforme à ces meilleures pratiques tout au long de son cycle de vie. Si votre machine virtuelle dérive ou dévie de ces pratiques, nous la corrigeons et la remettons dans l’état souhaité.

Enfin, l’interface est incroyablement simple.


## <a name="prerequisites"></a>Prérequis

Il existe plusieurs conditions préalables à prendre en compte avant d’essayer d’activer le service Azure Automanage sur vos machines virtuelles.

- Les machines virtuelles concernées doivent être exclusivement Windows Server.
- Les machines virtuelles doivent être en cours d’exécution.
- Les machines virtuelles doivent se trouver dans une région prise en charge.
- L’utilisateur doit disposer des autorisations appropriées (voir paragraphe ci-dessous).
- Automanage ne prend pas en charge les abonnements sandbox à ce stade

Vous devez disposer du rôle **Contributeur** sur le groupe de ressources contenant vos machines virtuelles pour activer Automanage sur ces dernières à l’aide d’un compte Automanage existant. Si vous activez Automanage avec un nouveau compte Automanage, vous devez disposer des autorisations suivantes sur votre abonnement : Rôles **Propriétaire** ou **Contributeur**, ainsi que **Administrateur de l’accès utilisateur**. 

> [!NOTE]
> Si vous souhaitez utiliser Automanage sur une machine virtuelle qui est connectée à un espace de travail dans un autre abonnement, vous devez disposer des autorisations décrites ci-dessus pour chaque abonnement.

Il est également important de noter que le service Automanage ne prend en charge que des machines virtuelles Windows situées dans les régions suivantes : Europe Ouest, USA Est, USA Ouest 2, Canada Centre, USA Centre-Ouest.

## <a name="participating-services"></a>Services participant

:::image type="content" source="media\automanage-virtual-machines\intelligently-onboard-services.png" alt-text="Services intégrés de manière intelligente.":::

Pour obtenir la liste complète des services Azure participant, ainsi que leurs profils de configuration pris en charge, consultez [Meilleures pratiques d’Azure Automanage pour machines virtuelles](virtual-machines-best-practices.md).

 Nous vous intégrerons automatiquement à ces services participant. Ils sont essentiels pour notre livre blanc sur les meilleures pratiques, que vous pouvez trouver dans notre [Cloud Adoption Framework](/azure/cloud-adoption-framework/manage/azure-server-management).

Pour tous ces services, nous les intégrons automatiquement, les configurons, surveillons leur dérive et corrigeons toute dérive détectée.


## <a name="enabling-automanage-for-vms-in-azure-portal"></a>Activation du service Automanage pour machines virtuelles dans le portail Azure

Dans le portail Azure, vous pouvez activer le service Automanage sur une machine virtuelle existante ou lorsque vous créez une machine virtuelle. Pour les étapes concises de ce processus, consultez le [Démarrage rapide du service Automanage pour machines virtuelles](quick-create-virtual-machines-portal.md).

Si c’est la première fois que vous activez le service Automanage pour votre machine virtuelle, vous pouvez rechercher dans le portail Azure **Automanage – Meilleures pratiques pour les machines virtuelles Azure**. Cliquez sur **Activer sur une machine virtuelle existante**, sélectionnez les machines virtuelles que vous souhaitez intégrer, puis cliquez sur **Sélectionner** et **Activer**.

La seule situation dans laquelle vous pourriez être amené à interagir avec cette machine virtuelle pour gérer ces services serait si nous tentions de corriger votre machine virtuelle sans succès. Quand nous parvenons à corriger correctement votre machine virtuelle, nous rétablissons sa conformité sans même vous en informer.


## <a name="configuration-profiles"></a>Profils de configuration

Lorsque vous activez le service Automanage pour votre machine virtuelle, un profil de configuration est requis. Les profils de configuration constituent le fondement de ce service. Ils définissent précisément les services auxquels nous intégrons vos machines et, dans une certaine mesure, la configuration de ces services.

### <a name="default-configuration-profiles"></a>Profils de configuration par défaut

Deux profils de configuration sont actuellement disponibles.

- Le profil de configuration **Meilleures pratiques pour les machines virtuelles Azure – Dev/Test** est conçu pour les machines de développement/test.
- Le profil de configuration **Meilleures pratiques pour les machines virtuelles Azure – Production** est destiné à la production.

La raison de cette différentiation est que certains services sont recommandés en fonction de la charge de travail en cours d’exécution. Par exemple, sur une machine de Production, nous vous intégrons automatiquement au service Sauvegarde Azure. En revanche, pour une machine de Dev/Test, un service de sauvegarde occasionnerait un coût inutile, car les machines de Dev/Test ont généralement une moindre incidence sur l’activité.

### <a name="customizing-a-configuration-profile-using-preferences"></a>Personnalisation d’un profil de configuration à l’aide de préférences

En plus des services standard auxquels nous vous intégrons, nous vous permettons de configurer un sous-ensemble de préférences. Ces préférences sont autorisées au sein d’un éventail d’options de configuration qui ne transgressent pas nos meilleures pratiques. Par exemple, dans le cas du service Sauvegarde Azure, nous vous autorisons à définir la fréquence de la sauvegarde et le jour de la semaine où elle se produit. En revanche, nous ne vous permettons *pas* de désactiver complètement le service Sauvegarde Azure.

> [!NOTE]
> Dans le profil de configuration de Dev/Test, nous ne sauvegardons pas du tout la machine virtuelle.

Vous pouvez ajuster les paramètres d’un profil de configuration par défaut par le biais des préférences. Découvrez comment créer une préférence [ici](virtual-machines-custom-preferences.md).

> [!NOTE]
> Vous ne pouvez pas modifier le profil de configuration sur votre machine virtuelle tant que le service Automanage est activé. Vous devez désactiver le service Automanage pour cette machine virtuelle, puis le réactiver avec le profil de configuration et les préférences souhaités.


## <a name="automanage-account"></a>Compte Automanage

Le Compte Automanage est le contexte de sécurité ou l’identité sous laquelle les opérations automatisées se produisent. En règle générale, il est inutile de sélectionner l’option Compte Automanage, mais s’il existe un scénario de délégation dans lequel vous souhaitez diviser la gestion automatisée (peut-être entre deux administrateurs système), cette option vous permet de définir une identité Azure pour chacun de ces administrateurs.

Dans l’interface du portail Azure, lorsque vous activez le service Automanage sur vos machines virtuelles, le panneau **Activer la meilleure pratique de machine virtuelle Azure** contient une liste déroulante Avancées, qui vous permet d’attribuer ou de créer manuellement le Compte Automanage.

> [!NOTE]
> Vous devez disposer du rôle **Contributeur** sur le groupe de ressources contenant vos machines virtuelles pour activer Automanage sur ces dernières à l’aide d’un compte Automanage existant. Si vous activez Automanage avec un nouveau compte Automanage, vous devez disposer des autorisations suivantes sur votre abonnement : Rôles **Propriétaire** ou **Contributeur**, ainsi que **Administrateur de l’accès utilisateur**.


## <a name="status-of-vms"></a>État des machines virtuelles

Dans le portail Azure, accédez à la page **Automanage – Meilleurs pratiques pour les machines virtuelles Azure**, qui répertorie toutes vos machines virtuelles gérées automatiquement. Vous verrez ici l’état global de chaque machine virtuelle.

:::image type="content" source="media\automanage-virtual-machines\configured-status.png" alt-text="Liste des machines virtuelles configurées.":::

Pour chaque machine virtuelle répertoriée, les informations suivantes s’affichent : Nom, Profil de configuration, Préférence de configuration, État, Compte, Abonnement et Groupe de ressources.

La colonne **État** peut afficher les états suivants :
- *En cours* : la machine virtuelle vient d’être activée et est en cours de configuration.
- *Configurée* : la machine virtuelle est configurée et aucune dérive n’est détectée.
- *Échec*  : la machine virtuelle a dérivé et nous n’avons pas pu la corriger.

Si vous voyez l’**État** *Échec*, vous pouvez résoudre des problèmes de déploiement via le groupe de ressources dans lequel se trouve votre machine virtuelle. Accédez à **Groupes de ressources**, sélectionnez votre groupe de ressources, cliquez sur **Déploiements**, puis consultez l’état *Échec* contenant tous les détails de l’erreur.


## <a name="disabling-automanage-for-vms"></a>Désactivation du service Automanage pour machines virtuelles

Vous pouvez décider un jour de désactiver le service Automanage sur certaines machines virtuelles. Par exemple, si votre machine exécute une charge de travail sécurisée extrêmement sensible, et si vous devez la verrouiller encore plus que ne le ferait Azure naturellement, vous devez configurer la machine en dehors des meilleures pratiques Azure.

Pour ce faire, dans le portail Azure, accédez à la page **Automanage – Meilleures pratiques pour les machines virtuelles Azure** qui répertorie toutes vos machines virtuelles gérées automatiquement. Activez la case à cocher en regard de la machine virtuelle que vous souhaitez désactiver dans le service Automanage, puis cliquez sur le bouton **Désactiver la gestion automatique**.

:::image type="content" source="media\automanage-virtual-machines\disable-step-1.png" alt-text="Désactivation du service Automanage sur une machine virtuelle.":::

Lisez attentivement le message dans la fenêtre contextuelle qui s’affiche avant d’accepter de **Désactiver**.

> [!NOTE]
> La désactivation de l’autogestion dans une machine virtuelle entraîne le comportement suivant :
>
> - La configuration de la machine virtuelle et des services qu’elle contient ne change pas.
> - Tous les frais associés à ces services resteront facturables et seront accumulés.
> - Tous les comportements du service Automanage s’arrêtent immédiatement.


Tout d’abord, nous n’annulons l’intégration de la machine virtuelle à l’un des services auxquels nous l’avons intégrée et sur lesquels nous l’avons configurée. Par conséquent, les frais associés à ces services resteront facturables. Vous devez donc annuler l’intégration si nécessaire. Dans ce cas, tout comportement du service Automanage s’arrête immédiatement. Par exemple, nous cessons surveiller la dérive de la machine virtuelle.


## <a name="next-steps"></a>Étapes suivantes

Cet article vous a expliqué que le service Automanage pour machines virtuelles vous évite d’avoir à connaître, intégrer et configurer les meilleures pratiques pour les services Azure. En outre, si une machine que vous avez intégrée au service Automanage pour machines virtuelles dérive des profils de configuration définis, nous rétablissons automatiquement la conformité.

Essayez d’activer le service Automanage pour machines virtuelles dans le portail Azure.

> [!div class="nextstepaction"]
> [Activer le service Automanage pour machines virtuelles dans le portail Azure](quick-create-virtual-machines-portal.md)