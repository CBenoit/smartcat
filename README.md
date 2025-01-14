<p align="center">
  <a href="https://github.com/efugier/smartcat/discussions">
    <img src="https://img.shields.io/badge/commmunity-discussion-blue?style=flat-square" alt="community discussion">
  </a>
  <a href="https://github.com/efugier/smartcat/actions/workflows/ci.yml">
      <img src="https://github.com/efugier/smartcat/actions/workflows/ci.yml/badge.svg?branch=main" alt="Github Actions CI Build Status">
  </a>
  <a href="https://crates.io/crates/smartcat">
      <img src="https://img.shields.io/crates/v/smartcat.svg?style=flat-square" alt="crates.io">
  </a>
  <br>
</p>

<p align="center">
  <img src="assets/sc_logo.png" width="200">
</p>

# smartcat (sc)

Puts a brain behind `cat`! CLI interface to bring language models in the Unix ecosystem and allow power users to make the most out of llms.

![](assets/workflow.gif)

- [Installation](#installation-)
- [Usage](#usage)
- [A few examples to get started 🐈‍⬛](#a-few-examples-to-get-started-)
  - [Manipulate file and text streams](#manipulate-file-and-text-streams)
  - [Integrating with editors](#integrating-with-editors)
- [Configuration](#configuration)
- [Developping](#developping)

## Installation

With [rust and cargo](https://www.rust-lang.org/tools/install) installed and setup:

```
cargo install smartcat
```

(the binary is named `sc`)

Or download directly the binary compiled for your platform from the [release page](https://github.com/efugier/smartcat/releases).


---

On the first run, `smartcat` will ask you to generate some default configuration files if it cannot find them.
More about that in the [configuration section](#Configuration).

A `default` prompt is needed for `smartcat` to know which api and model to hit.

## Usage

```text
Usage: sc [OPTIONS] [CONFIG_PROMPT]

Arguments:
  [CONFIG_PROMPT]  which prompt in the config to fetch [default: default]

Options:
  -r, --repeat-input
          whether to repeat the input before the output, useful to extend instead of replacing
  -p, --custom-prompt <CUSTOM_PROMPT>
          custom prompt to append before the input
  -s, --system-message <SYSTEM_MESSAGE>
          system "config"  message to send after the prompt and before the first user message
  -c, --context <CONTEXT>
          context string (will be file content if it resolves to an existing file's path) to
          include after the system message and before first user message
  -a, --after-input <AFTER_INPUT>
          suffix to add after the input and the custom prompt
      --api <API>
          overrides which api to hit [possible values: openai]
  -m, --model <MODEL>
          overrides which model (of the api) to use
  -f, --file <FILE>
          skip reading from the input and read this file instead
  -i, --input <INPUT>
          skip reading from input and use that value instead
  -h, --help
          Print help
  -V, --version
          Print version
```

Currently only supporting openai and chatgpt but build to work with multiple ones seemlessly if competitors emerge.

You can use it to **accomplish tasks in the CLI** but **also in your editors** (if they are good unix citizens, i.e. work with shell commands and text streams) to complete, refactor, write tests... anything!

The key to make this work seamlessly is a good default prompt that tells the model to behave like a CLI tool an not write any unwanted text like markdown formatting or explanations.

## A few examples to get started 🐈‍⬛

Ask anything without leaving the confort of your terminal

```
sc -i "sed command to remove trailaing whitespaces at the end of all non-markdown files?"
> sed -i '' 's/[ \t]*$//' *.* !(*.md)
```

```
sc -i "shell script to migrate a repository from pipenv to poetry" >> poetry_mirgation.sh
```

use the `-i` so that it doesn't wait for piped input.

### Manipulate file and text streams

```
cat Cargo.toml | sc -p "write a short poem about the content of the file"

A file named package,
Holds the keys of a software's age.
With a name, version, and edition too,
The content speaks of something new.

Dependencies lie within,
With toml, clap, ureq, and serde in,
The stars denote any version will do,
As long as the features are included, too.

A short poem of the file's content,
A glimpse into the software's intent.
With these keys and dependencies,
A program is born, fulfilling needs.
```

```
sc -f Cargo.toml -p "translate the following file in json" | save Cargo.json
```

```
cat my_stuff.py | \
sc -p "write a parametrized test suite for the following code using pytest" \
-s "output only the code, as a standalone file with the imports. \n" \
-a "" \
> test.py
```

If you find yourself reusing prompts often, you can create a dedicated config entries and it becomes the following:

```
sc write_tests -f my_file.py > test.py
```

see example in the [configuration section](#Configuration).

Skipping input to talk directly to the model (but mind the default prompt)

```
sc empty -i "Do you like trains?"

So if you wonder, do I like the trains of steel and might,
My answer lies in how they're kin to code that runs so right.
Frameworks and libraries, like stations, stand so proud
And programmers, conductors, who make the engines loud.
```

### Integrating with editors

The key for a good integration in editors is a good default prompt (or set of) combined with the `-p` flag for precising the task at hand.
The `-r` flag can be used to decide whether to replace or extend the selection.

#### Vim

Start by selecting some text, then press `:`. You can then pipe the selection content to `smartcat`.

```
:'<,'>!sc -p "replace the versions with wildcards"
```

```
:'<,'>!sc -p "fix the typos in this text"
```

will **replace** the current selection with the same text transformed by the language model.

```
:'<,'>!sc -p "implement the traits FromStr and ToString for this struct" -r
```

```
:'<,'>!sc write_test -r
```

will **append** at the end of the current selection the result of the language model.

...

With some remapping you may have your most reccurrent action attached to few keystrokes e.g. `<leader>wt`!

#### Helix and Kakoune

Same concept, different shortcut, simply press the pipe key to redirect the selection to `smarcat`.

```
pipe:sc write_test -r
```

These are only some ideas to get started, go nuts!

# Configuration

- by default lives at `$HOME/.config/smartcat`
- the directory can be set using the `SMARTCAT_CONFIG_PATH` environement variable
- use `#[<input>]` as the placeholder for input when writing prompts
- the default model is `gpt-4` but I recommend trying the latest ones and see which one works best for you. I currently use `gpt-4-1106-preview`.

Two files are used:

`.api_configs.toml`

```toml
[openai]  # each api has their own config section with api and url
url = "https://api.openai.com/v1/chat/completions"
api_key = "<your_api_key>"
```

`prompts.toml`

```toml
[default]  # a prompt is a section
api = "openai"  # must refer to an entry in the `.api_configs.toml` file
model = "gpt-4-1106-preview"

[[default.messages]]  # then you can list messages
role = "system"
content = """\
You are an extremely skilled programmer with a keen eye for detail and an emphasis on readable code. \
You have been tasked with acting as a smart version of the cat unix program. You take text and a prompt in and write text out. \
For that reason, it is of crucial importance to just write the desired output. Do not under any circumstance write any comment or thought \
as you output will be piped into other programs. Do not write the markdown delimiters for code as well. \
Sometimes you will be asked to implement or extend some input code. Same thing goes here, write only what was asked because what you write will \
be directly added to the user's editor. \
Never ever write ``` around the code. \
Now let's make something great together!
"""

[empty]  # always nice to have an empty prompt available
api = "openai"
model = "gpt-4-1106-preview"
messages = []

[write_tests]
api = "openai"
model = "gpt-4-1106-preview"

[[write_tests.messages]]
role = "system"
content = """\
You are an extremely skilled programmer with a keen eye for detail and an emphasis on readable code. \
You have been tasked with acting as a smart version of the cat unix program. You take text and a prompt in and write text out. \
For that reason, it is of crucial importance to just write the desired output. Do not under any circumstance write any comment or thought \
as you output will be piped into other programs. Do not write the markdown delimiters for code as well. \
Sometimes you will be asked to implement or extend some input code. Same thing goes here, write only what was asked because what you write will \
be directly added to the user's editor. \
Never ever write ``` around the code. \
Now let's make something great together!
"""

[[write_tests.messages]]
role = "user"
# the following placeholder string #[<input>] will be replaced by the input
# each message seeks it and replaces it
content ='''Write tests using pytest for the following code. Parametrize it if appropriate.

#[<input>]
'''
```

see [the config setup file](./src/config.rs) for more details.

## Developping

Some tests rely on environement variables and don't behave well with multi-threading so make sure to test with

```
cargo test -- --test-threads=1
```

### State of the project

Smartcat has reached an acceptable feature set. The focus is now on upgrading the codebase quality as I hadn't really touched rust since 2019 and it shows.

#### TODO

- [ ] make it available on homebrew

#### Ideas:

- interactive mode to have conversations and make the model iterate on the last answer
- fetch more context from the codebase
