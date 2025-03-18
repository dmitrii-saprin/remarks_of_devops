# 📌 How to Export GitLab Projects Using `gitlab-rails console` and a Bash Script

In this guide, we'll cover exporting GitLab projects using the internal **`gitlab-rails console`** utility and automating the process with a **Bash script**. This approach is especially useful when standard Rake tasks (`gitlab:import_export:export`) encounter issues or limitations. You'll find detailed instructions, verification steps, and a ready-to-use script below.

---

## 🎯 What We're Doing

We'll export GitLab projects (e.g., `your_project/infrastructure/terraform` and `your_project/infrastructure/tfmodules`) into `.tar.gz` archives containing:

- Repository
- Metadata (issues, merge requests, pipelines, etc.)
- Everything required for importing into another GitLab instance

If standard Rake tasks fail, we'll use `gitlab-rails console` alongside a Bash script.

---

## 🔹 What is `gitlab-rails console`?

`gitlab-rails console` is an interactive **Ruby console** built into GitLab, allowing direct interaction with internal application objects and the database.

### Why Use `gitlab-rails console`?

- ✅ **Bypassing Errors:** Useful when Rake tasks fail (e.g., nested groups).
- ✅ **Flexibility:** Verify access, check project structures, and initiate exports from one interface.
- ✅ **Administrative Access:** Available only on self-managed GitLab with admin rights.
- ✅ **Asynchronous:** Exports run asynchronously in the background via Sidekiq.

---

## 🛠️ Step-by-Step Guide

### 1. Launching `gitlab-rails console`

Connect to your GitLab server via SSH and run:

```bash
sudo gitlab-rails console
```

You’ll see the prompt: `irb(main):001:0>`.

### 2. Checking and Initiating Export

#### Check User and Project

```ruby
user = User.find_by_username('your_user')
project = Project.find_by_full_path('your_project/infrastructure/terraform')
```

If either returns `nil`, verify:

- Username (GitLab profile)
- Exact project path (project URL)

#### Start Export

```ruby
ProjectExportWorker.new.perform(user.id, project.id)
puts "Export started for #{project.full_path}"
```

Output:

```
Export started for your_project/infrastructure/terraform
```

**Note:** Messages like `Scoped order is ignored, it's forced to be batch order.` are normal Sidekiq behavior.

### 3. Locating the Exported Archive

The archive is created in:

```
/var/opt/gitlab/gitlab-rails/uploads/-/system/import_export_upload/export_file/
```

Exit the console (`exit`) and run:

```bash
find /var/opt/gitlab/gitlab-rails/uploads/ -name "*.tar.gz" -mtime -1
```

📌 `-mtime -1` finds files created within the last 24 hours.

Example output:

```
/var/opt/gitlab/gitlab-rails/uploads/-/system/import_export_upload/export_file/<number>/<timestamp>_your_project_infrastructure_terraform_export.tar.gz
```

The archive typically resides in a numbered subdirectory (e.g., `<number>`).

If the archive doesn't appear immediately, wait **5–30 minutes** and repeat.

---

## 🤖 Automation via Bash Script

Use the **export.sh** script for multiple projects:

### `export.sh` Script

```bash
#!/bin/bash

# === 🔹 Configuration ===
GITLAB_USER="your_user"
OUTPUT_DIR="archives"
REPO_LIST=("your_project/infrastructure/terraform" "your_project/infrastructure/tfmodules")

# === 🔹 Functions ===
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1"
}

export_project() {
    local repo_path="$1"
    local archive_path="${OUTPUT_DIR}/${repo_path}-export.tar.gz"
    local temp_dir="/var/opt/gitlab/gitlab-rails/uploads/-/system/import_export_upload/export_file"
    local log_file="${OUTPUT_DIR}/${repo_path}-export.log"

    mkdir -p "$(dirname "$archive_path")"
    if [[ -f "$archive_path" ]]; then
        log "ℹ️ Archive $archive_path already exists, skipping."
        return 0
    fi

    log "📤 Starting export for project $repo_path..."
    echo "user = User.find_by_username('$GITLAB_USER'); project = Project.find_by_full_path('$repo_path'); ProjectExportWorker.new.perform(user.id, project.id)" | sudo gitlab-rails console > "$log_file" 2>&1

    local max_attempts=360
    local attempt=0
    while [[ $attempt -lt $max_attempts ]]; do
        local temp_archive=$(sudo find "$temp_dir" -name "*.tar.gz" -mtime -1)
        if [[ -n "$temp_archive" && -f "$temp_archive" ]]; then
            sudo mv "$temp_archive" "$archive_path"
            log "✅ Export for project $repo_path saved: $archive_path"
            return 0
        fi
        log "⏳ Waiting for export completion of $repo_path... (Attempt: $((attempt+1))/$max_attempts)"
        sleep 10
        ((attempt++))
    done

    log "❌ Error exporting $repo_path after 60 minutes! Details in $log_file"
    cat "$log_file"
    return 1
}

log "🚀 Starting project exports..."
mkdir -p "$OUTPUT_DIR"

for repo in "${REPO_LIST[@]}"; do
    log "🔧 Processing repository: $repo"
    export_project "$repo"
done

log "🎉 All exports completed!"
```

---

## 📌 How the Script Works

- ✅ **Configuration:** User (`your_user`), output directory (`archives`), and project list (`REPO_LIST`).
- ✅ **`log` Function:** Logs actions with timestamps.
- ✅ **`export_project` Function:**
    - Checks existing archives.
    - Initiates exports via `gitlab-rails console`.
    - Waits up to 60 minutes for archive creation.
    - Moves the archive to specified output.
- ✅ **Main Process:** Iterates over listed projects, calling `export_project`.

---

🚀 Using `gitlab-rails console` with a Bash script provides a reliable method for exporting GitLab projects when standard methods fail. Configure `GITLAB_USER`, `OUTPUT_DIR`, and `REPO_LIST` according to your needs to easily create backups or migrate projects. 🔥