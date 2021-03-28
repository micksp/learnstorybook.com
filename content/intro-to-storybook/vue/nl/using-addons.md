---
title: 'Addons'
tocTitle: 'Addons'
description: 'Leer de populaire Controls addon te integeren en te gebruiken'
commit: '45b6600'
---
Storybook heeft een robuust ecosysteem van [addons](https://storybook.js.org/docs/vue/configure/storybook-addons) die je kunt gebruiken om de developer experience voor iedereen in het team te te vergroten. [Hier](https://storybook.js.org/addons) staan ze allemaal. 

Als je deze hele tutorial hebt gevolgd ben je al meerdere addons tegen gekomen en heb je er een geinstalleerd in het hoofdstuk [Testen] (/intro-to-storybook/vue/nl/test/).

Er zijn addons voor elke denkbare use case. Het zou te lang duren om ze allemaal te beschrijven. We gaan een van de meest populaire integreren: [Controls](https://storybook.js.org/docs/vue/essentials/controls). 

## Wat is `Controls`?
Controls laat ontwerpers en ontwikkelaars eenvoudig het gedrag van een component onderzoeken door met de argumenten te _spelem_ zonder dat daar code voor nodig is. Controle maakt een addon sectie naast de stories zodat je de argumenten live kunt wijzigen. 

Nieuwe installaties van Storybook bevatten standaar de Controls addon zonder dat extra configuratie nodig is. 

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/controls-in-action.mp4"
    type="video/mp4"
  />
</video>

## Addons maken nieuwe Storybook workflows mogelijk
Storybook is een [component-driven development environment](https://www.componentdriven.org/). De Conrols addon maakt daar een interactieve documentatie tool van.

### Controls gebruiken om rand gevallen te vinden

Met Controls kunnen QA Engineers, UI Engineers en andere stakeholders de grenzen van de component opzoeken. Zie het volgende voorbeeld: hoe zou `Task` zich gedragen als we een **ENORME** string zouden toevoegen?

![De content rechts wordt afgebroken!](/intro-to-storybook/task-edge-case.png)

Precies, de tekst gaat verder na de de kaders van de `Task` component.

Met Controls kunnen we snel controleren hoe de component zich gedraagt bij verschillende inlut, in dit geval een lange string. Hiermee is er minder werk nodig om UI problemen te ontdekken.

We gaan het afbreek issue in de `Task` component oplossen door een stijl toe te voegen:

```html
<!-- src/components/Task.vue -->

<input
  type="text"
  :value="task.title"
  readonly
  placeholder="Input title"
  style="text-overflow: ellipsis;"
/>
```

![Dat ziet er beter uit.](/intro-to-storybook/edge-case-solved-with-controls.png)

Opgelost! De text wordt nu afgebroken als het bij de kaders komt en hangevuld met nette ellipsen. 

### Een nieuwe story toevoegen om regressies te voorkomen

In de toekomst kunnen we dit issue reproduceren door de sting opnieuw in te voeren via Controls. Maar het is eenvoudiger om een story te schrijven die deze randzaak toont. Hiermee wordt de test coverage vergroot en maakt de grens van de component expliciet voor de rest van het team.

Voeg een nieuwe story toe voor het lange string issue in `Task.stories.js`:

```js
// src/components/Task.stories.js

const longTitleString = `Deze taaknaam is absurd lang. Zo lang dat ie niet zal gaan passen in de toegestane ruimte. Wat zal er gebeuren? De tekst kan door de ster die een vastgezette taak vertegenwoordigt worden geschreven. Of hij kan afgebroken worden als ie bij de ster komt. Hopelijk niet!`;

export const LongTitle = Template.bind({});
LongTitle.args = {
  task: {
    ...Default.args.task,
    title: longTitleString,
  },
};
```

Nu kunnen we het probleem eenvoudig reproduceren en er aan werken. 

<video autoPlay muted playsInline loop>
  <source
    src="/intro-to-storybook/task-stories-long-title.mp4"
    type="video/mp4"
  />
</video>

Bij het [visueel testing](/intro-to-storybook/vue/nl/test/) zien we of de ellipsen oplossing blijft werken.
Zonder test coverage is het waarschijnlijker dat obscure randzaken over het hoofd worden gezien!

<div class="aside"><p>💡 Controls is prima om niet-ontwikkelaars de mogelijkheid te geven om te spelen met de componenten en stories, veel meer nog dan we hier gezien hebben here. Lees de <a href="https://storybook.js.org/docs/vue/essentials/controls">officiele documentatie</a> voor meer informatie. Er zijn nog veel meer mogelijkheden met addons om Storybook aan te sluiten bij je workflow.</div>

### Merge wijzigingen

Vergeet niet je wijzigingen te mergen in git!
