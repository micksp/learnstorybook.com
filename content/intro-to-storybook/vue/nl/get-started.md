---
title: 'Storybook voor Vue tutorial'
tocTitle: 'Beginnen'
description: 'Vue Storybook Installeren in je ontwikkelomgeving'
commit: '9e3165c'
---

Storybook draait naast je app tijdens ontwikkelen. De applicatie helpt je bij creëren van UI componenten geissoleert van de business logic en context van die applicatie. Deze editie van Learn Storybook is bedoeld voor Vue; er zijn andere edities voor [React](/intro-to-storybook/react/en/get-started), [React Native](/intro-to-storybook/react-native/en/get-started/), [Angular](/intro-to-storybook/angular/en/get-started), [Svelte](/intro-to-storybook/svelte/en/get-started) en [Ember](/intro-to-storybook/ember/en/get-started).

![Storybook en je app](/intro-to-storybook/storybook-relationship.jpg)

## Installeren Vue Storybook

We moeten een aantal stappen doorlopen om het build proces te installeren in onze omgeving. Om te beginnen willen we [degit](https://github.com/Rich-Harris/degit) gebruiken om het build proces te installeren. Via dit package kun je templates downloaden (deels gebouwde applicaties met wat standaard configuratie) om je te helpen snel op gang te komen met de ontwikkel workflow. 

Voer de volgende commando's uit:

```bash
# Clone de template
npx degit chromaui/intro-storybook-vue-template taskbox

cd taskbox

# Installeer afhankelijkheden
yarn
```

<div class="aside">
💡 Deze template bevat alle benodigde stijlen, assets, en basis benodigde configuraties voor deze versie van de tutorial.
</div>

Nu kunnen we controleren of de diverse omgevingen van onze applicatie goed werken:

```bash
# Start de test runner (Jest) in een terminal:
yarn test:unit

# Start de component explorer op poort 6006:
yarn storybook

# Start de frontend app op poort 8080:
yarn serve
```

Dit zijn onze drie aandachtsgebieden: geautomatiseerde test (Jest), component ontwikkeling (Storybook), en de app zelf.

![3 aandachtsgebieden](/intro-to-storybook/app-three-modalities-vue.png)

Afhankelijk van in welk deel van de applicatie je aan het werk bent zul je een of meer van deze aandachtsgebieden gelijktijdig willen starten. Omdat onze focus nu op het maken van een enkele UI component beperken we ons tot draaien van Storybook.

## Commit changes

Op dit punt is het een goed idee om de bestanden aan een locale repository toe te voegen. Draai de volgende commando's om de locale repository te initialiseren en de huidige wijzigingen te comitten.

```shell
$ git init
```

gevolgd door:

```shell
$ git add .
```

En tenslotte:

```shell
$ git commit -m "first commit"
```

We gaan beginnen met onze eerste component!