# Servire Workspace UI Alignment — v0.8.8

## Purpose

Servire now uses a common outer workspace width for the operational dashboard and all embedded component workspaces, while preserving the dashboard's existing comfortable content density.

## Changes

- Component navigation tabs now use two lines: the Latin component name on the first line and the English responsibility underneath.
- The Servire dashboard outer shell now uses the same 1680 px maximum width as component workspace routes.
- Servire's operational dashboard content remains constrained to 1200 px and centred within that common shell.
- Embedded component iframes continue to use the wider workspace and are not artificially narrowed.

## Rationale

This aligns the Servire header and workspace navigation across routes without compressing information-dense component UIs such as Moderari and Rogare. The distinction between component name and responsibility is also clearer and avoids increasingly long single-line tab labels.
