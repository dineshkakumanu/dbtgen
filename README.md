# .dbtgen


## Contents

- [Installation](#installation)
- [Commands](#commands)
  - [`model`](#model)
  - [`model-properties`](#model-properties)
  - [`source`](#source)
  - [`package`](#package)
  - [`clean`](#clean)


## Installation

The package should be installed when the setting up the virtual environment, but if not, it can be 
installed using the following command:

```
pip install -e .dbtgen
```

## Commands:

The sub-commands / actions available to run with `dbtgen` are:

- `dbtgen model [OPTIONS]`
- `dbtgen model-properties [OPTIONS]`
- `dbtgen source [OPTIONS]`
- `dbtgen package`
- `dbtgen clean`

---

### model

```
  dbtgen model [OPTIONS]
```

Options:
- `-s` (`--select`): The model/schema selected (e.g. `-s staging`)
- `-o` (`--overwrite`): Overwrite existing models in the target folder of the project
- `-r` (`--run`): Create the model files (by default, this is not used)

This command is used to help generate dbt model files (`.sql`) using a parameterised SQL template and model scoped variables defined in YAML. 

The nested folder structure within `./.dbtgen/models/` represents the same structure used in `./models` (the default models directory for the project). When the application runs, it iterates through every sub-directory and checks for two types of files:

#### Template file (`*.sql`)

These files contain parameterised SQL representing the contents of dbt models. Variables can be specified using curly braces (e.g. `{my_variable}`). The variable keys used here must be those defined in the models variable file (see below). The file name of the template file itself can (and should) be parameterised. This will be used to generate the individual file names for the models.

There can be an arbitrary number of templates within a folder.

#### Models variable file (`models.yml`)

This defines the model scoped variables. The values for these variables defined here will be used in place of those found in any template file(s).

_YAML structure_

```yaml
# models.yml

models:
  <model-name>:
    <key>: <value>
```

When the application finds a template file it will, by default, check for a `models.yml` in the same directory. If it can't find one, it will traverse through parent directories and take the first one it encounters. This allows us to use one models variable file to generate multiple different dbt models using many nested templates.

#### Compile mode (default)

By default, the application will execute in compile mode and print the contents of each model file to terminal. You should check these look as expected before executing in `run` mode.

_Example usage_

```shell
dbtgen model
dbtgen model -s ods.saba
```

#### Run mode

By passing in the command line flag `-r` or `--run` will create the models in the projects models directory.

_Example usage_

```shell
dbtgen model -s staging -r
dbtgen model -s staging -r -o
```


---

### model-properties

```
  dbtgen model-properties [OPTIONS]
```

Options:
- `-s` (`--select`): The qualified path of directory containing the models (e.g. `-s staging`)
- `-sc` (`--schema`): The target schema in Snowflake (e.g. `-sc prod_analytics.staging`). Note, for generating accurate recency tests, `prod` data is recommended
- `-uv` (`--use-views`): Whether to use view objects only
- `-ut` (`--use-tables`): Whether to use table objects only
- `-w` (`--warn-only`): Use severity warn only for all recency tests (by default, this will generate both warn and error tests)

This command is used to scrape data from Snowflake and generate a model_properties file.

The structure of the file generated will include:
- model names
- model recency schema tests
- `SYS_` columns and schema tests

Model properties files are generated with file name `.dbtgen__*.yml` and are git ignored by default. This requires the user to check and rename (or replace any other properties file) if they are happy with the contents.

_Example usage_

```shell
dbtgen model-properties -s staging -sc prod_analytics.staging -uv --target prod
```

### TO DO

Include deep merge of existing complex YAML structures, in order to overlay any pre-existing model properties file with the one generated by the application. This will preserve descriptions in the `.yml` file that is created.


---

### source

```
  dbtgen source [OPTIONS]
```

Options:
- `-s` (`--select`): The model/schema selected (e.g. `-s raw.stripe`)
- `-o` (`--overwrite`): Overwrite existing sources file (default=False)
- `-t` (`--target`): The target environment to use to generate the source files (e.g. `-t prod`). Note, by default this is the target name of the default dbt profile (e.g. `dev007`)
- `-p` (`--profile`): The dbt profile to use

Generates a dbt sources file using table/views existing in a selected database and schema

_Example usage_

```shell
dbtgen source -s raw.stripe -t prod
```

---

### package

```
  dbtgen package
```

This command takes all dbt models defined in properties files (`*.yml`) from the dbt project and creates a sources YAML file for each. 

These are generated in: `./.export/<project-name>/sources/`.

These sources files will form part of another "export" dbt project that can be used as a package 
and imported into any dependent project(s).


---

### clean

```
  dbtgen clean
```

This will clean up all temporary files created by `dbtgen`. 

The files which are cleaned by this command are controlled using the `params.CLEAN_PATHS` list of glob strings.
