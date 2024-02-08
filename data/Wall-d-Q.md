duplicate implementation of impl block for u128 

``
impl<AssetId> From<AssetAmount<AssetId>> for u128 {
	fn from(value: AssetAmount<AssetId>) -> Self {
		value.amount
	}
}
impl<AssetId> From<&AssetAmount<AssetId>> for u128 {
	fn from(value: &AssetAmount<AssetId>) -> Self {
		value.amount
	}
}``

https://github.com/code-423n4/2024-02-hydradx/blob/main/HydraDX-node/pallets/stableswap/src/types.rs#L87-L96