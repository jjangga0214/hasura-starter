# ![hasura-starter](./asset/hasura-starter.svg)

- A boilerplate for `docker-compose` and `migrations`, setting development environment.
- A cheatsheet for Hasura cli.
- A guide for beginners.

[![license](https://img.shields.io/badge/license-MIT-ff4081.svg?style=flat-square&labelColor=black)](./LICENSE)
[![Hasura v1.0.0-beta.10](https://img.shields.io/badge/Hasura-v1.0.0--beta.10-18ffff.svg?style=flat-square&labelColor=black)](./docker-compose.yml)
![pr welcome](https://img.shields.io/badge/PRs-welcome-09FF33.svg?style=flat-square&labelColor=black)

## Before Getting Started

Clone this repo.

Copy and edit **.env.example** file as you want.

```bash
cp .env.example .env
```

The file uses **Variable Expansion** format to reduce duplication, which [`dotenv-expand`](https://github.com/motdotla/dotenv-expand) supports. `docker-compose` doesn't officially mention if it supports the format. It simply says it expects `VAR=VAL`([docs](https://docs.docker.com/compose/env-file/)). However, it seems `docker-compose` does work with the format. If not, you can use a tool like [`dotenv-cli`](https://github.com/entropitor/dotenv-cli) for populating "expanded" values. (e.g. `dotenv -- docker-compose up -d`)

Note that you should uncomment `HASURA_GRAPHQL_ADMIN_SECRET` from [docker-compose.yml](docker-compose.yml) and configure it from env file, if you want to enable authentication (Refer to [docs](https://docs.hasura.io/1.0/graphql/manual/auth/authentication/index.html) for more detail).

Default ports are `8080` for Hasura, `5432` for Postgres. Before you change them, consider reading [Networking](#Networking) section.

## Getting Started

Run Postgres and Hasura.

```bash
docker-compose up -d
# if you name the env file differently,
# you can set "env_file" field for docker-compose (https://docs.docker.com/compose/environment-variables)
# or use a tool like "dotenv-cli" (https://github.com/entropitor/dotenv-cli)
dotenv -e .env.dev -- docker-compose up -d
```

Stop/Debug/Clear Postgres and Hasura.

```bash
# stop hasura and postgres
docker-compose down
# print logs on the terminal. Good for debugging.
docker-compose up
# stop hasura and postgres, and remove docker volumes as well
docker-compose down --volumes
```

## Cli Cheatsheet

Here are `hasura` CLI cheatsheets for easy copy-and-paste. If you haven't used Cli, first be familar with [migrations](https://deploy-preview-3042--hasura-docs.netlify.com/graphql/manual/migrations/existing-database.html) concept.

There are more available commands. Refer to [docs](https://docs.hasura.io/1.0/graphql/manual/hasura-cli/index.html) or `hasura <COMMAND> --help` for more detail.

`--project` option specifies a directory hasura cli should target. If you go into the directory(`cd hasura`), you don't need to specify it.

If you enabled `HASURA_GRAPHQL_ADMIN_SECRET`, then you should provide the env var (or the option `--admin-secret` <ADMIN_SECRET>) to the cli.
For convenience of development, just populate env file to your shell (maybe with a shell plugin like [`dotenv`](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/dotenv)), or use a tool like [`dotenv-cli`](https://github.com/entropitor/dotenv-cli).

(Comments are my personal shorthand notation, which are not executable.)

```bash
# hasura:_
hasura --project hasura
# hasura:console => open web console to generate migration files
hasura --project hasura console
# hasura:migrate:_
hasura --project hasura migrate
# hasura:status => show migration status
hasura --project hasura migrate status
# hasura:pull => pull migration from Hasura. Generally only used once (as a first migration) per a project.
hasura --project hasura migrate create --from-server <NAME_OF_SINGLE_MIGRATION>
# hasura:push => push(applies) all applicable migration files to Hasura
hasura --project hasura migrate apply
# hasura:push:count => push a given number of migration(s) to Hasura
hasura --project hasura migrate apply --up <NUMBER>
# hasura:push:version => push a specific version of migration to Hasura
hasura --project hasura migrate apply --version <VERSION>
# => mark an migration is applied, without actual execution. Check out "Squash" section under "Migrations".
hasura --project hasura migrate apply --skip-execution --version <VERSION>
# hasura:rollback:count => roll back a given number of migration(s) from Hasura
hasura --project hasura migrate apply --down <NUMBER>
# hasura:rollback:version => roll back a specific version of migration from Hasura
hasura --project hasura migrate apply --type down --version <VERSION>
```

### To another stage (e.g. production)

Use `--endpoint` option for remote Hasura. The option overrides the default endpoint specified in [hasura/config.yaml](hasura/config.yaml) (So you don't have to specify it when dealing with local Hasura).

For practice, I recommend run migration commands against both local and remote Hasura. You can deploy an instance of Hasura by one(few)-click and free on Heroku. Refer to this very simple [Heroku deployment guide](https://docs.hasura.io/1.0/graphql/manual/getting-started/heroku-simple.html).

```sh
# e.g. HASURA_ENDPOINT=http://another-graphql-instance.herokuapp.com
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate create --from-server <NAME_OF_SINGLE_MIGRATION>
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate apply
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate apply --up <NUMBER>
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate apply --version <VERSION>
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate apply --down <NUMBER>
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate apply --type down --version <VERSION>
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate status
```

### Examples

```bash
hasura:pull 'init'
hasura:push:count 2
hasura:push:version 1550925483858
hasura:rollback:count 2
hasura:rollback:version 1550925483858
```

## [Migrations](https://docs.hasura.io/1.0/graphql/manual/migrations/manage-migrations.html)

You don't need to execute `hasura init` by yourself, as migrations directory([/hasura/migrations](/hasura/migrations)) and the config file([/hasura/config.yaml](/hasura/config.yaml)) are already here.

This projects assumes `migrations`(,which includes `metadata`) handled by Hasura. But you can choose another migration tool over Hasura. If so, you only need Hasura to handle `metadata`. For that case, refer to [docs](https://docs.hasura.io/1.0/graphql/manual/migrations/manage-metadata.html) for more detail.

### [cli-migrations](https://docs.hasura.io/1.0/graphql/manual/migrations/auto-apply-migrations.html)

Under [/hasura/migrations](/hasura/migrations), there is `1573631107183_init`. It's a sample migration which creates `User` table and make Hasura [`track`](https://docs.hasura.io/1.0/graphql/manual/schema/using-existing-database.html#step-1-track-tables-views) it. By given configuration, not plain `hasura/graphql-engine:<version>`, but `hasura/graphql-engine:<version>.cli-migrations` is used as docker image. The image automatically applies migrations when Hasura starts. Remove `1573631107183_init` for your migrations.

### Squash

You can "squash" multiple migrations into one by taking [several steps](https://blog.hasura.io/resetting-hasura-migrations/).
But there has been issues(e.g. [#2724](https://github.com/hasura/graphql-engine/issues/2724)) about inconvenience.
To make it simple, community members developed a tool like [hasura-squasher](https://github.com/domasx2/hasura-squasher) and [manual workflow](https://github.com/hasura/graphql-engine/issues/2724#issuecomment-524547524).
Finally, a new command `hasura migrate squash` is introduced with a new migration structure on `v1.0.0-beta.9`.
As of writing(`v1.0.0-beta.10`), the command is still in preview(not stable). For more detail, check out the [changelog](https://github.com/hasura/graphql-engine/releases/tag/v1.0.0-beta.9).

## Networking

### Sync Hasura port

By given configuration, `8080` port is given to Hasura. To change it, you should sync the port number specified in [hasura/config.yaml](hasura/config.yaml) and `HASURA_ENDPOINT_PORT` in your env file.

### Host networking

By given configuration, Hasura and Postgres communicates each other by docker's [host mode networking](https://docs.docker.com/network/host).
No matter how you configure `DB_ENDPOINT_PORT` from env file, Postgres is exposed to Hasura by `5432` port, the default value of Postgres image.
That's why `5432` is hard-coded at `HASURA_GRAPHQL_DATABASE_URL` in [docker-compose.yml](docker-compose.yml). For the same reason, service name `postgres` is used rather than `localhost` on `HASURA_GRAPHQL_DATABASE_URL`.

```yml
environment:
  HASURA_GRAPHQL_DATABASE_URL: "postgres://${POSTGRES_USERNAME}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DATABASE}"
```

### Postgres exposure

By given configuration, Postgres is exposed to the host(your OS) by the port of `DB_ENDPOINT_PORT` from env file. This is useful on development as you can directly access to Postgres without Hasura. However, if you want to use [docker-compose.yml](docker-compose.yml) on production (e.g. `docker swarm` for `docker-compose`), you might reconsider how you should configure it.

## Seed Data

Automatically seeding sample data can be very useful on local development workflow. Currently, Hasura doesn't support the "seed" feature (or some more generic feature that could be easily used). As a workaround, you can manually put a sql file with `INSERT` statements into migrations directory. To ensure seeding is executed AFTER Postgres schema is fully migrated, you can manually name it with very high timestamp value. However, as generally seed is only for development environment, _"seeding as migration"_ approach makes migration inconvenient, or even dangerous (e.g. You should be careful to migration for production). There's [an issue and discussion](https://github.com/hasura/graphql-engine/issues/2431#issuecomment-520459657) about this, and I personally expect this problem to be officially solved soon.

As an another workaround, you can write a little program that calls GraphQL mutations or executes `INSERT` sql statements.

For example, if you use node.js,

```bash
docker-compose up && node seed.js
```

Of course, the program should watches (e.g. continuous health check polling) Hasura endpoint to make sure migrations are completed before inputting data.

## Custom Business Logic

Custom logics are to be integrated with Hasura-generated GraphQL, as a single endpoint. They are also useful when Hasura's authorization system can't cover your complex requirements for specific tasks. There are various ways depending on your needs.

- [Postgres View](https://docs.hasura.io/1.0/graphql/manual/queries/derived-data.html): You can expose computed/derived data as GraphQL query/subscription.
- [Postgres Function](https://docs.hasura.io/1.0/graphql/manual/queries/custom-functions.html): You can expose custom GraphQL query/subscription. (Currently not for mutation)
- [Remote Schema](https://docs.hasura.io/1.0/graphql/manual/remote-schemas/index.html): You can develop your own GraphQL server (or external GraphQL API), and integrate it with Hasura (Similar to schema stiching or Apollo Federation).
- [Remote Join](https://blog.hasura.io/remote-joins-a-graphql-api-to-join-database-and-other-data-sources/)(WIP. Comming Soon): You can precisely "join" (as like SQL join) external GraphQL for the schema you exactly want.
- [Actions](https://deploy-preview-3042--hasura-docs.netlify.com/graphql/manual/actions/index.html)(WIP. Comming Soon): You can define custom mutation in SDL by using types Hasura generates, and write "Action handler", which can be 'HTTP handler', 'Postgres functions' or 'Postgres PLV8'. It can be async as well (subscription/query for the result is auto generated by Hasura.)

## Event Trigger

When an event(e.g. `INSERT`, `UPDATE`, `DELETE`) is occurred on a Postgres table, Hasura can invoke webhooks. Refer to [docs](https://deploy-preview-3042--hasura-docs.netlify.com/graphql/manual/event-triggers/index.html) for more detail.

## 3Factor App

It's somewhat radical event-driven, async, reactive architecture pattern (Redux-like backend?!). The concept itself is independant from Hasura, however Hasura can be used for the architecture implementation. Refer to [3factor.app](https://3factor.app/) or an [example](https://github.com/hasura/3factor-example/).

## License

[MIT License](./LICENSE). Copyright &copy; 2019, Gil B. Chan <[bnbcmindnpass@gmail.com](mailto:bnbcmindnpass@gmail.com)> ([@jjangga0214](https://github.com/jjangga0214))
