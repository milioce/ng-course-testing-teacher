# Tests básicos

## 1. Creamos un proyecto con Angular CLI
---


```bash
$ ng new pruebas
```

Al crear un nuevo proyecto con Angular CLI ya nos incluye varias herramientas de testing configuradas-

```json
    "jasmine-core": "~3.5.0",
    "jasmine-spec-reporter": "~4.2.1",
    "karma": "~5.0.0",
    "karma-chrome-launcher": "~3.1.0",
    "karma-coverage-istanbul-reporter": "~2.1.0",
    "karma-jasmine": "~3.0.1",
    "karma-jasmine-html-reporter": "^1.4.2",
    "protractor": "~7.0.0",
```

- Jasmine: es el framework de testing.
- Karma: Gestor de tareas y configuraciones
- Protractor para Test e2e


Se ha creado un componente `app.component.ts` y un fichero con sus tests `app.component.spec.ts`

Todos los archivos *.spec.ts serán evaluados por las pruebas.

Ejecutamos los tests con `ng test`
(Levantamos el servidor de pruebas)

<br>

## 2. Pruebas unitarias básicas
---
- Probar strings, Números, Booleanos, Arrays, Clases
- Cobertura de pruebas: lineas de código cubiertas por nuestras pruebas

Todas las pruebas tendran mínimo un describe() y un it()
- describe() => Suite de pruebas
- it() => Caso de prueba

<br>

> Paso 1. Creamos un fichero `app/basicos/cuenta.ts`

***`app/basicos/cuenta.ts`***
```ts
export class Cuenta {
  banco = 'Banco Testing';
  titular: string;
  saldo: number;
  servicios = ['hipoteca', 'tarjeta', 'seguro', 'bizum'];

  constructor(titular: string, saldo: number) {
    this.titular = titular;
    this.saldo = saldo;
  }

  nombre() {
    return `Cuenta de ${this.titular}`;
  }

  ingresar(importe: number) {
    this.saldo += importe;

    return this.saldo;
  }

  retirar(importe: number) {
    if (this.saldo >= importe) {
      this.saldo -= importe;
    }

    return this.saldo;
  }

  haySaldo() {
    return this.saldo > 0;
  }
}
```
<br>

> Paso 2. Creamos el fichero de pruebas `app/basicos/cuenta.spec.ts`

Para crear una prueba necesitamos
- un `describe()` para definir el juego de pruebas
- un `it()` para añadir un caso de prueba

```ts
describe('Pruebas de Cuenta', () => {

  it('Prueba 1', () => {
    expect(true).toBe(true);
  });

});
```
<br>

> Paso 3. Creamos una primera prueba con `strings`

```ts
import { Cuenta } from "./cuenta";

describe('Pruebas de Cuenta', () => {

  it('Debe devolver el nombre del banco', () => {
    const cuenta = new Cuenta('Emilio', 100);

    expect(cuenta.banco).toBe('Banco Testing');
  });

});
```
- Ejecutamos los tests con `npm run test`
- Muestra los resultados y se queda eschuchando cambios (watch)

<br>

> Paso 4. Añadimos otra prueba con `string`

```ts
import { Cuenta } from "./cuenta";

describe('Pruebas de Cuenta', () => {

  it('Debe devolver el nombre del banco', () => {
    const cuenta = new Cuenta('Emilio', 100);

    expect(cuenta.banco).toBe('Banco Testing');
  });

  it('El nombre de la cuenta debe contener el nombre del titular', () => {
    const titular = 'Emilio';
    const cuenta = new Cuenta(titular, 100);

    expect(cuenta.nombre()).toContain(titular);
  });

});
```
- Se vuelven a ejecutar los tests
- Cambiar el nombre del banco en `cuenta.ts` para ver que falla el test
<br>

<br>

> Paso 5. Añadimos pruebas con números

```ts
...
describe('Pruebas de números', () => {

 it('El saldo debe ser 150 si ingresamos 50 en una cuenta con 100', () => {
    const cuenta = new Cuenta('Emilio', 100);
    cuenta.ingresar(50);

    expect(cuenta.saldo).toBe(150);
  });

    it('El saldo no debe cambiar si retiras más dinero del disponible', () => {
    const cuenta = new Cuenta('Emilio', 100);
    cuenta.retirar(150);

    expect(cuenta.saldo).toBe(100);
  });

})
```
<br>

> Paso 6. Añadimos pruebas con booleanos

```ts
import { mensaje, increment, userLogged, getColors } from "./basicos";

describe('preubas de booleanos', () => {

  it('Debe devolver true', () => {
    const res = userLogged();
    expect(res).toBeTruthy();

    expect(res).not.toBeFalsy();
    expect(res).toBe(true);
  });

});
```
<br>

> Paso 7. Añadimos pruebas con arrays

```ts
  it('Debe tener al menos tres servicios', () => {
    const cuenta = new Cuenta('Emilio', 100);

    expect(cuenta.servicios.length).toBeGreaterThanOrEqual(3);
  });

  it('Deben existir los servicios "tarjeta" y "seguro"', () => {
    const cuenta = new Cuenta('Emilio', 100);

    expect(cuenta.servicios).toContain('tarjeta');
    expect(cuenta.servicios).toContain('seguro');
  });
```
<br>

> Paso 8. Ciclo de vida de las pruebas

- Hay código repetido: inicializamos la cuenta en cada prueba
- Definimos una vez la cuenta al inicio del `describe`

<br>

```ts
import { Cuenta } from "./cuenta";

describe('Pruebas de Cuenta', () => {

  const cuenta = new Cuenta('Emilio', 100);

  it('Debe devolver el nombre del banco', () => {
    // const cuenta = new Cuenta('Emilio', 100);

    expect(cuenta.banco).toBe('Banco Testing');
  });

  it('El nombre de la cuenta debe contener el nombre del titular', () => {
    const titular = 'Emilio';
    const cuenta = new Cuenta(titular, 100);

    expect(cuenta.nombre()).toContain(titular);
  });

  it('El saldo debe ser 150 si ingresamos 50 en una cuenta con 100', () => {
    // const cuenta = new Cuenta('Emilio', 100);
    cuenta.ingresar(50); // Deja el saldo cambiado

    expect(cuenta.saldo).toBe(150);
  });

  it('El saldo no debe cambiar si retiras más dinero del disponible', () => {
    // const cuenta = new Cuenta('Emilio', 100);
    cuenta.retirar(150);

    expect(cuenta.saldo).toBe(100);
  });

  it('haySaldo debe devolver false si retiras todo el saldo', () => {
    // const cuenta = new Cuenta('Emilio', 100);

    expect(cuenta.haySaldo()).toBe(true);

    cuenta.retirar(100);
    expect(cuenta.haySaldo()).toBeFalsy();
  });

  it('Debe tener al menos tres servicios', () => {
    // const cuenta = new Cuenta('Emilio', 100);

    expect(cuenta.servicios.length).toBeGreaterThanOrEqual(3);
  });

  it('Deben existir los servicios "tarjeta" y "seguro"', () => {
    // const cuenta = new Cuenta('Emilio', 100);

    expect(cuenta.servicios).toContain('tarjeta');
    expect(cuenta.servicios).toContain('seguro');
  });

});
```

**`Fail`**: Fallan las pruebas porque las pruebas dejan el saldo modificado, para la siguiente prueba.

- Las pruebas deben estar aisladas
- Para solucionarlo inicializamos la cuenta antes de ejecutarse cada prueba

<br>

> Paso 9. Definimos funciones para cada evento del ciclo de vida


Ciclo de vida de las pruebas:

|Función                  |                                            |
| ---                     | ---                                        |
| `beforeAll(() => {});`  | Se ejecuta antes de empezar las pruebas    |
| `beforeEach(() => {});` | Se ejecuta antes de ejecutar cada prueba   |
| `afterEach(() => {});`  | Se ejecuta después de ejecutar cada prueba |
| `afterAll(() => {});`   | Se ejecuta después de finalizar todas las pruebas |

<br>

- Declaro la variable al inicio del `describe()`
- Inicializamos la variable en el `beforeEach()`

```ts
describe('Pruebas de Cuenta', () => {

  let cuenta: Cuenta;

   // Ciclo de vida de las pruebas
   beforeAll(() => {
    console.log('beforeAll');
  });

  beforeEach(() => {
    console.log('beforeEach');
    cuenta = new Cuenta('Emilio', 100);
  });

  afterEach(() => {
    console.log('afterEach');
  });

  afterAll(() => {
    console.log('afterAll');
  });

  it('Debe devolver el nombre del banco', () => {
    expect(cuenta.banco).toBe('Banco Testing');
  });

  ...
```

El resultado final quedaría como

``` ts
import { Cuenta } from "./cuenta";

describe('Pruebas de Cuenta', () => {

  let cuenta: Cuenta;

   // Ciclo de vida de las pruebas
   beforeAll(() => {});

  beforeEach(() => {
    cuenta = new Cuenta('Emilio', 100);
  });

  afterEach(() => {});

  afterAll(() => {});

  it('Debe devolver el nombre del banco', () => {
    expect(cuenta.banco).toBe('Banco Testing');
  });

  it('El nombre de la cuenta debe contener el nombre del titular', () => {
    const titular = 'Emilio';
    const cuenta = new Cuenta(titular, 100);

    expect(cuenta.nombre()).toContain(titular);
  });

  it('El saldo debe ser 150 si ingresamos 50 en una cuenta con 100', () => {
    cuenta.ingresar(50);

    expect(cuenta.saldo).toBe(150);
  });

  it('El saldo no debe cambiar si retiras más dinero del disponible', () => {
    cuenta.retirar(150);

    expect(cuenta.saldo).toBe(100);
  });

  it('haySaldo debe devolver false si retiras todo el saldo', () => {
    expect(cuenta.haySaldo()).toBe(true);

    cuenta.retirar(100);
    expect(cuenta.haySaldo()).toBeFalsy();
  });

  it('Debe tener al menos tres servicios', () => {
    expect(cuenta.servicios.length).toBeGreaterThanOrEqual(3);
  });

  it('Deben existir los servicios "tarjeta" y "seguro"', () => {
    expect(cuenta.servicios).toContain('tarjeta');
    expect(cuenta.servicios).toContain('seguro');
  });

});
```
<br>

## Saltar pruebas
---

Podemos saltarnos una prueba o una suite completa.
- `xdescribe()` se salta la suite de prueba
- `xit()` se salta la prueba