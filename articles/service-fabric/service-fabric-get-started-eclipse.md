---
title: Plug-in Azure Service Fabric pour Eclipse
description: En savoir plus sur la prise en main d’Azure Service Fabric dans Java à l’aide d’Eclipse et du plug-in Service Fabric fourni.
author: rapatchi
ms.topic: conceptual
ms.date: 04/06/2018
ms.author: rapatchi
ms.custom: devx-track-java
ms.openlocfilehash: b85206f9b1e92607bba7b6f141b700922c129b65
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656918"
---
# <a name="service-fabric-plug-in-for-eclipse-java-application-development"></a>Plug-in Service Fabric pour le développement d’applications Java sous Eclipse
Eclipse est l’un des environnements de développement intégrés (IDE) les plus largement utilisés par les développeurs Java. Dans cet article, nous décrivons comment configurer votre environnement de développement Eclipse pour utiliser Azure Service Fabric. Découvrez comment installer le plug-in Service Fabric, créer une application Service Fabric et déployer votre application Service Fabric dans un cluster Service Fabric local ou distant sur Eclipse. 

> [!NOTE]
> Le plug-in Eclipse n’est actuellement pas pris en charge sous Windows. 

> [!IMPORTANT]
> Assurez-vous que JDK 8 est installé sur le système et qu’il est sélectionné dans Eclipse.

## <a name="install-or-update-the-service-fabric-plug-in-in-eclipse"></a>Installer ou mettre à jour le plug-in Service Fabric sur Eclipse
Vous pouvez installer un plug-in Service Fabric sur Eclipse. Ce plug-in peut aider à simplifier le processus de génération et le déploiement des services Java.

> [!IMPORTANT]
> Le plug-in Service Fabric nécessite Eclipse Neon ou une version ultérieure. Consultez les instructions qui suivent cette note pour savoir comment vérifier votre version d’Eclipse. Si vous avez installé une version antérieure d’Eclipse, vous pouvez télécharger des versions les plus récentes à partir du [site Eclipse](https://www.eclipse.org). Nous vous déconseillons de procéder à l’installation par dessus (par écrasement) une installation existante d’Eclipse. Vous pouvez soit la supprimer avant d’exécuter le programme d’installation ou installer la nouvelle version dans un répertoire différent. 
> 
> Sur Ubuntu, nous vous recommandons de procéder à l’installation directement depuis le site Eclipse au lieu d’utiliser le programme d’installation d’un package (`apt` ou `apt-get`). Ceci vous assure d’obtenir la version la plus récente d’Eclipse. 

Installez Eclipse Neon ou ultérieur à partir du [site d’Eclipse](https://www.eclipse.org).  Installez également la version 2.2.1 ou ultérieure du Buildship (le plug-in Service Fabric n’est pas compatible avec les versions antérieures de Buildship) :
-   Pour vérifier les versions des composants installés, dans Eclipse, accédez à **Aide** > **About Eclipse (Au sujet d’Eclipse)**  > **Détails de l’installation**.
-   Pour mettre à jour Buildship, consultez [Eclipse Buildship : plug-ins Eclipse pour Gradle][buildship-update].
-   Pour vérifier et installer les mises à jour pour Eclipse, accédez à **Aide** > **Rechercher les mises à jour**.

Installez le plug-in Service Fabric : dans Eclipse, accédez à **Aide** > **Installer un nouveau logiciel**.
1. Dans la zone **Work with** (Utiliser avec), entrez https:\//dl.microsoft.com/eclipse.
2. Cliquez sur **Add**.

   ![Plug-in Service Fabric pour Eclipse][sf-eclipse-plugin-install]
3. Sélectionnez le plug-in Service Fabric, puis cliquez sur **Suivant**.
4. Terminez les étapes d’installation, puis acceptez les termes du contrat de licence du logiciel Microsoft.

Si le plug-in Service Fabric est déjà installé, installez la dernière version. 
1. Pour vérifier les mises à jour disponibles, accédez à **Aide** > **About Eclipse (Au sujet d’Eclipse)**  > **Détails de l’installation**. 
2. Dans la liste des plug-ins installés, sélectionnez Service Fabric, puis cliquez sur **Mettre à jour**. Les mises à jour disponibles seront installées.
3. Lorsque vous mettez à jour le plug-in Service Fabric, actualisez également le projet Gradle.  Cliquez avec le bouton droit sur **build.gradle**, puis sélectionnez **Actualiser**.

> [!NOTE]
> La lenteur de l’installation ou de la mise à jour du plug-in Service Fabric peut être due à un paramètre d’Eclipse. Eclipse collecte des métadonnées sur toutes les modifications pour mettre à jour les sites enregistrés avec votre instance d’Eclipse. Pour accélérer le processus de vérification et l’installation d’une mise à jour du plug-in Service Fabric, accédez à **Sites logiciels disponibles**. Désactivez les cases à cocher pour tous les sites, sauf pour celui qui pointe vers l’emplacement du plug-in Service Fabric (https:\//dl.microsoft.com/eclipse/azure/servicefabric).

> [!NOTE]
>Si Eclipse ne fonctionne pas comme prévu sur votre Mac ou exige vous soyez un super utilisateur pour l’exécuter, accédez au dossier **ECLIPSE_INSTALLATION_PATH**, puis au sous-dossier **Eclipse.app/Contents/MacOS**. Démarrez Eclipse en exécutant `./eclipse`.

## <a name="create-a-service-fabric-application-in-eclipse"></a>Créer une application Service Fabric dans Eclipse

1.  Dans Eclipse, accédez à **Fichier** > **Nouveau** > **Autre**. Sélectionnez **Projet Service Fabric**, puis cliquez sur **Suivant**.

    ![Nouveau projet Service Fabric, page 1][create-application/p1]

2.  Saisissez un nom pour votre projet, puis cliquez sur **Suivant**.

    ![Nouveau projet Service Fabric, page 2][create-application/p2]

3.  Dans la liste des modèles, sélectionnez **Modèle de service**. Sélectionnez votre type de modèle de service (acteur, sans état, conteneur ou binaire invité), puis cliquez sur **Suivant**.

    ![Nouveau projet Service Fabric, page 3][create-application/p3]

4.  Saisissez le nom du service et les détails du service, puis cliquez sur **Terminer**.

    ![Nouveau projet Service Fabric, page 4][create-application/p4]

5. Lorsque vous créez votre premier projet Service Fabric, dans la boîte de dialogue **Ouvrir la perspective associée**, cliquez sur **Oui**.

    ![Nouveau projet Service Fabric, page 5][create-application/p5]

6.  Votre nouveau projet ressemble à ceci :

    ![Nouveau projet Service Fabric, page 6][create-application/p6]

## <a name="build-a-service-fabric-application-in-eclipse"></a>Développer une application Service Fabric dans Eclipse

1.  Cliquez avec le bouton droit de la souris sur votre nouvelle application Service Fabric, puis sélectionnez **Service Fabric**.

    ![Menu contextuel Service Fabric][publish/RightClick]

2. Dans le menu contextuel, sélectionnez l’une des options suivantes :
    -   Pour créer l’application sans suppression, cliquez sur **Créer l’application**.
    -   Pour supprimer, puis générer l’application, cliquez sur **Régénérer l’application**.
    -   Pour supprimer l’application des artefacts générés, cliquez sur **Nettoyer l’application**.

## <a name="deploy-a-service-fabric-application-to-the-local-cluster-with-eclipse"></a>Déployer une application Service Fabric sur le cluster local avec Eclipse

Une fois que vous avez créé votre application Service Fabric, suivez les étapes ci-dessous pour la déployer sur le cluster local.

1. Si vous n’avez pas démarré le cluster local, suivez les instructions de la section [Configurer un cluster local](./service-fabric-get-started-linux.md#set-up-a-local-cluster) pour démarrer votre cluster local et vous assurer qu’il s’exécute correctement.
2. Cliquez avec le bouton droit de la souris sur votre application Service Fabric, puis sélectionnez **Service Fabric**.

    ![Menu contextuel Service Fabric][publish/RightClick]

3.  Dans le menu contextuel, cliquez sur **Déployer une application**.
4.  Vous pouvez suivre la progression du déploiement dans la fenêtre de la console.
5.  Pour vérifier que votre application est en cours d’exécution, ouvrez Service Fabric Explorer sur votre cluster local dans une fenêtre de navigateur `http://localhost:19080/Explorer`. Développez le nœud **Applications** et vérifiez que votre application est en cours d’exécution. 

Pour savoir comment déboguer votre application dans Eclipse en utilisant le cluster local, consultez l’article [Débogage de votre application Java Service Fabric avec Eclipse](./service-fabric-debugging-your-application-java.md).

Vous pouvez également déployer votre application dans le cluster local à l’aide de la commande **Publier une application** :

1. Cliquez avec le bouton droit de la souris sur votre application Service Fabric, puis sélectionnez **Service Fabric**.
2. Dans le menu contextuel, cliquez sur **Publier une application...** .
3. Dans la fenêtre **Publier une application**, choisissez **PublishProfiles/Local.json** comme profil cible, puis cliquez sur **Publier**.

    ![Boîte de dialogue Publier - Local](./media/service-fabric-get-started-eclipse/localjson.png)

    Par défaut, le profil de publication Local.json est configuré pour publier sur le cluster local. Pour plus d’informations sur les paramètres de connexion et de point de terminaison présents dans les profils de publication, consultez la section suivante.

## <a name="publish-your-service-fabric-application-to-azure-with-eclipse"></a>Publier votre application Service Fabric sur Azure avec Eclipse

Pour publier votre application dans le cloud, suivez les étapes ci-dessous :

1. Pour publier votre application sur un cluster sécurisé dans le cloud, vous avez besoin d’un certificat X.509 pour pouvoir communiquer avec votre cluster. Dans les environnements de test et de développement, le certificat utilisé est souvent le certificat du cluster. Dans les environnements de production, le certificat doit être un certificat client distinct du certificat de cluster. Vous avez besoin aussi bien du certificat que de la clé privée. Le fichier de certificat (et de la clé) doit être au format PEM. Vous pouvez créer un fichier PEM contenant le certificat et la clé privée à partir d’un fichier PFX à l’aide de la commande openssl suivante :

    ```bash
    openssl pkcs12 -in your-cert-file.pfx -out your-cert-file.pem -nodes -passin pass:your-pfx-password
    ```

   Si le fichier PFX n’est pas protégé par mot de passe, utilisez `--passin pass:` comme dernier paramètre.

2. Ouvrez le fichier **Cloud.json** sous le répertoire **PublishProfiles**. Vous devez configurer les informations d’identification de sécurité et de point de terminaison du cluster en fonction de votre cluster.

   - Le champ `ConnectionIPOrURL` contient l’adresse IP ou l’URL de votre cluster. Notez que la valeur ne contient pas le schéma d’URL (`https://`).
   - Par défaut, le champ `ConnectionPort` doit être défini sur `19080`, sauf si vous avez explicitement modifié ce port pour votre cluster.
   - Le champ `ClientKey` doit pointer vers un fichier .pem ou .key au format PEM sur votre ordinateur local qui contient la clé privée pour le certificat de votre client ou cluster.
   - Le champ `ClientCert` doit pointer vers un fichier .pem ou .crt au format PEM sur votre ordinateur local qui contient les données de certificat pour le certificat de votre client. ou cluster. 

     ```bash
     {
         "ClusterConnectionParameters":
         {
            "ConnectionIPOrURL": "lnxxug0tlqm5.westus.cloudapp.azure.com",
            "ConnectionPort": "19080",
            "ClientKey": "[path_to_your_pem_file_on_local_machine]",
            "ClientCert": "[path_to_your_pem_file_on_local_machine]"
         }
     }
     ```

2. Cliquez avec le bouton droit de la souris sur votre application Service Fabric, puis sélectionnez **Service Fabric**.
3. Dans le menu contextuel, cliquez sur **Publier une application...** .
3. Dans la fenêtre **Publier une application**, choisissez **PublishProfiles/Cloud.json** comme profil cible, puis cliquez sur **Publier**.

    ![Boîte de dialogue Publier - Cloud](./media/service-fabric-get-started-eclipse/cloudjson.png)

4. Vous pouvez suivre la progression de la publication dans la fenêtre de la console.
5. Pour vérifier que votre application est en cours d’exécution, ouvrez Service Fabric Explorer sur votre cluster Azure dans une fenêtre de navigateur. Pour l’exemple ci-dessus, il s’agit de `https://lnxxug0tlqm5.westus.cloudapp.azure.com:19080/Explorer`. Développez le nœud **Applications** et vérifiez que votre application est en cours d’exécution. 

Sur les clusters Linux sécurisés, si votre application contient des services Reliable Services, vous devez également configurer un certificat que vos services pourront utiliser pour appeler les API du runtime Service Fabric. Pour en savoir plus, consultez [Configurer une application Reliable Services à exécuter sur les clusters Linux](./service-fabric-configure-certificates-linux.md#configure-a-reliable-services-app-to-run-on-linux-clusters).

Pour savoir rapidement comment déployer une application Reliable Services de Service Fabric écrite en Java sur un cluster Linux sécurisé, consultez [Démarrage rapide : Déployer une application Reliable Services Java](./service-fabric-quickstart-java-reliable-services.md).

## <a name="deploy-a-service-fabric-application-by-using-eclipse-run-configurations"></a>Déployer une application Service Fabric en utilisant des configurations d’exécution Eclipse

Une autre méthode pour déployer votre application Service Fabric consiste à utiliser des configurations d’exécution Eclipse.

1. Dans Eclipse, accédez à **Exécuter** > **Configurations d’exécution**.
2. Sous **Projet Gradle**, sélectionnez la configuration d’exécution **ServiceFabricDeployer**.
3. Dans le volet droit, sous l’onglet **Arguments**, vérifiez que les paramètres **ip**, **port**, **clientCert** et **clientKey** sont correctement définis pour votre déploiement. Par défaut, les paramètres sont définis pour un déploiement sur le cluster local, comme le montre la capture d’écran suivante. Pour publier votre application dans Azure, vous pouvez modifier les paramètres pour qu’ils contiennent les détails de point de terminaison et les informations d’identification de sécurité pour votre cluster Azure. Pour plus d’informations, consultez la section précédente, [Publier votre application Service Fabric sur Azure avec Eclipse](#publish-your-service-fabric-application-to-azure-with-eclipse).

    ![Boîte de dialogue Configuration de série de tests en local](./media/service-fabric-get-started-eclipse/run-config-local.png)

5. Vérifiez que le **Répertoire de travail** pointe vers l’application que vous souhaitez déployer. Pour modifier l’application, cliquez sur le bouton **Espace de travail**, puis sélectionnez l’application souhaitée.
6. Cliquez sur **Appliquer**, puis sur **Exécuter**.

Votre application est générée et déployée en quelques instants. Vous pouvez contrôler le statut de déploiement dans Service Fabric Explorer.  

## <a name="add-a-service-fabric-service-to-your-service-fabric-application"></a>Ajouter un service Service Fabric à votre application Service Fabric

Pour ajouter un service Service Fabric à une application Service Fabric existante, procédez comme suit :

1.  Cliquez avec le bouton droit de la souris sur le projet auquel vous souhaitez ajouter un service, puis cliquez sur **Service Fabric**.

    ![Ajouter un service Service Fabric, page 1][add-service/p1]

2.  Cliquez sur **Ajouter un service Service Fabric** et terminez les d’étapes pour ajouter un service au projet.
3.  Sélectionnez le modèle de service que vous souhaitez ajouter à votre projet et cliquez sur **Suivant**.

    ![Ajouter un service Service Fabric, page 2][add-service/p2]

4.  Entrez le nom du service (et autres détails, si nécessaire), puis cliquez sur le bouton **Ajouter un service**.  

    ![Ajouter un service Service Fabric, page 3][add-service/p3]

5.  Une fois le service ajouté, la structure globale de votre projet ressemble au projet suivant :

    ![Ajouter un service Service Fabric, page 4][add-service/p4]

## <a name="edit-manifest-versions-of-your-service-fabric-java-application"></a>Modifier les versions de manifeste de votre application Java Service Fabric

Pour modifier les versions de manifeste, cliquez avec le bouton droit sur le projet, accédez à **Service Fabric** et sélectionnez **Modifier les versions de manifeste...**  à partir de la liste déroulante du menu. Dans l’Assistant, vous pouvez mettre à jour les versions du manifeste d’application, du manifeste de service et des packages **Code**, **Config** et **Data**.

Si vous activez l’option **Mettre à jour automatiquement les versions des applications et des services**, puis que vous mettez à jour une version, les versions des manifestes seront automatiquement mises à jour. Par exemple, vous commencez par cocher la case, vous changez la version du **Code** de 0.0.0 à 0.0.1, puis vous cliquez sur **Terminer**. La version du manifeste de service et du manifeste d’application sera automatiquement mise à jour à 0.0.1.

## <a name="upgrade-your-service-fabric-java-application"></a>Mettre à niveau votre application Java Service Fabric

Prenons l’exemple d’un scénario de mise à niveau. Admettons que vous avez créé le projet **App1** à l’aide du plug-in Service Fabric dans Eclipse. Vous l’avez déployé à l’aide du plug-in pour créer une application nommée **fabric:/App1Application**. Le type d’application est **App1ApplicationType**, et la version de l’application est 1.0. Vous souhaitez à présent mettre à niveau votre application sans interrompre la disponibilité.

Tout d’abord, apportez les éventuelles modifications à votre application, puis reconstruisez le service modifié. Mettez à jour le fichier manifeste du service modifié (ServiceManifest.xml) avec les versions mises à jour du service (Code, Config ou Données, comme il convient). Modifiez également le manifeste de l’application (ApplicationManifest.xml) avec le numéro de version mis à jour de l’application et le service modifié.  

Pour mettre à niveau votre application à l’aide d’Eclipse, vous pouvez créer un profil de configuration d’exécution en double. Ensuite, utilisez-le pour mettre à niveau votre application selon vos besoins.

1.  Accédez à **Exécuter** > **Configurations d’exécution**. Dans le volet de gauche, cliquez sur la petite flèche à gauche de **Projet Gradle**.
2.  Cliquez avec le bouton droit de la souris sur **ServiceFabricDeployer**, puis sélectionnez **Dupliquer**. Entrez un nouveau nom pour cette configuration, par exemple, **ServiceFabricUpgrader**.
3.  Dans le volet de droite, sous l’onglet **Arguments**, remplacez **-Pconfig='deploy'** par **-Pconfig='upgrade'** , puis cliquez sur **Appliquer**.

Ce processus crée et enregistre un profil de configuration d’exécution que vous pouvez utiliser à tout moment pour mettre à niveau votre application. Il permet également d’obtenir la dernière version mise à jour du type d’application à partir du fichier de manifeste d’application.

La mise à niveau de l’application prend quelques minutes. Vous pouvez contrôler la mise à niveau de l’application dans Service Fabric Explorer.

## <a name="migrating-old-service-fabric-java-applications-to-be-used-with-maven"></a>Migration d’anciennes applications Java Service Fabric à utiliser avec Maven
Nous avons récemment déplacé les bibliothèques Java Service Fabric vers un référentiel Maven depuis le Kit de développement logiciel (SDK) Java Service Fabric. Tandis que les nouvelles applications que vous générez à l’aide d’Eclipse génèrent les derniers projets mis à jour (en mesure de fonctionner avec Maven), vous pouvez mettre à jour vos services sans état Service Fabric existants ou vos applications Java d’acteur existantes ayant utilisé le Kit de développement logiciel (SDK) précédemment, afin d’utiliser les dépendances Java Service Fabric à partir de Maven. Suivez les étapes mentionnées [ici](service-fabric-migrate-old-javaapp-to-use-maven.md) pour vous assurer qu’une version plus ancienne de votre application fonctionne avec Maven.

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment créer une application Reliable Services écrite en Java et comment la déployer localement et dans Azure, consultez [Démarrage rapide : Déployer une application Reliable Services Java](./service-fabric-quickstart-java-reliable-services.md).
- Pour savoir comment déboguer une application sur votre cluster local, consultez l’article [Débogage de votre application Java Service Fabric avec Eclipse](./service-fabric-debugging-your-application-java.md).
- Pour savoir comment surveiller et diagnostiquer les applications Service Fabric, consultez la page [Surveillance et diagnostic des services dans une configuration de développement d’ordinateur local](./service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux.md).

<!-- Images -->

[sf-eclipse-plugin-install]: ./media/service-fabric-get-started-eclipse/service-fabric-eclipse-plugin.png

[create-application/p1]:./media/service-fabric-get-started-eclipse/create-application/p1.png
[create-application/p2]:./media/service-fabric-get-started-eclipse/create-application/p2.png
[create-application/p3]:./media/service-fabric-get-started-eclipse/create-application/p3.png
[create-application/p4]:./media/service-fabric-get-started-eclipse/create-application/p4.png
[create-application/p5]:./media/service-fabric-get-started-eclipse/create-application/p5.png
[create-application/p6]:./media/service-fabric-get-started-eclipse/create-application/p6.png

[publish/Publish]: ./media/service-fabric-get-started-eclipse/publish/Publish.png
[publish/RightClick]: ./media/service-fabric-get-started-eclipse/publish/RightClick.png

[add-service/p1]: ./media/service-fabric-get-started-eclipse/add-service/p1.png
[add-service/p2]: ./media/service-fabric-get-started-eclipse/add-service/p2.png
[add-service/p3]: ./media/service-fabric-get-started-eclipse/add-service/p3.png
[add-service/p4]: ./media/service-fabric-get-started-eclipse/add-service/p4.png

<!-- Links -->
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship
