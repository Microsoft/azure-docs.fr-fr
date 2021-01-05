---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 03/18/2019
ms.author: alkohli
ms.openlocfilehash: 05673885336dbfb9a70843bd0ccc4e700091ad7e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177732"
---
|     Code d'erreur     |      Description de l'erreur     |
|--------------------|--------------------------|
|    100             | Le nom du conteneur ou du partage doit comporter entre 3 et 63 caractères.|
|    101             | Le nom du conteneur ou du partage doit uniquement comporter des lettres, des chiffres ou des traits d’union.|
|    102             | Le nom du conteneur ou du partage doit uniquement comporter des lettres, des chiffres ou des traits d’union.|
|    103             | Le nom du blob ou du fichier contient des caractères de contrôle non pris en charge.|
|    104             | Le nom du blob ou du fichier contient des caractères de contrôle non autorisés.|
|    105             | Le nom du blob ou du fichier contient trop de segments (chaque segment est séparé par une barre oblique -/).|
|    106             | Le nom du blob ou du fichier est trop long.|
|    107             | L’un des segments du nom de blob ou de fichier est trop long. |
|    108             | La taille du fichier dépasse la taille de fichier maximale autorisée pour le chargement.    |
|    109             | L’alignement du blob ou du fichier est incorrect.  |
|    110             | Le nom du blob ou du fichier encodé au format Unicode n’est pas valide.|
|    111             | Le nom ou le préfixe du fichier ou du blob est un nom réservé qui n’est pas pris en charge (par exemple, COM1).|
|    2000            | Une absence de concordance entre les ETags indique l’existence d’un conflit entre un objet blob de blocs dans le cloud et sur l’appareil. Pour résoudre ce conflit, supprimez soit le fichier figurant dans le cloud, soit celui stocké sur l’appareil.    |
|    2001            | Un problème inattendu s’est produit au cours du traitement d’un fichier après le chargement de ce dernier.    Si cette erreur est signalée et persiste pendant plus de 24 heures, contactez le Support. |
|    2002            | Le fichier est déjà ouvert dans un autre processus et ne peut être chargé tant que le descripteur est ouvert.|
|    2003            | Impossible d’ouvrir le fichier pour le chargement. Si cette erreur est signalée, contactez le Support Microsoft.|
|    2004            | Impossible de se connecter au conteneur pour y charger les données.|
|    2005            | Impossible de se connecter au conteneur, car les autorisations de compte sont erronées ou obsolètes. Vérifiez votre accès.|
|    2006            | Impossible de charger les données dans le compte, car le compte ou le partage sont désactivés.|
|    2007            | Impossible de se connecter au conteneur, car les autorisations de compte sont erronées ou obsolètes. Vérifiez votre accès.|
|    2008            | Impossible d’ajouter de nouvelles données, car le conteneur est plein. Vérifiez les spécifications Azure relatives aux tailles de conteneur prises en charge en fonction du type. Par exemple, le service Azure Files prend uniquement en charge une taille de fichier maximale de 5 To.|
|    2009            | Impossible de charger les données, car le conteneur associé au partage n’existe pas.|    
|    2997            | Une erreur inattendue s’est produite. Il s’agit d’une erreur temporaire qui se résoudra d’elle-même.|
|    2998            | Une erreur inattendue s’est produite. L’erreur peut se résoudre d’elle-même, mais si elle persiste pendant plus de 24 heures, contactez le Support Microsoft.|
|    16000           | Impossible de récupérer ce fichier.|
|    16001           | Impossible de récupérer ce fichier, car il existe déjà sur votre système local.|
|    16002           |Impossible d’actualiser ce fichier, car il n’est pas complètement chargé.|

