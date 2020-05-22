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
      - You can opt out of Angular routing
      - Choose css for stylesheet format
    - `cd [project name]`

### Install NgRx Store

- `npm install @ngrx/store`

### Install Angular Material and Bootstrap for styling

- `ng add @angular/material`
  - Choose custom theme

- `npm install --save bootstrap`
- `npm install --save jquery`

Open angular.json and replace `styles` and `scripts` with this:

```json
"styles": [
  "./node_modules/bootstrap/dist/css/bootstrap.css",
  "src/custom-theme.scss",
  "src/styles.css"
],
"scripts": [
  "./node_modules/jquery/dist/jquery.js",
  "./node_modules/bootstrap/dist/js/bootstrap.js"
]
```

Create a new file called material-module.ts and import the following modules from Angular Material that we will use later

- `cd src/material-module.ts`

```ts
import {FormsModule, ReactiveFormsModule} from '@angular/forms';
import {MatButtonModule} from '@angular/material/button';
import {MatInputModule} from '@angular/material/input';
import {NgModule} from '@angular/core';

@NgModule({
  exports: [
    FormsModule,
    ReactiveFormsModule,
    MatButtonModule,
    MatInputModule
  ]
})
export class MaterialModule {}
```
Open app.module.ts and import MaterialModule:

```ts
import { MaterialModule } from './material-module';
...
imports: [
    MaterialModule,
    ...
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

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
import { Ingredient } from './ingredient';

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
import { Action } from '@ngrx/store';
import { Recipe } from '../recipe';

// 1
export const  ADD_RECIPE = '[RECIPE] Add'

// 2
export class addRecipe implements Action {
    readonly type = ADD_RECIPE

    constructor(public payload: Recipe) {}
}

// 3
export type Actions = addRecipe;
```

1. We are defining the type of action in the form of a string.
2. We are creating a class for each action with a constructor where pass in the payload.
3. We are exporting our action classes so that we can use them in our reducer.


### State

Create a new file called state.ts and add the following code:

- `cd src/app/recipe/recipe-store/state.ts`

```ts
import { Ingredient } from '../ingredient';

export class RecipeState {
  name: string;
  ingredients: Ingredient[];
}
```

We will import this file within the components that we want to access ngrx store


### Reducer

A reducer is a pure function that change state, by that I mean they make a copy of the existing state and can change properties on the new state.

Create a new file called reducer.ts and add the following code:

- `cd src/app/recipe/recipe-store/reducer.ts`

```ts
import * as recipeActions from './actions';
import { RecipeState } from './state';

// 1
export const initialState: RecipeState[] = [
  {
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
  },
  {
    name: 'Pasta',
    ingredients: [
      {
        name: 'pasta',
        quantity: '1 box'
      },
      {
        name: 'cheese',
        quantity: '1/2 cup'
      }
    ]
  }
];

// 2
export function reducer(state: RecipeState[] = initialState, action: recipeActions.Actions) {

// 3
  switch(action.type) {
      case recipeActions.ADD_RECIPE:
          return [...state, action.payload];
      default:
          return state;
  }
}
```

1. Here we are creating our initial state, this is not necessary if you don't want to define a state right away.
2. This is our reducer that takes in a state, this is where we set our initialState and an action.
3. We use a switch case to determine the type of action. If the case is adding a recipe, we return the new state which takes in the previous state and our action.
  - `...state` is called a spread operator and it makes a copy of the exisiting state


### Import our reducer and storeModule

Open app.module.ts and update with the following code:

```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
import { MaterialModule } from 'src/material-module';
import { RecipeComponent } from './recipe/recipe.component';
import { StoreModule } from '@ngrx/store';
import { reducer } from './recipe/recipe-store/reducer';

@NgModule({
  declarations: [
    AppComponent,
    RecipeComponent
  ],
  imports: [
    MaterialModule,
    BrowserModule,
    AppRoutingModule,
    BrowserAnimationsModule,
    StoreModule.forRoot({
      recipe: reducer
    })
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```


### Reading from ngrx store

First, open recipe.component.ts and add the following code:

```ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Recipe } from './recipe';

@Component({
  selector: 'app-recipe',
  templateUrl: './recipe.component.html',
  styleUrls: ['./recipe.component.css']
})
export class RecipeComponent implements OnInit {
  // 1
  recipes$: Observable<Recipe[]>;

  constructor(
    private store: Store<any>
  ) { }

  // 2
  ngOnInit(): void {
    this.recipes$ = this.store.select('recipe');
  }
}
```

1. We are defining an Observable called recipes$ which will be used to display the recipes in the template.
  - Observables are often named with a "$" sign at the end so it's easier to find observables when looking through the code
2. We are accessing the ngrx store on ngOnInit()
  - We are selecting `recipe` which is defined as a property in `app.module.ts` like this: 
```ts
  StoreModule.forRoot({
      recipe: reducer
    })
``` 
This calls the recipe reducer and returns the recipe state.

Next, open recipe.component.html and add the following code:

```html
<h1>Recipes</h1>
<div *ngFor="let recipe of recipes$ | async">
  <h3>
    {% raw %}{{recipe.name}}{% endraw %}
  </h3>
  <ul>
    <li *ngFor="let ingredients of recipe.ingredients">
      {% raw %}{{ingredients.quantity}} {{ingredients.name}}{% endraw %}
    </li>
  </ul>
</div>
```

- The `async` pipe allows us to subscribe to an observable inside the template. It will also automatically unsubscribe so we don't have to do this in the component either.

Next, open app.component.html and remove all code and add:

```html
<div class="container">
    <div class="row">
        <div class="col-6">
            <app-recipe></app-recipe>
        </div>
    </div>
</div>
```

Finally, we have read from ngrx store. 
You can now save your project and in the terminal in the /src/ folder run:

- `ng serve --o`

This will open http://localhost:4200 and you should get something that looks like this:

<kbd>
  <img src="/assets/img/read-ngrx-store.png">
</kbd>

### Writing to ngrx store

Here we are going to be calling our ADD_RECIPE action.

First, create a new component called 'create-recipe'

- `cd /src/app/`
- `ng g c create-recipe`

Next, open create-recipe.component.ts and add the following code:

```ts
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Recipe } from '../recipe/recipe';
import * as recipeActions from '../recipe/recipe-store/actions';
import { FormBuilder, FormGroup, FormArray } from '@angular/forms';


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
  ) {
    this.createForm();
  }


  ngOnInit(): void {
  }

// 1
  createForm(): void {
    this.recipeForm = this.formBuilder.group({
      name: '',
      ingredients: this.formBuilder.array([this.ingredients])
    });
  }

// 2
  get ingredients(): FormGroup {
    return this.formBuilder.group({
      name: '',
      quantity: ''
    });
  }

// 3
  addIngredient(): void {
    (this.recipeForm.controls['ingredients'] as FormArray).push(this.ingredients);
  }

// 4
  addRecipe() {
    if (this.recipeForm.valid) {
      const formValue = this.recipeForm.getRawValue();
      const recipeObject: Recipe = {
        name: formValue.name,
        ingredients: formValue.ingredients
      }
      this.store.dispatch(new recipeActions.addRecipe({ name: recipeObject.name, ingredients: recipeObject.ingredients }));
      this.recipeForm.reset();
    }
  }
}
```
1. We need to initialize the formGroup `this.recipeForm` with a `name` and a formArray of `ingredients` where we are passing in a formGroup, `this.ingredients`.
  - This allows us to add multiple ingredients of formGroups into a formArray.
2. Here we are getting the ingredients, which is a formGroup with a `name` and `quantity`.
3. In the template there is a button to 'Add Ingredient' this will call this method and add another formGroup of `name` and `quantity` to the form itself.
4. In the template there is another button to 'Add Recipe' this will call this method.
  - First, it will check to make sure the form is valid
  - Next, it will get the form values and create an object of type `Recipe`
  - Then, we can call `this.store.dispatch` which takes in an object of type `Recipe` that has a name and ingredients property.
  - Finally, we clear the form so it's ready to add a new recipe.

Now open create-recipe.component.html and add the following code:

```html
<h1>Add New Recipe</h1>
<div class="row">
    <div class="col">
        <form [formGroup]="recipeForm">
            <mat-form-field>
                <input type="text" matInput
                    placeholder="Recipe Name"
                    formControlName="name"
                    required>
            </mat-form-field>
            <div class="row" formArrayName="ingredients">
                <div class="col">
                     <div *ngFor="let test of recipeForm.get('ingredients')['controls']; let i = index" [formGroupName]="i">
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
                        </mat-form-field><br/>
                    </div>                    
                    <button class="buttons" mat-raised-button type="button" (click)="addIngredient()">Add Ingredient</button>
                    <button class="buttons" mat-raised-button type="button" (click)="addRecipe()">Add Recipe</button>
                </div>
            </div>
        </form>
    </div>
</div>
```
Add some styling to create-recipe.component.css:

```css
.space {
    padding: 10px 20px 10px 0;
}

.buttons {
    margin-right: 10px;
}
```

Update app.component.html to display the create-recipe component

```html
<div class="container">
    <div class="row">
        <div class="col-6">
            <app-create-recipe></app-create-recipe>
        </div>
        <div class="col-6">
            <app-recipe></app-recipe>
        </div>
    </div>
</div>
```

We can now write to ngrx store. 
You can now save your project and open http://localhost:4200 and you should see something like this:

<kbd>
  <img src="/assets/img/write-ngrx-store-1.png">
</kbd>

Adding a recipe:

<kbd>
  <img src="/assets/img/write-ngrx-store-2.png">
</kbd>

<kbd>
  <img src="/assets/img/write-ngrx-store-3.png">
</kbd>