# vrci

Build and publish VRChat avatars and worlds from CI, without a desktop, and
without handing your account to anyone.

> **You're early.** Nothing here is finished. Most of these repos are
> scaffolding with a plan attached. If you found this looking for something
> to install today, come back later. If you found this because you have had
> the same problems, read on.

## Why this exists

Uploading VRChat content means sitting at a machine that can build it,
watching Unity work, and doing nothing else until it finishes.

That is fine once. It is less fine when you want to push a small change at
midnight and the build wants twenty minutes of your evening and every core
your CPU has. It is worse when the update is not urgent at all, you just
want to be in VRChat, and instead you are waiting on a progress bar before
you can launch the game. On a weaker machine it stops being an
inconvenience and becomes a wall: not a framerate cost, an actual "I cannot
play until this is done."

vrci moves that work somewhere else. Push a change, go do something else,
and let the build happen without you. Hop into VRChat immediately and let
the avatar update land while you are already in a world, picked up the next
time you travel between instances.

The stance underneath it: VRChat content authoring should get to use the
same tooling that conventional game development takes for granted.
Continuous integration, review, automated publishing, and version history
are ordinary in gamedev. There is no good reason avatars and worlds should
be stuck doing everything by hand on one specific computer.

## What it makes possible

Today, the goal is simple: your builds happen on CI instead of on your desk.

Beyond that, the interesting part is what CI has always unlocked everywhere
else.

**Contributions that do not require the maintainer to build anything.**
Someone opens a PR against a world. The maintainer reviews it, decides it is
good, merges it. The publish happens on its own. The maintainer's job moves
from "pull this down, open Unity, build it, supervise the upload" to "make
sure this is right." Building locally is still the correct way to vet a
large change. It just stops being the only way anything ever ships.

**Trivial changes with no human in the loop at all.** A string edit, an
asset swap, a typo in a sign. Changes that can be made from the GitHub web
editor and published without anyone opening a project.

**Open source worlds that can actually accept help.** Community worlds could
take contributions the way software does, without every change routing
through one person's machine and one person's evening.

**Teams publishing to a shared account** without every member needing the
credentials or the hardware. That is future scope, not something that works
today, but it is what the architecture is being shaped for.

**Publishing without owning a machine that can build.** You hold your own
credentials. You run your own gateway. Nobody else is in the middle, and you
are not trusting some third party with your login to get around not having a
capable PC.

## How it works

You deploy a gateway. It is a server component that holds one long-lived
VRChat session and acts as the only thing that ever talks to VRChat with
your credentials attached.

CI runs Unity headlessly, builds your content, and hands the result to your
gateway. The build environment never sees your VRChat credentials or your
session. It talks to your gateway, and your gateway talks to VRChat.

Everything is yours: your gateway, your account, your machine. vrci is the
plumbing between them, not a service you sign up for.

### What the pipeline is built on

The aim is for this to be boring and GitHub-native. Design intent rather
than a feature list, since most of it is still being built:

- **Incremental builds.** Unity's `Library` folder and resolved packages are
  cached between runs, so a build is not starting from zero every time. This
  is what makes building on every push reasonable instead of wasteful.
- **Your workflow file stays short.** The pipeline lives in reusable actions.
  Content repos reference them with `uses:` rather than each carrying its own
  copy of a hundred lines of YAML that then drift apart.
- **Failures point at themselves.** Each stage gets its own collapsible log
  group: license, dependencies, editor startup, build, publish. When
  something breaks overnight, you should be able to tell which stage died
  from the run summary without reading a full Unity log.
- **Pinned versions.** Package versions are pinned rather than floated. The
  SDK moving under you should be a decision you make, not something that
  happens to you while you sleep.
- **No Unity Hub.** Builds run on standard [game.ci](https://game.ci/) Unity
  images with the editor invoked directly. The Hub does not cooperate
  headlessly and is not in the loop at all.
- **GitHub-hosted by default, self-hosted if you want it.** Nothing depends
  on GitHub's own runners specifically, so moving builds onto your own
  hardware stays an option rather than a rewrite.
- **The incompat list is fetched, not bundled.** Packages known to break
  headless builds are published as a list your pipeline reads at build time,
  so it stays current without you bumping a version to receive it.

## The repos

| Repo | What it is |
|---|---|
| [`vrci`](https://github.com/vrci-team/vrci) | Home. Architecture, progress, and setup docs as they get written. |
| [`gateway`](https://github.com/vrci-team/gateway) | The server component you deploy. Holds the session, owns everything VRChat-facing. |
| [`core`](https://github.com/vrci-team/core) | The Unity side. Talks to your gateway. |
| [`autobuild`](https://github.com/vrci-team/autobuild) | Triggers SDK builds headlessly. |
| [`actions`](https://github.com/vrci-team/actions) | Reusable GitHub Actions that run the pipeline. |
| [`incompat`](https://github.com/vrci-team/incompat) | Packages known to break headless builds, and why. |
| [`test-avatar`](https://github.com/vrci-team/test-avatar) / [`test-world`](https://github.com/vrci-team/test-world) | Real content used to prove the pipeline works before it touches anything that matters. |
| [`ci`](https://github.com/vrci-team/ci) | Orchestration mode. Not started yet. |

## Prior art and thanks

vrci is not built from nothing. It leans heavily on work by
[**@MisakaL**](https://github.com/Misaka-L), across several projects:

- [**VRChat Content Publisher**](https://github.com/project-vrcz/content-publisher)
  and its Unity connect package, which are the base this extends. The idea
  of moving compression, auth, and upload out of the editor is theirs, and
  it is the thing that makes the rest of this possible.
- [**yet-another-sdk-patch**](https://github.com/project-vrcz/yet-another-sdk-patch),
  used more or less as a library here.
- [**vrchat-content-auto-build**](https://github.com/vrcau/vrchat-content-auto-build),
  now unmaintained, which vrci revives and rebuilds against the current SDK.

Conversations with them shaped a lot of the design. A fair amount of this
project would not exist without their work, and the parts that would exist
would be considerably worse.

Also relied on:

- [**Tailscale**](https://tailscale.com/), currently required to connect
  your CI to your gateway securely. The goal is to make this optional as the
  networking gets more modular, but for now you need it.
- [**gharp**](https://github.com/muhac/actions-runner-pool), if you want
  self-hosted runners shared across your personal repos. Not required, just
  a good fit for the problem.
- [**unity-ci**](https://game.ci/) for the Unity Docker images that make
  headless builds practical at all.

## Status

Early. Actively being built. Expect things to move, break, and get renamed.

If you are here because you have hit the same wall, issues and discussions
are open, and the thinking is documented as it happens rather than after the
fact.

---

vrci is not affiliated with or endorsed by VRChat Inc.
