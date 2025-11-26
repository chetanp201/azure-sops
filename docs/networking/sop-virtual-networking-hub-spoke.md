# SOP – Azure Virtual Networking (Hub-Spoke Topology)  
**Version:** 1.0 | **Valid as of:** Nov 2025  
**Owner:** Azure Networking CoE | **Review cycle:** Every 6 months

!!! info "Quick Links"
    [:material-pencil: Edit this SOP](https://github.com/chetanp201/azure-sops/edit/main/docs/networking/sop-virtual-networking-hub-spoke.md) | 
    [:material-clock-outline: Last Updated](#){ .git-revision-date-localized-plugin }

## Objective  
Deploy a production-grade, CAF/WAF-aligned Hub-Spoke topology that is repeatable, secure-by-default, and policy-enforced in every customer landing zone.

## Scope  
Mandatory for ALL new Azure environments and any brownfield refactoring.

## Prerequisites  
- Subscription placed under correct Management Group (Connectivity or Landing Zones)  
- Azure Policy “Deploy Landing Zone – Networking” initiative assigned  
- Engineer has Contributor + User Access Administrator on the subscription  
- IaC templates from official repo (Bicep or Terraform)

## Step-by-Step Deployment

| Step | Action | Tool | Key Details / Success Criteria |
|------|--------|------|-------------------------------|
| 1 | Deploy Hub VNet | Bicep/Terraform module | Address space: e.g., 10.100.0.0/22<br>Subnets: AzureBastion, AzureFirewall, GatewaySubnet (optional), JumpBox |
| 2 | Deploy Azure Firewall Premium + Firewall buổi Policy | Same module | Firewall Healthy, default DNAT/SNAT/App rules in place |
| 3 | Enable DDoS Standard Protection Plan | Portal or IaC | Plan linked to Hub VNet |
| 4 | Deploy Spoke VNet(s) | Spoke module | Minimum subnets: Application (/24), Database (/24), PrivateEndpoints (/26) |
| 5 | VNet Peering (bidirectional) | Automatic in module | Status = Connected both directions<br>Allow forwarded traffic = Yes<br>Use remote gateways = Yes (hub side) |
| 6 | Route Tables & UDRs | Automatic | 0.0.0.0/0 → AzureFirewall private IP on all spoke subnets<br>RFC1918 propagated if ER/VPN present |
| 7 | Private DNS Resolver (or DNS forwarders) in Hub | Module | All spokes use Hub resolver |
| 8 | Deploy mandatory privatelink.* DNS zones & VNet links | Module | Zones linked to Hub + all Spokes |
| 9 | Run Azure Policy compliance check | Portal | 100% compliant with Networking initiative |
|10| Validate effective routes & NSG flow | Network Watcher | Traffic to internet → Firewall, PaaS → Private Endpoint |

## Mandatory Naming Convention
vnet-<env>-hub-<region>-001
vnet-<env>-spoke-<workload>-<region>-001
fw-<env>-hub-<region>-001
rt-<env>-spoke-<workload>-default-001


## Post-Deployment Checklist (copy into ticket)
- [ ] Hub & Spoke VNets created  
- [ ] Azure Firewall Healthy  
- [ ] Peerings Connected both directions  
- [ ] 0.0.0.0/0 → Firewall on all spokes  
- [ ] Private DNS zones linked to all VNets  
- [ ] Azure Policy = 100% compliant  
- [ ] Topology diagram updated in repo  
- [ ] Customer sign-off obtained

## Approved Variations (need architect approval)
| Scenario                     | Allowed? | Alternative |
|------------------------------|----------|------------|
| Third-party NVA (Palo Alto, etc.) | Yes   | Replace AzFW module |
| Multi-region connectivity    | Yes      | Global peering + regional firewalls |
| Existing ExpressRoute/VPN    | Yes      | Add GatewaySubnet + BGP propagation |

## Rollback Steps
1. Delete workload resources in spokes  
2. Remove spoke → hub peering  
3. Delete spoke VNet  
4. (Full rollback) Delete subscription-level deployment
