# pantheon-support-tools

Various scripts for supporting Pantheon at the University of Michigan.

## Setup

Make sure you have `terminus` installed and working.  You will need to configure both a machine token as well as a Pantheon SSH key (preferably encrypted with a passphrase and loaded into `ssh-agent`) configured and working.

Make sure `~/.terminus/config` exists and has the following contents:

```yaml
http_max_retries: 10
http_retry_backoff: 30
options:
  interactive: false
  verbose: true
  no-interaction: true
  'yes': true
```

Install:

```bash
git clone git@github.com:its-pantheon/pantheon-support-tools.git
cd pantheon-support-tools

brew install rust-parallel
```

Make sure the following command runs successfully and returns a WordPress version number:

```bash
terminus wp its-wws-test1.live -- core version
```

## get_pantheon_domains

Creates a text file listing all the custom domains attached to the Live environment of Pantheon sites on paid plans in our Pantheon organization. It skips domains connected to other environments, and skips sites that have Live environments but are on the Sandbox plan.

The script will create a scratch (working) directory each time it is run.  The directory will be under `./tmp`, if it exists, otherwise it will be under `/tmp`.  The scratch directory is deleted if the script finishes successfully, but is kept for debugging if the script exits with an error.

Example:

```bash
time ./get_all_pantheon_domains
Getting site list...
Getting domains for each Live site...
  [00:00:26] Commands Done/Total: 193/193 [##########################################################################################################] ETA 00:00:002025-03-05T13:15:52.949923Z ERROR rust_parallel::output::task: command failed: cmd="/opt/homebrew/bin/bash",args=["-c", "get_domains engin-ioe-2-webapps"],line=stdin:5 exit_status=1
2025-03-05T13:16:02.955841Z ERROR rust_parallel::output::task: command failed: cmd="/opt/homebrew/bin/bash",args=["-c", "get_domains umma-inside-website"],line=stdin:82 exit_status=1
2025-03-05T13:16:16.361475Z ERROR rust_parallel::output::task: command failed: cmd="/opt/homebrew/bin/bash",args=["-c", "get_domains phar-copweb"],line=stdin:187 exit_status=1
2025-03-05T13:16:17.956581Z ERROR rust_parallel: fatal error in main: command failures: commands_run=193 total_failures=3 spawn_errors=0 timeouts=0 io_errors=0 exit_status_errors=3
Done. List of domains is in:
pantheon-domains-20250305.txt

real	0m32.015s
user	0m41.465s
sys	0m15.768s
```

The sites with errors above are ones that are on paid plans but don't have any domains connected.


## generate_wp_plugins_list

Creates a CSV file containing information about all plugins and themes installed for WordPress websites' Live and Dev environments in a pantheon orginization (workspace).  This is useful for finding which sites are running versions of a plugin/theme that have a particular security vulnerability.

The script will create a scratch (working) directory each time it is run.  The directory will be under `./tmp`, if it exists, otherwise it will be under `/tmp`.  The scratch directory is deleted if the script finishes successfully, but is kept for debugging if the script exits with an error.


Example:

```bash
$ time ./generate_wp_plugins_list
===== Getting site IDs
===== Waking dev sites
  [00:00:47] Commands Done/Total: 169/169 [##########################################################################################################] ETA 00:00:00
 [notice] OK >> dev-ath-macc.pantheonsite.io responded
 [notice] OK >> dev-ath-test-pantheon.pantheonsite.io responded
[... lots more notices omitted ...]
===== Getting plugins/themes
⠂ [00:00:03] Commands Done/Total:  0/38 [------------------------------------------------------------------------------------------------------------] ETA 00:00:00Warning: Permanently added '[appserver.live.3451851f-50e1-40bf-91e7-dc3df0e71bc5.drush.in]:2222' (RSA) to the list of known hosts.
 [notice] Command: ai-onlineteaching.live -- wp plugin list [Exit: 0] (Attempt 1/1)
⠂ [00:00:03] Commands Done/Total:  1/39 [##>---------------------------------------------------------------------------------------------------------] ETA 00:02:20Warning: Permanently added '[appserver.live.cc970b37-a1b3-444c-a933-7e4c6b397925.drush.in]:2222' (RSA) to the list of known hosts.
 [notice] Command: aaum-alumni-association.live -- wp plugin list [Exit: 0] (Attempt 1/1)
[... lots more warniing messages omitted ...]
===== Getting plugins/themes


===== Done, plugin list is in:
pantheon-wordpress-plugins-20250305.csv

real	5m27.047s
user	5m0.992s
sys	1m7.645s
$
```

## generate_drupal_modules_list

Creates a CSV file containing information about all modules and themes installed for Drupal websites' Live and Dev environments in a pantheon orginization (workspace).  This is useful for finding which sites are running versions of a moule/theme that have a particular security vulnerability.

The script will create a scratch (working) directory each time it is run.  The directory will be under `./tmp`, if it exists, otherwise it will be under `/tmp`.  The scratch directory is deleted if the script finishes successfully, but is kept for debugging if the script exits with an error.


## generate_site_git_filelists 

Creates lists of all files under /code (that is, all files in Git) in the Dev and Live environments of all websites in the organization.  These lists can then be searched to find sites that have a particuar file as a part of their codebase -- for example, to find sites using unknown plugins that use a vulnerable version of a third-party dependency.

Usage:
```bash
./generate_site_git_filelists
```

Expect the script to take ~10 - 15 minutes per 100 sites in the organization.  The results will be in a directory named `site-git-filelists-YYYYMMDD-HHMM`.  This directory will contain a subdirectory for each site, which in turn will contain the files `files-dev` and (if the Live environment has been initialized for the site) `files-live`.  The file `log.txt` will show any errors encountered while checking the sites' git repositories.

Example: Find all site that have SimpleSAMLphp installed, regardless of whether it is installed as a part of a plugin or if it was manually installed.  The file we're searching for, `SimpleSAML/SessionHandlerPHP.php`, should be something that is always installed if the library is installed, but unlikely to exist in any other code.

```bash
$ grep SimpleSAML/SessionHandlerPHP.php site-git-filelists-20250314-1636/*/files-*
site-git-filelists-20250314-1636/graham-graham-website/files-dev:100644 blob 99931bd001e12c06ef3593d94a7540aa4b1b76de   12143	vendor/simplesamlphp/simplesamlphp/src/SimpleSAML/SessionHandlerPHP.php
site-git-filelists-20250314-1636/graham-graham-website/files-live:100644 blob 99931bd001e12c06ef3593d94a7540aa4b1b76de   12143	vendor/simplesamlphp/simplesamlphp/src/SimpleSAML/SessionHandlerPHP.php
site-git-filelists-20250314-1636/its-cjatest/files-dev:100644 blob f1d70e31f41e501cfbee8048dfe35e3ae4abcce3   11717	private/simplesamlphp-2.2.2/src/SimpleSAML/SessionHandlerPHP.php
site-git-filelists-20250314-1636/its-cjatest/files-dev:100644 blob f1d70e31f41e501cfbee8048dfe35e3ae4abcce3   11717	private/simplesamlphp/src/SimpleSAML/SessionHandlerPHP.php
site-git-filelists-20250314-1636/its-csg-stonesoup/files-dev:100644 blob f1d70e31f41e501cfbee8048dfe35e3ae4abcce3   11717	private/simplesamlphp/src/SimpleSAML/SessionHandlerPHP.php
site-git-filelists-20250314-1636/panda1/files-dev:100644 blob 6045c59aac2b96c333d45109a69d993ab4ae0d4f   13089	vendor/simplesamlphp/simplesamlphp/lib/SimpleSAML/SessionHandlerPHP.php
site-git-filelists-20250314-1636/panda1/files-live:100644 blob 6045c59aac2b96c333d45109a69d993ab4ae0d4f   13089	vendor/simplesamlphp/simplesamlphp/lib/SimpleSAML/SessionHandlerPHP.php
site-git-filelists-20250314-1636/umdd/files-dev:100644 blob f1d70e31f41e501cfbee8048dfe35e3ae4abcce3   11717	vendor/simplesamlphp/simplesamlphp/src/SimpleSAML/SessionHandlerPHP.php
site-git-filelists-20250314-1636/umdd/files-live:100644 blob f1d70e31f41e501cfbee8048dfe35e3ae4abcce3   11717	vendor/simplesamlphp/simplesamlphp/src/SimpleSAML/SessionHandlerPHP.php
$ 
```

