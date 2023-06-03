# git-hooks-secrets-handling
Provide a way how to encrypt automatically sensitive data (using sops and gpg) before committing data, and checking them out to worktree decrypted, possibly to let you edit them before committing and automatically encrypt them again.

## Objectives

- encrypt sensitive key values in files( according to keys name, and predefined sensitive key names) before committing them.
- supported files are - `json`, `yaml` and `env`.
- decrypted encrypted files that was encrypted before committing on the following events:
  1. When checking out a branch with encrypted data.
  2. When cloning a repo and the default branch contains encrypted files.
  3. When pulling or merging a branch , and the merge commit contains an encrypted file - TBD. 


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
  a. If environment variable `GIT_TEMPLATE_DIR` is defined with templates , then copy the files there. \
  b. Otherwise, if git configuration variable is defined with this directory, copy them to there, you can find if it's defined by the following command:
      ```shell
      git config --path --get init.templatedir
      ```
   c. Otherwise, to default directory `/usr/share/git-core/templates`
   
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

