# pantheon-support-tools

Various scripts for supporting Pantheon at the University of Michigan.

## Setup

Make sure you have `terminus` installed and working.  You will need to configure both a machine token as well as a Pantheon SSH key (preferably encrypted with a passphrase and loaded into `ssh-agent`) configured and working.

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

