# Node Vue Devcontainer

[![YouTube Explainer](https://img.youtube.com/vi/E_B-dxVSYec/0.jpg)](https://www.youtube.com/watch?v=E_B-dxVSYec)

This project contains the following features:

* [Dev Container](https://containers.dev/)
  * [Base Image](https://github.com/devcontainers/images/tree/main/src/javascript-node)
  * [VS Code Documentation](https://code.visualstudio.com/docs/devcontainers/containers)
* [Node.js](https://nodejs.org/en) and [Express](https://expressjs.com/) backend
* [Vue 3](https://vuejs.org/) frontend
* [MySQL](https://hub.docker.com/_/mysql) and [phpMyAdmin](https://hub.docker.com/_/phpmyadmin) service containers

## Prerequisites

* [Docker Desktop](https://www.docker.com/products/docker-desktop/)
* [VS Code](https://code.visualstudio.com/)
  * [Dev Containers Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)
  * [Docker Extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker)

## Step 1 - Define the Dev Container

1. Create a new project folder and open it in VS Code
2. Use the command palette to add dev container files to the project ([Documentation](https://code.visualstudio.com/docs/devcontainers/create-dev-container))
3. Customize dev container files:

### `devcontainer.json`

```json
// For format details, see https://aka.ms/devcontainer.json. For config options, see the
// README at: https://github.com/devcontainers/templates/tree/main/src/javascript-node
{
	"name": "Node.js, Vue 3 & MySQL",  //
	// Or use a Dockerfile or Docker Compose file. More info: https://containers.dev/guide/dockerfile
	// "image": "mcr.microsoft.com/devcontainers/javascript-node:1-22-bookworm"
	"dockerComposeFile": "docker-compose.yml",
	"service": "app",
	"workspaceFolder": "/workspaces/${localWorkspaceFolderBasename}",

	// Features to add to the dev container. More info: https://containers.dev/features.
	"features": {
		"ghcr.io/devcontainers-contrib/features/vue-cli:2": {}
	},

	// Use 'forwardPorts' to make a list of ports inside the container available locally.
	"forwardPorts": [3001],

	// Use 'postCreateCommand' to run commands after the container is created.
	"postCreateCommand": "cd server && npm install && cd ../client && npm install",

	// Configure tool-specific properties.
	"customizations": {
		"vscode": {
			"extensions": [
				"Vue.volar"
			]
		}
	}

	// Uncomment to connect as root instead. More info: https://aka.ms/dev-containers-non-root.
	// "remoteUser": "root"
}
```

This can be easily customized to support different frontends or backends by adding different features or customizations. Also note the forwarded port `3001` can be changed. 

### `docker-compose.yml`

```yml
version: '3.8'

services:
  app:
    build: 
      context: .
      dockerfile: Dockerfile
    volumes:
      - ../..:/workspaces:cached
    # Overrides default command so things don't shut down after the process ends.
    command: sleep infinity
  
  # Additional Services
  mysqlnvd:
    image: mysql:lts
    container_name: mysqlnvd
    environment:
      MYSQL_HOST: mysqlnvd
      MYSQL_DATABASE: database
      MYSQL_USER: user
      MYSQL_PASSWORD: password
      MYSQL_RANDOM_ROOT_PASSWORD: fact

  phpmyadminnvd:
    image: phpmyadmin:latest
    container_name: phpmyadminnvd
    environment:
      PMA_ARBITRARY: 1
    ports:
      - '8080:80'
```

Likewise, the additional services defined in this file can be updated to include things like Mongo, PostgreSQL, or other Docker containers. If your system has a reverse proxy such as [Traefik](https://traefik.io/traefik/), you can add the labels here to enable external routing. 

### `Dockerfile`

```dockerfile
FROM mcr.microsoft.com/devcontainers/javascript-node:22-bookworm
```

If addtional items should be included in the container they can be added here. For example, I typically install `zsh` (my preferred terminal), `gnupg` (to sign commits), `npm-check-updates` (to easily update `npm` packages) and `better-commits` (to create more useful commit messages). 

```dockerfile
FROM mcr.microsoft.com/devcontainers/javascript-node:22-bookworm

# [Optional] Uncomment this section to install additional OS packages.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends gnupg2 zsh

# [Optional] Install global node modules
RUN su node -c "npm install -g npm-check-updates better-commits"
```

## Step 2 - Load the Dev Container

1. In VS Code, use the command palette to reopen the folder in a dev container. The first time this action is performed, it may take several minutes to download base images and build the container. This will be much faster in the future.
2. You will get an error stating that the last step failed the first time - this is because we have not created the `server` and `client` projects yet, so running `npm install` in those folders will not work.

## Step 3 - Create the `server` and `client` Projects

1. Use the [Express Generator](https://expressjs.com/en/starter/generator.html) to create an Express project in the `server` folder.
  1. I typically install [`nodemon`](https://www.npmjs.com/package/nodemon) and configure a `dev` command in the `package.json` file to use `nodemon` with this setup. 
  2. By default the Express Generator does not include a `.gitignore` file in the `server` folder. I recommend using [this one](https://github.com/github/gitignore/blob/main/Node.gitignore) as a starter.
2. Use the [`create-vue`](https://vuejs.org/guide/quick-start) scaffolding tool to create a Vue 3 project in the `client` folder.
  1. If you plan on using another frontend, such as React or Angular you can easily substitute it at this point. 
  2. By default, `vite` only attaches to internal ports and uses a random port. I recommend customizing the `dev` script in the `package.json` file to match include explicit host and port entries: `"dev": "vite --host 0.0.0.0 --port 3001"`
3. For a `vite` project, customize the `vite.config.js` to proxy API connections to the backend as desired. See that file in this repo for an example.
  1. Similar configuration can be done for other frontend frameworks

## Step 4 - Create VS Code Tasks

1. Create the file `.vscode/tasks.json` with the following content:

```json
// .vscode/tasks.json
{
  // See https://go.microsoft.com/fwlink/?LinkId=733558
  // for the documentation about the tasks.json format
  "version": "2.0.0",
  "tasks": [
    {
        "label": "Watch Server",
        "type": "shell",
        "command": "cd server && npm run dev",
        "group": "build",
        "presentation": {
            "group": "buildGroup",
            "reveal": "always",
            "panel": "new",
            "echo": false
        }
    },
    {
        "label": "Watch Client",
        "type": "shell",
        "command": "cd client && npm run dev",
        "group": "build",
        "presentation": {
            "group": "buildGroup",
            "reveal": "always",
            "panel": "new",
            "echo": false
        }
    },
    {
        "label": "Watch All",
        "dependsOn": [
            "Watch Server",
            "Watch Client"
        ],
        "group": "build",
        "runOptions": {
            "runOn": "folderOpen"
        }
    },
  ]
}
```

This will add a **Watch All** task to the tasks that can be found in the command palette under the **Run Task** option. It will also automatically run both tasks when the VS Code window is opened in the future. 

## Step 5 - Test Your Project

1. Use the **Watch All** task to start the frontend and backend in the terminal. 
2. Open [http://localhost:3001](http://localhost:3001) to see your project.
3. phpMyAdmin can be accessed at [http://localhost:8080](http://localhost:8080). Use the credentials found in the `docker-compose.yml` file to log in to the database.

This project includes a small modification in the default Vue project to query data from the `/api/` route in the Express backend project to demonstrate using `vite` to proxy connections from the frontend to the backend. 

## Step 6 - Deploy

This project **DOES NOT** include appropriate Docker configurations for deployment. **DO NOT USE** the dev container Dockerfile for deployment.

I can follow up with a proper deployment setup if desired - contact me!