+++
date = '2025-12-09T22:47:21-03:00'
title = 'Making Carinata for Lima'
+++

I've been using MacOS for work for a few months now, and since I work at [Chainguard](https://www.chainguard.dev/) I need
to test our images with `docker` specifically because of some quirks with our test environments (and thats also what
our customers use sometimes), the problem with it is that every "docker implementation" on MacOS is either proprietary,
very inconvenient, doesn't work, or sometimes breaks on `docker-cli` updates.

Looking at them, we have:

- [Docker Desktop](https://docs.docker.com/desktop/) - Unfortunately proprietary but a pretty good one!
- [Orbstack](https://orbstack.dev/) - Seems pretty good, but also proprietary (and I haven't managed to set it up :3)
- [Podman Desktop](https://podman-desktop.io/) - Not proprietary! Awesome! I've tried using it and our test environments
dislike it due to the [docker API implemented by `podman`](https://podman.io/blogs/2020/07/01/rest-versioning.html)
being slightly too old for the latest `docker-cli` packaged by
[wolfi](https://github.com/wolfi-dev/os/blob/main/docker.yaml) and
[homebrew](https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/d/docker.rb)
- [Rancher Desktop](https://rancherdesktop.io/) - Not proprietary either, and insanely close to perfection! But the VM
it exposes sadly sometimes has way too old of a `dockerd` running... (a couple of releases old, not even that much, but
it still breaks my usecase.)

So that ended up leading me to [Lima](https://lima-vm.io/)! It is a
[CNCF-backed project](https://www.cncf.io/projects/lima/) that exposes a declarative way of creating virtual machines
that integrate with your host system, somewhat similarly to what [distrobox](https://distrobox.it/) or
[toolbx](https://containertoolbx.org/) do [on Linux](https://docs.getaurora.dev/guides/software/#distrobox-containers)
(but with VMs instead of containers :P). I messed around with it a bit and found out that there was a
["rootful docker" template](https://github.com/lima-vm/lima/blob/master/templates/docker-rootful.yaml),
that exposes a docker socket to your host that you can later
[symlink to `/var/run/docker.sock`](https://podman-desktop.io/docs/migrating-from-docker/customizing-docker-compatibility)...
which was exactly what I needed! An up-to-date docker installation on a linux guest that would seamlessly be integrated with my
host.

I used this setup for a bit and it seemed perfectly fine, latest docker, worked for what I had, and didn't break- well,
not exactly. Since I'm a person that thinkers _a lot_ with their system, I need to have an environment that is very
resiliant to my experimentation and use cases,
[which is _not_ the case with Ubuntu or Debian](https://docs.projectbluefin.io/FAQ/#rationale), since those can break
easily with package updates, modifications to `/usr`, or just the general messiness of managing a package-based system
with state all over the system where I can't exactly pinpoint the differences between one day's state to another one.
Since I've been using [`bootc`](https://bootc-dev.github.io/) for a while now on all my machines
([Bluefin for desktop](https://projectbluefin.io/),
[CentOS Stream image for my router](https://github.com/tulilirockz/taxifolia), and others), I thought this could be
a wonderful use case for it.

So thats exactly what I did! I was already messing with an image called [Zirconium](https://github.com/zirconium-dev/zirconium)
[where I had added aarch64](https://github.com/zirconium-dev/zirconium/pull/50) together with my [friend Bri (!)](http://github.com/b-),
so this was mostly just a matter of copy and pasting a bunch of GitHub Actions code and throwing together a small image
with `cloud-init` so that Lima could hook itself into it and set up the users (took a little to figure out but was pretty cool)
One symlink here, another service enabled there, and I finally had my perfect `docker` guest virtual machine!
An ephemeral, tightly controlled, automatically updating, read-only `/` environment, where `docker` and _only_ docker
(I guess SSH too) runs and does the sole job of exposing a docker socket under `$HOME/.lima/docker/sock/docker.sock` so
 that I can symlink and use it for my work.

This was an awesome experiment and I loved working on it! Most of my primary objectives are done with this
(I guess now I just need to implement automatic signing for the image), and it _should_ "just work" for a long while
without me having to maintain it all the time, and I wont need to worry about updates breaking things either! (or me
editing something from `/usr` and breaking the entire world :3c)

You can take a look at [the repository](https://github.com/tulilirockz/carinata) to figure out how to use it, I hope
you like it too!

