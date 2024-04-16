# Sequelize: CLI

- https://github.com/sequelize/cli/tree/v6.4.1

## Where migration table name set?

- :beginner: [Migrations | Sequelize docs](https://sequelize.org/docs/v6/other-topics/migrations/)
	- `migrationStorageTableName`

- `src/commands/migrate.js`
	- `migrate()`
		- `getMigrator()` at `src/core/migrator.js`
			- `const migrator = new Umzug()`
		- :arrow_right_hook: `ensureCurrentMetaSchema(migrator)` at `src/core/migrator.js`
			- `const tableName = migrator.options.storageOptions.tableName;`
		- :arrow_right_hook: `getTableName()` at `src/helpers/umzug-helper.js`
			- `getStorageOption(type + 'StorageTableName')`
			- `storageTableName = {migration: 'SequelizeMeta'`

## How migration generator from template works?

- :zap: `migration:generatate`

- :page_with_curl: `src/commands/migration_generate.js`
	- :arrow_right_hook: `helpers.template.render('migrations/skeleton.js'
		- путь жёстко задан, не изменить, только копировать код
			-
	- :arrow_right_hook: `helpers.path.getMigrationPath(args.name)`
		- :page_with_curl: `src/helpers/template-helper.js`
			- :arrow_right_hook: `helpers.asset.read(path)`
				- `fs.readFileSync().toString()`
			- :arrow_right_hook: `_.template(template)(locals || {});`
			- :arrow_right_hook: `beautify(content, options);`
	- `import Sequelize, { QueryInterface } from 'sequelize-cli/lib/helpers/';`