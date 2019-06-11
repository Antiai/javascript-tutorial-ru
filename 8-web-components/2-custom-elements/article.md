
# Пользовательские элементы

Мы можем создавать пользовательские HTML-элементы, описываемые своим собственным классом со своими методами, свойствами, событиями и т.д.

Как только пользовательский элемент определен, мы можем использовать его наравне с нативными HTML-элементами.

Это здорово, так как HTML-словарь достаточно широк, но не безграничен. В нем не существует `<easy-tabs>`, `<sliding-carousel>`, `<beautiful-upload>`... или просто вообразите любой другой тег, который может нам понадобиться.

Мы можем определить их через специальный класс, и потом использовать так же, как если бы они всегда были частью HTML.

Существуют 2 вида пользовательских элементов:

1. **Автономные пользовательские элементы** -- полностью новые элементы, расширяющие абстрактный `HTMLElement` класс.
2. **Измененные встроенные элементы** -- расширяющие встроенные элементы, такие как `HTMLButtonElement` и т.п.

Сначала мы создадим автономные элементы, а затем измененные встроенные.

Для создания пользовательского элемента нам необходимо сообщить браузеру некоторые детали этого элемента: как отображать, что делать, когда элемент добавлен или удален со страницы и т.д.

Это достигается через создание класса с особыми методами. Это просто, поскольку существуют только несколько таких методов, и все они необязательные.

Набросок с полным списком методов:

```js
class MyElement extends HTMLElement {
  constructor() {
    super();
    // элемент создан
  }

  connectedCallback() {
    // браузер вызывает этот метод, когда элемент добавляется в объект document
    // (может быть многократно вызван, если элемент периодически добавляется/удаляется)
  }

  disconnectedCallback() {
    // браузер вызывает этот метод, когда элемент удаляется из объекта document
    // (может быть многократно вызван, если элемент периодически добавляется/удаляется)
  }

  static get observedAttributes() {
    return [/* массив имен аттрибутов для отслеживания изменений */];
  }

  attributeChangedCallback(name, oldValue, newValue) {
    // вызывается в момент, когда один из перечисленных выше атрибутов изменен
  }

  adoptedCallback() {
    // вызывается, когда элемент перемещается в новый document
    // (происходит в document.adoptNode, очень редко используется)
  }

  // здесь могут быть другие методы и свойства элемента
}
```

После этого, нам необходимо зарегистрировать элемент:

```js
// даем браузеру знать, что <my-element> обслуживается нашим новым классом
customElements.define("my-element", MyElement);
```

Теперь для любых HTML-элементов с тэгом `<my-element>` создается экземпляр `MyElement` и вызываются вышеупомянутые методы. Мы также можем вызвать `document.createElement('my-element')` в JavaScript.

```smart header="Имя пользовательского элемента должно содержать дефис `-`"
Имя пользовательского элемента должно содержать дефис `-`, например `my-element` и `super-button` - допустимые имена, а `myelement` уже нет.

Это необходимо для того, чтобы гарантировать отсутствие конфликтов имен между встроенными и пользовательскими HTML-элементами.
```

## Пример: "time-formatted"

Например, существует элемент `<time>` в HTML, для вывода даты/времени. Но он не выполняет никакого форматирования сам по себе.

Создадим элемент `<time-formatted>`, который отображает время в удобном формате с учетом локали:


```html run height=50 autorun="no-epub"
<script>
*!*
class TimeFormatted extends HTMLElement { // (1)
*/!*

  connectedCallback() {
    let date = new Date(this.getAttribute('datetime') || Date.now());

    this.innerHTML = new Intl.DateTimeFormat("default", {
      year: this.getAttribute('year') || undefined,
      month: this.getAttribute('month') || undefined,
      day: this.getAttribute('day') || undefined,
      hour: this.getAttribute('hour') || undefined,
      minute: this.getAttribute('minute') || undefined,
      second: this.getAttribute('second') || undefined,
      timeZoneName: this.getAttribute('time-zone-name') || undefined,
    }).format(date);
  }

}

*!*
customElements.define("time-formatted", TimeFormatted); // (2)
*/!*
</script>

<!-- (3) -->
*!*
<time-formatted datetime="2019-12-01"
*/!*
  year="numeric" month="long" day="numeric"
  hour="numeric" minute="numeric" second="numeric"
  time-zone-name="short"
></time-formatted>
```

1. Класс имеет только один метод `connectedCallback()` -- браузер вызывает его, когда элемент `<time-formatted>` добавляется на страницу (или когда HTML-парсер определяет его), а так же для удобного форматирования времени используется встроенная утилита форматирования дат [Intl.DateTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat), имеющая хорошую кроссбраузерную поддержку.
2. Мы должны зарегистрировать наш новый элемент через `customElements.define(tag, class)`.
3. И теперь мы можем использовать его повсюду.


```smart header="Модернизация пользовательских элементов"
Если браузер встречает какие-либо `<time-formatted>` элементы до вызова `customElements.define`, это не ошибка. Просто элемент еще неизвестен, как и любой другой нестандартный тэг.

Такие "undefined" элементы могут быть стилизованы через CSS-селектор `:not(:defined)`.

Когда вызывается `customElement.define`, они "модернизируются": для каждого элемента
создается новый экземпляр класса `TimeFormatted`, и вызывается метод `connectedCallback`. Они становятся `:defined`.

Для получения информации о пользовательских элементах существуют методы:
- `customElements.get(name)` -- возвращает класс по значению указанного `name` пользовательского элемента,
- `customElements.whenDefined(name)` -- возвращает промис, который выполняется (не возвращая значения), когда пользовательский элемент с указанным `name` становится определенным.
```

```smart header="Вывод в `connectedCallback`, не в `constructor`"
В примере выше, содержимое элемента выводится(создается) в `connectedCallback`.

Почему не в `constructor`?

Причина проста: когда вызывается `constructor`, еще слишком рано. Экземпляр элемента уже создан, но еще не размещен. Браузер еще не обработал/присвоил атрибуты на этом этапе: вызов `getAttribute` вернул бы `null`. Поэтому мы действительно не можем выводить там.

Кроме того, если задуматься, так лучше с точки зрения производительности -- отложить работу до тех пор, когда это станет действительно необходимо.

`connectedCallback` срабатывает, когда элемент добавлен в document. Не просто, когда вставлен в другой элемент в качестве дочернего, а в момент, когда уже стал частью страницы. Так мы можем построить отдельный DOM, создать элементы и подготовить их для дальнейшего использования. Они будут выведены только тогда, когда это все уже выполнено на странице.
```

## Слежение за атрибутами

В текущей реализации `<time-formatted>` после вывода элемента, дальнейшие изменения атрибута не произведут никакого эффекта. Это странно для HTML-элемента. Обычно, когда мы меняем атрибут, например `a.href`, мы ожидаем, что изменения произойдут незамедлительно. Итак, давайте исправим это.

Мы можем следить за атрибутами, передав их список в статический геттер `observedAttributes()`. Для таких атрибутов вызывается `attributeChangedCallback`, когда они изменяются. Он не выполняется для любого атрибута из соображений производительности.

Here's a new `<time-formatted>`, that auto-updates when attributes change:

```html run autorun="no-epub" height=50
<script>
class TimeFormatted extends HTMLElement {

*!*
  render() { // (1)
*/!*
    let date = new Date(this.getAttribute('datetime') || Date.now());

    this.innerHTML = new Intl.DateTimeFormat("default", {
      year: this.getAttribute('year') || undefined,
      month: this.getAttribute('month') || undefined,
      day: this.getAttribute('day') || undefined,
      hour: this.getAttribute('hour') || undefined,
      minute: this.getAttribute('minute') || undefined,
      second: this.getAttribute('second') || undefined,
      timeZoneName: this.getAttribute('time-zone-name') || undefined,
    }).format(date);
  }

*!*
  connectedCallback() { // (2)
*/!*
    if (!this.rendered) {
      this.render();
      this.rendered = true;
    }
  }

*!*
  static get observedAttributes() { // (3)
*/!*
    return ['datetime', 'year', 'month', 'day', 'hour', 'minute', 'second', 'time-zone-name'];
  }

*!*
  attributeChangedCallback(name, oldValue, newValue) { // (4)
*/!*
    this.render();
  }

}

customElements.define("time-formatted", TimeFormatted);
</script>

<time-formatted id="elem" hour="numeric" minute="numeric" second="numeric"></time-formatted>

<script>
*!*
setInterval(() => elem.setAttribute('datetime', new Date()), 1000); // (5)
*/!*
</script>
```

1. The rendering logic is moved to `render()` helper method.
2. We call it once when the element is inserted into page.
3. For a change of an attribute, listed in `observedAttributes()`, `attributeChangedCallback` triggers.
4. ...and re-renders the element.
5. At the end, we can easily make a live timer.

## Rendering order

When HTML parser builds the DOM, elements are processed one after another, parents before children. E.g. if we have `<outer><inner></inner></outer>`, then `<outer>` element is created and connected to DOM first, and then `<inner>`.

That leads to important consequences for custom elements.

For example, if a custom element tries to access `innerHTML` in `connectedCallback`, it gets nothing:

```html run height=40
<script>
customElements.define('user-info', class extends HTMLElement {

  connectedCallback() {
*!*
    alert(this.innerHTML); // empty (*)
*/!*
  }

});
</script>

*!*
<user-info>John</user-info>
*/!*
```

If you run it, the `alert` is empty.

That's exactly because there are no children on that stage, the DOM is unfinished. HTML parser connected the custom element `<user-info>`, and will now proceed to its children, but just didn't yet.

If we'd like to pass information to custom element, we can use attributes. They are available immediately.

Or, if we really need the children, we can defer access to them with zero-delay `setTimeout`.

This works:

```html run height=40
<script>
customElements.define('user-info', class extends HTMLElement {

  connectedCallback() {
*!*
    setTimeout(() => alert(this.innerHTML)); // John (*)
*/!*
  }

});
</script>

*!*
<user-info>John</user-info>
*/!*
```

Now the `alert` in line `(*)` shows "John", as we run it asynchronously, after the HTML parsing is complete. We can process children if needed and finish the initialization.

On the other hand, this solution is also not perfect. If nested custom elements also use `setTimeout` to initialize themselves, then they queue up: the outer `setTimeout` triggers first, and then the inner one.

So the outer element finishes the initialization before the inner one.

Let's demonstrate that on example:

```html run height=0
<script>
customElements.define('user-info', class extends HTMLElement {
  connectedCallback() {
    alert(`${this.id} connected.`);
    setTimeout(() => alert(`${this.id} initialized.`));
  }
});
</script>

*!*
<user-info id="outer">
  <user-info id="inner"></user-info>
</user-info>
*/!*
```

Output order:

1. outer connected.
2. inner connected.
2. outer initialized.
4. inner initialized.

We can clearly see that the outer element does not wait for the inner one.

There's no built-in callback that triggers after nested elements are ready. But we can implement such thing on our own. For instance, inner elements can dispatch events like `initialized`, and outer ones can listen and react on them.

## Customized built-in elements

New elements that we create, such as `<time-formatted>`, don't have any associated semantics. They are unknown to search engines, and accessibility devices can't handle them.

But such things can be important. E.g, a search engine would be interested to know that we actually show a time. And if we're making a special kind of button, why not reuse the existing `<button>` functionality?

We can extend and customize built-in elements by inheriting from their classes.

For example, buttons are instances of `HTMLButtonElement`, let's build upon it.

1. Extend `HTMLButtonElement` with our class:

    ```js
    class HelloButton extends HTMLButtonElement { /* custom element methods */ }
    ```

2. Provide an third argument to `customElements.define`, that specifies the tag:
    ```js
    customElements.define('hello-button', HelloButton, *!*{extends: 'button'}*/!*);
    ```    
    There exist different tags that share the same class, that's why it's needed.

3. At the end, to use our custom element, insert a regular `<button>` tag, but add `is="hello-button"` to it:
    ```html
    <button is="hello-button">...</button>
    ```

Here's a full example:

```html run autorun="no-epub"
<script>
// The button that says "hello" on click
class HelloButton extends HTMLButtonElement {
*!*
  constructor() {
*/!*
    super();
    this.addEventListener('click', () => alert("Hello!"));
  }
}

*!*
customElements.define('hello-button', HelloButton, {extends: 'button'});
*/!*
</script>

*!*
<button is="hello-button">Click me</button>
*/!*

*!*
<button is="hello-button" disabled>Disabled</button>
*/!*
```

Our new button extends the built-in one. So it keeps the same styles and standard features like `disabled` attribute.

## References

- HTML Living Standard: <https://html.spec.whatwg.org/#custom-elements>.
- Compatiblity: <https://caniuse.com/#feat=custom-elements>.

## Summary

Custom elements can be of two types:

1. "Autonomous" -- new tags, extending `HTMLElement`.

    Definition scheme:

    ```js
    class MyElement extends HTMLElement {
      constructor() { super(); /* ... */ }
      connectedCallback() { /* ... */ }
      disconnectedCallback() { /* ... */  }
      static get observedAttributes() { return [/* ... */]; }
      attributeChangedCallback(name, oldValue, newValue) { /* ... */ }
      adoptedCallback() { /* ... */ }
     }
    customElements.define('my-element', MyElement);
    /* <my-element> */
    ```

2. "Customized built-in elements" -- extensions of existing elements.

    Requires one more `.define` argument, and `is="..."` in HTML:
    ```js
    class MyButton extends HTMLButtonElement { /*...*/ }
    customElements.define('my-button', MyElement, {extends: 'button'});
    /* <button is="my-button"> */
    ```

Custom elements are well-supported among browsers. Edge is a bit behind, but there's a polyfill <https://github.com/webcomponents/webcomponentsjs>.
