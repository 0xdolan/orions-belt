# Molecule Test Example for OB-TPL-001

This directory demonstrates a complete Molecule test suite for the `OB-TPL-001_simple_example.yml` playbook, following the [Orion's Belt Playbook Testing Standards](../../docs/PLAYBOOK_STANDARDS.md).

## Directory Structure

```
OB-TPL-001_simple_example/
├── OB-TPL-001_simple_example.yml    # Main playbook
├── molecule/
│   ├── default/                      # Default test scenario
│   │   ├── molecule.yml             # Main configuration
│   │   ├── converge.yml             # Playbook execution
│   │   └── verify.yml               # Verification tests
│   └── check_mode/                  # Check mode scenario
│       ├── molecule.yml             # Check mode configuration
│       └── converge.yml             # Check mode execution
└── README.md                        # This file
```

## Test Scenarios

### Default Scenario
**Purpose**: Full lifecycle testing including lint, converge, idempotence, and verify
**Command**: `molecule test -s default`

**What it tests**:
- ✅ Syntax validation
- ✅ Linting with ansible-lint
- ✅ Playbook execution
- ✅ Idempotence (second run shows no changes)
- ✅ Verification of results
- ✅ Standards compliance

### Check Mode Scenario
**Purpose**: Validate dry-run functionality and syntax
**Command**: `molecule test -s check_mode`

**What it tests**:
- ✅ Check mode execution
- ✅ Syntax validation without making changes
- ✅ Variable validation

## Running the Tests

### Prerequisites
```bash
# Install Molecule and dependencies
pip install molecule[docker] ansible-core ansible-lint

# Ensure Docker is running
docker --version
```

### Full Test Suite
```bash
# Run complete test suite
molecule test -s default

# Run check mode tests
molecule test -s check_mode

# Run both scenarios
molecule test --all
```

### Individual Steps
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

## Verification Tests

The `verify.yml` playbook includes comprehensive checks:

### Security Verification
- ✅ Target hosts validation
- ✅ Secure file permissions
- ✅ No hardcoded secrets

### Functionality Verification
- ✅ File creation
- ✅ Correct content
- ✅ Backup file creation
- ✅ Directory structure

### Standards Compliance
- ✅ Naming conventions
- ✅ Metadata structure
- ✅ Error handling patterns
- ✅ Variable management

## Test Variables

The test uses these variables:
```yaml
target_hosts: all
orion_file_path: /tmp/molecule_test_file.txt
orion_file_content: "This file was created by Molecule testing"
orion_backup_enabled: true
```

## Expected Results

### Successful Test Run
```
PLAY [Converge - Execute Simple Example Playbook] *******************************
TASK [Include the simple example playbook] *************************************
PLAY [Simple Example Playbook] *************************************************
TASK [Validate target_hosts is provided] **************************************
TASK [Validate required variables] ********************************************
TASK [Create example file with content] ***************************************
TASK [Create directory structure] *********************************************
TASK [Write content to file] **************************************************
TASK [Verify file was created successfully] ***********************************
TASK [Display file information] ***********************************************
TASK [Log execution summary] **************************************************
TASK [Display completion message] *********************************************

PLAY [Verify Simple Example Playbook Execution] *******************************
TASK [Verify target_hosts variable is used] **********************************
TASK [Verify test file exists] ***********************************************
TASK [Assert test file exists] ***********************************************
TASK [Verify file permissions are secure] ************************************
TASK [Assert secure file permissions] ****************************************
TASK [Verify file content] ***************************************************
TASK [Assert correct file content] *******************************************
TASK [Check for backup files] ************************************************
TASK [Assert backup file exists] *********************************************
TASK [Check for hardcoded secrets in test file] ******************************
TASK [Assert no hardcoded secrets] *******************************************
TASK [Verify directory structure] ********************************************
TASK [Assert directory exists and is accessible] *****************************
TASK [Verify playbook follows naming conventions] ****************************
TASK [Verify metadata structure] *********************************************
TASK [Verify error handling structure] ***************************************
TASK [Verify variable management] ********************************************
```

## Troubleshooting

### Common Issues

#### Docker Permission Issues
```bash
# Fix Docker permissions
sudo chmod 666 /var/run/docker.sock

# Or use podman instead
export MOLECULE_DRIVER=podman
```

#### Container Issues
```bash
# Clean up stuck containers
molecule destroy --all

# Force recreate containers
molecule create --force
```

#### Test Failures
1. Check that the playbook syntax is correct
2. Verify all required variables are defined
3. Ensure the test environment is clean
4. Review the verification playbook for specific failure reasons

## Integration with CI/CD

This test suite can be integrated into CI/CD pipelines:

```yaml
# GitHub Actions example
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

## Standards Compliance

This test suite demonstrates compliance with:
- ✅ [Playbook Testing Standards](../../docs/PLAYBOOK_STANDARDS.md)
- ✅ [Security Standards](../../docs/PLAYBOOK_STANDARDS.md#security-standards)
- ✅ [Error Handling Standards](../../docs/PLAYBOOK_STANDARDS.md#error-handling-and-rollback)
- ✅ [Variable Management Standards](../../docs/PLAYBOOK_STANDARDS.md#variable-management-standards)

## Next Steps

1. **Customize for your playbook**: Copy this structure and modify for your specific playbook
2. **Add more test scenarios**: Create additional scenarios for different environments
3. **Enhance verification**: Add more specific checks for your playbook's functionality
4. **Integrate with CI/CD**: Add the test suite to your development workflow

---

**Note**: This is an example test suite. Modify the verification tests to match your specific playbook's expected outcomes and requirements.
