# Pruebas en el HTML del componente

## 1. Crear componente incrementador.
---

``` bash
$ ng g c intermedio/increment

CREATE src/app/intermedio/increment/increment.component.html (24 bytes)
CREATE src/app/intermedio/increment/increment.component.spec.ts (647 bytes)
CREATE src/app/intermedio/increment/increment.component.ts (287 bytes)
CREATE src/app/intermedio/increment/increment.component.css (0 bytes)
UPDATE src/app/app.module.ts (595 bytes)
```

`app/intermedio/increment/increment.component.html`
``` html
<h3>
    {{ title }} - {{ progress }}
</h3>

<button class="btn btn-primary" type="button" (click)="changeValue(-5)">
    <i class="mdi mdi-minus"></i>
</button>

<input type="number" class="form-control" name="progress"
  [(ngModel)]="progress"
  (ngModelChange)="onChanges($event)"
  min="1" max="100">

<button class="btn btn-primary" type="button" (click)="changeValue(+5)">
    <i class="mdi mdi-plus"></i>
</button>
```
<br>

`app/intermedio/increment/increment.component.ts`
``` ts
import { Component, Input, Output, ViewChild, OnInit, EventEmitter, ElementRef } from '@angular/core';

@Component({
  selector: 'app-increment',
  templateUrl: './increment.component.html',
  styles: []
})
export class IncrementadorComponent implements OnInit {
  @Input() title: string = 'Title';
  @Input() progress: number = 50;

  @Output() valueChanged: EventEmitter<number> = new EventEmitter();

  constructor() { }

  ngOnInit() { }

  changeValue( value: number ) {
    let newProgress = this.progress + value;

    if ( newProgress > 100 ) {
      newProgress = 100;
    } else if ( newProgress < 0 ) {
      newProgress = 0;
    }

    if (newProgress !== this.progress) {
      this.progress = newProgress;
      this.valueChanged.emit( this.progress );
    }
  }

}
```
<br>

`app/intermedio/increment/increment.component.spec.ts`
```ts
import { ComponentFixture, TestBed } from '@angular/core/testing';

import { IncrementComponent } from './increment.component';

describe('IncrementComponent', () => {
  let component: IncrementComponent;
  let fixture: ComponentFixture<IncrementComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ IncrementComponent ],
      imports: [FormsModule]
    })
    .compileComponents();
  });

  beforeEach(() => {
    fixture = TestBed.createComponent(IncrementComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

## 2. Prueba que el texto leyenda esta dentro de un h3
---

- Damos valor a la propiedad del componente.
- Lanzamos la detección de cambios de Angular
- Obtenemos el elemento del HTML
  - fixture.debugElement.query, .queryAll
  - By.css, By.id, By.directive
- Comprobamos que el innerHTML contenga el texto buscado

`app/intermedio/increment/increment.component.spec.ts`
```ts
import { By } from '@angular/platform-browser';
...

  it('Se debe mostrar el título', () => {

    component.title = 'Testing title';

    fixture.detectChanges();

    const elem: HTMLElement = fixture.debugElement.query(
      By.css('h3')
    ).nativeElement;

    expect(elem.innerHTML).toContain('Testing title');

  })
```
<br>

## 3. Comprobar que el valor del input se actualiza.
---

Comprobar que el valor del input se actualiza cuando llamamos a la función cambiarValor()

`app/intermedio/increment/increment.component.spec.ts`
```ts
...

  it('Se debe mostrar en el input el valor del progreso', () => {
    component.changeValue(+5);
    fixture.detectChanges();

    fixture.whenStable().then(() => {
      const input = fixture.debugElement.query(By.css('input'));
      const elem: HTMLInputElement = input.nativeElement;

      expect(elem.value).toBe('55');
    });
  });
```
- Si no usamos `fixture.whenStable()` se evalua `elem.value` antes de que termine el proceso de detección de cambios.

<br>

## Comprobar los eventos de los elementos HTML
---

- Angular `DebugElement` nos ofrece un método para disparar eventos: `triggerEventHandler`
  <br>Sirve para eventos nativos (event binding), @HostListener() y @Output()
- El handler se ejecuta de forma sincrona
- No dispara la detección de cambios

<br>

`app/intermedio/increment/increment.component.spec.ts`
```ts
...

  it('Se debe incrementar en 5 al hacer click en el botón', () => {
    const buttons = fixture.debugElement.queryAll(By.css('.btn'));
    console.log(buttons);

    buttons[0].triggerEventHandler('click', null);

    expect(component.progress).toBe(45);

    // <my-component (data)="onData($event)"></my-component>
    // const myComponent = fixture.debugElement.query(By.directive(MyComponent));
    // myComponent.triggerEventHandler('data', { id: '1' });

    // Si uso button[0].nativeElement, debería lanzar el evento con dispatchEvent()
    // y lanzar el fixture.detectChanges()
  });
```
<br>

- `debugElement.triggerEventHandler(eventName: string, eventObj: any)`
  - eventName es el nombre del evento
  - eventObj es el objeto que se envía el evento

Por ejemplo, en este componente

```html
<my-component (data)="onData($event)"></my-component>
```

Podríamos lanzar este evento.

```ts
  const myComponent = fixture.debugElement.query(By.directive(MyComponent));
  myComponent.triggerEventHandler('data', { id: '1' });
```