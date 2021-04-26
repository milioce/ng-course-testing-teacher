# Testing intermedio

## Tipos de pruebas
---
- Event emitter
- Formularios
- Validaciones
- Espías
- Simular retornos de funciones
- Simular llamadas de funciones
- Code coverage

<br>

## Event Emitter
---

Añadimos un eventEmitter a la clase `Cuenta`
- Copiamos `app/basico/cuenta.ts`a `app/intermedio/cuenta.ts`

<br>

> Paso 1. Añado la propiedad cambia de tipo `EventEmitter`

`app/intermedio/cuenta.ts`
```ts diff
+ import { EventEmitter } from "@angular/core";

export class Cuenta {
  banco = 'Banco Testing';
  titular: string;
  saldo: number;
  servicios = ['hipoteca', 'tarjeta', 'seguro', 'bizum'];
+ cambia = new EventEmitter<number>();

  constructor(titular: string, saldo: number) {
    this.titular = titular;
    this.saldo = saldo;
  }

  nombre() {
    return `Cuenta de ${this.titular}`;
  }

  ingresar(importe: number) {
    this.saldo += importe;
+   this.cambia.emit(this.saldo);

    return this.saldo;
  }

  retirar(importe: number) {
    if (this.saldo >= importe) {
      this.saldo -= importe;
    }
+   this.cambia.emit(this.saldo);

    return this.saldo;
  }

  haySaldo() {
    return this.saldo > 0;
  }
}
```
<br>

> Paso 2. Creo el fichero `cuenta.spec.ts`

<br>

- Podemos subscribirnos directamente al Output y ejecutar `retirar()` para que emita un valor.
- Sin el `done` como argumento. Si incluimos el `done` como argumento, debemos ejecutarlo o nunca se completará la prueba

`app/intermedio/cuenta.spec.ts`
``` ts
import { Cuenta } from "./cuenta";

describe('Prueba de Cuenta - intermedio', () => {

  let cuenta: Cuenta;

  beforeEach(() => cuenta = new Cuenta('Emilio', 100));

  it('Se emite el nuevo saldo al retirar dinero', () => {
    let saldo = 100;

    cuenta.cambia.subscribe(data => saldo = data);
    cuenta.retirar(50);

    expect(saldo).toBe(50);
  });

});
```
<br>

> Paso 3. Usar la función `done`.

- Podemos usar la funcion `done()` como argumento en un `beforeEach` o en un `it`
- La prueba no se inicia hasta que en el `beforeEach` se invoque el `done`
- La prueba no se completa hasta se invoca el `done`

`app/intermedio/cuenta.spec.ts`
``` ts
...

  it('Se emite el nuevo saldo al retirar dinero', (done) => {
    let saldo = 100;

    cuenta.cambia.subscribe(data => {
      saldo = data;
      expect(saldo).toBe(50);
      done();
    });

    cuenta.retirar(50);
  });
```
<br>

## Formularios y validaciones
---
<br>

> Paso 1. Creamos un formulario

<br>

`app/intermedio/formulario.ts`
```ts
import { FormBuilder, FormGroup, Validators } from "@angular/forms";

export class LoginForm {
  form: FormGroup;

  constructor(fb: FormBuilder) {
    this.form = fb.group({
      email: ['', [Validators.required, Validators.email]],
      password: ['', Validators.required]
    });
  }

}
```
<br>

> Paso 2. Creamos el fichero de pruebas `app/intermedio/formulario.spec.ts`

<br>

```ts
import { FormBuilder } from "@angular/forms";
import { LoginForm } from "./formulario";

describe('LoginForm', () => {
  let loginForm: LoginForm;

  beforeEach(() => {
    loginForm = new LoginForm(new FormBuilder());
  });

  it('', () => {
  });

});
```
<br>

> Paso 3. Probamos que el formulario tiene dos campos

```ts
  ...

  it('Se debe crear un formulario con dos campos', () => {
    expect(loginForm.form.contains('email')).toBeTrue();
    expect(loginForm.form.contains('password')).toBeTrue();
  });
```
<br>

> Paso 4. Probamos que el email es obligatorio

```ts
  ...

  it('El email debe ser obligatorio', () => {
    const control = loginForm.form.get('email');
    control.setValue('');

    expect(control.valid).toBeFalse();
    expect(control.hasError('required')).toBeTrue();
  });
```
<br>

> Paso 5. Probamos que el email es un valor válido

```ts
  ...

  it('El email debe ser correo válido', () => {
    const control = loginForm.form.get('email');

    control.setValue('Emilio');
    expect(control.valid).toBeFalse();
    expect(control.hasError('email')).toBeTrue();

    control.setValue('emilio@gmail.com');
    expect(control.valid).toBeTrue();
  });
```
<br>

## Espías
---

> ### Paso 1. Crear fichero en `app/users`:

<br>

- Servicio `UsersService`
- El servicio tiene una dependencia de `HttpClient`
- Las url del api dan igual porque las vamos a simular

`app/users/users.service.ts`
```ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable()
export class UsersService {

  constructor( public http: HttpClient ) { }

  getUsers() {
    return this.http.get<any[]>('...');
  }

  addUser( user: any ) {
    return this.http.post('...', user );
  }

  deleteUser( id: string ) {
    return this.http.delete('...' );
  }

}
```

`app/users/users.component.ts`
```ts
import { Component, OnInit } from '@angular/core';
import { UsersService } from './users.service';

@Component({
  selector: 'app-users',
  template: `
    <p>
      Users works!
    </p>
  `,
  styles: []
})
export class UsersComponent implements OnInit {

  public users: any[] = [];
  public errorMessage: string;

  constructor( public _usersService: UsersService ) { }

  ngOnInit() {
    this._usersService.getUsers()
          .subscribe( users => this.users = users );
  }

  addUser() {
    const user = { nombre: 'Emilio' };

    this._usersService.addUser(user)
          .subscribe(
            newUser => this.users.push(newUser),
            err => this.errorMessage = err
          );
  }

  deleteUser(id: string) {
    const confirmar = confirm('Estas seguro que desea borrar este usuario');

    if ( confirmar ) {
      this._usersService.deleteUser( id );
    }

  }

}
```
<br>

> ### Paso 2. Crear el fichero de pruebas para el componente

<br>

- En el `describe`
  - Definimos una variable para el componente
  - Creamos una instancia del servicio MedicosService.
  <br>necesita una dependencia (http) le paso null porque no lo vamos a usar

- En el `beforeEach`
  - Creamos una instancia del Componente y le pasamos el servicio

- En el `it`
  - Tenemos que llamar a onInit manualmente, porque así no aplica el ciclo de vida de Angular


`app/users/users.component.spec.ts`
```ts
import { UsersComponent } from "./users.component";
import { UsersService } from "./users.service";

describe('UsersComponent', () => {

  let component: UsersComponent;
  const service = new UsersService(null);

  beforeEach(() => {
    component = new UsersComponent(service);

  });

  it('Debe cargar los usuarios', () => {

    component.ngOnInit();

    expect( component.users.length ).toBeGreaterThan(0);
  });

});
```

- **`Error`** El servicio que hay en el `onInit` no va a devolver datos
- Necesitamos simular los datos usando un espía

<br>

> ### Paso 3. Crear un espía para devolver datos

<br>

`spyOn(service, 'getUsers' )`: Espía el objeto `service` y reemplaza el método `getUsers` por el espía

`spyOn().and.callFake(fn)`: invoca una implementación falsa

`spyOn().and.returnValue(value)`: devuelve un valor


`app/users/users.component.spec.ts`
```ts
import { of } from "rxjs";
...

  it('Debe cargar los usuarios', () => {
    const users = ['Emilio', 'Juan', 'Ana'];

    spyOn(service, 'getUsers' ).and.callFake(() => {
      return of(users);
    });

    component.ngOnInit();

    expect( component.users.length ).toBeGreaterThan(0);
  });
```
<br>

> ### Paso 4. Probamos que `addUser` invoca al servicio del API

<br>

Probamos que al ejecutar el `UsersComponent.addUsers()` se este llamando al método del servicio.

- Solo vamos a probar que se invoca el servicio del API
- Creamos un espia que no devuelve nada, no lo necesitamos
- Verificamos que el espía se ha llamado

`app/users/users.component.spec.ts`
```ts
import { EMPTY, of } from "rxjs";
...

  it('addUser invoca el servicio del API ', () => {

    const spy = spyOn(service, 'addUser').and.callFake(() => {
      return EMPTY;
    });

    component.addUser();

    expect(spy).toHaveBeenCalled();
  });
```
<br>

> ### Paso 5. Probamos que `addUser` añade un usuario al array

<br>

`app/users/users.component.spec.ts`
```ts
...

 it('addUser añade un usuario al array ', () => {

    const user = { id: 1, nombre: 'Carlos' };
    const spy = spyOn(service, 'addUser').and.returnValue(
      of(user)
    );

    component.addUser();

    expect(component.users.indexOf(user)).toBeGreaterThanOrEqual(0);
  });
```
<br>

> ### Paso 6. Probamos un error en el observable

<br>

`app/users/users.component.spec.ts`
```ts
import { EMPTY, of, throwError } from "rxjs";
...

  it('Si addUser falla, en errorMessage tenemos el error del servicio ', () => {

    const error = 'No se pudo agregar el medico';
    const spy = spyOn(service, 'addUser').and.returnValue(
      throwError(error)
    );

    component.addUser();

    expect(component.errorMessage).toBe(error);
  });
```
<br>

> ### Paso 7. Probamos el método `deleteUser`

<br>

- Vamos a probar solo que `deleteUser` invoca el metodo del servicio.
- Tenemos que espiar tambien `window.confirm` para que no pare la ejecución

`app/users/users.component.spec.ts`
```ts
...

  it('deleteUser debe llamar al invocar el método delete del servicio', () => {

    spyOn(window, 'confirm').and.returnValue(true);

    const spy = spyOn(service, 'deleteUser').and.returnValue(EMPTY);

    component.deleteUser('1');
    expect(spy).toHaveBeenCalledWith('1');
  });
```
<br>

## Code coverage
---

``` bash
$ npm run test -- --code-coverage
```
<br>

Crea una carpeta `coverage/prpjectName` que contiene un `index.html`
