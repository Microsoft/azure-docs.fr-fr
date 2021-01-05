---
title: En savoir plus sur les dernières publications du SE invité de Microsoft Azure | Microsoft Docs
description: Dernières informations de publication et de compatibilité du Kit de développement logiciel (SDK) pour le SE invité de Microsoft Azure Cloud Services.
services: cloud-services
documentationcenter: na
author: yohaddad
editor: ''
ms.assetid: 6306cafe-1153-44c7-8554-623b03d59a34
ms.service: cloud-services
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: tbd
ms.date: 11/17/2020
ms.author: yohaddad
ms.openlocfilehash: a5cc0fdf0473cb8ceafd164a34df6f0bd66dd6d3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96013698"
---
# <a name="azure-guest-os-releases-and-sdk-compatibility-matrix"></a>Versions du SE invité et matrice de compatibilité du Kit de développement logiciel (SDK) Azure
Fournit des informations récentes sur les dernières publications du SE (système d’exploitation) invité de Microsoft Azure pour Cloud Services. Ces informations vous permettent de planifier votre mise à niveau avant la désactivation d’un SE invité. Si vous configurez vos rôles pour utiliser les mises à jour *automatiques* d’un SE invité comme cela est décrit dans [Paramètres de mise à jour du SE invité Azure][Azure Guest OS Update Settings], il n’est pas indispensable que vous lisiez cette page.

> [!IMPORTANT]
> Le contenu de cette page s’applique aux rôles web et de travail Cloud Services, qui s’exécutent sur un SE invité. Il **ne s’applique pas** aux machines virtuelles IaaS.
>
>


> [!TIP]
>  Abonnez-vous au [Flux RSS de mise à jour du SE invité] pour recevoir en temps voulu les notifications relatives à toutes les modifications du SE invité.
>
>

> [!IMPORTANT]
> Seules les 2 dernières versions du système d’exploitation invité seront prises en charge et disponibles dans le portail Microsoft Azure.
>
>

Vous ne savez pas comment mettre à jour votre système d’exploitation invité ? Consultez [cet article][cloud updates].

## <a name="news-updates"></a>Nouvelles mises à jour

###### <a name="november-17-2020"></a>**17 novembre 2020**
Publication de l’OS invité octobre. 

###### <a name="october-10-2020"></a>**10 octobre 2020**
Publication du système d’exploitation invité du mois de septembre. 

###### <a name="september-5-2020"></a>**5 septembre 2020**
Publication du système d’exploitation invité du mois d’août. 

###### <a name="august-17-2020"></a>**17 août 2020**
Le système d’exploitation invité de juillet a été publié. 

###### <a name="august-10-2020"></a>**10 août 2020**
Le système d’exploitation invité de juin a été publié. 

###### <a name="june-2-2020"></a>**2 juin 2020**
Le système d’exploitation invité de mai a été publié. 

###### <a name="may-4-2020"></a>**4 mai 2020**
Le système d’exploitation invité d’avril a été publié. 

###### <a name="april-2-2020"></a>**2 avril 2020**
Le système d’exploitation invité de mars a été publié. 

###### <a name="march-5-2020"></a>**5 mars 2020**
Le SE invité de février a publié. 

###### <a name="january-24-2020"></a>**24 janvier 2020**
Le SE invité de janvier a publié. 

###### <a name="january-8-2020"></a>**8 janvier 2020**
Le système d’exploitation invité de décembre a été publié.

###### <a name="december-5-2019"></a>**5 décembre 2019**
Publication du système d’exploitation invité du mois de novembre.

###### <a name="november-1-2019"></a>**1er novembre 2019**
Publication de l’OS invité octobre.

###### <a name="october-7-2019"></a>**7 octobre 2019**
Publication du système d’exploitation invité du mois de septembre.

###### <a name="september-4-2019"></a>**4 septembre 2019**
Publication du système d’exploitation invité du mois d’août.

###### <a name="july-26-2019"></a>**26 juillet 2019**
Le système d’exploitation invité de juillet a été publié.

###### <a name="july-8-2019"></a>**8 juillet 2019**
Le système d’exploitation invité de juin a été publié.

###### <a name="june-6-2019"></a>**6 juin 2019**
Le système d’exploitation invité de mai a été publié.

###### <a name="may-7-2019"></a>**7 mai 2019**
Le système d’exploitation invité d’avril a été publié.

###### <a name="march-26-2019"></a>**26 mars 2019**
Le système d’exploitation invité de mars a été publié.

###### <a name="march-12-2019"></a>**12 mars 2019**
Le SE invité de février a publié.

###### <a name="february-5-2019"></a>**5 février 2019**
Le SE invité de janvier a publié.

###### <a name="january-24-2019"></a>**24 janvier 2019**
Le système d’exploitation invité de famille 6 (Windows Server 2019) a été publié.

###### <a name="january-7-2019"></a>**7 janvier 2019**
Le système d’exploitation invité de décembre a été publié.

###### <a name="december-14-2018"></a>**14 décembre 2018**
Publication du système d’exploitation invité du mois de novembre.

###### <a name="november-8-2018"></a>**8 novembre 2018**
Publication de l’OS invité octobre.

###### <a name="october-12-2018"></a>**12 octobre 2018**
Publication du système d’exploitation invité du mois de septembre.

## <a name="releases"></a>Publications

## <a name="family-6-releases"></a>Publications de famille 6
**Windows Server 2019**

.NET Framework installé : 3.5, 4.7.2

> [!NOTE]
> Le SDK Windows Azure pour .NET - 3.0 peut être téléchargé [ici][Windows Azure SDK].
>
>Étapes d'installation :
>1. Veuillez désinstaller les anciennes versions du fichier MicrosoftAzureAuthoringTools*.msi
>2. Installer le [kit SDK Azure pour .NET - 3.0][Windows Azure SDK]
>3. Redémarrer votre machine
>4. Créer un projet de service cloud et ajouter un seul rôle de travail
>5. Définir la famille de système d’exploitation sur 6 et générer un paquet
>6. Déployer le paquet sur Azure à l’aide du portail Azure ou de Visual Studio
>
>La version de système d’exploitation Family 6 applique TLS 1.2 en désactivant explicitement TLS 1.0 et 1.1 et en définissant un ensemble spécifique de suites de chiffrement. En savoir [plus].


| Chaîne de configuration | Date de publication | Date de désactivation |
| --- | --- | --- |
|  WA-GUEST-OS-6.24_202010-02 |  17 novembre 2020  |  Post 6.26  |  
|  WA-GUEST-OS-6.23_202009-01  |  10 octobre 2020  |  Post 6.25  |  
|~~WA-GUEST-OS-6.22_202008-02~~|  5 septembre 2020  |  17 novembre 2020  |  
|~~WA-GUEST-OS-6.21_202007-01~~|  17 août 2020  |  10 octobre 2020  |  
|~~WA-GUEST-OS-6.20_202006-02~~|  10 août 2020  |  5 septembre 2020  |  
|~~WA-GUEST-OS-6.19_202005-02~~|  2 juin 2020  |  17 août 2020  |  
|~~WA-GUEST-OS-6.18_202004-01~~|  4 mai 2020  |  10 août 2020  |  
|~~WA-GUEST-OS-6.17_202003-01~~|  2 avril 2020  |  2 juin 2020  |  
|~~WA-GUEST-OS-6.16_202002-01~~|  5 mars 2020  |  4 mai 2020  |  
|~~WA-GUEST-OS-6.15_202001-01~~|  24 janvier 2020  |  2 avril 2020  |  
|~~WA-GUEST-OS-6.14_201912-01~~| 8 janvier 2020 | 5 mars 2020 |  
|~~WA-GUEST-OS-6.13_201911-01~~| 5 décembre 2019 | 24 janvier 2020 |  
|~~WA-GUEST-OS-6.12_201910-01~~| 1er novembre 2019 | 8 janvier 2020 |  
|~~WA-GUEST-OS-6.11_201909-01~~| 7 octobre 2019 | 5 décembre 2019 |  
|~~WA-GUEST-OS-6.10_201908-01~~| 4 août 2019 | 1er novembre 2019  |  
|~~WA-GUEST-OS-6.9_201907-0~~|26 juillet 2019 | 7 octobre 2019 |
|~~WA-GUEST-OS-6.8_201906-01~~|8 juillet 2019 |4 août 2019 |
|~~WA-GUEST-OS-6.7_201905-01~~ |6 juin 2019 |26 juillet 2019 |
|~~WA-GUEST-OS-6.6_201904-01~~ |7 mai 2019 |8 juillet 2019 |
|~~WA-GUEST-OS-6.5_201903-01~~ |26 mars 2019 |6 juin 2019 |
|~~WA-GUEST-OS-6.4_201902-01~~ |12 mars 2019 |7 mai 2019 |
|~~WA-GUEST-OS-6.3_201901-01~~ |5 février 2019 |26 mars 2019 |
|~~WA-GUEST-OS-6.2_201812-01~~ |24 janvier 2019 |12 mars 2019 |
|~~WA-GUEST-OS-6.1_201811-01~~ |24 janvier 2019 |5 février 2019 |

## <a name="family-5-releases"></a>Publications de famille 5
**Windows Server 2016**

.NET Framework installé : 3.5, 4.6.2

> [!NOTE]
> Le mot de passe RDP pour la famille 5 de SE doit comporter au moins 10 caractères.
>


| Chaîne de configuration | Date de publication | Date de désactivation |
| --- | --- | --- |
|  WA-GUEST-OS-5.48_202010-02  |  17 novembre 2020  |  Post 5.50  | 
|  WA-GUEST-OS-5.47_202009-01  |  10 octobre 2020  |  Post 5.49  | 
|~~WA-GUEST-OS-5.46_202008-02~~|  5 septembre 2020  |  17 novembre 2020  |  
|~~WA-GUEST-OS-5.45_202007-01~~|  17 août 2020  |  10 octobre 2020  |  
|~~WA-GUEST-OS-5.44_202006-02~~|  10 août 2020  |  5 septembre 2020  |  
|~~WA-GUEST-OS-5.43_202005-02~~|  2 juin 2020  |  17 août 2020  |  
|~~WA-GUEST-OS-5.42_202004-01~~|  4 mai 2020  |  10 août 2020  |  
|~~WA-GUEST-OS-5.41_202003-01~~|  2 avril 2020  |  2 juin 2020  |  
|~~WA-GUEST-OS-5.40_202002-01~~|  5 mars 2020  |  4 mai 2020  |  
|~~WA-GUEST-OS-5.39_202001-01~~|  24 janvier 2020  |  2 avril 2020  |  
|~~WA-GUEST-OS-5.38_201912-01~~| 8 janvier 2020 | 5 mars 2020 |  
|~~WA-GUEST-OS-5.37_201911-01~~| 5 décembre 2019 | 24 janvier 2020 |  
|~~WA-GUEST-OS-5.36_201910-01~~| 1er novembre 2019 | 8 janvier 2020 |  
|~~WA-GUEST-OS-5.35_201909-01~~| 7 octobre 2019 | 5 décembre 2019 |  
|~~WA-GUEST-OS-5.34_201908-01~~|  4 août 2019  | 1er novembre 2019 |  
|~~WA-GUEST-OS-5.33_201907-01~~| 26 juillet 2019 | 7 octobre 2019 |  
|~~WA-GUEST-OS-5.32_201906-01~~|8 juillet 2019 |4 août 2019 |
|~~WA-GUEST-OS-5.31_201905-01~~ |6 juin 2019 |26 juillet 2019 |
|~~WA-GUEST-OS-5.30_201904-01~~ |7 mai 2019 |8 juillet 2019 |
|~~WA-GUEST-OS-5.29_201903-01~~ |26 mars 2019 |6 juin 2019 |
|~~WA-GUEST-OS-5.28_201902-01~~ |12 mars 2019 |7 mai 2019 |
|~~WA-GUEST-OS-5.27_201901-01~~ |5 février 2019 |26 mars 2019 |
|~~WA-GUEST-OS-5.26_201812-01~~ |7 janvier 2019 |12 mars 2019 |
|~~WA-GUEST-OS-5.25_201811-01~~ |14 décembre 2018 |5 février 2019 |
|~~WA-GUEST-OS-5.24_201810-01~~ |8 novembre 2018 |7 janvier 2019 |
|~~WA-GUEST-OS-5.23_201809-01~~ |12 octobre 2018 |14 décembre 2018 |

## <a name="family-4-releases"></a>Publications de famille 4
**Windows Server 2012 R2**

.NET Framework installé : 3.5, 4.5.1, 4.5.2

| Chaîne de configuration | Date de publication | Date de désactivation |
| --- | --- | --- |
|  WA-GUEST-OS-4.83_202010-02  |  17 novembre 2020  |  Post 4.85  | 
|  WA-GUEST-OS-4.82_202009-01  |  10 octobre 2020  |  Post 4.84  | 
|~~WA-GUEST-OS-4.81_202008-02~~|  5 septembre 2020  |  17 novembre 2020  | 
|~~WA-GUEST-OS-4.80_202007-01~~|  17 août 2020  |  10 octobre 2020  | 
|~~WA-GUEST-OS-4.79_202006-02~~|  10 août 2020  |  5 septembre 2020  | 
|~~WA-GUEST-OS-4.78_202005-02~~|  2 juin 2020  |  17 août 2020  |  
|~~WA-GUEST-OS-4.77_202004-01~~|  4 mai 2020  |  10 août 2020  |  
|~~WA-GUEST-OS-4.76_202003-01~~|  2 avril 2020  |  2 juin 2020  |  
|~~WA-GUEST-OS-4.75_202002-01~~|  5 mars 2020  |  4 mai 2020  |  
|~~WA-GUEST-OS-4.74_202001-01~~|  24 janvier 2020  |  2 avril 2020  |  
|~~WA-GUEST-OS-4.73_201912-01~~| 8 janvier 2020 | 5 mars 2020 |  
|~~WA-GUEST-OS-4.72_201911-01~~| 5 décembre 2019 | 24 janvier 2020 |  
|~~WA-GUEST-OS-4.71_201910-01~~| 1er novembre 2019 | 8 janvier 2020 |  
|~~WA-GUEST-OS-4.70_201909-01~~| 7 octobre 2019 | 5 décembre 2019 |  
|~~WA-GUEST-OS-4.69_201908-01~~| 4 août 2019 | 1er novembre 2019 |  
|~~WA-GUEST-OS-4.68_201907-01~~| 26 juillet 2019  | 7 octobre 2019 |
|~~WA-GUEST-OS-4.67_201906-01~~| 8 juillet 2019 |4 août 2019 |
|~~WA-GUEST-OS-4.66_201905-01~~ |6 juin 2019 |26 juillet 2019 |
|~~WA-GUEST-OS-4.65_201904-01~~ |7 mai 2019 |8 juillet 2019 |
|~~WA-GUEST-OS-4.64_201903-01~~ |26 mars 2019 |6 juin 2019 |
|~~WA-GUEST-OS-4.63_201902-01~~ |12 mars 2019 |7 mai 2019 |
|~~WA-GUEST-OS-4.62_201901-01~~ |5 février 2019 |26 mars 2019 |
|~~WA-GUEST-OS-4.61_201812-01~~ |7 janvier 2019 |12 mars 2019 |
|~~WA-GUEST-OS-4.60_201811-01~~ |14 décembre 2018 |5 février 2019 |
|~~WA-GUEST-OS-4.59_201810-01~~ |8 novembre 2018 |7 janvier 2019 |
|~~WA-GUEST-OS-4.58_201809-01~~ |12 octobre 2018 |14 décembre 2018 |

## <a name="family-3-releases"></a>Publications de famille 3
**Windows Server 2012**

.NET Framework installé : 3.5, 4.5

| Chaîne de configuration | Date de publication | Date de désactivation |
| --- | --- | --- |
|  WA-GUEST-OS-3.90_202010-02  |  17 novembre 2020  |  Post 3.92  |  
|  WA-GUEST-OS-3.89_202009-01  |  10 octobre 2020  |  Post 3.91  |  
|~~WA-GUEST-OS-3.88_202008-02~~|  5 septembre 2020  |  17 novembre 2020  |  
|~~WA-GUEST-OS-3.87_202007-01~~|  17 août 2020  |  10 octobre 2020  |  
|~~WA-GUEST-OS-3.86_202006-02~~|  10 août 2020  |  5 septembre 2020  |  
|~~WA-GUEST-OS-3.85_202005-02~~|  2 juin 2020  |  17 août 2020  |  
|~~WA-GUEST-OS-3.84_202004-01~~|  4 mai 2020  |  10 août 2020  |  
|~~WA-GUEST-OS-3.83_202003-01~~|  2 avril 2020  |  2 juin 2020  |  
|~~WA-GUEST-OS-3.82_202002-01~~|  5 mars 2020  |  4 mai 2020  |  
|~~WA-GUEST-OS-3.81_202001-01~~|  24 janvier 2020  |  2 avril 2020  |  
|~~WA-GUEST-OS-3.80_201912-01~~| 8 janvier 2020 | 5 mars 2020 |  
|~~WA-GUEST-OS-3.79_201911-01~~| 5 décembre 2019 | 24 janvier 2020 |  
|~~WA-GUEST-OS-3.78_201910-01~~| 1er novembre 2019 | 8 janvier 2020 |  
|~~WA-GUEST-OS-3.77_201909-01~~| 7 octobre 2019 | 5 décembre 2019 |  
|~~WA-GUEST-OS-3.76_201908-01~~|  4 août 2019  |  1er novembre 2019  |  
|~~WA-GUEST-OS-3.75_201907-01~~| 26 juillet 2019 | 7 octobre 2019 |
|~~WA-GUEST-OS-3.74_201906-01~~| 8 juillet 2019 |4 août 2019 |
|~~WA-GUEST-OS-3.73_201905-01~~ |6 juin 2019 |26 juillet 2019 |
|~~WA-GUEST-OS-3.72_201904-01~~ |7 mai 2019 |8 juillet 2019 |
|~~WA-GUEST-OS-3.71_201903-01~~ |26 mars 2019 |6 juin 2019 |
|~~WA-GUEST-OS-3.70_201902-01~~ |12 mars 2019 |7 mai 2019 |
|~~WA-GUEST-OS-3.69_201901-01~~ |5 février 2019 |26 mars 2019 |
|~~WA-GUEST-OS-3.68_201812-01~~ |7 janvier 2019 |12 mars 2019 |
|~~WA-GUEST-OS-3.67_201811-01~~ |14 décembre 2018 |5 février 2019 |
|~~WA-GUEST-OS-3.66_201810-01~~ |8 novembre 2018 |7 janvier 2019 |
|~~WA-GUEST-OS-3.65_201809-01~~ |12 octobre 2018 |14 décembre 2018 |

## <a name="family-2-releases"></a>Publications de famille 2
**Windows Server 2008 R2 SP1**

.NET Framework installé : 3.5 (y compris 2.0 et 3.0), 4.5

| Chaîne de configuration | Date de publication | Date de désactivation |
| --- | --- | --- |
|  WA-GUEST-OS-2.103_202010-02  |  17 novembre 2020  |  Post 2.105  |  
|  WA-GUEST-OS-2.102_202009-01  |  10 octobre 2020  |  Post 2.104  |  
|~~WA-GUEST-OS-2.101_202008-02~~|  5 septembre 2020  |  17 novembre 2020 |    
|~~WA-GUEST-OS-2.100_202007-01~~|  17 août 2020  |  10 octobre 2020  |  
|~~WA-GUEST-OS-2.99_202006-02~~|  10 août 2020  | 5 septembre 2020  |  
|~~WA-GUEST-OS-2.98_202005-02~~|  2 juin 2020  |  17 août 2020  |  
|~~WA-GUEST-OS-2.97_202004-01~~|  4 mai 2020  |  10 août 2020  |  
|~~WA-GUEST-OS-2.96_202003-01~~|  2 avril 2020  |  2 juin 2020  |  
|~~WA-GUEST-OS-2.95_202002-01~~|  5 mars 2020  |  4 mai 2020  |  
|~~WA-GUEST-OS-2.94_202001-01~~|  24 janvier 2020  |  2 avril 2020  |  
|~~WA-GUEST-OS-2.93_201912-01~~| 8 janvier 2020 | 5 mars 2020 |  
|~~WA-GUEST-OS-2.92_201911-01~~| 5 décembre 2019 | 24 janvier 2020 |  
|~~WA-GUEST-OS-2.91_201910-01~~| 1er novembre 2019 | 8 janvier 2020 |  
|~~WA-GUEST-OS-2.90_201909-01~~| 7 octobre 2019 | 5 décembre 2019 |  
|~~WA-GUEST-OS-2.89_201908-01~~| 4 août 2019 | 1er novembre 2019 |  
|~~WA-GUEST-OS-2.88_201907-01~~| 26 juillet 2019 | 7 octobre 2019 |
|~~WA-GUEST-OS-2.87_201906-01~~|8 juillet 2019 | 4 août 2019 |
|~~WA-GUEST-OS-2.86_201905-01~~ |6 juin 2019 |26 juillet 2019 |
|~~WA-GUEST-OS-2.85_201904-01~~ |7 mai 2019 |8 juillet 2019 |
|~~WA-GUEST-OS-2.84_201903-01~~ |26 mars 2019 |6 juin 2019 |
|~~WA-GUEST-OS-2.83_201902-01~~ |12 mars 2019 |7 mai 2019 |
|~~WA-GUEST-OS-2.82_201901-01~~ |5 février 2019 |26 mars 2019 |
|~~WA-GUEST-OS-2.81_201812-01~~ |7 janvier 2019 |12 mars 2019 |
|~~WA-GUEST-OS-2.80_201811-01~~ |14 décembre 2018 |5 février 2019 |
|~~WA-GUEST-OS-2.79_201810-01~~ |8 novembre 2018 |7 janvier 2019 |
|~~WA-GUEST-OS-2.78_201809-01~~ |12 octobre 2018 |14 décembre 2018 |

## <a name="msrc-patch-updates"></a>Mises à jour correctives MSRC
La liste des correctifs inclus dans chaque publication mensuelle du SE invité est disponible [ici][patches].

## <a name="sdk-support"></a>Prise en charge des Kits de développement logiciel (SDK)
La [stratégie de déclassement pour le Kit de développement logiciel (SDK) Azure][retire policy sdk] indique que seules les versions postérieures à la version 2.2 sont prises en charge. Cependant, certaines familles de SE invités permettent l’utilisation de versions antérieures. Vous devez toujours utiliser le Kit de développement logiciel (SDK) pris en charge le plus récent.

| Famille de système d’exploitation invité | Versions de kit SDK compatibles |
| --- | --- |
| 6 |Version 2.9.6+ |
| 5 |Versions 2.9.5.1 et ultérieures |
| 4 |Versions 2.1 et ultérieures |
| 3 |Versions 1.8 et ultérieures |
| 2 |Versions 1.3 et ultérieures |
| 1 |Versions 1.0 et ultérieures |

## <a name="guest-os-release-information"></a>Informations de publication du système d’exploitation invité
Trois dates sont importantes à propos des publications du SE invité : la date de **publication**, la date de **désactivation** et la date **d’expiration**. Un SE invité est considéré comme disponible quand il est proposé dans le portail et qu’il peut être sélectionné comme SE invité cible. Quand un SE invité atteint sa date de **désactivation**, il est supprimé d’Azure. Toutefois, tous les services cloud qui ciblent ce SE invité continuent de fonctionner normalement.

Durant la période entre la date de **désactivation** et la date **d’expiration**, vous pouvez facilement passer de votre SE invité actuel à une version plus récente. Si vous utilisez la mise à jour *automatique* pour votre SE invité, vous disposez toujours de la version la plus récente. Vous n’avez donc pas à vous soucier de la date d’expiration.

Après la date **d’expiration** du SE invité, tous les services cloud qui utilisent encore ce SE invité sont arrêtés, supprimés ou mis à niveau de manière forcée. Des informations supplémentaires sur la stratégie de déclassement sont fournies [ici][retirepolicy].

## <a name="guest-os-family-version-explanation"></a>Informations sur les versions des familles de système d’exploitation invité
Les familles de SE invité sont basées sur les versions publiées de Microsoft Windows Server. Le SE invité est le système d’exploitation sous-jacent sur lequel s’exécute Azure Cloud Services. Chaque SE invité possède un numéro de famille, de version et de publication.

* **Guest OS family**  
  Version du système d’exploitation Windows Server sur laquelle est basé un SE invité. Par exemple, la *famille 3* est basée sur Windows Server 2012.
* **Version de SE invité**  
  Correspond à l’image de la famille de SE invité et aux correctifs [Microsoft Security Response Center (MSRC)][msrc] appropriés qui sont disponibles à la date de production de la nouvelle version du SE invité. Il est possible que les correctifs ne soient pas tous inclus.

    Les numéros commencent à 0 et augmentent de 1 chaque fois qu’un nouvel ensemble de mises à jour est ajouté. Les zéros à droite sont uniquement affichés s’ils sont importants. Autrement dit, la version 2.10 est une version différente, bien plus récente, que la version 2.1.
* **Publication de SE invité**  
  Nouvelle publication d’une version de SE invité. Une nouvelle publication est produite si Microsoft détecte des problèmes pendant les tests nécessitant des modifications. La publication la plus récente remplace toujours les publications précédentes, publiques ou non. Le portail Azure permet uniquement aux utilisateurs de choisir la publication la plus récente pour une version donnée. Les déploiements s’exécutant sur une publication antérieure ne sont généralement pas mis à niveau de force en fonction de la gravité du bogue.

Dans l’exemple ci-dessous, 2 est la famille, 12 est la version et « rel2 » est la publication.

**Publication du SE invité** - 2.12 rel2

**Chaîne de configuration pour cette publication** - WA-GUEST-OS-2.12_201208-02

Ces informations sont intégrées dans la chaîne de configuration d’un SE invité, ainsi qu’une date indiquant les correctifs MSRC envisagés pour cette publication. Dans cet exemple, les correctifs MSRC produits pour Windows Server 2008 R2 jusqu’à août 2012 inclus ont été envisagés pour l’inclusion. Seuls les correctifs s’appliquant spécifiquement à cette version de Windows Server sont inclus. Par exemple, si un correctif logiciel de MSRC s’applique à Microsoft Office, il ne sera pas inclus, car ce produit ne fait pas partie de l’image de base de Windows Server.

## <a name="guest-os-system-update-process"></a>Processus de mise à jour du système d’exploitation invité
Cette page contient des informations sur les prochaines publications de SE invités. Les clients ont indiqué qu’ils souhaitent être informés lors d’une publication, car leurs rôles de service cloud redémarreront s’ils sont définis pour la mise à jour « Automatique ». Les publications de systèmes d’exploitation invités ont généralement lieu 2 à 3 jours au moins après la publication de mise à jour MSRC survenant le deuxième mardi de chaque mois. Les nouvelles publications incluent tous les correctifs MSRC adéquats pour chaque famille de SE invité.

Microsoft Azure publie constamment des mises à jour. Le SE invité n’est qu’une mise à jour parmi d’autres. Une version peut être affectée par de nombreux facteurs trop nombreux pour être répertoriés ici. En outre, Azure s’exécute sur des centaines de milliers d’ordinateurs. Cela signifie qu’il est impossible de donner une date et une heure précises pour le redémarrage de vos rôles. Nous travaillons sur un plan visant à limiter ou minuter les redémarrages.

La propagation complète sur Azure d’une nouvelle publication de SE invité peut prendre du temps. Lorsque les services sont mis à jour pour le nouveau SE invité, ils sont redémarrés en respectant les domaines de mise à jour. Les services définis pour utiliser des mises à jour « Automatiques » obtiennent une publication en premier. Après la mise à jour, la nouvelle version du SE invité est répertoriée pour votre service dans le portail Azure. De nouvelles publications peuvent se produire pendant cette période. Il est possible que certaines versions soient déployées sur de longues périodes et donc que des redémarrages de mise à niveau automatiques ne se produisent pas pendant plusieurs semaines après la date officielle de la publication. Une fois qu’un SE invité est disponible, vous pouvez choisir explicitement cette version à partir du portail ou dans votre fichier de configuration.

Pour consulter un grand nombre d’informations utiles sur les redémarrages et des orientations vers d’autres informations techniques sur les mises à jour de SE invités et hôtes, consultez le blog de MSDN intitulé [Redémarrage d’instance de rôle en raison d’une mise à niveau du SE][restarts].

Si vous procédez à une mise à jour manuelle de votre SE invité, consultez la page [Stratégie de suppression du SE invité][retirepolicy] pour plus d’informations.

## <a name="guest-os-supportability-and-retirement-policy"></a>Stratégie de prise en charge et de suppression du système d’exploitation invité
La prise en charge et la stratégie de suppression du SE invité sont détaillées [ici][retirepolicy].

[cloud updates]: ./cloud-services-update-azure-service.md
[Flux RSS de mise à jour du SE invité]: https://raw.githubusercontent.com/MicrosoftDocs/azure-cloud-services-files/master/GuestOS/GuestOSFeed.xml
[Install .NET on a Cloud Service Role]: ./cloud-services-dotnet-install-dotnet.md?WT.mc_id=azurebg_email_Trans_963_RevisedNET_Update
[Azure Guest OS Update Settings]: cloud-services-how-to-configure-portal.md
[ssl3 announcement]: https://azure.microsoft.com/blog/2014/12/09/azure-security-ssl-3-0-update/
[Microsoft Security Advisory 3009008]: /security-updates/SecurityAdvisories/2015/3009008
[ssl3-fixit]: https://go.microsoft.com/?linkid=9863266
[MS14-066]: /security-updates/SecurityBulletins/2014/ms14-066
[MS14-046]: /security-updates/SecurityBulletins/2014/ms14-046
[retire policy sdk]: /previous-versions/azure/reference/dn479282(v=azure.100)
[server and gos]: https://msdn.microsoft.com/library/dn775043.aspx
[azuresupport]: https://azure.microsoft.com/support/options/
[net install pkg]: https://www.microsoft.com/download/details.aspx?id=42643
[msrc]: https://technet.microsoft.com/security/dn440717.aspx
[update guest os portal]: https://msdn.microsoft.com/library/gg433101.aspx
[update guest os svc]: https://msdn.microsoft.com/library/gg456324.aspx
[restarts]: /archive/blogs/kwill/role-instance-restarts-due-to-os-upgrades
[patches]: cloud-services-guestos-msrc-releases.md
[retirepolicy]: cloud-services-guestos-retirement-policy.md
[fam1retire]: cloud-services-guestos-family1-retirement.md
[fix]: /security-updates/SecurityBulletins/2017/ms17-010
[Windows Azure SDK]: https://www.microsoft.com/en-us/download/details.aspx?id=54917
[plus]: ./applications-dont-support-tls-1-2.md
