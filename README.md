# Cross Platform Python Development Docker & Jenkins

## Context

For reasons outside my control I needed to use [Jenkins][jenkins].

Was surprised that Jenkins required specifying the shell for all terminal commands.
Not really looking to do two `Jenkinsfile`s prefixing with `bat` vs `sh`,
I created a relatively platform invariant file and the corresponding `Dockerfile`s.

The agents were both pretty barebones, so you may find that you can offload
steps to the agent and avoid the `Dockerfile` all together. In my case this was
a useful exercise to understand more about `docker` and Jenkins.

## Gotchas

1. `Dockerfile.windows` runs on Windows Server Core, which needs a specific license on the machine run.

  > The Windows container feature is only available on Windows Server 2016 (Core and with Desktop Experience), Windows 10 Professional and Enterprise (Anniversary Edition) and later.
  > > See details on: [Docker - Windows Server Core][ms-windowsservercore]
2. I had enormous troubles with the `PATH` variable on the unix agent and had to
   add a workaround. I'm sure there is a better way, and perhaps you won't have
   this problem.

[jenkins]: https://www.jenkins.io "Jenkins Website"
[ms-windowsservercore]: https://hub.docker.com/_/microsoft-windows-servercore
