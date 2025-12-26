# /status - Project Status Check

Check the current status of the MCP Inspector project.

## Instructions

Run the following checks and report the results:

1. **Git Status**: Show current branch and uncommitted changes

   ```bash
   git status --short
   git branch --show-current
   ```

2. **Recent Commits**: Show last 5 commits

   ```bash
   git log --oneline -5
   ```

3. **Dependencies**: Check if node_modules exists

   ```bash
   ls -la node_modules 2>/dev/null | head -5 || echo "node_modules not found - run npm install"
   ```

4. **Build Status**: Check if build directories exist
   ```bash
   ls -la client/dist server/build cli/build 2>/dev/null || echo "Not built yet - run npm run build"
   ```

## Output Format

Report status in a concise table format showing:

- Current branch
- Uncommitted changes count
- Build status (built/not built)
- Dependencies status (installed/not installed)
