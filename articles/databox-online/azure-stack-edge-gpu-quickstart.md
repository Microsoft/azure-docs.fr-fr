---
title: Démarrage rapide pour configurer et déployer Azure Stack Edge GPU | Microsoft Docs
description: Démarrez le déploiement d’Azure Stack Edge GPU après la réception de l’appareil.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: quickstart
ms.date: 09/17/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to prepare the portal to quickly deploy Azure Stack Edge so I can use it to transfer data to Azure.
ms.openlocfilehash: 200e7f191cb19588fa79ba685c91a75dabd0156a
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91347997"
---
# <a name="quickstart-get-started-with-azure-stack-edge-pro-with-gpu"></a>Démarrage rapide : Bien démarrer avec Azure Stack Edge Pro avec GPU 

Ce guide de démarrage rapide détaille les prérequis et les étapes à suivre pour déployer votre appareil Azure Stack Edge Pro GPU. Les étapes de ce guide de démarrage rapide sont à effectuer dans le portail Azure et dans l’interface utilisateur web locale de l’appareil. 

La procédure totale doit durer environ une heure et demie. Pour obtenir des instructions pas à pas détaillées, consultez le [Tutoriel : Préparer le déploiement d’Azure Stack Edge Pro GPU](azure-stack-edge-gpu-deploy-prep.md#deployment-configuration-checklist). 


## <a name="prerequisites"></a>Prérequis

Avant de procéder au déploiement, vérifiez que les prérequis suivants sont en place :

1. L’appareil Azure Stack Edge Pro GPU est livré sur votre site, [déballé](azure-stack-edge-gpu-deploy-install.md#unpack-the-device) et [monté en rack](azure-stack-edge-gpu-deploy-install.md#rack-the-device). 
1. Configurez votre réseau afin que votre appareil puisse atteindre les [ports et modèles d’URL listés](azure-stack-edge-gpu-system-requirements.md#networking-port-requirements). 
1. Vous avez un accès Propriétaire ou Contributeur à un abonnement Azure.
1. Dans le portail Azure, accédez à **Accueil > Abonnements -> votre-abonnement -> Fournisseurs de ressources**. Recherchez `Microsoft.DataBoxEdge` et inscrivez le fournisseur de ressources. Répétez cette opération pour inscrire `Microsoft.Devices` si vous envisagez de créer une ressource IoT Hub afin de déployer des charges de travail de calcul.
1. Assurez-vous d’avoir au minimum 2 adresses IP libres, statiques et contiguës pour les nœuds Kubernetes et au moins 1 adresse IP statique pour le service IoT Edge. Pour chaque module ou service externe à déployer, vous aurez besoin d’une adresse IP supplémentaire.
1. Consultez la [liste de vérification du déploiement](azure-stack-edge-gpu-deploy-checklist.md), qui indique tout ce dont vous avez besoin pour la configuration de l’appareil. 


## <a name="deployment-steps"></a>Étapes du déploiement

1. **Installation** : connectez le PORT 1 à un ordinateur client à l’aide d’un câble croisé ou d’une carte Ethernet USB. Connectez au moins un autre port d’appareil pour les données, de préférence 25 GbE, (du PORT 3 au PORT 6) à Internet via au moins 1 commutateur GbE et des câbles en cuivre SFP+. Connectez les cordons d’alimentation fournis aux unités d’alimentation et à des unités de distribution d’alimentation distinctes. Appuyez sur le bouton d’alimentation du panneau avant pour allumer l’appareil.  

    Consultez [Matrice d’interopérabilité de la série Cavium FastlinQ 41000](https://www.marvell.com/documents/xalflardzafh32cfvi0z/) et [Produits compatibles avec la carte réseau Mellanox double port 25G ConnectX-4 canaux](https://docs.mellanox.com/display/ConnectX4LxFirmwarev14271016/Firmware+Compatible+Products) pour obtenir les commutateurs et câbles réseau compatibles.

    Voici la configuration de câblage minimale nécessaire au déploiement de votre appareil :  ![Fond de panier d’un appareil câblé](./media/azure-stack-edge-gpu-quickstart/backplane-min-cabling-1.png)

2. **Connecter** : Configurez les paramètres IPv4 sur la carte Ethernet sur votre ordinateur avec l’adresse IP statique **192.168.100.5** et le sous-réseau **255.255.255.0**. Ouvrez votre navigateur et connectez-vous à l’interface utilisateur web locale de l’appareil à l’adresse https://192.168.100.10. Cela peut prendre quelques minutes. Passez à la page web quand vous voyez l’avertissement relatif au certificat de sécurité.

3. **Connexion** : connectez-vous à l’appareil avec le mot de passe par défaut *Password1*. Changez le mot de passe d’administrateur de l’appareil. Il doit contenir entre 8 et 16 caractères appartenant à au moins trois des catégories suivantes : caractères majuscules, minuscules, numériques et spéciaux.

4. **Configuration du réseau** : acceptez la configuration DHCP par défaut pour le port de données connecté si vous disposez d’un serveur DHCP sur votre réseau. Si ce n’est pas le cas, fournissez une adresse IP, un serveur DNS et une passerelle par défaut. Pour plus d’informations, consultez [Paramètres réseau](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md#configure-network).

5. **Configuration du réseau de calcul** : créez un commutateur virtuel en activant un port sur votre appareil. Entrez 2 adresses IP statiques libres et contiguës pour les nœuds Kubernetes sur le réseau sur lequel vous avez créé le commutateur. Fournissez au moins 1 adresse IP statique pour le service Hub IoT Edge afin d’accéder aux modules de calcul et 1 adresse IP statique pour chaque service ou conteneur supplémentaire auquel vous souhaitez accéder à partir de l’extérieur du cluster Kubernetes. 

    Kubernetes est nécessaire pour déployer toutes les charges de travail conteneurisées. Pour plus d’informations, consultez [Paramètres réseau de calcul](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md#enable-compute-network).

6. **Configuration du proxy web** : si vous utilisez un proxy web dans votre environnement, entrez l’adresse IP du serveur proxy web dans `http://<web-proxy-server-FQDN>:<port-id>`. Définissez l’authentification sur **Aucun**. Pour plus d’informations, consultez [Paramètres de proxy web](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md#configure-web-proxy).

7. **Configuration de l’appareil** : entrez un nom d’appareil et un domaine DNS ou acceptez les valeurs par défaut. 

8. **Configuration du serveur de mise à jour** : acceptez le serveur Microsoft Update par défaut ou spécifiez un serveur WSUS (Windows Server Update Services) et le chemin du serveur. 

9. **Configuration des paramètres d’heure** : acceptez les paramètres d’heure par défaut ou définissez le fuseau horaire ainsi que le serveur NTP principal et le serveur NTP secondaire sur le réseau local ou en tant que serveurs publics.

10. **Configuration des certificats** : si vous avez changé le nom de l’appareil et/ou le domaine DNS, vous devez générer des certificats ou ajouter des certificats pour activer l’appareil. 

    - Pour tester des charges de travail de non-production, utilisez l’[option Générer des certificats](azure-stack-edge-gpu-deploy-configure-certificates.md#generate-device-certificates). 
    - Si vous apportez vos propres certificats comprenant la ou les chaînes de signature, [ajoutez les certificats](azure-stack-edge-gpu-deploy-configure-certificates.md#bring-your-own-certificates) au format approprié. Veillez à charger d’abord la chaîne de signature. Consultez [Créer des certificats](azure-stack-edge-j-series-create-certificates-tool.md) et [Charger des certificats par le biais de l’interface utilisateur locale](azure-stack-edge-gpu-deploy-configure-certificates.md#bring-your-own-certificates).

11. **Activer** : Pour obtenir la clé d’activation 

    1. Dans le portail Azure, accédez à votre **ressource Azure Stack Edge > Vue d’ensemble > Configuration de l’appareil > Activer > Générer la clé**. Copiez la clé. 
    1. Dans l’interface utilisateur web locale, accédez à **Bien démarrer > Activer** et indiquez la clé d’activation. Une fois la clé appliquée, l’activation de l’appareil prend quelques minutes. Téléchargez le fichier `<device-serial-number>`.json quand vous êtes invité à stocker de manière sécurisée les clés d’appareil nécessaires pour une récupération ultérieure. 

12. **Configuration du calcul** : dans le portail Azure, accédez à **Vue d’ensemble > Appareil**. Vérifiez que l’appareil est **En ligne**. Dans le volet gauche, accédez à **Computing en périphérie > Bien démarrer > Configurer le computing en périphérie > Calcul**. Fournissez un service IoT Hub existant ou nouveau et attendez environ 20 minutes que le calcul soit configuré. Pour plus d’informations, consultez [Tutoriel : Configurer le calcul sur un appareil Azure Stack Edge Pro avec GPU](azure-stack-edge-gpu-deploy-configure-compute.md).

Vous êtes prêt à déployer des charges de travail de calcul sur votre appareil via [IoT Edge](azure-stack-edge-gpu-deploy-sample-module-marketplace.md), [`kubectl`](azure-stack-edge-gpu-create-kubernetes-cluster.md) ou [Kubernetes avec Azure Arc](azure-stack-edge-gpu-deploy-arc-kubernetes-cluster.md) ! Si vous rencontrez des problèmes lors de l’installation, découvrez comment [résoudre les problèmes d’appareil](), les [problèmes de commande](azure-stack-edge-gpu-troubleshoot.md), les [problèmes de certificat](azure-stack-edge-j-series-certificate-troubleshooting.md) ou les problèmes liés à Kubernetes. 

## <a name="next-steps"></a>Étapes suivantes

[Installer Azure Stack Edge Pro GPU](./azure-stack-edge-gpu-deploy-install.md)



