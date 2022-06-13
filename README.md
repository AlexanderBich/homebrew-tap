# homebrew-tap

This is Clover Health's Homebrew Tap, which features formulas to help pin
Postgres to 13.2 and Postgis to 3.1.1.

Homebrew will pull in the latest version of formulas when they are upgraded,
meaning that users can inadvertently be upgraded to Postgresql 10. The
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

After this, install Clover's custom brew tap:

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

Now you will have both 13.1 and the latest version of Postgres installed.
Switch to 13.1 with:

```sh
brew link postgresql 13.1
```

Postgis 3.1.1 can be installed with:

```sh
brew install alexanderbich/tap/postgis
```

Try running and accessing Postgres with the following:

```sh
brew services start postgresql
psql postgres  # It should show 13.1 as the version on the prompt
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
