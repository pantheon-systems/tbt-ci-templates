# Wordpress

This folder contains different files and configurations for Wordpress sites.

For any CI provider, you should copy the .ci folder to your project and then follow the instructions in the right section below.

## CircleCI

## Gitlab CI

## Github Actions

If you need to enable Github Actions for an existing project, you should copy github to .github and then add the following secrets to Github Actions configuration:

- ADMIN_EMAIL
- ADMIN_PASSWORD
- ADMIN_USERNAME
- TERMINUS_TOKEN
- TERMINUS_SITE
- SSH_PRIVATE_KEY
- GH_TOKEN
