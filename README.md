# pallet-xcm-ibc
`pallet-xcm-ibc` implements the IBC module and relevant XCM interfaces to send and receive IBC packets carrying XCM messages.


## Design

- Tries to maintain ordered delivery of XCM messages as per standard.
- Ensures, as far as it possible, that message will be delivered as per design of XCM executor
- Initial pallet design assumes well-working IBC relayers on per message

*NOTE: batching and grouping messages for better XCM order and IBC timeouts handling can be done later*


## Opening IBC channel

The pallet implements IBC [trait Module](https://github.com/ComposableFi/centauri/blob/master/ibc/modules/src/core/ics26_routing/context.rs).
It handles opens `ORDERED` channel handshake protocol.

*NOTE: there is design which will maintain XCM [absolutes](https://substrate.stackexchange.com/questions/6831/how-does-the-xcvm-architecture-ensure-the-absoluteness-principle-described) on top of ORDERED_ALLOW_TIMEOUT and UNORDERED, which can be done later`

XCM export `channel` prefixed with pallet instance name is used as IBC `port` name to open.

XCM version is used as IBC `version` value.

## Sending XCM message over IBC

To send a message over IBC, this pallet implements [trait ExportXcm](https://github.com/paritytech/polkadot/blob/master/xcm/xcm-executor/src/traits/export.rs).

`validate`:
- Checks the existence of XCM `channel` which is `IBC` port
- IBC connection opened
- Validates that genesis `NetworkId` is equal to genesis in IBC `connection`
- Checks if any of the relayers are eager to transfer the message for its `weight` and `timeout`, either it errors.
- It locks some relayers amounts who were eager to deliver the message.

`deliver`:
- `HandlerMessage.SendPacket` with relevant origins and encoded XCM message (direct send used only for `ORDERED` channels)

## Receiving XCM message

When an IBC packet is received, it is parsed into a versioned exported data and XCM message. 

If desired message weight `trait ExecuteXcm::execute_xcm` fits the budget, message is executed.

If message is overweight, execution fails.


On `on_recv_packet` if XCM message executed sucssefully, sends success `acknowledge`. 
In case of logical failure of transaction success `acknowledge` is written. Examples priviledge escalation, transaction format or not enough assets.
Different kind of error is sent in case of general execution failure. Examples, failed to parse XCM message or channel is non operational.
Whole XCM execution is in one transaction.
If XCM message contained `Query` and response is ready immidiately, response will be sent in ackowlegment.

On `on_acknowledgement_packet`, this pallet removes outgoing XCM message from sent queue with success event in case of success.
In case of fail, invokes relevant callback and writes fail event.

On `on_timeout_packet`, it callbacks to runtime provided callback and removes message from storage. With error event.

Pallet implementes weight and message limits similar to Cumulus `queue` pallets for DMP and XCMP.

This store XCM version of each channel opened. On receive of IBC message

It uses Unordered IBC channels.

Only XCM V3 and onwards supported.

Generally this pallet follows security rules, for exampled for decoding and weights, similar to what `xcmp-queue` and `dmp-queue` do.

Sends SCAL encoded XCM messages

`NOTE: later in can ease integration of XCM when ProtoBug encoding of XCM is available`

```rust
	#[pallet::config]
	pub trait Config: frame_system::Config {
		type RuntimeEvent: From<Event<Self>> + IsType<<Self as frame_system::Config>::RuntimeEvent>;

		type XcmExecutor: ExecuteXcm<Self::RuntimeCall>;

		/// Information on the avaialble XCMP channels.
		type ChannelInfo: GetChannelInfo;

		type OnFailAck: OnFailAck;
    
                type OnTimeout: OnTimeout;

		/// The origin that is allowed to execute overweight messages.
		type ExecuteOverweightOrigin: EnsureOrigin<Self::RuntimeOrigin>;

		/// The origin that is allowed to resume or suspend the XCMP queue.
		type ControllerOrigin: EnsureOrigin<Self::RuntimeOrigin>;
    
		/// The weight information of this pallet.
		type WeightInfo: WeightInfo;
		
		type AdvertisedXcmVersion: Get<XcmVersion>;
	}
``` 
