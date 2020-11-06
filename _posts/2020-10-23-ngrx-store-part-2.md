---
layout: post
title: NgRx Store Tutorial - Recipe Book (Part 2)
tags: [lighthorse, angular, ngrx, typescript, angular cli]
excerpt_separator: <!--more-->
author: matthews58
---

Part two of NgRx Store Tutorial - Recipe Book post.

<!--more-->
### NgRx Effects

This blog post builds on [NgRx Store Tutorial - Recipe Book (Part 1)](https://teamlighthorse.github.io/2020/05/15/ngrx-store-tutorial.html) with the addition of @ngrx/effects that will handle side effects in our application.

What is a side effect?

When a user makes a change or submits something, we have to make a change on the server. This change on the server and the response to the client is a side effect.
An NgRx effect listens for particular action types that are triggered and then performs some side effect when that action happens.

In this example our use case for effects will be for loading data. The idea here is to have three actions: getData, getDataSuccess, and getDataFailure. We will set up an @Effect to listen for a particular action, in our case it will be getData, this will trigger the HTTP request. If the request is successfull we'll trigger the getDataSuccess action. If the request fails we'll trigger the getDataFailure action.

### Database and API

We need some test data so we can make an HTTP request and retrieve that data. This is just for testing purposes so I used json-server. [Json-server](https://www.npmjs.com/package/json-server) is a npm package that creates a rest json web service, backed by a simple database such as a json file.

  - install json-server using npm
    - `npm install json-server`
  - in the root of your project create a database file and name it `db.json`. Add any amount of recipes, for example:

```json
  {
  "recipes": [
    {
      "id": 1,
      "name": "Chicken and Rice",
      "ingredients": [
        {
          "name": "Chicken",
          "quantity": "1 lb"
        },
        {
          "name": "Rice",
          "quantity": "3/4 cup"
        },
        {
          "name": "water",
          "quantity": "1 cup"
        },
        {
          "name": "Condensed cream of mushroom soup",
          "quantity": "1 can (10.5 oz)"
        },
        {
          "name": "black pepper",
          "quantity": "1/4 tsp"
        },
        {
          "name": "paprika",
          "quantity": "1/4 tsp"
        }
      ]
    }
  ]
}
```
Let's start our json server
  - in the root of your project run:
  - `json-server ./db.json`

Go to: http://localhost:3000/recipes and verify that your data is there

### Angular Service

In order to communicate with our server we will need to import `HttpClientModule` into our app.module.ts so that we can inject `HttpClient` into our application class like our service we are about to create.

  - Add 'HttpClientModule' to app.module.ts:

  ```ts
  ...
  import { HttpClientModule } from '@angular/common/http';
  
  @NgModule({
  ...
    imports: [
      HttpClientModule,
      ...
    ]
  })
  ```

Let's create a service in Angular to communicate with our json-server 

  - `cd src/app/recipe`
  - `ng g service recipe`
    - This command will create a recipe.service.ts and recipe.service.spec.ts file

  - Add this to the recipe.service.ts file:

  ```ts
  import { Injectable } from '@angular/core';
  import { Observable } from 'rxjs';
  import { Recipe } from './recipe';
  import { HttpClient } from '@angular/common/http';
  import { delay } from 'rxjs/operators';

  @Injectable({
    providedIn: 'root'
  })
  export class RecipeService {

    constructor(private http: HttpClient) { }

    getRecipes(): Observable<Recipe[]> {
      return this.http.get<Recipe[]>('http://localhost:3000/recipes')
        .pipe(
          delay(500)
        ); 
    }

    addRecipe(recipe: Recipe) {
      return this.http.post<Recipe>('http://localhost:3000/recipes', recipe)
        .pipe(
          delay(500)
        );
    }

    deleteRecipe(recipeId: number) {
      return this.http.delete(`http://localhost:3000/recipes/${recipeId}`)
    }
  }
  ```

First thing we did here was inject the HttpClient into our constructor to perform HTTP requests. Then we've added three functions to GET recipes, POST a new recipe and DELETE a recipe. The delay is optional but I've added it to show a loading spinner in the UI which we'll get to later.


### Action Updates

We are going to edit our actions.ts file quite a bit compared to part 1 because we are adding more actions and we were using an older version of NgRx.

In order to update to NgRx 9 we will need to have the minimum versions installed: 
  - Angular version 9.x
  - Angular CLI version 9.0.1
  - Typescript version 3.7.x
  - RxJS version 6.5.x
Go to package.json file and verify that you have the minimum versions installed, if not run the following command: 
  - `ng update`
This command is using Angular CLI to update your application and its dependencies.
Now run this command to update NgRx to v9:
  - `ng update @ngrx/store`
Take a look in package.json again and verify that you have at least @ngrx version 9.x

In actions.ts add:

```ts
import { createAction, props } from '@ngrx/store';
import { Recipe } from '../recipe';

export enum RecipeActionTypes {
    GetRecipes = '[RECIPE] Get recipes',
    GetRecipesSuccess = '[RECIPE] Get recipes success',
    GetRecipesFailure = '[RECIPE] Get recipes failure',

    AddRecipe = '[RECIPE] Add recipe',
    AddRecipeSuccess = '[RECIPE] Add recipe success',
    AddRecipeFailure = '[RECIPE] Add recipe failure',

    DeleteRecipe = '[RECIPE] Delete recipe',
    DeleteRecipeSuccess = '[RECIPE] Delete recipe success',
    DeleteRecipeFailure = '[RECIPE] Delete recipe failure',
}

export const getRecipes = createAction(
    RecipeActionTypes.GetRecipes
);

export const getRecipesSuccess = createAction(
    RecipeActionTypes.GetRecipesSuccess,
    props<{ recipes: Recipe[] }>()
);

export const getRecipesFailure = createAction(
    RecipeActionTypes.GetRecipesFailure,
    props<{ error: any }>()
);

export const addRecipe = createAction(
	RecipeActionTypes.AddRecipe,
	props<{ recipe: Recipe }>()
);

export const addRecipeSuccess = createAction(
	RecipeActionTypes.AddRecipeSuccess,
	props<{ recipe: Recipe }>()
);

export const addRecipeFailure = createAction(
	RecipeActionTypes.AddRecipeFailure,
	props<{ error: Error }>()
);

export const deleteRecipe = createAction(
	RecipeActionTypes.DeleteRecipe,
	props<{ recipeId: number }>()
);
```
We now have a more asyncronous friendly actions file. As an example, getRecipes which allows us to call the `GetRecipes` action and then based on the response from the server it should either dispatch a `GetRecipesSuccess` or `GetRecipesFailure` action.
Then we've exported our actions so that we can dispatch them.

### Updated states

In part one I had you create a Ingredient.ts class and a recipe.ts class. For simplicity, we are going to combine those classes and add an extra property of an id to both classes so that we can remove that specific recipe. 
Go ahead and remove the ingredient.ts file and update the recipe.ts file to include:

```ts
export class Recipe {
  id: number;
  name: string;
  ingredients: Ingredient[];
}

export class Ingredient {
  id: number;
  name: string;
  quantity: string;
}
```

### Reducer Updates

Our reducer is going to change drastically as well due to the NgRx update and to handle the new actions we've added.
We also have included a RecipeState interface because we're dealing with asynchronous data so we have the option in the UI to show that we're loading and potential errors.
Here is the updated reducer file:

```ts
import * as actions from './actions';
import { Recipe } from '../recipe';
import { createReducer, on, Action } from '@ngrx/store';

export interface RecipeState {
  recipes: Recipe[],
  loading: boolean,
  error: Error
}

export const initialRecipeState: RecipeState = {
  recipes: [],
  loading: false,
  error: undefined
};

const reducer = createReducer(
  initialRecipeState,
	on(actions.getRecipes, state => {
		return {
      ...state,
      loading: true
    };
  }),
  on(actions.getRecipesSuccess, (state, { recipes })=> {
		return {
      ...state,
      recipes,
      loading: false
    };
  }),
  on(actions.getRecipesFailure, (state, { error }) => {
		return {
      ...state,
      error,
      loading: false
    };
  }),
  on(actions.addRecipe, state => {
    return {
      ...state,
      loading: true
    }
  }),
  on(actions.addRecipeSuccess, (state, { recipe }) => {
    return {
      ...state,
      loading: false,
      recipes: state.recipes.concat(recipe)
    }
  }),
  on(actions.addRecipeFailure, (state, { error }) => {
    return {
      ...state,
      loading: false,
      error
    }
  }),
  on(actions.deleteRecipe, (state, { recipeId }) => {
    return {
      ...state,
      loading: false,
      recipes: state.recipes.filter(recipe => recipe.id !== recipeId)
    }
  })
);

export function recipeReducer(state: RecipeState | undefined, action: Action) {
	return reducer(state, action);
}
```

With the NgRx update we can now create a reducer function which handles the actions for managing the state of the recipes. In part 1, we used a switch statement which was the previously defined way before reducer creators were introduced in the updated NgRx version. Here we use the NgRx `on` function that takes one or more actions with a given state change. Our reducer now is setting our loading property to true each time we make an async call. Depending on if that action succeeds or fails we set loading to false and use the payload to determine our UI.

### Effects

Install @ngrx/effects using Angular CLI to set up automatically:
  - `ng add @ngrx/effects`

[NgRx Effects guide](https://ngrx.io/guide/effects#writing-effects) does a really good job breaking down an Effect with an example of how to write one which is similar to what we're going to do.

Create a file in src/app/recipe/recipe-store/effects.ts and let's add effects for getRecipes and addRecipe:

```ts
import { Injectable } from '@angular/core';
import { ofType, Actions, createEffect } from '@ngrx/effects';
import * as recipeActions from './actions';
import { RecipeService } from '../recipe.service';
import { mergeMap, map, switchMap, catchError } from 'rxjs/operators';
import { Recipe } from '../recipe';
import { of } from 'rxjs';

@Injectable()
export class RecipeEffects {

    constructor(
        private actions: Actions,
        private recipeServie: RecipeService
    ) {}
 
    // listening for a recipe action of type getRecipes
    // then calls the recipe service to getRecipes which will return
    // an observable of type RecipeBook then we dispatch a getRecipes success action
    // which will stop the loading and return the recipes
    recipeBook$ = createEffect(() => this.actions.pipe(
        ofType(recipeActions.getRecipes),
        mergeMap(
            () => this.recipeServie.getRecipes()
            .pipe(
                map(recipes =>  {
                    return recipeActions.getRecipesSuccess({recipes})
                }),
                catchError(error => of(recipeActions.getRecipesFailure(error)))
            )
        )
    ));

    // listening for a recipe action of type addRecipe
    // then calls the recipe service to addRecipe which will return
    // an observable of type Recipe then we dispatch a addRecipe success action
    // which will stop the loading and add the new recipe into the recipe book
    saveRecipe = createEffect(() => this.actions.pipe(
        ofType(recipeActions.addRecipe),
        mergeMap(
            (action) => this.recipeServie.addRecipe(action.recipe)
                .pipe(
                    map((data: Recipe) => {
                        return recipeActions.addRecipeSuccess({recipe: data})
                    }),
                    catchError(error => of(recipeActions.getRecipesFailure(error)))
                )
            )
        )
    );
}
```

Here we first inject our Actions our recipeService which has all our api calls into the class. We are using NgRx `createEffect` factory method to create effects. Next we listen for specific actions by using `ofType`. We can then use rxjs operator `mergeMap` to merge these streams and call our service getRecipes() which returns an observable of type RecipeBook, with our recipes observable we can `map` over the response and either dispatch a getRecipesSuccess action if successful, which will then stop loading and return the recipes, or use catchError to dispatch an observable of(getRecipesFailure(error)). We use the exact pattern for our saveRecipe effect.

In our app.module.ts we'll need to add our Effects using `EffectsModule.forRoot()`:

```ts
  import { EffectsModule } from '@ngrx/effects';
  import { RecipeEffects } from './recipe/recipe-store/effects';
  
  @NgModule({
  ...
    imports: [
      EffectsModule.forRoot([RecipeEffects]),
      ...
    ]
  })
```

### Display in our UI

We can update our recipe.component.html file to see our effects work:

```html
<h1>Recipes</h1>
<div class="spinner" *ngIf="(loading$ | async)">
  <mat-spinner></mat-spinner>
</div>
<div *ngIf="(recipes$ | async) as recipes">
  <div *ngFor="let recipe of recipes">
    <mat-card class="recipeCard">
      <div class="recipeDelete">
          <button mat-icon-button
            class="float-right"
            (click)="deleteRecipe(recipe)">
            <mat-icon color="warn">
              clear
            </mat-icon>
          </button>
      </div>
      <div>
        <h3>
          {{recipe.name}}
        </h3>
        <ul>
          <li *ngFor="let ingredients of recipe.ingredients">
            {{ingredients.quantity}} {{ingredients.name}}
          </li>
        </ul>
      </div>
    </mat-card>
  </div>
</div>
```

What's different here from part 1, is that we've add a material spinner for when our state is loading and then we've just added some more styling for displaying the recipes.

We'll need to add a little css style to center the spinner in our recipe.component.css file:

```css
.spinner {
    display: flex; 
    align-items: center; 
    justify-content: center;
}
```

We'll also need to import the MatProgressSpinnerModule in order to use it. In material-module.ts add:
```ts
import {MatProgressSpinnerModule} from '@angular/material/progress-spinner';

@NgModule({
  exports: [
    ...
    MatProgressSpinnerModule
  ]
})
export class MaterialModule {}
```

We'll also need to update our recipe.component.ts file in order to use the observables in our template.

```ts
import { Component, OnInit } from "@angular/core";
import { Store } from "@ngrx/store";
import { Observable, Subscription } from "rxjs";
import { Recipe } from "./recipe";
import * as recipeActions from "./recipe-store/actions";
import { RecipeService } from './recipe.service';
import { RecipeState } from './recipe-store/reducer';

@Component({
  selector: "app-recipe",
  templateUrl: "./recipe.component.html",
  styleUrls: ["./recipe.component.css"]
})
export class RecipeComponent implements OnInit {
  recipes$: Observable<RecipeState>;
  loading$: Observable<boolean>;
  error$: Observable<any>;
  recipeList: Recipe[];
  isLoading: boolean;
  subscriptions: Subscription[] = [];

  constructor( 
    private store: Store<any>,
    private recipeService: RecipeService
    ) 
  { }

  ngOnInit(): void {
    this.recipes$ = this.store.select(store => store.recipe.recipes);
    this.loading$ = this.store.select(store => store.recipe.loading);
    this.error$ = this.store.select(store => store.recipe.error);
    
    this.store.dispatch(recipeActions.getRecipes());
  }

  ngOnDestroy(): void {
    this.subscriptions.forEach(s => s.unsubscribe());
  }

  deleteRecipe(recipe: Recipe) {
    const deleteSub = this.recipeService.deleteRecipe(recipe.id)
      .subscribe(
        () => this.store.dispatch(recipeActions.deleteRecipe({recipeId: recipe.id})),
        (error: any) => console.log('There was an error processing your request')
      )
    this.subscriptions.push(deleteSub);
  }
}
```

We create our three observable variables for recipes, loading and errors. OnInit() we set these variables by selecting a piece of our state. For example, we set our recipes$ observable by calling this.store.select(store => store.recipe.recipes) this `store.recipe` is coming from our app.module.ts file:
```ts 
StoreModule.forRoot({
  recipe: recipeReducer
}),
```

We see the property `recipe` is of type recipeReducer which is our function in our reducer file:
```ts
export interface RecipeState {
  recipes: Recipe[],
  loading: boolean,
  error: Error
}

...

export function recipeReducer(state: RecipeState | undefined, action: Action) {
	return reducer(state, action);
}
```
and the `recipes` comes from the RecipeState that we return in this function. Once we set our variables, we dispatch our getRecipes action which will kick off our recipeBook effect.

A few other aesthetic changes I made were:

create-recipe.component.ts:

```html
<h1>Create Recipe</h1>
<div class="row">
    <div class="col">
        <form [formGroup]="recipeForm">
            <mat-form-field>
                <input type="text" matInput
                    placeholder="Recipe Name"
                    formControlName="name"
                    required>
            </mat-form-field>
            <div formArrayName="ingredients">
                <div class="row" *ngFor="let ingredient of recipeForm.get('ingredients')['controls']; let i = index; last as isLast; first as isFirst" [formGroupName]="i">
                    <div class="col">
                        <mat-form-field class="space">
                            <mat-label>Ingredient Name</mat-label>
                            <input matInput 
                                type="text"
                                formControlName="name" 
                                placeholder="Ingredient Name">
                        </mat-form-field>
                        <mat-form-field>
                            <mat-label>Ingredient Quantity</mat-label>
                            <input matInput 
                                formControlName="quantity" 
                                placeholder="Ingredient Quantity">
                        </mat-form-field>
                        <button mat-icon-button 
                            type="button"
                            (click)="addIngredient()">
                            <mat-icon>add_circle_outline</mat-icon>
                        </button>
                        <button mat-icon-button 
                            type="button"
                            color="warn"
                            (click)="removeIngredient(i)">
                            <mat-icon>highlight_off</mat-icon>
                        </button>
                    </div>
                </div>
                <button class="buttons" mat-raised-button type="button" (click)="addRecipe()">Create Recipe</button>
            </div>
        </form>
    </div>
</div>
```

create-recipe.component.ts

```ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Recipe } from '../recipe/recipe';
import * as recipeActions from '../recipe/recipe-store/actions';
import { FormBuilder, FormGroup, FormArray, FormControl } from '@angular/forms';

@Component({
  selector: 'app-create-recipe',
  templateUrl: './create-recipe.component.html',
  styleUrls: ['./create-recipe.component.css']
})
export class CreateRecipeComponent implements OnInit {
  recipeForm: FormGroup;

  constructor(
    private store: Store<any>,
    private formBuilder: FormBuilder,
  ) {}

  ngOnInit(): void {
    this.createForm();
  }

  createForm(): void {
    this.recipeForm = this.formBuilder.group({
      name: '',
      ingredients: this.formBuilder.array([this.createIngredientFormGroup()])
    });
  }

  private createIngredientFormGroup(): FormGroup {
    return new FormGroup({
      'name': new FormControl(''),
      'quantity': new FormControl('')
    })
  }

  addIngredient(): void {
    const ingredients = this.recipeForm.get('ingredients') as FormArray;
    ingredients.push(this.createIngredientFormGroup());
  }

  
  removeIngredient(i: number) {
    const ingredients = this.recipeForm.get('ingredients') as FormArray;
    if (ingredients.length > 1) {
      ingredients.removeAt(i);
    } else {
      ingredients.reset();
    }
  }

  addRecipe() {
    if (this.recipeForm.valid) {
      const formValue = this.recipeForm.getRawValue();
      const recipeObject: Recipe = {
        id: undefined,
        name: formValue.name,
        ingredients: formValue.ingredients
      }
      this.store.dispatch(recipeActions.addRecipe({recipe: recipeObject}));
      this.recipeForm.reset();
    }
  }
}
```