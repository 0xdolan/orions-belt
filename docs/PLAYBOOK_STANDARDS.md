# Orion's Belt Playbook Development Standards

## Overview

This document establishes the definitive standards for developing Ansible playbooks within the Orion's Belt project. These standards ensure consistency, security, and maintainability across all automation scripts while supporting AI-assisted development, community contributions, and multiple execution modes.

---

## Table of Contents

1. [Core Principles](#core-principles)
2. [File Naming and Structure](#file-naming-and-structure)
3. [Playbook Metadata](#playbook-metadata)
4. [Task Conventions](#task-conventions)
5. [Host Targeting Standards](#host-targeting-standards)
6. [Variable Management Standards](#variable-management-standards)
7. [Security Standards](#security-standards)
8. [Error Handling and Rollback](#error-handling-and-rollback)
9. [Playbook Testing Standards](#playbook-testing-standards)
10. [Documentation Standards](#documentation-standards)
11. [Execution Mode Compatibility](#execution-mode-compatibility)
12. [Community Contribution Guidelines](#community-contribution-guidelines)
13. [Compliance and Framework Mapping](#compliance-and-framework-mapping)
14. [Version Control and Release Management](#version-control-and-release-management)

---

## Core Principles

### 1.1 Security First
- **Least Privilege**: Use task-level `become` instead of playbook-level
- **Input Validation**: Always validate external inputs and variables
- **Secret Management**: Use Ansible Vault for sensitive data
- **Audit Trail**: Log all security-relevant actions

### 1.2 Idempotency
- **State-Aware**: Use declarative modules over imperative commands
- **Check Before Change**: Verify current state before making modifications
- **Safe Re-runs**: Playbooks must be safely executable multiple times

### 1.3 Modularity
- **Single Responsibility**: Each playbook should have one clear purpose
- **Reusable Components**: Use roles for complex, reusable functionality
- **Clear Dependencies**: Explicitly define and validate dependencies

### 1.4 Maintainability
- **Consistent Structure**: Follow established patterns and conventions
- **Clear Documentation**: Comprehensive comments and descriptions
- **Version Control**: Proper versioning and change tracking

---

## File Naming and Structure

### 2.1 File Naming Convention

**Format**: `OB-[ID]_[description].yml` (hardening playbooks) or `OB-[PREFIX]-[ID]_[description].yml` (utility/config)

**Examples**:
- `OB-004_secure_ssh.yml` (hardening playbook)
- `OB-012_configure_firewall.yml` (hardening playbook)
- `OB-UTIL-002_ansible_ping.yml` (utility playbook)
- `OB-CONFIG-001_nginx_setup.yml` (configuration playbook)

**Naming Patterns**:
- **Hardening Playbooks**: `OB-[ID]_[description].yml` (no category prefix)
- **Utility Playbooks**: `OB-UTIL-[ID]_[description].yml`
- **Configuration Playbooks**: `OB-CONFIG-[ID]_[description].yml`

**Note**: Categorization is primarily handled through the directory structure rather than filename prefixes.

### 2.2 Directory Structure

```
orions-belt/playbooks/
├── os_hardening/
│   ├── authentication/
│   ├── network/
│   ├── filesystem/
│   └── boot/
├── utilities/
├── server_configurations/
├── cloud_hardening/
└── network_hardening/
```

### 2.3 File Organization

Each playbook should include:
- **Main Playbook**: Primary YAML file
- **Templates**: Jinja2 templates (if needed)
- **Handlers**: Task handlers (if needed)
- **Tests**: Molecule test files
- **Documentation**: README or inline documentation

---

## Playbook Metadata

### 3.1 Required Metadata Block

Every playbook must begin with a standardized metadata block:

```yaml
---
- name: "OB-[ID] | [Human Readable Title]"
  hosts: "{{ target_hosts }}"
  become: false  # Use task-level become instead
  gather_facts: true
  
  vars:
    # --- BEGIN ORION'S BELT METADATA ---
    orion_metadata:
      playbook_id: "OB-[ID]"
      title: "[Human Readable Title]"
      description: "[Clear, one-sentence description]"
      version: "1.0.0"
      author: "[Author Name]"
      created_date: "YYYY-MM-DD"
      last_modified: "YYYY-MM-DD"
      category: "[category]"
      os_compatibility:
        - "Debian 12"
        - "RHEL 9"
        - "Ubuntu 22.04"
      architecture_support:
        - "x86_64"
        - "arm64"
      dependencies: []
      tags:
        - "security"
        - "[category]"
        - "[specific-feature]"
      execution_modes:
        - "standalone"
        - "ob.sh"
        - "forge"
    # --- END ORION'S BELT METADATA ---
```

### 3.2 Metadata Field Requirements

**Required Fields**:
- `playbook_id`: Unique identifier following naming convention
- `title`: Human-readable title
- `description`: Clear, concise description
- `version`: Semantic versioning (start at 1.0.0)
- `author`: Original author name
- `created_date`: ISO date format
- `last_modified`: ISO date format
- `category`: Functional category
- `os_compatibility`: List of supported operating systems
- `architecture_support`: List of supported architectures

**Recommended Fields**:
- `dependencies`: List of dependent playbooks
- `tags`: Searchable tags for categorization
- `execution_modes`: Supported execution methods

### 3.3 Attribution Requirements

**Required Attribution**: All playbooks must include the standard Orion's Belt attribution header.

**For Community Contributors**: 
- The attribution template is available in the project's contribution guidelines
- Contact the maintainers for the current attribution template
- Attribution must be included in all submitted playbooks

**For Internal Development**:
- Use the attribution template from the private repository
- Ensure attribution is updated when playbook ownership changes

**Attribution Location**: 
- Must be at the top of each playbook file
- Before the YAML frontmatter
- Include copyright, license, and author information

---

## Task Conventions

### 4.1 Task Naming

**Format**: `[Verb] | [Noun] | [Details]`

**Examples**:
- `Ensure | SSH Protocol | is set to 2`
- `Configure | Firewall Rules | for web servers`
- `Validate | Target Hosts | are specified`

### 4.2 Task Structure

```yaml
- name: "Ensure | SSH Protocol | is set to 2"
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?Protocol'
    line: 'Protocol 2'
    state: present
    validate: 'sshd -t -f %s'
  become: true
  tags:
    - ssh
    - security
    - "control:CIS-5.3.1"
  notify: restart sshd
```

### 4.3 Module Usage Guidelines

**Preferred Modules**:
- `ansible.builtin.lineinfile` - For configuration file modifications
- `ansible.builtin.template` - For complex configuration files
- `ansible.builtin.service` - For service management
- `ansible.builtin.file` - For file and directory operations
- `ansible.builtin.package` - For package management

**Avoid When Possible**:
- `ansible.builtin.shell` - Use only when no module exists
- `ansible.builtin.command` - Use only when no module exists
- `ansible.builtin.raw` - Use only for bootstrap scenarios

### 4.4 Privilege Escalation

**Guidelines**:
- Use `become: true` at task level, not playbook level
- Specify `become_user` when escalating privileges
- Use `become_method: sudo` explicitly when needed
- Avoid `NOPASSWD` in sudoers when possible

```yaml
- name: "Configure | SSH Daemon | settings"
  ansible.builtin.template:
    src: sshd_config.j2
    dest: /etc/ssh/sshd_config
    mode: '0600'
    backup: yes
  become: true
  become_user: root
  become_method: sudo
  notify: restart sshd
```

---

## Host Targeting Standards

### 5.1 Mandatory Host Targeting Requirements

**CRITICAL SECURITY RULE**: All playbooks **MUST** use `hosts: "{{ target_hosts }}"` and **NEVER** use hardcoded host specifications.

#### 5.1.1 Forbidden Patterns

```yaml
# ❌ NEVER USE THESE PATTERNS
- name: Example Playbook
  hosts: all  # DANGEROUS - targets all hosts
  become: yes

- name: Example Playbook  
  hosts: webservers  # DANGEROUS - hardcoded group
  become: yes

- name: Example Playbook
  hosts: 192.168.1.10  # DANGEROUS - hardcoded IP
  become: yes
```

#### 5.1.2 Required Pattern

```yaml
# ✅ ALWAYS USE THIS PATTERN
- name: "OB-[CATEGORY]-[ID] | [Title]"
  hosts: "{{ target_hosts }}"
  become: false  # Use task-level become
  gather_facts: true
```

### 5.2 Host Targeting Validation

**Mandatory Pre-Task** (must be included in every playbook):

```yaml
pre_tasks:
  - name: "Validate | Target Hosts | are specified"
    ansible.builtin.fail:
      msg: |
        CRITICAL SECURITY ERROR: target_hosts variable is not set!
        
        This playbook requires explicit specification of target hosts to prevent
        accidental execution on unintended systems.
        
        Usage: ansible-playbook playbook.yml --extra-vars "target_hosts=your_hosts"
        
        Examples:
        - target_hosts=webservers
        - target_hosts=db_servers
        - target_hosts=192.168.1.10
        - target_hosts=server1,server2,server3
        - target_hosts=anvil-acme-corp-env-prod-*
    when: target_hosts is not defined or target_hosts == ""
```

### 5.3 Host Targeting Methods

#### 5.3.1 Standalone Execution

```bash
# Target specific hosts
ansible-playbook OB-004_secure_ssh.yml \
  -e "target_hosts=webservers"

# Target specific IP addresses
ansible-playbook OB-004_secure_ssh.yml \
  -e "target_hosts=192.168.1.10,192.168.1.11"

# Target using patterns
ansible-playbook OB-004_secure_ssh.yml \
  -e "target_hosts=anvil-acme-corp-env-prod-*"
```

#### 5.3.2 ob.sh Integration

```bash
# ob.sh automatically handles target_hosts variable
./ob.sh run OB-004_secure_ssh
```

#### 5.3.3 Forge Interface

The Forge platform automatically provides the `target_hosts` variable based on:
- Selected hosts in the UI
- Host group selections
- Environment-based targeting

### 5.4 Host Group Targeting

When targeting host groups managed through the Forge UI:

```yaml
# Use standardized host group names
- name: "Target Production Web Servers"
  hosts: "{{ target_hosts }}"
  vars:
    target_group: "web-servers-prod"
    environment: "production"
```

### 5.5 Limit vs Hosts Modification

**Use `--limit` for ad-hoc execution, never modify the playbook's `hosts` line**:

```bash
# ✅ CORRECT: Use --limit for subset execution
ansible-playbook OB-004_secure_ssh.yml \
  -e "target_hosts=webservers" \
  --limit "web1,web2"

# ❌ INCORRECT: Modifying playbook hosts line
# Never change hosts: "{{ target_hosts }}" in the playbook
```

---

## Variable Management Standards

### 6.1 Variable Precedence Hierarchy

**Orion's Belt Variable Resolution Order** (highest to lowest priority):

1. **Host Variables** (`host_vars/[hostname].yml`)
2. **Host Group Variables** (`group_vars/[group].yml`)
3. **Environment Variables** (`group_vars/all/main.yml`)
4. **Playbook Variables** (`vars:` section)
5. **Default Values** (fallback defaults)

### 6.2 Variable Naming Convention

**Format**: `[context]_[purpose]_[name]`

**Examples**:
- `ssh_port`
- `firewall_allowed_ports`
- `user_admin_groups`
- `orion_database_port`
- `orion_api_timeout`

### 6.3 Variable Definition Patterns

#### 6.3.1 Environment-Level Variables

```yaml
# group_vars/all/main.yml
network:
  ssh:
    port: 22
    protocol: 2
    allow_root_login: false
    password_authentication: false
    key_authentication: true
    max_auth_tries: 3
    client_alive_interval: 300
    client_alive_count_max: 2
    login_grace_time: 60

security_hardening:
  firewall:
    default_policy: "drop"
    allow_icmp: true
    rate_limit_ssh: true
    ssh_rate_limit: "5/minute"
```

#### 6.3.2 Playbook Variable Resolution

```yaml
vars:
  # SSH Configuration - resolve from environment with defaults
  ssh_port: "{{ network.ssh.port | default(22) }}"
  ssh_protocol: "{{ network.ssh.protocol | default(2) }}"
  ssh_root_login: "{{ network.ssh.allow_root_login | default('no') | string }}"
  ssh_password_auth: "{{ network.ssh.password_authentication | default('no') | string }}"
  ssh_pubkey_auth: "{{ network.ssh.key_authentication | default('yes') | string }}"
  ssh_max_auth_tries: "{{ network.ssh.max_auth_tries | default(3) }}"
  ssh_client_alive_interval: "{{ network.ssh.client_alive_interval | default(300) }}"
  ssh_client_alive_count_max: "{{ network.ssh.client_alive_count_max | default(2) }}"
  ssh_login_grace_time: "{{ network.ssh.login_grace_time | default(60) }}"
  
  # Firewall Configuration
  firewall_default_policy: "{{ security_hardening.firewall.default_policy | default('drop') }}"
  firewall_allow_icmp: "{{ security_hardening.firewall.allow_icmp | default(true) }}"
  firewall_rate_limit_ssh: "{{ security_hardening.firewall.rate_limit_ssh | default(true) }}"
  firewall_ssh_rate_limit: "{{ security_hardening.firewall.ssh_rate_limit | default('5/minute') }}"
```

### 6.4 Forbidden Variable Practices

#### 6.4.1 No Hardcoded Secrets

```yaml
# ❌ NEVER DO THIS
vars:
  database_password: "mypassword123"  # DANGEROUS
  api_key: "sk-1234567890abcdef"      # DANGEROUS
  admin_password: "admin123"          # DANGEROUS
```

#### 6.4.2 No Environment-Specific Values

```yaml
# ❌ NEVER DO THIS
vars:
  database_host: "prod-db.company.com"  # Environment-specific
  api_endpoint: "https://api.prod.com"  # Environment-specific
  log_level: "debug"                     # Should be configurable
```

### 6.5 Secret Management with Ansible Vault

#### 6.5.1 Vault Variable Definition

```yaml
# group_vars/all/vault.yml (encrypted)
vault_database_password: "{{ vault_database_password }}"
vault_api_key: "{{ vault_api_key }}"
vault_ssh_private_key: "{{ vault_ssh_private_key }}"
```

#### 6.5.2 Secure Variable Usage

```yaml
# In playbook
- name: "Set | Database Password | from vault"
  community.mysql.mysql_user:
    name: root
    password: "{{ vault_database_password }}"
  no_log: true  # CRITICAL: Prevent logging of secrets

- name: "Configure | API Key | for service"
  ansible.builtin.lineinfile:
    path: /etc/service/config.conf
    regexp: '^api_key='
    line: 'api_key={{ vault_api_key }}'
  no_log: true
```

### 6.6 Input Validation

#### 6.6.1 Variable Validation

```yaml
pre_tasks:
  - name: "Validate | SSH Port | is within valid range"
    ansible.builtin.assert:
      that:
        - ssh_port is defined
        - ssh_port is number
        - ssh_port >= 1
        - ssh_port <= 65535
      fail_msg: "SSH port must be a number between 1 and 65535"
      success_msg: "SSH port validation passed."

  - name: "Validate | Target Hosts | are specified"
    ansible.builtin.assert:
      that:
        - target_hosts is defined
        - target_hosts != ""
      fail_msg: "CRITICAL SECURITY ERROR: target_hosts variable is not set!"
      success_msg: "Target hosts validation passed."
```

#### 6.6.2 Complex Variable Validation

```yaml
- name: "Validate | Username | format"
  ansible.builtin.assert:
    that:
      - "username is match('^[a-z_][a-z0-9_-]*$')"
      - username | length >= 3
      - username | length <= 32
    fail_msg: "Invalid username format. Must be 3-32 characters, lowercase letters, numbers, underscores, and hyphens only."
```

### 6.7 Variable Documentation

#### 6.7.1 Required Variable Documentation

```yaml
vars:
  # SSH Configuration
  # These variables are resolved from the environment configuration
  # with sensible defaults. Override in host_vars or group_vars as needed.
  ssh_port: "{{ network.ssh.port | default(22) }}"
  ssh_protocol: "{{ network.ssh.protocol | default(2) }}"
  
  # Security Settings
  # Boolean values are converted to strings for consistency
  ssh_root_login: "{{ network.ssh.allow_root_login | default('no') | string }}"
  ssh_password_auth: "{{ network.ssh.password_authentication | default('no') | string }}"
```

### 6.8 Variable Testing

#### 6.8.1 Variable Resolution Testing

```yaml
# molecule/default/tests/test_variables.yml
---
- name: "Test | Variable Resolution | for SSH configuration"
  hosts: localhost
  connection: local
  gather_facts: false
  
  tasks:
    - name: "Verify | SSH Port | is properly resolved"
      ansible.builtin.assert:
        that:
          - ssh_port is defined
          - ssh_port is number
          - ssh_port == 22  # Default value
```

### 6.9 Forge UI Variable Management

When using variables defined in the Forge UI:

```yaml
# Variables defined in Forge UI are automatically available
# Use the orion_ prefix to avoid conflicts
vars:
  database_port: "{{ orion_database_port | default(5432) }}"
  api_timeout: "{{ orion_api_timeout | default(30) }}"
  log_level: "{{ orion_log_level | default('info') }}"
```

### 6.10 Secret Management

**Use Ansible Vault**:
```yaml
# group_vars/all/vault.yml (encrypted)
vault_db_password: "{{ vault_db_password }}"
vault_api_key: "{{ vault_api_key }}"

# In playbook
- name: "Set | Database Password | from vault"
  community.mysql.mysql_user:
    name: root
    password: "{{ vault_db_password }}"
  no_log: true  # CRITICAL: Prevent logging of secrets
```

### 6.11 Input Validation

```yaml
pre_tasks:
  - name: "Validate | Target Hosts | are specified"
    ansible.builtin.assert:
      that:
        - target_hosts is defined
        - target_hosts != ""
      fail_msg: "CRITICAL SECURITY ERROR: target_hosts variable is not set!"
      success_msg: "Target hosts validation passed."
```

---

## Security Standards

### 7.1 Host Targeting Validation

**Mandatory Pre-Task**:
```yaml
pre_tasks:
  - name: "Validate | Target Hosts | are specified"
    ansible.builtin.fail:
      msg: |
        CRITICAL SECURITY ERROR: target_hosts variable is not set!
        
        This playbook requires explicit specification of target hosts to prevent
        accidental execution on unintended systems.
        
        Usage: ansible-playbook playbook.yml --extra-vars "target_hosts=your_hosts"
        
        Examples:
        - target_hosts=webservers
        - target_hosts=db_servers
        - target_hosts=192.168.1.10
        - target_hosts=server1,server2,server3
    when: target_hosts is not defined or target_hosts == ""
```

### 7.2 Ansible Vault for Secrets Management

**CRITICAL SECURITY RULE**: All sensitive data **MUST** be encrypted using Ansible Vault. Never store secrets in plain text.

#### 7.2.1 Vault File Creation

```bash
# Create encrypted variables file
ansible-vault create group_vars/all/vault.yml

# Edit existing encrypted file
ansible-vault edit group_vars/all/vault.yml

# Encrypt single variable inline (SECURE METHOD - NO HISTORY)
# ⚠️ SECURITY WARNING: Never use direct command line arguments for passwords
# ❌ DANGEROUS: ansible-vault encrypt_string --stdin-name 'db_password' 'MyPassword!'
#    This leaves the password in bash history!
# ❌ DANGEROUS: echo 'password' > file (leaves password in bash history)
# ❌ DANGEROUS: echo -n 'password' | command (leaves password in bash history)

# ✅ Method 1: Use read command (most secure)
read -s -p "Enter password: " password
echo "$password" | ansible-vault encrypt_string --stdin-name 'db_password' -
unset password

# ✅ Method 2: Use a text editor (secure)
# Create a temporary file, edit it manually, then use it
touch /tmp/temp_password
# Edit /tmp/temp_password with your preferred editor (vim, nano, etc.)
# Then run:
ansible-vault encrypt_string --stdin-name 'db_password' < /tmp/temp_password
rm /tmp/temp_password

# ✅ Method 3: Use ansible-vault create/edit (recommended)
# Create or edit an encrypted file directly
ansible-vault create group_vars/all/vault.yml
# or
ansible-vault edit group_vars/all/vault.yml
```

#### 7.2.2 Vault Variable Usage

```yaml
# group_vars/all/vault.yml (encrypted)
vault_database_password: "{{ vault_database_password }}"
vault_api_key: "{{ vault_api_key }}"
vault_ssh_private_key: "{{ vault_ssh_private_key }}"

# In playbook
- name: "Set | Database Password | from vault"
  community.mysql.mysql_user:
    name: root
    password: "{{ vault_database_password }}"
  no_log: true  # CRITICAL: Prevent logging of secrets

- name: "Configure | API Key | for service"
  ansible.builtin.lineinfile:
    path: /etc/service/config.conf
    regexp: '^api_key='
    line: 'api_key={{ vault_api_key }}'
  no_log: true
```

#### 7.2.3 Vault Execution

```bash
# Interactive password prompt
ansible-playbook playbook.yml --ask-vault-pass

# Password file (more secure for automation)
ansible-playbook playbook.yml --vault-password-file ~/.vault_pass.txt

# Environment variable
export ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass.txt
ansible-playbook playbook.yml
```

### 7.3 Principle of Least Privilege

**Guidelines**:
- Use `become: true` at task level, not playbook level
- Specify `become_user` when escalating privileges
- Use dedicated service accounts for Ansible connections
- Limit sudo privileges to only necessary commands

```yaml
- name: "Configure | SSH Daemon | settings"
  ansible.builtin.template:
    src: sshd_config.j2
    dest: /etc/ssh/sshd_config
    mode: '0600'
    backup: yes
  become: true
  become_user: root
  become_method: sudo
  notify: restart sshd

- name: "Deploy | Application Code | from git"
  ansible.builtin.git:
    repo: 'https://github.com/orions-belt/webapp.git'
    dest: /var/www/my_app
    version: main
  become: true
  become_user: www-data  # Less privileged user
```

### 7.4 Secure Module Usage

**CRITICAL**: Avoid `shell` and `command` modules when possible. Use dedicated Ansible modules for better security.

#### 7.4.1 Preferred Modules

```yaml
# ✅ SAFE: Use dedicated modules
- name: "Create | User | account"
  ansible.builtin.user:
    name: "{{ username }}"
    state: present
    shell: /bin/bash

- name: "Manage | Service | state"
  ansible.builtin.systemd:
    name: nginx
    state: started
    enabled: yes

- name: "Install | Package | via package manager"
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: yes
```

#### 7.4.2 When Shell/Command is Necessary

```yaml
# ⚠️ CAUTION: Only when no module exists
- name: "Execute | Custom Script | for legacy system"
  ansible.builtin.shell: |
    /usr/local/bin/legacy_script.sh \
      --user "{{ username }}" \
      --config "{{ config_file }}"
  args:
    creates: /var/log/legacy_script.log  # Make idempotent
  become: true
  # Validate inputs before use
  when: username is match('^[a-z_][a-z0-9_-]*$')
```

### 7.5 Sensitive Data Protection

**Use `no_log: true`** for any task handling:
- Passwords
- API keys
- Private certificates
- Vault variables

### 7.6 File Permissions

**Secure Defaults**:
- Configuration files: `0600` (owner read/write only)
- Executable files: `0755` (owner read/write/execute, others read/execute)
- Log files: `0644` (owner read/write, others read)

### 7.7 Input Sanitization

```yaml
- name: "Validate | Username | format"
  ansible.builtin.assert:
    that:
      - "username is match('^[a-z_][a-z0-9_-]*$')"
      - username | length >= 3
      - username | length <= 32
    fail_msg: "Invalid username format. Must be 3-32 characters, lowercase letters, numbers, underscores, and hyphens only."
```

---

## Error Handling and Rollback

### 8.1 Block/Rescue/Always Structure

**MANDATORY**: Use `block`/`rescue`/`always` for critical multi-task operations that could leave systems in inconsistent states.

#### 8.1.1 Basic Structure

```yaml
- name: "Deploy | Application | with rollback capability"
  block:
    - name: "Step 1: Backup current application"
      ansible.builtin.archive:
        path: /var/www/html/*
        dest: "/opt/backups/webapp-{{ ansible_date_time.iso8601 }}.tar.gz"
        format: gz

    - name: "Step 2: Deploy new version"
      ansible.builtin.copy:
        src: "files/webapp-{{ app_version }}/"
        dest: /var/www/html/
        owner: www-data
        group: www-data

    - name: "Step 3: Restart web service"
      ansible.builtin.systemd:
        name: nginx
        state: restarted

    - name: "Step 4: Run health check"
      ansible.builtin.uri:
        url: http://localhost/health
        return_content: yes
      register: health_check
      failed_when: "'OK' not in health_check.content"

  rescue:
    - name: "ROLLBACK: Restore application from backup"
      ansible.builtin.unarchive:
        src: "/opt/backups/webapp-{{ ansible_date_time.iso8601 }}.tar.gz"
        dest: /var/www/html/
        remote_src: yes
      when: backup_path is defined

    - name: "ROLLBACK: Restart web service after restore"
      ansible.builtin.systemd:
        name: nginx
        state: restarted

    - name: "Notify team of failed deployment"
      ansible.builtin.debug:
        msg: "Deployment failed on {{ inventory_hostname }} and was rolled back. Failure info: {{ ansible_failed_result }}"

    - name: "Force failure of the play"
      ansible.builtin.fail:
        msg: "Deployment failed and was rolled back. Halting execution."

  always:
    - name: "CLEANUP: Ensure permissions are correct"
      ansible.builtin.file:
        path: /var/www/html
        owner: www-data
        group: www-data
        recurse: yes

    - name: "CLEANUP: Remove old backups"
      ansible.builtin.find:
        paths: /opt/backups
        age: "7d"
        patterns: "webapp-*.tar.gz"
      register: old_backups

    - name: "Remove old backup files"
      ansible.builtin.file:
        path: "{{ item.path }}"
        state: absent
      loop: "{{ old_backups.files }}"
```

### 8.2 State Control with `changed_when` and `failed_when`

#### 8.2.1 `failed_when` for Custom Failure Conditions

```yaml
- name: "Check | Chrony Process | status"
  ansible.builtin.command: pgrep chronyd
  register: chrony_process
  changed_when: false  # This command never makes a change
  failed_when: chrony_process.rc != 0 and "expected_to_be_running" in group_names
  # Fail only if process isn't running AND host is supposed to have it
```

#### 8.2.2 `changed_when` for Idempotency

```yaml
- name: "Verify | SSH Connectivity | to target hosts"
  ansible.builtin.ping:
  register: ping_result
  # Override default changed=true to make this a pure check
  changed_when: false

- name: "Display | Ping Status | for each host"
  ansible.builtin.debug:
    msg: "Host {{ inventory_hostname }} is reachable."
  when: ping_result.ping == 'pong'
```

### 8.3 Backup Strategy

**Always Create Backups** for critical configuration files:

```yaml
- name: "Create | Configuration Backup | before changes"
  ansible.builtin.copy:
    src: /etc/ssh/sshd_config
    dest: "/etc/ssh/sshd_config.backup.{{ ansible_date_time.epoch }}"
    remote_src: yes
    backup: yes
```

### 8.4 File-Level Rollback with `backup` Parameter

**MANDATORY**: Use `backup: yes` for modules that modify critical configuration files.

```yaml
- name: "Deploy | Sudoers Configuration | with backup"
  ansible.builtin.copy:
    src: files/sudoers_secure
    dest: /etc/sudoers
    owner: root
    group: root
    mode: '0440'
    validate: 'visudo -cf %s'  # CRITICAL: always validate sudoers
    backup: yes  # Creates backup like /etc/sudoers.28374.2023-10-27@10:30:00~
```

### 8.5 Rollback Handlers

```yaml
handlers:
  - name: "Rollback | SSH Configuration | on failure"
    ansible.builtin.copy:
      src: "/etc/ssh/sshd_config.backup.{{ ansible_date_time.epoch }}"
      dest: /etc/ssh/sshd_config
      mode: '0600'
    listen: "rollback ssh config"
```

### 8.6 Pre-Production Validation

**MANDATORY**: Run playbooks with `--check` and `--diff` before production deployment.

```bash
# Dry run to see what would change
ansible-playbook playbook.yml -e "target_hosts=webservers" --check --diff

# Check mode with verbose output
ansible-playbook playbook.yml -e "target_hosts=webservers" --check --diff -v
```

### 8.7 Play-Level Error Control

**Use `any_errors_fatal`** when a single failure should stop the entire play:

```yaml
- name: "Critical | System Configuration | that must succeed"
  hosts: "{{ target_hosts }}"
  any_errors_fatal: true  # Stop play if any task fails
  
  tasks:
    - name: "Configure | Critical Security Setting | 1"
      ansible.builtin.lineinfile:
        path: /etc/security/limits.conf
        line: '* soft nofile 65536'
        state: present

    - name: "Configure | Critical Security Setting | 2"
      ansible.builtin.lineinfile:
        path: /etc/security/limits.conf
        line: '* hard nofile 65536'
        state: present
```

---

## Playbook Testing Standards

### 9.1 Molecule Requirement

**Mandatory Testing**: Every new or modified playbook/role **must** include a complete and passing Molecule test suite. This ensures that all automation is validated for correctness, idempotence, and compliance with standards across all supported execution modes.

**Scope**: This requirement applies to:
- All new playbooks and roles
- Any modifications to existing playbooks/roles
- Community contributions and pull requests
- Internal development work

### 9.2 Directory Structure

**Standard Molecule Structure**:
```
playbook_name/
├── molecule/
│   ├── default/
│   │   ├── molecule.yml          # Main configuration
│   │   ├── create.yml            # Container creation
│   │   ├── destroy.yml           # Container cleanup
│   │   ├── converge.yml          # Playbook execution
│   │   ├── verify.yml            # Verification tests
│   │   └── prepare.yml           # Pre-execution setup
│   └── check_mode/               # Additional scenario
│       ├── molecule.yml
│       ├── converge.yml
│       └── verify.yml
└── playbook.yml
```

**Required Files**:
- `molecule.yml` - Main configuration file
- `converge.yml` - Playbook execution (usually references the main playbook)
- `verify.yml` - Verification playbook
- `create.yml` - Container creation (optional, uses defaults if not present)
- `destroy.yml` - Container cleanup (optional, uses defaults if not present)

### 9.3 Standard molecule.yml Configuration

**Template Configuration**:
```yaml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: debian12
    image: geerlingguy/docker-debian12-ansible:latest
    pre_build_image: true
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:rw
    cgroupns_mode: host
    privileged: true
    environment:
      container: docker
    ansible_connection: docker
    ansible_python_interpreter: /usr/bin/python3
provisioner:
  name: ansible
  config_options:
    defaults:
      interpreter_python: auto_silent
      host_key_checking: false
      retry_files_enabled: false
    ssh_connection:
      ssh_args: -o ControlMaster=auto -o ControlPersist=60s
  playbooks:
    converge: converge.yml
    verify: verify.yml
verifier:
  name: ansible
  playbooks:
    verify: verify.yml
```

**Configuration Details**:
- **Driver**: Docker (recommended) or Podman
- **Platform**: `geerlingguy/docker-debian12-ansible` (standard test platform)
- **Provisioner**: Ansible with optimized settings
- **Verifier**: Ansible for comprehensive verification

### 9.4 Test Scenarios

**Required Scenarios**:

#### 9.4.1 Default Scenario
**Purpose**: Full lifecycle testing including lint, converge, idempotence, and verify
**Command**: `molecule test -s default`

**Execution Steps**:
1. **dependency** - Install role dependencies
2. **lint** - Run ansible-lint checks
3. **syntax** - Validate YAML and Ansible syntax
4. **create** - Create test containers
5. **prepare** - Pre-execution setup (if needed)
6. **converge** - Execute the playbook
7. **idempotence** - Verify idempotency (second run shows no changes)
8. **verify** - Run verification tests
9. **destroy** - Clean up test containers

#### 9.4.2 Check Mode Scenario
**Purpose**: Validate dry-run functionality and syntax
**Command**: `molecule test -s check_mode`

**Configuration**:
```yaml
# molecule/check_mode/molecule.yml
---
driver:
  name: docker
platforms:
  - name: debian12
    image: geerlingguy/docker-debian12-ansible:latest
    pre_build_image: true
    command: ${MOLECULE_DOCKER_COMMAND:-""}
    volumes:
      - /sys/fs/cgroup:/sys/fs/cgroup:rw
    cgroupns_mode: host
    privileged: true
    environment:
      container: docker
    ansible_connection: docker
    ansible_python_interpreter: /usr/bin/python3
provisioner:
  name: ansible
  config_options:
    defaults:
      interpreter_python: auto_silent
      host_key_checking: false
      retry_files_enabled: false
  playbooks:
    converge: converge.yml
verifier:
  name: ansible
```

**Converge Playbook**:
```yaml
# molecule/check_mode/converge.yml
---
- name: Check mode test
  hosts: all
  gather_facts: true
  tasks:
    - name: Include the playbook in check mode
      ansible.builtin.import_playbook: ../../playbook.yml
      vars:
        ansible_check_mode: true
```

### 9.5 Verification Playbooks

**Standard verify.yml Template**:
```yaml
---
- name: Verify playbook execution
  hosts: all
  gather_facts: true
  tasks:
    # Verify target_hosts validation
    - name: Verify target_hosts variable is used
      ansible.builtin.assert:
        that:
          - target_hosts is defined
        fail_msg: "Playbook must use target_hosts variable for security"
    
    # Verify file permissions (example)
    - name: Verify file permissions are secure
      ansible.builtin.stat:
        path: /etc/example/config.conf
      register: config_file
      
    - name: Assert secure file permissions
      ansible.builtin.assert:
        that:
          - config_file.stat.mode is defined
          - config_file.stat.mode | regex_replace('^0', '') | int <= 644
        fail_msg: "Configuration file has insecure permissions"
    
    # Verify service status (example)
    - name: Verify service is running
      ansible.builtin.systemd:
        name: "{{ service_name | default('example-service') }}"
      register: service_status
      
    - name: Assert service is active
      ansible.builtin.assert:
        that:
          - service_status.status.ActiveState == "active"
        fail_msg: "Service is not running"
    
    # Verify no sensitive data exposure
    - name: Check for hardcoded secrets
      ansible.builtin.shell: |
        grep -r "password.*=.*['\"][^'\"]*['\"]" /etc/example/ || true
      register: secret_check
      changed_when: false
      
    - name: Assert no hardcoded secrets
      ansible.builtin.assert:
        that:
          - secret_check.stdout == ""
        fail_msg: "Found potential hardcoded secrets in configuration"
    
    # Verify backup files exist (if applicable)
    - name: Check for backup files
      ansible.builtin.find:
        paths: /etc/example/
        patterns: "*.backup"
      register: backup_files
      
    - name: Assert backup files exist
      ansible.builtin.assert:
        that:
          - backup_files.matched > 0
        fail_msg: "No backup files found - backup strategy may not be working"
```

**Verification Requirements**:
- **Security Checks**: Verify target_hosts usage, file permissions, no hardcoded secrets
- **State Verification**: Confirm desired state is achieved
- **Error Handling**: Test error conditions and rollback mechanisms
- **Standards Compliance**: Verify adherence to playbook standards

### 9.6 Execution Workflow

**Standard Testing Commands**:

#### 9.6.1 Full Test Suite
```bash
# Run complete test suite (default scenario)
molecule test -s default

# Run with verbose output
molecule test -s default --verbose

# Run specific steps only
molecule converge -s default
molecule verify -s default
```

#### 9.6.2 Individual Steps
```bash
# Syntax and linting
molecule lint -s default
molecule syntax -s default

# Container management
molecule create -s default
molecule destroy -s default

# Playbook execution
molecule converge -s default

# Verification
molecule verify -s default

# Idempotence test
molecule idempotence -s default
```

#### 9.6.3 Check Mode Testing
```bash
# Test check mode scenario
molecule test -s check_mode

# Run check mode only
molecule converge -s check_mode
```

### 9.7 Integration with CI/CD

**GitHub Actions Example**:
```yaml
name: Molecule Tests
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.9'
          
      - name: Install dependencies
        run: |
          pip install molecule[docker] ansible-core ansible-lint
          
      - name: Run Molecule tests
        run: |
          molecule test -s default
          molecule test -s check_mode
```

### 9.8 Troubleshooting

**Common Issues and Solutions**:

#### 9.8.1 Container Issues
```bash
# Clean up stuck containers
molecule destroy --all

# Force recreate containers
molecule create --force
```

#### 9.8.2 Permission Issues
```bash
# Fix Docker permissions
sudo chmod 666 /var/run/docker.sock

# Use podman instead
export MOLECULE_DRIVER=podman
```

#### 9.8.3 Network Issues
```bash
# Use host networking
# Add to molecule.yml platforms section:
networks:
  - name: host
```

### 9.9 Best Practices

**Testing Guidelines**:
- **Test Early**: Write tests before or alongside playbook development
- **Test Often**: Run tests frequently during development
- **Test Everything**: Cover all code paths and edge cases
- **Test Realistically**: Use realistic test data and scenarios
- **Document Tests**: Clearly document test purpose and expected outcomes

**Performance Considerations**:
- Use lightweight base images for faster testing
- Parallelize tests when possible
- Cache dependencies to speed up builds
- Use check mode for quick syntax validation

---

## Documentation Standards

### 10.1 Inline Documentation

**Task Comments**:
```yaml
# Configure SSH daemon to use protocol 2 only
# This addresses CIS control 5.3.1 and NIST AC-02
- name: "Ensure | SSH Protocol | is set to 2"
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?Protocol'
    line: 'Protocol 2'
    state: present
    validate: 'sshd -t -f %s'  # Validate config before applying
  become: true
  tags:
    - ssh
    - security
    - "control:CIS-5.3.1"
```

### 10.2 Playbook Header Documentation

```yaml
---
# Orion's Belt Playbook: OB-004
# Purpose: Secure SSH daemon configuration
# Author: [Author Name]
# Version: 1.0.0
# 
# [Standard Attribution Header Required - See Section 3.3]
# 
# This playbook implements secure SSH configuration settings
# to comply with security hardening requirements.
# 
# Prerequisites:
# - SSH server installed
# - Root or sudo access
# 
# Usage:
#   ansible-playbook OB-004_secure_ssh.yml -e "target_hosts=webservers"
#   ./ob.sh run OB-004_secure_ssh
# 
# Security Notes:
# - Creates backup before making changes
# - Validates configuration before applying
# - Supports rollback on validation failure
```

---

## Execution Mode Compatibility

### 11.1 Standalone Execution

**Command Format**:
```bash
ansible-playbook OB-[CATEGORY]-[ID]_[description].yml \
  -e "target_hosts=your_hosts" \
  --vault-password-file ~/.vault_pass.txt
```

### 11.2 ob.sh Integration

**Command Format**:
```bash
./ob.sh run OB-[CATEGORY]-[ID]_[description]
```

### 11.3 Forge Interface

**Requirements**:
- Compatible with Forge platform execution
- No hardcoded paths or dependencies
- Proper error reporting for UI integration
- Support for interactive prompts (when implemented)

---

## Community Contribution Guidelines

### 12.1 Contribution Workflow

1. **Fork Repository**: Create fork of orions-belt
2. **Create Branch**: Use `contrib` branch as base
3. **Follow Standards**: Adhere to all standards in this document
4. **Test Thoroughly**: Include Molecule tests
5. **Submit PR**: Target `contrib` branch
6. **Review Process**: Address feedback and iterate

### 12.2 Quality Assurance

**Required Checks**:
- [ ] Follows naming conventions
- [ ] Includes complete metadata
- [ ] Passes ansible-lint
- [ ] Includes Molecule tests
- [ ] Validates target_hosts
- [ ] Uses proper privilege escalation
- [ ] Handles errors gracefully
- [ ] Documents thoroughly

### 12.3 Review Criteria

**Technical Review**:
- Security best practices
- Code quality and readability
- Test coverage and validation
- Performance considerations

**Functional Review**:
- Meets stated objectives
- Handles edge cases
- Provides proper error messages
- Supports all execution modes

---

## Compliance and Framework Mapping

### 13.1 Compliance Tagging

**Tag Format**: `control:[FRAMEWORK]-[CONTROL_ID]`

**Examples**:
- `control:CIS-5.3.1`
- `control:NIST-AC-02`
- `control:STIG-SRG-OS-000480`

### 13.2 Framework Integration

**Supported Frameworks**:
- CIS (Center for Internet Security)
- NIST (National Institute of Standards and Technology)
- STIG (Security Technical Implementation Guide)
- DISA (Defense Information Systems Agency)

### 13.3 Compliance Tagging Usage

**In Playbooks**:
```yaml
- name: "Ensure | SSH Protocol | is set to 2"
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^#?Protocol'
    line: 'Protocol 2'
    state: present
    validate: 'sshd -t -f %s'
  become: true
  tags:
    - ssh
    - security
    - "control:CIS-5.3.1"
  notify: restart sshd
```

**For Execution**:
```bash
# Run only specific controls
ansible-playbook hardening.yml --tags "control:CIS-5.3.1"

# Skip certain controls
ansible-playbook hardening.yml --skip-tags "control:CIS-5.3.2"
```

---

## Version Control and Release Management

### 14.1 Versioning Strategy

**Semantic Versioning**: `MAJOR.MINOR.PATCH`

- **MAJOR**: Breaking changes or major feature additions
- **MINOR**: New features or significant improvements
- **PATCH**: Bug fixes and minor improvements

### 14.2 Change Documentation

**Required for Each Release**:
- Changelog entries
- Migration notes (if applicable)
- Testing validation
- Compliance impact assessment

### 14.3 Release Process

1. **Development**: Create feature branch from `contrib`
2. **Testing**: Validate against all supported platforms
3. **Review**: Technical and security review
4. **Integration**: Merge to `contrib` branch
5. **Validation**: Automated and manual testing
6. **Release**: Merge to `main` branch with proper tagging

---

## Conclusion

These standards ensure that all Orion's Belt playbooks are:
- **Secure**: Following security best practices
- **Maintainable**: Well-documented and structured
- **Testable**: Comprehensive test coverage
- **Compliant**: Supporting security frameworks
- **Compatible**: Working across all execution modes

Adherence to these standards is mandatory for all playbook contributions and will be enforced through automated checks and code review processes.

---

## References

- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Ansible Security Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#security-best-practices)
- [Molecule Documentation](https://molecule.readthedocs.io/)
- [Ansible Vault Documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
- [CIS Benchmarks](https://www.cisecurity.org/benchmarks/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)
