---
layout: post
title:  Database Backup with Git
tags:   sysadmin database git

summary: >
    I came up with a simple, intuitive database backup solution using Git to
    track revisions.
---

I'm new to system/database adminstration. I'm currently running a web
application that recieves around ~55,000 hits/day, and whose database grows by
about 60 MiB of plain-text per day.

I came up with a simple, intuitive database backup solution using Git to track
revisions. (And I [wasn't the first][ref-article].) If you've got a
small- to medium-sized database, this may work for you too.


Hand-versioning: really inefficient
-----------------------------------

Backing up your database is important, but one must also consider keeping
*revisions* of those backups, since if you keep only the latest backup, but you
risk overwriting a good backup with bad ones.

    bad-dump.sql -> good-dump-from-yesterday.sql

You might consider writing each backup to its own file, but that means each
backup will consume the entire size of the database each backup, in my case,
~10 GiB per day.

    dump1.sql   10 GiB
    dump2.sql   10 GiB
    dump3.sql   10 GiB

If most of the data in your database is static, that's a lot of wasted space.x


Enter: Git
----------

[Git][] is a revision control system used primarily in software development to
track changes in source code over time. A database backup, a.k.a a [database
dump][db-dump], is just a bunch of SQL statements in a text file, so why not
use Git to track changes in your database backups as well?

[git]:http://en.wikipedia.org/wiki/Git_(software)
[db-dump]:http://en.wikipedia.org/wiki/Database_dump

The backup process is simple: Dump your database over the previous backup, and
commit. Here's an example backup script using MySQL:

{% highlight sh %}
#! /bin/sh

# Settings
BACKUP_DIR="/path/to/your/backup/directory"
DB_USER="[your database user]"
DB_PASS="[your database user's password]"
DB="[database name]"
DB_DUMP="$BACKUP_DIR/$DB.sql"

# Create your backup directory if it doesn't exist
mkdir -p $BACKUP_DIR

# Dump the database
mysqldump -u $DB_USER -p$DB_PASS --skip-extended-insert $DB &gt; $DB_DUMP

# Change to the backup directory and initialize a new Git repo if necessary
cd $BACKUP_DIR
git init

# Add the database to the repo and commit
git add $DB_DUMP
git commit -m "Update database dump"
{% endhighlight %}

A couple things to note about this script:

- `--skip-extended-insert` makes mysqldump create an insert statement on a new
  line for each row. This means a larger initial commit, but all future commits
  will be smaller due to the fact that Git tracks *line* changes.
- Running `git init` on an existing repository is safe, and will not overwrite
  things that are already there.

[man-git-init]:http://linux.die.net/man/1/git-init

Now, just throw this script in `/etc/cron.daily/` for instant, daily versioned
database backup.

Now what?
---------

Anything you can do with a normal Git repository, including:

- Push to a remote repository on a **backup server**
- Pull the latest database changes to your **development server**
- **View changes** to your database with `git whatchanged`

Optimizations
-------------

As [suggested](http://viget.com/extend/backup-your-database-in-git#comment-400539436)
by Bram Schoenmakers, here are some Git configurations that should decrease
disk usage of the repository:

- **core.compression = 9** - Flag for gzip to specify the compression level for
  blobs and packs. Level 1 is fast with larger file sizes, level 9 takes more
  time but results in better compression.
- **repack.usedeltabaseoffset = true** - Defaults to *false* for compatibility
  reasons, but is supported with Git >=1.4.4.
- **pack.windowMemory = 100m** - (Re)packing objects may consume lots of
  memory. To prevent all your resources [from going] down the drain it's useful
  to put some limits on that. There is also *pack.deltaCacheSize*.
- **pack.window = 15** - Defaults to *10*. With a higher value, Git tries
  harder to find similar blobs.
- **gc.auto = 1000** - Defaults to *6700*. As indicated in
  [the article][ref-article], it is recommended to run `git gc` every once in a
  while. Personally, I run `git gc --auto everyday`, so [it] only packs things
  when there's enough garbage. `git gc --auto` normally only triggers the
  packing mechanism when there are 6700 loose objects around. This flag lowers
  this amount.
- **gc.autopacklimit 10** - Defaults to *50*. Every time you run `git gc`, a
  new pack is generated [of] the loose objects. Over time you get too many
  packs which wastes space. It is a good idea to combine all packs once in a
  while into a single pack, so all objects can be combined and deltified. By
  default `git gc` does this when there are 50 packs around. But for this
  situation a lower number may be better.

References
----------

- ["Backup your Database in Git"][ref-article]
   by David Eisinger
- [git-init man page](http://linux.die.net/man/1/git-init)

[ref-article]:http://viget.com/extend/backup-your-database-in-git
