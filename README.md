# buildkite-queues
The Buildkite documentation is not perfectly clear on
the difference between normal agent `tags` and the special
`queue` tag. This repo presents a series of examples to
see how they operate in practice.

**[UPDATE]:** The main takeaway is that `queue` is a plain old
regular agent tag, except that agents and jobs that do not 
explicitly define a `queue` tag are automatically assigned
`queue=default`.

## Get started
You will need the Buildkite agent token from your buildkite
account to run these examples.

Useful Buildkite documentation:
* [Running Buildkite Agent with Docker](https://buildkite.com/docs/agent/v3/docker).
* [Setting tags on agents](https://buildkite.com/docs/agent/v3/cli-start#setting-tags)
* [Buildkite Agent Job Queues](https://buildkite.com/docs/agent/v3/queues)

## Background
When starting a buildkite agent, you can assign it zero or
more custom `tag`s. The keys and values of these tags can be
anything you want, but it makes sense to define a structured
format for organizational purposes.

These agent tags are reported to Buildkite and appear in
the agent's metadata. Go to the [agents list](https://buildkite.com/organizations/oasislabs/agents)
in your account and click on a random agent to see the list
of tags it was assigned when it was started.

These agent tags can be used to specify certain jobs to run
on agents with matching tags.

For example, suppose you had a job to build a go binary.
You could assign an agent a tag of `go=true` and
then specify the job to require an agent with that tag:

```
steps:
  - command: "script.sh"
    agents:
      go: 'true'
```

If an agent does not have the tag `go=true`, then it will
never run this job. This is a trivial example and you could
obviously get fancier and specify the specific version of
go running on an agent, etc. The possibilities are endless.

**NOTE:** The main point here is that if a job specifies an
agent requirement, then an agent without the corresponding
tag will never run that job.

HOWEVER, of course there is a caveat. The `queue` tag is a special case.

Whether you set it explicitly or not, every single build
agent is assigned a `queue`. By default, the config file
for an agent has `tags=queue=default`, which explicitly
sets the default queue. However, even if an agent is not
explicitly assigned a queue, it will implicitly be added
to the default queue.

An agent must match **ALL** of the tags specified by a job
in order to run that job. If the agent matches all of the tags,
but is in the wrong queue, then it won't run the job. If the agent
is in the correct queue, but does not match all of the other
tags specified by the job, then it won't run the job.

## Run the examples
To see how tags and queues interact with each other,
I created a series of examples in this repo. The pipeline
is setup to several jobs, each of which has a different
setup of tags and queues (exploring most of the interesting
permutations).

To avoid the possibility of committing the buildkite agent token
to git and also avoid littering my shell history with commands
containing the agent token, I put the agent token into the file 
`~/.buildkite/agent_token` and then use the following 
command to run the buildkite agent:

```
docker \
  run \
  -it \
  -e BUILDKITE_AGENT_TOKEN="`cat ~/.buildkite/agent_token`" \
  -v "$PWD/config/buildkite-agent.cfg:/buildkite/buildkite-agent.cfg:ro" \
  buildkite/agent
```

Notice that there are multiple buildkite configuration files in
this repo. You can run the buildkite agent with a different config
file and then observe which jobs are able to run on that agent.

Here is a summary:

![buildkite-agent-vs-jobs-summary-chart](https://github.com/conorgil/buildkite-example-queues/buildkite-agent-vs-jobs-summary-chart.png)

A screenshot of each scenario is included in the `images/` directory.

## API
Note that the Buildkite Rest API v3 returns tags in a field called
`meta_data`. I believe legacy versions of the Buildkite API used the
term metadata and that was renamed to tags in v3.
