# Start here: set up your repository and coding agent

This guide is for learners who have not used GitHub or an AI coding agent before. When you finish,
you will have:

- a GitHub repository named `MyOntologyServer`;
- an optional local copy of that repository on your computer;
- one coding agent connected to the correct repository;
- a clear understanding of whether the agent is working locally or in the cloud; and
- a safe first task that proves the setup works without changing files.

You do not need to install every agent in this guide. Pick one primary route and return to the
alternatives later if they become useful.

Product availability, plan requirements, and interface labels can change. Use the linked official
documentation when what you see differs from a screenshot or instruction.

## 1. Create `MyOntologyServer` on GitHub

You need a free [GitHub account](https://github.com/signup).

1. Sign in to GitHub.
2. Open [Create a new repository](https://github.com/new).
3. Choose your personal account or organisation as the owner.
4. Enter `MyOntologyServer` as the repository name.
5. Add a short description, such as `My guided ontology-service project`.
6. Choose **Private** while learning unless you deliberately want the project to be public.
7. Select **Add a README file**.
8. Leave `.gitignore` and licence unselected for now; the later prompts add the appropriate project
   files deliberately.
9. Select **Create repository**.

The GitHub repository is the remote, shared copy of your project. At this point it contains one
file, `README.md`.

GitHub also provides an official
[repository-creation walkthrough](https://docs.github.com/en/get-started/start-your-journey/creating-a-repository-for-your-project-on-github).

## 2. Choose local or cloud work

Both routes can build the same project:

| | Local agent | Cloud agent |
|---|---|---|
| Where the agent runs | On your computer | In a provider-hosted environment |
| How it gets the code | You select a cloned folder | You authorise a GitHub repository |
| Sees uncommitted local files | Yes | No |
| Uses tools installed on your computer | Yes | No |
| Must your computer remain available? | Usually yes | Usually no |
| How changes appear | Directly in your local working tree | Usually on a remote branch or pull request |

**Recommended first route:** clone the repository and use a local desktop agent. It makes the
relationship between files, Git changes, tests, commits, and pushes easier to see.

Use a cloud agent when you prefer not to configure a local development environment or want a task
to continue while your computer is off.

## 3. Clone the repository locally

Cloning creates a working copy of the GitHub repository on your computer. Local agents work against
this folder.

### Recommended: GitHub Desktop

[Download GitHub Desktop](https://desktop.github.com/download/), install it, and sign in to the same
GitHub account that owns `MyOntologyServer`.

Then:

1. Open `MyOntologyServer` on GitHub.
2. Select **Code**.
3. Select **Open with GitHub Desktop**.
4. Choose a local folder where you keep projects.
5. Select **Clone**.
6. In GitHub Desktop, select **Repository → Show in Finder** on macOS or
   **Repository → Show in Explorer** on Windows.
7. Confirm the folder contains `README.md`.

GitHub documents the same process in
[Cloning a repository with GitHub Desktop](https://docs.github.com/en/desktop/adding-and-cloning-repositories/cloning-a-repository-from-github-to-github-desktop).

### Command-line alternative

If you already use a terminal and Git:

```bash
git clone https://github.com/YOUR-GITHUB-NAME/MyOntologyServer.git
cd MyOntologyServer
```

Replace `YOUR-GITHUB-NAME` with the account or organisation that owns the repository. GitHub's
[cloning guide](https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository)
also covers HTTPS, SSH, GitHub CLI, and empty repositories.

The `OntologyServerBuilder` repository contains the learning prompts. `MyOntologyServer` is the
separate repository in which your agent will build the service. Do not accidentally select the
builder repository when starting an agent session.

## 4. Connect one coding agent

Choose one of the following routes. You can switch later without restarting the project because Git
and the checked-in repository files remain the shared source of truth.

### Option A: Codex on desktop

1. Install the current ChatGPT desktop application using
   [OpenAI's Codex getting-started page](https://openai.com/codex/get-started/).
2. Open the app and sign in with your ChatGPT account.
3. Open **Codex** from the application menu.
4. Add a project and select the locally cloned `MyOntologyServer` folder.
5. Choose a local environment when the app offers an environment choice.
6. Keep the default approval or sandbox protections enabled.

Codex should now be able to read and edit files inside that selected folder. Selecting a local
folder does not automatically give it access to every GitHub repository or every folder on your
computer.

OpenAI's current desktop application includes Codex for working with local files, repositories,
terminals, and developer tools. See
[ChatGPT desktop and Codex](https://help.openai.com/en/articles/20001276/).

### Option B: Claude Code Desktop

1. [Download Claude Desktop](https://claude.com/download), install it, and sign in.
2. Open the **Code** tab.
3. Select **Local** as the environment.
4. Select the locally cloned `MyOntologyServer` folder.
5. Keep the normal permission prompts enabled.

Claude Code Desktop can then read the project files, propose changes, run commands, and show a
visual diff. Claude Code plan availability may differ from the general Claude Desktop application;
check the current requirements in the
[Claude Code desktop quickstart](https://code.claude.com/docs/en/desktop-quickstart).

On Windows, Claude Code's local mode requires Git. Follow its quickstart if the Code tab reports
that Git is missing.

### Option C: GitHub Copilot cloud agent

This route works directly from GitHub and does not require a local clone.

1. Confirm your GitHub account has an eligible Copilot plan.
2. Open `MyOntologyServer` on GitHub.
3. Open the repository's **Agents** tab, or go to
   [github.com/copilot/agents](https://github.com/copilot/agents).
4. Select `MyOntologyServer` in the repository selector.
5. Submit a small task.
6. Review the session log, branch, and pull request produced by the agent.

Organisation-owned repositories may require an administrator to enable Copilot agents. Follow
GitHub's current
[Copilot agent quickstart](https://docs.github.com/en/copilot/how-tos/copilot-on-github/use-copilot-agents/overview).

### Another coding agent

If you use a different product, find one of these two setup actions in its official documentation:

- **Local:** open or select the cloned `MyOntologyServer` folder.
- **Cloud:** install or authorise its GitHub integration for `MyOntologyServer`.

Grant access only to the repository needed for the tutorial. Do not install an unverified extension
or paste a personal access token into a chat prompt.

## 5. Prove the connection safely

Start with a read-only task:

```text
Read this repository and list the files you can see. Confirm the repository name and current
branch. Do not create, edit, delete, commit, push, or install anything.
```

A correct result should identify `MyOntologyServer` and `README.md`.

Stop and correct the setup if the agent:

- reports `OntologyServerBuilder` instead of `MyOntologyServer`;
- cannot see `README.md`;
- is working against a different GitHub account or organisation;
- asks for access to unrelated repositories; or
- proposes changes even though the task was read-only.

For a local agent, reselect the cloned folder. For a cloud agent, check the repository selector and
the GitHub integration's repository permissions.

## 6. Begin the learning sequence

Keep OntologyServerBuilder open in your browser so you can copy one prompt at a time.

1. Give the agent the complete
   [reusable contract](prompts/00-reusable-contract.md).
2. Wait for the agent to acknowledge the contract.
3. Give it [Prompt 1](prompts/01-scaffold-and-domain-model.md).
4. Review the changes and acceptance checks before continuing.
5. Continue through the prompts in README order.

Do not paste every implementation prompt into one message. The stages are intentionally separated
so you can inspect, test, and understand each increment before the next one begins.

When a prompt says **commit locally but do not push**, the agent may create a commit in the local
clone, but it must not publish that commit to GitHub until you explicitly ask. A cloud agent may
need to use a remote branch because its environment is hosted; review its branch and pull request
instead of treating that as an invisible local commit.

## 7. Understand the Git words used in the guide

- **Repository:** the project and its version history.
- **Remote repository:** the shared copy hosted on GitHub.
- **Clone:** a local working copy of the remote repository.
- **Branch:** an isolated line of work.
- **Working tree:** the files currently visible in a local clone.
- **Commit:** a named snapshot in the repository history.
- **Push:** publish local commits to a remote repository.
- **Pull:** download and integrate remote commits.
- **Pull request:** a GitHub review proposing that one branch be merged into another.
- **Diff:** the exact lines added, removed, or changed.

The coding agent can perform these operations, but you remain responsible for reviewing what it
changed and deciding what is merged.

## 8. Work safely

- Start with a private learning repository.
- Give an agent access only to `MyOntologyServer`.
- Keep approval and sandbox protections enabled.
- Never place passwords, API keys, personal access tokens, `.env` contents, or production data in a
  prompt or committed file.
- Ask the agent to use a branch for changes that will be pushed.
- Read the diff before committing or merging.
- Require the agent to run the checks named by each prompt.
- Do not approve a destructive command you do not understand.
- Do not merge a pull request merely because an agent created it.

## 9. Optional: use an agent inside VS Code

This is not required for the learning sequence. If you already use VS Code, open the cloned
`MyOntologyServer` folder and install only the official extension for your chosen provider:

- [Codex IDE extension](https://developers.openai.com/codex/ide/)
- [Claude Code in VS Code](https://code.claude.com/docs/en/ide-integrations)
- [GitHub Copilot in VS Code](https://code.visualstudio.com/docs/copilot/setup)

The repository and Git concepts do not change. The extension works against the folder opened in VS
Code, so verify the window shows `MyOntologyServer` before starting.

## 10. Common setup problems

### The local agent cannot see `README.md`

The wrong folder is open. Select the cloned `MyOntologyServer` directory, not its parent directory
and not `OntologyServerBuilder`.

### The agent can edit files but cannot push

Local folder access and GitHub authentication are separate. Sign in through GitHub Desktop,
GitHub CLI, or the coding agent's supported GitHub integration. Do not send credentials through the
prompt.

### The cloud agent cannot find the repository

Check that its GitHub App or OAuth integration is authorised for `MyOntologyServer`. Organisation
policies may require an administrator to approve the integration.

### The cloud agent cannot see a local change

Cloud agents start from GitHub branches. Commit and push the required change first, or continue the
task locally. Uncommitted files on your computer do not exist in the cloud environment.

### A command works in Terminal but not in the desktop agent

Desktop applications may use a different environment or executable search path. Use the agent's
local-environment settings or official troubleshooting guide instead of repeatedly approving the
same failing command.
