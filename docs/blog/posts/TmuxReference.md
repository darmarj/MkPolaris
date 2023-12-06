---
date: 2022-06-15
authors: [darmarj]
description: >
  TmuxReference
categories:
  - Tmux
---

# Tmux
## 🔗 [Cheat Sheet & Quick Reference](https://tmuxcheatsheet.com/)

### <p style="text-align: center"><span class="rouge">Sessions</span></p>
|            Description         |      Command       |    |       Description            |      Command            |
|            -----------         |      -------       |    |       -----------            |      -------            |
|       **Start a new settion**  |      tmux          |    |    **Show all sessions**     |      tmux ls            |
|                                |      tmux new      |    |                              |  tmux list-sessions     |
|                                |   tmux new-session |    |   **Attach to last session** | tmux a                  |
|                                |      <span class="jade">:new</span>          |    |                              |   tmux at               |
|                                |                    |    |                              |   tmux attach           |
|                                |                    |    |                              |   tmux attach-session  |
| **Start a new session with the name mysession** | tmux new -s mysession | |   **kill/delete session _mysession_**  |   tmux kill-ses -t mysession|
|                                |  <span class="jade">:new -s mysession</span> |    |                              |   tmux kill-session -t mysession  |
| **Attach to a session with the name _mysession_** |  tmux a -t mysession  |    |   **kill/delete all session but _mysession_**  |  tmux kill-session -a -t mysession   |
|                                |  tmux at -t mysession   |    |  **kill/delete all session but the _current_**  |  tmux kill-session -a   |
|                                |  tmux attach -t myssesion  |     |                       |                       |
|                                |  tmux attach-session -t mysession   |        |                                   |
|  **Rename session**            |  Ctrl + b  $      |   |   **Session and Window Preview**  |   Ctrl + b  w        |
|  **Detach from session**       |  Ctrl + b  d      |   |   **Move to previous session**    |   Ctrl + b  (        |
|  **Detach others on the session (Maximize window by detch other clients)**  |  <span class="jade">:attac -d</span>    |   |  **Move to next session**  |  Ctrl + b  )  |


### <p style="text-align: center"><span class="rouge">Windows</span></p>
|       Description         |     Command     |         Description     |     Command         |
|       -----------         |     -------     |         -----------     |     -------         |
| **Start a new session with the name _mysession_** | tmux new -s mysession -n mywindow |  **Next window**  |  Ctrl + b  n  |
| **Create window**         |  Ctrl + b  c    |     **Switc/select window by number**   |     Ctrl + b  0...9     |
| **Rename current window** |  Ctrl + b  ,    |     **Toggle last active window**  |    Ctrl + b  l    |
| **Close current window**  |  Ctrl + b  &    |**Reorder windows, swap window number 2(src) and 1(dst)**| <span class="jade">:swap-window -s 2 -t 1</span> |
| **List windows**          |  Ctrl + b  w    |     **Move current window to the left by one position** | <span class="jade">:swap-window -t -1</span>  |
| **Previous window**       |  Ctrl + b  p    |                       |                        |


### <p style="text-align: center"><span class="rouge">Panes</span></p>
|       Description         |     Command     |         Description     |     Command         |
|       -----------         |     -------     |         -----------     |     -------         |
|   **Toggle last active pane**   |  Ctrl + b  ;    |    **Show pane numbers**    |   Ctrl + b  q       |
|   **Split pane with horizontal layout**  |   Ctrl + b  %    |     **Switch/select pane by number**    |  Ctrl + b  q  0..9  |
|   **Split pane with vertical layout**    |   Ctrl + b  "    |     **Toggle pane zoom**    |   Ctrl + b  z   |
|   **Move te current pane left**       |   Ctrl + b  {   |     **Convert pane into a window**  |   Ctrl + b  !   |
|   **Move the current pane right**     |   Ctrl + b  }   |     **Resize current pane height(holding second key is optional)** |  Ctrl +b  KEY  |
|                                       |                 |             |   Ctrl + b   Ctrl + KEY  |
|   **Switch to pane to the direction** |   Ctrl + b KEY  |     **Resize current pane width(holding scond key is optional)** |  Ctrl + b  KEY  |
|                                       |                 |             |   Ctrl + b   Ctrl + KEY  |
|   **Toggle synchronize-panes(send command to all panes)** |  <span class="jade">:setw synchronize-panes</span>    |     **Close curent pane**  |   Ctrl + b  X   |
|   **Switch to next pane**     |   Ctrl + b  o     |       |       |


### <p style="text-align: center"><span class="rouge">Copy Mode</span></p>
|       Description         |     Command     |         Description     |     Command         |
|       -----------         |     -------     |         -----------     |     -------         |
|   **use vi keys in buffer**   |  <span class="jade">:setw -g mode-keys vi</span>  |  **Search forward**  |   /    |
|   **Enter copy mode**     |   Ctrl +b  [      |   **Search backward**     |   ?   |
|   **Enter copy mode and scroll one page up**  |   Ctrl + b  PgUp      |   **Next keyword occurance**  |       n       |
|   **Quit mode**   |   q   |       **Previous keyword occurance**  |   N   |
|   **Go to top line**  |   g   |   **Start selection**     |   Spacebar    |
|   **Go to bottom line**   |   G   |   **Clear selection**     |   ESC     |
|   **Scroll up**       |   KEY     |   **Copy selection**  |   Enter   |
|   **Scroll down**     |   KEY     |   **Paste contents of buffer_0i**  |   Ctrl + b  ]     |
|   **Move cursor left**    |   h   |   **disaplay buffer_0 contents**  |   <span class="jade">:show-buffer</span>  |
|   **Move cursor down**    |   j   |   **copy entire visible contents of pane to a buffer**  |   <span class="jade">:capture-pane</span>  |
|   **Move cursor up**      |   k   |   **Show all buffers**  |   <span class="jade">:list-buffers</span>  |
|   **Move cursor right**   |   l   |   **Show all buffers and paste selected**  |   <span class="jade">:choose-buffer</span>  |
|   **Move cursor forward one word at a time**  |   w   |   **Save buffer contents to _buf.txt_**   |   <span class="jade">:save-buffer buf.txt</span>
|   **Move cursor backward one word at a time** |   b   |   **delete buffer_1**   |   <span class="jade">:delete-buffer -b 1/span>


### <p style="text-align: center"><span class="rouge">Misc</span></p>
|       Description         |     Command     |         Description     |     Command         |
|       -----------         |     -------     |         -----------     |     -------         |
|   **Enter command mode**  |   Ctrl + b  :   |  **Set OPTION for all windows**  |  <span class="jade">:setw -g OPTION</span>   |
|   **Set OPTION for all sessions**     |   <span class="jade">:set -g OPTION</span>    |   **Enable mouse mode**   |   <span class="jade">:set mouse on</span>     |


### <p style="text-align: center"><span class="rouge">Help</span></p>
|       Description         |     Command     |         Description     |     Command         |
|       -----------         |     -------     |         -----------     |     -------         |
|                           |   tmux list-keys      |   **Show every session, window, pane, etc...**    |   tmux info   |
|   **List key bindings(shortcuts)**    |   Ctrl + b  ?     |       |       |
