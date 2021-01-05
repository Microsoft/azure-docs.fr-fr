---
title: Bien démarrer avec TmaxSoft OpenFrame dans les machines virtuelles Azure
description: Réhébergez vos charges de travail mainframe IBM z/OS en utilisant l’environnement TmaxSoft OpenFrame sur des machines virtuelles Azure.
author: njray
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.openlocfilehash: 215a17ced6be4cc8792ac1a06115450bfbccac99
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94963264"
---
# <a name="get-started-with-tmaxsoft-openframe-on-azure"></a>Bien démarrer avec TmaxSoft OpenFrame sur Azure

Déplacez vos ressources mainframe existantes dans Microsoft Azure à l’aide de TmaxSoft OpenFrame. Cette solution de réhébergement à succès crée un environnement d’émulation dans Azure, ce qui vous permet de migrer rapidement les applications. Aucun reformatage n’est nécessaire.

## <a name="openframe-rehosting-environment"></a>Environnement de réhébergement OpenFrame

Configurez un environnement OpenFrame dans Azure pour des charges de travail de développement, de démonstration, de test ou de production. Comme le montre la figure suivante, OpenFrame comporte plusieurs composants qui créent l’environnement d’émulation mainframe dans Azure. Par exemple, les services en ligne OpenFrame remplacent l’intergiciel (middleware) mainframe comme IBM Customer Information Control System (CICS). Avec son composant TJES, OpenFrame Batch remplace le sous-système d’entrée de travail (JES) du mainframe IBM. 

![Processus de réhébergement OpenFrame](media/openframe-01.png)

> [!NOTE]
> Pour exécuter l’environnement OpenFrame dans Azure, vous devez disposer d’une licence produit valide ou d’une licence d’essai fournie par TmaxSoft.

## <a name="openframe-components"></a>Composants d’OpenFrame

Les composants suivants font partie de l’environnement OpenFrame dans Azure :

- **Outils de migration**, dont OFMiner, solution qui analyse les ressources mainframe avant de les migrer vers Azure.
- **Compilateurs**, dont OFCOBOL, compilateur qui interprète les programmes COBOL du mainframe ; OFPLI, qui interprète les programmes PL/I du mainframe ; et OFASM, compilateur qui interprète les programmes assembleurs du mainframe.
- Composants **front-end**, dont JEUS (Java Enterprise User Solution), serveur d’application web certifié pour Java Enterprise Edition 6.OFGW et le composant passerelle OpenFrame qui fournit un écouteur 3270.
- Environnement d’**application**. OpenFrame Base est l’intergiciel (middleware) qui gère l’ensemble du système. OpenFrame Server Type C (OSC) remplace l’intergiciel (middleware) du mainframe et IBM CICS.
- **Base de données relationnelle**, comme Tibero (présentée ici), Oracle Database, Microsoft SQL Server, IBM Db2 ou MySQL. Les applications OpenFrame utilisent le protocole ODBC (Open Database Connectivity) pour communiquer avec la base de données.
- La **sécurité** via TACF, module de service qui contrôle l’accès utilisateur aux systèmes et aux ressources. 
- **OFManager** est une solution qui fournit les fonctions d’exploitation et de gestion d’OpenFrame dans l’environnement web.

![Architecture OpenFrame](media/openframe-02.png)

## <a name="next-steps"></a>Étapes suivantes

- [Installer TmaxSoft OpenFrame sur Azure](./install-openframe-azure.md)
