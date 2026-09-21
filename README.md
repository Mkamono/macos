test


[tasks.profile]
description = "Select profiles for this machine"
usage = '''
arg "<profiles>" var=#true help="Profiles to enable"
'''
run = '''
#!/usr/bin/env bash
set -euo pipefail

config_dir="$HOME/.config/mise"

eval "profiles=($usage_profiles)"

for profile in "${profiles[@]}"; do
  file="$config_dir/config.$profile.toml"
  if [[ ! -f "$file" ]]; then
    echo "Unknown profile: $profile" >&2
    echo "Available profiles:" >&2
    find "$config_dir" -maxdepth 1 -name 'config.*.toml' \
      -exec basename {} \; |
      sed -E 's/^config\.|\.toml$//g' |
      sort >&2
    exit 1
  fi
done

{
  echo 'auto_env = true'
  printf 'env = ['
  sep=''
  for profile in "${profiles[@]}"; do
    printf '%s"%s"' "$sep" "$profile"
    sep=', '
  done
  echo ']'
} > "$config_dir/miserc.toml"

echo "Enabled profiles: ${profiles[*]}"
'''
