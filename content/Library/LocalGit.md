---
tags: library
description: "Auto-commits changes to local git history"
---

# Configuration
To enable this, add this to your `CONFIG.md`:
```yaml
git:
  autoSnapshot: 10 # Commit every 10 minutes

```

# Script Logic

This script runs a "Snapshot" loop.

```space-lua
-- 1. Define the config schema
config.define("git", {
  type = "object",
  properties = {
    autoSnapshot = schema.number()
  }
})

-- 2. Helper to run git commands safely
function runGit(args)
  -- Safety check: does /content/.git exist?
  -- We assume /content is the mount point for the repo
  local check = shell.run("test", {"-d", "/content/.git"})
  if check.code ~= 0 then
    return nil
  end

  -- THE FIX: Prepend "-C /content" to every command
  -- This forces git to look at the Real Data, not the Symlinks
  local gitArgs = {"-C", "/content"}
  for _, arg in ipairs(args) do
    table.insert(gitArgs, arg)
  end

  local res = shell.run("git", gitArgs)
  if res.code ~= 0 then
    print("Git Error: " .. res.stderr)
    return false
  end
  return true
end

-- 3. The Snapshot Function
function snapshot()
  -- Check for changes (Targeting /content)
  local status = shell.run("git", {"-C", "/content", "status", "--porcelain"})
  
  if status.stdout == "" then
    print("Git: Clean (No changes)")
    return
  end

  -- Add and Commit
  print("Git: Snapshotting changes...")
  runGit({"add", "."})
  runGit({"commit", "-m", "Snapshot: " .. os.date("%Y-%m-%d %H:%M:%S")})
  editor.flashNotification("📚 Knowledge Snapshot Taken")
end

-- 4. Manual Command
command.define {
  name = "Git: Snapshot Now",
  run = function()
    snapshot()
  end
}

-- 5. Auto-Loop (Cron Job)
local interval = config.get("git.autoSnapshot")
if interval then
  print("Git: Auto-Snapshot enabled every " .. interval .. " minutes")
  local lastRun = 0
  
  event.listen {
    name = "cron:secondPassed",
    run = function()
      local now = os.time()
      -- Run only if enough time passed
      if (now - lastRun) / 60 >= interval then
        lastRun = now
        snapshot()
      end
    end
  }
end
