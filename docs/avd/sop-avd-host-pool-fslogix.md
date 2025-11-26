# SOP – Azure Virtual Desktop (AVD) Host Pool + FSLogix Profile Container
**Version:** 1.0 | **Valid as of:** Nov 2025  
**Owner:** Azure AVD CoE | **Review cycle:** Every 6 months

!!! info "Quick Links"
    [:material-pencil: Edit this SOP](https://github.com/chetanp201/azure-sops/edit/main/docs/avd/sop-avd-host-pool-fslogix.md) | 
    [:material-clock-outline: Last Updated](#){ .git-revision-date-localized-plugin }

## Objective  
Deploy a production-grade Azure Virtual Desktop (AVD) host pool with FSLogix profile containers, ensuring high availability, performance, and security for enterprise virtual desktop infrastructure.

## Scope  
Mandatory for ALL new AVD deployments. Applies to pooled and personal host pools. Includes FSLogix profile container setup with Azure Files Premium.

## Prerequisites  
- Azure subscription with appropriate permissions (Contributor + User Access Administrator)
- Azure AD (Entra ID) tenant with AVD licensing (Windows 10/11 Enterprise E3/E5 or Microsoft 365)
- Network connectivity (Hub-Spoke topology recommended per Networking SOP)
- Storage account with Premium File Shares enabled
- Domain controller accessible (on-premises or Azure AD DS)
- AVD service principal or user with "Desktop Virtualization Contributor" role

## Step-by-Step Deployment

| Step | Action | Tool | Key Details / Success Criteria |
|------|--------|------|-------------------------------|
| 1 | Create Resource Group | Portal/CLI/PowerShell | Naming: `rg-<env>-avd-<region>-001` |
| 2 | Deploy Azure Files Premium Account | Portal or Bicep/Terraform | Performance: Premium<br>Redundancy: ZRS (recommended) or LRS<br>Enable large file shares |
| 3 | Create File Share for FSLogix Profiles | Azure Portal | Share name: `fslogix-profiles`<br>Quota: Based on user count (e.g., 5TB for 500 users) |
| 4 | Configure Azure Files Active Directory Authentication | Azure AD DS or On-Prem AD | Domain join storage account<br>Assign share permissions (Storage File Data SMB Share Elevated Contributor) |
| 5 | Create AVD Workspace | Portal/CLI/PowerShell | Naming: `wkspc-<env>-avd-<region>-001`<br>Location: Same region as host pool |
| 6 | Create AVD Host Pool | Portal/CLI/PowerShell | Pool type: Pooled or Personal<br>Load balancing: Breadth-first or Depth-first<br>Max session limit: 10 (pooled) or 1 (personal) |
| 7 | Register Session Host VMs | ARM Template/Bicep/Terraform | Windows 10/11 Enterprise multi-session or Windows Server 2022<br>Domain join required<br>Install AVD agent + bootloader |
| 8 | Configure FSLogix on Session Hosts | Group Policy or Registry | Enable FSLogix Profile Container<br>Set VHDLocations = `\\<storage-account>.file.core.windows.net\fslogix-profiles`<br>Set VHDNamePattern = `%username%` |
| 9 | Configure FSLogix Cloud Cache (Optional) | Registry/Group Policy | Enable for HA scenarios<br>Primary: Azure Files<br>Secondary: Azure Files (different region) or Azure Blob |
| 10 | Assign Users/Groups to Application Group | Portal/PowerShell | Create Desktop Application Group<br>Assign Entra ID users or groups<br>Validate user can access AVD feed |
| 11 | Configure Network Security | NSG/Azure Firewall | Allow RDP (3389) from authorized IPs<br>Allow SMB (445) to Azure Files<br>Block internet access (use Azure Firewall) |
| 12 | Enable Monitoring & Diagnostics | Log Analytics Workspace | Enable diagnostic settings on host pool<br>Configure alerts for host health, session count, errors |
| 13 | Configure Scaling Plan (Pooled) | Portal/PowerShell | Peak hours: Start VMs 30 min before<br>Off-peak: Shutdown after 10 min idle<br>Min/Max hosts based on capacity |
| 14 | Validate User Profile Persistence | Test login | Log in → Create files on desktop → Log out → Log in → Verify files persist |
| 15 | Run Post-Deployment Validation | PowerShell/Portal | All session hosts "Available"<br>Users can connect<br>FSLogix profiles created in Azure Files<br>No errors in Event Viewer |

## Mandatory Naming Convention
```
rg-<env>-avd-<region>-001
wkspc-<env>-avd-<region>-001
hp-<env>-avd-<pool-type>-<region>-001
ag-<env>-avd-desktop-<region>-001
sa<env>avd<region>001 (storage account, no hyphens)
fslogix-profiles (file share name)
```

## Post-Deployment Checklist (copy into ticket)
- [ ] Resource Group created with correct naming
- [ ] Azure Files Premium account deployed (ZRS recommended)
- [ ] File share created with appropriate quota
- [ ] Azure Files AD authentication configured
- [ ] AVD Workspace created
- [ ] AVD Host Pool created (pooled or personal)
- [ ] Session Host VMs deployed and registered (minimum 2 for HA)
- [ ] FSLogix installed and configured on all session hosts
- [ ] FSLogix registry keys set (VHDLocations, VHDNamePattern)
- [ ] Users/groups assigned to Application Group
- [ ] Network security rules configured (NSG/Azure Firewall)
- [ ] Log Analytics workspace connected
- [ ] Scaling plan configured (if pooled)
- [ ] User profile persistence validated
- [ ] Monitoring alerts configured
- [ ] Documentation updated
- [ ] Customer sign-off obtained

## Approved Variations (need architect approval)

| Scenario | Allowed? | Alternative |
|----------|----------|------------|
| Azure NetApp Files for profiles | Yes | Higher performance, higher cost. Use for >1000 users |
| FSLogix without Cloud Cache | Yes | Single region only. Not recommended for HA |
| Personal host pools | Yes | 1:1 user-to-VM mapping. Higher cost, better isolation |
| Azure AD Domain Services (AAD DS) | Yes | If no on-premises AD. Limited to 5,000 objects |
| Azure Files Standard (not Premium) | No | Insufficient IOPS for profile containers |
| Third-party profile solutions (Citrix, etc.) | Yes | Requires architect approval and migration plan |

## FSLogix Configuration Details

### Registry Settings (Group Policy Recommended)
```
HKLM\SOFTWARE\FSLogix\Profiles
  Enabled = 1 (DWORD)
  VHDLocations = \\<storage-account>.file.core.windows.net\fslogix-profiles
  VHDNamePattern = %username%
  DeleteLocalProfileWhenVHDShouldApply = 1
  FlipFlopProfileDirectoryName = 1
  PreventLoginWithFailure = 1
  ProfileType = 0 (0=Profile, 1=ODFC, 2=Both)
```

### Cloud Cache Configuration (HA Scenarios)
```
HKLM\SOFTWARE\FSLogix\Profiles
  CCEnalbed = 1
  CCDLocations = type=Azure,connectionString="DefaultEndpointsProtocol=https;AccountName=...;EndpointSuffix=core.windows.net;"
```

## Performance Optimization
- **Azure Files Premium**: Minimum 100 GiB (provides baseline IOPS). Scale up for more IOPS
- **VHD Size**: Start with 30GB per user, monitor and adjust
- **Antivirus Exclusions**: Exclude FSLogix VHD files and folders on session hosts
- **Network**: Ensure low latency between session hosts and Azure Files (<5ms recommended)

## Rollback Steps
1. Remove user assignments from Application Group
2. Stop all session host VMs
3. Backup FSLogix profiles from Azure Files (if needed)
4. Delete Application Group
5. Delete Host Pool
6. Delete Workspace
7. (Optional) Delete Azure Files share and storage account
8. Delete Resource Group

## Troubleshooting

### Common Issues
- **Users cannot connect**: Check Application Group assignments, network connectivity, host pool status
- **Profiles not persisting**: Verify FSLogix registry settings, Azure Files permissions, network connectivity
- **Slow logon**: Check Azure Files IOPS, VHD size, network latency
- **Session hosts unavailable**: Check VM status, domain join, AVD agent health

### Diagnostic Commands
```powershell
# Check FSLogix status
Get-ItemProperty -Path "HKLM:\SOFTWARE\FSLogix\Profiles"

# Check Azure Files connectivity
Test-NetConnection -ComputerName <storage-account>.file.core.windows.net -Port 445

# Check AVD host pool status
Get-AzWvdHostPool -ResourceGroupName <rg-name> -Name <host-pool-name>
```

## References
- [Azure Virtual Desktop Documentation](https://docs.microsoft.com/azure/virtual-desktop/)
- [FSLogix Documentation](https://docs.microsoft.com/fslogix/)
- [Azure Files Performance](https://docs.microsoft.com/azure/storage/files/storage-files-scale-targets)
- [AVD Architecture Best Practices](https://docs.microsoft.com/azure/virtual-desktop/architecture-recommendations)

