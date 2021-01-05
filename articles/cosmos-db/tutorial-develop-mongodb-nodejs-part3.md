---
title: Créer l’interface utilisateur de l’application Angular avec l’API d’Azure Cosmos DB pour MongoDB (Partie 3)
description: Il s’agit de la partie 3 de cette série de didacticiels sur la création d’une application MongoDB avec Angular et Node sur Azure Cosmos DB à l’aide des mêmes API que celles utilisées pour MongoDB.
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/26/2018
ms.author: jopapa
ms.custom: seodec18, devx-track-js
ms.reviewer: sngun
ms.openlocfilehash: 181af5cf26d19a9f51e8d456e777badf7efa224d
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93097819"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---build-the-ui-with-angular"></a>Créer une application Angular avec l’API d’Azure Cosmos DB pour MongoDB - Générer l’interface utilisateur avec Angular
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

Ce tutoriel en plusieurs parties montre comment créer une application écrite en Node.js avec Express et Angular, puis comment la connecter à votre [compte Cosmos configuré avec l’API de Cosmos DB pour MongoDB](mongodb-introduction.md).

La partie 3 de ce didacticiel est basée sur la [partie 2](tutorial-develop-mongodb-nodejs-part2.md) et aborde les tâches suivantes :

> [!div class="checklist"]
> * Générer l’interface utilisateur Angular
> * Utiliser CSS pour définir la forme et le fond
> * Tester l’application en local

## <a name="video-walkthrough"></a>Vidéo de procédure pas à pas

> [!VIDEO https://www.youtube.com/embed/MnxHuqcJVoM]

## <a name="prerequisites"></a>Prérequis

Avant de commencer cette partie du didacticiel, assurez-vous d’avoir effectué les étapes de la [partie 2](tutorial-develop-mongodb-nodejs-part2.md) du didacticiel.

> [!TIP]
> Ce didacticiel vous guide à travers les étapes pour générer l’application pas à pas. Si vous souhaitez télécharger le projet terminé, vous pouvez obtenir l’application complète à partir du [référentiel cosmosdb-angular](https://github.com/Azure-Samples/angular-cosmosdb) sur GitHub.

## <a name="build-the-ui"></a>Créer l’interface utilisateur

1. Dans Visual Studio Code, cliquez sur le bouton Arrêter :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part3/stop-button.png"::: pour arrêter l’application Node. 

2. Dans la fenêtre d’invite de commandes pour Windows ou la fenêtre Terminal pour Mac, entrez la commande suivante pour générer un composant héros. Dans ce code g = générer, c = composant, heroes = nom du composant, et il utilise une structure de fichier plat (--plat) afin de ne pas créer de sous-dossier.

    ```
    ng g c heroes --flat 
    ```

    La fenêtre du terminal affiche la confirmation des nouveaux composants.

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part3/install-heros-component.png" alt-text="Installation du composant héros":::

    Examinons les fichiers qui ont été créés et mis à jour. 

3. Dans Visual Studio Code, dans le volet **Explorateur** , accédez au nouveau dossier **src\app** et ouvrez le nouveau fichier **heroes.component.ts** généré dans le dossier de l’application. Ce fichier de composant TypeScript a été créé par la commande précédente.

    > [!TIP]
    > Si le dossier de l’application ne s’affiche pas dans Visual Studio Code, entrez Commande +Maj +P sur Mac ou Ctrl + Maj + P sur Windows pour ouvrir la palette de commandes, puis saisissez *Recharger la fenêtre* pour voir les modifications système.

4. Dans le même dossier, ouvrez le fichier **app.module.ts**. Vous remarquez que `HeroesComponent` a été ajouté aux déclarations dans la ligne 5 et importé dans la ligne 10.

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part3/app-module-file.png" alt-text="Ouvrez le fichier app-module.ts":::

5. Revenez au fichier **heroes.component.html** et copiez dans ce code. Le `<div>` est le conteneur pour toute la page. Dans le conteneur, il y a une liste de héros à créer de telle manière que lorsque vous cliquez sur l’un d’eux, vous pouvez le sélectionner pour le modifier ou le supprimer de l’interface utilisateur. L’HTML a été optimisé pour que vous sachiez lequel est sélectionné. Vous pouvez également ajouter un nouveau héros ou modifier un héros existant dans la zone d’édition. 

    ```html
    <div>
      <ul class="heroes">
        <li *ngFor="let hero of heroes" (click)="onSelect(hero)" [class.selected]="hero === selectedHero">
          <button class="delete-button" (click)="deleteHero(hero)">Delete</button>
          <div class="hero-element">
            <div class="badge">{{hero.id}}</div>
            <div class="name">{{hero.name}}</div>
            <div class="saying">{{hero.saying}}</div>
          </div>
        </li>
      </ul>
      <div class="editarea">
        <button (click)="enableAddMode()">Add New Hero</button>
        <div *ngIf="selectedHero">
          <div class="editfields">
            <div>
              <label>id: </label>
              <input [(ngModel)]="selectedHero.id" placeholder="id" *ngIf="addingHero" />
              <label *ngIf="!addingHero" class="value">{{selectedHero.id}}</label>
            </div>
            <div>
              <label>name: </label>
              <input [(ngModel)]="selectedHero.name" placeholder="name" />
            </div>
            <div>
              <label>saying: </label>
              <input [(ngModel)]="selectedHero.saying" placeholder="saying" />
            </div>
          </div>
          <button (click)="cancel()">Cancel</button>
          <button (click)="save()">Save</button>
        </div>
      </div>
    </div>
    ```

7. Maintenant que l’HTML est en place, nous devons ajouter l’ajouter au fichier **heroes.component.ts** pour interagir avec le modèle. Le code suivant ajoute le modèle à votre fichier de composant. Un constructeur a été ajouté. Il permet d’obtenir des héros et d’initialiser le composant du service de héros afin de récolter toutes les données. Ce code ajoute également toutes les méthodes nécessaires pour gérer les événements dans l’interface utilisateur. Vous pouvez copier le code suivant et remplacer le code existant dans **heroes.component.ts**. Des erreurs devraient s’afficher dans les zones Héros et HeroService puisque les composants correspondants ne sont pas encore importés, vous résoudrez ces erreurs dans la prochaine section. 

    ```ts
    import { Component, OnInit } from '@angular/core';

    @Component({
      selector: 'app-heroes',
      templateUrl: './heroes.component.html',
      styleUrls: ['./heroes.component.scss']
    })
    export class HeroesComponent implements OnInit {
      addingHero = false;
      heroes: any = [];
      selectedHero: Hero;
    
      constructor(private heroService: HeroService) {}
    
      ngOnInit() {
       this.getHeroes();
      }

      cancel() {
        this.addingHero = false;
        this.selectedHero = null;
      }

      deleteHero(hero: Hero) {
        this.heroService.deleteHero(hero).subscribe(res => {
          this.heroes = this.heroes.filter(h => h !== hero);
          if (this.selectedHero === hero) {
            this.selectedHero = null;
          }
        });
      }

      getHeroes() {
        return this.heroService.getHeroes().subscribe(heroes => {
          this.heroes = heroes;
        });
      }

      enableAddMode() {
        this.addingHero = true;
        this.selectedHero = new Hero();
      }

      onSelect(hero: Hero) {
        this.addingHero = false;
        this.selectedHero = hero;
      }

      save() {
        if (this.addingHero) {
          this.heroService.addHero(this.selectedHero).subscribe(hero => {
            this.addingHero = false;
            this.selectedHero = null;
            this.heroes.push(hero);
          });
        } else {
          this.heroService.updateHero(this.selectedHero).subscribe(hero => {
            this.addingHero = false;
            this.selectedHero = null;
          });
        }
      }
    }
    ```

8. Dans **Explorateur** , ouvrez le fichier **app/app.module.ts** et mettez à jour la section importations pour ajouter une importation pour un module `FormsModule`. La section importation doit maintenant ressembler à ce qui suit :

    ```
    imports: [
      BrowserModule,
      FormsModule
    ],
    ```

9. Dans le fichier **app/app.module.ts** , ajoutez une importation pour le nouveau module FormsModule dans la ligne 3. 

    ```
    import { BrowserModule } from '@angular/platform-browser';
    import { NgModule } from '@angular/core';
    import { FormsModule } from '@angular/forms';
    ```

## <a name="use-css-to-set-the-look-and-feel"></a>Utiliser CSS pour définir la forme et le fond

1. Dans le volet Explorateur, ouvrez le fichier **src/client/styles.scss**.

2. Copiez le code suivant dans le fichier **styles.scss** , et remplacez le contenu existant dans le fichier.

    ```css
    /* You can add global styles to this file, and also import other style files */

    * {
      font-family: Arial;
    }
    h2 {
      color: #444;
      font-weight: lighter;
    }
    body {
      margin: 2em;
    }
    
    body,
    input[text],
    button {
      color: #888;
      // font-family: Cambria, Georgia;
    }
    button {
      font-size: 14px;
      font-family: Arial;
      background-color: #eee;
      border: none;
      padding: 5px 10px;
      border-radius: 4px;
      cursor: pointer;
      cursor: hand;
      &:hover {
        background-color: #cfd8dc;
      }
      &.delete-button {
        float: right;
        background-color: gray !important;
        background-color: rgb(216, 59, 1) !important;
        color: white;
        padding: 4px;
        position: relative;
        font-size: 12px;
      }
    }
    div {
      margin: .1em;
    }

    .selected {
      background-color: #cfd8dc !important;
      background-color: rgb(0, 120, 215) !important;
      color: white;
    }

    .heroes {
      float: left;
      margin: 0 0 2em 0;
      list-style-type: none;
      padding: 0;
      li {
        cursor: pointer;
        position: relative;
        left: 0;
        background-color: #eee;
        margin: .5em;
        padding: .5em;
        height: 3.0em;
        border-radius: 4px;
        width: 17em;
        &:hover {
          color: #607d8b;
          color: rgb(0, 120, 215);
          background-color: #ddd;
          left: .1em;
        }
        &.selected:hover {
          /*background-color: #BBD8DC !important;*/
          color: white;
        }
      }
      .text {
        position: relative;
        top: -3px;
      }
      .saying {
        margin: 5px 0;
      }
      .name {
        font-weight: bold;
      }
      .badge {
        /* display: inline-block; */
        float: left;
        font-size: small;
        color: white;
        padding: 0.7em 0.7em 0 0.5em;
        background-color: #607d8b;
        background-color: rgb(0, 120, 215);
        background-color:rgb(134, 183, 221);
        line-height: 1em;
        position: relative;
        left: -1px;
        top: -4px;
        height: 3.0em;
        margin-right: .8em;
        border-radius: 4px 0 0 4px;
        width: 1.2em;
      }
    }

    .header-bar {
      background-color: rgb(0, 120, 215);
      height: 4px;
      margin-top: 10px;
      margin-bottom: 10px;
    }

    label {
      display: inline-block;
      width: 4em;
      margin: .5em 0;
      color: #888;
      &.value {
        margin-left: 10px;
        font-size: 14px;
      }
    }

    input {
      height: 2em;
      font-size: 1em;
      padding-left: .4em;
      &::placeholder {
          color: lightgray;
          font-weight: normal;
          font-size: 12px;
          letter-spacing: 3px;
      }
    }

    .editarea {
      float: left;
      input {
        margin: 4px;
        height: 20px;
        color: rgb(0, 120, 215);
      }
      button {
        margin: 8px;
      }
      .editfields {
        margin-left: 12px;
      }
    }
    ``` 
3. Enregistrez le fichier . 

## <a name="display-the-component"></a>Afficher le composant

Maintenant que nous avons le composant, comment l’afficher sur l’écran ? Commençons par modifier les composants par défaut dans **app.component.ts**.

1. Dans le volet Explorateur, ouvrez **/app/app.component.ts** , remplacez le titre par Heroes et placez le nom du composant que vous avez créé dans **heroes.components.ts** (app-heroes) pour faire référence à ce nouveau composant. Le contenu du fichier doit maintenant se présenter comme suit : 

    ```ts
    import { Component } from '@angular/core';

    @Component({
      selector: 'app-root',
      templateUrl: './app.component.html',
      styleUrls: ['./app.component.scss'],
      template: `
      <h1>Heroes</h1>
      <div class="header-bar"></div>
      <app-heroes></app-heroes>
    `
    })
    export class AppComponent {
      title = 'app';
    }

    ```

2. Nous faisons référence à d’autres composants dans **heroes.components.ts** , comme le composant héros, que nous devons également créer. Dans l’invite de commandes CLI Angular, utilisez la commande suivante pour créer un modèle héros et un fichier nommé **hero.ts** , où g = générer, cl = classe et hero = nom de classe.

    ```bash
    ng g cl hero
    ```

3. Dans le volet Explorateur, ouvrez **src\client\app\hero.ts**. Dans **hero.ts** , remplacez le contenu du fichier par le code suivant, afin d’ajouter une classe héros avec un ID, un nom et un message.

    ```ts
      export class Hero {
      id: number;
      name: string;
      saying: string;
    }
    ```

4. Revenez à **heroes.components.ts** et notez qu’à la ligne `selectedHero: Hero;` (ligne 10), `Hero` est souligné en rouge. 

5. Cliquez sur le terme `Hero`, et Visual Studio affiche une icône d’ampoule sur le côté gauche du bloc de code. 

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part3/light-bulb.png" alt-text="Icône Ampoule dans Visual Studio Code":::

6. Cliquez sur l’ampoule, puis cliquez sur **Importer héros depuis « /app/hero ».** ou **Importer héros depuis « ./hero ».** (Le message diffère selon votre configuration)

    Une nouvelle ligne de code apparaît à la ligne 2. Si la ligne 2 fait référence à client/app/hero, modifiez-la afin de référencer le fichier héros à partir du dossier local (./hero). La ligne 2 doit se présenter comme suit :

   ```
   import { Hero } from "./hero";
   ``` 

    Nous nous sommes occupés du modèle, mais nous devons encore créer le service.

## <a name="create-the-service"></a>Créer le service

1. Dans l’invite de commandes CLI Angular, entrez la commande suivante pour créer un service héros dans **app.module.ts** , où g = générer, s = service, hero = nom du service, -m = placer dans app.module.

    ```bash
    ng g s hero -m app.module
    ```

2. Dans Visual Studio Code, revenez à **heroes.components.ts**. Notez qu’à la ligne `constructor(private heroService: HeroService) {}` (ligne 13), `HeroService` est souligné en rouge. Cliquez sur `HeroService` et l’ampoule apparaît sur le côté gauche du bloc de code. Cliquez sur l’ampoule, puis sur **Importer HeroService depuis « ./hero.service ».** ou **Importer HeroService depuis « client/app/hero.service ».**

    Quand vous cliquez sur l’ampoule, une nouvelle ligne de code est insérée à la ligne 2. Si la ligne 2 fait référence au dossier /app/hero.service, modifiez-la afin de référencer le fichier hero à partir du dossier local (./hero.service). La ligne 2 doit se présenter comme suit :
    
    ```javascript
    import { HeroService } from "./hero.service"
    ```

3. Dans Visual Studio Code, ouvrez **hero.service.ts** et copiez le code suivant en remplaçant le contenu du fichier.

    ```ts
    import { Injectable } from '@angular/core';
    import { HttpClient } from '@angular/common/http';
    
    import { Hero } from './hero';
    
    const api = '/api';

    @Injectable()
    export class HeroService {
      constructor(private http: HttpClient) {}

      getHeroes() {
        return this.http.get<Array<Hero>>(`${api}/heroes`)
      }

      deleteHero(hero: Hero) {
        return this.http.delete(`${api}/hero/${hero.id}`);
      }

      addHero(hero: Hero) {
        return this.http.post<Hero>(`${api}/hero/`, hero);
      }

      updateHero(hero: Hero) {
        return this.http.put<Hero>(`${api}/hero/${hero.id}`, hero);
      }
    }
    ```

    Ce code utilise la dernière version de HttpClient offerte par Angular. Comme vous devez fournir ce module, ce sera la prochaine étape.

4. Dans Visual Studio Code, ouvrez **app.module.ts** et importez le module HttpClientModule en mettant à jour la section d’importation afin d’inclure HttpClientModule.

    ```ts
    imports: [
      BrowserModule,
      FormsModule,
      HttpClientModule
    ],
    ```

5. Dans **app.module.ts** , ajoutez le module HttpClientModule et importez les instructions de la liste d’importation.

    ```ts
    import { HttpClientModule } from '@angular/common/http';
    ```

6. Enregistrez tous les fichiers dans Visual Studio Code.

## <a name="build-the-app"></a>Générer l’application

1. Dans l’invite de commandes, entrez la commande suivante pour générer l’application Angular. 

    ```bash
    ng b
    ``` 

    Si vous rencontrez des problèmes, la fenêtre de terminal affiche des informations sur les fichiers à corriger. Une fois la génération terminée, les nouveaux fichiers sont envoyés dans le dossier **dist**. Vous pouvez consulter les nouveaux fichiers dans le dossier **dist** si vous le souhaitez.

    Exécutons maintenant l’application.

2. Dans Visual Studio Code, cliquez sur le bouton **Déboguer** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part2/debug-button.png"::: sur le côté gauche, puis cliquez sur le bouton **Démarrer le débogage** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part3/start-debugging-button.png":::.

3. Ouvrez un navigateur internet et accédez à **localhost:3000** pour voir l’application s’exécuter en local.

     :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part3/azure-cosmos-db-mongodb-mean-app.png" alt-text="Application héros s’exécutant localement":::

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du didacticiel, vous avez :

> [!div class="checklist"]
> * Généré l’interface utilisateur Angular
> * Testé l’application en local

Vous pouvez passer à la partie suivante du didacticiel afin de créer un compte Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Créer un compte Azure Cosmos DB à l’aide d’Azure CLI](tutorial-develop-mongodb-nodejs-part4.md)
