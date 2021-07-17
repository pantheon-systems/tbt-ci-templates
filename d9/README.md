# Drupal 9

This folder contains different files and configurations for Drupal 9 sites.

For any CI provider, you should copy the .ci folder to your project root and the corresponding folder under the providers folder also to your project root. Then, follow the instructions in the right section below.

## CircleCI

TBA

## Gitlab CI

TBA

## Github Actions

If you need to enable Github Actions for an existing project, after copying files you should add the following secrets to Github Actions configuration:

- ADMIN_EMAIL
- ADMIN_PASSWORD
- ADMIN_USERNAME
- TERMINUS_TOKEN
- TERMINUS_SITE
- SSH_PRIVATE_KEY
- GH_TOKEN
