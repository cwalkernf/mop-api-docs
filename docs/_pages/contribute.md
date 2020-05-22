---
title: Improve this Site
permalink: /contribute/
toc: true
sidebar:
    nav: "v1guides"

---

Thank you for improving this site! This site is open source and the API maintainers welcome your contributions. [Please reach out](/help/) if you have any questions.

## Content
The content of this site lives in the top-level directory `/docs` in the GitHub repo [netfoundry/mop-api-docs](https://github.com/netfoundry/mop-api-docs/tree/master/docs). Minor changes can be made directly in GitHub's editor. Changes are published automatically by GitHub pages when merged to master.

## Theme
The theme lives in the top-level `/` in the same GitHub repo [netfoundry/mop-api-docs](https://github.com/netfoundry/mop-api-docs) as the content. The theme is forked from Minimal Mistakes which publishes an excellent [quick-start guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/). The idea is to minimize changes to the parent theme so that we can later merge in improvements from upstream.

## Preview
These steps provide a local preview server at **[http://localhost:4000/](http://localhost:4000/)**

1. Install
    1. [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git), 
    2. [Docker Engine](https://docs.docker.com/engine/install/), and 
    3. [Docker Compose](https://docs.docker.com/compose/install/)
2. clone the repo 

    ```bash
    git clone git@github.com:netfoundry/mop-api-docs.git
    ```

3. switch to the content area `/docs` in the cloned working copy

    ```bash
    cd ./mop-api-docs/docs
    ```

4. execute

    ```bash
    docker-compose up --build preview
    ```

5. **browse to [http://localhost:4000/](http://localhost:4000/)**

6. Be aware that *search is not available* in the local preview.
7. Stop the preview container
    1. Focus the terminal where the container is running
    2. Send a `SIGTERM`, typically `Ctrl-c`
    
8. Alternatively, you could 
    1. run the preview container in the background
        
        ```bash
        docker-compose up --build --detach preview
        ``` 

    2. and later stop the preview container

        ```bash
        docker-compose down --remove-orphans
        ```

## Things to Know

### Workflow

1. fork the repo, unless you have access to main repo
2. clone the repo
3. make a change
4. create a local branch
5. commit your change
6. push your commit to the remote
7. create a pull request for your branch
8. change the base repo to "netfoundry/mop-api-docs" 
9. submit your pull request
10. monitor for build failures

We'll be able to simplify this somewhat by removing the need to change the base repo when the fork link is broken. For now, it seems there's no way to configure the default base repo.


### Content &amp; Theme
* Local changes to files in `/docs` will be picked up immediately by Jekyll, except `/docs/_config.yml` which requires restarting the preview container.
* Optionally, before running the container, export your [GitHub API token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) as `JEKYLL_GITHUB_TOKEN` and it will be made available to Jekyll for querying metadata from the GitHub API.
* Most content is in `/docs/_pages/` with meaningful names. You can add or edit Kramdown (GitHub-flavored Markdown) `.md` or Liquid template `.html` files.
* Theme changes could go in the top-level theme or become overrides in corresponding child directories under `/docs`. For example, `/docs/_layouts/default.html` overrides `/_layouts/default.html` and is available immediately in the local preview. Changes to the top-level theme don't become visible in the local preview until they're merged to the master branch in the Git remote.

### Build &amp; Publish
* All merges to the master branch are automatically published by GitHub pages.
* Pushes to any branch trigger [a Travis build](https://travis-ci.org/github/netfoundry/mop-api-docs). The Travis build
    * validates the changes with Jekyll, and
    * updates the Algolia search index. The Travis env var `ALGOLIA_API_KEY` is the secret key for updating the Algolia index that was configured in `/docs/_config.yml`.
* The Travis job reports are sent to Slack in *netfoundry.slack.com#dev-notifications*
    * The Slack token is encrypted with `travis` CLI in `/.travis.yml` by following steps prescribed in [the Travis App for Slack](https://netfoundry.slack.com/apps/A0F81FP4N-travis-ci?next_id=0)
    ```bash
    # in /docs
    docker-compose run --rm build
    # iteractive in running container
    gem install travis --no-document
    travis encrypt "netfoundry:{redacted}" --add notifications.slack -r netfoundry/mop-api-docs
    ```
    The result of this is a new encrypted Slack token in the existing Travis config file `/.travis.yml` which you'll need to commit and push to the Git remote to become part of your pull request.
* The GitHub repo has a branch protection for master that requires a successful Travis build.
* The domain name developer.netfoundry.io is a `CNAME` resource record in the netfoundry.io hosted zone in Route53. The `RDATA` of the record is the GitHub Pages sub-domain.
    ```bash
    ❯ dig +short -tCNAME developer.netfoundry.io.
    netfoundry.github.io.
    ```