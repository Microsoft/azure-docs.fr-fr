---
title: Gérer les projets de migration à grande échelle à l’aide d’Azure Migrate
description: Apprenez à utiliser efficacement Azure Migrate sur les ressources client déléguées.
ms.date: 12/4/2020
ms.topic: how-to
ms.openlocfilehash: 16b92f3aa4dc3bfcb71eb232170c4df30348f8db
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97095387"
---
# <a name="manage-migration-projects-at-scale-with-azure-migrate"></a>Gérer les projets de migration à grande échelle à l’aide d’Azure Migrate

En tant que fournisseur de services, vous avez peut-être intégré les locataires de plusieurs clients à [Azure Lighthouse](../overview.md). Azure Lighthouse permet aux fournisseurs de services d’effectuer des opérations à grande échelle sur plusieurs locataires Azure Active Directory (Azure AD) à la fois, améliorant ainsi l’efficacité des tâches de gestion.

[Azure Migrate](../../migrate/migrate-services-overview.md) fournit un hub centralisé pour évaluer et migrer les serveurs, l’infrastructure, les applications et les données locales vers Azure. En règle générale, les partenaires qui effectuent des évaluations et une migration à grande échelle pour plusieurs clients doivent accéder individuellement à chaque abonnement client à l’aide du [modèle d’abonnement Fournisseur de solutions Cloud (CSP)](/partner-center/customers-revoke-admin-privileges) ou en [créant un utilisateur invité dans le locataire client](/azure/active-directory/external-identities/what-is-b2b).

L’intégration d’Azure Lighthouse à Azure Migrate permet aux fournisseurs de services de découvrir, d’évaluer et de migrer des charges de travail pour différents clients à grande échelle, tout en permettant aux clients d’avoir une visibilité et un contrôle complets de leurs environnements. Grâce à la gestion déléguée des ressources Azure, les fournisseurs de services ont une vue unique de tous les projets Azure Migrate qu’ils gèrent sur plusieurs locataires client.

> [!NOTE]
> Par le biais d’Azure Lighthouse, les partenaires peuvent effectuer la détection, l’évaluation et la migration de machines virtuelles VMware locales, de machines virtuelles Hyper-V, de serveurs physiques et d’instances AWS/GCP. Il existe deux options pour une [migration de machine virtuelle VMware](../../migrate/server-migrate-overview.md). À l’heure actuelle, seule la méthode de migration basée sur l’agent peut être utilisée lors de l’utilisation d’un projet de migration dans un abonnement client délégué. La migration à l’aide de la réplication sans agent n’est actuellement pas prise en charge via l’accès délégué à l’étendue du client.

Cette rubrique fournit une vue d’ensemble de l’utilisation d’[Azure Migrate](../../migrate/migrate-services-overview.md) de manière évolutive.

> [!TIP]
> Bien que nous faisons référence aux fournisseurs de services et aux clients dans cette rubrique, ces instructions s’appliquent également aux [entreprises utilisant Azure Lighthouse pour gérer plusieurs locataires](../concepts/enterprise.md).

Selon votre scénario, vous pouvez créer le projet Azure Migrate dans le locataire client ou dans votre locataire gestionnaire. Examinez les considérations ci-dessous et déterminez quel modèle répond le mieux aux besoins de votre client en matière de migration.

## <a name="create-an-azure-migrate-project-in-the-customer-tenant"></a>Créer un projet Azure Migrate dans le locataire client

L’une des possibilités d’utilisation d’Azure Lighthouse consiste à créer le projet Azure Migrate dans le locataire client. Les utilisateurs du locataire gestionnaire peuvent alors sélectionner l’abonnement du client lors de la création d’un projet de migration. À partir du locataire gestionnaire, le fournisseur de services peut effectuer les opérations de migration nécessaires. Cela peut inclure le déploiement de l’appliance Azure Migrate pour découvrir les charges de travail, l’évaluation des charges de travail en regroupant les machines virtuelles et en calculant les coûts liés au cloud, l’examen de l’état de préparation des machines virtuelles et l’exécution de la migration.

Dans ce scénario, aucune ressource n’est créée ni stockée dans le locataire gestionnaire, même si les étapes de découverte et d’évaluation peuvent être lancées et exécutées à partir de ce locataire. Toutes les ressources, telles que les projets de migration, les rapports d’évaluation pour les charges de travail locales et les ressources migrées sur la destination cible, seront déployées dans l’abonnement client. Toutefois, le fournisseur de services peut accéder à tous les projets clients à partir de sa propre expérience de locataire et de portail.

Cette approche réduit le changement de contexte pour les fournisseurs de services qui travaillent avec plusieurs clients, tout en permettant aux clients de conserver toutes leurs ressources dans leurs propres locataires.

Le workflow de ce modèle est similaire à ce qui suit :

1. Le client est [intégré à Azure Lighthouse](onboard-customer.md). Le rôle intégré Contributeur est requis pour l’identité qui sera utilisée avec Azure Migrate. Pour obtenir un exemple d’utilisation de ce rôle, consultez l’exemple de modèle [delegated-resource-management-azmigrate](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegated-resource-management-azmigrate).
1. L’utilisateur désigné se connecte au locataire gestionnaire dans le portail Azure, puis accède à Azure Migrate. Cet utilisateur [crée un projet Azure Migrate](/azure/migrate/create-manage-projects), en sélectionnant l’abonnement client délégué approprié.
1. L’utilisateur [effectue ensuite les étapes de découverte et d’évaluation](../../migrate/tutorial-discover-vmware.md).

   Pour les machines virtuelles VMware, avant de configurer l’appliance, vous pouvez définir l’étendue de la découverte pour limiter la détection à des centres de donnés vCenter Server, des clusters, un dossier de clusters, des hôtes, un dossier d’hôtes ou des machines virtuelles individuelles. Pour définir l’étendue, affectez des autorisations sur le compte utilisé par l’appliance pour accéder à vCenter Server. Cela est utile si les machines virtuelles de plusieurs clients sont hébergées sur l’hyperviseur. Vous ne pouvez pas limiter l’étendue de détection d’Hyper-V.

    > [!NOTE]
    > Vous pouvez découvrir et évaluer les machines virtuelles VMware pour la migration à l’aide d’Azure Migrate par le biais de l’accès délégué fourni par Azure Lighthouse. Pour la migration de machines virtuelles VMware, seule la méthode basée sur un agent est actuellement prise en charge lors de l’utilisation d’un projet de migration dans un abonnement client délégué.

1. Lorsque l’abonnement client cible est prêt, procédez à la migration par le biais de l’accès accordé par Azure Lighthouse. Le projet de migration contenant les résultats de l’évaluation et les ressources migrées sera créé dans le locataire client sous l’abonnement cible.

> [!TIP]
> Avant la migration, une zone d’atterrissage doit être déployée pour approvisionner les ressources d’infrastructure de base et préparer l’abonnement vers lequel les machines virtuelles seront migrées. Pour accéder à des ressources ou en créer dans cette zone d’atterrissage, le rôle intégré Propriétaire peut être nécessaire, ce qui n’est actuellement pas pris en charge dans Azure Lighthouse. Pour de tels scénarios, le client peut devoir fournir un rôle d’accès invité ou déléguer l’accès administrateur par le biais du modèle d’abonnement CSP. Pour découvrir une approche de la création de zones d’atterrissage mutualisées, consultez la [solution de démonstration Multi-tenant-Landing-Zones](https://github.com/Azure/Multi-tenant-Landing-Zones) sur GitHub.

## <a name="create-an-azure-migrate-project-in-the-managing-tenant"></a>Créer un projet Azure Migrate dans le locataire gestionnaire

Dans ce scénario, les opérations liées à la migration, telles que la découverte et l’évaluation, sont toujours effectuées par les utilisateurs dans le locataire gestionnaire. Toutefois, le projet de migration et toutes les ressources pertinentes résident dans le locataire partenaire, et le client n’a pas de visibilité directe sur le projet (bien que les évaluations puissent être partagées avec les clients si nécessaire). La destination de migration de chaque client est l’abonnement du client.

Cette approche permet aux fournisseurs de services de démarrer rapidement des projets de découverte et d’évaluation de la migration, en les extrayant des abonnements et des locataires des clients.

Le workflow de ce modèle est similaire à ce qui suit :

1. Le client est [intégré à Azure Lighthouse](onboard-customer.md). Le rôle intégré Contributeur est requis pour l’identité qui sera utilisée avec Azure Migrate. Pour obtenir un exemple d’utilisation de ce rôle, consultez l’exemple de modèle [delegated-resource-management-azmigrate](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegated-resource-management-azmigrate).
1. L’utilisateur désigné se connecte au locataire gestionnaire dans le portail Azure, puis accède à Azure Migrate. Cet utilisateur [crée un projet Azure Migrate](/azure/migrate/create-manage-projects) dans un abonnement appartenant au locataire gestionnaire.
1. L’utilisateur [effectue ensuite les étapes de découverte et d’évaluation](../../migrate/tutorial-discover-vmware.md). Les machines virtuelles locales sont découvertes et évaluées au sein du projet de migration créé dans le locataire gestionnaire, puis migrées à partir de là.

   Si vous gérez plusieurs clients dans le même hôte Hyper-V, vous pouvez découvrir toutes les charges de travail en même temps. Les machines virtuelles spécifiques au client peuvent être sélectionnées dans le même groupe, une évaluation peut être créée et la migration peut être effectuée en sélectionnant l’abonnement du client approprié comme destination cible. Il n’est pas nécessaire de limiter l’étendue de la découverte, et vous pouvez conserver une vue d’ensemble complète de toutes les charges de travail client dans un projet de migration.

1. Quand vous êtes prêt, poursuivez la migration en sélectionnant l’abonnement client délégué comme destination cible pour la réplication et la migration des charges de travail. Les ressources nouvellement créées existent dans l’abonnement du client, tandis que les données d’évaluation et les ressources relatives au projet de migration sont conservées dans le locataire gestionnaire.

REMARQUE : Vous devez modifier le fichier de paramètres pour qu’il corresponde à votre environnement avant le déploiement. https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegated-resource-management-azmigrate

## <a name="partner-recognition-for-customer-migrations"></a>Reconnaissance de partenaires pour les migrations client

En tant que membre du programme [Microsoft Partner Network](https://partner.microsoft.com), vous pouvez lier votre ID de partenaire à des informations d’identification utilisées pour gérer des ressources client déléguées. Par l’intermédiaire de Partner Admin Link (PAL), Microsoft peut attribuer à votre organisation une influence et des revenus consommés par Azure en fonction des tâches que vous effectuez pour les clients, y compris les projets de migration.

Pour plus d’informations, consultez [Lier votre ID de partenaire pour suivre votre impact sur les ressources déléguées](partner-earned-credit.md).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez [Azure Migrate](../../migrate/migrate-services-overview.md).
- Découvrez les [Expériences de gestion inter-locataire](../concepts/cross-tenant-management-experience.md).

