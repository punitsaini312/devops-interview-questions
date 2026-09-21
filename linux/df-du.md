df told you a filesystem is full. du will tell you which folder is at fault, if you point it at the right place.

The top-level scan
Start at the mount that's full and drill down one level at a time:

bash

sudo du -h --max-depth=1 /var 2>/dev/null | sort -h
Three flags earning their keep:

--max-depth=1 limits recursion to one level, so you get a line per direct subfolder instead of every file.
sort -h sorts by human-readable size, smallest to largest.
2>/dev/null throws away the "permission denied" noise from folders you can't read.
The biggest folder sits at the bottom. Repeat inside that folder:

bash

sudo du -h --max-depth=1 /var/log 2>/dev/null | sort -h
Two or three iterations and you've found the guilty folder or file.

ncdu: an interactive alternative
ncdu is a scrollable, interactive version of du. It isn't installed on this machine by default. On a box with package access you'd add it first:

bash

sudo apt install -y ncdu
Then point it at a folder:

bash

ncdu /var
Arrow keys drill into folders, and d deletes the selected entry. It's faster than repeated du calls when you're triaging a full disk under pressure, so it's worth installing on any box you'll be called to debug at odd hours. On an offline machine the install won't reach the package servers, so du piped into sort -h stays your reliable fallback.

What NOT to do
Never rm -rf in /var (or anywhere else) based on a hunch. Confirm which files are safe to delete first. Active processes are writing to files right now, and deleting one of those triggers the "df and du disagree" mystery from the last node without actually freeing space.
