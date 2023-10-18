# git-hooks-secrets-handling
Provide a way how to encrypt automatically sensitive data (using sops and gpg) before committing data, and checking them out to worktree decrypted, possibly to let you edit them before committing and automatically encrypt them again.

## Objectives

- encrypt sensitive key values in files( according to keys name, and predefined sensitive key names) before committing them.
- supported files are - `json`, `yaml` and `env`.
- decrypted encrypted files that was encrypted before committing on the following events:
  1. When checking out a branch with encrypted data.
  2. When cloning a repo and the default branch contains encrypted files.
  3. When pulling or merging a branch , and the merge commit contains an encrypted file - TBD.
  4. logic will be disabled by default , in order to activate it, need to set environment variable GIT_AUTO_ENC_DEC=true in the terminal session.


## Prerequisites.

1. Need to install [sops](https://github.com/mozilla/sops/releases) 
2. Need to install [GnuPg](https://gnupg.org/download/)
3. Need to create key-pair on local machine using GPG
```shell
# Create key pair
gpg --batch --passphrase '' --quick-gen-key your@email.com default default
# Make sure it's registered and defined
gpg --fingerprint your@email.com
```

4. Need to install [jq](https://jqlang.github.io/jq/download/) and [yq](https://mikefarah.gitbook.io/yq/v/v3.x/) 
5. Need to copy files [post-checkout](./post-checkout) and [pre-commit](./pre-commit) to default templates directory `/usr/share/git-core/templates`, or to the location of git templates in your system: \
  a. If environment variable `GIT_TEMPLATE_DIR`/hooks is defined with templates , then copy the files there. \
  b. Otherwise, if git configuration variable is defined with this directory, copy them to there, you can find if it's defined by the following command:
      ```shell
      echo $(git config --path --get init.templatedir)/hooks 
      ```
   c. Otherwise, to default directory `/usr/share/git-core/templates/hooks`
   
6. The key-pairs generated in section 3 should be shared in a secret place in a secured way, so 
   all individuals working on the repo should have the keys and only them, so they will be able to encrypt and decrypt files containing secret data, this way repo with sensitive data can be defined with public scope , if only sensitive data is the only concern and reason  of the repo to be private.

7. The one that created the keys should export the keys to files , with the following procedure:
```shell
gpg --export -a --armor your@email.com > ./public.key
gpg --export -a --armor your@email.com > ./public.key
```

8. all others in his team that needs access to work on secrets data, should import the files ( after fetch them in a secured way from vault or some other secrets management solution):
```shell
gpg --import public.key
gpg --allow-secret-key-import --import private.key
```

9. Plant your fingerprint id instead of mine in the pre-commit hook
```shell
sed -i 's/zgrinber@redhat.com/your@email.com/g' pre-commit 
```

## Demo Procedure

* In order to activate the logic for the current terminal session, kindly turn of environment variable  `GIT_AUTO_ENC_DEC`
```shell
export GIT_AUTO_ENC_DEC=true
```

1. If the git is new cloned repository or newly locally created repo using `git init` command, then hooks will be fetched from template directory automatically, otherwise, if the repository created before hooks templates were modified, then need to refresh the directory of the repo:
```shell
git init
```

2. Create 6 files - 2 jsons, 2 yamls, 2 environment files - for each pair, one will be with sensitive data, and one without sensitive data

file-with-secrets.env
```shell
cat > file-with-secrets.env << EOF 
password=myPassword
token=HK534g82h21kkS
name=Moshe
profession=doctor
key=value
ke2=value2
animal=horse
secret=veryConfidentialSecret
hobby=hunting
movie=taxiDriver
credentials=hiddenCredentials
EOF
```

file-no-secrets.env
```shell
cat > file-no-secrets.env << EOF 
name=Moshe
profession=doctor
key=value
ke2=value2
animal=horse
hobby=hunting
movie=taxiDriver
EOF
```

file-with-secrets.json
```shell
cat > file-with-secrets.json << EOF 
{
  "password": "myPassword",
  "token": "HK534g82h21kkS",
  "name": "Moshe",
  "profession": "doctor",
  "age": "54"
}

EOF
```
file-no-secrets.json
```shell
cat > file-no-secrets.json << EOF 
{
  "name": "Moshe",
  "profession": "doctor",
  "age": "54"
}
EOF
```

file-with-secrets.yaml
```shell
cat > file-with-secrets.yaml << EOF 
password: myPassword
token: HK534g82h21kkS
name: Moshe
profession: doctor
key: value
ke2: value2
animal: horse
secret: veryConfidentialSecret!!
EOF
```
file-no-secrets.yaml
```shell
cat > file-no-secrets.yaml << EOF 
name: Moshe
profession: doctor
key: value
ke2: value2
animal: horse
EOF
```

3. Add all file to index (Stage them)
```shell
git add file-*
```

4. show that all created files are staged
```shell
git status
```
Output:
```shell
[zgrinber@zgrinber git-hooks-secrets-handling]$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        new file:   file-no-secrets.env
        new file:   file-no-secrets.json
        new file:   file-no-secrets.yaml
        new file:   file-with-secrets.env
        new file:   file-with-secrets.json
        new file:   file-with-secrets.yaml
```

5. Now commit the new content
```shell
git commit -sm "encrypt and decrypt demo"
```
Output:
```shell
command "gpg" exists on system
command "sops" exists on system
command "jq" exists on system
command "yq" exists on system
processing file name - file-no-secrets.env
decision for dotenv file file-no-secrets.env : 1
processing file name - file-no-secrets.json
decision for json file file-no-secrets.json : 1
processing file name - file-no-secrets.yaml
decision for yaml file file-no-secrets.yaml : 1
processing file name - file-with-secrets.env
found file-with-secrets.env with sensitive content, determinded to encrypt
decision for dotenv file file-with-secrets.env : 0
[PGP]    WARN[0000] Deprecation Warning: GPG key fetching from a keyserver within sops will be removed in a future version of sops. See https://github.com/mozilla/sops/issues/727 for more information. 
processing file name - file-with-secrets.json
found file-with-secrets.json with sensitive content, determinded to encrypt
decision for json file file-with-secrets.json : 0
[PGP]    WARN[0000] Deprecation Warning: GPG key fetching from a keyserver within sops will be removed in a future version of sops. See https://github.com/mozilla/sops/issues/727 for more information. 
processing file name - file-with-secrets.yaml
found file-with-secrets.yaml with sensitive content, determinded to encrypt
decision for yaml file file-with-secrets.yaml : 0
[PGP]    WARN[0000] Deprecation Warning: GPG key fetching from a keyserver within sops will be removed in a future version of sops. See https://github.com/mozilla/sops/issues/727 for more information. 
[main 053dab1] encrypt and decrypt demo
 9 files changed, 94 insertions(+), 76 deletions(-)
 create mode 100644 file-no-secrets.env
 create mode 100644 file-no-secrets.json
 create mode 100644 file-no-secrets.yaml
 create mode 100644 file-with-secrets.env
 create mode 100644 file-with-secrets.json
 create mode 100644 file-with-secrets.yaml
```
Note: According to logs of pre-commit hook, it appears that only files with sensitive data were actually encrypted.
you can verify it your self that all files "with-secrets" are encrypted (only sensitive data inside them) , and all "no-secrets" files are all in plain-text.


7. Let's see the status
```shell
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)
```

6. Now that all sensitive data was encrypted, you can perform git checkout HEAD to decrypt encrypted files in order to modify the secrets, and then follow the seqeuence of add -->commit, and an automatic new encryption will be triggered again on new data.
```shell
git checkout 
```
Output:
```shell
moving from branch 053dab15583f3df5cf91fe83e70546cdeb470394 to branch 053dab15583f3df5cf91fe83e70546cdeb470394, and decrypting all yaml, json and env files in new branch that are encrypted.
Object 053dab15583f3df5cf91fe83e70546cdeb470394 has no note
decrypting file-with-secrets.env
successfully decrypted file-with-secrets.env
decrypting file-with-secrets.json
successfully decrypted file-with-secrets.json
decrypting file-with-secrets.yaml
successfully decrypted file-with-secrets.yaml
```

7. Run git status to see the change
```shell
git status
```
Output:
```shell
On branch main
Your branch is ahead of 'origin/main' by 1 commit.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   file-with-secrets.env
        modified:   file-with-secrets.json
        modified:   file-with-secrets.yaml
```

8. Now these files are decrypted and in text-plain, you can edit them, and repeat steps 3-5 in order to apply new changes to the files and automatically encrypt them.
9. In case you don't want to change them , then just run the following command ( it will only remove decrypted changes from worktree, all your unstaged work will be preserved)
```shell
git notes show HEAD | grep "\S" | xargs -i git restore $(git rev-parse --show-toplevel)/{} --worktree
```
10. You can simplify the above by defining an alias to do that more easily.
```shell
alias throw-decrypted-files="git notes show HEAD | grep \"\S\" | xargs -i git restore $(git rev-parse --show-toplevel)/{} --worktree"
throw-decrypted-files
```
