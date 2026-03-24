# Setting Up OpenClaw
The purpose of this repo is to help skip the frustrating experience of setting up OpenClaw when trying to use its secure Docker containerization
features. For power users that want to have a reliable agent that can both run `exec` and filesystem tools and enable connection of
OpenClaw with particular folders on the laptop such as an Obsidian vault, the best secure but powerful setup is to have a **non-sandboxed gateway + sandboxed agent**.

However, the numerous bugs, vibe-coded documentation, and obscure permissions design of the codebase make it extremely difficult
to get the OpenClaw agent running properly inside the Docker sandbox. Hence, the instructions here were diligently compiled to help
other users navigate setting up OpenClaw agents inside Docker without having to go through numerous hours of trial and error which
I unfortunately had to experience.

The configuration used here successfully sets up an OpenClaw agent that:
* Runs securely inside Docker container (except the gateway which does not need to be sandboxed)
* Can access and modify specifically mounted external directories in the container (e.g. an Obsidian vault on your desktop)
* Can browse the web using Brave Search (`web_search` and `web_fetch` tools) from within the container
* Interfaces on multiple channels (e.g. Feishu, Telegram)

You will not need to worry about any agents mishandling the filesystem as the blast radius is limited to within the Docker sandbox.

## Instructions

1. First, make sure Docker engine is properly installed. On macOS you may need to download Docker Desktop.
2. Next, install openclaw using `npm i -g openclaw` and onboard with `openclaw onboard`. Provide API keys and add the channels you need.
3. Set up the docker image for the agent only sandbox by running the `sandbox-setup.sh` script provided in the repo. It is an
exact copy of `scripts/sandbox-setup.sh` from the `openclaw` repository. The reason it is provided here is so you can avoid needing to 
`git clone` the entire `openclaw` repo (it is very bloated and you only need a few files in the repo for docker setup).
4. Set up the sandboxed browser image as well by running `scripts/sandbox-browser-setup.sh`.

## `openclaw.json` Config

The most important part of the set up is to have a properly configured `openclaw.json` config file to avoid gnarly bugs due to permissions.

Important points:
* Set any external folders you want the agent to be able to access within its sandbox in the `"binds": [...]` list. It should be in
* Set `tools.fs.workspaceOnly` to `false` in order to allow the agent to access beyond `/workspace` to external mounts
the format `"/path/to/folder:/source:rw"` (for more info see [custom bind mounts](https://docs.openclaw.ai/gateway/sandboxing#custom-bind-mounts))
* External mount will not be accessible to agent if you do not give it the `exec` tool
* Due to 

Make sure the following config settings are all within your `openclaw.json` file. It should be **exactly as shown** except for API keys and file paths:
```
{
  "browser": {
    "ssrfPolicy": {
      "dangerouslyAllowPrivateNetwork": true
    }
  },
  "agents": {
    "defaults": {
      "workspace": "/Users/username/.openclaw/workspace",
      "sandbox": {
        "mode": "all",
        "workspaceAccess": "rw",
        "docker": {
          "network": "bridge",
          "binds": [
            "/Users/username/Desktop/folder/notes:/vault:rw"
          ],
          "dangerouslyAllowExternalBindSources": true
        },
        "browser": {
          "enabled": true,
          "binds": []
        }
      }
    },
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": [
            "group:fs",
            "group:web",
            "browser",
            "exec",
            "bash"
          ]
        }
      }
    ]
  },
  "tools": {
    "profile": "minimal",
    "fs": {
      "workspaceOnly": false
    },
    "exec": {
      "applyPatch": {
        "workspaceOnly": false
      }
    },
    "allow": [
      "group:fs",
      "group:web",
      "browser",
      "web_search",
      "web_fetch",
      "exec",
      "bash"
    ],
    "deny": [
      "process",
      "message"
    ],
    "web": {
      "search": {
        "enabled": true,
        "provider": "brave",
        "apiKey": "BRAVE_API_KEY"
      }
    },
    "sandbox": {
      "tools": {
        "allow": [
          "group:fs",
          "group:web",
          "browser",
          "web_search",
          "web_fetch",
          "exec",
          "bash"
        ],
        "deny": [
          "process",
          "message"
        ]
      }
    }
  },
}
```
