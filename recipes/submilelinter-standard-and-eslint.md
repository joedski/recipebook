SublimeLinter: Having both eslint and contrib-standard Installed
======

This is here just because I so rarely peek at the SublimeLinter settings docs.
I usually use a homespun style centered on readable code in the absence of syntax hilighting,
but many projects use [standard](http://standardjs.com), so I want to tell SublimeLinter to use
SublimeLinter-contrib-standard rather than SublimeLinter-contrib-eslint in such cases,
doing so in a Sublime Text Project file instead of polluting their repo with ST artifacts.

Literally all it does is disable `eslint` and enable `standard`.

In the `.sublime-project` file:

```json
{
  "folders":
  [
    "..."
  ],
  "SublimeLinter":
  {
    "linters":
    {
      "eslint": {
        "@disable": true
      },
      "standard": {
        "@disable": false
      }
    }
  }
}
```
