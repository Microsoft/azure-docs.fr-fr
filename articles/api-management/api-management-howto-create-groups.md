---
title: Gérer des comptes de développeurs à l'aide de groupes dans Gestion des API Azure
titleSuffix: Azure API Management
description: Apprenez à gérer des comptes de développeurs à l’aide de groupes dans Gestion des API Azure. Créez des groupes, puis associez-les à des produits ou des développeurs.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 02/13/2018
ms.author: apimpm
ms.openlocfilehash: ea674981036b4be292329a4b30b43180ed26d642
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92092781"
---
# <a name="how-to-create-and-use-groups-to-manage-developer-accounts-in-azure-api-management"></a>Création et utilisation de groupes pour gérer les comptes de développeurs dans Gestion des API Azure

Dans Gestion des API, les groupes permettent de gérer la visibilité des produits pour les développeurs. Les produits sont d'abord visibles pour les groupes. Les développeurs de ces groupes peuvent afficher les produits associés aux groupes et s'y abonner. 

Le service Gestion des API possède les groupes système suivants, qui ne sont pas modifiables :

* **Administrateurs**  : les administrateurs des abonnements Azure sont membres de ce groupe. Les administrateurs gèrent les instances du service Gestion des API, créant les API, opérations et produits qui sont utilisés par les développeurs.
* **Développeurs**  : les utilisateurs authentifiés du portail des développeurs appartiennent à ce groupe. Les développeurs sont les clients qui génèrent des applications grâce à vos API. Les développeurs bénéficient d'un accès au portail des développeurs et génèrent des applications qui appellent les opérations d'une API.
* **Invités** : les utilisateurs non authentifiés du portail des développeurs, comme les prospects, qui consultent le portail des développeurs d’une instance d’API Management appartiennent à ce groupe. Ils peuvent recevoir certains accès en lecture seule, comme la possibilité d'afficher les API, mais pas de les appeler.

Outre ces groupes système, les administrateurs peuvent créer des groupes personnalisés ou [utiliser des groupes externes dans des locataires Azure Active Directory qui leur sont associés][leverage external groups in associated Azure Active Directory tenants]. Des groupes externes et personnalisés peuvent être utilisés avec des groupes système offrant une certaine visibilité aux développeurs et un accès aux produits d’API. Vous pourriez, par exemple, créer un groupe personnalisé pour les développeurs affiliés à une organisation partenaire spécifique et leur permettre d’accéder aux API à partir d’un produit contenant uniquement des API pertinentes. Un utilisateur peut être membre de plusieurs groupes.

Ce guide explique comment les administrateurs de l'instance Gestion des API peuvent ajouter de nouveaux groupes et les associer à des produits et des développeurs.

En plus de créer et gérer des groupes dans le portail de publication, vous pouvez créer et gérer vos groupes à l'aide de l’entité [Groupe](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-group-entity) de l’API REST de gestion des API.

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="prerequisites"></a>Conditions préalables requises

Effectuez les tâches indiquées dans cet article : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="create-a-group"></a><a name="create-group"> </a>Créer un groupe

Cette section montre comment ajouter un nouveau groupe à votre compte Gestion des API.

1. Sélectionnez l’onglet **Groupes** à gauche de l’écran.
2. Cliquez sur **+Ajouter** .
3. Entrez un nom unique pour le groupe, et éventuellement une description.
4. Appuyez sur **Créer** .

    ![Ajouter un nouveau groupe](./media/api-management-howto-create-groups/groups001.png)

Une fois le groupe créé, il est ajouté à la liste **Groupes** . <br/>Pour modifier le **nom** ou la **description** du groupe, cliquez sur le nom du groupe, puis sur **Paramètres** .<br/>Pour supprimer le groupe, cliquez sur son nom, puis appuyez sur **Supprimer** .

Maintenant que le groupe est créé, il peut être associé à des produits et des développeurs.

## <a name="associate-a-group-with-a-product"></a><a name="associate-group-product"> </a>Associer un groupe à un produit

1. Sélectionnez l’onglet **Produits** vers la gauche.
2. Cliquez sur le nom du produit souhaité.
3. Appuyez sur **Contrôle d’accès** .
4. Cliquez sur **+ Ajouter un groupe** .

    ![Capture d’écran mettant en évidence le bouton Ajouter un groupe.](./media/api-management-howto-create-groups/groups002.png)
5. Sélectionnez le groupe que vous souhaitez ajouter.

    ![Capture d’écran qui montre le groupe sélectionné et met en surbrillance le bouton Sélectionner.](./media/api-management-howto-create-groups/groups003.png)

    Pour supprimer un groupe du produit, cliquez sur **Supprimer** .

    ![Supprimer un groupe](./media/api-management-howto-create-groups/groups004.png)

Une fois le produit associé à un groupe, les développeurs de ce groupe peuvent le voir et s'y abonner.

> [!NOTE]
> Pour ajouter des groupes Azure Active Directory, consultez la rubrique [Comment autoriser des comptes de développeur utilisant Azure Active Directory dans Gestion des API Azure](api-management-howto-aad.md).

## <a name="associate-groups-with-developers"></a><a name="associate-group-developer"> </a>Associer des groupes aux développeurs

Cette section montre comment associer des groupes à des membres.

1. Sélectionnez l’onglet **Groupes** à gauche de l’écran.
2. Sélectionnez **Membres** .

    ![Ajouter un membre](./media/api-management-howto-create-groups/groups005.png)
3. Appuyez sur **+Ajouter** et sélectionnez un membre.

    ![Capture d’écran qui met en surbrillance le bouton Ajouter, l’utilisateur sélectionné et le bouton Sélectionner.](./media/api-management-howto-create-groups/groups006.png)
4. Appuyez sur **Sélectionner** .

Une fois l’association entre le développeur et le groupe ajoutée, vous pouvez la consulter dans l’onglet **Utilisateurs** .

## <a name="next-steps"></a><a name="next-steps"> </a>Étapes suivantes

* Une fois le développeur ajouté à un groupe, il peut voir tous les produits associés à ce groupe et s'y abonner. Pour plus d'informations, consultez la page [Création et publication d'un produit dans Gestion des API Azure][How create and publish a product in Azure API Management].
* En plus de créer et gérer des groupes dans le portail de publication, vous pouvez créer et gérer vos groupes à l'aide de l’entité [Groupe](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-group-entity) de l’API REST de gestion des API.

[Create a group]: #create-group
[Associate a group with a product]: #associate-group-product
[Associate groups with developers]: #associate-group-developer
[Next steps]: #next-steps

[How create and publish a product in Azure API Management]: api-management-howto-add-products.md

[Get started with Azure API Management]: get-started-create-service-instance.md
[Create an API Management service instance]: get-started-create-service-instance.md
[leverage external groups in associated Azure Active Directory tenants]: api-management-howto-aad.md
