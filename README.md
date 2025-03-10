# htmx-axum-template

Template for web apps using:
- htmx
- tailwindcss
- axum for server
- minijinja for templating
- vite for bundling js
- pnpm for package management
- pre-commit for linting both backend and frontend code
- x86_64-unknown-linux-musl as rust compile target to produce a fully static binary

## How this works

Directory structure:

```
├── app             /* app/ contains the vite project and all the frontend files (html, css, js) */
├── Cargo.lock      /* top level is the rust package */
├── Cargo.toml
├── src
```

Apart from using the html templates in `app/` for htmx, the axum server also simply statically serves all files in the `app/` dir next to it at path `/`.

At development time, any changes to the static frontend src files in `app/` are immediately reflected.

At production time, simply move the `dist/` output of `vite build` to an `app/` dir next to the compiled binary.

This however introduce the following quirks:
- during development, `cargo run` must be run at the project root or the `app/` folder will not be served
- js dependency imports must import directly from `../node_modules/` since a simple static file server doesn't know how to resolve esm imports
- the frontend files must use the tailwindcss build output during development time, since a simple static file server doesn't know how to preprocess css
- we cannot use vite's publicDir feature in order to preserve the same directory structure at dev and prod time

## Setup

### Requirements

- [pnpm](https://pnpm.io/installation)
- [rust](https://www.rust-lang.org/tools/install)

### Install pre-commit

`./install-precommit.sh`

### Install js dependencies

`cd app && pnpm install`

## Development

### Server

`cargo run` in project root to start the development server

### CSS

Run `pnpm tailwind-watch` in the `app` folder for tailwind CSS live-reloading

### Both

`./dev.sh` runs `pnpm tailwind-watch` in the background and `cargo run dev` in the foreground, killing both processes if interrupted (Ctrl+C). This still requires full page reloads to get any updated CSS styles.

### Adding static files

Add the files to `app/`, treating it as the `/` path, then update the `viteStaticCopy` plugin in `vite.config.js` to include them.

## Production

`docker build -t my-web-app .`

`docker run -p 3000:3000 --name my-web-app --init --rm my-web-app`
