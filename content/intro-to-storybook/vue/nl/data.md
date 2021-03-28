---
title: 'Data toevoegen'
tocTitle: 'Data'
description: 'Data toevoegen aan je UI component'
commit: 'fa1c954'
---

Tot nu toe hebben we geisoleerde stateless componenten gemaakt -perfect voor Storybook, maar uiteindelijk niet bruikbaar tot we ze wat data geven in onze app.

In deze tutorial focussen we niet op het maken van een app, dus we gaan daar verder niet op in. Maar we nemen even een moment om een gebruikelijke methode te bekijken om data naar toe te voegen aan container componenten.

## Container componenten

Zoals onze `TaskList` component nu is geschreven is die "presentational" (zie [deze blog post](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)) wat wil zeggen dat deze niet praat met iets extern behalve zijn eigen implementatie. Om data erin te krijgen, hebben we een "container" nodig.

Dit voorbeeld gebruikt [Vuex](https://vuex.vuejs.org), Vue's standaard datamanagement bibliotheken, om een eenvoudig datamodel voor onze app te maken. Deze methode is echter net zo goed toepasbaar bij andere datamanagement bibliotheken zoals [Apollo](https://www.apollographql.com/client/) en [MobX](https://mobx.js.org/).

Installeer vuex met behulp van dit commando:

```bash
yarn add vuex
```

We gaan een standaard vuex store maken in een bestand `src/store.js` die reageert op actions die de state van de tasks veranderen:

```javascript
// src/store.js

import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
    tasks: [
      { id: '1', title: 'Something', state: 'TASK_INBOX' },
      { id: '2', title: 'Something more', state: 'TASK_INBOX' },
      { id: '3', title: 'Something else', state: 'TASK_INBOX' },
      { id: '4', title: 'Something again', state: 'TASK_INBOX' },
    ],
  },
  mutations: {
    ARCHIVE_TASK(state, id) {
      state.tasks.find(task => task.id === id).state = 'TASK_ARCHIVED';
    },
    PIN_TASK(state, id) {
      state.tasks.find(task => task.id === id).state = 'TASK_PINNED';
    },
  },
  actions: {
    archiveTask({ commit }, id) {
      commit('ARCHIVE_TASK', id);
    },
    pinTask({ commit }, id) {
      commit('PIN_TASK', id);
    },
  },
});
```

We kunnen de store vrij eenvoudig toevoegen aan de component hiërarchie in de top-level app component (`src/App.vue`):

```html
<!--src/App.vue -->

<template>
  <div id="app">
    <task-list />
  </div>
</template>

<script>
  import store from './store';
  import TaskList from './components/TaskList.vue';

  export default {
    name: 'app',
    store,
    components: {
      TaskList,
    },
  };
</script>
<style>
  @import './index.css';
</style>
```

Vervolgens laten we onze `TaskList` de data uit de store lezen. Eerst verplaatsen we onze presentationele versie naar een bestand `src/components/PureTaskList.vue` (hernoemen van de component naar `PureTaskList`), en plaatsen die in een container.

In `src/components/PureTaskList.vue`:

```html
<!-- src/components/PureTaskList.vue -->

<template>
  <!-- same content as before -->
</template>

<script>
  import Task from './Task';
  export default {
    name: 'PureTaskList',
    // same content as before
  };
</script>
```

In `src/components/TaskList.vue`:

```html
<!-- src/components/TaskList.vue -->

<template>
  <PureTaskList :tasks="tasks" v-on="$listeners" @archive-task="archiveTask" @pin-task="pinTask" />
</template>

<script>
  import PureTaskList from './PureTaskList';
  import { mapState, mapActions } from 'vuex';

  export default {
    components: { PureTaskList },

    methods: mapActions(['archiveTask', 'pinTask']),

    computed: mapState(['tasks']),
  };
</script>
```

De reden om de poresentationele versie van de `TaskList` apart te houden is dat testen en isoleren eenvoudiger is. Omdat de component niet afhankelijk is van de store, is het eenvoudiger te verwerken vanuit een test perspectief. We hernoemen `src/components/TaskList.stories.js` naar `src/components/PureTaskList.stories.js` en zorgen dat de stories de presentationele versie gebruiken:

```javascript
// src/components/PureTaskList.stories.js

import PureTaskList from './PureTaskList';
import * as TaskStories from './Task.stories';

export default {
  component: PureTaskList,
  title: 'PureTaskList',
  decorators: [() => '<div style="padding: 3rem;"><story /></div>'],
};

const Template = (args, { argTypes }) => ({
  components: { PureTaskList },
  props: Object.keys(argTypes),
  // We are reusing our actions from task.stories.js
  methods: TaskStories.actionsData,
  template: '<PureTaskList v-bind="$props" @pin-task="onPinTask" @archive-task="onArchiveTask" />',
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

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/finished-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

<div class="aside">

</div>

Op dezelfde manier moeten we `PureTaskList` gebruiken in de Jest test:

```js
// tests/unit/PureTaskList.spec.js

import Vue from 'vue';
import PureTaskList from '../../src/components/PureTaskList.vue';
//👇 Our story imported here
import { WithPinnedTasks } from '../../src/components/PureTaskList.stories';

it('renders pinned tasks at the start of the list', () => {
  // render PureTaskList
  const Constructor = Vue.extend(PureTaskList);
  const vm = new Constructor({
    //👇 Story's args used with our test
    propsData: WithPinnedTasks.args,
  }).$mount();
  const firstTaskPinned = vm.$el.querySelector('.list-item:nth-child(1).TASK_PINNED');

  // We expect the pinned task to be rendered first, not at the end
  expect(firstTaskPinned).not.toBe(null);
});
```

<div class="aside">
💡 Door deze wijziging hebben de snapshots een update nodig. Start het test commando met de <code>-u</code> vlag om ze te updaten. En natuurlijk: vergeet je wijzigingen te comitten in git!
</div>
