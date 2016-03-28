[Heroku Buildpack](https://devcenter.heroku.com/articles/buildpacks) for [Kong](https://getkong.org) with Postgres
=========================
Based on Kong version 0.8.0rc1 patched for compatibility with Heroku. Kong is also temporarily patched to use pgmoon with SSL capability included (not currently merged in the official pgmoon repo)

This is a fork of Mars' original Kong buildpack - i've temporarily stripped out Cassandra while i play with the new Postgres support in 0.8.0rc1.

_Links to Kong documentation in this README still point to 0.7.x until 0.8.x is online._

Usage
-----

### Beginner

Deploy the [heroku-kong app](https://github.com/chrisanderton/heroku-kong-pg) to get started.

### Expert

* `kong.yml`
  * config template in `config/kong.yml.etlua`
    * buildpack detects this file in the app
    * [sample config file](config/kong.yml.etlua.sample)
* Lua source in the app
  * [Kong plugins](https://getkong.org/docs/0.7.x/plugin-development/):
    * `lib/kong/plugins/{NAME}`
    * Add each Kong plugin name to the `plugins_available` list in `config/kong.yml.etlua` 
    * See: [Plugin File Structure](https://getkong.org/docs/0.7.x/plugin-development/file-structure/)
  * Lua rocks
    * specify in the app's `.luarocks` file
    * each line is `{NAME} {VERSION}`
  * Other Lua source modules
    * `lib/{NAME}.lua` or
    * `lib/{NAME}/init.lua`

### Environment variables

  * `PORT` exposed on the app/dyno
    * set automatically by the Heroku dyno manager
  * `KONG_CLUSTER_SECRET` symmetric encryption key
    * generate value with command `serf keygen`; requires [Serf](https://www.serfdom.io/downloads.html)
  * `DATABASE_URL`
    * Postgres datastore

Background
----------
The first time this buildpack builds an app, the build time will be significantly longer as Kong and its dependencies are compiled from source. **The compiled artifacts are cached to speed up subsequent builds.**

We vendor the sources for Lua, LuaRocks, & OpenResty/Nginx and compile them with a writable `/app/.heroku` prefix. Attempts to bootstrap Kong on Heroku using existing [Lua](https://github.com/leafo/heroku-buildpack-lua) & [apt](https://github.com/heroku/heroku-buildpack-apt) buildpacks failed due to their compile-time prefixes of `/usr/local` which is read-only in a dyno.

OpenSSL 1.0.2 (required by OpenResty) is also compiled from source, as the versions included in the Cedar 14 stack & apt packages for Ubuntu/Trusty are too old.

Kong is installed from a forked source repo that includes [minimal changes for compatibility with the Heroku runtime](https://github.com/Mashape/kong/compare/release/0.8.0...chrisanderton:0.8.0-external-supervisor).


Modification
------------
This buildpack caches its compilation artifacts from the sources in `vendor/`. Changes to the sources in `vendor/` will be detected and the cache ignored.

If you need to trigger a full rebuild without changing the source, use the [Heroku Repo CLI plugin](https://github.com/heroku/heroku-repo) to purge the cache:

```bash
heroku repo:purge_cache
```
