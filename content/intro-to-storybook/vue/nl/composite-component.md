---
title: 'een samengestelde component maken'
tocTitle: 'Samengestelde component'
description: 'Maak een samengestelde component met behulp van eenvoudige componenten'
commit: '98335e5'
---
In het vorige hoofdstuk hebben we onze eerste component gemaakt. In dit hoofdstuk bouwen we daar op voort om de TaskList component te maken, een lijst van Tasks componenten.
We voegen componenten samen en gaan zien wat er gebeurt als we meer complexiteit introduceren.

## Tasklist
Taskbox benadrukt vastgezette taken door ze boven de standaard taken te positioneren. Dit levert twee variaties van TaskList op waar we stories voor nodig hebben: standaard items en vastgezette (`pinned`) items. 

![default and pinned tasks](/intro-to-storybook/tasklist-states-1.png)

Omdat `Task` data asynchroon kan worden verstuurd hebben we **ook** een laad-status nodig als we geen verbinding hebben. Aanvullend hebben we een lege status nodig bij ontbreken van taken.

![empty and loading tasks](/intro-to-storybook/tasklist-states-2.png)

## Voorbereiding

Een samengesteld component is niet veel anders als de basis componenten die er deel van uitmaken. Maak een `TaskList` component en het bijbehorende story bestand: `src/components/TaskList.vue` and `src/components/TaskList.stories.js`.

Begin met een basis implementatie van de `TaskList`. Je moet de eerder gedefinieerde `Task` component daarvoor importeren en de attributen doorgeven als inputs.

```html
<!-- src/components/TaskList.vue -->

<template>
  <div class="list-items">
    <template v-if="loading">
      loading
    </template>
    <template v-else-if="isEmpty">
      empty
    </template>
    <template v-else>
      <Task v-for="task in tasks" :key="task.id" :task="task" v-on="$listeners" />
    </template>
  </div>
</template>

<script>
  import Task from './Task';
  export default {
    name: 'TaskList',
    components: { Task },
    props: {
      tasks: { type: Array, required: true, default: () => [] },
      loading: { type: Boolean, default: false },
    },
    computed: {
      isEmpty() {
        return this.tasks.length === 0;
      },
    },
  };
</script>
```
Vervolgens maak je de test statussen voor de `TaskList` in het story bestand.  

```javascript
// src/components/TaskList.stories.js

import TaskList from './TaskList';
import * as TaskStories from './Task.stories';

export default {
  component: TaskList,
  title: 'TaskList',
  decorators: [() => '<div style="padding: 3rem;"><story /></div>'],
};

const Template = (args, { argTypes }) => ({
  components: { TaskList },
  props: Object.keys(argTypes),
  // We are reusing our actions from task.stories.js
  methods: TaskStories.actionsData,
  template: '<TaskList v-bind="$props" @pin-task="onPinTask" @archive-task="onArchiveTask" />',
});

export const Default = Template.bind({});
Default.args = {
  // Shaping the stories through args composition.
  // The data was inherited from the Default story in task.stories.js.
  tasks: [
    { ...TaskStories.Default.args.task, id: '1', title: 'Task 1' },
    { ...TaskStories.Default.args.task, id: '2', title: 'Task 2' },
    { ...TaskStories.Default.args.task, id: '3', title: 'Task 3' },
    { ...TaskStories.Default.args.task, id: '4', title: 'Task 4' },
    { ...TaskStories.Default.args.task, id: '5', title: 'Task 5' },
    { ...TaskStories.Default.args.task, id: '6', title: 'Task 6' },
  ],
};

export const WithPinnedTasks = Template.bind({});
WithPinnedTasks.args = {
  // Shaping the stories through args composition.
  // Inherited data coming from the Default story.
  tasks: [
    ...Default.args.tasks.slice(0, 5),
    { id: '6', title: 'Task 6 (pinned)', state: 'TASK_PINNED' },
  ],
};

export const Loading = Template.bind({});
Loading.args = {
  tasks: [],
  loading: true,
};

export const Empty = Template.bind({});
Empty.args = {
  // Shaping the stories through args composition.
  // Inherited data coming from the Loading story.
  ...Loading.args,
  loading: false,
};
```

<div class="aside">
💡 <a href="https://storybook.js.org/docs/vue/writing-stories/decorators"><b>Decorators</b></a> zijn een manier om willekeurige wrapper aan een story toe te voegen. In dit geval gebruiken we een decorator key in de default export om stijling toe te voegen. Maar het kan ook gebruikt worden om een andere context in een component te definieren, zoals we later zullen zien.
</div>

Door `TaskStories` te importeren kunnen we de argumenten (afgekort tot args) [samenstellen](https://storybook.js.org/docs/vue/writing-stories/args#args-composition) in de stories men minimale inspanning. Op die manier zijn de data en actions (mocked callbacks) aanwezig voor beide componenten.

Zoek nu de nieuwe `TaslList` stories in Storybook.

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/inprogress-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

## Build out the states
Onze component is nog rudimentair maar we hebben een goed idee van de stories waar we naartoe werken. Mogelijk denk je dat de `.list-items` wrapper te eenvoudig is. Dat klopt - normaal gesproken zullen we geen nieuwe component toevoegen alleen om er een wrapper omheen te zetten. Maar de **werkelijke complexiteit** van de `TaskList` component wordt duidelijk in de edge cases `WithPinnedTasks`, `loading`, en `empty`.

```html
<!-- src/components/TaskList.vue -->

<template>
  <div class="list-items">
    <template v-if="loading">
      <div v-for="n in 6" :key="n" class="loading-item">
        <span class="glow-checkbox" />
        <span class="glow-text"> <span>Loading</span> <span>cool</span> <span>state</span> </span>
      </div>
    </template>

    <div v-else-if="isEmpty" class="list-items">
      <div class="wrapper-message">
        <span class="icon-check" />
        <div class="title-message">You have no tasks</div>
        <div class="subtitle-message">Sit back and relax</div>
      </div>
    </div>

    <template v-else>
      <Task v-for="task in tasksInOrder" :key="task.id" :task="task" v-on="$listeners" />
    </template>
  </div>
</template>

<script>
  import Task from './Task';
  export default {
    name: 'TaskList',
    components: { Task },
    props: {
      tasks: { type: Array, required: true, default: () => [] },
      loading: { type: Boolean, default: false },
    },
    computed: {
      tasksInOrder() {
        return [
          ...this.tasks.filter(t => t.state === 'TASK_PINNED'),
          ...this.tasks.filter(t => t.state !== 'TASK_PINNED'),
        ];
      },
      isEmpty() {
        return this.tasks.length === 0;
      },
    },
  };
</script>
```
De toegevoegde markup resulteert in de volgende UI:

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/finished-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

Let op de positie van het vastgezette item in de lijst. We willen het vastgezette item bovenin de lijst om er een prioriteit aan te geven voor de eindgebruiker.

## Automated testing

In het vorige hoofdstuk hebben we geleerd een snapshot te maken van de teststories door Storyshots te gebruiken. met `Tasks` was er niet veel complexiteit om te testen of de component correct gerendered wordt. Omdat `TaskList een extra laag aan complexiteit toevoegt willen we controleren dat bepaalde input bepaalde output tot gevolg heeft zodanig dat automatisch testen mogelijk is. Om dit te bereiken maken we een unittest die gekoppeld is aan test renderer met [Jest](https://facebook.github.io/jest/).

![Jest logo](/intro-to-storybook/logo-jest.png)

### Unit tests with Jest

Met behulp van Storybook stories gecombineerd met handmatige visuele tests en snapshot tests (zie hierboven) komen we al ver om bugs te voorkomen. Als componenten een grote verscheidenheid aan use cases beslaan en we gebruiken gereedschap dat menselijke controle verzekert bij elke wijziging van de story, zullen bugs minder waarschijnlijk zijn. 

Soms zit het venijn echter in de staart. In die gevallen is een testframework nodig dat speciaal voor dat doel ontwikkeld is. Wat ons brengt bij Unittests.

In ons geval willen we `TaskList` renderen met vastgezette taken **voor** niet-vastgezette taken die in de `tasks` property worden doorgegeven. Hoewel we een story hebben die precies dat test (`WithPinnedTasks`) is het moeilijker voor een menselijke controle om te zien dat de lijst **gestopt** is met correct sorteren en dat we dus een bug ontdekt hebben. Het ziet er niet direct uit als **Fout!** voor de vluchtige blik. 

Om dit probleem te voorkomen kunnen we Jest gebruiken om een Story te renderen naar de DOM en wat DOM queries uit te voeren om afwijkende elementen in de output te ontdekken. Het mooie van het story formaat is dat we de story kunnen importeren in de testdefinitie om het vervolgens daar te renderen! 

Maak een bestand met de naam `tests/unit/TaskList.spec.js`. Hierin worden de tests en assertions over de output gedefinieerd.  

```javascript
// tests/unit/TaskList.spec.js

import Vue from 'vue';
import TaskList from '../../src/components/TaskList.vue';
//👇 Our story imported here
import { WithPinnedTasks } from '../../src/components/TaskList.stories';

it('renders pinned tasks at the start of the list', () => {
  // render Tasklist
  const Constructor = Vue.extend(TaskList);
  const vm = new Constructor({
    //👇 Story's args used with our test
    propsData: WithPinnedTasks.args,
  }).$mount();
  const firstTaskPinned = vm.$el.querySelector('.list-item:nth-child(1).TASK_PINNED');

  // We expect the pinned task to be rendered first, not at the end
  expect(firstTaskPinned).not.toBe(null);
});
```

![TaskList test runner](/intro-to-storybook/tasklist-testrunner.png)

Merk op dat we de `withPinnedTasksData` takenlijst konden hergebruiken in zowel de story als de unittest. Op deze manier kunnen we de bestaande code (de voorbeelden die interessante configuraties van de component representeren) blijven gebruiken op meer en meer manieren.

Merk ook op dat deze test nogal fragile is. Het is mogelijk dat het project volwassen wordt en dat de precieze implementatie van `Task` wijzigt -- misschien een andere Classnaam -- waardoor de test zal breken en moet worden bijgewerkt. Dit is niet een probleem maar eerder een waarschuwing om voorzichtig te zijn met vrijlijk gebruik van unittests voor UI. Zijn zijn niet eenvoudig te onderhouden. Het is goed om visueel, snapshot en visueel regressie te  (zie [testing chapter](/intro-to-storybook/vue/en/test/) gebruiken waar mogelijk. 

<div class="aside">
💡 Vergeet niet je wijzigingen te committen in git!
</div>
