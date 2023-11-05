<!-- LTeX: enabled=false -->
# nvim-tinygit
<!-- LTeX: enabled=true -->
<a href="https://dotfyle.com/plugins/chrisgrieser/nvim-tinygit"><img src="https://dotfyle.com/plugins/chrisgrieser/nvim-tinygit/shield"/></a>

Lightweight and nimble git client for nvim.

<img src="https://github.com/chrisgrieser/nvim-tinygit/assets/73286100/009d9139-f429-49e2-a244-15396fb13d7a"
	alt="showcase commit message input field"
	width=65%>

*Commit Message Input with highlighting*

<img src="https://github.com/chrisgrieser/nvim-tinygit/assets/73286100/123fcfd9-f989-4c10-bd98-32f62ea683c3"
	alt="showcase commit message notification"
	width=50%>

*Informative notifications with highlighting (using `nvim-notify`)*

<img src="https://github.com/chrisgrieser/nvim-tinygit/assets/73286100/99cc8def-760a-4cdd-9aea-fbd1fb3d1ecb"
	alt="Pasted image 2023-10-11 at 18 49 40"
	width=60%>

*Search File history ("git pickaxe") and inspect the commit diffs.*

## Table of Contents

<!-- toc -->

- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
	* [Smart-Commit](#smart-commit)
	* [Amend](#amend)
	* [Fixup & Squash Commits](#fixup--squash-commits)
	* [GitHub Interaction](#github-interaction)
	* [Push](#push)
	* [Search File History ("git pickaxe")](#search-file-history-git-pickaxe)
	* [Stash](#stash)
- [Improved Interactive Rebasing](#improved-interactive-rebasing)
- [Configuration](#configuration)
	* [Appearance of Input Field](#appearance-of-input-field)
	* [Use Telescope for selections](#use-telescope-for-selections)
- [Non-Goals](#non-goals)
- [Credits](#credits)

<!-- tocstop -->

## Features
- **Smart-Commit**: Open a popup to enter a commit message with syntax
  highlighting and indicators for [commit message
  overlength](https://stackoverflow.com/questions/2290016/git-commit-messages-50-72-formatting).
  If there are no staged changed, stages all changes before doing so (`git add -A`).
- Commit messages have syntax highlighting, indicators for [commit message
  overlength](https://stackoverflow.com/questions/2290016/git-commit-messages-50-72-formatting),
  and options to automatically open references GitHub issues in the browser
  after committing, `git push` if the repo is clean, spellcheck, enforce
  conventional commits, …
- Quick commands for amend, stash, fixup, and squash commits.
- Search issues & PRs. Open the selected issue or PR in the browser.
- Open the GitHub URL of the current file or selection.
- Search the file history for a string ("git pickaxe"), show results in a diff
  with filetype syntax highlighting.
- Improvements for interactive rebasing with nvim as sequence editor.

## Installation

```lua
-- lazy.nvim
{
	"chrisgrieser/nvim-tinygit",
	ft = { "gitrebase", "gitcommit" }, -- so ftplugins are loaded
	dependencies = {
		"stevearc/dressing.nvim",
		"rcarriga/nvim-notify", -- optional, but recommended
	},
},

-- packer
use {
	"chrisgrieser/nvim-tinygit",
	requires = {
		"stevearc/dressing.nvim",
		"rcarriga/nvim-notify", -- optional, but recommended
	},
}
```

Install the Treesitter parser for git commits for some syntax highlighting of
your commit messages like emphasized conventional commit keywords: `TSInstall
gitcommit`

## Usage

### Smart-Commit
- Open a commit popup. If there are no staged changes, stage all changes (`git
  add -A`) before the commit. Only supports the commit subject line.
- Optionally run `git push` if the repo is clean after committing.
- The title of the input field displays what actions are going to be performed.
  You can see at glance, whether all changes are going to be committed or whether
  there a `git push` is triggered afterward, so there are no surprises.

```lua
require("tinygit").smartCommit { pushIfClean = false } -- options default to `false`
```

**Example Workflow**
Assuming these keybindings:

```lua
vim.keymap.set("n", "ga", "<cmd>Gitsigns add_hunk<CR>") -- gitsigns.nvim
vim.keymap.set("n", "gc", function() require("tinygit").smartCommit() end)
vim.keymap.set("n", "gp", function() require("tinygit").push() end)
```

1. Stage some hunks (changes) via `ga`.
2. Use `gc` to enter a commit message.
3. Repeat 1 and 2.
4. When done, `gp` to push the commits.

Using `pushIfClean = true` allows you to combine staging, committing, and
pushing into a single step, when it is the last commit you intend to make.

```lua
-- to enable normal mode in the commit message input field, configure
-- dressing.nvim like this:
require("dressing").setup({ 
	input = { insert_only = false }
})
```

### Amend
- `amendOnlyMsg` just opens the commit popup to change the last commit message,
  and does not stage any changes.
- `amendNoEdit` keeps the last commit message; if there are no staged changes,
  it stages all changes (`git add -A`), like `smartCommit`.
- Optionally runs `git push --force` afterward. (Remember to only do this when
  you work alone on the branch though.)

```lua
-- options default to `false`
require("tinygit").amendOnlyMsg { forcePush = false }
require("tinygit").amendNoEdit { forcePush = false }
```

### Fixup & Squash Commits
- `fixupCommit` lets you select a commit from the last X commits and runs `git
  commit --fixup` on the selected commit (that is, marking the commit for a
  future `git rebase --autosquash`).
- Use `squashInstead = true` to squash instead of fixup (`git commit --squash`)

```lua
-- options show default values
require("tinygit").fixupCommit { 
	selectFromLastXCommits = 15
	squashInstead = false, 
}
```

### GitHub Interaction
- Search issues & PRs. (Requires `curl`.)
- The appearance of the selector is controlled by `dressing.nvim`. (You can
  [configure `dressing` to use `telescope`](#use-telescope-for-selections).)

```lua
-- state: all|closed|open (default: all)
-- type: all|issue|pr (default: all)
require("tinygit").issuesAndPrs { type = "all", state = "all" }

-- alternative: if the word under the cursor is of the form `#123`,
-- just open that issue/PR
require("tinygit").openIssueUnderCursor()
```

- Open the current file at GitHub in the browser and copy the URL to the system clipboard.
- Normal mode: open the current file or repo.
- Visual mode: open the current selection.

```lua
-- file|repo (default: file)
require("tinygit").githubUrl("file")
```

### Push

```lua
-- options default to `false`
require("tinygit").push { pullBefore = false, force = false }
```

### Search File History ("git pickaxe")
- Search the git history of the current file for a term ("git pickaxe").
- The search is case-insensitive and supports regex.
- Select from the matching commits to open a diff popup.
- Keymaps in the diff popup:
	* `n`/`N`: go to the next/previous occurrence of the query.
	* `<Tab>`/`<S-Tab>`: cycle through the commits.
	* `yh`: yank the commit hash to the system clipboard.

```lua
require("tinygit").searchFileHistory()
```

### Stash

```lua
require("tinygit").stashPush()
require("tinygit").stashPop()
```

## Improved Interactive Rebasing
`tinygit` also comes with some improvements for interactive rebasing (`git
rebase -i`) with nvim: 
- Improved syntax highlighting of commit messages.
- `<Tab>` (normal mode): Cycle through the common rebase actions: `pick`,
  `reword`, `fixup`, `squash`, `drop`. Also supports their short forms.

> [!NOTE]
> This requires that your git editor (or sequence editor) is set to use `nvim`.
> You can do so with `git config --global core.editor "nvim"`.


If you want to disable those modifications, set:

```lua
vim.g.tinygit_no_rebase_ftplugin = true
```

## Configuration
The `setup` call is optional. These are the default settings:

```lua
local defaultConfig = {
	commitMsg = {
		-- Why 50/72 is recommended: https://stackoverflow.com/q/2290016/22114136
		mediumLen = 50,
		maxLen = 72,

		-- When conforming the commit message popup with an empty message, fill
		-- in this message. `false` to disallow empty commit messages.
		emptyFillIn = "chore", ---@type string|false

		-- disallow commit messages without a conventional commit keyword
		enforceConvCommits = {
			enabled = false,
			-- stylua: ignore
			keywords = {
				"chore", "build", "test", "fix", "feat", "refactor", "perf",
				"style", "revert", "ci", "docs", "break", "improv",
			},
		},

		-- enable vim's builtin spellcheck for the commit message input field
		-- (configured to ignore capitalization and correctly consider camelCase)
		spellcheck = false, 

		-- if commit message references issue/PR, open it in the browser
		openReferencedIssue = false,
	},
	asyncOpConfirmationSound = true, -- currently macOS only
	issueIcons = {
		openIssue = "🟢",
		closedIssue = "🟣",
		openPR = "🟩",
		mergedPR = "🟪",
		closedPR = "🟥",
	},
	searchFileHistory = {
		diffPopupWidth = 0.8, -- float, 0 to 1
		diffPopupHeight = 0.8, -- float, 0 to 1
		diffPopupBorder = "single",
	},
}
```

### Appearance of Input Field

```lua
-- see: https://github.com/stevearc/dressing.nvim#configuration
require("dressing").setup({ 
	input = { 
		insert_only = false, -- enable normal mode in the input field
		-- other appearance settings
	}
})
```

### Use Telescope for selections

```lua
-- see: https://github.com/stevearc/dressing.nvim#configuration
require("dressing").setup({ 
	select = { 
		backend = { "telescope" },
		-- other appearance settings
	}
})
```

## Non-Goals
- Become a full-fledged git client. Use
  [neogit](https://github.com/NeogitOrg/neogit) for that.
- Add features available in
  [gitsigns.nvim](https://github.com/lewis6991/gitsigns.nvim). `tinygit` is
  intended to complement `gitsigns.nvim` with some simple commands, not replace
  it.
- UI Customization. Configure
  [dressing.nvim](https://github.com/stevearc/dressing.nvim) for that.

<!-- vale Google.FirstPerson = NO -->
## Credits
**About Me**  
In my day job, I am a sociologist studying the social mechanisms underlying the
digital economy. For my PhD project, I investigate the governance of the app
economy and how software ecosystems manage the tension between innovation and
compatibility. If you are interested in this subject, feel free to get in touch.

**Blog**  
I also occasionally blog about vim: [Nano Tips for Vim](https://nanotipsforvim.prose.sh)

**Profiles**  
- [reddit](https://www.reddit.com/user/pseudometapseudo)
- [Discord](https://discordapp.com/users/462774483044794368/)
- [Academic Website](https://chris-grieser.de/)
- [Twitter](https://twitter.com/pseudo_meta)
- [Mastodon](https://pkm.social/@pseudometa)
- [ResearchGate](https://www.researchgate.net/profile/Christopher-Grieser)
- [LinkedIn](https://www.linkedin.com/in/christopher-grieser-ba693b17a/)

<a href='https://ko-fi.com/Y8Y86SQ91' target='_blank'>
<img
	height='36'
	style='border:0px;height:36px;'
	src='https://cdn.ko-fi.com/cdn/kofi1.png?v=3'
	border='0'
	alt='Buy Me a Coffee at ko-fi.com'
/></a>
