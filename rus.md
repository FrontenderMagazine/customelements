# Настраиваемые элементы: как определять новые элементы в HTML

**Внимание!** В этой статье обсуждаются API, которые еще не полностью стандартизированы
и вполне могут измениться. Будьте осторожны с использованием экспериментальных
API внутри своих проектов.

## Введение

Вебу не хватает выразительности. Это легко проиллюстрировать — вот, посмотрите
на «современное» веб-приложение вроде Gmail:

![Gmail][1]

Современные веб-приложения это каша из `<div>`ов.

В каше из `<div>`ов нет ничегошеньки современного. А все-таки вот так мы сейчас
разрабатываем веб-приложения. И это весьма печально. Не стоит ли нам все-таки
потребовать несколько больше от нашей платформы?

### Секси-разметка. Давайте воплотим ее в жизнь.

HTML дает нам отличный инструмент для структурирования документа, но его словарь
ограничен элементами, определенными в [стандарте HTML ][2].

Ну а если бы разметка Gmail была бы не настолько ужасной, а, наоборот, красивой:

    <hangout-module>
      <hangout-chat from="Пол Эдди">
        <hangout-discussion>
          <hangout-message from="Пол" profile="profile.png"
              profile="118075919496626375791" datetime="2013-07-17T12:02">
            <p>По полной врубаюсь в эту штуку — веб-компоненты.</p>
            <p>Слышал о такой?</p>
          </hangout-message>
        </hangout-discussion>
      </hangout-chat>
      <hangout-chat>...</hangout-chat>
    </hangout-module>

[Попробуйте демо!][3]

Уф, глоток свежего воздуха! Все в этом приложении на своем месте: оно **осмысленно**,
его **легко понять** и, что лучше всего, **легко поддерживать**. Мне (или вам) в
будущем будет абсолютно понятно, что это приложение делает, стоит только взглянуть
на его декларативный скелет.

Кастомные элементы, на помощь! Вы — наша единственная надежда!

## Начинаем

[Кастомные элементы][4] **позволяют веб-разработчикам определять новые типы
HTML-элементов**. Эта спецификация — одна из нескольких новых корневых API,
которые проходят по ведомству [веб-компонентов][5], но из всех них, пожалуй,
самая важная. Веб-компоненты просто не могут существовать без тех функций,
которые предоставляют кастомные элементы:

  1. возможность определять новые элементы HTML/DOM;
  2. создавать элементы, которые расширяют функции других элементов;
  3. логически объединять кастомную функциональность в один тэг;
  4. расширять API существующих DOM-элементов.

### Регистрация новых элементов

Кастомные элементы можно создать с помощью функции `document.registerElement()`:

    var XFoo = document.registerElement('x-foo');
    document.body.appendChild(new XFoo());

Первый аргумент `document.registerElement()` — название тэга элемента. Это название
**обязательно должно содержать дефис (-)**. Например, `x-tags`, `my-element` и
`my-awesome-app` — это разрешенные имена для новых элементов, а `tabs` и `foo_bar`
использовать нельзя. Это ограничение позволяет парсеру отличать кастомные элементы
от обычных и обеспечивает будущую совместимость, когда к HTML будут добавляться
новые тэги.

Второй (необязательный) аргумент — объект, который описывает прототип элемента.
Здесь к элементам можно добавлять кастомную функциональность (например,
публичные свойства и методы). Но об этом — [позже][6].

По умолчанию кастомные элементы наследуют от `HTMLElement`. Таким образом,
предыдущий пример соответствует следующему коду:

    var XFoo = document.registerElement('x-foo', {
      prototype: Object.create(HTMLElement.prototype)
    });

Вызов `document.registerElement('x-foo')` обучает браузер новому элементу и возвращает
функцию-конструктор, которую можно использовать для того, чтобы создавать
экземпляры `x-foo`. Если вы не хотите использовать конструктор, то есть и
другие [способы инициализации элемента][7].

Если вы не хотите, чтобы конструктор находился внутри глобального элемента `window`,
его можно поместить в некое пространство имен (`var myapp = {}; myapp.XFoo = document.registerElement('x-foo');`)
или вообще нигде не сохранять на него ссылку.

### Расширение встроенных элементов

Допустим, вы недовольны обычной, простой кнопкой `button`. Вы бы хотели
серьезно расширить её возможности, чтобы кнопка стала мега-кнопкой. Для этого,
чтобы расширить элемент `button`, вам нужно создать новый элемент, который
наследует прототип `HTMLButtonElement`:

    var MegaButton = document.registerElement('mega-button', {
      prototype: Object.create(HTMLButtonElement.prototype)
    });

Чтобы создать **элемент A**, расширяющий **элемент B**, **элемент A** должен
наследовать прототип от **элемента B**.

Такие кастомные элементы называются _кастомными элементами расширения типа_. Они
наследуют от конкретного `HTMLElement`, как бы говоря: «элемент X — это Y».

Пример:

    <button is="mega-button">


### Обновление элементов

Задумывались ли вы когда-нибудь, почему HTML-парсер не ругается на нестандартные
тэги? Например, он совершенно не будет против, если мы объявим на странице `<randomtag>`.
Согласно [спецификации HTML][8]:

Для HTML-элементов, которые не определены в этой спецификации, должен использоваться
интерфейс `HTMLUnknownElement`.

Прости, `randomtag`! Ты у нас нестандартный и наследуешь от `HTMLUnknownElement`.

А вот кастомных элементов это не касается. **Элементы с корректными именами для
кастомных элементов наследуют от `HTMLElement`.** Это можно проверить, открыв
консоль: Ctrl+Shift+J (или Cmd+Opt+J на Mac) и вставив следующие строчки кода —
обе возвратят `true`:


    // «tabs» — некорректное имя для кастомного элемента
    document.createElement('tabs').__proto__ === HTMLUnknownElement.prototype

    // «x-tabs» — корректное имя для кастомного элемента
    document.createElement('x-tabs').__proto__ == HTMLElement.prototype


> **Примечание:** `<x-tabs>` все равно будет являться `HTMLUnknownElement` в тех
браузерах, которые не поддерживают `document.registerElement()`.

#### Неопознанные элементы

Поскольку кастомные элементы регистрируются через скрипт (`document.registerElement()`),
**они могут быть объявлены или созданы _до того_, как браузер зарегистрирует их
определение**. Например, на странице вы можете определить `x-tabs`, а
`document.registerElement('x-tabs')` выполнить намного позднее.

Перед тем, как элементы обновят свои определения, они называются **неопознанными
элементами**. Это HTML-элементы, у которых есть корректное имя для кастомного
элемента, но они еще не зарегистрированы.

Эта таблица может помочь немного прояснить ситуацию:

<table class="test-table">
    <thead>
        <tr>
            <th>Название</th>
            <th>Наследует от</th>
            <th>Примеры</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Неопознанный элемент</td>
            <td>HTMLElement</td>
            <td>&lt;x-tabs&gt;, &lt;my-element&gt;, &lt;my-awesome-app&gt;</td>
        </tr>
        <tr>
            <td>Неизвестный элемент</td>
            <td>HTMLUnknownElement</td>
            <td>&lt;tabs&gt;, &lt;foo_bar&gt;</td>
        </tr>
    </tbody>
</table>

Неопознанные элементы находятся как бы в лимбе: это потенциальные кандидаты для
браузера, с которыми он может работать дальше. Браузер говорит: «Ну что же, в вас
 есть все качества, которые я ищу в новом элементе. Я обещаю, что сделаю вас
 настоящим элементом, когда мне дадут ваше определение».

## Инициализация элементов

Все общие приемы создания элементов относятся и к кастомным элементам. Как и
любой стандартный элемент, его можно объявить в HTML или создать внутри DOM
с помощью JavaScript.

### Инициализация кастомных тэгов

**Объявите** их:

    <x-foo></x-foo>

**Создайте DOM** с помощью JavaScript:

    var xFoo = document.createElement('x-foo');
    xFoo.addEventListener('click', function(e) {
      alert('Спасибо!');
    });

Используйте **оператор `new`**:

    var xFoo = new XFoo();
    document.body.appendChild(xFoo);

### Инициализация элементов расширения типа

Инициализировать кастомные элементы, расширяющие тип, можно практически так же,
как и кастомные тэги.

**Объявите** их:

    <!-- <button> — это мега-кнопка -->
    <button is="mega-button">

**Создайте DOM** с помощью JavaScript:

    var megaButton = document.createElement('button', 'mega-button');
    // megaButton instanceof MegaButton === true

Как видите, функция `document.createElement()` может принимать атрибут `is=""`
в качестве своего второго параметра.

Используйте **оператор `new`**:

    var megaButton = new MegaButton();
    document.body.appendChild(megaButton);

Итак, мы разобрались, как использовать `document.registerElement()` для того, чтобы
рассказать браузеру о новом тэге… ну и что? Пока ничего не происходит. Давайте
добавим свойства и методы.

## Добавляем публичные свойства и методы

Самое интересное в кастомных элементах — то, что вы можете дополнять
функциональные возможности элемента как вам угодно, доопределив его свойства и
методы. Это можно представить себе как способ создавать для своего элемента
публичный API.

Вот полный пример:

    var XFooProto = Object.create(HTMLElement.prototype);

    // 1. Определяем для x-foo метод foo().
    XFooProto.foo = function() {
      alert('вызван метод foo()');
    };

    // 2. Определяем свойство «bar» (только для чтения).
    Object.defineProperty(XFooProto, "bar", {value: 5});

    // 3. Регистрируем определение x-foo.
    var XFoo = document.registerElement('x-foo', {prototype: XFooProto});

    // 4. Создаем элемент x-foo.
    var xfoo = document.createElement('x-foo');

    // 5. Добавляем его на страницу.
    document.body.appendChild(xfoo);


Конечно, есть тысяча способов сконструировать прототип. Если вам не нравится создавать
прототипы так, как описано выше, вот краткая версия этого кода:


    var XFoo = document.registerElement('x-foo', {
      prototype: Object.create(HTMLElement.prototype, {
        bar: {
          get: function() { return 5; }
        },
        foo: {
          value: function() {
            alert('вызван метод foo()');
          }
        }
      })
    });


Первый формат позволяет использовать [`Object.defineProperty`][9] из ES5.
Второй позволяет использовать [get/set][10].

### Коллбэки на протяжении жизненного цикла элемента

Элементы могут определять специальные методы для того, чтобы вы могли включиться
в интересные моменты их существования. Эти методы называются **колбэками
жизненного цикла**. У каждого есть свое название и смысл:

<table class="test-table">
    <thead>
        <tr>
            <th>Название коллбэка</th>
            <th>Вызывается, когда</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>createdCallback</td>
            <td>создан экземпляр элемента</td>
        </tr>
        <tr>
            <td>enteredDocumentCallback</td>
            <td>экземпляр вставлен в документ</td>
        </tr>
        <tr>
            <td>leftDocumentCallback</td>
            <td>экземпляр удален из документа</td>
        </tr>
        <tr>
            <td>attributeChangedCallback(attrName, oldVal, newVal)</td>
            <td>был добавлен, удален или изменен атрибут</td>
        </tr>
    </tbody>
</table>

**Пример:** определяем `createdCallback()` и `enteredDocumentCallback()` для `x-foo`:

    var proto = Object.create(HTMLElement.prototype);

    proto.createdCallback = function() {...};
    proto.enteredDocumentCallback = function() {...};

    var XFoo = document.registerElement('x-foo', {prototype: proto});


**Все коллбэки жизненного цикла необязательны**, определяйте их тогда, когда
это имеет смысл. Например, если у вас достаточно сложный элемент, который должен
открывать соединение к IndexedDB в `createdCallback()`. Тогда перед тем, как этот
элемент будет удален из DOM, подчистите все внутри `leftDocumentCallback()`.
**Примечание:** нельзя рассчитывать только на это: может случиться и так, что
пользователь просто закроет таб, но все-таки думайте об этом как о возможности
для оптимизации.

Еще один сценарий использования коллбэков жизненного цикла — устанавливать на
элементе обработчики событий по умолчанию:

    proto.createdCallback = function() {
      this.addEventListener('click', function(e) {
        alert('Спасибо!');
      });
    };


## Добавляем разметку

Мы создали `x-foo`, описали для него JavaScript-API, но тэг пустой! Давайте
выведем внутри него какой-нибудь HTML?

Здесь нам пригодятся [коллбэки жизненного цикла][11]. А если конкретно, можно
использовать `createdCallback()` и приписать элементу какой-нибудь HTML по
умолчанию:


    var XFooProto = Object.create(HTMLElement.prototype);

    XFooProto.createdCallback = function() {
      this.innerHTML = "Я — x-foo-with-markup!";
    };

    var XFoo = document.registerElement('x-foo-with-markup', {prototype: XFooProto});


Инициализируем этот тэг и смотрим на него в DevTools (правый клик, выбираем
«просмотр элемента») и видим:

    ▾<x-foo-with-markup>
       <b>Я — x-foo-with-markup!</b>
     </x-foo-with-markup>


### Храним внутреннюю логику в Shadow DOM

Сам по себе [Shadow DOM][12] — это мощный инструмент для независимого хранения
контента. Используйте его вместе с кастомными элементами — и все приобретет
магический оттенок!

Shadow DOM дает кастомным элементам:

  1. возможность прятать от пользователя внутреннюю сторону своей реализации;
  2. [изоляция стилей][13] — бесплатно!

Создавать элемент Shadow DOM можно точно так же, как и создавать элемент
разметки. Разница содержится в `createdCallback()`:


    var XFooProto = Object.create(HTMLElement.prototype);

    XFooProto.createdCallback = function() {
      // 1. Создаем теневой корневой элемент
      var shadow = this.createShadowRoot();

      // 2. Помещаем в него разметку
      shadow.innerHTML = "Я внутри Shadow DOM элемента!";
    };

    var XFoo = document.registerElement('x-foo-shadowdom', {prototype: XFooProto});


Вместо того, чтобы устанавливать `.innerHTML` элемента, я создал теневой
корневой элемент для `x-foo-shadowdom` и поместил туда разметку. Если внутри
инструментов разработчика у вас включена настройка «Показывать Shadow DOM»,
то вы увидите, что `#document-fragment` можно раскрыть:

    ▾<x-foo-shadowdom>
       ▾#document-fragment
         <b>Я внутри Shadow DOM элемента!</b>
     </x-foo-shadowdom>

Вот и он, теневой корневой элемент!

### Создаем элементы по шаблону

[HTML-шаблоны][14] — это еще один новый низкоуровневый API, который прекрасно
вписывается в мир кастомных элементов.

Если кто еще не знает, [элемент `<template>`][15] позволяет вас объявлять
фрагменты DOM, которые парсятся, с ними ничего не происходит на этапе загрузке
страницы, но потом они инициализируются через JavaScript. HTML-шаблоны —
идеальный формат для того, чтобы объявлять структуру кастомного элемента.

**Пример:** регистрируем элемент, созданный из `<template>` и Shadow DOM:

    <template id="sdtemplate">
      <style>
        p { color: orange; }
      </style>
      <p>Я внутри Shadow DOM. Моя разметка взята из &lt;template&gt;.</p>
    </template>

    <script>
    var proto = Object.create(HTMLElement.prototype, {
      createdCallback: {
        value: function() {
          var t = document.querySelector('#sdtemplate');
          this.createShadowRoot().appendChild(t.content.cloneNode(true));
        }
      }
    });
    document.registerElement('x-foo-from-template', {prototype: proto});
    </script>

В этой паре строк кода довольно много всего. Давайте разберемся во всем, что
происходит:

  1. мы зарегистрировали в HTML новый элемент: `<template>`;
  2. из `<template>` мы создали DOM элемента;
  3. все страшные детали элемента спрятаны в Shadow DOM;
  4. Shadow DOM дает элементу изоляцию стилей: т.е. `p {color: orange;}` не
  заливает оранжевым всю страницу.

Отлично!

## Стилизация кастомных элементов

Как и в случае любого HTML-тэга, ваш кастомный тэг можно стилизовать
используя селекторы:

    <style>
      app-panel {
        display: flex;
      }
      [is="x-item"] {
        transition: opacity 400ms ease-in-out;
        opacity: 0.3;
        flex: 1;
        text-align: center;
        border-radius: 50%;
      }
      [is="x-item"]:hover {
        opacity: 1.0;
        background: rgb(255, 0, 255);
        color: white;
      }
      app-panel > [is="x-item"] {
        padding: 5px;
        list-style: none;
        margin: 0 7px;
      }
    </style>

    <app-panel>
      <li is="x-item">До</li>
      <li is="x-item">Ре</li>
      <li is="x-item">Ми</li>
    </app-panel>

### Стилизация элементов, использующих Shadow DOM

Кроличья нора ведет гораздо, _гораздо_ глубже, когда вы создаете решения использующие
Shadow DOM. [Кастомным элементам, которые используют Shadow DOM][16] достаются
и все преимущества последней.

Shadow DOM дает элементу изоляцию стилей. Стили, которые определяются в теневом
корневом элементе, не выходят за его пределы и не растекаются по странице.
**В случае кастомного элемента сам элемент — и есть корневой элемент для стилей.**
Свойства изоляции стилей, кроме того, позволяют кастомным элементам определять
и стили по умолчанию для самих себя.

Стилизация Shadow DOM — обширная тема! Если вы хотите узнать ее лучше, советую
прочесть несколько моих статей:

* [«Руководство по стилизации элементов»][24] в [документации по Polymer][25].
* [«Второй курс по Shadow DOM: CSS и стили»][26] на html5rocks.com

### Предотвращаем мигание контента с помощью :unresolved

Чтобы предотвратить мигание контента ([FOUC][17]), в спецификации по кастомным
элементам предусмотрен новый CSS-псевдокласс `:unresolved`. Вы можете целенаправленно
использовать его на [неопознанных элементах][18], и он будет применяться ровно
до тех пор, пока браузер не вызовет `createdCallback()` (см. [методы жизненного цикла][11]).
После того, как это произойдет, элемент больше не является неопознанным, он
обновится и превратился в элемент, соответствующий его определению.

CSS-псевдокласс `:unresolved` поддерживается Chrome 29.

**Пример**: заставляем тэги `x-foo` всплывать, когда они зарегистрированы:

    <style>
      x-foo {
        opacity: 1;
        transition: opacity 300ms;
      }
      x-foo:unresolved {
        opacity: 0;
      }
    </style>

Учтите, что `:unresolved` применяется только к [неопознанным элементам][18], но
не к элементам, которые наследуют от `HTMLUnkownElement` (см. [Обновление элементов][19]).

    <style>
      /* применить пунктирную границу ко всем неопознанным элементам */
      :unresolved {
        border: 1px dashed red;
        display: inline-block;
      }
      /* неопознанные x-panel — красные */
      x-panel:unresolved {
        color: red;
      }
      /* после того, как определение x-panel регистрируется, они становятся зелеными */
      x-panel {
        color: green;
        display: block;
        padding: 5px;
        display: block;
      }
    </style>

    <panel>
      Я черного цвета, потому что :unresolved не относится к «panel».
      Это недопустимое имя для кастомного элемента.
    </panel>

    <x-panel>Я красного цвета, потому что подхожу под селектор x-panel:unresolved.</x-panel>

Для более подробной информации об `:unresolved` смотрите [Руководство по стилизации
элементов][20] Polymer.

## История и поддержка браузерами

### Определение функциональности

Определить, поддерживает ли браузер эту функциональность, довольно просто — нужно
проверить, существует ли `document.registerElement()`:


    function supportsCustomElements() {
      return 'registerElement' in document;
    }

    if (supportsCustomElements()) {
      // Отлично!
    } else {
      // Используйте другие библиотеки для создания компонентов.
    }


### Поддержка браузерами

`document.registerElement()` впервые начал поддерживаться в Chrome 27 и Firefox ~23.
Однако спецификация с тех пор несколько развилась. Последняя спецификация
поддерживается начиная с Chrome 31.

Кастомные элементы в Chrome 31 можно включить, поставив флаг на «Экспериментальных
функциях веб-платформы» в `about:flags`.

Пока что поддержка браузерами далека от совершенства, но есть несколько отличных
полифиллов:

* [полифилл][27] внутри [Polymer][28] от Google;
* [x-tags][29] Mozilla.

### Что случилось HTMLElementElement?

Те, кто следил за работой по разработке стандарта, знают, что раньше существовал
`<element>`. Это была самая крутая вещь на деревне. Ее можно использовать, чтобы
декларативно регистрировать новые элементы:

    <element name="my-element">
      ...
    </element>

К сожалению, с этой спецификацией было слишком много проблем — когда [обновлять статус элемента][19],
было несколько проблемных сценариев и сценариев, в которых наступал совсем уж конец света.
Решить это было нельзя. `element` пришлось положить на полку. В августе 2013 Дмитрий Глазков
объявил в [public-webapps][21] о его удалении из спецификации, по крайней мере пока.

Нужно отметить, что внутри Polymer существует декларативная форма регистрации
элемента: `<polymer-element>`. Как они это делают? Используется
`document.registerElement('polymer-element')` и приемы, которые я описал в главе
«[Создание элементов из шаблона][22]».

## Заключение

Кастомные элементы дают нам инструмент для расширения словаря HTML, возможность
научить его новым приемам и прыгать со скоростью света по веб-платформе.
Совместите их с другими низкоуровневыми API — Shadow DOM и `<template>` — и вы
увидите полную картину веб-компонентов. Разметка может снова стать сексуальной!

Если вам интересно начать работать с веб-компонентам, посмотрите на [Polymer][23].
Здесь более чем достаточно информации для старта.

   [1]: img/gmail.png
   [2]: http://www.whatwg.org/specs/web-apps/current-work/multipage/
   [3]: https://html5-demos.appspot.com/static/webcomponents-bdconf/demos/components/hangouts/index.html
   [4]: https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/custom/index.html
   [5]: https://www.w3.org/TR/2013/WD-components-intro-20130606/
   [6]: #dobavlyaempublichnesvoystvaimetod
   [7]: #initsializatsiyalementov
   [8]: http://www.whatwg.org/specs/web-apps/current-work/multipage/elements.html#htmlunknownelement
   [9]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty
   [10]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/get
   [11]: #kollbkinaprotyazheniizhiznennogotsiklalementa
   [12]: http://www.html5rocks.com/tutorials/webcomponents/shadowdom/
   [13]: http://www.html5rocks.com/tutorials/webcomponents/shadowdom-201/
   [14]: https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/templates/index.html
   [15]: http://www.html5rocks.com/tutorials/webcomponents/template/
   [16]: #hranimvnutrennyuyulogikuvshadowdom
   [17]: http://en.wikipedia.org/wiki/Flash_of_unstyled_content
   [18]: #neopoznannelement
   [19]: #obnovlenielementov
   [20]: http://www.polymer-project.org/articles/styling-elements.html#preventing-fouc
   [21]: http://lists.w3.org/Archives/Public/public-webapps/2013JulSep/0287.html
   [22]: #sozdaemlementposhablonu
   [23]: http://www.polymer-project.org/
   [24]: http://www.polymer-project.org/articles/styling-elements.html
   [25]: http://www.polymer-project.org/
   [26]: http://www.html5rocks.com/tutorials/webcomponents/shadowdom-201/
   [27]: http://www.polymer-project.org/platform/custom-elements.html
   [28]: http://polymer-project.org/
   [29]: http://www.x-tags.org/
