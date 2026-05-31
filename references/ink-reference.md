# Ink Reference

Practical reference for building terminal user interfaces with Ink. Covers components, hooks, layout, keyboard interaction, and CLI wiring.

## What Is Ink?

Ink is a React-based library for rendering UI in the terminal. Instead of `<div>` and `<span>`, you use `<Box>` and `<Text>`. Instead of click events, you use keyboard input via `useInput()`.

Ink runs inside any terminal. It does not support mouse clicks by default. All interaction is keyboard-driven.

## Project Bootstrap

```bash
npx create-ink-app my-app
cd my-app
npm install
npm run build
node dist/cli.js
```

## Directory Structure

```text
my-app/
├── package.json
├── tsconfig.json
└── source/
    ├── cli.tsx
    └── app.tsx
```

After `npm run build`, compiled output goes to `dist/`.

## package.json Fields That Matter

```json
{
  "name": "my-app",
  "bin": "dist/cli.js",
  "type": "module"
}
```

- `name` becomes the CLI command name after `npm link`
- `bin` is the file executed as the command
- `type: "module"` enables ESM imports

## CLI Entry Point

```tsx
#!/usr/bin/env node
import React from 'react';
import {render} from 'ink';
import meow from 'meow';
import App from './app.js';

const cli = meow(`
  Usage
    $ my-app

  Options
    --name  Your name

  Examples
    $ my-app --name=Jane
`, {
  importMeta: import.meta,
  flags: {
    name: {type: 'string'},
  },
});

render(<App name={cli.flags.name} />);
```

The shebang tells the OS to run the file with Node. `meow` parses flags. `render()` starts the Ink TUI.

## Core Components

### Box

Layout primitive based on flexbox.

```tsx
import {Box} from 'ink';

<Box flexDirection="row">
  <Box width={20}>Left</Box>
  <Box flexGrow={1}>Right</Box>
</Box>

<Box flexDirection="column">
  <Box>Header</Box>
  <Box>Body</Box>
</Box>

<Box borderStyle="single" paddingX={1}>
  Content
</Box>
```

Useful props:

```tsx
<Box
  flexDirection="row"
  flexGrow={1}
  width={20}
  width="50%"
  height={10}
  padding={1}
  paddingX={1}
  paddingY={1}
  marginTop={1}
  alignItems="center"
  justifyContent="space-between"
  alignSelf="center"
  borderStyle="single"
  borderColor="cyan"
/>
```

### Text

Styled terminal text.

```tsx
import {Text} from 'ink';

<Text bold>Bold text</Text>
<Text italic>Italic text</Text>
<Text underline>Underlined</Text>
<Text strikethrough>Strikethrough</Text>
<Text color="cyan">Cyan text</Text>
<Text color="green" bold>Bold green</Text>
<Text dimColor>Dimmed text</Text>
<Text inverse>Inverse background and foreground</Text>
```

Named colors include `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan`, `white`, `gray`, and `grey`.

### Newline

```tsx
import {Newline} from 'ink';

<Text>Line one</Text>
<Newline />
<Text>Line two after a blank line</Text>
```

### Spacer

```tsx
import {Spacer} from 'ink';

<Box flexDirection="row">
  <Text>Left</Text>
  <Spacer />
  <Text>Right</Text>
</Box>
```

### Static

```tsx
import {Static, Box, Text} from 'ink';

<Static items={logLines}>
  {(line, index) => (
    <Box key={index}>
      <Text>{line}</Text>
    </Box>
  )}
</Static>
```

## Core Hooks

### useInput

```tsx
import {useInput} from 'ink';

useInput((input, key) => {
  if (input === 'q') {}
  if (key.escape) {}
  if (key.return) {}
  if (key.tab) {}
  if (key.upArrow) {}
  if (key.downArrow) {}
  if (key.leftArrow) {}
  if (key.rightArrow) {}
  if (key.backspace) {}
  if (key.delete) {}
  if (key.ctrl && input === 'c') {}
  if (key.meta) {}
  if (key.shift) {}
});
```

Every mounted `useInput` handler receives every key press. Guard panel-specific handlers with focus state.

```tsx
function Sidebar({focused}: {focused: boolean}) {
  useInput((input, key) => {
    if (!focused) {
      return;
    }

    // Handle sidebar input.
  });
}
```

### useApp

```tsx
import {useApp} from 'ink';

const {exit} = useApp();
exit();
exit(new Error('Something went wrong'));
```

### useStdin

```tsx
import {useStdin} from 'ink';

const {stdin, setRawMode, isRawModeSupported} = useStdin();
```

Usually `useInput()` is enough.

### useStdout

```tsx
import {useStdout} from 'ink';

const {stdout} = useStdout();
const width = stdout.columns;
const height = stdout.rows;
```

### useFocus and useFocusManager

```tsx
import {useFocus, useFocusManager} from 'ink';

function Panel() {
  const {isFocused} = useFocus();

  return (
    <Box borderColor={isFocused ? 'cyan' : undefined} borderStyle="single">
      <Text>{isFocused ? 'Focused' : 'Not focused'}</Text>
    </Box>
  );
}

function App() {
  const {focusNext, focusPrevious} = useFocusManager();

  useInput((input, key) => {
    if (key.tab) {
      focusNext();
    }

    if (key.shift && key.tab) {
      focusPrevious();
    }
  });

  return (
    <>
      <Panel />
      <Panel />
    </>
  );
}
```

## Layout Patterns

### Header and Body

```tsx
<Box flexDirection="column">
  <Box borderStyle="single" paddingX={1}>
    <Text bold>Header</Text>
  </Box>
  <Box flexGrow={1}>Body</Box>
</Box>
```

### Sidebar and Content

```tsx
<Box flexDirection="row">
  <Box borderStyle="single" width={20} flexDirection="column">
    Sidebar
  </Box>
  <Box borderStyle="single" flexGrow={1} flexDirection="column">
    Content
  </Box>
</Box>
```

### Popup Pattern

```tsx
{isPopupOpen && (
  <Box
    borderStyle="double"
    borderColor="magenta"
    flexDirection="column"
    paddingX={1}
    marginTop={1}
    width="90%"
    alignSelf="center"
  >
    <Text bold color="magenta">Popup Title</Text>
    <Text>Popup content here.</Text>
    <Text dimColor>Press x to close</Text>
  </Box>
)}
```

## Common Patterns

### Bounded Index

```tsx
if (key.upArrow) {
  setIndex(index => Math.max(0, index - 1));
}

if (key.downArrow) {
  setIndex(index => Math.min(items.length - 1, index + 1));
}
```

### Reset Popup on Selection Change

```tsx
useEffect(() => {
  setIsPopupOpen(false);
}, [selectedIndex]);
```

### Close Popup Before Quit

```tsx
useInput((input, key) => {
  if (key.escape || input === 'q') {
    if (isPopupOpen) {
      setIsPopupOpen(false);
      return;
    }

    exit();
  }
});
```

### Highlight Active Item

```tsx
{items.map((item, index) => (
  <Text key={item} color={index === selectedIndex ? 'cyan' : undefined}>
    {index === selectedIndex ? '>' : '-'} {item}
  </Text>
))}
```

## Testing with ink-testing-library

```tsx
import {render} from 'ink-testing-library';
import test from 'ava';
import chalk from 'chalk';
import App from './source/app.js';

test('renders greeting', testContext => {
  const {lastFrame} = render(<App name="Jane" />);
  testContext.is(lastFrame(), `Hello, ${chalk.green('Jane')}!`);
});

test('uses default name', testContext => {
  const {lastFrame} = render(<App name={undefined} />);
  testContext.true(lastFrame().includes('Stranger'));
});
```

Run tests with `npm test`.

## Checklist

- Scaffold with `npx create-ink-app`
- Set up `cli.tsx` with `meow`
- Build the main `App` component in `app.tsx`
- Use `Box` for layout and `Text` for output
- Add `useInput()` so the app stays alive
- Use `useApp()` for clean exit behavior
- Track selection, focus, and popup state with `useState`
- Guard panel-specific input with a `focused` flag or focus hooks
- Build and test with `npm run build && node dist/cli.js`

## External Reference

- [Ink GitHub repository](https://github.com/vadimdemedes/ink)
