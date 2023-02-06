# pallet-xcm-ibc

`pallet-xcm-ibc` imlementes IBC module and relevant XCM interfaces to send and receive IBC packets carrying XCM messages.

In order to send message over IBC, it imlementes `MessageExporter`.
On receive of IBC packet, it parses XCM message and calls `XcmExecutor::execute`.

It imlementes Centauri IBC module interface to allow open channels with other IBC modules. 
Each port/channel is mapped to `origin`.
