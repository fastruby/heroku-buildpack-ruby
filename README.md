# Heroku Buildpack for Ruby (next_rails dual-boot fork)

![ruby](https://raw.githubusercontent.com/heroku/buildpacks/refs/heads/main/assets/images/buildpack-banner-ruby.png)

This is a fork of [Heroku's official Ruby buildpack](https://github.com/heroku/heroku-buildpack-ruby). The **only** thing it adds is dual-boot support through the `BUNDLE_GEMFILE` environment variable. For all standard Ruby, Rack, and Rails buildpack behavior, and for full documentation, use the official buildpack and the [Heroku Ruby Support](https://devcenter.heroku.com/articles/ruby-support) docs.

> ## ⚠️ Read this first: changing `BUNDLE_GEMFILE` requires a rebuild, not just a config change
>
> Heroku triggers a **re-release but not a rebuild** when you change a config var. The build is what installs gems, so flipping `BUNDLE_GEMFILE` on its own never installs the other Rails version's gems.
>
> Example: an app built with Rails 6.1 only has 6.1 gems in its slug. If you then change `BUNDLE_GEMFILE` to `Gemfile.next` (Rails 7.0):
>
> - Heroku runs a new release with the changed env but does **not** rebuild the slug.
> - The slug still has only the previous version's gems. In some cases the re-release may fail, and Heroku might revert the config var to its previous value. If that happens, the env change ends up **not actually applied at runtime**, so Heroku reports the var changed while the app keeps running the previous version. Confusing, but expected.
>
> **To switch versions: change `BUNDLE_GEMFILE` and then trigger a deploy (e.g. push a commit) so the correct gems are installed at build time.** Do not rely on flipping the config var by itself.

## What this fork is for

It lets a single app switch between its current Gemfile and a [`next_rails`](https://github.com/fastruby/next_rails)-style alternative (e.g. `Gemfile.next`) at deploy time. `next_rails` is maintained by [FastRuby.io](https://www.fastruby.io).

- `BUNDLE_GEMFILE` **unset** → behaves exactly like the official `heroku/ruby` buildpack (uses `Gemfile` / `Gemfile.lock`). Drop-in replacement.
- `BUNDLE_GEMFILE=Gemfile.next` → builds and runs against `Gemfile.next` / `Gemfile.next.lock`.

## Usage

1. Confirm the app boots locally on the next version:

   ```sh
   BUNDLE_GEMFILE=Gemfile.next bundle exec rails -v
   ```

2. Point the app at this fork (pin to a branch or tag):

   ```sh
   heroku buildpacks:set https://github.com/fastruby/heroku-buildpack-ruby#use_gemfile_next_v359 -a <app>
   ```

3. Set the config var:

   ```sh
   heroku config:set BUNDLE_GEMFILE=Gemfile.next -a <app>
   ```

4. Deploy. This is the rebuild that installs the next gems (see the warning above):

   ```sh
   git push heroku <branch>:main
   ```

5. Verify:

   ```sh
   heroku run "bundle exec rails -v" -a <app>
   ```

To go back to the current version, unset the var and redeploy:

```sh
heroku config:unset BUNDLE_GEMFILE -a <app>
# then deploy again
```

## Alternative: a next-only buildpack

If you want an app that always runs the next version without managing `BUNDLE_GEMFILE`, use our sibling fork [`fastruby/heroku-buildpack-ruby-gemfile-next`](https://github.com/fastruby/heroku-buildpack-ruby-gemfile-next). It always uses `Gemfile.next` / `Gemfile.next.lock` and needs no `BUNDLE_GEMFILE`. To return to the current version, switch the app's buildpack back to the official `heroku/ruby`.

## Documentation

This fork only changes which Gemfile drives the build. For everything else, see:

- [Official Heroku Ruby buildpack](https://github.com/heroku/heroku-buildpack-ruby)
- [Heroku Ruby Support](https://devcenter.heroku.com/articles/ruby-support)
- [Buildpacks](https://devcenter.heroku.com/articles/buildpacks)
- [next_rails](https://github.com/fastruby/next_rails)

## License

See [LICENSE](LICENSE). Originally created by Heroku, Inc.
