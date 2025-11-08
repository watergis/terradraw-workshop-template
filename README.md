# terradraw-workshop-template

This repository is a templated used in TerraDraw workshop at FOSS4G Auckland.

## Project dependencies

As default, the following libraries are installed in SvelteKit project.

- maplibre-gl
- terra-draw
- terra-draw-maplibre-gl-adapter

## Clone this repository

```sh
git clone git@github.com:watergis/terradraw-workshop-template.git
cd terradraw-workshop-template
```

## Launch a project locally

Once you've cloned a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```sh
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

After opening localhost:5173 in your browser, you will see the OSM map like below.

![demo.png](./demo.png)

## Building

To create a production version of your app:

```sh
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://svelte.dev/docs/kit/adapters) for your target environment.
