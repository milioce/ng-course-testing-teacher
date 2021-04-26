# Testing de integración

## Crear componente con Angular CLI

```bash
$ ng g c intermedio/users/user --flat -t -s
```

- Crea el componente `user.component.ts`
- Crea el fichero de pruebas `user.component.spec.ts`

<br>

`app/intermedio/users/user.component.ts`
``` ts
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-user',
  template: `
    <p>user works!</p>
  `,
  styles: []
})
export class UserComponent implements OnInit {

  constructor() { }

  ngOnInit(): void {
  }

}
```
<br>

`app/intermedio/users/user.component.spec.ts`
``` ts
import { ComponentFixture, TestBed } from '@angular/core/testing';

import { UserComponent } from './user.component';

describe('UserComponent', () => {
  let component: UserComponent;
  let fixture: ComponentFixture<UserComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ UserComponent ]
    })
    .compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(UserComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```
<br>

## Herramienta de pruebas `TestBed`

Revisamos el fichero de pruebas

<br>

> ### Paso 1. Importar TestBed

```ts
import { ComponentFixture, TestBed } from '@angular/core/testing';

```

- `TestBed` es la herramienta de testing más importante de Angular.
  <br> Crea un *test module* que emula un modlo en Angular `@NgModule`
- `TestBed` devuelve un fixture que nos sirve para acceder al HTML.
El fixture es similar a jQuery, nos facilita acceder al Dom.

<br>

> ### Paso 2. Configurar el Módulo

```ts
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ UserComponent ]
    })
    .compileComponents();
  });
```

- `TestBed.conconfigureTestingModule` configura un módulo con las mismas propiedades que un `@NgModule`
- Compila el componente: carga el HTML, active el ciclo de vida, activa el ciclo detección de cambios.

<br>

> ### Paso 3. Obtenemos una instancia del fixture y del componente

``` ts
  beforeEach(() => {
    fixture = TestBed.createComponent(UserComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

```
- `fixture` nos permite acceder al DOM del componente.

<br>

> ### Paso 4. Añade una prueba de que el componente se ha creado.

```ts
  it('should create', () => {
    expect(component).toBeTruthy();
  });
```
<br>

> ### Paso 5. Ejecutamos el test

Vemos que el componente `UserComponent` se crea correctamente

<br>

## Uso de servicios
---

<br>

> Inyectamos el servicio `UsersService` en el componente `UserComponent`

``` ts
import { Component, OnInit } from '@angular/core';
import { UsersService } from './users.service';

...
export class UserComponent implements OnInit {
  users: any[] = [];

  constructor(private _service: UsersService) { }

  ngOnInit(): void {
  }

  getMedico() {
    this._service.getUsers().subscribe(users => this.users = users);
  }

}
```

- **`Error`**: "No provider for UsersService!"

<br>

> Añadimos el `provider` para el servicio `UserService`

```ts
import { UsersService } from './users.service';
...

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ UserComponent ],
+     providers: [UsersService],
    })
    .compileComponents();
  });
```

- **`Error`**: "No provider for HttpClient!"

<br>

> Importar el Módulo `HttpClientModule`

Necesitamos importar el módulo `HttpClientModule` para que esté disponible el servicio `HttpClient`

```ts
  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ UserComponent ],
      providers: [UsersService],
      imports: [HttpClientModule]
    })
    .compileComponents();
  });
```

Ahora el test ya vuelve a funcionar.

<br>
