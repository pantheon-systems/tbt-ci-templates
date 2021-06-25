# Drupal 7

This folder contains different files and configurations for Drupal 7 sites.

For any CI provider, you should copy the .ci folder to your project and then follow the instructions in the right section below.

## CircleCI

## Gitlab CI

## Github Actions

If you need to enable Github Actions for an existing project, you should copy github to .github and then add the following secrets to Github Actions configuration:

- TERMINUS_TOKEN
- TERMINUS_SITE
- SSH_PRIVATE_KEY
- GIT_EMAIL

