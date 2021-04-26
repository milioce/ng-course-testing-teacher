
## Herramientas de test
---
- **Jasmine** es un framework de BDD (*Behavior-Driven development*) para pruebas con JS
- **Karma** es un gestor de tareas. Tiene un fichero de configuración donde podemos definir la inicialización, los informes, el framework a utilizar, el navegador, etc

## Jasmine
---

### Suite
Una **Suite** es un grupo de pruebas relacionadas.
- Se define mediante la función `describe()` que tiene dos parámetros:
  - Nombre de la prueba unitaria
  - Función que contiene el código de la prueba
- Normalmente cada fichero de pruebas tiene un `describe`
<br>

### Specs
**Spec** o especificaciones, son funciones que devuelven verdadero o falso, según lo esperado.
- Se definen con la función `it()` que tiene dos parámetros
  - Título
  - Función con el código de la prueba que devuelve verdadero o falso.
- Dentro de cada Spec se crean uno o varios `expect` (expectativa), que realizas comprobaciones entre el valor actual y el esperado.
- Las variables creadas dentro la suite se pueden usar dentro de todas las Spec.

<br>

*Ejemplo de una Suite*
```ts
describe('A suite is a function', () => {
  let a;

  it('a spec is a function', () => {
    a = true;

    expect(a).toBe(true);
  });

});
```
<br>

### Matchers
Un **matcher** implementa una comparación booleana entre el valor actual y el valor esperado.

<br>

Ejemplo de funciones más comunes

| Matcher                    |                                          |
| ---                        | ---                                      |
| expect(x).toEqual(y)       | ambos valores son iguales                |
| expect(x).toBe(y)          | Ambos objetos son iguales                |
| expect(x).toMatch(pattern) | valor cumple un patrón                   |
| expect(x).toBeDefined()    | el valor esta definido                   |
| expect(x).toBeUndefined()  | el valor no esta definido                |
| expect(x).toBeNull()       | verifica si un valor es nulo             |
| expect(x).toBeTruthy()     | El valor es verdadero (!!valor === true) |
| expect(x).toBeTrue()       | El valor es igual a true                 |
| expect(x).toBeFalsy()      | El valor es falso (!!valor === false)    |
| expect(x).toContains(y)    | Si el valor contiene a y                 |
| expect(x).toBeLessThan(y)  | Si el valor es menor que y               |
| expect(x).not.toBe(true);  | Negar una función                        |

<br>

### Setup and Teardown.

Jasmine nos ofrece 4 funciones globales:
- beforeEach(() => {})
- afterEach(() => {})
- beforeAll(() => {})
- afterAll(() => {})

Se utilizan para resetear variables antes de cada Spec.

Hay que tener cuidado con beforeAll() y afterAll() porque solo se ejecutan una vez, y no se resetean entre cada Spec.

<br>

### Fail
Podemos provocar un Fallo


### Spies
Un Spies puede bloquear cualquier función y rastrear las llamadas que se le hacen y los argumentos que se le pasan.

Un spy solo existe en el bloque `describe` o en el bloque `it` donde se ha definido.

Hay matchers especiales para interactuar con espías
  - expect(spy).toHaveBeenCalled()
  - expect(spy).toHaveBeenCalledTimes()
  - expect(spy).toHaveBeenCalledWith()

```ts
describe("A spy", function() {
  var foo, bar = null;

  beforeEach(function() {
    foo = {
      setBar: function(value) {
        bar = value;
      }
    };

    spyOn(foo, 'setBar');

    foo.setBar(123);
    foo.setBar(456, 'another param');
  });

  // Mira si el espía ha sido llamado
  it("tracks that the spy was called", function() {
    expect(foo.setBar).toHaveBeenCalled();
  });

  // Mira si el espía ha sido llamado x veces
  it("tracks that the spy was called x times", function() {
    expect(foo.setBar).toHaveBeenCalledTimes(2);
  });

  // Comprueba si ha sido llamado con los argumentos esperados.
  it("tracks all the arguments of its calls", function() {
    expect(foo.setBar).toHaveBeenCalledWith(123);
    expect(foo.setBar).toHaveBeenCalledWith(456, 'another param');
  });

});

```
<br>
