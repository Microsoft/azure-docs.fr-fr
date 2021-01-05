---
title: Créer et gérer un cluster Kubernetes sur un appareil Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Décrit comment créer et gérer un cluster Kubernetes sur un appareil Azure Stack Edge Pro avec GPU par le biais de l’interface Windows PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 6f6d2b126cd9a0acbbbdb8d17ce7345bbf17b556
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94635906"
---
# <a name="connect-to-and-manage-a-kubernetes-cluster-via-kubectl-on-your-azure-stack-edge-pro-gpu-device"></a>Se connecter à un cluster Kubernetes et le gérer par le biais de kubectl sur votre appareil Azure Stack Edge Pro avec GPU

Sur votre appareil Azure Stack Edge Pro, un cluster Kubernetes est créé lorsque vous configurez le rôle de calcul. Une fois le cluster Kubernetes créé, vous pouvez vous y connecter et le gérer localement à partir d’un ordinateur client par le biais d’un outil natif comme *kubectl*.

Cet article explique comment se connecter à un cluster Kubernetes sur votre appareil Azure Stack Edge Pro, puis comment le gérer avec *kubectl*. 


## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez accès à un appareil Azure Stack Edge Pro.

2. Vous avez activé votre appareil Azure Stack Edge Pro, comme décrit dans [Activer Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).

3. Vous avez activé le rôle de calcul sur l’appareil. Un cluster Kubernetes a également été créé sur l’appareil quand vous avez configuré le calcul sur l’appareil conformément aux instructions données dans [Configurer le calcul sur votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-configure-compute.md).

4. Vous avez accès à un système client Windows exécutant PowerShell 5.0 ou une version ultérieure pour accéder à l’appareil. Vous pouvez également utiliser un autre client avec un [système d’exploitation pris en charge](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). 

5. Vous avez accès au point de terminaison de l’API Kubernetes depuis la page **Appareil** de votre interface utilisateur web locale. Pour plus d’informations, consultez les instructions données dans [Obtenir le point de terminaison de l’API Kubernetes](azure-stack-edge-gpu-deploy-configure-compute.md#get-kubernetes-endpoints).


## <a name="connect-to-powershell-interface"></a>Se connecter à l’interface PowerShell

Une fois le cluster Kubernetes créé, vous pouvez y accéder pour créer des espaces de noms et des utilisateurs, puis attribuer des espaces de noms aux utilisateurs. Pour cela, vous devez vous connecter à l’interface PowerShell de l’appareil. Effectuez les étapes suivantes sur le client Windows exécutant PowerShell.

[!INCLUDE [Connect to admin runspace](../../includes/azure-stack-edge-gateway-connect-minishell.md)]


## <a name="configure-cluster-access-via-kubernetes-rbac"></a>Configurer l’accès au cluster via Kubernetes RBAC

Une fois le cluster Kubernetes créé, vous pouvez utiliser l’outil *kubectl* par le biais de la ligne de commande pour accéder au cluster. 

Cette approche vous amène à créer un espace de noms et un utilisateur. Vous associez ensuite l’utilisateur à l’espace de noms. Vous devez également obtenir un *fichier config* qui vous permet d’utiliser un client Kubernetes pour communiquer directement avec le cluster Kubernetes que vous avez créé sans avoir à vous connecter à l’interface PowerShell de votre appareil Azure Stack Edge Pro.

1. Créez un espace de noms. Tapez :

    `New-HcsKubernetesNamespace -Namespace <string>` 

    > [!NOTE]
    > Pour les utilisateurs et les espaces de noms, les [conventions de nommage de sous-domaines DNS](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names) s’appliquent.

    Voici un exemple de sortie :

    `[10.100.10.10]: PS> New-HcsKubernetesNamespace -Namespace "myasetest1"`

2. Créez un utilisateur et obtenez un fichier config. Tapez :

    `New-HcsKubernetesUser -UserName <string>`

    > [!NOTE]
    > Vous ne pouvez pas utiliser *aseuser* comme nom d’utilisateur, car ce nom est réservé à l’utilisateur par défaut associé à l’espace de noms IoT pour Azure Stack Edge Pro.

    Voici un exemple de sortie du fichier config :
   
    ```powershell
    [10.100.10.10]: PS> New-HcsKubernetesUser -UserName "aseuser1"
        apiVersion: v1
        clusters:
        - cluster:
            certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01ERXlPVEUyTlRFeE4xb1hEVE13TURFeU5qRTJOVEV4TjFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTXNpCkdZUHB0U1VWcDhKZEdVcHE1MVBURWhsZm8wWkk3YXFBdUlrOHZWdUFCZHJQK1FBSzFxcEN1di93NjIwbUtpZ0QKak1aT3Q4QkREREppWHF6UDZRZm5Oc0U2VXBHMnh0YnYrcTZHV2R5K0t6WkxMbXlwWGY3VjlzZEJnejVKVDNvYQpIdzFja2NTUklHSlV3UWxTbklNaHJUS3JUNDZFUUp3d282TmlNUzZMZDZieVk3WkUrTGg3OS9aNEhLanhTRmhMClc5ZG8veThZR3FXUDZmZTFmMmVmSkhUeGtwR05HZE1UVjNuOFlCZ0pSRzdrNjh0N2MrZ1NhbUlVWVJpTUNSNFAKYlFxcFpscWYvV2REZEJHOFh6aDJ0M1l4SkVIMm00T0Z1cSsvUitMYm95aHdKbmNMdVJ5OEpNZWlwTEQ3UlN0QwpZTDNNR0EzN2JieTRyYm4zVzg4Q0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFNbzFwWlBtQzV1cmRPZUJhSWQ4eEQzRkxCMG8KTlErbXBXMWpDd0ZtY3h6dUtlWmRsNXc2N0tuS2JTcDR0TXo1cXg3bUtSc0UxcnBoWkh2VHlKUXg1ZFk2ZE1Kdgp5d2FQZjBpT05TNlU2cC9INE12U1dJaEtJZ1FuTnE1dDh4TjJCNnZpQW41RmZoRkx6WEQrUlZGSm42cnovWkZnCmV6MHpxTkNKYmcvelFucFROcmQ2cnFFRHpoSVFZOVdYVWQycFh3ZXRqUXJpMkpZamh4NmtEcTVoRkZTM0FLUnIKOWlQTVQxaWNkR1NUMFVvM1hIZ1k2ck45WGp3MHFrY2I0Sy83UlVVWlRvS3dKamROR3lNTnpad000L2puR0p5SwpQTE9ycU5Ddlkvb0lkVEM5eVZVY3VRbXVlR0VqT20xUnN1RDFHYVE0RTZwakppVWJpMVdrajJ1bFhOWT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
            server: https://10.128.47.90:6443
            name: kubernetes
        contexts:
        - context:
            cluster: kubernetes
            user: aseuser1
            name: aseuser1@kubernetes
        current-context: aseuser1@kubernetes
        kind: Config
        preferences: {}
        users:
        - name: aseuser1
            user:
            client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMwVENDQWJtZ0F3SUJBZ0lJWlFXcjY2cGFWSm93RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBeE1qa3hOalV4TVRkYUZ3MHlNVEF4TWpneU1qVTJNVGRhTUJNeApFVEFQQmdOVkJBTVRDR0Z6WlhWelpYSXhNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDCkFRRUF4R3RDaXJ2cVhGYm5wVmtaYlVPRWQ3cXg2UzNVZ092MlhHRHNKM2VYWXN0bUxQVjMrQnlBcWwyN211L1AKaWdmaWt6MG9QSW1iYmtvcVJkamlYckxFZnk0N3dHcEhzdUhUOHNLY2tHTnJleFE2bXhaZ29xaU1nL2FuMUpMdwpiOFEvVnlQaWdVdUt6eVBseEhUZmlmSVM5MzR1VnZVZUc0dzlMRjAyZ2s2Nitpc0ZtanhsVmhseWRMNlc2UmZTCjl0OGpNMEFkdEpJL0xNbE13RHJJRVdFKzM4WDVNelJhQkJYNnlzNDFWSkZxekcwdW14dHdxN2pGOXp1UTE4ekIKalRZaDl3OWVKcDJwS2Fvak5tNE9SSDh4SzVSaUhocjJ2anFJWXkxRDd2WDh0b0U1K05HNmxHZjh5L1NvQnNRbQpmOG9vL1k3SEZmQXVGdlN6WUc1RUlQTFM4UUlEQVFBQm95Y3dKVEFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEClZSMGxCQXd3Q2dZSUt3WUJCUVVIQXdJd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFNR1BxY0YzS1BCbHZ0K24KN1NOcGE3anhWYkhZVGxyNTgwVWxzek93WEwwVnVPUUlCYmN2djIzZk9HNkhDZlQ0bWxBU0JRWVNZcmpLMjJTTwpTWld4cjNQUDlhVzNHajkxc0ttSnc1ZUF1WFhQbUJpK1RWQzBvY0ZLaEQvZ0o1aC93YnBaVndpVjVyRWE5Kzc2CnhNcFAzRld6dG5tT1hPaEl6UFNlR3B4YWpwQXd3ZXd4QU0yb0xGRFZFcy9XTFFMODJZM3NFcE93NVNaSVJJNXMKUHhMUTVnV1ZPM2x2SXcwZ3IrdkJlanZSOUZKaWVuTWFRdGdjSVgyRmpDaDBRMHVYRkdsTVNXWEljbjRLRTR0TApQSFFMalRSVUwyVnRXcW1YZ1RBM3RzN01DcGNRTFdPZFJUYkpSejZCbkc1aXVwcDdOSlFvYW9YcWpNVk5DVDZCCllYMEd0Skk9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
            client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBeEd0Q2lydnFYRmJucFZrWmJVT0VkN3F4NlMzVWdPdjJYR0RzSjNlWFlzdG1MUFYzCitCeUFxbDI3bXUvUGlnZmlrejBvUEltYmJrb3FSZGppWHJMRWZ5NDd3R3BIc3VIVDhzS2NrR05yZXhRNm14WmcKb3FpTWcvYW4xSkx3YjhRL1Z5UGlnVXVLenlQbHhIVGZpZklTOTM0dVZ2VWVHNHc5TEYwMmdrNjYraXNGbWp4bApWaGx5ZEw2VzZSZlM5dDhqTTBBZHRKSS9MTWxNd0RySUVXRSszOFg1TXpSYUJCWDZ5czQxVkpGcXpHMHVteHR3CnE3akY5enVRMTh6QmpUWWg5dzllSnAycEthb2pObTRPUkg4eEs1UmlIaHIydmpxSVl5MUQ3dlg4dG9FNStORzYKbEdmOHkvU29Cc1FtZjhvby9ZN0hGZkF1RnZTellHNUVJUExTOFFJREFRQUJBb0lCQUVVSUVXM2kxMTQycU5raQo5RjNEWWZZV1pscTJZYjRoc0FjTmhWSGxwUTN5d0dsQ3FEUktDQ3BZSVF3MkJqSFR6WnpEM0xWU0E0K0NmMUxuCkE4QVdnaHJVcStsWE1QVzhpcG9DTGJaTlNzUUord0x3bld2dFl0MHFQaGZtd0p2M1UrK1RUQkwyOHNVVUw3ZVkKLzh0aWlhbno3ZU5mNklIMENyZmgxcnQ3WWhsemtRd1hBVHNScVJja0dMaTgrdGN5WnVzdGFhbENUSzBGRTdCaQpBUGE5a2w1SG56eCs4TTcvNWladHkwTUIxYWpWMnlGblBkUmlKSFVCb1AxVVV0QUthYjVZU0RvNllkZ2pIUTRHCjNWN1l1YWZobnVFMXA0VVIvUkloVVdjRlVVaTFBOFpZMFdnd1BDTmhnMWpQZU5vb2Y1UHpRbEY1OTRBREVwUUYKOFR2bG92RUNnWUVBOWZZbUxyY0tlQ0JiMTFoQVhoTi91Z1RTbU5xNnpFL1pPSWl6M0xwckdjRDhvWDdCVW9GcgplelkxbktSS2tkczE5OERnVjlQZUhuNzllQTRoMjM5RkIwNFFhMUJBdUVMRzRsdHJ3VlNxaFBENUR6YkcrSEhSCnJtYThVMEpUSmVVS0tJVjRUUGxlTzFtK2tjbkRJVXY1ckpwZDVXU3RvcUhXdk9RZkEvRUF0VlVDZ1lFQXpHOTcKTitCZVVvbFNiREttVUNGdTdPZGhYSXJYR3RnSEorZ2JOMDlnSHRURG5PY0IxZ1NzNkpZa1FPQU9qbWFxK05lRAp5SUF1NytheWlFRmpyT2tzTGhkSTREUXNkWFZveFFGVko1V1JwWlk3UTVRaFZpYUR2enR4NDlzSDlKSkplM2U0Cnl3NWdpNGkxKy90MnY2eWRKcWdNQ0xxOHlEdFRrcE9PSitkbkp5MENnWUJwZ3lpcURaZU9KTU9CUTdpSkl2QSsKQ21lVmJ1K0hTaEd6TU9HSHBPamc2V3IybEh1Mk94S3lqblM5TjdWTmtLNDhGQitwVFpnUm1RUi9CZ0Q4T2tLUQplYXFOZnFYazViQ1AxZ3dKcVpwazRVTFdoZmNoQ1NLY0lESlZ2VFFTSTRrU0RQK29kYWs0Nkt6WnVhWGRtTXdJCmdVZ2FhZkFhdmpaeVhhSDRmT0NDNlFLQmdHVXJCaDh3dVh5KzJEc1RGWnF4OE9McjNoS2Q0clUyRXRSODJIc1cKbk1xbEgraVZxU0x3VFdFTWJBUnUzTVU3cVlCYnBxdWlRNWdVNG1UcmR4Z3FpK0tEUTEwd2RJL3IrbDBEdTlCTApCRGlkajlaeGg4M0tZWWhSTXBzLzJULys1TDVsRU4zcnozczl2RkZtcisxS3pycENqeklDdDBtZmtrd0hHV0pGCjhaWkJBb0dBVXB3aUIrcWlHbkpxU1FtZHNSZFVabGFBaTRpbGhaa01RYTRHem95ZFQ3OTVHTm44ZThBRjd3WHMKTGpyYjdEV1FwakdCMnZpUlkySUZBVmIyKzZsdDlwOVJRMTZnSmxpNU5ZRXVvQWRoWXBsVWdBZGFHWHNGNHdabwo3SHFHTHBGdmUxVU5Gb0dQdkxpWUNrUFVYdGduQ3dNb0R2SEpKNzVYMXl6ckh6cmxUS1k9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==  
        
    [10.100.10.10]: PS>
    ```
    

3. Un fichier config s’affiche en texte brut. Copiez ce fichier et enregistrez-le en tant que fichier *config*. 

    > [!IMPORTANT]
    > N’enregistrez pas le fichier config au format *.txt*, mais plutôt sans aucune extension de fichier.

4. Le fichier config doit résider dans le dossier `.kube` de votre profil utilisateur sur l’ordinateur local. Copiez le fichier dans ce dossier dans votre profil utilisateur.

    ![Emplacement du fichier config sur le client](media/azure-stack-edge-j-series-create-kubernetes-cluster/location-config-file.png)

5. Associez l’espace de noms à l’utilisateur que vous avez créé. Tapez :

    `Grant-HcsKubernetesNamespaceAccess -Namespace <string> -UserName <string>`

    Voici un exemple de sortie :

    `[10.100.10.10]: PS>Grant-HcsKubernetesNamespaceAccess -Namespace "myasetest1" -UserName "aseuser1"`

    Une fois que vous avez le fichier config, vous n’avez pas besoin d’un accès physique au cluster. Si votre client peut effectuer un test ping sur l’adresse IP de l’appareil Azure Stack Edge Pro, vous devez être en mesure de diriger le cluster avec les commandes *kubectl*.

6. Démarrez une nouvelle session PowerShell sur votre client. Vous n’avez pas besoin d’être connecté à l’interface de l’appareil. Vous pouvez maintenant installer `kubectl` sur votre client à l’aide de la commande suivante :

    ```powershell
    PS C:\windows\system32> curl https://storage.googleapis.com/kubernetes-release/release/v1.15.2/bin/windows/amd64/kubectl.exe -O kubectl.exe
    
    PS C:\windows\system32>
    ```
    Par exemple, si le nœud principal Kubernetes exécutait la version v1.15.2, installez la version v1.15.2 sur le client.

    > [!IMPORTANT]
    > Téléchargez un client dont la version ne diffère pas de plus d’une version mineure par rapport à la version du nœud principal. La version du client peut être supérieure d’une seule version mineure à celle du nœud principal. Par exemple, un nœud principal v1.3 peut fonctionner avec des nœuds v1.1, v1.2 et v.1.3 et avec des clients v1.2, v1.3 et v1.4. Pour plus d’informations sur la version du client Kubernetes, consultez [Stratégie de prise en charge des versions et des différences de version Kubernetes](https://kubernetes.io/docs/setup/release/version-skew-policy/#supported-version-skew). Pour plus d’informations sur la version du serveur Kubernetes sur Azure Stack Edge Pro, accédez à Obtenir la version du serveur Kubernetes.<!-- insert link-->
    > Parfois, `kubectl` est préinstallé sur votre système si vous exécutez Docker pour Windows ou d’autres outils. Il est important de télécharger la version spécifique de `kubectl` comme indiqué dans cette section pour utiliser ce cluster kubernetes. 

    L’installation prend plusieurs minutes.

7. Vérifiez que la version installée est celle que vous avez téléchargée. Vous devez spécifier le chemin absolu de l’emplacement auquel `kubectl.exe` a été installé sur votre système.
    
    ```powershell
    PS C:\Users\myuser> C:\windows\system32\kubectl.exe version
    Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.2", GitCommit:"f6278300bebbb750328ac16ee6dd3aa7d3549568", GitTreeState:"clean", BuildDate:"2019-08-05T09:23:26Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"windows/amd64"}
    Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.1", GitCommit:"4485c6f18cee9a5d3c3b4e523bd27972b1b53892", GitTreeState:"clean", BuildDate:"2019-07-18T09:09:21Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
    PS C:\Users\myuser>
    ``` 
    
    Pour plus d’informations sur les commandes `kubectl` utilisées pour gérer le cluster Kubernetes, consultez [Vue d’ensemble de kubectl](https://kubernetes.io/docs/reference/kubectl/overview/).

8. Ajoutez une entrée DNS au fichier hosts sur votre système. 

    1. Exécutez le Bloc-notes en tant qu’administrateur et ouvrez le fichier `hosts` situé à l’emplacement `C:\windows\system32\drivers\etc\hosts`. 
    2. Utilisez les informations que vous avez enregistrées à partir de la page **Appareil** dans l’interface utilisateur locale à l’étape précédente pour créer l’entrée dans le fichier hosts. 

        Par exemple, copiez ce point de terminaison `https://compute.asedevice.microsoftdatabox.com/[10.100.10.10]` pour créer l’entrée suivante avec l’adresse IP et le domaine DNS de l’appareil : 

        `10.100.10.10 compute.asedevice.microsoftdatabox.com`

9. Pour vérifier que vous pouvez vous connecter aux pods Kubernetes, tapez :
    
    ```powershell
    PS C:\Users\myuser> kubectl get pods -n "myasetest1"
    No resources found.
    PS C:\Users\myuser>
    ```
Vous pouvez maintenant déployer vos applications dans l’espace de noms, puis afficher ces applications et leurs journaux.

> [!IMPORTANT]   
> Il existe de nombreuses commandes que vous ne pouvez pas exécuter, par exemple, celles qui nécessitent un accès administrateur. Vous pouvez uniquement effectuer les opérations autorisées sur l’espace de noms.


## <a name="remove-kubernetes-cluster"></a>Supprimer le cluster Kubernetes

Pour supprimer le cluster Kubernetes, vous devez supprimer la configuration de calcul.

Pour obtenir des instructions détaillées, accédez à [Supprimer la configuration du calcul](azure-stack-edge-j-series-manage-compute.md#remove-compute-configuration).
   

## <a name="next-steps"></a>Étapes suivantes

- [Déployez une application sans état sur votre Azure Stack Edge Pro](azure-stack-edge-j-series-deploy-stateless-application-kubernetes.md).
