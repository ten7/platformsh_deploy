# Platformsh Deploy

An Ansible role to deploy a site to Platformsh from an external git repo.

# Overview

Platformsh has the ability to deploy sites over standard git. Often, however, you may wish to maintain your site in your own repository to take advantage of additional tooling such as Pull Requests, code reviews, and issue tracker integration.

Beyond integration, you may also want to:
* Use a different branch strategy such as Git Flow.
* Use `main` instead of `master` for your primary branch.

This role allows you to do all of these things.

## How it works

This role does *not* configure a multi-remote git repository. Instead, the Platformsh repository is cloned in a temporary directory and then changes are rsynced to it from the source repository. This avoids git conflicts during the build process.

Furthermore, it allows some differences to exist between the two repos in key ways.

### Platformsh-specific gitignore

In your source repository, you can define a `.gitignore-platformsh` file. This file will not be used for ignores by git in your source respository. During the build, however, it is copied to `.gitignore` in the Platformsh repository, allowing you to create different ignores for the two sites. It is highly recommended to copy the `.gitignore` supplied by Platformsh when initially setting up your site.

## Requirements

* The ansible.posix collection must be installed.
* rsync must be installed
* git must be installed
* If using npm, npm must be installed.
* The target platformsh site must be in git mode.

## Role Variables

```yaml
platformsh_deploy:
  source:
    git_dir: 'path/to/source/git/dir'
  target:
    ssh_key_base64: 'abcdef1234567890'
    ssh_pub_base64: 'abcdef1234567890'
    platformsh_machine_token: ''
    site_id: ''
    env_id: ''
    repo_url: "ssh://codeserver.dev.abcd-ef12-3456-7890@codeserver.dev.abcd-ef12-3456-7890.drush.in:2222/~/repository.git"
    git_branch: 'master'
    git_commit_message: "Made with <3 by robots"
```

Where:

* `platformsh_deploy.source.git_dir` is the path to locally cloned git repository from the external git host. It assumed that this repo is already cloned and on the expected branch for the deploy. Required.
* `platformsh_deploy.target.ssh_key_base64` is the Base64 encoded SSH private key used to communicate with Platformsh. Required.
* `platformsh_deploy.target.ssh_pub_base64` is the Base64 encoded SSH public key used to communicate with Platformsh. Required.
* `platformsh_deploy.target.repo_url` is the SSH URL to the Platformsh site repository. Required.
* `platformsh_deploy.target.git_branch` is the git branch to push to the Platformsh site repository. Required.
* `platformsh_deploy.target.git_commit_message` is the git commit message to use when pushing to the Platformsh site repository. Optional, defaults to "Commit by ten7.platformsh_deploy".

### Custom tasks

Often, you may wish to execute custom CI code during the build and deployment process. You can accomplish that with `include_tasks`:

```yaml
platformsh_deploy:
  source:
    git_dir: 'path/to/source/git/dir'
  build:
    include_tasks:
      - "path/to/my/build.yml"
  target:
    ssh_key_base64: 'abcdef1234567890'
    ssh_pub_base64: 'abcdef1234567890'
    platformsh_machine_token: ''
    site_id: ''
    env_id: ''
    repo_url: "ssh://codeserver.dev.abcd-ef12-3456-7890@codeserver.dev.abcd-ef12-3456-7890.drush.in:2222/~/repository.git"
    git_branch: 'master'
    git_commit_message: "Made with <3 by robots"
  post_deploy:
    include_tasks:
      - "path/to/my/post_deploy.yml"
```

Where:

* `platformsh_deploy.build.include_tasks` is a list of paths to an Ansible tasks file to execute during the build but before the deploy. Optional.
* `platformsh_deploy.post_deploy.include_tasks` is a list of paths to an Ansible tasks file to execute after the deploy. Optional.

The paths for `include_tasks` can be absolute, or relative to the playbook from which this role is executing.

## Dependencies

* ansible.posix
* The `terminus` command must be installed to use Post Deploy tasks.

## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
    - hosts: servers
      vars:
        platformsh_deploy:
          source:
            git_dir: 'path/to/source/git/dir'
          build:
            npm_dir: 'path/in/repo/to/package.json'
            npm_build_script_name: 'build'
          target:
            ssh_key_base64: 'abcdef1234567890'
            ssh_pub_base64: 'abcdef1234567890'
            repo_url: "ssh://codeserver.dev.abcd-ef12-3456-7890@codeserver.dev.abcd-ef12-3456-7890.drush.in:2222/~/repository.git"
            git_branch: 'master'
            git_commit_message: "Made with <3 by robots"
      roles:
         - { role: ten7.platformsh_deploy }
```

## License

GPL v3

## Author Information

This role was created by [TEN7](https://ten7.com/).
