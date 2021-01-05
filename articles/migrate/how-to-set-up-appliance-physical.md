---
title: Configurer une appliance Azure Migrate pour les serveurs physiques
description: Découvrez comment configurer une appliance Azure Migrate pour l’évaluation de serveurs physiques.
author: vikram1988
ms.author: vibansa
ms.manager: abhemraj
ms.topic: how-to
ms.date: 04/15/2020
ms.openlocfilehash: 21d40b1962a3e4000876686659b76d019991785d
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96751680"
---
# <a name="set-up-an-appliance-for-physical-servers"></a>Configurer une appliance pour les serveurs physiques

Cet article explique comment configurer l’appliance Azure Migrate si vous évaluez des serveurs physiques avec l’outil Azure Migrate : Server Assessment.

L’appliance Azure Migrate est une appliance légère utilisée par Azure Migrate Server Assessment pour effectuer les opérations suivantes :

- Découvrir les serveurs locaux.
- Envoyer des métadonnées et des données de performances concernant les serveurs découverts à Azure Migrate Server Assessment.

[En savoir plus](migrate-appliance.md) sur les appliances d’Azure Migrate.


## <a name="appliance-deployment-steps"></a>Étapes de déploiement d’appliance

Pour configurer l’appliance, vous devez :
- Fournir un nom d’appliance et générer une clé de projet Azure Migrate sur le portail.
- Télécharger un fichier compressé avec le script du programme d’installation Azure Migrate à partir du portail Azure.
- Extraire le contenu du fichier compressé. Lancer la console PowerShell avec des privilèges administratifs.
- Exécuter le script PowerShell pour lancer l’application web de l’appliance.
- Configurer l’appliance pour la première fois, puis l’inscrire auprès du projet Azure Migrate en utilisant la clé de projet Azure Migrate.

### <a name="generate-the-azure-migrate-project-key"></a>Générer la clé de projet Azure Migrate

1. Dans **Objectifs de migration** > **Serveurs** > **Azure Migrate : Server Assessment**, sélectionnez **Découvrir**.
2. Dans **Découvrir des machines** > **Vos machines sont-elles virtualisées ?** , sélectionnez **Physiques ou autres (AWS, GCP, Xen, etc.)** .
3. Dans **1 : Générer une clé de projet Azure Migrate**, attribuez un nom à l’appliance Azure Migrate que vous allez configurer pour la découverte de serveurs physiques ou virtuels. Le nom doit être alphanumérique et se composer au maximum de 14 caractères.
1. Cliquez sur **Générer une clé** pour lancer la création des ressources Azure nécessaires. Ne fermez pas la page Découvrir des machines pendant la création des ressources.
1. Une fois les ressources Azure créées, une **clé de projet Azure Migrate** est générée.
1. Copiez la clé, car vous en aurez besoin pour terminer l’inscription de l’appliance pendant sa configuration.

### <a name="download-the-installer-script"></a>Télécharger le script du programme d’installation

Dans **2 : Télécharger l’appliance Azure Migrate**, cliquez sur **Télécharger**.

   ![Sélections pour la découverte de machines](./media/tutorial-assess-physical/servers-discover.png)


   ![Sélections pour la génération d’une clé](./media/tutorial-assess-physical/generate-key-physical.png)

### <a name="verify-security"></a>Vérifier la sécurité

Vérifiez que le fichier compressé est sécurisé avant de le déployer.

1. Sur l’ordinateur où vous avez téléchargé le fichier, ouvrez une fenêtre de commande d’administrateur.
2. Exécutez la commande suivante pour générer le code de hachage du fichier compressé :
    - ```C:\>CertUtil -HashFile <file_location> [Hashing Algorithm]```
    - Exemple d’utilisation du cloud public : ```C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public.zip SHA256 ```
    - Exemple d’utilisation du cloud du secteur public : ```  C:\>CertUtil -HashFile C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-USGov.zip MD5 ```
3.  Vérifiez la dernière version de l’appliance et les paramètres des [valeurs de hachage](tutorial-discover-physical.md#verify-security).
 

## <a name="run-the-azure-migrate-installer-script"></a>Exécuter le script du programme d’installation Azure Migrate
Le script du programme d’installation effectue les opérations suivantes :

- Installe des agents et une application web pour la découverte et l’évaluation des serveurs physiques.
- Installe des rôles Windows, notamment le service d’activation Windows, IIS et PowerShell ISE.
- Télécharge et installe un module réinscriptible IIS. [Plus d’informations](https://www.microsoft.com/download/details.aspx?id=7435)
- Met à jour une clé de Registre (HKLM) avec les détails de paramètres persistants pour Azure Migrate.
- Crée les fichiers suivants sous le chemin :
    - **Fichiers de configuration** : %Programdata%\Microsoft Azure\Config
    - **Fichiers journaux** : %Programdata%\Microsoft Azure\Logs

Exécutez le script comme suit :

1. Extrayez le fichier compressé dans un dossier sur le serveur qui hébergera l’appliance.  Veillez à ne pas exécuter le script sur un ordinateur d’une appliance Azure Migrate existante.
2. Lancez PowerShell sur le serveur ci-dessus avec un privilège administratif (élevé).
3. Remplacez le répertoire PowerShell par le dossier dans lequel le contenu a été extrait du fichier compressé téléchargé.
4. Exécutez le script nommé **AzureMigrateInstaller.ps1** via la commande suivante :

    - Pour le cloud public : 
    
        ``` PS C:\Users\administrator\Desktop\AzureMigrateInstaller-Server-Public> .\AzureMigrateInstaller.ps1 ```
    - Pour Azure Government : 
    
        ``` PS C:\Users\Administrators\Desktop\AzureMigrateInstaller-Server-USGov>.\AzureMigrateInstaller.ps1 ```

    Une fois son exécution terminé, le script lance l’application web de l’appliance.

Si vous rencontrez des problèmes, vous pouvez accéder aux journaux de script dans C:\ProgramData\Microsoft Azure\Logs\AzureMigrateScenarioInstaller_<em>horodatage</em>.log pour les résoudre.



### <a name="verify-appliance-access-to-azure"></a>Vérifier l’accès de l’appliance à Azure

Vérifiez que la machine virtuelle de l’appliance peut se connecter aux URL Azure pour les clouds [publics](migrate-appliance.md#public-cloud-urls) et du [secteur public](migrate-appliance.md#government-cloud-urls).

### <a name="configure-the-appliance"></a>Configurer l’appliance

Configurez l’appliance pour la première fois.

1. Ouvrez un navigateur sur une machine qui peut se connecter à l’appliance, puis ouvrez l’URL de l’application web de l’appliance : **https://*nom ou adresse IP de l’appliance* : 44368**.

   Vous pouvez aussi ouvrir l’application à partir du bureau en cliquant sur le raccourci de l’application.
2. Acceptez les **termes du contrat de licence** et lisez les informations relatives aux tiers.
1. Dans l’application web > **Configurer les prérequis**, procédez comme suit :
    - **Connectivité** : L’application vérifie que le serveur a accès à Internet. Si le serveur utilise un proxy :
        - Cliquez sur **Configurer le proxy** et spécifiez l’adresse du proxy (sous la forme http://ProxyIPAddress ou http://ProxyFQDN) ) et le port d’écoute.
        - Spécifiez les informations d’identification si le proxy nécessite une authentification.
        - Seuls les proxys HTTP sont pris en charge.
        - Si vous avez ajouté des détails de proxy ou désactivé le proxy et/ou l’authentification, cliquez sur **Enregistrer** pour relancer la vérification de la connectivité.
    - **Synchronisation de l’heure** : L’heure est vérifiée. L’heure de l’appliance doit être synchronisée avec l’heure Internet pour que la découverte de serveur fonctionne correctement.
    - **Installer les mises à jour** : L’évaluation de serveur Azure Migrate vérifie que les dernières mises à jour sont installées sur l’appliance. Une fois la vérification terminée, vous pouvez cliquer sur **Afficher les services de l’appliance** pour voir l’état et les versions des composants s’exécutant sur l’appliance.

### <a name="register-the-appliance-with-azure-migrate"></a>Inscrire l’appliance auprès d’Azure Migrate

1. Collez la **clé de projet Azure Migrate** copiée à partir du portail. Si vous n’avez pas la clé, accédez à **Évaluation de serveur > Découvrir > Gérer les appliances existantes**, sélectionnez le nom d’appliance que vous avez indiqué pendant la génération de la clé, puis copiez la clé correspondante.
1. Cliquez sur **Connexion**. Une invite de connexion Azure s’ouvre dans un nouvel onglet du navigateur. S’il n’apparaît pas, vérifiez que vous avez désactivé le bloqueur de fenêtres publicitaires dans le navigateur.
1. Sous le nouvel onglet, connectez-vous avec votre nom d’utilisateur et votre mot de passe Azure.
   
   La connexion avec un code PIN n’est pas prise en charge.
3. Une fois la connexion établie, revenez à l’application web. 
4. Si le compte d’utilisateur Azure utilisé pour se connecter dispose des [autorisations](./tutorial-discover-physical.md) adéquates sur les ressources Azure créées pendant la génération de la clé, l’inscription de l’appliance est lancée.
1. Une fois l’inscription de l’appliance terminée, vous pouvez consulter les détails de l’inscription en cliquant sur **Afficher les détails**.


## <a name="start-continuous-discovery"></a>Démarrer la découverte en continu

À présent, connectez-vous à partir de l’appliance aux serveurs physiques à découvrir, puis démarrez la découverte.

1. À l’**Étape 1 : Fournir les informations d’identification pour la découverte des serveurs physiques ou virtuels Windows et Linux**, cliquez sur **Ajouter les informations d’identification** pour attribuer un nom convivial aux informations d’identification, ajoutez un **nom d’utilisateur** et un **mot de passe** pour un serveur Windows ou Linux. Cliquez sur **Enregistrer**.
1. Si vous souhaitez ajouter plusieurs informations d’identification à la fois, cliquez sur **Ajouter** pour enregistrer et ajouter d’autres informations d’identification. Plusieurs informations d’identification sont prises en charge pour la découverte de serveurs physiques.
1. Dans **Étape 2 : Fournir les détails du serveur physique ou virtuel**, cliquez sur **Ajouter une source de découverte** pour spécifier l’**adresse IP/nom de domaine complet** du serveur ainsi que le nom convivial des informations d’identification pour se connecter au serveur.
1. Vous pouvez soit **ajouter un seul élément** à la fois, soit **ajouter plusieurs éléments** simultanément. Il existe également une option permettant de fournir les détails du serveur via **Importer au format CSV**.

    ![Sélections pour l’ajout de la source de découverte](./media/tutorial-assess-physical/add-discovery-source-physical.png)

    - Si vous choisissez **Ajouter un seul élément**, vous pouvez choisir le type de système d’exploitation, attribuer un nom convivial aux informations d’identification, ajouter l’**adresse IP/nom de domaine complet** du serveur, puis cliquer sur **Enregistrer**.
    - Si vous choisissez **Ajouter plusieurs éléments**, vous pouvez ajouter plusieurs enregistrements à la fois en spécifiant l’**adresse IP/nom de domaine complet** du serveur avec le nom convivial des informations d’identification dans la zone de texte. **Vérifiez** les enregistrements ajoutés, puis cliquez sur **Enregistrer**.
    - Si vous choisissez **Importer au format CSV** _(sélectionné par défaut)_ , vous pouvez télécharger un fichier de modèle CSV, compléter le fichier avec l’**adresse IP/nom de domaine complet** du serveur et le nom convivial des informations d’identification. Importez ensuite le fichier dans l’appliance, **vérifiez** les enregistrements dans le fichier, puis cliquez sur **Enregistrer**.

1. Dès que vous cliquez sur Enregistrer, l’appliance tente de valider la connexion aux serveurs ajoutés et affiche l’**état de validation** de chaque serveur dans le tableau.
    - Si la validation échoue pour un serveur, examinez l’erreur en cliquant sur **Échec de la validation** dans la colonne État du tableau. Corrigez le problème et recommencez la validation.
    - Pour supprimer un serveur, cliquez sur **Supprimer**.
1. Vous pouvez **revalider** la connectivité aux serveurs à tout moment avant de lancer la découverte.
1. Cliquez sur **Démarrer la découverte** pour lancer la découverte des serveurs validés. Une fois la découverte lancée, vous pouvez vérifier l’état de la découverte de chaque serveur dans le tableau.


Ceci démarre la découverte. Il faut environ deux minutes par serveur pour que les métadonnées du serveur détecté s’affichent sur le portail Azure.

## <a name="verify-servers-in-the-portal"></a>Vérifier les serveurs dans le portail

Une fois la découverte terminée, vous pouvez vérifier que les serveurs apparaissent dans le portail.

1. Ouvrez le tableau de bord Azure Migrate.
2. Dans la page **Azure Migrate - Serveurs** > **Azure Migrate : Server Assessment**, cliquez sur l’icône qui affiche le nombre de **Serveurs découverts**.


## <a name="next-steps"></a>Étapes suivantes

Essayez l’[évaluation des serveurs physiques](tutorial-assess-physical.md) avec Azure Migrate Server Assessment.