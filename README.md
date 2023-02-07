# pallet-xcm-ibc

`pallet-xcm-ibc` imlementes IBC module and relevant XCM interfaces to send and receive IBC packets carrying XCM messages.

In order to send message over IBC, it imlementes `MessageExporter`.
On receive of IBC packet, it parses XCM message and calls `XcmExecutor::execute`.

It imlementes [Centauri](https://github.com/ComposableFi/centauri/) IBC `Module` interface to allow open channels with other IBC modules. 
Each port/channel is mapped to `XcmMultilocation` `origin`.

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

