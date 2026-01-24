# 22 - Networking Scripts

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- curl and wget for HTTP requests
- API interaction and JSON parsing
- HTTP methods (GET, POST, PUT, DELETE)
- SSH automation and remote execution
- Network monitoring scripts
- File transfer automation

---

## 🌐 curl - Transfer Data with URLs

### **Basic curl Usage:**

```bash
#!/bin/bash

# Simple GET request
curl https://api.example.com

# Save to file
curl https://example.com -o output.html
curl https://example.com > output.html

# Follow redirects
curl -L https://example.com

# Show headers
curl -I https://example.com

# Verbose output (debugging)
curl -v https://example.com

# Silent mode (no progress)
curl -s https://example.com

# Fail silently on errors
curl -f https://example.com

# Combined: silent, show errors, fail on HTTP errors
curl -sSf https://example.com
```

### **HTTP Methods:**

```bash
#!/bin/bash

# GET (default)
curl https://api.example.com/users

# POST with data
curl -X POST https://api.example.com/users \
    -H "Content-Type: application/json" \
    -d '{"name":"John","email":"john@example.com"}'

# POST from file
curl -X POST https://api.example.com/users \
    -H "Content-Type: application/json" \
    -d @data.json

# PUT (update)
curl -X PUT https://api.example.com/users/123 \
    -H "Content-Type: application/json" \
    -d '{"name":"John Updated"}'

# DELETE
curl -X DELETE https://api.example.com/users/123

# PATCH (partial update)
curl -X PATCH https://api.example.com/users/123 \
    -H "Content-Type: application/json" \
    -d '{"email":"newemail@example.com"}'
```

### **Headers and Authentication:**

```bash
#!/bin/bash

# Custom headers
curl -H "User-Agent: MyScript/1.0" https://api.example.com
curl -H "Accept: application/json" https://api.example.com

# Multiple headers
curl -H "Content-Type: application/json" \
     -H "Authorization: Bearer token123" \
     https://api.example.com

# Basic authentication
curl -u username:password https://api.example.com
curl --user username:password https://api.example.com

# Bearer token
TOKEN="your_token_here"
curl -H "Authorization: Bearer $TOKEN" https://api.example.com

# API key in header
curl -H "X-API-Key: your_api_key" https://api.example.com
```

### **Advanced curl Options:**

```bash
#!/bin/bash

# Set timeout
curl --connect-timeout 5 https://api.example.com
curl --max-time 30 https://api.example.com

# Retry on failure
curl --retry 3 https://api.example.com
curl --retry 3 --retry-delay 2 https://api.example.com

# Download with progress bar
curl -# -O https://example.com/file.zip

# Rate limit (bytes per second)
curl --limit-rate 100K https://example.com/largefile.zip

# Continue partial download
curl -C - -O https://example.com/largefile.zip

# Multiple URLs
curl https://example.com/page1.html https://example.com/page2.html

# Save cookies
curl -c cookies.txt https://example.com

# Send cookies
curl -b cookies.txt https://example.com
```

---

## 📥 wget - Network Downloader

### **Basic wget Usage:**

```bash
#!/bin/bash

# Download file
wget https://example.com/file.zip

# Save with different name
wget -O output.zip https://example.com/file.zip

# Continue partial download
wget -c https://example.com/largefile.zip

# Background download
wget -b https://example.com/file.zip

# Limit download speed
wget --limit-rate=200k https://example.com/file.zip

# Retry on failure
wget --tries=3 https://example.com/file.zip

# Mirror entire website
wget --mirror --no-parent https://example.com

# Download all images from page
wget -r -l 1 -A jpg,png https://example.com
```

---

## 🔧 JSON Parsing with jq

### **Installing jq:**

```bash
#!/bin/bash

# macOS
brew install jq

# Ubuntu/Debian
sudo apt-get install jq

# CentOS/RHEL
sudo yum install jq
```

### **Basic jq Usage:**

```bash
#!/bin/bash

# Pretty print JSON
echo '{"name":"John","age":30}' | jq '.'

# Extract value
echo '{"name":"John","age":30}' | jq '.name'
# Output: "John"

# Extract number (remove quotes)
echo '{"name":"John","age":30}' | jq -r '.age'
# Output: 30

# Extract from array
echo '[{"name":"John"},{"name":"Jane"}]' | jq '.[0].name'
# Output: "John"

# Extract all names
echo '[{"name":"John"},{"name":"Jane"}]' | jq '.[].name'
# Output:
# "John"
# "Jane"

# Filter array
echo '[{"name":"John","age":30},{"name":"Jane","age":25}]' | \
    jq '.[] | select(.age > 28)'
```

---

## 🎯 API Integration Examples

### **Example 1: GitHub API Client**

```bash
#!/bin/bash

GITHUB_USER="octocat"
API_URL="https://api.github.com"

# Get user info
get_user_info() {
    curl -s "$API_URL/users/$GITHUB_USER" | jq '.'
}

# Get repositories
get_repos() {
    curl -s "$API_URL/users/$GITHUB_USER/repos" | \
        jq -r '.[] | .name'
}

# Get repo details
get_repo_details() {
    local repo=$1
    curl -s "$API_URL/repos/$GITHUB_USER/$repo" | \
        jq '{name: .name, stars: .stargazers_count, forks: .forks_count}'
}

# Create issue (requires authentication)
create_issue() {
    local repo=$1
    local title=$2
    local body=$3
    local token=$4
    
    curl -s -X POST \
        -H "Authorization: token $token" \
        -H "Content-Type: application/json" \
        -d "{\"title\":\"$title\",\"body\":\"$body\"}" \
        "$API_URL/repos/$GITHUB_USER/$repo/issues"
}

# Usage
echo "User info:"
get_user_info

echo -e "\nRepositories:"
get_repos
```

### **Example 2: REST API CRUD Operations**

```bash
#!/bin/bash

API_URL="https://jsonplaceholder.typicode.com"

# CREATE - POST
create_post() {
    local title=$1
    local body=$2
    
    curl -s -X POST "$API_URL/posts" \
        -H "Content-Type: application/json" \
        -d "{\"title\":\"$title\",\"body\":\"$body\",\"userId\":1}" | \
        jq '.'
}

# READ - GET
get_post() {
    local id=$1
    curl -s "$API_URL/posts/$id" | jq '.'
}

# UPDATE - PUT
update_post() {
    local id=$1
    local title=$2
    local body=$3
    
    curl -s -X PUT "$API_URL/posts/$id" \
        -H "Content-Type: application/json" \
        -d "{\"id\":$id,\"title\":\"$title\",\"body\":\"$body\",\"userId\":1}" | \
        jq '.'
}

# DELETE
delete_post() {
    local id=$1
    curl -s -X DELETE "$API_URL/posts/$id"
    echo "Post $id deleted"
}

# LIST all
list_posts() {
    curl -s "$API_URL/posts" | jq -r '.[] | "\(.id): \(.title)"'
}

# Usage
echo "Creating post..."
create_post "My Title" "My Content"

echo -e "\nGetting post 1..."
get_post 1

echo -e "\nListing all posts..."
list_posts | head -5
```

### **Example 3: Weather API**

```bash
#!/bin/bash

# Using wttr.in (no API key needed)
get_weather() {
    local location=$1
    curl -s "wttr.in/${location}?format=3"
}

# Using OpenWeatherMap (requires API key)
get_weather_detailed() {
    local city=$1
    local api_key="your_api_key_here"
    
    local response=$(curl -s \
        "https://api.openweathermap.org/data/2.5/weather?q=$city&appid=$api_key&units=metric")
    
    echo "City: $(echo $response | jq -r '.name')"
    echo "Temperature: $(echo $response | jq -r '.main.temp')°C"
    echo "Conditions: $(echo $response | jq -r '.weather[0].description')"
    echo "Humidity: $(echo $response | jq -r '.main.humidity')%"
}

# Usage
get_weather "London"
# Output: London: ⛅️  +15°C
```

---

## 🔐 SSH Automation

### **Basic SSH Commands:**

```bash
#!/bin/bash

# Execute remote command
ssh user@server "ls -la"

# Execute multiple commands
ssh user@server "cd /var/log && tail -n 20 syslog"

# Execute local script on remote server
ssh user@server 'bash -s' < local_script.sh

# Copy environment variables
ssh user@server "export VAR='value' && ./script.sh"
```

### **SSH with Key-Based Authentication:**

```bash
#!/bin/bash

# Generate SSH key (one-time setup)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Copy key to server (one-time setup)
ssh-copy-id user@server

# Now you can SSH without password
ssh user@server
```

### **Remote Script Execution:**

```bash
#!/bin/bash

REMOTE_HOST="user@server"

# Execute commands on remote server
execute_remote() {
    local commands=$1
    ssh "$REMOTE_HOST" "$commands"
}

# Check disk usage
check_disk() {
    execute_remote "df -h"
}

# Check running processes
check_processes() {
    execute_remote "ps aux | grep nginx"
}

# Restart service
restart_service() {
    local service=$1
    execute_remote "sudo systemctl restart $service"
}

# Deploy application
deploy_app() {
    ssh "$REMOTE_HOST" << 'EOF'
        cd /var/www/app
        git pull origin main
        npm install
        pm2 restart app
EOF
}

# Usage
check_disk
check_processes
restart_service nginx
```

---

## 📂 File Transfer Automation

### **Using scp:**

```bash
#!/bin/bash

# Copy file to remote server
scp local_file.txt user@server:/remote/path/

# Copy file from remote server
scp user@server:/remote/file.txt local_path/

# Copy directory recursively
scp -r local_directory/ user@server:/remote/path/

# Copy multiple files
scp file1.txt file2.txt user@server:/remote/path/

# Preserve timestamps and permissions
scp -p file.txt user@server:/remote/path/

# Limit bandwidth (KB/s)
scp -l 1000 largefile.zip user@server:/remote/path/
```

### **Using rsync:**

```bash
#!/bin/bash

# Sync directory to remote server
rsync -avz local_dir/ user@server:/remote/dir/

# Sync from remote server
rsync -avz user@server:/remote/dir/ local_dir/

# Dry run (preview changes)
rsync -avzn local_dir/ user@server:/remote/dir/

# Delete files on destination that don't exist on source
rsync -avz --delete local_dir/ user@server:/remote/dir/

# Exclude files
rsync -avz --exclude='*.log' local_dir/ user@server:/remote/dir/

# Show progress
rsync -avz --progress local_dir/ user@server:/remote/dir/

# Backup script
backup_to_remote() {
    local source=$1
    local destination=$2
    
    rsync -avz --delete \
        --exclude='*.tmp' \
        --exclude='node_modules' \
        "$source" "$destination"
}

# Usage
backup_to_remote "/home/user/data" "user@server:/backups/data"
```

---

## 🔍 Network Monitoring Scripts

### **Example 1: Website Monitor**

```bash
#!/bin/bash

URLS=(
    "https://example.com"
    "https://api.example.com"
    "https://blog.example.com"
)

check_website() {
    local url=$1
    local response=$(curl -s -o /dev/null -w "%{http_code}" --max-time 10 "$url")
    
    if [ "$response" = "200" ]; then
        echo "✓ $url is up (HTTP $response)"
        return 0
    else
        echo "✗ $url is down (HTTP $response)"
        return 1
    fi
}

# Check all URLs
for url in "${URLS[@]}"; do
    check_website "$url"
done
```

### **Example 2: API Health Check**

```bash
#!/bin/bash

API_URL="https://api.example.com/health"
ALERT_EMAIL="admin@example.com"

check_api() {
    local response=$(curl -s --max-time 5 "$API_URL")
    local status=$(echo "$response" | jq -r '.status')
    
    if [ "$status" = "healthy" ]; then
        echo "✓ API is healthy"
        return 0
    else
        echo "✗ API is unhealthy"
        send_alert "API health check failed"
        return 1
    fi
}

send_alert() {
    local message=$1
    echo "$message" | mail -s "API Alert" "$ALERT_EMAIL"
}

# Run check
check_api
```

### **Example 3: SSL Certificate Monitor**

```bash
#!/bin/bash

check_ssl_expiry() {
    local domain=$1
    
    # Get certificate expiry date
    local expiry=$(echo | openssl s_client -servername "$domain" \
        -connect "$domain:443" 2>/dev/null | \
        openssl x509 -noout -enddate | cut -d= -f2)
    
    local expiry_epoch=$(date -d "$expiry" +%s)
    local now_epoch=$(date +%s)
    local days_remaining=$(( (expiry_epoch - now_epoch) / 86400 ))
    
    echo "Domain: $domain"
    echo "Expires: $expiry"
    echo "Days remaining: $days_remaining"
    
    if [ $days_remaining -lt 30 ]; then
        echo "⚠️  Certificate expires soon!"
        return 1
    fi
    
    return 0
}

# Check multiple domains
for domain in example.com api.example.com; do
    check_ssl_expiry "$domain"
    echo ""
done
```

---

## 🎯 Practical Examples

### **Example 4: Download Manager**

```bash
#!/bin/bash

DOWNLOAD_DIR="./downloads"
LOG_FILE="download.log"

download_file() {
    local url=$1
    local filename=$(basename "$url")
    
    echo "[$(date)] Downloading: $url" | tee -a "$LOG_FILE"
    
    if curl -# -f -o "$DOWNLOAD_DIR/$filename" "$url"; then
        echo "[$(date)] ✓ Success: $filename" | tee -a "$LOG_FILE"
        return 0
    else
        echo "[$(date)] ✗ Failed: $filename" | tee -a "$LOG_FILE"
        return 1
    fi
}

# Create download directory
mkdir -p "$DOWNLOAD_DIR"

# Download multiple files
URLS=(
    "https://example.com/file1.zip"
    "https://example.com/file2.zip"
    "https://example.com/file3.zip"
)

for url in "${URLS[@]}"; do
    download_file "$url" &
done

wait
echo "All downloads completed"
```

### **Example 5: Slack/Discord Webhook**

```bash
#!/bin/bash

# Slack webhook
send_slack_message() {
    local webhook_url=$1
    local message=$2
    
    curl -X POST "$webhook_url" \
        -H "Content-Type: application/json" \
        -d "{\"text\":\"$message\"}"
}

# Discord webhook
send_discord_message() {
    local webhook_url=$1
    local message=$2
    
    curl -X POST "$webhook_url" \
        -H "Content-Type: application/json" \
        -d "{\"content\":\"$message\"}"
}

# Usage
SLACK_WEBHOOK="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
send_slack_message "$SLACK_WEBHOOK" "Deployment completed successfully!"
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you make a POST request with curl?**
   <details>
   <summary>Answer</summary>
   curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' URL
   </details>

2. **How do you parse JSON in bash?**
   <details>
   <summary>Answer</summary>
   Use jq: echo '{"name":"John"}' | jq '.name'
   </details>

3. **How do you copy a file to a remote server?**
   <details>
   <summary>Answer</summary>
   scp file.txt user@server:/path/
   </details>

4. **How do you execute a command on a remote server via SSH?**
   <details>
   <summary>Answer</summary>
   ssh user@server "command"
   </details>

5. **How do you follow redirects with curl?**
   <details>
   <summary>Answer</summary>
   curl -L URL
   </details>

---

## 📝 Key Takeaways

✅ **curl for HTTP requests and API calls**  
✅ **jq for JSON parsing**  
✅ **wget for downloading files**  
✅ **ssh for remote command execution**  
✅ **scp/rsync for file transfers**  
✅ **-H for custom headers in curl**  
✅ **-s for silent mode, -f to fail on errors**  

---

## 🚀 Next Steps

You can now automate any network task!

**Next lesson:** [23 - System Administration Scripts](23-system-administration.md) - User management, services, logs

---

## 💡 Pro Tips

**curl wrapper function:**
```bash
api_call() {
    curl -sSf -H "Authorization: Bearer $TOKEN" "$@"
}
```

**Check HTTP status:**
```bash
curl -s -o /dev/null -w "%{http_code}" URL
```

**Parallel downloads:**
```bash
xargs -P 5 -n 1 curl -O < urls.txt
```

Network automation = DevOps superpower! 🌐
