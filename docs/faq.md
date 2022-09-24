
### Where can I find help if I have further questions?
You can join our [Discord](https://discord.gg/6GTJBkZA9Q) server. In case you have issues with bridge, feel free to create issue in github.

### Can you build an implementation for my project?
We don't currently have capacity to build implementations of ChainBridge. However, you can jump into the relevant channel on our Discord to find other teams who want to work on the same implementation as you.

### Can you run a relayer for my project?
Our focus right now is on building software to enable other teams to run bridges. At this time we are not looking to run relayers for any projects or bridge implementations.


### I want to try out ChainBridge locally, how can I do that?
You can follow these instructions to run an [Ethereum <> Substrate bridge locally](local.md)

### I'm having issues running ChainBridge with Ganache
ChainBridge is not compatible with Ganache. Please try running Geth or any real node instead.

### Update relayer events for your subsrate chain
If you integrate the chainbridge pallet (in octopus pallets project ) into your substrate app chain, you need to update the chainbridge relayer events based on your custom events in your substrate app chain, like [octopus pallets events](https://github.com/octopus-network/ChainBridge/blob/oct-dev/shared/substrate/events.go#L186-L203):
```golang
type Events struct {
	types.EventRecords
	events.Events
	cEvents
	Erc721_Minted            []EventErc721Minted            //nolint:stylecheck,golint
	Erc721_Transferred       []EventErc721Transferred       //nolint:stylecheck,golint
	Erc721_Burned            []EventErc721Burned            //nolint:stylecheck,golint
	Example_Remark           []EventExampleRemark           //nolint:stylecheck,golint
	Registry_Mint            []EventRegistryMint            //nolint:stylecheck,golint
	Registry_RegistryCreated []EventRegistryRegistryCreated //nolint:stylecheck,golint
	Registry_RegistryTmp     []EventRegistryTmp             //nolint:stylecheck,golint
    // add octopus pallets events
    //------------
	ChainBridgeAssets_Created             []types.EventAssetCreated             //nolint:stylecheck,golint
	ChainBridgeAssets_Issued              []types.EventAssetIssued              //nolint:stylecheck,golint
	ChainBridgeAssets_Transferred         []types.EventAssetTransferred         //nolint:stylecheck,golint
	ChainBridgeAssets_Burned              []types.EventAssetBurned              //nolint:stylecheck,golint
	ChainBridgeAssets_TeamChanged         []types.EventAssetTeamChanged         //nolint:stylecheck,golint
	ChainBridgeAssets_OwnerChanged        []types.EventAssetOwnerChanged        //nolint:stylecheck,golint
	ChainBridgeAssets_Frozen              []types.EventAssetFrozen              //nolint:stylecheck,golint
	ChainBridgeAssets_Thawed              []types.EventAssetThawed              //nolint:stylecheck,golint
	ChainBridgeAssets_AssetFrozen         []types.EventAssetAssetFrozen         //nolint:stylecheck,golint
	ChainBridgeAssets_AssetThawed         []types.EventAssetAssetThawed         //nolint:stylecheck,golint
	ChainBridgeAssets_Destroyed           []types.EventAssetDestroyed           //nolint:stylecheck,golint
	ChainBridgeAssets_ForceCreated        []types.EventAssetForceCreated        //nolint:stylecheck,golint
	ChainBridgeAssets_MetadataSet         []types.EventAssetMetadataSet         //nolint:stylecheck,golint
	ChainBridgeAssets_MetadataCleared     []types.EventAssetMetadataCleared     //nolint:stylecheck,golint
	ChainBridgeAssets_ApprovedTransfer    []types.EventAssetApprovedTransfer    //nolint:stylecheck,golint
	ChainBridgeAssets_ApprovalCancelled   []types.EventAssetApprovalCancelled   //nolint:stylecheck,golint
	ChainBridgeAssets_TransferredApproved []types.EventAssetTransferredApproved //nolint:stylecheck,golint
	ChainBridgeAssets_AssetStatusChanged  []types.EventAssetAssetStatusChanged  //nolint:stylecheck,golint
    //------------
}
```