# Web component

## Arrancar proyecto con ParcelJS

Sobre la raiz del proyecto, con ParcelJS instalado (Yo tengo instalada la versión 1.12.5)

```sh
parcel index.html
```

## Definción de la clase javascript

Creamos la clase con nombre <code>TooltipNotify</code> con nomenclatura PascalCase que extiende de la clase principal <code>HTMLElement</code>

```javascript
class TooltipNotify extends HTMLElement {}
```

## Registro de customElement

Con esto, y hasta aquí, el navegador ya reconoce nuestra nueva etiqueta, definiendo como primer parámetro el nombre HTML que tendrá, con almenos un guión medio, y omo segundo parámetro el nombre de la clase

```javascript
customElements.define("tooltip-notify", TooltipNotify);
```

## Constructor de clase

Si usamos contructor tendremos que pasar el método <code>super()</code> que hace referencia a la clase que extendemos

La palabra clave <code>this</code> hace referencia a mi mismo, a la clase, sus atributos, métodos... Llamamos al método <code>attachShadow()</code> con <code>mode: "open"</code> para crear un <code>shadowRoot</code>. Luego veremos la función <code>render()</code> que será la encargada de añadir el HTML de nuestro WebComponent

```javascript
class TooltipNotify extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: "open" });
    this.render();
  }
}
```

## Definición de propiedades

Definimos propiedades que tendrá el componente

```javascript
class TooltipNotify extends HTMLElement {
  fill = "#141414";
  count = "0";
  bulletColor = "green";
}
```

Si queremos que puedan ser actualizadas desde la vista necesitamos devolverlo como array en el método estático <code>observedAttributes() {}</code>. Para actualizar el nuevo valor necesitamos implementar el método <code>attributeChangedCallback(attr, oldValue, newValue)</code> que será llamado cada vez que algún valor de los que antes especificamos que serían susceptibles de cambio. Dentro del código del método con <code>switch</code> se actualizan las propiedades al nuevo valor. Además, hay que volver a pasar el método que realiza el pintado, que veremos en el siguiente punto

```javascript
class TooltipNotify extends HTMLElement {
  static get observedAttributes() {
    return ["fill", "count", "bullet-color"];
  }

  attributeChangedCallback(attr, oldValue, newValue) {
    if (oldValue === newValue) return;

    switch (attr) {
      case "fill":
        this.fill = newValue;
        break;
      case "count":
        this.count = newValue;
        break;
      case "bullet-color":
        this.bulletColor = newValue;
        break;
    }
    this.render();
  }
}
```

## Pintado del HTML

El método <code>render()</code> no es más que un método que dentro contiene texto para añadir nuestros CSS y HTML con algunas variables internas para que sea configurable. Este método añade HTML mediante la llamada al método <code>innerHTML</code> del <code>shadowRoot</code>

```javascript
class TooltipNotify extends HTMLElement {
  render() {
    this.shadowRoot.innerHTML = `
      <style>
        *:where(:not(iframe, canvas, img, svg, video):not(svg *)) {
          all: initial;
          display: revert;
          box-sizing: border-box;
        }
        ol {
          counter-reset: list;
          list-style: none;
          margin-block-start: 0;
          margin-block-end: 0;
          margin-inline-start: 0;
          margin-inline-end: 0;
          padding-inline-start: 0;
          display: inline-grid;
          gap: clamp(12px, 2vw, 18px);
          box-shadow: 1px 1px 4px rgb(0 0 0 / 20%);
          padding: clamp(16px, 3vw, 26px);
          border-radius: 3px;
          background-color: #fff;
          min-width: min(400px, 80vw);
          position: absolute;
          z-index: 9;
          visibility: hidden;
        }
        @media(max-width: 639.98px) {
          ol {
            transform: translateY(43px);
            width: calc(100vw);
            left: 0;
            right: 0;
          }
        }
        @media(min-width: 640px) {
          ol {
            transform: translate(calc(-18px - 50%), 40px);
          }
        }
        ::slotted(li) {
          counter-increment: list;
          display: grid;
          grid-template-columns: 2.6em 1fr;
          align-items: center;
          font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
          font-size: 1rem;
          font-weight: 400;
        }
        ::slotted(li)::before {
          content: counter(list);
          background-color: ${this.bulletColor};
          font-family: sans-serif;
          color: #fff;
          font-size: 13px;
          text-align: center;
          border-radius: 50%;
          width: 2em;
          height: 2em;
          line-height: 2.1em;
          display: inline-block;
        }
        strong {
          font-weight: 600;
        }
        button {
          display: inline-flex;
          cursor: pointer;
        }
        ::slotted(li) {
          transform: translateX(-10px);
          opacity: 0;
          pointer-events: none;
        }
        .open {
          visibility: visible;
        }
        .open ::slotted(li) {
          transform: translateX(0);
          opacity: 1;
          transition: transform 0.4s ease-out, opacity 0.3s linear;
        }
        ${this.transitions()}
      </style>
      <button type="button">
        <svg xmlns="http://www.w3.org/2000/svg" aria-hidden="true" viewBox="0 0 36 36" width="36" height="36">
          <path fill="${
            this.fill
          }" d="M18 2.1a16 16 0 1 0 16 16 16 16 0 0 0-16-16zm-.1 5.28a2 2 0 1 1-2 2 2 2 0 0 1 2-2zm3.6 21.25h-7a1.4 1.4 0 1 1 0-2.8h2.1v-9.2H15a1.4 1.4 0 1 1 0-2.8h4.4v12h2.1a1.4 1.4 0 1 1 0 2.8z" />
        </svg>
      </button>
      <ol count="{$this.count}">
        <slot></slot>
      </ol>
    `;
    this.transitions();
}
```

## Añadir eventos al WebComponent

El evento añadido es para hacer efecto <code>toggle</code> sobre el botón, al hacer <code>click</code> se añade o quita la clase <code>open</code>

Se ha creado una función auxiliar <code>handerEvent()</code> este fin

```javascript
class TooltipNotify extends HTMLElement {
  handlerEvent() {
    this.shadowRoot.querySelector("ol").classList.toggle("open");
  }
  attachEvents() {
    this.shadowRoot
      .querySelector("button")
      .addEventListener("click", this.handlerEvent.bind(this));
  }
  detachEvents() {
    this.shadowRoot
      .querySelector("button")
      .removeEventListener("click", this.handlerEvent.bind(this));
  }
}
```

## Registro de eventos a nuestro WebComponent

Una vez creado los eventos necesitamos comunicarlo a nuestro WebComponent, para ello, por extender la clase <code>HTMLElement</code> tenemos varios métodos que podemos implementar

```javascript
class TooltipNotify extends HTMLElement {
  connectedCallback() {
    this.attachEvents();
  }
  disconnectedCallback() {
    this.detachEvents();
  }
}
```

Como su nombre puede hacer intuir, uno es para cuando tenemos el WebComponent añadido <code>connectedCallback()</code> y el otro cuando se desconecta <code>disconnectedCallback()</code>

## Codepen del Webcomponent

En este PEN puede verse el <a href="https://codepen.io/ivan_albizu/pen/abLYbRL" target="_blank" rel="noopener">WebComponente funcionando</a>
