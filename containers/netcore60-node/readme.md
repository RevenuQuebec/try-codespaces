# About netcore60-node

Container for dotnet 6.0 & node development.

Docker

```
docker pull petganrq/try-it:devcontainers_netcore60-node-latest
```

## Configurations and tools included into the docker image

- **C.UTF-8** - as locale (Debian can't support other locales as of present)
- **PATH** - the following is added to linux path
  - *~/.dotnet/tools*
  - *./node_modules/.bin*
- **Bash Completetion**
- **vscode** - a non root user in case someone needs it. Some tools have issues, so use it if it works for you.
- **nvm** - Node Version Manager is installed in /home/nvm/.nvm so it can be used from **root** and **vscode** users. Bash completion is configured for all users.
- **yarn** - Yarn Package Manager
- **Node LTE** - pre-installed with **nvm**
- **Docker CE CLI**
- **Docker Compose**
- **Dapr CLI**
- **Azure CLI**
- **Hashicorp Terraform**

## Visual Studio Code Remote or Codespaces

### devcontainer.json for Remote Containers

```json
{
  "name": "dotnet 6.0 & node lte",
  "image": "petganrq/try-it:devcontainers_netcore60-node-latest",
  // If you prefere to customize the docker image copy "Dockerfile" to the ".devcontainer" folder, uncomment "dockerFile" below and comment "image"
  // "dockerFile": "Dockerfile",
  "settings": {
    "terminal.integrated.shell.linux": "/bin/bash",
    // Make sure that the user profile is sourced for the shell.
    "terminal.integrated.shellArgs.linux": [
      "--login"
    ],
    "omnisharp.enableEditorConfigSupport": true,
    "nxConsole.useNVM": true
  },
  "extensions": [
    "editorconfig.editorconfig",
    "doggy8088.netcore-extension-pack",
    "fudge.auto-using",
    "dannymcgee.csharp-grammar-extended",
    "yzhang.markdown-all-in-one",
    "alanwalk.markdown-toc",
    "johnpapa.angular-essentials",
    "alexiv.vscode-angular2-files",
    "formulahendry.auto-close-tag",
    "formulahendry.auto-rename-tag",
    "steoates.autoimport",
    "vscode-icons-team.vscode-icons",
    "ms-azuretools.vscode-docker",
    "vsls-contrib.codetour",
    "jhipster-ide.jdl",
    "mikestead.dotenv"
  ],
  "forwardPorts": [
    // Developemnt backend default http
    5000,
    // Development backend default https
    5001
  ],
  "containerEnv": {
    "NUGET_XMLDOC_MODE": "none",
    "ASPNETCORE_ENVIRONMENT": "Development",
    "ASPNETCORE_URLS" :"http://localhost:5000",
    // Pass in the host directory for Docker mount commands from inside the container
    "DEV_HOST_PROJECT_PATH": "${localWorkspaceFolder}"
  },
  "remoteEnv": {
    "NUGET_XMLDOC_MODE": "none",
    "ASPNETCORE_ENVIRONMENT": "Development",
    "ASPNETCORE_URLS" :"http://localhost:5000",
    // Pass in the host directory for Docker mount commands from inside the container
    "DEV_HOST_PROJECT_PATH": "${localWorkspaceFolder}"
  },
  "mounts": [
    "source=/var/run/docker.sock,target=/var/run/docker.sock,type=bind",
    // VSCode Server extension local cache mount
    // Exclude special folder to be mapped to the dev container
    "source=dotnet-project-obj,target=${containerWorkspaceFolder}/src/dotnet-project/obj/,type=volume",
    "source=dotnet-project-bin,target=${containerWorkspaceFolder}/src/dotnet-project/bin/,type=volume",
    "source=dotnet-project-out,target=${containerWorkspaceFolder}/src/dotnet-project/out/,type=volume",
    "source=node-project-node_modules,target=${containerWorkspaceFolder}/src/node-project/node_modules/,type=volume"
  ],
  // Default Remote workspace bind
  //   doc: https://code.visualstudio.com/docs/remote/containers-advanced#_changing-the-default-source-code-mount
  "workspaceMount": "source=${localWorkspaceFolder},target=${containerWorkspaceFolder},type=bind,consistency=cached"
  // ,"workspaceFolder": "/workspaces"
  // Uncomment to connect as a non-root user. See https://aka.ms/vscode-remote/containers/non-root.
  // ,"remoteUser": "vscode"
}

```

### tasks.json Examples

```json
{
  "version": "2.0.0",
  "tasks": [
    // ...
    // Dotnet build
    {
      "label": "build",
      "command": "dotnet",
      "type": "process",
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "args": [
        "build",
        "${workspaceFolder}"
      ],
      "problemMatcher": "$tsc"
    },
    // ...
    // Node - start runner
    {
      "label": "npm start",
      "type": "shell",
      "options": {
      "cwd": "${workspaceFolder}/src/node-project"
      },
      "linux": {
        "command": "nvm",
        "args": [
            "exec", "default",
            "npm",
            "start"
          ]
      },
      "windows":{
        "command": "npm",
        "args": [
          "start"
        ]
      },
      "isBackground": true,
      "problemMatcher": "$tsc"
    }
    // ...
  ]
}
```

### launch.json Examples

```json
{
  // ...
  "version": "0.2.0",
  "configurations": [
    {
      "name": "dotnet-project (web)",
      "type": "coreclr",
      "request": "launch",
      "preLaunchTask": "build",
      "cwd": "${workspaceFolder}/src/dotnet-project",
      "stopAtEntry": false,
      "serverReadyAction": {
          "action": "openExternally",
          "pattern": "^\\s*Now listening on:\\s+(http?://\\S+)",
          "uriFormat": "%s/"
      },
      "windows": {
        "program": "dotnet",
        "args": [
          "run"
        ]
      },
      "linux": {
        // In order to kill the process correctly use only linux generated executable.
        // In linux if you use "dotnet run" or "${workspaceFolder}/src/dotnet-project/bin/Debug/netcoreapp6.0/dotnet-project.dll" the main process won't be killed when you stop the debugging.
        "program": "${workspaceFolder}/src/dotnet-project/bin/Debug/netcoreapp6.0/dotnet-project"
      }
    }
    // ...
  ]
}
```
