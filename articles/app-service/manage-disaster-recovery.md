---
title: Reprendre l’activité après une défaillance à l’échelle d’une région
description: Découvrez comment Azure App Service vous permet d’assurer les fonctions de continuité des activités et de reprise d’activité après sinistre (BCDR). Récupérez votre application après une défaillance à l’échelle d’une région dans Azure.
ms.topic: how-to
ms.date: 06/09/2020
ms.custom: subject-moving-resources
ms.openlocfilehash: 1388dc11254324f74efcbaa55c97cac2ccd0c026
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87073733"
---
# <a name="move-an-app-service-app-to-another-region"></a>Déplacer une application App Service vers une autre région

Cet article explique comment remettre en ligne des ressources App Service dans une autre région Azure lors d’un sinistre qui affecte l’ensemble d’une région Azure. Quand un incident met l’ensemble d’une région Azure hors connexion, toutes les applications App Service hébergées dans cette région sont placées en mode de reprise d’activité après sinistre. Des fonctionnalités sont disponibles pour vous permettre de restaurer l’application dans une autre région ou de récupérer des fichiers à partir de l’application impactée.

Les ressources App Service sont spécifiques à une région et ne peuvent pas être déplacées d’une région à l’autre. Vous devez restaurer l’application dans une nouvelle application dans une autre région, puis créer des configurations de mise en miroir ou des ressources pour la nouvelle application.

## <a name="prerequisites"></a>Prérequis

- Aucun. La [restauration à partir d’une capture instantanée](app-service-web-restore-snapshots.md) exige généralement le niveau **Premium**, mais en mode de reprise d’activité après sinistre, elle est automatiquement activée pour votre application impactée, quel que soit le niveau dans lequel se trouve celle-ci.

## <a name="prepare"></a>Préparation

Identifiez toutes les ressources App Service que l’application impactée utilise actuellement. Par exemple :

- Applications App Service
- [Plans App Service](overview-hosting-plans.md)
- [Emplacements de déploiement](deploy-staging-slots.md)
- [Domaines personnalisés achetés dans Azure](manage-custom-dns-buy-domain.md)
- [Certificats SSL](configure-ssl-certificate.md)
- [Intégration de Réseau virtuel Microsoft Azure](web-sites-integrate-with-vnet.md)
- [Connexions hybrides](app-service-hybrid-connections.md)
- [Identités managées](overview-managed-identity.md)
- [Paramètres de sauvegarde](manage-backup.md)

Certaines ressources, telles que les certificats importés ou les connexions hybrides, contiennent une intégration avec d’autres services Azure. Pour plus d’informations sur la façon de déplacer ces ressources d’une région à l’autre, consultez la documentation des services respectifs.

## <a name="restore-app-to-a-different-region"></a>Restaurer une application dans une autre région

1. Créez une application App Service dans une *autre* région Azure que celle de l’application impactée. Il s’agit de l’application cible dans le scénario de reprise d’activité après sinistre.

1. Dans le [portail Azure](https://portal.azure.com), accédez à la page de gestion de l’application impactée. Dans une région Azure défaillante, l’application impactée affiche un texte d’avertissement. Cliquez sur le texte d’avertissement.

    ![Capture d’écran de la page de l’application concernée. Une notification d’avertissement est visible qui décrit la situation et fournit un lien pour restaurer l’application.](media/manage-disaster-recovery/restore-start.png)

1. Dans la page **Restaurer la sauvegarde**, configurez l’opération de restauration conformément au tableau suivant. Lorsque vous avez terminé, cliquez sur **OK**.

   | Paramètre | Valeur | Description |
   |-|-|-|
   | **Capture instantanée (préversion)** | Sélectionnez une capture instantanée. | Les deux captures instantanées les plus récents sont disponibles. |
   | **Destination de la restauration** | **Application existante** | Cliquez sur la remarque ci-dessous, qui indique **Cliquez ici pour changer l’application de destination de la restauration**, puis sélectionnez l’application cible. Dans un scénario de sinistre, vous pouvez uniquement restaurer la capture instantanée dans une application se trouvant dans une autre région Azure. |
   | **Restaurer la configuration du site** | **Oui** | |

    ![Capture d’écran de la page Restaurer la sauvegarde. Un instantané spécifique, les options répertoriées dans le tableau précédent et le bouton OK sont mis en surbrillance.](media/manage-disaster-recovery/restore-configure.png)

3. Configurez [tout le reste](#prepare) dans l’application cible pour mettre en miroir l’application impactée, puis vérifiez votre configuration.

4. Lorsque vous êtes prêt pour que le domaine personnalisé soit dirigé vers l’application cible, [remappez le nom de domaine](manage-custom-dns-migrate-domain.md#remap-the-active-dns-name).

## <a name="recover-app-content-only"></a>Uniquement récupérer le contenu de l’application

Si vous voulez uniquement récupérer les fichiers de l’application impactée sans la restaurer, effectuez les étapes suivantes :

1. Dans le [portail Azure](https://portal.azure.com), accédez à la page de gestion de l’application impactée, puis cliquez sur **Obtenir le profil de publication**.

    ![Capture d’écran de la page de l’application concernée. Une notification d’avertissement est visible, mais elle n’est pas mise en surbrillance. L’élément Obtenir le profil de publication est mis en surbrillance à la place.](media/manage-disaster-recovery/get-publish-profile.png)

1. Ouvrez le fichier téléchargé, puis recherchez le profil de publication dont le nom contient `ReadOnly - FTP`. Il s’agit du profil de reprise d’activité. Par exemple :

    ```xml
    <publishProfile profileName="%app-name% - ReadOnly - FTP" publishMethod="FTP" publishUrl="ftp://%ftp-site%/site/wwwroot" ftpPassiveMode="True" userName="%app-name%\$%app-name%" userPWD="" destinationAppUrl="http://%app-name%.azurewebsites.net" SQLServerDBConnectionString="" mySQLDBConnectionString="" hostingProviderForumLink="" controlPanelLink="http://windows.azure.com" webSystem="WebSites">
        <databases />
    </publishProfile>
    ```
    
    Copiez trois valeurs d’attribut : 
        
    - `publishUrl` : nom d’hôte FTP
    - `userName` et `userPWD` : informations d’identification FTP

1. Utilisez le client FTP de votre choix, puis connectez-vous à l’hôte FTP de l’application impactée à l’aide du nom d’hôte et des informations d’identification.

1. Une fois connecté, téléchargez l’intégralité du dossier */site/wwwroot*. La capture d’écran suivante montre comment vous effectuez le téléchargement dans [FileZilla](https://filezilla-project.org/).

    ![Capture d’écran d’une hiérarchie de fichiers FileZilla. Le dossier wwwroot est mis en surbrillance et son menu contextuel est visible. Dans ce menu, Télécharger est mis en surbrillance.](media/manage-disaster-recovery/download-content.png)

## <a name="next-steps"></a>Étapes suivantes
[Restaurer une application dans Azure à partir d’une capture instantanée](app-service-web-restore-snapshots.md)
