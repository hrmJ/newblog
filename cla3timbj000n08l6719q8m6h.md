# Wasn't sure about turborepo, but the kids insisted on it....


Last summer I was thinking about starting a toy project of some sorts. The
webdev landscape is evolving fast, and I have a long backlog of things I
would like to give a try. Here are just some:

- Js monorepos instead of separate repos for frontend / backend / utility packages
- pnpm instead of yarn / npm
- Mock service worker for testing instead of knex / mocking fetch
- The already-out-of-beta-stage svelte-kit
- The qwik framework

Instead of figuring out a complex real project or waiting a couple of years for
a chance to try these out at my day job, I found an opportunity a lot closer:
in my two kids and their love for an old card game called 77 funny questions
and silly answers. In this two-player game one is given a stack of random
questions and the other a stack of equally random but applicable answers.
The problem was that the same questions and answers had already been used so often
the game was getting a bit too repetitive. I could not help seeing a chance for
another web app! Perhaps one utilizing a monorepo architecture run with pnpm,
thoroughly tested with mock service workers and....

You might argue that this type of a project does not necessarily call for
something as complex as a monorepo, but nevertheless I decided to
give it a try. Turned out it gave me content for at least a couple of blog
posts. Let me, In this first one, summarise how I used turborepo to
set up the basic structure of the app.

## Getting started with a simple monorepo

You don't have study linguistics for 5 years in order to understand that the
first part of the word _monorepo_ is derived from Greek and probably means
something consisting of only a single instance. Tracing the etymology of
_repository_ all the way back to Latin's _reponere_ ('reposition') might
be a taller task but, then again, probably not entirely necessary in
order to understand the basic idea: monorepos are a way to build applications
inside a single repository. Instead of building a good old php monolith, you
could still, however, have separate apps for your backend, frontend etc.

All this I had kind of figured out. The actual point of using
something like turborepo wasn't, however, completely clear to me. What better
way to fill in the gaps than check it out in practice?

For my simple card game needs I wanted to get a monorepo up and running with
the following components:

1. A svelte(kit)-powered frontend
2. A fastify-powered api
3. A common eslint config shared as a package
4. A shared prettier config

Later on (or if the project would be more complex), perhaps also:

5. A shared component library
6. Other apis and frontends

### Where do I configure my apps?

The fast way to get started with turborepo is to run `npx create-turbo@latest`.
The initialization script let's you choose your package manager (I chose
pnpm to test that out, too) and scaffolds a basic folder structure for a
turborepo project. After some stripping down, mine looked like the following;

```

├── apps
├── package.json
├── packages
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
├── README.md
└── turbo.json
├── packages

```

At first, I was a bit confused: which config file do I need to edit and
what commands do I need to run in order to start developing the project's apps?
`pnpm-workspace.yaml` just contains the following:

```
packages:
  - "apps/*"
  - "packages/*"
```

Should I add something here? How about `package.json`?

```
{
  "name": "qag",
  "version": "0.0.0",
  "private": true,
  "workspaces": [
    "apps/*",
    "packages/*"
  ],
  "scripts": {
    "build": "turbo run build",
    "dev": "turbo run dev --parallel",
    "lint": "turbo run lint",
    "format": "prettier --write \"**/*.{ts,tsx,md}\""
  },
  "devDependencies": {
    "prettier": "latest",
    "turbo": "latest"
  },
  "engines": {
    "npm": ">=8.0.0",
    "node": ">=16.14.0"
  },
  "dependencies": {},
  "packageManager": "pnpm@7.0.0-rc.6",
  "testEnvironment": "jsdom"
}

```

Nothing new here: the "workspaces" part is familiar, but does not really seem
like the place to write configuration for the different parts of the monorepo.

Here's the catch: it was way simpler than I thought. No need to configure, just
start creating apps by adding content to the apps folder. To add a backend just
start a regular nodejs project in, say, `apps/backend`. To start a frontend create
`apps/frontend`. For my use case, I went with the following structure:

```
apps/
├── qagApi
│   ├── migrations
│   ├── node_modules
│   ├── package.json
│   ├── src
│   ├── tsconfig.json
│   ├── utils.ts
│   └── vitest.config.ts
└── qagFront
    ├── node_modules
    ├── package.json
    ├── package-lock.json
    ├── README.md
    ├── src
    ├── static
    ├── svelte.config.js
    ├── tsconfig.json
    ├── vite.config.js
    └── vitest.config.js
```

As you can see, at `qagApi` I have a regular nodejs api with its own
package.json and subfolders. `qagFront`, again, is a sveltekit app with its own
structure.

### Npm run dev X 1??

The cool thing is, with these sub apps ready, I can just cd to the project's
root and run `pnpm dev`, which yields:

```
> qag@0.0.0 dev /home/juho/projects/qag
> turbo run dev --parallel

qagfront:dev: > qagfront@0.0.1 dev /home/juho/projects/qag/apps/qagFront
qagfront:dev: > vite dev --host 192.168.0.9
qagfront:dev:
qagapi:dev:
qagapi:dev: > qagapi@1.0.0 dev /home/juho/projects/qag/apps/qagApi
qagapi:dev: > export $(cat .env | xargs)  && ts-node-dev --respawn src/server.ts
qagapi:dev:
qagapi:dev: [INFO] 18:52:31 ts-node-dev ver. 2.0.0 (using ts-node ver. 10.8.1, typescript ver. 4.7.4)
qagfront:dev:
qagfront:dev:   VITE v3.0.0  ready in 1876 ms
qagfront:dev:
qagfront:dev:   ➜  Local:   http://192.168.0.9:5173/
qagfront:dev:   ➜  Network: http://192.168.0.9:5173/
qagfront:dev:   ➜  Network: http://172.20.0.1:5173/
qagfront:dev:   ➜  Network: http://172.21.0.1:5173/
qagfront:dev:   ➜  Network: http://172.18.0.1:5173/


```

As you can see, this is turborepo running my two js projects in parallel with
a single command. Not a bad dev experience! For me, this might already be
enough of an argument for actually using something like turborepo as a real
architecture of a serious app. Apparently, the real benefits are, however, in
optimizing the build process and making deployments fast as heck, reducing the
workload of a ci/cd pipeline.... These benefits, [I
hear](https://turborepo.org/docs/core-concepts/caching), are achieved, at least
in part, by utilizing a caching system for project-specific tasks such as `npm run build`, `npm run test` etc.. Something to dig into later -- for now, I'm
just enjoying the simple dev flow.

### Installing packages

Let's look at `pnpm-workspace.yaml` once more:

```
packages:
  - "apps/*"
  - "packages/*"
```

Besides `apps` there is `packages`. The same way I can declare separate apps
(such as a frontend and a backend) inside the former, I can create _shared libraries
used by these apps_ inside the latter. For my toy project, I created the following:

```
├── eslint
│   ├── index.js
│   ├── node_modules
│   └── package.json
├── prettier
│   ├── index.json
│   └── package.json

```

In the same vein as apps, these, too, are just separate js projects
inside the packages folder. The same way as `apps/qaApi` contains the backend app,
`./eslint` contains a simple eslint config distributed
as package and the same holds for `./prettier`. The beauty of the monorepo system
is that making modifications and updating -- or even just installing -- the
packages in the apps is a breeze.

```
cd apps/qagFront/
npm install eslint-config
```

voilá, the shared eslint-config now in qagFront. If I had a separate library
for utility functions called my-utility-lib (at `packages/my-utility-lib`), I would

1. make changes in `packages/my-utility-lib`
2. run the build command for my-utility-lib
3. run `pnpm update my-utility-lib`
   - inside e.g. `apps/qapApi`
   - OR inside the project root
4. Have the new changes available


## What's next?

Setting up pnpm + turborepo was a fun exercise. The next challenge I chose was
to set up api testing inside the fastify-based backend + intergration testing
the frontend using mock service worker. Stay tuned!
