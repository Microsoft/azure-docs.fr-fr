---
title: Gérer les appareils dans votre application Azure IoT Central | Microsoft Docs
description: En tant qu’opérateur, apprenez à gérer des appareils dans votre application Azure IoT Central. Découvrez comment gérer des appareils individuels et effectuer des importations et exportations en bloc des appareils de votre application.
author: dominicbetts
ms.author: dobett
ms.date: 10/08/2020
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: peterpr
ms.custom: contperf-fy21q2
ms.openlocfilehash: 2f0b6feea5e586c87191b22f42e3ab86e85ba7b3
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032522"
---
# <a name="manage-devices-in-your-azure-iot-central-application"></a>Gérer les appareils dans votre application Azure IoT Central

Cet article explique comment gérer, en tant qu’opérateur, des appareils dans une application Azure IoT Central. En tant qu’opérateur, vous pouvez :

- Utiliser la page **Appareils** pour visualiser, ajouter et supprimer des appareils connectés à votre application Azure IoT Central
- Importer et exporter des appareils en bloc.
- Tenir à jour un inventaire de vos appareils.
- Maintenir à jour les métadonnées de votre appareil en modifiant les valeurs stockées dans les propriétés de l’appareil à partir des vues.
- Contrôler le comportement de vos appareils en mettant à jour un paramètre sur un appareil spécifique à partir des vues

Pour savoir comment gérer des groupes personnalisés d’appareils, consultez [Tutoriel : Utiliser des groupes d’appareils pour analyser les données de télémétrie des appareils](tutorial-use-device-groups.md).

## <a name="view-your-devices"></a>Voir vos appareils

Pour voir un appareil particulier :

1. Choisissez **Appareils** dans le volet gauche. Vous y voyez la liste complète de vos appareils et modèles d’appareils.

1. Choisissez un modèle d’appareil.

1. Le volet droit de la page **Appareils** présente la liste des appareils créés à partir de ce modèle d’appareils. Choisissez un des appareils pour afficher la page de détails correspondante :

    ![Page Détails de l’appareil](./media/howto-manage-devices/devicelist.png)

## <a name="add-a-device"></a>Ajout d’un appareil

Pour ajouter un appareil à votre application Azure IoT Central :

1. Choisissez **Appareils** dans le volet gauche.

1. Choisissez le modèle d’appareils à partir duquel vous souhaitez créer un appareil.

1. Choisissez + **Nouveau**.

1. Définissez la bascule **Simulé** sur **Activé** ou **Désactivé**. Un appareil réel correspond à un appareil physique que vous connectez à votre application Azure IoT Central. Un appareil simulé a des exemples de données générés par Azure IoT Central.

1. Sélectionnez **Create** (Créer).

1. Cet appareil apparaît maintenant dans la liste des appareils pour ce modèle. Sélectionnez l’appareil pour voir la page de détails correspondante, qui contient toutes les vues relatives à l’appareil.

## <a name="import-devices"></a>Importer des appareils

Pour connecter un grand nombre d’appareils à votre application, vous pouvez importer en bloc des appareils à partir d’un fichier CSV. Le fichier CSV doit disposer des colonnes et en-têtes suivants :

* **IOTC_DeviceID** : l’ID d’appareil peut contenir des lettres, des chiffres et le caractère `-`.
* **IOTC_DeviceName** : cette colonne est facultative.

Pour l’inscription en masse d’appareils :

1. Choisissez **Appareils** dans le volet gauche.

1. Dans le volet gauche, choisissez le modèle d’appareil pour lequel vous souhaitez créer des appareils en masse.

    > [!NOTE]
    > Si vous n’avez pas encore de modèle d’appareil, vous pouvez importer les appareils sous **Tous les appareils** et les inscrire sans modèle. Une fois les appareils importés, vous pouvez les migrer vers un modèle.

1. Sélectionnez **Importer**.

    ![Action d’importation](./media/howto-manage-devices/bulkimport1a.png)


1. Sélectionnez le fichier CSV contenant la liste des ID d’appareil à importer.

1. L’importation de l’appareil démarre une fois le fichier téléchargé. Vous pouvez suivre l’état de l’importation dans le volet Opérations sur les appareils. Ce volet s’affiche automatiquement au démarrage de l’importation. Vous pouvez également y accéder en cliquant sur l’icône représentant une cloche dans le coin supérieur droit.

1. Une fois l’importation terminée, un message de réussite s’affiche dans le volet Opérations sur les appareils.

    ![Importation réussie](./media/howto-manage-devices/bulkimport3a.png)

Si l’importation de l’appareil échoue, un message d’erreur s’affiche dans le volet Opérations sur les appareils. Un fichier journal capturant toutes les erreurs est généré et vous pouvez le télécharger.

## <a name="migrate-devices-to-a-template"></a>Migrer des appareils vers un modèle

Si vous inscrivez des appareils en démarrant l’importation sous **Tous les appareils**, les appareils sont créés sans association à un modèle d’appareil. Les appareils doivent être associés à un modèle pour explorer les données et les autres informations concernant l’appareil. Procédez comme suit pour associer les appareils à un modèle :

1. Choisissez **Appareils** dans le volet gauche.

1. Dans le volet gauche, choisissez **Tous les appareils** :

    ![Appareils non associés](./media/howto-manage-devices/unassociateddevices1a.png)

1. Utilisez le filtre sur la grille pour savoir si la colonne **Modèle d’appareil** présente la valeur **Non attribué(e)** pour l’un de vos appareils.

1. Sélectionnez les appareils que vous souhaitez associer à un modèle :

1. Sélectionnez **Migrer** :

    ![Associer des appareils](./media/howto-manage-devices/unassociateddevices2a.png)

1. Choisissez le modèle dans la liste des modèles disponibles, puis sélectionnez **Migrer**.

1. Les appareils sélectionnés sont associés au modèle d’appareil que vous avez choisi.

## <a name="export-devices"></a>Exporter des appareils

Pour connecter un appareil réel à IoT Central, vous avez besoin de sa chaîne de connexion. Vous pouvez exporter en bloc les détails de l’appareil pour obtenir les informations dont vous avez besoin afin de créer les chaînes de connexion de l'appareil. Le processus d’exportation crée un fichier CSV avec l’identité de l’appareil, le nom de l’appareil et les clés pour tous les appareils sélectionnés.

Pour exporter en bloc des appareils à partir de votre application :

1. Choisissez **Appareils** dans le volet gauche.

1. Dans le volet gauche, choisissez le modèle d’appareil à partir duquel vous souhaitez exporter les appareils.

1. Sélectionnez les appareils que vous souhaitez exporter, puis l’action **Exporter**.

    ![Exporter](./media/howto-manage-devices/export1a.png)

1. Le processus d’exportation démarre. Vous pouvez suivre l’état dans le volet Opérations sur les appareils.

1. Une fois l’exportation terminée, un message de réussite s’affiche avec un lien pour télécharger le fichier généré.

1. Cliquez sur le lien **Télécharger le fichier** pour télécharger le fichier vers un dossier local sur le disque.

    ![Exportation réussie](./media/howto-manage-devices/export2a.png)

1. Le fichier CSV exporté contient les colonnes suivantes : ID de l’appareil, nom de l’appareil, clés d’appareil et empreintes numériques de certificat X509 :

    * IOTC_DEVICEID
    * IOTC_DEVICENAME
    * IOTC_SASKEY_PRIMARY
    * IOTC_SASKEY_SECONDARY
    * IOTC_X509THUMBPRINT_PRIMARY
    * IOTC_X509THUMBPRINT_SECONDARY

Pour plus d’informations sur les chaînes de connexion et la connexion d’appareils réels à votre application IoT Central, consultez [Connectivité des appareils dans Azure IoT Central](concepts-get-connected.md).

## <a name="delete-a-device"></a>Supprimer une unité

Pour supprimer un appareil réel ou simulé depuis votre application Azure IoT Central :

1. Choisissez **Appareils** dans le volet gauche.

1. Choisissez le modèle d’appareil de l’appareil que vous souhaitez supprimer.

1. Utilisez les outils de filtre pour filtrer et rechercher vos appareils. Cochez les cases en regard des appareils à supprimer.

1. Choisissez **Supprimer**. Vous pouvez suivre l’état de la suppression dans le volet Opérations sur les appareils.

## <a name="change-a-property"></a>Modifier une propriété

Les propriétés de cloud sont les métadonnées associées à l’appareil comme la ville et le numéro de série. Les propriétés de cloud existent uniquement dans l’application IoT Central et ne sont pas synchronisées avec vos appareils. Les propriétés accessibles en écriture contrôlent le comportement d’un appareil et vous permettent de définir l’état d’un appareil à distance, par exemple en définissant la température cible d’un thermostat.  Les propriétés de l’appareil sont définies par l’appareil et sont en lecture seule dans IoT Central. Vous pouvez afficher et mettre à jour des propriétés dans les vues **Détails de l’appareil** relatives à votre appareil.

1. Choisissez **Appareils** dans le volet gauche.

1. Choisissez le modèle de l’appareil dont les propriétés doivent être modifiées, puis sélectionnez l’appareil cible.

1. Choisissez la vue qui contient les propriétés de votre appareil. Vous pouvez y entrer des valeurs, puis sélectionner **Enregistrer** en haut de la page. Vous voyez les propriétés de votre appareil et leurs valeurs actuelles. Les propriétés de cloud et les propriétés accessibles en écriture comportent des champs modifiables, tandis que les propriétés de l’appareil sont en lecture seule. Vous pouvez voir l’état de synchronisation des propriétés accessibles en écriture en bas du champ. 

1. Modifiez les propriétés selon les valeurs souhaitées. Vous pouvez modifier plusieurs propriétés à la fois et les mettre à jour en même temps.

1. Choisissez **Enregistrer**. Si vous avez enregistré des propriétés accessibles en écriture, les valeurs sont envoyées à votre appareil. Quand l’appareil confirme la modification de la propriété accessible en écriture, l’état revient à **synchronisé**. Si vous avez enregistré une propriété de cloud, la valeur est mise à jour.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à gérer des appareils dans votre application Azure IoT Central, l’étape suivante suggérée consiste à apprendre à [configurer des règles](howto-configure-rules.md) pour vos appareils.
