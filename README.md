# vscode-cnf

[cnf](https://github.com/canonical/cnf) language support for VS Code: syntax
highlighting through a
[TextMate](https://macromates.com/manual/en/language_grammars) grammar,
comment toggling, bracket matching and indentation. cnf is a configuration
file format focused on human readability and correctness; the grammar follows
the 1.0rc1 specification.

```
service:
    name: "now"       # inline comment
    ports: [80, 443]
    env:
        - "A=1"
        - "B=2"
    script:
        """
        #!/bin/sh
        echo "not today"
        """
```

The grammar is [`syntaxes/cnf.tmLanguage.json`](syntaxes/cnf.tmLanguage.json),
with the scope name `source.cnf`. Other consumers of TextMate grammars
(JetBrains IDEs, Shiki, Linguist) can read it from here as well.

## Usage

Download the `.vsix` of a [release](https://github.com/canonical/vscode-cnf/releases)
and install it with `code --install-extension cnf-<version>.vsix`.

The `.cnf` suffix is also used by unrelated formats such as `my.cnf` and
`openssl.cnf`. Where those are opened too, select the language per file or
with the `files.associations` setting.

## Scopes

| Scope | Syntax |
|---|---|
| `comment.line.number-sign.cnf` | Standalone or inline `#` comment. |
| `meta.path.cnf` | One or more keys joined by `.`, up to the `:`. |
| `support.type.property-name.cnf` | A key. Quoted keys also carry their string scope. |
| `punctuation.accessor.dot.cnf` | The `.` between the keys of a path. |
| `punctuation.separator.key-value.cnf` | The `:` after a path. |
| `string.quoted.double.cnf`, `string.quoted.single.cnf` | `"..."` and `'...'`. |
| `string.quoted.triple.double.cnf`, `string.quoted.triple.single.cnf` | `"""..."""` and `'''...'''` on one line. |
| `string.quoted.triple.double.block.cnf`, `string.quoted.triple.single.block.cnf` | Fenced string blocks. |
| `constant.character.escape.cnf` | An escape of a double-quoted family. |
| `constant.character.escape.newline.cnf` | A `\` that ends a line of a `"""` block. |
| `constant.numeric.integer.{decimal,hexadecimal,octal,binary}.cnf`, `constant.numeric.float.cnf` | Numbers. |
| `constant.language.boolean.{true,false}.cnf`, `constant.language.null.cnf` | `true`, `false`, `null`. |
| `meta.map.cnf`, `meta.list.cnf` | Inline map `{a: 1}` and inline list `[1, 2]`. |
| `punctuation.definition.{map,list}.{begin,end}.cnf`, `punctuation.separator.{map,list}.cnf` | Their brackets and commas. |
| `punctuation.definition.list.item.cnf` | The `-` of a dashed list item. |
| `punctuation.definition.list.fence.cnf` | The `---` of a fenced list. |
| `invalid.illegal.unknown-escape.cnf` | An escape outside of the set, or out of range. |
| `invalid.illegal.cnf` | Text where a key or a value was due. |

## Strictness

A TextMate grammar matches one line at a time and keeps no account of the
indentation, so it highlights rather than validates. It does mark what it can
tell from a line alone:

- unknown escapes, `\u` escapes in the surrogate range or above U+10FFFF;
- numbers the spec rejects, such as `066`, `1.` or `1E5`;
- unquoted words in place of a value, and a `:` that is not glued to its path.

A fenced string block closes only at the same fence behind the same prefix,
so a fence at another indentation, a `---` or a `#` inside of it are content.
Inline values end with their line, which keeps an unclosed quote or bracket
from spilling over the rest of the file.

The layout rules need the indentation or the data and are left unchecked:
mixed or mismatched prefixes, misplaced comments, blank lines between a key
and its fence, duplicate keys. For those see
[tree-sitter-cnf](https://github.com/canonical/tree-sitter-cnf).

## License

MIT, see [LICENSE](LICENSE).

## Development

This project uses [Workshop](https://ubuntu.com/workshop) for its development
environment:

```bash
workshop launch
workshop run -- test
workshop run -- build
```

Open the project in VS Code and press `F5` to start an Extension Development
Host. Without Workshop, `nix develop` or any Node.js 24 with `npm ci` and
`npm test` does the same.

The tests are cnf files annotated for
[vscode-tmgrammar-test](https://github.com/PanAeon/vscode-tmgrammar-test):
a `# ^^^ scope` line asserts the scopes of the text above it, and
`- scope` asserts their absence.

Releases follow [.kb/release.md](.kb/release.md).
