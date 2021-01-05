---
title: Activer un registre de conteneurs Edge sur un appareil Azure Stack Edge Pro avec GPU
description: Décrit comment activer un registre de conteneurs Edge local sur un appareil Azure Stack Edge Pro avec GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 11/11/2020
ms.author: alkohli
ms.openlocfilehash: bccb6fa33007082737997c7282fb286c38e3bbd7
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465282"
---
# <a name="enable-edge-container-registry-on-your-azure-stack-edge-pro-gpu-device"></a>Activer un registre de conteneurs Edge sur votre appareil Azure Stack Edge Pro avec GPU

Cet article explique comment activer le registre de conteneurs Edge et l’utiliser à partir du cluster Kubernetes sur votre appareil Azure Stack Edge Pro. L’exemple utilisé dans l’article explique comment envoyer une image d’un registre source, dans ce cas, le registre de conteneurs Microsoft, vers le registre de l’appareil Azure Stack Edge, à savoir le registre de conteneurs Edge.

### <a name="about-edge-container-registry"></a>À propos du registre de conteneurs Edge

Les applications de calcul en conteneur s’exécutent sur des images conteneurs et ces images sont stockées dans des registres. Les registres peuvent être publics (tels que Docker Hub), privés ou un fournisseur de services cloud managé tel que Azure Container Registry. Pour plus d’informations, consultez [À propos des registres, des dépôts et des images](../container-registry/container-registry-concepts.md).

Un registre de conteneurs Edge fournit un dépôt Edge, sur votre appareil Azure Stack Edge Pro. Vous pouvez utiliser ce registre pour stocker et gérer vos images conteneurs privées.

Dans un environnement à plusieurs nœuds, les images conteneurs peuvent être téléchargées et envoyées au registre de conteneurs Edge une seule fois. Toutes les applications Edge peuvent utiliser le registre de conteneurs Edge pour les déploiements suivants.


## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez accès à un appareil Azure Stack Edge Pro.

1. Vous avez activé votre appareil Azure Stack Edge Pro, comme décrit dans [Activer Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).

1. Vous avez activé le rôle de calcul sur l’appareil. Un cluster Kubernetes a également été créé sur l’appareil quand vous avez configuré le calcul sur l’appareil conformément aux instructions données dans [Configurer le calcul sur votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-configure-compute.md).

1. Vous avez accès au point de terminaison de l’API Kubernetes depuis la page **Appareil** de votre interface utilisateur web locale. Pour plus d’informations, consultez les instructions données dans [Obtenir le point de terminaison de l’API Kubernetes](azure-stack-edge-gpu-deploy-configure-compute.md#get-kubernetes-endpoints).

1. Vous avez accès à un système client avec un [système d’exploitation pris en charge](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). Si vous utilisez un client Windows, le système doit exécuter PowerShell 5.0 ou une version ultérieure pour accéder à l’appareil.

    1. Si vous souhaitez extraire et envoyer vos propres images conteneurs, assurez-vous que le client Docker est installé sur le système. Si vous utilisez un client Windows, [installez Docker Deskop sur Windows](https://docs.docker.com/docker-for-windows/install/).  


## <a name="enable-container-registry-as-add-on"></a>Activer le registre de conteneurs en tant que module complémentaire

La première étape consiste à activer le registre de conteneurs Edge en tant que module complémentaire.

1. [Connectez-vous à l'interface PowerShell de l'appareil](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface). 

1. Pour activer le registre de conteneurs en tant que module complémentaire, tapez : 

    `Set-HcsKubernetesContainerRegistry`
    
    Cette opération peut prendre plusieurs minutes.

    Voici un exemple de sortie de cette commande :  
            
    ```powershell
    [10.128.44.40]: PS>Set-HcsKubernetesContainerRegistry
    Operation completed successfully. Use Get-HcsKubernetesContainerRegistryInfo for credentials    
    ```
            
1. Pour afficher les détails du registre de conteneurs, tapez :

    `Get-HcsKubernetesContainerRegistryInfo`

    Voici un exemple de sortie de cette commande :  
    
    ```powershell
    [10.128.44.40]: PS> Get-HcsKubernetesContainerRegistryInfo
                
    Endpoint                                   IPAddress    Username     Password
    --------                                   ---------    --------     --------
    ecr.dbe-hw6h1t2.microsoftdatabox.com:31001 10.128.44.41 ase-ecr-user i3eTsU4zGYyIgxV
    ``` 

1. Notez le nom d’utilisateur et le mot de passe figurant dans la sortie de `Get-HcsKubernetesContainerRegistryInfo`. Ces informations d’identification sont utilisées pour se connecter au registre de conteneurs Edge lors de l’envoi des images.         


## <a name="manage-container-registry-images"></a>Gérer les images du registre de conteneurs

Vous voulez peut-être accéder au registre de conteneurs en dehors de votre appareil Azure Stack Edge ou envoyer/extraire des images dans le registre.

Pour accéder au registre de conteneurs Edge, procédez comme suit :

1. Obtenez les détails du point de terminaison pour le registre de conteneurs Edge.
    1. Dans l’interface utilisateur locale de l’appareil, accédez à **Appareil**.
    1. Localisez le **point de terminaison du registre du conteneur Edge**.
        ![Page Point de terminaison du registre de conteneurs Edge sur l’appareil](media/azure-stack-edge-gpu-edge-container-registry/get-edge-container-registry-endpoint-1.png) 
    1. Copiez ce point de terminaison et créez une entrée DNS correspondante dans le fichier `C:\Windows\System32\Drivers\etc\hosts` de votre client pour la connexion au point de terminaison du registre de conteneurs Edge. 

        <IP address of the Kubernetes main node>    <Edge container registry endpoint> 
        
        ![Ajouter une entrée DNS pour le point de terminaison du registre de conteneurs Edge](media/azure-stack-edge-gpu-edge-container-registry/add-domain-name-service-entry-hosts-1.png)    

1. Téléchargez le certificat du registre de conteneurs Edge à partir de l’interface utilisateur locale. 
    1. Dans l’interface utilisateur locale de votre appareil, accédez à **Certificats**.
    1. Recherchez l’entrée correspondant à **Certificat du registre de conteneurs Edge**. À droite de cette entrée, sélectionnez **Télécharger** pour télécharger le certificat du registre de conteneurs Edge sur votre système client, que vous utiliserez pour accéder à votre appareil. 

        ![Télécharger le certificat de point de terminaison du registre de conteneurs Edge](media/azure-stack-edge-gpu-edge-container-registry/download-edge-container-registry-endpoint-certificate-1.png)  

1. Installez le certificat téléchargé sur le client. Si vous utilisez un client Windows, procédez comme suit : 
    1. Sélectionnez le certificat, puis dans l’Assistant **Importation de certificat**, sélectionnez l’emplacement de magasin **Ordinateur local**. 

        ![Installer le certificat 1](media/azure-stack-edge-gpu-edge-container-registry/install-certificate-1.png) 
    
    1. Installez le certificat sur votre ordinateur local dans le magasin racine approuvé. 

        ![Installer le certificat 2](media/azure-stack-edge-gpu-edge-container-registry/install-certificate-2.png) 

1. Une fois le certificat installé, redémarrez le client Docker sur votre système.

1. Connectez-vous au registre de conteneurs Edge. Tapez :

    `docker login <Edge container registry endpoint> -u <username> -p <password>`

    Indiquez le point de terminaison de registre de conteneurs Edge à partir de la page **Appareils**, ainsi que le nom d’utilisateur et le mot de passe que vous avez obtenus dans la sortie de `Get-HcsKubernetesContainerRegistryInfo`. 

1. Utilisez les commandes Docker push ou pull pour envoyer ou extraire des images conteneurs du registre de conteneurs.
 
    1. Extrayez une image à partir de l’image de registre de conteneurs Microsoft. Tapez :
        
        `docker pull <Full path to the container image in the Microsoft Container Registry>`
       
    1. Créez un alias de l’image que vous avez extraite avec le chemin d’accès complet à votre registre.

        `docker tag <Path to the image in the Microsoft container registry> <Path to the image in the Edge container registry/Image name with tag>`

    1. Envoyez l’image vers votre registre.
    
        `docker push <Path to the image in the Edge container registry/Image name with tag>`

    1. Exécutez l’image que vous avez envoyée dans votre registre.
    
        `docker run -it --rm -p 8080:80 <Path to the image in the Edge container registry/Image name with tag>`

        Voici un exemple de sortie des commandes pull et push :

        ```powershell
        PS C:\WINDOWS\system32> docker login ecr.dbe-hw6h1t2.microsoftdatabox.com:31001 -u ase-ecr-user -p 3bbo2sOtDe8FouD
        WARNING! Using --password via the CLI is insecure. Use --password-stdin.
        Login Succeeded
        PS C:\WINDOWS\system32> docker pull mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine
        1.17.5-alpine: Pulling from oss/nginx/nginx
        Digest: sha256:5466bbc0a989bd1cd283c0ba86d9c2fc133491ccfaea63160089f47b32ae973b
        Status: Image is up to date for mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine
        mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine
        PS C:\WINDOWS\system32> docker tag mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        PS C:\WINDOWS\system32> docker push ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        The push refers to repository [ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx]
        bba7d2385bc1: Pushed
        77cae8ab23bf: Pushed
        2.0: digest: sha256:b4c0378c841cd76f0b75bc63454bfc6fe194a5220d4eab0d75963bccdbc327ff size: 739
        PS C:\WINDOWS\system32> docker run -it --rm -p 8080:80 ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        2020/11/10 00:00:49 [error] 6#6: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080", referrer: "http://localhost:8080/"
        172.17.0.1 - - [10/Nov/2020:00:00:49 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://localhost:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36" "-"
        ^C
        PS C:\WINDOWS\system32>    
        ```
    
1. Accédez à `http://localhost:8080` pour afficher le conteneur en cours d’exécution. Dans ce cas, le serveur web Nginx s’affiche.

    ![Afficher le conteneur en cours d’exécution](media/azure-stack-edge-gpu-edge-container-registry/view-running-container-1.png)

    Pour arrêter et supprimer le conteneur, appuyez sur `Control+C`.

 

## <a name="use-edge-container-registry-images-via-kubernetes-pods"></a>Utiliser des images de registre de conteneurs Edge via des pods Kubernetes

Vous pouvez maintenant déployer l’image que vous avez envoyée dans votre registre de conteneurs Edge à partir des pods Kubernetes.

1. Pour déployer l’image, vous devez configurer l’accès au cluster via *kubectl*. Créez un espace de noms, un utilisateur, accordez à l’utilisateur l’accès à l’espace de noms et récupérez un fichier *config*. Assurez-vous que vous pouvez vous connecter aux pods Kubernetes. 
    
    Suivez les étapes décrites dans [Se connecter à un cluster Kubernetes et le gérer par le biais de kubectl sur votre appareil Azure Stack Edge Pro avec GPU](azure-stack-edge-gpu-create-kubernetes-cluster.md). 

    Voici un exemple de sortie pour un espace de noms sur votre appareil à partir duquel l’utilisateur peut accéder au cluster Kubernetes.

    ```powershell
    [10.128.44.40]: PS>New-HcsKubernetesNamespace -Namespace myecr
    [10.128.44.40]: PS>New-HcsKubernetesUser -UserName ecruser
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01URXdOVEF6TkRJek1Gb1hEVE13TVRFd016QXpOREl6TUZvd0ZURVRNQkVnNjOVRLWndCQ042cm1XQms2eXFwcXI1MUx6bApTaXMyTy91UEJ2YXNSSUUzdzgrbmEwdG1aTERZZ2F6MkQwMm42Q29mUmtyUTR2d1lLTnR1MlpzR3pUdz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
        server: https://compute.dbe-hw6h1t2.microsoftdatabox.com:6443
        name: kubernetes
        ===================CUT=========================================CUT==============
        client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMwRENDQWJpZ0F3SUJBZ0lJYmVWRGJSTzZ3ell3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURFeE1EVXdNelF5TXpCYUZ3MHlNVEV4TURreU16UTRNal
        ===================CUT=========================================CUT==============
        DMVUvN3lFOG5UU3k3b2VPWitUeHdzCjF1UDByMjhDZ1lCdHdRY0ZpcFh1blN5ak16dTNIYjhveFI2V3VWWmZldFFKNElKWEFXOStVWGhKTFhyQ2x4bUcKWHRtbCt4UU5UTzFjQVNKRVZWVDd6Tjg2ay9kSU43S3JIVkdUdUxlUDd4eGVjV2VRcWJrZEVScUsxN0liTXpiVApmbnNxc0dobEdmLzdmM21kTGtyOENrcWs5TU5aM3MvUVIwRlFCdk94ZVpuUlpTeDVGbUR5S1E9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

    [10.128.44.40]: PS>Grant-HcsKubernetesNamespaceAccess -Namespace myecr -UserName ecruser
    [10.128.44.40]: PS>kubectl get pods -n "myecr"
    No resources found.
    PS C:\WINDOWS\system32>
    ```  

2. Les secrets d’extraction d’images sont déjà définis dans tous les espaces de noms Kubernetes sur votre appareil. Vous pouvez vous procurer des secrets à l’aide de la commande `get secrets`. Voici un exemple de sortie :

    ```powershell
    PS C:\WINDOWS\system32> .\kubectl.exe get secrets -n myecr
    NAME                  TYPE                                  DATA   AGE
    ase-ecr-credentials   kubernetes.io/dockerconfigjson        1      99m
    default-token-c7kww   kubernetes.io/service-account-token   3      107m
    sec-smbcredentials    microsoft.com/smb                     2      99m
    PS C:\WINDOWS\system32>   
    ```    

3. Déployez un pod sur votre espace de noms à l’aide de kubectl. Utilisez le code `yaml` suivant. 

    Remplacez l’image `<image-name>` par l’image envoyée au registre de conteneurs. Faites référence aux secrets de vos espaces de noms à l’aide de imagePullSecrets avec un nom `ase-ecr-credentials`.
    
    ```yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        imagePullPolicy: Always
      imagePullSecrets:
      - name: ase-ecr-credentials
    ```

4. Appliquez le déploiement dans l’espace de noms que vous avez créé à l’aide de la commande apply. Vérifiez que le conteneur est en cours d’exécution. Voici un exemple de sortie :
   
    ```yml
    PS C:\Windows\System32> .\kubectl.exe apply -f .\deployment.yml -n myecr
    pod/nginx configured
    PS C:\Windows\System32> .\kubectl.exe get pods -n myecr
    NAME    READY   STATUS    RESTARTS   AGE
    nginx   1/1     Running   0          27m
    PS C:\Windows\System32>
    ```

## <a name="delete-container-registry-images"></a>Supprimer des images du registre de conteneurs

L’espace de stockage du registre de conteneurs Edge est hébergé sur un partage local au sein de votre appareil Azure Stack Edge Pro, qui est limité par le stockage disponible sur l’appareil. Il vous incombe de supprimer du registre de conteneurs les images Docker inutilisées à l’aide de l’API Docker HTTP v2 (https://docs.docker.com/registry/spec/api/).  

Pour supprimer une ou plusieurs images conteneurs, procédez comme suit :

1. Définissez le nom de l’image sur l’image que vous souhaitez supprimer.

    ```powershell
    PS C:\WINDOWS\system32> $imageName="nginx"    
    ```

1. Définir le nom d’utilisateur et le mot de passe du registre de conteneurs en tant qu’informations d’identification PS

    ```powershell
    PS C:\WINDOWS\system32> $username="ase-ecr-user"
    PS C:\WINDOWS\system32> $password="3bbo2sOtDe8FouD"
    PS C:\WINDOWS\system32> $securePassword = ConvertTo-SecureString $password -AsPlainText -Force
    PS C:\WINDOWS\system32> $credential = New-Object System.Management.Automation.PSCredential ($username, $securePassword)    
    ```

1. Répertorier les balises associées à l’image

    ```powershell
    PS C:\WINDOWS\system32> $tags = Invoke-RestMethod -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/nginx/tags/list" | Select-Object -ExpandProperty tags
    PS C:\WINDOWS\system32> $tags
    2.0
    PS C:\WINDOWS\system32> $tags = Invoke-RestMethod -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/$imageName/tags/list" | Select-Object -ExpandProperty tags
    PS C:\WINDOWS\system32> $tags
    2.0
    PS C:\WINDOWS\system32>    
    ```
  
1. Répertoriez le condensé associé à la balise que vous souhaitez supprimer. Cela utilise $tags à partir de la sortie de la commande ci-dessus. Si vous avez plusieurs balises, sélectionnez l’une d’entre elles et utilisez la commande suivante.

    ```powershell
    PS C:\WINDOWS\system32> $response = Invoke-WebRequest -Method Head -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/$imageName/manifests/$tags" -Headers @{ 'Accept' = 'application/vnd.docker.distribution.manifest.v2+json' }
    PS C:\WINDOWS\system32> $digest = $response.Headers['Docker-Content-Digest']
    PS C:\WINDOWS\system32> $digest
    sha256:b4c0378c841cd76f0b75bc63454bfc6fe194a5220d4eab0d75963bccdbc327ff
    PS C:\WINDOWS\system32>    
    ```
1. Supprimer l’image à l’aide du condensé de image:tag

    ```powershell
    PS C:\WINDOWS\system32> Invoke-WebRequest -Method Delete -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/$imageName/manifests/$digest" | Select-Object -ExpandProperty StatusDescription    
    ```

Une fois que vous avez supprimé les images inutilisées, l’espace associé aux images non référencées est automatiquement récupéré par un processus qui s’exécute la nuit. 

## <a name="next-steps"></a>Étapes suivantes

- [Déployez une application sans état sur votre Azure Stack Edge Pro](azure-stack-edge-j-series-deploy-stateless-application-kubernetes.md).
