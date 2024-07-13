# Coding and data training (Python/SQL focused)

Resources, tools, and tricks to understanding the data/coding landscape.

## Simple command-line navigation and setup

The command line can be a really useful tool for things like installing packages and quick file movement. There are also many command-line interface (or CLI) tools that help with things like google cloud operations, and you can quickly make python scripts into command line tools. It helps to know a few basics before continuing.

On a Mac, the Terminal app is a useful way to access a CLI, and it automatically uses zsh, which is very similar to Linux's bash. Most commands here will assume you use something like the Terminal to run the commands. Open a Terminal window (it may take a few seconds to login), enter a command, like `echo Robbie Polski`, and then press enter. Unless otherwise specified, brackets in the command indicate entering your own version of the info. For instance `echo [your name]` would mean using the command `echo Robbie Polski`. If there are multiple lines in the command, you can enter each line of the command and press enter before moving to the next line. Here are a few quick commands:

- The first command to get used to is switching directories with `cd [new-directory]`. Include a period in the directory name if you want to start from the current directory. Two periods means move backward a directory. A tilde `~` means your home directory. Switch to your documents folder with `cd ~/documents` (note that zsh, which you are using, is case-insensitive). You could have also changed first to home with `cd ~` and then `cd documents` or `cd ./documents`
- List what is available in the current directory with the `ls` command. If you need to see any hidden files or files with a period at the start of the name, use `ls -a`, and `ls -l` will include extra details about each file or directory, like its permissions, size, and its last modified time.
- Find the current directory with `pwd`
- Copy with `cp [path/to/old-file] [path/to/new-file]`
- Move files, or change file names with `mv [path/to/original_file] [path/to/new_file]`
- Make new directories with `mkdir [new-directory]`
- You can copy-paste text into the terminal command by using `cmd+c` and `cmd+v`.
- Store basic commands to run on starting Terminal in the `~/.zshrc` file. If you do not have the file already, go to your home directory with `cd ~` and then use `touch .zshrc` to create it.
- The `PATH` variable contains which directories are searched for commands. So, for instance, when you enter `python file.py` to run the `file.py` python script, the directories stored in the `PATH` environment variable are searched for a python file to use. If you have multiple python versions, you can choose which one to use, by default, by adding the path to the executable to your `PATH`. For instance, add a line to the .zshrc file: `export PATH="[path/to/directory]:$PATH"`. Some scripts here will automatically load directions like this to the .zshrc file.

### Vim absolute basics

You might encounter a vim editor at some point, which is a text editor in the CLI. You will know when you encounter it if you see multiple lines of text (or blank lines) that you can move around in using the arrow keys. (If there is no text, it will appear frozen). It helps to know the following commands:

- Insert text by pressing the `i` key and then writing the text. You then have to press the escape key before entering other commands to escape the insert method.
- Exit the editor with `:wq` (then enter), which saves the file.
- If you want to exit without saving changes, use `:q!` (then enter).
- If you want any more help with vim, use the command `:help` at any point from the vim editor. Or from the main Terminal, the `vimtutor` command will run you through a decent more in-depth tutorial.

### Terminal setup

There are also ways to configure the Terminal. For instance, add the following lines to the file named .zshrc (you can add them using vim by first typing `vim ~/.zshrc` or by simply opening the file and copy-pasting):

```bash
# Command line prompt colors

function parse_git_branch() {
    git branch 2> /dev/null | sed -n -e 's/^\* \(.*\)/[\1]/p'
}

COLOR_DEF=$'\e[0m'
COLOR_USR=$'\e[38;5;243m'
COLOR_DIR=$'\e[38;5;197m'
COLOR_GIT=$'\e[38;5;39m'
setopt PROMPT_SUBST
export PROMPT='%{$COLOR_USR%}%n %{$COLOR_DIR%}%1d %{$COLOR_GIT%}$(parse_git_branch)%{$COLOR_DEF%}$'
```

This will automatically display your username, current directory, and current git branch (if applicable) with different colors before the Terminal prompt.

## Repeatable installations

Although it may appear confusing at first, you won't regret precisely defining your installation in the long term. In general, the idea is to keep your entire setup minimal and extremely reproducible, by yourself on another workstation, by other individual users, by the test and prod servers/programs, etc. You are going to forget how exactly you set it up, particularly if it involved a bunch of untracked command line procedures. So write it down in documentation, modify the docs if you see that it works differently on Windows vs. Mac, etc. This is the basis of automated infrastructure as code, as well. You want to make sure you have the same setup and that it can be spun up quickly somewhere else.

Python and some other languages, with multitudes of packages, quickly become nightmares when tracking dependencies between different people working on the same project and local vs. deployment operations. Most packages simply specify to use `pip install [PKG]` or `conda install [PKG]`, and you're done. This stores python packages in a single base location. While this works pretty well at first, you will eventually run into the problem of needing different versions of a package for different repos. You will also need to specify which packages were required to run the commands in your current repo. This gets hard to track if your base python includes all the packages from other repos on your computer as well.

### Virtual environments

A useful solution, that's accepted in many circles, is a python environment specifically defined and stored locally for each repository. This stores all the repo-specific packages very close to the repo and makes them more easily searchable. Common text editors like VSCode can easily parse import statements and provide shortcuts based on the python tools available to the repo using this method, as well. This is the "virtual environment method", and there are a few ways to do it, but you can start with

```sh
python3 -m venv .venv
```

You will see a new folder appear called .venv that includes a pyvenv.cfg file and some folders related to running python, and the lib/ folder that includes packages. If you look at pyvenv.cfg (a text file), you will see the base location of the python you're using here and the version, like 3.12.4. If you want a different python version, keep going. We'll get there. Use what you have for now. Make sure this .venv folder is included in your .gitignore, so all the python packages don't get unnecessarily checked into git.

To use this virtual environment directly from a command prompt, you have to remember to activate it. When in the base repo location, use

```sh
source activate .venv/bin/activate
```

and you can use `deactivate` at any time to exit from the virtual environment.

Install a package with `pip install numpy`. You will see the package folder(s) now at `.venv/lib/pythonx.xx/site-packages/numpy/`. Since numpy is a fundamental package, it doesn't have any other dependencies, so this is all that installs. Install another package with `pip install pillow` (a simple image processing library), and then check your packages with `pip list`.

Use `pip freeze > requirements.txt` to make a list of your installed packages and versions in the `requirements.txt` text file. Now anyone can install this list of packages by downloading this requirements file and running `pip install -r requirements.txt`. If you are making a simple package with simple dependencies, this may be all you need. You can check requirements.txt into version control and provide others with the ability to reproduce your installation. However, many packages like Pandas, FastAPI, MLFlow, and Scikit-learn involve complex sets of dependencies that have additional issues.

### The problem with pip install

Now install a package with lots of dependencies. Use `pip install mlflow` to install the machine learning experiment tracking package. When you use the command `pip list` now, you will see a long list of packages, only two of which are the packages you ultimately want: mlflow and numpy. These additional packages are dependencies of mlflow.

Now uninstall the mlflow package with `pip uninstall mlflow`. When you use `pip list` again now, you will see that all the dependency packages are left behind, and only mlflow was removed. You can now see how hard it will be to clean up old packages that you don't want. Maybe you want to keep the `pyarrow` package because it's a dependency for a necessary package, but maybe it's left behind from a package you're no longer using. Additionally, if you are trying to slim down your configuration to make it quick to install on a prod server, you need something different.

### Modern python dependency management

There are useful tools these days that help manage dependencies more robustly and that provide the added benefit of treating your repo like a python package. This makes it easy to build and publish your package with a few extra button presses, and it automatically supplies dependency lists to anyone installing the package as well.

PDM is what we will use here, but poetry is another version that is similar. First get rid of your previous virtual environment by deactivating and then using `git rm -rf .venv` to erase it, and `git rm requirements.txt`. Then install PDM somehow. If on a Mac, use homebrew: `brew install pdm` (get homebrew [here](https://brew.sh/)), or follow the PDM installation commands on [this page](https://pdm-project.org/en/latest/).

PDM allows easy installation and switching between python versions. First make sure you have the desired python version installed. You can check what is available with `pdm python list`. Find the version you want, like python 3.12, and install with `pdm python install 3.12`. Now this version is available to you for any repo through PDM, however it has not been attached to your current repo yet.

Use PDM to create a virtual environment with `pdm venv create 3.12` (or whatever version you want). This will create a virtual environment with the desired python version. If you have not installed the desired version yet, you may need to retry `pdm python install [VERSION]` and then `pdm venv create [VERSION]` again.

Now you may notice that if you activate the environment (same way with `source .venv/bin/activate`), you cannot run `pip install numpy` because pip is not installed. The following method is the replacement.

### pyproject.toml and dependency management

Copy over the pyproject.toml file from this repository and replace the text like name, author, email, description. In the `requires-python` field, you can specify for someone to use a certain range of python versions. Specify your license with a [python classifier](https://pypi.org/classifiers/) if possible, or you can use a text file (`license = {file = LICENSE.txt}`). The classifier "Private :: Do Not Upload" is useful for private company software as well. I used the open-source Apache Software License here.

Erase the dependencies section in the current pyproject.toml file to build dependencies from scratch.

A few other fields aren't overly important right now. For instance, the `version` field specifies the current version of the python package when publishing (not important otherwise). The `build-system` section of the file is used for building packages and can be kept the same. In the `tool.pdm` section, the `distribution` field specifies whether to treat it like a library/package (if true, it will be included in your installed packages on the command line for testing).

There is much more you can do with the [pyproject.toml](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/) file, but we won't go into too many details here.

For adding dependencies, like numpy and mlflow, simply use `pdm add numpy mlflow` (can do one package at a time or any number of packages at a time). A few things happened in that automated command:

1. The dependency structure of the requested packages was worked out so they are compatible with each other.
2. These two packages are added to the pyproject.toml file under `dependencies` (locked to >= the current versions).
3. A pdm.lock file was created that more specifically records what packages are installed. This file also includes hash details to nail down the specific version and equivalent versions on different operating systems.

List all of your packages with `pdm list` or `pdm list --tree`.

Now try removing mlflow with `pdm remove mlflow`. You will see that all the dependency packages were removed as well.

Now you have a full specification of your packages in pdm.lock for repeatable installations, but you have a list of the essentials in pyproject.toml and easy methods for cleanup. There are a few extra files in the .gitignore here that you should add, but you should check the pyproject.toml and pdm.lock files into version control. When someone clones your repo, they can simply setup the venv and then run `pdm sync` to quickly reproduce the same configuration. You should try this yourself by committing and pushing the new changes to GitHub, and cloning the repo to a new folder. Once cloned, use `pdm venv create 3.12` and then `pdm sync`, and you should find the same python environment.

Read through the PDM project page, and specifically the [dependency management section](https://pdm-project.org/en/latest/usage/dependency/) for more interesting use cases.

### Repeatable installation summary

PDM supports repeatable and maintainable python installations and sets your repo up to be used as a python package that can be easily distributed to others. Through PDM, you can select the python version you want and install a virtual environment with `pdm venv create [VERSION]`. If you are starting a new repo, setup a pyproject.toml file, and use `pdm add [PKG]` to add packages and `pdm remove [PKG]` to remove packages. If you are cloning someone else's repo or want to sync your virtual environment with a different branch of your own, make sure you have the pyproject.toml and pdm.lock files and just use `pdm sync`.
