---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 3467a5d5daa300f82c7b81641ab7e262259d9285
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95553368"
---
### <a name="upgrade-sharepoint-2010-to-sharepoint-2013-and-then-install-the-storsomple-adapter-for-sharepoint"></a>Effectuez une mise à niveau de SharePoint 2010 vers SharePoint 2013, puis installez l'adaptateur StorSimple pour SharePoint
> [!IMPORTANT]
> Les fichiers précédemment déplacés vers le stockage externe via RBS ne sont pas disponibles, jusqu'à ce que la mise à niveau soit terminée et la fonctionnalité RBS réactivée. Pour limiter l'impact sur l'utilisateur, effectuez la mise à niveau ou la réinstallation pendant une fenêtre de maintenance planifiée.
> 
> 

#### <a name="to-upgrade-sharepoint-2010-to-sharepoint-2013-and-then-install-the-adapter"></a>Pour mettre à niveau SharePoint 2010 vers SharePoint 2013, puis installer l'adaptateur
1. Dans la batterie de serveurs SharePoint 2010, notez le chemin d'accès du magasin d'objets blob pour les objets blob externalisés et les bases de données de contenu pour lesquelles RBS est activé. 
2. Installez et configurez la batterie de serveurs SharePoint 2013. 
3. Déplacez les bases de données, les applications et les collections de sites de la batterie de serveurs SharePoint 2010 vers la batterie de serveurs SharePoint 2013. Pour obtenir des instructions, consultez la [Vue d’ensemble du processus de mise à niveau vers SharePoint 2013](/SharePoint/upgrade-and-update/overview-of-the-upgrade-process).
4. Installez l’adaptateur StorSimple pour SharePoint sur la nouvelle batterie de serveurs. Accédez à [Installation de l’adaptateur StorSimple pour SharePoint](#install-the-storsimple-adapter-for-sharepoint) pour plus d’informations.
5. En utilisant les informations que vous avez notées à l'étape 1, activez RBS pour le même ensemble de bases de données de contenu et fournissez le même chemin d'accès du magasin d’objets blob utilisé dans l'installation de SharePoint 2010. Accédez à [Configuration de RBS](#configure-rbs) pour plus d’informations sur les procédures. Après avoir terminé cette étape, les fichiers précédemment externalisés doivent être accessibles depuis la nouvelle batterie de serveurs. 

### <a name="upgrade-the-storsimple-adapter-for-sharepoint"></a>Mise à niveau de l’adaptateur StorSimple pour SharePoint
> [!IMPORTANT]
> Vous devez planifier cette mise à niveau pour qu’elle se produise pendant une fenêtre de maintenance planifiée pour les raisons suivantes :
> 
> * Le contenu précédemment externalisé n’est pas disponible avant la réinstallation de l'adaptateur.
> * Tout contenu téléchargé sur le site après la désinstallation de la version précédente de l'adaptateur StorSimple pour SharePoint, mais avant l'installation de la nouvelle version, est stocké dans la base de données de contenu. Vous devez déplacer ce contenu vers l’appareil StorSimple après l'installation du nouvel adaptateur. Vous pouvez utiliser la cmdlet PowerShell `RBS Migrate()` Microsoft incluse avec SharePoint pour migrer le contenu. Pour plus d'informations, consultez [Migration du contenu vers ou à partir de RBS](/previous-versions/office/sharepoint-foundation-2010/ff628255(v=office.14)). 
> 
> 

#### <a name="to-upgrade-the-storsimple-adapter-for-sharepoint"></a>Pour mettre à niveau l’adaptateur StorSimple pour SharePoint
1. Désinstallez la version précédente de l'adaptateur StorSimple pour SharePoint.
   
   > [!NOTE]
   > Cela désactive automatiquement RBS sur les bases de données de contenu. Toutefois, les objets blob existants restent sur l’appareil StorSimple. Étant donné que RBS est désactivé et que les objets blob n'ont pas été transférés à nouveau sur les bases de données de contenu, toutes les demandes concernant ces objets blob échouent. 
   > 
   > 
2. Installation du nouvel adaptateur StorSimple pour SharePoint. Le nouvel adaptateur reconnaît automatiquement les bases de données de contenu précédemment activées ou désactivées pour RBS et utilise les paramètres précédents.