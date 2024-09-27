# GEN and IPM Wordpress Websites

by Brian Aderer  
#### for Mary Ann Liebert Publishing  
https://liebertpub.com  
https://brianaderer.com

updated 27 SEP 2024

## High Level and History

GEN and IPM are Wordpress sites, migrated from a legacy CMS around 2018. They are built around a website builder plugin called TagDiv, which pivots off the NewsPaper theme. Both GEN and IPM are child themes of Newspaper.

## Construction

GEN and IPM are both composed of no less than 13 repo's each, and growing. This allows code sharing and abstraction, and creates single points of maintenance environment wide. The structure is a 'theme' repo and a 'plugins' repo at the top level, each with sub-repo's. The TagDiv files are all managed outside of version control due to the TagDiv update process, and therefore must be included separately.

## Hosting

Sites are currently hosted on a managed WordPress Engine account. Local by Flywheel is used for Local development. We are actively looking to replace the development environment, looking at MAMP as a primary candidate.

## Code Build and Deploy

We use a GitHub actions based CI/CD process where code is built to the server on code push or pull, depending on branch rules. This process is defined in `.yml` files on a per branch bases in `./github/workflows` for any given repo which requires a code build. In general, pushing to the staging branch will trigger a build, but prod requires a staging -> main PR. More on Github actions here:  
https://docs.github.com/en/actions

## Recovery  

Recovery can be accomplished via the dev@liebertpub.com address which is the owner seat for the Mary Ann Liebert GitHub Account here:  

https://github.com/Mary-Ann-Liebert-Dev

This contains all 31 repo's which currently constitute our codebase, about 2/3 of which are in active use.

## Performance

Problems with the initial migration of the site, overreliance on Third party plugins, and TagDiv itself have over the years created issues in the data structure, both with size and integrity. Our Post Meta table alone on GEN is over 500MB.  This creates server performance issues as well as difficulties migrating the site due to the sheer size of the data pool. Cleaning up the data structure and reducing the plugin count by in-housing custom features are both ongoing primary initiatives for site maintenance and improvement.

## Structure

Currently the hierarchy is laid out as follows. Each subrepo has its own readme (where applicable). More info on each subrepo can be found in the theme .gitmodule file.

Child theme: genengnews(GEN) / clinicalomics(IPM):
|+ GetAuthors: retrieves and outputs author information  
|+ HubspotForms: renders hubspot forms  
|+ Image_Backup: prevents rendering broken images by creating the requested image size from a source file if present, and using a fallback if not  
|+ MalEnv: Sets up constants from the .env file. This allows for default in constant values, and failing gracefully if the .env is incomplete  
|+ MalHooks: Hooks into various post save actions to send API calls to external services maintaining data sync's. Currently interfaces with Hubspot and Portable Profile servers
|+ MalPostman: Abstracted CURL handler classes to interface with external API endpoints  
|+ PPWidget: Renders the in-page portable profile widget and scripts  
|+ Straive: Retrieves and renders the straive data into the page. Includes a custom DB handler which is extremely important for performance optimization  
|+ StraiveForm: Renders and controls Straive's tagging interface

Plugins: All plugins for the theme with the exception of TagDiv:  
|+ mal-data-migrator: Utility classes for handling migrations and data cleanses. Exposes endpoints via WP REST API, and also has an admin interface which is deprecated due to WPE issues. Currently its run via a desktop handler, which I wrote in python. Repo for that is here: git@github.com:Mary-Ann-Liebert-Dev/PythonDataConsole.git

## Installation

I will cover here just the specifics of this particular install, assuming that you already know how to install wordpress in your target environment, and migrate the database. This process could be covered in another document.  

- First, install TagDiv. The easiest way of doing this is to go into the server, copy and paste the Newspaper theme from the ./themes folder into your ./themes folder. Do the same for all plugins prefaced by `td` from the ./plugins folder. Activate all the td plugins.

### - Set up the Repo and pull the files

Create the appropriate directory in your `./themes/` folder and from your Git Bash terminal run  
`git init`  
`git remote add origin [repo address: ssh notation]`  
`git fetch --all`  
`git checkout staging`  
`git pull origin staging`  

This will checkout the staging branch and pull all the theme files from the repo. You can now activate your respective Newspaper child theme that you just pulled. It will give you a critical error at this point. Now do the same to `./plugins`. From your admin console, compare the plugins activations with the live site and make sure they are identical.

### - Install SubRepo's  

In a git bash enabled terminal, navigate to the theme root location and run

`git submodule update --init --recursive --remote`

To get all the most recent included submodule packages. Then do the same to `./plugins`

### - Install Composer

Composer is used extensively in this theme. You must run the install process in all relevant locations or the theme will not work. From the root `./wp-content` directory these locations are: 
 - `/themes/[theme_name]` (your active theme directory, either genengnews or clinicalomics)  
 - `/themes/[theme_name]/MalAdsManager/googleads-php-lib`  
 - `/plugins/mal-data-migrator`  

In a ***POWERSHELL*** (WIN) terminal navigate to each of the above directories and run  
 `composer install`
to install the relevant composer packages and create the necessary `/vendor` directory and `autoload.php`.  
**NOTE** `/vendor` and `autoload` files are not included in the repo, but are required by the theme. You will get critical errors if this step is skipped or not carried out properly.

### - Install NPM

From the main theme directory run 

`npm install`
`npm run build:css`

This installs all the relevant node modules, and treeshakes the Tailwind package off the current theme files. If you are not working with Tailwind, this step will suffice.  

**NOTE** I recommend all further CSS be written in Tailwind for performance and reliability. Tailwind is already built into the theme. If you are working with theme CSS, run  

`npm run watch:css`  

To watch your theme files and automatically regenerate the build CSS on changes. More on tailwind here:  

https://tailwindcss.com/

### IF All went well, you should now have a working MAL Wordpress site!!!

### Other services

There is also a Python based Reporter package that works in concert with the Mal Data Migrator plugin. It runs on an AWS server and is triggered by a cron job. See Ed Gonzalez for server access. It also serves as a desktop handler for migrations to handle post iteration and data aggregation during bulk data operations. Its Repo is here:  
https://github.com/Mary-Ann-Liebert-Dev/PythonDataConsole

