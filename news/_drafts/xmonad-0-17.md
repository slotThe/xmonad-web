---
---

# xmonad and xmonad-contrib 0.17.0 are available

New versions of xmonad and xmonad-contrib have been released.  Check out
our [download page]({{ site.baseurl }}{% link download.md %}) for
instructions on where to get them.

It's been a little over 3 years since xmonad 0.15, and a little over 2 years
since xmonad-contrib 0.16. A lot has happened. This post is an overview of
the most important changes. To celebrate this milestone, we announce [a contest
for a new xmonad logo](#logo-contest).

Last but not least, [we're looking to raise funds](#looking-for-funding) to
keep the XMonad project and community alive and relevant in the next decade.
With the [final X.Org Server 21.1 release][xorg-server-21.1] being announced
today, there are some obvious challenges ahead of us.

<!-- TODO: check/adjust -->
[xorg-server-21.1]: https://lists.x.org/archives/xorg-announce/2021-October/003114.html

### Table of Contents
{:.no_toc}

  + TOC
  {:toc}

## xmonad 0.17.0

<!-- TODO: $ git shortlog v0.15..origin/master --summary --no-merges | awk '{ sum += $1; count += 1 } END { print sum, count }' -->
This release includes 156 non-merge commits by 22 contributors!  For a full
summary of all the changes, see [xmonad's CHANGES.md] file.

[xmonad's CHANGES.md]: https://github.com/xmonad/xmonad/blob/v0.17.0/CHANGES.md

### Major Breaking Changes

  + `util/GenerateManpage.hs` is no longer distributed in the tarball.

### Selected Features and Improvements

  + Migrated `X.L.LayoutCombinators.(|||)` into `XMonad.Layout`,
    providing the ability to directly jump to a layout with the
    `JumpToLayout` message.

  + Improved handling of XDG directories.

    1. If all three of xmonad's environment variables
       (`XMONAD_DATA_DIR,` `XMONAD_CONFIG_DIR`, and `XMONAD_CACHE_DIR`)
       are set, use them.

    2. If there is a build script called `build` (see
       [here](https://github.com/xmonad/xmonad-testing/tree/master/build-scripts)
       for usage examples) or configuration `xmonad.hs` in `~/.xmonad`,
       set all three directories to `~/.xmonad`.

    3. Otherwise, use the `xmonad` directory in `XDG_DATA_HOME`,
       `XDG_CONFIG_HOME`, and `XDG_CACHE_HOME` (or their respective
       fallbacks).  These directories are created if necessary.

    In the cases of 1. and 3., the build script or executable is
    expected to be in the config dir.

    Additionally, the xmonad config binary and intermediate object files
    were moved to the cache directory (only relevant if using XDG or
    `XMONAD_CACHE_DIR`).

  + Recompilation now detects `stack.yaml` (can be a symlink) alongside
    `xmonad.hs` and switches to using `stack ghc`.

    Additionally, deprecation warnings during recompilation are no
    longer suppressed.  These can still be suppressed manually using an
    `OPTIONS_GHC` pragma with `-Wno-deprecations`.

## xmonad-contrib 0.17.0

<!-- TODO: $ git shortlog v0.16..origin/master --summary --no-merges | awk '{ sum += $1; count += 1 } END { print sum, count }' -->
This release includes 582 non-merge commits by 57 contributors!  For a full
summary of all the changes, see [xmonad-contrib's CHANGES.md] file.

[xmonad-contrib's CHANGES.md]: https://github.com/xmonad/xmonad-contrib/blob/v0.17.0/CHANGES.md

### Major Breaking Changes

  + All modules that export bitmap fonts as their default

    - If xmonad is compiled with XFT support (the default), use an XFT
      font instead.  The previous default expected an X11 misc font
      (PCF), which is not supported in pango 1.44 anymore and thus some
      distributions have stopped shipping these.

      ^

  + All modules still exporting a `defaultFoo` constructor

    - All of these were now removed.  You can use the re-exported `def`
      from `Data.Default` instead.

      ^

  + `XMonad.Hooks.StatusBar`, `XMonad.Hooks.StatusBar.PP` (previously
    `XMonad.Hooks.DynamicLog`) and `XMonad.Util.Run`

    - `spawnPipe` no longer uses binary mode handles and defaults to the
      current locale encoding instead, `xmonadPropLog` and
      `xmonadPropLog'` now encode their input string in UTF-8, and
      `dynamicLogString` no longer encodes its output in UTF-8.  When
      these functions are used together, everything should continue to
      work as it always has, but in isolation behaviour might change.

      To get the old `spawnPipe` behaviour, `spawnPipeWithNoEncoding`
      can now be used, and `spawnPipeWithUtf8Encoding` was added as well
      to force UTF-8 regardless of locale.

    - `XMonad.Hooks.DynamicLog` will be deprecated soon in favor of
      `XMonad.Hooks.StatusBar` which offers a nicer and easier to use
      interface for status bars. Laptop users who (dis)connect external
      monitors dynamically should definitely try this new interface:
      `XMonad.Hooks.DynamicBars` has been deprecated already.

  + `XMonad.Hooks.EwmhDesktops`

    - It is no longer recommended to use standalone hooks directly:

      Instead of `ewmhDesktopsStartup`, `ewmhDesktopsLogHook(Custom)` and
      `ewmhDesktopsEventHook(Custom)`, users should now use the `ewmh`
      combinator. The `Custom` variants have been replaced by a [more
      composable interface][EwmhDesktops:customization] which now also allows
      simply marking windows that request focus as urgent instead of
      immediately focusing them.

      Instead of `fullscreenEventHook`, use `ewmhFullscreen`, which now
      advertises fullscreen support to applications, fixing fullscreen in
      [mpv][] and many others.

  + `XMonad.Hooks.Script`

    - `execScriptHook` now has an `X` constraint (was: `MonadIO`), due
      to changes in how the xmonad core handles XDG directories.

      ^

  + `XMonad.Actions.WorkspaceNames`

    - The type of `getWorkspaceNames` was changed to fit into the new
      `ppRename` field of `PP`.

      ^

[EwmhDesktops:customization]: https://xmonad.github.io/xmonad-docs/xmonad-contrib-0.17.0/XMonad-Hooks-EwmhDesktops.html#g:2
[mpv]: https://mpv.io/

### Selected Features and Improvements

  + `XMonad.Actions.EasyMotion`

    A new module that allows selection of visible screens using a key
    chord—inspired by [vim-easymotion]. See the animation in the
    vim-easymotion repo to get some idea of the functionality of this
    EasyMotion module.

  + `XMonad.Prompt.OrgMode`

    A prompt for interacting with [org-mode]. It can be used to quickly
    save TODOs, NOTEs, and the like with the additional capability to
    schedule/deadline a task, or use the primary selection as the
    contents of the note.

  + `XMonad.Hooks.StatusBar`

    A new module, providing a nicer, composable interface for status bars that
    replaces `XMonad.Hooks.DynamicLog` and `XMonad.Hooks.DynamicBars`.
    Supports property-based as well as pipe-based status bars, multiple status
    bars (Xinerama), and takes care of restarting the bars as needed.

  + `XMonad.Hooks.Rescreen`

    A new module, providing custom hooks for screen (xrandr) configuration
    changes. These are used internally by `XMonad.Hooks.StatusBar` to restart
    status bars/systrays after xrandr, and can also be used to invoke xrandr
    or autorandr when an output is (dis)connected.

  + `XMonad.Util.Hacks`

    A collection of hacks and fixes that should be easily acessible to
    users, like a fix for the fullscreen behaviour of chromium based
    applications when using windowed fullscreen, or a fix to make
    certain Java applications play more nicely with xmonad.

  + `XMonad.Hooks.WindowSwallowing`

    A new module to implement window swallowing; optionally via
    sublayouting.  Hide parent windows like terminals when opening other
    programs (like image viewers) from within them, restoring them once
    the child application closes.

  + `XMonad.Util.ClickableWorkspaces`

    A new module providing `clickablePP`, which when applied to the `PP`
    pretty-printer used by `XMonad.Hooks.StatusBar.PP`, will make the
    workspace tags clickable in XMobar (for switching focus).

    Note that `XMonad.Layout.IndependentScreens` and
    `XMonad.Actions.WorkspaceNames` now also use the same composable
    `ppRename` interface as `XMonad.Util.ClickableWorkspaces` so it's much
    easier to use them with one another.

  + `XMonad.Util.Loggers`

    - Added `logTitles` to log all window titles (focused and unfocused
      ones) on the focused workspace, as well as `logTitlesOnScreen` as
      a screen-specific variant thereof.

      ^

  + `XMonad.Actions.TopicSpace`

    - Added `TopicItem`, as well as the helper functions `topicNames`,
      `tiActions`, `tiDirs`, `noAction`, and `inHome` for a drastically
      simpler specification of topics.

      ^

  + `XMonad.Hooks.ServerMode`

    - To make it easier to use, the `xmonadctl` client is now included
      in `scripts/`.

      ^

[vim-easymotion]: https://github.com/easymotion/vim-easymotion
[org-mode]: https://orgmode.org/

## Logo Contest

We'd also like to announce a contest to create a new xmonad logo to
replace the current one.  If you want to participate, please submit at
most *three* logos (in SVG format) to <!-- TODO: DISCUSSION --> until
the 31st of January.  **Please do not vote for a logo yet—voting will
start after the above deadline.**  Your logo will need to be licensed
under a suitable freely distributable license; for example, the CC BY-SA
4.0.

The prize for the winner is `$100`!

For voting, we will use an [instant-runoff] type system.  In short:

  - Users rank a minimum of three submissions according to their
    preferences in decreasing order.
  - If a logo wins a majority of first-preference votes (> 50%) it is
    chosen.
  - If not, we eliminate the logo with the least amount of first
    preference votes, exposing the second preference of those who voted
    for it.
  - Rinse and repeat until we have a majority.
  - If at the end of all that we still don't have a majority, we will
    act as a tiebreaker and internally vote for a winner from the
    remaining logos.

We will retain veto power in case of inappropriate logos.  Team members
whose submitted entries are amongst the remaining choices will not
partake in the potential tiebreaking vote.

[instant-runoff]: https://en.wikipedia.org/wiki/Instant-runoff_voting

## Looking for Funding

TL;DR:
[![GitHub Sponsors](https://img.shields.io/github/sponsors/xmonad?label=GitHub%20Sponsors&logo=githubsponsors){:.d-inline}][gh:xmonad:sponsors]
[![Open Collective](https://img.shields.io/opencollective/all/xmonad?label=Open%20Collective&logo=opencollective){:.d-inline}][oc:xmonad]

XMonad has been around since 2007 and has a great track record in stability.
There's an active, vibrant community of users and developers who help each
other and contribute fixes and extensions. Keeping this community organized,
reviewing and merging contributions and responding to bug reports is almost a
full-time job, however, not to mention doing actual development. We've been
struggling to find volunteers to do all this work, especially the less
exciting bits like documentation.

Despite that struggle, we managed to repay a portion of our technological and
organizational debt over the last year, and to pick up the pace of
development:

![activity chart]({{ site.baseurl }}/images/{{ page.slug }}/commits.svg){:.img-fluid .px-md-5}

We're worried about the sustainability of that pace, however. As with the
previous generations of maintainers, our studies and sabbaticals won't last
forever.

**Your help is needed to keep the project alive and well. We'd like to raise
enough funds to enable at least one core developer to work on XMonad full-time
(or several part-time).** To learn more about our plans for the future, see
our [GitHub Sponsors][gh:xmonad:sponsors] profile.

Thanks to [one unexpected appearance on the Hacker News front
page][hn:xmonad], we already gathered a couple dozen sponsors who donate in
total over $300 a month. We're incredibly grateful for this support! Our
fiscal host is the [Open Source Collective][OSC], so our budget is fully
transparent on [our Collective's page][oc:xmonad].

**If you also want to aid our efforts going forward, please consider becoming
a supporter as well. We accept both monthly and one-time donations through
[GitHub Sponsors][gh:xmonad:sponsors] and [Open Collective][oc:xmonad].**

Thank you!

[Wayland]: https://en.wikipedia.org/wiki/Wayland_(display_server_protocol)
[waymonad]: https://github.com/waymonad/waymonad
[OSC]: https://www.oscollective.org/
[hn:xmonad]: https://news.ycombinator.com/item?id=28793941
[oc:xmonad]: https://opencollective.com/xmonad
[gh:xmonad:sponsors]: https://github.com/sponsors/xmonad
