{{#include ../../include/header011.md}}

# Kakoune

If you are a Kakoune user and you'd like something to be added to this page,
please file a [GitHub Issue][project::cb::issues].

## Rust Language Support

You can use `kak-lsp` with `rust-analyzer`.

You want to install just the RA server, without the official VSCode plugin.

You can manage it via `rustup`:

```sh
rustup component add rust-analyzer
```

Or you can build/install it yourself from git:

```sh
git clone https://github.com/rust-lang/rust-analyzer
cd rust-analyzer
git checkout release # use the `release` branch instead of `main`
cargo xtask install --server
```

The easiest way to set up `kak-lsp` is using `plug.kak`.

If you don't have `plug.kak`, put the following in `~/.config/kak/kakrc`:

```kak
evaluate-commands %sh{
    plugins="$kak_config/plugins"
    mkdir -p "$plugins"
    [ ! -e "$plugins/plug.kak" ] && \
        git clone -q https://github.com/andreyorst/plug.kak.git "$plugins/plug.kak"
    printf "%s\n" "source '$plugins/plug.kak/rc/plug.kak'"
}
plug "andreyorst/plug.kak" noload
```

And then to set up `kak-lsp` with Rust support:

```kak
plug "kak-lsp/kak-lsp" do %{
    cargo install --force --path .
} config %{
    set global lsp_cmd "kak-lsp -s %val{session}"

    # create a command to let you restart LSP if anything goes wrong / gets glitched
    define-command lsp-restart -docstring 'restart lsp server' %{ lsp-stop; lsp-start }

    # helper command to enable LSP
    define-command -hidden lsp-init %{
        lsp-enable-window
        # preferences:
        set window lsp_auto_highlight_references true
        lsp-auto-signature-help-enable
        # keybind: use "," to get a menu of available LSP commands
        map global normal "," ": enter-user-mode lsp<ret>" -docstring "LSP mode"
    }

    hook global KakEnd .* lsp-exit

    # autoenable LSP when opening Rust files
    hook global WinSetOption filetype=rust %{
        lsp-init
    }
}
# formatting settings for Rust files
hook global BufSetOption filetype=rust %{
    set buffer tabstop 4
    set buffer indentwidth 4
    set buffer formatcmd 'rustfmt'
    set buffer autowrap_column 100
    expandtab
}
```

Put the following in `~/.config/kak-lsp/kak-lsp.toml` to use `rust-analyzer`:

```toml
[server]
# Shut down the `rust-analyzer` process after a period of inactivity
timeout = 900

[language.rust]
filetypes = ["rust"]
roots = ["Cargo.toml"]
command = "rust-analyzer"
settings_section = "rust-analyzer"

[language.rust.settings.rust-analyzer]
# Proc Macro support is important for Bevy projects
procMacro.enable = true
# disable hover actions, can be laggy on complex projects like Bevy
hoverActions.enable = false
# use the data generated by `cargo check`
cargo.loadOutDirsFromCheck = true
```

