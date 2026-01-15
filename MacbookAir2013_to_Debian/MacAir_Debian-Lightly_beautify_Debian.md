## Lightly beautify Debian (Trixie 13.x) for a headless/server setup

Goal: keep the system **minimal**, but make SSH sessions nicer with:

- **fastfetch** (system info on login)
    
- **starship** (clean prompt)
    

---

## 1) Install packages

On Debian trixie, both are available via APT:

```bash
sudo apt update
sudo apt install -y fastfetch starship
```

Verify:

```bash
fastfetch --version
starship --version
```

---

## 2) Create a lightweight Starship config

Create the folder and config file:

```bash
mkdir -p ~/.config/starship
cat > ~/.config/starship/starship.toml <<'EOF'
# ~/.config/starship/starship.toml
# Minimal, server-friendly Starship config (good over SSH)

"$schema" = "https://starship.rs/config-schema.json"

add_newline = false
command_timeout = 1000

# Compact prompt layout
format = "$username$hostname$directory$git_branch$git_status$character"

[username]
show_always = false

[hostname]
ssh_only = true
style = "dimmed"
format = "[@$hostname]($style) "

[directory]
truncation_length = 3
truncate_to_repo = true

[git_branch]
format = "[$symbol$branch]($style) "

[git_status]
format = "[$all_status$ahead_behind]($style) "

[character]
success_symbol = "➜ "
error_symbol = "➜ "
EOF
```

---

## 3) Enable fastfetch + starship in `~/.bashrc`

Edit `~/.bashrc`:

```bash
nvim ~/.bashrc
```

Add this near the end:

```bash
# --- fastfetch + starship (interactive shells only) ---
case $- in
  *i*) ;;
  *) return ;;
esac

# fastfetch: show system info on SSH login (recommended for servers)
if [ -n "$SSH_CONNECTION" ] && command -v fastfetch >/dev/null 2>&1; then
  fastfetch
fi

# starship: prompt
export STARSHIP_CONFIG="$HOME/.config/starship/starship.toml"
if command -v starship >/dev/null 2>&1; then
  eval "$(starship init bash)"
fi
```

Apply changes:

```bash
source ~/.bashrc
```

---

## 4) Notes / tweaks

- If you want fastfetch to run **also on local login** (not only SSH), remove the SSH check:
    
    ```bash
    if command -v fastfetch >/dev/null 2>&1; then
      fastfetch
    fi
    ```
    
- If you ever want to temporarily disable fastfetch:
    
    ```bash
    FASTFETCH_DISABLE=1 bash
    ```
    
    (Then wrap your fastfetch line with a check like `[ -z "$FASTFETCH_DISABLE" ] && fastfetch`.)
    

---