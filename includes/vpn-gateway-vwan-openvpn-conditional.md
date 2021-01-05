---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 02/14/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 410570302eec418f1e4bcb75d6413936a96b5171
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77471514"
---
L’accès conditionnel permet un contrôle d’accès précis pour chaque application. Pour pouvoir utiliser l’accès conditionnel, vous devez avoir appliqué une licence Azure AD Premium 1 ou supérieure aux utilisateurs qui seront soumis aux règles de l’accès conditionnel.

1. Accédez à la page **Applications d’entreprise – Toutes les applications**, puis cliquez sur **VPN Azure**.

   - Cliquez sur **Accès conditionnel**.
   - Cliquez sur **Nouvelle stratégie** pour ouvrir le volet **Nouveau**.
2. Dans le volet **Nouveau**, accédez à **Attributions -> Utilisateurs et groupes**. Sous l’onglet **Utilisateurs et groupes ->** **Inclure** :

   - Cliquez sur **Sélectionner des utilisateurs et des groupes**.
   - Cliquez sur **Utilisateurs et groupes**.
   - Cliquez sur **Sélectionner** pour sélectionner le groupe ou l’ensemble d’utilisateurs qui doit être concerné par MFA.
   - Cliquez sur **Done**.

   ![Attributions](./media/vpn-gateway-vwan-openvpn-azure-ad-mfa/mfa-ca-assignments.png)
3. Dans le volet **Nouveau**, accédez au volet **Contrôles d’accès -> Accorder** :

   - Cliquez sur **Accorder l’accès**.
   - Cliquez sur **Exiger une authentification multifacteur**.
   - Cliquez sur **Demander tous les contrôles sélectionnés**.
   - Cliquez sur **Sélectionner**.
   
   ![Accorder l’accès – MFA](./media/vpn-gateway-vwan-openvpn-azure-ad-mfa/mfa-ca-grant-mfa.png)
4. Dans la section **Activer la stratégie** :

   - Sélectionnez **Activé**.
   - Cliquez sur **Créer**.

   ![Activer la stratégie](./media/vpn-gateway-vwan-openvpn-azure-ad-mfa/mfa-ca-enable-policy.png)