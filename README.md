Stalker - a job queueing DSL for Beanstalk
==========================================

[Beanstalkd](http://kr.github.com/beanstalkd/) is a fast, lightweight queueing backend inspired by mmemcached.  The [Ruby Beanstalk client](http://beanstalk.rubyforge.org/) is a bit raw, however, so Stalker provides a thin wrapper to make job queueing from your Ruby app easy and fun.

Queueing jobs
-------------

From anywhere in your app:

    require 'stalker'

    Stalker.enqueue('email.send', :to => 'joe@example.com')
    Stalker.enqueue('post.cleanup.all')
    Stalker.enqueue('post.cleanup', :id => post.id)

Working jobs
------------

In a standalone file, typically jobs.rb or worker.rb:

    require 'stalker'
    include Stalker

    job 'email.send' do |args|
      Pony.send(:to => args['to'], :subject => "Hello there")
    end

    job 'post.cleanup.all' do |args|
      Post.all.each do |post|
        enqueue('post.cleanup', :id => post.all)
      end
    end

    job 'post.cleanup' do |args|
      Post.find(args['id']).cleanup
    end

Running
-------

First, make sure you have Beanstalkd installed and running:

    $ sudo port install beanstalkd
    $ beanstalkd

Stalker:

    $ sudo gem install stalker

Now run a worker using the stalk binary:

    $ stalk jobs.rb
    [Sat Apr 17 14:13:40 -0700 2010] Working 3 jobs  :: [ email.send post.cleanup.all post.cleanup ]

Stalker will log to stdout as it starts working each job.

Filter to a list of jobs you wish to run with an argument:

    $ stalk jobs.rb post.cleanup.all,post.cleanup
    [Sat Apr 17 14:13:40 -0700 2010] Working 2 jobs  :: [ post.cleanup.all post.cleanup ]

In a production environment you may run one or more high-priority workers (limited to short/urgent jobs) and any number of regular workers (working all jobs).  For example, two workers working just the email.send job, and four running all jobs:

    $ for i in 1 2; do stalk jobs.rb email.send > log/urgent-worker.log 2>&1; end
    $ for i in 1 2 3 4; do stalk jobs.rb > log/worker.log 2>&1; end

Tidbits
-------

* Jobs are serialized as JSON, so you should stick to strings, integers, arrays, and hashes as arguments to jobs.  e.g. don't pass full Ruby objects - use something like an ActiveRecord/MongoMapper/CouchRest id instead.
* Because there are no class definitions associated with jobs, you can queue jobs from anywhere without needing to include your full app's environment.
* If you need to change the location of your Beanstalk from the default (localhost:11300), set BEANSTALK_URL in your environment, e.g. export BEANSTALK_URL=beanstalk://example.com:11300/
* The stalk binary is just for convenience, you can also run a worker with a straight Ruby command:
    $ ruby -r jobs -e Stalker.work

Meta
----

Created by Adam Wiggins

Heavily inspired by [Minion](http://github.com/orionz/minion) by Orion Henry

Released under the MIT License: http://www.opensource.org/licenses/mit-license.php

http://github.com/adamwiggins/stalker

