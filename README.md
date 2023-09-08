# 作业五 POE 存证

## 创建存证

代码片段：

```rust
#[pallet::call_index(0)]
#[pallet::weight(0)]
pub fn create_claim(
	origin: OriginFor<T>,
	claim: BoundedVec<u8, T::MaxClaimLength>,
) -> DispatchResultWithPostInfo {
	let sender = ensure_signed(origin)?;
	ensure!(!Proofs::<T>::contains_key(&claim), Error::<T>::ProofAlreadyExist);
	Proofs::<T>::insert(
		&claim,
		(sender.clone(), frame_system::Pallet::<T>::block_number()),
	);
	Self::deposit_event(Event::ClaimCreated(sender, claim));
	Ok(().into())
}
```

## 撤销存证

代码片段：

```rust
#[pallet::call_index(1)]
#[pallet::weight(0)]
pub fn revoke_claim(
    origin: OriginFor<T>,
    claim: BoundedVec<u8, T::MaxClaimLength>,
) -> DispatchResultWithPostInfo {
    let sender = ensure_signed(origin)?;
    let (owner, _) = Proofs::<T>::get(&claim).ok_or(Error::<T>::ClaimNotExist)?;
    ensure!(owner == sender, Error::<T>::NotClaimOwner);
    Proofs::<T>::remove(&claim);
    Self::deposit_event(Event::ClaimRevoked(sender, claim));
    Ok(().into())
}
```

## 转移存证

代码片段：

```rust
#[pallet::call_index(2)]
#[pallet::weight(0)]
pub fn transfer_claim(
    origin: OriginFor<T>,
    destination: T::AccountId,
    claim: BoundedVec<u8, T::MaxClaimLength>,
) -> DispatchResultWithPostInfo {
    let sender = ensure_signed(origin)?;
    let (owner, _) = Proofs::<T>::get(&claim).ok_or(Error::<T>::ClaimNotExist)?;
    ensure!(owner == sender, Error::<T>::NotClaimOwner);
    ensure!(owner != destination, Error::<T>::DestinationIsClaimOwner);
    Proofs::<T>::remove(&claim);
    Proofs::<T>::insert(
        &claim,
        (destination.clone(), <frame_system::Pallet<T>>::block_number()),
    );
    Self::deposit_event(Event::ClaimTransferred(sender, destination, claim));
    Ok(().into())
}
```
