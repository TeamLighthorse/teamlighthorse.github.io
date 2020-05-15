---
layout: post
title: NgRx Store Tutorial - Recipe Book
tags: [lighthorse, new blog post]
excerpt_separator: <!--more-->
---

This post will walk you through creating a new Angular Project using NgRx Store.

<!--more-->


### Create a new project using Angular CLI

- Open a terminal window inside VSCode
    - `ng new [project name]`
    - `cd [project name]`


### Install NgRx Store

- `npm install @ngrx/store`


### Generate component

- `cd src/app/`
- `ng g c recipe`
- `cd recipe`


### Generate Ingredient and Recipe classes and add the following code

- `ng g class ingredient`

```ts
export class Ingredient {
    name: string;
    quantity: string;
}
```


- `ng g class recipe`

```ts
export class Recipe {
    name: string;
    ingredients: Ingredient[];
}
```


### Actions

Create a recipe-store folder with an actions.ts file inside and add the following code:

- `cd src/app/recipe/recipe-store/actions.ts`

Here we are creating an action; an action has two things:

- A type, which describes what is happening
- An optional payload of data

```ts
import { createAction, props } from '@ngrx/store';
import { Recipe } from '../recipe';

// 1
export enum RecipeAcionTypes {
  ADD_RECIPE = '[RECIPE] Add',
  REMOVE_RECIPE = '[RECIPE] Remove',
  UPDATE_RECIPE = '[RECIPE] Update'
}

// 2
export const addRecipe = createAction(
  RecipeAcionTypes.ADD_RECIPE,
  props<{ recipe: Recipe }>()
);

export const removeRecipe = createAction(
  RecipeAcionTypes.REMOVE_RECIPE,
  props<{ index: number }>()
);

export const updateRecipe = createAction(
  RecipeAcionTypes.UPDATE_RECIPE,
  props<{ recipe: Recipe }>()
);
```

1. We are defining the type of actions in the form of a string.
2. We are creating `consts` for our actions. 
    - We are using `props` to define our payload.


### State

Create a new file called state.ts and add the following code:

- `cd src/app/recipe/recipe-store/state.ts`

```ts
import { Ingredient } from './../ingredient';
import { Recipe } from '../recipe';

export const initialState: Recipe = {
  name: 'Chicken & Broccoli',
  ingredients: [
    {
      name: 'chicken',
      quantity: '12 oz'
    },
    {
      name: 'broccoli',
      quantity: '8 oz'
    }
  ]
};

export const emptyState: Recipe = {
  name: undefined,
  ingredients: undefined
};

export class RecipeState {
  name: string;
  ingredients: Ingredient[];
}
```

We will import this file within the components that we want to access ngrx store


### Reducer

Create a new file called reducer.ts and add the following code:

- `cd src/app/recipe/recipe-store/reducer.ts`

```ts
import { createReducer, on, Action } from '@ngrx/store';
import * as recipeActions from './actions';
import { initialState, RecipeState } from './state';

// 1
const featureReducer = createReducer(
  initialState,
  on(recipeActions.addRecipe, (state, { recipe }) => ({ ...state, recipe })),
  on(recipeActions.removeRecipe, (state, { index }) => ({ ...state, index })),
  on(recipeActions.updateRecipe, (state, { recipe }) => ({ ...state, recipe })),
);

export function reducer(state: RecipeState, action: Action) {
  return featureReducer(state, action);
}
```

1. We need to create a `const` for our reducer with `on` methods for each action. 
    - The first parameter of the `on` method is the action to trigger on.
    - The second parameter is a handler that takes in state and returns a new version of state.


### Import our reducer and storeModule

Open app.module.ts and update with the following code:

```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

import { StoreModule } from '@ngrx/store';
import { AppComponent } from './app.component';
import { RecipeComponent } from './recipe/recipe.component';
import { reducer } from './recipe/recipe-store/reducer';

@NgModule({
  declarations: [
    AppComponent,
    RecipeComponent
  ],
  imports: [
    BrowserModule,
    StoreModule.forRoot({
      recipe: reducer
    })
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```


### Generate a component to read from ngrx/store

- `ng g c read`

.....







