---
author:
  name: Linode
  email: docs@linode.com
description: 'This guide shows how to use Hugo to make changes to the Linode theme, and goes over the directory structure for our development environment.'
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
modified: 2017-11-10
modified_by:
  name: Linode
published: 2017-11-09
title: "Linode Documentation Developer's Guide"
title_short: 'Development Guide'
---

# About Linode Documentation

Linode's documentation site is built using the [Hugo](https://gohugo.io) static website generator, and is composed of two Github repos:

- **linode/hercules**: This repo contains the source code and build tasks for the site. This repo is private, and is only accessed by the Linode docs team.
- **linode/docs**: This is where the markdown files for Linode's guides are maintained. It also contains a build of the Hugo theme for local editing. This repo is public, and freelance authors or members of the open-source community can clone the repo or submit pull requests.

This guide will focus on working with the documentation setup from the perspective of a member of the Linode docs team. For information about how to use linode/docs to set up a local development environment, see our [Contributing](/docs/CONTRIBUTING.md) guide.

# Install Dependencies


## OSX

1. Install git on your local machine:

		git

2. Install Homebrew:

		/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

3. Install Hugo and Node.js/NPM:

	 	brew install hugo
	 	brew install node

## Linux

1. Install git on your local machine:

	 	sudo apt install git

2.  Install Node.js and NPM:

		 cd ~
		 curl -sL https://deb.nodesource.com/setup_7.x -o nodesource_setup.sh
		 sudo bash nodesource_setup.sh
		 sudo apt install nodejs
		 sudo apt install build-essential

3.  Go to the [Hugo releases](https://github.com/gohugoio/hugo/releases) page and download the most up to date binary for your platform:

		 curl -OL https://github.com/gohugoio/hugo/releases/download/v0.30.2/hugo_0.30.2_Linux-64bit.tar.gz
		 tar -xvzf hugo_0.30.2_Linux-64bit.tar.gz
		 sudo mv hugo /usr/local/bin


# Set Up the Environment

1.  Clone both of the Linode repos into the same root directory.

		mkdir ~/linode && cd ~/linode
		git clone https://github.com/linode/hercules
		git clone https://github.com/linode/docs

2.  Navigate to the `hercules` root directory and install the dependencies:

		cd hercules
		npm install


# Configure SSH

Your Linode SSH key will be credsynced to work with the docs1 and docs2 boxes. You can configure `~/.ssh/config` to use the correct hostnames and user.

1.  Edit '~/.ssh/config`:

        Host docs1.linode.com
            Hostname docs1.linode.com
            User docs
            IdentityFile ~/.ssh/id_rsa

        Host docs2.linode.com
            Hostname docs2.linode.com
            User docs
            IdentityFile ~/.ssh/id_rsa


2.  Use `ssh-keyscan` to add docs1 and docs2 to your known hosts file:

		ssh-keyscan docs1.linode.com >> ~/.ssh/known_hosts
		ssh-keyscan docs2.linode.com >> ~/.ssh/known_hosts


  If after completing this step, the `gulp publish` task described below still throws an error, manually use SSH to connect to each of the boxes and type 'yes' when asked if you are sure you wish to continue connecting.



# Basic Gulp Tasks

When working with the Hugo repo, most of the workflow will use Gulp. This section will review the available Gulp tasks.

1.  Build the Hugo theme:

		gulp build

	This task uses any changes in the local linode/hercules repo and uses them to build a new version of the theme. These changes are also automatically added to the built theme within the linode/docs repo.

2.  Start the development server:

		gulp dev

	This task will build the theme and start a development server with hot-reloading. The default address is localhost:1313/docs. While this server is running, any style or layout changes will be automatically updated in the browser. Keep in mind that`gulp dev` runs the `build` task as part of its pipeline. As a result, starting the development server will also change the linode/docs repo as a side effect.

3.  Publish the hercules repo:

		gulp publish --target=test --version=v1.0.0

	This will publish the changes to a target server, generally either `test` or `production`. Choose the server to deploy to using the `--target` option. You **must** specify a version to publish or the deployment will fail. Use `git tag` to list the available tags.

For more information about these tasks, see the [README](https://github.com/linode/hercules/blob/master/README.md) file for the linode/hercules repository.

# Hugo structure

This section will take a closer look at the Hugo directory structure. For more information, see the [Hugo documentation](http://http://gohugo.io/documentation/).

Running `ls` from the hercules root directory reveals the following structure:

		hercules/
			archetypes/
			assets/
				hbs
				images
				js
				stylesheets
				vendors
			config.toml
			dist/
			gulpfile.js
			layouts/
				. . .
				partials/
				shortcodes/
			scripts/
			static/
			tasks/
			themes/

* `archetypes`: These are template files for documents. Any archetypes for internal use can be stored in this folder; templates for generating new guides should live in `linode/docs/archetypes`.
* `assets`: This is where javascript files, stylesheets, and external vendors (including Lunr.js) are located. Any CSS changes should be done within this directory.
* `config.toml`: This is the standard Hugo config file. It is unlikely that you will need to make changes to this file; if necessary, see the Hugo documentation for more information.
* `dist`: This directory is the build target. It includes minified HTML files for the built docs. The contents of this directory should not be directly edited.
* `gulpfile.js`: The Gulpfile contains the basic build tasks discussed above.
* `layouts`: The custom partials for the site's layouts are stored here. The shortcodes used in the Markdown files for guides are located here as well.
* `scripts`: The original migration scripts, used to convert the older Middleman/ERB syntax to the Hugo format, are kept here.
* `static`: Compiled/minified static assets, including stylesheets and javascript files, are stored here.
* `tasks`: This directory contains more detailed configuration for the Gulp build and deploy tasks. Information about the test and production servers is also stored here in `config.json` (discussed below).
* `themes`: This directory should not be edited directly.

# Development Workflow

1.  From the root Hercules directory, pull the latest version of the site and create a branch for your edit.

		cd ~/linode/hercules
		git pull
		git checkout -b edit-layout

2.  Start the development server.

		gulp dev

3.  Make any changes necessary to the layout. Save and commit the changes.

		git add --all
		git commit

4.  Build the changes to update the theme.

		gulp build

5.  Navigate to `linode/docs` and check to see that the theme changes have been added locally.

		cd ~/linode/docs
		git status

6.  Commit and push the changes to the docs repo. Make sure to tag the commit and specify the tag name when pushing:

		git add --all
		git commit
		git tag 2017-01-01
		git push origin 2017-01-01

8.  Return to linode/hercules and deploy the site to production (or test).

		gulp publish --target='production'  --version=2017-01-01

	When deploying to production you will be prompted to confirm the action.

  The `--version` tag must be present, and the tag specified must be locally available. If you omit a version target or refer to a remote tag that has not been pulled, the publish task will fail.


# Configuring Servers

The gulp publish task can deploy to any server listed in `linode/hercules/tasks/config.json`. To change the server configuration, edit `hercules/tasks/config.json`:

    {
        "servers": {
            "test": {
                "hostname": "192.0.2.0",
                "username": "docs",
                "destination": "/home/username/www",
                "baseURL": "https://docstest.linode.com/docs/"
            },
            "production": {
                "hostname": "198.51.100.0",
                "username": "docs",
                "hugoEnv": "prod",
                "destination": "/home/username/www"
            }
        }
    }