# Agentic Sandboxes

This repository is a set of Docker Sandbox configurations intended to be used in the development of code.
The purposes of these files, alongside Docker Sandbox (sbx) is to create isolated, low-access LLM instances which can solve complex problems in isolation without interfering with the host computer.

## Project Structure

This repository is divided into two sections: `kits` and `mixins`:
- `kits` define the initial setup of a sandbox using a particular LLM interface (e.g., copilot).
    - NOTE: These are generally copied from https://github.com/docker/sbx-kits-contrib
- `mixins` define agentic parameters, skills, and instructions that can be applied across LLM interfaces

## How to use

### A. Priming the agent

1. Take a look at the `kits` directory and identify the kit you want to run (e.g., the copilot-template kit)
2. Run the kit with a command like `sbx run --kit ./copilot copilot-sandbox`
4. Confirm the sandbox is running and connected to the remote LLM server.
5. Take a look at the `mixins` directory and identify a mixin that applies to your use case.
6. Apply the mixin(s) that you need to your running sandbox like `sbx kit add my-sandbox ./my-kit`
    - NOTE: you will likely want to apply multiple kits -- some to add additional skills + context, and others to provide specific instructions for the task at hand.

### B. Instructing
At this point, we have a sandbox running with a generic context helpful for our use case (e.g., an agent prepared to handle Python coding tasks).
Our next step is to point this sandbox at a problem, and give it specific instructions.

1. Provide the Sandbox with a copy of the relevant context / working files (e.g., a git repo)
    - PREFERRED: use `sbx run --clone <my-sandbox> ./path/to/repo` to clone a git repo to the Sandbox environment (COPIED)
    - use `sbx run <my-sandbox> ./path/to/repo` to simply mount your directory to the sandbox (SHARED)
2. Provide the Sandbox specific instructions

> NOTE: I want to set up my own system for providing the LLM specific instructions -- also in a way that I can add new tasks continuously.
> I will set up an 'instructions' directory by default in the sandbox environment, and upload new instructions via `sbx cp ./local/path/instructions_1.md /instructions/instructions_1_MMDDYYYY_HHMMSS.md`


## Example setup:
```sh
#!/bin/sh

# Script to set up a default Docker sandbox environment with Copilot for programming tasks

#!bin/bash

# Script to set up basic copilot sandbox + attach relevant mixins for coding


# Allow user to specify repo -- default to current directory
GIT_REPO=${1:-./}

# -------------------
# -- SANDBOX SETUP --
SANDBOX_NAME=copilot-sandbox
SANDBOX_DIR=$HOME/dev/kstine93/agentic-sandboxes



# Create sandbox from kit template (no mounted file):
sbx create \
    --memory=16g \
    --cpus=2 \
    $SANDBOX_DIR/kits/copilot/ \
    --name $SANDBOX_NAME \
    --kit $SANDBOX_DIR/mixins/software-craftsman \ # Sandbox mixin
    --kit $SANDBOX_DIR/mixins/python-programmer \ # Sandbox mixin

sbx run --name $SANDBOX_NAME --clone $GIT_REPO

```