# CLI Commands

Gemini CLI supports several built-in commands to help you manage your session, customize the interface, and control its behavior. These commands are prefixed with a forward slash (`/`), an at symbol (`@`), or an exclamation mark (`!`).

## Slash commands (`/`)

Slash commands provide meta-level control over the CLI itself.

### Built-in Commands

- **`/bug`**
  - **Description:** File an issue about Gemini CLI. By default, the issue is filed within the GitHub repository for Gemini CLI. The string you enter after `/bug` will become the headline for the bug being filed. The default `/bug` behavior can be modified using the `bugCommand` setting in your `.gemini/settings.json` files.

- **`/chat`**
  - **Description:** Save and resume conversation history for branching conversation state interactively, or resuming a previous state from a later session.
  - **Sub-commands:**
    - **`save`**
      - **Description:** Saves the current conversation history. You must add a `<tag>` for identifying the conversation state.
      - **Usage:** `/chat save <tag>`
    - **`resume`**
      - **Description:** Resumes a conversation from a previous save.
      - **Usage:** `/chat resume <tag>`
    - **`list`**
      - **Description:** Lists available tags for chat state resumption.

- **`/clear`**
  - **Description:** Clear the terminal screen, including the visible session history and scrollback within the CLI. The underlying session data (for history recall) might be preserved depending on the exact implementation, but the visual display is cleared.
  - **Keyboard shortcut:** Press **Ctrl+L** at any time to perform a clear action.

- **`/compress`**
  - **Description:** Replace the entire chat context with a summary. This saves on tokens used for future tasks while retaining a high level summary of what has happened.

- **`/copy`**
  - **Description:** Copies the last output produced by Gemini CLI to your clipboard, for easy sharing or reuse.

- **`/editor`**
  - **Description:** Open a dialog for selecting supported editors.

- **`/extensions`**
  - **Description:** Lists all active extensions in the current Gemini CLI session. See [Gemini CLI Extensions](../extension.md).

- **`/help`** (or **`/?`**)
  - **Description:** Display help information about Gemini CLI, including available commands and their usage.

- **`/mcp`**
  - **Description:** List configured Model Context Protocol (MCP) servers, their connection status, server details, and available tools.
  - **Sub-commands:**
    - **`desc`** or **`descriptions`**:
      - **Description:** Show detailed descriptions for MCP servers and tools.
    - **`nodesc`** or **`nodescriptions`**:
      - **Description:** Hide tool descriptions, showing only the tool names.
    - **`schema`**:
      - **Description:** Show the full JSON schema for the tool's configured parameters.
  - **Keyboard Shortcut:** Press **Ctrl+T** at any time to toggle between showing and hiding tool descriptions.

- **`/memory`**
  - **Description:** Manage the AI's instructional context (hierarchical memory loaded from `GEMINI.md` files).
  - **Sub-commands:**
    - **`add`**:
      - **Description:** Adds the following text to the AI's memory. Usage: `/memory add <text to remember>`
    - **`show`**:
      - **Description:** Display the full, concatenated content of the current hierarchical memory that has been loaded from all `GEMINI.md` files. This lets you inspect the instructional context being provided to the Gemini model.
    - **`refresh`**:
      - **Description:** Reload the hierarchical instructional memory from all `GEMINI.md` files found in the configured locations (global, project/ancestors, and sub-directories). This command updates the model with the latest `GEMINI.md` content.
    - **Note:** For more details on how `GEMINI.md` files contribute to hierarchical memory, see the [CLI Configuration documentation](./configuration.md#4-geminimd-files-hierarchical-instructional-context).

- **`/restore`**
  - **Description:** Restores the project files to the state they were in just before a tool was executed. This is particularly useful for undoing file edits made by a tool. If run without a tool call ID, it will list available checkpoints to restore from.
  - **Usage:** `/restore [tool_call_id]`
  - **Note:** Only available if the CLI is invoked with the `--checkpointing` option or configured via [settings](./configuration.md). See [Checkpointing documentation](../checkpointing.md) for more details.

- **`/stats`**
  - **Description:** Display detailed statistics for the current Gemini CLI session, including token usage, cached token savings (when available), and session duration. Note: Cached token information is only displayed when cached tokens are being used, which occurs with API key authentication but not with OAuth authentication at this time.

- [**`/theme`**](./themes.md)
  - **Description:** Open a dialog that lets you change the visual theme of Gemini CLI.

- **`/auth`**
  - **Description:** Open a dialog that lets you change the authentication method.

- **`/about`**
  - **Description:** Show version info. Please share this information when filing issues.

- [**`/tools`**](../tools/index.md)
  - **Description:** Display a list of tools that are currently available within Gemini CLI.
  - **Sub-commands:**
    - **`desc`** or **`descriptions`**:
      - **Description:** Show detailed descriptions of each tool, including each tool's name with its full description as provided to the model.
    - **`nodesc`** or **`nodescriptions`**:
      - **Description:** Hide tool descriptions, showing only the tool names.

- **`/privacy`**
  - **Description:** Display the Privacy Notice and allow users to select whether they consent to the collection of their data for service improvement purposes.

- **`/quit`** (or **`/exit`**)
  - **Description:** Exit Gemini CLI.

### Custom Commands

For a quick start, see the [example](#example-a-pure-function-refactoring-command) below.

Custom commands allow you to save and reuse your favorite or most frequently used prompts as personal shortcuts within Gemini CLI. You can create commands that are specific to a single project or commands that are available globally across all your projects, streamlining your workflow and ensuring consistency.

#### File Locations & Precedence

Gemini CLI discovers commands from two locations, loaded in a specific order:

1.  **User Commands (Global):** Located in `~/.gemini/commands/`. These commands are available in any project you are working on.
2.  **Project Commands (Local):** Located in `<your-project-root>/.gemini/commands/`. These commands are specific to the current project and can be checked into version control to be shared with your team.

If a command in the project directory has the same name as a command in the user directory, the **project command will always be used.** This allows projects to override global commands with project-specific versions.

#### Naming and Namespacing

The name of a command is determined by its file path relative to its `commands` directory. Subdirectories are used to create namespaced commands, with the path separator (`/` or `\`) being converted to a colon (`:`).

- A file at `~/.gemini/commands/test.toml` becomes the command `/test`.
- A file at `<project>/.gemini/commands/git/commit.toml` becomes the namespaced command `/git:commit`.

#### TOML File Format (v1)

Your command definition files must be written in the TOML format and use the `.toml` file extension.

##### Required Fields

- `prompt` (String): The prompt that will be sent to the Gemini model when the command is executed. This can be a single-line or multi-line string.

##### Optional Fields

- `description` (String): A brief, one-line description of what the command does. This text will be displayed next to your command in the `/help` menu. **If you omit this field, a generic description will be generated from the filename.**

---

#### Example: A "Pure Function" Refactoring Command

Let's create a global command that asks the model to refactor a piece of code.

**1. Create the file and directories:**

First, ensure the user commands directory exists, then create a `refactor` subdirectory for organization and the final TOML file.

```bash
mkdir -p ~/.gemini/commands/refactor
touch ~/.gemini/commands/refactor/pure.toml
```

**2. Add the content to the file:**

Open `~/.gemini/commands/refactor/pure.toml` in your editor and add the following content. We are including the optional `description` for best practice.

```toml
# In: ~/.gemini/commands/refactor/pure.toml
# This command will be invoked via: /refactor:pure

description = "Asks the model to refactor the current context into a pure function."

prompt = """
Please analyze the code I've provided in the current context.
Refactor it into a pure function.

Your response should include:
1. The refactored, pure function code block.
2. A brief explanation of the key changes you made and why they contribute to purity.
"""
```

**3. Run the Command:**

That's it! You can now run your command in the CLI. First, you might add a file to the context, and then invoke your command:

```
> @my-messy-function.js
> /refactor:pure
```

Gemini CLI will then execute the multi-line prompt defined in your TOML file.

This initial version of custom commands is focused on static prompts. Future updates are planned to introduce more dynamic capabilities, including:

- **Argument Support:** Passing arguments from the command line directly into your `prompt` template.
- **Shell Execution:** Creating commands that can run local shell scripts to gather context before running the prompt.

## At commands (`@`)

At commands are used to include the content of files or directories as part of your prompt to Gemini. These commands include git-aware filtering.

- **`@<path_to_file_or_directory>`**
  - **Description:** Inject the content of the specified file or files into your current prompt. This is useful for asking questions about specific code, text, or collections of files.
  - **Examples:**
    - `@path/to/your/file.txt Explain this text.`
    - `@src/my_project/ Summarize the code in this directory.`
    - `What is this file about? @README.md`
  - **Details:**
    - If a path to a single file is provided, the content of that file is read.
    - If a path to a directory is provided, the command attempts to read the content of files within that directory and any subdirectories.
    - Spaces in paths should be escaped with a backslash (e.g., `@My\ Documents/file.txt`).
    - The command uses the `read_many_files` tool internally. The content is fetched and then inserted into your query before being sent to the Gemini model.
    - **Git-aware filtering:** By default, git-ignored files (like `node_modules/`, `dist/`, `.env`, `.git/`) are excluded. This behavior can be changed via the `fileFiltering` settings.
    - **File types:** The command is intended for text-based files. While it might attempt to read any file, binary files or very large files might be skipped or truncated by the underlying `read_many_files` tool to ensure performance and relevance. The tool indicates if files were skipped.
  - **Output:** The CLI will show a tool call message indicating that `read_many_files` was used, along with a message detailing the status and the path(s) that were processed.

- **`@` (Lone at symbol)**
  - **Description:** If you type a lone `@` symbol without a path, the query is passed as-is to the Gemini model. This might be useful if you are specifically talking _about_ the `@` symbol in your prompt.

### Error handling for `@` commands

- If the path specified after `@` is not found or is invalid, an error message will be displayed, and the query might not be sent to the Gemini model, or it will be sent without the file content.
- If the `read_many_files` tool encounters an error (e.g., permission issues), this will also be reported.

## Shell mode & passthrough commands (`!`)

The `!` prefix lets you interact with your system's shell directly from within Gemini CLI.

- **`!<shell_command>`**
  - **Description:** Execute the given `<shell_command>` in your system's default shell. Any output or errors from the command are displayed in the terminal.
  - **Examples:**
    - `!ls -la` (executes `ls -la` and returns to Gemini CLI)
    - `!git status` (executes `git status` and returns to Gemini CLI)

- **`!` (Toggle shell mode)**
  - **Description:** Typing `!` on its own toggles shell mode.
    - **Entering shell mode:**
      - When active, shell mode uses a different coloring and a "Shell Mode Indicator".
      - While in shell mode, text you type is interpreted directly as a shell command.
    - **Exiting shell mode:**
      - When exited, the UI reverts to its standard appearance and normal Gemini CLI behavior resumes.

- **Caution for all `!` usage:** Commands you execute in shell mode have the same permissions and impact as if you ran them directly in your terminal.
