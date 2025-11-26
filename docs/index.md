# Azure Professional Services SOP Portal

!!! success "Your single source of truth for Azure deployment procedures"

Welcome to the **Azure Professional Services Standard Operating Procedures (SOP) Portal**. This portal provides production-grade, repeatable procedures for deploying and managing Azure infrastructure.

---

## 🔍 Quick Search

Use the search bar at the top to quickly find SOPs by keyword, technology, or use case.

---

## 📋 Active SOPs

### Networking
{: .grid-list }

- [:material-network: **Virtual Networking – Hub & Spoke**](networking/sop-virtual-networking-hub-spoke.md)
  - Deploy production-grade Hub-Spoke topology with Azure Firewall
  - CAF/WAF-aligned, policy-enforced
  - *Last Updated: Nov 2025*

### AVD (Azure Virtual Desktop)
{: .grid-list }

- [:material-monitor: **AVD Host Pool + FSLogix**](avd/sop-avd-host-pool-fslogix.md) {: .new }
  - Complete AVD deployment with FSLogix profile containers
  - High availability and performance optimization
  - *Last Updated: Nov 2025*

---

## 🆕 Recently Added

!!! tip "Latest Updates"
    - **Nov 2025**: Added AVD Host Pool + FSLogix SOP
    - **Nov 2025**: Initial release with Hub-Spoke Networking SOP

---

## 📊 SOP Status

| Category | Count | Status |
|----------|-------|--------|
| Networking | 1 | ✅ Active |
| AVD | 1 | ✅ Active |
| Security | 0 | 🚧 Coming Soon |
| Landing Zones | 0 | 🚧 Coming Soon |
| Governance | 0 | 🚧 Coming Soon |
| Cost Optimization | 0 | 🚧 Coming Soon |

---

## 🎯 Design Principles

Every SOP in this portal follows these principles:

- ✅ **Production-Grade**: Tested and validated in real customer environments
- ✅ **Repeatable**: Step-by-step procedures with clear success criteria
- ✅ **Secure-by-Default**: Security best practices built-in
- ✅ **Policy-Enforced**: Aligned with Azure Policy and CAF/WAF
- ✅ **Well-Documented**: Includes naming conventions, checklists, and rollback procedures

---

## 📚 SOP Template Structure

All SOPs include:

1. **Header**: Version, Owner, Review cycle
2. **Objective & Scope**: Clear purpose and applicability
3. **Prerequisites**: Required permissions, resources, and setup
4. **Step-by-Step Deployment**: Detailed table with actions, tools, and success criteria
5. **Naming Conventions**: Mandatory naming patterns
6. **Post-Deployment Checklist**: Copy-paste ready validation checklist
7. **Approved Variations**: Scenarios requiring architect approval
8. **Rollback Steps**: Safe rollback procedures
9. **References**: Links to official documentation

---

## 🤝 Contributing

To suggest improvements or report issues:

1. Click the **Edit** link on any SOP page
2. Make your changes
3. Submit a pull request

All changes are reviewed by the respective CoE (Center of Excellence) owners.

---

## 📞 Support

For questions or clarifications about any SOP:

- Open an issue in the [GitHub repository](https://github.com/chetanp201/azure-sops/issues)
- Contact the SOP owner listed in each document
- Reach out to the Azure Professional Services team

---

!!! info "Portal Information"
    - **Repository**: [chetanp201/azure-sops](https://github.com/chetanp201/azure-sops)
    - **Live Site**: [chetanp201.github.io/azure-sops](https://chetanp201.github.io/azure-sops)
    - **Last Updated**: Nov 2025
