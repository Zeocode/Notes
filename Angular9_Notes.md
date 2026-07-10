# Angular 9 Notes

---

## Angular Framework

- TypeScript Framework — open source
- Frontend Framework
- Makes it easy to build applications within **web / mobile / desktop**

### Major Features
- Declarative templates, dependency injection, unit testing

---

## Angular 9 Supports

1. **Ivy Engine** (Rendering Engine)
2. **Bazel Script** (Faster build and testing)

---

## Angular 9 Features

1. Ivy Engine (new default rendering engine)
2. Smaller builds
3. Modularity
4. Speed and Performance
5. Cross Platform — Mobile, Web, Desktop
6. Improved Lazy Loading & i18n improvements *(Note: "EPA" in original notes was unclear/incorrect)*
7. TypeScript language support

> Angular versions are backward compatible

---

## AngularJS vs Angular 2+

| AngularJS | Angular 2+ |
|-----------|------------|
| JavaScript based | TypeScript based (Model-View-ViewModel) |
| MVC | MVVM |
| No mobile support | Mobile support |
| No plugin development | Plugin development |

---

## Upgrade / Update Commands

```bash
# Upgrade to a specific version
ng update @angular/cli@8 @angular/core@8
# Upgrade to latest
ng update @angular/cli @angular/core
```

---

## Install and Run Angular App

```bash
npm i -g @angular/cli            # ng --version
ng new <project-name>            # should go inside project folder to run it
ng serve                         # ng serve --port 4200
```

### Install Bootstrap

```bash
npm i bootstrap
# @import '~bootstrap/dist/css/bootstrap.min.css';  → in styles.css file
```

---

## Angular Folder Structure

1. **e2e** — end-to-end test scripts; sending more Protractor Framework used to run end-to-end tests
   - `app.po.ts` — Protractor will have `.po` file extension
   - `app.e2e-spec.ts` — end-to-end test script

2. **node_modules** — folder where all modules and libraries reside

3. **src** → **app** — core/area of the application coding
   - **env** — we configure various environments for:
     - development stage
     - testing stage
     - production stage
   - **assets** — MVM
   - **environments**
   - **images**
   - **styles**
   - **icons**
   - **polyfills** — browser outdated → then polyfills provide ES6 functionality (backward compatibility)
   - **Main.ts** — booting the Angular application

4. **angular.json** — file for all the configuration of our Angular project
5. **package.json** — dependencies of the project
   - detailed dependencies / production / config
6. **package-lock.json** — modules versions of dependency that Angular project requires (sub modules)
7. **karma.conf.json** — Karma runner file — configure and test script
8. **styles.css**

---

## JIT vs AOT

| JIT (Just In Time) | AOT (Ahead Of Time) |
|--------------------|---------------------|
| Loads application slower since it compiles | Faster load, no compilation during runtime |
| Downloads compiler and adds it to code — big size | Doesn't download compiler; compiles the application in building stage (small size) |
| Suitable in development mode | Suitable in production mode |
| `ng build` / `ng serve` | `ng build --aot` / `ng serve --aot` |
| Template binding errors shown at runtime | Template binding errors shown at build time |

---

## Angular CLI

1. Command line interface tool to create the application and run it.
   - `ng new <project-name>` — to create components, modules, directives, pipes

```bash
ng --version
ng generate component login    # or ng g c login
ng generate service auth-service
ng generate module auth
ng g pipe highercyn
ng serve                       # transpile from TypeScript to vanilla JS
```

---

## Modules

- Logical functional group
- Plug and play features
- Allows to be modular / easy to maintain / easy to deploy
- Reduce app performance overhead by not loading modules which are not required

Angular must have one module — by default: **AppModule**

1. **Entry Module** — needs to be defined by `@NgModule`
   - declares components
   - imports other modules
   - providers: services used
   - bootstrap — which component to load

2. **Custom Modules** — `ng g module <module-name>`

---

## Decorators

- TypeScript features; it is used to pass meta-data

**Types:**
- Class Decorators — `@Component`, `@NgModule`
- Property Decorators — `@Input`, `@Output`, `@ViewChild`, `@ContentChild` *(correction: @ContentChild is a Property decorator, not Method)*
- Method Decorators — `@HostListener`
- Parameter Decorators — `@Inject`

---

## Components

1. Smallest block / unit of functionality
2. Plug and play
3. Easy maintenance and development
4. Tree hierarchy
5. **App Component** — Root
   - By default, in index.html `<app-root>`
6. Component → **Selector** / **Template URL** / **StylesURL**

```bash
ng generate component <component-name>   # or: ng g c <component-name>
```

7. Components inside components (use component selector inside parent component)
8. `ng generate component <component-name>` *(Correction: NOT `ng component` — that is invalid)*
9. Location of generation matters — goes to respective modules; parent module updates accordingly

---

## Directives

- Special instructions used to modify HTML elements

**Types:**

1. **Component Directives** — special directive which has its own template and can add elements in HTML.

2. **Structural Directives** — add/delete HTML elements
   - `*ngFor`, `*ngIf`, `*ngSwitch`

3. **Attribute Directives** — modifies attributes of HTML elements
   - `[ngStyle]`, `[ngClass]`, `[(ngModel)]`

### Examples:

```html
<!-- ngClass -->
<div [ngClass]="isSpecial ? 'special' : ''">This div is special</div>

<!-- ngStyle -->
<div *ngIf="showMsg" [ngStyle]="{'color': 'red'}">
</div>

<div *ngIf="showMsg" [ngStyle]="{'color': 'colorVal'}">
</div>
```

### Structural Directives:

```html
<!-- ngIf -->
<div *ngIf="condition" then thenBlock else elseBlock>
  <ng-template #thenBlock> true </ng-template>
  <ng-template #elseBlock> false </ng-template>
</div>

<!-- ngFor -->
<div *ngFor="let contact of contactList">
  <div [ngSwitch]="'switch expression'">
    <div *ngSwitchCase="'name1'">...</div>
    <div [ngSwitchDefault]>...</div>
  </div>
</div>
```

---

## Data Binding

Defines how data flows and how the data gets updated.

### 1-Way Data Binding

**Component to View:**
- Interpolation
- Property Binding
- Style Binding
- Attribute Binding

**View to Component:**
- Event Binding

### 2-Way Data Binding
- `ngModel`

---

## Binding Examples

### ① Interpolation
```html
<div>{{ title }}</div>
```

### ② Property Binding
```html
[property] = "expression"
```

**app.component.html:**
```html
<div [ngStyle]="{'color': colorVal}"></div>
<p [innerHTML]="'substitute'"></p>
<div [ngClass]="'all'">Some Text</div>
<input [placeholder]="'placeholderVal'">
<a [href]="hrefVal">Google</a>
<a [href]="hrefVal" [auto.updateLink]>UpdatedLink > Google</a>
```

**app.component.ts:**
```typescript
colorVal = 'red';
placeholder = 'Enter value';
hrefValue = 'http://google.com';
updatedLink = 'http://localhost.com';
```

### ③ Event Binding (from View to Component)

**app.component.html:**
```html
<button (click)="showAlert()">Click Me</button>
```

**app.component.ts:**
```typescript
showAlert() {
  alert('Hello from AlertFunction!');
}
```

---

## Template Reference Variables

A **template reference variable** uses the `#` symbol to declare a variable that refers to a DOM element, component, or directive within the template. It can then be used elsewhere in the same template.

### Syntax
```html
<element #variableName></element>
```

### Examples

```html
<!-- Referencing an input element -->
<input #myInput type="text">
<button (click)="greet(myInput.value)">Greet</button>

<!-- Accessing value directly in template without going to component -->
<input #nameRef type="text">
<p>{{ nameRef.value }}</p>

<!-- Referencing a child component -->
<app-child #childRef></app-child>
<button (click)="childRef.someMethod()">Call Child Method</button>
```

### In Component Class (using @ViewChild)
```typescript
import { ViewChild, ElementRef } from '@angular/core';

@ViewChild('myInput') myInput: ElementRef;

ngAfterViewInit() {
  this.myInput.nativeElement.focus();  // access the DOM element directly
}
```

> ⚠️ **Note:** Template reference variables are **only accessible within the template** they are declared in. To access them in the component class, use `@ViewChild`.

---

## Two-Way Data Binding — ngModel

> ⚠️ **Important:** `[(ngModel)]` requires **`FormsModule`** to be imported in `app.module.ts`

```typescript
import { FormsModule } from '@angular/forms';
@NgModule({ imports: [FormsModule] })
```

**app.component.html:**
```html
<input [(ngModel)]="firstName">
<div>{{ firstName }}</div>
```

---

## Component Interaction (b/w Child and Parent)

- `@Input` — Parent to Child
- `@Output` (EventEmitter) — Child to Parent

**Inside parent HTML:**
```html
<app-input-output [item]="currentItem" (newItemEvent)="addItem($event)">
</app-input-output>
```

**Child Template:**
```typescript
@Output() newItemEvent = new EventEmitter<string>();

addNewItem(value: string) {
  this.newItemEvent.emit(value);
}
```

**Child HTML:**
```html
<input type="text" id="item-input" #newItem>
<button type="button" (click)="addNewItem(newItem.value)">
```

**Parent Component:**
```typescript
items = ['item1', 'item2', 'item3', 'item4'];

addItem(newItem: string) {
  this.items.push(newItem);
}
```

**Parent Template:**
```html
<app-item-output (newItemEvent)="addItem($event)">
</app-item-output>
```

---

## Routing

**Topics covered:**
- Routing
- Routing Strategy
- Base HREF
- Routes Module
- Router Outlet
- Configuring Routes
- Router Link — `<a routerLink="/home">` or `ng this.router.navigate(['/home'])`
- Parameterized Routes
- Wildcard Routes
- Query Params in Routes
- Child Routes
- Modules with specific Routes
- Lazy Loading
- Route Guards

### Routing Definition

- Mechanism to manage "pages" and "routing"
- Navigation between various views
- Router Module — has everything
- Routes — singleton — only one instance

```
@angular/router
- array
- states
- All batteries included for Router — everything can be done with Routes itself
```

---

## ng build vs ng serve

| ng build | ng serve |
|----------|----------|
| Used for building the app and deploying build artifacts | Used for fast, local and iterative development |
| — | Changes reflected simultaneously |
| `ng build` writes generated build artifacts to output folder — `/dist/` | `ng serve` does not do this; writes build artifacts to memory for faster development |
| If we want to reflect change of our source code to app: `ng build --watch` | — |

---

## How Angular Loads the Application

1. **angular.json** — contains various properties and configuration of Angular project
   - It has reference to `main.ts`
   - Builder looks for this file; mentions it in angular.json

2. **main.ts** — entry section
   - Calls `bootstrapModule(AppModule)`
   - Tells how to bootstrap via app
   - Can be changed to other module

3. **AppModule** — contains bootstrap: `AppComponent`
   - Contains import array for sub-modules
   - Contains declaration array for sub-components
   - Then `index.html` loaded

---

## Routing Strategy

### Angular Routings:
1. All routing/path or navigation requirements are handled by Router package
2. Navigate from one component view to another component using the router
3. Configure various types of routes:
   - Default routes
   - Child routes
   - Wildcard routes
   - Query params
   - Not required
   - Lazy loading

4. **Route array** — required once; searched in this array
   - `app-routing.module`

### Routing Location Strategy:

1. **Path Location** or **Hash Location**
   - `/home` → `#/home`

2. Choose any strategy
3. No constraints or restrictions

**app.module:**
```typescript
providers: [
  { provide: LocationStrategy, useClass: HashLocationStrategy }
]
```

### Routing — Base HREF
- This means default cause: `("/")` 
- `ng build --base-href "/demo"` → to change base HREF
- Wrong configuration leads to wrong folder.

---

## Routing Module

- Placeholder for all routes
- Best practice to have all routes in one place
- `ng generate module app-routing --flat --module=app`

```typescript
import { Routes, RouterModule } from '@angular/router';

const routes: Routes = [];   // to configure and add routes

// Define our module
@NgModule({
  imports: [RouterModule.forRoot(routes)],  // forRoot() — used ONLY in root AppRoutingModule
  exports: [RouterModule]
})

export class AppRoutingModule {}
```

> ⚠️ **Important:** Use `RouterModule.forRoot(routes)` only in the **root** routing module. For **feature/lazy-loaded modules**, use `RouterModule.forChild(routes)` instead.

### Router Outlet

- Defines where the output needs to be displayed for a route
- More than 1 router outlets allowed
- Can be specified at App Module or in individual module

```html
<router-outlet></router-outlet>
```

### Configuring Routes

- Path
- Component
- redirectTo
- children

> **Note:** To lazy load module — `ng generate module <name> --routing true --module app -- route module`

---

## app.component.html (Routing)

```html
<ul>
  <li><a href="/products">Products</a></li>
  <li><a href="/product-view">Product View</a></li>
  <li><a href="/product-edit">Product Edit</a></li>
</ul>

<!-- app-routing.module.ts -->
const routes: Routes = [
  { path: 'products', component: ProductComponent },
  { path: 'product-acc', component: ProductViewController },
  { path: 'product-adv', component: ProductEditComponent }
];
```

### Parameterized Routes

```typescript
{ path: 'product/:id', component: 'ComponentName' }
// product/10

{ path: 'product/:id/:id2', component: 'ComponentName' }
// product/10/20
```

**Component:**
```typescript
param = '';

constructor(private activatedRoute: ActivatedRoute) {
  this.activatedRoute.params.subscribe((data) => {
    this.param = data.id;
  });
}
```

### Query Parameters

- Search / required — for filtering/sorting/use
- GET HTTP Method calls: has no sensitive info → for this
- POST HTTP Method calls
- Implementation: same as above — must use for query params

### Redirecting

```typescript
{ path: '/', redirectTo: 'product-view', pathMatch: 'full' }
```

### Wildcard Route

```typescript
{ path: '**', component: AnyComponent }
```

### Child Routes

```typescript
{ path: 'products',
  children: [{ path: 'product-child', component: ComponentX }]
}
```

---

## Route Guards

- To secure certain paths
- `ng g guard <guardName>`
- `CanActivate`/`CanDeactivate`/`CanLoad`
- Checks only after authentication
- `{ path: 'admin', component: AdminHomeComponent }`
- `CanActivate` — `(Cleaning)` Guard

---

## Bootstrap via CDN

**(Not encouraged)**
1. Choose reliable CDN, copy URL for stylesheet and JS file
2. Add link in `index.html`
3. Our responsibility to maintain CDN sources and versions
4. Not encouraged in PROD environment

### Bootstrap via npm

```bash
npm i bootstrap jquery popper --save
# import style and script in angular.json
# restart angular app
```

**Mistakes:**
1. Path of script and style
2. Link of CDN outdated
3. Wrong version

---

## Forms — Two Types

- **Template Driven Forms**
- **Reactive Forms**

### Lazy Loaded

1. Create feature module: `ng g module <customer> --route <customer> --module appmodule`
2. Configure load children in app-routing:
   ```typescript
   { path: 'orders', loadChildren: () => import('./orders/orders.module').then(m => m.OrdersModule) }
   ```

---

## Template Driven Forms

1. Easy to implement
2. Simple and straightforward
3. Validations and form elements are in template
4. Import `FormsModule` in app module

### Setup
```typescript
// app.module.ts
import { FormsModule } from '@angular/forms';
@NgModule({ imports: [FormsModule] })
```

### Small Example

**app.component.ts:**
```typescript
export class AppComponent {
  user = {
    name: '',
    email: '',
    password: ''
  };

  onSubmit(form: any) {
    console.log('Form Submitted:', form.value);
  }
}
```

**app.component.html:**
```html
<form #loginForm="ngForm" (ngSubmit)="onSubmit(loginForm)">

  <input
    type="text"
    name="name"
    [(ngModel)]="user.name"
    required
    minlength="3"
    #name="ngModel"
    placeholder="Enter Name">
  <span *ngIf="name.invalid && name.touched">
    Name is required (min 3 characters)
  </span>

  <input
    type="email"
    name="email"
    [(ngModel)]="user.email"
    required
    email
    #email="ngModel"
    placeholder="Enter Email">
  <span *ngIf="email.invalid && email.touched">
    Enter a valid email
  </span>

  <input
    type="password"
    name="password"
    [(ngModel)]="user.password"
    required
    #password="ngModel"
    placeholder="Enter Password">
  <span *ngIf="password.invalid && password.touched">
    Password is required
  </span>

  <button type="submit" [disabled]="loginForm.invalid">Submit</button>

</form>
```

> ⚠️ **Key points:**
> - `#loginForm="ngForm"` — template reference to the whole form
> - `#name="ngModel"` — template reference to individual control
> - `required`, `email`, `minlength` — validation added directly in HTML
> - `.touched` — ensures error shows only after user interacts with field



1. Form elements / interactions / validations handled by component class
2. Use built-in `FormGroup` and `FormControl`
3. Better data binding
4. Best for error handling
5. `ReactiveFormsModule` import in app module
6. Allows to create complex requirements for Form
7. More logic in component class and less in HTML

### When to Use

**Template Driven Forms:**
1. Form — simple, straightforward
2. Basic field — Fixed
3. No complex validation

**Reactive Forms:**
1. Form complex
2. Dynamic components
3. Advanced validation
4. Dependent elements
5. Dynamic form generation based on user preference

---

## Reactive Forms — In Depth

### Setup
```typescript
// app.module.ts
import { ReactiveFormsModule } from '@angular/forms';
@NgModule({ imports: [ReactiveFormsModule] })
```

---

### FormBuilder

`FormBuilder` is a helper service that shortens the code needed to create `FormGroup` and `FormControl` instances.

```typescript
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

export class RegisterComponent {
  registerForm: FormGroup;

  constructor(private fb: FormBuilder) {
    this.registerForm = this.fb.group({
      name: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]],
      age: ['', [Validators.required, Validators.min(18)]]
    });
  }

  onSubmit() {
    console.log(this.registerForm.value);
  }
}
```

```html
<form [formGroup]="registerForm" (ngSubmit)="onSubmit()">
  <input formControlName="name" placeholder="Name">
  <span *ngIf="registerForm.get('name')?.invalid">Name is required</span>

  <input formControlName="email" placeholder="Email">
  <span *ngIf="registerForm.get('email')?.invalid">Invalid email</span>

  <button type="submit" [disabled]="registerForm.invalid">Submit</button>
</form>
```

---

### Built-in Validators

```typescript
Validators.required           // field must not be empty
Validators.email              // must be valid email format
Validators.min(18)            // minimum numeric value
Validators.max(100)           // maximum numeric value
Validators.minLength(3)       // minimum string length
Validators.maxLength(20)      // maximum string length
Validators.pattern('^[a-z]+$') // must match regex
```

---

### FormArray

`FormArray` is used when you have a **dynamic list of form controls** (e.g. adding multiple phone numbers).

```typescript
import { FormBuilder, FormArray, Validators } from '@angular/forms';

export class PhoneFormComponent {
  form = this.fb.group({
    phones: this.fb.array([
      this.fb.control('', Validators.required)  // one phone by default
    ])
  });

  constructor(private fb: FormBuilder) {}

  get phones() {
    return this.form.get('phones') as FormArray;
  }

  addPhone() {
    this.phones.push(this.fb.control('', Validators.required));
  }

  removePhone(index: number) {
    this.phones.removeAt(index);
  }
}
```

```html
<form [formGroup]="form">
  <div formArrayName="phones">
    <div *ngFor="let phone of phones.controls; let i = index">
      <input [formControlName]="i" placeholder="Phone {{ i + 1 }}">
      <button (click)="removePhone(i)">Remove</button>
    </div>
  </div>
  <button (click)="addPhone()">Add Phone</button>
</form>
```

---

### Custom Validators

```typescript
import { AbstractControl, ValidationErrors } from '@angular/forms';

// Custom validator function
export function forbiddenNameValidator(control: AbstractControl): ValidationErrors | null {
  const forbidden = control.value === 'admin';
  return forbidden ? { forbiddenName: { value: control.value } } : null;
}

// Usage in FormBuilder
this.form = this.fb.group({
  username: ['', [Validators.required, forbiddenNameValidator]]
});
```

```html
<span *ngIf="form.get('username')?.errors?.['forbiddenName']">
  "admin" is not allowed as a username
</span>
```

---

### Async Validators

Used when validation requires an **API call** (e.g. checking if username already exists).

```typescript
import { AbstractControl, AsyncValidatorFn, ValidationErrors } from '@angular/forms';
import { Observable, of } from 'rxjs';
import { map, catchError } from 'rxjs/operators';

export function uniqueUsernameValidator(userService: UserService): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    return userService.checkUsername(control.value).pipe(
      map(isTaken => isTaken ? { usernameTaken: true } : null),
      catchError(() => of(null))
    );
  };
}

// Usage in FormBuilder
this.form = this.fb.group({
  username: ['', Validators.required, uniqueUsernameValidator(this.userService)]
  //                   sync validators ^        async validator ^
});
```

```html
<span *ngIf="form.get('username')?.errors?.['usernameTaken']">
  Username already taken!
</span>
<span *ngIf="form.get('username')?.pending">
  Checking availability...
</span>
```

---

### valueChanges Observable

Subscribe to form or control value changes in real time.

```typescript
this.registerForm.get('email')?.valueChanges.subscribe(value => {
  console.log('Email changed to:', value);
});

// Listen to entire form
this.registerForm.valueChanges.subscribe(formValue => {
  console.log('Form changed:', formValue);
});
```

---

## Angular HTTP

### Observables

1. Sequence of data that is emitted over a period of time (asynchronously)
2. Observable part of RxJS library
3. Import Observable into our component
4. Data can be any type — string, events
5. Async operations, where it is most used:
   - HTTP
   - Routing
   - Event Handling
6. **Observer** — to track changes of observable continuously
7. Observer has many methods:
   - `next()`
   - `error()`
   - `complete()`
8. We use `subscribe` to process data sent by observable
9. Multiple subscribers for an observable allowed
10. We can also unsubscribe from an observable

### Observable Example:

```typescript
orderStatus: any;
data: Observable<any>;

constructor() {}

ngOnInit(): void {
  data = new Observable((observer) => {
    setTimeout(() => {
      observer.next('In Processing');
    }, 2000);

    setTimeout(() => {
      observer.next('Processing');
    }, 5000);

    setTimeout(() => {
      observer.next('Computed');
    }, 8000);
  });

  this.data.subscribe((val) => {
    this.orderStatus = val;
  });

  this.data.subscribe((val2) => {
    console.log('Second Subscription');
  });
}
```

---

## Services

1. Services are used for reusing the common shared functionality
2. Services are singleton — need to inject
3. Need not be included in modules
4. Any number of services
5. Used for sharing data
6. **Dependency Injection**
7. Injected at runtime into components
8. Code becomes highly efficient and easy to maintain
9. The code becomes much easier to maintain, easier to test
10. Values and methods and lots in Services
11. `ng g service <service-name>`
12. `@Injectable`
13. `providedIn: 'root'` — available to all

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class ContactService {
  constructor() {}
  
  getContacts() {
    const contactList = [
      { contactId: 1, contactName: 'ARC' },
      { contactId: 2, contactName: 'Tutorials' },
      { contactId: 3, contactName: 'Youtube' },
      { contactId: 4, contactName: 'Angular' }
    ];
    return contactList;
  }
}
```

### contacts.component.ts

```typescript
imports { ContactsService } from '../contacts.service';

@Component({ ... })

export class ContactsComponent implements OnInit {
  constructor(private contactsService: ContactsService) {}

  contactList = [];

  ngOnInit(): void {
    this.contactList = this.contactsService.getContacts();
  }
}
```

### contacts.component.html

```html
<ul>
  <li *ngFor="let contact of contactList">
    {{ contact.contactName }}
  </li>
</ul>
```

---

## HttpClient

1. `HttpClient` is used for performing HTTP request and responses
2. `HttpClient` service is available in `@angular/common/http`
3. `HttpClientModule` needs to be imported into the module
4. `HttpClient` gives functionality like headers, interceptors, etc.

### HttpClient Methods

| Method | Description |
|--------|-------------|
| `get()` | Fetch data |
| `post()` | Send data |
| `put()` | Update data |
| `delete()` | Delete data |
| `head()` | — |
| `jsonp()` | — |
| `options()` | — |
| `patch()` | Partial update |

### Calling from Template

```typescript
// 14. Import the service into any number of components
// 15. We can call any methods inside the service

callingFromTemplate() {
  console.log('Calling from Template Directory');
}
```

**Inside HTML:**
```html
<button (click)="ContactsService.callingFromTemplate()">
  > Calling Directly
</button>
```

---

---

## Benefits of HttpClient

1. Includes Observable APIs
2. Can have the HTTP Headers in options
3. Includes the highly testability features
4. Includes typed request
5. Includes response objects
6. Includes request and response interceptions
7. Includes error handling

### CRUD Operations

- **Create** — POST method call
- **Read** — GET method call → update entire resource
- **Update** — PUT method call → params() updated part of resource
- **Delete** — DELETE method call

---

## HttpClient GET

1. Import `HttpClientModule` in our App Module
2. Import `HttpClient` in our service or component whenever we are making HTTP request
   - Good practice to have all HTTP calls in services
   - Becomes reusable and easy to maintain
   - Easy to share between various components

3. Inject the `HttpClient` in the constructor method of the class
4. Implement the GET method call
5. Import the Services into the required calling component class
6. Call the method to make the HTTP request

### Component Class

```typescript
export class ContactsComponent implements OnInit {
  constructor(private contactsService: ContactsService) {}

  contactList: any;

  ngOnInit(): void {
    this.contactsService.getContacts().subscribe((data) => {
      this.contactList = data;
    });
  }
}
```

### Service Class (with HttpClient)

```typescript
export class ContactsService {
  constructor(private httpClient: HttpClient) {}

  getContacts() {
    const httpHeaders = new HttpHeaders();
    httpHeaders.append('content-type', 'application/json');
    return this.httpClient.get('http://localhost:3000/contacts',
      { headers: httpHeaders });
  }
}
```

---

## HTTP POST

```typescript
// contact.service.ts
createContact() {
  return this.httpClient.post('http://localhost:3000/contacts', createBody);
}

// contact component
addNewContact() {
  const newFormData = { id: 6, firstname: 'Raj', lastname: 'Kumar' };
  this.contactService.createContact(newFormData).subscribe((data) => {
  });
}

// inside html
<button (click)="addNewContact()">
```

---

## HTTP PUT

```typescript
// contact.service.ts
updateContact(contactId, updateBody) {
  const endpointURL = 'http://localhost:3000/contacts/' + contactId;
  return this.httpClient.put(endpointURL, updateBody);
}

// contacts.component
updateContact(contactId) {
  const newFormData = {
    id: contactId,
    firstname: 'Steve',
    lastname: 'Johnson'
  };
  this.contactService.updateContact(contactId, newFormData);
}

// inside html
<button (click)="updateContact(3)">
```

---

## HTTP DELETE

```typescript
// contact.service.ts
deleteContact(contactId) {
  const deleteEndpoint = 'http://localhost:3000/contacts/' + contactId;
  return this.httpClient.delete(deleteEndpoint);
}

// contacts.component
deleteContact(contactId) {
  this.contactsService.deleteContact(contactId).subscribe((data) => { });
}
```

---

## HTTP Headers

- Contains common data about the request
- Immutable
- e.g. `Authorization`, `content-type`
- Can pass through any HTTP method
- 7 methods: `Append`, `has`, `get`, `keys`, `delete`, ...
- `HttpHeaders` — can send any number of headers
- No duplicate headers to be sent
- Can create custom headers

```typescript
let httpHeaders = new HttpHeaders({
  'content-type': 'application/json',
  'Authorization': 'ARCTutorials402',
  'timeout': '3000'
});

// Custom headers
httpHeaders = httpHeaders.set('arc-tutorials-id', '110');
let time = httpHeaders.get('timeout');
```

```typescript
if (time == '3000') {
  httpHeaders = httpHeaders.set('Authorization', '...');
}

return this.httpClient.get('http://localhost:3000/contacts',
  { headers: httpHeaders });
```

---

## HTTP Params

- Can be seen in URL
- Using `HttpParams`, we can define it using class
- Immutable
- 8 methods: `#+` to string
  - ① from object
  - ② from string

```typescript
// contact.service.ts
getContactById() {
  const httpParams = new HttpParams({
    fromObject: { query: 'mark' }
  });

  this.httpClient.get('http://localhost:3000/contacts',
    { params: httpParams });
}

// http://localhost:3000/contacts? query=mark
```

---

## Interceptors

1. Used to process request in the middle
2. Can have multiple interceptors
   - `ng generate interceptor <name-logger>`

**Use cases:**
- Log that data
- Show loader / icon
- Default format

```typescript
// app.module providers:
{ provide: HTTP_INTERCEPTORS, useClass: LoggerInterceptor, multi: true }
```

**Interceptor code:**
```typescript
console.log('Request made to ' + request.url);
let httpAuth = request.headers.get('Authorization');
httpAuth = 'addedValue' + httpAuth;
request.headers.append('Annotation', httpAuth);
console.log(httpAuth);
return next.handle(request);
```

---

## Angular Component Lifecycle

**8 lifecycle hooks** (most used ↑)

1. `ngOnChanges()`
2. `ngOnInit()`
3. `ngDoCheck()`
4. `ngAfterContentInit()`
5. `ngAfterContentChecked()`
6. `ngAfterViewInit()`
7. `ngAfterViewChecked()`
8. `ngOnDestroy()`

---

### ① ngOnChanges()
- Whenever input changes, is called
- Called first before `ngOnInit`
- Used in every component where input is there

### ② ngOnInit()
- Used to initialize component
- Called after input values are set; once component is initialized
- Added to every component by default
- Called only once

### ③ ngDoCheck()
- Called after during all change detections
- It is a run through the view by Angular to detect changes/update

### ④ ngAfterContentInit()
- Called only once first
- Called after first run through & initialize component

### ⑤ ngAfterContentChecked()
- Called after every `ngDoCheck()`
- On the **first run**, called after `ngAfterContentInit()`
- On **subsequent runs**, called after every `ngDoCheck()` *(correction: it does NOT wait for `ngAfterContentInit` after the first run)*

### ⑥ ngAfterViewInit()
- Called after Angular initializes component and child components
- Called only once after first `ngAfterContentChecked` is initialized

### ⑦ ngAfterViewChecked()
- Comes after every component initialized and checked (component and child components)
- First call after `ngAfterViewInit()`
- Called after every `ContentChecked()` calls

### ⑧ ngOnDestroy()
- Used to clean up code after component

---

## @ContentChild & @ViewChild

- **@ContentChild** — property decorator — used to access a child element/component that is projected into the component via **content projection** (`<ng-content>`). It queries content inserted from *outside* the component (i.e., between its tags in the parent template).
- **@ViewChild** — property decorator — used to get a reference to a child component, directive, or DOM element *within the component's own template*. Allows calling public APIs and methods on it.

> ⚠️ **Correction note:** `@ContentChild` is a *Property* decorator, not a *Method* decorator as it may have appeared in the original notes.

---

## ng-content (Content Projection)

`ng-content` allows you to **pass HTML content from a parent component into a child component's template** — similar to "slots" in other frameworks.

### Single-slot Projection

The child component has one `<ng-content>` placeholder — whatever is placed between the child's tags in the parent gets projected there.

**child.component.html:**
```html
<div class="card">
  <h2>Card Title</h2>
  <ng-content></ng-content>   <!-- projected content goes here -->
</div>
```

**parent.component.html:**
```html
<app-card>
  <p>This paragraph is projected into the card!</p>
</app-card>
```

**Output:**
```html
<div class="card">
  <h2>Card Title</h2>
  <p>This paragraph is projected into the card!</p>
</div>
```

---

### Multi-slot Projection

Use the `select` attribute to project content into **specific named slots**.

**child.component.html:**
```html
<div class="card">
  <div class="header">
    <ng-content select="[card-header]"></ng-content>
  </div>
  <div class="body">
    <ng-content select="[card-body]"></ng-content>
  </div>
  <div class="footer">
    <ng-content select="[card-footer]"></ng-content>
  </div>
</div>
```

**parent.component.html:**
```html
<app-card>
  <h2 card-header>My Title</h2>
  <p card-body>My body content goes here.</p>
  <button card-footer>Click Me</button>
</app-card>
```

---

### Conditional Projection

Use `*ngIf` with `<ng-template>` and `@ContentChild` to conditionally project content.

**child.component.ts:**
```typescript
import { Component, ContentChild, TemplateRef } from '@angular/core';

@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <ng-content select="[card-header]"></ng-content>
      <ng-container *ngIf="showBody">
        <ng-content select="[card-body]"></ng-content>
      </ng-container>
    </div>
  `
})
export class CardComponent {
  showBody = true;
}
```

---

## ng-template

`ng-template` is an Angular element used to **define a template that is not rendered by default**. It is used with structural directives or rendered programmatically.

- Not rendered in the DOM unless explicitly used
- Used with `*ngIf else`, `*ngFor`, `ngSwitch`, and `ViewContainerRef`

### With ngIf else
```html
<div *ngIf="isLoggedIn; else loginTemplate">
  Welcome back, User!
</div>

<ng-template #loginTemplate>
  <p>Please log in to continue.</p>
</ng-template>
```

### With ngSwitch
```html
<ng-container [ngSwitch]="role">
  <ng-template *ngSwitchCase="'admin'">
    <p>Admin Dashboard</p>
  </ng-template>
  <ng-template *ngSwitchCase="'user'">
    <p>User Dashboard</p>
  </ng-template>
  <ng-template ngSwitchDefault>
    <p>Guest View</p>
  </ng-template>
</ng-container>
```

### Rendering ng-template Programmatically
```typescript
import { ViewChild, TemplateRef, ViewContainerRef } from '@angular/core';

@ViewChild('myTemplate') myTemplate: TemplateRef<any>;

constructor(private vcr: ViewContainerRef) {}

showTemplate() {
  this.vcr.createEmbeddedView(this.myTemplate);
}
```

```html
<ng-template #myTemplate>
  <p>This is rendered programmatically!</p>
</ng-template>
<button (click)="showTemplate()">Show</button>
```

---

## ng-container

`ng-container` is a **grouping element that doesn't add any extra DOM element**. It is useful when you need to apply a structural directive but don't want to add an extra HTML wrapper.

- Renders no element in the DOM
- Can hold structural directives
- Can hold multiple directives together

### Without ng-container (extra div added)
```html
<div *ngIf="isLoggedIn">   <!-- unwanted extra div in DOM -->
  <p>Welcome!</p>
</div>
```

### With ng-container (no extra element in DOM)
```html
<ng-container *ngIf="isLoggedIn">
  <p>Welcome!</p>
</ng-container>
```

### Applying Multiple Directives
```html
<!-- Can't use *ngIf and *ngFor on same element -->
<!-- WRONG -->
<div *ngFor="let item of items" *ngIf="item.active">{{ item.name }}</div>

<!-- CORRECT — use ng-container to separate -->
<ng-container *ngFor="let item of items">
  <div *ngIf="item.active">{{ item.name }}</div>
</ng-container>
```

### With ngSwitch
```html
<ng-container [ngSwitch]="status">
  <p *ngSwitchCase="'active'">Active</p>
  <p *ngSwitchCase="'inactive'">Inactive</p>
  <p *ngSwitchDefault>Unknown</p>
</ng-container>
```

> ⚠️ **Key Difference:**
> | | Rendered in DOM? | Can hold directives? |
> |---|---|---|
> | `ng-content` | Yes (projects content) | No |
> | `ng-template` | No (unless explicitly used) | Yes |
> | `ng-container` | No (invisible wrapper) | Yes |

---



Pipes are used to **transform data** in templates before displaying it. They don't change the original data — only the displayed output.

### Syntax
```html
{{ value | pipeName }}
{{ value | pipeName : argument }}
```

---

### Built-in Pipes

```html
<!-- DatePipe -->
{{ today | date }}                    <!-- May 24, 2026 -->
{{ today | date: 'dd/MM/yyyy' }}      <!-- 24/05/2026 -->

<!-- UpperCase / LowerCase -->
{{ 'hello' | uppercase }}             <!-- HELLO -->
{{ 'HELLO' | lowercase }}             <!-- hello -->

<!-- CurrencyPipe -->
{{ 1234.5 | currency }}               <!-- $1,234.50 -->
{{ 1234.5 | currency: 'INR' }}        <!-- ₹1,234.50 -->

<!-- DecimalPipe -->
{{ 3.14159 | number: '1.2-2' }}       <!-- 3.14 -->

<!-- SlicePipe -->
{{ [1,2,3,4,5] | slice: 1:3 }}        <!-- [2, 3] -->

<!-- JsonPipe (useful for debugging) -->
{{ user | json }}                     <!-- { "name": "John", "age": 25 } -->

<!-- AsyncPipe (auto subscribes/unsubscribes to Observable) -->
{{ data$ | async }}
```

---

### Custom Pipe — Small Example

```bash
ng generate pipe multiply
```

**multiply.pipe.ts**
```typescript
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({ name: 'multiply' })
export class MultiplyPipe implements PipeTransform {
  transform(value: number, factor: number): number {
    return value * factor;
  }
}
```

**template:**
```html
{{ 5 | multiply: 3 }}    <!-- Output: 15 -->
{{ 10 | multiply: 2 }}   <!-- Output: 20 -->
```

---

### Chaining Pipes
```html
{{ 'hello world' | uppercase | slice: 0:5 }}   <!-- HELLO -->
```

---

### Pure vs Impure Pipes

| Pure | Impure |
|------|--------|
| Default — called only when input value changes | Called on every change detection cycle |
| Better performance | Can be slow — use carefully |
| `pure: true` (default) | `pure: false` |

```typescript
@Pipe({ name: 'myPipe', pure: false })  // impure pipe
```

---

## TrackBy in ngFor

By default, when a list updates, Angular **re-renders the entire list** in the DOM — even if only one item changed. `trackBy` tells Angular how to identify each item so it only re-renders what actually changed.

### Without TrackBy (inefficient)
```html
<div *ngFor="let user of users">
  {{ user.name }}
</div>
```
Every time `users` array changes, Angular destroys and recreates all DOM elements.

### With TrackBy (efficient)

**component.ts:**
```typescript
users = [
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' },
  { id: 3, name: 'Bob' }
];

trackByUserId(index: number, user: any): number {
  return user.id;   // Angular tracks by id — only changed items re-render
}
```

**component.html:**
```html
<div *ngFor="let user of users; trackBy: trackByUserId">
  {{ user.name }}
</div>
```

> ⚠️ **Best Practice:** Always use `trackBy` when working with large lists or lists that update frequently (e.g. real-time data, pagination).

---

## Testing in Angular

Angular uses **Jasmine** as the testing framework and **Karma** as the test runner. `TestBed` is Angular's primary API for writing unit tests.

### Key Concepts

| Term | Description |
|------|-------------|
| **Jasmine** | Testing framework — provides `describe`, `it`, `expect` |
| **Karma** | Test runner — runs tests in the browser |
| **TestBed** | Angular testing utility — sets up component/module environment |
| **spec.ts** | Test file — auto-generated with every component/service/pipe |

```bash
ng test    # runs all tests using Karma
```

---

### Testing a Component

**counter.component.ts:**
```typescript
@Component({
  selector: 'app-counter',
  template: `
    <p>{{ count }}</p>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  count = 0;

  increment() {
    this.count++;
  }
}
```

**counter.component.spec.ts:**
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { CounterComponent } from './counter.component';

describe('CounterComponent', () => {
  let component: CounterComponent;
  let fixture: ComponentFixture<CounterComponent>;

  beforeEach(() => {
    TestBed.configureTestingModule({
      declarations: [CounterComponent]
    });
    fixture = TestBed.createComponent(CounterComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();  // triggers ngOnInit
  });

  it('should create the component', () => {
    expect(component).toBeTruthy();
  });

  it('should start with count 0', () => {
    expect(component.count).toBe(0);
  });

  it('should increment count on calling increment()', () => {
    component.increment();
    expect(component.count).toBe(1);
  });

  it('should render incremented count in DOM', () => {
    component.increment();
    fixture.detectChanges();   // update the DOM
    const compiled = fixture.nativeElement as HTMLElement;
    expect(compiled.querySelector('p')?.textContent).toContain('1');
  });
});
```

---

### Testing a Service

**calc.service.ts:**
```typescript
@Injectable({ providedIn: 'root' })
export class CalcService {
  add(a: number, b: number): number {
    return a + b;
  }
}
```

**calc.service.spec.ts:**
```typescript
import { TestBed } from '@angular/core/testing';
import { CalcService } from './calc.service';

describe('CalcService', () => {
  let service: CalcService;

  beforeEach(() => {
    TestBed.configureTestingModule({});
    service = TestBed.inject(CalcService);
  });

  it('should be created', () => {
    expect(service).toBeTruthy();
  });

  it('should add two numbers correctly', () => {
    expect(service.add(2, 3)).toBe(5);
  });

  it('should return 0 when adding 0 and 0', () => {
    expect(service.add(0, 0)).toBe(0);
  });
});
```

---

### Testing a Pipe

**multiply.pipe.spec.ts:**
```typescript
import { MultiplyPipe } from './multiply.pipe';

describe('MultiplyPipe', () => {
  let pipe: MultiplyPipe;

  beforeEach(() => {
    pipe = new MultiplyPipe();  // pipes can be tested without TestBed
  });

  it('should create the pipe', () => {
    expect(pipe).toBeTruthy();
  });

  it('should multiply 5 by 3 to return 15', () => {
    expect(pipe.transform(5, 3)).toBe(15);
  });

  it('should return 0 when multiplied by 0', () => {
    expect(pipe.transform(5, 0)).toBe(0);
  });
});
```

---

### Jasmine Matchers Reference

```typescript
expect(value).toBe(5)             // strict equality ===
expect(value).toEqual({a: 1})     // deep equality (objects/arrays)
expect(value).toBeTruthy()        // truthy value
expect(value).toBeFalsy()         // falsy value
expect(value).toBeNull()          // null
expect(value).toContain('text')   // string/array contains
expect(value).toBeGreaterThan(3)  // greater than
expect(fn).toThrow()              // function throws an error
expect(spy).toHaveBeenCalled()    // spy was called
```

---

*End of Angular 9 Notes*

