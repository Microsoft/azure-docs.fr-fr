---
title: Glossaire d’attachement d’application MSIX Windows Virtual Desktop – Azure
description: Un glossaire des termes et concepts de l’attachement d’application MSIX.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 7132eae073f3d53a104536076ae801ec9ff93e5f
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96518666"
---
# <a name="msix-app-attach-glossary"></a>Glossaire Attachement d’application MSIX

Cet article est une liste de définitions des termes et concepts clés liés à l’attachement d’application MSIX.

## <a name="msix-container"></a>Conteneur MSIX

Un conteneur MSIX est l’emplacement où les applications MSIX sont exécutées. Pour en savoir plus, voir [Conteneurs MSIX](/windows/msix/msix-container).

## <a name="msix-application"></a>Application MSIX 

Une application stockée dans un fichier .MSIX.

## <a name="msix-package"></a>Package MSIX 

Un package MSIX est un fichier ou une application MSIX.

## <a name="msix-share"></a>Partage MSIX

Un partage MSIX est un partage réseau qui contient des packages MSIX étendus. Les partages MSIX prennent en charge SMB 3 ou version ultérieure. Les applications sont transférées à partir de ce partage MSIX sans avoir à déplacer les fichiers d’application sur le lecteur système.

## <a name="msix-image"></a>Image MSIX

Une image MSIX est un fichier VHD, VHDx ou CIM qui contient une ou plusieurs applications MSIX empaquetées. Chaque application est fournie dans l’image MSIX à l’aide de l’outil MSIXMGR.

## <a name="repackage"></a>Repackager

Le repackaging prend une application non-MSIX et la convertit en MSIX à l’aide de l’outil MSIX Packaging Tool (MPT). Pour plus d’informations, consultez [Vue d’ensemble de l’outil MSIX Packaging Tool](/windows/msix/packaging-tool/tool-overview).

## <a name="expand-an-msix-package"></a>Étendre un package MSIX

L’extension d’un package MSIX est un processus en plusieurs étapes. Elle prend le fichier MSIX et place son contenu dans un fichier VHD(x) ou CIM. 

Pour étendre un package MSIX :

1. Prenez un package MSIX (fichier MSIX).
2. Renommez le fichier MSIX en fichier .zip.
3. Décompressez le fichier. zip résultant dans un dossier.
4. Créez un disque dur virtuel de la même taille que le dossier.
5. Montez le disque dur virtuel.
6. Initialisez un disque.
7. Créez une partition.
8. Formatez la partition.
9. Copiez le contenu décompressé dans le disque dur virtuel.
10. Utilisez l’outil MSIXMGR pour appliquer des listes de contrôle d’accès (ACL, access-control list) au contenu du disque dur virtuel.
11. Démontez le fichier VHD(x) ou [CIM](#cim).

## <a name="upload-an-msix-package"></a>Charger un package MSIX 

Le chargement d’un package MSIX implique le chargement du fichier VHD(x) ou [CIM](#cim) qui contient un package MSIX étendu sur le partage MSIX.

Dans Windows Virtual Desktop, les chargements se produisent une fois par partage MSIX. Une fois que vous avez chargé un package, tous les pools d’hôtes dans le même abonnement peuvent y faire référence.

## <a name="add-an-msix-package"></a>Ajouter un package MSIX

Dans Windows Virtual Desktop, l’ajout d’un package MSIX le lie à un pool d’hôtes.

## <a name="publish-an-msix-package"></a>Publier un package MSIX 

Dans Windows Virtual Desktop, un package MSIX publié doit être attribué à un utilisateur ou groupe d’utilisateurs Active Directory Domain Services (AD DS) ou Azure Active Directory (Azure AD).

## <a name="staging"></a>Staging

La mise en lots implique deux choses :

- Le montage du VHD(x) ou [CIM](#cim) sur la machine virtuelle.
- La notification au système d’exploitation que le package MSIX est disponible pour l’inscription.

## <a name="registration"></a>Inscription

L’inscription rend un package MSIX mis en lots disponible pour vos utilisateurs. L’inscription se fait par utilisateur. Si vous n’avez pas explicitement inscrit une application pour cet utilisateur spécifique, ce dernier ne pourra pas exécuter l’application.

Il existe deux types d’inscription : normale et différée.

### <a name="regular-registration"></a>Inscription normale

Dans l’inscription normale, chaque application attribuée à un utilisateur est entièrement inscrite. L’inscription se produit au moment où l’utilisateur se connecte à la session, ce qui peut avoir un impact sur le temps nécessaire pour commencer à utiliser Windows Virtual Desktop.

### <a name="delayed-registration"></a>Inscription différée

Dans une inscription différée, chaque application attribuée à l’utilisateur n’est inscrite que partiellement. Inscription partielle signifie que la vignette du menu Démarrer et les associations de fichiers de double-clic sont inscrites. L’inscription se produit lorsque l’utilisateur se connecte à sa session, de sorte qu’elle a un impact minimal sur le temps nécessaire pour commencer à utiliser Windows Virtual Desktop. L’inscription se termine uniquement lorsque l’utilisateur exécute l’application dans le package MSIX.

L’inscription différée est actuellement la configuration par défaut pour l’attachement d’application MSIX.

## <a name="deregistration"></a>Désinscription

La désinscription supprime un package MSIX inscrit, mais qui n’est pas en cours d’exécution pour un utilisateur. La désinscription se produit lorsque l’utilisateur se déconnecte de sa session. Pendant la désinscription, l’attachement d’application MSIX transmet les données d’application spécifiques à l’utilisateur au profil utilisateur local.

## <a name="destage"></a>Retrait

Le retrait indique au système d’exploitation qu’un package ou une application MSIX qui n’est pas en cours d’exécution et qui n’est pas mis en lots pour un utilisateur peut être démonté. Cela supprime toutes les références à ce dernier dans le système d’exploitation.

## <a name="cim"></a>CIM

.CIM est une nouvelle extension de fichier associée à Composite Image Files System (CimFS). Le montage et le démontage des fichiers CIM sont plus rapides que ceux des fichiers VHD. Un fichier CIM consomme également moins d’UC et de mémoire qu’un fichier VHD.

Un fichier CIM est un fichier avec une extension .CIM qui contient des métadonnées et au moins six fichiers supplémentaires incluant des données réelles. Les fichiers contenus dans le fichier CIM n’ont pas d’extensions. Le tableau suivant répertorie des exemples de fichiers que vous pourriez trouver dans un fichier CIM :

| Nom de fichier | Extension | Taille |
|-----------|-----------|------|
| VSC | CIM | 1 Ko |
| objectid_b5742e0b-1b98-40b3-94a6-9cb96f497e56_0 | N/D | 27 ko |
| objectid_b5742e0b-1b98-40b3-94a6-9cb96f497e56_1 | N/D | 20 Ko |
| objectid_b5742e0b-1b98-40b3-94a6-9cb96f497e56_2 | N/D | 42 ko |
| region_b5742e0b-1b98-40b3-94a6-9cb96f497e56_0 | N/D | 428 ko |
| region_b5742e0b-1b98-40b3-94a6-9cb96f497e56_1 | N/D | 217 ko |
| region_b5742e0b-1b98-40b3-94a6-9cb96f497e56_2 | N/D | 264 132 ko |

Le tableau suivant est une comparaison des performances entre VHD et CimFS. Ces chiffres sont le résultat d’une série de tests avec 500 fichiers de 300 Mo pour chaque format exécutés sur une machine DSv4.

|  Spécifications                          | Disque dur virtuel (VHD)                    | CimFS   |
|---------------------------------|--------------------------|-----------|
| Temps de montage moyen     | 356 ms                     | 255 ms      |
| Temps de démontage moyen   | 1615 ms                    | 36 ms       |
| Consommation de mémoire | 6 % (de 8 Go)                      | 2 % (de 8 Go)       |
| UC (nombre de pics d’utilisation)          | Maximum atteint plusieurs fois | Aucun impact |

## <a name="next-steps"></a>Étapes suivantes

Si vous souhaitez en savoir plus sur l’attachement d’application MSIX, consultez notre [page de présentation](what-is-app-attach.md) et notre [FAQ](app-attach-faq.md). Sinon, commencez avec [Configurer l’attachement d’application](app-attach.md).
