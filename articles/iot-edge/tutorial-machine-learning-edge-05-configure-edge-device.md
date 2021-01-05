---
title: 'Tutoriel : Configurer un appareil IoT Edge - Machine Learning sur Azure IoT Edge'
description: Dans ce tutoriel, vous allez configurer une machine virtuelle Azure exécutant Linux en tant qu’appareil Azure IoT Edge servant de passerelle transparente.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 2/5/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.custom: amqp, devx-track-azurecli
ms.openlocfilehash: bc62590b9517b2c6d16fdf2637990b845248d2ec
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932351"
---
# <a name="tutorial-configure-an-iot-edge-device"></a>Tutoriel : Configurer un appareil IoT Edge

Dans cet article, nous allons configurer une machine virtuelle Azure qui exécute Linux comme un appareil IoT Edge servant de passerelle transparente. La configuration d’une passerelle transparente permet aux appareils de se connecter à Azure IoT Hub à travers la passerelle sans savoir que la passerelle existe. De la même façon, l’utilisateur qui interagit avec les appareils dans Azure IoT Hub n’a pas conscience de l’appareil de passerelle intermédiaire. À la fin, nous ajouterons des données d’analytique Edge à notre système en ajoutant des modules IoT Edge à la passerelle transparente.

Les étapes décrites dans cet article sont généralement effectuées par un développeur de cloud.

Dans cette section du tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créer des certificats pour permettre à votre appareil de passerelle de se connecter de façon sécurisée à vos appareils en aval.
> * Créer un appareil IoT Edge.
> * Créer une machine virtuelle Azure pour simuler votre appareil IoT Edge.

## <a name="prerequisites"></a>Prérequis

Cet article fait partie d’une série décrivant l’utilisation d’Azure Machine Learning sur IoT Edge. Chaque article de la série s’appuie sur le travail réalisé dans le cadre de l’article précédent. Si vous êtes tombé directement sur cet article, reportez-vous au [premier article](tutorial-machine-learning-edge-01-intro.md) de la série.

## <a name="create-certificates"></a>Créer des certificats

Pour qu’un appareil fonctionne comme une passerelle, il doit être capable de se connecter de manière sécurisée aux appareils en aval. Azure IoT Edge vous permet d’utiliser une infrastructure à clé publique (PKI) pour configurer des connexions sécurisées entre les appareils. Dans ce cas, nous autorisons un appareil IoT en aval à se connecter à un appareil IoT Edge faisant office de passerelle transparente. Pour assurer un niveau raisonnable de sécurité, l’appareil en aval doit confirmer l’identité de l’appareil IoT Edge. Pour plus d’informations sur la façon dont les appareils IoT Edge utilisent des certificats, consultez [Détails d’utilisation des certificats Azure IoT Edge](iot-edge-certs.md).

Dans cette section, nous créons les certificats auto-signés à l’aide d’une image Docker que nous devons générer et exécuter. Nous avons choisi d’utiliser une image Docker pour effectuer cette étape, car elle réduit considérablement le nombre d’étapes nécessaires pour créer les certificats sur la machine de développement Windows. Consultez [Créer des certificats de démonstration pour tester des fonctionnalités d’appareils IoT Edge](how-to-create-test-certificates.md) pour comprendre ce que nous avons automatisé avec l’image Docker.

1. Connectez-vous à votre machine virtuelle de développement.

2. Créez un dossier avec le chemin et le nom suivants : `c:\edgeCertificates`.

3. S’il n’est pas déjà en cours d’exécution, démarrez **Docker pour Windows** à partir du menu Démarrer de Windows.

4. Ouvrez Visual Studio Code.

5. Sélectionnez **Fichier** > **Ouvrir le dossier...** et choisissez **C:\\source\\IoTEdgeAndMlSample\\CreateCertificates**.

6. Dans le volet de l’Explorateur, cliquez avec le bouton droit sur le fichier **dockerfile** et choisissez **Générer l’image**.

7. Dans la boîte de dialogue, acceptez la valeur par défaut du nom et de l’étiquette de l’image : **createcertificates: latest**.

    ![Créer des certificats dans Visual Studio Code](media/tutorial-machine-learning-edge-05-configure-edge-device/create-certificates.png)

8. Attendez la fin de la génération.

    > [!NOTE]
    > Vous risquez de voir un avertissement indiquant qu’une clé publique est manquante. Vous pouvez ignorer cet avertissement. De même, vous voyez un avertissement de sécurité qui vous recommande de vérifier/réinitialiser les autorisations sur votre image. Vous pouvez l’ignorer pour cette image.

9. Dans la fenêtre de terminal Visual Studio Code, exécutez le conteneur createcertificates.

    ```cmd
    docker run --name createcertificates --rm -v c:\edgeCertificates:/edgeCertificates createcertificates /edgeCertificates
    ```

10. Docker vous invite à accéder au lecteur **c:\\** . Sélectionnez **Partager**.

11. À l’invite, indiquez vos informations d’identification.

12. Dès que le conteneur a fini son exécution, recherchez les fichiers suivants dans **c:\\edgeCertificates** :

    * c:\\edgeCertificates\\certs\\azure-iot-test-only.root.ca.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device-full-chain.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device.cert.pem
    * c:\\edgeCertificates\\certs\\new-edge-device.cert.pfx
    * c:\\edgeCertificates\\private\\new-edge-device.key.pem

## <a name="upload-certificates-to-azure-key-vault"></a>Charger des certificats dans Azure Key Vault

Pour stocker nos certificats de manière sécurisée et les rendre accessibles sur plusieurs appareils, nous les chargeons dans Azure Key Vault. Comme vous pouvez le voir dans la liste ci-dessus, nous avons deux types de fichiers de certificat : PFX et PEM. Les fichiers PFX correspondent aux certificats de coffre de clés à charger dans Key Vault. Les fichiers PEM sont en texte intégral et correspondent aux secrets de coffre de clés. Nous utilisons le coffre de clés associé à l’espace de travail Azure Machine Learning que nous avons créé en exécutant les [notebooks Azure](tutorial-machine-learning-edge-04-train-model.md#run-azure-notebooks).

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre espace de travail Azure Machine Learning.

2. Dans la page Vue d’ensemble de l’espace de travail Azure Machine Learning, recherchez le nom du **coffre de clés**.

    ![Copier le nom du coffre de clés](media/tutorial-machine-learning-edge-05-configure-edge-device/find-key-vault-name.png)

3. Sur votre machine de développement, chargez les certificats dans Key Vault. Remplacez **\<subscriptionId\>** et **\<keyvaultname\>** par vos informations de ressource.

    ```powershell
    c:\source\IoTEdgeAndMlSample\CreateCertificates\upload-keyvaultcerts.ps1 -SubscriptionId <subscriptionId> -KeyVaultName <keyvaultname>
    ```

4. Si vous y êtes invité, connectez-vous à Azure.

5. Le script s’exécute pendant quelques minutes et génère la liste des nouvelles entrées Key Vault.

    ![Sortie du script Key Vault](media/tutorial-machine-learning-edge-05-configure-edge-device/key-vault-entries-output.png)

## <a name="create-iot-edge-device"></a>Créer un appareil IoT Edge

Pour connecter un appareil Azure IoT Edge à un hub IoT, nous créons d’abord une identité pour l’appareil dans le hub. Nous prenons la chaîne de connexion de l’identité d’appareil dans le cloud et l’utilisons pour configurer le runtime sur notre appareil IoT Edge. Une fois que l’appareil configuré se connecte au hub, nous pouvons déployer des modules et envoyer des messages. Nous pouvons aussi modifier la configuration de l’appareil IoT Edge physique en changeant son identité d’appareil correspondante dans le hub IoT.

Pour ce tutoriel, nous créons la nouvelle identité d’appareil à l’aide de Visual Studio Code. Vous pouvez également effectuer ces étapes en utilisant le portail Azure ou Azure CLI.

1. Sur votre machine de développement, ouvrez Visual Studio Code.

2. Développez le cadre **Azure IoT Hub** dans la vue de l’Explorateur Visual Studio Code.

3. Cliquez sur les points de suspension et sélectionnez **Créer un appareil IoT Edge**.

4. Donnez un nom à l’appareil. Pour des raisons pratiques, nous utilisons le nom **aaTurbofanEdgeDevice** afin qu’il s’affiche en haut de la liste des appareils.

5. Le nouvel appareil apparaît dans la liste des appareils.

    ![Voir le nouvel appareil aaTurbofanEdgeDevice dans l’Explorateur VS Code](media/tutorial-machine-learning-edge-05-configure-edge-device/iot-hub-devices-list.png)

## <a name="deploy-azure-virtual-machine"></a>Déployer une machine virtuelle Azure

Nous utilisons l’image [Azure IoT Edge sur Ubuntu](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft_iot_edge.iot_edge_vm_ubuntu?tab=Overview) de la Place de marché Azure pour créer notre appareil IoT Edge dans ce tutoriel. L’image Azure IoT Edge sur Ubuntu installe le dernier runtime Azure IoT Edge et ses dépendances au démarrage. Nous déployons la machine virtuelle à l’aide d’un script PowerShell, `Create-EdgeVM.ps1`, d’un modèle Resource Manager, `IoTEdgeVMTemplate.json`, et d’un script d’interpréteur de commandes, `install packages.sh`.

### <a name="enable-programmatic-deployment"></a>Activer le déploiement par programmation

Pour utiliser l’image de la Place de marché dans un déploiement scripté, nous devons activer le déploiement par programmation de l’image.

1. Connectez-vous au portail Azure.

1. Sélectionnez **Tous les services**.

1. Dans la barre de recherche, entrez et sélectionnez **Place de marché**.

1. Dans la barre de recherche de la Place de marché, entrez et sélectionnez **Azure IoT Edge sur Ubuntu**.

1. Sélectionnez le lien hypertexte **Bien démarrer** pour effectuer le déploiement programmatiquement.

1. Sélectionnez le bouton **Activer**, puis **Enregistrer**.

    ![Activer le déploiement par programmation des machines virtuelles](media/tutorial-machine-learning-edge-05-configure-edge-device/deploy-ubuntu-vm.png)

1. Vous voyez une notification de réussite.

### <a name="create-virtual-machine"></a>Créer une machine virtuelle

Ensuite, exécutez le script afin de créer la machine virtuelle pour votre appareil IoT Edge.

1. Ouvrez une fenêtre PowerShell et accédez au répertoire **EdgeVM**.

    ```powershell
    cd c:\source\IoTEdgeAndMlSample\EdgeVM
    ```

2. Exécutez le script suivant pour créer la machine virtuelle.

    ```powershell
    .\Create-EdgeVm.ps1
    ```

3. Quand vous y êtes invité, indiquez des valeurs pour chaque paramètre. Pour l’abonnement, le groupe de ressources et l’emplacement, nous vous recommandons d’utiliser les mêmes valeurs que celles utilisées pour toutes les ressources tout au long de ce tutoriel.

    * **ID d’abonnement Azure** : disponible dans le portail Azure
    * **Nom du groupe de ressources** : un nom facile à mémoriser pour regrouper les ressources dans ce tutoriel
    * **Emplacement** : Emplacement Azure où la machine virtuelle est créée. Par exemple, westus2 ou northeurope. Pour plus d’informations, consultez tous les [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/).
    * **AdminUsername** : nom du compte administrateur à utiliser pour la connexion à la machine virtuelle
    * **AdminPassword** : mot de passe à définir pour AdminUsername sur la machine virtuelle

4. Pour que le script puisse configurer la machine virtuelle, vous devez vous connecter à Azure avec les informations d’identification associées à l’abonnement Azure que vous utilisez.

5. Le script vérifie les informations pour la création de votre machine virtuelle. Sélectionnez **y** ou **Entrée** pour continuer.

6. Pendant plusieurs minutes, le script exécute les étapes suivantes :

    * Créer le groupe de ressources s’il n’existe pas déjà
    * Créer la machine virtuelle
    * Ajouter des exceptions NSG pour la machine virtuelle sur les ports 22 (SSH), 5671 (AMQP), 5672 (AMPQ) et 443 (TLS)
    * Installer [Azure CLI](/cli/azure/install-azure-cli-apt)

7. Le script génère la chaîne de connexion SSH permettant de se connecter à la machine virtuelle. Copiez la chaîne de connexion pour la prochaine étape.

    ![Copiez la chaîne de connexion SSH pour la machine virtuelle](media/tutorial-machine-learning-edge-05-configure-edge-device/vm-ssh-connection-string.png)

## <a name="connect-to-your-iot-edge-device"></a>Se connecter à votre appareil IoT Edge

Les différentes sections suivantes configurent la machine virtuelle Azure que nous avons créée. La première étape consiste à se connecter à la machine virtuelle.

1. Ouvrez une invite de commandes et collez la chaîne de connexion SSH que vous avez copiée à partir de la sortie du script. Entrez vos propres informations pour le nom d’utilisateur, le suffixe et la région en fonction des valeurs que vous avez fournies au script PowerShell dans la section précédente.

    ```cmd
    ssh -l <username> iotedge-<suffix>.<region>.cloudapp.azure.com
    ```

2. Quand vous êtes invité à valider l’authenticité de l’hôte, tapez **yes** et sélectionnez **Entrée**.

3. Quand vous y êtes invité, indiquez votre mot de passe.

4. Ubuntu affiche un message de bienvenue, puis une invite de type `<username>@<machinename>:~$`.

## <a name="download-key-vault-certificates"></a>Télécharger les certificats de coffre de clés

Plus haut dans cet article, nous avons chargé des certificats dans Key Vault afin de les rendre disponibles pour notre appareil IoT Edge et notre appareil de nœud terminal. L’appareil de nœud terminal est un appareil en aval qui utilise l’appareil IoT Edge comme passerelle pour communiquer avec IoT Hub.

Nous décrivons l’appareil de nœud terminal plus loin dans le tutoriel. Dans cette section, téléchargez les certificats sur l’appareil IoT Edge.

1. À partir de la session SSH sur la machine virtuelle Linux, connectez-vous à Azure avec Azure CLI.

    ```azurecli
    az login
    ```

1. Vous êtes invité à ouvrir un navigateur sur <https://microsoft.com/devicelogin> et fournir un code unique. Vous pouvez effectuer ces étapes sur votre machine locale. Fermez la fenêtre de navigateur dès que vous êtes authentifié.

1. Quand l’authentification est correcte, la machine virtuelle Linux se connecte et liste vos abonnements Azure.

1. Sélectionnez l’abonnement Azure à utiliser pour les commandes Azure CLI.

    ```azurecli
    az account set --subscription <subscriptionId>
    ```

1. Créez un répertoire sur la machine virtuelle pour les certificats.

    ```bash
    sudo mkdir /edgeMlCertificates
    ```

1. Téléchargez les certificats que vous avez stockés dans le coffre de clés : new-edge-device-full-chain.cert.pem, new-edge-device.key.pem et azure-iot-test-only.root.ca.cert.pem

    ```azurecli
    key_vault_name="<key vault name>"
    sudo az keyvault secret download --vault-name $key_vault_name --name new-edge-device-full-chain-cert-pem -f /edgeMlCertificates/new-edge-device-full-chain.cert.pem
    sudo az keyvault secret download --vault-name $key_vault_name --name new-edge-device-key-pem -f /edgeMlCertificates/new-edge-device.key.pem
    sudo az keyvault secret download --vault-name $key_vault_name --name azure-iot-test-only-root-ca-cert-pem -f /edgeMlCertificates/azure-iot-test-only.root.ca.cert.pem
    ```

## <a name="update-the-iot-edge-device-configuration"></a>Mettre à jour la configuration de l’appareil IoT Edge

Le runtime IoT Edge utilise le fichier `/etc/iotedge/config.yaml` pour conserver sa configuration. Nous devons mettre à jour trois informations dans ce fichier :

* **Chaîne de connexion d’appareil** : chaîne de connexion de l’identité de cet appareil dans IoT Hub
* **Certificats :** certificats à utiliser pour les connexions établies avec les appareils en aval
* **Nom d’hôte :** nom de domaine complet (FQDN) de l’appareil IoT Edge de la machine virtuelle.

L’image *Azure IoT Edge sur Ubuntu* que nous avons utilisée pour créer la machine virtuelle IoT Edge est fournie avec un script d’interpréteur de commandes qui met à jour config.yaml avec la chaîne de connexion.

1. Dans Visual Studio Code, cliquez avec le bouton droit sur l’appareil IoT Edge, puis sélectionnez **Copier la chaîne de connexion de l’appareil**.

    ![Copier la chaîne de connexion à partir de Visual Studio Code](media/tutorial-machine-learning-edge-05-configure-edge-device/copy-device-connection-string-command.png)

2. Dans votre session SSH, exécutez la commande pour mettre à jour le fichier config.yaml avec la chaîne de connexion de votre appareil.

    ```bash
    sudo /etc/iotedge/configedge.sh "<your_iothub_edge_device_connection_string>"
    ```

Par la suite, nous mettons à jour les certificats et le nom d’hôte en modifiant directement config.yaml.

1. Ouvrez le fichier config.yaml.

    ```bash
    sudo nano /etc/iotedge/config.yaml
    ```

2. Mettez à jour la section des certificats de config.yaml en supprimant le `#` du début et en définissant le chemin pour que le fichier ressemble à l’exemple suivant :

    ```yaml
    certificates:
      device_ca_cert: "/edgeMlCertificates/new-edge-device-full-chain.cert.pem"
      device_ca_pk: "/edgeMlCertificates/new-edge-device.key.pem"
      trusted_ca_certs: "/edgeMlCertificates/azure-iot-test-only.root.ca.cert.pem"
    ```

    Vérifiez que la ligne **certificates:** n’est pas précédée d’un espace et que chaque certificat imbriqué est mis en retrait de deux espaces.

    Dans nano, cliquez avec le bouton droit pour coller le contenu de votre Presse-papiers à la position actuelle du curseur. Pour remplacer la chaîne, utilisez les touches de direction pour accéder à la chaîne à remplacer, supprimez la chaîne, puis cliquez avec le bouton droit pour coller à partir de la mémoire tampon.

3. Dans le portail Azure, accédez à votre machine virtuelle. Copiez le nom DNS (FQDN de la machine) à partir de la section **Vue d’ensemble**.

4. Collez le FQDN dans la section du nom d’hôte de config.yaml. Vérifiez que le nom est en minuscules.

    ```yaml
    hostname: '<machinename>.<region>.cloudapp.azure.com'
    ```

5. Enregistrez et fermez le fichier (`Ctrl + X`, `Y`, `Enter`).

6. Redémarrez le démon iotedge.

    ```bash
    sudo systemctl restart iotedge
    ```

7. Vérifiez l’état du démon IoT Edge (après la commande, tapez « :q » pour quitter).

    ```bash
    systemctl status iotedge
    ```

8. Si vous voyez des erreurs (texte en couleur avec le préfixe « \[ERROR\] ») dans l’état, examinez les journaux du démon pour avoir des informations détaillées sur l’erreur.

    ```bash
    journalctl -u iotedge --no-pager --no-full
    ```
## <a name="clean-up-resources"></a>Nettoyer les ressources

Ce tutoriel fait partie d’un ensemble où chaque article s’appuie sur le travail effectué dans les articles précédents. Ne nettoyez pas les ressources avant d’avoir terminé le dernier tutoriel.

## <a name="next-steps"></a>Étapes suivantes

Nous venons de terminer la configuration d’une machine virtuelle Azure comme passerelle transparente d’Azure IoT Edge. Nous avons d’abord généré des certificats de test que nous avons chargés dans Azure Key Vault. Ensuite, nous avons utilisé un script et un modèle Resource Manager pour déployer la machine virtuelle avec l’image « Ubuntu Server 16.04 LTS + Azure IoT Edge runtime » de la Place de marché Azure. Une fois la machine virtuelle en cours d’exécution, nous nous sommes connectés via SSH. Nous nous sommes connectés à Azure et nous avons téléchargé des certificats à partir de Key Vault. Nous avons effectué plusieurs mises à jour de la configuration du runtime IoT Edge en mettant à jour le fichier config.yaml.

Passez à l’article suivant pour générer des modules IoT Edge.

> [!div class="nextstepaction"]
> [Créer et déployer des modules IoT Edge personnalisés](tutorial-machine-learning-edge-06-custom-modules.md)
