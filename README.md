# Exmaple of using kue with node's cluster

Quick modification of the cluster example code in the README for [kue](https://github.com/LearnBoost/kue)


# Run

run the kue monitor server:

     node kue_server.js


run the process:

     node test_cluster.js

This will also spew jobs to the console.

Open up a browser to <http://localhost:3001>

If you tweak the processing times in test_cluster.js, you can make the
jobs stick around long enough to see them.  On my computer they barely
show up.

# Note

I separated the server monitor from the job creation/processing code
because it was confusing to me on first glance what was going on.

# The difference with kue's README

The primary difference with the stock kue readme is that I'm just
following the node.js cluster documentation found
[here](http://nodejs.org/api/cluster.html#cluster_cluster)

Quite literally, I just cut and paste the code, then made it work.

So:

``` javascript

var kue = require('kue')
  , cluster = require('cluster')
  , jobs = kue.createQueue();

var numCPUs = require('os').cpus().length;


if (cluster.isMaster) {
    var redis = kue.redis.createClient()

    for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
    }
    function create2() {
        var name = ['tobi', 'loki', 'jane', 'manny'][Math.random() * 4 | 0];
        var job = jobs.create('email', {
            title: 'emailing ' + name + '', body: 'hello'
        }).save();

        setTimeout(create2, Math.random() * 1000 | 0);
    }

    create2();
    jobs.on('job complete', function(id){
        kue.Job.get(id, function(err, job){
            if (err) return;
            job.remove(function(err){
                if (err) throw err;
                console.log('removed completed job #%d', job.id);
            });
        });
    });

} else {

  // not cluster.isMaster, but rather a spawned job

  jobs.process('email', function(job, done){
    var pending = 5
      , total = pending;

    var interval = setInterval(function(){
      job.log('sending!');
      job.progress(total - pending, total);
      --pending || done();
      pending || clearInterval(interval);
    }, 10);
  });
}
```
