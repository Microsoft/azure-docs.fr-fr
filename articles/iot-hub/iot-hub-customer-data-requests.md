---
title: Demandes de données client pour les appareils Azure IoT Hub
description: La plupart des appareils gérés dans Azure IoT Hub ne sont pas personnels, mais certains le sont. Cet article parle de la capacité des administrateurs à exporter ou supprimer des données personnelles d’un appareil.
author: robinsh
ms.author: robinsh
ms.date: 05/16/2018
ms.topic: conceptual
ms.service: iot-hub
services: iot-hub
ms.openlocfilehash: df1d4f3a12c6e15e2954bb0e02454ef58b22cb73
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92535788"
---
# <a name="customer-data-request-features-for-azure-iot-hub-devices"></a>Fonctionnalités de demande de données client pour les appareils Azure IoT Hub

Le service Azure IoT Hub est un service cloud basé sur des API REST conçu pour les clients d’entreprise qui permet une communication bidirectionnelle sécurisée entre des millions d’appareils et un service Azure partitionné.

[!INCLUDE [gdpr-related guidance](../../includes/gdpr-intro-sentence.md)]

Un administrateur client assigne un identificateur (ID) d’appareil à chaque appareil. Les données d’appareil reposent sur l’ID d’appareil assigné. Microsoft ne conserve aucune information et n’a pas accès aux données permettant la corrélation entre l’ID d’appareil et l’utilisateur.

La plupart des appareils gérés dans Azure IoT Hub ne sont pas des appareils personnels. Par exemple, il peut s’agir d’un thermostat de bureau ou d’un robot d’usine. Toutefois, les clients peuvent décider que certains appareils sont personnels et gérer eux-mêmes leurs propres méthodes de suivi des ressources ou de l’inventaire afin d’associer des appareils à des personnes. Azure IoT Hub gère et stocke toutes les données associées aux appareils comme s’il s’agissait de données personnelles.

Les administrateurs clients peuvent utiliser le Portail Azure ou les API REST du service pour répondre aux demandes d’informations en exportant ou en supprimant les données associées à un ID d’appareil.

Si vous utilisez la fonctionnalité de routage du service Azure IoT Hub pour transférer les messages des appareils vers d’autres services, l’administrateur client doit effectuer les demandes de données pour chaque point de terminaison de routage afin d’exécuter une demande complète pour un appareil donné. Pour plus d’informations, consultez la documentation de référence de chaque point de terminaison. Pour plus d’informations sur les points de terminaison pris en charge, consultez l’article [Référence - Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md).

Si vous utilisez la fonctionnalité d’intégration à Azure Event Grid du service Azure IoT Hub, les demandes de données doivent être effectuées par l’administrateur client pour chaque abonné à ces événements. Pour plus d’informations, consultez l’article [Réagir aux événements IoT Hub en utilisant Event Grid](iot-hub-event-grid.md).

Si vous utilisez la fonctionnalité d’intégration à Azure Monitor du service Azure IoT Hub pour créer des journaux de ressources, l’administrateur client doit exécuter les demandes de données sur les journaux d’activité stockés. Pour plus d’informations, consultez [Surveiller IoT Hub](monitor-iot-hub.md).

## <a name="deleting-customer-data"></a>Suppression des données client

Les administrateurs clients peuvent utiliser le panneau des appareils IoT de l’extension Azure IoT Hub dans le Portail Azure pour supprimer un appareil, ce qui supprime les données associées à cet appareil.

Il est également possible d’effectuer des opérations de suppression pour les appareils à l’aide des API REST. Pour plus d’informations, consultez [Service - Supprimer un service](/azure/iot-hub/iot-c-sdk-ref/iothub-registrymanager-h/iothubregistrymanager-deletedevice).

## <a name="exporting-customer-data"></a>Exportation des données client

Les administrateurs de locataires peuvent utiliser les opérations de copie et de collage dans le volet des appareils IoT de l’extension Azure IoT Hub dans le Portail Azure pour exporter les données associées à un appareil.

Il est également possible d’effectuer des opérations d’exportation pour les appareils à l’aide des API REST. Pour plus d’informations, consultez [Service - Obtenir un appareil](/azure/iot-hub/iot-c-sdk-ref/iothub-registrymanager-h/iothubregistrymanager-getdevice).

> [!NOTE]
> Lorsque vous utilisez les services d’entreprise Microsoft, Microsoft génère des informations appelées journaux d’activité générés par le système. Certains journaux d’activité générés par le système Azure IoT Hub ne sont pas accessibles ni exportables par les administrateurs clients. Ces journaux d’activité, qui consignent les actions factuelles effectuées au sein du service et les données de diagnostic relatives aux appareils individuels.

## <a name="links-to-additional-documentation"></a>Liens vers la documentation complémentaire

La documentation complète des API du service Azure IoT Hub se trouve dans [API du service IoT Hub](/rest/api/iothub/service/configuration).