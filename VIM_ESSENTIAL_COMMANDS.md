**Last Updated:** November 2025  
**Author:** SUMITESHWAR KUMAR

---

# **Essential Vim Commands**

## Vim Modes and Workflow

### Understanding Vim Modes
Vim has three main modes:
- **Normal Mode** (default): For navigation, editing commands, and mode switching
- **Insert Mode**: For typing text (entered with `i`, `a`, `o`, etc.)
- **Visual Mode**: For selecting text (entered with `v`, `V`, `Ctrl+v`)

### When to Press ESC
- **Always press `ESC` or `Ctrl+[`** to exit Insert Mode and return to Normal Mode
- Press ESC after typing text to execute commands
- Press ESC twice if you're unsure which mode you're in
- ESC is your "safe key" - it always returns you to Normal Mode

### Common Workflow Patterns
1. **To edit text**: Navigate in Normal Mode → Press `i`/`a`/`o` → Type text → Press `ESC`
2. **To select and edit**: Press `v`/`V` → Navigate to select → Press `d`/`y`/`c` → (if changing, type new text → `ESC`)
3. **To save**: `ESC` → `:w` → `Enter`
4. **To quit**: `ESC` → `:q` → `Enter` (or `:wq` to save and quit)

### Quick Tips
- You must be in Normal Mode to execute most commands
- If commands don't work, press `ESC` first
- Use `u` to undo mistakes (works in Normal Mode)
- Press `Ctrl+c` as alternative to `ESC` in Insert Mode

## Basic Navigation
- `h` - Move left
- `l` - Move right
- `j` - Move down
- `k` - Move up
- `gg` - Go to top of file
- `G` - Go to bottom of file
- `:n` - Go to line number n

## Line Movement
- `0` - Start of line (column 0)
- `^` - First non-whitespace character of the line
- `$` - End of line

## Word Navigation
- `w` - Move to start of next word
- `W` - Move to next WORD (ignores punctuation)
- `b` - Move to start of previous word
- `B` - Move to previous WORD (ignores punctuation)
- `e` - Move to end of current/next word
- `E` - Move to end of WORD

## Editing
- `i` - Insert before cursor
- `I` - Insert at beginning of line
- `a` - Append after cursor
- `A` - Append at end of line
- `o` - Open new line below
- `O` - Open new line above
- `x` - Delete character under cursor
- `dd` - Delete current line
- `ndd` - Delete n lines (e.g., 4dd)
- `yy` - Yank (copy) current line
- `nyy` - Yank n lines
- `p` - Paste below
- `P` - Paste above
- `u` - Undo
- `Ctrl + r` - Redo

## Visual Mode
- `v` - Start visual mode (characterwise)
- `V` - Start linewise visual mode
- `Ctrl + v` - Start blockwise visual mode
### Navigation in Visual Mode
- `w` - Move forward by word
- `b` - Move backward by word
- `$` - Move to end of line
- `0` - Move to beginning of line
- `h/j/k/l` - Move cursor (left/down/up/right)
- `gg` - Move to beginning of file
- `G` - Move to end of file
### Operations on Selection
- `d` - Delete selection
- `y` - Yank (copy) selection
- `c` - Change selection (delete and enter insert mode)
- `>` - Indent selection
- `<` - Unindent selection
- `~` - Toggle case of selection
- `u` - Make selection lowercase
- `U` - Make selection uppercase

## Search and Replace
- `/pattern` - Search forward for pattern
- `?pattern` - Search backward for pattern
- `n` - Next search result
- `N` - Previous search result
- `:%s/old/new/g` - Replace all 'old' with 'new' in file

## Line Numbers
- `:set number` - Show line numbers
- `:set nu` - Show line numbers (shorthand)
- `:set relativenumber` - Show relative line numbers

## Saving and Quitting
- `:w` - Save file
- `:q` - Quit
- `:wq` - Save and quit
- `:q!` - Quit without saving