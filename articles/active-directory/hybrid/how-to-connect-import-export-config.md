---
title: Comment importer et exporter des paramètres de configuration Azure AD Connect
description: Cet article décrit les questions fréquemment posées sur le provisionnement cloud.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/13/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: da80af9fe598186fa25d59601c9fa4faccb4286a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87447040"
---
# <a name="import-and-export-azure-ad-connect-configuration-settings-public-preview"></a>Importer et exporter des paramètres de configuration Azure AD Connect (préversion publique)

Les déploiements Azure Active Directory (Azure AD) Connect varient d’une installation de forêt unique en mode Express à des déploiements complexes qui se synchronisent sur plusieurs forêts à l’aide de règles de synchronisation personnalisées. En raison du grand nombre d’options et de mécanismes de configuration, il apparaît essentiel de comprendre les paramètres actifs et de pouvoir déployer rapidement un serveur avec une configuration identique. Cette fonctionnalité introduit la possibilité de cataloguer la configuration d’un serveur de synchronisation donné et d’importer les paramètres dans un nouveau déploiement. Différents instantanés de paramètres de synchronisation peuvent être comparés afin de visualiser facilement les différences entre deux serveurs ou le même serveur au fil du temps.

Chaque fois que la configuration est modifiée à partir de l’Assistant Azure AD Connect, un nouveau fichier de paramètres JSON horodaté est automatiquement exporté vers  **%ProgramData%\AADConnect**. Le nom de fichier des paramètres se présente sous la forme **Applied-SynchronizationPolicy-*.JSON** où la dernière partie de ce nom correspond à un timestamp.

> [!IMPORTANT]
> Seules les modifications apportées par Azure AD Connect sont automatiquement exportées. Toutes les modifications apportées à l’aide de PowerShell, de Synchronization Service Manager ou de l’éditeur de règles de synchronisation doivent être exportées à la demande, si nécessaire, pour conserver une copie à jour. L’exportation à la demande peut également être utilisée pour placer une copie des paramètres dans un emplacement sécurisé à des fins de récupération d’urgence.

## <a name="export-azure-ad-connect-settings"></a>Exporter les paramètres d'Azure AD Connect 

Pour afficher un résumé de vos paramètres de configuration, ouvrez l’outil Azure AD Connect et sélectionnez la tâche supplémentaire nommée **Afficher ou Exporter la configuration actuelle** . Un résumé rapide de vos paramètres s’affiche, de même que la possibilité d’exporter la configuration complète de votre serveur.

Par défaut, les paramètres sont exportés vers **%ProgramData%\AADConnect**. Vous pouvez également choisir d’enregistrer les paramètres dans un emplacement protégé pour garantir la disponibilité en cas de sinistre. Les paramètres sont exportés à l’aide du format de fichier JSON et ne doivent pas être créés ou modifiés à des fins de cohérence logique. L’importation d’un fichier créé à la main ou modifié n’est pas prise en charge et peut entraîner des résultats inattendus.

## <a name="import-azure-ad-connect-settings"></a>Importer les paramètres d'Azure AD Connect

Pour importer des paramètres précédemment exportés :
 
1. Installez **Azure AD Connect** sur un nouveau serveur.
1. Sélectionnez l’option **Personnaliser** après la page d’**Accueil**.
1. Cliquez sur **Importer les paramètres de synchronisation**. Recherchez le fichier de paramètres JSON précédemment exporté.
1. Sélectionnez **Installer**.

   ![Capture d’écran affichant l’écran Installer les composants nécessaires.](media/how-to-connect-import-export-config/import1.png)

> [!NOTE]
> Remplacez les paramètres de cette page, comme l’utilisation de SQL Server plutôt que la base de données locale ou l’utilisation d’un compte de service existant plutôt que l’attribut VSA par défaut. Ces paramètres ne sont pas importés à partir du fichier de paramètres de configuration. Ils sont destinés à des fins de comparaison et d’information.

### <a name="import-installation-experience"></a>Expérience d’importation de l’installation 

L’expérience d’importation de l’installation est intentionnellement simple avec des entrées minimales de l’utilisateur afin de facilement reproduire un serveur existant.

Voici les seules modifications qui peuvent être apportées lors de l’expérience d’installation. Toutes les autres modifications peuvent être apportées après l’installation à partir de l’Assistant Azure AD Connect :
- **Informations d’identification Azure Active Directory** : Le nom de compte de l’administrateur général Azure utilisé pour configurer le serveur d’origine est suggéré par défaut. Il *doit* être modifié si vous souhaitez synchroniser les informations dans un nouveau répertoire.
- **Connexion utilisateur** : Les options de connexion configurées pour votre serveur d’origine sont sélectionnées par défaut et demandent automatiquement les informations d’identification ou d’autres informations requises lors de la configuration. Dans de rares cas, il peut s’avérer nécessaire de configurer un serveur avec différentes options pour ne pas modifier le comportement du serveur actif. Sinon, appuyez simplement sur **Suivant** pour utiliser les mêmes paramètres.
- **Informations d’identification du répertoire local** : Pour chaque répertoire local inclus dans vos paramètres de synchronisation, vous devez fournir des informations d’identification afin de créer un compte de synchronisation ou fournir un compte de synchronisation personnalisé créé préalablement. Cette procédure est identique à l’expérience de nouvelle installation à cette exception près : vous ne pouvez pas ajouter ou supprimer de répertoires.
- **Options de configuration** : Comme pour une nouvelle installation, vous pouvez choisir de configurer les paramètres initiaux pour déterminer s’il faut démarrer la synchronisation automatique ou activer le mode de préproduction. La principale différence réside dans le fait que le mode de préproduction est intentionnellement activé par défaut pour permettre la comparaison des résultats de configuration et de synchronisation avant d’exporter activement les résultats vers Azure.

![Capture d’écran affichant l’écran Connexion de vos annuaires.](media/how-to-connect-import-export-config/import2.png)

> [!NOTE]
> Un seul serveur de synchronisation peut remplir le rôle principal et exporter activement les modifications de configuration vers Azure. Tous les autres serveurs doivent être placés en mode de préproduction.

## <a name="migrate-settings-from-an-existing-server"></a>Migrer les paramètres à partir d'un serveur existant 

Si un serveur existant ne prend pas en charge la gestion des paramètres, vous pouvez choisir de mettre à niveau le serveur sur place ou de migrer les paramètres à utiliser vers un nouveau serveur de préproduction.

La migration requiert l’exécution d’un script PowerShell qui extrait les paramètres existants à utiliser dans une nouvelle installation. Cette méthode est recommandée pour cataloguer les paramètres de votre serveur existant, puis les appliquer à un serveur de préproduction nouvellement installé. La comparaison des paramètres du serveur d’origine avec le serveur nouvellement créé permet de visualiser rapidement les modifications entre les serveurs. Comme toujours, suivez le processus de certification de votre organisation pour vous assurer qu’aucune configuration supplémentaire ne s’impose.

### <a name="migration-process"></a>Processus de migration 
Pour migrer les paramètres :

1. Lancez **AzureADConnect.msi** sur le nouveau serveur intermédiaire et arrêtez-vous sur la page d’**accueil** d’Azure AD Connect.

1. Copiez **MigrateSettings.ps1** à partir du répertoire Microsoft Azure AD Connect\Tools vers un emplacement du serveur existant. Un exemple est C:\setup, où setup correspond à un répertoire créé sur le serveur existant.

   ![Capture d’écran montrant les répertoires Azure AD Connect.](media/how-to-connect-import-export-config/migrate1.png)

1. Exécutez le script comme indiqué ci-dessous et enregistrez l’intégralité du répertoire de configuration de serveur de niveau inférieur. Copiez ce répertoire sur le nouveau serveur de préproduction. Notez que vous devez copier l’intégralité du dossier **Exported-ServerConfiguration-** * sur le nouveau serveur.

   ![Capture d’écran illustrant le script dans Windows PowerShell.](media/how-to-connect-import-export-config/migrate2.png)
   ![Capture d’écran qui montre la copie du dossier Exported-ServerConfiguration-*.](media/how-to-connect-import-export-config/migrate3.png)

1. Lancez **Azure AD Connect** en double-cliquant sur l’icône du bureau. Acceptez les termes du contrat de licence logiciel Microsoft, puis sur la page suivante, sélectionnez **Personnaliser**.
1. Cliquez sur la case à cocher **Importer les paramètres de synchronisation**. Sélectionnez **Parcourir** pour parcourir le dossier Exported-ServerConfiguration-* copié. Sélectionnez le MigratedPolicy. json pour importer les paramètres migrés.

   ![Capture d’écran qui montre l’option Importer les paramètres de synchronisation.](media/how-to-connect-import-export-config/migrate4.png)

## <a name="post-installation-verification"></a>Vérification après l’installation 

La comparaison du fichier de paramètres initialement importés avec le fichier de paramètres exportés du serveur nouvellement déployé est une étape essentielle pour comprendre les différences entre le déploiement prévu et le déploiement obtenu. L’utilisation de votre application de comparaison de texte favorite côte à côte offre une visualisation instantanée qui met rapidement en évidence les modifications souhaitées ou accidentelles.

Si de nombreuses étapes de configuration manuelles sont désormais évitées, vous devez systématiquement suivre le processus de certification de votre organisation pour vous assurer qu’aucune configuration supplémentaire ne s’impose. Une telle configuration peut intervenir si vous utilisez des paramètres avancés non capturés dans la préversion publique de gestion des paramètres.

Voici quelques limitations connues :
- **Règles de synchronisation** : La priorité d’une règle personnalisée doit être comprise dans la plage réservée 0 pour éviter les conflits avec les règles standard de Microsoft. Le placement d’une règle personnalisée en dehors de la plage réservée peut se traduire par un décalage de votre règle personnalisée au fur et à mesure que des règles standard sont ajoutées à la configuration. Un problème similaire se produit si votre configuration contient des règles standard modifiées. La modification d’une règle standard est vivement déconseillée et l’emplacement de la règle est susceptible d’être incorrect.
- **Réécriture d’appareil** : Ces paramètres sont catalogués. Ils ne sont actuellement pas appliqués pendant la configuration. Si la réécriture d'appareil a été activée pour votre serveur d’origine, vous devez configurer manuellement cette fonctionnalité sur le serveur nouvellement déployé.
- **Types d’objets synchronisés**: Bien qu’il soit possible de limiter la liste des types d’objets synchronisés (utilisateurs, contacts, groupes, etc.) à l’aide de Synchronization Service Manager, cette fonctionnalité n’est actuellement pas prise en charge via les paramètres de synchronisation. Une fois l’installation terminée, vous devez réappliquer manuellement la configuration avancée.
- **Profils d'exécution personnalisés** : Ben qu’il soit possible de modifier l’ensemble des profils d’exécution par défaut à l’aide de Synchronization Service Manager, cette fonctionnalité n’est actuellement pas prise en charge via les paramètres de synchronisation. Une fois l’installation terminée, vous devez réappliquer manuellement la configuration avancée.
- **Configurer la hiérarchie de l’approvisionnement** : Cette fonctionnalité avancée du Synchronization Service Manager n’est pas prise en charge via les paramètres de synchronisation. Elle doit être reconfigurée manuellement une fois le déploiement initial terminé.
- **Authentification SSRS à l'aide des services de fédération Active Directory (AD FS) et PingFederate** : Les méthodes de connexion associées à ces fonctionnalités d’authentification sont automatiquement présélectionnées. Vous devez fournir de manière interactive tous les autres paramètres de configuration requis.
- **Une règle de synchronisation personnalisée désactivée sera importée comme activée**: Une règle de synchronisation personnalisée désactivée sera importée comme activée. Veillez à la désactiver également sur le nouveau serveur.

 ## <a name="next-steps"></a>Étapes suivantes

- [Matériel et conditions préalables](how-to-connect-install-prerequisites.md) 
- [Paramètres Express](how-to-connect-install-express.md)
- [Paramètres personnalisés](how-to-connect-install-custom.md)
- [Installation des agents Azure AD Connect Health](how-to-connect-health-agent-install.md) 
