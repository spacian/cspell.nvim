# cspell.nvim

A companion plugin for [nvimtools/none-ls.nvim](https://github.com/nvimtools/none-ls.nvim),
adding support for [cspell] diagnostics and code actions.

## How to setup in lazyvim

Make sure you have `cpsell` installed, and then add the following
to your plugin configuration directory.

```lua
return {
  {
    "nvimtools/none-ls.nvim",
    event = "VeryLazy",
    depends = { "davidmh/cspell.nvim" },
    opts = function(_, opts)
      local cspell = require("cspell")
      opts.sources = opts.sources or {}
      table.insert(
        opts.sources,
        cspell.diagnostics.with({
          diagnostics_postprocess = function(diagnostic)
            diagnostic.severity = vim.diagnostic.severity.HINT
          end,
        })
      )
      table.insert(opts.sources, cspell.code_actions)
    end,
  },
}
```

## Diagnostics

```lua
local cspell = require('cspell')
require("null-ls").setup {
    sources = {
        cspell.diagnostics
    }
}
```

### Defaults

- Filetypes: `{}`
- Method: `diagnostics`
- Command: `cspell`
- Args: dynamically resolved (see [diagnostics source])

## Code Actions

```lua
local cspell = require('cspell')
require("null-ls").setup {
    sources = {
        cspell.diagnostics,
        cspell.code_actions,
    }
}
```

### Defaults

- Filetypes: `{}`
- Method: `code_action`

## Configuration options

All the configuration properties are optional and they're used for the code actions.

But if you define them, make sure to add them to both the diagnostics **and** the code_actions.
We need to do that to start reading and parsing the CSpell configuration asynchronously as soon
as we get the first diagnostic.

```lua
local config = {
  -- The CSpell configuration file can take a few different names this option
  -- lets you specify which name you would like to use when creating a new
  -- config file from within the `Add word to cspell json file` action.
  --
  -- See the currently supported files in https://github.com/davidmh/cspell.nvim/blob/main/lua/cspell/helpers.lua
  config_file_preferred_name = 'cspell.json',

  -- A list of directories that contain additional cspell.json config files or
  -- support the creation of a new config file from a code action
  --
  -- looks for a cspell config in the ~/.config/ directory, or creates a file in the directory
  -- using 'config_file_preferred_name' when a code action for one of the locations is selected
  cspell_config_dirs = { "~/.config/" }

  --- A way to define your own logic to find the CSpell configuration file.
  ---@params directory The same directory defined in the source,
  --             defaulting to vim.loop.cwd()
  ---@return string|nil The path of the json file
  find_json = function(directory)
  end,

  -- Will find and read the cspell config file synchronously, as soon as the
  -- code actions generator gets called.
  --
  -- If you experience UI-blocking during the first run of this code action, try
  -- setting this option to false.
  -- See: https://github.com/davidmh/cspell.nvim/issues/25
  read_config_synchronously = true,

  ---@param cspell string The contents of the CSpell config file
  ---@return table
  decode_json = function(cspell_str)
  end,

  ---@param cspell table A lua table with the CSpell config values
  ---@return string
  encode_json = function(cspell_tbl)
  end,

  ---@param payload UseSuggestionSuccess
  on_use_suggestion = function(payload)
      -- Includes:
      payload.misspelled_word
      payload.suggestion
      payload.cspell_config_path
      payload.generator_params
  end

  ---@param payload AddToJSONSuccess
  on_add_to_json = function(payload)
      -- Includes:
      payload.new_word
      payload.cspell_config_path
      payload.generator_params

      -- For example, you can format the cspell config file after you add a word
      os.execute(
          string.format(
              "jq -S '.words |= sort' %s > %s.tmp && mv %s.tmp %s",
              payload.cspell_config_path,
              payload.cspell_config_path,
              payload.cspell_config_path,
              payload.cspell_config_path
          )
      )
  end

  ---@param payload AddToDictionarySuccess
  on_add_to_dictionary = function(payload)
      -- Includes:
      payload.new_word
      payload.cspell_config_path
      payload.generator_params
      payload.dictionary_path

      -- For example, you can sort the dictionary after adding a word
      os.execute(
          string.format(
              "sort %s -o %s",
              payload.dictionary_path,
              payload.dictionary_path
          )
      )
  end

  --- DEPRECATED
  --- Callback after a successful execution of a code action.
  ---@param cspell_config_file_path string|nil
  ---@param params GeneratorParams
  ---@param action_name 'use_suggestion'|'add_to_json'|'add_to_dictionary'
  on_success = function(cspell_config_file_path, params, action_name)
  end
}

local cspell = require('cspell')
require("null-ls").setup {
    sources = {
        cspell.diagnostics.with({ config = config }),
        cspell.code_actions.with({ config = config }),
    }
}
```

### Notes

- The code action source depends on the diagnostics, so make sure to register it too.

## Tests

The test suite depends on plenary.nvim.

Run `./tests/run.sh` in the root of the project to run the suite or use [neotest]
to run individual tests from within Neovim.

To avoid a dependency on any plugin managers, the test suite will set up its
plugin runtime under the `./tests` directory to always have a plenary version
available.

If you run into plenary-related issues while running the tests, make sure you
have an up-to-date version of the plugin by clearing that cache with
`rm -rf .tests/`.

All tests expect the latest Neovim master.

# TODO

- [ ] Custom configuration examples

# Credits

<!-- cSpell:disable -->

These sources were initially written in jose-elias-alvarez/null-ls.nvim, with
contributions from: [@JA-Bar], [@PumpedSardines], [@Saecki], [@Sloff], [@marianozunino],
[@mtoohey31] and [@yoo].

[null-ls]: https://github.com/jose-elias-alvarez/null-ls.nvim
[cspell]: https://github.com/streetsidesoftware/cspell
[diagnostics source]: https://github.com/davidmh/cspell.nvim/blob/main/lua/cspell/diagnostics/init.lua
[@JA-Bar]: https://github.com/JA-Bar
[@PumpedSardines]: https://github.com/PumpedSardines
[@Saecki]: https://github.com/Saecki
[@Sloff]: https://github.com/Sloff
[@marianozunino]: https://github.com/marianozunino
[@mtoohey31]: https://github.com/mtoohey31
[@yoo]: https://github.com/yoo
[neotest]: https://github.com/nvim-neotest/neotest
