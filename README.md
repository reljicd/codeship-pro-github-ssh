# [Setting GitHub SSH keys in Codeship Pro](https://github.com/codeship-library/setting-ssh-private-key-in-pro)

## Initialize Project

- Clone this repo
- Initialize as a new git repo -- `rm -rf .git && git init && git add . && git commit -m 'first commit'`

## Selecting a Private Key

### Option A -- Generate a public and private ssh key

- To generate a `codeship_deploy_key` and `codeship_deploy_key.pub` file, modify the following command with your own email address and run our [ssh-helper tool](https://github.com/codeship-library/docker-utilities/tree/master/ssh-helper) in your project directory:

```
docker run -it --rm -v $(pwd):/keys/ codeship/ssh-helper generate "<YOUR_EMAIL>"
```

### Option B -- Use your own pre-existing private ssh key

- Copy file to project directory and rename to `codeship_deploy_key` (must be a key that does not require a passphrase)

## Prepare the Environment Variables file

- Run the following command from the project directory:

```
docker run -it --rm -v $(pwd):/keys/ codeship/ssh-helper prepare
```

- Process will store `PRIVATE_SSH_KEY` value as a one liner entry into the `codeship.env` file. If `codeship.env` already exists, the `PRIVATE_SSH_KEY` entry will be appended to it.
- Remove the `codeship_deploy_key` (!)
- Add `codeship.env` to your `.gitignore` file (!)

## Encrypt the Environment Variables file

- Install our [jet cli tool](https://documentation.codeship.com/pro/jet-cli/installation/) on your local machine
- Setup your repository on your SCM of choice
- Grab the git url of the repository and create a Codeship Pro project
- From your Codeship 'Project Settings' > 'General' page, scroll down to AES key section and click 'Download Key'
- Move downloaded key to your project directory and rename to `codeship.aes`
- Add `codeship.aes` to your `.gitignore` file (!)
- Add [any additional environment variables](https://documentation.codeship.com/pro/builds-and-configuration/environment-variables/#encrypting-your-environment-variables) to the `codeship.env` file
- Run `jet encrypt codeship.env codeship.env.encrypted`
- The `codeship.env.encrypted` will be safe to check into your git repository

## Add public key to your SCM of choice

- [Add the codeship_deploy_key.pub to your Github account](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)
- Demo is set to attempt a connection with Github. You may use Bitbucket or Gitlab as well, but will require tweaking test steps in `codeship-steps.yml`

## Run `jet steps`

- Run `jet steps`
- Steps should pass, demonstrating that `/root/.ssh/id_rsa` is now accessible to the main app via volumes
- Be sure to modify the volume pathing to `.ssh` in the `codeship-services.yml` if the container user is not `root`
- Add `.ssh` directory to your `.gitignore` file (!)