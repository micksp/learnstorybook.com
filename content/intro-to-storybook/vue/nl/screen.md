---
title: 'Een scherm maken'
tocTitle: 'Schermen'
description: 'Construeer een scherm met componenten'
commit: '99d3d65'
---

We hebben ons toegespitst op het maken van UI's van de grond af aan: klein beginnen en complexiteit toevoegen. Op die manier konden we componenten in isolatie bouwen, uitzoeken welke data nodig is en ermee spelen in Storybook. Allemaal zonder een server op te hoeven starten of schermen te bouwen!

In dit hoodfstuk gaan we hierop verderbouwen door componenten te combineren in een scherm and het scherm ontwikkelen in Storybook.

## Geneste containercomponenten

Omdat onze app erg eenvoudig is, is het scherm wat we gaan bouwen ook erg eenvoudig, door simpelweg de `TaskList` container component (die zijn eigen data krijgt via Vuex) in wat layout en een top-level error veld toevoegen vanuit de store (laten we ervan uitgaan dat we dat veld gebruiken als we een probleem hebben met de connectie naar de server). We starten door een presentationele component te maken `PureInboxScreen.vue` in de map `src/components`: 

```html
<!-- src/components/PureInboxScreen.vue -->

<template>
  <div>
    <div v-if="error" class="page lists-show">
      <div class="wrapper-message">
        <span class="icon-face-sad" />
        <div class="title-message">Oh no!</div>
        <div class="subtitle-message">Something went wrong</div>
      </div>
    </div>

    <div v-else class="page lists-show">
      <nav>
        <h1 class="title-page">
          <span class="title-wrapper">Taskbox</span>
        </h1>
      </nav>
      <TaskList />
    </div>
  </div>
</template>

<script>
  import TaskList from './TaskList.vue';
  export default {
    name: 'PureInboxScreen',
    components: { TaskList },
    props: {
      error: { type: Boolean, default: false },
    },
  };
</script>
```
Vervolgens maken we een container, die de data oppakt voor de `PureInboxScreen` in `src/components/InboxScreen.vue`:

```html
<!-- src/components/InboxScreen.vue -->

<template>
  <PureInboxScreen :error="error" />
</template>

<script>
  import PureInboxScreen from './PureInboxScreen';
  import { mapState } from 'vuex';

  export default {
    name: 'InboxScreen',
    components: { PureInboxScreen },
    computed: mapState(['error']),
  };
</script>
```
Vervolgens passen we dde `App` component aan om het `InboxScreen` the renderen (Uiteindelijk zullen we een router gebruiken om het juiste scherm te kiezen, maar daar maken we ons nu nog geen zorgen over):

```html
<!-- src/App.vue -->

<template>
  <div id="app">
    <InboxScreen />
  </div>
</template>

<script>
  import store from './store';
  import InboxScreen from './components/InboxScreen.vue';
  export default {
    name: 'app',
    store,
    components: {
      InboxScreen,
    },
  };
</script>

<style>
  @import './index.css';
</style>
```
Het wordt interessant als we de story gaan renderen in Storybook. 

Zoals we eerder zagen is de `TaskList` component is een **container** die de `PureTaskList` presentationele component. 
Container componenten kunnen per definitie niet eenvoudigweg gerendered worden in isolatie; ze verwachten een context te ontvangen of aan een servce gekoppled te worden. Dat betekent dat we de service of context die nodig is moeten mocken (d.w.z. een doe-alsof versie aanleveren) om de container in Storybook te renderen.

Toen we de `TaskList` in Storybook plaatsten konden we dit onderwerp vermijden door de `PureTaskList` te renderen en de container te vermijden. Iets dergelijks gaan we nu ook doen door het `PureInboxScreen`in Storybook te renderen. 
 Voor de `PureInboxScreen` is dat echter wat moeilijker, omdat de `PureInboxScreen` zelf welliswaar presentationeel is maar de `TaskList` niet. In zekere zin is het `PureInboxScreen` vervuild door er een container aan toe te voegen. Als we dus de sories gaan instellen in `src/components/PureInboxScreen.stories.js`: 

```javascript
// src/components/PureInboxScreen.stories.js

import PureInboxScreen from './PureInboxScreen.vue';

export default {
  title: 'PureInboxScreen',
  component: PureInboxScreen,
};

const Template = (args, { argTypes }) => ({
  components: { PureInboxScreen },
  props: Object.keys(argTypes),
  template: '<PureInboxScreen v-bind="$props" />',
});

export const Default = Template.bind({});

export const Error = Template.bind({});
Error.args = { error: true };
```
Hoewel de `error` story prima werkt zien we dat we een probleem hebben in de `default` story, omdat de `TaskList` geen Vuex store heeft om verbinding mee te maken. (Je zou ook soortgelijke problemen tegenkomen wanneer je probeert de `PureInboxScreen` te testen met een unit test).

![inbox is kapot](/intro-to-storybook/broken-inboxscreen-vue.png)

Een manier om dit probleem te omzeilen, is om nooit container componenten waar dan ook in je app te renderen, behalve op het hoogste niveau, en in plaats daarvan alle data behoefte door te geven in de componenthiërarchie.

Developers **zullen** echter onvermijdelijk containers verder naar beneden in de componenthiërarchie moeten renderen. Als we de app grotendeels of geheel in Storybook willen renderen (en dat doen we!) hebben we een oplossing voor dit probleem nodig.

<div class="aside">
💡 Nota bene, data doorgeven in de hiërarchie is een correcte aanpak, met name als je  <a href="http://graphql.org/">GraphQL</a> gebruikt. Dit is hoe we <a href="https://www.chromatic.com">Chromatic</a> hebben gebouwd, samen met meer dan 800 stories.
</div>

## Stories voorzien van context

Het goede nieuws is dat we eenvoudig een Vuex store kunnen aanleveren aan het `PureInboxScreen` in een story! we kunnen een nieuwe store aanmaken in ons story bestand en dat als context doorsturen aan de story:

```javascript
// src/components/PureInboxScreen.stories.js

import Vue from 'vue';
import Vuex from 'vuex';
import PureInboxScreen from './PureInboxScreen.vue';
import { action } from '@storybook/addon-actions';
import * as TaskListStories from './PureTaskList.stories';

Vue.use(Vuex);

export const store = new Vuex.Store({
  state: {
    tasks: TaskListStories.Default.args.tasks,
  },
  actions: {
    pinTask(context, id) {
      action('pin-task')(id);
    },
    archiveTask(context, id) {
      action('archive-task')(id);
    },
  },
});

export default {
  title: 'PureInboxScreen',
  component: PureInboxScreen,
  excludeStories: /.*store$/,
};

const Template = (args, { argTypes }) => ({
  components: { PureInboxScreen },
  props: Object.keys(argTypes),
  template: '<PureInboxScreen v-bind="$props" />',
  store,
});

export const Default = Template.bind({});

export const Error = Template.bind({});
Error.args = { error: true };
```
Mocked context verzorgen voor ander libraries kan vergelijkbaar, bijvoorbeeld voor [Apollo](https://www.npmjs.com/package/apollo-storybook-decorator), [Relay](https://github.com/orta/react-storybooks-relay-container) en anderen.

Door de statussen in Storybook te bekijken kunnen we eenvoudig testen of we het goed gedaan hebben: 

<video autoPlay muted playsInline loop >

  <source
    src="/intro-to-storybook/finished-inboxscreen-states.mp4"
    type="video/mp4"
  />
</video>

## Component-Driven Development

We zijn van onderaf begonnen met `Task` en zijn vervolgens overgegaan naar `TaskList`, nu hebben we een UI voor een volledig scherm. Ons `InboxScreen` biedt plaats aan een geneste containercomponent en bevat bijhorende stories.

<video autoPlay muted playsInline loop style="width:480px; height:auto; margin: 0 auto;">
  <source
    src="/intro-to-storybook/component-driven-development-optimized.mp4"
    type="video/mp4"
  />
</video>

[**Component-Driven Development**](https://www.componentdriven.org/) stelt je in staat om de complexiteit geleidelijk uit te breiden naarmate je hoger komt in de componenthiërarchie. De voordelen zijn onder andere een meer doelgericht ontwikkelproces en een grotere dekking van alle mogelijke UI-permutaties. Kortom, CDD helpt je bij het bouwen van kwalitatief betere en complexere UI's.

We zijn nog niet klaar - het werk is nog niet gedaan wanneer de UI is gebouwd. We moeten er ook voor zorgen dat het na verloop van tijd stabiel blijft.

<div class="aside">
💡 Vergeet niet je wijzigingen te comitten in git!
</div>
