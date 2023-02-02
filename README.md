# pallet-xcm-ibc

This pallets subscribes to IBC packets to `IBC-XCM` port.

Upon recive, it parses version XCM message out of it and sends for execution into `xcm-executor`.

Pallet implementes IBC queue to IBC bridge. All IBC messages routed via it depending on relevant origin and send IBC packets.

Uses Centauri.
