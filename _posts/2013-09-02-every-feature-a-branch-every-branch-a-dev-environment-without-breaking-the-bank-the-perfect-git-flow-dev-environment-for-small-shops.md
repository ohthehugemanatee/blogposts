---
layout: post
comments: true
title: Every feature a branch, every branch a dev environment... without breaking
  the bank! The perfect git-flow dev environment for small shops
created: 1378148748
categories:
 - drupal
 - git flow
 - workflow
---
<h2><strong>What and Why?</strong></h2><p>Everyone loves the <strong>git flow</strong> method for managing branches in development projects. If you've been living under a rock and don't know what git flow is, here's <a href="http://nvie.com/posts/a-successful-git-branching-model/">the original blog post that brought order to branching chaos</a>. Honestly you can get a really good idea just by looking at the diagram:</p><p><br><img alt="" src="/downloads/drupal/git-flow.png" style="border-width: 0px; border-style: solid; margin: 10px; float: left;" height="484" width="363"></p><p>Git-flow is a great branching methodology. It leverages git's merge strength to the max, and lets you focus on the important things. The only problem is: it requires a lot of development environments.</p><p>This isn't actually a problem for many projects. Most coding projects keep their important work - you guessed it - in code, and a new development environment is just a "git checkout -b branchname" away. But for Drupal projects it can be a real pain. Setting up a new environment means copying codebase, files, and db... not particularly hard to do, but enough effort that most people take short cuts. For this reason, most firms don't really implement git flow in a complete way. I know of a few shops who, feeling guilty about not implementing git flow in their shared resources, tell their developers to do it on their localhosts. I don't know any developers who listen, though. It always seems easier to just shortcut.</p><p>In the last few months, we've seen the release of a couple of Drupal-oriented hosting offerings specifically tailored to fill this gap. Commerce Guys launched the <a href="http://commerceguys.com/product/commerce-platform">Commerce Platform</a> and Pantheon launched <a href="https://www.getpantheon.com/multidev">Multidev</a> within a few weeks of each other. If you haven't heard of these products, go check them out now. They both offer development environments that clearly have git-flow in mind, where you get a perfect production-copy AWS instance for each new branch you create. This is really cool stuff: each branch gets its own virtual server, in seconds! An environment totally identical to live!</p><p>These are great products (as far as I've heard), but they're not tailored to the needs of a Drupal shop. They only support single repositories, and just a handful of branches. The typical small or medium sized Drupal shop will have multiple features on the go for several active clients at once. We're not talking about managing 4 or 5 branches here, like Multidev supports. Go check your ticketing system now. Across all your projects, you're probably managing at least 50 features or bugfixes. In real git-flow, each of those is supposed to get a unique branch and development environment. I have no idea what kind of a deal Pantheon would cut me for enough multidev environments to manage 15 repos and 200 branches at a time, but I don't think that's going to be a standard package anytime soon.</p><p>So what's the small or medium sized Drupal shop to do? We want a way to realize the benefits of git flow, without breaking the bank.</p><p>The truth is that you don't really NEED a separate copy of the live server for each branch. You need a separate environment, but git flow accounts for a staging step in golive. There's nothing wrong with building all your development branches on one shared environment, and maintaining a separate staging environment per client. That's actually the Normal Way To Do It.&nbsp;And if all our dev environments are on the same server, suddenly spinning up a new environment doesn't look so complicated after all.<em> </em></p><h2>How</h2><p><em>We're going to use git hooks to automatically provision a new Drupal environment every time a new branch is pushed</em>. This means keeping current checkouts of each branch in a logical structure, while coordinating a database and a files directory for each. Of course you have to clean up when a branch is destroyed, too.</p><p>These scripts are based on my environment. If you're a subdomain person, or if your environment is otherwise different from mine, these scripts shouldn't be hard to manipulate for your needs... it's just bash, after all. In this structure, each repo gets a subdirectory, and each branch gets a subdirectory underneath. We'll use the Master branch as the main development branch, since that's where developers are most likely to make accidental pushes, and I'd rather not have those get to live! This means your main development branch will be at http://development.example.com/reponame/master . Your dev copy of the live site will be at http://development.example.com/reponame/live , and so on.</p><p>While you're testing these scripts, set up a special repo just for the testing. You can put the hook in that repo's .git/hooks/ directory. Later on you can install it for all your repos, however your git host handles that.</p><p>There's one caveat on this system: <strong>never let anyone develop directly in any of these directories</strong>. Any changes anyone needs to make to a site's codebase should be made through the repository anyway, but this system actually relies on that rule. So make sure your devs don't have access to the dev server's live checkouts, give them stern talkings-to, whatever you have to do. Save yourself the trouble of tracking down the weird errors and loss of work that can result, and force people to use the repo properly.</p><p>To allow a working tree for your server's git repo, you need to set the config for the repos this will apply to. In most git hosting environments you can add these configuration variables to your git user's ~/.gitconfig file. While you're testing, you can add them to .git/config in your testing repo.</p>
<pre class="brush: bash; auto-links: true; collapse: false; first-line: 1; html-script: false; smart-tabs: true; tab-size: 4; toolbar: true; codetag" title=".gitconfig">[core]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bare = false
[receive]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; denycurrentbranch = ignore

</pre>
<p><strong>bare = false</strong>&nbsp; This tells git that this repo DOES have a working tree associated with it.</p><p><strong>denycurrentbranch = ignore</strong> This option is what normally protects a repository with a working tree from receiving pushes. Normally if your non-bare repository receives a push, your repo's HEAD will end up out of sync with that working tree. That's a problem waiting to happen. We set it to "ignore," because we know that each push is going to be immediately followed by a checkout to bring that working tree back into sync. We also know that the only person touching these files is going to be the git script, right?</p><p>Now we add the actual git post-receive hook. This is where the magic happens. Note that I rely on the gitolite variable $GL_REPO here. If you don't use gitolite, you'll have to find another way to get the repository name. You'll have to set a few variables at the top of the file before this is useful:</p><p><strong>mysql_user / mysql_pw</strong>: The mysql user doesn't have to be root, it can be anyone with access to create and drop databases.</p><p><strong>worktree_root</strong>: The root directory where all your repo checkouts should go. On my server, this is the webroot.</p><p><strong>http_root</strong>: The root URL for your development environment. Your repo and branch names will be appended to this to give devs their easy-to-click URL in email and commit messages.</p><p><strong>support_files</strong>: Where you will keep the support files, like the clone-db-files.sh script and your template settings.php .</p>
<pre class="brush: bash; auto-links: true; collapse: false; first-line: 1; html-script: false; smart-tabs: true; tab-size: 4; toolbar: true; codetag" title=".git/hooks/post-receive">#!/bin/bash
# sets up a new working tree for every branch.
# cleans up after itself when branches are deleted
# sets up provisional DBs, too

mysql_user='root'
mysql_pw='your-mysql-password'
worktree_root="/var/www/public_html"
http_root="http://example.com"
support_files="/home/git"

while read oldrev newrev ref
do
&nbsp; branch=`echo $ref |cut -d/ -f3`
&nbsp; worktree="$worktree_root/$GL_REPO/$branch"
&nbsp; repo_sanitized=`echo ${GL_REPO//[-._]/}`
&nbsp; branch_sanitized=`echo ${branch//[-._]/}`
&nbsp; db_name="$repo_sanitized"_"$branch_sanitized"

&nbsp; # Exit nicely if this is the gitolite-admin repo.
&nbsp; if [ $GL_REPO = 'gitolite-admin' ]; then
&nbsp;&nbsp;&nbsp; exit 0
&nbsp; fi

&nbsp; # Did we delete a branch? Delete the DB and checkout
&nbsp; if [ $newrev = '0000000000000000000000000000000000000000' ]; then
&nbsp;&nbsp;&nbsp; if [ ! $branch = 'master' ]; then
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; echo "Deleting working tree and DB for branch $branch."
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; rm -rf $worktree
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mysqladmin -f -u "$mysql_user" --password="$mysql_pw" drop $db_name
&nbsp;&nbsp;&nbsp; else
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; echo "Refusing to delete working tree and db for master branch."
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; exit 1
&nbsp;&nbsp;&nbsp; fi
&nbsp; else
&nbsp;&nbsp;&nbsp; # Make the worktree if it doesn't exist
&nbsp;&nbsp;&nbsp; if [ ! -d "$worktree" ]; then
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mkdir -p $worktree
&nbsp;&nbsp;&nbsp; fi
&nbsp;&nbsp;&nbsp; # Check out the latest version in place.
&nbsp;&nbsp;&nbsp; git --work-tree=$worktree checkout -f -q $branch
&nbsp;&nbsp;&nbsp; echo "Checked out updated code to $http_root/$GL_REPO/$branch"
&nbsp;&nbsp;&nbsp; # Are we missing settings.php ? this could be a new branch or a fresh repo.
&nbsp;&nbsp;&nbsp; if [ ! -e "$worktree/sites/default/settings.php" ]; then
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; # Create settings.php from a template
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cp "$support_files"/template.settings.php "$worktree"/sites/default/settings.php
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; sed -i -e "s/_placeholder_/$db_name/g" "$worktree"/sites/default/settings.php
&nbsp;&nbsp;&nbsp; fi
&nbsp;&nbsp;&nbsp; # Is it a new branch?
&nbsp;&nbsp;&nbsp; if [ $oldrev = '0000000000000000000000000000000000000000' ]; then
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; # Create the DB
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mysqladmin -u "$mysql_user" --password="$mysql_pw" create $db_name
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; # Copy the DB from master branch. This is asynchronous because some DBs take a loooong time.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if [ ! $branch = "master" ]; then
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; email=`git log --format="%ae" "$newrev" -1`
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "$support_files"/clone-db-files.sh $GL_REPO $branch $email &amp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; echo "New branch detected, setting up db and files directories. This can take some time, depending on the size of the site. We'll email your git address ($email) when it's ready to use."
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; else
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; # New repo! Create the files directory and give them the install URL.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mkdir $worktree/sites/default/files
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; chmod ug+rwx $worktree/sites/default/files
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; echo "New repo detected, creating empty DB and files directory. Your site is prepared; go and install Drupal at&nbsp; $http_root/$GL_REPO/$branch/install.php"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; fi
&nbsp;&nbsp;&nbsp; fi
&nbsp;&nbsp;&nbsp; if [ ! $branch = 'master' ]; then
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; # so that HEAD always sits at master
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; git --work-tree=$worktree_root/$GL_REPO/master checkout -q -f master &amp;
&nbsp;&nbsp;&nbsp; fi
&nbsp; fi
done
</pre>
<p>We detect the new branch by the fact that the old ref is all zeros, and a deleted branch by the fact that the new ref is all zeros. We use mysqladmin to create or delete the branch database, depending on the operation. We create settings.php from a template, that has a placeholder for the DB name. Since these are just development environments, you should have all sensitive information scrubbed anyway... so this script assumes you use one mysql username and password for all your development sites.</p><p>I separated out the script which actually duplicates the db and files directories, because that tends to take awhile. With a small Drupal site it's a matter of seconds, but the largest site I tested with took almost 15 minutes to duplicate the 750MB DB. That's a real problem if your developer is sitting there waiting for their commit to finish, but not so bad if it happens in the background and sends a notification when it's done. So here is the duplication script. You'll have to set the same set of variables as in the git hook above, plus a logfile location. This script is run as the git user, so I find it's easiest to keep the logfile in the git user's homedir.</p>
<pre class="brush: bash; auto-links: true; collapse: false; first-line: 1; html-script: false; smart-tabs: true; tab-size: 4; toolbar: true; codetag" title="~git/clone-db-files.sh">#!/bin/bash
# Copy DB from master to a new branch, and email the results. The destination DB should already be created.
# Arguments: project name, branch, email address

mysql_user='root'
mysql_pw='mysql-password'
worktree_root="/var/www/public_html"
http_root="http://ec2-54-226-103-245.compute-1.amazonaws.com"
logfile="/home/git/new-branches.log"

project=$1
branch=$2
email=$3
repo_sanitized=`echo ${project//[-._]/}`
branch_sanitized=`echo ${branch//[-._]/}`
db_name="$repo_sanitized"_"$branch_sanitized"
mysql_start_vars="SET AUTOCOMMIT=0; SET UNIQUE_CHECKS=0; SET FOREIGN_KEY_CHECKS=0; SET GLOBAL INNODB_FLUSH_LOG_AT_TRX_COMMIT=2;"
mysql_stop_vars="SET AUTOCOMMIT=1; SET UNIQUE_CHECKS=1; SET FOREIGN_KEY_CHECKS=1; SET GLOBAL INNODB_FLUSH_LOG_AT_TRX_COMMIT=1;"

# Copy the DB from master branch
cat &lt;(echo "$mysql_start_vars") &lt;(mysqldump --opt --quick -u "$mysql_user" --password="$mysql_pw" "$project"_master) &lt;(echo "$mysql_stop_vars") | mysql -u "$mysql_user" --password="$mysql_pw" $db_name &gt;&gt; $logfile &amp;

# Copy the files directories - all sites except "all"
cd $worktree_root/$project/master
umask 002
if [ `find ./sites -type d -not -path "*/sites/all/*" -name 'files' -print0` ]; then
&nbsp; find ./sites -type d -not -path "*/sites/all/*" -name 'files' -print0|xargs -0 -I{} cp -R --no-preserve=mode,ownership --parents "{}" $worktree_root/$project/$branch/ &gt;&gt; $logfile &amp;
  wait
&nbsp; # Certain systems don't respect no-preserve in copy, so this is a just-in-case to make sure file modes are OK
&nbsp; cd $worktree_root/$project/$branch
&nbsp; find ./sites -not -path "*/sites/all/*" -name 'files' -print0 |xargs -0 -i{} chmod -R ug+rw "{}"
else
&nbsp; echo "WARN: No files directories found in Master branch checkout."
fi

# wait for the previous commands to finish processing
wait


# Send confirmation email
message="Your new branch environment for $branch is ready to use. The database and files have been copied from the master branch, and code is checked out in place. You can access the new environment at $http_root/$project/$branch."

echo $message | /usr/bin/mail -s "Environment is ready for $project branch: $branch" $email</pre>
<p>I opted not to use drush for the cloning, and I expect I'll get some flak for it. Basically I feel that this isn't really Drush's use case. We don't need the flexibility Drush provides, and we can trade it off for speed optimizations that are only acceptable in our very specific use case.</p><p>The only other file you need is the template settings.php file, which is not exactly rocket science. If you're following best practices, settings.php is never committed into your repo. You can see our solution for keeping settings.php customizations for dev in the repo at the bottom of our template file here. Note that since this is a development environment, it's assumed that you've already pruned sensitive information out of the database. That means it's safe to use a single username and password for all those dev databases (don't you DARE do this on production sites, though!). The up shot of this is that you can have a nice template settings.php that applies to all of the sites in your dev environment. If you're uncomfortable with it, there's nothing wrong with adding random password generation to the script above... just make sure to post your solution for that in the comments. :)</p>
<pre class="brush: php; auto-links: true; collapse: false; first-line: 1; html-script: false; smart-tabs: true; tab-size: 4; toolbar: true; codetag" title="template.settings.php">&lt;?php

// Git flow settings.php template

$databases = array (
&nbsp; 'default' =&gt;
&nbsp; array (
&nbsp;&nbsp;&nbsp; 'default' =&gt;
&nbsp;&nbsp;&nbsp; array (
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'database' =&gt; '_placeholder_',
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'username' =&gt; 'your_mysql_username_here',
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'password' =&gt; 'your_mysql_password_here',
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'host' =&gt; 'localhost',
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'port' =&gt; '',
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'driver' =&gt; 'mysql',
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 'prefix' =&gt; '',
&nbsp;&nbsp;&nbsp; ),
&nbsp; ),
);

# 5rings settings include. If you got custom settings, put em in here.
if (file_exists('sites/default/5rings.settings.php')) {
&nbsp; include('sites/default/5rings.settings.php');
}

</pre>
<p>Note the last line theres - that's how we include any settings that we need for dev environments. We just commit a file called 5rings.settings.php with customizations like disabling securepages, setting stage_file_proxy target, etc. That won't do any damage on live (though it's available there for easy reference), and it ensures that any special configuration needs are still tracked in our repo. Not to mention, it makes it possible for us to easily clone repos like this!</p><p>Enjoy your new git flow development environment. I welcome any questions or suggestions in the comments.</p>
