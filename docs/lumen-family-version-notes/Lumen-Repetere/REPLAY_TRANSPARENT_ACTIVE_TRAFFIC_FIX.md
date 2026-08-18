# Replay Transparent Active-Traffic Fix

Replay consumes only an explicit supported `\obt replay` command when that command is the terminal user event in the incoming provider payload.

A historical Replay command retained in conversation context is not consumed when a later assistant or tool event is present. Such requests remain transparent and are forwarded to Lumen, including while a replay is active.

Client cancellation of the active start request propagates through the HTTP stream to Lumen. Replay records the run as cancelled and returns to transparent mode.
