---
title: 'Een eenvoudige component maken'
tocTitle: 'Eenvoudige component'
description: 'Maak een eenvoudige component in isolatie'
commit: 'f03552f'
---

We gaan onze uI component bouwen volgens de [Component-Driven Development](https://www.componentdriven.org/) (CDD) methodologie. Via deze methode bouw je UI's van de bodem af op waarbij je begint met een component en eindigt met schermen. CDD helpt je de complexiteit die komt kijken bij het bouwen van de UI beheersbaar te maken. 

## Task

![Task component in drie statussen](/intro-to-storybook/task-states-learnstorybook.png)

`Task` is de basis component in onze app. Elke taak ziet er iets anders uit afhankelijk van de status waarin deze verkeert. We tonen een aan- of uitgevinkte checkbox, enige informatie over de taak en een “pin” knop, die mogelijk maakt de taak omhoog of omlaag door de lijst te verplaatsen. Alles bij elkaar hebben we deze properties nodig:

- `title` – een string die de taak beschrijft
- `state` - Op welke lijst staat de taak en is hij afgevinkt?

Om de `Task` te gaan bouwen beginnen we met de test statussen die overeen komen met de verschillende typen taken die in de figuur hierboven worden getoond. Vervolgens gebruiken we Storybook om de component in isolatie te bouwen waarbij we verzonnen data gebruiken. Gaandeweg testen we de component 'visueel' voor elke status.   

Dit proces lijkt zodanig op [Test-driven development](https://en.wikipedia.org/wiki/Test-driven_development) (TDD) dat we het "[Visual TDD](https://www.chromatic.com/blog/visual-test-driven-development)" noemen. 

## Voorbereiding

Ten eerste maken we de task component en het bijbehorende story bestand: `src/components/Task.vue` en `src/components/Task.stories.js`.

We beginnen bet de rudimentaire implementatie van de `Task`, waarbij we de atributen toevoegen waarvan we weten dat we ze nodig gaan hebben. 

```html
<!-- src/components/Task.vue -->

<template>
  <div class="list-item">
    <input type="text" readonly :value="task.title" />
  </div>
</template>

<script>
  export default {
    name: 'Task',
    props: {
      task: {
        type: Object,
        required: true,
        default: () => ({ id: '', state: '', title: '' }),
        validator: task => ['id', 'state', 'title'].every(key => key in task),
      },
    },
  };
</script>
```

In de code hierboven renderen we eenvoudige markup voor `Task` gebaseerd op de bestannde HTML structuur van de Todo app.

Hieronder bouwen we de drie test statussen in het story bestand:

```javascript
// src/components/Task.stories.js

import Task from './Task';
import { action } from '@storybook/addon-actions';

export default {
  title: 'Task',
  component: Task,
  // Our exports that end in "Data" are not stories.
  excludeStories: /.*Data$/,
};

export const actionsData = {
  onPinTask: action('pin-task'),
  onArchiveTask: action('archive-task'),
};

const Template = (args, { argTypes }) => ({
  components: { Task },
  props: Object.keys(argTypes),
  methods: actionsData,
  template: '<Task v-bind="$props" @pin-task="onPinTask" @archive-task="onArchiveTask" />',
});

export const Default = Template.bind({});
Default.args = {
  task: {
    id: '1',
    title: 'Test Task',
    state: 'TASK_INBOX',
    updatedAt: new Date(2018, 0, 1, 9, 0),
  },
};

export const Pinned = Template.bind({});
Pinned.args = {
  task: {
    ...Default.args.task,
    state: 'TASK_PINNED',
  },
};

export const Archived = Template.bind({});
Archived.args = {
  task: {
    ...Default.args.task,
    state: 'TASK_ARCHIVED',
  },
};
```

Er zijn twee basis niveau's van organisatie in Storybook: de component de de kind-stories. Je kunt elke story als een toestand of "permutatie" van de component zien. Je kunt zoveel stories toevoegen als je nodig hebt.  

- **Component**
  - Story
  - Story
  - Story

Om Storybook te vertellen over de component die we aan het documenteren zijn, maken we een `default` export die de volgende onderdelen bevat: 

- `component` -- de component zelf,
- `title` -- Naam van de component in het menu van de Storybook app,
- `excludeStories` -- informatie die nodig is voor de story maar die niet gerendered mag worden door de Storybook app.

Om de stories te definieren exporteren we een functie voor elke test status om een story te genereren. De story is een functie die een gerendered element oplevert (dit is een component class en set properties) in een bepaalde status---precies zoals een [Stateless Functional Component](https://vuejs.org/v2/guide/render-function.html#Functional-Components).

Omdat we meerdere permutaties hebben van onze component is het handig deze toe te wijzen aan een `Template` variablele. Daarmee introduceren we een standaard werkwijze in de stories die de hoeveelheid code die we moeten schrijven en onderhouden reduceert. 

<div class="aside">
💡 <code>Template.bind({})</code> is een <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind">standaard JavaScript</a> techniek om een kopie te maken van een functie. We gebruiken deze techniek om elke story zijn eigen properties te kunnen laten zetten maar gebruik te maken van dezelfde implementatie.
</div>

Door arguments of kortweg [`args`](https://storybook.js.org/docs/vue/writing-stories/args) kunnen we de component live wijzigen met de controls addon zonder datt we Storybook hoeven te herstarten. Zodra een [`args`](https://storybook.js.org/docs/vue/writing-stories/args) waarde wijzigt, verandert de component ook. 

Als we een story maken gebruiken we de basis `task` arg om de weergave van de task te maken die de component verwacht. Dit gebeurt typisch aan de hand van hoe de werkelijke data eruit ziet. En ook hier, door deze weergave te `export`eren kunnen we hem hergebruiken in andere stories, zoals we verderop zullen zien.

Met behulp van `action()` kunnen we een callback maken die in het `actions panel` van de Storybook UI verschijnen als er op geklikt wordt. Als we de pin button bouwen, kunnen vervolgens vaststellen in de test UI of de button click succesvol was. 

Omdat we dezelfde set acties nodig hebben in al onze permutaties van de component is het handig ze te bundelen in een `actionsData` variabele en deze steeds door te geven aan de story definitie (waar ze beschikbaar zijn via de `methods` property).
Een goede bijkomstigheid van het bundelen van acties die de component nodig heeft, is dat we deze kunnen `export`eren om ze te kunnen hergebruiken in stories die deze component hergebruiken. We zullen dit later zien.

<div class="aside">
💡 <a href="https://storybook.js.org/docs/vue/essentials/actions"><b>Actions</b></a> helpen om interacties te verifieren als je een UI component in isolatie bouwt. Vaak jheb je geen toegang tot de functies en state die je in de context van de app hebt. Gebruik <code>action()</code> om ze als stub binnen te halen.
</div>

## Configuratie

We moeten een paar zaken aanpassen in de Storybook configuratie zodat nieta alleen toegevoegde stories worden opgemerkt, maar dat we ook gebruik kunnen maken van het eerder geintrocuceerde CSS bestand in het [vorige hoofdstuk](/intro-to-storybook/vue/nl/get-started)

Begin met wijzigen van je Storybook configuratie bestand (`.storybook/main.js`) in het volgende:

```javascript
// .storybook/main.js

module.exports = {
  //👇 Location of our stories
  stories: ['../src/components/**/*.stories.js'],
  addons: ['@storybook/addon-links', '@storybook/addon-essentials'],
};
```

Pas na deze wijziging, in de `.storybook` directory, je `preview.js` aan naar het volgende:

```javascript
// .storybook/preview.js

import '../src/index.css'; //👈 The app's CSS file goes here

//👇 Configures Storybook to log the actions( onArchiveTask and onPinTask ) in the UI.
export const parameters = {
  actions: { argTypesRegex: '^on[A-Z].*' },
};
```

[`parameters`](https://storybook.js.org/docs/vue/writing-stories/parameters) worden typisch gebruikt om het gedrag van Storybook's `features` en `addons` te wijzigen. In ons geval gaan we ze gebruiken om te bepalen hoe de `actions` (mocked callbacks) worden afgehandeld.

Met behulp van `actions` kunnen we een callback maken die in het `actions panel` van de Storybook UI verschijnen als er op geklikt wordt. Als we de pin button bouwen, kunnen vervolgens vaststellen in de test UI of de button click succesvol was. 

Hierna zal hetstarten van de Storybook server de testcases opleveren voor de drie Task statussen:

<video autoPlay muted playsInline controls >
  <source
    src="/intro-to-storybook//inprogress-task-states.mp4"
    type="video/mp4"
  />
</video>

## Opbouwen van de statussen

Nu de installatie van Storybook en de testcases klaar zijn, kunnen we de HTML van de component aan gaan passen om te kloppen met het ontwerp.

De component is nogal rudimentair op dit moment. We gaan een aantal aanpassingen doorvoeren zodat het overeenkomt met het ontwerp, zonder te veel uitleg te geven over de details:

```html
<!-- src/components/Task.vue -->

<template>
  <div class="list-item" :class="task.state">
    <label class="checkbox">
      <input type="checkbox" :checked="isChecked" disabled name="checked" />
      <span class="checkbox-custom" @click="$emit('archive-task', task.id)" />
    </label>
    <div class="title">
      <input type="text" :value="task.title" readonly placeholder="Input title" />
    </div>

    <div class="actions">
      <a v-if="!isChecked" @click="$emit('pin-task', task.id)">
        <span class="icon-star" />
      </a>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'Task',
    props: {
      task: {
        type: Object,
        required: true,
        default: () => ({ id: '', state: '', title: '' }),
        validator: task => ['id', 'state', 'title'].every(key => key in task),
      },
    },
    computed: {
      isChecked() {
        return this.task.state === 'TASK_ARCHIVED';
      },
    },
  };
</script>
```

De toegevoegde markup gecombineerd met de CSS levert de volgende UI op:

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/finished-task-states.mp4"
    type="video/mp4"
  />
</video>

## Component built!

We hebben sucessvol de component opgebouwd zonder een server nodig te hebben of onze frontend applicatie op te starten. De volgende stap is om de overige Taskbox componenten een voor een op te bouwen op dezelfde manier. 

Zoals je ziet is beginnen met bouwen van componenten in isolatie eenvoudig en snel. We kunnen hoge kwaliteit, mooie UI met minder bugseenvoudig en snel realiseren omdat we elke mogelijke status kunnen testen.  

## Automated Testing

Storybook gaf ons een geweldige mogelijkheid om tijdens de bouw de applicatie visueel te testen. De stories zorgen dat we onze applicatie niet visueel brekenb tijdens doorontwikkelen van de app. Echter dit is een volledig handmatig proces op dit moment. Iemand moet alle tests handmatig af om zeker te weten dat niets gebroken is. Kan dit ook automatisch?  

### Snapshot testing

Snapshot testing betekent opnemen van een `known-good` output bij een gegeven input en vervolgens de component als `fout` markeren als die output in de toekomst wijzigt. Dit is een aanvulling op Storybook, omdat Storybook een snelle manier is om een nieuwe versie van een component te bekijken en wijzigingen te tonen. 

<div class="aside">
💡 Zorg dat de componenten data renderen die niet wijzigt, zodat de snapshot test niet steeds fout gaat. Kijk uit voor datums of random gegenereerde waarden.
</div>

Met de [Storyshots addon](https://github.com/storybooks/storybook/tree/master/addons/storyshots) wordt een snapshot gecreert voor elke story. Maak hiervoor de volgende wijziging in de afhankelijkheden:

```bash
yarn add -D @storybook/addon-storyshots jest-vue-preprocessor
```

Creer vervolgens het bestand `tests/unit/storybook.spec.js` met de volgende inhoud:

```javascript
// tests/unit/storybook.spec.js

import initStoryshots from '@storybook/addon-storyshots';

initStoryshots();
```

We moeten het volgende toevoegen in `jest.config.js`:

```js
  // jest.config.js

  transformIgnorePatterns: ["/node_modules/(?!(@storybook/.*\\.vue$))"],
```

Nadat dit gedaan is, kun je `yarn test:unit` starten en zie je de volgende output: 

![Task test runner](/intro-to-storybook/task-testrunner.png)

We hebben nu een snapshot test voor elke `Task` story. Als we de implementatie van `Task` wijzigen krijgen we een waarschuwing om de wijziging te verifieren. 

<div class="aside">
💡 Vergeet niet je wijzigingen te comitten in git!
</div>
