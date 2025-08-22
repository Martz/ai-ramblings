# CLI Best Practices Guide

A comprehensive guide for building high-quality command-line interfaces using modern Node.js tools and libraries.

## Table of Contents
1. [Core Architecture](#core-architecture)
2. [Essential Packages](#essential-packages)
3. [Interactive UI Components](#interactive-ui-components)
4. [Command Design Patterns](#command-design-patterns)
5. [State Management](#state-management)
6. [User Experience Guidelines](#user-experience-guidelines)
7. [Error Handling](#error-handling)
8. [Testing Strategies](#testing-strategies)
9. [Performance Optimisation](#performance-optimisation)
10. [Accessibility](#accessibility)

## Core Architecture

### Dual-Mode Design
Build CLIs that support both interactive and non-interactive modes:

```typescript
// Support both modes seamlessly
if (options.json || options.nonInteractive) {
  // Direct command execution
  return executeCommand(options);
} else {
  // Interactive UI with Ink
  const { waitUntilExit } = render(<InteractiveUI />);
  await waitUntilExit();
}
```

### Separation of Concerns
- **Commands**: Business logic and data processing
- **Components**: UI presentation and interaction
- **Services**: Data access and external integrations
- **Utils**: Shared utilities and helpers

## Essential Packages

### Command Framework

#### Commander.js
The de facto standard for Node.js command-line interfaces:

```typescript
import { Command } from 'commander';

const program = new Command()
  .name('mycli')
  .description('CLI application description')
  .version('1.0.0')
  .option('--verbose', 'Detailed output')
  .option('--json', 'JSON output for scripting');

program
  .command('action <required> [optional]')
  .description('Perform an action')
  .option('-f, --flag', 'Optional flag')
  .action(async (required, optional, options) => {
    // Command logic
  });
```

### Interactive UI

#### Ink (React for CLI)
Build interactive terminal UIs using React components:

```typescript
import React from 'react';
import { render, Box, Text } from 'ink';

const App = () => (
  <Box flexDirection="column">
    <Text color="cyan" bold>Interactive CLI</Text>
    <Text>Building with React in the terminal!</Text>
  </Box>
);

render(<App />);
```

#### Key Ink Packages
- `ink`: Core framework
- `ink-text-input`: Text input fields
- `ink-select-input`: Selection lists
- `ink-spinner`: Loading spinners
- `ink-table`: Table display
- `ink-link`: Clickable links
- `ink-gradient`: Gradient text
- `ink-big-text`: ASCII art text

### User Input

#### Inquirer.js
For complex prompts and user interactions:

```typescript
import inquirer from 'inquirer';

const answers = await inquirer.prompt([
  {
    type: 'input',
    name: 'username',
    message: 'Enter username:',
    validate: input => input.length > 3
  },
  {
    type: 'list',
    name: 'action',
    message: 'Select action:',
    choices: ['Create', 'Update', 'Delete']
  },
  {
    type: 'confirm',
    name: 'confirm',
    message: 'Are you sure?',
    default: false
  }
]);
```

### Display & Formatting

#### Chalk
Terminal string styling:

```typescript
import chalk from 'chalk';

console.log(chalk.green('✓ Success'));
console.log(chalk.red.bold('✗ Error'));
console.log(chalk.yellow.underline('Warning'));
console.log(chalk.rgb(123, 45, 67)('Custom colour'));
```

#### CLI-Table3
Beautiful tables in the terminal:

```typescript
import Table from 'cli-table3';

const table = new Table({
  head: ['Name', 'Status', 'Progress'],
  colWidths: [30, 15, 20],
  style: {
    head: ['cyan'],
    border: ['grey']
  }
});

table.push(
  ['Task 1', chalk.green('Complete'), '100%'],
  ['Task 2', chalk.yellow('In Progress'), '60%']
);

console.log(table.toString());
```

#### Ora
Elegant terminal spinners:

```typescript
import ora from 'ora';

const spinner = ora('Loading...').start();

try {
  await longRunningTask();
  spinner.succeed('Task completed');
} catch (error) {
  spinner.fail('Task failed');
}
```

### Utilities

#### Update Notifier
Notify users of updates:

```typescript
import updateNotifier from 'update-notifier';

const notifier = updateNotifier({ pkg });
notifier.notify();
```

#### Boxen
Create boxes in the terminal:

```typescript
import boxen from 'boxen';

console.log(boxen('Important Message', {
  padding: 1,
  margin: 1,
  borderStyle: 'double',
  borderColor: 'yellow'
}));
```

## Interactive UI Components

### Building Reusable Components

#### Table Selector Component
```typescript
interface TableSelectorProps<T> {
  items: T[];
  onSelect: (item: T) => void;
  columns: Column<T>[];
  pageSize?: number;
}

const TableSelector = <T,>({ items, onSelect, columns, pageSize = 10 }: TableSelectorProps<T>) => {
  const [page, setPage] = useState(0);
  const [selectedIndex, setSelectedIndex] = useState(0);
  
  useInput((input, key) => {
    if (key.upArrow) navigateUp();
    if (key.downArrow) navigateDown();
    if (key.return) onSelect(currentItem);
  });
  
  // Render paginated table
  return <Table data={paginatedData} />;
};
```

### State Management with Context

```typescript
const AppContext = createContext<AppState>();

export const AppProvider: React.FC = ({ children }) => {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <AppContext.Provider value={{ state, dispatch }}>
      {children}
    </AppContext.Provider>
  );
};
```

### Keyboard Navigation

```typescript
useInput((input, key) => {
  // Navigation
  if (key.upArrow) moveUp();
  if (key.downArrow) moveDown();
  if (key.leftArrow) moveLeft();
  if (key.rightArrow) moveRight();
  
  // Actions
  if (key.return) select();
  if (key.escape) back();
  if (input === ' ') toggleSelection();
  
  // Shortcuts
  if (key.ctrl && input === 'c') exit();
  if (key.ctrl && input === 'z') undo();
  if (key.ctrl && input === 'y') redo();
  
  // Search
  if (input === '/') activateSearch();
  if (input === 'n') nextSearchResult();
  if (input === 'N') previousSearchResult();
});
```

## Command Design Patterns

### Progressive Disclosure
Start simple, reveal complexity as needed:

```typescript
// Basic usage
$ mycli create project-name

// Advanced usage with options
$ mycli create project-name --template react --typescript --git

// Expert usage with configuration
$ mycli create project-name --config custom.json
```

### Intelligent Defaults

```typescript
const defaults = {
  port: process.env.PORT || 3000,
  env: process.env.NODE_ENV || 'development',
  config: findConfigFile() || './config.json'
};
```

### Command Aliases

```typescript
program
  .command('status')
  .alias('s')
  .alias('st')
  .description('Show current status');
```

### Subcommands

```typescript
const db = program.command('db');

db.command('migrate')
  .description('Run database migrations');

db.command('seed')
  .description('Seed database with data');

db.command('reset')
  .description('Reset database');
```

## State Management

### Undo/Redo Pattern

```typescript
class StateManager<T> {
  private history: T[] = [];
  private future: T[] = [];
  private current: T;
  
  constructor(initial: T) {
    this.current = initial;
  }
  
  execute(newState: T) {
    this.history.push(this.current);
    this.current = newState;
    this.future = []; // Clear redo stack
  }
  
  undo(): T | null {
    if (this.history.length === 0) return null;
    this.future.push(this.current);
    this.current = this.history.pop()!;
    return this.current;
  }
  
  redo(): T | null {
    if (this.future.length === 0) return null;
    this.history.push(this.current);
    this.current = this.future.pop()!;
    return this.current;
  }
}
```

### Configuration Management

```typescript
class ConfigManager {
  private configPath: string;
  private cache: Config | null = null;
  
  async load(): Promise<Config> {
    if (this.cache) return this.cache;
    
    // Check multiple locations
    const locations = [
      process.env.CONFIG_PATH,
      path.join(os.homedir(), '.config', 'app', 'config.json'),
      path.join(process.cwd(), 'config.json')
    ];
    
    for (const location of locations) {
      if (await exists(location)) {
        this.cache = await readJson(location);
        return this.cache;
      }
    }
    
    return this.createDefault();
  }
  
  async save(config: Config): Promise<void> {
    await writeJson(this.configPath, config);
    this.cache = config;
  }
}
```

## User Experience Guidelines

### Feedback Patterns

#### Success States
```typescript
console.log(chalk.green('✅ Operation completed successfully'));
console.log(chalk.green.bold('\n🎉 All tests passed!'));
```

#### Error States
```typescript
console.log(chalk.red('❌ Operation failed'));
console.log(chalk.red.bold('Error:'), error.message);
console.log(chalk.gray('Run with --debug for more information'));
```

#### Warning States
```typescript
console.log(chalk.yellow('⚠️  Warning: This action cannot be undone'));
console.log(chalk.yellow('📝 Note: Configuration file not found, using defaults'));
```

#### Progress Indication
```typescript
// For determinate progress
const progressBar = new ProgressBar(':bar :percent :etas', {
  total: files.length,
  width: 40,
  complete: '█',
  incomplete: '░'
});

// For indeterminate progress
const spinner = ora({
  text: 'Processing...',
  spinner: 'dots'
}).start();
```

### Help Text

```typescript
program.configureHelp({
  sortSubcommands: true,
  subcommandTerm: (cmd) => cmd.name()
});

program.addHelpText('after', `
Examples:
  $ mycli create my-project
  $ mycli build --watch
  $ mycli test --coverage
  
For more information, visit: https://docs.example.com
`);
```

### Input Validation

```typescript
const validateInput = {
  email: (input: string) => {
    const valid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(input);
    return valid || 'Please enter a valid email address';
  },
  
  required: (input: string) => {
    return input.trim().length > 0 || 'This field is required';
  },
  
  minLength: (min: number) => (input: string) => {
    return input.length >= min || `Minimum ${min} characters required`;
  },
  
  numeric: (input: string) => {
    return !isNaN(Number(input)) || 'Please enter a valid number';
  }
};
```

## Error Handling

### Graceful Degradation

```typescript
class CLIError extends Error {
  constructor(
    message: string,
    public code: string,
    public suggestions?: string[]
  ) {
    super(message);
  }
}

process.on('uncaughtException', (error) => {
  console.error(chalk.red('Unexpected error:'), error.message);
  console.log(chalk.gray('Please report this issue at: github.com/...'));
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error(chalk.red('Unhandled promise rejection:'), reason);
  process.exit(1);
});
```

### User-Friendly Error Messages

```typescript
try {
  await riskyOperation();
} catch (error) {
  if (error.code === 'ENOENT') {
    console.error(chalk.red('File not found:'), error.path);
    console.log(chalk.gray('Make sure the file exists and you have read permissions'));
  } else if (error.code === 'EACCES') {
    console.error(chalk.red('Permission denied:'), error.path);
    console.log(chalk.gray('Try running with elevated permissions'));
  } else {
    console.error(chalk.red('Operation failed:'), error.message);
    if (options.debug) {
      console.error(chalk.gray(error.stack));
    }
  }
  process.exit(1);
}
```

## Testing Strategies

### Unit Testing Commands

```typescript
import { jest } from '@jest/globals';

describe('CLI Commands', () => {
  it('should execute command with options', async () => {
    const mockFn = jest.fn();
    await program.parseAsync(['node', 'cli', 'command', '--option']);
    expect(mockFn).toHaveBeenCalledWith(expect.objectContaining({
      option: true
    }));
  });
});
```

### Integration Testing

```typescript
import { exec } from 'child_process';
import { promisify } from 'util';

const execAsync = promisify(exec);

describe('CLI Integration', () => {
  it('should output correct format', async () => {
    const { stdout } = await execAsync('node cli.js status --json');
    const result = JSON.parse(stdout);
    expect(result).toHaveProperty('status');
  });
});
```

### Mocking Terminal Interactions

```typescript
import { render } from 'ink-testing-library';

describe('Interactive Components', () => {
  it('should handle user input', () => {
    const { stdin, lastFrame } = render(<App />);
    
    stdin.write('\t'); // Tab
    expect(lastFrame()).toContain('Next item selected');
    
    stdin.write('\r'); // Enter
    expect(lastFrame()).toContain('Item confirmed');
  });
});
```

## Performance Optimisation

### Lazy Loading

```typescript
// Only load heavy dependencies when needed
const loadHeavyDependency = async () => {
  const { default: HeavyModule } = await import('heavy-module');
  return new HeavyModule();
};

program
  .command('heavy-operation')
  .action(async () => {
    const module = await loadHeavyDependency();
    await module.execute();
  });
```

### Caching Strategies

```typescript
class DataCache {
  private cache = new Map<string, { data: any; timestamp: number }>();
  private ttl = 5 * 60 * 1000; // 5 minutes
  
  get(key: string): any | null {
    const entry = this.cache.get(key);
    if (!entry) return null;
    
    if (Date.now() - entry.timestamp > this.ttl) {
      this.cache.delete(key);
      return null;
    }
    
    return entry.data;
  }
  
  set(key: string, data: any): void {
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });
  }
}
```

### Streaming Large Data

```typescript
import { Transform } from 'stream';

const processLargeFile = async (filepath: string) => {
  const readStream = fs.createReadStream(filepath);
  const transformStream = new Transform({
    transform(chunk, encoding, callback) {
      // Process chunk
      const processed = processChunk(chunk);
      callback(null, processed);
    }
  });
  
  return pipeline(readStream, transformStream, process.stdout);
};
```

## Accessibility

### Screen Reader Support

```typescript
// Provide alternative text for visual elements
console.log(chalk.green('✓') + ' Task completed');
// Also output: "Success: Task completed" for screen readers

// Use semantic output
console.log('Status: Active');
console.log('Progress: 75%');
console.log('Remaining: 3 items');
```

### Colour-Blind Friendly

```typescript
// Don't rely solely on colour
console.log(chalk.green('✓ Success')); // Symbol + colour
console.log(chalk.red('✗ Failed'));   // Symbol + colour

// Provide alternative indicators
const status = success 
  ? chalk.green('[PASS]')
  : chalk.red('[FAIL]');
```

### Keyboard-Only Navigation

```typescript
// Ensure all functionality is keyboard accessible
useInput((input, key) => {
  // Provide keyboard shortcuts for all actions
  if (key.tab) focusNext();
  if (key.shift && key.tab) focusPrevious();
  if (input === '?') showHelp();
  if (input === 'q') quit();
});
```

## Best Practices Summary

### Do's ✅
- Provide both interactive and non-interactive modes
- Use consistent colour coding and symbols
- Implement comprehensive keyboard shortcuts
- Cache expensive operations
- Provide helpful error messages with suggestions
- Support configuration files and environment variables
- Include progress indicators for long operations
- Implement undo/redo where appropriate
- Use semantic exit codes
- Provide --json output for scripting

### Don'ts ❌
- Don't block the event loop with synchronous operations
- Don't assume terminal capabilities (check for TTY)
- Don't hardcode paths or configuration
- Don't show stack traces to users (unless --debug)
- Don't require sudo/admin unless necessary
- Don't modify user files without confirmation
- Don't use console.log for structured output
- Don't ignore accessibility requirements
- Don't forget to handle Ctrl+C gracefully
- Don't make breaking changes without version bumps

## Conclusion

Building high-quality CLIs requires careful consideration of user experience, performance, and accessibility. By leveraging modern tools like Ink, Commander, and the ecosystem of CLI utilities, you can create powerful, user-friendly command-line interfaces that delight developers and users alike.

Remember: A great CLI feels intuitive to beginners while providing power features for advanced users. Progressive disclosure, intelligent defaults, and consistent patterns are key to achieving this balance.
