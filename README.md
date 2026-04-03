# RP2350 TFTP Client PoC

## Overview

This project implements a basic TFTP client on an RP2350 board connected to a Wiznet W5500 SPI Ethernet adapter.

- Only IPv4 is supported — no VLAN support.
- Goal: demonstrate that firmware stored in a SPI‑NOR (such as BMC code or ROM) can be validated and updated before system startup using low-cost microcontrollers, Ethernet, and industry-standard protocols from a high-level language (Go) without human intervention.

## Hardware Setup

Please refere to HARDWARE.md as to have a clearer view at how to assemble your setup

This Proof of Concept (PoC) relies on a microcontroller connected to:

- 1× SPI-to-Ethernet adapter (example: Wiznet W5500)
- 1× SPI MUX to bridge a SPI‑NOR between the microcontroller and a target system

Process flow:

1. The microcontroller boots and issues:
   - DHCP request
   - ARP request
   - TFTP file request
2. (Not yet implemented) Compare the received TFTP file with the SPI‑NOR content.
3. On mismatch, update the SPI‑NOR content.
4. When finished, enter DeepSleep mode and wait for the next reset.
5. Switch the SPI MUX control to the main system.

## Roadmap / Protocol Flow

- Use DHCP, ARP, and TFTP to download a secondary-stage firmware.
- Reset the microcontroller and execute the secondary firmware from SRAM.
- Secondary firmware will retrieve SPI‑NOR content via HTTP (TCP) — secure or unsecure — and update SPI‑NOR as needed.
- Intention: avoid integrating a TCP stack in immutable uC ROM to reduce attack surface; keep the boot ROM handling only simple protocols.

## Requirements

- DHCP server that assigns an IP address and provides a bootfile name (DHCP lease time is currently ignored by the client; ensure leases are long enough for the TFTP transfer).
- TFTP server reachable via the default gateway (this will later be changed to use DHCP Option 66/67).
- The client logs the start and end of the TFTP transfer.

Sample servers:
- TFTP server example: https://github.com/vejmarie/golang-tftp-example
- DHCP server example: https://github.com/coredhcp/coredhcp.git

## Build & Flash

### Environment

- Requires TinyGo.
- Use the `tasks` scheduler for RP2350:
  - The default multicore scheduler has a race condition that can deadlock when the garbage collector runs (observed during multiple transfers). The `tasks` scheduler is more multicore-friendly and avoids this issue.

### Performance Notes

- Use `-opt=2` for best performance.
- This PoC uses a TFTP block size of 1400 bytes (default TFTP block size is 512 bytes). Ensure your TFTP server supports this block size.
- Peak W5500 TFTP throughput is roughly 1.6 MB/s.

### Flash command

```bash
tinygo flash -monitor -target pico2 \
  -scheduler tasks \
  -opt=2 \
  -stack-size=32KB ./main.go
```

## Development Notes

- Code is partially written using AI-assisted tools.
- Primary goal: proof-of-concept for firmware validation and update prior to system boot.
- Secondary-stage HTTP/TCP update phase will be implemented in SRAM-executed firmware to avoid adding TCP complexity to immutable boot ROM.

## Security Considerations

- Minimize the code and protocols in immutable boot ROM to reduce attack surface.
- Use simple, well-understood protocols (DHCP, ARP, TFTP) for the initial boot-stage operations.
- The more complex TCP/HTTP logic will run in secondary firmware loaded into SRAM.

## License

MIT. See LICENSE.md file for details.

## Acknowledgements

- Wiznet W5500 hardware
- Rapsberry Pi Pico(2) hardware
- Hewlett Packard Enterprise
