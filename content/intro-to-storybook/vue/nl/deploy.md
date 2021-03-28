---
title: 'Deploy Storybook'
tocTitle: 'Deploy'
description: 'Storybook online deployen'
commit: '107b7ce'
---

In deze tutorial hebben we componenten lokaal op onze development machine gebouwd. Er komt een punt waar we ons werk moeten delen om feedback van het team te krijgen. We gaan Storybook online deployen om te zorgen dat het team de UI implementatie kan reviewen.

## Exporteren als static app

Om Storybook te deployen, moeten we het eerst exporteren als een statische web-app. Deze functionaliteit is ingebouwd in Storybook en al geconfigureerd.

Door `yarn build-storybook` uit te voeren zal een statische versie van het Storybook worden gemaakt in de directory `storybook-static`. Deze kan vervolgens naar elke static site hosting service worden gedeployed.

## Storybook publiceren

In deze tutorial gebruiken we [Chromatic](https://www.chromatic.com/), een gratis publiceerservice die door de developers van Storybook is gemaakt. Hiermee kunnen we ons Storybook veilig in de cloud publiceren en hosten.

### Setup a repository in GitHub

Voor we beginnen moet onze code in sync zijn met een remote versie control systeem. Toen we het project startten in het [hoofdstuk 'beginnen'](/intro-to-storybook/vue/nl/get-started) hebben we een locale repository gemaakt. Op dit punt hebben we al een aantal commits in die repositories gemaakt die we kunnen pushen naar de remore repository.

Ga naar GitHub en maak [hier](https://github.com/new) een nieuwe repository voor ons project. Noem de repository "taskbox", hetzelfde als onze locale project.

![GitHub setup](/intro-to-storybook/github-create-taskbox.png)

Copieer in de nieuwe repo het `origin` url en voeg die toe aan het locale git project met dit commando:

```bash
$ git remote add origin https://github.com/<je gebruikersnaam>/taskbox.git
```

Push de locale repository tenslotte naar GitHub:

```bash
$ git push -u origin main
```

### Chromatic toevoegen

Voeg het chromatic package toe als development afhankelijkheid.

```bash
yarn add -D chromatic
```

[Login bij Chromatic](https://www.chromatic.com/start) met je GitHub account na installatie van het package (Chromatic heeft maar weinig rechten nodig).
Maak vervolgens een nieuw project met de naam "taskbox" en synchronyseer dat met de GithHub repository die we zojuist aangemaakt hebben.

Klik op `Choose GitHub repo` onder `collaborators` en selecteer je repository.

<video autoPlay muted playsInline loop style="width:520px; margin: 0 auto;">
  <source
    src="/intro-to-storybook/chromatic-setup-learnstorybook.mp4"
    type="video/mp4"
  />
</video>

Kopieer het unieke `project-token` dat gegenereerd is voor je project. Voer nu het volgende commando uit in de commandline om het Storybook te builden en deployen. Vervang `project-token` met je eigen project-token.

```bash
yarn chromatic --project-token=<project-token>
```

![Chromatic running](/intro-to-storybook/chromatic-manual-storybook-console-log.png)

Als het commando klaar is, ontvang je een link waar je storybook gepubliceerd is. Deel de link met je team om feedback te krijgen.

![Storybook deployed with chromatic package](/intro-to-storybook/chromatic-manual-storybook-deploy-6-0.png)

We hebben Storybook nu gepubliceerd met 1 commando. Maar het is lastig om steeds dit commando uit te voeren. Het is beter de laatste versie van de code te publiceren zodra we code pushen naar de remote repository. We hebben een `CI/CD` oplossing nodig.

## Continuous deployment met Chromatic

Doordat het project in een GithHub repository is opgenomen kunnen we een continuous integration CI gebruiken om het Storybook automatisch te deployen. [GitHub Actions](https://github.com/features/actions) is een gratis service die beschikbaar is in GithHub en het eenvoudig maakt om automatisch te publiceren.

### Voeg een GitHub Action toe om Storybook te publiceren

Maak een nieuwe directory `.github` in de root directory van het project. Maak daarbinnen een nieuwe directory `workflows`. Maak een nieuw bestand `chromatic.yml` zoals hieronder weergegeven. Vervang het project-token met je eigen project-token.

```yaml
# .github/workflows/chromatic.yml

# Workflow name
name: 'Chromatic Deployment'

# Event for the workflow
on: push

# List of jobs
jobs:
  test:
    # Operating System
    runs-on: ubuntu-latest
    # Job steps
    steps:
      - uses: actions/checkout@v1
      - run: yarn
        #👇 Adds Chromatic as a step in the workflow
      - uses: chromaui/action@v1
        # Options required for Chromatic's GitHub Action
        with:
          #👇 Chromatic projectToken, see https://www.learnstorybook.com/intro-to-storybook/vue/en/deploy/ to obtain it
          projectToken: project-token
          token: ${{ secrets.GITHUB_TOKEN }}
```

<div class="aside"><p>💡 Om het kort te houden gebruiken we hier geen <a href="https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets">GitHub secrets</a>. `Secrets` zijn beveiligde environment variabelen waar GitHub in voorziet zodat je je <code>project-token</code> niet hardcoded hoeft op te nemen.</p></div>

### Commit de action

Voeg de wijzigingen toe aan de repository via de command line:

```bash
git add .
```

Commit als volgt:

```bash
git commit -m "GitHub action setup"
```

Push de wijziging naar de remote repository:

```bash
git push origin main
```

Vanaf nu wordt elke keer dat je pushed naar GithHub de wijziging doorgezet naar Chromatic. Je kunt alle gepubliveerde Storybooks vinden op het build scherm in Chromatic.

![Chromatic user dashboard](/intro-to-storybook/chromatic-user-dashboard.png)

Klik op te laatste build, die zou bovenaan moeten staan.

Klik vervolgens op de `View Storybook` knop om de laatste versie van het Storybook te zien.

![Storybook link on Chromatic](/intro-to-storybook/chromatic-build-storybook-link.png)

Gebruik de link om te delen met de teamleden. Dit werkt goed in het standaard app deployment proces of om over je werk op te scheppen 💅.
