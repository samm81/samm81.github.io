---
layout: post
title: auto import from an sd card on linux
---

the promise of linux is that if you dream it, and if you have the know-how, you can make it happen. [my father][1] gifted me a camera to take on a road trip through the southwest usa and I've used it ever since, you can see me work my way through my growing backlog on in [instagram][2] if you'd like. for editing I use [darktable][3], an open source alternative to adobe's lightroom.

it's really not a big deal but the small hurdle of having to manually import photos off the sd card raises the activation energy of editing photos just barely too high, and I end up letting photos languish on the camera for weeks. or at least this is what I managed to convince myself of before I went on a six hour coding binge to solve this "problem" instead of editing photos.

I keep my photos on an external drive, and the import process consists of first identifying the most recent photo imported, then copying only new photos from the sd card onto the external drive (see note 1), and finally running an [`exiftool`][4] command to rename & sort the images into folders based on their creation date. the dream is for all this to happen as soon as I plug my sd card in (and get notifications as it happens!)

I'm already using [`udiskie`][5] to auto-mount externally connected devices, including the sd card. my first thought was that maybe `udiskie` exposed a hook to run a command on mount, but I didn't find anything in the docs. I breifly perused the codebase toying with the idea of adding it as a feature, but decided against it.

`udiskie` ultimately is just a front end for [`udisks`][6], which exposes device events on [`dbus`][7]. I played briefly with that & managed to query the bus about connected devices, but then decided to go looking for a higher level way forward.

some online searching suggested writing a [`udev`][8] rule, but that same searching also warned that such scripts can be buggy (sometimes they first before the device is ready to read from) and also that some systems kill the scripts if they run more than a couple of seconds. seemed brittle.

finally I realized that since `udiskie` was already taking care of the auto-mount, all I had to do was watch for the folder it created. so a solution using `inotifywait` was born.

the actual script is much longer (as bash scripts tend to be) but the core of it is

```bash
# auto-import.bash
# THIS SCRIPT WILL NOT WORK AS IS
# for the full script, see appendix

# normally I set strict mode `set -euo pipefail` etc

# use `inotifywait` to sleep until `udiskie` mounts the sd card
# we also limit `event`s to `'create'` so as to only run on mount
[[ \
  $(inotifywait --quiet --event 'create' "$udiskie_mountpoint") \
  == \
  "$udiskie_mountpoint CREATE,ISDIR $udiskie_sd_card_folder_name" \
]] && run

run() {
  import_photos
  update_timestamp
  sort_photos
}

# we've saved the timestamp of previous import to `last-import-timestamp.txt`
# use `exiftool` `-if` option to find photos newer than that timestamp
# then copy them to the external drive
import_photos() {
  for photo in $( \
    exiftool \
      -recurse -quiet -printFormat '$directory/$filename' \
      -if "\$createdate gt \"$(cat last-import-timestamp.txt)\"" \
      "$udiskie_mountpoint/$udiskie_sd_card_folder_name" \
  ); do
    cp "$photo" "$external_drive_folder"
  done
}

# save the timestamp of the last imported photo to `last-import-timestamp.txt`
update_timestamp() {
  exiftool \
      -quiet -printFormat '$createdate' "$external_drive_folder" \
    | sort -r \
    | head -n 1 \
    > last-import-timestamp.txt
}

# use `exiftool` to sort photos into folders by date and rename
sort_photos() {
  exiftool -r \
    '-FileName<DateTimeOriginal' \
    -d '%Y/%m/%d/%Y_%m_%d-%H_%M_%S%%-.c.%%le' \
    "$external_drive_folder"
}
```

when run, this will handle the first time the sd card is inserted, then exit. I want it to be restarted, so this calls for supervision. I run [void linux][9] which uses [`runit`][10] as the `init` system. I already had it configured to start a user level supervisor, so it was quite easy to add a new service which called my auto-import script. that way, every time the script finished, `runit` would just start it right back up again, ready to wait for the next time the sd card was inserted.

the added bonus is notifications. `udiskie` already sends notifications on mounting a device, but I also wanted notifications about the auto-import so that I could ensure it was working as expected.

the trick here is to have _another_ service which follows the logs of the auto-import service & pipes them to `notify-send` (or whatever notification program you'd like).

adding logging to the auto-import service is as simple as adding a new dir, `log/`, to the service folder and adding an executable file, `run`, which contains `exec svlogd -tt "$LOGDIR"`. then the notification service is (approximately) as follows:

```bash
# logfile-notify.bash
# same caveats as above
# THIS SCRIPT WILL NOT WORK AS IS
# for the full script, see appendix

# normally I set strict mode `set -euo pipefail` etc

tail -F -n 0 "${LOGFILE}" | while read -r logline; do
  # I do some parsing to strip the timestamp, which I've excluded here
  notify-send "${SERVICE_NAME}: $logline"
done
```

🎉 tada! 🎉 all in all it took me around 5 hours yesterday evening. but what's amazing about it to me is that if I was on a different system - mac or windows - I don't think this sort of hacking about would have been possible. you'd be dependent on the os to provide this feature, or perhaps you could download some program off the internet that would promise to do it, but if those options don't exist, you're screwed 🤷

on a personal note it was incredibly pleasing to see how far I've come in terms of being a power user - this project was possible because I had accumulated knowledge of `runit`, `exiftool`, and just general scripting ability. this was one of those moments where all the time I spent investing in really getting to know my tools and my ssytem paid off 😄

(1) darktable is staunchly "non-destructive", meaning that it never touches the photos themselves, it does all the editing through sidecar xml files. this means that I could just import all photos from the sd card every time, but copying from an sd card onto an external drive is kind of slow. on the order of seconds. plus it's not ✨ elegant ✨

[1]: https://www.flickr.com/photos/daoist56/
[2]: https://www.instagram.com/athousandcups_/
[3]: https://www.darktable.org/
[4]: https://exiftool.org/
[5]: https://github.com/coldfix/udiskie/
[6]: https://github.com/storaged-project/udisks
[7]: https://www.freedesktop.org/wiki/Software/dbus/
[8]: https://wiki.archlinux.org/title/Udev
[9]: https://voidlinux.org/
[10]: http://smarden.org/runit/

#### appendix

`service/auto-import/run`

```bash
#!/bin/sh
UDISKIE_MOUNT_DIR="/run/media/${USER}"
UDISKIE_NIKON_MOUNT_DIR='NIKON D7100'
TIMESTAMP_FILE="$(dirname "$0")/timestamp-last-import.txt"

udiskie_mount_dir="$UDISKIE_MOUNT_DIR"
udiskie_nikon_mount_dir_name="$UDISKIE_NIKON_MOUNT_DIR"
timestamp_file="$TIMESTAMP_FILE"
dir_fotos="${HOME}/mount/foto-box/camera/import"

exec "$(dirname "$0")/nikon-auto-import.bash" \
  "$udiskie_mount_dir" \
  "$udiskie_nikon_mount_dir_name" \
  "$timestamp_file" \
  "$dir_fotos" \
  2>&1
```

`service/auto-import/nikon-auto-import.bash`

```bash
#!/usr/bin/env bash
# unofficial strict mode
# note bash<=4.3 chokes on empty arrays with set -o nounset
# http://redsymbol.net/articles/unofficial-bash-strict-mode/
# https://sharats.me/posts/shell-script-best-practices/
set -o errexit
set -o nounset
set -o pipefail

IFS=$'\n\t'
shopt -s nullglob globstar

[[ "${TRACE:-0}" == "1" ]] && set -o xtrace

log() {
  echo "${1:?missing log message}"
}

dir_watch_event_create() {
  local created
  created="${1:?missing arg}"
  inotifywait --quiet --event 'create' "$created"
}

dir_wait_event_create() {
  local dir_watch dir_create
  dir_watch="${1:?missing arg}"
  dir_create="${2:?missing arg}"
  dir_created_output="${dir_watch}/ CREATE,ISDIR $dir_create"
  [[ $(dir_watch_event_create "$dir_watch") == "$dir_created_output" ]]
}

fotos_newer_than() {
  local dir_fotos timestamp
  dir_fotos="${1:?missing arg}"
  timestamp="${2:?missing arg}"

  set +o errexit
  exiftool \
    -recurse -quiet -printFormat '$directory/$filename' \
    -if "\$createdate gt \"$timestamp\"" \
    "$dir_fotos"
  exit_status="$?"
  set -o errexit
  [[ "$exit_status" -ne 0 && "$exit_status" -ne 2 ]] \
    && log "\!\! exiftool failed with exit status $exit_status" \
    && exit 1
}

copy_and_update_timestamp() {
  local dir_import_from timestamp_last_import dir_import_to timestamp_file
  dir_import_from="${1:?missing arg}"
  timestamp_last_import="${2:?missing arg}"
  dir_import_to="${3:?missing arg}"
  timestamp_file="${4:?missing arg}"

  for foto in $(fotos_newer_than "$dir_import_from" "$timestamp_last_import"); do
    cp -t "$dir_import_to" "$foto"
  done

  exiftool \
      -quiet -printFormat '$createdate' "$dir_import_to" \
    | sort -r \
    | head -n 1 \
    > "$timestamp_file"
}

exiftool_sort() {
  local dir
  dir="${1:?missing arg}"
  cd "${dir}/.."
  "./exiftool-sort.bash" -quiet "./$(basename "$dir")"
}

fotos_import_sort() {
  local dir_import_to
  dir_import_to="${1:?missing arg}"
  exiftool_sort "$dir_import_to"
  log 'fotos sorted'
}

copy_and_sort() {
  local dir_import_from timestamp_last_import dir_import_to timestamp_file
  dir_import_from="${1:?missing arg}"
  timestamp_last_import="${2:?missing arg}"
  dir_import_to="${3:?missing arg}"
  timestamp_file="${4:?missing arg}"
  copy_and_update_timestamp "$dir_import_from" "$timestamp_last_import" "$dir_import_to" "$timestamp_file"

  fotos_new_num="$(find "$dir_import_to" -type f | wc -l)"
  if [[ "$fotos_new_num" -gt 0 ]]; then
    log "$fotos_new_num fotos imported"
    fotos_import_sort "$dir_import_to"
  else
    log 'no fotos to import'
  fi

}

auto_import() {
  local dir_import_from timestamp_file timestamp_last_import dir_import_to
  dir_import_from="${1:?missing arg}"
  timestamp_file="${2:?missing arg}"
  timestamp_last_import="${3:?missing arg}"
  dir_fotos="${4:?missing arg}"
  if [[ -d "$dir_fotos" ]]; then
    log "$dir_fotos exists, importing directly"
    copy_and_sort "$dir_import_from" "$timestamp_last_import" "$dir_fotos" "$timestamp_file"
  else
    log "$dir_fotos does not exist, aborting"
  fi
}

main() {
  local udiskie_mount_dir udiskie_nikon_mount_dir_name udiskie_nikon_mount_dir
  udiskie_mount_dir="${1:?missing arg}"
  udiskie_nikon_mount_dir_name="${2:?missing arg}"
  udiskie_nikon_mount_dir="${udiskie_mount_dir}/${udiskie_nikon_mount_dir_name}"
  local dir_import_from timestamp_last_import dir_import_to
  dir_import_from="$udiskie_nikon_mount_dir"
  timestamp_file="${3:?missing arg}"
  timestamp_last_import="$(cat "$timestamp_file" 2>/dev/null)"
  dir_fotos="${4:?missing arg}"

  dir_wait_event_create "$udiskie_mount_dir" "$udiskie_nikon_mount_dir_name" \
    && auto_import \
      "$dir_import_from" \
      "$timestamp_file" \
      "$timestamp_last_import" \
      "$dir_fotos"
}

main "$@"

exit 0
```

`service/auto-import/exiftool-sort.bash`

```bash
#!/usr/bin/env bash
# unofficial strict mode
# note bash<=4.3 chokes on empty arrays with set -o nounset
# http://redsymbol.net/articles/unofficial-bash-strict-mode/
# https://sharats.me/posts/shell-script-best-practices/
set -o errexit
set -o nounset
set -o pipefail

IFS=$'\n\t'
shopt -s nullglob globstar

[[ "${TRACE:-0}" == "1" ]] && set -o xtrace

main() {
  directory="${1:?missing required arg 1}"
  exiftool -r \
    '-FileName<DateTimeOriginal' \
    -d '%Y/%m/%d/%Y_%m_%d-%H_%M_%S%%-.c.nef.xmp' \
    -ext 'xmp' \
    "${opts[@]}" \
    "$directory"
  exiftool -r \
    '-FileName<DateTimeOriginal' \
    -d '%Y/%m/%d/%Y_%m_%d-%H_%M_%S%%-.c.%%le' \
    "${opts[@]}" \
    "$directory"
}

opts=()
while true; do
  case "${1:-}" in
    --*|-*) opts+=("$1"); shift 1 ;;
    *) break ;;
  esac
done

main "$@"

exit 0
```

`service/auto-import/log/run`

```bash
#!/bin/sh

SERVICE_NAME='auto-import'
CACHE="${XDG_CACHE_DIR:-$HOME/.cache}"
LOGDIR="$CACHE/service-${SERVICE_NAME}/log"

mkdir -p "$LOGDIR"
exec svlogd -tt "$LOGDIR"
```

note that this is in the `service-wm` directory, because I have two different user level supervisors, one that is launched when `runit` runs (so at boot) and doesn't need access to the window manager, and one that is launched by the window manager, and so has access to information it needs to do things like display notifications.

`service-wm/logfile-notifier/run`

```bash
#!/bin/sh

SERVICE="auto-import"
LOGFILE_DIR="${HOME}/.cache/service-${SERVICE}/log"
LOGFILE="${LOGFILE_DIR}/current"

exec "./logfile-notifyd" "$LOGFILE"
```

`service-wm/logfile-notifier/logfile-notifyd`

```bash
#!/usr/bin/env bash
# unnoficial strict mode, note Bash<=4.3 chokes on empty arrays with set -u
# http://redsymbol.net/articles/unofficial-bash-strict-mode/
set -euo pipefail
IFS=$'\n\t'
shopt -s nullglob globstar

progname-parse() {
  logfile="${1:?must supply logfile}"
  # assumes logfile looks something like '~/.cache/service-name/log/current'
  strip_log_dir="echo ${logfile%/*/*}"
  echo "${strip_log_dir##*/}"
}

logline-strip-timestamp() {
  # sample logline
  # 2022-07-31_23:30:00.92430 starting, please wait
  logline="${1:?must supply logline}"
  cut -d ' ' -f 2- <(echo "$logline")
}

notify() {
  msg="${1:?must supply message}"
  progname="${2:?must supply program name}"
  notify-send "${progname}: $msg"
}

main() {
  logfile="${1:?must supply logfile}"
  progname="${2:-$(progname-parse "$logfile")}"

  tail -F -n 0 "${logfile}" | while read -r logline; do
    notify "$(logline-strip-timestamp "$logline")" "$progname"
  done
}

main "$@"
```
