---
author: cephalin
ms.service: app-service-web
ms.topic: include
ms.date: 11/03/2016
ms.author: cephalin
ms.openlocfilehash: 2f490a5b12484a91e963d068810b292d7761521a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95986538"
---
> [!NOTE]
> Avec Azure DNS, vous pouvez configurer un nom DNS personnalisé pour Azure App Service. Pour plus d’informations, consultez [Use Azure DNS to provide custom domain settings for an Azure service](../articles/dns/dns-custom-domain.md#app-service-web-apps) (Utiliser DNS Azure pour fournir des paramètres de domaine personnalisé pour un service Azure).
>
>

1. Connectez-vous au site web de votre fournisseur de domaine.

1. Trouvez la page de gestion des enregistrements DNS. Chaque fournisseur de domaine ayant sa propre interface d’enregistrements DNS, consultez la documentation de votre fournisseur. Recherchez les zones du site qui portent les mentions **Nom de domaine**, **DNS** ou **Gestion du nom de serveur**.

   Vous trouvez généralement la page des enregistrements DNS en affichant vos informations de compte, puis en recherchant un lien comme **Mes domaines**. Accédez à cette page, puis recherchez un lien nommé comme **Fichier de zone**, **Enregistrements DNS**, ou **Configuration avancée**.

   La capture d’écran suivante est un exemple d’une page d’enregistrements DNS :

   ![Capture d’écran montrant un exemple de page d’enregistrements DNS.](./media/app-service-web-access-dns-records-no-h/example-record-ui.png)

1. Dans l’exemple de capture d’écran, sélectionnez **Ajouter** pour créer un enregistrement. Certains fournisseurs ont différents liens pour ajouter divers types d’enregistrements. Là encore, consultez la documentation du fournisseur.

> [!NOTE]
> Pour certains fournisseurs tels que GoDaddy, les modifications apportées aux enregistrements DNS n’entrent en vigueur que lorsque vous cliquez sur un lien **Enregistrer les modifications** distinct.
