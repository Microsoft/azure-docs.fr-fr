---
title: Résoudre les problèmes d’Azure Bastion | Microsoft Docs
description: Dans cet article, découvrez comment résoudre les problèmes liés à Azure Bastion.
services: bastion
author: charwen
ms.service: bastion
ms.topic: troubleshooting
ms.date: 10/16/2019
ms.author: charwen
ms.openlocfilehash: f3c142491363f30513877ae4368f291430aa3675
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85831928"
---
# <a name="troubleshoot-azure-bastion"></a>Dépanner Azure Bastion

Cet article explique comment résoudre les problèmes liés à Azure Bastion.

## <a name="unable-to-create-an-nsg-on-azurebastionsubnet"></a><a name="nsg"></a>Impossible de créer un groupe de sécurité réseau sur AzureBastionSubnet

**Q :** Lorsque j’essaie de créer un groupe de sécurité réseau sur le sous-réseau Azure Bastion, j’obtiens l’erreur suivante : *Le groupe de sécurité réseau <NSG name> ne dispose pas des règles nécessaires pour le sous-réseau Azure Bastion AzureBastionSubnet*.

**R :** Si vous créez un groupe de sécurité réseau et l’appliquez à *AzureBastionSubnet*, vérifiez que vous avez ajouté les règles suivantes à votre groupe de sécurité réseau. Si vous n’ajoutez pas ces règles, la création/mise à jour du groupe de sécurité réseau échouera.

1. Connectivité du plan de contrôle : entrée sur 443 à partir de GatewayManager
2. Journalisation des diagnostics et autres : sortie sur 443 vers AzureCloud (Les étiquettes régionales incluses dans cette étiquette de service ne sont pas encore prises en charge.)
3. Machine virtuelle cible - Accès sortant sur les ports 3389 et 22 vers VirtualNetwork

Un exemple de règles de groupe de sécurité réseau est disponible à des fins de référence dans le [modèle de démarrage rapide](https://github.com/Azure/azure-quickstart-templates/tree/master/101-azure-bastion-nsg).
Pour plus d’informations, consultez [Guide pour les groupes de sécurité réseau pour Azure Bastion](bastion-nsg.md).

## <a name="unable-to-use-my-ssh-key-with-azure-bastion"></a><a name="sshkey"></a>Impossible d’utiliser ma clé SSH avec Azure Bastion

**Q :** Lorsque j’essaie d’accéder à mon fichier de clé SSH, j’obtiens l’erreur suivante : *La clé privée SSH doit commencer par « -----BEGIN RSA PRIVATE KEY----- » et se terminer par « -----END RSA PRIVATE KEY----- »* .

**R :** Azure Bastion prend en charge uniquement les clés SSH RSA l’heure actuelle. Veillez à sélectionner un fichier de clé qui est une clé privée RSA pour SSH, avec la clé publique configurée sur la machine virtuelle cible. 

Par exemple, vous pouvez utiliser la commande suivante pour créer une clé SSH RSA :

**ssh-keygen -t rsa -b 4096 -C "email@domain.com"**

Sortie :

```
ashishj@dreamcatcher:~$ ssh-keygen -t rsa -b 4096 -C "email@domain.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ashishj/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ashishj/.ssh/id_rsa.
Your public key has been saved in /home/ashishj/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:c+SBciKXnwceaNQ8Ms8C4h46BsNosYx+9d+AUxdazuE email@domain.com
The key's randomart image is:
+---[RSA 4096]----+
|      .o         |
| .. ..oo+. +     |
|=.o...B==.O o    |
|==o  =.*oO E     |
|++ .. ..S =      |
|oo..   + =       |
|...     o o      |
|         . .     |
|                 |
+----[SHA256]-----+
```

## <a name="unable-to-sign-in-to-my-windows-domain-joined-virtual-machine"></a><a name="domain"></a>Impossible de se connecter à ma machine virtuelle jointe à un domaine Windows

**Q :** Je ne parviens pas à me connecter à ma machine virtuelle Windows jointe à un domaine.

**R :** Azure bastion prend en charge la connexion de machines virtuelles jointes à un domaine pour la connexion au domaine par nom d’utilisateur uniquement. Lorsque vous spécifiez les informations d’identification du domaine dans le portail Azure, utilisez le format UPN (username@domain) au lieu du format *domaine\nom d’utilisateur* pour vous connecter. Cela est pris en charge pour les machines virtuelles jointes à un domaine ou hybrides (à la fois jointes à un domaine et jointes à Azure AD). Cela n’est pas pris en charge pour les machines virtuelles jointes à Azure AD uniquement.

## <a name="file-transfer-issues"></a><a name="filetransfer"></a>Problèmes de transfert de fichiers

**Q :** Le transfert de fichiers est-il pris en charge avec Azure Bastion ?

**R :** Le transfert de fichiers n’est pas pris en charge pour le moment. Nous travaillons sur l’ajout de sa prise en charge.

## <a name="black-screen-in-the-azure-portal"></a><a name="blackscreen"></a>Écran noir dans le portail Azure

**Q :** Lorsque j’essaie de me connecter avec Azure Bastion, j’obtiens un écran noir dans le portail Azure.

**R :** Cela se produit lorsqu’il y a un problème de connectivité réseau entre votre navigateur web et Azure Bastion (votre pare-feu Internet client peut bloquer le trafic WebSocket ou un problème similaire) ou entre Azure Bastion et votre machine virtuelle cible. La plupart des cas comprennent un groupe de sécurité réseau appliqué soit à AzureBastionSubnet, soit au sous-réseau de votre machine virtuelle cible qui bloque le trafic RDP/SSH dans votre réseau virtuel. Autorisez le trafic WebSocket sur le pare-feu Internet de votre client et vérifiez les groupes de sécurité réseau du sous-réseau de votre machine virtuelle cible.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, visitez la [FAQ de Bastion](bastion-faq.md).
