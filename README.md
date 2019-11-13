# hasura-compose-boilerplate

Boilerplate of `docker-compose` and `migration` for Hasura with Postgres.

[![license](https://img.shields.io/badge/license-MIT-ff4081.svg?style=flat-square&labelColor=black)](./LICENSE)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-ffab00.svg?style=flat-square&labelColor=black)](https://conventionalcommits.org)
[![pr welcome](https://img.shields.io/badge/PRs-welcome-09FF33.svg?style=flat-square&labelColor=black)]()

## Getting started

Copy and edit **.env.example** file as you want. Note that the file uses **Variable Expansion** format, which [`dotenv-expand`](https://github.com/motdotla/dotenv-expand) supports. I intentionally wrote with the format to reduce duplication. However, `docker-compose` will not recognize the format by itself. You can use a tool like [dotenv](https://github.com/entropitor/dotenv-cli) for populating those values well.

```bash
cp .env.example .env
```

Run Postgres and Hasura.

```bash
dotenv -- docker-compose up -d
# or if you name the env file differently
dotenv -e .env.dev -- docker-compose up -d
```

```bash
# db:up => runs Hasura and Postgres
dotenv -- docker-compose up -d
# db:up:debug => with printing
dotenv -- docker-compose up
# db:down => stops hasura and postgres
dotenv -- docker-compose down
# db:down:clean => stops hasura and postgres and removes docker volumes
dotenv -- docker-compose down --volumes
```

## Cheetsheets

`--project` option specifiy where hasura director reside

```bash
# hasura:_
hasura --project hasura
# hasura:console => open hasura web console to generate migration files
hasura --project hasura console
# hasura:migrate:_
hasura --project hasura migrate
# hasura:pull => pull migration files from hasura server
hasura --project hasura migrate create --from-server <NAME>
# hasura:status => show migration status
hasura --project hasura migrate status
# hasura:push => push(applies) migration files to hasura server
hasura --project hasura migrate apply
# hasura:push:count => push a given number of migration(s) to hasura server
hasura --project hasura migrate apply --up <NUMBER>
# hasura:push:version => push specific version of migration to hasura server
hasura --project hasura migrate apply --version <VERSION>
# hasura:rollback:count => roll back a given number of migration(s) from hasura server
hasura --project hasura migrate apply --down <NUMBER>
# hasura:rollback:version => roll back a specific version of migration from hasura server
hasura --project hasura migrate apply --type down --version <VERSION>
```

### To another stage (e.g. production)

Use `--endpoint` option for remote Hasura. The option overrides the default endpoint specifiy in [hasura/config.yaml](hasura/config.yaml).

```bash
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate create --from-server
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate apply
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate apply --up
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate apply --version
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate apply --down
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate apply --type down --version
hasura --project hasura --endpoint $HASURA_ENDPOINT migrate status
```

### Examples

```bash
hasura:pull 'init'

hasura:push:count 2

hasura:push:version:prod 1550925483858

hasura:rollback:count 2

hasura:rollback:version 1550925483858
```

## License

[MIT License](./LICENSE). Copyright &copy; 2019, Gil B. Chan <[bnbcmindnpass@gmail.com](mailto:bnbcmindnpass@gmail.com)> ([@jjangga0214](https://github.com/jjangga0214))
