# How to Contribute to Orion's Belt

Thank you for your interest in contributing to Orion's Belt! We are excited to build a strong community around this project and welcome your contributions.

### Our Contribution Workflow

We've designed our contribution workflow to be transparent and to ensure every script in Orion's Belt is robust, secure, and well-documented. Your authorship is preserved and credited at every step.

1.  **Open an Issue:** Before starting, please [open an issue](https://github.com/IncudoLABS/orions-belt/issues) to discuss your idea. This helps us prevent duplicate work and ensure your contribution aligns with the project's goals.

2.  **Fork & Create a Feature Branch:**
    *   Fork the repository to your own GitHub account.
    *   Create your new feature branch **based on our `contrib` branch**. This is crucial as `contrib` contains the latest community-submitted code.
    ```bash
    # From your local fork, get the latest changes from the upstream remote
    git fetch upstream
    # Create your new branch based on the upstream contrib branch
    git checkout -b your-feature-name upstream/contrib
    ```

3.  **Develop Your Script:** Follow the comprehensive instructions in our **[Script Management Guide](docs/public-orions-belt-script-management-guide.md)** and **[Playbook Standards](docs/PLAYBOOK_STANDARDS.md)**. Pay close attention to:
    *   **File Naming:** Use the `OB-[ID]_[description].yml` format (e.g., `OB-004_secure_ssh.yml`).
    *   **Metadata:** Provide all required metadata fields as specified in the standards.
    *   **Security:** Use `hosts: "{{ target_hosts }}"` and avoid hardcoded secrets.
    *   **Testing:** Include Molecule tests for your playbook where applicable.

4.  **Submit a Pull Request:** Submit your Pull Request (PR) to our **`contrib`** branch.

### Automated Quality Checks

When you submit a PR, our automated workflow will run several checks to ensure your playbook meets our standards:

**Basic Validation:**
- ✅ **YAML Syntax**: Ensures your playbook has valid YAML syntax
- ✅ **File Naming**: Verifies your playbook follows the `OB-[ID]_[description].yml` naming convention
- ✅ **Security Standards**: Checks for proper `target_hosts` usage and no hardcoded secrets

**Ansible Linting:**
- ✅ **Code Quality**: Runs `ansible-lint` to check for best practices and potential issues
- ✅ **Standards Compliance**: Ensures your playbook follows Ansible and project standards

**What Happens If Checks Fail:**
- The PR will be blocked from merging until all issues are resolved
- You'll receive clear error messages explaining what needs to be fixed
- You can push additional commits to fix issues and re-trigger the checks

**Getting Help:**
- Review the [Playbook Standards](docs/PLAYBOOK_STANDARDS.md) for detailed requirements
- Check the [Template Examples](templates/) for working examples
- Open an issue if you need help understanding or fixing any checks

### The Review and Integration Process

Your contribution will go through a multi-stage review process before being published in a final release.

**Stage 1: Public Review (in the `contrib` branch)**

First, your PR will be reviewed publicly by maintainers and community members. We'll focus on:
*   Functionality and code quality.
*   Adherence to the script management guide.
*   General feasibility and alignment with the project's goals.

**Stage 2: Internal Vetting & Enrichment**

Once merged into `contrib`, your script enters our internal pipeline. This is a critical step where we ensure it meets the rigorous quality and security standards required for enterprise environments. This process involves:

*   **Extensive Testing:** We run a suite of automated tests against a wide array of operating systems and configurations that go beyond standard CI checks.
*   **Compliance Alignment & Metadata Enrichment:** We validate the script against the requirements of numerous security frameworks. This involves an enrichment process where we map the script to a comprehensive set of controls, enhancing its value and applicability.

This internal process ensures your contribution is not only valuable to the open-source community but is also of a professional, enterprise-ready standard.

**Stage 3: Publication to `main`**

After passing our internal quality and security gates, your contribution, with your original authorship intact, is merged into the `main` branch and becomes part of an official, stable release of Orion's Belt.

We look forward to collaborating with you!