---
title: Messages d’erreur RDP spécifiques des machines virtuelles Azure | Microsoft Docs
description: Découvrez la signification des messages d’erreur spécifiques que vous pouvez recevoir lorsque vous essayez d’utiliser la connexion Bureau à distance à une machine virtuelle Windows dans Azure
keywords: erreur bureau à distance,erreur de connexion bureau à distance,impossible se connecter à la machine virtuelle,résolution des problèmes de connexion bureau à distance
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: top-support-issue,azure-service-management,azure-resource-manager
ms.assetid: 5feb1d64-ee6f-4907-949a-a7cffcbc6153
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: f3e4c411223a3e8fc0602602cd941a00f7a19cca
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87028241"
---
# <a name="troubleshooting-specific-rdp-error-messages-to-a-windows-vm-in-azure"></a>Résolution des messages d’erreur RDP spécifiques des machines virtuelles Windows dans Azure
Il peut arriver que vous receviez un message d’erreur spécifique lorsque vous utilisez la connexion Bureau à distance à une machine virtuelle Windows dans Azure. Cet article décrit en détail certains des messages d’erreur les plus couramment rencontrés, ainsi que les étapes de dépannage à suivre pour les résoudre. Si vous rencontrez des problèmes de connexion à votre machine virtuelle avec le protocole RDP mais que vous ne recevez pas de message d’erreur spécifique, consultez le [guide de résolution des problèmes de Bureau à distance](troubleshoot-rdp-connection.md).

Pour plus d’informations sur les messages d’erreur spécifiques, consultez les sections suivantes :

* [La session distante a été déconnectée, car aucun serveur de licences Bureau à distance n’est disponible pour fournir une licence](#rdplicense).
* [Le bureau à distance ne trouve pas le « name » de l’ordinateur](#rdpname).
* [Une erreur d’authentification s’est produite. L’autorité de sécurité locale ne peut pas être contactée](#rdpauth).
* [Message d’erreur de sécurité Windows : Vos informations d’identification n’ont pas fonctionné](#wincred).
* [Cet ordinateur ne peut pas se connecter à l’ordinateur distant](#rdpconnect).

<a id="rdplicense"></a>

## <a name="the-remote-session-was-disconnected-because-there-are-no-remote-desktop-license-servers-available-to-provide-a-license"></a>La session distante a été déconnectée, car aucun serveur de licences Bureau à distance n’est disponible pour fournir une licence.
Cause : La période de grâce du Gestionnaire de licences de 120 jours pour le rôle de serveur Bureau à distance a expiré et vous devez installer les licences.

Pour contourner ce problème, enregistrez une copie locale du fichier RDP à partir du portail, puis exécutez cette commande au niveau d’une invite de commande PowerShell pour vous connecter. Cette étape ne désactive la licence que pour cette connexion :

```powershell
mstsc <File name>.RDP /admin
```

Si vous n’avez pas besoin de plus de deux connexions Bureau à distance simultanées à la machine virtuelle, vous pouvez utiliser le Gestionnaire de serveur pour supprimer le rôle de serveur Bureau à distance.

Pour plus d’information, consultez le billet de blog [Azure VM fails with « No Remote Desktop License Servers available » (La machine virtuelle Azure échoue et affiche le message « Aucun serveur de licences Bureau à distance n’est disponible »)](/archive/blogs/mast/rdp-to-azure-vm-fails-with-no-remote-desktop-license-servers-available).

<a id="rdpname"></a>

## <a name="remote-desktop-cant-find-the-computer-name"></a>Le bureau à distance ne trouve pas le « name » de l’ordinateur.
Cause : Le client Bureau à distance de votre ordinateur ne peut pas résoudre le nom de l’ordinateur dans les paramètres du fichier RDP.

Solutions possibles :

* Si vous êtes sur un intranet d’entreprise, vérifiez que votre ordinateur a bien accès au serveur proxy et qu’il peut lui transmettre le trafic HTTPS.
* Si vous utilisez un fichier RDP stocké localement, essayez d’utiliser celui généré par le portail. Grâce à cette étape, vous êtes certain de disposer du nom DNS correct de la machine virtuelle ou du service cloud et du port du point de terminaison de la machine virtuelle. Voici un exemple de fichier RDP généré par le portail :

    ```output
    full address:s:tailspin-azdatatier.cloudapp.net:55919
    prompt for credentials:i:1
    ```

La partie adresse de ce fichier RDP comporte :

* Le nom de domaine complet du service cloud qui contient la machine virtuelle ("tailspin-ici azdatatier.cloudapp.NET" dans cet exemple).
* Le port TCP externe du point de terminaison pour le trafic Bureau à distance (55919).

<a id="rdpauth"></a>

## <a name="an-authentication-error-has-occurred-the-local-security-authority-cannot-be-contacted"></a>Une erreur d’authentification s’est produite. L’autorité de sécurité locale ne peut pas être contactée.
Cause : La machine virtuelle cible n’a pas pu localiser l’autorité de sécurité dans la partie nom d’utilisateur de vos informations d’identification.

Lorsque votre nom d’utilisateur est au format *AutoritéSécurité*\\*NomUtilisateur* (par exemple, CORP\User1), la partie *AutoritéSécurité* est soit le nom d’ordinateur de la machine virtuelle (pour l’autorité de sécurité locale), soit un nom de domaine Active Directory.

Solutions possibles :

* Si le compte est local à la machine virtuelle, vérifiez que le nom de la machine virtuelle est correctement orthographié.
* Si le compte se trouve dans un domaine Active Directory, vérifiez l’orthographe du nom de domaine.
* S’il s’agit d’un compte de domaine Active Directory et que le nom de domaine est orthographié correctement, vérifiez qu’un contrôleur de domaine est disponible dans ce domaine. Dans les réseaux virtuels Azure qui contiennent des contrôleurs de domaine, il est courant qu’un contrôleur de domaine soit indisponible, car il n’a pas démarré. Pour contourner ce problème, vous pouvez utiliser un compte d’administrateur local, plutôt qu’un compte de domaine.

<a id="wincred"></a>

## <a name="windows-security-error-your-credentials-did-not-work"></a>Message d’erreur de sécurité Windows : Vos informations d’identification n’ont pas fonctionné.
Cause : La machine virtuelle cible ne peut pas valider le nom et le mot de passe de votre compte.

Un ordinateur Windows peut valider les informations d’identification d’un compte local ou d’un compte de domaine.

* Pour les comptes locaux, utilisez la syntaxe *NomOrdinateur*\\*NomUtilisateur* (par exemple : SQL1\Admin4798).
* Pour les comptes de domaine, utilisez la syntaxe *NomDomaine*\\*NomUtilisateur* (par exemple, CONTOSO\johndoe).

Si vous avez promu votre machine virtuelle en tant que contrôleur de domaine d’une nouvelle forêt Active Directory, le compte d’administrateur local auquel vous êtes connecté est converti en un compte équivalent avec le même mot de passe dans la nouvelle forêt et le nouveau domaine. Le compte local est alors supprimé.

Par exemple, si vous êtes connecté au compte local DC1\DCAdmin et que vous avez promu la machine virtuelle en tant que contrôleur de domaine dans une nouvelle forêt pour le domaine corp.contoso.com, le compte local DC1\DCAdmin est supprimé et un compte de domaine (CORP\DCAdmin) est créé avec le même mot de passe.

Vérifiez que le nom du compte est un nom qui peut être considéré comme valide par la machine virtuelle, et que le mot de passe est correct.

Pour modifier le mot de passe du compte administrateur local, voir [Réinitialisation d’un mot de passe ou du Service Bureau à distance pour les machines virtuelles Windows](reset-rdp.md).

<a id="rdpconnect"></a>

## <a name="this-computer-cant-connect-to-the-remote-computer"></a>Cet ordinateur ne peut pas se connecter à l’ordinateur distant.
Cause : Le compte utilisé pour vous connecter ne dispose pas des droits de connexion au Bureau à distance.

Chaque ordinateur Windows dispose d’un groupe local d’utilisateurs du Bureau à distance, qui contient les comptes et groupes autorisés à s’y connecter à distance. Les membres du groupe local d’administrateurs y ont également accès, même si ces comptes ne sont pas répertoriés dans le groupe local des utilisateurs du Bureau à distance. Pour les ordinateurs associés à un domaine, le groupe local d’administrateurs contient également les administrateurs du domaine en question.

Vérifiez que le compte que vous utilisez pour vous connecter dispose des droits de connexion au Bureau à distance. Pour résoudre ce problème, utilisez un compte d’administrateur local ou de domaine pour vous connecter via le Bureau à distance. Pour ajouter le compte de votre choix au groupe local des utilisateurs du Bureau à distance, utilisez le composant logiciel enfichable Microsoft Management Console (**Outils système > Utilisateurs et groupes locaux > Groupes > Utilisateurs du Bureau à distance**).

## <a name="next-steps"></a>Étapes suivantes
Si aucune de ces erreurs ne s’est produite et que vous avez rencontré un problème inconnu en vous connectant avec le protocole RDP, consultez le [guide de dépannage du Bureau à distance](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).

* Pour connaître les étapes de résolution des problèmes d’accès aux applications exécutées sur une machine virtuelle, consultez [Résolution des problèmes d’accès à une application exécutée sur une machine virtuelle Azure](./troubleshoot-app-connection.md?toc=/azure/virtual-machines/linux/toc.json).
* Si vous rencontrez des problèmes pour vous connecter à une machine virtuelle Linux dans Azure avec SSH (Secure Shell), consultez [Résolution des problèmes de connexions SSH à une machine virtuelle de Linux dans Azure](./troubleshoot-ssh-connection.md?toc=/azure/virtual-machines/linux/toc.json).
