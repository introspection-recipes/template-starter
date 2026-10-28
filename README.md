# Starter Template

A minimal Pi coding agent with repository inspection, shell, and file-editing tools.

## Quick start

Install Node.js 24 or newer and the Introspection CLI, then create your own recipe from this template. `init` also installs the compatible Pi harness and Recipes extension.

```bash
npm install -g @introspection-ai/cli
introspection init coding-agent template-starter
cd coding-agent
```

Validate the recipe and start a fresh local Pi session:

```bash
introspection local --runtime coding-agent
```

Ask:

> Inspect this repository and summarize its structure. Do not change any files.

The local loop needs no Introspection login or cloud runtime. Pi may ask you to configure the model provider on the first run.

## Move the recipe through its lifecycle

The everyday flow is:

```text
Local Pi → Development → Staging → Production → Learn and repeat
```

### 1. Change and prove it locally

Customize `SYSTEM.md` and `agents/agent.yaml`, then repeat the local prompt above in fresh sessions. Test any new tools or instructions as well as ordinary coding requests that should keep working.

### 2. Create the runtime and test development

Development requires the recipe's first runtime. Commit the locally proven recipe, push it to your own GitHub repository, then:

1. In the Introspection app, open your organization's **Integrations** page and grant the Introspection GitHub App access to the repository.
2. Open the target project, go to **Runtimes**, and select **New runtime**.
3. Choose the repository and the runtime in `.introspection/coding-agent.yaml`, confirm that its recipe path is `.`, and create the first version from `main`.
4. In **Versions**, confirm that the immutable version's recipe commit matches the `main` commit you intended to deploy.

Once the runtime exists, exercise uncommitted changes through the cloud development path:

```bash
introspection login
introspection dev --runtime coding-agent
```

Leave the command running, open the development chat URL it prints, and repeat the repository-summary prompt. Saved changes to `SYSTEM.md` or `agents/agent.yaml` are picked up without a commit or push. Stopping `introspection dev` removes the local overlay; it does not deploy a version.

### 3. Verify a pull-request candidate in staging

Push a feature branch and open a pull request. The GitHub integration creates an immutable candidate version for that commit; do not create another runtime for the same agent.

In the runtime's **Versions** view, find the candidate by branch and commit, pin it to **Staging**, and select **Preview**. Repeat the repository-summary prompt and confirm that the conversation used the intended candidate commit without an unhandled error.

### 4. Merge it to production

After the staging behavior and pull request are approved, merge into the repository's configured production branch. The GitHub integration creates the immutable version and activates it for production; there is no separate promotion command.

Run one small production check through the stable `coding-agent` runtime and confirm that it resolves to the merged recipe commit.

### 5. Learn and repeat

Use production conversations and recurring patterns to choose the smallest useful change, then return to the local loop. See the [agent development lifecycle](https://docs.introspection.dev/guides/development-lifecycle) for the full workflow.

## Add an MCP server

An MCP server is declared in the recipe, but where it points is decided by a *binding*, outside the recipe. The recipe names a logical id, and each place it runs binds that id to a real server. The template ships with every list empty, ready to fill.

1. **Declare it** in `package.json` under `pi.mcp.servers`, with an `id`, whether it is `required`, and the tools it exposes:

   ```json
   "mcp": {
     "servers": [
       { "id": "my-server", "required": true, "tools": { "include": ["search"], "exclude": [] } }
     ]
   }
   ```

2. **Opt the agent in** under `mcp.servers` in `agents/agent.yaml`. A declared server gives an agent no tools until it opts in:

   ```yaml
   mcp:
     mode: tools
     servers:
       my-server:
         include:
           - search
   ```

3. **Bind it** where the recipe runs:
   - **In the cloud**, connect the server to the runtime in the Introspection app; add the connector to `runtime.connectors` in `.introspection/coding-agent.yaml` if the runtime needs one. Credentials stay with the platform and are injected at egress.
   - **Locally**, keep a personal binding file outside the repository and point `PI_RECIPES_MCP_LOCAL_CONFIG` at it.
   - **For evaluations**, list fakes in `.pi/mcp.local.json`, as below.

### Best practices

- **Never put a credential, a real endpoint or a `${TOKEN}` reference in the repository.** `introspection dev` stages untracked files with `git add -A`, so even an uncommitted file inside the repository travels in the development patch.
- **Commit `.pi/mcp.local.json` only as an evaluation binding.** Every server must be `streamable_http` to a loopback URL (`localhost`, `127.0.0.0/8` or `::1`) with no auth. `introspection check` and the pre-commit hook reject anything else. The template commits it with an empty `servers` list.
- **The committed file only affects evaluations.** On an eval task, each id it names binds to that fake instead of the runtime's live server. A production or development conversation never reads it, so it cannot redirect real traffic. Point it at a fake served by a sidecar of the eval task, such as `http://localhost:8000/mcp`. Sidecars share the sandbox's network, so loopback reaches them.
- **Don't add `.pi/mcp.local.json` to `.gitignore`.** An ignored file never reaches an evaluation's checkout.
- **Include only the tools the agent needs**, in both the declaration and the agent's opt-in.

## What's included

- A ready-to-run Pi coding agent
- Shared behavior in `SYSTEM.md`
- Model and tool configuration in `agents/agent.yaml`
- Runtime metadata in `.introspection/coding-agent.yaml`
- Empty MCP declarations in `package.json`, `agents/agent.yaml` and `.pi/mcp.local.json`, ready to fill

[Read the Recipes documentation →](https://docs.introspection.dev/recipes)
