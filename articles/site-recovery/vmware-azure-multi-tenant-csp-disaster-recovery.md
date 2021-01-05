---
title: Configurer la récupération d’urgence VMware sur Azure dans un environnement multilocataire avec Site Recovery et le programme du fournisseur de solutions cloud | Microsoft Docs
description: Décrit comment configurer la récupération d’urgence de VMware dans un environnement multilocataire avec Azure Site Recovery.
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/27/2018
ms.author: mayg
ms.openlocfilehash: 334cbef7460306ef2954881a1cab918573e1e371
ms.sourcegitcommit: 4313e0d13714559d67d51770b2b9b92e4b0cc629
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2020
ms.locfileid: "91400636"
---
# <a name="set-up-vmware-disaster-recovery-in-a-multi-tenancy-environment-with-the-cloud-solution-provider-csp-program"></a>Configurer la récupération d'urgence VMware dans un environnement multilocataire avec le programme du fournisseur de solutions cloud

Le [programme CSP](https://partner.microsoft.com/cloud-solution-provider) met en avant des témoignages de collaboration pour tous les services cloud de Microsoft, notamment Microsoft 365, Enterprise Mobility Suite et Microsoft Azure. Grâce au fournisseur de solutions cloud, les partenaires établissent une relation de bout en bout avec leurs clients et deviennent leur point de contact principal. Les partenaires peuvent déployer des abonnements Azure pour les clients et combiner ces abonnements avec leurs propres offres personnalisées à valeur ajoutée.

Avec [Azure Site Recovery](site-recovery-overview.md), en tant que partenaires, vous pouvez gérer la récupération d’urgence pour les clients directement par le biais du fournisseur de solutions cloud. Vous pouvez également utiliser le fournisseur de solutions cloud pour configurer des environnements Site Recovery et permettre aux clients de gérer leurs propres besoins de récupération d’urgence en libre-service. Dans les deux cas, les partenaires jouent le rôle de liaison entre la récupération de site et leurs clients. Les partenaires assument la relation client et facturent l’utilisation de Site Recovery.

Cet article décrit comment un partenaire peut créer et gérer des abonnements de locataires par le biais du fournisseur de solutions cloud, dans le cadre d’un scénario de réplication VMware multilocataire.

## <a name="prerequisites"></a>Conditions préalables requises

Pour configurer la réplication VMware, vous avez besoin d’effectuer les opérations suivantes :

- [Préparer](tutorial-prepare-azure.md) des ressources Azure, notamment un abonnement Azure, un réseau virtuel Azure et un compte de stockage.
- [Préparer](vmware-azure-tutorial-prepare-on-premises.md) des serveurs et machines virtuelles VMware locaux.
- Pour chaque locataire, créez un serveur d’administration distinct capable de communiquer avec les machines virtuelles du locataire et vos serveurs vCenter. Vous seul en tant que partenaire devez avoir des droits d’accès à ce serveur d’administration. Découvrez-en plus sur les [environnements multilocataires](vmware-azure-multi-tenant-overview.md).

## <a name="create-a-tenant-account"></a>Créer un compte locataire

1. Via [Microsoft Partner Center](https://partnercenter.microsoft.com/), connectez-vous à votre compte CSP.
2. Dans le menu **Tableau de bord**, sélectionnez **Clients**.
3. Sur la page qui s’affiche, cliquez sur le bouton **Ajouter un client**.
4. Dans la page **Nouveau client**, renseignez les informations du compte du locataire.

    ![La page Informations du compte](./media/vmware-azure-multi-tenant-csp-disaster-recovery/customer-add-filled.png)

5. Ensuite, cliquez sur **Suivant : Abonnements**.
6. Dans la page de sélection des abonnements, cochez la case **Microsoft Azure**. Vous pouvez ajouter d’autres abonnements maintenant ou à tout moment.
7. Sur la page **Révision**, vérifiez les détails du locataire, puis cliquez sur **Envoyer**.
8. Une fois que vous avez créé le compte locataire, une page de confirmation affichant les détails du compte par défaut et le mot de passe de cet abonnement apparaît. Enregistrez les informations et modifiez le mot de passe ultérieurement si nécessaire, par le biais de la page de connexion au portail Azure.

Vous pouvez partager ces informations avec le locataire en l’état, ou vous pouvez créer et partager un compte distinct si nécessaire.

## <a name="access-the-tenant-account"></a>Accéder au compte locataire

Vous pouvez accéder à l’abonnement du locataire par le biais du tableau de bord du Microsoft Partner Center.

1. Dans la page **Clients**, cliquez sur le nom du compte locataire.
2. Dans la page **Abonnements** du compte locataire, vous pouvez surveiller les abonnements de compte existants et en ajouter d’autres si nécessaire.
3. Pour gérer les opérations de récupération d’urgence du locataire, sélectionnez **Toutes les ressources (portail Azure)** . Cela vous permet d’accéder aux abonnements Azure du locataire.

    ![Le lien Toutes les ressources](./media/vmware-azure-multi-tenant-csp-disaster-recovery/all-resources-select.png)  

4. Vous pouvez vérifier l’accès en cliquant sur le lien Azure Active Directory en haut à droite du portail Azure.

    ![Lien Azure Active Directory](./media/vmware-azure-multi-tenant-csp-disaster-recovery/aad-admin-display.png)

Vous pouvez maintenant effectuer et gérer toutes les opérations Site Recovery pour le locataire dans le portail Azure. Pour accéder à l’abonnement locataire via CSP pour la récupération d’urgence gérée, suivez la procédure décrite précédemment.

## <a name="assign-tenant-access-to-the-subscription"></a>Affecter un accès locataire à l’abonnement

1. Vérifiez que l’infrastructure de récupération d’urgence est configurée. Les partenaires accèdent aux abonnements du locataire par le biais du portail du fournisseur de solutions cloud, que la récupération d’urgence soit gérée ou en libre-service. Configurez votre coffre, puis inscrivez l’infrastructure auprès des abonnements du locataire.
2. Fournir au locataire le [compte que vous avez créé](#create-a-tenant-account).
3. Vous pouvez ajouter un nouvel utilisateur à l’abonnement du locataire par le biais du portail du fournisseur de solutions cloud comme suit :

    a) Accédez à la page d’abonnement du fournisseur de solutions cloud du locataire, puis sélectionnez l’option **Utilisateurs et licences**.

      ![La page d’abonnement CSP du locataire](./media/vmware-azure-multi-tenant-csp-disaster-recovery/users-and-licences.png)

    b) Créez maintenant un utilisateur en entrant les informations adéquates et en sélectionnant des autorisations ou en chargeant la liste des utilisateurs dans un fichier CSV.
    
    c) Une fois que vous avez créé un utilisateur, revenez au portail Azure. Dans la page **Abonnement**, sélectionnez l’abonnement approprié.

    d) Sélectionnez **Contrôle d’accès (IAM)** , puis cliquez sur **Attributions de rôle**.

    (e) Cliquez sur **Ajouter une attribution de rôle** pour ajouter un utilisateur avec le niveau d’accès approprié. Les utilisateurs qui ont été créés via le portail CSP apparaissent sous l’onglet Attributions de rôle.

      ![Ajouter un utilisateur](./media/vmware-azure-multi-tenant-csp-disaster-recovery/add-user-subscription.png)

- Le rôle *Contributeur* suffit pour la plupart des opérations de gestion. Les utilisateurs disposant de ce niveau d’accès peuvent effectuer toutes les opérations sur un abonnement, à l’exception du changement de niveau d’accès (pour cette opération, le niveau d’accès *Propriétaire* est nécessaire).
- Site Recovery propose en outre trois [rôles d’utilisateur prédéfinis](site-recovery-role-based-linked-access-control.md), qui peuvent être utilisés pour limiter davantage les niveaux d’accès en fonction des besoins.

## <a name="multi-tenant-environments"></a>Environnements multilocataires

Il existe trois modèles multilocataires principaux :

* **Fournisseur de services d’hébergement partagé (HSP)** : le partenaire possède l’infrastructure physique et utilise des ressources partagées (vCenter, centres de données, stockage physique, etc.) pour héberger les machines virtuelles de plusieurs locataires sur la même infrastructure. Le partenaire peut fournir la gestion de la récupération d’urgence en tant que service géré, ou le locataire peut disposer de la récupération d’urgence en tant que solution en libre-service.

* **Fournisseur de services d’hébergement dédié** : le partenaire possède l’infrastructure physique mais utilise des ressources dédiées (plusieurs vCenter, magasins de données physiques, etc.) pour héberger les machines virtuelles de chaque locataire sur une infrastructure distincte. Le partenaire peut fournir la gestion de la récupération d’urgence en tant que service géré, ou le locataire peut en disposer en tant que solution en libre-service.

* **Fournisseur de services gérés (MSP)** : le client possède l’infrastructure physique qui héberge les machines virtuelles, et le partenaire réalise l’activation et la gestion de la récupération d’urgence.

En configurant les abonnements du client comme décrit dans cet article, vous pouvez démarrer rapidement l’activation des clients dans un des modèles d’architecture mutualisée pertinents. Plus d’informations sur les différents modèles d’architecture mutualisée et l’activation locale des contrôles d’accès, consultez [cette page](vmware-azure-multi-tenant-overview.md).

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](site-recovery-role-based-linked-access-control.md) pour gérer les déploiements Azure Site Recovery.
- Découvrez plus en détail [l’architecture](vmware-azure-architecture.md) de la réplication VMware sur Azure.
- [Passez en revue le tutoriel](vmware-azure-tutorial.md) pour répliquer des machines virtuelles VMware sur Azure.
Apprenez-en davantage sur les [environnements mutualisés](vmware-azure-multi-tenant-overview.md) pour répliquer des machines virtuelles VMware sur Azure.
