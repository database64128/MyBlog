# Basic Git Usage

We introduce some basic Git usage for a quick start.

## 1. Change some configurations

List current `global` settings:
```bash
$ git config --global --list
```
In order to use Git to make changes to a repository, a name and an email must be set at least.
```bash
$ git config --global user.name  "John Doe"
$ git config --global user.email "johndoe@example.com"
```
Optionally, set the default editor to `nano`:
```bash
$ git config --global core.editor "nano -w"
```

## 2. Configure SSH keys for Git (Optional)

To use SSH to interact with a remote repository, you need to configure an SSH key and assign it to Git.

First generate an SSH key:
```bash
$ ssh-keygen -t ed25519
```

Then contact your online repository provider to add your public key.

Add the following to `~/.ssh/config`:
```
Host github.com
	hostname github.com
	User database64128
	IdentityFile ~/.ssh/github_database64128_laptop_20190306_ed25519
```

## 3. Get a repo to work on

Clone a repo from the web using `SSH`:
```bash
$ git clone git@github.com:database64128/MyBlog.git
```
Or using `https`:
```bash
$ git clone https://github.com/database64128/MyBlog.git
```
To create a repository from an existing directory of files, you can simply run `git init` in that directory.

## 4. Add, commit

Any file change including adding/renaming/moving/modifying requires an `add` operation to take effect in the eyes of `Git`.

`status -s` gives you a short overview of these changes.

```bash
$ git status -s
A  README.md
$ nano README.md
$ git status -s
AM README.md
$ git add README.md
```

When you are finished with changes and staged what you want to `add`, you run `git commit` to actually record the snapshot.

```bash
$ git commit -m "Fixed some typos"
```

## 5. Push, pull
To push your committed changes to a remote repository, simply use `push`:
```bash
$ git push origin master
```

To fetch and automatically merge all changes from a remote repository, use `pull`:
```bash
$ git pull
```