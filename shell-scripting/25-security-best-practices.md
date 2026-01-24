# 25 - Security Best Practices

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Secure credential management
- Input validation and sanitization
- Preventing command injection
- File permission security
- Encryption and hashing
- Secure API interactions
- Security auditing and hardening

---

## 🔐 Credential Management

### **NEVER Hardcode Credentials:**

```bash
#!/bin/bash

# ❌ WRONG - Hardcoded credentials
USERNAME="admin"
PASSWORD="secret123"
API_KEY="abc123xyz"

# ✅ CORRECT - Use environment variables
USERNAME="${DB_USERNAME}"
PASSWORD="${DB_PASSWORD}"
API_KEY="${API_KEY}"

# ✅ CORRECT - Load from secure config
source /etc/app/credentials.conf

# ✅ CORRECT - Use credential store
PASSWORD=$(vault kv get -field=password secret/db)
```

### **Environment Variables:**

```bash
#!/bin/bash

# Check required environment variables
required_vars=(
    "DB_USERNAME"
    "DB_PASSWORD"
    "API_KEY"
)

for var in "${required_vars[@]}"; do
    if [ -z "${!var}" ]; then
        echo "Error: Required environment variable $var is not set"
        exit 1
    fi
done

# Use credentials
mysql -u "$DB_USERNAME" -p"$DB_PASSWORD" -h "$DB_HOST"
```

### **Secure Configuration Files:**

```bash
#!/bin/bash

CONFIG_FILE="/etc/app/credentials.conf"

# Set secure permissions (owner read-only)
create_secure_config() {
    local config_file=$1
    
    # Create with restricted permissions
    (umask 077 && touch "$config_file")
    
    # Set ownership
    sudo chown root:root "$config_file"
    
    # Set permissions (600 = rw-------)
    sudo chmod 600 "$config_file"
}

# Load configuration securely
load_config() {
    local config_file=$1
    
    # Check file exists
    if [ ! -f "$config_file" ]; then
        echo "Error: Config file not found: $config_file"
        return 1
    fi
    
    # Check permissions
    local perms=$(stat -c '%a' "$config_file" 2>/dev/null || stat -f '%A' "$config_file")
    if [ "$perms" != "600" ]; then
        echo "Error: Insecure permissions on config file"
        return 1
    fi
    
    # Source configuration
    # shellcheck disable=SC1090
    source "$config_file"
}
```

### **Using Pass (Password Manager):**

```bash
#!/bin/bash

# Initialize pass (one-time setup)
# pass init your-gpg-key-id

# Store password
store_password() {
    local name=$1
    local password=$2
    echo "$password" | pass insert -e "$name"
}

# Retrieve password
get_password() {
    local name=$1
    pass show "$name"
}

# Usage
DB_PASSWORD=$(get_password "database/production/password")
```

---

## 🛡️ Input Validation

### **Validate All Inputs:**

```bash
#!/bin/bash

# Validate email
validate_email() {
    local email=$1
    local regex="^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
    
    if [[ ! $email =~ $regex ]]; then
        echo "Error: Invalid email format"
        return 1
    fi
    
    return 0
}

# Validate IP address
validate_ip() {
    local ip=$1
    local regex="^([0-9]{1,3}\.){3}[0-9]{1,3}$"
    
    if [[ ! $ip =~ $regex ]]; then
        echo "Error: Invalid IP format"
        return 1
    fi
    
    # Check octets are 0-255
    IFS='.' read -ra OCTETS <<< "$ip"
    for octet in "${OCTETS[@]}"; do
        if [ "$octet" -gt 255 ]; then
            echo "Error: Invalid IP address"
            return 1
        fi
    done
    
    return 0
}

# Validate number
validate_number() {
    local num=$1
    
    if ! [[ $num =~ ^[0-9]+$ ]]; then
        echo "Error: Not a valid number"
        return 1
    fi
    
    return 0
}

# Validate file path (prevent path traversal)
validate_path() {
    local path=$1
    
    # Check for path traversal
    if [[ $path =~ \.\. ]]; then
        echo "Error: Path traversal not allowed"
        return 1
    fi
    
    # Check for absolute path
    if [[ $path != /* ]]; then
        echo "Error: Path must be absolute"
        return 1
    fi
    
    return 0
}

# Validate username (alphanumeric only)
validate_username() {
    local username=$1
    
    if ! [[ $username =~ ^[a-zA-Z0-9_-]{3,32}$ ]]; then
        echo "Error: Invalid username format"
        return 1
    fi
    
    return 0
}
```

### **Sanitize Input:**

```bash
#!/bin/bash

# Remove special characters
sanitize_alphanumeric() {
    local input=$1
    echo "$input" | sed 's/[^a-zA-Z0-9]//g'
}

# Remove shell metacharacters
sanitize_shell() {
    local input=$1
    # Remove potentially dangerous characters
    echo "$input" | sed 's/[;&|`$()<>]//g'
}

# Sanitize filename
sanitize_filename() {
    local filename=$1
    
    # Remove path components
    filename=$(basename "$filename")
    
    # Remove special characters
    filename=$(echo "$filename" | sed 's/[^a-zA-Z0-9._-]/_/g')
    
    # Limit length
    filename=${filename:0:255}
    
    echo "$filename"
}

# URL encode
url_encode() {
    local string=$1
    echo "$string" | jq -sRr @uri
}

# Usage
USER_INPUT="../../etc/passwd"
SAFE_INPUT=$(sanitize_filename "$USER_INPUT")
echo "Safe: $SAFE_INPUT"
```

---

## 💉 Prevent Command Injection

### **NEVER Use eval with User Input:**

```bash
#!/bin/bash

# ❌ DANGEROUS - Command injection vulnerability
USER_INPUT="file.txt; rm -rf /"
eval "cat $USER_INPUT"

# ✅ SAFE - Direct execution
cat "$USER_INPUT"
```

### **Avoid Using User Input in Commands:**

```bash
#!/bin/bash

# ❌ DANGEROUS
filename=$1
cat $(ls | grep "$filename")

# ✅ SAFE - Quote variables
filename=$1
cat "$filename"

# ❌ DANGEROUS
query=$1
mysql -e "SELECT * FROM users WHERE name='$query'"

# ✅ SAFE - Use parameterized queries or escape
query=$1
escaped_query=$(printf '%s' "$query" | sed "s/'/''/g")
mysql -e "SELECT * FROM users WHERE name='$escaped_query'"
```

### **Use Arrays for Commands:**

```bash
#!/bin/bash

# ✅ SAFE - Use arrays
USER_FILE=$1

# Build command as array
cmd=(find /data -name "$USER_FILE" -type f)

# Execute safely
"${cmd[@]}"

# For complex commands
OPTIONS=()
[ -n "$OPTION1" ] && OPTIONS+=("-o" "$OPTION1")
[ -n "$OPTION2" ] && OPTIONS+=("-x" "$OPTION2")

command "${OPTIONS[@]}" "$FILE"
```

---

## 🔒 File Permissions and Ownership

### **Set Secure Permissions:**

```bash
#!/bin/bash

# Create file with secure permissions
create_secure_file() {
    local file=$1
    
    # Set umask for this operation
    (
        umask 077  # Files: 600, Dirs: 700
        touch "$file"
    )
}

# Set secure permissions on existing file
secure_file() {
    local file=$1
    
    # Remove all group and other permissions
    chmod 600 "$file"
    
    # Set owner
    chown root:root "$file"
}

# Create secure directory
create_secure_directory() {
    local dir=$1
    
    mkdir -p "$dir"
    chmod 700 "$dir"
    chown root:root "$dir"
}

# Check file permissions
check_permissions() {
    local file=$1
    local expected_perms=$2
    
    local actual_perms=$(stat -c '%a' "$file" 2>/dev/null || stat -f '%A' "$file")
    
    if [ "$actual_perms" != "$expected_perms" ]; then
        echo "Warning: Insecure permissions on $file"
        echo "Expected: $expected_perms, Got: $actual_perms"
        return 1
    fi
    
    return 0
}
```

---

## 🔐 Encryption and Hashing

### **Hash Passwords:**

```bash
#!/bin/bash

# Hash password with SHA256
hash_password() {
    local password=$1
    echo -n "$password" | sha256sum | cut -d' ' -f1
}

# Hash with salt
hash_password_salt() {
    local password=$1
    local salt=$(openssl rand -hex 16)
    local hash=$(echo -n "$password$salt" | sha256sum | cut -d' ' -f1)
    
    echo "$salt:$hash"
}

# Verify hashed password
verify_password() {
    local password=$1
    local stored=$2
    
    local salt=$(echo "$stored" | cut -d: -f1)
    local hash=$(echo "$stored" | cut -d: -f2)
    
    local computed=$(echo -n "$password$salt" | sha256sum | cut -d' ' -f1)
    
    [ "$hash" = "$computed" ]
}

# Usage
PASSWORD="mypassword"
STORED=$(hash_password_salt "$PASSWORD")
echo "Stored: $STORED"

if verify_password "$PASSWORD" "$STORED"; then
    echo "Password verified"
fi
```

### **Encrypt/Decrypt Files:**

```bash
#!/bin/bash

# Encrypt file with GPG
encrypt_file() {
    local file=$1
    local recipient=$2
    
    gpg --encrypt --recipient "$recipient" "$file"
    
    # Securely delete original
    shred -u "$file"
}

# Decrypt file
decrypt_file() {
    local encrypted_file=$1
    
    gpg --decrypt "$encrypted_file"
}

# Encrypt with password
encrypt_with_password() {
    local file=$1
    local output=$2
    
    openssl enc -aes-256-cbc -salt -in "$file" -out "$output"
}

# Decrypt with password
decrypt_with_password() {
    local encrypted_file=$1
    local output=$2
    
    openssl enc -aes-256-cbc -d -in "$encrypted_file" -out "$output"
}
```

---

## 🌐 Secure API Interactions

### **HTTPS Only:**

```bash
#!/bin/bash

# ✅ Always use HTTPS
api_call() {
    local url=$1
    
    # Verify it's HTTPS
    if [[ ! $url =~ ^https:// ]]; then
        echo "Error: Only HTTPS URLs are allowed"
        return 1
    fi
    
    # Make request with certificate verification
    curl -sSf --cacert /etc/ssl/certs/ca-certificates.crt "$url"
}

# Validate SSL certificate
validate_ssl() {
    local domain=$1
    
    if openssl s_client -connect "$domain:443" </dev/null 2>/dev/null | \
       openssl x509 -noout -checkend 0; then
        echo "SSL certificate is valid"
        return 0
    else
        echo "SSL certificate is invalid or expired"
        return 1
    fi
}
```

### **Secure Token Management:**

```bash
#!/bin/bash

# Store token securely
store_token() {
    local token=$1
    local token_file="/var/run/app/token"
    
    # Create directory with secure permissions
    mkdir -p "$(dirname "$token_file")"
    chmod 700 "$(dirname "$token_file")"
    
    # Store token with secure permissions
    (umask 077 && echo "$token" > "$token_file")
}

# Load token
load_token() {
    local token_file="/var/run/app/token"
    
    if [ ! -f "$token_file" ]; then
        echo "Error: Token file not found"
        return 1
    fi
    
    cat "$token_file"
}

# API call with token
secure_api_call() {
    local url=$1
    local token=$(load_token)
    
    curl -sSf \
        -H "Authorization: Bearer $token" \
        -H "User-Agent: SecureScript/1.0" \
        "$url"
}
```

---

## 🔍 Security Auditing

### **Audit Script:**

```bash
#!/bin/bash

AUDIT_LOG="/var/log/security_audit.log"

log_audit() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" >> "$AUDIT_LOG"
}

# Check for world-writable files
check_world_writable() {
    log_audit "Checking for world-writable files..."
    
    find / -type f -perm -002 2>/dev/null | while read file; do
        log_audit "WARNING: World-writable file: $file"
    done
}

# Check for SUID files
check_suid_files() {
    log_audit "Checking for SUID files..."
    
    find / -type f -perm -4000 2>/dev/null | while read file; do
        log_audit "INFO: SUID file: $file"
    done
}

# Check for files without owner
check_orphan_files() {
    log_audit "Checking for orphan files..."
    
    find / -nouser -o -nogroup 2>/dev/null | while read file; do
        log_audit "WARNING: Orphan file: $file"
    done
}

# Check SSH configuration
check_ssh_config() {
    log_audit "Checking SSH configuration..."
    
    # Check for PermitRootLogin
    if grep -q "^PermitRootLogin yes" /etc/ssh/sshd_config 2>/dev/null; then
        log_audit "WARNING: Root login via SSH is enabled"
    fi
    
    # Check for PasswordAuthentication
    if grep -q "^PasswordAuthentication yes" /etc/ssh/sshd_config 2>/dev/null; then
        log_audit "WARNING: Password authentication is enabled"
    fi
}

# Check for weak passwords
check_weak_passwords() {
    log_audit "Checking password policies..."
    
    # Check password age
    awk -F: '{if ($2 != "" && $2 != "!" && $2 != "*") print $1}' /etc/shadow | \
    while read user; do
        chage -l "$user" 2>/dev/null | grep "Password expires" | grep -q "never" && \
            log_audit "WARNING: Password for $user never expires"
    done
}

# Run all checks
run_security_audit() {
    log_audit "=== Security Audit Started ==="
    
    check_world_writable
    check_suid_files
    check_orphan_files
    check_ssh_config
    check_weak_passwords
    
    log_audit "=== Security Audit Completed ==="
}

run_security_audit
```

---

## 🎯 Security Checklist

```bash
#!/bin/bash

# Security best practices checklist

security_checklist=(
    "✓ Never hardcode credentials"
    "✓ Validate all user inputs"
    "✓ Sanitize file paths"
    "✓ Use quotes around variables"
    "✓ Avoid eval with user input"
    "✓ Set restrictive file permissions (600/700)"
    "✓ Use HTTPS for API calls"
    "✓ Hash passwords with salt"
    "✓ Encrypt sensitive data"
    "✓ Log security events"
    "✓ Run scripts with least privilege"
    "✓ Use secure random generators"
    "✓ Implement timeouts"
    "✓ Clean up temporary files"
    "✓ Validate SSL certificates"
)

for item in "${security_checklist[@]}"; do
    echo "$item"
done
```

---

## 🎯 Quick Check: Do You Understand?

1. **How should you store passwords in scripts?**
   <details>
   <summary>Answer</summary>
   Never hardcode - use environment variables or credential stores
   </details>

2. **What file permissions for sensitive config files?**
   <details>
   <summary>Answer</summary>
   600 (rw-------) - owner read/write only
   </details>

3. **How do you prevent command injection?**
   <details>
   <summary>Answer</summary>
   Validate input, quote variables, avoid eval, use arrays
   </details>

4. **Should you use HTTP or HTTPS for APIs?**
   <details>
   <summary>Answer</summary>
   Always HTTPS with certificate verification
   </details>

5. **How do you sanitize filename input?**
   <details>
   <summary>Answer</summary>
   Use basename, remove special chars, limit length
   </details>

---

## 📝 Key Takeaways

✅ **Never hardcode credentials**  
✅ **Validate and sanitize all inputs**  
✅ **Quote all variables**  
✅ **Avoid eval with user input**  
✅ **Set restrictive file permissions (600/700)**  
✅ **Use HTTPS only**  
✅ **Hash passwords with salt**  
✅ **Encrypt sensitive data**  
✅ **Audit and log security events**  

---

## 🚀 Next Steps

Your scripts are now secure!

**Next lesson:** [26 - Testing and Debugging](26-testing-debugging.md) - Test frameworks and debugging tools

---

## 💡 Pro Tips

**Secure script template:**
```bash
#!/bin/bash
set -euo pipefail
umask 077  # Secure file creation
# Load credentials from env
# Validate all inputs
# Quote all variables
```

**Test for vulnerabilities:**
```bash
shellcheck script.sh  # Static analysis
bandit script.sh      # Security linting
```

**Principle of least privilege:**
```bash
# Run as non-root when possible
sudo -u appuser ./script.sh
```

Security is not optional! 🔐
