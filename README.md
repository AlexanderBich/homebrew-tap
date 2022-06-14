# homebrew-tap

This is a Homebrew Tap, which features formulas to help pin
Postgres to 13.2 and Postgis to 3.1.1.

Homebrew will pull in the latest version of formulas when they are upgraded,
meaning that users can inadvertently be upgraded to Postgresql 14. The
postgresql.rb formula here ensures that 13.2 is installed, and the postgis.rb
formulate ensures that 3.1.1 is installed.

## Installing Postgres 13.2 and Postgis 3.1.1

First ensure you have upgraded to the latest homebrew:

```sh
brew update
```

It's recommended to go ahead and clear any existing Postgres and Postgis
installations:

```sh
brew uninstall postgis --force
brew uninstall postgresql --force
brew cleanup
brew services stop postgresql
```

After this, install icu4c 6.9.1:

```sh
# edit icu4c configuration
brew edit icu4c # and replace the content with the content from icu4c.rb from this repo

# later reinstall icu4c (it may require uninstalling all installed packages that got stuck in make install)
brew reinstall icu4c
 
# the code was taken from https://stackoverflow.com/a/54340076/8406319
```

After this, install AlexanderBich's custom brew tap:

```sh
brew tap alexanderbich/homebrew-tap
```

Then install the latest version of Postgres and unlink it. This is done first so that
we can explicitly switch to an earlier version before installing Postgis:

```sh
brew install postgresql
brew unlink postgresql
```

The previous install of postgresql runs `initdb`, which creates database structures incompatible with 13.2. This needs to be removed with:

```sh
rm -rf /usr/local/var/postgres 

# for apple m1 it's
rm -rd /opt/homebrew/var/postgres
```

Now install Postgres from this tap with:

```sh
brew install alexanderbich/tap/postgresql
```

Now you will have both 13.2 and the latest version of Postgres installed.
Switch to 13.2 with:

```sh
brew link alexanderbich/tap/postgresql # if it's not already linked
```

Postgis 3.1.1 can be installed with:

```sh
brew install alexanderbich/tap/postgis
```

Create semilink for the right version of libproj (in theory you can install 19th version manually, but I'm not sure how to do it)

```sh
# apple m1 (please change <YOUR_VERSION> to the actual version you have, you can check it with "ls /opt/homebrew/opt/proj/lib/")
ln -s /opt/homebrew/opt/proj/lib/libproj.<YOUR_VERSION>.dylib /opt/homebrew/opt/proj/lib/libproj.19.dylib

# for apple intel, windows or linux please use /usr/local/opt/proj/lib/, or something elsewhere you store your homebrew packages, then using the analogy above go to "proj/lib" folder and do the same thing there
# source for this solution is taken from here: https://github.com/OSGeo/homebrew-osgeo4mac/issues/174#issuecomment-371333912
```

Try running and accessing Postgres with the following:

```sh
brew services start postgresql
psql postgres  # It should show 13.2 as the version on the prompt
```

After running `psql postgres`, type the following in the prompt to verify your Postgis installation:

```sh
drop extension postgis;  -- This might fail if it wasn't previously installed
create extension postgis;  -- This should pass
select ST_Distance(
  ST_GeometryFromText('POINT(-118.4079 33.9434)', 4326), -- Los Angeles (LAX)
  ST_GeometryFromText('POINT(2.5559 49.0083)', 4326)     -- Paris (CDG)
);  -- This should print a row with 121.898285970107 as a value
```
