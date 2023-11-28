# 🐊 Interrogatorade

This is a mod for the [BooruDatasetTagManager](https://github.com/starik222/BooruDatasetTagManager)'s autocaption service that allows you to modify the returned tags.

## 📦 Features

- [x] Rename tags
- [x] Whitelist and blacklist tags
- [x] Adjust confidence values
- [x] Modify threshold requirements on a per-tag basis
- [x] Add tag aliases or exclusions
- [x] Apply regex patterns to set up rules for multiple tags at once
- [x] Optionally integrates with [Unprompted](https://github.com/ThereforeGames/unprompted) for performing complex operations on tags
- [x] Various improvements to the RPC service including image sanity checks and debug messages

## 🔧 Installation

Simply clone or download this repository into the `interrogator_rpc` directory of the BooruDatasetTagManager. Launch using the included `run_interrogatorade.bat` file.

## 📚 Usage

### `middleman.json`

The `middleman.json` file is where you define your tag rules. The JSON key is the name of the tag you want to modify, and the value is a dictionary of settings. For example, `"handshake":{"rename":"shaking_hands","confidence":0.75}` will replace the `handshake` tag with `shaking_hands` and force its confidence value to 0.75 (which in turn influences its placement on the tag list).

You can also supply a regex pattern as the key, which will apply the settings to all tags that match your pattern. For example, `"^.*_hair$":{"rename":"hair"}` will rename all tags that end with `_hair` to simply `hair`.

Here is a complete list of the available JSON settings:

- `rename`: The new name of the tag that will be returned to the client
- `pre`: The new confidence value of the tag, applied before the threshold check
- `post`: The confidence value that will be returned to the client if the original confidence passes the threshold check
- `threshold`: The threshold value to compare the tag's confidence value against, useful if the sensitivity of certain tags doesn't quite match your global threshold value
- `ban`: If `true`, the tag will be removed from the list of tags that will be returned to the client
- `aliases`: If this tag passes the threshold check, you can send a list of additional tags to the client
- `excludes`: If this tag passes the threshold check, this excludes a list of other tags from being returned to the client

Additionally, several special keys are supported:

- `_BLACKLIST_TAGS`: Takes a list of tag names to prohibit, e.g. `"_BLACKLIST_TAGS":["handshake","dancing"]`
- `_WHITELIST_TAGS`: Takes a list of allowed tag names; all other tag names will be prohibited
- `_DEBUG_TAGS`: Takes a list of tags to print information about in the console
- `_UNPROMPTED`: (See below)
- `_INIT`: (See below)

### Unprompted

For complex tag operations, you can enable integration with [Unprompted](https://github.com/ThereforeGames/unprompted), a plaintext templating language I wrote for Stable Diffusion.

To do so, simply set the `_UNPROMPTED` key of `middleman.json` to the path of your Unprompted installation, e.g. `"_UNPROMPTED":"C:/programs/automatic-stable-diffusion-webui/extensions/_unprompted"`.

If this is your first time using Unprompted, you should also install the modules listed in its `requirements.txt`.

Now you can take advantage of shortcodes in your `middleman.json` file. For example, `"handshake":{"post":"[random _min=0.5 _max=0.9]"}` will set the confidence value of the `handshake` tag to a random decimal number between 0.5 and 0.9.

Here are some of the other things you can do with Unprompted:

- You can `[get]` the current tag properties. For example, `"handshake":{"post":"[max '{get confidence}' 0.75]"}` will cap the returned confidence of `handshake` to 0.75. Specifically, you can access `tag`, `confidence`, and `threshold` within the value of any JSON key.
- All tag names are automatically converted to variables. For example, `"handshake":{"post":"[if 'two_people >= 0.5']0.9[/if]"}` will set the confidence value of `handshake` to 0.9 if the `two_people` tag has a confidence of 0.5 or greater.
- You can create new tags with `[sets]` by using the `tag_` prefix. For example, `"handshake":{"post":"[sets tag_custom_text=0.9]"}` will create a new tag called `custom_text` with a confidence of 0.9.
- You can use the `_INIT` key to run arbitrary code at the start of interrogation. For example, `"_INIT":"[call 'C:/some/path/unprompted_init]"` will execute the `unprompted_init.txt` file in the specified directory. This is useful for setting up variables or custom functions.