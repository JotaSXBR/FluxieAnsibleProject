# TODO - Future Enhancements & Project Roadmap

This document outlines potential future enhancements, features, and areas for improvement for this Ansible project. Contributions are welcome!

## 🚀 Features & Functionality

*   **[ ] Add More Services:**
    *   **[ ] Loki:** For log aggregation and querying, integrated with Grafana.
    *   **[ ] Alertmanager:** For handling alerts from Prometheus.
    *   **[ ] Other application examples:** Showcase how to add a generic user application behind Traefik.
*   **[ ] Enhanced Service Version Management:**
    *   **[ ] Dedicated Variables File:** Move service versions to a separate `versions.yml` or similar for easier updates.
    *   **[ ] Fetch Latest Stable Tags:** Optionally add logic to fetch latest stable tags for services (requires careful consideration for stability).
*   **[ ] Alternative Configurations/Profiles:**
    *   **[ ] Service Toggles:** Allow users to easily enable/disable deployment of specific services (e.g., Jaeger, Portainer) via Ansible variables.
    *   **[ ] Network Configuration Options:** Offer more flexibility in Docker network setup if needed.
*   **[ ] Backup and Restore Strategy:**
    *   **[ ] Implement Backup Scripts/Role:** Add Ansible tasks or scripts for backing up persistent volumes (Prometheus data, Grafana DB, Portainer data, Traefik certs).
    *   **[ ] Document Restore Process:** Clearly outline steps for restoring from backups.
*   **[ ] User-Defined Application Integration:**
    *   **[ ] Clear Documentation/Example:** Provide a step-by-step guide or a sample role for users to add their own Dockerized applications to be managed by Traefik.

## 🛡️ Security Improvements

*   **[ ] Ansible Vault for Secrets:**
    *   **[ ] Traefik Basic Auth:** Encrypt the `traefik-auth` basic authentication credentials.
    *   **[ ] Grafana Admin Password:** Manage the initial Grafana admin password securely.
    *   **[ ] Other Future Secrets:** Establish a pattern for all sensitive variables.
*   **[ ] Secure Grafana Initialization:**
    *   **[ ] Random Initial Admin Password:** Investigate setting a secure, random initial admin password for Grafana and outputting it to the user.
    *   **[ ] Prompt for Grafana Admin Password:** Alternatively, prompt the user during playbook execution.
*   **[ ] Two-Factor Authentication (2FA):**
    *   **[ ] SSH 2FA:** Research and optionally implement 2FA for SSH access (e.g., using `google-authenticator`).
    *   **[ ] Service 2FA:** Explore 2FA options for Traefik dashboard, Grafana, Portainer (if supported natively or via auth proxies).
*   **[ ] Integrate Security Scanning/Auditing:**
    *   **[ ] Lynis:** Add a role or task to run Lynis for system auditing and report findings.
    *   **[ ] Docker Image Vulnerability Scanning:** Integrate a tool to scan Docker images for known vulnerabilities.
*   **[ ] Extend Fail2Ban:**
    *   **[ ] Traefik/Web Services:** Investigate configuring Fail2Ban for web services if applicable (e.g., if custom login pages are added).
*   **[ ] Review/Refine UFW Rules:** Ensure UFW rules are optimal and provide guidance for users adding new applications.
*   **[ ] Immutable Docker Tags:** Encourage or enforce the use of immutable Docker image tags (e.g., `image:tag@sha256:...`) for better security and reproducibility, perhaps as an option.

## ⚙️ Operations & Maintenance

*   **[ ] Automated Testing:**
    *   **[ ] Ansible Linting:** Integrate `ansible-lint` into a CI/CD pipeline or pre-commit hook.
    *   **[ ] Molecule Testing:** Develop Molecule tests for each Ansible role (`common`, `docker`, `observability`) to ensure they function correctly and are idempotent.
*   **[ ] Enhanced Logging Strategy:**
    *   **[ ] Docker Log Configuration:** Provide more advanced options for Docker log drivers and rotation within `daemon.json`.
    *   **[ ] Centralized Logging (Loki):** Implement Loki as mentioned in Features.
*   **[ ] Container Resource Management:**
    *   **[ ] CPU/Memory Limits:** Expose variables in `docker-compose.yml.j2` to allow users to set CPU and memory limits/reservations for services.
*   **[ ] Idempotency Review:** Thoroughly review all Ansible tasks to confirm they are idempotent.
*   **[ ] Docker Compose Health Checks:** Add or improve health checks in `docker-compose.yml.j2` for all services.
*   **[ ] Graceful Service Updates:** Ensure Docker Compose updates services gracefully, minimizing downtime (already good, but worth a check for complex stateful services).

## 📖 Documentation

*   **[ ] Create Visual Architecture Diagram:** Generate the architecture diagram suggested in `README.md`.
*   **[ ] Role-Specific READMEs:**
    *   **[ ] `roles/common/README.md`**
    *   **[ ] `roles/docker/README.md`**
    *   **[ ] `roles/observability/README.md`**
    *   Detail variables, tasks, dependencies, and purpose for each role.
*   **[ ] Troubleshooting Guide:**
    *   **[ ] `TROUBLESHOOTING.md`:** Create a dedicated file or section in the main README for common issues, error messages, and their solutions.
*   **[ ] Detailed Configuration Guides:**
    *   **[ ] Advanced Traefik:** Explain how to add custom middleware, routers, services beyond the basics.
    *   **[ ] Grafana Provisioning:** Detail how to add more datasources or dashboards via provisioning.
    *   **[ ] Prometheus Scraping:** Explain how to add new scrape targets to Prometheus.
*   **[ ] In-Depth Security Explanation:** Expand the "Security Considerations" section in the README with more details on *why* certain measures are taken.
*   **[ ] Contribution Guidelines:** If the project grows, create a `CONTRIBUTING.md` with more detailed instructions for contributors.

## ✨ Refactoring & Cleanup

*   **[ ] Consistent Variable Naming:** Review all Ansible variables for consistent naming conventions (e.g., `project_service_variable_name`).
*   **[ ] Template/File Paths in Roles:**
    *   **[ ] Grafana Assets:** Move the `grafana/` directory (copied by the `observability` role) into `roles/observability/files/grafana/` for better role encapsulation. Update the task accordingly.
    *   **[ ] Review Other Paths:** Check other template/copy tasks for optimal path usage.
*   **[ ] Review Docker Network Strategy:** Ensure `traefik-public` and `monitor-net` are optimally configured and documented.
*   **[ ] Tagging Strategy for Ansible Plays/Tasks:** Review and refine the use of tags for more granular playbook execution.
*   **[ ] Remove `ansible_distribution_release` for Docker Repo:** The Docker installation uses `{{ ansible_distribution_release }}` (e.g., `bookworm`, `jammy`). While this usually works, Docker's repositories are typically based on Ubuntu LTS names (`jammy`, `focal`). For Debian, it might be better to map `ansible_distribution_codename` to the closest Ubuntu LTS or ensure the specific Debian codename is supported by Docker's repos. *Self-correction: The current approach is generally fine as Docker's script usually handles this, but worth a double check for robustness across many Debian/Ubuntu versions.*

---
This list is a starting point and can be expanded or reprioritized as the project evolves.
