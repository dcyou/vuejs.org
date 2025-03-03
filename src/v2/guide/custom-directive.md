---
title: Directives personnalisées
type: guide
order: 302
---

## Introduction

<div class="vueschool"><a href="https://vueschool.io/lessons/create-vuejs-directive?friend=vuejs" target="_blank" rel="sponsored noopener" title="Cours Vue.js gratuit sur les directives personnalisées">Regarder le cours gratuit sur Vue School</a></div>

En supplément de l'ensemble de directives fournies par défaut (`v-model` et `v-show`), Vue vous permet également d'enregistrer vos propres directives. Notez qu'avec Vue 2.0, les composants sont la forme principale de réutilisabilité et d'abstraction du code. Il y a cependant des cas où vous aurez juste besoin d'un accès de bas niveau aux éléments du DOM, et c'est là que les directives personnalisées vous seraient utiles. Un exemple pourrait être la prise du focus sur un élément de champ, comme celui-ci :

{% raw %}
<div id="simplest-directive-example" class="demo">
  <input v-focus>
</div>
<script>
Vue.directive('focus', {
  inserted: function (el) {
    el.focus()
  }
})
new Vue({
  el: '#simplest-directive-example'
})
</script>
{% endraw %}

Quand la page se charge, cet élément prend le focus (notez que `autofocus` ne fonctionne pas sur Safari mobile). En fait, si vous n'avez cliqué sur rien du tout depuis votre arrivée sur la page, le champ ci-dessus devrait avoir le focus. À présent, jetons un œil à la directive qui pourrait accomplir cela :

``` js
// Enregistrer une directive globale appelée `v-focus`
Vue.directive('focus', {
  // Quand l'élément lié est inséré dans le DOM...
  inserted: function (el) {
    // L'élément prend le focus
    el.focus()
  }
})
```

Si vous préférez enregistrer à la place la directive en local, les composants acceptent également l'option `directives` :

``` js
directives: {
  focus: {
    // définition de la directive
    inserted: function (el) {
      el.focus()
    }
  }
}
```

Puis dans un template, vous pouvez utiliser le nouvel attribut `v-focus` sur n'importe quel élément, comme celui-ci :

``` html
<input v-focus>
```

## Fonctions de hook

Un objet de définition de directive peut fournir plusieurs fonctions de hook (toutes optionnelles) :

- `bind` : appelée une seule fois quand la directive est attachée à l'élément. C'est ici que vous pouvez effectuer les actions uniques d'initialisation.

- `inserted`: appelée quand l'élément lié a été inséré dans son nœud parent (cela garantit uniquement sa présence dans le nœud parent, mais pas nécessairement dans le document principal).

- `update`: appelée après que le composant conteneur VNode ait été mis à jour, __mais possiblement avant que ses enfants aient été mis à jour__. La valeur de la directive peut ou pas avoir changé, mais vous pouvez ignorer les mises à jour inutiles en comparant les valeurs actuelles et anciennes de la liaison (voir plus bas les arguments de hook).

<p class="tip">Nous couvrirons VNodes plus en détail [plus tard](./render-function.html#DOM-virtuel), quand nous discuterons des [fonctions de rendu](./render-function.html).</p>

- `componentUpdated`: appelée après que le composant conteneur VNode __et les VNode de ses enfants__ aient été mis à jour.

- `unbind`: appelée uniquement une fois, quand la directive est déliée de l'élément.

Nous allons explorer les arguments passés à ces hooks (c.-à-d. `el`, `binding`, `vnode`, et `oldVnode`) dans la prochaine section.

## Arguments des hooks d'une directive

Les hooks d'une directive ont accès à ces arguments :

- `el` : l'élément sur lequel la directive est liée. Cela peut être utilisé pour directement manipuler le DOM.
- `binding` : un objet contenant les propriétés suivantes.
  - `name` : le nom de la directive, sans le préfixe `v-`.
  - `value` : la valeur passée à la directive. Par exemple dans `v-my-directive="1 + 1"`, la valeur serait `2`.
  - `oldValue` : la valeur précédente, seulement disponible dans `update` et `componentUpdated`. Elle est disponible, que la valeur ait changé ou non.
  - `expression` : l'expression liée en tant que chaine de caractères. Par exemple dans `v-my-directive="1 + 1"`, l'expression serait `"1 + 1"`.
  - `arg` : l'argument passé à la directive, s'il y en a une. Par exemple dans `v-my-directive:foo`, l'argument serait `"foo"`.
  - `modifiers` : un objet contenant les modificateurs, s'il y en a. Par exemple dans `v-my-directive.foo.bar`, l'objet des modificateurs serait `{ foo: true, bar: true }`.
- `vnode` : le nœud virtuel produit par le compilateur Vue. Voir l'[API VNode](../api/#Interface-VNode) pour tous les détails.
- `oldVnode` : le nœud virtuel précédent, seulement disponible dans les hooks `update` et `componentUpdated`.

<p class="tip">À l'exception de `el`, vous devez traiter ces arguments comme étant en lecture seule (« read-only ») et ne jamais les modifier. Si vous souhaitez partager des informations entre les hooks, il est recommandé de le faire à travers les attributs de données sur mesure de ses éléments (voir [dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset)).</p>

Un exemple de directive personnalisée utilisant plusieurs de ces propriétés :

``` html
<div id="hook-arguments-example" v-demo:foo.a.b="message"></div>
```

``` js
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'bonjour !'
  }
})
```

{% raw %}
<div id="hook-arguments-example" v-demo:foo.a.b="message" class="demo"></div>
<script>
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})
new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'bonjour !'
  }
})
</script>
{% endraw %}

### Arguments de directive dynamiques

Les arguments de directive peuvent être dynamiques. Par exemple, dans `v-mydirective:[argument]="value"`, l'`argument` peut-être mis à jour en se basant sur la propriété de donnée dans notre instance de composant ! Cela rend nos directives personnalisées flexibles à travers notre application.

Imaginons que vous vouliez créer une directive personnalisée qui vous permet d'attacher des éléments sur votre page en utilisant le positionnement fixe. Nous pourrions créer une directive personnalisée ou la valeur met à jour le positionnement vertical en nombre de pixel, comme ceci :

```html
<div id="baseexample">
  <p>Faite défiler la page vers le bas ↓</p>
  <p v-pin="200">Je suis attaché à 200px depuis le haut de page.</p>
</div>
```

```js
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    el.style.top = binding.value + 'px'
  }
})

new Vue({
  el: '#baseexample'
})
```

Cela va attacher l'élément à 200px depuis le haut de la page. Mais que ce passe t-il si nous somme dans un scénario ou nous avons besoin d'attacher l'élément sur la gauche à la place du haut ? Ici nous avons un argument dynamique qui peut être mis à jour pour chaque instance du composant de manière très pratique :


```html
<div id="dynamicexample">
  <h3>Faite défiler la page vers le bas ↓</h3>
  <p v-pin:[direction]="200">Je suis attaché à 200px depuis la gauche de la page.</p>
</div>
```

```js
Vue.directive('pin', {
  bind: function (el, binding, vnode) {
    el.style.position = 'fixed'
    var s = (binding.arg == 'left' ? 'left' : 'top')
    el.style[s] = binding.value + 'px'
  }
})

new Vue({
  el: '#dynamicexample',
  data: function () {
    return {
      direction: 'left'
    }
  }
})
```

Résultat :

{% raw %}
<iframe height="200" style="width: 100%;" class="demo" scrolling="no" title="Dynamic Directive Arguments" src="//codepen.io/team/Vue/embed/rgLLzb/?height=300&theme-id=32763&default-tab=result" frameborder="no" allowtransparency="true" allowfullscreen="true">
  Voir le Pen <a href='https://codepen.io/team/Vue/pen/rgLLzb/'>Arguments de directive dynamiques</a> par Vue
  (<a href='https://codepen.io/Vue'>@Vue</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
{% endraw %}

Nos directives personnalisées sont maintenant assez flexibles pour supporter différents cas d'utilisation.

## Fonction abrégée

Dans de nombreux cas, vous pourriez vouloir le même comportement pour les hooks `bind` et `update`, sans avoir besoin des autres hooks. Par exemple :

``` js
Vue.directive('color-swatch', function (el, binding) {
  el.style.backgroundColor = binding.value
})
```

## Objets littéraux

Si votre directive a besoin de plusieurs valeurs, vous pouvez également passer un objet JavaScript. Souvenez-vous, une directive peut accepter n'importe quelle expression JavaScript.

``` html
<div v-demo="{ color: 'white', text: 'bonjour !' }"></div>
```

``` js
Vue.directive('demo', function (el, binding) {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text)  // => "bonjour !"
})
```
