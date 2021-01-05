---
title: Gérer des jeux d’enregistrements DNS et des enregistrements avec Azure DNS
description: Azure DNS permet de gérer les jeux d’enregistrements DNS et les enregistrements lors de l’hébergement de votre domaine.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 10/6/2018
ms.author: rohink
ms.openlocfilehash: 22ceba69ecf865d906021068a39a9d273b842ca2
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965729"
---
# <a name="manage-dns-records-and-record-sets-by-using-the-azure-portal"></a>Gestion d’enregistrements et de jeux d’enregistrements DNS à l’aide du portail Azure

Cet article explique comment gérer les jeux d’enregistrements et les enregistrements pour votre zone DNS À l’aide du portail Azure.

Il est important de comprendre la différence entre les jeux d’enregistrements DNS et les enregistrements DNS individuels. Un jeu d’enregistrements est une collection d’enregistrements dans une zone ayant le même nom et le même type. Pour plus d’informations, consultez [Création de jeux d’enregistrements et d’enregistrements DNS à l’aide du portail Azure](./dns-getstarted-portal.md).

## <a name="create-a-new-record-set-and-record"></a>Création d’un jeu d’enregistrements et d’un enregistrement

Pour créer un jeu d’enregistrements dans le portail Azure, consultez [Création d’enregistrements DNS à l’aide du portail Azure](./dns-getstarted-portal.md).

## <a name="view-a-record-set"></a>Affichage d’un jeu d’enregistrements

1. Dans le portail Azure, accédez au panneau **zone DNS** .
2. Recherchez le jeu d’enregistrements et sélectionnez-le. Les propriétés du jeu d’enregistrements s’affichent.

    ![Recherche d’un jeu d’enregistrements](./media/dns-operations-recordsets-portal/searchset500.png)

## <a name="add-a-new-record-to-a-record-set"></a>Ajouter un nouvel enregistrement à un jeu d’enregistrements

Vous pouvez ajouter jusqu'à 20 enregistrements à n'importe quel jeu d'enregistrements. Un jeu d’enregistrements ne peut pas contenir deux enregistrements identiques. Des jeux d'enregistrements vides (avec zéro enregistrement) peuvent être créés, mais ils n'apparaîtront pas sur les serveurs de noms Azure DNS. Les jeux d’enregistrements du type CNAME peuvent contenir un enregistrement au maximum.

1. Dans le panneau des **propriétés du jeu d’enregistrements** de votre zone DNS, cliquez sur le jeu d’enregistrements auquel vous souhaitez ajouter un enregistrement.

    ![Sélection d’un jeu d'enregistrements](./media/dns-operations-recordsets-portal/selectset500.png)

2. Spécifiez les propriétés du jeu d’enregistrements en remplissant les champs.

    ![Ajouter un enregistrement](./media/dns-operations-recordsets-portal/addrecord500.png)

3. Cliquez sur **Enregistrer** en haut du panneau pour enregistrer vos paramètres. Puis fermez le panneau.
4. Dans l’angle, vous verrez que l’enregistrement est en cours de sauvegarde.

    ![Enregistrement d’un jeu d'enregistrements](./media/dns-operations-recordsets-portal/saving150.png)

Une fois que l’enregistrement a été sauvegardé, les valeurs du panneau **Zone DNS** reflètent le nouvel enregistrement.

## <a name="update-a-record"></a>Mise à jour d’un enregistrement

Lors de la mise à jour d’un enregistrement dans un jeu d’enregistrements existant, les champs pouvant être mis à jour varient selon le type d’enregistrement que vous utilisez.

1. Dans le panneau **Propriétés du jeu d’enregistrements** de votre jeu d’enregistrements, recherchez l’enregistrement.
2. Modifiez l’enregistrement. Lorsque vous modifiez un enregistrement, vous pouvez modifier les paramètres disponibles pour l’enregistrement. Dans l’exemple suivant, le champ **Adresse IP** est sélectionné et l’adresse IP est en cours de modification.

    ![Modification d’un enregistrement](./media/dns-operations-recordsets-portal/modifyrecord500.png)

3. Cliquez sur **Enregistrer** en haut du panneau pour enregistrer vos paramètres. Une notification s’affiche dans le coin supérieur droit indiquant que l’enregistrement a été sauvegardé.

    ![Ajout d’un jeu d’enregistrements](./media/dns-operations-recordsets-portal/saved150.png)

Une fois que l’enregistrement a été sauvegardé, les valeurs du jeu d’enregistrements dans le panneau **Zone DNS** reflètent l’enregistrement mis à jour.

## <a name="remove-a-record-from-a-record-set"></a>Suppression d’un enregistrement d’un jeu d’enregistrements

Vous pouvez utiliser le portail Azure pour supprimer des enregistrements d’un jeu d’enregistrements. Notez que la suppression du dernier enregistrement d’un jeu d’enregistrements ne supprime pas le jeu d’enregistrements.

1. Dans le panneau **Propriétés du jeu d’enregistrements** de votre jeu d’enregistrements, recherchez l’enregistrement.
2. Cliquez sur l’enregistrement que vous souhaitez supprimer. Puis sélectionnez **Supprimer**.

    ![Suppression d’un enregistrement](./media/dns-operations-recordsets-portal/removerecord500.png)

3. Cliquez sur **Enregistrer** en haut du panneau pour enregistrer vos paramètres.
4. Une fois que l’enregistrement a été supprimé, les valeurs du jeu d’enregistrements dans le panneau **Zone DNS** reflètent la suppression de l’enregistrement.

## <a name="delete-a-record-set"></a><a name="delete"></a>Suppression d’un jeu d'enregistrements

1. Dans le panneau **Propriétés du jeu d’enregistrements** de votre jeu d’enregistrements, cliquez sur **Supprimer**.

    ![Supprimer un jeu d’enregistrements](./media/dns-operations-recordsets-portal/deleterecordset500.PNG)

2. Un message s’affiche vous demandant si vous souhaitez supprimer le jeu d’enregistrements.
3. Vérifiez que le nom correspond au jeu d’enregistrements que vous souhaitez supprimer, puis cliquez sur **Oui**.
4. Dans le panneau de **Zone DNS** , vous pouvez vérifier que le jeu d’enregistrements n’est plus visible.

## <a name="work-with-ns-and-soa-records"></a>Utilisation d’enregistrements NS et SOA

Les enregistrements NS et SOA qui sont créés automatiquement sont gérés différemment des autres types d’enregistrements.

### <a name="modify-soa-records"></a>Modification d'enregistrements SOA

Vous ne pouvez pas ajouter ou supprimer d’enregistrements dans le jeu d’enregistrements SOA créé automatiquement à l’extrémité de la zone (nom = « \@ »). Vous pouvez cependant modifier les paramètres dans l’enregistrement SOA (à l’exception de « l’hôte ») et pendant la durée de vie du jeu d’enregistrements.

### <a name="modify-ns-records-at-the-zone-apex"></a>Modification d’enregistrements NS à l’extrémité de la zone

Le jeu d’enregistrements NS à l’apex de la zone est créé automatiquement avec chaque zone DNS. Il contient les noms des serveurs de noms Azure DNS attribués à la zone.

Vous pouvez ajouter des serveurs de noms supplémentaires à ce jeu d’enregistrements NS, pour prendre en charge le co-hébergement de domaines avec plusieurs fournisseurs DNS. Vous pouvez également modifier la durée de vie et les métadonnées pour ce jeu d’enregistrements. Toutefois, vous ne pouvez pas supprimer ni modifier les serveurs de noms Azure DNS préremplis.

Notez que cela s’applique uniquement au jeu d’enregistrements NS défini à l’apex de la zone. Les autres jeux d’enregistrements NS dans votre zone (tels que ceux utilisés pour déléguer des zones enfants) peuvent être modifiés sans contrainte.

### <a name="delete-soa-or-ns-record-sets"></a>Suppression de jeux d’enregistrements SOA ou NS

Vous ne pouvez pas supprimer les jeux d’enregistrements SOA et NS à l’extrémité de la zone (nom = « \@ ») qui sont créés automatiquement quand la zone est créée. Ils sont automatiquement supprimés lorsque vous supprimez la zone.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Azure DNS, consultez la [Vue d’ensemble d’Azure DNS](dns-overview.md).
* Pour plus d’informations sur l’automatisation de DNS, consultez la rubrique [Création des zones DNS et de jeux d’enregistrements à l’aide du Kit de développement logiciel (SDK) .NET](dns-sdk.md).
* Pour plus d’informations sur les enregistrements DNS inversés, consultez l’article [Vue d’ensemble des DNS inversés et assistance dans Azure](dns-reverse-dns-overview.md).
* Pour plus d’informations sur les enregistrements d’alias Azure DNS, consultez la [Vue d’ensemble des enregistrements d’alias Azure DNS](dns-alias.md).