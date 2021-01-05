---
title: Azure VMware Solution by CloudSimple – Configurer vCenter sur le cloud privé pour vRealize Automation
description: Décrit comment configurer un serveur VMware vCenter sur votre cloud privé CloudSimple en tant que point de terminaison pour VMware vRealize Automation
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/19/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: df73acfc469a8b7b5329b61095aefdbd73baafd4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77024838"
---
# <a name="set-up-vcenter-on-your-private-cloud-for-vmware-vrealize-automation"></a>Configurer vCenter sur votre cloud privé pour VMware vRealize Automation

Vous pouvez configurer un serveur VMware vCenter sur votre cloud privé CloudSimple en tant que point de terminaison pour VMware vRealize Automation.

## <a name="before-you-begin"></a>Avant de commencer

Effectuez les tâches suivantes avant de configurer le serveur vCenter :

* Configurez une [connexion VPN de site à site](vpn-gateway.md#set-up-a-site-to-site-vpn-gateway) entre votre environnement local et votre cloud privé.
* [Configurez le transfert DNS de demandes DNS locales](on-premises-dns-setup.md) sur les serveurs DNS de votre cloud privé.
* Soumettez [une demande de support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) pour créer un utilisateur administratif IaaS vRealize Automation avec le jeu d’autorisations répertorié dans le tableau suivant.

| Valeur d'attribut | Autorisation |
------------ | ------------- |  
| Magasin de données |  Allouer de l’espace <br> Parcourir la banque de données |
| Cluster de banque de données | Configurer un cluster de banque de données |
| Dossier | Créer un dossier <br>Supprimer un dossier |
| Global |  Gérer des attributs personnalisés<br>Définir un attribut personnalisé |
| Réseau | Attribuer un réseau |
| Autorisations | Modifier les autorisations |
| Ressource | Affecter une machine virtuelle à un pool de ressources<br>Migrer une machine virtuelle hors tension<br>Migrer une machine virtuelle sous tension |
| Inventaire des machines virtuelles |  Créer à partir de ce qui existe<br>Création<br>Déplacer<br>Supprimer | 
| Interaction des machines virtuelles |  Configurer un support CD<br>Interaction de la console<br>Connexion de l’appareil<br>Mise hors tension<br>Mise sous tension<br>Réinitialiser<br>Interrompre<br>Installation des outils | 
| Configuration de la machine virtuelle |  Ajouter un disque existant<br>Ajouter un nouveau disque<br>Ajouter ou supprimer<br>Supprimer un disque<br>Avancé<br>Modifier le nombre d’UC<br>Modifier la ressource<br>Étendre le disque virtuel<br>Suivi des modifications du disque<br>Mémoire<br>Modifier les paramètres de l’appareil<br>Renommer<br>Définir l’annotation (version 5.0 et versions ultérieures)<br>Paramètres<br>Positionnement du fichier d’échange |
| Approvisionnement |  Personnaliser<br>Modèle de clone<br>Machine virtuelle de clone<br>Déployer un modèle<br>Lire les spécifications de personnalisation |
| État de la machine virtuelle | Créer une capture instantanée<br>Supprimer une capture instantanée<br>Restaurer la capture instantanée |

## <a name="install-vrealize-automation-in-your-on-premises-environment"></a>Installer vRealize Automation dans votre environnement local

1. Connectez-vous à l’appliance serveur IaaS vRealize Automation dans le rôle d’administrateur IaaS que le support CloudSimple a créé pour vous.
2. Déployez un agent vSphere pour le point de terminaison vRealize Automation.
    1. Accédez à https://*vra-url*:5480/installer, où *vra-url* est l’URL que vous utilisez pour accéder à l’interface utilisateur d’administration de vRealize Automation.
    2. Cliquez sur **Programme d'installation IaaS** pour télécharger le programme d'installation.<br>
    La convention d’affectation de noms pour le fichier du programme d'installation est setup_*vra-url*@5480.exe.
    3. Exécutez le programme d’installation. Dans l’écran de bienvenue, cliquez sur **Suivant**.
    4. Acceptez le CLUF, puis cliquez sur **Suivant**.
    5. Fournissez les informations de connexion, cliquez sur **Accepter le certificat**, puis cliquez sur **Suivant**.
    ![Informations d'identification vRA](media/configure-vra-endpoint-login.png)
    6. Sélectionnez **Installation personnalisée** et **Agents proxy**, puis cliquez sur **Suivant**.
    ![Type d’installation de vRA](media/configure-vra-endpoint-install-type.png)
    7. Entrez les informations d'identification du serveur IaaS, puis cliquez sur **Suivant**. Si vous utilisez Active Directory, entrez le nom d’utilisateur au format **domaine\utilisateur**. Sinon, utilisez le format **user@domain** .
    ![Informations de connexion vRA](media/configure-vra-endpoint-account.png)
    8. Pour les paramètres de proxy, entrez **vSphere** pour **Type d’agent**. Entrez le nom de l’agent.
    9. Entrez le nom de domaine complet du serveur IaaS dans les champs **Hôte de service du gestionnaire** et **Hôte de service web du gestionnaire de modèle**. Cliquez sur **Tester** pour tester la connexion de chacune des valeurs de nom de domaine complet. Si le test échoue, modifiez vos paramètres DNS afin que le nom d’hôte du serveur IaaS soit résolu.
    10. Entrez un nom pour le point de terminaison du serveur vCenter pour le cloud privé. Enregistrez le nom pour une utilisation ultérieure dans le processus de configuration.

        ![Proxy d’installation de vRA](media/configure-vra-endpoint-proxy.png)

    11. Cliquez sur **Suivant**.
    12. Cliquez sur **Installer**.

## <a name="configure-the-vsphere-agent"></a>Configurer l’agent vSphere

1. Accédez à https://*vra-url*/vcac et connectez-vous en tant que **ConfigurationAdmin**.
2. Sélectionnez **Infrastructure** > **Points de terminaison** > **Points de terminaison**.
3. Sélectionnez **Nouveau** > **vSphere** > **virtual**.
4. Entrez le nom du point de terminaison vSphere que vous avez spécifié dans la procédure précédente.
5. Pour **Adresse**, entrez l’URL du cloud privé vCenter Server au format https://*vcenter-fqdn*/sdk, où *vcenter-fqdn* est le nom du serveur vCenter.
6. Entrez les informations d’identification pour l’utilisateur administratif IaaS vRealize Automation que le support CloudSimple a créé pour vous.
7. Cliquez sur **Tester la connexion** pour valider les informations d’identification de l’utilisateur. Si le test échoue, vérifiez l’URL, les informations du compte et le [nom du point de terminaison](#verify-the-endpoint-name), puis refaites le test.
8. Après un test réussi, cliquez sur **OK** pour créer le point de terminaison vSphere.
    ![accès à la configuration du point de terminaison vRA](media/configure-vra-endpoint-vra-edit.png)

### <a name="verify-the-endpoint-name"></a>Vérifier le nom du point de terminaison

Pour identifier le nom correct du point de terminaison du serveur vCenter, procédez comme suit :

1. Ouvrez une invite de commandes sur l’appliance IaaS.
2. Changer le répertoire en C:\Program Files (x86)\VMware\vCAC\Agents\agent-name, où *agent-name* est le nom que vous avez attribué au point de terminaison du serveur vCenter.
3. Exécutez la commande suivante :

```
..\..\Server\DynamicOps.Vrm.VRMencrypt.exe VRMAgent.exe.config get
```

Le résultat ressemble à ce qui suit. La valeur du champ `managementEndpointName` est le point de terminaison.

```
managementEndpointName: cslab1pc3-vc
doDeletes: true
```
