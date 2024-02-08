<!--
SPDX-FileCopyrightText: NOI Techpark <digital@noi.bz.it>

SPDX-License-Identifier: CC0-1.0
-->

# Flightdata realtime

[![REUSE Compliance](https://github.com/noi-techpark/webcomp-flightdata-realtime/actions/workflows/reuse.yml/badge.svg)](https://github.com/noi-techpark/odh-docs/wiki/REUSE#badges)
[![REUSE status](https://api.reuse.software/badge/github.com/noi-techpark/webcomp-flightdata-realtime)](https://api.reuse.software/info/github.com/noi-techpark/webcomp-flightdata-realtime)

A search engine for search and consult the realtime flights data around a specific given airport. A detailed map display the single flight status.

## Table of contents

- [Usage](#usage)
- [Gettings started](#getting-started)
- [Tests and linting](#tests-and-linting)
- [Deployment](#deployment)
- [Docker environment](#docker-environment)
- [Information](#information)

## Usage

Include vue.js (https://unpkg.com/vue) and the webcompscript file `dist/noi-flightdata-realitime.min.js` in your HTML and define the web component like this:

```html
<noi-flightdata-realitime 
    theme="OpenDataHub"
    showTopDescription="true"
    langPack="{}"
    styleUrl="./demo-assets/custom.css"
    id="atc-main"
    language="en"
    localairport="BZO"
    timezone="Europe/Berlin">
</noi-flightdata-realitime>
```
For a full example, configuration and available custom properties please look at [the demo](demo.html).

### Attributes

#### language

The language in which the component is rendered

Type: string<br>
Options: "en", "it", "de"

Translations can be changed or added via the langPack attribute: [the demo](demo.html)


#### localairport

The attribute localairport can be passed as a airport IATA code which defines the local airports from where flights depart or arrive.

Type: string<br>
Examples: "BZO", "BER", "VRN"


#### showTopDescription

Show or not the top inner component description. Default is "true".

Type: Boolean<br>
Options: "true", "false"


#### theme

This attribute can be used to apply a predefined set of colors.

Type: string<br>
Options: "OpenDataHub", "Skyalps"


#### styleUrl

You can specify a url from an external stylesheet (css). This stylesheet will then be appended to the shadowroot and thus allows overriding most css rules.
If this option is used, an unique "id" must be assigned to the element.

The configurations can by seen at: [the demo](demo.html)


#### timezone

The attribute timezone can be passed as a timezone standard. Full list can be found here https://www.npmjs.com/package/tzdata.

Type: string<br>
Defalut: "Europe/Rome"


#### colors

Main webcomponent colors. A full customization in colors can be made by using a custom css via styleUrl attribute.

Type: Object<br>
Default:
```
{
    --noi-primary: white;
    --noi-mid: #f5f5f5;
    --noi-secondary: black;
    --noi-text-primary: black;
    --noi-text-secondary: white;
    --noi-light-bg: #F5F4F6;
    --noi-font-family: 'Barlow Semi Condensed';
    --basic-font-size: 16px;
}
```


## Getting started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes.

### Prerequisites

To build the project, the following prerequisites must be met:

- Node 18 / NPM 8

For a ready to use Docker environment with all prerequisites already installed and prepared, you can check out the [Docker environment](#docker-environment) section.

### Source code

Get a copy of the repository:

```bash
git clone https://github.com/noi-techpark/webcomp-flightdata-realtime
```

Change directory:

```bash
cd webcomp-flightdata-realtime/
```

### Dependencies

Download all dependencies:

```bash
npm install
```

### Develope/Build

Build the project:

```bash
npm start
```

The necessary files to run the web component are provided under dist/. 
dist/demo.html can be viewed locally in your browser.

## Deployment

To create the distributable files, execute the following command:

```bash
npm run build
```

## Docker environment

For the project a Docker environment is already prepared and ready to use with all necessary prerequisites.

These Docker containers are the same as used by the continuous integration servers.

### Installation

Install [Docker](https://docs.docker.com/install/) (with Docker Compose) locally on your machine.

### Dependenices

First, install all dependencies:

```bash
docker-compose run --rm app /bin/bash -c "npm install"
```

### Start and stop the containers

Before start working you have to start the Docker containers:

```
docker-compose up --build --detach
```

After finished working you can stop the Docker containers:

```
docker-compose stop
```

### Running commands inside the container

When the containers are running, you can execute any command inside the environment. Just replace the dots `...` in the following example with the command you wish to execute:

```bash
docker-compose run --rm app /bin/bash -c "..."
```

Some examples are:

```bash
docker-compose run --rm app /bin/bash -c "npm run test"
```

## Information

### License

The code in this project is licensed under the GNU AFFERO GENERAL PUBLIC LICENSE Version 3 license. See the [LICENSE.md](LICENSE.md) file for more information.

### REUSE

This project is [REUSE](https://reuse.software) compliant, more information about the usage of REUSE in NOI Techpark repositories can be found [here](https://github.com/noi-techpark/odh-docs/wiki/Guidelines-for-developers-and-licenses#guidelines-for-contributors-and-new-developers).

Since the CI for this project checks for REUSE compliance you might find it useful to use a pre-commit hook checking for REUSE compliance locally. The [pre-commit-config](.pre-commit-config.yaml) file in the repository root is already configured to check for REUSE compliance with help of the [pre-commit](https://pre-commit.com) tool.

Install the tool by running:
```bash
pip install pre-commit
```
Then install the pre-commit hook via the config file by running:
```bash
pre-commit install
```

