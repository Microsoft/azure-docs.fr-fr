---
title: Tableau de prise en charge de sauvegarde SAP HANA
description: Dans cet article,découvrez les scénarios pris en charge et les limitations lorsque vous utilisez Sauvegarde Azure pour sauvegarder des bases de données SAP HANA sur des machines virtuelles Azure.
ms.topic: conceptual
ms.date: 11/7/2019
ms.custom: references_regions
ms.openlocfilehash: cb7f31986b9ebd91c631d2b896600039c93084de
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96607090"
---
# <a name="support-matrix-for-backup-of-sap-hana-databases-on-azure-vms"></a>Matrice de prise en charge pour la sauvegarde des bases de données SAP HANA sur des machines virtuelles Azure

Le service Sauvegarde Azure prend en charge la sauvegarde des bases de données SAP HANA sur Azure. Cet article résume les scénarios pris en charge et les limitations qui se présentent lorsque vous utilisez Sauvegarde Azure pour sauvegarder des bases de données SAP HANA sur des machines virtuelles Azure.

> [!NOTE]
> La fréquence de sauvegarde de fichier journal peut désormais être définie sur un minimum de 15 minutes. Les sauvegardes de fichiers journaux ne commencent à s’effectuer qu’en cas de réussite d’une sauvegarde complète de la base de données.

## <a name="scenario-support"></a>Prise en charge du scénario

| **Scénario**               | **Configurations prises en charge**                                | **Configurations non prises en charge**                              |
| -------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **Topologie**               | SAP HANA s’exécutant sur des machines virtuelles Azure Linux uniquement                    | Grandes instances HANA (HLI)                                   |
| **Régions**                   | **Disponibilité générale :**<br> **Amériques**  – USA Centre, USA Est 2, USA Est, USA Centre Nord, USA Centre Sud, USA Ouest 2, USA Centre-Ouest, USA Ouest, Canada Centre, Canada Est, Brésil Sud <br> **Asie-Pacifique** – Australie Centre, Australie Centre 2, Australie Est, Australie Sud-Est, Japon Est, Japon Ouest, Corée Centre, Corée Sud, Asie Est, Asie Sud-Est, Inde Centre, Inde Sud, Inde Ouest, Chine Est, Chine Nord, Chine Est 2, Chine Nord 2 <br> **Europe** – Europe Ouest, Europe Nord, France Centre, Royaume-Uni Sud, Royaume-Uni Ouest, Allemagne Nord, Allemagne Centre-Ouest, Suisse Nord, Suisse Ouest, Suisse Centre Nord, Norvège Est, Norvège Ouest <br> **Moyen-Orient et Afrique** – Afrique du Sud Nord, Afrique du Sud Ouest, Émirats arabes unis Nord, Émirats arabes unis Centre  <BR>  **Régions Azure Government** | France Sud, Allemagne Centre, Allemagne Nord-Est, US Gov IOWA |
| **Versions du système d’exploitation**            | SLES 12 avec SP2, SP3, SP4 et SP5 ; SLES 15 avec SP0, SP1 et SP2 <br><br>  Depuis le 1er août 2020, la sauvegarde SAP HANA pour RHEL (7.4, 7.6, 7.7 et 8.1) est en disponibilité générale.                |                                             |
| **Versions HANA**          | SDC sur HANA 1.x, MDC sur HANA 2.x <= SPS04 Rev 48, SPS05 (en attente de validation pour les scénarios de chiffrement activés)      |                                                            |
| **Déploiements HANA**       | SAP HANA sur une machine virtuelle Azure unique - Scale-up uniquement. <br><br> Pour les déploiements à haute disponibilité, les deux nœuds présents sur les deux ordinateurs sont traités individuellement avec des chaînes de données distinctes.               | Montée en charge <br><br> Dans les déploiements à haute disponibilité, la sauvegarde ne bascule pas automatiquement vers le nœud secondaire. La configuration de la sauvegarde doit être effectuée séparément pour chaque nœud.                                           |
| **Instances HANA**         | Une seule SAP HANA instance sur une seule machine virtuelle Azure - montée en puissance uniquement | Plusieurs instances de SAP HANA sur une seule machine virtuelle                  |
| **Types de bases de données HANA**    | Conteneur de base de données unique (SDC) sur 1.x, conteneur de bases de données multiples (MDC) sur 2.x | MDC dans HANA 1. x                                              |
| **Taille de la base de données HANA**     | Bases de données HANA de taille <= 2 To (il ne s’agit pas de la taille de la mémoire du système HANA)               |                                                              |
| **Types de sauvegarde**           | Sauvegardes complètes, différentielles, incrémentielles (préversion) et de fichier journal                          |  Instantanés                                       |
| **Types de restaurations**          | Reportez-vous à la note SAP HANA [1642148](https://launchpad.support.sap.com/#/notes/1642148) pour connaître les types de restauration pris en charge |                                                              |
| **Limites de sauvegarde**          | Jusqu’à 2 To de taille de sauvegarde complète par instance de SAP HANA (limite conditionnelle)         |                                                              |
| **Configurations spéciales** |                                                              | SAP HANA + Dynamic Tiering <br>  Clonage via LaMa        |

------

>[!NOTE]
>Sauvegarde Azure ne s’ajuste pas automatiquement au changement d’heure lorsque vous sauvegardez une base de données SAP HANA qui s’exécute dans une machine virtuelle Azure.
>
>Modifiez la stratégie manuellement en fonction des besoins.

> [!NOTE]
> Vous pouvez maintenant [superviser les travaux de sauvegarde et de restauration](./sap-hana-db-manage.md#monitor-manual-backup-jobs-in-the-portal) (sur la même machine) déclenchés à partir de clients natifs HANA (SAP HANA Studio/Cockpit/DBA Cockpit) sur le portail Azure.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [sauvegarder des bases de données SAP HANA qui s’exécutent sur des machines virtuelles Azure](./backup-azure-sap-hana-database.md)
* Découvrez comment [restaurer des bases de données SAP HANA qui s’exécutent sur des machines virtuelles Azure](./sap-hana-db-restore.md)
* Découvrez comment [gérer des bases de données SAP HANA sauvegardées à l’aide de Sauvegarde Azure](sap-hana-db-manage.md)
* Découvrez comment [résoudre les problèmes courants lors de la sauvegarde de bases de données SAP HANA](./backup-azure-sap-hana-database-troubleshoot.md)
