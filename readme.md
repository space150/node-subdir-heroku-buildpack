# space150 node-subdir Heroku Buildpack
Inspired by the awesome [timanovsky/subdir-heroku-buildpack](https://github.com/timanovsky/subdir-heroku-buildpack) project. We created this buildpack because we often have projects that require *multiple* directories, and Tim's project only supported keeping a single directory. We also need to handle some additional `package.json` tweaking that is specific to Node.js & Heroku.

As an example, consider the following repository structure:
- `/`
  - `core/` Contains a shared codebase that is used by both the `web` and `admin` projects.
  - `web/` Contains the project for serving a headless public website. Dependent on the `core` project.
  - `admin/` Contains the project for serving a private administration site. Also dependent on the `core` project. The public website must not have any knowledge or reference to this `admin` site's existence.
  
In this case, we need that `core` library to survive when deploying the `web` and `admin` sites to Heroku, since the `npm run build` command for both `admin` and `web` relies on that `core` library existing at build-time. Additionally, we need the ability to instruct Heroku which project to launch via a Config variable.

## Heroku Setup
1. Have your repository mirror the directory structure referenced above. The exact folder names are not important; you can name them however you like.
1. Log into Heroku & create two Heroku Config Vars in your project:
  1. `BUILDPACK_START` points to the directory in your repository that contains the `package.json` to run. This config allows Heroku to be configured to launch one of many projects in the repository. In our example above, this would get set to `admin` for the admin project, and `web` for the public project.
  1. `BUILDPACK_KEEP` is a `;`-delimited list that includes all of the directories to retain. In our example case above, to configure the `public` site this would get set to `web;core`. On the `admin` site, this would get set to `admin;core`.
1. Set this buildpack as the first item in the buildpack chain. Paste in the Git URL `https://github.com/space150/node-subdir-heroku-buildpack.git`. We recommend appending a specific commit hash to this Git URL so that any future changes we make to the `main` branch do not impact your project. We fully reserve the right to make breaking changes at any time without any advance warning or notification.

## How it works
1. Creates a `package.json` file on the root of the build artifact that points the `build` and `start` commands to the configured `BUILDPACK_START` directory. The buildpack also attempts to set the `{ engines: { node }}` value in this new root `package.json` file to match whatever is entered in your `BUILDPACK_START/package.json` file.
1. Takes the configured `BUILDPACK_KEEP` directories and copies them to the root of the build artifact.
1. Deletes everything in the Heroku build directory.
1. Copies the build artifact into the Heroku build directory.
1. Deletes the build artifact, leaving the final files in the Heroku build directory.

## Development Setup & Testing
1. Create some local directories to mirror the Heroku `build`, `cache`, and `env` buildpack directory. On my machine, it looks like:
  1. Build: `/Users/developer/dev/node-subdir-heroku-buildpack/.test/build`
  1. Cache: `/Users/developer/dev/node-subdir-heroku-buildpack/.test/cache`
  1. Env: `/Users/developer/dev/node-subdir-heroku-buildpack/.test/env`
1. Copy your repository into your configured `build` directory.
1. Create `BUILDPACK_START` and `BUILDPACK_KEEP` files in your `env` buildpack directory. The contents of each file should be the values you want the buildpack to use.
1. While testing, it is recommended to create a `BUILDPACK_DEV` file in the `env` buildpack directory as well to disable the normal cleaning process. If you don't do this, you will have to clear out & re-copy your repository files in the `test/build` directory after each test.
1. Run the `./bin/compile` script passing in the paths to each of your buildpack directories. Here's how it looks on my machine: `sh ./bin/compile /Users/developer/dev/node-subdir-heroku-buildpack/.test/build /Users/developer/dev/node-subdir-heroku-buildpack/.test/cache /Users/developer/dev/node-subdir-heroku-buildpack/.test/env`
1. Check the `build` or `cache` directory to verify that your edits worked.