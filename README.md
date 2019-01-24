# buildkite-queues
The Buildkite documentation is not perfectly clear on
the difference between normal agent `tags` and the special
`queue` tag. This repo presents a series of examples to
see how they operate in practice.

## Get started
You will need the Buildkite agent token from your buildkite
account to run these examples.

Useful Buildkite documentation:
* [Running Buildkite Agent with Docker](https://buildkite.com/docs/agent/v3/docker).
* [Setting tags on agents](https://buildkite.com/docs/agent/v3/cli-start#setting-tags)
* [Buildkite Agent Job Queues](https://buildkite.com/docs/agent/v3/queues)

To avoid the possibility of committing the agent token to git
and littering my shell history with commands containing the
agent token, I put the agent token into the file 
`~/.buildkite/agent_token` and then use the following 
command to run the buildkite agent:
```
docker \
  run \
  -it \
  -e BUILDKITE_AGENT_TOKEN="`cat ~/.buildkite/agent_token`" \
  -v "$PWD/buildkite-agent.cfg:/buildkite/buildkite-agent.cfg:ro" \
  buildkite/agent
```


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

## Example 1 - agent has default queue, job does not specify queue

## Example 2 - agent has no queue, job does not specify queue

## Example 3 - agent has custom queue, job does not specify queue

## Example 4 - agent has custom queue, job specifies custom queue

## Example 5 - agent has custom queue, job specifies custom queue and node version

## Example 5 - agent has custom queue and node version, job specifies custom queue and node version

