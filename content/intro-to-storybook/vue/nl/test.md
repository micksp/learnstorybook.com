---
title: 'Testen UI componenten'
tocTitle: 'Testen'
description: 'Manieren om UI componenten te testen'
commit: 8bf107e
---

Geen Storybook tutorial zou compleet zijn zonder testen. Testen is essentieel voor het maken van UI's van hoge kwaliteit. In modulaire systemen kunnen kleine tweaks leiden tot grote regressies. Tot nu toe zijn we drie soorten tests tegengekomen:

- **Manuele tests** vertrouwen erop dat developers handmatig naar een component kijken om te controleren of deze correct is. Ze helpen ons om het uiterlijk van een component tijdens het bouwen te controleren.
- **Snapshot tests** met Storyshots leggen de gerenderde markup van een component vast. Ze helpen ons op de hoogte te blijven van markup-wijzigingen die renderfouten en waarschuwingen veroorzaken.
- **Unit tests** met Jest controleren of de output van een component hetzelfde blijft gegeven een vaste input. Ze zijn bedoeld voor het testen van de functionele kwaliteiten van een component.

## “Maar ziet het er goed uit?”

Helaas zijn de bovengenoemde testmethoden alleen niet voldoende om UI-bugs te voorkomen. UI's zijn lastig te testen omdat design subjectief en genuanceerd is. Visuele tests zijn te handmatig, snapshot tests veroorzaken te veel valse positieven bij gebruik voor UI en unit tests op pixelniveau zijn van beperkte waarde. Een complete Storybook-teststrategie omvat ook visuele regressie tests.

## Visuele tests voor Storybook

Visuele regressie tests, ook visuele tests genoemd, zijn ontworpen om veranderingen in het uiterlijk te ontdekken. Ze werken door screenshots van elke story te nemen en ze per commit te vergelijken om eventuele veranderingen te ontdekken. Dit werkt goed voor het verifiëren van grafische elementen zoals lay-out, kleur, grootte en contrast.

<video autoPlay muted playsInline loop style="width:480px; margin: 0 auto;">
  <source
    src="/intro-to-storybook/visual-regression-testing.mp4"
    type="video/mp4"
  />
</video>

Storybook is een fantastisch hulpmiddel voor visuele regressie tests, omdat elke story in wezen een testspecificatie is. Bij elke keer dat we een story schrijven of bijwerken, krijgen we gratis een specificatie!

Er zijn een aantal hulpmiddelen voor visuele regressie tests. Voor professionele teams raden we [**Chromatic**](https://www.chromatic.com/) aan, een add-on gemaakt door de ontwikkelaars van Storybook die tests in de cloud uitvoert. Dit biedt ook de mogelijkheid om het Storybook online te publiceren zoals we gezien hebben in het [vorige hoofdstuk](/intro-to-storybook/vue/nl/deploy/).

## Een UI wijziging opmerken

Visuele regressie tests werken door het vergelijken van beelden van nieuwe gerenderde UI code en de baseline beelden. Er verschijnt een waarschuwing als een UI verandering opgemerkt wordt.

Door de achtergrond van de `Task` component aan te passen kunnen we zien hoe dit werkt.

Begin bij het aanmaken van een nieuwe branch voor deze verandering:

```bash
git checkout -b change-task-background
```

Wijzig de `Task` component naar het volgende:

```html
<!-- src/components/Task.vue -->

<input
  type="text"
  :value="task.title"
  readonly
  placeholder="Input title"
  style="background: red;"
/>
```

Dit levert een nieuwe achtergrondkleur op voor een item.

![task background change](/intro-to-storybook/chromatic-task-change.png)

Voeg het bestand toe aan Git:

```bash
git add .
```

Commit de wijziging:

```bash
git commit -m "change task background to red"
```

En push de wijziging naar de repository:

```bash
git push -u origin change-task-background
```

Open tenslotte je Github repository en maak een pull request voor de `change-task-background` branch.

![Een PR in GitHub aanmaken voor task](/github/pull-request-background.png)

Voeg een beschrijvende tekst toe aan je pull request en klik `Create pull request`. Klik op de "🟡 UI Tests" PR check onderaan de pagina.

![Created a PR in GitHub for task](/github/pull-request-background-ok.png)

Dit laat je zien hoe de UI wijzigingen door jouw commit zijn opgemerkt.

![Wijziging opgemerkt door Chromatic](/intro-to-storybook/chromatic-catch-changes.png)

Er zijn veel wijzigingen! De component hiërarchie waar `Task` een kind is van `TaskList` en `Inbox` betekent dat één kleine verandering een sneeuwbaleffect kan teweegbrengen toresulterend in grote regressies. Om deze reden hebben ontwikkelaars visuele regressietests nodig als aanvulling op andere testmethodes.

![UI minor tweaks major regressions](/intro-to-storybook/minor-major-regressions.gif)

## Wijzigingen controleren

Visuele regressie tests zorgen ervoor dat componenten niet per ongeluk veranderen. Maar het is nog steeds aan ons om te bepalen of veranderingen opzettelijk zijn of niet.

Als een wijziging opzettelijk is, moeten we de _baseline_ bijwerken zodat toekomstige tests worden vergeleken met de nieuwste versie van de story. Als een wijziging onbedoeld is, moet deze worden gecorrigeerd.

<video autoPlay muted playsInline loop style="width:480px; margin: 0 auto;">
  <source
    src="/intro-to-storybook/website-workflow-review-merge-optimized.mp4"
    type="video/mp4"
  />
</video>

Aangezien moderne apps zijn opgebouwd uit componenten, is het belangrijk dat we op componentniveau testen. Hierdoor kunnen we de oorzaak van een verandering, de component, identificeren in plaats van te reageren op symptomen van een verandering, de schermen en samengestelde componenten.

## Wijzigingen mergen

Wanneer we klaar zijn met controleren, kunnen we de UI-wijzigingen met vertrouwen mergen - wetende dat updates niet per ongeluk bugs zullen introduceren. Als de nieuwe `red` achtergrond correct is, accepteer dan de wijzigingen. Zo niet, draai de wijziging dan terug naar de vorige versie.

![Wijzigingen klaar om gemerged te worden](/intro-to-storybook/chromatic-review-finished.png)

Storybook helpt ons componenten **te bouwen**; testen helpt ons ze **te onderhouden**. De vier soorten UI-tests die in deze tutorial zijn behandeld, zijn visuele-, snapshot-, unit- en visuele regressie tests. De laatste drie kunnen geautomatiseerd worden door ze aan een CI toe te voegen, zoals we dit net hebben opgezet. Dit helpt ons componenten te shippen zonder je zorgen te hoeven maken over verborgen bugs. Deze illustratie toont hele workflow:

![Visuele regressie testing workflow](/intro-to-storybook/cdd-review-workflow.png)
