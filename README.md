# A reviewer helper script

This script helps with reviewing, merging (cherry-picking) and restoring PRs on Foreman and related projects.

## Installation

The only dependency besides Ruby is git. The script runs `git` commands directly, no gem was able to do everything I needed in a good way.

Simply put the script into a directory which is in your path. I use `~/.bin`, so in my case I just `cp rpr ~/.bin`

The script changes its behavior based on the name with which it's being called. So you might want to create few more symlinks, for example above I recommend following

* `ln -s ~/.bin/rpr ~/.bin/crp`
* `ln -s ~/.bin/rpr ~/.bin/rrpr`
* `ln -s ~/.bin/rpr ~/.bin/usc`

### Authentication

You can create a file with github OAuth token so your Github queries are authenticated. This is useful if your IP reached the limit of 5000 calls per day. In order
to use it, create OAuth token go to [https://github.com/settings/tokens] and create a new token. Then store this to `.token` file in the same repository where you
installed reviewer, e.g. `~/.bin/.token`.

## Usage

### Review Pull Request

Navigate to any repository where you want to review the pull request. Then run

`rpr $id`

where $id is the number of the pull request you want to review. If the script finds config/database.yml file (rails application) it will try to do the backup of database in tmp/db_backups. If there's already a back (e.g. from previous review) it asks if it should override. Since you have a backup you can safely run db:migrate and seed.

After the backup is done (or skipped if this is not a rails app), it fetches information from github about the PR. It adds remote repository, the name is the github user who sent the PR. Then it checkouts new branch in form of `review/pr$id` and pull code from the PR to this new branch. Then you're ready for reviewing.

### Restore from Review Pull Request

Once you finished reviewing and you want to get back to normal operation, you can call `rrpr`. It will checkout the default branch (develop for foreman, master otherwise) and if it found a backup of database, it asks whether it should restore it. The review branch is deleted so you can rerevew the PR later.

### CheRry Pick

If you're happy with the PR you might want to cherry-pick it. I have always separate repository that has write access to main repositories so I'm running the command there. It expects that the branch that's currently active is the branch that I want the PR to be cherry-picked to. The cherry-pick is similar to `rpr` but you run `crp $id`, where $id is again the id of PR that should be cherry-picked. The script won't do `git push`, you should verify it did the right thing, you did the right thing and then push to remote repository.

### Update Source Code

In dev env, you often want to update source codes of all plugins/dependencies that you're using from git checkout. In order to do so, run the script with `usc` name. That will scan current bundler configuration, find all sources that lists path and then, one by one, it does `git pull upstream $branch` in those directories. The $branch is in fact currently checked branch in that git repo.
