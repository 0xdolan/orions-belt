# Orion's Belt Playbook Templates

This directory contains example playbooks and roles that demonstrate the [Orion's Belt Playbook Standards](../docs/PLAYBOOK_STANDARDS.md) in practice. These templates serve as starting points for developers and community contributors.

## Template Overview

### Simple Example Playbook
**File:** `OB-TPL-001_simple_example.yml`

A single-file playbook that demonstrates:
- ✅ Proper metadata structure (`orion_metadata`)
- ✅ Dynamic host targeting (`hosts: "{{ target_hosts }}"`)
- ✅ Input validation and security checks
- ✅ Block/rescue/always error handling
- ✅ Variable management with defaults
- ✅ File operations with backup
- ✅ Comprehensive logging

**Usage:**
```bash
# Standalone execution
ansible-playbook -i inventory OB-TPL-001_simple_example.yml -e "target_hosts=web_servers"

# Using ob.sh
./ob.sh OB-TPL-001_simple_example.yml web_servers

# Via Forge UI
# Select this playbook and target hosts through the interface
```

### Role Example Playbook
**File:** `OB-TPL-002_role_example.yml`

A playbook that demonstrates role usage with:
- ✅ Role-based architecture
- ✅ Variable passing to roles
- ✅ Integration with vault variables
- ✅ Service management patterns

**Role Structure:** `roles/example_role/`
- `tasks/main.yml` - Main role tasks with error handling
- `handlers/main.yml` - Service restart/reload handlers
- `defaults/main.yml` - Default variable values
- `vars/main.yml` - Role-specific variables
- `vars/vault.yml` - Encrypted sensitive variables
- `templates/service.conf.j2` - Jinja2 configuration template

## Standards Compliance

Both templates fully comply with the Orion's Belt Playbook Standards:

### Security Standards ✅
- Dynamic host targeting with validation
- No hardcoded secrets (uses vault)
- Proper file permissions
- Input sanitization

### Error Handling ✅
- Block/rescue/always structure
- Comprehensive error logging
- Cleanup on failure
- State control with `changed_when`/`failed_when`

### Variable Management ✅
- Clear variable precedence
- Default values for optional variables
- Vault integration for secrets
- Input validation

### Documentation ✅
- Comprehensive headers
- Usage examples
- Clear variable documentation
- Attribution references

## Getting Started

1. **Review the Standards:** Read [PLAYBOOK_STANDARDS.md](../docs/PLAYBOOK_STANDARDS.md) to understand the requirements.

2. **Choose a Template:**
   - Use `OB-TPL-001_simple_example.yml` for simple, single-file playbooks
   - Use `OB-TPL-002_role_example.yml` and `roles/example_role/` for complex, reusable automation

3. **Customize for Your Needs:**
   - Copy the template files
   - Update metadata and variables
   - Modify tasks for your specific requirements
   - Add your own vault variables

4. **Test Your Playbook:**
   - Run with `--check` mode first
   - Test on a development environment
   - Verify idempotence (second run should show no changes)

## Template Customization

### For Simple Playbooks
1. Copy `OB-TPL-001_simple_example.yml`
2. Update the `orion_metadata` section
3. Modify the `tasks` section for your specific automation
4. Update variable definitions and validation

### For Role-Based Playbooks
1. Copy `OB-TPL-002_role_example.yml` and the `roles/example_role/` directory
2. Rename the role directory to match your use case
3. Update role variables in `defaults/main.yml`
4. Modify tasks in `tasks/main.yml`
5. Add your own templates and handlers as needed

## Vault Integration

The role example demonstrates secure vault usage:

1. **Enable vault variables:**
   ```yaml
   vars:
     vault_vars: true
   ```

2. **Create real encrypted variables:**
   ```bash
   # Secure method (no bash history)
   read -s -p "Enter password: " password
   echo "$password" | ansible-vault encrypt_string --stdin-name 'db_password' -
   unset password
   ```

3. **Replace placeholder values in `vars/vault.yml`**

## Testing

Both templates include comprehensive error handling and logging. For complete testing with Molecule, see the [Molecule Test Example](OB-TPL-001_simple_example/README.md) for `OB-TPL-001_simple_example.yml`.

### Basic Testing

1. **Syntax check:**
   ```bash
   ansible-playbook --syntax-check OB-TPL-001_simple_example.yml
   ```

2. **Dry run:**
   ```bash
   ansible-playbook --check OB-TPL-001_simple_example.yml -e "target_hosts=localhost"
   ```

3. **Linting:**
   ```bash
   ansible-lint OB-TPL-001_simple_example.yml
   ```

### Molecule Testing

For comprehensive testing following the [Playbook Testing Standards](../docs/PLAYBOOK_STANDARDS.md):

```bash
# Navigate to the test directory
cd OB-TPL-001_simple_example

# Run complete test suite
molecule test -s default

# Run check mode tests
molecule test -s check_mode

# Run both scenarios
molecule test --all
```

The Molecule test suite includes:
- ✅ Syntax validation
- ✅ Linting with ansible-lint
- ✅ Playbook execution
- ✅ Idempotence verification
- ✅ Standards compliance checks
- ✅ Security verification
- ✅ Functionality verification

## Contributing

When creating new templates:

1. Follow the established naming convention (`OB-TPL-[ID]_[description].yml`)
2. Ensure full compliance with [PLAYBOOK_STANDARDS.md](../docs/PLAYBOOK_STANDARDS.md)
3. Include comprehensive documentation
4. Test thoroughly before submission
5. Update this README with new template information

## Support

For questions about these templates or the playbook standards:
- Review [PLAYBOOK_STANDARDS.md](../docs/PLAYBOOK_STANDARDS.md)
- Check the [CONTRIBUTING.md](../CONTRIBUTING.md) guide
- Open an issue in the Orion's Belt repository

---

**Note:** These templates are examples and should be customized for your specific use cases. Always test in a safe environment before deploying to production.
