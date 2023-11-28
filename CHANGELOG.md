# Changelog
All notable changes to this project will be documented in this file.

## 0.3.0 - 28 November 2023
### Added
- Gave this project an official name and its own repo: Interrogatorade 
- The keys in `middleman.json` are now regex patterns, e.g. `"handshake|shake":{"rename":"shaking_hands","confidence":0.75}` will replace both `handshake` and `shake` with `shaking_hands` and force their confidence values to 0.75
- New file `middleman_example.json`

### Changed
- The `name` setting has been renamed to `rename`
- Cap dimensions of image object at 2000x2000 to prevent OOM errors
- Adjusted console log outputs
- Applied yapf lint formatting

### Fixed
- Basic tag name conversion for compatibility with Unprompted variables, more work needed here

## 0.2.0 - 22 November 2023
### Added
- New setting `excludes`: If this tag passes the threshold check, you can exclude a list of other tags from being returned to the client (the opposite of `aliases`)
- The image object is now passed to Unprompted which allows you to make tag assessments based on image properties such as aesthetic score, dimensions, etc.
- All interrogator tags are now passed to Unprompted as user variables
- Unprompted variables beginning with `tag_` will be passed to the client as additional tags (undescores in the variable names will be replaced with spaces)
- Similarly, you can set `tag_something` to 0 to remove the `something` tag from the list of tags that will be returned to the client
- Print warnings about invalid tags in the blacklist, whitelist, or debug list

### Changed
- `unprompted_init.txt` renamed to `unprompted_init_example.txt`

### Fixed
- Fixed error that would occur if Unprompted was not enabled
- Additional fixes for blacklist and whitelist

## 0.1.0 - 21 November 2023
### Added
- New special key `_INIT`: An arbitrary string that is processed by Unprompted at the start of interrogation, useful for setting up variables or custom functions, etc.
- New special key `_WHITELIST_TAGS`: If present, only tags in this list will be returned to the client, useful if you want to evaluate a batch of images for specific tags
- New setting `_DEBUG_TAGS`: Takes a list of tag names to print diagnostic info about in the console
- New setting `debug`: Prints diagnostic info about the tag to the console
- New setting `alias_multiplier`: Adjusts the confidence value of each alias, thereby interspersing synonymous tags throughout the list as opposed to stacking them next to each other
- New file `unprompted_init.txt`: A template for housing useful functions which you can include with `"_INIT":"[call 'full/path/to/unprompted_init.txt']"`
- New init function `multiply`: Context-aware shorthand for multiplication, e.g. `"ring":{"confidence":"[call multiply by=0.25]"}` will multiply the confidence value of the `ring` tag by 0.25
- New init function `bounds`: Constrain the context variable to a range, e.g `"ring":{"confidence":"[call bounds min=0.7 max=0.95]"}` will ensure that the confidence value of the `ring` tag is at least 0.7 and no greater than 0.95

### Changed
- Moved Unprompted cleanup routine to the end of the interrogation cycle
- Renamed `confidence` to `pre`
- Renamed `post_confidence` to `post`
- Renamed `_BANNED_TAGS` to `_BLACKLIST_TAGS`

## 0.0.1 - 20 November 2023
### About
This PR introduces a number of improvements for the Tagger's RPC service, mostly notably the Middleman postprocessor. To use the postprocessor, create `middleman.json` in your `interrogator_rpc` directory.

The Middleman JSON key is a tag name, and the value is a dictionary of settings. For example, `"handshake":{"name":"shaking_hands","confidence":0.75}` will replace the `handshake` tag with `shaking_hands` and force its confidence value to 0.75 (which can be used to influence its placement on the tag list.)

Here is a complete list of the available settings:

- `name`: The new name of the tag that will be returned to the client
- `confidence`: The new confidence value of the tag, applied before the threshold check
- `post_confidence`: The confidence value that will be returned to the client if the original confidence passes the threshold check
- `threshold`: The threshold value to compare the tag's confidence value against, useful if the sensitivity of certain tags doesn't quite match the global threshold value
- `ban`: If `true`, the tag will be removed from the list of tags that will be returned to the client
- `aliases`: Additional tags that will be returned to the client

Additionally, a couple special keys are supported:

- `_BANNED_TAGS` takes a list of tag names to prohibit, e.g. `"_BANNED_TAGS":["handshake","dancing"]`
- `_UNPROMPTED` takes a path to a copy of [Unprompted](https://github.com/ThereforeGames/unprompted), my templating language that transforms plaintext with complex operations, e.g. `"handshake":{"confidence":"[random _min=0.5 _max=0.9]"}`

If using Unprompted, you can reference the original tag properties with `[get]`, e.g. `"handshake":{"confidence":"[max '{get confidence}' 0.75]"}`, which will cap the returned confidence to 0.75.

### Added
- Support for `middleman.json` that enables postprocessing of the returned tags
- `CHANGELOG.md` to track changes
- `run.bat` for Windows users
- Global `DEBUG` flag in for diagnostics in lieu of a proper logger