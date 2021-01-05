---
title: Guide de contrôle du comportement d’arrêt de Windows dans Azure Lab Services | Microsoft Docs
description: Procédure d'arrêt automatique d’une machine virtuelle Windows inactive et de suppression de la commande d’arrêt Windows.
ms.topic: article
ms.date: 09/29/2020
ms.openlocfilehash: 248bbeabaf704ba636e2f82c7a93d0ee90a09f22
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94647696"
---
# <a name="guide-to-controlling-windows-shutdown-behavior"></a>Guide de contrôle du comportement d’arrêt de Windows

Azure Lab Services propose plusieurs contrôles de coût pour veiller à ce que les machines virtuelles Windows ne s’exécutent pas de manière inattendue :
 - [Définir un calendrier](./tutorial-setup-classroom-lab.md#set-a-schedule-for-the-lab)
 - [Définir des quotas pour les utilisateurs](./how-to-configure-student-usage.md#set-quotas-for-users)
 - [Activer l’arrêt automatique lors de la déconnexion](./how-to-enable-shutdown-disconnect.md)

Même avec ces contrôles de coût, il arrive qu'une machine virtuelle Windows continuer de s’exécuter de manière inattendue et entraîne une déduction du quota de l’étudiant :

- **Fenêtre RDP ouverte**
  
    Lorsqu’un étudiant se connecte à sa machine virtuelle avec RDP, il peut, par inadvertance, laisser la fenêtre RDP ouverte.  Tant que la fenêtre RDP est ouverte, le paramètre d'**arrêt automatique lors de la déconnexion** n'est pas activé puisqu’il se déclenche uniquement après déconnexion de la session RDP.

- **La commande d’arrêt de Windows est utilisée pour désactiver la machine virtuelle**
  
    Un étudiant peut utiliser la commande d’arrêt de Windows, ou d’autres mécanismes d’arrêt fournis dans Windows, pour éteindre la machine virtuelle plutôt que d'utiliser le [bouton d'arrêt d'Azure Lab Services](./how-to-use-classroom-lab.md#start-or-stop-the-vm).  Dans ce cas, du point de vue d'Azure Lab Services, la machine virtuelle est toujours en cours d’utilisation.
    
Pour éviter de telles situations, ce guide propose une procédure permettant d'arrêter automatiquement une machine virtuelle Windows inactive et de supprimer la commande d’arrêt de Windows à partir du menu **Démarrer**.  

> [!NOTE]
> Une machine virtuelle peut également être déduite de manière inattendue du quota lorsque l’étudiant démarre sa machine virtuelle, mais ne se connecte jamais à cette dernière avec RDP.  Actuellement, ce guide ne répond *pas* à un tel scénario.  Il convient plutôt de rappeler aux étudiants de se connecter immédiatement à leur machine virtuelle avec RDP après démarrage, à défaut de quoi ils doivent arrêter la machine virtuelle.

## <a name="remove-windows-shutdown-command-from-start-menu"></a>Supprimer la commande d'arrêt de Windows à partir du menu Démarrer

Les paramètres **Stratégie de groupe locale** Windows vous permettent également de supprimer la commande d'arrêt à partir du menu **Démarrer**.

Pour supprimer la commande d’arrêt, vous pouvez vous connecter au modèle de machine virtuelle et exécuter le script PowerShell ci-dessous.

```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "HidePowerOptions" -Value 1 -Force
```

Vous pouvez également suivre cette procédure manuelle utilisant le modèle de machine virtuelle :

1. Appuyez sur la touche Windows, entrez **gpedit**, puis sélectionnez **Modifier la stratégie de groupe (Panneau de configuration)** .

1. Accédez à **Configuration de l'ordinateur > Modèles d’administration > Menu Démarrer et Barre des tâches**.  

    ![Éditeur de stratégie de groupe locale](./media/how-to-windows-shutdown/group-policy-shutdown.png)

1. Cliquez avec le bouton droit sur **Supprimer et empêcher l’accès aux commandes Arrêter, Redémarrer, Mettre en veille et Mettre en veille prolongée**, puis cliquez sur **Modifier**.

1. Sélectionnez le paramètre **Activé**, puis cliquez sur **OK** :
 
   ![Paramètre d’arrêt](./media/how-to-windows-shutdown/edit-shutdown.png)

1. Notez que la commande d'arrêt n'apparaît plus sous le menu **Démarrer** de Windows ; seule la commande **Déconnecter** s’affiche.

    ![Commande d'arrêt](./media/how-to-windows-shutdown/start-menu.png)

## <a name="next-steps"></a>Étapes suivantes
Consultez l’article relatif à la préparation d’un modèle de machine virtuelle Windows : [Guide de configuration d’un modèle de machine Windows dans Azure Lab Services](how-to-prepare-windows-template.md)