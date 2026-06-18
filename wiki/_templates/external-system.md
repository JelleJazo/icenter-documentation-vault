---
type: external-system
title: ""
status: draft
module: ""
source-paths: []
last-reviewed: ""
tags: [external-system]
created: YYYY-MM-DD
updated: YYYY-MM-DD
---

# <System name>

## What it is

<Vendor / role / where it physically lives if relevant.>

## How iCenter talks to it

- Protocol: <HTTP / SQL / file drop / TCP / OPC / serial / …>
- Auth: <…>
- Direction: <read | write | bidirectional>
- Code: `<Class.Method>` in `<source-path>`

## What iCenter sends

<Data / commands. List concretely. Anything that can move a machine → `#safety-relevant`.>

## What iCenter receives

<Data / events. Include failure shapes.>

## Failure mode

- What happens when it's down?
- Does iCenter retry? Back off? Drop?
- Is there manual recovery?

## Configuration

- Connection string / endpoint in `<config file>` key `<key>`

## Safety classification

- [ ] iCenter can drive this system → `#safety-relevant`
- [ ] Failure can halt production?

## Open questions

- <…>
