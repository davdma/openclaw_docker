# openclaw_docker
The purpose of this repo is to help you skip the frustrating experience of setting up OpenClaw in order to take advantage of its
secure agent docker sandbox feature (non-sandboxed gateway + sandboxed agent).

This repo has all the necessary files to set up openclaw with dockerized agent.

## Instructions

First, install openclaw using `npm i -g openclaw`.

Make sure to paste the

## `openclaw.json` Config

The most important part of the config file to avoid gnarly bugs due to permissions:

```
  "browser": {
    "ssrfPolicy": {
      "dangerouslyAllowPrivateNetwork": true
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "anthropic/claude-sonnet-4-6"
      },
      "models": {
        "anthropic/claude-sonnet-4-6": {}
      },
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
```
