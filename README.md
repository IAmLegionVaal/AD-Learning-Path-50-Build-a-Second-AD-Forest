# AD Learning Path 50 — Build a Second AD Forest

## Objective
Create an independent second forest named `partner.lab` for cross-forest DNS and trust testing without extending the existing `corp.lab` forest.

## Prerequisites
- Separate VM `PDC01`
- Static address such as `10.20.20.10`
- Isolated routed lab connectivity
- No DNS namespace conflict
- Independent administrator and DSRM credentials

## Setup
1. Install and patch Windows Server on `PDC01`.
2. Configure its static address and confirm the new subnet does not overlap existing networks.
3. Install the AD DS role and management tools.
4. Create a new forest `partner.lab` with NetBIOS name `PARTNER` and DNS installed.
5. After restart, configure `PDC01` to use its internal DNS service.
6. Validate the forest independently before configuring any trust.
7. Create a normal test user and security group for later cross-forest access tests.

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
$DSRM = Read-Host 'Unique PARTNER forest DSRM password' -AsSecureString
Install-ADDSForest -DomainName 'partner.lab' -DomainNetbiosName 'PARTNER' `
    -InstallDns -SafeModeAdministratorPassword $DSRM -Force
```

## Validation
```powershell
Get-ADForest | Format-List RootDomain,Domains,GlobalCatalogs,ForestMode
Get-ADDomain | Format-List DNSRoot,NetBIOSName,PDCEmulator,DomainMode
dcdiag.exe /v
Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.partner.lab
Get-SmbShare -Name SYSVOL,NETLOGON
```

## Evidence
Store the second-forest topology, static addressing, forest/domain properties, `dcdiag`, DNS SRV results, SYSVOL/NETLOGON checks, independent credential handling, test identities, errors/remediation, and final pass/fail status under `evidence/`.

## Troubleshooting
- Commands return the wrong forest: run them on `PDC01` or specify the target server.
- DNS namespace conflict: redesign before promotion.
- Cross-subnet connectivity fails: fix lab routing and firewall without bridging to production.

## Security notes
A forest is an administrative and security boundary. Keep administrator, DSRM, and service credentials separate between forests.

## Cleanup
Keep the second forest until trust and recovery activities are complete. Demote or destroy it only through a documented lab cleanup.

## References
- Microsoft Learn: Create a forest design
- Microsoft Learn: Deploy a new forest

## Next activity
`AD-Learning-Path-51-Configure-and-Validate-a-Forest-Trust`
