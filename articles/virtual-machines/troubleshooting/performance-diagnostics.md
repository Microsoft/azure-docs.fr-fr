---
title: Diagnostics de performances pour les machines virtuelles Azure| Microsoft Docs
description: Présente les diagnostics de performances Azure pour Windows.
services: virtual-machines-windows'
documentationcenter: ''
author: anandhms
manager: cshepard
editor: przlplx
tags: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 9/20/2018
ms.author: anandh
ms.openlocfilehash: 04b70e593e8b1bee8beb72ac88bc8441bc38bb9a
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91963242"
---
# <a name="performance-diagnostics-for-azure-virtual-machines"></a>Diagnostics de performances pour les machines virtuelles Azure

L’outil de diagnostics de performances vous permet de résoudre les problèmes de performances qui peuvent affecter une machine virtuelle Windows ou Linux. Les scénarios de résolution des problèmes pris en charge incluent les vérifications rapides des problèmes connus et bonnes pratiques ainsi que des problèmes complexes qui impliquent le ralentissement des performances de la machine virtuelle ou une utilisation élevée du processeur, de l’espace disque ou de la mémoire.

Vous pouvez exécuter les diagnostics de performances directement depuis le portail Azure, où vous pouvez également examiner les insights et un rapport sur différents journaux d’activité, la configuration riche et les données de diagnostic. Nous vous recommandons d’exécuter les diagnostics de performances et de passer en revue les insights et données de diagnostic avant de contacter le support Microsoft.

> [!NOTE]
> Pour Windows, les diagnostics de performances sont actuellement pris en charge sur les machines virtuelles sur lesquelles le kit SDK .NET version 4.5 ou ultérieure est installé. Pour obtenir la procédure permettant d’exécuter les diagnostics de performances sur les machines virtuelles classiques, consultez [Extension de machine virtuelle des diagnostics de performances Azure](performance-diagnostics-vm-extension.md).

## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

### <a name="windows"></a>Windows

* Windows Server 2019
* Windows Server 2016
* Windows Server 2012 R2
* Windows Server 2012
* Windows Server 2008 R2
* Windows 10
* Windows 8.1
* Windows 8

### <a name="linux"></a>Linux

- Les distributions prises en charge sont les suivantes :

    | Distribution               | Version                                         |
    |----------------------------|-------------------------------------------------|
    | Oracle Linux Server        | 6.10 [`*`], 7.3, 7.6, 7.5 |
    | CentOS                     | 6.5 [`*`], 7.6                                    |
    | RHEL                       | 7.2, 7.5, 8.0 [`*`]                               |
    | Ubuntu                     | 14.04, 16.04, 18.04, 20.04                               |
    | Debian                     | 8, 9, 10 [`*`]                                    |
    | SLES                       | 12 SP4 [`*`]                                      |
    |                            |                                                   |

>[!Note]
>[`*`] Reportez-vous aux [Problèmes connus](how-to-use-perfinsights-linux.md#known-issues)

## <a name="install-and-run-performance-diagnostics-on-your-vm"></a>Installer et exécuter les diagnostics de performances sur votre machine virtuelle

Les diagnostics de performances installent une extension de machine virtuelle qui exécute un outil de diagnostics nommé PerfInsights. PerfInsights est disponible à la fois sur [Windows](./how-to-use-perfinsights.md) et sur [Linux](./how-to-use-perfinsights-linux.md). Pour installer et exécuter les diagnostics de performances, effectuez les étapes suivantes :

1. Dans la colonne de commandes de gauche, sélectionnez **Machines virtuelles**.
1. Dans la liste des noms de machine virtuelle, sélectionnez la machine virtuelle sur laquelle vous souhaitez exécuter les diagnostics.
1. Dans la colonne de commandes de droite, sélectionnez **Diagnostics de performances**.

    ![Capture d’écran du portail Azure, avec le bouton Installer les diagnostics de performances mis en surbrillance](media/performance-diagnostics/performance-diagnostics-install.png)

    > [!NOTE]
    > Dans cette capture d’écran, le panneau des noms de machine virtuelle est masqué.
1. Sélectionnez un compte de stockage (facultatif).

    Si vous souhaitez utiliser un seul compte de stockage pour stocker les résultats des diagnostics de performances de plusieurs machines virtuelles, vous pouvez sélectionner un compte de stockage en cliquant sur le bouton **Paramètres** dans la barre d’outils. Cliquez sur le bouton **OK** une fois que vous avez sélectionné le compte de stockage.

    Si vous ne spécifiez pas un compte de stockage, un compte de stockage sera créé par défaut.

    ![Capture d’écran du panneau Diagnostics de performances avec le bouton de barre d’outils Paramètres mis en surbrillance](media/performance-diagnostics/settings-button.png)

    ![Capture d’écran de la sélection de compte de stockage à partir du panneau des paramètres de diagnostics de performances](media/performance-diagnostics/select-storage-account.png)

1. Sélectionnez le bouton **Installer les diagnostics de performances**.
1. Cochez la case **Exécuter les diagnostics** si vous souhaitez exécuter un diagnostic une fois l’installation terminée. Si vous faites ce choix, vous serez en mesure de sélectionner le scénario d’analyse des performances et les options associées.

    ![Capture d’écran du bouton Installer les diagnostics de performances](media/performance-diagnostics/install-diagnostics-button.png)

## <a name="select-an-analysis-scenario-to-run"></a>Sélectionner un scénario d’analyse à exécuter

Les scénarios d’analyse suivants sont disponibles à partir du portail Azure. Sélectionnez une analyse, en fonction du problème de performances que vous rencontrez. Sélectionnez les options de durée et de trace en fonction des besoins pour l’analyse.

* **Analyse rapide des performances**  
    Recherche les problèmes connus, analyse les bonnes pratiques et collecte des données de diagnostic. L’exécution de cette analyse prend plusieurs minutes. En savoir plus sur [Windows](./how-to-use-perfinsights.md) ou [Linux](./how-to-use-perfinsights-linux.md)

* **Analyse des performances**  
    Inclut toutes les vérifications de l’analyse rapide des performances et supervise la consommation élevée des ressources. Utilisez cette version pour résoudre les problèmes de performances d’ordre général, notamment l’utilisation élevée du processeur, de la mémoire et du disque. Cette analyse prend de 30 secondes à 15 minutes, selon la durée sélectionnée. En savoir plus sur [Windows](./how-to-use-perfinsights.md) ou [Linux](./how-to-use-perfinsights-linux.md)

* **Analyse avancée des performances** `*`  
    Inclut toutes les vérifications de l’analyse des performances et collecte une ou plusieurs des traces, comme indiqué dans les sections suivantes. Ce scénario permet de résoudre les problèmes complexes qui nécessitent des traces supplémentaires. L’exécution de ce scénario sur de plus longues périodes augmente la taille globale de la sortie de diagnostics, selon la taille de la machine virtuelle et les options de trace sélectionnées. L’exécution de cette analyse prend de 30 secondes à 15 minutes, selon la durée sélectionnée. [En savoir plus](./how-to-use-perfinsights.md)

* **Analyse de fichiers Azure** `*`  
    Inclut toutes les vérifications de l’analyse des performances et capture une trace réseau et des compteurs SMB. Utilisez ce scénario pour résoudre les problèmes de performances des fichiers Azure. L’exécution de cette analyse prend de 30 secondes à 15 minutes, selon la durée sélectionnée. [En savoir plus](./how-to-use-perfinsights.md)

>[!Note]
>[`*`] Ces scénarios d’analyse sont uniquement pris en charge sur Windows.

![Capture d’écran du volet Exécuter les diagnostics dans le panneau Diagnostics de performances](media/performance-diagnostics/run-diagnostics-pane.png)

### <a name="provide-symptoms-optional"></a>Fournir des symptômes (facultatif)

Sélectionnez des symptômes présélectionnés dans la liste ou ajoutez-en de nouveaux. Ceci nous permet d’améliorer l’analyse à l’avenir.

### <a name="provide-support-request-number-if-available-optional"></a>Fournir le numéro de demande de support, s’il est disponible (facultatif)

Si vous travaillez avec un ingénieur du support Microsoft sur un ticket de support existant, indiquez le numéro du ticket de support.

### <a name="review-the-privacy-policy-and-legal-terms-and-select-the-check-box-to-acknowledge-required"></a>Passer en revue la politique de confidentialité et les conditions juridiques, puis cocher la case pour les accepter (obligatoire)

Pour exécuter les diagnostics, vous devez accepter les conditions juridiques et la politique de confidentialité.

### <a name="select-ok-to-run-the-diagnostics"></a>Sélectionner OK pour exécuter les diagnostics

Une notification s’affiche au début de l’installation des diagnostics de performances. Une fois l’installation terminée, vous voyez une notification indiquant que l’installation a réussi. L’analyse sélectionnée est ensuite exécutée pendant la durée spécifiée. Il est alors judicieux de reproduire le problème de performances afin que les données de diagnostic puissent être capturées au moment approprié.

Une fois l’analyse terminée, les éléments suivants sont chargés sur les tables Azure et un conteneur d’objets blob dans le compte de stockage spécifié :

* Tous les insights et informations associées sur l’exécution
* Un fichier compressé (.zip) de sortie (appelé **PerformanceDiagnostics_yyyy-MM-dd_hh-mm-ss-fff.zip** ) sur Windows et un fichier tar ball (appelé **PerformanceDiagnostics_yyyy-MM-dd_hh-mm-ss-fff.tar.gz** ) sur Linux qui contient les fichiers journaux
* Un rapport HTML

Après le chargement, un nouveau rapport de diagnostics est répertorié dans le portail Azure.

![Capture d’écran d’une liste de rapports de diagnostics dans le panneau Diagnostics de performances](media/performance-diagnostics/diagnostics-report-list.png)

## <a name="how-to-change-performance-diagnostics-settings"></a>Comment modifier les paramètres des diagnostics de performances

Utilisez le bouton de barre d’outils **Paramètres** pour modifier le compte de stockage où les insights et la sortie des diagnostics peuvent être stockés. Vous pouvez utiliser le même compte de stockage pour plusieurs machines virtuelles qui utilisent les diagnostics de performances. Quand vous changez le compte de stockage, les anciens rapports et insights ne sont pas supprimés. Toutefois, ils ne s’affichent plus dans la liste de rapports de diagnostics.

## <a name="review-insights-and-performance-diagnostics-report"></a>Passer en revue les insights et le rapport de diagnostics de performances

Chaque exécution de diagnostics contient une liste d’insights et de recommandations, des ressources affectées, des fichiers journaux et d’autres informations de diagnostics riches qui sont collectées, ainsi qu’un rapport pour une lecture hors connexion. Pour obtenir une liste complète de toutes les données de diagnostic collectées, consultez **Quelles informations sont collectées par PerfInsights ?** sur [Windows](how-to-use-perfinsights.md#what-kind-of-information-is-collected-by-perfinsights) ou [Linux](how-to-use-perfinsights-linux.md#what-kind-of-information-is-collected-by-perfinsights).

### <a name="select-a-performance-diagnostics-report"></a>Sélectionner un rapport de diagnostics de performances

Vous pouvez utiliser la liste de rapports de diagnostics pour rechercher tous les rapports de diagnostics qui ont été exécutés. La liste inclut des informations sur l’analyse utilisée ainsi que les insights trouvés et leurs niveaux d’impact. Sélectionnez une ligne pour afficher plus de détails.

![Capture d’écran de la sélection d’un rapport de diagnostics dans le panneau Diagnostics de performances](media/performance-diagnostics/select-report.png)

### <a name="review-a-performance-diagnostics-report"></a>Passer en revue un rapport de diagnostics de performances

Chaque rapport de diagnostics de performances peut contenir plusieurs insights et indiquer un niveau d’impact élevé, moyen ou faible. Chaque insight contient également des recommandations pour atténuer le problème. Les insights sont regroupés pour faciliter le filtrage.

Les niveaux d’impact représentent le risque de problèmes de performances en fonction de facteurs comme une configuration incorrecte, des problèmes connus ou des problèmes signalés par d’autres utilisateurs. Vous n’êtes peut-être pas encore confronté à un ou plusieurs des problèmes répertoriés. Par exemple, vous pouvez avoir des fichiers de base de données et des fichiers journaux SQL sur le même disque de données. Cette condition présente un risque élevé de goulots d’étranglement et d’autres problèmes de performances si l’utilisation de la base de données est élevée, alors que vous pouvez ne rien remarquer si l’utilisation est faible.

![Capture d’écran du panneau Vue d’ensemble du rapport de diagnostics de performances](media/performance-diagnostics/performance-diagnostics-report-overview.png)

### <a name="reviewing-performance-diagnostics-insights-and-recommendations"></a>Examen des recommandations et insights associés aux diagnostics de performances

Vous pouvez sélectionner un insight pour afficher plus de détails sur les ressources affectées, les atténuations suggérées et les liens de référence.

![Capture d’écran du détail d’un insight associé aux diagnostics de performances](media/performance-diagnostics/insight-detail.png)

### <a name="download-and-review-the-full-performance-diagnostics-report"></a>Télécharger et passer en revue l’intégralité du rapport de diagnostics de performances

Vous pouvez utiliser le bouton **Télécharger le rapport** pour télécharger un rapport HTML qui contient des informations de diagnostics riches supplémentaires, telles que le stockage et la configuration réseau, les compteurs de performances, les traces, la liste de processus et les journaux d’activité. Le contenu dépend de l’analyse sélectionnée. Pour une résolution des problèmes avancée, le rapport peut contenir des informations supplémentaires et des diagrammes interactifs qui sont liés à l’utilisation élevée du processeur, à l’utilisation élevée du disque et aux processus qui consomment trop de mémoire. Pour plus d’informations sur le rapport de diagnostics de performances, consultez [Windows](how-to-use-perfinsights.md#review-the-diagnostics-report) ou [Linux](how-to-use-perfinsights-linux.md#review-the-diagnostics-report).

## <a name="manage-performance-diagnostics-reports"></a>Gérer les rapports de diagnostics de performances

Vous pouvez supprimer un ou plusieurs rapports de diagnostics de performances à l’aide du bouton **Supprimer le rapport**.

## <a name="how-to-uninstall-performance-diagnostics"></a>Comment désinstaller les diagnostics de performances

Vous pouvez désinstaller les diagnostics de performances d’une machine virtuelle. Cette action supprime l’extension de machine virtuelle, mais n’affecte pas les données de diagnostic qui se trouvent dans le compte de stockage.

![Capture d’écran de la barre d’outils du panneau Diagnostics de performances avec le bouton Désinstaller mis en surbrillance](media/performance-diagnostics/uninstal-button.png)

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="where-is-the-diagnostics-data-from-my-vm-stored"></a>Où sont stockées les données de diagnostic de ma machine virtuelle

Tous les rapports et insights de diagnostics de performances sont stockés dans votre propre compte de stockage. Les insights sont stockés dans des tables Azure. Le fichier compressé des rapports est stocké dans un conteneur d’objets blob nommé azdiagextnresults.

Vous pouvez afficher les informations de compte de stockage en utilisant le bouton Paramètres dans la barre d’outils.

### <a name="how-do-i-share-this-data-with-microsoft-customer-support"></a>Comment partager ces données avec le support technique de Microsoft

Il existe plusieurs façons de partager le rapport de diagnostics avec Microsoft.

**Option 1 :** partager automatiquement le dernier rapport  
Quand vous ouvrez un ticket de support auprès de Microsoft, il est important de partager le rapport de diagnostics de performances. Si vous avez choisi de partager ces informations avec Microsoft pendant que vous exécutez les diagnostics (en cochant la case **I agree to share diagnostics information with Microsoft** [J’accepte de partager des informations de diagnostic avec Microsoft]), Microsoft pourra accéder au rapport à partir de votre compte de stockage à l’aide d’un lien SAP vers le fichier zip de sortie pendant 30 jours à partir de la date d’exécution. Seul le dernier rapport est disponible pour l’ingénieur du support.

**Option 2 :** générer une signature d’accès partagé pour le fichier compressé du rapport de diagnostics  
Vous pouvez partager un lien vers le fichier compressé des rapports à l’aide de signatures d’accès partagé. Pour ce faire, procédez comme suit :

1. Dans le portail Azure, accédez au compte de stockage dans lequel sont stockées les données de diagnostic.
1. Sélectionnez **Objets blob** sous la section **Service Blob**.
1. Sélectionnez le conteneur **azdiagextnresults**.
1. Sélectionnez le fichier compressé de sortie du rapport de diagnostics de performances que vous souhaitez partager.
1. Sous l’onglet **Générer une signature d’accès partagé**, sélectionnez les critères de partage.
1. Cliquez sur **Générer un jeton et une URL SAP d’objet blob**.
1. Copiez **l’URL SAP d’objet blob** et partagez-la avec l’ingénieur du support.

**Option 3 :** télécharger le rapport à partir du compte de stockage

Vous pouvez également localiser le fichier compressé du rapport de diagnostics de performances à l’aide des étapes 1 à 4 de l’option 2. Choisissez de télécharger le fichier, puis partagez-le par e-mail, ou demandez les instructions pour charger le fichier à l’ingénieur du support.  

### <a name="how-do-i-capture-the-diagnostics-data-at-the-correct-time"></a>Comment capturer les données de diagnostic au bon moment

Chaque exécution des diagnostics de performances comporte deux phases :

1. Installer ou mettre à jour l’extension de machine virtuelle des diagnostics de performances.
1. Exécuter les diagnostics pendant la durée spécifiée.

Pour l’instant, il n’existe aucun moyen facile de savoir exactement quand l’installation de l’extension de machine virtuelle est terminée. En règle générale, cette installation prend environ entre 45 secondes et 1 minute. Une fois l’extension de machine virtuelle installée, vous pouvez exécuter vos étapes de reproduction afin que les diagnostics de performances capturent le jeu correct de données pour la résolution des problèmes.

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez examiné les rapports et insights de diagnostics de performances, si vous ne pouvez toujours pas déterminer la cause du problème et avez besoin d’aide, vous pouvez ouvrir un ticket de support auprès du support technique de Microsoft.

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [Site du support Azure](https://azure.microsoft.com/support/options/), puis cliquez sur **Obtenir un support**. Pour plus d’informations sur le support Azure, lisez la [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).