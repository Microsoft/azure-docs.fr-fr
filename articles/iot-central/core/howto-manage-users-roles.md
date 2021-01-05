---
title: Gérer les utilisateurs et rôles dans votre application Azure IoT Central | Microsoft Docs
description: Comment gérer les rôles et utilisateurs de votre application Azure IoT Central en qualité d’administrateur
author: lmasieri
ms.author: lmasieri
ms.date: 12/05/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: corywink
ms.openlocfilehash: f6c45b8d9804f16c4e59d259f562cc03f187e6a0
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92122975"
---
# <a name="manage-users-and-roles-in-your-iot-central-application"></a>Gérer les utilisateurs et rôles dans votre application Azure IoT Central

Cet article explique comment, en tant qu’administrateur, vous pouvez ajouter, modifier et supprimer des utilisateurs dans votre application Azure IoT Central. L’article explique également comment gérer des rôles dans votre application Azure IoT Central.

Pour accéder à la section **Administration** et l’utiliser, vous devez avoir le rôle **Administrateur** dans l’application Azure IoT Central. Si vous créez une application Azure IoT Central, le rôle **Administrateur** vous est automatiquement attribué pour cette application.

## <a name="add-users"></a>Ajouter des utilisateurs

Chaque utilisateur doit avoir un compte d’utilisateur avant de pouvoir se connecter et accéder à une application Azure IoT Central. Les comptes Microsoft et les comptes Azure Active Directory sont pris en charge dans Azure IoT Central. Actuellement, les groupes Azure Active Directory ne sont pas pris en charge dans Azure IoT Central.

Pour plus d’informations, consultez les sections [Aide sur le compte Microsoft](https://support.microsoft.com/products/microsoft-account?category=manage-account) et [Démarrage rapide : Ajouter de nouveaux utilisateurs à Azure Active Directory](../../active-directory/fundamentals/add-users-azure-active-directory.md).

1. Pour ajouter un utilisateur à une application IoT Central, accédez à la page **Utilisateurs** dans la section **Administration**.
    
    > [!div class="mx-imgBorder"]
    >![Gestion des utilisateurs](media/howto-manage-users-roles/manage-users-pnp.png)

1. Pour ajouter un utilisateur, dans la page **Utilisateurs**, choisissez **+ Ajouter un utilisateur**.

1. Choisissez un rôle pour l’utilisateur dans la liste déroulante **Rôle**. En savoir plus sur les rôles dans la section [Gérer les rôles](#manage-roles) de cet article.

    > [!div class="mx-imgBorder"]
    >![Ajouter un utilisateur et sélectionner un rôle](media/howto-manage-users-roles/add-user-pnp.png)

    > [!NOTE]
    > Un utilisateur qui se trouve dans un rôle personnalisé qui lui accorde l’autorisation d’ajouter d’autres utilisateurs peut uniquement ajouter des utilisateurs à un rôle avec des autorisations identiques ou moins nombreuses que son propre rôle.

Si un ID d’utilisateur IoT Central est supprimé d’Azure Active Directory puis rajouté, l’utilisateur ne peut pas se connecter à l’application IoT Central. Pour réactiver l’accès, l’administrateur IoT Central doit supprimer et rajouter l’utilisateur dans l’application.

### <a name="edit-the-roles-that-are-assigned-to-users"></a>Modifier les rôles attribués aux utilisateurs

Une fois attribués, les rôles ne sont plus modifiables. Pour modifier le rôle attribué à un utilisateur, supprimez cet utilisateur puis rajoutez-le avec un autre rôle.

> [!NOTE]
> Les rôles attribués sont spécifiques de l’application IoT Central et ne peuvent pas être gérés à partir du portail Azure.

## <a name="delete-users"></a>Suppression d’utilisateurs

Pour supprimer des utilisateurs, sélectionnez une ou plusieurs cases à cocher sur la page **Utilisateurs**. Puis sélectionnez **Supprimer**.

## <a name="manage-roles"></a>Gérer les rôles

Les rôles vous permettent de contrôler qui, au sein de votre organisation, peut effectuer différentes tâches dans IoT Central. Vous pouvez attribuer trois rôles intégrés aux utilisateurs de votre application. Vous pouvez également [créer des rôles personnalisés](#create-a-custom-role) si vous avez besoin d’un contrôle plus fin.

> [!div class="mx-imgBorder"]
> ![Gérer la sélection de rôles](media/howto-manage-users-roles/manage-roles-pnp.png)

### <a name="administrator"></a>Administrateur

Les utilisateurs du rôle **Administrateur** peuvent gérer et contrôler toutes les parties de l’application, notamment la facturation.

L’utilisateur qui crée une application reçoit automatiquement le rôle **Administrateur**. Le rôle **Administrateur** doit toujours être attribué à au moins un utilisateur.

### <a name="builder"></a>Générateur

Les utilisateurs du rôle **Créateur** peuvent gérer toutes les parties de l’application, mais ne peuvent pas changer les onglets Administration ou Exportation continue des données.

### <a name="operator"></a>Opérateur

Les utilisateurs du rôle **Opérateur** peuvent surveiller l’intégrité et l’état des appareils. Ils ne sont pas autorisés à apporter des modifications aux modèles d’appareils ni à administrer l’application. Les opérateurs peuvent ajouter et supprimer des appareils, gérer des ensembles d’appareils et exécuter des analytiques et des travaux. 

## <a name="create-a-custom-role"></a>Créer un rôle personnalisé

Si votre solution nécessite des contrôles d’accès plus précis, vous pouvez créer des rôles personnalisés avec des ensembles d’autorisations personnalisés. Pour créer un rôle personnalisé, accédez à la page **Rôle** dans la section **Administration** de votre application. Sélectionnez ensuite **+ Nouveau rôle**, puis ajoutez un nom et une description pour votre rôle. Sélectionnez les autorisations requises par votre rôle, puis choisissez **Enregistrer**.

Vous pouvez ajouter des utilisateurs à votre rôle personnalisé de la même façon que vous ajoutez des utilisateurs à un rôle intégré.

> [!div class="mx-imgBorder"]
> ![Créer un rôle personnalisé](media/howto-manage-users-roles/create-custom-role-pnp.png)

### <a name="custom-role-options"></a>Options de rôle personnalisé

Lorsque vous définissez un rôle personnalisé, vous choisissez le jeu d’autorisations qu’un utilisateur reçoit s’il est membre du rôle. Certaines autorisations dépendent les unes des autres. Par exemple, si vous ajoutez l’autorisation **Mettre à jour les tableaux de bord d’application** à un rôle, l’autorisation **Afficher les tableaux de bord d’application** est automatiquement ajoutée. Les tableaux suivants résument les autorisations disponibles et leurs dépendances, que vous pouvez utiliser lors de la création de rôles personnalisés.

#### <a name="managing-devices"></a>Gestion des appareils

**Autorisations du modèle d’appareil**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None     |
| Gérer | Affichage <br/> Autres dépendances : Afficher les instances de service  |
| Contrôle total | Afficher, Gérer <br/> Autres dépendances : Afficher les instances de service |

**Autorisations des instances d’appareils**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None <br/> Autres dépendances : Afficher les modèles d’appareil et les groupes d’appareils |
| Update | Affichage <br/> Autres dépendances : Afficher les modèles d’appareil et les groupes d’appareils  |
| Créer | Affichage <br/> Autres dépendances :  Afficher les modèles d’appareil et les groupes d’appareils  |
| DELETE | Affichage <br/> Autres dépendances : Afficher les modèles d’appareil et les groupes d’appareils  |
| Exécuter des commandes | Mettre à jour, Afficher <br/> Autres dépendances : Afficher les modèles d’appareil et les groupes d’appareils  |
| Contrôle total | Afficher, Mettre à jour, Créer, Supprimer, Exécuter des commandes <br/> Autres dépendances : Afficher les modèles d’appareil et les groupes d’appareils  |

**Autorisations des groupes d’appareils**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None <br/> Autres dépendances : Afficher les modèles d’appareil et les instances d’appareils |
| Update | Affichage <br/> Autres dépendances : Afficher les modèles d’appareil et les instances d’appareils   |
| Créer | Afficher, Mettre à jour <br/> Autres dépendances :  Afficher les modèles d’appareil et les instances d’appareils   |
| DELETE | Affichage <br/> Autres dépendances :  Afficher les modèles d’appareil et les instances d’appareils   |
| Contrôle total | Afficher, Mettre à jour, Créer, Supprimer <br/> Autres dépendances : Afficher les modèles d’appareil et les instances d’appareils |

**Autorisations de gestion de la connectivité des appareils**

| Nom | Les dépendances |
| ---- | -------- |
| Lire l’instance | None <br/> Autres dépendances : Afficher les modèles d’appareil, les groupes d’appareils et les instances d’appareils |
| Gérer l’instance | None |
| Lire au niveau global | None   |
| Gérer au niveau global | Lire au niveau global |
| Contrôle total | Lire l’instance, Gérer l’instance, Lire au niveau global, Gérer au niveau global. <br/> Autres dépendances : Afficher les modèles d’appareil, les groupes d’appareils et les instances d’appareils |

**Autorisations de travaux**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None <br/> Autres dépendances : Afficher les modèles d’appareils, les instances d’appareils et les groupes d’appareils |
| Update | Affichage <br/> Autres dépendances : Afficher les modèles d’appareils, les instances d’appareils et les groupes d’appareils |
| Créer | Afficher, Mettre à jour <br/> Autres dépendances :  Afficher les modèles d’appareils, les instances d’appareils et les groupes d’appareils |
| DELETE | Affichage <br/> Autres dépendances :  Afficher les modèles d’appareils, les instances d’appareils et les groupes d’appareils |
| Execute | Affichage <br/> Autres dépendances : Afficher les modèles d’appareils, les instances d’appareils et les groupes d’appareils ; Mettre à jour les instances d’appareils ; Exécuter des commandes sur des instances d’appareils |
| Contrôle total | Afficher, Mettre à jour, Créer, Supprimer, Exécuter <br/> Autres dépendances :  Afficher les modèles d’appareils, les instances d’appareils et les groupes d’appareils ; Mettre à jour les instances d’appareils ; Exécuter des commandes sur des instances d’appareils |

**Autorisations de règles**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None <br/> Autres dépendances : Afficher les modèles d’appareils |
| Update | Affichage <br/> Autres dépendances : Afficher les modèles d’appareils |
| Créer | Afficher, Mettre à jour <br/> Autres dépendances :  Afficher les modèles d’appareils |
| DELETE | Affichage <br/> Autres dépendances : Afficher les modèles d’appareils |
| Contrôle total | Afficher, Mettre à jour, Créer, Supprimer <br/> Autres dépendances : Afficher les modèles d’appareils |

#### <a name="managing-the-app"></a>Gestion de l’application

**Autorisations des paramètres d’application**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None     |
| Update | Affichage   |
| Copier | Affichage <br/> Autres dépendances : Afficher les modèles d’appareils, les instances d’appareils, les groupes d’appareils, les tableaux de bord, l’exportation de données, la personnalisation, les liens d’aide, les rôles personnalisés et les règles |
| DELETE | Affichage   |
| Contrôle total | Afficher, Mettre à jour, Copier, Supprimer <br/> Autres dépendances : Afficher les modèles d’appareils, les groupes d’appareils, les tableaux de bord d’application, l’exportation de données, la personnalisation, les liens d’aide, les rôles personnalisés et les règles |

**Autorisations d’exportation du modèle d’application**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None     |
| Exporter | Affichage <br/> Autres dépendances :  Afficher les modèles d’appareils, les instances d’appareils, les groupes d’appareils, les tableaux de bord, l’exportation de données, la personnalisation, les liens d’aide, les rôles personnalisés et les règles |
| Contrôle total | Afficher, Exporter <br/> Autres dépendances :  Afficher les modèles d’appareils, les groupes d’appareils, les tableaux de bord d’application, l’exportation de données, la personnalisation, les liens d’aide, les rôles personnalisés et les règles |

**Autorisations de facturation**

| Nom | Les dépendances |
| ---- | -------- |
| Gérer | None     |
| Contrôle total | Gérer |

#### <a name="managing-users-and-roles"></a>Gestion des utilisateurs et des rôles

**Autorisations de rôles personnalisés**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None |
| Update | Affichage |
| Créer | Afficher, Mettre à jour |
| DELETE | Affichage |
| Contrôle total | Afficher, Mettre à jour, Créer, Supprimer |

**Autorisations de gestion des utilisateurs**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None <br/> Autres dépendances : Afficher les rôles personnalisés |
| Ajouter | Affichage <br/> Autres dépendances :  Afficher les rôles personnalisés |
| DELETE | Affichage <br/> Autres dépendances :  Afficher les rôles personnalisés |
| Contrôle total | Afficher, Ajouter, Supprimer <br/> Autres dépendances :  Afficher les rôles personnalisés |

> [!NOTE]
> Un utilisateur qui se trouve dans un rôle personnalisé qui lui accorde l’autorisation d’ajouter d’autres utilisateurs peut uniquement ajouter des utilisateurs à un rôle avec des autorisations identiques ou moins nombreuses que son propre rôle.

#### <a name="customizing-the-app"></a>Personnalisation de l’application

**Autorisations des tableaux de bord d’applications**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None     |
| Update | Affichage   |
| Créer | Afficher, Mettre à jour |
| DELETE | Affichage   |
| Contrôle total | Afficher, Mettre à jour, Créer, Supprimer |

**Autorisations des tableaux de bord personnels**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None     |
| Update | Affichage   |
| Créer | Afficher, Mettre à jour   |
| DELETE | Affichage   |
| Contrôle total | Afficher, Mettre à jour, Créer, Supprimer |

**Autorisations Marque, Icône favorite et Couleurs**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None     |
| Update | Affichage   |
| Contrôle total | Afficher, Mettre à jour |

**Autorisations de liens d’aide**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None     |
| Update | Affichage   |
| Contrôle total | Afficher, Mettre à jour |

#### <a name="extending-the-app"></a>Extension de l’application

**Autorisations d’exportation de données**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None     |
| Update | Affichage   |
| Créer | Afficher, Mettre à jour  |
| DELETE | Affichage   |
| Contrôle total | Afficher, Mettre à jour, Créer, Supprimer |

**Autorisations de jeton d’API**

| Nom | Les dépendances |
| ---- | -------- |
| Affichage | None     |
| Créer | Affichage   |
| DELETE | Affichage   |
| Contrôle total | Afficher, Créer, Supprimer |

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à gérer les utilisateurs et les rôles dans votre application Azure IoT Central, l’étape suivante suggérée est d’apprendre à [gérer votre facture](howto-view-bill.md).