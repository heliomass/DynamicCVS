# Dynamic CVS
Dynamic CVS aims to bring some command line features of Git to CVS.

## What is It?
Since I have to work with CVS on a daily basis, this project was born out of general frustration with some of its limitations, especially the constant need to contact the CVS server for nearly all operations.

Dynamic CVS aims to draw inspiration from Git by providing you with the following information without the need to hit the server each time:  
- Am I in a CVS directory?  
- What branch am I on?  
- Are there any uncommitted files?

You can then use this information to display information about the current directory, similar to Git:

```
cvs:(MY_CVS_BRANCH) ✗
```

Dynamic CVS has an expandable hook system, meaning new commands can be added in the future, and existing commands can have extra functionality added either prior to or after their execution.

## Usage
### Step 1: Clone the Repository and Set Up the Path
Clone this repository and ensure that your `PATH` environment variable includes the directory containing `dynamic_cvs`.

### Step 2: Create an Alias around CVS
In your `.bashrc` (or shell equivalent of that file) add the following line:

```shell
alias cvs='dynamic_cvs'
```

### Step 3: Set Up Your Prompt
If you're a ZSH user, you'll probably already have a prompt element to display Git information. You can implement something similar for CVS now by adding the following to your `PROMPT` variable:

```shell
PROMPT=\
... your current prompt lines... \
%{$fg_bold[blue]%}$(which dynamic_cvs > /dev/null && dynamic_cvs has && echo "cvs:(%{$fg_bold[red]%}$(dynamic_cvs branchname)%{$fg_bold[blue]%}) %{$fg_bold[yellow]%}$(dynamic_cvs uncommitted)") \
... the rest of your prompt lines...
```

And for Bash users, something like this for your prompt:
```shell
PS1='$(which dynamic_cvs 2> /dev/null && dynamic_cvs has && echo "cvs:($(dynamic_cvs branchname)) $(dynamic_cvs uncommitted)")$ '
```

Alternatively, you can just manually invoke `cvs branchname` or `cvs uncommitted` from the command line.

Note that when entering a directory for the first time with Dynamic CVS, a question mark symbol will display until you've run `cvs update` at least once.

Also, running `cvs has` will tell you if there's CVS data in the current directory by means of an exit code (0 if there is, 1 if not).

## How Does it Work?
Dynamic CVS works by keeping a hash of the file structure for the current directory and its children. It does this in two ways.

Firstly, it creates hooks around CVS commands like `update` and `commit`. These hooks update the hash as necessary to reflect the state of the remote repository on the server.

Secondly, new commands like `cvs uncommitted` uses these hashes to compare the current directory state with its view of the server state.

Dynamic CVS can also show you the current branch name without needing to contact the server based on local information that CVS stores.

## Outstanding Items
Dynamic CVS is a work in progress. Below is a list of to-do items which are needed to improve the experience.

- Depending on the number of files in the directory, the hash can take a moment to calculate resulting in a perceivable delay to your prompt when in a CVS directory.  
- Configurable symbols from the environment (eg: an alternative to "✗" for uncommitted files)  
- Ignore CVS tags within files (eg: `$Id$` and `$Header$`) since these can change without the file needing to be recommitted.  
- Running `cvs update` should set up all subdirectories for Dynamic CVS.   Currently if you jump into a sub-directory, you may need to run `cvs update` manually to get the latest info.  
- Enable a pipe for the `cvs update` hook. Currently there's no screen output to stdout until the cvs command has completed, which doesn't make for a great user experience.  
- When running a CVS command as a dry run (ie: `cvs -n ...`) we shouldn't update any of Dynamic CVS' info.  
- If you commit only some changes files, Dynamic CVS doesn't handle this situation particularly gracefully.
- Checking out a repository should initialise Dynamic CVS.
