This branch is used to develop the **UEFI Redfish Feature**. The code base of development is based on the release of **edk2-stable201811** tag. The code was implemented follow below design and released in 2019.1. The follow up **UEFI Redfish POC code re-architecture** (refer to "EDK2 Redfish POC Code re-architecture Project" section) was proposed in 2019.8 in order to make edk2 Redfish support more flexible and easier to be integrated to OEM edk2 code base. The UEFI Redfish POC code re-architecture is working in progress.

The branch owner:
Fu Siyuan <siyuan.fu@intel.com>, Ye Ting <ting.ye@intel.com>, Wang Fan <fan.wang@intel.com>, Wu Jiaxin <jiaxin.wu@intel.com>, Chang Abner <abner.chang@hpe.com>, Wang Nickle <nickle.wang@hpe.com>

## Introduction
UEFI Redfish is an efficient and secure solution for end users to remote control and configure UEFI pre-OS environment by leveraging the RESTful API.  It's simple for end users to access the data from UEFI firmware defined in JSON format.

One of the design goals for UEFI Redfish solution is to provide a scalable implementation which allow users to easily add/remove/modify each independent Redfish configure features (RedfishBiosDxe & RedfishBootInfoDxe). This is done by extracting the generic logic to a single UEFI driver model driver (RedfishConfigDxe), and several library instances (DxeRedfishLib & BaseJsonLib).

![UEFI Redfish Driver Layout](https://github.com/tianocore/edk2-staging/blob/UEFI_Redfish/Images/RedfishDriverStack.png?raw=true)

#### Supported Features
  * Protocols
    * EFI RestEx Service Binding Protocol
    * EFI RestEx Protocol
    * Redfish ConfigHandler Protocol
    * Redfish Credential Protocol

  * Configuration Items via UEFI Redfish
    * [ISCSI Boot Keywords](http://www.uefi.org/confignamespace).
    * HII Opcodes/Questions marked with REST_SYTLE flag or in REST_SYTLE formset.
    * BootOrder/BootNext variables.

  * Redfish Schemas
    * [AttributeRegistry](https://redfish.dmtf.org/schemas/v1/AttributeRegistry.v1_1_0.json)
    * [ComputerSystemCollection](https://redfish.dmtf.org/schemas/ComputerSystemCollection.json)
    * [ComputerSystem](https://redfish.dmtf.org/schemas/v1/ComputerSystem.v1_5_0.json)
    * [Bios](https://redfish.dmtf.org/schemas/v1/Bios.v1_0_2.json)
    * [BootOptionCollection](https://redfish.dmtf.org/schemas/BootOptionCollection.json)
    * [BootOption](https://redfish.dmtf.org/schemas/BootOption.v1_0_0.json)

    If any additional Redfish Schema or a new version of above Schemas are required to be supported, please send the email to edk2-devel mailing list by following [edk2-satging process](https://github.com/tianocore/edk2-staging).

#### Related Modules
  The following modules are related to UEFI Redfish solution, **RedfishPkg** is the new package to support UEFI Redfish solution:
  * **RedfishPkg\RestExDxe\RestExDxe.inf** - UEFI driver to enable standardized RESTful access to resources from UEFI environment.

  * **RedfishPkg\Library\DxeRedfishLib** - Library to Create/Read/Update/Delete (CRUD) resources and provide basic query abilities by using [URI/RedPath](https://github.com/DMTF/libredfish).

  * **RedfishPkg\Library\BaseJsonLib** - Library to encode/decode JSON data.

  * **RedfishPkg\RedfishConfigDxe\RedfishConfigDxe.inf** - UEFI driver to execute registered Redfish Configuration Handlers:

    * **RedfishPkg\Features\RedfishBiosDxe\RedfishBiosDxe.inf** - DXE driver to register Redfish configuration handler to process "Bios" schema and "AttributeRegistry" schema.

    * **RedfishPkg\Features\Features\RedfishBootInfoDxe\RedfishBootInfoDxe.inf** - DXE driver to register Redfish configuration handler to process Boot property defined in "ComputerSystem" schema.

  * Platform Components for NT32:
    * **Nt32Pkg\RedfishPlatformDxe\RedfishPlatformDxe.inf** - UEFI sample platform driver for NT32 to fill the SMBIOS table 42 and publish Redfish Credential info.

    * **Nt32Pkg\Application\RedfishPlatformConfig\RedfishPlatformConfig.inf** - UEFI application for NT32 to publish Redfish Host Interface Record.

  * Misc
   * BaseTools - VfrCompile changes to support Rest Style Formset/Flag.

   * MdePkg - Headers related to Rest Style Formset/Flag.

   * MdeModulePkg - Extract more general APIs in UefiHiiLib & DxeHttpLib & DxeNetLib.

   * NetworkPkg -  1) UefiPxeBcDxe & HttpBootDxe: Consume new APIs defined in DxeHttpLib & DxeNetLib. 2) HttpDxe: Cross-Subnet support. 3) IScsiDxe: REST Style FORMSET support.

   * Nt32Pkg - 1) Enable UEFI Redfish feature in NT32 platform. 2) Fix TLS build error with CryptoPkg from edk2-stable201811 tag.

   * Profile Simulator - 1) A guidance and a patch to enable UEFI support for Redfish Profile Simulator. 2) An user guide for how to configure UEFI Redfish through Postman (**RedfishTool\PostmanToConfigUefiRedfish UserGuide\UserGuide.md**).

## Delivery of FW authentication information to UEFI Redfish
The platform using this Redfish solution need to have a platform driver to install Redfish Credential Protocol (see RedfishPkg/Include/Protocol/RedfishCredential.h) to allow UEFI firmware to get firmware-BMC authentication credential for use, instead of using the “RedfishFWCredentials” variable defined in DSP0270, in order to avoid storing the firmware-BMC authentication credential into any insecure storage.

The EFI_REDFISH_CREDENTIAL_PROTOCOL.StopService() is defined to notify BMC Redfish service to stop provide configuration service to this platform. This interface shall be called when the platform is about to leave an safe UEFI environment. Upon receiving such notification, the BMC shall stop providing the authentication credential to UEFI firmware, close all existing Redfish session with the original credential, and reject any further Redfish login request from UEFI firmware until next platform reset.

## Promote to edk2 Trunk
If a subset feature or a bug fix in this staging branch could meet below requirement, it could be promoted to edk2 trunk and removed from this staging branch:
* Meet all edk2 required quality criteria.
* Support both IA32 and X64 Platform.
* Ready for product integration.

## Related Materials
1. DSP0270 - Redfish Host Interface Specification, 1.0.1

2. DSP0266 - Redfish Scalable Platforms Management API Specification, 1.5.0

3. UEFI Configuration Namespace Registry - http://www.uefi.org/confignamespace

4. Redfish Schemas - https://redfish.dmtf.org/schemas/v1/

5. UEFI Specification - http://uefi.org/specifications

# EDK2 Redfish POC Code re-architecture Project
Intel Redfish POC Code Rearchitecture.pdf (under branch root) is uploaded as the reference. This proposal has discussed in TianoCore design meeting and the goal of re-architecture project is to make edk2 Redfish support more flexible and easier to be integrated to OEM edk2 code base.
HPE is making code changes on this branch based on the proposal.

![UEFI Redfish final Solution Layout](https://github.com/tianocore/edk2-staging/blob/UEFI_Redfish/Images/FinalSolution.jpg?raw=true)

## Re-architecture of EFI Redfish Driver Stacks
Below are items of UEFI Redfish re-architecture project,

### EFI Redfish Host Interface DXE driver
The abstract DXE driver to create SMBIOS type 42 record through EFI SMBIOS protocol according to SMBIOS type 42h device descriptor and protocol type data provided by PlatformHostInterfaceLib. On edk2 open source implementation, SMBIOS type 42 data is retrieved from EFI variable which is created using RedfishPlatformConfig.efi under EFI shell. OEM may provide itsown PlatformHostInterfaceLib instance for platform-specific implementation.
***UEFI spec ECR is required***

### EFI Refish Credential DXE driver
Generic EFI Redfish Credential DXE driver which incoporates with RedfishPlatformCredentialLib to acquire the Redfish service credential. On edk2 open source implementation, the credential is hardcoded using the fixed Account/Password. OEM may provide itsown RedfishPlatformCredentialLib instance for platform-specific implementation.
***UEFI spec ECR is required***

### EFI REST EX UEFI driver for Redfish service
Refine EFI REST EX UEFI driver for Redfish service. 

### EFI Redfish Discover UEFI driver
EFI Redfish Discover Protocol implementation (UEFI spec 2.8, section 31.1). Only support Redfish service discovery over Redfish Host Interface.

### EFI Redfish Config UEFI driver
Refine EFI Redfish Config UEFI driver, use EFI Redfish Discover Protcol to discover Redfish service and remove the dependencies with SMBIOS Redfish Host Interface.

### EFI Redfish Feature drivers
Refine EFI Redfish Feature drivers, remove the dependencies of SMBIOS record type 42. EFI REST protocol instance and Redfish service information is passed to EFI Redfish feature drivers from EFI Redfish Config UEFI driver.

### EFI REST JSON Structure DXE driver
EFI REST JSON Structure DXE implementation (UEFI spec 2.8, section 29.7.3).

### EFI Redfish REST JSON C Structure drivers
EFI Redfish REST JSON to C Structure convertor implementations (UEFI spec 2.8, section 31.2).

# Timeline
| Time | Event | Related Modules |
|:----:|:-----:|:--------------:|
| 2019.01 | Initial open source release of UEFI Redfish feature. | Refer to "Related Modules" |
| 2019.08 | UEFI Redfish feature re-architecture proposal in Tianocore design meeting | |
| 2019.10 | Final solution of UEFI Redfish feature| Refer to above figure| |
| 2020.01 (planed)| Contribute UEFI Redfish re-architecture code to edk2-staging| |
